# Task 2 — Learn from Video Resources with Annotations
### Week 1 · Autonomous Aerial Systems · CSOT 2026

**Estimated time:** 2 hours (videos + note-taking)

**Deliverable:** Annotation notes file covering the things u feel necessary.

---

## Videos 

Watch in this order: Video 1 → Video 2 → (write requirements) → Video 3 → Video 4 → (build model).

- Video1 : https://youtu.be/likYtvPMi_4?si=LdMDr5mo344aipJP
- Video2 : https://youtu.be/FTP9j-4FAig?si=hSJ92OvDt8-VFRug
- Video3 : https://youtu.be/kxFRBKJurI4?si=jFt3n17xizsoM2ku
- Video4 : https://youtu.be/Dx8YTXxVjAU?si=GV2TinO263Rfw-qU

---

## Video 1 — Intro to Quadcopter Autopilot & MBD

Learn by answering each question as it comes up in the video.

- Q1: How many rotors does a quadcopter have and how are they arranged?
- Q2: Which rotors spin CW and which CCW? Why does this alternating pattern matter?
- Q3: How does the quadcopter produce thrust (up/down)?
- Q4: How does it produce roll (tilt left/right)?
- Q5: How does it produce pitch (tilt forward/back)?
- Q6: How does it produce yaw (rotate on vertical axis)?
- Q7: What are the 3 main stages of Model-Based Design?
- Q8: Which companies use MBD? List at least 3.
- Q9: Draw the autopilot block diagram shown. Label every block.

---

## Video 2 — Autopilot Function Requirements

- Q10: How does the presenter define a functional requirement?
- Q11: What makes a requirement measurable?
- Q12: Why is "the drone should fly stably" NOT a good requirement?
- Q13: List every autopilot function the presenter names, with its metric if stated.
- Q14: What is requirements traceability?
- Q15: What is the difference between a functional and a performance requirement?

---

## Video 3 — Autopilot System Architecture

- Q16: Draw the full system architecture from this video. Label every layer and subsystem.
- Q17: What are the inputs and outputs of the Flight Dynamics block?
- Q18: What does the "mixer" do? What goes in and what comes out?
- Q19: Where does the sensor data enter the system?
- Q20: What is the difference between the inner loop and outer loop in the controller?

---

## Video 4 — Modelling Flight Dynamics & Physical Systems

- Q21: What coordinate frame is used (NED / ENU / body)? What are the sign conventions?
- Q22: Write down the thrust equation: how does rotor speed produce force?
- Q23: Write down the torque mixing equations for roll, pitch, and yaw.
- Q24: What are Euler's equations of motion for rotation?
- Q25: What is a trim condition? How is hover thrust computed analytically?
- Q26: Sketch the Simulink subsystem structure shown in the video.

---

## Synthesis Questions (answer after all 4 videos)

1. Draw the full autopilot pipeline from memory: sensor → estimator → controller → mixer → motors → plant → back to sensor.
2. What are the 8 official system requirements (R1–R8)? For each, write why it matters for a real drone in one sentence.
3. What is one thing you did not fully understand across the 4 videos? Write the question clearly.

---

## Submission

Save as `notes_YOUR_NAME.md`, `.txt`, or `.pdf`.

**Minimum for full marks:** Answers to all 3 synthesis questions. Bullet points are fine.

**Scores 0:** "Watched all videos" with no content.
