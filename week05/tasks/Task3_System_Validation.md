# Task 3 — System Validation & RMSE Report
### Week 5 · Autonomous Aerial Systems · CSOT 2026

>**Estimated time:** 1 hour

>**Deliverable:** `trajectory_plots_wk5_YOUR_NAME.png` + `rmse_validation_wk5_YOUR_NAME.png`

---

## Evaluation Window

All RMSE values are computed over the **hover window: t = 5 s to 35 s** (30 seconds). The evaluation window matches Week 4 for direct comparison. Takeoff transients and landing are excluded.

| Metric | Target |
|---|---|
| x position RMSE | < 0.5 m (R6) |
| y position RMSE | < 0.5 m (R6) |
| z altitude RMSE | < 0.15 m (carry-forward) |
| Roll φ RMSE | < 3° |
| Pitch θ RMSE | < 3° |
| Yaw ψ RMSE | < 5° |

---

## Step 1 — Simulation Setup

Confirm before running:
- Simulation Stop Time: 60 s
- Wind model: ACTIVE (F_wind = [0.22;0;0] + Dryden noise)
- arm_cmd Constant: 1
- All BLWN blocks: Ts = 0.001 (not 0.005)
- All To Workspace blocks saving: `x_hat_x`, `x_hat_y`, `x_hat_log`, `x_true`, `y_true`, `z`, `phi`, `theta`, `psi`, `mode`

Run the simulation.

---

## Step 2 — Extract Signals

```matlab
% ─── Time vector ─────────────────────────────────────────────────
t      = out.x_hat_x.Time;

% ─── EKF position estimates ──────────────────────────────────────
x_est  = squeeze(out.x_hat_x.Data);
y_est  = squeeze(out.x_hat_y.Data);

% ─── EKF full state (for attitude) ───────────────────────────────
xh = squeeze(out.x_hat_log.Data);        % 9 × N after squeeze
if size(xh,1) ~= 9
    xh = xh';                            % ensure 9 × N
end
z_est     = xh(3,:)';
phi_est   = xh(7,:)';
theta_est = xh(8,:)';
psi_est   = xh(9,:)';

% ─── True states ─────────────────────────────────────────────────
x_true    = squeeze(out.x_true.Data);
y_true    = squeeze(out.y_true.Data);
z_true    = squeeze(out.z.Data);
phi_true  = squeeze(out.phi.Data);
theta_true= squeeze(out.theta.Data);
psi_true  = squeeze(out.psi.Data);

% ─── Resample onto common time vector if needed ──────────────────
if ~isequal(out.x_true.Time, t)
    x_true    = interp1(out.x_true.Time,    x_true,    t,'linear','extrap');
    y_true    = interp1(out.y_true.Time,    y_true,    t,'linear','extrap');
    z_true    = interp1(out.z.Time,         z_true,    t,'linear','extrap');
    phi_true  = interp1(out.phi.Time,       phi_true,  t,'linear','extrap');
    theta_true= interp1(out.theta.Time,     theta_true,t,'linear','extrap');
    psi_true  = interp1(out.psi.Time,       psi_true,  t,'linear','extrap');
end

% ─── Mode signal ─────────────────────────────────────────────────
mode_data = squeeze(out.mode.Data);
```

---

## Step 3 — Compute RMSE

```matlab
% ─── Hover window ────────────────────────────────────────────────
mask = (t >= 5.0) & (t <= 35.0);

% ─── Position errors (vs setpoint, not vs true — this is control RMSE) ──
RMSE_x = sqrt(mean(x_est(mask).^2));          % setpoint = 0
RMSE_y = sqrt(mean(y_est(mask).^2));          % setpoint = 0
RMSE_z = sqrt(mean((z_est(mask) - (-1.0)).^2)); % setpoint = -1.0

% ─── Attitude estimation errors (EKF vs truth) ───────────────────
err_phi   = atan2(sin(phi_est(mask)-phi_true(mask)),   cos(phi_est(mask)-phi_true(mask)));
err_theta = atan2(sin(theta_est(mask)-theta_true(mask)),cos(theta_est(mask)-theta_true(mask)));
err_psi   = atan2(sin(psi_est(mask)-psi_true(mask)),   cos(psi_est(mask)-psi_true(mask)));

RMSE_phi   = sqrt(mean(err_phi.^2))   * 180/pi;
RMSE_theta = sqrt(mean(err_theta.^2)) * 180/pi;
RMSE_psi   = sqrt(mean(err_psi.^2))   * 180/pi;

% ─── Print summary table ─────────────────────────────────────────
function s = pf(ok); if ok; s='PASS'; else; s='FAIL'; end; end

fprintf('\n=================================================================\n');
fprintf('Week 5 — Full Autopilot RMSE (30 s hover, wind ON, EKF-in-loop)\n');
fprintf('=================================================================\n');
fprintf('Position  x  RMSE: %6.4f m    [Target < 0.50 m]   %s\n', RMSE_x,   pf(RMSE_x<0.5));
fprintf('Position  y  RMSE: %6.4f m    [Target < 0.50 m]   %s\n', RMSE_y,   pf(RMSE_y<0.5));
fprintf('Altitude  z  RMSE: %6.4f m    [Target < 0.15 m]   %s\n', RMSE_z,   pf(RMSE_z<0.15));
fprintf('Roll    phi  RMSE: %6.4f deg  [Target < 3.00 deg] %s\n', RMSE_phi,  pf(RMSE_phi<3));
fprintf('Pitch theta  RMSE: %6.4f deg  [Target < 3.00 deg] %s\n', RMSE_theta,pf(RMSE_theta<3));
fprintf('Yaw     psi  RMSE: %6.4f deg  [Target < 5.00 deg] %s\n', RMSE_psi,  pf(RMSE_psi<5));
fprintf('=================================================================\n\n');
```

**Take a screenshot of this output.** Save as `rmse_validation_wk5_YOUR_NAME.png`.

---

## Step 4 — Trajectory Plots

```matlab
% ─── Figure: Full autopilot trajectory ───────────────────────────
figure('Position', [50 50 1400 900]);
sgtitle('Week 5 — Full Autopilot (EKF-in-Loop, Wind ON, 60 s)', ...
        'FontSize', 14, 'FontWeight', 'bold');

% ── Panel 1: z trajectory ────────────────────────────────────────
subplot(2,2,1);
plot(t, -z_est,   'b-',  'LineWidth', 1.5, 'DisplayName', 'EKF altitude');
hold on;
plot(t, -z_true,  'r--', 'LineWidth', 1.0, 'DisplayName', 'True altitude');
yline(1.0, 'k:', 'Setpoint 1 m', 'LineWidth', 1.2);
xlabel('Time (s)'); ylabel('Altitude (m)');
title(sprintf('Altitude   RMSE = %.4f m', RMSE_z));
legend; grid on; xlim([0 60]);

% ── Panel 2: x trajectory ────────────────────────────────────────
subplot(2,2,2);
plot(t, x_est,  'b-',  'LineWidth', 1.5, 'DisplayName', 'EKF x');
hold on;
plot(t, x_true, 'r--', 'LineWidth', 1.0, 'DisplayName', 'True x');
yline(0, 'k:', 'Setpoint 0 m', 'LineWidth', 1.2);
xlabel('Time (s)'); ylabel('North x (m)');
title(sprintf('North Position   RMSE = %.4f m', RMSE_x));
legend; grid on; xlim([0 60]);

% ── Panel 3: y trajectory ────────────────────────────────────────
subplot(2,2,3);
plot(t, y_est,  'b-',  'LineWidth', 1.5, 'DisplayName', 'EKF y');
hold on;
plot(t, y_true, 'r--', 'LineWidth', 1.0, 'DisplayName', 'True y');
yline(0, 'k:', 'Setpoint 0 m', 'LineWidth', 1.2);
xlabel('Time (s)'); ylabel('East y (m)');
title(sprintf('East Position   RMSE = %.4f m', RMSE_y));
legend; grid on; xlim([0 60]);

% ── Panel 4: Attitude ────────────────────────────────────────────
subplot(2,2,4);
plot(t, rad2deg(phi_est),   'g-',  'LineWidth', 1.2, 'DisplayName', '\phi (EKF)');
hold on;
plot(t, rad2deg(theta_est), 'm-',  'LineWidth', 1.2, 'DisplayName', '\theta (EKF)');
plot(t, rad2deg(psi_est),   'k-',  'LineWidth', 1.2, 'DisplayName', '\psi (EKF)');
plot(t, rad2deg(phi_true),  'g--', 'LineWidth', 0.8, 'DisplayName', '\phi (true)');
plot(t, rad2deg(theta_true),'m--', 'LineWidth', 0.8, 'DisplayName', '\theta (true)');
xlabel('Time (s)'); ylabel('Angle (deg)');
title('Attitude: EKF vs Truth');
legend('Location','best'); grid on; xlim([0 60]);

saveas(gcf, 'trajectory_plots_wk5_YOUR_NAME.png');
fprintf('Saved: trajectory_plots_wk5_YOUR_NAME.png\n');
```

---

## Expected Results

| Metric | Typical well-tuned range |
|---|---|
| x RMSE | 0.10 – 0.35 m (wind in x direction, PIDs correct) |
| y RMSE | 0.05 – 0.20 m (less disturbance in y) |
| z RMSE | 0.03 – 0.10 m |
| φ RMSE | 0.5° – 2° |
| θ RMSE | 0.5° – 2.5° (higher with position PIDs commanding pitch) |
| ψ RMSE | 1° – 4° |

If x RMSE exceeds 0.5 m: increase X_Position_PID Ki by 0.02 and re-run. Ki accumulates integral action that corrects the steady offset from the constant wind.

---

## Checklist

- [ ] Simulation runs 60 s without crash
- [ ] Stateflow mode sequences 1 → 2 → 3 → 4 in scope
- [ ] x RMSE < 0.5 m (R6 PASS)
- [ ] y RMSE < 0.5 m (R6 PASS)
- [ ] z RMSE < 0.15 m (PASS)
- [ ] φ, θ RMSE < 3° (PASS)
- [ ] ψ RMSE < 5° (PASS)
- [ ] RMSE Command Window output screenshot saved as `rmse_validation_wk5_YOUR_NAME.png`
- [ ] 4-panel trajectory figure saved as `trajectory_plots_wk5_YOUR_NAME.png`
- [ ] All panels have axis labels, RMSE in title, grid, legend
