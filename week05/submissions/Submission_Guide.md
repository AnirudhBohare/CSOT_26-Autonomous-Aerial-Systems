# Week 5 — Submission Guide
### Autonomous Aerial Systems · CSOT 2026 · AeroClub, IIT Delhi

>**Deadline:** 16 Jul 2026, 11:59 PM IST

>**Late penalty:** −5 pts per day

---

## What to Submit

| # | File | Points | Required? |
|---|---|---|---|
| 1 | `autopilot_wk5_YOUR_NAME.slx` | 15 pts | Core |
| 2 | `trajectory_plots_wk5_YOUR_NAME.png` | 5 pts | Core |
| 3 | `rmse_validation_wk5_YOUR_NAME.png` | 5 pts | Core |
| 4 | `monte_carlo_wk5_YOUR_NAME.png` | 10 pts | Bonus |
| 5 | `mc_rmse_table_wk5_YOUR_NAME.png` | 10 pts | Bonus |

Replace `YOUR_NAME` with your actual name (e.g. `autopilot_wk5_Anirudh.slx`).

---

## What the `.slx` Must Contain

Evaluators will open your model in MATLAB R2026 and run it with simulation time = 60 s. Your model must:

- **Run without errors** from a fresh MATLAB session
- Contain all **Week 1–4 components** intact (plant, sensors, EKF, PID, Stateflow)
- Have **EKF estimates** (not true plant states) connected to ALL PID feedback ports
- Contain `X_Position_PID` and `Y_Position_PID` blocks:
  - X PID: error = 0 − x_hat(1) → output → theta_setpoint (with saturation ±0.26 rad)
  - Y PID: error = 0 − x_hat(2) → Gain(−1) → phi_setpoint (with saturation ±0.26 rad)
  - phi_setpoint and theta_setpoint wired into the inner attitude PID orientation_setpoint
- Have **To Workspace** blocks logging: `x_hat_log`, `x_hat_x`, `x_hat_y`, `x_hat_vx`, `x_hat_vy`, `x_true`, `y_true`, `z`, `phi`, `theta`, `psi`, `mode`

---

## What the Trajectory Plot Must Show

`trajectory_plots_wk5_YOUR_NAME.png` must be a 2×2 figure:

1. **Altitude** — EKF z vs true z over 60 s, setpoint line at 1 m
2. **North (x)** — EKF x vs true x over 60 s, setpoint line at 0 m
3. **East (y)** — EKF y vs true y over 60 s, setpoint line at 0 m
4. **Attitude** — φ, θ, ψ (EKF and true) over 60 s

Each panel must have: axis labels, title with RMSE value, grid.

---

## What the RMSE Screenshot Must Show

`rmse_validation_wk5_YOUR_NAME.png` must be a Command Window screenshot:

```
=================================================================
Week 5 — Full Autopilot RMSE (30 s hover, wind ON, EKF-in-loop)
=================================================================
Position  x  RMSE: X.XXXX m    [Target < 0.50 m]   PASS
Position  y  RMSE: X.XXXX m    [Target < 0.50 m]   PASS
Altitude  z  RMSE: X.XXXX m    [Target < 0.15 m]   PASS
Roll    phi  RMSE: X.XXXX deg  [Target < 3.00 deg]  PASS
Pitch theta  RMSE: X.XXXX deg  [Target < 3.00 deg]  PASS
Yaw     psi  RMSE: X.XXXX deg  [Target < 5.00 deg]  PASS
=================================================================
```

All 6 states must be present regardless of PASS/FAIL.

---

## How to Submit

| # | Step |
|---|---|
| 1 | Open your Google Drive folder `Autopilot_YOURNAME` |
| 2 | Create a subfolder named `week05_submissions` |
| 3 | Upload all 3 core files (and 2 bonus files if complete) |
| 4 | Set sharing to "Anyone with link can view" |
| 5 | Submit Drive link via Google Form: https://forms.gle/qaFpcEgGAxvwbfCp8 |

---

## Scoring Rubric

### Core (25 pts)
| Criterion | Max | What evaluators check |
|---|---|---|
| EKF fully in loop — all true states replaced, position logged | 4 pts | No plant state wires to PID; x_hat_x, x_hat_y in workspace |
| X Position PID — error, saturation, cascade to theta_setpoint | 4 pts | X_Position_PID block; sat ±0.26 rad; theta_setpoint wired to Pitch PID |
| Y Position PID — error, Gain(−1), cascade to phi_setpoint | 3 pts | Y_Position_PID block; Gain=−1 present; phi_setpoint wired to Roll PID |
| R6 validated: x RMSE < 0.5 m, y RMSE < 0.5 m | 5 pts | RMSE screenshot shows PASS for both |
| z, φ, θ, ψ RMSE within Week 4 targets | 3 pts | All four PASS in RMSE screenshot |
| Trajectory plots — 4-panel, RMSE in titles, all setpoints | 3 pts | Figure layout, axis labels, RMSE values in titles |
| RMSE table — 6 states, hover window, PASS/FAIL | 3 pts | All 6 rows present, correct values |
| **Core Total** | **25 pts** | |

### Bonus (20 pts)
| Criterion | Max | What evaluators check |
|---|---|---|
| 30-run MC loop implemented correctly | 4 pts | Script present; set_param changes seeds; runs complete |
| Per-run RMSE: 6 states, 30×6 matrix | 4 pts | rmse_mc matrix correct dimensions; computed from hover window |
| Histogram figure — 6 panels, target lines, pass% | 6 pts | All 6 panels; xline at target; pass rate text annotation |
| Summary table — mean ± σ, max, pass% for all states | 6 pts | All 6 states in table; pass rate > 90% for x and y |
| **Bonus Total** | **20 pts** | |

---

## Common Mistakes

| Mistake | Fix |
|---|---|
| True x/y from plant used instead of EKF estimates | Position PID must use x_hat_delayed(1) and x_hat_delayed(2), NOT Translational Dynamics outputs |
| Y PID sign correct but no Gain(−1) block | Without the negation, y error causes divergence; add Gain=−1 explicitly |
| phi/theta_setpoint overrides Fz altitude control | Check signal routing — theta_setpoint goes to Pitch_PID setpoint only; Fz_total path is separate |
| Monte Carlo seeds not actually changing | Use get_param after set_param to confirm the seed changed; close and reopen model if stuck |
| RMSE hover window uses t=6–36 s instead of 5–35 s | Task 3 and Bonus use t=5–35 s for the 30-second hover window — recheck mask |
