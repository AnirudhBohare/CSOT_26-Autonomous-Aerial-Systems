# Task 3 — Build the 6-DOF Flight Dynamics Model
### Week 1 · Autonomous Aerial Systems · CSOT 2026

**Estimated time:** 2.5 hours

**Deliverable:** `autopilot_wk1_YOUR_NAME.slx` + trim results (10 pts)

**Watch first:** Video Resources 

---

## What Is a 6-DOF Model?

6-DOF stands for 6 Degrees of Freedom — a rigid body moving in 3D space has 3 translational DOFs (x, y, z) and 3 rotational DOFs (roll, pitch, yaw). Your Simulink model will compute how these 6 states evolve over time given rotor thrust inputs.

This is the **plant** in your control system. Every controller, sensor, and estimator you build in Weeks 2–4 connects to this model. Get it right now.

---

## Physics Background

### Coordinate Frame
Use the **NED (North-East-Down)** body frame convention:
- x → forward (North)
- y → rightward (East)  
- z → downward (Down) ← gravity acts in **+z** direction in NED

### Rotor Layout (+ configuration)
```
        Motor 1 (Front) — CCW
              |
Motor 4 (Left) — CW  ←——[BODY]——→  Motor 2 (Right) — CW
              |
        Motor 3 (Rear) — CCW
```
- Motors 1 & 3 spin **counter-clockwise (CCW)**
- Motors 2 & 4 spin **clockwise (CW)**
- This alternating pattern cancels yaw torque during hover

### Thrust and Torque Per Rotor
Each rotor i produces:
- Thrust: `Fi = kT × ωi²` (upward, in body -z direction)
- Reaction torque: `Qi = kQ × ωi²` (sign depends on spin direction)

### Force & Torque Mixing
Total force and torques on the body:

```
Fz (total thrust) = kT × (ω1² + ω2² + ω3² + ω4²)

τ_roll  = kT × L × (ω4² - ω2²)           [left-right difference]
τ_pitch = kT × L × (ω1² - ω3²)           [front-rear difference]
τ_yaw   = kQ × (ω1² + ω3² - ω2² - ω4²)  [CCW minus CW]
```

Where L is the arm length (centre to rotor).

### Equations of Motion

**Translational (in NED inertial frame):**
```
ẍ = (Fz/m) × (sin(pitch)·cos(yaw) + sin(roll)·sin(yaw)·cos(pitch)) ... [from rotation matrix]
ÿ = ...
z̈ = g - (Fz/m) × cos(roll)·cos(pitch)
```

In Simulink, use the **Rotation Matrix** block to transform body-frame thrust to inertial frame, then apply Newton's second law.

**Rotational (body frame — Euler's equations):**
```
ṗ = (τ_roll  - (Izz - Iyy)·q·r) / Ixx
q̇ = (τ_pitch - (Ixx - Izz)·p·r) / Iyy
ṙ = (τ_yaw   - (Iyy - Ixx)·p·q) / Izz
```

**Kinematics (body rates to Euler angle rates):**
```
φ̇ (roll rate)  = p + (q·sin(φ) + r·cos(φ))·tan(θ)
θ̇ (pitch rate) = q·cos(φ) - r·sin(φ)
ψ̇ (yaw rate)   = (q·sin(φ) + r·cos(φ)) / cos(θ)
```

---

## Simulink Model Structure

Build your model as **4 subsystems connected in series.** Do not build it as one flat block diagram — it will become unmanageable by Week 3.

```
[ω1, ω2, ω3, ω4] → [Force & Torque Mixer] → [Fz, τ_roll, τ_pitch, τ_yaw]
                                                          ↓
                                          [Translational Dynamics] → [x, y, z, vx, vy, vz]
                                          [Rotational Dynamics]    → [p, q, r]
                                          [Kinematics]             → [φ, θ, ψ]
```

### Subsystem 1 — Force & Torque Mixer
**Inputs:** ω1, ω2, ω3, ω4 (rad/s)
**Outputs:** Fz (N), τ_roll (N·m), τ_pitch (N·m), τ_yaw (N·m)

This is pure algebra — just the mixing equations above. Use Gain blocks and Sum blocks. No integrators here.

### Subsystem 2 — Translational Dynamics
**Inputs:** Fz, φ, θ, ψ (from Kinematics — use an initial value for first run)
**Outputs:** x, y, z, ẋ, ẏ, ż

Steps:
1. Use a **Fcn** or **MATLAB Function** block to compute the rotation matrix R(φ, θ, ψ)
2. Multiply body-frame thrust vector [0; 0; -Fz] by R to get inertial-frame force
3. Add gravity [0; 0; g·m] in NED
4. Divide by mass → acceleration
5. Integrate twice → velocity, position

### Subsystem 3 — Rotational Dynamics
**Inputs:** τ_roll, τ_pitch, τ_yaw, p, q, r (feedback)
**Outputs:** ṗ, q̇, ṙ → integrate → p, q, r

Use Euler's equations directly. Implement the cross-product coupling terms (Izz-Iyy)·q·r etc. with Product and Gain blocks.

### Subsystem 4 — Kinematics
**Inputs:** p, q, r, φ, θ (feedback)
**Outputs:** φ̇, θ̇, ψ̇ → integrate → φ, θ, ψ

Implement the kinematic equations. Use **Trigonometric Function** blocks for sin, cos, tan. Be careful with tan(θ) — it is singular at θ = ±90°, which is fine for this project (we never fly inverted).

---

## Step-by-Step Build Order

Follow this order — it lets you test each piece before connecting:

**Step 1:** Create a new Simulink model. Save it immediately as `autopilot_wk1_YOUR_NAME.slx`.

**Step 2:** Set solver to **Fixed-step, ode4 (Runge-Kutta), step size = 0.001 s**. (Simulation → Model Settings → Solver)

**Step 3:** Build Subsystem 1 (Mixer). Test it: feed constant ω = 500 rad/s to all 4 inputs. Fz should be `4 × kT × 500² = 29 N`. τ values should all be 0.

**Step 4:** Build Subsystem 3 (Rotational Dynamics). Test: feed zero torques, zero initial rates. Output should stay at zero.

**Step 5:** Build Subsystem 4 (Kinematics). Connect output of Sub 3. Test with zero input — angles should stay at zero.

**Step 6:** Build Subsystem 2 (Translational Dynamics). Connect Sub 1 output and Sub 4 output. Test with hover thrust — altitude should not change.

**Step 7:** Connect all 4 subsystems. Add **To Workspace** blocks on x, y, z, φ, θ, ψ outputs to log data.

**Step 8:** Add **Scope** blocks on key signals for visual inspection.

---

## Hover Trim Analysis

### What is Trim?
Trim is the equilibrium condition where the quadcopter hovers at constant altitude with zero velocity and zero angular rates. At trim, all accelerations are zero.

### Computing Analytical Hover Thrust
At hover, the total thrust equals weight:
```
4 × kT × ω_hover² = m × g
ω_hover = sqrt((m × g) / (4 × kT))
        = sqrt((0.5 × 9.81) / (4 × 2.9×10⁻⁵))
        = sqrt(42284)
        ≈ 205.6 rad/s
```

### Running the Trim Simulation
1. Set all 4 rotor inputs to `ω_hover = 205.6 rad/s` using Constant blocks.
2. Set all initial conditions to zero (position = [0,0,0], attitude = [0,0,0], rates = [0,0,0]).
3. Run simulation for **5 seconds**.
4. Check the Scope on z — it must stay at 0 ± 0.01 m.
5. Check roll and pitch — must stay at 0 ± 0.01°.

### Computing % Error
In the MATLAB command window after simulation:
```matlab
omega_hover_analytical = sqrt((0.5 * 9.81) / (4 * 2.9e-5));

% Read your simulated hover rotor speed (the value you had to set to achieve hover)
omega_hover_simulated = 205.6; % replace with your value if different

pct_error = abs(omega_hover_simulated - omega_hover_analytical) / omega_hover_analytical * 100;
fprintf('Hover thrust % error: %.2f%%\n', pct_error);
```

Target: **< 5% error**.

### Trim Results Table
Create a file `trim_results_YOUR_NAME.md` with this table:

| Parameter | Analytical | Simulated | % Error |
|---|---|---|---|
| Hover rotor speed ω (rad/s) | 205.6 | ___ | ___% |
| Altitude z after 5 s (m) | 0.000 | ___ | — |
| Roll φ after 5 s (°) | 0.000 | ___ | — |
| Pitch θ after 5 s (°) | 0.000 | ___ | — |

Include a screenshot of your z-altitude Scope output showing it is flat.

---

## Common Mistakes

| Mistake | Symptom | Fix |
|---|---|---|
| Wrong sign on gravity in NED | Drone accelerates upward instead of hovering | In NED, gravity is +g in z direction. Check your sign convention. |
| Missing cross-coupling in rotational dynamics | Model diverges when rotated | Add the (Izz-Iyy)·q·r terms — they matter at non-zero rates |
| Integrator initial conditions not set | Model starts with garbage state | Double-click each Integrator block → set Initial condition to 0 |
| Solver step too large | Simulation oscillates or blows up | Use ode4, step size 0.001 s |
| Flat model (no subsystems) | -2 pts on structure criterion | Group related blocks into subsystems using Ctrl+G |

---

## Checklist

- [ ] Model saved as `autopilot_wk1_YOUR_NAME.slx`
- [ ] Solver set to ode4, fixed-step, 0.001 s
- [ ] 4 subsystems created: Mixer, Translational, Rotational, Kinematics
- [ ] All subsystem inputs/outputs correctly labelled
- [ ] Hover trim simulation run for 5 seconds
- [ ] z-altitude stays flat (< ±0.01 m deviation)
- [ ] % error computed and < 5%
- [ ] Trim results table saved as separate file
- [ ] Scope screenshot of flat altitude included
