# Task 2(a) — Wind Disturbance Model
### Week 2 · Autonomous Aerial Systems · CSOT 2026

Estimated time: 45 min

---

## What You Are Modelling

Wind on a quadcopter acts as an external force on the body frame. A steady 3 m/s crosswind from the north exerts a drag force on the airframe proportional to the wind speed and the vehicle's drag coefficient.

For this project we use a simplified but physically reasonable model:
- Steady component: constant force vector in inertial frame (3 m/s → F_wind_x)
- Turbulence component: Band-Limited White Noise added on top

The drag force magnitude is estimated as:
  F_drag = 0.5 * rho * Cd * A * v_wind^2

Where:
  rho = 1.225 kg/m^3 (air density at sea level)
  Cd  = 1.0 (bluff body drag coefficient for quadcopter frame)
  A   = 0.04 m^2 (estimated frontal area)
  v_wind = 3 m/s

Substituting: F_drag = 0.5 * 1.225 * 1.0 * 0.04 * 9 = 0.22 N

---

## Build the Wind Subsystem

Create a subsystem named "Wind Model" with no inputs and output F_wind [3x1].

Inside:

Block 1 — Steady crosswind force:
  Constant block → value: [0.22; 0; 0] (force in body X direction, NED)

Block 2 — Turbulence:
  Band-Limited White Noise block
    Noise power: [1e-6; 1e-6; 0]
    Sample time: 0.001

Block 3 — Sum both:
  Add Block: Constant + Noise → output F_wind [3x1]

Connect F_wind to the wind input port in your Translational Dynamics subsystem.

---

## Test

Run a 10 second hover simulation with the wind model connected.

Expected result: the quadcopter drifts in the X direction because there is no controller yet to reject the disturbance.

Scope on x position should show a parabolic drift (constant acceleration from wind force).

If x stays flat → wind is not connected correctly. Check signal routing into Translational Dynamics.

---

## Checklist

- [ ] Wind Model subsystem created
- [ ] Constant force [0.22; 0; 0] N applied
- [ ] Band-Limited White Noise turbulence added
- [ ] F_wind connected to Translational Dynamics
- [ ] 10 s hover shows X-position drift in scope

---

# Task 2(b) — IMU Sensor Model
### Week 2 · Autonomous Aerial Systems · CSOT 2026

**Estimated time:** 1.5 hours
**Deliverable:** `imu_plots_YOUR_NAME.png` — comparison plots for accelerometer and gyroscope with SNR values

---

## What an IMU Measures

An Inertial Measurement Unit (IMU) contains two sensors:

**Accelerometer** — measures specific force (not pure acceleration):
```
a_measured = a_true - g_body + bias_accel + noise_accel
```
Where `g_body` is the gravity vector rotated into body frame. In hover, the accelerometer reads `+g` upward even when the drone is stationary — it measures the reaction force against gravity.

**Gyroscope** — measures angular rate (body rates p, q, r):
```
ω_measured = ω_true + bias_gyro + noise_gyro
```

---

## Noise Model Parameters

Use these realistic MEMS IMU parameters (based on MPU-6050 class sensor):

| Parameter | Accelerometer | Gyroscope |
|---|---|---|
| Noise density | 400 µg/√Hz | 0.005 °/s/√Hz |
| Constant bias | 0.05 m/s² | 0.01 °/s |
| Sample rate | 1000 Hz (0.001 s) | 1000 Hz |
| Bias instability | 60 µg | 3.6 °/hr |

**Converting noise density to simulation:**

For Band-Limited White Noise block in Simulink:
```matlab
% Accelerometer noise power
Fs = 1000;               % Hz
noise_density = 400e-6 * 9.81;  % m/s²/√Hz (converting g/√Hz)
noise_power_accel = noise_density^2 * Fs;   % ≈ 1.54e-3 m²/s⁴

% Gyroscope noise power
noise_density_gyro = 0.005 * pi/180;  % rad/s/√Hz (converting deg/s/√Hz)
noise_power_gyro = noise_density_gyro^2 * Fs;  % ≈ 3.8e-8 rad²/s²
```

---

## Building the IMU Subsystem

### Subsystem Inputs
From your Week 1 plant:
- `a_body` — true body-frame acceleration [3×1] (from Translational Dynamics)
- `omega_body` — true body rates [p, q, r] (from Rotational Dynamics)
- `phi, theta, psi` — attitude angles (from Kinematics)

### Subsystem Outputs
- `accel_meas` — measured accelerometer output [3×1] in m/s²
- `gyro_meas` — measured gyroscope output [3×1] in rad/s

---

## Step-by-Step Build

### Accelerometer Model

1. **Gravity in body frame:** Use a MATLAB Function block:
   ```matlab
   function g_body = gravity_body(phi, theta, psi)
       g = 9.81;
       g_body = [g*sin(theta);
                -g*sin(phi)*cos(theta);
                -g*cos(phi)*cos(theta)];
   end
   ```
2. **Specific force:** `f_specific = a_body - g_body`
3. **Constant bias:** Add a Constant block `[0.05; 0.05; 0.05]` (m/s²)
4. **White noise:** Three **Band-Limited White Noise** blocks (one per axis):
   - Noise power: `1.54e-3`
   - Sample time: `0.001`
   - Different seed for each axis (e.g., 23341, 23342, 23343)
5. **Sum:** `accel_meas = f_specific + bias + noise`

### Gyroscope Model

1. **True rates:** Take `[p; q; r]` directly from your plant
2. **Constant bias:** Add Constant `[0.01; 0.01; 0.01] × (pi/180)` rad/s
3. **White noise:** Three Band-Limited White Noise blocks:
   - Noise power: `3.8e-8`
   - Sample time: `0.001`
   - Seeds: 34451, 34452, 34453
4. **Sum:** `gyro_meas = omega_true + bias + noise`

### Wrap in Parent Subsystem

Group both models inside an `IMU` subsystem. This IMU subsystem goes inside your `Sensor Suite` parent subsystem.

---

## Generating the Comparison Plots

Run a 10-second hover simulation. Log:
- True accelerometer z-axis: `a_body(3)` (from plant)
- Measured accelerometer z-axis: `accel_meas(3)` (from IMU output)
- True gyro x-axis: `p` (from plant)
- Measured gyro x-axis: `gyro_meas(1)` (from IMU output)

```matlab
% Plot accelerometer comparison
figure('Position', [100 100 1000 400]);

subplot(1,2,1);
plot(tout, a_body_z_log, 'b-', 'LineWidth', 1.5, 'DisplayName', 'True');
hold on;
plot(tout, accel_meas_z_log, 'r-', 'LineWidth', 0.8, 'DisplayName', 'Measured');
xlabel('Time (s)'); ylabel('Acceleration (m/s²)');
title('Accelerometer Z-axis'); legend; grid on;

subplot(1,2,2);
plot(tout, gyro_p_log, 'b-', 'LineWidth', 1.5, 'DisplayName', 'True');
hold on;
plot(tout, gyro_meas_p_log, 'r-', 'LineWidth', 0.8, 'DisplayName', 'Measured');
xlabel('Time (s)'); ylabel('Angular Rate (rad/s)');
title('Gyroscope X-axis (roll rate)'); legend; grid on;

sgtitle('IMU — True vs Measured Signals');
saveas(gcf, 'imu_plots_YOUR_NAME.png');
```

---

## Computing SNR

```matlab
% SNR for accelerometer z-axis
signal_power_accel = mean(a_body_z_log.^2);
noise_power_accel_actual = mean((accel_meas_z_log - a_body_z_log - 0.05).^2);
SNR_accel_dB = 10 * log10(signal_power_accel / noise_power_accel_actual);
fprintf('Accelerometer SNR (z-axis): %.1f dB\n', SNR_accel_dB);

% SNR for gyroscope x-axis
signal_power_gyro = mean(gyro_p_log.^2) + 1e-12; % small epsilon to avoid log(0)
noise_power_gyro_actual = mean((gyro_meas_p_log - gyro_p_log - 0.01*pi/180).^2);
SNR_gyro_dB = 10 * log10(signal_power_gyro / noise_power_gyro_actual);
fprintf('Gyroscope SNR (x-axis): %.1f dB\n', SNR_gyro_dB);
```

**Expected ranges:**
- Accelerometer SNR: 35–55 dB
- Gyroscope SNR: 40–65 dB

If your values are outside these ranges, check your noise power calculation — units are the most common error (g vs m/s², deg vs rad).

---

## Checklist

- [ ] IMU subsystem inside `Sensor Suite` parent subsystem
- [ ] Accelerometer model: specific force + bias + noise
- [ ] Gravity rotation into body frame implemented correctly
- [ ] Gyroscope model: true rates + bias + noise
- [ ] Band-Limited White Noise blocks with correct noise power values
- [ ] 10-second hover simulation run
- [ ] True vs measured plots saved for accelerometer (z) and gyroscope (x)
- [ ] SNR computed and values noted (must be in 35–65 dB range)
- [ ] Plots saved as `imu_plots_YOUR_NAME.png`

