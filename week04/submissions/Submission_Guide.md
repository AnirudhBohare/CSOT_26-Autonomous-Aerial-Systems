# Week 4 — Submission Guide
### Autonomous Aerial Systems · CSOT 2026 · AeroClub, IIT Delhi

>**Deadline:** 7 July 2026, 11:59 PM IST

>**Late penalty:** −5 pts per day

---

## What to Submit

Exactly 3 files — no written report:

| # | File | Points |
|---|---|---|
| 1 | `autopilot_wk4_YOUR_NAME.slx` | 12 pts |
| 2 | `ekf_comparison_wk4_YOUR_NAME.png` | 5 pts |
| 3 | `ekf_rmse_wk4_YOUR_NAME.png` | 3 pts |

Replace `YOUR_NAME` with your actual name (e.g. `autopilot_wk4_Anirudh.slx`).

---

## What the `.slx` Must Contain

Evaluators will open your model in MATLAB R2026 and run it for 40 seconds. Your model must:

- **Run without errors** from a fresh MATLAB session (no missing workspace variables)
- Contain the **EKF_Sensor_Fusion subsystem** with:
  - MATLAB Function block implementing the full predict + update cycle
  - Unit Delay block for x_prev with correct initial condition
  - Unit Delay block for P_prev with correct initial condition (9×9 matrix)
- Have **all PID feedback wires connected to EKF estimates** (not true plant states):
  - `x_est(7)` → Roll PID error
  - `x_est(8)` → Pitch PID error
  - `x_est(9)` → Yaw PID error
  - `-x_est(3)` → Altitude PID error
- Retain all **Week 2 and Week 3 subsystems**: plant, sensors, wind, cascaded PID, Stateflow
- Have **To Workspace** blocks on at minimum: `x_est`, `z_true`, `phi_true`, `theta_true`, `psi_true`, `mode`

---

## What the Comparison Plot Must Show

`ekf_comparison_wk4_YOUR_NAME.png` must be a 2×2 figure containing:

1. **Altitude:** True altitude (blue) vs. EKF estimate (red dashed) vs. setpoint (black dotted), 0–40 s
2. **Roll:** True roll vs. EKF roll estimate, with ±5° bounds
3. **Pitch:** True pitch vs. EKF pitch estimate, with ±5° bounds
4. **Altitude error:** True error vs. EKF-feedback error vs. ±0.5 m and ±0.3 m bounds

Each subplot must have: axis labels, title (with RMSE value), legend, grid.

---

## What the RMSE Screenshot Must Show

`ekf_rmse_wk4_YOUR_NAME.png` must be a screenshot of the MATLAB Command Window showing:

```
=============================================
EKF Performance Report (30 s hover window)
=============================================
Altitude RMSE vs setpoint  (R1):  X.XXXX m  [target < 0.5 m]  PASS
Altitude estimation RMSE   (R5):  X.XXXX m
Roll estimation RMSE       (R5):  X.XXXX deg
Pitch estimation RMSE      (R5):  X.XXXX deg
=============================================
```

---

## How to Submit

| # | Step |
|---|---|
| 1 | Open your Google Drive folder `Autopilot_YOURNAME` (created in Week 1) |
| 2 | Create a subfolder named `week04_submissions` |
| 3 | Upload all 3 files to that subfolder |
| 4 | Ensure sharing is set to "Anyone with link can view" |
| 5 | Submit the Drive link via the Google Form: [Google Form](https://forms.gle/qaFpcEgGAxvwbfCp8) |

---

## Assessment Rubric

| Criterion | Max | What evaluators check |
|---|---|---|
| EKF predict step — process model + F Jacobian | 4 pts | f(x,u) correct for all 9 states; F non-zero entries present |
| EKF update step — Kalman gain, Joseph covariance form | 4 pts | K correct dimensions (9×2); Joseph form used; psi innovation wrapped |
| Q and R matrices — physically justified | 3 pts | Q diagonal; R values near sensor noise levels from Week 2 |
| Closed-loop integration — no true-state leakage | 4 pts | All 4 PID feedback signals come from x_est (verified by model review) |
| RMSE comparison plot — 4 panels, labelled | 3 pts | Titles with RMSE values; labels; legends; PASS/FAIL annotations |
| Altitude RMSE < 0.5 m with EKF feedback | 2 pts | Computed over 30 s hover window, wind active |
| **Total** | **20 pts** | |

---

## Common Submission Mistakes

| Mistake | Fix |
|---|---|
| True-state wires still present alongside EKF wires | Delete all direct plant→PID connections; use only x_est |
| P_prev Unit Delay IC = 0 (scalar) | Set IC to `diag([1,1,1,0.5,0.5,0.5,0.1,0.1,0.1])` — 9×9 matrix |
| RMSE computed over wrong window | Use t = 6 to 36 s only (hover window) |
| `.slx` depends on workspace Q or R | Define Q and R inside the MATLAB Function block, not as workspace vars |
| Missing `z_true` To Workspace block | Without it, RMSE comparison cannot be run by evaluators |
