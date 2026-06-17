# Week 3 — Submission Guide
### Autonomous Aerial Systems · CSOT 2026 · AeroClub, IIT Delhi

**Deadline:** 23 Jun 2026, 11:59 PM IST

**Late penalty:** −5 pts per day

---

## What to Submit

Exactly 3 files — no written report:

| # | File | Points |
|---|---|---|
| 1 | `autopilot_wk3_YOUR_NAME.slx` | 15 pts |
| 2 | `hover_plots_wk3_YOUR_NAME.png` | 6 pts |
| 3 | `rmse_result_wk3_YOUR_NAME.png` | 4 pts |

Replace `YOUR_NAME` with your actual name (e.g. `autopilot_wk3_Anirudh.slx`).

---

## What the `.slx` Must Contain

Evaluators will open your model in MATLAB R2026 and run it. Your model must:

- **Run without errors** from a fresh MATLAB session (no missing files or paths)
- Contain the **Controller subsystem** with:
  - Inner attitude PID: Roll, Pitch, Yaw PIDs (all 3 axes)
  - Outer altitude PID connected to inner Pitch PID
- Contain the **Stateflow Flight Manager** with:
  - IDLE, TAKEOFF, HOVER, LAND states
  - Correct transition conditions
  - `z_setpoint` and `motors_enabled` outputs connected
- Retain the **Week 2 subsystems**: Wind Model, IMU, Barometer, Magnetometer
- Have **To Workspace** blocks on at minimum: `z`, `phi`, `theta`, `psi`, `mode`

---

## What the Hover Plots Must Show

`hover_plots_wk3_YOUR_NAME.png` must be a 2×2 figure containing:

1. **Altitude vs time** — true altitude (blue) vs setpoint (red dashed) over 40 s
2. **Roll angle vs time** — φ in degrees with ±5° bounds
3. **Pitch angle vs time** — θ in degrees with ±5° bounds
4. **Altitude error vs time** — (alt_true − 1.0 m) with ±0.3 m bounds

Each subplot must have: axis labels, title (including PASS/FAIL for the relevant requirement), legend, grid.

---

## What the RMSE Screenshot Must Show

`rmse_result_wk3_YOUR_NAME.png` must be a screenshot of the MATLAB Command Window showing:

```
========================================
Altitude RMSE (30 s hover, wind ON): X.XXXX m
R1 STATUS: PASS (< 0.3 m)
========================================
Max roll  during hover: X.XX deg
Max pitch during hover: X.XX deg
R2 STATUS: PASS (roll & pitch within ±5°)
```

(Adapt format if yours shows different R2 result — show the actual numbers regardless.)

---

## How to Submit

| # | Step |
|---|---|
| 1 | Open your Google Drive folder `Autopilot_YOURNAME` (created in Week 1) |
| 2 | Create a subfolder named `week03_submissions` |
| 3 | Upload all 3 files to that subfolder |
| 4 | Ensure sharing is set to "Anyone with link can view" |
| 5 | Submit the Drive link via the Google Form: [Google Form](https://forms.gle/qaFpcEgGAxvwbfCp8) |

**Do not** open a GitHub PR for submissions this week — use only the Google Drive + Form process.

---

## Assessment Rubric

| Criterion | Max | What evaluators check |
|---|---|---|
| Inner attitude PID — all 3 axes functional | 5 pts | Roll, pitch, yaw PIDs present, gains non-zero, no saturation |
| Outer altitude PID — connected to inner loop | 4 pts | Altitude PID output feeds pitch setpoint; feedforward present |
| Stateflow chart — all 4 modes with transitions | 4 pts | States and transition conditions correct; default to IDLE |
| Altitude RMSE < 0.3 m (R1) | 6 pts | Measured over 30 s hover with wind active |
| Roll/pitch within ±5° under wind (R2) | 3 pts | Max deviation during hover window |
| Hover plots — 4 panels, labelled | 3 pts | Titles, labels, legends, PASS/FAIL annotations |
| **Total** | **25 pts** | |

---

## Common Mistakes

| Mistake | Fix |
|---|---|
| Model runs but Controller subsystem missing | Group PID blocks into a labelled subsystem |
| Stateflow chart outputs not connected | Wire `z_setpoint` and `motors_enabled` at top level |
| RMSE computed over wrong window | Use t = 6 to 36 s (hover window only, not takeoff) |
| Plots saved at low resolution | Use `saveas(gcf, 'file.png')` — default is 96 DPI, sufficient |
| Wind disabled during RMSE test | R3 requires wind active. Do not disable Wind Model |
| `.slx` depends on a workspace variable | Use Constant blocks inside model, not workspace vars |
