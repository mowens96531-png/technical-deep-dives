# Fluxor: Architecture Notes

*A real-world data-collection system for robot learning: the operation that captures skilled physical work, and the pipeline that turns it into training data. One verified session; the collection network is currently paused.*

Architecture notes for a technical reader. No collection operating system, no contact taxonomy, no product source. The private boundary is at the end.

## The thesis

World models need data that barely exists in structured form: multi-modal recordings of skilled physical manipulation (force, motion, contact, intent, outcome). That data is generated constantly by tradespeople and recorded by no one. Fluxor is built to capture it at the source.

Two halves: an **operation** that gets sensors onto real workers on real job sites (recruiting, consent, kit logistics, scheduling, QC, payouts), and a **pipeline** that turns raw capture into robot-learning-ready data. These notes are about the pipeline.

## The pipeline: raw capture → training data

A single session is several unsynchronized streams: multiple IMUs (body- and tool-mounted, so you get both limb motion and tool force/torque), point-of-view video, and the worker's own spoken narration. The pipeline fuses them into one aligned, labeled, quality-graded record.

- **Time synchronization.** The streams start at different moments and drift. Alignment is the foundation (every downstream label depends on the timestamps being right), so this is where the care goes.
- **Fusion at 50 Hz.** The IMU streams are fused onto a common time base with the video and narration into a single resampled multi-modal record.
- **Quality scoring as a gate, not a report.** Each session gets an automated score across sensor completeness, data integrity, sample-rate consistency, and cross-sensor agreement. **Below threshold, a session doesn't ship and the contractor isn't paid full rate.** Aligning the metric with the incentive is half the system: a quality bar that doesn't touch pay gets gamed.
- **Task segmentation.** The continuous session is cut into discrete tasks ("turn off power," "remove the cover screw," "hand-thread the new cover") from the narration and motion.
- **Contact-event weak labeling.** The narration is a free intent channel: a worker saying *"tightening until I feel resistance"* is at once an action annotation, a skill-level signal, and a causal explanation, at zero marginal cost. The pipeline weak-labels candidate contact events (hand ↔ tool ↔ object) by grounding the transcript against the motion.
- **Export in the formats robot-learning teams actually train on:** HDF5, LeRobot, RLDS, Cosmos. Deliberate: the data is only valuable if it drops into an existing training stack without a translation layer.

One verified session ran end to end: ~24,700 processed samples and ~227 candidate contact events, cleared the quality gate, and packaged into the standard training formats (HDF5, LeRobot, RLDS, Cosmos). The session was my own, because I instrument myself before asking anyone else to wear the kit.

## The design calls

- **Narration is the cheapest rich signal, so build around it.** Most capture systems treat audio as an afterthought. Here it's the intent-labeling backbone. It makes the data action-conditioned for free.
- **Tiered capture on purpose.** IMU + narration is cheap and physics-rich; video layers in only where demand justifies the cost. The cheapest tier has to run profitably alone, because demand for the richer tiers isn't guaranteed.
- **The quality gate is the product.** Anyone can sync sensors. A dataset buyers trust needs a gate that rejects bad sessions *and* an incentive structure that doesn't corrupt the labels. That's the unglamorous, load-bearing part.

## What it's built toward

Not a pile of clips. A **contact taxonomy** (a structured vocabulary of how skilled hands meet tools and materials) and the **data layer** that robotics teams, safety products, and the trades' own tools sit on top of. Data sales are the wedge that funds it.

## What stays private

The contact taxonomy, the collection operating system, and the product source aren't published here.

---
*Relevance: this is, structurally, the problem a physical-AI company solves at scale: get sensors onto real people doing real work, and turn messy multi-modal capture into training data buyers trust. The pipeline and the quality-gate discipline are where I'd be useful on day one.*
