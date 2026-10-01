Claude you may delete items out of this file once they are complete and your memory updated about them. New questions for me may be placed in a questions document in a format that leaves a space for me to place answers and offers multiple choice when there are clear options.
Once answered questions are implemented they may be removed as well.

——

(Your inbox to me. 2026-10-01: **the old API is gone for good, and the way back in is through BMW's own app.** BMW blocked every non-BMW client from the MyBMW API on 2025-09-29 with what looks like app attestation; Home Assistant pulled its integration and `bimmer_connected` was archived on 2026-03-09. The official replacement, CarData, is real and documented but **read-only**, and self-service only in the EU and UK. Nobody gets lock, unlock or climate over an API anymore except BMW's app, BMW's Alexa skill, and approved charging partners (charging only).

So the plan in [`integration-spec.md`](integration-spec.md) is a command bridge that drives **BMW's own clients**: the My BMW Shortcuts actions on an iPhone (or a Mac, if BMW allows the app there), or the Alexa skill from Home Assistant, with CarData for state where it exists. Start with [research/01](research/01-current-state.md) for the facts and [research/02](research/02-options.md) for the options. A type-checked Swift CarData client is in [research/03](research/03-cardata-swift-sample.md).

[Questions](Questions.md) round 1 has four; Q1 (US or EU car) matters most. [Tasks-Joe](Tasks-Joe.md) has three five-minute checks that would settle the biggest unknowns without a car.)

——

## Queue

- After Q1: if US, watch the CarData portal for US self-service (the `bmw-cardata-ha` README is the canary); if EU/UK, get a Client ID and run research/03 for real.
- After Tasks-Joe 1 and 2: write up the Shortcuts bridge as a step-by-step build, including the email-trigger automation and the Home Assistant notify config.
- After Q2: prototype the chosen backend.
- After Q3 (a): create the Forgejo repo and move the CarData client into `BMWKit` with tests against recorded responses.
