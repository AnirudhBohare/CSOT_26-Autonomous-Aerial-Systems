# Week 2 — Environment & Sensor Models
### Autonomous Aerial Systems · CSOT 2026 · AeroClub, IIT Delhi

> **Dates:** 9–15 Jun 2026  
> **Videos:** 5 & 6 from the playlist  
> **Points:** 15 pts  
> **Deadline:** 15 Jun 2026, 11:59 PM IST

---

## What This Week Is About

Your Week 1 model is a perfect quadcopter — it knows exactly where it is and what forces act on it. No real vehicle has this luxury. Real sensors are noisy, biased, and drift over time. Real environments have wind. This week you make your simulation realistic.

You will add four subsystems to your existing Week 1 plant:
1. **Wind disturbance model** — 3 m/s crosswind as an external force on the body
2. **IMU model** — accelerometer and gyroscope with noise, bias, and random walk
3. **Barometer model** — altitude measurement with pressure noise and bias
4. **Magnetometer model** — heading measurement with Gaussian noise

By end of week your model has realistic sensor outputs ready for the EKF in Week 4.

**Key metric:** SNR of IMU accelerometer output in dB.

---

## Videos to Watch


| # | Title | Watch before |
|---|---|---|
| Video 1 | Modelling Environments & 3D Simulation | https://youtu.be/yPA0cmXuYB4?si=tDqhgb4DAw0z346A |
| Video 2 | Modelling Virtual Sensors | https://youtu.be/di04jfUNRxE?si=jjKvei46ieS3e8ve |

---

## Tasks This Week

### Task 1 — Prepare your Week 1 model 

- Save Week 1 model as autopilot_wk2_YOUR_NAME.slx
- Add sensor output ports and wind input port at top level

### Task 2 — Wind Disturbance Model 

- 3 m/s steady crosswind + turbulence (Band-Limited White Noise)
- Applied as external force in body X-direction to Translational Dynamics

### Task 3 — IMU Model 

- Accelerometer: true accel + gravity + bias (0.05 m/s²) + noise (0.003 m/s²/sqrtHz)
- Gyroscope: true rates + bias (0.01 rad/s) + noise (0.0003 rad/s/sqrtHz) + bias instability
- Sample rate: 200 Hz

### Task 4 — Barometer Model 

- True altitude + bias (0.1 m) + noise (sigma = 0.05 m) + 50 Hz ZOH

### Task 5 — Magnetometer Model 

- True yaw + bias (0.05 rad) + noise (sigma = 0.02 rad) + 50 Hz ZOH

### Task 6 — SNR Computation & Comparison Plots 

- Run 30 s hover, log true + measured signals
- Compute SNR for IMU accelerometer X-axis
- Generate 4 comparison plots (one per sensor)

---

## What to Submit

All files in submissions/week02 — no written reports:

| # | File | Points |
|---|---|---|
| 1 | autopilot_wk2_YOUR_NAME.slx | 9 pts |
| 2 | comparison plots (4 PNGs or 1 combined) | 4 pts |
| 3 | snr_result_YOUR_NAME.png (MATLAB command window showing SNR) | 2 pts |

See submissions/SUBMISSION_GUIDE.md for full instructions.

---

## Scoring

| Criterion | Points |
|---|---|
| All 4 sensor subsystems present and connected | 4 pts |
| Correct noise parameters | 3 pts |
| Wind model causes visible X-drift in hover | 1 pt |
| Comparison plots — all 4 sensors, labelled | 4 pts |
| SNR reported and realistic (40–70 dB) | 2 pts |
| Baro and mag downsampled with ZOH at 50 Hz | 1 pt |
| **Total** | **15 pts** |

---

## Suggested Time Split (4 hours)

| Activity | Time |
|---|---|
| Videos 5 & 6 | 30 min |
| Wind model | 45 min |
| IMU model | 1 hr |
| Barometer | 30 min |
| Magnetometer | 30 min |
| SNR + plots | 45 min |

---

## Tips

- Build in order: Wind → IMU → Barometer → Magnetometer → Plots
- Label every sensor output clearly: accel_meas, gyro_meas, baro_alt_meas, mag_heading_meas
- SNR below 30 dB = noise too high. SNR above 80 dB = noise too low. Target 40–70 dB.
- Use Zero-Order Hold block (sample time 0.02 s) to downsample baro and mag to 50 Hz
- Save plots: saveas(gcf, 'plots_wk2_YOUR_NAME.png')

---

## End-of-Week Checklist

- [ ] Model saved as autopilot_wk2_YOUR_NAME.slx
- [ ] Wind, IMU, Barometer, Magnetometer subsystems all present
- [ ] All sensor outputs correctly named and labelled
- [ ] 30 s simulation runs without errors
- [ ] 4 comparison plots generated and saved
- [ ] SNR computed and in 40–70 dB range
- [ ] PR opened: [Week 2] Your Name — Submission

Resources: resources/RESOURCES.md

AeroClub, IIT Delhi · CSOT 2026
