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


 # Database Design — Incident Reporting & ML Trust System

This project uses a simple and reliable database structure to support:

- Anonymous incident reporting
- Machine learning trust scoring
- Device motion verification
- Evidence file uploads
- Police review and confirmation
- Hotspot (cluster) detection

## Core Tables Overview

1. `devices` — anonymous reporter devices
2. `reports` — incident reports
3. `evidence_files` — photos and videos
4. `ml_results` — machine learning scores
5. `police_reviews` — police decisions
6. `hotspots` — detected incident clusters
7. `hotspot_reports` — links reports to hotspots

## devices : This is to mean "Anonymous Reporter Devices

Stores anonymous device behavior and trust history.
No names or phone numbers are stored.

**Table: devices**

- device_id (Primary Key)
- device_hash (unique anonymous identifier)
- first_seen_at
- total_reports
- trusted_reports
- flagged_reports
- device_trust_score

**Purpose:**
Tracks how reliable a device has been over time.

One device can submit many reports.

## reports — Incident Reports (Main Table).

This is the core table of the system. Each row is one incident report.

**Table: reports**

- report_id (Primary Key)
- device_id (Foreign Key → devices.device_id)
- incident_type
- description

**Location fields**

- latitude
- longitude
- gps_accuracy

**Motion snapshot fields**

- motion_level
- movement_speed
- was_stationary

**Status fields**

- reported_at
- rule_status
- is_flagged

**Purpose:**
Stores what happened, where, when, and basic motion verification data captured at report time.

## evidence_files — Photos and Videos

Stores uploaded evidence file links.
Actual files are stored in object storage — only links are stored here.

**Table: evidence_files**

- evidence_id (Primary Key)
- report_id (Foreign Key → reports.report_id)
- file_url
- file_type
- uploaded_at

## ml_results — Machine Learning Scores

Stores machine learning evaluation results for each report.

**Table: ml_results**

- ml_id (Primary Key)
- report_id (Foreign Key → reports.report_id)
- trust_score
- prediction_label
- model_version
- evaluated_at

**Purpose:**
Stores how trustworthy the ML model believes the report is.

## police_reviews — Police Decisions

Stores police confirmation or rejection of reports.

**Table: police_reviews**

- review_id (Primary Key)
- report_id (Foreign Key → reports.report_id)
- decision
- review_note
- reviewed_at
- unit_code

**Decision values:**

- confirmed
- rejected
- needs_investigation

**Purpose:**
Provides the true decision used for ML training and system feedback.

A report may or may not have a police review yet.

## hotspots — Detected Incident Clusters

Stores hotspot areas detected using DBSCAN clustering.

**Table: hotspots**

- hotspot_id (Primary Key)
- center_lat
- center_long
- radius_meters
- incident_count
- risk_level
- detected_at
- time_window_hours

**Purpose:**
Each row represents one high-risk area formed by grouping nearby trusted reports.

## hotspot_reports — Hotspot Membership (Bridge Table)

Links reports to hotspot clusters.

**Table: hotspot_reports**

- hotspot_id (Foreign Key → hotspots.hotspot_id)
- report_id (Foreign Key → reports.report_id)
- Primary Key (hotspot_id, report_id)

**Purpose:**
Allows many reports to belong to one hotspot cluster.

## Relationship Summary

- One device → many reports
- One report → many evidence files
- One report → one ML result
- One report → zero or one police review
- One hotspot → many reports
- `hotspot_reports` links reports to hotspots
