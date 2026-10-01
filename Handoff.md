# Handoff: BMW-Test, 2026-10-01

Written after one session. Orientation, not the record; the record is [`STATUS.md`](STATUS.md).

## Where to start

| Want | Read |
|---|---|
| What the project is | [`Charter.md`](Charter.md), then [`README.md`](README.md) |
| The facts | [`research/01-current-state.md`](research/01-current-state.md). Every claim has a dated source; the attestation explanation of the block is labeled as inference and should stay that way until someone publishes a teardown |
| The options | [`research/02-options.md`](research/02-options.md) |
| The design | [`integration-spec.md`](integration-spec.md), a proposal until Questions round 1 is answered |
| What is decided | [`Questions.md`](Questions.md). Round 1 (Q1 to Q4) is open |
| What needs Joe | [`Tasks-Joe.md`](Tasks-Joe.md) |

## Things a fresh agent should know

- **There is no car.** Nothing can be tested end to end. Code is verified by compiling it; behavior claims about the app come from owners' reports and need Joe's phone to confirm.
- **Do not build or recommend a reverse-engineered MyBMW client.** That decision is in research/02 §F with its reasons. CarData and BMW's own clients are the boundary.
- **Code does not live in this folder.** Per Joe's global rules, code goes in `~/Code/<org>/<repo>` with a Forgejo remote. The only code so far is inline in research/03; creating a repo waits on Q3.
- **CarData's 50 calls a day is account-wide**, and exhausting it also locks the owner out of the portal. Any code that polls must count calls.
- The US "BMW CarData" from 2020 is a consent page, not the API. Easy to confuse in search results.

## Suggested skills

- `dataviz`, if a comparison of the options gets charted.
- `home-assistant-skills:home-assistant-best-practices`, before writing any Home Assistant automation, button or MQTT sensor config for the bridge.
- `geomap`, only if CarData location data ever needs inspecting.
- A Swift/SwiftPM workflow (the Shifter/Readulator house pattern in Joe's global instructions) once Q3 creates `BMWKit`.
