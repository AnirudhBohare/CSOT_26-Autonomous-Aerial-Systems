# Task 2 — EKF MATLAB Function Block in Simulink
### Week 4 · Autonomous Aerial Systems · CSOT 2026

>**Estimated time:** 2 hours

>**Deliverable:** Working EKF subsystem in your `.slx`, tested open-loop.

>**Prerequisites:** Task 1 complete (you have the math ready to code).

---

## Architecture

The EKF runs as a MATLAB Function block with two Unit Delay blocks feeding back the previous state and covariance:

```
                     ┌────────────────────────────────┐
  accel_meas (3×1) ─►│                                │
  gyro_meas  (3×1) ─►│      EKF MATLAB Function       │─► x_est (9×1)
  baro_alt_meas(1) ─►│       [predict + update]       │─► P_est (9×9)
  mag_psi_meas (1) ─►│                                │
                     └────────────┬───────────────────┘
                                  │
                         x_prev (9×1) ◄── [Unit Delay, IC=x0]  ◄── x_est
                         P_prev (9×9) ◄── [Unit Delay, IC=P0]  ◄── P_est
```

The Unit Delay blocks create a one-step memory — they hold the previous output and feed it back as input. This converts the algebraic feedback loop into a delay-free discrete system.

---

## Step 1 — Prepare Sensor Inputs

From your Week 2 model, the following signals are already available:
- `accel_meas` (3×1, m/s²) — from IMU subsystem
- `gyro_meas` (3×1, rad/s) — from IMU subsystem
- `baro_alt_meas` (1×1, m, positive up) — from Barometer subsystem
- `mag_psi_meas` (1×1, rad) — magnetometer heading output

**Combining for EKF:**
At the top level, use a **Mux** block to combine:
```
u      = [accel_meas; gyro_meas]    → 6×1 Mux → EKF input port 1
z_meas = [baro_alt_meas; mag_psi_meas] → 2×1 Mux → EKF input port 2
```

If your magnetometer does not output psi directly, use this approximation from the Week 2 mag model:
```matlab
mag_psi_meas = atan2(mag_meas(2), mag_meas(1));   % heading from x,y components
```
Add a MATLAB Function block or embedded code for this if needed.

---

## Step 2 — Create the EKF Subsystem

1. In your top-level model, add a new **Subsystem** block. Label it `EKF_Sensor_Fusion`.
2. Inside the subsystem, place one **MATLAB Function** block. Label it `EKF_Core`.
3. Configure MATLAB Function block inputs:
   - `u` — 6×1 double (accelerometer + gyro)
   - `z_meas` — 2×1 double (baro altitude + mag heading)
   - `x_prev` — 9×1 double (previous state from Unit Delay)
   - `P_prev` — 9×9 double (previous covariance from Unit Delay)
4. Configure outputs:
   - `x_est` — 9×1 double
   - `P_est` — 9×9 double

---

## Step 3 — MATLAB Function Block Code

Paste the following into the MATLAB Function block editor:

```matlab
function [x_est, P_est] = ekf_core(u, z_meas, x_prev, P_prev)
% 9-state EKF for quadcopter sensor fusion
% State:    x = [px;py;pz;vx;vy;vz;phi;theta;psi]
% Input:    u = [ax_b;ay_b;az_b;p;q;r]  (IMU body-frame)
% Meas:     z = [baro_alt; mag_psi]
% Frame:    NED (z positive down, altitude = -pz)

%% --- Constants ---
g  = 9.81;
Ts = 0.001;   % sample time — must match model step size

Q = diag([0.01, 0.01, 0.01, ...     % position process noise
          0.10, 0.10, 0.10, ...     % velocity process noise
          0.01, 0.01, 0.01]);       % attitude process noise

R = diag([0.50, 0.10]);             % [baro variance, mag heading variance]

%% --- Unpack previous state ---
px  = x_prev(1);  py  = x_prev(2);  pz  = x_prev(3);
vx  = x_prev(4);  vy  = x_prev(5);  vz  = x_prev(6);
phi = x_prev(7);  th  = x_prev(8);  psi = x_prev(9);

%% --- Unpack IMU input ---
ax = u(1);  ay = u(2);  az = u(3);
p  = u(4);  q  = u(5);  r  = u(6);

%% =========================================================
%% PREDICT STEP
%% =========================================================

% Trig shortcuts
cphi = cos(phi);  sphi = sin(phi);
cth  = cos(th);   sth  = sin(th);
cps  = cos(psi);  sps  = sin(psi);
tth  = tan(th);

% Rotation matrix: body -> NED  (3x3)
R_bn = [cth*cps,  sphi*sth*cps - cphi*sps,  cphi*sth*cps + sphi*sps;
        cth*sps,  sphi*sth*sps + cphi*cps,  cphi*sth*sps - sphi*cps;
        -sth,     sphi*cth,                  cphi*cth               ];

% NED acceleration from body IMU + gravity
a_NED = R_bn * [ax; ay; az] + [0; 0; g];

% Euler angle rates from gyro
phi_dot = p + sphi*tth*q + cphi*tth*r;
th_dot  = cphi*q - sphi*r;
psi_dot = (sphi/cth)*q + (cphi/cth)*r;

% Process model: f(x_prev, u)
f = [vx;    vy;    vz;
     a_NED(1); a_NED(2); a_NED(3);
     phi_dot; th_dot; psi_dot];

% Predicted state (Euler integration)
x_pred = x_prev + Ts * f;

% Wrap psi to [-pi, pi]
x_pred(9) = atan2(sin(x_pred(9)), cos(x_pred(9)));

% State Jacobian F = df/dx  (evaluated at x_prev)
F = zeros(9,9);

% Position block
F(1,4) = 1;  F(2,5) = 1;  F(3,6) = 1;

% Velocity block: d(R_bn*a_b)/d(phi,theta,psi)
F(4,7) = ( cphi*sth*cps + sphi*sps)*ay + (-sphi*sth*cps + cphi*sps)*az;
F(4,8) = -sth*cps*ax + sphi*cth*cps*ay + cphi*cth*cps*az;
F(4,9) = -cth*sps*ax + (-sphi*sth*sps - cphi*cps)*ay + (-cphi*sth*sps + sphi*cps)*az;

F(5,7) = ( cphi*sth*sps - sphi*cps)*ay + (-sphi*sth*sps - cphi*cps)*az;
F(5,8) = -sth*sps*ax + sphi*cth*sps*ay + cphi*cth*sps*az;
F(5,9) =  cth*cps*ax + ( sphi*sth*cps - cphi*sps)*ay + ( cphi*sth*cps + sphi*sps)*az;

F(6,7) =  cphi*cth*ay - sphi*cth*az;
F(6,8) = -cth*ax - sphi*sth*ay - cphi*sth*az;

% Attitude block: Euler kinematics Jacobian
F(7,7) =  cphi*tth*q - sphi*tth*r;
F(7,8) =  sphi*(1/cth)^2*q + cphi*(1/cth)^2*r;
F(8,7) = -sphi*q - cphi*r;
F(9,7) =  cphi/cth*q - sphi/cth*r;
F(9,8) =  sphi*sth/cth^2*q + cphi*sth/cth^2*r;

% Discretised Jacobian
Fd = eye(9) + Ts * F;

% Predicted covariance
P_pred = Fd * P_prev * Fd' + Q;

%% =========================================================
%% UPDATE STEP
%% =========================================================

% Measurement model h(x_pred)
h = [-x_pred(3);    % baro altitude = -pz (NED)
      x_pred(9)];   % mag heading   =  psi

% Measurement Jacobian H  (2x9)
H      = zeros(2,9);
H(1,3) = -1;   % d(baro)/d(pz) = -1
H(2,9) =  1;   % d(mag)/d(psi) =  1

% Innovation
y = z_meas - h;
% Wrap psi innovation
y(2) = atan2(sin(y(2)), cos(y(2)));

% Innovation covariance
S = H * P_pred * H' + R;

% Kalman gain
K = P_pred * H' / S;

% Updated state
x_est = x_pred + K * y;
x_est(9) = atan2(sin(x_est(9)), cos(x_est(9)));   % wrap psi

% Joseph form covariance update (numerically stable)
IKH   = eye(9) - K * H;
P_est = IKH * P_pred * IKH' + K * R * K';

% Symmetrise P to prevent drift
P_est = 0.5 * (P_est + P_est');
```

---

## Step 4 — Add Unit Delay Blocks

The MATLAB Function block is a **feedthrough** block (output depends on current inputs). To avoid algebraic loops, the x_est and P_est must feed back through Unit Delay blocks.

**In the EKF_Sensor_Fusion subsystem:**

1. Add **Unit Delay** block for `x_prev`:
   - Sample time: Ts = 0.001
   - Initial condition: `x0` workspace variable (9×1 zeros) or set directly to `[0;0;0;0;0;0;0;0;0]`
   - Input: `x_est` output from EKF_Core
   - Output: `x_prev` input to EKF_Core

2. Add **Unit Delay** block for `P_prev`:
   - Sample time: Ts = 0.001
   - Initial condition: Set to `P0` — a 9×9 matrix. In the block dialog, enter: `diag([1,1,1,0.5,0.5,0.5,0.1,0.1,0.1])`
   - Input: `P_est` output from EKF_Core
   - Output: `P_prev` input to EKF_Core

**Diagram inside EKF_Sensor_Fusion:**
```
u ──────────────────────────────►  EKF_Core ──► x_est (subsystem output)
z_meas ─────────────────────────►             ─► P_est (subsystem output)
x_prev ◄──[Unit Delay, IC=x0]───────────────────┘ (x_est feedback)
P_prev ◄──[Unit Delay, IC=P0]───────────────────┘ (P_est feedback)
```

---

## Step 5 — Configure the Sample Time

The EKF runs at the same rate as the Simulink model (Ts = 0.001 s).

1. In the MATLAB Function block header, the Ts constant = 0.001. This must match.
2. Set the Unit Delay sample time to 0.001 in both blocks.
3. Check that your model solver is **Fixed-step, ode4 (Runge-Kutta), step size 0.001**.

---

## Step 6 — Open-Loop Test (Do This Before Task 3)

Before replacing any feedback, run the EKF **alongside** the true plant state. This lets you compare EKF vs. truth.

At the top level, connect:
- EKF inputs: from sensor subsystem (accel_meas, gyro_meas, baro_alt_meas, mag_psi_meas)
- Controller inputs: still from plant true states (do NOT change yet)
- Add `To Workspace` blocks on `x_est` from EKF

Run a 40-second simulation and in MATLAB:

```matlab
%% Plot EKF altitude estimate vs. true
alt_ekf  = -squeeze(out.x_est.Data(3,:,:));  % -pz (EKF estimate)
alt_true = -squeeze(out.z.Data);              % -z_plant (true)
t        =  out.x_est.Time;

figure;
subplot(2,2,1);
plot(t, alt_true, 'b-', 'DisplayName','True altitude'); hold on;
plot(t, alt_ekf,  'r--', 'DisplayName','EKF estimate');
xlabel('Time (s)'); ylabel('Altitude (m)');
title('Altitude: True vs EKF'); legend; grid on;

%% Roll estimate
phi_ekf  = squeeze(out.x_est.Data(7,:,:)) * 180/pi;
phi_true = squeeze(out.phi.Data) * 180/pi;

subplot(2,2,2);
plot(t, phi_true, 'b-', 'DisplayName','True roll'); hold on;
plot(t, phi_ekf,  'r--', 'DisplayName','EKF roll');
xlabel('Time (s)'); ylabel('Roll (deg)'); title('Roll: True vs EKF'); legend; grid on;

%% Pitch estimate
th_ekf  = squeeze(out.x_est.Data(8,:,:)) * 180/pi;
th_true = squeeze(out.theta.Data) * 180/pi;

subplot(2,2,3);
plot(t, th_true, 'b-', 'DisplayName','True pitch'); hold on;
plot(t, th_ekf,  'r--', 'DisplayName','EKF pitch');
xlabel('Time (s)'); ylabel('Pitch (deg)'); title('Pitch: True vs EKF'); legend; grid on;

%% Yaw estimate
psi_ekf  = squeeze(out.x_est.Data(9,:,:)) * 180/pi;
psi_true = squeeze(out.psi.Data) * 180/pi;

subplot(2,2,4);
plot(t, psi_true, 'b-', 'DisplayName','True yaw'); hold on;
plot(t, psi_ekf,  'r--', 'DisplayName','EKF yaw');
xlabel('Time (s)'); ylabel('Yaw (deg)'); title('Yaw: True vs EKF'); legend; grid on;
sgtitle('EKF Open-Loop: Estimate vs True State');
```

**What good EKF tracking looks like:**
- Altitude EKF should converge to true altitude within 2–3 seconds
- Attitude EKF should track roll/pitch within a few degrees
- Yaw may show more lag — this is expected (magnetometer is noisy)

**If the EKF diverges (estimates blow up):**
- Check that the Rotation matrix R_bn is computed at `x_prev` angles (not mixed from x_pred)
- Check Unit Delay initial conditions — P0 must be positive definite
- Increase Q (reduces sensitivity to model mismatch)
- Check that Ts = 0.001 matches the model step size

---

## Common Mistakes

| Mistake | Symptom | Fix |
|---|---|---|
| P_prev Unit Delay IC is scalar 0 | Kalman gain K is near-zero forever | Set IC to `diag([1,1,1,0.5,0.5,0.5,0.1,0.1,0.1])` |
| Ts inside MATLAB Function != model step | EKF estimates grow unbounded | Set Ts = 0.001 inside function |
| psi innovation not wrapped | Large yaw jump causes correction to flip sign | Use `atan2(sin(y(2)), cos(y(2)))` |
| P not symmetrised | P becomes non-positive-definite after ~10 s | Add `P_est = 0.5*(P_est + P_est')` |
| baro_alt_meas sign inverted | EKF z-estimate drifts in wrong direction | baro outputs positive-up altitude; EKF pz is positive-down. h_baro = -pz is correct |

---

## Checklist

- [ ] EKF_Sensor_Fusion subsystem created at top level
- [ ] MATLAB Function block: all 5 inputs/outputs declared with correct sizes
- [ ] Predict step: process model f, Jacobian F, P_pred coded
- [ ] Update step: h(x), H, innovation y (with psi wrapping), K, x_est, P_est coded
- [ ] Joseph form covariance update used
- [ ] Unit Delay for x_prev: IC = zeros(9,1)
- [ ] Unit Delay for P_prev: IC = diag([1,1,1,0.5,0.5,0.5,0.1,0.1,0.1])
- [ ] Both Unit Delays sample time = 0.001
- [ ] Open-loop test: EKF altitude converges within 3 s of simulation
- [ ] Open-loop test: EKF roll and pitch track truth within ±5°
