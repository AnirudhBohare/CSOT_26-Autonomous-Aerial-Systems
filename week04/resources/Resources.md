# Week 4 — Resources & References
### Autonomous Aerial Systems · CSOT 2026 · AeroClub, IIT Delhi

---

## Primary Video

| # | Title | Link |
|---|---|---|
| Video 1 | Sensor Fusion | https://youtu.be/d_9MnFtN6kA?si=7UMDHqj8mRkjCdnw |

Watch Video before starting Task 1.

---

## MathWorks Official Documentation

| Resource | What it covers | Link |
|---|---|---|
| MATLAB Function block | How to declare inputs/outputs, set sample time, handle persistent state | https://www.mathworks.com/help/simulink/slref/matlabfunction.html |
| Unit Delay block | Sample time, initial conditions, use in feedback loops | https://www.mathworks.com/help/simulink/slref/unitdelay.html |
| Selector block | Extract individual elements or rows from a vector signal | https://www.mathworks.com/help/simulink/slref/selector.html |
| Kalman Filter block | MathWorks built-in EKF (useful for reference / cross-checking your implementation) | https://www.mathworks.com/help/control/ref/extendedkalmanfilter.html |
| IMU Sensor Model | MathWorks documentation on modelling accelerometers + gyros | https://www.mathworks.com/help/nav/ref/imusensor.html |

---

## Reference Papers

### 1 — EKF for Quadcopter State Estimation (top-rated)
**Title:** Localization of a Quadrotor UAV Using Sensor Fusion of IMU, Sonar, and Magnetometer  
**Authors:** Madyastha et al., MDPI Remote Sensing, 2019  
**Why read it:** Directly comparable implementation — 9-state EKF fusing IMU + barometer + magnetometer, with complete Jacobian derivations. This is the benchmark implementation for this course.  
**Link:** https://doi.org/10.3390/rs12030505  

### 2 — EKF Theory (concise)
**Title:** An Introduction to the Kalman Filter  
**Authors:** Welch & Bishop, UNC Technical Report TR95-041  
**Why read it:** Most concise explanation of the predict-update cycle and the role of Q, R, P. 8 pages. Read Section 1 and 2 only.  
**Link:** https://www.cs.unc.edu/~welch/media/pdf/kalman_intro.pdf  

### 3 — Euler Angle Kinematics Reference
**Title:** Small Unmanned Aircraft: Theory and Practice — Chapter 2  
**Authors:** Beard & McLain  
**Why read it:** Complete derivation of the Euler angle rate equations (phi_dot, theta_dot, psi_dot from body angular rates). Equations 2.13–2.16 are exactly what you need for the EKF attitude rows.  
**Note:** Available through IIT Delhi library access.

---

## EKF Quick Reference Card

```
EKF Algorithm Summary (9-state, 2-measurement)
───────────────────────────────────────────────
GIVEN:
  x_prev (9×1)   previous state estimate
  P_prev (9×9)   previous covariance
  u      (6×1)   IMU measurement [ax;ay;az;p;q;r]
  z_meas (2×1)   [baro_alt; mag_psi]
  Q      (9×9)   process noise covariance (tuning parameter)
  R      (2×2)   measurement noise covariance (tuning parameter)

PREDICT:
  x_pred = x_prev + Ts * f(x_prev, u)        % Euler integrate
  F_d    = I + Ts * ∂f/∂x|_{x_prev}          % discrete Jacobian
  P_pred = F_d * P_prev * F_d' + Q           % propagate covariance

UPDATE:
  h      = [-x_pred(3); x_pred(9)]           % measurement model
  y      = z_meas - h                         % innovation
  y(2)   = atan2(sin(y(2)), cos(y(2)))        % wrap psi innovation
  H      = [0,0,-1,0,0,0,0,0,0;              % measurement Jacobian
            0,0, 0,0,0,0,0,0,1]
  S      = H*P_pred*H' + R                    % innovation covariance
  K      = P_pred*H'/S                        % Kalman gain
  x_est  = x_pred + K*y                       % corrected state
  x_est(9) = atan2(sin(x_est(9)),cos(x_est(9)))  % wrap psi
  IKH    = I - K*H
  P_est  = IKH*P_pred*IKH' + K*R*K'          % Joseph form (stable)
  P_est  = 0.5*(P_est + P_est')              % symmetrise
```

---

## Physical Parameters Quick Reference

```
m   = 0.5 kg
g   = 9.81 m/s²
L   = 0.225 m
kT  = 2.9e-5 N/(rad/s)²
kQ  = 1.1e-6 N·m/(rad/s)²
Ixx = Iyy = 0.0049 kg·m²
Izz = 0.0088 kg·m²
Ts  = 0.001 s (model step size)

Frame: NED (x North, y East, z Down)
Hover setpoint: z_NED = -1.0 m  (altitude = +1.0 m)
Hover thrust: Fz = m*g = 4.905 N
```

---

## Accelerometer Measurement Model (important for EKF!)

The accelerometer measures **specific force**, not kinematic acceleration:

```
accel_meas = a_kinematic_body - g_body + bias + noise
           = R^T * a_NED - R^T * [0;0;g] + bias + noise
```

At hover (a_NED = 0, level attitude):
```
accel_meas_z ≈ -g ≈ -9.81 m/s²   (pointing down in NED, sensor reads negative)
```

To get NED kinematic acceleration from the measurement:
```
a_NED = R_bn * accel_meas + [0;0;g]
```
This is exactly how the velocity derivative is computed in the EKF process model.

---

## Noise Parameter Guidance

The Band-Limited White Noise blocks from Week 2 set `noise_power = variance * Ts`. Your R matrix should match the **variance** (not noise_power):

| Sensor | Suggested variance | R entry |
|---|---|---|
| Barometer altitude | 0.5 m² | R(1,1) = 0.5 |
| Magnetometer heading | 0.10 rad² (~5.7°) | R(2,2) = 0.1 |

If you used different noise powers in Week 2, set:
```
variance = noise_power / Ts = noise_power / 0.001
```

---

## Getting Help

GitHub Issues: tag `week4-question`
AeroClub contact: CSOT 2026 communication channel

