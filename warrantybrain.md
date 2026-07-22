# WarrantyBrain: Architecture Notes

*A live Shopify app that turns warranty claims into automated, auditable resolutions, and the claim history into cross-merchant intelligence. Live on the Shopify App Store. Built and operated solo.*

Architecture notes for a technical reader: how the system is built and why. No product source, no thresholds, no exact hashing scheme. The private boundary is at the end.

## The wedge: safe claim automation

The visible product is simple. A claim comes in; WarrantyBrain retrieves the evidence (order, policy clause, photos, serial history) and recommends a resolution (refund, replacement, or return) that the merchant approves in one click. A human stays on every customer-facing action.

The hard part isn't the recommendation. It's making the **money-touching execution** safe enough to run unsupervised once approved. A refund, a replacement order, and a Shopify return are irreversible, side-effectful, and racing against webhook retries and a merchant clicking twice. The execution layer is built around four properties:

- **Idempotency.** A retried webhook can't double-apply: inbound events are deduplicated through a unique event-key ledger, so the same dispute or claim event resolves to exactly one side effect.
- **Terminal-state guards.** Each claim moves through an explicit state machine, enforced at the executor, not the UI. An already-refunded claim can't be refunded again; a fraud-denied claim can't be silently downgraded to "review."
- **Optimistic concurrency across workers.** The app runs multiple workers over one store; safety comes from an atomic status transition (a claim advances only if a compare-and-set on its current state succeeds), so two workers can't both act on the same pending claim. A failed money action stays *retryable* and can never falsely report success.
- **Reversibility + audit.** Every action writes an audit record before and after the Shopify mutation. Every *reversible* action (credit, replacement, return) has a defined undo; the one irreversible action, a refund, is gated for explicit approval before it fires.

The rule underneath: **money and data move one class of change at a time, behind a gate, with an undo.** I certify this by walking the live embedded Shopify admin and firing real mutations against a test store, not by trusting a green unit test.

## The moat: a privacy-preserving cross-merchant fraud network

Automation gets a merchant in the door. What compounds is the intelligence, and warranty fraud is a *network* problem. A serial abuser, a resold serial number, a photo reused across claims, a claimant with a chargeback history: none of these are visible inside one store. They're visible **across** stores. So the interesting system is a fraud network that links signals across merchants **without any merchant ever seeing another's data.**

- A **per-claim fraud score** accumulates deltas from independent detectors: perceptual-hash photo reuse (near-duplicate images match even after re-saving), serial-number duplication/abuse, AI-generated-text heuristics on the claim narrative, and dispute/chargeback history.
- A **cross-store ledger keyed on one-way hashed identifiers**, never raw identities. A store contributes to and queries the network through hashed identifiers and content fingerprints: it learns *"this claimant pattern has been flagged elsewhere"* without learning who, or where.
- A **chargeback correlation loop.** Shopify dispute webhooks feed back into the network, and the system correlates denials to later chargebacks over a rolling window, so the network learns which decisions actually cost money.
- A **decisions engine** that reads the fraud score and gates the automation: low-risk claims flow to one-click resolution; higher-risk claims escalate to human review before any money moves. The engine is the join between the intelligence and the wedge.

The network effect: every resolved claim, every dispute, every flag enriches the shared signal, so the next decision (at *every* store) is a little sharper. Analytics revenue is linear; the network's value compounds.

## Proven vs. not

- **Proven:** the automation and money-path safety are live and certified against a real store. The detectors and the cross-store ledger are built and running.
- **Not proven:** the *network effect at scale*. Cross-merchant intelligence is only as strong as the number of merchants in it, and that density doesn't exist yet. The architecture is designed for it; the flywheel hasn't spun.

## What stays private

The detector weightings, the escalation thresholds, the exact hashing and aggregation scheme, and the product source aren't published here. The value is in the machinery; these notes are about the shape and the reasoning.

---
*Relevance to data operations: this is the same discipline a collection pipeline needs: idempotent ingestion, explicit state, cross-worker safety, and never trusting "it passed" over "it's verified on the live system."*
