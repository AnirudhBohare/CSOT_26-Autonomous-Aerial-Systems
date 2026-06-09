# Task 1 — Prepare Your Week 1 Model
### Week 2 · Autonomous Aerial Systems · CSOT 2026

Estimated time: 15 min

---

## Steps

1. Open autopilot_wk1_YOUR_NAME.slx in MATLAB R2026
2. Go to File → Save As → save as autopilot_wk2_YOUR_NAME.slx
   Work only in this new file from here on. Keep Week 1 clean.

3. At the top level of your model, add the following output ports (or To Workspace blocks):
   - accel_meas [3x1]
   - gyro_meas [3x1]
   - baro_alt_meas [1x1]
   - mag_heading_meas [1x1]

4. Add a Wind input subsystem placeholder at the top level connected to your Translational Dynamics subsystem.
   Inside Translational Dynamics, add an input port called F_wind [3x1] and sum it with the body force before dividing by mass.

5. Verify the base model still runs after this structural change. Fix any broken signals before proceeding.

---

## Checklist

- [ ] New file saved as autopilot_wk2_YOUR_NAME.slx
- [ ] 4 sensor output ports added at top level
- [ ] Wind input port added to Translational Dynamics
- [ ] Base model still runs without errors
