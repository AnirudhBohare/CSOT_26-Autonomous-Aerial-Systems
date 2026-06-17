# Week 3 — Flight Control Law & Flight Manager
### Autonomous Aerial Systems · CSOT 2026 · AeroClub, IIT Delhi

> **Dates:** 16–22 Jun 2026
> 
> **Points:** 25 pts
> 
> **Deadline:** 22 Jun 2026, 11:59 PM IST

---

## What This Week Is About

Your Week 2 model now has a realistic physical plant with wind disturbance and noisy sensor outputs. But without a controller, the drone just drifts and falls. This week you close the loop.

You will build two major systems:

1. **Cascaded PID Controller** — an inner attitude loop (roll, pitch, yaw stabilisation) nested inside an outer position loop (altitude hold). This architecture is used in virtually every commercial autopilot.
2. **Flight Mode Manager** — a Stateflow chart that sequences the drone through Idle → Takeoff → Hover → Land, with each mode activating different PID setpoints.

By end of week your model can autonomously hold altitude and attitude under wind disturbance. This directly satisfies R1 (altitude hold ±0.3 m) and R2 (attitude ±5°).

**Key metric:** Altitude RMSE over a 30-second hover — lower is better. Target: RMSE < 0.3 m.

---

## Videos to Watch

| # | Title | Link |
|---|---|---|
| Video 1 | Flight Control Law Design | https://youtu.be/aH8pbcoEg3E?si=XQZoIUKtspBn981k |
| Video 2 | Flight Management Design | https://youtu.be/9YKvnXHYiyw?si=quaVNfKqzA0yDX2g |

Links in [`resources/Resources.md`](./resources/Resources.md)

---

## Tasks This Week

### Task 1 — Cascaded PID Controller (2 hrs)
**Guide:** [`tasks/Task1_Cascaded_PID.md`](./tasks/Task1_Cascaded_PID.md)

Build a two-layer PID structure:
- **Inner loop (attitude):** Roll, pitch, and yaw PID controllers. Input: angle error. Output: torque command. Tune this first.
- **Outer loop (position):** Altitude PID (Z-position). Input: altitude error. Output: desired pitch/roll angle. Connect to inner loop.

### Task 2 — Stateflow Flight Manager (1.5 hrs)
**Guide:** [`tasks/Task2_Stateflow_Manager.md`](./tasks/Task2_Stateflow_Manager.md)

Build a 4-state Stateflow chart: Idle → Takeoff → Hover → Land. Each state sets different PID setpoints and activates/deactivates the appropriate control loops.

### Task 3 — Hover Test & RMSE (0.5 hr)
**Guide:** [`tasks/Task3_RMSE_Test.md`](./tasks/Task3_RMSE_Test.md)

Run a 30-second closed-loop hover simulation with wind active. Compute altitude RMSE in MATLAB. Plot step response. Verify R1 and R2.

---

## What to Submit

All files in `week03/submissions/YOUR_NAME/`:

| # | File | Points |
|---|---|---|
| 1 | `autopilot_wk3_YOUR_NAME.slx` | 15 pts |
| 2 | `hover_plots_wk3_YOUR_NAME.png` | 6 pts |
| 3 | `rmse_result_wk3_YOUR_NAME.png` | 4 pts |

See [`submissions/Submission_Guide.md`](./submissions/Submission_Guide.md) for full instructions.

---

## Scoring

| Criterion | Points |
|---|---|
| Inner attitude PID — functional, all 3 axes | 5 pts |
| Outer altitude PID — connected to inner loop | 4 pts |
| Stateflow chart — all 4 modes with correct transitions | 4 pts |
| Altitude RMSE < 0.3 m during 30 s hover | 6 pts |
| Roll/pitch within ±5° under wind (R2) | 3 pts |
| Hover plots — altitude, roll, pitch vs time, labelled | 3 pts |
| **Total** | **25 pts** |

---

## Suggested Time Split (4–5 hours)

| Activity | Time |
|---|---|
| Videos 7 & 8 | 1 hr |
| Inner attitude PID | 1 hr |
| Outer altitude PID + connect to plant | 1 hr |
| Stateflow flight manager | 45 min |
| 30 s hover test + RMSE + plots | 45 min |

---

## Tips

- **Tune inner loop first, always.** The outer altitude loop cannot work until the inner attitude loop is stable. A good inner loop settles roll/pitch error in < 0.5 s.
- **Start with high proportional gain on attitude, then reduce until oscillation stops.** Add derivative gain last.
- **For altitude hold:** a simple P or PD controller is enough to meet R1 in simulation. Add integral term only if you see steady-state offset.
- **Stateflow transitions:** use time-based guards (e.g. `after(5, sec)`) for Takeoff → Hover. Do not rely on altitude feedback in the guard — it makes the chart hard to debug.
- **Wind compensation:** your Week 2 wind model applies 0.22 N in body X. The PID controller must reject this. If altitude error stays > 0.3 m, your integral gain is too low or derivative too high.

---

## End-of-Week Checklist

- [ ] Model saved as `autopilot_wk3_YOUR_NAME.slx`
- [ ] Inner attitude PID built and tuned (roll, pitch, yaw)
- [ ] Outer altitude PID built and connected to inner loop
- [ ] Controller connected to Week 2 plant (with wind and sensors)
- [ ] Stateflow chart built with Idle, Takeoff, Hover, Land states
- [ ] 30-second hover simulation runs without diverging
- [ ] Altitude RMSE computed and < 0.3 m
- [ ] Roll and pitch stay within ±5° under wind
- [ ] Hover plots saved and labelled
- [ ] PR opened: `[Week 3] Your Name — Submission`

---

Resources: [`resources/Resources.md`](./resources/Resources.md)

*Questions? Open a GitHub Issue tagged `week3-question`.*
*AeroClub, IIT Delhi · CSOT 2026*
