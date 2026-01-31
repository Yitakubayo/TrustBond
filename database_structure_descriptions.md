DATABASE STRUCTURE

**Anonymous Crime Reporting & Hotspot Detection System (Musanze District)**

## CORE REPORTING & TRUST TABLES

## 1. devices — Anonymous Reporter Devices

**Purpose:**
Stores anonymous device behavior and long-term trust without identifying the user.

| Column             | Type         | Description                         |
| ------------------ | ------------ | ----------------------------------- |
| device_id          | UUID (PK)    | Internal unique device identifier   |
| device_hash        | VARCHAR(255) | Anonymous hashed device fingerprint |
| first_seen_at      | TIMESTAMP    | First report submission time        |
| total_reports      | INT          | Total submitted reports             |
| trusted_reports    | INT          | Confirmed valid reports             |
| flagged_reports    | INT          | Rejected or suspicious reports      |
| device_trust_score | DECIMAL(5,2) | Calculated trust score (0–100)     |

## 2. reports — Incident Reports (Main Table)

**Purpose:**
Stores each incident submission (raw, unverified).

| Column         | Type                 | Description                 |
| -------------- | -------------------- | --------------------------- |
| report_id      | UUID (PK)            | Unique report ID            |
| device_id      | UUID (FK → devices) | Reporting device            |
| incident_type  | VARCHAR(100)         | Type of incident            |
| description    | TEXT                 | Incident description        |
| latitude       | DECIMAL(10,7)        | Incident latitude           |
| longitude      | DECIMAL(10,7)        | Incident longitude          |
| gps_accuracy   | DECIMAL(6,2)         | GPS accuracy (meters)       |
| motion_level   | ENUM                 | low / medium / high         |
| movement_speed | DECIMAL(6,2)         | Device speed                |
| was_stationary | BOOLEAN              | Device stationary flag      |
| reported_at    | TIMESTAMP            | Submission time             |
| rule_status    | ENUM                 | passed / flagged / rejected |
| is_flagged     | BOOLEAN              | Suspicious indicator        |

## 3. evidence_files — Report Evidence

**Purpose:**
Stores evidence metadata linked to reports.

| Column          | Type                 | Description         |
| --------------- | -------------------- | ------------------- |
| evidence_id     | UUID (PK)            | Evidence ID         |
| report_id       | UUID (FK → reports) | Related report      |
| file_url        | VARCHAR(500)         | Storage path        |
| file_type       | ENUM                 | photo / video       |
| media_latitude  | DECIMAL(10,7)        | Media GPS latitude  |
| media_longitude | DECIMAL(10,7)        | Media GPS longitude |
| captured_at     | TIMESTAMP            | Media capture time  |
| is_live_capture | BOOLEAN              | Live capture flag   |
| uploaded_at     | TIMESTAMP            | Upload time         |

## 4. ml_predictions — Machine Learning Outputs

**Purpose:**
Stores ML predictions only (not truth).

| Column           | Type                 | Description                     |
| ---------------- | -------------------- | ------------------------------- |
| prediction_id    | UUID (PK)            | Prediction ID                   |
| report_id        | UUID (FK → reports) | Evaluated report                |
| trust_score      | DECIMAL(5,2)         | ML confidence score             |
| prediction_label | ENUM                 | likely_real / suspicious / fake |
| model_version    | VARCHAR(50)          | Model version                   |
| evaluated_at     | TIMESTAMP            | Evaluation time                 |

## 5. police_users — Police Accounts & Roles

**Purpose:**
Stores police officers who authenticate to review and confirm reports.

| Column             | Type                                 | Description           |
| ------------------ | ------------------------------------ | --------------------- |
| police_user_id     | INT (PK)                             | Unique police user ID |
| username           | VARCHAR(50)                          | Login username        |
| password_hash      | VARCHAR(255)                         | Hashed password       |
| full_name          | VARCHAR(150)                         | Officer full name     |
| badge_number       | VARCHAR(50)                          | Police badge number   |
| role               | ENUM('admin','supervisor','officer') | System role           |
| assigned_sector_id | SMALLINT (FK → sectors)             | Assigned sector       |
| assigned_unit      | VARCHAR(100)                         | Police station / unit |
| is_active          | BOOLEAN                              | Account status        |
| created_at         | TIMESTAMP                            | Account creation time |
| last_login_at      | TIMESTAMP                            | Last login time       |

## 6. police_reviews — Ground Truth Decisions

**Purpose:**
Official confirmation used for accountability and ML training.

| Column         | Type                     | Description                          |
| -------------- | ------------------------ | ------------------------------------ |
| review_id      | UUID (PK)                | Review ID                            |
| report_id      | UUID (FK → reports)     | Reviewed report                      |
| police_user_id | INT (FK → police_users) | Reviewing officer                    |
| decision       | ENUM                     | confirmed / rejected / investigation |
| review_note    | TEXT                     | Officer notes                        |
| unit_code      | VARCHAR(50)              | Police unit identifier               |
| reviewed_at    | TIMESTAMP                | Review time                          |

📌 **This is the ONLY place police users link to reports**
✔ Clean accountability
✔ ML ground truth
✔ No anonymity violation

## GIS & ADMINISTRATIVE LOCATION (Musanze-Only)

*(Inserted once from NISR shapefiles — derived via spatial queries)*

## 7. sectors

| Column      | Type          | Description      |
| ----------- | ------------- | ---------------- |
| sector_id   | SMALLINT (PK) | Sector ID        |
| sector_name | VARCHAR(100)  | Sector name      |
| geometry    | GEOMETRY      | Polygon boundary |

## 8. cells

| Column    | Type                     | Description   |
| --------- | ------------------------ | ------------- |
| cell_id   | SMALLINT (PK)            | Cell ID       |
| sector_id | SMALLINT (FK → sectors) | Parent sector |
| cell_name | VARCHAR(100)             | Cell name     |
| geometry  | GEOMETRY                 | Cell polygon  |

## 9. villages

| Column       | Type                   | Description     |
| ------------ | ---------------------- | --------------- |
| village_id   | SMALLINT (PK)          | Village ID      |
| cell_id      | SMALLINT (FK → cells) | Parent cell     |
| village_name | VARCHAR(100)           | Village name    |
| geometry     | GEOMETRY               | Village polygon |

## HOTSPOT & ANALYTICS TABLES

## 10. hotspots — Risk Clusters

**Purpose:**
Represents clustered high-risk locations.

| Column            | Type          | Description              |
| ----------------- | ------------- | ------------------------ |
| hotspot_id        | INT (PK)      | Hotspot ID               |
| center_lat        | DECIMAL(10,7) | Cluster center latitude  |
| center_long       | DECIMAL(10,7) | Cluster center longitude |
| radius_meters     | DECIMAL(8,2)  | Cluster radius           |
| incident_count    | INT           | Reports count            |
| risk_level        | ENUM          | low / medium / high      |
| time_window_hours | INT           | Time window              |
| detected_at       | TIMESTAMP     | Detection time           |

## 11. hotspot_reports — Report–Hotspot Bridge

| Column                                        | Type                 | Description |
| --------------------------------------------- | -------------------- | ----------- |
| hotspot_id                                    | INT (FK → hotspots) | Hotspot     |
| report_id                                     | UUID (FK → reports) | Report      |
| **PRIMARY KEY (hotspot_id, report_id)** |                      |             |

## 12. incident_groups — Duplicate Incident Grouping

**Purpose:**
Groups multiple reports of the same incident.

| Column        | Type          | Description            |
| ------------- | ------------- | ---------------------- |
| group_id      | UUID (PK)     | Incident group ID      |
| incident_type | VARCHAR(100)  | Incident category      |
| center_lat    | DECIMAL(10,7) | Group center latitude  |
| center_long   | DECIMAL(10,7) | Group center longitude |
| start_time    | TIMESTAMP     | Earliest report        |
| end_time      | TIMESTAMP     | Latest report          |
| report_count  | INT           | Reports in group       |
| created_at    | TIMESTAMP     | Group creation time    |

