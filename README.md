# Michael P. Owens: Technical Deep-Dives

Architecture notes on the systems behind my ventures, written for a technical reader.

I ship production systems by directing AI coding agents under a written verification doctrine, but the architecture, the hard-problem judgment, and the design decisions are mine. These notes are the depth behind the one-line summaries on [michaelpowens.com](https://www.michaelpowens.com): how each system is actually built, the calls I made, and what's proven versus still open.

They deliberately stop at the IP boundary. No product source, no calibration constants, no thresholds, no corpus. Each note says what stays private. The point is the *shape* of the system and the reasoning, not the machinery.

- **[WarrantyBrain](warrantybrain.md)**: safe, idempotent claim automation, and a privacy-preserving cross-merchant fraud network. Live on the Shopify App Store.
- **[Fluxor](fluxor.md)**: multi-sensor fusion and a quality-gated pipeline that turns skilled physical work into robot-learning training data.
- **[BarbellAI](barbellai.md)**: a no-cameras sensing stack (inertial + mmWave radar), a sim-to-real form-perception approach, and a preregistered validation discipline.

Two habits run through all three: **first principles before frameworks** (find the binding constraint, not the loudest problem) and **nothing is done until it's verified on the live system**, where a second set of eyes with fresh context tries to refute every change before it ships. I record failures as verdicts; the honest FAILs are in here too.
