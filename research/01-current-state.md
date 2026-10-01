# 01. The current state of BMW remote access

**As of 2026-10-01.** Short version: the API the MyBMW app uses still exists and still does everything (climate, lock, unlock, horn, lights, charging), but since 2025-09-29 BMW rejects every client that is not its own app. The official replacement, BMW CarData, is read-only and fully self-service only in the EU and UK. There is no official way for an individual to send a command to a BMW from their own code.

## Timeline

| Date | What happened | Source |
|---|---|---|
| 2020-06-29 | BMW of North America launches "BMW CarData" in the US. This is a data-transparency feature (a report of what the car has sent BMW, and consent for named third parties), not a developer API. | [BMW USA press release](https://www.press.bmwgroup.com/usa/article/detail/T0310166EN_US/bmw-cardata:-secure-and-private-control-of-vehicle-data-for-customers) |
| 2024-11 (approx.) | BMW adds hCaptcha to the MyBMW login. `bimmer_connected` (the Python library behind Home Assistant's integration) starts requiring a captcha token on first login. | [bimmer_connected docs](https://bimmer-connected.readthedocs.io/en/stable/), evcc maintainer comment in [evcc #24123](https://github.com/evcc-io/evcc/discussions/24123) |
| 2025-07-31 | MyBMW app pushes a notice: from September 2025, remote charge control will only work through approved electricity providers; other integrations will be blocked "for security reasons". | [HA core #149750](https://github.com/home-assistant/core/issues/149750), [evcc #22643](https://github.com/evcc-io/evcc/discussions/22643) |
| 2025-08-30 to 09-03 | Aggressive quota on the MyBMW API (403 "Out of call volume quota", roughly 100 calls per 24 hours). Spoofing the app's `x-user-agent` header briefly worked around it. | [HA core #152646](https://github.com/home-assistant/core/issues/152646), [Consumer Rights Wiki](https://consumerrights.wiki/w/BMW_API_restrictions) |
| 2025-09-12 | EU Data Act applies. BMW opens the CarData Customer Portal (REST + MQTT stream) to private users in the EU and UK. Read-only. | [bimmer_connected #745](https://github.com/bimmerconnected/bimmer_connected/discussions/745) |
| 2025-09-26 to 09-29 | Full block. Third-party requests get `HTTPStatusError: Blocked`; header spoofing no longer works. BMW describes it as "additional security checks within the MyBMW app". Energy companies using customer credentials were cut off too. | [bimmer_connected #749](https://github.com/bimmerconnected/bimmer_connected/issues/749), [HA alert](https://alerts.home-assistant.io/alerts/bmw_connected_drive/) |
| 2025-10-03 | Home Assistant publishes the "BMW APIs blocked" alert. The `bmw_connected_drive` core integration was later removed. | [HA alert](https://alerts.home-assistant.io/alerts/bmw_connected_drive/), [HA integration page](https://www.home-assistant.io/integrations/bmw_connected_drive/) |
| 2025-12-03 | Kaluza announces a direct, BMW-sanctioned smart-charging API link in the UK. Same pattern as Gridio, Octopus and others: approved business partners get charge control, nobody gets lock or climate. | [electrive](https://www.electrive.com/2025/12/03/kaluza-and-bmw-introduce-direct-smart-charging-link-for-uk-drivers/), [Gridio](https://www.gridio.io/blog/gridio-and-bmw-agree-official-api-integration) |
| 2026-03-09 | `bimmer_connected` repository archived, marked "Deprecated/Non-functional". | [GitHub](https://github.com/bimmerconnected/bimmer_connected) |
| 2026-09-24 | My BMW for iOS, version 6.9.2 (US build, bundle `de.bmw.connected.mobile20.na`, seller BMW North America). The app still offers remote lock/unlock, climate and trunk, plus Siri Shortcuts actions since 6.5. | iTunes lookup API, [App Store](https://apps.apple.com/us/app/my-bmw/id1519457734) |

## How the block works

BMW has not published the mechanism and nobody has published a teardown, so this part is **inference**. The facts that constrain it: the website and the official app keep working with the same account; header spoofing worked during the quota phase and stopped working at the full block; BMW's own wording is "security checks within the MyBMW app". That pattern fits app attestation (Apple App Attest / Google Play Integrity, or a signing key held inside the app binary) rather than anything account-based. Attestation is the kind of check a script cannot pass without either a real, unmodified copy of the app on a real device or extracting and replaying secrets from the binary, which is a moving target BMW can rotate at will.

## What still works, officially

| Channel | Read | Command | Region | Notes |
|---|---|---|---|---|
| **My BMW app** | yes | yes | all | The only full-featured client. Remote services need an active ConnectedDrive / Remote Engine Start entitlement for the vehicle. |
| **My BMW Siri Shortcuts / App Intents** | limited | yes | all (iOS) | Added in app 6.5: start climate, open trunk; Siri lock/unlock reported by owners. These run inside BMW's app, so they pass BMW's checks. See [02](02-options.md). |
| **My BMW Alexa skill** | limited | yes | US, UK, DE | Documented commands: climate, lock, unlock, horn, lights, remote start, door/window status. Whether it survived the 2025 changes is **unverified**; it is BMW's own skill, so the block should not apply to it. |
| **BMW CarData (customer API)** | yes | **no** | EU + UK self-service; US "needs testers" | REST, 50 calls per day per account, plus an MQTT stream with no published limit. Device-code OAuth. Details below. |
| **Approved partner APIs** (Smartcar Energy, Gridio, Kaluza, Octopus, ...) | charging data | start/stop charge only | Europe | Business partners only. Smartcar's BMW Energy API: 17 European markets, BEV/PHEV only, AC charge start/stop capped at 80 commands per 30 days, data only within about 200 m of the consented address. |
| **BMW Labs "Home" in-car app** | - | - | where Labs ships | Connects the car's display to *your* Home Assistant (ETA, garage door, doorbell). BMW says it has "no ability for remote services". It is HA-in-the-car, not car-in-HA. |

## BMW CarData, technically

This is the only official programmatic interface, so it is worth knowing exactly. Taken from BMW's customer documentation as mirrored in [`kvanbiesen/bmw-cardata-ha`](https://github.com/kvanbiesen/bmw-cardata-ha/blob/main/cardata_api_documentation.md) (last commit 2026-09-25), the [Go client](https://github.com/tjamet/bmw-cardata), and the HA integration's source.

- **Setup**: in the regional MyBMW portal, open CarData, generate a Client ID, and subscribe it to "CarData API" and/or "CarData Stream". Then run the device-code flow.
- **Auth**: `POST https://customer.bmwgroup.com/gcdm/oauth/device/code` with `client_id`, `response_type=device_code`, `scope=authenticate_user openid cardata:api:read cardata:streaming:read`, and a PKCE S256 challenge. The user approves at `verification_uri_complete`. Then `POST /gcdm/oauth/token` with `grant_type=urn:ietf:params:oauth:grant-type:device_code` and the verifier.
- **Tokens**: access token (REST, 1 hour), ID token (MQTT password, 1 hour), refresh token (2 weeks; past that the device flow has to be repeated), and `gcid` (account ID, also the MQTT username). Refreshing returns a fresh set and resets the two-week clock, so a running service never has to re-prompt.
- **REST**: `https://api-cardata.bmwgroup.com`, header `x-version: v1`, bearer token. Endpoints: `/customers/vehicles/mappings`, `/customers/containers` (define which telematic keys you want; max 10), `/customers/vehicles/{vin}/telematicData?containerId=...`, `/basicData`, `/chargingHistory`, `/smartMaintenanceTyreDiagnosis`, `/image`. **50 calls per day per account**, and the lockout is account-wide (it also breaks the portal), per [evcc #33544](https://github.com/evcc-io/evcc/issues/33544).
- **Stream**: MQTT over TLS at `customer.streaming-cardata.bmwgroup.com:9000`, username = `gcid`, password = ID token, topic `<VIN>/#`. Keys look like `vehicle.cabin.infotainment.navigation.currentLocation`, `vehicle.drivetrain.lastRemainingRange`, door and lock states, charging power. Updates arrive when the car sends them (owners report every 20 minutes or so, more often while driving or charging), not on demand.
- **Writes**: none. There is no command endpoint in the spec.

## Bottom line

The capability the charter is after (remote climate, lock, unlock from our own code) is no longer exposed by BMW to anyone except its own app, its own voice skills, and approved charging partners. Every workable route to commands goes **through a BMW-owned client** (the app's Shortcuts actions or the Alexa skill) or **around the cloud entirely** (hardware at the car). Reading state is solved in Europe by CarData and is an open question in the US. [02](02-options.md) evaluates the routes.
