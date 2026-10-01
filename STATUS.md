# Status

Session log, newest at the top.

## 2026-10-01, session 1

Read the charter and set up the project in the BigSkyBase layout.

**Verified the current state** ([research/01](research/01-current-state.md)). The MyBMW API still exists and powers the app, but BMW has rejected all non-BMW clients since 2025-09-29, after a quota phase from 2025-08-30 and a July 2025 in-app warning. The mechanism is unpublished; app attestation is the inference, because header spoofing worked during the quota phase and stopped at the block. `bimmer_connected` was archived 2026-03-09 and Home Assistant removed its core integration. BMW CarData (EU Data Act, 2025-09-12) is the official replacement and is read-only, 50 REST calls a day plus an MQTT stream, self-service in the EU and UK only. BMW Labs' Home Assistant app turned out to be HA-in-the-car with no remote services.

**Evaluated the routes** ([research/02](research/02-options.md)). Commands are only reachable through BMW-owned clients (the My BMW Shortcuts actions, the My BMW Alexa skill) or hardware at the car. Reverse engineering the app API was rejected: it means defeating app integrity checks, risks the account, and the best-staffed community project already gave up on it. Chose a two-part design ([`integration-spec.md`](integration-spec.md)): CarData for state, a command bridge with pluggable backends for commands, success confirmed by watching state.

**Sample code.** Wrote a dependency-free Swift CarData client ([research/03](research/03-cardata-swift-sample.md)), type-checked with `swiftc` against the macOS 14 SDK. Not run: no Client ID. Kept inline rather than creating a Forgejo repo, pending Q3.

**Unverified and flagged:** whether My BMW runs on Apple silicon Macs, the exact Shortcuts action list, whether unlock demands Face ID from an automation, whether the Alexa skill still works, and US CarData availability. The first three are Tasks-Joe items.

Opened Questions round 1 (Q1 to Q4).
