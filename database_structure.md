# TrustBond Database Structure v2.0

## Privacy-Preserving Anonymous Community Incident Reporting System

---

## System Overview

This database structure contains **15 tables** designed to support:

- Anonymous incident reporting (no personal data stored)
- **3-Rule Verification System** enhanced by **Machine Learning**
- Evidence grouping (multiple reports of same incident → one unified case)
- Police dashboard for case management
- Public safety map for community awareness

---

## How Machine Learning Works with 3-Rule Verification

### The 3 Hardcoded Verification Rules

| Rule                              | What It Checks           | Pass Criteria                                   |
| --------------------------------- | ------------------------ | ----------------------------------------------- |
| **1. Location Verification**      | GPS coordinates validity | Within Rwanda boundaries, accuracy ≤ 100 meters |
| **2. Motion Sensor Verification** | Device movement patterns | Sensor score ≥ 30 (proves human holding device) |
| **3. Duplicate Detection**        | Similar nearby reports   | Groups reports within 200m radius + 24hr window |

### How ML Enhances These Rules

```
┌─────────────────────────────────────────────────────────────────────┐
│                    REPORT SUBMISSION FLOW                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   CITIZEN SUBMITS REPORT                                            │
│            │                                                        │
│            ▼                                                        │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │         STEP 1: 3-RULE VERIFICATION (Hardcoded)             │   │
│   ├─────────────────────────────────────────────────────────────┤   │
│   │  Rule 1: Location Check                                     │   │
│   │          ├── GPS within Rwanda boundaries?                  │   │
│   │          └── Location accuracy ≤ 100 meters?                │   │
│   │                                                             │   │
│   │  Rule 2: Motion Sensor Check                                │   │
│   │          ├── Accelerometer data present?                    │   │
│   │          └── Motion score ≥ 30?                             │   │
│   │                                                             │   │
│   │  Rule 3: Duplicate Detection                                │   │
│   │          ├── Similar report from same device? (reject)      │   │
│   │          └── Similar reports nearby? (group into case)      │   │
│   │                                                             │   │
│   │  RESULT: Pass all 3? → Continue to ML analysis              │   │
│   │          Fail any? → Flag for review or reject              │   │
│   └─────────────────────────────────────────────────────────────┘   │
│            │                                                        │
│            ▼                                                        │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │         STEP 2: MACHINE LEARNING ANALYSIS                   │   │
│   ├─────────────────────────────────────────────────────────────┤   │
│   │                                                             │   │
│   │  ML MODEL receives all data from 3 rules and analyzes:      │   │
│   │                                                             │   │
│   │  📍 Location Pattern Analysis:                               │   │
│   │     • Learns GPS spoofing patterns (fake locations)         │   │
│   │     • Identifies areas with genuine vs suspicious reports   │   │
│   │     • Detects impossible location jumps (teleportation)     │   │
│   │                                                             │   │
│   │  📱 Motion Pattern Analysis:                                 │   │
│   │     • Learns authentic human device movement signatures     │   │
│   │     • Detects bot/automated submission patterns             │   │
│   │     • Identifies scripted motion data                       │   │
│   │                                                             │   │
│   │  🔗 Clustering Intelligence:                                 │   │
│   │     • Improves grouping of related reports                  │   │
│   │     • Learns incident type patterns per location            │   │
│   │     • Predicts likely true incidents vs false reports       │   │
│   │                                                             │   │
│   │  OUTPUT:                                                    │   │
│   │     • ml_trust_score (0.00 - 1.00)                          │   │
│   │     • priority_level (low/medium/high/critical)             │   │
│   │     • anomaly_flags (suspicious patterns detected)          │   │
│   │     • cluster_confidence (how certain reports belong)       │   │
│   └─────────────────────────────────────────────────────────────┘   │
│            │                                                        │
│            ▼                                                        │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │         STEP 3: FINAL PROCESSING                            │   │
│   ├─────────────────────────────────────────────────────────────┤   │
│   │                                                             │   │
│   │  • Report stored with ML scores                             │   │
│   │  • Linked to Unified Case (grouped with similar reports)    │   │
│   │  • Priority assigned based on ML prediction                 │   │
│   │  • Police dashboard shows case ordered by priority          │   │
│   │                                                             │   │
│   │  CITIZEN VIEW: "Report received" (doesn't know about ML)    │   │
│   │  POLICE VIEW: Full ML analysis + all grouped evidence       │   │
│   └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### ML Training Process

| Phase                   | Description                                                          |
| ----------------------- | -------------------------------------------------------------------- |
| **Initial Training**    | Model trained on historical reports where police confirmed validity  |
| **Continuous Learning** | Each police decision (verified/rejected) improves the model          |
| **Feature Inputs**      | Location data, motion sensor readings, time patterns, device history |
| **Model Output**        | Trust score, priority level, anomaly flags, cluster confidence       |

### What ML Learns Over Time

1. **Location Patterns**: Which areas have genuine reports vs where fake reports originate
2. **Time Patterns**: When genuine incidents typically occur vs suspicious timing
3. **Device Behavior**: How legitimate reporters use their devices vs bots
4. **Incident Correlations**: What incident types occur together in which locations

---

## 15 Tables Summary

| #   | Table Name            | Purpose                                                            | Category  |
| --- | --------------------- | ------------------------------------------------------------------ | --------- |
| 1   | `devices`             | Anonymous reporter devices with trust scores                       | Core      |
| 2   | `locations`           | Rwanda geography hierarchy (Province→District→Sector→Cell→Village) | Core      |
| 3   | `incident_types`      | Incident taxonomy (categories + types)                             | Core      |
| 4   | `unified_cases`       | Grouped incidents for police investigation                         | Core      |
| 5   | `incident_reports`    | Individual citizen submissions                                     | Core      |
| 6   | `report_evidence`     | Media files (photos, videos, audio)                                | Core      |
| 7   | `ml_models`           | Trained ML models for verification                                 | ML        |
| 8   | `ml_predictions`      | ML analysis results per report                                     | ML        |
| 9   | `police_users`        | Police authentication and roles                                    | Police    |
| 10  | `notifications`       | Alerts for police officers                                         | Police    |
| 11  | `hotspots`            | Crime cluster detection (multiple crime types per area)            | Analytics |
| 12  | `daily_statistics`    | Pre-aggregated analytics                                           | Analytics |
| 13  | `public_safety_zones` | Anonymized public safety map                                       | Public    |
| 14  | `system_settings`     | Application configuration                                          | System    |
| 15  | `activity_logs`       | Complete audit trail                                               | System    |

---

# TABLE 1: devices

## Anonymous Reporter Devices

**Purpose:** Tracks anonymous device behavior and trust history. No personal data stored.

| Field                   | Type                   | Description                               |
| ----------------------- | ---------------------- | ----------------------------------------- |
| **device_id** (PK)      | CHAR(36)               | UUID - Unique device identifier           |
| **device_fingerprint**  | VARCHAR(255)           | SHA-256 hash of hardware (non-reversible) |
| **platform**            | ENUM('android', 'ios') | Mobile platform                           |
| **app_version**         | VARCHAR(20)            | App version installed                     |
| **os_version**          | VARCHAR(30)            | Operating system version                  |
| **device_language**     | VARCHAR(10)            | Preferred language                        |
| **current_trust_score** | DECIMAL(5,2)           | Trust score (0-100), default 50           |
| **total_reports**       | INT DEFAULT 0          | Total reports submitted                   |
| **verified_reports**    | INT DEFAULT 0          | Reports confirmed as valid                |
| **false_reports**       | INT DEFAULT 0          | Reports confirmed as false                |
| **is_blocked**          | BOOLEAN DEFAULT FALSE  | Device blocked from reporting             |
| **block_reason**        | VARCHAR(255)           | Reason for blocking                       |
| **blocked_at**          | TIMESTAMP              | When blocked                              |
| **blocked_by**          | INT                    | FK → police_users                         |
| **push_token**          | VARCHAR(500)           | Encrypted push notification token         |
| **first_seen_at**       | TIMESTAMP              | First activity                            |
| **last_active_at**      | TIMESTAMP              | Last activity                             |

**Indexes:**

- PRIMARY KEY (device_id)
- UNIQUE (device_fingerprint)
- INDEX (current_trust_score)
- INDEX (is_blocked)

---

# TABLE 2: locations

## Rwanda Administrative Geography (Hierarchical)

**Purpose:** Single self-referential table for all Rwanda geography levels.

| Field                | Type                                                      | Description                          |
| -------------------- | --------------------------------------------------------- | ------------------------------------ |
| **location_id** (PK) | INT AUTO_INCREMENT                                        | Unique location identifier           |
| **parent_id** (FK)   | INT                                                       | Parent location (NULL for provinces) |
| **location_type**    | ENUM('province', 'district', 'sector', 'cell', 'village') | Level in hierarchy                   |
| **name**             | VARCHAR(100)                                              | Location name                        |
| **code**             | VARCHAR(40)                                               | Unique location code                 |
| **latitude**         | DECIMAL(10,8)                                             | Center latitude                      |
| **longitude**        | DECIMAL(11,8)                                             | Center longitude                     |
| **boundary_geojson** | JSON                                                      | Boundary polygon (optional)          |
| **population**       | INT                                                       | Population count                     |
| **is_active**        | BOOLEAN DEFAULT TRUE                                      | Active status                        |

**Hierarchy:**

```
Province (parent_id = NULL)
  └── District (parent_id = province_id)
       └── Sector (parent_id = district_id)
            └── Cell (parent_id = sector_id)
                 └── Village (parent_id = cell_id)
```

**Indexes:**

- PRIMARY KEY (location_id)
- INDEX (parent_id)
- INDEX (location_type)
- UNIQUE (code)
- INDEX (latitude, longitude)

---

# TABLE 3: incident_types

## Incident Taxonomy (Categories + Types Combined)

**Purpose:** Hierarchical incident classification. Categories have parent_id = NULL, types have parent_id = category.

| Field              | Type                    | Description                                  |
| ------------------ | ----------------------- | -------------------------------------------- |
| **type_id** (PK)   | SMALLINT AUTO_INCREMENT | Unique identifier                            |
| **parent_id** (FK) | SMALLINT                | NULL = Category, otherwise = parent category |
| **name**           | VARCHAR(100)            | Category or type name                        |
| **description**    | TEXT                    | Description                                  |
| **icon_name**      | VARCHAR(50)             | Material icon name                           |
| **color_hex**      | CHAR(7)                 | Color code (#1976D2)                         |
| **severity_level** | TINYINT                 | 1-5 (only for types, not categories)         |
| **display_order**  | TINYINT                 | Order in list                                |
| **is_active**      | BOOLEAN DEFAULT TRUE    | Active status                                |

**Example Data:**

```
type_id | parent_id | name              | severity_level
--------|-----------|-------------------|---------------
1       | NULL      | Theft             | NULL (category)
2       | NULL      | Violence          | NULL (category)
3       | 1         | Pickpocketing     | 2
4       | 1         | Vehicle Break-in  | 4
5       | 2         | Assault           | 5
```

**Indexes:**

- PRIMARY KEY (type_id)
- INDEX (parent_id)
- INDEX (is_active, display_order)

---

# TABLE 4: unified_cases

## Grouped Incident Cases for Law Enforcement

**Purpose:** Groups multiple reports of the same incident into ONE case for police. Citizens don't see this grouping.

| Field                        | Type                                                                    | Description                           |
| ---------------------------- | ----------------------------------------------------------------------- | ------------------------------------- |
| **case_id** (PK)             | CHAR(36)                                                                | UUID - Unique case identifier         |
| **case_number**              | VARCHAR(30)                                                             | Human-readable (CASE-2026-0130-001)   |
| **incident_type_id** (FK)    | SMALLINT                                                                | FK → incident_types                   |
| **location_id** (FK)         | INT                                                                     | FK → locations (resolved area)        |
| **latitude**                 | DECIMAL(10,8)                                                           | Case centroid latitude                |
| **longitude**                | DECIMAL(11,8)                                                           | Case centroid longitude               |
| **radius_meters**            | DECIMAL(8,2)                                                            | Radius covering all reports           |
| **address_description**      | VARCHAR(255)                                                            | Landmark description                  |
| **incident_date**            | DATE                                                                    | Date incident occurred                |
| **incident_time_start**      | TIME                                                                    | Earliest reported time                |
| **incident_time_end**        | TIME                                                                    | Latest reported time                  |
| **report_count**             | INT DEFAULT 1                                                           | Number of reports in case             |
| **evidence_count**           | INT DEFAULT 0                                                           | Total evidence files                  |
| **reporter_count**           | INT DEFAULT 1                                                           | Unique devices that reported          |
| **combined_trust_score**     | DECIMAL(5,2)                                                            | Weighted average trust score          |
| **ml_priority_score**        | DECIMAL(5,2)                                                            | ML-calculated priority (0-100)        |
| **ml_confidence**            | DECIMAL(5,4)                                                            | ML confidence in validity (0-1)       |
| **verification_status**      | ENUM('pending', 'verified', 'suspicious', 'false')                      | Overall verification                  |
| **location_verified**        | BOOLEAN                                                                 | Rule 1: Location check passed         |
| **motion_verified**          | BOOLEAN                                                                 | Rule 2: Motion sensor check passed    |
| **duplicate_checked**        | BOOLEAN                                                                 | Rule 3: Duplicate detection completed |
| **case_status**              | ENUM('new', 'reviewing', 'investigating', 'resolved', 'closed')         | Case workflow                         |
| **priority**                 | ENUM('low', 'normal', 'high', 'urgent') DEFAULT 'normal'                | Case priority                         |
| **assigned_officer_id** (FK) | INT                                                                     | FK → police_users                     |
| **assigned_at**              | TIMESTAMP                                                               | When assigned                         |
| **hotspot_id** (FK)          | INT                                                                     | FK → hotspots (if part of cluster)    |
| **resolution_type**          | ENUM('confirmed', 'false_report', 'duplicate', 'no_action', 'referred') | How resolved                          |
| **resolution_notes**         | TEXT                                                                    | Resolution details                    |
| **resolved_at**              | TIMESTAMP                                                               | When resolved                         |
| **resolved_by** (FK)         | INT                                                                     | FK → police_users                     |
| **created_at**               | TIMESTAMP                                                               | Case creation                         |
| **updated_at**               | TIMESTAMP                                                               | Last update                           |

**Indexes:**

- PRIMARY KEY (case_id)
- UNIQUE (case_number)
- INDEX (incident_type_id)
- INDEX (location_id)
- INDEX (latitude, longitude)
- INDEX (incident_date)
- INDEX (case_status, priority)
- INDEX (ml_priority_score)
- INDEX (assigned_officer_id)

---

# TABLE 5: incident_reports

## Individual Citizen Report Submissions

**Purpose:** Stores each citizen's report. Multiple reports can link to one unified_case. Citizens see their individual report status.

| Field                           | Type                                                    | Description                        |
| ------------------------------- | ------------------------------------------------------- | ---------------------------------- |
| **report_id** (PK)              | CHAR(36)                                                | UUID - Unique report identifier    |
| **device_id** (FK)              | CHAR(36)                                                | FK → devices (anonymous reporter)  |
| **case_id** (FK)                | CHAR(36)                                                | FK → unified_cases (grouped case)  |
| **incident_type_id** (FK)       | SMALLINT                                                | FK → incident_types                |
| **title**                       | VARCHAR(200)                                            | Optional short title               |
| **description**                 | TEXT                                                    | Incident description               |
| **latitude**                    | DECIMAL(10,8)                                           | GPS latitude                       |
| **longitude**                   | DECIMAL(11,8)                                           | GPS longitude                      |
| **location_accuracy_meters**    | DECIMAL(8,2)                                            | GPS accuracy                       |
| **location_source**             | ENUM('gps', 'network', 'manual')                        | How location obtained              |
| **location_id** (FK)            | INT                                                     | FK → locations (resolved location) |
| **reported_at**                 | TIMESTAMP                                               | Submission time                    |
| **incident_occurred_at**        | TIMESTAMP                                               | When incident happened             |
| **time_approximate**            | BOOLEAN DEFAULT FALSE                                   | User unsure of exact time          |
| **accelerometer_data**          | JSON                                                    | Raw motion sensor data             |
| **gyroscope_data**              | JSON                                                    | Rotation sensor data               |
| **motion_score**                | DECIMAL(5,2)                                            | 0-100 motion authenticity score    |
| **device_orientation**          | VARCHAR(20)                                             | portrait, landscape, flat          |
| **battery_level**               | TINYINT                                                 | Battery % at submission            |
| **network_type**                | VARCHAR(20)                                             | wifi, 4g, 3g                       |
| **location_check_passed**       | BOOLEAN                                                 | Rule 1 result                      |
| **motion_check_passed**         | BOOLEAN                                                 | Rule 2 result                      |
| **is_duplicate**                | BOOLEAN DEFAULT FALSE                                   | Same-device duplicate rejected     |
| **duplicate_of_report_id** (FK) | CHAR(36)                                                | Original report if duplicate       |
| **is_grouped**                  | BOOLEAN DEFAULT FALSE                                   | Merged into multi-device case      |
| **report_status**               | ENUM('submitted', 'processing', 'verified', 'rejected') | Report workflow                    |
| **rejection_reason**            | VARCHAR(255)                                            | Why rejected (if applicable)       |
| **citizen_notified**            | BOOLEAN DEFAULT FALSE                                   | Citizen received status update     |
| **app_version**                 | VARCHAR(20)                                             | App version used                   |
| **created_at**                  | TIMESTAMP                                               | Record creation                    |

**Indexes:**

- PRIMARY KEY (report_id)
- INDEX (device_id)
- INDEX (case_id)
- INDEX (incident_type_id)
- INDEX (latitude, longitude)
- INDEX (location_id)
- INDEX (reported_at)
- INDEX (report_status)

---

# TABLE 6: report_evidence

## Evidence Files (Photos, Videos, Audio)

**Purpose:** Stores evidence metadata. Files stored in encrypted cloud storage.

| Field                 | Type                            | Description                       |
| --------------------- | ------------------------------- | --------------------------------- |
| **evidence_id** (PK)  | CHAR(36)                        | UUID - Unique evidence identifier |
| **report_id** (FK)    | CHAR(36)                        | FK → incident_reports             |
| **case_id** (FK)      | CHAR(36)                        | FK → unified_cases                |
| **evidence_type**     | ENUM('photo', 'video', 'audio') | Type of evidence                  |
| **file_name**         | VARCHAR(255)                    | Original file name                |
| **file_path**         | VARCHAR(500)                    | Encrypted storage path            |
| **file_size_bytes**   | INT                             | File size                         |
| **mime_type**         | VARCHAR(100)                    | MIME type                         |
| **duration_seconds**  | SMALLINT                        | Duration (video/audio only)       |
| **width_pixels**      | SMALLINT                        | Width (photo/video)               |
| **height_pixels**     | SMALLINT                        | Height (photo/video)              |
| **file_hash**         | CHAR(64)                        | SHA-256 for integrity             |
| **captured_at**       | TIMESTAMP                       | When media was captured           |
| **capture_latitude**  | DECIMAL(10,8)                   | GPS from EXIF                     |
| **capture_longitude** | DECIMAL(11,8)                   | GPS from EXIF                     |
| **is_valid**          | BOOLEAN DEFAULT TRUE            | Quality check passed              |
| **quality_score**     | DECIMAL(5,2)                    | 0-100 quality score               |
| **uploaded_at**       | TIMESTAMP                       | Upload time                       |

**Indexes:**

- PRIMARY KEY (evidence_id)
- INDEX (report_id)
- INDEX (case_id)
- INDEX (evidence_type)

---

# TABLE 7: ml_models

## Machine Learning Models for Verification

**Purpose:** Stores trained ML models used to enhance the 3-rule verification system.

| Field                   | Type                                                                            | Description                                  |
| ----------------------- | ------------------------------------------------------------------------------- | -------------------------------------------- |
| **model_id** (PK)       | INT AUTO_INCREMENT                                                              | Unique model identifier                      |
| **model_name**          | VARCHAR(100)                                                                    | Model name                                   |
| **model_type**          | ENUM('trust_scoring', 'anomaly_detection', 'clustering', 'priority_prediction') | What model does                              |
| **model_version**       | VARCHAR(20)                                                                     | Version number (v1.0.0)                      |
| **description**         | TEXT                                                                            | What this model does                         |
| **algorithm**           | VARCHAR(50)                                                                     | Algorithm used (RandomForest, XGBoost, etc.) |
| **input_features**      | JSON                                                                            | List of features model uses                  |
| **output_format**       | JSON                                                                            | What model outputs                           |
| **training_data_count** | INT                                                                             | Number of samples trained on                 |
| **training_accuracy**   | DECIMAL(5,4)                                                                    | Training accuracy (0-1)                      |
| **validation_accuracy** | DECIMAL(5,4)                                                                    | Validation accuracy (0-1)                    |
| **model_file_path**     | VARCHAR(500)                                                                    | Path to model file                           |
| **model_file_hash**     | CHAR(64)                                                                        | Model file integrity hash                    |
| **is_active**           | BOOLEAN DEFAULT FALSE                                                           | Currently in use                             |
| **activated_at**        | TIMESTAMP                                                                       | When activated                               |
| **activated_by** (FK)   | INT                                                                             | FK → police_users                            |
| **trained_at**          | TIMESTAMP                                                                       | When training completed                      |
| **trained_by** (FK)     | INT                                                                             | FK → police_users                            |
| **last_retrained_at**   | TIMESTAMP                                                                       | Last retraining                              |
| **performance_metrics** | JSON                                                                            | Detailed performance data                    |
| **created_at**          | TIMESTAMP                                                                       | Record creation                              |

**Model Types Explained:**

| Model Type            | Purpose                                    | Enhances Which Rule  |
| --------------------- | ------------------------------------------ | -------------------- |
| `trust_scoring`       | Calculates overall trust score for reports | All 3 rules          |
| `anomaly_detection`   | Detects GPS spoofing, bot submissions      | Rule 1 + Rule 2      |
| `clustering`          | Improves incident grouping accuracy        | Rule 3               |
| `priority_prediction` | Predicts urgency/priority of cases         | Final prioritization |

**Indexes:**

- PRIMARY KEY (model_id)
- INDEX (model_type, is_active)
- INDEX (trained_at)

---

# TABLE 8: ml_predictions

## ML Analysis Results Per Report

**Purpose:** Stores ML prediction outputs for each incident report.

| Field                      | Type                                                | Description                              |
| -------------------------- | --------------------------------------------------- | ---------------------------------------- |
| **prediction_id** (PK)     | CHAR(36)                                            | UUID - Unique prediction identifier      |
| **report_id** (FK)         | CHAR(36)                                            | FK → incident_reports                    |
| **case_id** (FK)           | CHAR(36)                                            | FK → unified_cases                       |
| **model_id** (FK)          | INT                                                 | FK → ml_models (model used)              |
| **ml_trust_score**         | DECIMAL(5,4)                                        | Predicted trust (0.0000-1.0000)          |
| **ml_confidence**          | DECIMAL(5,4)                                        | Model confidence in prediction           |
| **predicted_priority**     | ENUM('low', 'medium', 'high', 'critical')           | ML suggested priority                    |
| **predicted_validity**     | ENUM('likely_genuine', 'uncertain', 'likely_false') | Validity prediction                      |
| **anomaly_score**          | DECIMAL(5,4)                                        | How anomalous (0=normal, 1=very unusual) |
| **anomaly_flags**          | JSON                                                | Specific anomalies detected              |
| **cluster_confidence**     | DECIMAL(5,4)                                        | How certain report belongs to its case   |
| **location_pattern_score** | DECIMAL(5,4)                                        | Location authenticity score              |
| **motion_pattern_score**   | DECIMAL(5,4)                                        | Motion authenticity score                |
| **time_pattern_score**     | DECIMAL(5,4)                                        | Time pattern score                       |
| **feature_importance**     | JSON                                                | Which features influenced prediction     |
| **raw_model_output**       | JSON                                                | Full model output for debugging          |
| **prediction_time_ms**     | INT                                                 | How long prediction took                 |
| **predicted_at**           | TIMESTAMP                                           | When prediction made                     |

**What Each Score Means:**

| Score                | Range       | Meaning                                     |
| -------------------- | ----------- | ------------------------------------------- |
| `ml_trust_score`     | 0.00 - 1.00 | Overall likelihood report is genuine        |
| `ml_confidence`      | 0.00 - 1.00 | How sure the model is about its prediction  |
| `anomaly_score`      | 0.00 - 1.00 | 0=completely normal, 1=highly suspicious    |
| `cluster_confidence` | 0.00 - 1.00 | How well this report fits with grouped case |

**Example Anomaly Flags:**

```json
{
  "gps_jump": true, // Location jumped impossibly fast
  "static_motion": false, // Normal motion detected
  "unusual_time": true, // Report at unusual hour for this area
  "repeat_pattern": false // Not a repeated false pattern
}
```

**Indexes:**

- PRIMARY KEY (prediction_id)
- INDEX (report_id)
- INDEX (case_id)
- INDEX (model_id)
- INDEX (ml_trust_score)
- INDEX (predicted_priority)
- INDEX (anomaly_score)

---

# TABLE 9: police_users

## Police Authentication and Roles

**Purpose:** Police accounts with role-based access control.

| Field                         | Type                                                                      | Description                   |
| ----------------------------- | ------------------------------------------------------------------------- | ----------------------------- |
| **user_id** (PK)              | INT AUTO_INCREMENT                                                        | Unique user identifier        |
| **username**                  | VARCHAR(50)                                                               | Unique username               |
| **email**                     | VARCHAR(255)                                                              | Unique email                  |
| **password_hash**             | VARCHAR(255)                                                              | Bcrypt hashed password        |
| **full_name**                 | VARCHAR(200)                                                              | Officer's name                |
| **badge_number**              | VARCHAR(50)                                                               | Unique badge number           |
| **phone_number**              | VARCHAR(20)                                                               | Contact phone                 |
| **role**                      | ENUM('super_admin', 'admin', 'commander', 'officer', 'analyst', 'viewer') | User role                     |
| **permissions**               | JSON                                                                      | Granular permissions override |
| **assigned_location_id** (FK) | INT                                                                       | FK → locations (jurisdiction) |
| **can_access_all_locations**  | BOOLEAN DEFAULT FALSE                                                     | National access               |
| **is_active**                 | BOOLEAN DEFAULT TRUE                                                      | Account active                |
| **two_factor_enabled**        | BOOLEAN DEFAULT FALSE                                                     | 2FA enabled                   |
| **two_factor_secret**         | VARCHAR(100)                                                              | TOTP secret                   |
| **password_changed_at**       | TIMESTAMP                                                                 | Last password change          |
| **failed_login_attempts**     | TINYINT DEFAULT 0                                                         | Failed logins                 |
| **locked_until**              | TIMESTAMP                                                                 | Account lockout time          |
| **last_login_at**             | TIMESTAMP                                                                 | Last successful login         |
| **last_login_ip**             | VARCHAR(45)                                                               | Last login IP                 |
| **created_at**                | TIMESTAMP                                                                 | Account creation              |
| **created_by** (FK)           | INT                                                                       | Who created account           |
| **updated_at**                | TIMESTAMP                                                                 | Last update                   |

**Role Permissions:**

| Role        | Description                             |
| ----------- | --------------------------------------- |
| super_admin | Full system access, ML model management |
| admin       | User management, settings               |
| commander   | District oversight, case assignments    |
| officer     | View/verify cases in jurisdiction       |
| analyst     | Analytics, ML insights, read-only       |
| viewer      | Dashboard only                          |

**Indexes:**

- PRIMARY KEY (user_id)
- UNIQUE (username)
- UNIQUE (email)
- UNIQUE (badge_number)
- INDEX (role)
- INDEX (assigned_location_id)

---

# TABLE 10: notifications

## Police Alerts and Notifications

**Purpose:** Alerts sent to police officers about cases.

| Field                    | Type                                                                                                | Description                           |
| ------------------------ | --------------------------------------------------------------------------------------------------- | ------------------------------------- |
| **notification_id** (PK) | CHAR(36)                                                                                            | UUID - Unique notification identifier |
| **user_id** (FK)         | INT                                                                                                 | FK → police_users (recipient)         |
| **notification_type**    | ENUM('new_case', 'urgent_case', 'case_update', 'hotspot_alert', 'assignment', 'ml_alert', 'system') | Type                                  |
| **title**                | VARCHAR(200)                                                                                        | Notification title                    |
| **message**              | TEXT                                                                                                | Notification content                  |
| **case_id** (FK)         | CHAR(36)                                                                                            | FK → unified_cases (if related)       |
| **hotspot_id** (FK)      | INT                                                                                                 | FK → hotspots (if related)            |
| **priority**             | ENUM('low', 'normal', 'high', 'urgent')                                                             | Urgency                               |
| **is_read**              | BOOLEAN DEFAULT FALSE                                                                               | Read status                           |
| **read_at**              | TIMESTAMP                                                                                           | When read                             |
| **is_dismissed**         | BOOLEAN DEFAULT FALSE                                                                               | Dismissed status                      |
| **action_url**           | VARCHAR(500)                                                                                        | Link to relevant page                 |
| **created_at**           | TIMESTAMP                                                                                           | When sent                             |
| **expires_at**           | TIMESTAMP                                                                                           | Auto-dismiss time                     |

**Indexes:**

- PRIMARY KEY (notification_id)
- INDEX (user_id, is_read)
- INDEX (notification_type)
- INDEX (created_at)

---

# TABLE 11: hotspots

## Crime Cluster Detection

**Purpose:** Detected incident clusters using ML-enhanced algorithms.

| Field                              | Type                                                  | Description                   |
| ---------------------------------- | ----------------------------------------------------- | ----------------------------- |
| **hotspot_id** (PK)                | INT AUTO_INCREMENT                                    | Unique hotspot identifier     |
| **hotspot_name**                   | VARCHAR(100)                                          | Generated name                |
| **location_id** (FK)               | INT                                                   | FK → locations (primary area) |
| **latitude**                       | DECIMAL(10,8)                                         | Centroid latitude             |
| **longitude**                      | DECIMAL(11,8)                                         | Centroid longitude            |
| **radius_meters**                  | DECIMAL(10,2)                                         | Cluster radius                |
| **boundary_geojson**               | JSON                                                  | Boundary polygon              |
| **case_count**                     | INT                                                   | Number of cases               |
| **total_reports**                  | INT                                                   | Total reports                 |
| **avg_trust_score**                | DECIMAL(5,2)                                          | Average trust score           |
| **avg_ml_confidence**              | DECIMAL(5,4)                                          | Average ML confidence         |
| **dominant_incident_type_id** (FK) | SMALLINT                                              | Most common type              |
| **dominant_type_percentage**       | DECIMAL(5,2)                                          | % of dominant type            |
| **risk_level**                     | ENUM('low', 'medium', 'high', 'critical')             | Risk assessment               |
| **priority_score**                 | DECIMAL(5,2)                                          | 0-100 priority                |
| **first_incident_at**              | TIMESTAMP                                             | Earliest incident             |
| **last_incident_at**               | TIMESTAMP                                             | Latest incident               |
| **peak_hour**                      | TINYINT                                               | 0-23, most active hour        |
| **peak_day_of_week**               | TINYINT                                               | 0-6, most active day          |
| **status**                         | ENUM('active', 'monitoring', 'addressed', 'resolved') | Status                        |
| **assigned_officer_id** (FK)       | INT                                                   | FK → police_users             |
| **detected_at**                    | TIMESTAMP                                             | Detection time                |
| **updated_at**                     | TIMESTAMP                                             | Last update                   |

**Indexes:**

- PRIMARY KEY (hotspot_id)
- INDEX (location_id)
- INDEX (latitude, longitude)
- INDEX (status, risk_level)

---

# TABLE 12: daily_statistics

## Pre-Aggregated Analytics

**Purpose:** Pre-calculated daily statistics for fast dashboard loading.

| Field                       | Type               | Description                      |
| --------------------------- | ------------------ | -------------------------------- |
| **stat_id** (PK)            | INT AUTO_INCREMENT | Unique stat identifier           |
| **stat_date**               | DATE               | Date of statistics               |
| **location_id** (FK)        | INT                | FK → locations (NULL = national) |
| **total_reports**           | INT DEFAULT 0      | Reports submitted                |
| **verified_reports**        | INT DEFAULT 0      | Reports verified                 |
| **rejected_reports**        | INT DEFAULT 0      | Reports rejected                 |
| **new_cases**               | INT DEFAULT 0      | Cases created                    |
| **resolved_cases**          | INT DEFAULT 0      | Cases resolved                   |
| **active_hotspots**         | INT DEFAULT 0      | Active hotspots                  |
| **avg_ml_trust_score**      | DECIMAL(5,4)       | Average ML trust score           |
| **avg_response_time_hours** | DECIMAL(5,2)       | Avg response time                |
| **ml_accuracy_rate**        | DECIMAL(5,4)       | ML prediction accuracy           |
| **top_incident_types**      | JSON               | Top 5 incident types with counts |
| **hourly_distribution**     | JSON               | Reports per hour (24 values)     |
| **calculated_at**           | TIMESTAMP          | When calculated                  |

**Indexes:**

- PRIMARY KEY (stat_id)
- UNIQUE (stat_date, location_id)
- INDEX (stat_date)
- INDEX (location_id)

---

# TABLE 13: public_safety_zones

## Anonymized Public Safety Map

**Purpose:** Aggregated safety data for public community map. No case-level details exposed.

| Field                  | Type                                       | Description            |
| ---------------------- | ------------------------------------------ | ---------------------- |
| **zone_id** (PK)       | INT AUTO_INCREMENT                         | Unique zone identifier |
| **location_id** (FK)   | INT                                        | FK → locations         |
| **latitude**           | DECIMAL(10,8)                              | Zone center latitude   |
| **longitude**          | DECIMAL(11,8)                              | Zone center longitude  |
| **radius_meters**      | INT DEFAULT 500                            | Zone radius            |
| **safety_level**       | ENUM('safe', 'caution', 'alert', 'danger') | Safety rating          |
| **safety_score**       | DECIMAL(5,2)                               | 0-100 score            |
| **incident_count_7d**  | INT DEFAULT 0                              | Incidents last 7 days  |
| **incident_count_30d** | INT DEFAULT 0                              | Incidents last 30 days |
| **trend**              | ENUM('improving', 'stable', 'worsening')   | Trend direction        |
| **dominant_category**  | VARCHAR(100)                               | Most common category   |
| **peak_risk_hours**    | JSON                                       | Hours with higher risk |
| **tips**               | JSON                                       | Safety tips for area   |
| **last_updated**       | TIMESTAMP                                  | When recalculated      |
| **is_visible**         | BOOLEAN DEFAULT TRUE                       | Show on public map     |

**Indexes:**

- PRIMARY KEY (zone_id)
- INDEX (location_id)
- INDEX (latitude, longitude)
- INDEX (safety_level)

---

# TABLE 14: system_settings

## Application Configuration

**Purpose:** Centralized system configuration including ML settings.

| Field                | Type                                        | Description               |
| -------------------- | ------------------------------------------- | ------------------------- |
| **setting_id** (PK)  | INT AUTO_INCREMENT                          | Unique setting identifier |
| **setting_key**      | VARCHAR(100)                                | Unique setting key        |
| **setting_value**    | TEXT                                        | Setting value             |
| **value_type**       | ENUM('string', 'number', 'boolean', 'json') | Data type                 |
| **category**         | VARCHAR(50)                                 | Setting category          |
| **description**      | TEXT                                        | What setting does         |
| **is_public**        | BOOLEAN DEFAULT FALSE                       | Visible to mobile app     |
| **requires_restart** | BOOLEAN DEFAULT FALSE                       | Needs service restart     |
| **updated_at**       | TIMESTAMP                                   | Last update               |
| **updated_by** (FK)  | INT                                         | FK → police_users         |

**Key Settings:**

| Category     | Key                           | Description                                |
| ------------ | ----------------------------- | ------------------------------------------ |
| verification | `location_accuracy_threshold` | Max GPS accuracy (default: 100m)           |
| verification | `motion_score_threshold`      | Min motion score (default: 30)             |
| verification | `duplicate_radius_meters`     | Duplicate detection radius (default: 200m) |
| verification | `duplicate_time_window_hours` | Duplicate time window (default: 24h)       |
| ml           | `ml_enabled`                  | Enable ML enhancement (default: true)      |
| ml           | `ml_trust_threshold`          | Min ML trust score (default: 0.3)          |
| ml           | `ml_auto_reject_threshold`    | Auto-reject below (default: 0.1)           |
| ml           | `ml_retrain_frequency_days`   | How often to retrain (default: 7)          |

**Indexes:**

- PRIMARY KEY (setting_id)
- UNIQUE (setting_key)
- INDEX (category)

---

# TABLE 15: activity_logs

## Complete Audit Trail

**Purpose:** Records all system activities for security and compliance.

| Field              | Type                                                | Description                     |
| ------------------ | --------------------------------------------------- | ------------------------------- |
| **log_id** (PK)    | BIGINT AUTO_INCREMENT                               | Unique log identifier           |
| **actor_type**     | ENUM('system', 'device', 'police_user', 'ml_model') | Who performed action            |
| **actor_id**       | VARCHAR(50)                                         | device_id, user_id, or model_id |
| **action_type**    | VARCHAR(100)                                        | What was done                   |
| **entity_type**    | VARCHAR(50)                                         | What entity affected            |
| **entity_id**      | VARCHAR(50)                                         | Entity identifier               |
| **action_details** | JSON                                                | Full action context             |
| **ip_address**     | VARCHAR(45)                                         | Client IP                       |
| **user_agent**     | VARCHAR(500)                                        | Client info                     |
| **request_id**     | CHAR(36)                                            | Request trace ID                |
| **success**        | BOOLEAN DEFAULT TRUE                                | Action succeeded                |
| **error_message**  | TEXT                                                | Error if failed                 |
| **created_at**     | TIMESTAMP                                           | When occurred                   |

**Common Action Types:**

| Action Type            | Description                |
| ---------------------- | -------------------------- |
| `report.submitted`     | New report received        |
| `report.verified`      | Report passed verification |
| `case.created`         | New unified case created   |
| `case.merged`          | Reports grouped into case  |
| `ml.prediction`        | ML analyzed a report       |
| `ml.model.activated`   | New ML model activated     |
| `police.login`         | Police user logged in      |
| `police.case.assigned` | Case assigned to officer   |

**Indexes:**

- PRIMARY KEY (log_id)
- INDEX (actor_type, actor_id)
- INDEX (action_type)
- INDEX (entity_type, entity_id)
- INDEX (created_at)

---

## Summary: How Everything Works Together

### 1. Citizen Submits Report

```
Mobile App → incident_reports table
           → 3 Rules Check (location, motion, duplicate)
           → ML Analysis → ml_predictions table
           → If similar reports exist → Link to unified_cases
           → Citizen sees: "Report received, being reviewed"
```

### 2. ML Enhancement Flow

```
┌────────────────────────────────────────────────────────────────┐
│                    ML ENHANCEMENT FLOW                         │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  incident_reports (new report arrives)                         │
│         │                                                      │
│         ▼                                                      │
│  ┌─────────────────────────────────────┐                       │
│  │  3 HARDCODED RULES (Always Run)     │                       │
│  ├─────────────────────────────────────┤                       │
│  │  • Rule 1: Location valid?          │──► location_check_    │
│  │  • Rule 2: Motion score ≥ 30?       │    passed             │
│  │  • Rule 3: Is duplicate?            │──► motion_check_      │
│  └─────────────────────────────────────┘    passed             │
│         │                                                      │
│         ▼                                                      │
│  ┌─────────────────────────────────────┐                       │
│  │  ML MODELS (Enhancement Layer)       │                       │
│  ├─────────────────────────────────────┤                       │
│  │                                     │                       │
│  │  ml_models table contains:          │                       │
│  │  ├── trust_scoring model            │                       │
│  │  ├── anomaly_detection model        │                       │
│  │  ├── clustering model               │                       │
│  │  └── priority_prediction model      │                       │
│  │                                     │                       │
│  │  Models analyze:                    │                       │
│  │  • GPS patterns (spoofing?)         │                       │
│  │  • Motion patterns (bot?)           │                       │
│  │  • Time patterns (suspicious?)      │                       │
│  │  • Historical patterns              │                       │
│  └─────────────────────────────────────┘                       │
│         │                                                      │
│         ▼                                                      │
│  ┌─────────────────────────────────────┐                       │
│  │  ml_predictions table stores:       │                       │
│  ├─────────────────────────────────────┤                       │
│  │  • ml_trust_score (0.0 - 1.0)       │                       │
│  │  • predicted_priority               │                       │
│  │  • anomaly_score                    │                       │
│  │  • anomaly_flags (JSON)             │                       │
│  │  • cluster_confidence               │                       │
│  └─────────────────────────────────────┘                       │
│         │                                                      │
│         ▼                                                      │
│  unified_cases updated with:                                   │
│  • ml_priority_score                                           │
│  • ml_confidence                                               │
│  • Priority ordering for police dashboard                      │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### 3. Police View

```
Police Dashboard sees:
- unified_cases (grouped incidents)
- All linked incident_reports
- All report_evidence (photos, videos)
- ml_predictions (why ML flagged/prioritized)
- Priority ordered by ML scores
```

### 4. Evidence Grouping Example

```
3 citizens report same car accident:

┌─────────────────────────────────────────────────────────────┐
│  incident_reports table:                                    │
│  ├── Report A (device_001, case_id = CASE-001)             │
│  ├── Report B (device_002, case_id = CASE-001)             │
│  └── Report C (device_003, case_id = CASE-001)             │
│                                                             │
│  ml_predictions table:                                      │
│  ├── Prediction for Report A (trust: 0.85)                  │
│  ├── Prediction for Report B (trust: 0.92)                  │
│  └── Prediction for Report C (trust: 0.78)                  │
│                                                             │
│  unified_cases table:                                       │
│  └── CASE-001                                               │
│      ├── report_count: 3                                    │
│      ├── combined_trust_score: 85.0                         │
│      ├── ml_priority_score: 92.0                            │
│      └── ml_confidence: 0.85                                │
│                                                             │
│  report_evidence table:                                     │
│  ├── Photo from Report A (case_id = CASE-001)              │
│  ├── Video from Report B (case_id = CASE-001)              │
│  └── Photo from Report C (case_id = CASE-001)              │
└─────────────────────────────────────────────────────────────┘

CITIZEN VIEW:
├── Device 001 sees: "Your report is being reviewed"
├── Device 002 sees: "Your report is being reviewed"
└── Device 003 sees: "Your report is being reviewed"

POLICE VIEW:
└── One case "CASE-001" with:
    ├── 3 linked reports
    ├── 3 pieces of evidence
    ├── ML trust score: 85%
    └── Priority: HIGH (ML predicted)
```

---

## Data Privacy Summary

| What Citizens Know          | What Police Know           |
| --------------------------- | -------------------------- |
| "Report submitted"          | Full ML analysis           |
| "Being reviewed"            | All grouped reports        |
| "Action taken"              | Trust scores               |
| Their own report only       | All evidence combined      |
| Never know about grouping   | Case investigation history |
| Never know about ML scoring | ML prediction details      |

---

## Relationship Summary

- One **device** → many **reports**
- One **report** → many **evidence files**
- One **report** → one **ML prediction**
- One **unified_case** → many **reports** (grouped)
- One **hotspot** → many **unified_cases** (different crime types in same area)
- One **police_user** → many **assigned cases**
- One **location** → many **child locations** (hierarchical)
- One **incident_type** → many **sub-types** (hierarchical)

---

## ML Hotspot & Clustering Logic

### Hotspots Are NOT Limited to One Crime Type

A **hotspot** represents a geographic area with high incident activity. It can contain:

- **Multiple different crime types** in the same area
- Example: A market area might have theft, vandalism, AND assault incidents

### What ML Analyzes for Hotspots

```
┌─────────────────────────────────────────────────────────────────┐
│                    HOTSPOT DETECTION                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ML clusters incidents by LOCATION, not by crime type:          │
│                                                                 │
│  Example Hotspot: "Nyabugogo Market Area"                       │
│  ├── 15 Theft incidents (45%)                                   │
│  ├── 8 Vandalism incidents (24%)                                │
│  ├── 6 Assault incidents (18%)                                  │
│  └── 4 Other incidents (13%)                                    │
│                                                                 │
│  Hotspot stores:                                                │
│  • dominant_incident_type_id = Theft (most common)              │
│  • But ALL crime types are tracked in the area                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Dashboard Analytics Display

The police dashboard shows comprehensive crime analytics:

```
┌─────────────────────────────────────────────────────────────────┐
│                 POLICE DASHBOARD ANALYTICS                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  📊 CRIME DISTRIBUTION BY REGION                                │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Kigali City:                                            │   │
│  │   • Theft: 234 cases (most common)                      │   │
│  │   • Vandalism: 89 cases                                 │   │
│  │   • Assault: 45 cases                                   │   │
│  │   • Fraud: 23 cases                                     │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  🔥 HOTSPOT AREAS (Multiple Crime Types)                        │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ 1. Nyabugogo Market - 33 incidents (mixed types)        │   │
│  │ 2. Kimironko Area - 28 incidents (mixed types)          │   │
│  │ 3. Downtown Kigali - 21 incidents (mixed types)         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  📈 INSIGHTS:                                                   │
│  • "Theft happens more in market regions"                      │
│  • "Assault peaks on weekend nights"                           │
│  • "Vandalism concentrated in urban areas"                     │
│  • "This area has 3 different crime types active"              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### daily_statistics Table Supports This

The `daily_statistics` table stores:

- `top_incident_types` (JSON) - Top 5 crime types with counts per region
- `hourly_distribution` (JSON) - When each crime type peaks

This enables dashboard insights like:

- "Theft incidents happen 40% more in this region than others"
- "This hotspot has 5 different crime types reported"
- "Assault cases peak between 10 PM - 2 AM"

---

## Table Count: 15 Tables

| Category      | Tables                                                                               | Count  |
| ------------- | ------------------------------------------------------------------------------------ | ------ |
| **Core**      | devices, locations, incident_types, unified_cases, incident_reports, report_evidence | 6      |
| **ML**        | ml_models, ml_predictions                                                            | 2      |
| **Police**    | police_users, notifications                                                          | 2      |
| **Analytics** | hotspots, daily_statistics                                                           | 2      |
| **Public**    | public_safety_zones                                                                  | 1      |
| **System**    | system_settings, activity_logs                                                       | 2      |
| **TOTAL**     |                                                                                      | **15** |
