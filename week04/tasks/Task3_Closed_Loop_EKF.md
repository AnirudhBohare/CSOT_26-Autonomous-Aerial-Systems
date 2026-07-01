# Task 3 — Closed-Loop Integration & RMSE Evaluation
### Week 4 · Autonomous Aerial Systems · CSOT 2026

**Estimated time:** 1 hour

**Deliverable:** `ekf_comparison_wk4_YOUR_NAME.png` + `ekf_rmse_wk4_YOUR_NAME.png`

**Prerequisites:** Task 2 open-loop test passing (EKF tracks true state).

---

## What You Are Doing

In Week 3, your PID controller received perfect true-state feedback from the plant. Now you will cut that connection and replace it with EKF estimates. This is the complete closed-loop autopilot — no true-state cheating anywhere in the feedback path.

```
Before (Week 3):
  Plant ──[true phi,theta,psi,z]──► PID Controller

After (Week 4):
  Plant ──[noisy sensor outputs]──► EKF ──[x_est]──► PID Controller
```

The RMSE target loosens slightly (from 0.3 m in Week 3 to 0.5 m here) because the controller now works with estimated states — some additional error is expected and physically realistic.

---

## Step 1 — Replace True-State Feedback Wires

In your top-level Simulink model, locate the feedback wires from the plant to the controller and replace them:

| Week 3 connection (true states) | Week 4 replacement (EKF estimates) |
|---|---|
| `phi` from plant → inner Roll PID | `x_est(7)` from EKF |
| `theta` from plant → inner Pitch PID | `x_est(8)` from EKF |
| `psi` from plant → inner Yaw PID | `x_est(9)` from EKF |
| `z` from plant → altitude PID | `-x_est(3)` (negate: EKF pz is positive down, altitude is positive up) |

**How to extract individual states from x_est (9×1 vector):**
Use a **Selector** block (Signal Routing library):
- Set Index: 7 → phi_est
- Set Index: 8 → theta_est
- Set Index: 9 → psi_est
- Set Index: 3, multiply by −1 → alt_est

Or use a **MATLAB Function** block:
```matlab
function [phi_e, th_e, psi_e, alt_e] = unpack_state(x_est)
  phi_e = x_est(7);
  th_e  = x_est(8);
  psi_e = x_est(9);
  alt_e = -x_est(3);   % altitude (positive up) = -pz
end
```

**Do NOT change the plant or sensor wiring.** The plant still outputs true states — these go to the sensor subsystem only (to generate noisy sensor measurements). The EKF then estimates state from those noisy measurements.

---

## Step 2 — Add Logging Blocks

Add `To Workspace` blocks on the following signals (use `timeseries` save format):

| Signal | Workspace name | Purpose |
|---|---|---|
| `x_est` (9×1) | `x_est` | Full EKF state estimate |
| `z` (plant true z) | `z_true` | For RMSE comparison |
| `phi`, `theta`, `psi` | `phi_true`, `theta_true`, `psi_true` | True attitude for comparison |
| `mode` (from Stateflow) | `mode` | To identify hover window |

---

## Step 3 — Run the 40-Second Simulation

Run the full Stateflow-managed sequence (arm_cmd = 1):
- 0–6 s: TAKEOFF (altitude ramps to 1 m)
- 6–36 s: HOVER (30-second evaluation window)
- 36–40 s: LAND

If the drone diverges (faster than Week 3):
- Check that the altitude error sign is correct: `error_z = alt_setpoint − alt_est` where `alt_est = -x_est(3)`
- The EKF altitude estimate has a short convergence transient (~1–2 s). If the drone oscillates during the first 2 s of hover, this is expected.
- If divergence continues after 5 s of hover, check that pz (not px or py) is used for altitude

---

## Step 4 — Compute EKF RMSE

Run the following in the MATLAB Command Window after simulation:

```matlab
%% ─── Extract signals ────────────────────────────────────────────────────
t = out.x_est.Time;

% EKF state estimates
x_est_data = squeeze(out.x_est.Data);   % 9×N after squeeze
if size(x_est_data,1) ~= 9
    x_est_data = x_est_data';            % ensure 9×N
end

alt_ekf   = -x_est_data(3,:)';          % EKF altitude (positive up)
phi_ekf   = x_est_data(7,:)' * 180/pi;
theta_ekf = x_est_data(8,:)' * 180/pi;
psi_ekf   = x_est_data(9,:)' * 180/pi;

% True states
alt_true   = -squeeze(out.z_true.Data);
phi_true   = squeeze(out.phi_true.Data)   * 180/pi;
theta_true = squeeze(out.theta_true.Data) * 180/pi;
psi_true   = squeeze(out.psi_true.Data)   * 180/pi;

%% ─── Hover window (6–36 s) ──────────────────────────────────────────────
hover_mask = (t >= 6.0) & (t <= 36.0);

alt_ekf_h   = alt_ekf(hover_mask);
phi_ekf_h   = phi_ekf(hover_mask);
theta_ekf_h = theta_ekf(hover_mask);
alt_true_h  = alt_true(hover_mask);
phi_true_h  = phi_true(hover_mask);
theta_true_h= theta_true(hover_mask);

%% ─── RMSE calculations ──────────────────────────────────────────────────
% EKF vs setpoint (controller performance)
RMSE_alt_ctrl = sqrt(mean((alt_ekf_h - 1.0).^2));

% EKF vs true state (estimation accuracy)
RMSE_alt_est   = sqrt(mean((alt_ekf_h   - alt_true_h).^2));
RMSE_phi_est   = sqrt(mean((phi_ekf_h   - phi_true_h).^2));
RMSE_theta_est = sqrt(mean((theta_ekf_h - theta_true_h).^2));

fprintf('\n=============================================\n');
fprintf('EKF Performance Report (30 s hover window)\n');
fprintf('=============================================\n');
fprintf('Altitude RMSE vs setpoint  (R1):  %.4f m  [target < 0.5 m] %s\n', ...
        RMSE_alt_ctrl, pass_fail(RMSE_alt_ctrl, 0.5));
fprintf('Altitude estimation RMSE   (R5):  %.4f m\n',  RMSE_alt_est);
fprintf('Roll estimation RMSE       (R5):  %.4f deg\n', RMSE_phi_est);
fprintf('Pitch estimation RMSE      (R5):  %.4f deg\n', RMSE_theta_est);
fprintf('=============================================\n\n');

function s = pass_fail(val, thresh)
  if val < thresh; s = '  PASS'; else; s = '  FAIL'; end
end
```

**Take a screenshot of this output.** Save as `ekf_rmse_wk4_YOUR_NAME.png`.

---

## Step 5 — Generate Comparison Plots

```matlab
%% ─── 4-panel comparison figure ─────────────────────────────────────────
figure('Position', [100 100 1300 900]);
sgtitle('Week 4 — EKF Closed-Loop: Estimate vs True State (30 s hover)', ...
        'FontSize', 14, 'FontWeight', 'bold');

%% Panel 1: Altitude
subplot(2,2,1);
plot(t, alt_true, 'b-',  'LineWidth', 1.5, 'DisplayName', 'True altitude'); hold on;
plot(t, alt_ekf,  'r--', 'LineWidth', 1.2, 'DisplayName', 'EKF estimate');
yline(1.0, 'k:', 'LineWidth', 1.0, 'DisplayName', 'Setpoint (1 m)');
yline(1.5, 'g:', 'LineWidth', 0.8, 'DisplayName', '+0.5 m bound');
yline(0.5, 'g:', 'LineWidth', 0.8);
xlim([0 40]); ylim([-0.5 2.0]);
xlabel('Time (s)'); ylabel('Altitude (m)');
title(sprintf('Altitude — RMSE_{ctrl} = %.3f m (%s)', ...
              RMSE_alt_ctrl, pass_fail_str(RMSE_alt_ctrl, 0.5)));
legend('Location', 'southeast'); grid on;

%% Panel 2: Roll
subplot(2,2,2);
phi_true_all = squeeze(out.phi_true.Data) * 180/pi;
phi_ekf_all  = x_est_data(7,:)' * 180/pi;
plot(t, phi_true_all, 'b-',  'LineWidth', 1.5, 'DisplayName', 'True roll'); hold on;
plot(t, phi_ekf_all,  'r--', 'LineWidth', 1.2, 'DisplayName', 'EKF roll');
yline( 5, 'k:', 'LineWidth', 0.8); yline(-5, 'k:', 'LineWidth', 0.8, 'DisplayName', '±5° PID bound');
xlim([0 40]); ylim([-15 15]);
xlabel('Time (s)'); ylabel('Roll (deg)');
title(sprintf('Roll — EKF RMSE = %.2f deg', RMSE_phi_est));
legend; grid on;

%% Panel 3: Pitch
subplot(2,2,3);
theta_true_all = squeeze(out.theta_true.Data) * 180/pi;
theta_ekf_all  = x_est_data(8,:)' * 180/pi;
plot(t, theta_true_all, 'b-',  'LineWidth', 1.5, 'DisplayName', 'True pitch'); hold on;
plot(t, theta_ekf_all,  'r--', 'LineWidth', 1.2, 'DisplayName', 'EKF pitch');
yline( 5, 'k:', 'LineWidth', 0.8); yline(-5, 'k:', 'LineWidth', 0.8, 'DisplayName', '±5° PID bound');
xlim([0 40]); ylim([-15 15]);
xlabel('Time (s)'); ylabel('Pitch (deg)');
title(sprintf('Pitch — EKF RMSE = %.2f deg', RMSE_theta_est));
legend; grid on;

%% Panel 4: Altitude error with and without EKF (vs setpoint)
subplot(2,2,4);
err_true = alt_true - 1.0;
err_ekf  = alt_ekf  - 1.0;
plot(t, err_true, 'b-',  'LineWidth', 1.5, 'DisplayName', 'True altitude error'); hold on;
plot(t, err_ekf,  'r--', 'LineWidth', 1.2, 'DisplayName', 'EKF altitude error');
yline( 0.5, 'g:', 'LineWidth', 1.0, 'DisplayName', '±0.5 m EKF bound');
yline(-0.5, 'g:', 'LineWidth', 1.0);
yline( 0.3, 'k:', 'LineWidth', 0.8, 'DisplayName', '±0.3 m Week 3 bound');
yline(-0.3, 'k:', 'LineWidth', 0.8);
xlim([0 40]); ylim([-1.5 1.5]);
xlabel('Time (s)'); ylabel('Error (m)');
title('Altitude Error vs Setpoint — True vs EKF Controlled');
legend('Location','northeast'); grid on;

saveas(gcf, 'ekf_comparison_wk4_YOUR_NAME.png');
fprintf('Plot saved: ekf_comparison_wk4_YOUR_NAME.png\n');

function s = pass_fail_str(val, thresh)
  if val < thresh; s = 'PASS'; else; s = 'FAIL'; end
end
```

---

## Expected Results

| Metric | Target | Typical well-tuned |
|---|---|---|
| Altitude RMSE vs. setpoint | < 0.5 m | 0.1 – 0.25 m |
| EKF altitude estimation error | — | < 0.05 m (baro is good) |
| EKF roll/pitch RMSE vs. true | — | < 2° (IMU + Euler integration) |
| Drone behaviour | Stable hover | May oscillate slightly more than Week 3 |

The slight increase in altitude RMSE vs. Week 3 (0.3 m → 0.5 m target) is expected — the controller now acts on estimated state, introducing a small additional error floor from the EKF.

---

## Troubleshooting

| Problem | Likely cause | Fix |
|---|---|---|
| Drone crashes immediately | Alt error sign inverted in EKF feedback | `alt_est = -x_est(3)`, then error = `alt_est - z_setpoint` |
| Drone hovers but oscillates more than Week 3 | EKF altitude estimate is laggy | Reduce R_baro (trust barometer more) |
| EKF altitude diverges upward | Q too large or P0 too large | Reduce Q_vel, Q_pos by factor 10 |
| Yaw diverges | R_mag too small | Increase R_mag to 1.0 rad² |
| `squeeze` error on x_est | Dimensions: x_est is stored as 9×1×N | `squeeze(out.x_est.Data)` then transpose if needed |

---

## Checklist

- [ ] All true-state feedback wires replaced with EKF estimates
- [ ] Altitude feedback: `alt_est = -x_est(3)` (not `x_est(3)`)
- [ ] Logging: `x_est`, `z_true`, `phi_true`, `theta_true`, `psi_true` → workspace
- [ ] 40-second simulation runs without diverging
- [ ] Altitude RMSE vs. setpoint < 0.5 m (R1 with EKF)
- [ ] MATLAB command window output screenshotted as `ekf_rmse_wk4_YOUR_NAME.png`
- [ ] 4-panel comparison plot saved as `ekf_comparison_wk4_YOUR_NAME.png`
- [ ] All subplots: axis labels, title, legend, grid
