# Week 4 — Sensor Fusion: Extended Kalman Filter
### Autonomous Aerial Systems · CSOT 2026 · AeroClub, IIT Delhi

> **Dates:** 30 Jun - 6 July 2026

> **Points:** 20 pts

> **Deadline:** 7 July 2026, 11:59 PM IST

---

## What This Week Is About

Your Week 3 model closes the loop with a cascaded PID controller — but it still reads **true states** directly from the plant (perfect φ, θ, ψ, z). Real autopilots never have this luxury. All they have is noisy, biased sensor outputs.

This week you fix that. You will build a **9-state Extended Kalman Filter (EKF)** that fuses your Week 2 sensor outputs (IMU, barometer, magnetometer) into a clean, probabilistically-optimal state estimate. Then you will replace the true-state feedback in your Week 3 model with the EKF estimate — making the closed-loop system fully sensor-based.

The EKF has two phases, executed every timestep:
1. **Predict** — propagate the state forward using the IMU as a process input (dead-reckoning)
2. **Update** — correct the prediction using slower, more accurate sensors (barometer, magnetometer)

By the end of this week your model satisfies requirements **R4** (EKF fusing ≥ 2 sensors) and **R5** (position RMSE < 0.5 m, attitude RMSE < 3°).

---

## Videos to Watch

| # | Title | Link |
|---|---|---|
| Video 1 | Sensor Fusion | https://youtu.be/d_9MnFtN6kA?si=9Vdr0Ip3sUrPc5XS |


---

## Tasks This Week

### Task 1 — EKF Mathematical Formulation (2 hrs)
**Guide:** [`tasks/Task1_EKF_Math.md`](./tasks/Task1_EKF_Math.md)

Define the 9-state process model, compute Jacobian matrices F and H, and set up process noise Q and measurement noise R in MATLAB. No Simulink yet — this is the mathematical foundation.

### Task 2 — EKF MATLAB Function Block in Simulink (2 hrs)
**Guide:** [`tasks/Task2_EKF_Simulink.md`](./tasks/Task2_EKF_Simulink.md)

Implement the full EKF predict-update cycle inside a MATLAB Function block. Use Unit Delay blocks to feed back the previous state estimate and covariance. Test open-loop (plant states visible for comparison).

### Task 3 — Closed-Loop Integration & RMSE Evaluation (1 hr)
**Guide:** [`tasks/Task3_Closed_Loop_EKF.md`](./tasks/Task3_Closed_Loop_EKF.md)

Replace the true-state feedback wires in your Week 3 PID controller with EKF estimates. Run the full Takeoff → Hover → Land sequence. Compare EKF vs. true state via RMSE plots.

---

## What to Submit

All files in `week04/submissions/YOUR_NAME/`:

| # | File | Points |
|---|---|---|
| 1 | `autopilot_wk4_YOUR_NAME.slx` | 12 pts |
| 2 | `ekf_comparison_wk4_YOUR_NAME.png` | 5 pts |
| 3 | `ekf_rmse_wk4_YOUR_NAME.png` | 3 pts |

See [`submissions/Submission_Guide.md`](./submissions/Submission_Guide.md) for full instructions.

---

## Scoring

| Criterion | Points |
|---|---|
| EKF predict step — process model + F Jacobian correct | 4 pts |
| EKF update step — Kalman gain, innovation, covariance update (Joseph form) | 4 pts |
| Q and R matrices — physically motivated, filter passes RMSE spec | 3 pts |
| Closed-loop integration — EKF outputs replace all true-state feedback | 4 pts |
| RMSE comparison plot — EKF vs. true, altitude + 3 attitude angles | 3 pts |
| Altitude RMSE < 0.5 m during hover with EKF-only feedback | 2 pts |
| **Total** | **20 pts** |

---

## Suggested Time Split (5–6 hours)

| Activity | Time |
|---|---|
| Video 9 | 1 hr |
| EKF math derivation (Task 1) | 2 hrs |
| EKF Simulink block (Task 2) | 1.5 hrs |
| Closed-loop test + plots (Task 3) | 1 hr |

---

## Tips

- **Test EKF in open loop first.** Run it alongside true-state feedback (not replacing it). Plot EKF vs. true. Only swap feedback once estimates look reasonable.
- **Start with large Q, R values.** Tune down from there. Q_pos = 0.01, Q_vel = 0.1, Q_att = 0.01; R_baro = 0.5, R_mag = 0.1 is a safe starting point.
- **Barometer is your primary correction.** It directly corrects z-position altitude hold. If baro correction is wrong, altitude will drift even with perfect attitude estimates.
- **Wrap psi to [−π, π].** The innovation for yaw must also be wrapped: y_psi = atan2(sin(y_psi), cos(y_psi)).
- **Use Joseph form for covariance update.** P_est = (I-KH)*P*(I-KH)' + K*R*K'. This prevents P from going non-positive-definite over long simulations.
- **Initialise P0 larger than you think.** P0 = diag([1,1,1,0.5,0.5,0.5,0.1,0.1,0.1]) works well for a cold-start at the origin.

---

## End-of-Week Checklist

- [ ] Video 9 watched (Sensor Fusion)
- [ ] 9-state vector defined: [x, y, z, vx, vy, vz, φ, θ, ψ]
- [ ] Process model f(x,u) written for all 9 states
- [ ] Jacobian F (9×9) derived and coded in MATLAB
- [ ] Measurement model h(x) defined: baro altitude + magnetometer heading
- [ ] Jacobian H (2×9) coded
- [ ] Q (9×9) and R (2×2) matrices set and physically justified
- [ ] EKF predict step working inside MATLAB Function block
- [ ] EKF update step working (Kalman gain, innovation, Joseph covariance form)
- [ ] Unit Delay blocks wired for x_est and P_est feedback
- [ ] Open-loop test: EKF tracks true state on altitude and attitude
- [ ] Closed-loop: all true-state feedback wires replaced with EKF estimates
- [ ] 40-second simulation runs without diverging
- [ ] Altitude RMSE < 0.5 m with EKF-only feedback
- [ ] 4-panel comparison plot saved and labelled
- [ ] PR opened: `[Week 4] Your Name — Submission`

---

Resources: [`resources/Resources.md`](./resources/Resources.md)

*Questions? Open a GitHub Issue tagged `week4-question`.*
*AeroClub, IIT Delhi · CSOT 2026*
