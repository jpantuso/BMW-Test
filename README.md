# BMW-Test

A test research project: can we still get an API for remotely controlling a BMW (precondition, lock, unlock) for our own app or Home Assistant, now that the old one is gone? No car to test with; research and sample code only.

**Started:** 2026-10-01

**→ The brief is [`Charter.md`](Charter.md).**
**→ The answer, with dates and sources, is [`research/01-current-state.md`](research/01-current-state.md): BMW blocked all third-party clients on 2025-09-29, and the official CarData API is read-only.**
**→ What we can do about it is [`research/02-options.md`](research/02-options.md): drive BMW's own clients (Shortcuts actions, Alexa skill) for commands, and use CarData for state.**
**→ The proposed design is [`integration-spec.md`](integration-spec.md).**
**→ Decisions waiting on you are in [`Questions.md`](Questions.md); five-minute checks only you can do are in [`Tasks-Joe.md`](Tasks-Joe.md).**

## Documents

| File | What it is |
|---|---|
| [`Charter.md`](Charter.md) | The original brief |
| [`STATUS.md`](STATUS.md) | Session log, newest first |
| [`Tasks.md`](Tasks.md) | Claude's inbox to Joe and work queue |
| [`Tasks-Joe.md`](Tasks-Joe.md) | Actions only Joe can do |
| [`Questions.md`](Questions.md) | Open decisions, with space for answers |
| [`Handoff.md`](Handoff.md) | Orientation for a fresh agent |
| [`integration-spec.md`](integration-spec.md) | **The proposed design**: CarData for state, a command bridge with pluggable backends |
| [`research/01-current-state.md`](research/01-current-state.md) | **What happened and what works now**: timeline, the block, the official channels, CarData's API in detail |
| [`research/02-options.md`](research/02-options.md) | **Every route to commands**, ranked, with the reasons for rejecting reverse engineering |
| [`research/03-cardata-swift-sample.md`](research/03-cardata-swift-sample.md) | Swift CarData client: PKCE, device-code login, refresh, REST. Type-checked, not run |
