# so101-sim2real

Sim-to-real robot-learning experiments with **NVIDIA GR00T N1.6** on an **SO-101** arm.
The focus is understanding and reducing the **simulation-to-real (Sim2Real) gap** for a
learned tabletop manipulation policy.

This repository is an **experimental record**, not a polished benchmark. Results are
reported as measured, hypotheses are kept separate from confirmed causes, and planned
experiments are labeled as planned.

---

## Rollout preview

A **sim-only-trained** GR00T policy running on the **real SO-101**. The clip shows three
consecutive rollouts — **two successes and one failure** — a small illustration of the
Sim2Real gap, not a success-rate measurement.

https://github.com/user-attachments/assets/55d65970-f24c-48bd-b7e2-405cfaff5fd5

<sub>Real SO-101 rollouts: two successes, one failure (sim-only-trained policy). A copy of the
clip also lives in the repo at <a href="media/so_101_rollout_good_bad.mp4">media/so_101_rollout_good_bad.mp4</a>.</sub>

---

## The problem

A GR00T-based policy is trained largely in simulation, then deployed on a physical robot.
Policies that look competent in sim routinely degrade on real hardware. The goal here is to
**measure that gap, localize where it comes from, and test interventions one variable at a
time** rather than chasing a single lucky rollout.

### The task

A tabletop manipulation task. The SO-101 must:

1. approach a vial,
2. grasp it,
3. transport it to a rack,
4. place / release the vial into a rack slot.

### Why stage-level evaluation

Binary end-to-end success hides *where* a policy breaks. Every rollout is scored by stage —
**pick → transport → place → end-to-end** — plus conditional metrics such as
`P(transport | pick)` and `P(place | arrival at rack)`. This distinguishes acquisition
failures from grasp-stability, transport, rack-localization, alignment, placement, and
release-timing failures.

> **Note on success criteria.** Simulator event messages such as
> `[RACK] vial_1 placed in rack` are *not* treated as ground truth — they can fire even when
> the vial did not actually enter a rack hole. Success is defined by **visually verified,
> physically meaningful** outcomes.

---

## Hardware setup

### Robot

- **SO-101 follower arm** — the arm under evaluation.
- **SO-101 leader arm** — used for teleoperation / collecting demonstrations.
- The real robot is controlled from a **Mac**.

### Cameras

Two cameras on the real robot:

- **wrist camera**
- **front / external camera**

Typical capture: **640 × 480 @ 30 FPS**. Camera indices vary between machines and sessions,
so they are not hard-coded in documentation.

---

## Compute architecture

Policy inference does **not** run locally. Observations are streamed from the Mac to a remote
**Brev** GPU instance, GR00T runs inference there, and predicted actions are streamed back.
The Mac↔Brev link uses **ZMQ**.

```text
Physical SO-101 + Cameras
          |
          |  observations (images + robot state)
          v
         Mac  ──────────────┐
          ^                 |  network / ZMQ
          |  actions        v
          |            Brev GPU server
          |                 |
          |                 |  GR00T N1.6 inference
          └─────────────────┘
                            |
                            v
                    predicted actions
```

Because inference is remote, **networking and inference latency are candidate Sim2Real
variables**, not just implementation details.

**Measured round-trip latency** (100-request test): mean ≈ 39.7 ms, median ≈ 38 ms,
P99 ≈ 42 ms, with one spike to ≈ 159 ms. Latency is real but has **not** been shown to be the
root cause of any observed grasp failures. Future timing work should decompose the full loop
(capture → preprocess → send → server queue → inference → response → execute) rather than
measuring RTT alone.

---

## Policy / model

- **NVIDIA GR00T N1.6**, a Vision-Language-Action policy conditioning on images, robot state,
  and a task instruction, generating **action chunks** rather than single-step actions.
- GR00T N1.6 uses a generative (flow-matching / denoising-style) action formulation, which
  represents multimodal action distributions better than a deterministic MSE regressor.
- Baseline checkpoint under evaluation:

  ```text
  aravindhs-NV/grootn16-finetune_sreetz-so101_teleop_vials_rack_left/checkpoint-10000
  ```

  Trained on ~75 expert demonstrations.

---

## Simulation

Simulation uses NVIDIA's SO-101 / GR00T workflow in **Isaac Lab / Isaac Sim**, which provides
the same vial→rack scene and event messages used above. Sim is the source of the baseline
policy and the environment for testing randomization-based interventions before they touch
real hardware.

---

## Baseline results

Stage-level baseline over 20 sim + 20 real trials (checkpoint-10000). Real data was collected
**after** a camera-alignment fix that matched the real camera viewpoint to sim — the initial
real runs were badly degraded by a camera-orientation mismatch.

| Stage (conditional)              | Sim   | Real  |
| -------------------------------- | ----: | ----: |
| Pick                             | 95%   | 75%   |
| Transport \| successful pick     | 94.7% | 100%  |
| Placement \| arrival at rack     | 55.6% | 53.3% |
| End-to-end success               | 50%   | 40%   |

**Takeaways (as measured):** the largest sim→real gap is at **acquisition** (pick 95% → 75%);
once the vial is securely grasped, downstream behavior is nearly identical to sim. In sim,
**placement** is the dominant failure stage. The real policy also shows a strong front-right
spatial prior and a preference for rack slot 1, and rack-occupancy avoidance is unreliable in
both domains. Small samples — see the full tables, per-trial logs, observations, and
hypotheses in **[experiments/baseline.md](experiments/baseline.md)**.

---

## Status & scope

- [x] Simulation stage-level baseline (20 trials) — see [experiments/baseline.md](experiments/baseline.md)
- [x] Real-robot stage-level baseline (20 trials, post camera-alignment fix)
- [x] Sim vs. real failure-distribution comparison
- [ ] Full timing decomposition (beyond network RTT)
- [ ] Sim2Real interventions, evaluated one variable at a time

Planned interventions include domain randomization, sim + small-real co-training, targeted
recovery demonstrations, and latency randomization. These are **planned**, not yet run.

Guiding loop for every experiment:

```text
baseline → identify dominant failure → form hypothesis
        → change one meaningful variable → rerun evaluation → compare quantitatively
```
