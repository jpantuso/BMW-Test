# 02. Options for getting an API back

What we want, per the [charter](../Charter.md): something our own app or Home Assistant can call to precondition, lock and unlock, plus enough state to make decisions. [01](01-current-state.md) establishes that BMW offers no such thing. These are the routes that remain, ranked.

## Summary

| # | Route | Commands | State | Needs | Risk | Verdict |
|---|---|---|---|---|---|---|
| A | **My BMW Shortcuts actions, triggered remotely** | climate, trunk; lock/unlock likely | via CarData (EU) | an always-available iPhone (or Apple silicon Mac, unverified) | low: BMW's own client | **Build first** |
| B | **Alexa skill driven by Home Assistant** | climate, lock, unlock, horn, lights, remote start | door/lock status by voice reply only | Echo account, `alexa_media_player` custom integration | low-medium: unofficial HA side, skill status unverified | **Build second, as fallback** |
| C | **CarData for state** | none | full | EU/UK account (US unclear) | none | **Use for the read side** |
| D | Spare key fob + ESP32 relay | lock, unlock, remote start where the fob has it | none | spare fob, soldering, car within fob range | low; local only | Good for "at home" only |
| E | Aftermarket telematics module with its own API | lock, unlock, remote start | some | install in car | voids nothing official but invasive; per-model fit unverified | Only if A and B fail |
| F | Reverse-engineer the MyBMW app API again | everything | everything | app attestation bypass | high: ToS, account suspension, breaks at BMW's whim | **Do not pursue** |
| G | Approved partner API | charge start/stop | charging | being a business partner, Europe | none | Not applicable |

## A. My BMW Shortcuts actions (recommended command path)

Since app 6.5 the My BMW app exposes actions to Shortcuts ("Start Climate Control", "Open Trunk"), and owners report Siri handling "lock my BMW" and "unlock my BMW" directly. Because these run inside BMW's own app on a real iPhone, they pass whatever attestation BMW enforces. The work is getting a remote trigger to a Shortcut.

Ways to fire a Shortcut from outside the phone, all native iOS:

1. **Personal automation on a message or email trigger.** iOS Shortcuts can run an automation immediately (no confirmation) when a message from a given sender containing given text arrives, or when an email matching sender/subject arrives. Home Assistant sends the email (SMTP notify) or iMessage; the phone runs "Start Climate Control". This is the simplest working bridge and needs no code at all.
2. **Our own iOS app.** An app can open `shortcuts://run-shortcut?name=Precondition` to run a named Shortcut. It foregrounds Shortcuts briefly. Fine for a button in our own app; useless for unattended automation.
3. **Apple silicon Mac running the iPhone app.** If BMW allows My BMW on the Mac App Store (unverified: see [Tasks-Joe](../Tasks-Joe.md)), its intents may be available to Shortcuts on the Mac, and `shortcuts run "Precondition"` from a shell gives Home Assistant an SSH-callable, unattended command. This would be the cleanest version of route A. Many vendors opt their iPhone apps out of the Mac, so this may simply be unavailable.
4. **Time, location, alarm and NFC triggers.** Not remote, but they cover the most common real use (precondition at 7:40 on weekdays, or when the "Leave for work" alarm is dismissed) with no bridge at all.

Caveats to verify on a real phone: which actions the current app version exposes (the list depends on vehicle and region), whether unlock asks for Face ID when run from an automation (Apple lets apps mark an intent as requiring an unlocked device, and BMW would be expected to do that for unlock), and whether the action returns success/failure to the Shortcut.

## B. Alexa skill driven by Home Assistant

The My BMW Alexa skill supports "ask BMW to activate the climate control", lock, unlock (with a voice PIN), horn, lights and remote start, in the US, UK and Germany. The `alexa_media_player` custom integration for Home Assistant can send a typed command to an Echo device as if spoken, so HA can say "ask BMW to start climate control" headlessly, with no phone involved. Two unknowns: whether BMW's skill still works after the 2025 changes (it should, being BMW's own cloud-to-cloud service), and how long `alexa_media_player`'s login keeps working (it is itself unofficial and periodically breaks when Amazon changes things). Unlock by voice requires a PIN, which is fine for automation since it is just text.

## C. CarData for state

For the read side, CarData is the right answer where it exists: official, free, with an MQTT stream that Home Assistant can consume either through [`kvanbiesen/bmw-cardata-ha`](https://github.com/kvanbiesen/bmw-cardata-ha) or a plain MQTT bridge, and that our own app can consume directly. The 50 REST calls per day matter only for the initial state and on-demand refreshes; the stream does the rest. **In the US it is not self-service as of this writing** (the HA integration lists the US portal as "needs testers or temp access"), and the US "CarData" from 2020 is a consent and reporting page, not this API. If the car is a US car, the read side has no official answer yet, and the Alexa skill's spoken status replies are the only fallback.

Swift sample code for the full CarData auth and REST flow is in [03](03-cardata-swift-sample.md).

## D. Spare key fob on a relay

The classic local hack: open a spare fob, wire its lock/unlock (and, on US cars with remote engine start, the start sequence) buttons to optocouplers or relays driven by an ESP32 running ESPHome, and power the fob from the board. Home Assistant gets local buttons with no cloud at all. Limits: works only within fob range (so a garage or driveway), a spare fob from BMW costs a few hundred dollars, and if the fob is stolen from the box it is a key to the car. Climate is reachable only on cars where the fob's remote start also runs climate.

## E. Aftermarket telematics

Remote-start and telematics modules (for example Fortin or iDatalink hardware with a cellular companion such as DroneMobile) speak to the car over CAN and expose their own cloud, some of which have community Home Assistant integrations. Support is per model and year, installation is invasive on a modern BMW, and the companion APIs are themselves usually unofficial. Listed for completeness; not investigated further unless A and B both fail.

## F. Reverse engineering the MyBMW app (rejected)

Technically this is the only route that gives a full, phone-free API. It is rejected because the block is app-integrity based ([01](01-current-state.md)), so a working client means extracting secrets from or instrumenting BMW's app, which violates BMW's terms, risks the ConnectedDrive account, and breaks every time BMW rotates its checks. The best-staffed community effort (`bimmer_connected`, used by thousands of HA installs) concluded it was not worth doing and archived the project on 2026-03-09.

## Recommended shape

The design that falls out of this is in [`integration-spec.md`](../integration-spec.md): a small **command bridge** with a single interface (`precondition`, `lock`, `unlock`) and pluggable backends (Shortcuts on a phone or Mac, Alexa, fob), plus **CarData** for state, exposed to Home Assistant as MQTT entities and to our own app as a Swift package.
