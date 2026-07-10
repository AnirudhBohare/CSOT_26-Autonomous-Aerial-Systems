# Task 2 — X/Y Position Hold
### Week 5 · Autonomous Aerial Systems · CSOT 2026

>**Estimated time:** 1.5 hours

>**Deliverable:** Two outer position PIDs (X → θ_setpoint, Y → φ_setpoint) cascaded into the existing inner attitude loop; drone holds x=0, y=0 within ±0.5 m RMS under wind.

---

## Why Pitch Controls North, Roll Controls East

A quadcopter moves horizontally by tilting. In NED coordinates with zero yaw:

- **Pitching forward (θ > 0)** tilts the thrust vector to produce a North (+x) force component → drone accelerates North.
- **Rolling right (φ > 0)** tilts the thrust vector to produce an East (+y) force component → drone accelerates East.

So to hold position against a North wind (+x disturbance), the X PID must command a **positive θ** (nose-down pitch) to produce opposing thrust. The mapping is:

```
X position error → X PID → θ_setpoint → Pitch PID → τ_pitch → North acceleration
Y position error → Y PID → φ_setpoint → Roll PID  → τ_roll  → East acceleration
```

This is the outer-inner cascade for position hold. The inner attitude PIDs from Week 3 remain unchanged.

**Sign check:** When the drone drifts North (x_hat(1) > x_setpoint=0), the error is negative (0 − x_hat(1) < 0), the X PID outputs a negative θ_setpoint (nose-up), the Pitch PID produces negative τ_pitch, and the drone decelerates northward then reverses. Verify this chain — a sign error here produces diverging oscillation.

---

## Step 1 — Add the X Position PID

Open the Controller Subsystem. Inside it, add a **PID Controller** block named `X_Position_PID`:

| Parameter | Value |
|---|---|
| Controller type | PID |
| Form | Parallel |
| Kp | 0.4 |
| Ki | 0.02 |
| Kd | 0.6 |
| Filter coefficient N | 20 |
| Sample time | −1 (continuous) |
| Output saturation | ±0.26 rad (±15°) — limits max pitch command |

**Input wiring:**
```matlab
error_x = x_setpoint − x_hat_delayed(1)
         = Constant(0) − Selector_output_x
```

Use a Sum block with signs `+-` (or just Constant(0) and subtract):

```
[Constant: 0] ──┐
                Sum (|+-) ──► [X_Position_PID] ──► [Saturation ±0.26] ──► theta_setpoint
[sel_x: x_hat(1)] ──┘
```

---

## Step 2 — Add the Y Position PID

Add a second **PID Controller** block named `Y_Position_PID`:

| Parameter | Value |
|---|---|
| Kp | 0.4 |
| Ki | 0.02 |
| Kd | 0.6 |
| N | 20 |
| Output saturation | ±0.26 rad (±15°) |

**Input wiring:**
```
[Constant: 0] ──┐
                Sum (|+-) ──► [Y_Position_PID] ──► [Saturation ±0.26] ──► phi_setpoint
[sel_y: y_hat(2)] ──┘
```

**Sign check for Y axis:** In NED, +y is East. When the drone drifts East (y_hat(2) > 0), error = 0 − y > 0, Y PID outputs positive φ_setpoint, Roll PID produces positive τ_roll, which rolls the drone right... but rolling right in NED causes East acceleration, making it drift MORE East. This is the wrong sign.

**Fix:** The Y PID output needs a **Gain = −1** before feeding φ_setpoint:

```
[Y_Position_PID] ──► [Gain: -1] ──► [Saturation ±0.26] ──► phi_setpoint
```

With this negation: positive y error → positive PID output → negative φ_setpoint (roll left) → leftward thrust → drone corrects back toward y=0. ✓

---

## Step 3 — Replace the Attitude Setpoint Constants

Currently the inner attitude PID receives `[phi_setpoint=0, theta_setpoint=0, psi_setpoint=0]` as a constant. Replace phi_setpoint and theta_setpoint with the position PID outputs:

**Before (Week 4):**
```
Constant([0;0;0]) ──► Reshape ──► Inner Attitude PID port 2 (orientation_setpoint)
```

**After (Week 5):**
```
phi_setpoint   (from Y_Position_PID → Gain(-1) → Saturation) ──┐
theta_setpoint (from X_Position_PID → Saturation)             ──┤ Mux([1;1;1]) ──► Reshape ──► Attitude PID port 2
Constant(0) for psi                                            ──┘
```

If the orientation_setpoint is currently built as a 3×1 from a Mux and Reshape, simply replace the two constant inputs with the PID outputs. The psi setpoint stays as Constant(0) (no yaw rate command).

---

## Step 4 — Tune Position PID Gains

Initial gains (Kp=0.4, Kd=0.6, Ki=0.02) are conservative. Tune by observing the x position trace:

**Tuning procedure:**
1. Run 60-second simulation, wind ON.
2. Plot `x_hat_x` vs time over the hover window.
3. If x oscillates with growing amplitude → reduce Kp or increase Kd.
4. If x has large steady-state offset (> 0.5 m mean) → increase Ki (max 0.05).
5. If y is worse than x → recheck the Y PID sign (Gain=−1 must be present).

**Expected position behaviour:** With 0.22 N wind in the +x direction, the drone will maintain ~0.1–0.3 m mean offset in x at hover with these gains. This is within the R6 target.

**Decoupling check:** Verify that changing X PID gains does not affect z. If altitude oscillates when you adjust position gains, the Fz path has been accidentally disturbed — check the signal routing inside the Controller Subsystem.

---

## Step 5 — Velocity Damping (Optional Improvement)

Pure position PID can oscillate because it has no velocity feedback between the position outer loop and the pitch angle inner loop. Add a derivative path directly on the EKF velocity estimate:

```matlab
theta_setpoint = X_PID(error_x) - Kd_vel * x_hat_delayed(4)  % vx damping
phi_setpoint   = Y_PID(error_y) + Kd_vel * x_hat_delayed(5)  % vy damping (note sign)
```

Implement as a weighted sum after the PID output:

```
[X_PID output] ──┐
                  Sum ──► [Saturation] ──► theta_setpoint
[x_hat_vx × (-0.3)] ──┘
```

Start with Kd_vel = 0.3. This damps the velocity oscillation without changing the position control structure. Increase to 0.5 if oscillation persists.

---

## Step 6 — Verify Position Hold Under Wind

Run a 60-second simulation:

```matlab
% Quick visual check after sim
t_vec = out.x_hat_x.Time;
x_est = squeeze(out.x_hat_x.Data);
y_est = squeeze(out.x_hat_y.Data);
z_est = squeeze(out.x_hat_log.Data);
if ndims(out.x_hat_log.Data) == 3
    z_est = squeeze(out.x_hat_log.Data); z_est = z_est(3,:)';
end

hover_mask = (t_vec >= 5) & (t_vec <= 35);
fprintf('Hover window position stats:\n');
fprintf('  x: mean=%.3f m  std=%.3f m\n', mean(x_est(hover_mask)), std(x_est(hover_mask)));
fprintf('  y: mean=%.3f m  std=%.3f m\n', mean(y_est(hover_mask)), std(y_est(hover_mask)));
```

**Pass criteria:** mean |x| < 0.5 m AND mean |y| < 0.5 m over hover window.

---

## Common Mistakes

| Mistake | Symptom | Fix |
|---|---|---|
| Y PID Gain(−1) missing | Drone drifts East with growing speed | Add Gain=−1 between Y_PID output and phi_setpoint |
| Saturation limits in degrees not radians | θ_setpoint = ±0.26 instead of ±15° | Set saturation to ±0.26 rad (= ±15 × pi/180) |
| theta_setpoint overrides altitude PID | z oscillates badly | The altitude PID controls Fz; theta_setpoint only goes to the inner Pitch PID |
| x_hat positions drifting even at hover | EKF IMU drift accumulating | Expected — position PIDs correct for this. If x drifts > 2 m/min, check Q(1:3) in EKF |
| Position PID Kp too large | Growing oscillation in x/y | Reduce Kp to 0.2, re-tune; position loop has slow dynamics (bandwidth ~0.3 Hz) |
| psi_setpoint replaced by accident | Drone yaws continuously | Confirm the Mux feeding orientation_setpoint has Constant(0) on port 3 |

---

## Checklist

- [ ] X_Position_PID block added: Kp=0.4, Ki=0.02, Kd=0.6, sat=±0.26 rad
- [ ] Y_Position_PID block added: same gains
- [ ] Y PID output passes through Gain=−1 before saturation
- [ ] theta_setpoint from X PID replaces the Constant(0) in orientation_setpoint Mux port 2
- [ ] phi_setpoint from Y PID replaces the Constant(0) in orientation_setpoint Mux port 1
- [ ] psi_setpoint remains Constant(0) in Mux port 3
- [ ] (Optional) velocity damping: −0.3×vx added to theta_setpoint, +0.3×vy to phi_setpoint
- [ ] 60-second sim runs without divergence
- [ ] Hover x RMSE < 0.5 m, y RMSE < 0.5 m
