# Week 5 — Full Autopilot Integration & System Validation
### Autonomous Aerial Systems · CSOT 2026 · AeroClub, IIT Delhi

> **Dates:** 10 July – 15 Jul 2026

> **Points:** 25 pts (core) + 20 pts (bonus) = 45 pts total

> **Deadline:** 16 Jul 2026, 11:59 PM IST

---

## What This Week Is About

You now have all the building blocks: a 6-DOF plant (Week 1), a full sensor suite (Week 2), a cascaded PID flight manager (Week 3), and an EKF state estimator (Week 4). This final week assembles them into a **complete sensor-based autopilot** — the kind that flies real quadcopters.

There are two major goals:

**Goal 1 — Close the EKF loop.** Replace every remaining true plant state signal feeding your PID with EKF estimates. The autopilot must fly using only what the sensors tell it.

**Goal 2 — Add X/Y position hold.** Extend the controller with outer position PIDs that command pitch and roll angles to hold the drone stationary over its takeoff point under wind disturbance. The EKF provides horizontal position and velocity estimates.

**Key metric — R6 Requirement:** Mean position error (x, y, z) over the 30-second hover window must be < 0.5 m with wind active. This is the capstone acceptance criterion for the full autopilot.

| State | Target hover RMSE |
|---|---|
| x position | < 0.5 m |
| y position | < 0.5 m |
| z (altitude) | < 0.15 m (carry-forward from Week 4) |
| Roll φ, Pitch θ | < 3° each |
| Yaw ψ | < 5° |

---

## Videos to Watch

| # | Title | Watch before |
|---|---|---|
| Video 1 | Full Autopilot Integration & Validation | Task 1 |

Link in [`resources/Resources.md`](./resources/Resources.md)

---

## Architecture: What the Final Model Looks Like

```
                 ┌─────────────────────────────────────────┐
                 │           SENSOR SUITE (Wk 2)           │
                 │  IMU · Barometer · Magnetometer · Wind   │
                 └────────────────┬────────────────────────┘
                                  │ a_meas, gyro, baro, mag
                                  ▼
                        ┌──────────────────┐
                        │   EKF (Wk 4)     │ x_hat [9×1]
                        │  9-state filter  │ ─────────────────┐
                        └──────────────────┘                  │
                                                              ▼
  setpoints ──►  ┌──────────────────────────────────────┐  [Unit Delay z⁻¹]
  x=0, y=0,      │          CONTROLLER (Wk 5)           │       │
  z=-1.0,        │  Position PIDs (X/Y) — NEW THIS WK   │  x_hat_delayed
  ψ=0            │  Altitude PID  (Wk 3)                │       │
                 │  Inner Attitude PIDs (Wk 3)          │ ◄─────┘
                 └────────────────┬─────────────────────┘
                                  │ Fz, τ_roll, τ_pitch, τ_yaw
                                  ▼
                         ┌────────────────┐
                         │  PLANT (Wk 1)  │──► true states (logged only)
                         └────────────────┘
```

---

## Tasks This Week

### Task 1 — EKF-in-the-Loop Integration (2 hrs) — 10 pts
**Guide:** [`tasks/Task1_EKF_Loop_Integration.md`](./tasks/Task1_EKF_Loop_Integration.md)

Complete the EKF closed loop: disconnect all remaining true state wires from the PID and connect EKF estimates. Add X/Y position error computation using EKF states 1 and 2. Wire position estimates to new outer PIDs.

### Task 2 — X/Y Position Hold (1.5 hrs) — 9 pts
**Guide:** [`tasks/Task2_Position_Hold.md`](./tasks/Task2_Position_Hold.md)

Implement two outer position PIDs (North PID → θ_setpoint, East PID → φ_setpoint). Cascade them into the existing inner attitude PID loop. Tune gains to hold position under the 0.22 N crosswind disturbance.

### Task 3 — System Validation & RMSE Report (1 hr) — 6 pts
**Guide:** [`tasks/Task3_System_Validation.md`](./tasks/Task3_System_Validation.md)

Run a 60-second closed-loop simulation (IDLE → TAKEOFF → HOVER → LAND) with wind and all sensor noise active. Generate trajectory plots and compute the full RMSE table. Verify the R6 target (position error < 0.5 m).

### Bonus Task — Monte Carlo Analysis (20 pts)
**Guide:** [`tasks/BonusTask_MonteCarlo.md`](./tasks/BonusTask_MonteCarlo.md)

Run 30 simulations with different BLWN noise seeds. Compute RMSE across the ensemble. Plot RMSE distributions and report mean ± 1σ. This demonstrates statistical robustness — the real measure of an autopilot's reliability.

---

## What to Submit

All files in `week05/submissions/YOUR_NAME/`:

| # | File | Points |
|---|---|---|
| 1 | `autopilot_wk5_YOUR_NAME.slx` | 15 pts |
| 2 | `trajectory_plots_wk5_YOUR_NAME.png` | 5 pts |
| 3 | `rmse_validation_wk5_YOUR_NAME.png` | 5 pts |
| (Bonus) | `monte_carlo_wk5_YOUR_NAME.png` | 10 pts |
| (Bonus) | `mc_rmse_table_wk5_YOUR_NAME.png` | 10 pts |

See [`submissions/Submission_Guide.md`](./submissions/Submission_Guide.md) for full instructions.

---

## Scoring

### Core (25 pts)
| Criterion | Points |
|---|---|
| EKF fully in loop — all true states replaced, position estimates logged | 4 pts |
| X PID (North): correct cascade to θ_setpoint, tuned | 4 pts |
| Y PID (East): correct cascade to φ_setpoint, tuned | 3 pts |
| R6 validated: position RMSE x < 0.5 m, y < 0.5 m (hover window) | 5 pts |
| Altitude and attitude RMSE within Week 4 targets (carry-forward) | 3 pts |
| Trajectory plots — 4-panel, labelled, all setpoints visible | 3 pts |
| RMSE table — all states, PASS/FAIL, correct hover window | 3 pts |
| **Core Total** | **25 pts** |

### Bonus (20 pts)
| Criterion | Points |
|---|---|
| 30-simulation Monte Carlo loop implemented correctly | 4 pts |
| Per-run RMSE computed for x, y, z, φ, θ, ψ | 4 pts |
| RMSE distribution histogram (6-panel or combined) | 6 pts |
| Summary table: mean ± σ for all states, compared to R6 targets | 6 pts |
| **Bonus Total** | **20 pts** |

---

## Suggested Time Split (5.5 hours)

| Activity | Time |
|---|---|
| Video 1 | 1 hr |
| EKF loop closure (disconnect true states, log x/y) | 1 hr |
| X/Y position PIDs — build and tune | 1.5 hrs |
| 60-second simulation + trajectory plots | 0.5 hr |
| RMSE computation and table | 0.5 hr |
| (Bonus) Monte Carlo loop + plots | 1.5 hrs |

---

## Tips

- **Tune position PIDs very conservatively.** A Kp that is too high causes the drone to oscillate between the desired position and overshoots. Start with Kp=0.3, Kd=0.5, Ki=0 and increase slowly.
- **The X/Y coupling is real.** A North PID commanding positive θ causes the drone to pitch forward, which changes roll dynamics. If the drone spirals, reduce both X and Y Kp simultaneously.
- **EKF position drifts without GPS.** The EKF integrates accelerometers for x/y position — IMU drift will accumulate. This is expected; the PIDs will compensate. Mean position error is the target, not zero drift.
- **Wind is in the +x direction (0.22 N).** Your North PID will work hardest. If position hold fails only in x, re-tune the X PID first.
- **For Monte Carlo:** use `sim()` inside a `for` loop, change the `seed` parameter on each BLWN block before each run using `set_param`.

---

## End-of-Week Checklist (Core)

- [ ] All true state wires from PID disconnected — EKF estimates wired instead
- [ ] x_hat(1) and x_hat(2) logged to workspace as `x_hat_x` and `x_hat_y`
- [ ] X position PID added: error = 0 − x_hat(1) → PID → theta_setpoint
- [ ] Y position PID added: error = 0 − x_hat(2) → PID → phi_setpoint (sign check!)
- [ ] theta_setpoint from X PID replaces the Constant(0) feeding Pitch_PID
- [ ] phi_setpoint from Y PID replaces the Constant(0) feeding Roll_PID
- [ ] Saturation on theta_setpoint: ±15° (±0.26 rad)
- [ ] Saturation on phi_setpoint: ±15° (±0.26 rad)
- [ ] 60-second closed-loop simulation runs without divergence
- [ ] Stateflow sequences: 1 → 2 → 3 → 4 in the mode signal
- [ ] Position RMSE x < 0.5 m (hover window, wind ON)
- [ ] Position RMSE y < 0.5 m
- [ ] Altitude RMSE z < 0.15 m
- [ ] 4-panel trajectory figure saved and labelled
- [ ] RMSE table screenshot saved from Command Window

## End-of-Week Checklist (Bonus)

- [ ] `for` loop over 30 seeds implemented in MATLAB script
- [ ] `set_param` used to change seed on all BLWN blocks before each sim run
- [ ] Per-run RMSE stored in matrix `rmse_mc` (30 × 6)
- [ ] Histogram plots generated for all 6 states
- [ ] Mean ± σ summary table printed and saved as PNG

---

Resources: [`resources/Resources.md`](./resources/Resources.md)

*Questions? Open a GitHub Issue tagged `week5-question`.*
*AeroClub, IIT Delhi · CSOT 2026 — Thank you for flying with us!*
