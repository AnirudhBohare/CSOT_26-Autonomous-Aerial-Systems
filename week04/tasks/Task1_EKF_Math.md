# Task 1 — EKF Mathematical Formulation
### Week 4 · Autonomous Aerial Systems · CSOT 2026

>**Estimated time:** 2 hours

>**Deliverable:** Full EKF math embedded in your `.slx` (via MATLAB Function block code). This task covers the derivation; Task 2 puts it into Simulink.

>**Watch first:** Video (Sensor Fusion)

---

## Why an EKF?

Your Week 2 sensors produce noisy, biased outputs. Your Week 3 controller uses clean true states from the plant — that is not realistic. The Extended Kalman Filter is the industry-standard solution:

- It **fuses** multiple noisy sensors into a single optimal estimate.
- It **tracks uncertainty** explicitly via a covariance matrix P.
- It handles **nonlinear dynamics** (rotation matrices, Euler kinematics) by linearising at each step.

The EKF operates at every timestep (Ts = 0.001 s) in two phases:

```
Predict:  x_pred = f(x_prev, u)        (use IMU as input — dead-reckoning)
          P_pred = F*P_prev*F' + Q

Update:   y      = z_meas - h(x_pred)  (innovation: measured vs. predicted)
          K      = P_pred*H'*(H*P_pred*H' + R)^-1   (Kalman gain)
          x_est  = x_pred + K*y
          P_est  = (I - K*H)*P_pred*(I - K*H)' + K*R*K'
```

---

## Step 1 — Define the State Vector

The EKF tracks 9 states in the NED frame (x positive North, y positive East, z positive Down):

```
x = [px; py; pz; vx; vy; vz; phi; theta; psi]
     pos. North  vel. N    roll    pitch   yaw
          East        E
          Down        D
```

**Dimensions:** 9×1 column vector
**Units:** metres, m/s, radians

---

## Step 2 — Process Model f(x, u)

The IMU provides the process input every timestep:
```
u = [ax_b; ay_b; az_b; p; q; r]   (6×1)
```
where `[ax_b, ay_b, az_b]` are accelerometer measurements (body frame, specific force in m/s²) and `[p, q, r]` are gyroscope measurements (body angular rates in rad/s).

### 2a — Position kinematics

```
x_dot(1) = vx   (px rate = North velocity)
x_dot(2) = vy   (py rate = East velocity)
x_dot(3) = vz   (pz rate = Down velocity)
```

### 2b — Velocity dynamics

The accelerometer measures **specific force** (non-gravitational acceleration in body frame). To get NED acceleration:

```
[vx_dot]   [R_bn] * [ax_b]   [0]
[vy_dot] =          [ay_b] + [0]
[vz_dot]            [az_b]   [g]
```

where g = 9.81 m/s² and R_bn is the 3×3 rotation matrix from body to NED:

```matlab
R_bn = [cth*cps, sphi*sth*cps - cphi*sps, cphi*sth*cps + sphi*sps;
        cth*sps, sphi*sth*sps + cphi*cps, cphi*sth*sps - sphi*cps;
        -sth,    sphi*cth,                cphi*cth               ];
```

where: `cphi = cos(phi)`, `sphi = sin(phi)`, `cth = cos(theta)`, `sth = sin(theta)`, `cps = cos(psi)`, `sps = sin(psi)`.

**Physical meaning:** Gravity is +g in the NED z direction (positive down). The accelerometer subtracts gravity, so we must add it back when computing velocity derivatives.

### 2c — Attitude kinematics (Euler rate equations)

```
x_dot(7) = phi_dot   = p + sin(phi)*tan(theta)*q + cos(phi)*tan(theta)*r
x_dot(8) = theta_dot = cos(phi)*q - sin(phi)*r
x_dot(9) = psi_dot   = (sin(phi)/cos(theta))*q + (cos(phi)/cos(theta))*r
```

### 2d — Discretisation (Euler integration)

At each step, integrate with sample time Ts = 0.001 s:
```
x_pred = x_prev + Ts * f(x_prev, u)
```

After prediction, wrap psi to [−π, π]:
```matlab
x_pred(9) = atan2(sin(x_pred(9)), cos(x_pred(9)));
```

---

## Step 3 — State Transition Jacobian F (9×9)

The Jacobian F = ∂f/∂x is evaluated at the current state estimate. It is used to propagate the covariance.

**Structure:** Most of F is zero. Non-zero blocks:

### Position rows (rows 1–3):
```
F(1,4) = 1   (∂px_dot/∂vx)
F(2,5) = 1   (∂py_dot/∂vy)
F(3,6) = 1   (∂pz_dot/∂vz)
```

### Velocity rows (rows 4–6) — angular dependence:

These come from differentiating R_bn * [ax;ay;az] with respect to phi, theta, psi:

```matlab
% Row 4: ∂vx_dot/∂(phi, theta, psi)
F(4,7) = (cphi*sth*cps + sphi*sps)*ay + (-sphi*sth*cps + cphi*sps)*az;
F(4,8) = -sth*cps*ax + sphi*cth*cps*ay + cphi*cth*cps*az;
F(4,9) = -cth*sps*ax + (-sphi*sth*sps - cphi*cps)*ay + (-cphi*sth*sps + sphi*cps)*az;

% Row 5: ∂vy_dot/∂(phi, theta, psi)
F(5,7) = (cphi*sth*sps - sphi*cps)*ay + (-sphi*sth*sps - cphi*cps)*az;
F(5,8) = -sth*sps*ax + sphi*cth*sps*ay + cphi*cth*sps*az;
F(5,9) = cth*cps*ax + (sphi*sth*cps - cphi*sps)*ay + (cphi*sth*cps + sphi*sps)*az;

% Row 6: ∂vz_dot/∂(phi, theta, psi)
F(6,7) = cphi*cth*ay - sphi*cth*az;
F(6,8) = -cth*ax - sphi*sth*ay - cphi*sth*az;
F(6,9) = 0;   % vz_dot has no psi dependence
```

### Attitude rows (rows 7–9) — self-coupling:

```matlab
% Row 7: ∂phi_dot/∂(phi, theta)
F(7,7) = cphi*tan(th)*q - sphi*tan(th)*r;
F(7,8) = sphi*(1/cos(th))^2*q + cphi*(1/cos(th))^2*r;

% Row 8: ∂theta_dot/∂phi
F(8,7) = -sphi*q - cphi*r;

% Row 9: ∂psi_dot/∂(phi, theta)
F(9,7) = cphi/cos(th)*q - sphi/cos(th)*r;
F(9,8) = sphi*sin(th)/cos(th)^2*q + cphi*sin(th)/cos(th)^2*r;
```

**Discretised Jacobian (for covariance propagation):**
```matlab
F_d = eye(9) + Ts * F;
```

---

## Step 4 — Measurement Model h(x)

You have two sensor measurements available for the EKF update:

### Measurement 1: Barometer altitude
```
z_baro  = baro_alt_meas    (positive up, metres)
h_1(x)  = -pz              (altitude = negative of NED z)
```

### Measurement 2: Magnetometer heading
```
z_mag   = mag_heading_meas  (yaw angle, radians, in NED)
h_2(x)  = psi               (state index 9)
```

Combined measurement vector:
```
z_meas = [z_baro; z_mag]    (2×1)
h(x)   = [-pz;    psi   ]   (2×1)
```

---

## Step 5 — Measurement Jacobian H (2×9)

```
H = zeros(2,9);
H(1,3) = -1;   % d(h_baro)/d(pz) = -1 (altitude = -pz)
H(2,9) =  1;   % d(h_mag)/d(psi) =  1
```

---

## Step 6 — Noise Covariance Matrices

### Process noise Q (9×9 diagonal)

Q reflects how much we trust our process model. Larger Q = trust sensors more.

```matlab
q_pos = 0.01;    % position process noise (m^2/s^2 per step) — low, kinematics are well-known
q_vel = 0.10;    % velocity process noise — higher, accelerometer has noise
q_att = 0.01;    % attitude process noise — gyro is relatively good

Q = diag([q_pos, q_pos, q_pos, ...
          q_vel, q_vel, q_vel, ...
          q_att, q_att, q_att]);
```

### Measurement noise R (2×2 diagonal)

R reflects how noisy each sensor is. Match to your Week 2 Band-Limited White Noise settings.

```matlab
r_baro = 0.50;   % barometer noise variance (m^2) — baro is moderately noisy
r_mag  = 0.10;   % magnetometer heading noise variance (rad^2) — mag is noisy

R = diag([r_baro, r_mag]);
```

**Tuning principle:** If the EKF lags behind true state, decrease R (trust measurements more). If EKF is jittery, increase R (trust model more). Do not change Q and R simultaneously.

---

## Step 7 — Initial Conditions

Set these as MATLAB workspace variables (or as Unit Delay initial conditions):

```matlab
x0 = zeros(9,1);            % drone starts at origin, zero velocity, level attitude
P0 = diag([1.0, 1.0, 1.0, ...   % 1 m initial position uncertainty
           0.5, 0.5, 0.5, ...   % 0.5 m/s initial velocity uncertainty
           0.1, 0.1, 0.1]);     % ~6° initial attitude uncertainty
```

---

## Step 8 — Verify Dimensions (Checklist)

Before writing the Simulink block, confirm:

| Variable | Dimension | Notes |
|---|---|---|
| x | 9×1 | State vector |
| P | 9×9 | Covariance matrix (symmetric, positive definite) |
| u | 6×1 | IMU input [ax;ay;az;p;q;r] |
| z_meas | 2×1 | [baro_alt; mag_psi] |
| f(x,u) | 9×1 | Process derivatives |
| F | 9×9 | State Jacobian |
| F_d | 9×9 | Discretised Jacobian |
| H | 2×9 | Measurement Jacobian |
| Q | 9×9 | Process noise (diagonal) |
| R | 2×2 | Measurement noise (diagonal) |
| K | 9×2 | Kalman gain |
| y | 2×1 | Innovation vector |

---

## Checklist

- [ ] State vector defined: 9 elements, correct physical meaning
- [ ] Process model f(x,u) written for all 9 rows
- [ ] Rotation matrix R_bn coded correctly
- [ ] F Jacobian: correct non-zero entries for velocity and attitude rows
- [ ] Measurement model h(x) defined: baro + magnetometer
- [ ] H Jacobian: 2×9 with correct non-zero entries
- [ ] Q matrix set with physically justified values
- [ ] R matrix set (match Week 2 noise magnitudes)
- [ ] x0 and P0 initialised
- [ ] Dimension table verified
