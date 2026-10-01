I created this after hearing Casey Liss lament about the lack of BMW control for preconditioning etc. thanks to their closing APIs that used to be available and used by home assistant  plugins. There was some discussion about using Claude to attack the problem but to me the conversation was coming at it from the wrong angle.

This repo is the result of 3m 6s effort by Opus 5.5 in Claude harness. It leverages months of projects I've done from which it has learned how I want it to work with me and vice versa. I treat it like a remote team. Docs in, docs out. The only thing I wrote is the charter.md giving it where to start, and this couple introductory paragraphs. I know learning to delegate and trust is hard for humans with humans, and potentially worse with AI. Bringing a few decades of experience running teams helped a lot.

The conclusion here is a bit of a head smacker; the existing BMW app has hooks!
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
