# Machine Learning Integration – Step by Step

This section explains **how machine learning works in this project**, step by step.

---

## STEP 1: Data Collection (Foundation)

Machine learning works only with **data** (information collected by the system).
The system collects **anonymous report data** from the mobile application.

No personal identity (names, phone numbers) is collected.

### Data collected from each report

- **Location data**
  - GPS location of the incident (where it happened)
  - Location accuracy (how close the phone location is to the real place)

- **Time data**
  - Date and time the report is sent (when it was reported)
  - Day of the week (Monday, Tuesday, etc.)

- **Report content**
  - Type of incident (theft, vandalism, suspicious activity, etc.)
  - Short written explanation (what the user says happened)

- **Evidence**
  - Photo or video, if provided (proof of the incident)

- **Device behavior (anonymous)**
  - Phone movement (shows the user was physically present)
  - Speed and movement pattern (how the phone was moving)

- **System-generated data**
  - Report ID (unique number for each report)
  - Anonymous device ID (system code that does not identify the user)

Machine learning analyzes patterns in reports, not people.

---

## STEP 2: Rule-Based Verification (Before Machine Learning)

Before machine learning is used, the system applies basic automatic rules.

### Example rules

- Location must match the reported incident
- Report time must be realistic
- Device must be close to the incident location
- Evidence must be original

### Output

Each report is marked as:
- Passed (looks okay)
- Flagged (needs attention)
- Rejected (clearly false)

At this stage, no police are involved.
This step is fully automatic.

---

## STEP 3: Feature Preparation (Making Data Usable for ML)

Machine learning cannot understand raw data directly.
The system converts data into **features** (simple values machine learning can read).

### Examples of features

- Distance from incident (how far the user was)
- GPS accuracy (how correct the location is)
- Evidence provided (yes or no)
- Report length (text size)
- Time category (day or night)
- Device trust history (past reliability)

---

## STEP 4: Initial Labeling (Teaching the System)

Machine learning must learn what is real and what is fake.

### How labels are created

- Police-confirmed past reports
- Multiple reports from the same area
- Clearly false reports

### Labels used

- Real / Verified
- False / Suspicious

This step uses past police decisions, not current ones.

---

## STEP 5: Training the Machine Learning Model

The system trains the machine learning model using labeled data.

The model learns:
- Real reports usually come from nearby devices
- Fake reports often have no evidence
- Trusted devices report consistently
- Some locations and times have repeated incidents

Models used:
- Logistic Regression
- Random Forest
- Gradient Boosting

---

## STEP 6: Trust Scoring (Machine Learning Decision)

When a new report is submitted:
1. The report passes rule checks
2. Machine learning evaluates the report
3. A Trust Score (0–100) is assigned

Example scores:
- 85 – Very reliable
- 40 – Suspicious
- 10 – Likely fake

At this stage, the decision is made only by the system.
Police have not confirmed or rejected the report yet.

---

## STEP 7: Hotspot Detection (Before Police Review)

Machine learning uses trust scores to identify **hotspots** (areas with many incidents in a short time).

- High trust reports count more
- Reports close in location and time are grouped
- Low trust reports are ignored

Hotspots are created before police confirmation to show early warning areas.

---

## STEP 8: Police Review and Decision

After machine learning scoring and hotspot detection:
1. Reports appear on the police dashboard
2. Police review the report and evidence
3. Police confirm or reject the report

Police decisions do not change the current trust score.
They are used only to improve future decisions.

---

## STEP 9: Continuous Learning (Using Police Feedback)

Machine learning improves over time using police decisions.

- Police decisions are saved
- Training data is updated
- The model retrains regularly

Results:
- Better future trust scores
- Fewer false reports
- More accurate hotspots

---

## STEP 10: Privacy Protection

- No names collected
- No phone numbers stored
- No facial recognition
- No tracking of individuals

Machine learning uses anonymous behavior data only.

---

## Simple System Flow

Report → Rules → Machine Learning → Hotspots → Police Review → Machine Learning Learns

---

## One-Sentence Summary

Machine learning evaluates reports first, police confirm or reject them afterward, and those police decisions help the system improve over time while fully protecting user privacy.
