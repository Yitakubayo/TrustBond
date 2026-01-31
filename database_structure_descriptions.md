# DATABASE STRUCTURE

## **1. devices — Anonymous Reporter Devices**

| Column             | Type         | Description                         |
| ------------------ | ------------ | ----------------------------------- |
| device_id          | UUID (PK)    | Internal unique device identifier   |
| device_hash        | VARCHAR(255) | Anonymous hashed device fingerprint |
| first_seen_at      | TIMESTAMP    | First report submission time        |
| total_reports      | INT          | Total submitted reports             |
| trusted_reports    | INT          | Confirmed valid reports             |
| flagged_reports    | INT          | Rejected or suspicious reports      |
| device_trust_score | DECIMAL(5,2) | Calculated trust score (0–100)     |

## **2. reports — Incident Reports (Main Table)**

| Column         | Type                                | Description                  |
| -------------- | ----------------------------------- | ---------------------------- |
| report_id      | UUID (PK)                           | Unique report identifier     |
| device_id      | UUID (FK → devices.device_id)      | Reporting device             |
| incident_type  | VARCHAR(100)                        | Incident category            |
| description    | TEXT                                | Incident description         |
| latitude       | DECIMAL(10,7)                       | Incident latitude            |
| longitude      | DECIMAL(10,7)                       | Incident longitude           |
| gps_accuracy   | DECIMAL(6,2)                        | GPS accuracy (meters)        |
| motion_level   | ENUM('low','medium','high')         | Motion intensity             |
| movement_speed | DECIMAL(6,2)                        | Device speed                 |
| was_stationary | BOOLEAN                             | Stationary flag              |
| reported_at    | TIMESTAMP                           | Submission time              |
| rule_status    | ENUM('passed','flagged','rejected') | Rule-based validation result |
| is_flagged     | BOOLEAN                             | Suspicious indicator         |

## **3. evidence_files — Report Evidence**

| Column          | Type                           | Description            |
| --------------- | ------------------------------ | ---------------------- |
| evidence_id     | UUID (PK)                      | Evidence ID            |
| report_id       | UUID (FK → reports.report_id) | Related report         |
| file_url        | VARCHAR(500)                   | Storage path           |
| file_type       | ENUM('photo','video')          | Media type             |
| media_latitude  | DECIMAL(10,7)                  | Media GPS latitude     |
| media_longitude | DECIMAL(10,7)                  | Media GPS longitude    |
| captured_at     | TIMESTAMP                      | Media capture time     |
| is_live_capture | BOOLEAN                        | Live capture indicator |
| uploaded_at     | TIMESTAMP                      | Upload time            |

## **4. ml_predictions — Machine Learning Outputs**

**Purpose**

Stores ML model predictions only (never treated as truth).

| Column           | Type                                    | Description         |
| ---------------- | --------------------------------------- | ------------------- |
| prediction_id    | UUID (PK)                               | Prediction ID       |
| report_id        | UUID (FK → reports.report_id)          | Evaluated report    |
| trust_score      | DECIMAL(5,2)                            | ML confidence score |
| prediction_label | ENUM('likely_real','suspicious','fake') | ML label            |
| model_version    | VARCHAR(50)                             | Model version       |
| evaluated_at     | TIMESTAMP                               | Evaluation time     |

## **5. police_users — Police Accounts & Roles**

**Purpose**

Stores authenticated police users who review reports.

| Column             | Type                                 | Description           |
| ------------------ | ------------------------------------ | --------------------- |
| police_user_id     | INT (PK)                             | Unique police user ID |
| username           | VARCHAR(50)                          | Login username        |
| password_hash      | VARCHAR(255)                         | Hashed password       |
| full_name          | VARCHAR(150)                         | Officer name          |
| badge_number       | VARCHAR(50)                          | Police badge          |
| role               | ENUM('admin','supervisor','officer') | User role             |
| assigned_sector_id | SMALLINT (FK → sectors.sector_id)   | Assigned sector       |
| assigned_unit      | VARCHAR(100)                         | Police unit           |
| is_active          | BOOLEAN                              | Account status        |
| created_at         | TIMESTAMP                            | Creation time         |
| last_login_at      | TIMESTAMP                            | Last login            |

## **6. police_reviews — Ground Truth Decisions**

| Column         | Type                                         | Description       |
| -------------- | -------------------------------------------- | ----------------- |
| review_id      | UUID (PK)                                    | Review ID         |
| report_id      | UUID (FK → reports.report_id)               | Reviewed report   |
| police_user_id | INT (FK → police_users.police_user_id)      | Reviewing officer |
| decision       | ENUM('confirmed','rejected','investigation') | Police decision   |
| review_note    | TEXT                                         | Officer notes     |
| unit_code      | VARCHAR(50)                                  | Police unit code  |
| reviewed_at    | TIMESTAMP                                    | Review time       |

## GIS & ADMINISTRATIVE LOCATION (Musanze Only)

## **7. sectors**

| Column      | Type          | Description      |
| ----------- | ------------- | ---------------- |
| sector_id   | SMALLINT (PK) | Sector ID        |
| sector_name | VARCHAR(100)  | Sector name      |
| geometry    | GEOMETRY      | Polygon boundary |

---

## **8. cells**

| Column    | Type                               | Description      |
| --------- | ---------------------------------- | ---------------- |
| cell_id   | SMALLINT (PK)                      | Cell ID          |
| sector_id | SMALLINT (FK → sectors.sector_id) | Parent sector    |
| cell_name | VARCHAR(100)                       | Cell name        |
| geometry  | GEOMETRY                           | Polygon boundary |

---

## **9. villages**

| Column       | Type                           | Description      |
| ------------ | ------------------------------ | ---------------- |
| village_id   | SMALLINT (PK)                  | Village ID       |
| cell_id      | SMALLINT (FK → cells.cell_id) | Parent cell      |
| village_name | VARCHAR(100)                   | Village name     |
| geometry     | GEOMETRY                       | Polygon boundary |

✔ User never selects these

✔ Derived from GPS → polygon lookup

## HOTSPOT & ANALYTICS TABLES

## **10. hotspots — Risk Clusters**

| Column            | Type                        | Description         |
| ----------------- | --------------------------- | ------------------- |
| hotspot_id        | INT (PK)                    | Hotspot ID          |
| center_lat        | DECIMAL(10,7)               | Center latitude     |
| center_long       | DECIMAL(10,7)               | Center longitude    |
| radius_meters     | DECIMAL(8,2)                | Cluster radius      |
| incident_count    | INT                         | Report count        |
| risk_level        | ENUM('low','medium','high') | Risk classification |
| time_window_hours | INT                         | Time window         |
| detected_at       | TIMESTAMP                   | Detection time      |

## **11. hotspot_reports — Report–Hotspot Bridge**

| Column                                        | Type                            | Description   |
| --------------------------------------------- | ------------------------------- | ------------- |
| hotspot_id                                    | INT (FK → hotspots.hotspot_id) | Hotspot       |
| report_id                                     | UUID (FK → reports.report_id)  | Report        |
| **PRIMARY KEY (hotspot_id, report_id)** |                                 | Composite key |

## **12. incident_groups — Duplicate Incident Grouping**

| Column        | Type          | Description       |
| ------------- | ------------- | ----------------- |
| group_id      | UUID (PK)     | Group ID          |
| incident_type | VARCHAR(100)  | Incident category |
| center_lat    | DECIMAL(10,7) | Group latitude    |
| center_long   | DECIMAL(10,7) | Group longitude   |
| start_time    | TIMESTAMP     | Earliest report   |
| end_time      | TIMESTAMP     | Latest report     |
| report_count  | INT           | Reports in group  |
| created_at    | TIMESTAMP     | Creation time     |

## ADMIN & SECURITY

## **13. audit_logs**

**Purpose**

Records administrative, police, and critical system actions.

| Column         | Type                                    | Description      |
| -------------- | --------------------------------------- | ---------------- |
| log_id         | BIGINT AUTO_INCREMENT (PK)              | Audit ID         |
| actor_type     | ENUM('system','police_user')            | Action source    |
| actor_id       | INT (FK → police_users.police_user_id) | Actor (nullable) |
| action_type    | VARCHAR(100)                            | Action performed |
| entity_type    | VARCHAR(50)                             | Affected table   |
| entity_id      | CHAR(36)                                | Affected record  |
| action_details | JSON                                    | Extra context    |
| ip_address     | VARCHAR(45)                             | IP address       |
| success        | BOOLEAN DEFAULT TRUE                    | Result           |
| created_at     | TIMESTAMP DEFAULT CURRENT_TIMESTAMP     | Time             |


I think we should add notification table.

i think we should not create a table for storing data for retraining the model coz we already have those data some where in the above tables, that would be duplication, we should extract the exact data for improving our dataset for re-training the model to impove it without using the new table, please provide your comment if you think it's crucial to still use that table.

for incident type we can set those types in the codes instead of creating the table then in report table we have attribute which will store that type already, that is what i think, thank you.

please your comment here?

I'm also still working to see i can merge sector,cell,.. into one location tables.

good night.
