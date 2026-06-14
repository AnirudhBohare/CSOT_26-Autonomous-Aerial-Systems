# Week 2 — Submission Guide
### Autonomous Aerial Systems · CSOT 2026 · AeroClub, IIT Delhi

Deadline: 15 Jun 2026, 11:59 PM IST
Late penalty: -5 pts per day

---

## What to Submit

Exactly 3 files in week02/submissions/YOUR_NAME/ — no written reports:

| # | File | Points |
|---|---|---|
| 1 | autopilot_wk2_YOUR_NAME.slx | 9 pts |
| 2 | plots_wk2_YOUR_NAME.png (or up to 4 separate PNGs) | 4 pts |
| 3 | snr_result_YOUR_NAME.png | 2 pts |

The .slx file is the main deliverable. Evaluators will open it in MATLAB R2026 and run it.

---

## What the .slx Must Contain

Evaluators will check for all of these inside your model:

- Wind Model subsystem — constant force + turbulence noise, connected to Translational Dynamics
- IMU subsystem — containing Accelerometer and Gyroscope subsystems with correct parameters
- Barometer subsystem 
- Magnetometer subsystem 
- All sensor outputs named: accel_meas, gyro_meas, baro_alt_meas, mag_heading_meas
- To Workspace blocks present for at least accel_meas and the true acceleration

---

## What the Plots Must Show

plots_wk2_YOUR_NAME.png must contain 4 subplots:
1. Accelerometer X-axis: true (blue) vs measured (red) over 10 s
2. Gyroscope X-axis: true vs measured over 10 s
3. Barometer: true altitude vs barometer output over 10 s
4. Magnetometer: true yaw vs magnetometer output over 10 s

Each subplot must have: axis labels, title, legend, grid.

---

## What the SNR Screenshot Must Show

snr_result_YOUR_NAME.png must be a screenshot of the MATLAB Command Window showing:
  IMU Accelerometer X-axis SNR: XX.XX dB
  IMU Gyroscope X-axis SNR: XX.XX dB



---

## How to Submit 

| # | File |
|---|---|
| 1 | Go to parent folder in GDrive named as `Autopilot_YOURNAME` |
| 2 | Inside it , create another folder named as `week02_submissions` |
| 3 | Add all the submission files in that week's subfolder |
| 4 | Now in the google form shared , share the link to the parent folder so that you can add subfolders for each subsequent week there itself |
| 5 | Also do not forget to provide access to read or write |

Share your drive links here: [Google Form](https://forms.gle/qaFpcEgGAxvwbfCp8)





## Week 2 Assessment

| Criterion | Max | My Score |
|---|---|---|
| All 4 sensor subsystems present | 5 | /5 |
| Correct noise parameters | 3 | /3 |
| Wind causes visible drift | 1 | /1 |
| Comparison plots (4 sensors) | 4 | /4 |
| SNR reported and realistic | 2 | /2 |
| Total | 15 | /15 |

---

## Common Mistakes

| Mistake | Fix |
|---|---|
| Model runs but sensors not connected at top level | Check output ports are wired to subsystems |
| Plots have no labels or legends | Add xlabel, ylabel, title, legend to every subplot |
