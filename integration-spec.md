# Integration spec

The proposed design for "an API we can use for our own app or Home Assistant", given what [research/01](research/01-current-state.md) found (no official command API, read-only CarData) and the routes ranked in [research/02](research/02-options.md). Status: **proposal**, waiting on [Questions](Questions.md) round 1.

## Shape

Two halves, deliberately separate because they have different sources and different failure modes.

**State: CarData.** A small service holds the CarData tokens (refreshing hourly, never letting the two-week refresh token lapse), keeps an MQTT connection to BMW's stream, and republishes each key to the local Mosquitto broker as `bmw/<VIN>/<key>`. Home Assistant reads these as MQTT sensors (or uses `bmw-cardata-ha` directly, which does the same thing); our own app reads the same topics or a tiny REST endpoint over Tailscale. REST calls to BMW are reserved for startup and explicit refreshes to stay under 50 per day.

**Commands: a bridge with pluggable backends.** One interface, three verbs to start with: `precondition`, `lock`, `unlock`. Each verb is dispatched to the first available backend:

| Backend | How a command reaches the car | Unattended? |
|---|---|---|
| Mac Shortcuts | `ssh mac shortcuts run "BMW Precondition"` | yes, if My BMW runs on the Mac |
| iPhone Shortcuts | HA sends an email or iMessage matching a personal automation on the phone | yes |
| Alexa | HA `alexa_media_player` sends "ask BMW to ..." to an Echo | yes |
| Fob relay | ESPHome switch on an ESP32 wired to a spare fob | yes, at home only |

Home Assistant sees three `button` entities regardless of backend. Our own app calls the same bridge, or on the phone itself just runs the Shortcut via `shortcuts://run-shortcut?name=...`.

**Confirmation.** No backend reports success reliably, so a command is confirmed by watching CarData for the matching state change (lock state, `climatization` status) and timing out after a few minutes. Where CarData is unavailable, commands are fire-and-forget.

## Out of scope

Reverse engineering the My BMW app's API ([research/02 §F](research/02-options.md#f-reverse-engineering-the-mybmw-app-rejected)). Charge control, which BMW reserves for approved partners.

## Open

- Market and vehicle (Q1, Q4) decide whether CarData is available at all.
- Which backend to prototype first (Q2).
- Whether code gets a repo now (Q3).
