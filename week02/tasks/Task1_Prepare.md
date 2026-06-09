# Task 1 — Prepare and Model Wind 
### Week 2 · Autonomous Aerial Systems · CSOT 2026

**Estimated time:** 1 hour
**Deliverable:** Screenshot of Wind subsystem in model + scope plot of wind force signal

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


---


# Wind Modelling

---

## Why Model Wind?

Your R3 requirement states the autopilot shall maintain stable flight at 3 m/s crosswind. You cannot test R3 without a wind model. More importantly, your controller and EKF in Weeks 3–4 will be tuned under wind — if you add wind later, you'll have to retune everything. Add it now.

---

## What the Wind Model Does

The wind model adds an **external force and torque** to the quadcopter body. It has two components:

1. **Constant crosswind** — a steady 3 m/s wind in one direction, modelled as a constant external force applied to the body frame.
2. **Dryden turbulence** — stochastic gusts from the Aerospace Blockset Dryden block, adding realistic time-varying wind on top of the steady component.

The combined force is added to the translational dynamics of your Week 1 plant — specifically, it adds to the external forces before Newton's second law is applied.

---

## Physics: Wind Force on the Body

Aerodynamic drag force from a crosswind of velocity `Vw` (m/s):

```
F_drag = 0.5 × ρ × Cd × A × Vw²
```

For our quadcopter:
- Air density ρ = 1.225 kg/m³
- Drag coefficient Cd = 1.0 (bluff body approximation)
- Frontal area A = 0.04 m² (approximate for 0.5 kg class drone)
- At 3 m/s: F_drag = 0.5 × 1.225 × 1.0 × 0.04 × 9 = **0.22 N**

This force acts in the body-frame y-direction for a side crosswind.

---

## Building the Wind Subsystem in Simulink

### Step 1 — Create the Wind Subsystem

Right-click on your canvas → New Subsystem. Name it `Wind Model`. It will have:
- **Outputs:** `F_wind_x`, `F_wind_y`, `F_wind_z`, `T_wind_x`, `T_wind_y`, `T_wind_z` (forces and torques, N and N·m)

### Step 2 — Constant Crosswind Component

Inside the Wind Model subsystem:

1. Add a **Constant** block. Set value to `0.22` (the drag force computed above).
2. Route this to `F_wind_y` output (side force).
3. Set `F_wind_x`, `F_wind_z`, and all torque outputs to 0 for the constant component.

### Step 3 — Dryden Turbulence Block

1. In the Simulink Library Browser, go to: **Aerospace Blockset → Environment → Wind**
2. Drag the **Dryden Wind Turbulence Model (Continuous)** block into your Wind subsystem.
3. Configure parameters:
   - **Altitude:** connect from your plant z-output (or use Constant = 10 m for now)
   - **Airspeed:** Constant block = 3 (m/s)
   - **Wind speed at 20 ft:** 3 (m/s) — this sets gust intensity
   - **Probability of exceedance:** `10^-3` (Moderate turbulence)
4. The block outputs `[ug, vg, wg]` — gust velocities in m/s. Convert to forces:
   - Multiply each by `0.5 × ρ × Cd × A` using Gain blocks
5. Add the Dryden gust forces to the constant crosswind using Sum blocks.

### Step 4 — Connect Wind to Plant

In your top-level model, connect the Wind Model outputs to new input ports in your **Translational Dynamics** subsystem. Inside Translational Dynamics, add these forces to the aerodynamic forces before dividing by mass:

```
a_inertial = (R × F_thrust + F_wind_inertial) / m + g_vector
```

> Note: The wind force from the Dryden block is already in body frame. Rotate to inertial frame using the same rotation matrix R you built in Week 1 before adding.

---

## Checking Your Wind Model

Run a 10-second hover simulation with wind active.

In MATLAB after simulation:
```matlab
% Plot wind force signal
figure;
plot(tout, F_wind_y_log);   % from To Workspace block on F_wind_y output
xlabel('Time (s)');
ylabel('Wind Force Y (N)');
title('Wind Force Signal — Constant + Dryden Turbulence');
grid on;
```

**Expected:** A signal that starts at ~0.22 N (constant crosswind) with stochastic fluctuations around it (Dryden gusts). Not a flat line, not white noise — somewhere in between.

Save this plot as part of your `imu_plots_YOUR_NAME.png` or as a separate file.

---

## What Happens to the Drone?

With your Week 1 plant and no controller, the 3 m/s crosswind should push the drone laterally. You should see:
- y-position drifting away from zero
- small roll response as the drone is blown sideways

This is expected — the controller (Week 3) will reject this disturbance. For now, just confirm the wind force is being applied.

---

## Checklist

- [ ] Wind Model subsystem created inside your `.slx`
- [ ] Constant crosswind force (0.22 N) in body-frame y
- [ ] Dryden Wind Turbulence block configured
- [ ] Wind forces connected to Translational Dynamics
- [ ] 10-second simulation run with wind active
- [ ] Wind force scope plot saved (shows constant + stochastic component)
- [ ] Screenshot of model showing Wind subsystem connected
