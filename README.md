# CSOT_26-Autonomous-Aerial-Systems
Model-Based Design for Autonomous Aerial Systems — from mathematical model to validated autopilot, entirely in simulation ( Simulink /MATLAB R2026b).
# 🛸 Autonomous Aerial Systems
### AeroClub, IIT Delhi · CSOT 2026 · Aerial Domain

> **Design and validate a complete quadcopter autopilot in MATLAB/Simulink using industry-standard Model-Based Design — from physics to fully autonomous flight, entirely in simulation.**

---

## 📋 Quick Facts

| Field | Details |
|---|---|
| **Series** | Autonomous Aerial Systems — Drone Autopilot Design & Simulation |
| **Club** | AeroClub, IIT Delhi |
| **Domain** | Aerial — CSOT 2026 |
| **Duration** | 5 Weeks (1 June – 6 July 2026) |
| **Effort** | 3–4 hours / week |
| **Tools** | MATLAB R2026b / Simulink |
| **Total Points** | 100 pts + 20 bonus |
| **Prerequisite** | Basic physics or control systems |
| **Final Output** | Autopilot system + report + demo video |

---

## 🎯 What You Will Build

By the end of Week 5, you will have a **single, cohesive Simulink project** that:

- Models the full **6-DOF flight dynamics** of a quadcopter
- Simulates a **realistic sensor suite** (IMU, barometer, magnetometer) with noise and bias
- Runs a **cascaded PID controller** for altitude hold and attitude stabilisation
- Uses an **Extended Kalman Filter (EKF)** for sensor fusion — no true states fed to the controller
- Executes a **3-waypoint autonomous GPS mission** with a 3 m/s crosswind active
- Passes a formal **Requirements Verification Matrix** against 8 system requirements

This mirrors exactly the workflow used at aerospace companies (Boeing, Airbus, NASA JPL, DJI) and is directly aligned with the **inter-IIT Tech Meet Aerial domain** challenge format.

---

## 🗓️ Weekly Overview

| Week | Dates | Topic | Points |
|---|---|---|---|
| [Week 1](./week01/README.md) | 25–31 May | Foundations & Requirements |
| [Week 2](./week02/README.md) | 1–7 Jun | Flight Dynamics Modelling (6-DOF) |
| [Week 3](./week03/README.md) | 8–14 Jun | Environment & Sensor Models | 15 pts |
| [Week 4](./week04/README.md) | 15–21 Jun | Flight Control Law & Flight Manager | 
| [Week 5](./week05/README.md) | 22–28 Jun | Sensor Fusion — Extended Kalman Filter | 
| [Week 6](./week06/README.md) | 29 Jun–6 Jul | Validation, Testing & Waypoint Mission | 
| | | **Total** | **100 + 20 bonus** |

---

## 📐 System Requirements (R1–R8)

These are the 8 formal requirements your autopilot must satisfy. Every week builds toward verifying them.

| ID | Requirement | Pass Criterion |
|---|---|---|
| **R1** | Altitude hold — maintain altitude during hover | Within ±0.3 m |
| **R2** | Attitude control — reject disturbances on roll/pitch | Within ±5° |
| **R3** | Disturbance rejection — stable flight in crosswind | Stable at 3 m/s |
| **R4** | Sensor noise tolerance — correct operation with IMU noise & bias | Operates correctly |
| **R5** | Autonomous navigation — 3-waypoint GPS mission | All 3 WPs reached |
| **R6** | State estimation — position/velocity via EKF | Error reported in m |
| **R7** | Graceful mode transitions — Takeoff → Hover → Land | No abrupt jumps |
| **R8** | Requirements traceability — all reqs verified via test matrix | Complete pass/fail matrix |

---

## 🗂️ Repository Structure

```
autonomous-aerial-systems/
│
├── README.md                      ← You are here
├── CONTRIBUTING.md                ← Submission rules & formatting guide
├── LEADERBOARD.md                 ← Live scores (updated within 48 hrs)
│
├── week01/                        ← Foundations & Requirements
│   ├── README.md                  ← Full week guide (read this first)
│   ├── tasks/
│   │   ├── TASK_1_install.md      ← MATLAB installation checklist
│   │   ├── TASK_2_videos.md       ← Video-watching guide with annotations
│   │   └── TASK_3_requirements.md ← How to write the requirements doc
│   ├── submissions/
│   │   ├── SUBMISSION_GUIDE.md    ← What to submit, how, and where
│   │   └── requirements_template.md ← Starter template for your doc
│   ├── evaluation/
│   │   ├── RUBRIC.md              ← Exact scoring criteria (for trainees)
│   │   └── EVALUATOR_GUIDE.md     ← Detailed guide for evaluators only
│   └── resources/
│       └── RESOURCES.md           ← Links, references, reading material
│
├── week02/ … week05/              ← Same structure, added each week
│
├── docs/
│   ├── PROBLEM_STATEMENT.md       ← Full official problem statement
│   └── REQUIREMENTS_MATRIX.md    ← Master R1–R8 verification matrix template
│
└── scripts/
    └── check_submission.m         ← MATLAB script to self-check Week 1 doc
```

---

## 🚀 Getting Started (Trainees)

### Step 1 — Fork this repository
Click **Fork** (top right) and clone your fork:
```bash
git clone https://github.com/YOUR_USERNAME/autonomous-aerial-systems.git
cd autonomous-aerial-systems
```

### Step 2 — Read Week 1
Go to [`week01/README.md`](./week01/README.md). Read it **completely** before doing anything.

### Step 3 — Install MATLAB
Follow [`week01/tasks/TASK_1_install.md`](./week01/tasks/TASK_1_install.md).

### Step 4 — Submit
Follow [`week01/submissions/SUBMISSION_GUIDE.md`](./week01/submissions/SUBMISSION_GUIDE.md) exactly.

---


## 📌 Important Links

- 📺 **Playlist:** [Drone Autopilot Design & Simulation in Simulink](https://youtube.com/playlist?list=PLw9UeyR2OgE2uzMYHbvc1SLhZFiI9uM4p)
- 📄 **Official Problem Statement:** [`docs/PROBLEM_STATEMENT.md`](./docs/PROBLEM_STATEMENT.md)
- 📬 **Submission portal:** CSOT 2025 portal (link shared separately)

---

## ⚖️ Rules

1. All Simulink models must run on **MATLAB R2026b** with no missing toolboxes.
2. Submissions after the deadline receive a penalty (leaderboard score deduction).
3. Your `.slx` file must be **self-contained** — no broken paths or missing blocks.
4. You may discuss concepts with peers but all code/models must be your own work.
5. Screenshots must show **your own** MATLAB environment (watermark/username visible).

---

*AeroClub · IIT Delhi · CSOT 2026 *
