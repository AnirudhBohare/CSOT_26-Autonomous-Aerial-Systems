# Bonus Task — Monte Carlo Analysis
### Week 5 · Autonomous Aerial Systems · CSOT 2026

>**Points:** 20 pts (bonus — added on top of the 25 core pts)

>**Estimated time:** 1.5 hours

>**Deliverable:** `monte_carlo_wk5_YOUR_NAME.png` + `mc_rmse_table_wk5_YOUR_NAME.png`

---

## What Monte Carlo Analysis Is

A single simulation run gives you one RMSE number. But that number depends on the random noise seed chosen for every Band-Limited White Noise block. A different seed produces a different noise sequence, a different disturbance, and a different RMSE.

Monte Carlo analysis answers: **across the full distribution of possible noise realisations, how does the autopilot perform?**

You run the same model 30 times with different seeds. Each run produces one RMSE value per state. After 30 runs you have a distribution of RMSE values — from which you can compute mean ± σ and check what fraction of runs pass the R6 targets.

This is standard practice in flight control validation — it demonstrates that your autopilot is robust, not just lucky on one noise draw.

---

## Step 1 — Identify All BLWN Seeds in Your Model

Your model has Band-Limited White Noise blocks in:

| Subsystem | Block name | Variable to change |
|---|---|---|
| IMU → Accelerometer model | BLWN1, BLWN2, BLWN3 | `seed` parameter |
| IMU → Gyroscope model | BLWN1, BLWN2, BLWN3 | `seed` parameter |
| Barometer | BLWN | `seed` parameter |
| Magnetometer | BLWN | `seed` parameter |
| Wind Model | BLWN | `seed` parameter |

Find the exact Simulink block paths for each. In MATLAB command window, before writing the loop, run:

```matlab
% Find all BLWN blocks and their paths
blwn_blocks = find_system('autopilot_wk5_YOUR_NAME', ...
              'BlockType', 'Reference', ...
              'SourceBlock', 'simulink/Sources/Band-Limited White Noise');
disp(blwn_blocks);
```

Note: In some MATLAB versions BLWN are tagged as `'ReferenceBlock'` instead. If the above returns empty, try:
```matlab
blwn_blocks = find_system('autopilot_wk5_YOUR_NAME', 'RegExp','on', ...
              'MaskType', 'Band-Limited White Noise');
```

Copy the returned cell array of paths into your Monte Carlo script. You will need these paths for `set_param`.

---

## Step 2 — Write the Monte Carlo Loop

Create a new MATLAB script named `run_monte_carlo_wk5.m` in the same folder as your `.slx`:

```matlab
%% Monte Carlo Analysis — Week 5 Autopilot
% Run the simulation 30 times with different BLWN seeds
% Compute RMSE for x, y, z, phi, theta, psi on each run

clear mc_rmse;
model_name = 'autopilot_wk5_YOUR_NAME';     % match your .slx filename (no extension)
load_system(model_name);

N_runs = 30;
hover_t_start = 5.0;
hover_t_end   = 35.0;

% Pre-allocate results: rows = runs, cols = [x, y, z, phi, theta, psi]
rmse_mc = zeros(N_runs, 6);

% BLWN block paths — paste output of find_system here
% (replace with your actual paths)
blwn_paths = {
    [model_name '/Sensor Suite/IMU/Accelerometer model/Band-Limited White Noise1'],
    [model_name '/Sensor Suite/IMU/Accelerometer model/Band-Limited White Noise2'],
    [model_name '/Sensor Suite/IMU/Accelerometer model/Band-Limited White Noise3'],
    [model_name '/Sensor Suite/IMU/Gyroscope model/Band-Limited White Noise'],
    [model_name '/Sensor Suite/IMU/Gyroscope model/Band-Limited White Noise1'],
    [model_name '/Sensor Suite/IMU/Gyroscope model/Band-Limited White Noise2'],
    [model_name '/Sensor Suite/Barometer/Band-Limited White Noise'],
    [model_name '/Sensor Suite/Magnetometer/Band-Limited White Noise'],
    [model_name '/Wind Model/Band-Limited White Noise'],
};

fprintf('Starting Monte Carlo: %d runs\n', N_runs);
tic;

for run_idx = 1:N_runs
    fprintf('  Run %d / %d ... ', run_idx, N_runs);
    
    % ── Set unique seeds for all BLWN blocks ───────────────────────
    base_seed = run_idx * 1000;     % ensures seeds differ across runs
    for k = 1:numel(blwn_paths)
        try
            set_param(blwn_paths{k}, 'seed', num2str(base_seed + k));
        catch e
            warning('Could not set seed on %s: %s', blwn_paths{k}, e.message);
        end
    end
    
    % ── Run simulation ─────────────────────────────────────────────
    simOut = sim(model_name, 'StopTime', '60');
    
    % ── Extract and compute RMSE ───────────────────────────────────
    t = simOut.x_hat_x.Time;
    mask = (t >= hover_t_start) & (t <= hover_t_end);
    
    x_est = squeeze(simOut.x_hat_x.Data);
    y_est = squeeze(simOut.x_hat_y.Data);
    
    xh = squeeze(simOut.x_hat_log.Data);
    if size(xh,1) ~= 9; xh = xh'; end
    z_est     = xh(3,:)';
    phi_est   = xh(7,:)';
    theta_est = xh(8,:)';
    psi_est   = xh(9,:)';
    
    phi_true  = squeeze(simOut.phi.Data);
    theta_true= squeeze(simOut.theta.Data);
    psi_true  = squeeze(simOut.psi.Data);
    
    % Resample true states if needed
    if ~isequal(simOut.phi.Time, t)
        phi_true   = interp1(simOut.phi.Time,   phi_true,   t,'linear','extrap');
        theta_true = interp1(simOut.theta.Time, theta_true, t,'linear','extrap');
        psi_true   = interp1(simOut.psi.Time,   psi_true,   t,'linear','extrap');
    end
    
    % RMSE computation
    rmse_mc(run_idx, 1) = sqrt(mean(x_est(mask).^2));
    rmse_mc(run_idx, 2) = sqrt(mean(y_est(mask).^2));
    rmse_mc(run_idx, 3) = sqrt(mean((z_est(mask) - (-1.0)).^2));
    rmse_mc(run_idx, 4) = sqrt(mean(atan2(sin(phi_est(mask)-phi_true(mask)), ...
                                          cos(phi_est(mask)-phi_true(mask))).^2)) * 180/pi;
    rmse_mc(run_idx, 5) = sqrt(mean(atan2(sin(theta_est(mask)-theta_true(mask)), ...
                                          cos(theta_est(mask)-theta_true(mask))).^2)) * 180/pi;
    rmse_mc(run_idx, 6) = sqrt(mean(atan2(sin(psi_est(mask)-psi_true(mask)), ...
                                          cos(psi_est(mask)-psi_true(mask))).^2)) * 180/pi;
    
    fprintf('x=%.3f y=%.3f z=%.4f phi=%.3f theta=%.3f psi=%.3f\n', rmse_mc(run_idx,:));
end

elapsed = toc;
fprintf('\nMonte Carlo complete: %.1f s total (%.1f s/run avg)\n\n', elapsed, elapsed/N_runs);

% ── Save results ──────────────────────────────────────────────────
save('mc_results_wk5.mat', 'rmse_mc');
```

---

## Step 3 — Generate Histogram Plots

```matlab
%% Monte Carlo — RMSE distribution plots
% (Run after the loop above, or load from mc_results_wk5.mat)
% load('mc_results_wk5.mat');   % uncomment if loading saved results

state_labels = {'x position (m)', 'y position (m)', 'z altitude (m)', ...
                'Roll \phi (deg)', 'Pitch \theta (deg)', 'Yaw \psi (deg)'};
targets     = [0.5, 0.5, 0.15, 3.0, 3.0, 5.0];
target_lbls = {'R6: 0.5 m', 'R6: 0.5 m', 'Wk4: 0.15 m', ...
               '3.0 deg', '3.0 deg', '5.0 deg'};

figure('Position', [50 50 1400 900]);
sgtitle(sprintf('Monte Carlo RMSE Distribution — %d Runs (30 s hover, wind ON)', N_runs), ...
        'FontSize', 13, 'FontWeight', 'bold');

for k = 1:6
    subplot(2, 3, k);
    data = rmse_mc(:, k);
    histogram(data, 10, 'FaceColor', [0.2 0.5 0.9], 'EdgeColor', 'white', 'FaceAlpha', 0.8);
    hold on;
    xline(targets(k), 'r--', 'LineWidth', 2, 'DisplayName', target_lbls{k});
    xline(mean(data), 'g-', 'LineWidth', 2, 'DisplayName', sprintf('Mean=%.4f', mean(data)));
    xlabel(state_labels{k}); ylabel('Count');
    title(sprintf('%s\nMean=%.4f, SD=%.4f', state_labels{k}, mean(data), std(data)));
    legend('Location','best'); grid on;
    pass_rate = mean(data < targets(k)) * 100;
    text(0.97, 0.92, sprintf('Pass: %.0f%%', pass_rate), ...
         'Units','normalized','HorizontalAlignment','right','FontSize',9,...
         'Color', [0 0.5 0], 'FontWeight','bold');
end

saveas(gcf, 'monte_carlo_wk5_YOUR_NAME.png');
fprintf('Saved: monte_carlo_wk5_YOUR_NAME.png\n');
```

---

## Step 4 — Generate Summary Table Screenshot

```matlab
%% Monte Carlo — Summary table (print and screenshot)
states     = {'x (m)', 'y (m)', 'z (m)', 'phi (deg)', 'theta (deg)', 'psi (deg)'};
targets_v  = [0.5,     0.5,     0.15,    3.0,          3.0,           5.0];

fprintf('\n=================================================================\n');
fprintf('Monte Carlo RMSE Summary — %d Runs  (30 s hover, wind ON)\n', N_runs);
fprintf('=================================================================\n');
fprintf('%-14s %10s %10s %10s %8s %8s\n', ...
        'State', 'Mean', 'Std Dev', 'Max', 'Target', 'Pass%');
fprintf('%s\n', repmat('-',1,64));
for k = 1:6
    d = rmse_mc(:,k);
    pct = mean(d < targets_v(k)) * 100;
    fprintf('%-14s %10.4f %10.4f %10.4f %8.3f %7.0f%%\n', ...
            states{k}, mean(d), std(d), max(d), targets_v(k), pct);
end
fprintf('=================================================================\n');
```

**Screenshot this output.** Save as `mc_rmse_table_wk5_YOUR_NAME.png`.

---

## Expected Monte Carlo Results

For a well-tuned autopilot you should see:

| State | Typical mean RMSE | Typical σ | Expected pass rate |
|---|---|---|---|
| x position | 0.10 – 0.25 m | 0.03 – 0.08 m | > 95% |
| y position | 0.05 – 0.15 m | 0.02 – 0.05 m | > 98% |
| z altitude | 0.02 – 0.08 m | 0.01 – 0.03 m | > 99% |
| Roll φ | 0.5° – 2° | 0.1° – 0.5° | > 98% |
| Pitch θ | 0.5° – 2° | 0.1° – 0.5° | > 95% |
| Yaw ψ | 1° – 4° | 0.3° – 1° | > 97% |

Pass rates below 90% for x or y suggest the position PIDs are marginal — increase Ki slightly and re-run.

---

## Common Mistakes

| Mistake | Fix |
|---|---|
| `set_param` fails — block path wrong | Run `find_system` again with the exact model name; paths are case-sensitive |
| All 30 runs return identical RMSE | Seed is not actually changing — verify `set_param` with `get_param` after setting |
| `sim()` returns error on some runs | Add try/catch around sim() and skip failed runs; report N successful runs |
| Monte Carlo takes > 30 min | Normal for 30 × 60-second sims. Run overnight or reduce to N=15 for a quicker result |

---

## Checklist

- [ ] `find_system` used to locate all BLWN block paths
- [ ] `for` loop runs N=30 simulations
- [ ] `set_param` changes seed before each run (verified with `get_param`)
- [ ] `rmse_mc` matrix filled: 30 rows × 6 columns
- [ ] Results saved to `mc_results_wk5.mat`
- [ ] 6-panel histogram figure saved as `monte_carlo_wk5_YOUR_NAME.png`
- [ ] Summary table screenshot saved as `mc_rmse_table_wk5_YOUR_NAME.png`
- [ ] x pass rate > 90%, y pass rate > 90%
- [ ] Mean ± σ reported for all 6 states
ENDOFFILE
echo "Done"
