# Week 5 — Resources & References
### Autonomous Aerial Systems · CSOT 2026 · AeroClub, IIT Delhi

---

## Primary Video

| # | Title | Link |
|---|---|---|
| Video 1 | Full Autopilot Integration & Validation | https://youtu.be/m1t5lDWMGk8?si=xaG1kjBRABCT8QhW |

Watch before Task 1.

---

## MathWorks Official Documentation

| Resource | What it covers | Link |
|---|---|---|
| `sim()` function | Running Simulink models from a MATLAB script | https://www.mathworks.com/help/simulink/slref/sim.html |
| `set_param` / `get_param` | Changing block parameters programmatically | https://www.mathworks.com/help/simulink/slref/set_param.html |
| `find_system` | Finding blocks by type or parameter | https://www.mathworks.com/help/simulink/slref/find_system.html |
| PID Controller block | Saturation, anti-windup, initial conditions | https://www.mathworks.com/help/simulink/slref/pidcontroller.html |
| Selector block | Indexing into vector signals | https://www.mathworks.com/help/simulink/slref/selector.html |
| Signal logging | To Workspace block and Simulink.SimulationOutput | https://www.mathworks.com/help/simulink/ug/logging-signals.html |

---

## Key Reference Papers

### 1 — Cascaded PID for Quadrotor Position Control (Top-Rated)
**Title:** Design of a Cascaded PID Controller for Quadrotor Position Control
**Why read it:** Section 3 derives exactly the X→θ, Y→φ cascade used in Task 2. Figure 3 shows the complete outer-inner loop structure. Section 4 covers gain tuning and saturation design.
**Keywords to search:** "quadrotor cascaded position control PID theta setpoint" on Google Scholar

### 2 — MDPI Remote Sensing 2019 (Carry-forward from Week 4)
**Title:** Quadrotor Localisation and Flight Control Using Inertial and Magnetic Sensors
**DOI:** https://www.mdpi.com/2072-4292/11/19/2251
**Why still relevant:** Section 6 covers the full closed-loop validation methodology — including position hold, Monte Carlo analysis framework, and statistical RMSE reporting — exactly what Tasks 3 and Bonus require.

### 3 — Monte Carlo Methods in Flight Control Validation
**Title:** Statistical Validation of Autopilot Systems Using Monte Carlo Simulation
**Why read it:** Explains why pass rate > 95% is the standard criterion (not mean RMSE alone). Covers seed selection, correlation between runs, and how to determine the minimum number of runs needed for statistical significance.
**Source:** IEEE Aerospace Conference proceedings (search Google Scholar)

---

## Simulink Techniques Quick Reference

### Running a sim from a script
```matlab
simOut = sim('model_name', 'StopTime', '60');
x = squeeze(simOut.variable_name.Data);
t = simOut.variable_name.Time;
```

### Changing BLWN seed programmatically
```matlab
set_param('model/path/to/block', 'seed', '12345');
% Verify it was set:
current_seed = get_param('model/path/to/block', 'seed');
```

### Finding all BLWN blocks
```matlab
blocks = find_system('model_name', 'BlockType', 'Reference', ...
                     'SourceBlock', 'simulink/Sources/Band-Limited White Noise');
```

---

## Physical Parameters (Final Reference)

```
m  = 0.5    kg           (mass)
g  = 9.81   m/s^2        (gravity)
L  = 0.225  m            (arm length)
kT = 2.9e-5 N/(rad/s)^2  (thrust coefficient)
kQ = 1.1e-6 N.m/(rad/s)^2 (torque coefficient)
Ixx = Iyy = 0.0049 kg.m^2
Izz = 0.0088 kg.m^2

Hover thrust   Fz  = m*g = 4.905 N
Hover setpoint z   = -1.0 m (NED: negative = above ground)
Position setpoints x = 0, y = 0
Wind disturbance   F_wind = [0.22; 0; 0] N (North direction)
```

---

## Full 5-Week Deliverable Summary

| Week | Key deliverable | Points |
|---|---|---|
| Week 1 | `autopilot_wk1.slx` — 6-DOF plant | 20 pts |
| Week 2 | `autopilot_wk2.slx` — sensors + noise | 15 pts |
| Week 3 | `autopilot_wk3.slx` — PID + Stateflow | 25 pts |
| Week 4 | `autopilot_wk4.slx` — EKF sensor fusion | 20 pts |
| Week 5 | `autopilot_wk5.slx` — full integration | 25 pts + 20 bonus |
| **Total** | | **105 pts + 20 bonus = 125 pts** |

---

## Getting Help

GitHub Issues: tag `week5-question`
AeroClub contact: CSOT 2026 communication channel
ENDOFFILE
