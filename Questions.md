# Open Questions

Questions that arose during research. Fill in answers under each question as decisions are made. Answered rounds move to the bottom once their answers are implemented.

**Round 1 is open.** Q1 and Q4 decide whether the read side exists at all; Q2 and Q3 decide what gets built first.

---

## Round 1, 2026-10-01

**Q1: Which market is the hypothetical car in?** BMW CarData, the only official data API, is self-service in the EU and UK and not yet in the US ([research/01](research/01-current-state.md)). A US car has no official read path today; an EU/UK car does. This also decides whether the Swift sample in [research/03](research/03-cardata-swift-sample.md) can ever be run.

- (a) **US** *(recommended as the assumption, since that is where you are; it makes the command bridge the whole project and CarData a watch item)*
- (b) EU or UK
- (c) Design for both

> **Answer:**

---

**Q2: Which command backend should be prototyped first?** All four are in [research/02](research/02-options.md) and [`integration-spec.md`](integration-spec.md). None can be tested end to end without a car, but A and B can be built to the point of "the right thing reaches BMW's app or skill".

- (a) **Shortcuts on the iPhone, triggered by email or message** *(recommended: BMW's own client, no code needed to prove it, and it is the basis for the Mac variant)*
- (b) Alexa skill via Home Assistant
- (c) Spare fob on an ESP32
- (d) Several in parallel

> **Answer:**

---

**Q3: Should the sample code get a repo now?** House style puts code in `~/Code/<org>/<repo>` backed by Forgejo, never under Drive. The CarData client currently lives inline in [research/03](research/03-cardata-swift-sample.md).

- (a) **Create a Forgejo repo and a SwiftPM package (`BMWKit`: CarData client, MQTT stream, command-bridge protocol) with macOS in platforms so `swift test` runs** *(recommended once Q1 says CarData is reachable; until then there is little to test)*
- (b) Keep samples inline in research documents for now
- (c) Repo plus a thin iOS app shell from the start

> **Answer:**

---

**Q4: Is there a specific model, year and iDrive version in mind?** It changes which Shortcuts actions exist (owners report the newer actions on iDrive 8 and later), whether the fob can remote-start, and whether the car has Remote Engine Start at all in its market. If it is purely hypothetical, I will assume a 2024 or later car on iDrive 8.5 or 9.

> **Answer:**

---

# Answered rounds

None yet.
