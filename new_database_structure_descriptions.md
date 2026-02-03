# FULL && COMPLETE DATABASE SCHEMA

## 1.devices — Anonymous Reporter Devices

| Column             | Type         | Description                  |
| ------------------ | ------------ | ---------------------------- |
| device_id          | UUID (PK)    | Internal device identifier   |
| device_hash        | VARCHAR(255) | Hashed anonymous fingerprint |
| first_seen_at      | TIMESTAMP    | First report time            |
| total_reports      | INT          | Total submitted reports      |
| trusted_reports    | INT          | Confirmed valid reports      |
| flagged_reports    | INT          | Suspicious/rejected reports  |
| device_trust_score | DECIMAL(5,2) | Reliability score (0–100)   |

## 2.incident_types — Incident Categories

| Column           | Type          | Description          |
| ---------------- | ------------- | -------------------- |
| incident_type_id | SMALLINT (PK) | Type ID              |
| type_name        | VARCHAR(100)  | Theft, Assault, etc. |
| description      | TEXT          | Type explanation     |
| severity_weight  | DECIMAL(3,2)  | Risk weight          |
| is_active        | BOOLEAN       | Availability         |
| created_at       | TIMESTAMP     | Creation time        |

## 3. reports — Incident Reports (Raw Submissions)

| Column              | Type                                | Description       |
| ------------------- | ----------------------------------- | ----------------- |
| report_id           | UUID (PK)                           | Report ID         |
| device_id           | UUID (FK → devices)                | Reporting device  |
| incident_type_id    | SMALLINT (FK → incident_types)     | Category          |
| description         | TEXT                                | Incident details  |
| latitude            | DECIMAL(10,7)                       | GPS latitude      |
| longitude           | DECIMAL(10,7)                       | GPS longitude     |
| gps_accuracy        | DECIMAL(6,2)                        | Accuracy (meters) |
| motion_level        | ENUM('low','medium','high')         | Movement level    |
| movement_speed      | DECIMAL(6,2)                        | Speed             |
| was_stationary      | BOOLEAN                             | Stationary flag   |
| village_location_id | INT (FK → locations)               | Derived village   |
| reported_at         | TIMESTAMP                           | Submission time   |
| rule_status         | ENUM('passed','flagged','rejected') | Rule check        |
| is_flagged          | BOOLEAN                             | Suspicious marker |
| feature_vector      | JSONB                               | Precomputed ML features      |
| ai_ready            | BOOLEAN                            | Indicates features extracted |
| features_extracted  | TIMESTAMP                           | Feature generation time      |


## 4. evidence_files — Report Evidence

| Column          | Type                  | Description   |
| --------------- | --------------------- | ------------- |
| evidence_id     | UUID (PK)             | Evidence ID   |
| report_id       | UUID (FK → reports)  | Parent report |
| file_url        | VARCHAR(500)          | Storage path  |
| file_type       | ENUM('photo','video') | Media type    |
| media_latitude  | DECIMAL(10,7)         | Media GPS     |
| media_longitude | DECIMAL(10,7)         | Media GPS     |
| captured_at     | TIMESTAMP             | Capture time  |
| is_live_capture | BOOLEAN               | Live capture  |
| uploaded_at     | TIMESTAMP             | Upload time   |
| perceptual_hash  | VARCHAR(128)                     | duplicate detection |
| blur_score       | DECIMAL(6,3)                     | image clarity       |
| tamper_score     | DECIMAL(6,3)                     | manipulation risk   |
| ai_quality_label | ENUM('good','poor','suspicious') | AI decision         |
| ai_checked_at    | TIMESTAMP                        | analysis time       |


## 5. ml_predictions — Machine Learning Outputs (NOT Truth)

| Column           | Type                                    | Description      |
| ---------------- | --------------------------------------- | ---------------- |
| prediction_id    | UUID (PK)                               | Prediction ID    |
| report_id        | UUID (FK → reports)                    | Evaluated report |
| trust_score      | DECIMAL(5,2)                            | ML confidence    |
| prediction_label | ENUM('likely_real','suspicious','fake') | Prediction       |
| model_version    | VARCHAR(50)                             | Model version    |
| evaluated_at     | TIMESTAMP                               | Evaluation time  |
| confidence       | DECIMAL(5,2)                            | Probability score                |
| explanation      | JSONB                                   | Feature importance / SHAP values |
| processing_time  | INT                                     | Performance monitoring           |
| model_type       | VARCHAR(50)                             | random_forest / anomaly / vision |
| is_final         | BOOLEAN                                  | Marks production decision        |


## 6. police_users — Police Accounts & Roles

| Column Name          | Data Type                            | Description / Notes              |
| -------------------- | ------------------------------------ | -------------------------------- |
| police_user_id       | INT (PK)                             | Unique user ID                   |
| first_name           | VARCHAR(150)                         | Officer first name               |
| middle_name          | VARCHAR(150)                         | Officer middle name (optional)   |
| last_name            | VARCHAR(150)                         | Officer last name                |
| email                | VARCHAR(255)                         | Officer email address            |
| phone_number         | VARCHAR(20)                          | Officer phone number             |
| password_hash        | VARCHAR(255)                         | Hashed password                  |
| badge_number         | VARCHAR(50)                          | Badge number                     |
| role                 | ENUM('admin','supervisor','officer') | User role                        |
| assigned_location_id | INT (FK → locations)                | Jurisdiction / assigned location |
| is_active            | BOOLEAN                              | Status: active/inactive          |
| created_at           | TIMESTAMP                            | Account creation date            |
| last_login_at        | TIMESTAMP                            | Last login date/time             |

## 7. police_reviews — Ground Truth Decisions

| Column         | Type                                         | Description     |
| -------------- | -------------------------------------------- | --------------- |
| review_id      | UUID (PK)                                    | Review ID       |
| report_id      | UUID (FK → reports)                         | Reviewed report |
| police_user_id | INT (FK → police_users)                     | Officer         |
| decision       | ENUM('confirmed','rejected','investigation') | Verdict         |
| review_note    | TEXT                                         | Notes           |
| reviewed_at    | TIMESTAMP                                    | Review time     |
| ground_truth_label| ENUM('real','fake')                        | training label       |
| confidence_level  | DECIMAL(5,2)                            | officer certainty    |
| used_for_training | BOOLEAN                                   | prevent data leakage |


## 8. locations — Administrative Boundaries (Musanze)

| Column             | Type                            | Description |
| ------------------ | ------------------------------- | ----------- |
| location_id        | INT (PK)                        | Location ID |
| location_type      | ENUM('sector','cell','village') | Level       |
| location_name      | VARCHAR(100)                    | Name        |
| parent_location_id | INT (FK → locations)           | Hierarchy   |
| geometry           | GEOMETRY                        | Polygon     |
| centroid_lat       | DECIMAL(10,7)                   | Center      |
| centroid_long      | DECIMAL(10,7)                   | Center      |
| is_active          | BOOLEAN                         | Status      |

## 9. hotspots — Risk Clusters

| Column            | Type                        | Description    |
| ----------------- | --------------------------- | -------------- |
| hotspot_id        | INT (PK)                    | Hotspot ID     |
| center_lat        | DECIMAL(10,7)               | Center         |
| center_long       | DECIMAL(10,7)               | Center         |
| radius_meters     | DECIMAL(8,2)                | Radius         |
| incident_count    | INT                         | Reports        |
| risk_level        | ENUM('low','medium','high') | Risk           |
| time_window_hours | INT                         | Time window    |
| detected_at       | TIMESTAMP                   | Detection time |

## 10. hotspot_reports — Hotspot Membership

| Column                                        | Type                 |
| --------------------------------------------- | -------------------- |
| hotspot_id                                    | INT (FK → hotspots) |
| report_id                                     | UUID (FK → reports) |
| **PRIMARY KEY (hotspot_id, report_id)** |                      |

## 11. incident_groups — Duplicate Incident Grouping

| Column           | Type                            | Description |
| ---------------- | ------------------------------- | ----------- |
| group_id         | UUID (PK)                       | Group ID    |
| incident_type_id | SMALLINT (FK → incident_types) | Category    |
| center_lat       | DECIMAL(10,7)                   | Center      |
| center_long      | DECIMAL(10,7)                   | Center      |
| start_time       | TIMESTAMP                       | Earliest    |
| end_time         | TIMESTAMP                       | Latest      |
| report_count     | INT                             | Reports     |
| created_at       | TIMESTAMP                       | Created     |

## 12. report_assignments — Case Handling Workflow

| Column         | Type                                                 | Description |
| -------------- | ---------------------------------------------------- | ----------- |
| assignment_id  | UUID (PK)                                            | Assignment  |
| report_id      | UUID (FK → reports)                                 | Report      |
| police_user_id | INT (FK → police_users)                             | Officer     |
| status         | ENUM('assigned','investigating','resolved','closed') | Status      |
| priority       | ENUM('low','medium','high','urgent')                 | Priority    |
| assigned_at    | TIMESTAMP                                            | Assigned    |
| completed_at   | TIMESTAMP                                            | Completed   |

## 13. notifications — System Alerts

| Column              | Type                                           | Description  |
| ------------------- | ---------------------------------------------- | ------------ |
| notification_id     | UUID (PK)                                      | Notification |
| police_user_id      | INT (FK → police_users)                       | Recipient    |
| title               | VARCHAR(150)                                   | Title        |
| message             | TEXT                                           | Message      |
| type                | ENUM('report','hotspot','assignment','system') | Category     |
| related_entity_type | VARCHAR(50)                                    | Table        |
| related_entity_id   | CHAR(36)                                       | Record       |
| is_read             | BOOLEAN                                        | Read         |
| created_at          | TIMESTAMP                                      | Created      |

## 14. audit_logs — Security & Accountability

| Column         | Type                         | Description |
| -------------- | ---------------------------- | ----------- |
| log_id         | BIGINT (PK)                  | Log ID      |
| actor_type     | ENUM('system','police_user') | Actor       |
| actor_id       | INT                          | User        |
| action_type    | VARCHAR(100)                 | Action      |
| entity_type    | VARCHAR(50)                  | Table       |
| entity_id      | CHAR(36)                     | Record      |
| action_details | JSON                         | Context     |
| ip_address     | VARCHAR(45)                  | IP          |
| success        | BOOLEAN                      | Status      |
| created_at     | TIMESTAMP                    | Time        |
