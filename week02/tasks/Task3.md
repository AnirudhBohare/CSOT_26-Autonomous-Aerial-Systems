# Task 3(a) — IMU Model (Accelerometer + Gyroscope)

Estimated time: 1 hour

---

## IMU Error Model Background

A real MEMS IMU corrupts its true measurement with several error sources. You will model three of them:

1. Constant Bias
   A fixed offset that does not change during a flight.
   Caused by hardware manufacturing imperfections.
   Calibrated on the ground but never perfectly removed.
   Model as: Gain block or Constant block added to the true signal.

2. Noise Density (White Noise)
   Random noise proportional to sqrt(bandwidth).
   Described in datasheets as noise density in units/sqrtHz.
   To convert to simulation noise power:
     sigma = NoiseDensity * sqrt(SampleRate)
   Model as: Band-Limited White Noise block.

3. Bias Instability (Gyroscope only)
   Slow random drift in the bias over time — pink/flicker noise.
   Important for gyroscopes, negligible for accelerometers on short flights.
   Model as: integrated white noise (white noise → Integrator → added to bias).

---

## Parameters

| Parameter | Accelerometer | Gyroscope |
|---|---|---|
| Constant bias | [0.05; 0.03; 0.04] m/s^2 | [0.01; 0.008; 0.012] rad/s |
| Noise density | 0.003 m/s^2/sqrtHz | 0.0003 rad/s/sqrtHz |
| Sample rate | 200 Hz | 200 Hz |
| Sigma (compute) | 0.003*sqrt(200) = 0.0424 m/s^2 | 0.0003*sqrt(200) = 0.00424 rad/s |
| Bias instability | N/A | 0.0005 rad/s |

---

## Build — Accelerometer Subsystem

Inputs:
  - a_true [3x1]: true specific force from Translational Dynamics
    (true acceleration in body frame + gravity transformed to body frame)
  - phi, theta: Euler angles (needed to rotate gravity into body frame)

Gravity in NED inertial frame: g_NED = [0; 0; 9.81]

Rotate gravity into body frame using rotation matrix R(phi, theta, psi):
  g_body = R^T * g_NED

True specific force (what the accelerometer actually measures):
  f_true = a_inertial_bodyframe - g_body

Add noise and bias:
  accel_meas = f_true + bias_accel + noise_accel

Build inside a subsystem named "Accelerometer":
  Step 1: Use MATLAB Function block to compute g_body from angles
  Step 2: Subtract g_body from inertial acceleration (both in body frame)
  Step 3: Add Constant block [0.05; 0.03; 0.04] for bias
  Step 4: Add Band-Limited White Noise:
            Noise power: [0.0018; 0.0018; 0.0018]  (= sigma^2 = 0.0424^2)
            Sample time: 0.005  (= 1/200 Hz)
  Step 5: Sum all three → accel_meas [3x1] output

---

## Build — Gyroscope Subsystem

Inputs:
  - omega_true [3x1]: true angular rates [p; q; r] from Rotational Dynamics

Build inside a subsystem named "Gyroscope":
  Step 1: Add Constant block [0.01; 0.008; 0.012] for constant bias
  Step 2: Add Band-Limited White Noise:
            Noise power: [0.000018; 0.000018; 0.000018]  (= 0.00424^2)
            Sample time: 0.005
  Step 3: Bias instability model:
            Band-Limited White Noise (power: 0.0000001, sample time: 0.005)
            → Integrator (IC: 0, limit output: YES, limits: [-0.005, 0.005])
            → adds slow drift to constant bias
  Step 4: Sum: omega_true + constant_bias + white_noise + bias_instability_drift
  Step 5: Output: gyro_meas [3x1]

---

## Wrap in IMU Subsystem

Create a top-level subsystem named "IMU" containing both Accelerometer and Gyroscope subsystems.

IMU inputs: a_true, omega_true, phi, theta
IMU outputs: accel_meas [3x1], gyro_meas [3x1]

---

## Common Mistakes

Wrong noise power units:
  Band-Limited White Noise takes POWER (sigma^2), not sigma.
  sigma = NoiseDensity * sqrt(SampleRate)
  power = sigma^2

Forgetting gravity:
  An accelerometer does NOT measure gravity-free acceleration.
  At rest on a table, it reads [0; 0; -9.81] m/s^2 (or [0;0;+9.81] in NED).
  Include the gravity rotation or your EKF will be badly off in Week 4.

Sample time mismatch:
  The Band-Limited White Noise sample time must match 1/SampleRate.
  For 200 Hz: sample time = 0.005 s

---

## Checklist

- [ ] Accelerometer subsystem built with correct inputs/outputs
- [ ] Gravity correctly rotated into body frame and included
- [ ] Constant bias added to accelerometer
- [ ] White noise added with correct power (0.0018 per axis)
- [ ] Gyroscope subsystem built
- [ ] Constant bias added to gyroscope
- [ ] White noise added with correct power
- [ ] Bias instability drift model included (Noise → Integrator)
- [ ] Both wrapped in "IMU" subsystem
- [ ] Model runs without errors at 200 Hz

---

# Task 3(b) — Barometer & Magnetometer Models

**Deliverable:** `baro_mag_plots_YOUR_NAME.png` + screenshot of both subsystems in Simulink

---

## Barometer

### What It Measures
A barometer measures atmospheric pressure and converts it to altitude using the barometric formula. In simulation, we skip the pressure step and model the altitude measurement directly:

```
alt_measured = alt_true + bias_baro + noise_baro
```

### Noise Parameters (based on MS5611 class barometer)

| Parameter | Value |
|---|---|
| Altitude noise std deviation | 0.1 m |
| Constant bias | 0.05 m |
| Update rate | 50 Hz (0.02 s) |

### Building the Barometer Subsystem

**Inputs:** `z` — true altitude from your plant (take negative z for altitude in NED: `alt = -z`)

**Outputs:** `alt_measured` — noisy altitude measurement in metres

Steps:
1. Negate the plant z-output: `alt_true = -z`  (NED: z is positive downward, altitude is positive upward)
2. Add Constant bias block: `0.05`
3. Add **Band-Limited White Noise** block:
   - Noise power: `0.01` (σ = 0.1 m → variance = 0.01 m²)
   - Sample time: `0.02`
   - Seed: `12345`
4. Sum: `alt_measured = alt_true + 0.05 + noise`

---

## Magnetometer

### What It Measures
A magnetometer measures the Earth's magnetic field vector to estimate heading (yaw angle). In simulation:

```
heading_measured = heading_true + declination_offset + noise_mag
```

**Magnetic declination** is the angle between magnetic north and true north — it varies by location. For IIT Delhi: approximately **0.7° east** (use 0.012 rad).

### Noise Parameters (based on HMC5883L class magnetometer)

| Parameter | Value |
|---|---|
| Heading noise std deviation | 0.5° (0.0087 rad) |
| Declination offset | 0.012 rad |
| Update rate | 100 Hz (0.01 s) |

### Building the Magnetometer Subsystem

**Inputs:** `psi` — true yaw angle from your plant Kinematics subsystem (rad)

**Outputs:** `heading_measured` — noisy heading measurement in rad

Steps:
1. Take `psi` directly from the Kinematics output
2. Add Constant bias block: `0.012` (declination offset)
3. Add **Band-Limited White Noise** block:
   - Noise power: `7.57e-5` (σ = 0.0087 rad → variance ≈ 7.57e-5 rad²)
   - Sample time: `0.01`
   - Seed: `98765`
4. Sum: `heading_measured = psi + 0.012 + noise`

---

## Subsystem Organisation

Both sensors go inside your `Sensor Suite` subsystem alongside the IMU:

```
Sensor Suite (parent subsystem)
├── IMU
│   ├── Accelerometer
│   └── Gyroscope
├── Barometer
└── Magnetometer
```

---

## Generating the Comparison Plots

Run a 30-second simulation where the drone is commanded to slowly rotate (yaw) — this makes the magnetometer comparison more interesting. Or run hover and accept that the heading comparison will be mostly flat.

```matlab
figure('Position', [100 100 1000 400]);

% Barometer comparison
subplot(1,2,1);
plot(tout, alt_true_log, 'b-', 'LineWidth', 1.5, 'DisplayName', 'True altitude');
hold on;
plot(tout, alt_meas_log, 'r-', 'LineWidth', 0.8, 'DisplayName', 'Barometer');
xlabel('Time (s)'); ylabel('Altitude (m)');
title('Barometer — True vs Measured'); legend; grid on;

% Magnetometer comparison
subplot(1,2,2);
plot(tout, psi_true_log * 180/pi, 'b-', 'LineWidth', 1.5, 'DisplayName', 'True heading');
hold on;
plot(tout, heading_meas_log * 180/pi, 'r-', 'LineWidth', 0.8, 'DisplayName', 'Magnetometer');
xlabel('Time (s)'); ylabel('Heading (degrees)');
title('Magnetometer — True vs Measured'); legend; grid on;

sgtitle('Barometer & Magnetometer — True vs Measured');
saveas(gcf, 'baro_mag_plots_YOUR_NAME.png');
```

**Expected:**
- Barometer: measured altitude slightly offset from true (+0.05 m bias) with ~0.1 m noise envelope
- Magnetometer: measured heading offset from true by ~0.7° (declination) with ±0.5° noise

---

## Checklist

- [ ] Barometer subsystem built with correct altitude sign (NED convention)
- [ ] Barometer noise power = 0.01, sample time = 0.02 s
- [ ] Magnetometer subsystem built with declination offset
- [ ] Magnetometer noise power = 7.57e-5, sample time = 0.01 s
- [ ] Both subsystems inside `Sensor Suite` parent subsystem
- [ ] Model screenshot taken showing Sensor Suite subsystem structure
- [ ] Comparison plots generated for altitude and heading
- [ ] Plots saved as `baro_mag_plots_YOUR_NAME.png`
