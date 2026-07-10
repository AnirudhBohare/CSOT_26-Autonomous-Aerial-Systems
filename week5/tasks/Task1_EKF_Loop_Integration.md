# Task 1 — EKF-in-the-Loop Integration
### Week 5 · Autonomous Aerial Systems · CSOT 2026

>**Estimated time:** 2 hours

>**Deliverable:** Complete closed-loop autopilot with all true plant state wires replaced by EKF estimates; EKF x/y position and velocity estimates routed to new outer PIDs; 60-second no-crash simulation.

>**Watch first:** Video 1 (Full Autopilot Integration & Validation)

---

## What Changes This Week

In Week 4 you implemented the EKF and connected it to the altitude and attitude PID feedback paths. This task verifies that integration is complete and extends it to horizontal position.

### Checklist of true state wires to replace

Open your Week 4 model and trace every PID feedback signal. Confirm that each of the following comes from the EKF (via Unit Delay), NOT from the plant directly:

| Signal | PID that uses it | EKF index | Status to verify |
|---|---|---|---|
| z (altitude) | Altitude PID | x_hat_delayed(3) | Should already be done from Wk 4 |
| φ (roll) | Roll PID inner | x_hat_delayed(7) | Should already be done from Wk 4 |
| θ (pitch) | Pitch PID inner | x_hat_delayed(8) | Should already be done from Wk 4 |
| ψ (yaw) | Yaw PID inner | x_hat_delayed(9) | Should already be done from Wk 4 |
| x (North pos) | X Position PID | x_hat_delayed(1) | **NEW this week** |
| y (East pos) | Y Position PID | x_hat_delayed(2) | **NEW this week** |

If any signal in the first four rows still comes from the plant directly (true state wire), fix it before adding the position loop.

---

## Step 1 — Audit and Complete the Wk4 EKF Loop

Before adding anything new, do a quick audit:

1. Right-click the wire going into the Altitude PID's feedback port → **Highlight Signal** → trace it back. It should terminate at the Unit Delay output, not at Translational Dynamics directly.
2. Repeat for Roll, Pitch, Yaw PID feedback ports.
3. Verify the Unit Delay has `Initial condition = zeros(9,1)` and `Sample time = 0.001`.

If any true state wire is still connected, delete it and add a **Selector** block referencing the Unit Delay output instead.

---

## Step 2 — Add Position and Velocity Logging

The EKF x/y position estimates (x_hat indices 1 and 2) are needed by the Task 2 position PIDs. Also log them for Task 3 RMSE computation.

Add **To Workspace** blocks:

```
Unit Delay output (x_hat_delayed, 9x1)
    │
    ├── Selector: index 1 ──► [To Workspace: x_hat_x, timeseries]
    ├── Selector: index 2 ──► [To Workspace: x_hat_y, timeseries]
    ├── Selector: index 4 ──► [To Workspace: x_hat_vx, timeseries]
    └── Selector: index 5 ──► [To Workspace: x_hat_vy, timeseries]
```

Also add logging for the true x/y from Translational Dynamics (for RMSE comparison in Task 3):

```
Translational Dynamics output: x (port 4) ──► [To Workspace: x_true, timeseries]
Translational Dynamics output: y (port 5) ──► [To Workspace: y_true, timeseries]
```

---

## Step 3 — Verify Stateflow Inputs Are Still Correct

The Stateflow chart outputs z_setpoint which feeds the Altitude PID. Make sure:
- Clock still connects to Chart input t
- arm_cmd (Constant = 1) still connects to Chart
- Chart output z_setpoint still reaches Controller Subsystem port 3

If you restructured the Controller Subsystem in Week 4, double-check these paths haven't broken.

---

## Step 4 — Add Selector Blocks for x/y Position

Position PIDs need x_hat(1) and x_hat(2) as feedback. Add two Selector blocks on the Unit Delay output:

| Selector block | Index | Feeds |
|---|---|---|
| `sel_x` | 1 | X Position PID (Task 2) |
| `sel_y` | 2 | Y Position PID (Task 2) |

Leave the outputs of these Selectors unconnected for now — you will wire them in Task 2.

**Selector block settings:**
- Input port dimensions: 9 (matches 9×1 EKF output)
- Index mode: One-based
- Index: 1 (or 2)
- Output size: -1 (scalar)

---

## Step 5 — Smoke Test (No Position PIDs Yet)

Before wiring the position loop, confirm the existing altitude + attitude loop still works:

1. Set simulation time = 40 s
2. Run
3. Verify:
   - `mode` workspace variable sequences 1 → 2 → 3 → 4
   - `z` workspace variable reaches −1.0 m during hover (mode = 3)
   - No Simulink warning about algebraic loops

**Expected z trace:** starts at 0, ramps to −1.0 m by ~5 s, holds for 30 s, ramps back.

If the drone crashes at t=0: the most common cause is that re-wiring the EKF feedback introduced a sign error in the altitude error computation. Check that z_setpoint (−1.0) and z_hat (−1.0 at hover) produce near-zero error.

---

## Step 6 — Increase Simulation Time to 60 seconds

The Week 5 simulation is 60 seconds instead of 40:
- t = 0–5 s: IDLE + TAKEOFF
- t = 5–35 s: HOVER (30-second evaluation window)
- t = 35–55 s: LAND
- t = 55–60 s: ground (motors off)

Change the Stateflow `after(30, sec)` in the HOVER state to `after(30, sec)` (keeping at 30 s hover) and set simulation **Stop Time = 60** in Model Settings.

---

## Common Mistakes

| Mistake | Symptom | Fix |
|---|---|---|
| True z still wired alongside EKF z | Altitude holds but EKF z shows offset | Check for duplicate wires; Simulink sums signals on the same port |
| Unit Delay initial condition is a scalar 0 | First step crashes | Set initial condition to `zeros(9,1)` — must be column vector |
| Selector index wrong | Position loop commands wrong direction | EKF state: x=1, y=2, z=3, vx=4, vy=5, vz=6, φ=7, θ=8, ψ=9 |
| To Workspace name collision with Wk4 | Workspace gets the wrong variable | Rename new logs x_hat_x, x_hat_y, x_true, y_true to avoid conflicts |

---

## Checklist

- [ ] All 4 attitude/altitude PID feedbacks confirmed as EKF-sourced
- [ ] Unit Delay: sample time 0.001, initial condition zeros(9,1)
- [ ] Selector blocks added for index 1 (x) and 2 (y) on Unit Delay output
- [ ] To Workspace added: x_hat_x, x_hat_y, x_hat_vx, x_hat_vy, x_true, y_true
- [ ] Stateflow inputs (t, arm_cmd, z_setpoint) all intact
- [ ] Smoke test: 40-second sim passes, z holds at −1.0 m
- [ ] Simulation Stop Time updated to 60 s
