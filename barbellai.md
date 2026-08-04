# BarbellAI: Architecture Notes

*Instrumented strength training: two custom sensors, a no-cameras sensing stack, and the data systems behind them, on the path to a physics ground-truth model for strength. Built solo. Patent pending.*

Architecture notes for a technical reader. The calibration machinery, the corpus, the simulation internals, and product source are the moat and stay private. Boundary at the end.

## The contrarian bet: no cameras

Everyone reaching for "AI form coaching" reaches for a camera. I bet against it, from first principles:

- A camera is **blind in a real gym**: occluded by bodies, racks, and bad angles.
- It makes the lifter **perform for it**: set up the phone, stay in frame. That friction kills the data flywheel before it starts.
- It's a **privacy liability** in a locker-room-adjacent space.

The alternative: a bar-mounted inertial clip and a rack-mounted mmWave radar puck. Radar sees through a crowded room, needs no line-of-sight cooperation, and captures nothing that looks like a photograph. The cost is that radar returns are *harder*: sparse, noisy point clouds and Doppler, not pixels. I would rather solve a hard sensing problem than ship an easy one that doesn't work in the room it lives in.

## The hard problems

- **Bar tracking from noisy sensors.** Inertial dead-reckoning drifts. The fix is anchoring integration to genuine zero-velocity events (the bar is momentarily still when it's racked or set down), a ZUPT-style reset against real stationary contacts instead of fighting the drift. (The rep turnaround is an instantaneous zero-crossing, not a stationary dwell, so it is *not* a usable anchor, a distinction the data forced.)
- **Sensor fusion: the "handshake."** The clip and the radar have to agree they're watching the same bar. I match the clip's acceleration envelope against the radar's motion signature to bind the two streams, a cross-sensor envelope handshake. Isolating the bar's returns from *other people moving in the scene* is the key open problem: a single scene-global radar view can't yet separate the bar from a nearby lifter, which is what the current dual-radar person-subtraction work is for.
- **A measurement law I had to learn from the data:** this radar profile sees *motion*, not *position*, for slow lifts. The bottom of a rep is momentarily invisible. That's a real sim-to-real gap, and naming it correctly redirected the whole approach toward a velocity channel rather than position reconstruction.
- **Form perception without cameras.** Ground truth for form comes from physics simulation and instrumented jigs, not video: a digital twin that renders radar-and-inertial returns of a parameterized lifter, so the perception model can learn from effectively unlimited labeled data with zero collection and zero cameras.

## How I run the engineering

The part most relevant to how I'd operate anywhere: **every experiment preregisters its pass/fail bar before the data is collected, and every result is recorded as a verdict, including FAIL and VOID.** When a keystone test failed, the honest record of *why* (the simulation's fault sigmas were inflated by comparing identical seeds, the rig's ~15° tilt wasn't modeled, n was too small) is what rebuilt the sim and the protocol. Several attempts later a *narrower* displacement cert held; the form-perception cert is still open. I don't let a system certify itself, and I don't let a good story survive a bad number.

## What's built, what's next

Built: two custom sensors (an inertial bar clip and an mmWave radar puck) designed, manufactured, and shipped across six board revisions, plus a working sensing bench on dev-board stand-ins. On that bench, bar-displacement measurements repeat to within a few centimeters, the clip and radar reliably lock onto the same bar, a radar training corpus is assembled from public and simulated sources, and the form-perception model trains against physics simulation.

Next: bring up the custom boards, isolate the bar from other movers in the scene (dual-radar person-subtraction), and validate against a criterion transducer. Load sensing and coaching efficacy are the builds after that. Each has a named deciding experiment. I don't claim what I haven't measured.

## The end-game

The sensor is the wedge: Olympic-level guidance for anyone, wherever they train. The company is the compounding record underneath: the **physics ground-truth model for strength**, built from millions of real, objective reps that no camera-based or memory-based competitor is capturing.

## What stays private

The calibration, the real-capture corpus and its composition, and the simulation internals are the machinery that makes it work, and they stay off this page. A provisional patent application is on file; what it covers stays off this page too.

---
*Relevance to a robotics conversation: this is a full physical-AI stack in miniature (custom sensors, sensor fusion, sim-to-real, a preregistered eval discipline, and a data flywheel) run by one person under a verification doctrine.*
