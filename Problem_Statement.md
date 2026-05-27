# Autonomous Aerial Systems — Official Problem Statement
### AeroClub, IIT Delhi · CSOT 2026 · Aerial Domain

*This is the official problem statement as submitted to CAIC by AeroClub, IIT Delhi.*

---

## 1. Background & Theme

Modern unmanned aerial vehicles (UAVs) are among the most complex cyber-physical systems in engineering. A quadcopter, despite its apparent simplicity, requires the seamless integration of rigid-body dynamics, multi-axis control, real-time sensor fusion, and intelligent flight management to operate autonomously and safely.

In the aerospace and defence industries, these systems are designed using **Model-Based Design (MBD)** — a methodology where the system is first built and tested entirely in simulation before a single line of embedded code is written or a physical prototype is assembled. Tools like MATLAB and Simulink are the industry standard for this workflow, used by companies including Boeing, Airbus, NASA JPL, DJI, and Skydio.

The inter-IIT Tech Meet (Aerial domain) consistently features challenges rooted in this same systems-engineering paradigm — requiring participants to understand not just how to fly a drone, but how to design, model, and validate the system that makes it fly.

**Theme:** Model-Based Design for Autonomous Aerial Systems — from mathematical model to validated autopilot, entirely in simulation.

---

## 2. Abstract

Autonomous Aerial Systems is a 5-week structured training series in which participants design and validate a complete quadcopter autopilot in MATLAB/Simulink, following industry-standard Model-Based Design practices. Starting from the governing physics of a quadcopter, participants progressively build each layer of an autopilot stack — flight dynamics model, virtual sensor suite, cascaded PID controller, Extended Kalman Filter for sensor fusion, and a structured flight manager. Each week, a new module is added to the same Simulink project, so the final submission is a single, cohesive, fully-integrated system.

The series culminates in a simulated autonomous waypoint-following mission validated against a formal requirements matrix — directly mirroring the verification workflow expected at the inter-IIT Tech Meet.

---

## 3. Problem Statement & Objectives

You are an avionics engineer at a UAV startup. Your team has been tasked with designing and validating a complete autopilot system for a quadcopter that must operate autonomously, reject environmental disturbances, and pass a full requirements-verification suite before flight clearance.

The system must satisfy the following requirements:

| ID | Requirement | Pass Criterion |
|---|---|---|
| R1 | Altitude hold | Maintain altitude within ±0.3 m during hover |
| R2 | Attitude control | Keep roll/pitch error within ±5° under disturbance |
| R3 | Disturbance rejection | Stable flight in crosswind up to 3 m/s |
| R4 | Sensor noise tolerance | Operate correctly with realistic IMU noise and bias |
| R5 | Autonomous navigation | Follow a 3-waypoint GPS mission without manual input |
| R6 | State estimation | Estimate position/velocity using sensor fusion (EKF) |
| R7 | Graceful mode transitions | Smooth Takeoff → Hover → Land state machine |
| R8 | Requirements traceability | All requirements verified via a test matrix |

---

## 4. Weekly Structure

Each week consists of 2–3 tutorial videos from the curated playlist, followed by one hands-on Simulink task submitted to the CSOT portal. Submissions are evaluated within 48 hours. The leaderboard is updated live.

| Week | Dates | Topic |
|---|---|---|
| Wk 1 | Jun 1–7 | Foundations & Flight Dynamics Modelling — MBD intro, 6-DOF physics model in Simulink |
| Wk 2 | Jun 8–14 | Environment & Sensor Models — Wind, IMU, baro, magnetometer |
| Wk 3 | Jun 15–21 | Flight Control Law & Manager — Cascaded PID + mode state machine |
| Wk 4 | Jun 22–28 | Sensor Fusion (EKF) — Extended Kalman Filter integration |
| Wk 5 | Jun 29–Jul 6 | Validation & Final Project — Requirements testing + waypoint mission |

---

## 5. Evaluation & Leaderboard

| Week | Deliverable | Key Metric | Points |
|---|---|---|---|
| Wk 1 | Requirements Document (≥8 functional requirements) | Completeness + measurability | 10 |
| Wk 2 | 6-DOF Simulink model + trim analysis | % error vs analytical hover thrust | -- |
| Wk 3 | Sensor models + wind environment + comparison plots | SNR of simulated IMU (dB) | -- |
| Wk 4 | Cascaded PID + 3-mode flight manager | Altitude RMSE (m) over 30 s hover | -- |
| Wk 5 | EKF sensor fusion, closed-loop test | Position estimation error (m) | -- |
| Wk 6 | Full system + waypoint mission + report + video | Mission success + overall RMSE | -- |
| | | **Total** | **100 (+20 bonus)** |


---

## 6. Learning Outcomes

1. **Simulink & MBD proficiency** — Build and simulate multi-subsystem models using Simulink, Simscape, and MATLAB Live Scripts — the same workflow used in aerospace R&D.
2. **Flight dynamics understanding** — Derive and implement the 6-DOF equations of motion for a quadcopter, understand thrust/torque mixing, and identify trim conditions.
3. **Control systems design** — Design, tune, and evaluate cascaded PID controllers using root-locus and frequency-domain methods; understand stability margins.
4. **Sensor modelling & noise analysis** — Model realistic IMU, barometer, and magnetometer behaviour including noise, bias, and calibration errors.
5. **Kalman filtering & state estimation** — Implement an Extended Kalman Filter for multi-sensor fusion; understand prediction-correction cycles and covariance propagation.
6. **Requirements-based engineering** — Write measurable system requirements, trace them to test cases, and produce a verification matrix — directly inter-IIT tech meet aligned.
7. **Technical documentation** — Produce a structured project report and recorded demonstration — ready for CV, portfolio, or GitHub publication.

---

## 7. Final Project — Autonomous Waypoint Mission

The Week 5 final project integrates all modules built across the series into a single Simulink project that executes an autonomous mission:

- Takeoff from ground to a defined altitude
- Navigate through 3 GPS waypoints in a simulated 3D environment
- Maintain altitude hold and attitude stability throughout, with a 3 m/s crosswind active
- Land safely at the origin point
- Log all states, sensor readings, and EKF estimates throughout the mission

**Submission package includes:**
- `.slx` Simulink project file — fully self-contained and runnable
- Requirements Verification Matrix (pass/fail for all 8 requirements)
- 2-minute screen-recorded demo video of the mission simulation
- 2-page project report: approach, results, limitations, future work

*Top performers from Autonomous Aerial Systems will be given priority consideration for AeroClub's inter-IIT Tech Meet team (Aerial domain).*

---

## 8. Incentives & Recognition

**For Participants:** 

- ECA credits 
- Live leaderboard ranking  
- CV-worthy project + GitHub portfolio  
- Inter-IIT Tech Meet team priority 
- Participation certificate on completion 

---

* AeroClub, IIT Delhi · CSOT 2026 · Series name: Autonomous Aerial Systems · Domain: Aerial*
