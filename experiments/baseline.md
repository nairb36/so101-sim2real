# SO-101 Sim2Real Baseline

**Model:** `aravindhs-NV/grootn16-finetune_sreetz-so101_teleop_vials_rack_left/checkpoint-10000`
**Training data:** ~75 expert demonstrations (teleop).

Success is scored on **visually verified, physically meaningful** outcomes, not simulator
event messages. Results below are as measured; items labeled *Observations* are what was seen,
items labeled *Hypotheses* are unconfirmed candidate explanations.

## Setup note: camera alignment before this baseline

Initially the real-world policy performed very poorly. The culprit turned out to be a
**difference in camera orientation on the real robot relative to simulation** — the real
viewpoint did not match the viewpoint the policy was trained on, and behavior degraded
substantially as a result.

The fix was a targeted **camera-alignment** pass to match the real camera pose to the sim
viewpoint, using Ilia Larchenko's visual alignment tool:
[`scripts/real_camera_align.py`](https://github.com/IliaLarchenko/lehome_solution/blob/main/scripts/real_camera_align.py).

**All data in this baseline was collected after this targeted camera-alignment fix.**

---

## Simulation (20 trials)

Columns: `#Vials` = vials in scene; `#Approached` = vials the policy approached;
`Miss Dir` = direction of a failed-pick miss; slot occupancy and place success as observed.

| # | #Vials | #Approached | Pick | Miss Dir | Travel | Slot | Occupied? | Place |
| -: | -----: | ----------: | :--: | :------- | :----: | :--: | :-------: | :---: |
| 1  | 2 | 2            | ✅ | -                | ✅ | 2 | ❌ | ✅ |
| 2  | 2 | 1            | ✅ | -                | ✅ | 3 | ❌ | ❌ |
| 3  | 2 | 1            | ✅ | -                | ✅ | 1 | ❌ | ❌ |
| 4  | 2 | 1            | ✅ | -                | ✅ | 2 | ❌ | ✅ |
| 5  | 2 | 2            | ✅ | -                | ❌ | - | -  | -  |
| 6  | 2 | 2            | ✅ | -                | ✅ | 3 | ❌ | ❌ |
| 7  | 3 | 2            | ✅ | -                | ✅ | 2 | ❌ | ✅ |
| 8  | 3 | 3            | ✅ | -                | ✅ | 4 | ❌ | ✅ |
| 9  | 3 | 3            | ✅ | -                | ✅ | 1 | ❌ | ❌ |
| 10 | 2 | 1            | ❌ | Right            | -  | - | -  | -  |
| 11 | 2 | 2            | ✅ | -                | ✅ | 2 | ❌ | ✅ |
| 12 | 2 | 1            | ✅ | -                | ✅ | 3 | ❌ | ✅ |
| 13 | 2 | 1            | ✅ | -                | ✅ | 1 | ❌ | ✅ |
| 14 | 2 | 1            | ✅ | -                | ✅ | 2 | ❌ | ❌ |
| 15 | 2 | 2            | ✅ | -                | ✅ | 3 | ❌ | ✅ |
| 16 | 2 | 1 (picked 2) | ✅ | Right, picked 2  | ✅ | 3 | ❌ | ❌ |
| 17 | 3 | 2            | ✅ | -                | ✅ | 2 | ❌ | ❌ |
| 18 | 3 | 3            | ✅ | -                | ✅ | 4 | ❌ | ✅ |
| 19 | 3 | 3            | ✅ | -                | ✅ | 1 | ❌ | ✅ |
| 20 | 2 | 2            | ✅ | -                | ✅ | 1 | ✅ | ❌ |
| **Total** | | | **19** | | **18** | | | **10** |
| **Total %** | | | **95%** | | **90%** | | | **50%** |

| Stage         | Raw Success | Conditional Success |
| ------------- | ----------: | ------------------: |
| Pick          | 95%         | 95%                 |
| Travel to rack| 90%         | 94.74%              |
| Place         | 50%         | 55.55%              |
| End to end    | 50%         | —                   |

**20-trial simulation baseline summary**
- End-to-end task success: **50%**
- Pick success: **95%**
- Successful transport to rack given a pick: **94.7%**
- Placement success given arrival at rack: **55.6%**
- **Placement is the dominant failure stage in simulation.**
- Two observed acquisition errors showed a rightward miss relative to the initially approached
  vial; in one case the policy subsequently grasped the adjacent vial.
- The policy attempted placement into an occupied rack slot in one trial — occupancy avoidance
  is not reliable.
- No convincing evidence currently supports the earlier hypothesis that two-vial scenes
  outperform three-vial scenes.

---

## Real world (20 trials)

| # | #Vials | #Approached | Pick | Miss Dir | Travel | Slot | Occupied? | Place |
| -: | -----: | ----------: | :--: | :------- | :----: | :--: | :-------: | :---: |
| 1  | 2 | 2   | ✅ | -                       | ✅ | 1 | ✅ | ❌ |
| 2  | 2 | 2   | ✅ | -                       | ✅ | 1 | ❌ | ✅ |
| 3  | 1 | 1   | ✅ | -                       | ✅ | 1 | ✅ | ❌ |
| 4  | 2 | 2   | ❌ | Left (towards prior dir?)| -  | - | -  | -  |
| 5  | 2 | 2   | ✅ | -                       | ✅ | 1 | ✅ | ❌ |
| 6  | 2 | 1   | ✅ | -                       | ✅ | 1 | ❌ | ✅ |
| 7  | 3 | 2   | ❌ | Drop                    | -  | - | -  | -  |
| 8  | 3 | 3   | ✅ | -                       | ✅ | 1 | ❌ | ✅ |
| 9  | 3 | NaN | ❌ | Prior area              | -  | - | -  | -  |
| 10 | 2 | 2   | ✅ | -                       | ✅ | 1 | ❌ | ✅ |
| 11 | 1 | 1   | ✅ | -                       | ✅ | 2 | ❌ | ✅ |
| 12 | 2 | 2   | ❌ | Right                   | -  | - | -  | -  |
| 13 | 2 | 2   | ✅ | -                       | ✅ | 1 | ❌ | ❌ |
| 14 | 2 | 2   | ✅ | -                       | ✅ | 3 | ❌ | ❌ |
| 15 | 1 | 1   | ✅ | -                       | ✅ | 3 | ❌ | ✅ |
| 16 | 2 | 2   | ❌ | Left (towards prior)    | -  | - | -  | -  |
| 17 | 3 | 3   | ✅ | -                       | ✅ | 1 | ❌ | ✅ |
| 18 | 3 | 3   | ✅ | -                       | ✅ | 1 | ❌ | ✅ |
| 19 | 3 | 3   | ✅ | -                       | ✅ | 1 | ❌ | ❌ |
| 20 | 2 | 2   | ✅ | -                       | ✅ | 3 | ❌ | ❌ |
| **Total** | | | **15** | | **15** | | | **8** |
| **Total %** | | | **75%** | | **75%** | | | **40%** |

| Stage         | Raw Success | Conditional Success |
| ------------- | ----------: | ------------------: |
| Pick          | 75%         | 75%                 |
| Travel to rack| 75%         | 100%                |
| Place         | 40%         | 53.33%              |
| End to end    | 40%         | —                   |

**20-trial real-world baseline summary**
- End-to-end task success: **40%**
- Pick success: **75%**
- Successful transport to rack given a pick: **100%**
- Placement success given arrival at rack: **53.3%**
- The primary sim-to-real degradation appears during vial selection / approach / grasp
  acquisition; downstream performance after a successful pick is very similar to simulation.
- The policy shows a strong spatial prior toward a front-right region of the workspace; when
  no vial is present there, it can still move toward that region and fail.
- Several failed acquisition attempts showed directional misses or movement toward the
  previously preferred region, suggesting residual spatial/viewpoint sensitivity.
- The policy attempted placement into an occupied rack slot in multiple trials, confirming
  that rack-occupancy avoidance is not reliable.
- Placement failures also occurred on free rack slots, with some attempts appearing spatially
  misaligned — possibly due to residual viewpoint/parallax or execution error.
- The policy showed a strong preference for rack slot 1 in the real-world trials, unlike the
  more distributed slot selection seen in simulation.
- Some successful real-world placements occurred despite imperfect alignment, suggesting real
  contact dynamics/geometry may occasionally be more forgiving than simulation.
- Camera alignment substantially improved behavior vs. the initial real-world runs, indicating
  that camera-viewpoint matching is an important deployment factor.

---

## Sim vs. Real comparison

| Metric / Observation              | Simulation | Real World | Interpretation |
| --------------------------------- | ---------: | ---------: | -------------- |
| End-to-end success                | 50%        | 40%        | 10 pp overall sim-to-real gap |
| Pick success                      | 95%        | 75%        | Largest quantitative gap; acquisition is the main real-world degradation |
| Transport given successful pick   | 94.7%      | 100%       | Essentially no degradation after secure acquisition |
| Placement given rack arrival      | 55.6%      | 53.3%      | Placement performance is nearly identical |
| End-to-end given successful pick  | 52.6%      | 53.3%      | Downstream performance after acquisition is effectively the same |
| Dominant failure stage            | Placement  | Selection / approach / grasp + placement | Additional real-world failures occur primarily before secure grasp |
| Spatial bias                      | Rightward acquisition errors observed | Strong front-right workspace preference | Suggests learned spatial prior and/or viewpoint sensitivity |
| Occupied-slot avoidance           | Failed in ≥1 trial | Failed in multiple trials | Rack-occupancy reasoning is not reliable |
| Rack-slot preference              | Relatively distributed | Strong preference for slot 1 | Possible residual visual/geometric or training-data bias |
| Camera viewpoint sensitivity      | —          | Behavior improved substantially after camera alignment | Camera-viewpoint matching is an important deployment variable |
| Placement tolerance               | Misaligned attempts often fail | Some imperfectly aligned attempts still succeed | Real contact dynamics may sometimes be more forgiving |
| 2 vs 3 vial scenes                | No convincing effect | No convincing effect | Earlier hypothesis not supported |
