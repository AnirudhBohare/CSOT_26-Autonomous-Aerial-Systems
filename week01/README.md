# Week 1 — Foundations, Requirements & Flight Dynamics
### Autonomous Aerial Systems · CSOT 2026 · AeroClub, IIT Delhi

> **Dates:** 1–8 Jun 2026
> 
> **Points:** 20 pts
> 
> **Deadline:** To be announced

---

## 🎯 What This Week Is About

Week 1 is little rigorous — it covers what would normally be two separate weeks, so plan your time accordingly. By Sunday you need to have done three things:

1. **Installed MATLAB R2026** and verified all required toolboxes.
2. **Written a formal Requirements Document** — the engineering specification your entire project will be tested against in Week 5.
3. **Built your first Simulink model** — a 6-DOF quadcopter dynamics model that you will build on every week from here.

This is the most front-loaded week. Everything in Weeks 2–5 plugs into the model you start here. Take the dynamics model seriously.

---


## 🔧 Tasks This Week

### Task 1 — Install MATLAB R2026 (1 hour)
**Guide:** [`tasks/TASK_1_install.md`](./tasks/TASK_1_install.md)

Required toolboxes: Simulink · Control System Toolbox · Aerospace Blockset · Stateflow · Signal Processing Toolbox

**Deliverable:** Screenshot of Simulink Library Browser showing all toolboxes. (1 pt)

---

### Task 2 — Watch Video Resources  (2 hours)
**Guide:** [`tasks/TASK_2_videos.md`](./tasks/TASK_2_videos.md)

Will be shared shortly

---

### Task 3 — Write the Requirements Document (1.5 hours)
**Guide:** [`tasks/TASK_3_requirements.md`](./tasks/TASK_3_requirements.md)
**Template:** [`submissions/requirements_template.md`](./submissions/requirements_template.md)

Write a formal requirements document with ≥8 measurable functional requirements covering all of R1–R8. Each requirement must follow the format:

```
[ID] | The autopilot shall [do X] | [numeric metric with unit] | [test method]
```

This document becomes your Week 5 verification matrix. Write it properly now.

**Deliverable:** `requirements_doc_YOUR_NAME.pdf` (8 pts)

---

### Task 4 — Build the 6-DOF Flight Dynamics Model (2.5 hours)
**Guide:** [`tasks/TASK_4_sixdof.md`](./tasks/TASK_4_sixdof.md)

This is the core engineering task of Week 1. You will build a Simulink model of a quadcopter's flight dynamics from scratch.

**What to build:**
- **Inputs:** 4 individual rotor thrust values (T1, T2, T3, T4)
- **Outputs:** Position [x, y, z], velocity [vx, vy, vz], attitude [roll, pitch, yaw], angular rates [p, q, r]
- **Subsystems:** Force & Torque Mixer → Translational Dynamics → Rotational Dynamics → Kinematics
- **Hover Trim block:** computes the equilibrium rotor speed for steady level flight

**Parameters to use:**
| Parameter | Value |
|---|---|
| Mass (m) | 0.5 kg |
| Gravity (g) | 9.81 m/s² |
| Ixx | 4.9 × 10⁻³ kg·m² |
| Iyy | 4.9 × 10⁻³ kg·m² |
| Izz | 8.8 × 10⁻³ kg·m² |
| Thrust coefficient (kT) | 2.9 × 10⁻⁵ N/(rad/s)² |
| Torque coefficient (kQ) | 5.7 × 10⁻⁷ N·m/(rad/s)² |
| Arm length (L) | 0.225 m |

**Trim verification:** Apply hover thrust to all 4 motors. Run the simulation for 5 seconds. Position and attitude must remain constant (within ±0.01 m / ±0.01°). Compute % error between your simulated hover thrust and the analytical value.

**Deliverable:** `autopilot_wk1_YOUR_NAME.slx` + trim results table (10 pts)

---

## 📦 What to Submit

All files in `week01/submissions/YOUR_NAME/`:

| # | File | Points |
|---|---|---|
| 1 | `requirements_doc_YOUR_NAME.pdf` | 8 pts |
| 2 | `autopilot_wk1_YOUR_NAME.slx` | 10 pts |
| 3 | `matlab_screenshot_YOUR_NAME.png` | 1 pt |
| 4 | `notes_YOUR_NAME.md / .txt / .pdf` | 1 pt |
| 5 | `trim_results_YOUR_NAME.md / .png` | included in slx pts |

Full submission guide: [`submissions/SUBMISSION_GUIDE.md`](./submissions/SUBMISSION_GUIDE.md)

---

## 📊 How You Are Scored

| Criterion | Points | What evaluators check |
|---|---|---|
| Requirements — completeness (R1–R8 all present) | 4 pts | All 8 official requirements covered |
| Requirements — measurability (numeric metric per req) | 4 pts | Each requirement has a number, unit, and test method |
| 6-DOF model — functional & subsystem-organised | 6 pts | Model runs, correct inputs/outputs, clean subsystem structure |
| 6-DOF model — trim analysis | 4 pts | % error vs analytical hover thrust, trim plot included |
| MATLAB installation screenshot | 1 pt | Library browser visible with all toolboxes |
| Video annotation notes | 1 pt | Evidence of engagement with all 4 videos |
| **Total** | **20 pts** | |

Full rubric: [`evaluation/RUBRIC.md`](./evaluation/RUBRIC.md)

---

## ⏱️ Suggested Time Split (7 hours total)

| Day | Task | Time |
|---|---|---|
| Day 1 | Install MATLAB R2026 + Watch Videos 1 & 2 | 2.5 hrs |
| Day 2 | Write requirements document | 1.5 hrs |
| Day 3 | Watch Videos 3 & 4 + start Simulink model | 2 hrs |
| Day 4 | Complete model, run trim analysis, submit | 1 hr |

---

## 💡 Tips

- **On the model:** Build one subsystem at a time. Start with the Force & Torque Mixer (pure algebra), then Translational Dynamics, then Rotational Dynamics. Test each subsystem before connecting them.
- **On requirements:** Every requirement must answer: "Can a MATLAB script automatically output PASS or FAIL based on this?" If not, rewrite it.
- **On time:** Task 4 (Simulink) is the hardest. If you're running out of time, submit a partial model with what you have — partial credit is better than nothing.
- **On toolboxes:** Install all 5 toolboxes now. Re-running the installer mid-week costs you an hour you don't have.

---

## ✅ End-of-Week Checklist

- [ ] MATLAB R2026 installed with all 5 toolboxes
- [ ] Library browser screenshot saved
- [ ] All 4 videos watched with notes covering both requirements and dynamics
- [ ] Requirements document has ≥ 8 requirements, all with numeric metrics
- [ ] All 8 official requirements (R1–R8) covered
- [ ] 6-DOF Simulink model built with correct inputs, outputs, and subsystems
- [ ] Hover trim verified — position and attitude stay constant
- [ ] % error computed and recorded
- [ ] All 4–5 files placed in `week01/submissions/YOUR_NAME/`
- [ ] Pull Request opened: `[Week 1] Your Name — Submission`

---

## 📚 Resources

[`resources/RESOURCES.md`](./resources/RESOURCES.md)

---

*Questions? Open a GitHub Issue tagged `week1-question`.*
*AeroClub, IIT Delhi · CSOT 2026*
