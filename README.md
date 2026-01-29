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


 # Database Design

This section lists  **each table** , its  **fields** , and a  **clear explanation of what every field means and why it exists** .

The design supports:

* Anonymous incident reporting
* Machine learning trust scoring
* Device motion verification
* Evidence file uploads
* Police review and confirmation
* Hotspot (cluster) detection

---

## 1. devices — Anonymous Reporter Devices

**Meaning:**
Stores anonymous device behavior and trust history. No names, phone numbers, or personal data are stored.

**Purpose:**
Tracks how reliable a reporting device is over time.

**Table: devices**

* **device_id (Primary Key)**
  Unique database ID for each device. Used to link the device to reports.
* **device_hash**
  Anonymous unique identifier generated from the device. Prevents duplicate identities while protecting privacy.
* **first_seen_at**
  Date and time when the device first submitted a report. Helps distinguish new devices from long-term reporters.
* **total_reports**
  Total number of reports submitted by the device. Used in behavior analysis and ML features.
* **trusted_reports**
  Number of reports confirmed as valid. Measures device reliability.
* **flagged_reports**
  Number of reports flagged or rejected. Helps detect spam or abuse.
* **device_trust_score**
  Calculated trust score for the device. Used by ML models when evaluating new reports.

---

## 2. reports — Incident Reports (Main Table)

**Meaning:**
Stores every reported incident. Each row represents one incident report.

**Purpose:**
Records what happened, where it happened, when it happened, and basic motion verification data.

**Table: reports**

* **report_id (Primary Key)**
  Unique identifier for each report. Links to ML results, police reviews, evidence, and hotspots.
* **device_id (Foreign Key → devices.device_id)**
  Identifies which device submitted the report. Links report credibility to device history.
* **incident_type**
  Category of the incident (e.g., theft, vandalism). Used for ML patterns and hotspot analysis.
* **description**
  Short text description of what happened. Used for human review and NLP-based ML features.

### Location Fields

* **latitude**
  GPS latitude of the incident location. Required for mapping and clustering.
* **longitude**
  GPS longitude of the incident location. Used with latitude for hotspot detection.
* **gps_accuracy**
  GPS accuracy in meters. Low accuracy reduces trust in location data and affects ML scoring.

### Motion Snapshot Fields

* **motion_level**
  Movement intensity (low / medium / high) at report time. Helps verify reporter presence.
* **movement_speed**
  Estimated movement speed of the device. Detects fake or remote submissions.
* **was_stationary**
  Indicates whether the device was still. Helps identify spoofed reports.

### Status Fields

* **reported_at**
  Date and time the report was submitted. Used for time-based analysis and hotspot windows.
* **rule_status**
  Result of rule-based validation (passed / flagged / rejected). Used as an ML feature.
* **is_flagged**
  Marks suspicious reports for fast filtering and alerts.

---

## 3. evidence_files — Photos and Videos

**Meaning:**
Stores links to evidence files related to reports.

**Purpose:**
Allows reports to include photos or videos without storing large files in the database.

**Table: evidence_files**

* **evidence_id (Primary Key)**
  Unique identifier for each evidence record.
* **report_id (Foreign Key → reports.report_id)**
  Identifies which report the evidence belongs to.
* **file_url**
  Path or cloud storage link to the file.
* **file_type**
  Type of file (photo or video). Used for processing and ML weighting.
* **uploaded_at**
  Date and time the evidence was uploaded. Tracks delayed evidence submissions.

---

## 4. ml_results — Machine Learning Scores

**Meaning:**
Stores machine learning evaluation results for reports.

**Purpose:**
Shows how trustworthy the system believes each report is.

**Table: ml_results**

* **ml_id (Primary Key)**
  Unique identifier for each ML evaluation.
* **report_id (Foreign Key → reports.report_id)**
  Identifies which report was evaluated.
* **trust_score**
  Numerical reliability score (0–100). Used for prioritization and hotspot inclusion.
* **prediction_label**
  ML decision label (likely_real / suspicious / fake). Easy for humans to understand.
* **model_version**
  Version of the ML model used. Supports retraining and audits.
* **evaluated_at**
  Timestamp when the ML model evaluated the report.

---

## 5. police_reviews — Police Decisions

**Meaning:**
Stores official police feedback on reports.

**Purpose:**
Provides ground-truth decisions for ML training and system trust.

**Table: police_reviews**

* **review_id (Primary Key)**
  Unique identifier for each police review.
* **report_id (Foreign Key → reports.report_id)**
  Identifies the reviewed report.
* **decision**
  Police decision: confirmed, rejected, or needs_investigation.
* **review_note**
  Officer comments or investigation notes.
* **reviewed_at**
  Date and time the review was completed. Used for response-time analysis.
* **unit_code**
  Police unit identifier (not personal names). Ensures accountability while protecting privacy.

---

## 6. hotspots — Detected Incident Clusters

**Meaning:**
Represents areas with clustered trusted incident reports.

**Purpose:**
Identifies high-risk locations using DBSCAN clustering.

**Table: hotspots**

* **hotspot_id (Primary Key)**
  Unique identifier for each hotspot cluster.
* **center_lat**
  Latitude of the hotspot center.
* **center_long**
  Longitude of the hotspot center.
* **radius_meters**
  Radius defining the affected area.
* **incident_count**
  Number of reports in the cluster. Indicates hotspot strength.
* **risk_level**
  Risk classification (low / medium / high). Used for dashboard prioritization.
* **detected_at**
  Time when the hotspot was detected.
* **time_window_hours**
  Time span covered by the clustered reports. Distinguishes short spikes from long trends.

---

## 7. hotspot_reports — (Bridge Table)

**Meaning:**
Links reports to hotspot clusters.

**Purpose:**
Shows which reports formed each hotspot.

**Table: hotspot_reports**

* **hotspot_id (Foreign Key → hotspots.hotspot_id)**
  Identifies the hotspot cluster.
* **report_id (Foreign Key → reports.report_id)**
  Identifies the report in the cluster.
* **Primary Key (hotspot_id, report_id)**
  Ensures each report is linked only once per hotspot.

---

## Relationship Summary

* One device → many reports
* One report → many evidence files
* One report → one ML result
* One report → zero or one police review
* One hotspot → many reports
* hotspot_reports links reports to hotspots
