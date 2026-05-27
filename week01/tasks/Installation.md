# Task 1 — Install MATLAB R2026
### Week 1 · Autonomous Aerial Systems · CSOT 2026

**Estimated time:** 1 hour (most of it is download time)
**Deliverable:** Screenshot of the Simulink Library Browser (1 pt)

---

## Required Toolboxes

Install all five — they are all used in Weeks 1–5. Do not skip any.

| Toolbox | First used |
|---|---|
| Simulink | Week 1 |
| Control System Toolbox | Week 3 |
| Aerospace Blockset | Week 1 |
| Stateflow | Week 3 |
| Signal Processing Toolbox | Week 2 |

---

## Installation Options

### Option A — IIT Delhi Campus Licence (Recommended)
1. Go to your institute's software portal and log in with institute credentials.
2. Download the **MATLAB R2026** installer for your OS.
3. Run the installer, sign in with your MathWorks account linked to your institute email.
4. Select all 5 toolboxes above on the product selection screen.
5. Complete installation (~30–60 min).

### Option B — MathWorks Student Licence
1. Go to [mathworks.com/academia/tah-portal](https://www.mathworks.com/academia/tah-portal.html)
2. Create a MathWorks account with your **institute email** (.iitd.ac.in )
3. Follow download and activation steps.

### Option C — MATLAB Online (temporary fallback only)
[matlab.mathworks.com](https://matlab.mathworks.com) — browser-based, free with academic licence. Works for Week 1 notes and requirements but **will not run all Simulink blocks from Week 2 onwards**. Sort your local install before the Week 2 deadline.

I'd suggest you to prefer Option A, it would be easier for future weeks.

---

## Verify Your Installation

Run these in the MATLAB Command Window:

```matlab
simulink          % Opens Simulink start page
ver('control')    % Should return version info, not an error
ver('aerospace')  % Should return version info
ver('stateflow')  % Should return version info
ver('signal')     % Should return version info
```

If any `ver()` command returns an error, that toolbox is missing — re-run the installer and add it.

---

## Taking the Required Screenshot

1. Open MATLAB → type `simulink` → press Enter
2. In the Simulink Start Page, press **Ctrl+Shift+L** to open the Library Browser
3. Expand the library tree — you should see all toolboxes listed
4. Screenshot the Library Browser with the following visible:
   - Simulink
   - Aerospace Blockset
   - Control System Toolbox
   - Signal Processing Toolbox
   - Stateflow
5. Save as `matlab_screenshot_YOUR_NAME.png`

---

## Common Problems

| Problem | Fix |
|---|---|
| Installer asks for licence file | Use online activation (MathWorks account sign-in) |
| Toolbox missing after install | Re-run installer → select missing toolbox |
| MATLAB opens, Simulink doesn't | Simulink was not selected during install — re-run |
| macOS security block | Right-click installer → Open → Open Anyway |
| Activation fails on campus network | Try a different network or disable VPN |

---

## Checklist

- [ ] MATLAB R2026 opens without errors
- [ ] `simulink` opens the Simulink start page
- [ ] `ver('control')` returns version info
- [ ] `ver('aerospace')` returns version info
- [ ] `ver('stateflow')` returns version info
- [ ] `ver('signal')` returns version info
- [ ] Screenshot saved: `matlab_screenshot_YOUR_NAME.png`
