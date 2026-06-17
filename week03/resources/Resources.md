# Week 3 — Resources & References
### Autonomous Aerial Systems · CSOT 2026 · AeroClub, IIT Delhi

---

## Primary Videos

| # | Title | Link |
|---|---|---|
| Video 1 | Flight Control Law Design | https://youtube.com/playlist?list=PLw9UeyR2OgE2uzMYHbvc1SLhZFiI9uM4p (Video 7) |
| Video 2 | Flight Management Design | https://youtube.com/playlist?list=PLw9UeyR2OgE2uzMYHbvc1SLhZFiI9uM4p (Video 8) |

Watch Video 1 before Task 1 (Cascaded PID). Watch Video 2 before Task 2 (Stateflow).

---

## MathWorks Official Documentation

| Resource | What it covers | Link |
|---|---|---|
| PID Controller block | All configuration options for the Simulink PID block — gain forms, anti-windup, filter | https://www.mathworks.com/help/simulink/slref/pidcontroller.html |
| Cascade PID Control | MathWorks example of cascaded loop — directly analogous to altitude-over-attitude | https://www.mathworks.com/help/control/ug/multiloop-control-design.html |
| Stateflow Getting Started | Official tutorial: creating charts, states, transitions, and data | https://www.mathworks.com/help/stateflow/getting-started-with-stateflow.html |
| Stateflow `after` operator | How `after(N, sec)` works as a time-based transition condition | https://www.mathworks.com/help/stateflow/ref/after.html |
| Tune PID with MATLAB | Auto-tuning approach using `pidtune()` if manual tuning is slow | https://www.mathworks.com/help/control/ref/pidtune.html |

---

## PID Gain Tuning Reference

### Manual Tuning (Ziegler–Nichols Ultimate Gain Method)

For the inner attitude loop:

1. Set Ki = 0, Kd = 0. Increase Kp until the system oscillates continuously (critical gain Ku, critical period Tu).
2. Then set: Kp = 0.6·Ku, Ki = 1.2·Ku/Tu, Kd = 0.075·Ku·Tu

This is a starting point — always reduce gains from here for robustness.

### Frequency-Domain Method

If you have the Control System Toolbox:
```matlab
% Linearise the attitude loop at hover trim
[A, B, C, D] = linmod('autopilot_wk3_YOUR_NAME');
sys = ss(A, B, C, D);

% Design PID for roll axis (input 1 = tau_roll, output = phi)
sys_roll = sys(phi_output_index, tau_roll_input_index);
C_pid = pidtune(sys_roll, 'PID');
fprintf('Kp=%.3f Ki=%.3f Kd=%.3f\n', C_pid.Kp, C_pid.Ki, C_pid.Kd);
```

### Gain Starting Points (from physics)

These are rough values derived from the drone inertia. Use as a starting point only:

**Inner attitude loop (Ixx = Iyy = 0.0049 kg·m²):**
- Natural frequency ωn ≈ 10 rad/s → Kp ≈ Ixx · ωn² ≈ 0.49 → use ~4.0 (aggressive hover)
- Damping ζ = 0.7 → Kd ≈ 2·ζ·ωn·Ixx ≈ 0.069 → use ~0.8

**Outer altitude loop (mass = 0.5 kg):**
- Slower: ωn ≈ 2 rad/s → Kp ≈ m · ωn² / kT_effective ≈ tuning empirically

---

## Stateflow Reference Card

```
State syntax:
┌──────────────────────────────┐
│ STATE_NAME                   │
├──────────────────────────────┤
│ entry: code executed once    │
│        on entering the state │
├──────────────────────────────┤
│ during: code executed every  │
│         time step            │
├──────────────────────────────┤
│ exit:  code executed once    │
│        on leaving the state  │
└──────────────────────────────┘

Transition syntax:
STATE_A ──[condition]──► STATE_B
STATE_A ──[after(5, sec)]──► STATE_B    % time-based
STATE_A ──[event_name]──► STATE_B       % event-based

Local data:
Right-click chart → Add Data → Local → type double
Use for: t_takeoff_start, t_land_start
```

---

## Reference Papers (Optional)

### 1 — Cascaded PID for Quadcopter
**Title:** Design and Implementation of a PD/PID-Controlled Quadrotor
**Why read it:** Section 3 derives the same cascaded PID structure you are building, with the inner/outer loop separation explained mathematically.
**Source:** ResearchGate (search the title)

### 2 — PX4 Flight Mode Architecture
**Title:** PX4 Flight Stack Documentation — Commander Module
**Why read it:** Shows how an industry flight controller (used in real drones) implements mode transitions. Directly analogous to your Stateflow chart.
**Link:** https://docs.px4.io/main/en/concept/flight_modes.html

### 3 — Simulink-Based Quadcopter Control (Practical)
**Title:** Modelling and Attitude Control of Quadrotor in Simulink/MATLAB
**Why read it:** Very close to your implementation — complete Simulink model with attitude PID. Figures show how subsystems are connected.
**Source:** IEEE Xplore or ResearchGate

---

## Physical Parameters Quick Reference

```
m  = 0.5    kg          (mass)
g  = 9.81   m/s²        (gravity)
L  = 0.225  m           (arm length, centre to rotor)
kT = 2.9e-5 N/(rad/s)²  (thrust coefficient)
kQ = 1.1e-6 N·m/(rad/s)² (torque coefficient)
Ixx = Iyy = 0.0049 kg·m²
Izz = 0.0088 kg·m²

Hover rotor speed: ω_hover = sqrt(m*g / (4*kT)) ≈ 205.63 rad/s
Hover thrust per motor: F_hover = kT * ω_hover² = m*g/4 = 1.226 N
Total hover thrust: Fz = m*g = 4.905 N
```

---

## Getting Help

GitHub Issues: tag `week3-question`
AeroClub contact: CSOT 2026 communication channel
