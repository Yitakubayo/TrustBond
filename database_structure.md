# TrustBond Database Design

## Complete Database Documentation

This provides a **complete overview database structure** for the TrustBond Privacy-Preserving Anonymous Community Incident Reporting System.

### System Features Supported:

- Anonymous incident reporting (privacy-preserving)
- Rwanda administrative geography (Provinces -> Districts -> Sectors ->  Cells -> Villages)
- Incident taxonomy (Categories and Types)
- Machine learning trust scoring (hybrid verification)
- Rule-based verification engine
- Device motion verification
- Evidence file uploads (photos, videos, audio)
- Police dashboard and authentication
- Hotspot detection (DBSCAN clustering)
- Real-time notifications
- Analytics and statistics
- Public community safety map
- Complete audit trail
- API management for integrations

---

# SECTION 1: CORE DEVICE MANAGEMENT

---

## 1.1 devices — Anonymous Reporter Devices

**Meaning:**
Stores anonymous device behavior and trust history. No names, phone numbers, or personal data are stored.

**Purpose:**
Tracks how reliable a reporting device is over time using pseudonymous identification.

| Field                    | Type                   | Description                                                                  |
| ------------------------ | ---------------------- | ---------------------------------------------------------------------------- |
| **device_id** (PK)       | CHAR(36)               | UUID - Unique identifier for each device                                     |
| **device_fingerprint**   | VARCHAR(255)           | SHA-256 hash of hardware characteristics - non-reversible, ensures anonymity |
| **platform**             | ENUM('android', 'ios') | Mobile platform type                                                         |
| **app_version**          | VARCHAR(20)            | Version of TrustBond app installed                                           |
| **os_version**           | VARCHAR(30)            | Operating system version                                                     |
| **device_language**      | VARCHAR(10)            | Preferred language (en=English)                                              |
| **current_trust_score**  | DECIMAL(5,2)           | Current trust score (0-100). New devices start at 50                         |
| **total_reports**        | INT                    | Total reports submitted by this device                                       |
| **trusted_reports**      | INT                    | Reports confirmed as valid                                                   |
| **suspicious_reports**   | INT                    | Reports marked as suspicious                                                 |
| **false_reports**        | INT                    | Reports confirmed as false                                                   |
| **is_blocked**           | BOOLEAN                | Whether device is blocked from reporting                                     |
| **block_reason**         | TEXT                   | Reason for blocking                                                          |
| **blocked_at**           | TIMESTAMP              | When device was blocked                                                      |
| **blocked_by**           | INT                    | Police user who blocked the device                                           |
| **push_token_encrypted** | VARCHAR(500)           | Encrypted FCM/APNs token for notifications                                   |
| **registered_at**        | TIMESTAMP              | First report submission time                                                 |
| **last_active_at**       | TIMESTAMP              | Last activity timestamp                                                      |
| **last_report_at**       | TIMESTAMP              | Last report submission time                                                  |

---

## 1.2 device_trust_history — Device Trust Score History

**Meaning:**
Historical record of device trust score changes over time.

**Purpose:**
Enables trend analysis and audit trail of trust score evolution.

| Field                        | Type         | Description                                                       |
| ---------------------------- | ------------ | ----------------------------------------------------------------- |
| **history_id** (PK)          | INT          | Unique identifier for history record                              |
| **device_id** (FK → devices) | CHAR(36)     | Device being tracked                                              |
| **trust_score**              | DECIMAL(5,2) | Trust score at this point                                         |
| **total_reports**            | INT          | Total reports at snapshot time                                    |
| **trusted_reports**          | INT          | Trusted reports at snapshot                                       |
| **suspicious_reports**       | INT          | Suspicious reports at snapshot                                    |
| **false_reports**            | INT          | False reports at snapshot                                         |
| **score_change**             | DECIMAL(5,2) | Change from previous snapshot                                     |
| **change_reason**            | VARCHAR(100) | Reason for change (report_verified, report_rejected, decay, etc.) |
| **calculated_at**            | TIMESTAMP    | When snapshot was taken                                           |

---

# SECTION 2: RWANDA ADMINISTRATIVE GEOGRAPHY

---

## 2.1 provinces — Rwanda Provinces

**Meaning:**
Top-level administrative divisions of Rwanda (5 provinces).

**Purpose:**
Hierarchical location reference for incident mapping.

| Field                  | Type          | Description                          |
| ---------------------- | ------------- | ------------------------------------ |
| **province_id** (PK)   | TINYINT       | Unique province identifier           |
| **province_name**      | VARCHAR(50)   | Province name                        |
| **province_code**      | CHAR(2)       | Two-letter province code             |
| **boundary_geojson**   | JSON          | Province boundary as GeoJSON polygon |
| **centroid_latitude**  | DECIMAL(10,8) | Center point latitude                |
| **centroid_longitude** | DECIMAL(11,8) | Center point longitude               |
| **population**         | INT           | Province population                  |
| **area_sq_km**         | DECIMAL(10,2) | Area in square kilometers            |
| **district_count**     | TINYINT       | Number of districts                  |
| **created_at**         | TIMESTAMP     | Record creation time                 |

---

## 2.2 districts — Rwanda Districts

**Meaning:**
District-level administration (30 districts in Rwanda).

**Purpose:**
Primary geographic unit for incident reporting and police jurisdiction.

| Field                            | Type          | Description                       |
| -------------------------------- | ------------- | --------------------------------- |
| **district_id** (PK)             | TINYINT       | Unique district identifier        |
| **province_id** (FK → provinces) | TINYINT       | Parent province                   |
| **district_name**                | VARCHAR(100)  | District name                     |
| **district_code**                | VARCHAR(10)   | Unique district code              |
| **boundary_geojson**             | JSON          | District boundary as GeoJSON      |
| **centroid_latitude**            | DECIMAL(10,8) | Center point latitude             |
| **centroid_longitude**           | DECIMAL(11,8) | Center point longitude            |
| **population**                   | INT           | District population               |
| **area_sq_km**                   | DECIMAL(10,2) | Area in square kilometers         |
| **sector_count**                 | TINYINT       | Number of sectors                 |
| **is_pilot_area**                | BOOLEAN       | TRUE for Musanze district (pilot) |
| **pilot_start_date**             | DATE          | When pilot program started        |
| **pilot_end_date**               | DATE          | When pilot program ends           |
| **is_active**                    | BOOLEAN       | Whether district is active        |
| **created_at**                   | TIMESTAMP     | Record creation time              |
| **updated_at**                   | TIMESTAMP     | Last update time                  |

---

## 2.3 sectors — Sectors within Districts

**Meaning:**
Sector-level administration within districts.

**Purpose:**
Finer geographic granularity for incident location and police assignment.

| Field                            | Type          | Description                |
| -------------------------------- | ------------- | -------------------------- |
| **sector_id** (PK)               | SMALLINT      | Unique sector identifier   |
| **district_id** (FK → districts) | TINYINT       | Parent district            |
| **sector_name**                  | VARCHAR(100)  | Sector name                |
| **sector_code**                  | VARCHAR(20)   | Unique sector code         |
| **boundary_geojson**             | JSON          | Sector boundary as GeoJSON |
| **centroid_latitude**            | DECIMAL(10,8) | Center point latitude      |
| **centroid_longitude**           | DECIMAL(11,8) | Center point longitude     |
| **population**                   | INT           | Sector population          |
| **area_sq_km**                   | DECIMAL(10,2) | Area in square kilometers  |
| **cell_count**                   | TINYINT       | Number of cells            |
| **police_station_name**          | VARCHAR(100)  | Local police station name  |
| **police_station_contact**       | VARCHAR(20)   | Police station phone       |
| **is_active**                    | BOOLEAN       | Whether sector is active   |
| **created_at**                   | TIMESTAMP     | Record creation time       |
| **updated_at**                   | TIMESTAMP     | Last update time           |

---

## 2.4 cells — Cells within Sectors

**Meaning:**
Cell-level administration (smallest official admin unit).

**Purpose:**
Most granular official geographic reference for incidents.

| Field                        | Type          | Description                              |
| ---------------------------- | ------------- | ---------------------------------------- |
| **cell_id** (PK)             | SMALLINT      | Unique cell identifier                   |
| **sector_id** (FK → sectors) | SMALLINT      | Parent sector                            |
| **cell_name**                | VARCHAR(100)  | Cell name                                |
| **cell_code**                | VARCHAR(30)   | Unique cell code                         |
| **boundary_geojson**         | JSON          | Cell boundary as GeoJSON                 |
| **centroid_latitude**        | DECIMAL(10,8) | Center point latitude                    |
| **centroid_longitude**       | DECIMAL(11,8) | Center point longitude                   |
| **population**               | INT           | Cell population                          |
| **area_sq_km**               | DECIMAL(8,2)  | Area in square kilometers                |
| **cell_leader_title**        | VARCHAR(50)   | Local leader title (Executive Secretary) |
| **is_active**                | BOOLEAN       | Whether cell is active                   |
| **created_at**               | TIMESTAMP     | Record creation time                     |
| **updated_at**               | TIMESTAMP     | Last update time                         |

---

## 2.5 villages — Villages within Cells

**Meaning:**
Village-level locations within cells.

**Purpose:**
Optional finer location detail for rural areas.

| Field                    | Type          | Description               |
| ------------------------ | ------------- | ------------------------- |
| **village_id** (PK)      | INT           | Unique village identifier |
| **cell_id** (FK → cells) | SMALLINT      | Parent cell               |
| **village_name**         | VARCHAR(100)  | Village name              |
| **village_code**         | VARCHAR(40)   | Unique village code       |
| **centroid_latitude**    | DECIMAL(10,8) | Center point latitude     |
| **centroid_longitude**   | DECIMAL(11,8) | Center point longitude    |
| **population**           | INT           | Village population        |
| **household_count**      | INT           | Number of households      |
| **is_active**            | BOOLEAN       | Whether village is active |
| **created_at**           | TIMESTAMP     | Record creation time      |
| **updated_at**           | TIMESTAMP     | Last update time          |

---

# SECTION 3: INCIDENT TAXONOMY

---

## 3.1 incident_categories — Main Incident Categories

**Meaning:**
Top-level incident categories (Theft, Vandalism, etc.).

**Purpose:**
Organizes incident types into logical groups for reporting and analysis.

| Field                 | Type         | Description                             |
| --------------------- | ------------ | --------------------------------------- |
| **category_id** (PK)  | TINYINT      | Unique category identifier              |
| **category_name**     | VARCHAR(100) | Category name                           |
| **description**       | TEXT         | Category description                    |
| **icon_name**         | VARCHAR(50)  | Material icon name for UI               |
| **color_hex**         | CHAR(7)      | Category color for UI (#1976D2)         |
| **display_order**     | TINYINT      | Order in category list                  |
| **is_active**         | BOOLEAN      | Whether category is active              |
| **requires_evidence** | BOOLEAN      | Force evidence upload for this category |
| **created_at**        | TIMESTAMP    | Record creation time                    |
| **updated_at**        | TIMESTAMP    | Last update time                        |

---

## 3.2 incident_types — Specific Incident Types

**Meaning:**
Specific incident types within categories.

**Purpose:**
Detailed incident classification for ML patterns and police response.

| Field                                      | Type         | Description                                  |
| ------------------------------------------ | ------------ | -------------------------------------------- |
| **type_id** (PK)                           | SMALLINT     | Unique type identifier                       |
| **category_id** (FK → incident_categories) | TINYINT      | Parent category                              |
| **type_name**                              | VARCHAR(100) | Type name                                    |
| **description**                            | TEXT         | Type description                             |
| **severity_level**                         | TINYINT      | 1=Minor, 2=Low, 3=Medium, 4=High, 5=Critical |
| **response_priority**                      | ENUM         | low, normal, high, urgent                    |
| **requires_photo**                         | BOOLEAN      | Photo required for this type                 |
| **requires_video**                         | BOOLEAN      | Video required for this type                 |
| **min_description_length**                 | INT          | Minimum description characters               |
| **icon_name**                              | VARCHAR(50)  | Specific icon override                       |
| **display_order**                          | TINYINT      | Order within category                        |
| **is_active**                              | BOOLEAN      | Whether type is active                       |
| **created_at**                             | TIMESTAMP    | Record creation time                         |
| **updated_at**                             | TIMESTAMP    | Last update time                             |

---

# SECTION 4: INCIDENT REPORTS (Core Transaction Table)

---

## 4.1 incident_reports — Main Reports Table

**Meaning:**
Core incident reports - every reported incident is stored here.

**Purpose:**
Central table linking all report data including location, evidence, verification, and ML scoring.

### Basic Information

| Field                                      | Type         | Description                     |
| ------------------------------------------ | ------------ | ------------------------------- |
| **report_id** (PK)                         | CHAR(36)     | UUID - Unique report identifier |
| **device_id** (FK → devices)               | CHAR(36)     | Reporting device (anonymous)    |
| **incident_type_id** (FK → incident_types) | SMALLINT     | Type of incident                |
| **title**                                  | VARCHAR(200) | Optional short title            |
| **description**                            | TEXT         | Detailed incident description   |

### Location Fields

| Field                            | Type          | Description                   |
| -------------------------------- | ------------- | ----------------------------- |
| **latitude**                     | DECIMAL(10,8) | GPS latitude (-90 to 90)      |
| **longitude**                    | DECIMAL(11,8) | GPS longitude (-180 to 180)   |
| **location_accuracy_meters**     | DECIMAL(8,2)  | GPS accuracy in meters        |
| **altitude_meters**              | DECIMAL(8,2)  | Altitude if available         |
| **location_source**              | ENUM          | gps, network, manual          |
| **district_id** (FK → districts) | TINYINT       | Resolved district             |
| **sector_id** (FK → sectors)     | SMALLINT      | Resolved sector               |
| **cell_id** (FK → cells)         | SMALLINT      | Resolved cell                 |
| **village_id** (FK → villages)   | INT           | Resolved village (optional)   |
| **address_description**          | VARCHAR(255)  | Optional landmark description |

### Temporal Fields

| Field                         | Type      | Description                     |
| ----------------------------- | --------- | ------------------------------- |
| **reported_at**               | TIMESTAMP | When report was submitted       |
| **incident_occurred_at**      | TIMESTAMP | When incident actually happened |
| **incident_time_approximate** | BOOLEAN   | User unsure of exact time       |

### Evidence Summary

| Field                      | Type    | Description                    |
| -------------------------- | ------- | ------------------------------ |
| **photo_count**            | TINYINT | Number of photos attached      |
| **video_count**            | TINYINT | Number of videos attached      |
| **audio_count**            | TINYINT | Number of audio files attached |
| **total_evidence_size_kb** | INT     | Total evidence file size       |

### Motion Sensor Data (Verification)

| Field                   | Type         | Description                               |
| ----------------------- | ------------ | ----------------------------------------- |
| **accelerometer_data**  | JSON         | {x, y, z, magnitude, samples[]}           |
| **gyroscope_data**      | JSON         | {x, y, z, rotation_rate}                  |
| **magnetometer_data**   | JSON         | Compass data for orientation              |
| **device_motion_score** | DECIMAL(5,2) | 0-100: Higher = more motion during report |
| **device_orientation**  | VARCHAR(20)  | portrait, landscape, flat                 |
| **battery_level**       | TINYINT      | Battery % at report time                  |
| **network_type**        | VARCHAR(20)  | wifi, 4g, 3g, 2g                          |

### Rule-Based Verification (Stage 1)

| Field                       | Type      | Description                           |
| --------------------------- | --------- | ------------------------------------- |
| **rule_check_status**       | ENUM      | pending, passed, failed, partial      |
| **rule_check_completed_at** | TIMESTAMP | When rule check finished              |
| **rules_passed**            | INT       | Number of rules passed                |
| **rules_failed**            | INT       | Number of rules failed                |
| **rules_total**             | INT       | Total rules executed                  |
| **rule_failure_reasons**    | JSON      | Array of {rule_id, rule_name, reason} |
| **is_auto_rejected**        | BOOLEAN   | TRUE if blocking rule failed          |

### ML Trust Scoring (Stage 2)

| Field                            | Type         | Description                          |
| -------------------------------- | ------------ | ------------------------------------ |
| **ml_model_id** (FK → ml_models) | INT          | Which model scored this              |
| **ml_trust_score**               | DECIMAL(5,2) | 0-100 trust score from ML            |
| **ml_confidence**                | DECIMAL(5,4) | Model confidence in prediction       |
| **ml_feature_vector**            | JSON         | Features used (for explainability)   |
| **ml_scored_at**                 | TIMESTAMP    | When ML scoring completed            |
| **trust_classification**         | ENUM         | Trusted, Suspicious, False, Pending  |
| **classification_reason**        | VARCHAR(255) | Why this classification was assigned |

### Police Verification (Ground Truth)

| Field                                      | Type      | Description                                             |
| ------------------------------------------ | --------- | ------------------------------------------------------- |
| **police_verified**                        | BOOLEAN   | Has police reviewed this                                |
| **police_verified_at**                     | TIMESTAMP | When verified                                           |
| **police_verified_by** (FK → police_users) | INT       | Officer who verified                                    |
| **police_verification_status**             | ENUM      | pending, confirmed, false, duplicate, insufficient_info |
| **police_verification_notes**              | TEXT      | Officer notes                                           |
| **police_priority**                        | ENUM      | low, normal, high, urgent                               |

### Workflow Status

| Field                                              | Type         | Description                                                                               |
| -------------------------------------------------- | ------------ | ----------------------------------------------------------------------------------------- |
| **report_status**                                  | ENUM         | submitted, rule_checking, ml_scoring, pending_review, investigating, resolved, rejected   |
| **processing_stage**                               | ENUM         | received, rule_validation, ml_scoring, clustering, ready_for_review, in_review, completed |
| **is_duplicate**                                   | BOOLEAN      | Is this a duplicate report                                                                |
| **duplicate_of_report_id** (FK → incident_reports) | CHAR(36)     | Original report if duplicate                                                              |
| **duplicate_confidence**                           | DECIMAL(5,2) | How confident it's a duplicate                                                            |

### Assignment & Resolution

| Field                                       | Type         | Description                                                       |
| ------------------------------------------- | ------------ | ----------------------------------------------------------------- |
| **assigned_officer_id** (FK → police_users) | INT          | Assigned officer                                                  |
| **assigned_unit**                           | VARCHAR(100) | Assigned police unit                                              |
| **assigned_at**                             | TIMESTAMP    | When assigned                                                     |
| **resolved_at**                             | TIMESTAMP    | When resolved                                                     |
| **resolution_type**                         | ENUM         | action_taken, no_action_needed, referred, false_report, duplicate |
| **resolution_notes**                        | TEXT         | Resolution details                                                |

### Hotspot Linkage

| Field                          | Type      | Description                   |
| ------------------------------ | --------- | ----------------------------- |
| **hotspot_id** (FK → hotspots) | INT       | Assigned hotspot if clustered |
| **added_to_hotspot_at**        | TIMESTAMP | When added to hotspot         |

### Metadata

| Field                  | Type        | Description                   |
| ---------------------- | ----------- | ----------------------------- |
| **app_version**        | VARCHAR(20) | App version used              |
| **submission_ip_hash** | VARCHAR(64) | Hashed IP for fraud detection |
| **created_at**         | TIMESTAMP   | Record creation               |
| **updated_at**         | TIMESTAMP   | Last update                   |

---

# SECTION 5: EVIDENCE MANAGEMENT

---

## 5.1 report_evidence — Evidence Files

**Meaning:**
Stores metadata for photos, videos, and audio evidence.

**Purpose:**
Links evidence files to reports with quality analysis and moderation status.

| Field                                 | Type          | Description                          |
| ------------------------------------- | ------------- | ------------------------------------ |
| **evidence_id** (PK)                  | CHAR(36)      | UUID - Unique evidence identifier    |
| **report_id** (FK → incident_reports) | CHAR(36)      | Parent report                        |
| **evidence_type**                     | ENUM          | photo, video, audio                  |
| **file_name**                         | VARCHAR(255)  | Original file name                   |
| **file_path**                         | VARCHAR(500)  | Encrypted storage path               |
| **file_size_bytes**                   | INT           | File size in bytes                   |
| **mime_type**                         | VARCHAR(100)  | MIME type (image/jpeg, video/mp4)    |
| **duration_seconds**                  | SMALLINT      | Duration for video/audio             |
| **width_pixels**                      | SMALLINT      | Width for photo/video                |
| **height_pixels**                     | SMALLINT      | Height for photo/video               |
| **file_hash_sha256**                  | CHAR(64)      | SHA-256 for integrity check          |
| **file_hash_perceptual**              | VARCHAR(64)   | pHash for image similarity detection |
| **captured_at**                       | TIMESTAMP     | When media was captured              |
| **capture_latitude**                  | DECIMAL(10,8) | GPS latitude from EXIF               |
| **capture_longitude**                 | DECIMAL(11,8) | GPS longitude from EXIF              |
| **camera_metadata**                   | JSON          | EXIF data: make, model, settings     |
| **blur_score**                        | DECIMAL(5,2)  | 0-100: Higher = sharper image        |
| **brightness_score**                  | DECIMAL(5,2)  | 0-100: Image brightness              |
| **is_low_quality**                    | BOOLEAN       | Flagged as low quality               |
| **quality_issues**                    | JSON          | Array of quality problems            |
| **content_moderation_status**         | ENUM          | pending, approved, flagged, rejected |
| **has_inappropriate_content**         | BOOLEAN       | Contains inappropriate content       |
| **moderation_flags**                  | JSON          | Content moderation results           |
| **moderated_at**                      | TIMESTAMP     | When moderated                       |
| **moderated_by** (FK → police_users)  | INT           | Who moderated                        |
| **is_processed**                      | BOOLEAN       | Processing completed                 |
| **is_deleted**                        | BOOLEAN       | Soft delete flag                     |
| **deleted_at**                        | TIMESTAMP     | When deleted                         |
| **deletion_reason**                   | VARCHAR(100)  | Why deleted                          |
| **uploaded_at**                       | TIMESTAMP     | Upload timestamp                     |

---

# SECTION 6: VERIFICATION RULES ENGINE

---

## 6.1 verification_rules — Configurable Rules

**Meaning:**
Configurable verification rules for report validation.

**Purpose:**
Defines rules for Stage 1 (rule-based) verification with flexible parameters.

| Field                              | Type         | Description                                          |
| ---------------------------------- | ------------ | ---------------------------------------------------- |
| **rule_id** (PK)                   | SMALLINT     | Unique rule identifier                               |
| **rule_name**                      | VARCHAR(100) | Human-readable rule name                             |
| **rule_code**                      | VARCHAR(50)  | Programmatic identifier                              |
| **rule_description**               | TEXT         | Rule description                                     |
| **rule_category**                  | ENUM         | spatial, temporal, motion, evidence, device, content |
| **rule_parameters**                | JSON         | Flexible rule configuration (thresholds, limits)     |
| **severity**                       | ENUM         | info, low, medium, high, critical                    |
| **is_blocking**                    | BOOLEAN      | If TRUE, failure auto-rejects report                 |
| **failure_score_penalty**          | DECIMAL(5,2) | Points deducted if rule fails                        |
| **execution_order**                | SMALLINT     | Order of rule execution                              |
| **is_active**                      | BOOLEAN      | Whether rule is enabled                              |
| **applies_to_categories**          | JSON         | NULL = all, or array of category_ids                 |
| **applies_to_districts**           | JSON         | NULL = all, or array of district_ids                 |
| **created_at**                     | TIMESTAMP    | Record creation                                      |
| **updated_at**                     | TIMESTAMP    | Last update                                          |
| **created_by** (FK → police_users) | INT          | Who created the rule                                 |

---

## 6.2 rule_execution_logs — Rule Execution Results

**Meaning:**
Logs each rule execution result for every report.

**Purpose:**
Audit trail and analysis of rule effectiveness.

| Field                                 | Type         | Description                   |
| ------------------------------------- | ------------ | ----------------------------- |
| **execution_id** (PK)                 | INT          | Unique execution identifier   |
| **report_id** (FK → incident_reports) | CHAR(36)     | Report being checked          |
| **rule_id** (FK → verification_rules) | SMALLINT     | Rule that was executed        |
| **passed**                            | BOOLEAN      | Whether rule passed           |
| **input_values**                      | JSON         | Values that were checked      |
| **threshold_values**                  | JSON         | Thresholds used               |
| **failure_reason**                    | TEXT         | Why it failed (if applicable) |
| **execution_time_ms**                 | DECIMAL(8,2) | Rule execution time           |
| **executed_at**                       | TIMESTAMP    | When rule was executed        |

---

# SECTION 7: MACHINE LEARNING SYSTEM

---

## 7.1 ml_models — ML Model Registry

**Meaning:**
Tracks ML model versions and performance metrics.

**Purpose:**
Model versioning, A/B testing, and performance monitoring.

| Field                              | Type         | Description                                       |
| ---------------------------------- | ------------ | ------------------------------------------------- |
| **model_id** (PK)                  | INT          | Unique model identifier                           |
| **model_name**                     | VARCHAR(100) | Model name                                        |
| **model_version**                  | VARCHAR(20)  | Version string (1.0.0)                            |
| **model_type**                     | VARCHAR(50)  | LogisticRegression, RandomForest, XGBoost, etc.   |
| **model_file_path**                | VARCHAR(500) | Path to serialized model file                     |
| **model_file_hash**                | VARCHAR(64)  | SHA-256 of model file                             |
| **model_size_kb**                  | INT          | Model file size                                   |
| **trained_at**                     | TIMESTAMP    | When model was trained                            |
| **training_dataset_size**          | INT          | Number of training samples                        |
| **training_duration_seconds**      | INT          | Training time                                     |
| **accuracy**                       | DECIMAL(5,4) | Model accuracy                                    |
| **precision_score**                | DECIMAL(5,4) | Precision metric                                  |
| **recall_score**                   | DECIMAL(5,4) | Recall metric                                     |
| **f1_score**                       | DECIMAL(5,4) | F1 score                                          |
| **auc_roc**                        | DECIMAL(5,4) | Area under ROC curve                              |
| **metrics_by_class**               | JSON         | {Trusted: {precision, recall}, Suspicious: {...}} |
| **confusion_matrix**               | JSON         | Confusion matrix data                             |
| **feature_names**                  | JSON         | List of feature names used                        |
| **feature_importance**             | JSON         | Feature importance scores                         |
| **threshold_trusted**              | DECIMAL(5,2) | Score >= this = Trusted (default 70)              |
| **threshold_suspicious**           | DECIMAL(5,2) | Score >= this = Suspicious (default 40)           |
| **is_active**                      | BOOLEAN      | Currently accepting predictions                   |
| **is_champion**                    | BOOLEAN      | Best performing model                             |
| **deployed_at**                    | TIMESTAMP    | When deployed to production                       |
| **deprecated_at**                  | TIMESTAMP    | When deprecated                                   |
| **deprecation_reason**             | VARCHAR(255) | Why deprecated                                    |
| **total_predictions**              | INT          | Total predictions made                            |
| **correct_predictions**            | INT          | Correct predictions count                         |
| **avg_inference_time_ms**          | DECIMAL(8,2) | Average prediction time                           |
| **training_notes**                 | TEXT         | Training documentation                            |
| **created_at**                     | TIMESTAMP    | Record creation                                   |
| **created_by** (FK → police_users) | INT          | Who created/trained                               |

---

## 7.2 ml_predictions — Prediction Logs

**Meaning:**
Logs all ML predictions for monitoring and retraining.

**Purpose:**
Track predictions, compare with ground truth, identify model drift.

| Field                                 | Type         | Description                                    |
| ------------------------------------- | ------------ | ---------------------------------------------- |
| **prediction_id** (PK)                | CHAR(36)     | UUID - Unique prediction identifier            |
| **report_id** (FK → incident_reports) | CHAR(36)     | Report that was scored                         |
| **model_id** (FK → ml_models)         | INT          | Model that made prediction                     |
| **feature_vector**                    | JSON         | Features used for prediction                   |
| **predicted_score**                   | DECIMAL(5,2) | 0-100 trust score                              |
| **predicted_class**                   | ENUM         | Trusted, Suspicious, False                     |
| **confidence**                        | DECIMAL(5,4) | Model confidence                               |
| **class_probabilities**               | JSON         | {Trusted: 0.75, Suspicious: 0.20, False: 0.05} |
| **actual_class**                      | ENUM         | Ground truth (filled when police verifies)     |
| **is_correct**                        | BOOLEAN      | Whether prediction was correct                 |
| **inference_time_ms**                 | DECIMAL(8,2) | Prediction time                                |
| **predicted_at**                      | TIMESTAMP    | When prediction was made                       |
| **verified_at**                       | TIMESTAMP    | When ground truth was added                    |

---

## 7.3 ml_training_data — Training Dataset

**Meaning:**
Curated training data for model retraining.

**Purpose:**
Maintains high-quality labeled data from police verifications.

| Field                                 | Type         | Description                                         |
| ------------------------------------- | ------------ | --------------------------------------------------- |
| **training_id** (PK)                  | INT          | Unique training record ID                           |
| **report_id** (FK → incident_reports) | CHAR(36)     | Source report                                       |
| **feature_vector**                    | JSON         | Features extracted                                  |
| **label**                             | ENUM         | Trusted, Suspicious, False                          |
| **label_confidence**                  | DECIMAL(5,2) | Confidence in label (100 = police verified)         |
| **labeled_by** (FK → police_users)    | INT          | Who labeled                                         |
| **labeled_at**                        | TIMESTAMP    | When labeled                                        |
| **label_source**                      | ENUM         | police_verification, expert_review, consensus, auto |
| **dataset_split**                     | ENUM         | train, validation, test, holdout                    |
| **assigned_to_split_at**              | TIMESTAMP    | When assigned to split                              |
| **used_in_model_version**             | VARCHAR(20)  | Which model used this                               |
| **used_at**                           | TIMESTAMP    | When used in training                               |
| **is_high_quality**                   | BOOLEAN      | Quality flag                                        |
| **quality_issues**                    | JSON         | Any data quality concerns                           |
| **created_at**                        | TIMESTAMP    | Record creation                                     |

---

# SECTION 8: HOTSPOT DETECTION & CLUSTERING

---

## 8.1 hotspots — Detected Incident Clusters

**Meaning:**
Crime hotspots detected via trust-weighted DBSCAN clustering.

**Purpose:**
Identifies high-risk locations for police resource allocation.

### Cluster Identification

| Field                                     | Type     | Description                       |
| ----------------------------------------- | -------- | --------------------------------- |
| **hotspot_id** (PK)                       | INT      | Unique hotspot identifier         |
| **cluster_label**                         | INT      | DBSCAN cluster label              |
| **cluster_run_id** (FK → clustering_runs) | CHAR(36) | Which clustering run created this |

### Spatial Data

| Field                            | Type          | Description                        |
| -------------------------------- | ------------- | ---------------------------------- |
| **centroid_latitude**            | DECIMAL(10,8) | Cluster center latitude            |
| **centroid_longitude**           | DECIMAL(11,8) | Cluster center longitude           |
| **boundary_geojson**             | JSON          | Convex hull polygon around cluster |
| **radius_meters**                | DECIMAL(10,2) | Approximate radius                 |
| **area_sq_meters**               | DECIMAL(12,2) | Cluster area                       |
| **district_id** (FK → districts) | TINYINT       | Primary district                   |
| **sector_id** (FK → sectors)     | SMALLINT      | Primary sector                     |
| **cell_id** (FK → cells)         | SMALLINT      | Primary cell                       |
| **village_id** (FK → villages)   | INT           | Primary village                    |

### Cluster Statistics

| Field                       | Type          | Description                               |
| --------------------------- | ------------- | ----------------------------------------- |
| **report_count**            | INT           | Number of reports in cluster              |
| **unique_devices**          | INT           | Unique reporting devices                  |
| **avg_trust_score**         | DECIMAL(5,2)  | Average trust score                       |
| **min_trust_score**         | DECIMAL(5,2)  | Minimum trust score                       |
| **max_trust_score**         | DECIMAL(5,2)  | Maximum trust score                       |
| **std_trust_score**         | DECIMAL(5,2)  | Standard deviation                        |
| **weighted_trust_density**  | DECIMAL(10,4) | Trust-weighted density for prioritization |
| **trusted_report_count**    | INT           | Trusted reports in cluster                |
| **suspicious_report_count** | INT           | Suspicious reports in cluster             |
| **false_report_count**      | INT           | False reports in cluster                  |
| **police_verified_count**   | INT           | Police-verified reports                   |

### Temporal Data

| Field                    | Type      | Description                    |
| ------------------------ | --------- | ------------------------------ |
| **earliest_incident_at** | TIMESTAMP | First incident in cluster      |
| **latest_incident_at**   | TIMESTAMP | Last incident in cluster       |
| **time_span_hours**      | INT       | Duration of cluster activity   |
| **peak_hour**            | TINYINT   | 0-23, hour with most incidents |
| **peak_day_of_week**     | TINYINT   | 0=Sunday, 6=Saturday           |

### Incident Analysis

| Field                                               | Type         | Description                 |
| --------------------------------------------------- | ------------ | --------------------------- |
| **incident_type_distribution**                      | JSON         | {type_id: count, ...}       |
| **dominant_incident_type_id** (FK → incident_types) | SMALLINT     | Most common type            |
| **dominant_incident_pct**                           | DECIMAL(5,2) | Percentage of dominant type |

### Risk Assessment

| Field              | Type         | Description                        |
| ------------------ | ------------ | ---------------------------------- |
| **risk_level**     | ENUM         | low, medium, high, critical        |
| **priority_score** | DECIMAL(5,2) | 0-100 composite priority score     |
| **risk_factors**   | JSON         | Factors contributing to risk level |

### DBSCAN Parameters

| Field                     | Type          | Description                         |
| ------------------------- | ------------- | ----------------------------------- |
| **dbscan_epsilon_meters** | DECIMAL(10,2) | Epsilon parameter used              |
| **dbscan_min_samples**    | INT           | Min samples parameter used          |
| **trust_weight_enabled**  | BOOLEAN       | Whether trust weighting was applied |

### Status & Response

| Field                                          | Type         | Description                                       |
| ---------------------------------------------- | ------------ | ------------------------------------------------- |
| **is_active**                                  | BOOLEAN      | Hotspot still active                              |
| **status**                                     | ENUM         | new, monitoring, responding, addressed, recurring |
| **is_assigned**                                | BOOLEAN      | Has been assigned                                 |
| **assigned_to_officer_id** (FK → police_users) | INT          | Assigned officer                                  |
| **assigned_to_unit**                           | VARCHAR(100) | Assigned unit                                     |
| **assigned_at**                                | TIMESTAMP    | When assigned                                     |
| **is_addressed**                               | BOOLEAN      | Has been addressed                                |
| **addressed_at**                               | TIMESTAMP    | When addressed                                    |
| **addressed_by** (FK → police_users)           | INT          | Who addressed                                     |
| **resolution_notes**                           | TEXT         | Resolution details                                |
| **detected_at**                                | TIMESTAMP    | When hotspot was detected                         |
| **updated_at**                                 | TIMESTAMP    | Last update                                       |
| **last_report_added_at**                       | TIMESTAMP    | Last report added                                 |

---

## 8.2 hotspot_reports — Bridge Table

**Meaning:**
Links reports to hotspot clusters.

**Purpose:**
Many-to-many relationship between reports and hotspots.

| Field                                     | Type          | Description                        |
| ----------------------------------------- | ------------- | ---------------------------------- |
| **hotspot_id** (PK, FK → hotspots)        | INT           | Hotspot cluster                    |
| **report_id** (PK, FK → incident_reports) | CHAR(36)      | Report in cluster                  |
| **trust_weight**                          | DECIMAL(5,4)  | Report weight based on trust score |
| **distance_to_centroid_meters**           | DECIMAL(10,2) | Distance from cluster center       |
| **is_core_point**                         | BOOLEAN       | TRUE if DBSCAN core point          |
| **added_at**                              | TIMESTAMP     | When added to hotspot              |

---

## 8.3 hotspot_history — Hotspot Trends

**Meaning:**
Historical snapshots of hotspot metrics.

**Purpose:**
Track hotspot evolution over time.

| Field                          | Type         | Description                  |
| ------------------------------ | ------------ | ---------------------------- |
| **history_id** (PK)            | INT          | Unique history record        |
| **hotspot_id** (FK → hotspots) | INT          | Hotspot being tracked        |
| **snapshot_date**              | DATE         | Date of snapshot             |
| **report_count**               | INT          | Reports at this time         |
| **avg_trust_score**            | DECIMAL(5,2) | Average trust score          |
| **risk_level**                 | ENUM         | Risk level at snapshot       |
| **priority_score**             | DECIMAL(5,2) | Priority score               |
| **report_count_change**        | INT          | Change from previous         |
| **trust_score_change**         | DECIMAL(5,2) | Score change                 |
| **risk_level_changed**         | BOOLEAN      | Risk level changed           |
| **trend_direction**            | ENUM         | improving, stable, worsening |
| **created_at**                 | TIMESTAMP    | Snapshot time                |

---

## 8.4 clustering_runs — Clustering Execution Logs

**Meaning:**
Logs each DBSCAN clustering execution.

**Purpose:**
Track clustering runs, parameters, and results.

| Field                                | Type          | Description                  |
| ------------------------------------ | ------------- | ---------------------------- |
| **run_id** (PK)                      | CHAR(36)      | UUID - Unique run identifier |
| **district_id** (FK → districts)     | TINYINT       | NULL = all districts         |
| **epsilon_meters**                   | DECIMAL(10,2) | DBSCAN epsilon parameter     |
| **min_samples**                      | INT           | DBSCAN min_samples parameter |
| **trust_weight_enabled**             | BOOLEAN       | Trust weighting applied      |
| **min_trust_score_threshold**        | DECIMAL(5,2)  | Minimum trust to include     |
| **total_reports_processed**          | INT           | Total reports analyzed       |
| **reports_after_filtering**          | INT           | Reports after trust filter   |
| **date_range_start**                 | TIMESTAMP     | Report date range start      |
| **date_range_end**                   | TIMESTAMP     | Report date range end        |
| **clusters_found**                   | INT           | Number of clusters detected  |
| **noise_points**                     | INT           | Reports not in any cluster   |
| **avg_cluster_size**                 | DECIMAL(8,2)  | Average cluster size         |
| **execution_time_seconds**           | DECIMAL(10,2) | Run duration                 |
| **status**                           | ENUM          | running, completed, failed   |
| **error_message**                    | TEXT          | Error if failed              |
| **started_at**                       | TIMESTAMP     | Run start time               |
| **completed_at**                     | TIMESTAMP     | Run completion time          |
| **triggered_by** (FK → police_users) | INT           | NULL = scheduled             |

---

# SECTION 9: POLICE USER MANAGEMENT

---

## 9.1 police_users — Police Accounts

**Meaning:**
Police and admin user accounts for the dashboard.

**Purpose:**
Authentication, authorization, and jurisdiction management.

### Authentication

| Field             | Type         | Description            |
| ----------------- | ------------ | ---------------------- |
| **user_id** (PK)  | INT          | Unique user identifier |
| **username**      | VARCHAR(50)  | Unique username        |
| **email**         | VARCHAR(255) | Unique email address   |
| **password_hash** | VARCHAR(255) | Bcrypt hashed password |

### Personal Information

| Field                  | Type         | Description                |
| ---------------------- | ------------ | -------------------------- |
| **full_name**          | VARCHAR(200) | Officer's full name        |
| **badge_number**       | VARCHAR(50)  | Unique badge number        |
| **rank**               | VARCHAR(50)  | Police rank                |
| **phone_number**       | VARCHAR(20)  | Contact phone              |
| **profile_photo_path** | VARCHAR(500) | Profile photo storage path |

### Role & Permissions

| Field           | Type | Description                                             |
| --------------- | ---- | ------------------------------------------------------- |
| **role**        | ENUM | super_admin, admin, commander, officer, analyst, viewer |
| **permissions** | JSON | Granular permission flags                               |

### Assignment & Jurisdiction

| Field                                     | Type         | Description                      |
| ----------------------------------------- | ------------ | -------------------------------- |
| **assigned_district_id** (FK → districts) | TINYINT      | Primary assigned district        |
| **assigned_sector_id** (FK → sectors)     | SMALLINT     | Primary assigned sector          |
| **assigned_unit**                         | VARCHAR(100) | Police station/unit name         |
| **jurisdiction_district_ids**             | JSON         | Array of accessible district IDs |
| **can_access_all_districts**              | BOOLEAN      | Has national access              |

### Account Status

| Field                               | Type      | Description       |
| ----------------------------------- | --------- | ----------------- |
| **is_active**                       | BOOLEAN   | Account is active |
| **is_verified**                     | BOOLEAN   | Account verified  |
| **verified_at**                     | TIMESTAMP | When verified     |
| **verified_by** (FK → police_users) | INT       | Who verified      |

### Security

| Field                       | Type         | Description           |
| --------------------------- | ------------ | --------------------- |
| **two_factor_enabled**      | BOOLEAN      | 2FA enabled           |
| **two_factor_secret**       | VARCHAR(100) | TOTP secret           |
| **two_factor_backup_codes** | JSON         | Backup codes          |
| **password_changed_at**     | TIMESTAMP    | Last password change  |
| **must_change_password**    | BOOLEAN      | Force password change |
| **failed_login_attempts**   | TINYINT      | Failed login count    |
| **account_locked_until**    | TIMESTAMP    | Lockout expiration    |
| **last_failed_login_at**    | TIMESTAMP    | Last failed attempt   |

### Activity

| Field             | Type        | Description           |
| ----------------- | ----------- | --------------------- |
| **last_login_at** | TIMESTAMP   | Last successful login |
| **last_login_ip** | VARCHAR(45) | Last login IP         |
| **login_count**   | INT         | Total logins          |

### Preferences

| Field                        | Type | Description           |
| ---------------------------- | ---- | --------------------- |
| **preferred_language**       | ENUM | en                    |
| **notification_preferences** | JSON | Notification settings |
| **dashboard_settings**       | JSON | UI preferences        |

### Timestamps

| Field                              | Type      | Description         |
| ---------------------------------- | --------- | ------------------- |
| **created_at**                     | TIMESTAMP | Account creation    |
| **updated_at**                     | TIMESTAMP | Last update         |
| **created_by** (FK → police_users) | INT       | Who created account |

---

## 9.2 police_sessions — Active Sessions

**Meaning:**
Active login sessions for security management.

**Purpose:**
Session tracking, token management, and security.

| Field                           | Type         | Description               |
| ------------------------------- | ------------ | ------------------------- |
| **session_id** (PK)             | CHAR(36)     | UUID - Session identifier |
| **user_id** (FK → police_users) | INT          | User who owns session     |
| **token_hash**                  | VARCHAR(255) | Hashed session token      |
| **refresh_token_hash**          | VARCHAR(255) | Hashed refresh token      |
| **ip_address**                  | VARCHAR(45)  | Client IP address         |
| **user_agent**                  | TEXT         | Browser/client info       |
| **device_info**                 | JSON         | Device details            |
| **is_active**                   | BOOLEAN      | Session is valid          |
| **created_at**                  | TIMESTAMP    | Session start             |
| **last_activity_at**            | TIMESTAMP    | Last activity             |
| **expires_at**                  | TIMESTAMP    | Session expiration        |
| **revoked_at**                  | TIMESTAMP    | If revoked, when          |
| **revoked_reason**              | VARCHAR(100) | Why revoked               |

---

# SECTION 10: NOTIFICATIONS

---

## 10.1 notifications — Police Alerts

**Meaning:**
Notifications and alerts for police users.

**Purpose:**
Real-time alerting for new reports, hotspots, and assignments.

| Field                                     | Type         | Description                                                                                                                                                        |
| ----------------------------------------- | ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **notification_id** (PK)                  | INT          | Unique notification ID                                                                                                                                             |
| **recipient_user_id** (FK → police_users) | INT          | Who receives notification                                                                                                                                          |
| **notification_type**                     | ENUM         | new_report, high_trust_report, suspicious_report, hotspot_detected, hotspot_escalated, assignment, verification_needed, system_alert, model_update, weekly_summary |
| **title**                                 | VARCHAR(200) | Notification title                                                                                                                                                 |
| **message**                               | TEXT         | Notification message                                                                                                                                               |
| **report_id** (FK → incident_reports)     | CHAR(36)     | Related report                                                                                                                                                     |
| **hotspot_id** (FK → hotspots)            | INT          | Related hotspot                                                                                                                                                    |
| **priority**                              | ENUM         | low, normal, high, urgent                                                                                                                                          |
| **delivery_channels**                     | JSON         | ["web", "email", "push"]                                                                                                                                           |
| **is_read**                               | BOOLEAN      | Has been read                                                                                                                                                      |
| **read_at**                               | TIMESTAMP    | When read                                                                                                                                                          |
| **is_dismissed**                          | BOOLEAN      | Has been dismissed                                                                                                                                                 |
| **dismissed_at**                          | TIMESTAMP    | When dismissed                                                                                                                                                     |
| **action_taken**                          | VARCHAR(100) | What action user took                                                                                                                                              |
| **action_taken_at**                       | TIMESTAMP    | When action taken                                                                                                                                                  |
| **created_at**                            | TIMESTAMP    | Notification created                                                                                                                                               |
| **expires_at**                            | TIMESTAMP    | When notification expires                                                                                                                                          |

---

# SECTION 11: ANALYTICS & STATISTICS

---

## 11.1 daily_statistics — Aggregated Daily Metrics

**Meaning:**
Pre-aggregated daily metrics for fast dashboard queries.

**Purpose:**
Performance optimization for analytics dashboards.

| Field                            | Type     | Description           |
| -------------------------------- | -------- | --------------------- |
| **stat_id** (PK)                 | INT      | Unique stat record    |
| **stat_date**                    | DATE     | Date of statistics    |
| **district_id** (FK → districts) | TINYINT  | NULL = national       |
| **sector_id** (FK → sectors)     | SMALLINT | NULL = district level |

### Report Metrics

| Field                  | Type         | Description               |
| ---------------------- | ------------ | ------------------------- |
| **total_reports**      | INT          | Total reports             |
| **trusted_reports**    | INT          | Trusted classification    |
| **suspicious_reports** | INT          | Suspicious classification |
| **false_reports**      | INT          | False classification      |
| **pending_reports**    | INT          | Pending review            |
| **avg_trust_score**    | DECIMAL(5,2) | Average ML trust score    |
| **median_trust_score** | DECIMAL(5,2) | Median ML trust score     |

### Evidence Metrics

| Field                  | Type | Description         |
| ---------------------- | ---- | ------------------- |
| **reports_with_photo** | INT  | Reports with photos |
| **reports_with_video** | INT  | Reports with videos |

### Verification Metrics

| Field                           | Type         | Description               |
| ------------------------------- | ------------ | ------------------------- |
| **reports_police_verified**     | INT          | Police verified count     |
| **reports_confirmed**           | INT          | Confirmed as real         |
| **reports_rejected**            | INT          | Rejected as false         |
| **verification_rate**           | DECIMAL(5,2) | Percentage verified       |
| **avg_verification_time_hours** | DECIMAL(8,2) | Average verification time |

### Hotspot Metrics

| Field                 | Type | Description         |
| --------------------- | ---- | ------------------- |
| **active_hotspots**   | INT  | Currently active    |
| **new_hotspots**      | INT  | Newly detected      |
| **resolved_hotspots** | INT  | Resolved today      |
| **critical_hotspots** | INT  | Critical risk level |

### Device Metrics

| Field                        | Type         | Description              |
| ---------------------------- | ------------ | ------------------------ |
| **unique_reporting_devices** | INT          | Unique devices reporting |
| **new_devices**              | INT          | First-time devices       |
| **blocked_devices**          | INT          | Blocked devices          |
| **avg_device_trust_score**   | DECIMAL(5,2) | Average device trust     |

### Response Metrics

| Field                         | Type         | Description             |
| ----------------------------- | ------------ | ----------------------- |
| **reports_assigned**          | INT          | Assigned to officers    |
| **reports_resolved**          | INT          | Resolved today          |
| **avg_resolution_time_hours** | DECIMAL(8,2) | Average resolution time |

### Incident Analysis

| Field                                          | Type      | Description                |
| ---------------------------------------------- | --------- | -------------------------- |
| **incident_type_counts**                       | JSON      | {type_id: count, ...}      |
| **top_incident_type_id** (FK → incident_types) | SMALLINT  | Most common type           |
| **calculated_at**                              | TIMESTAMP | When stats were calculated |

---

## 11.2 incident_type_trends — Weekly Trends

**Meaning:**
Weekly trends by incident type.

**Purpose:**
Trend analysis and early warning detection.

| Field                                      | Type         | Description                    |
| ------------------------------------------ | ------------ | ------------------------------ |
| **trend_id** (PK)                          | INT          | Unique trend record            |
| **incident_type_id** (FK → incident_types) | SMALLINT     | Incident type                  |
| **district_id** (FK → districts)           | TINYINT      | NULL = national                |
| **week_start_date**                        | DATE         | Week start                     |
| **week_end_date**                          | DATE         | Week end                       |
| **year_week**                              | VARCHAR(7)   | YYYY-WW format                 |
| **report_count**                           | INT          | Reports this week              |
| **trusted_count**                          | INT          | Trusted reports                |
| **suspicious_count**                       | INT          | Suspicious reports             |
| **false_count**                            | INT          | False reports                  |
| **avg_trust_score**                        | DECIMAL(5,2) | Average trust score            |
| **police_verified_count**                  | INT          | Police verified                |
| **prev_week_count**                        | INT          | Previous week count            |
| **count_change**                           | INT          | Change from previous           |
| **count_change_pct**                       | DECIMAL(6,2) | Percentage change              |
| **trend_direction**                        | ENUM         | increasing, stable, decreasing |
| **associated_hotspots**                    | INT          | Related hotspots               |
| **calculated_at**                          | TIMESTAMP    | Calculation time               |

---

# SECTION 12: PUBLIC COMMUNITY SAFETY MAP

---

## 12.1 public_safety_zones — Anonymized Public Data

**Meaning:**
Anonymized, aggregated safety data for public map.

**Purpose:**
Privacy-preserving public safety visualization.

| Field                            | Type         | Description                           |
| -------------------------------- | ------------ | ------------------------------------- |
| **zone_id** (PK)                 | INT          | Unique zone identifier                |
| **zone_type**                    | ENUM         | grid, sector, cell, custom            |
| **zone_geometry**                | JSON         | GeoJSON polygon                       |
| **district_id** (FK → districts) | TINYINT      | District reference                    |
| **sector_id** (FK → sectors)     | SMALLINT     | Sector reference                      |
| **cell_id** (FK → cells)         | SMALLINT     | Cell reference                        |
| **grid_row**                     | INT          | Grid row if grid type                 |
| **grid_col**                     | INT          | Grid column if grid type              |
| **grid_size_meters**             | INT          | Grid cell size                        |
| **period_start**                 | DATE         | Period start date                     |
| **period_end**                   | DATE         | Period end date                       |
| **period_type**                  | ENUM         | daily, weekly, monthly                |
| **incident_count**               | INT          | Only shown if >= threshold            |
| **safety_score**                 | DECIMAL(5,2) | 0-100, higher = safer                 |
| **safety_level**                 | ENUM         | safe, moderate, elevated, high_risk   |
| **incident_breakdown**           | JSON         | {category_name: count} - no specifics |
| **top_concern**                  | VARCHAR(100) | Most common category name             |
| **trend_vs_prev_period**         | ENUM         | improving, stable, worsening          |
| **display_color**                | VARCHAR(7)   | Hex color for map                     |
| **is_visible**                   | BOOLEAN      | Show on public map                    |
| **min_display_threshold**        | INT          | Min incidents to show (privacy)       |
| **last_updated**                 | TIMESTAMP    | Last calculation                      |

---

# SECTION 13: SYSTEM CONFIGURATION

---

## 13.1 system_settings — Application Configuration

**Meaning:**
Centralized application configuration settings.

**Purpose:**
Dynamic configuration without code changes.

| Field                              | Type         | Description                                                                  |
| ---------------------------------- | ------------ | ---------------------------------------------------------------------------- |
| **setting_id** (PK)                | INT          | Unique setting ID                                                            |
| **setting_category**               | ENUM         | general, ml, verification, hotspot, notification, security, privacy, display |
| **setting_key**                    | VARCHAR(100) | Setting identifier                                                           |
| **setting_value**                  | JSON         | Setting value                                                                |
| **value_type**                     | ENUM         | string, number, boolean, json, array                                         |
| **display_name**                   | VARCHAR(200) | Human-readable name                                                          |
| **description**                    | TEXT         | Setting description                                                          |
| **validation_rules**               | JSON         | min, max, pattern, etc.                                                      |
| **default_value**                  | JSON         | Default value                                                                |
| **requires_admin**                 | BOOLEAN      | Admin only setting                                                           |
| **is_sensitive**                   | BOOLEAN      | Masked in logs                                                               |
| **is_active**                      | BOOLEAN      | Setting is active                                                            |
| **updated_at**                     | TIMESTAMP    | Last update                                                                  |
| **updated_by** (FK → police_users) | INT          | Who updated                                                                  |

---

# SECTION 14: AUDIT & ACTIVITY LOGGING

---

## 14.1 activity_logs — User Activity Audit

**Meaning:**
Complete audit trail of police user actions.

**Purpose:**
Security monitoring, compliance, and accountability.

| Field                                    | Type         | Description                                                               |
| ---------------------------------------- | ------------ | ------------------------------------------------------------------------- |
| **log_id** (PK)                          | BIGINT       | Unique log ID                                                             |
| **user_id** (FK → police_users)          | INT          | NULL for system actions                                                   |
| **user_type**                            | ENUM         | police, system, api                                                       |
| **action_type**                          | VARCHAR(50)  | Action performed                                                          |
| **action_category**                      | ENUM         | auth, report, hotspot, user_management, settings, ml, data_export, system |
| **action_description**                   | VARCHAR(500) | Action details                                                            |
| **report_id** (FK → incident_reports)    | CHAR(36)     | Related report                                                            |
| **hotspot_id** (FK → hotspots)           | INT          | Related hotspot                                                           |
| **affected_user_id** (FK → police_users) | INT          | User affected                                                             |
| **affected_table**                       | VARCHAR(100) | Table affected                                                            |
| **affected_record_id**                   | VARCHAR(100) | Record affected                                                           |
| **old_values**                           | JSON         | Previous values                                                           |
| **new_values**                           | JSON         | New values                                                                |
| **ip_address**                           | VARCHAR(45)  | Client IP                                                                 |
| **user_agent**                           | TEXT         | Browser/client                                                            |
| **session_id**                           | CHAR(36)     | Session ID                                                                |
| **was_successful**                       | BOOLEAN      | Action succeeded                                                          |
| **failure_reason**                       | TEXT         | Why it failed                                                             |
| **created_at**                           | TIMESTAMP    | When action occurred                                                      |

---

## 14.2 data_change_audit — Data Change Tracking

**Meaning:**
Detailed audit of all data changes (INSERT, UPDATE, DELETE).

**Purpose:**
Complete data change history for compliance.

| Field                              | Type         | Description                  |
| ---------------------------------- | ------------ | ---------------------------- |
| **audit_id** (PK)                  | BIGINT       | Unique audit ID              |
| **table_name**                     | VARCHAR(100) | Table that changed           |
| **record_id**                      | VARCHAR(100) | Record that changed          |
| **operation**                      | ENUM         | INSERT, UPDATE, DELETE       |
| **old_values**                     | JSON         | Previous values              |
| **new_values**                     | JSON         | New values                   |
| **changed_columns**                | JSON         | List of changed columns      |
| **changed_by** (FK → police_users) | INT          | Who made change              |
| **changed_by_type**                | ENUM         | police, system, api, trigger |
| **ip_address**                     | VARCHAR(45)  | Client IP                    |
| **application_context**            | VARCHAR(100) | Which app component          |
| **changed_at**                     | TIMESTAMP    | When changed                 |

---

# SECTION 15: USER FEEDBACK

---

## 15.1 app_feedback — Anonymous App Feedback

**Meaning:**
Anonymous feedback from mobile app users.

**Purpose:**
Collect user feedback for app improvement.

| Field                                         | Type         | Description                                                                                       |
| --------------------------------------------- | ------------ | ------------------------------------------------------------------------------------------------- |
| **feedback_id** (PK)                          | INT          | Unique feedback ID                                                                                |
| **device_id** (FK → devices)                  | CHAR(36)     | Optional device reference                                                                         |
| **feedback_type**                             | ENUM         | bug_report, feature_request, usability_issue, performance_issue, content_issue, compliment, other |
| **feedback_text**                             | TEXT         | Feedback content                                                                                  |
| **rating**                                    | TINYINT      | 1-5 star rating                                                                                   |
| **app_version**                               | VARCHAR(20)  | App version                                                                                       |
| **platform**                                  | ENUM         | android, ios                                                                                      |
| **os_version**                                | VARCHAR(30)  | OS version                                                                                        |
| **screen_name**                               | VARCHAR(100) | Screen where submitted                                                                            |
| **related_report_id** (FK → incident_reports) | CHAR(36)     | Related report                                                                                    |
| **attachment_count**                          | TINYINT      | Screenshots attached                                                                              |
| **is_reviewed**                               | BOOLEAN      | Has been reviewed                                                                                 |
| **reviewed_at**                               | TIMESTAMP    | When reviewed                                                                                     |
| **reviewed_by** (FK → police_users)           | INT          | Who reviewed                                                                                      |
| **review_notes**                              | TEXT         | Review comments                                                                                   |
| **review_status**                             | ENUM         | new, acknowledged, investigating, resolved, wont_fix                                              |
| **requires_followup**                         | BOOLEAN      | Needs follow-up                                                                                   |
| **followup_notes**                            | TEXT         | Follow-up details                                                                                 |
| **created_at**                                | TIMESTAMP    | Submission time                                                                                   |

---

## 15.2 feedback_attachments — Feedback Screenshots

**Meaning:**
Screenshots and files attached to feedback.

**Purpose:**
Visual context for bug reports and issues.

| Field                               | Type         | Description          |
| ----------------------------------- | ------------ | -------------------- |
| **attachment_id** (PK)              | INT          | Unique attachment ID |
| **feedback_id** (FK → app_feedback) | INT          | Parent feedback      |
| **file_name**                       | VARCHAR(255) | Original file name   |
| **file_path**                       | VARCHAR(500) | Storage path         |
| **file_size_bytes**                 | INT          | File size            |
| **mime_type**                       | VARCHAR(100) | MIME type            |
| **uploaded_at**                     | TIMESTAMP    | Upload time          |

---

# SECTION 16: API MANAGEMENT

---

## 16.1 api_keys — API Authentication

**Meaning:**
API keys for external system integrations.

**Purpose:**
Secure API access management.

| Field                                 | Type         | Description                       |
| ------------------------------------- | ------------ | --------------------------------- |
| **key_id** (PK)                       | INT          | Unique key ID                     |
| **key_name**                          | VARCHAR(100) | Key name/description              |
| **key_hash**                          | VARCHAR(255) | Hashed API key                    |
| **key_prefix**                        | VARCHAR(10)  | First 10 chars for identification |
| **owner_user_id** (FK → police_users) | INT          | Key owner                         |
| **owner_description**                 | VARCHAR(255) | Owner details                     |
| **permissions**                       | JSON         | Allowed endpoints/actions         |
| **rate_limit_per_minute**             | INT          | Rate limit                        |
| **allowed_ips**                       | JSON         | IP whitelist                      |
| **allowed_districts**                 | JSON         | Data access restrictions          |
| **is_active**                         | BOOLEAN      | Key is active                     |
| **last_used_at**                      | TIMESTAMP    | Last use time                     |
| **total_requests**                    | BIGINT       | Total API calls                   |
| **created_at**                        | TIMESTAMP    | Key creation                      |
| **expires_at**                        | TIMESTAMP    | Key expiration                    |
| **revoked_at**                        | TIMESTAMP    | If revoked, when                  |
| **revoked_reason**                    | VARCHAR(255) | Why revoked                       |

---

## 16.2 api_request_logs — API Request Logging

**Meaning:**
Logs API requests for monitoring and debugging.

**Purpose:**
API usage tracking and troubleshooting.

| Field                          | Type         | Description                   |
| ------------------------------ | ------------ | ----------------------------- |
| **log_id** (PK)                | BIGINT       | Unique log ID                 |
| **api_key_id** (FK → api_keys) | INT          | API key used                  |
| **endpoint**                   | VARCHAR(255) | API endpoint called           |
| **method**                     | ENUM         | GET, POST, PUT, PATCH, DELETE |
| **request_params**             | JSON         | Request parameters            |
| **response_status**            | INT          | HTTP status code              |
| **response_time_ms**           | INT          | Response time                 |
| **ip_address**                 | VARCHAR(45)  | Client IP                     |
| **user_agent**                 | VARCHAR(500) | Client identifier             |
| **had_error**                  | BOOLEAN      | Error occurred                |
| **error_message**              | TEXT         | Error details                 |
| **requested_at**               | TIMESTAMP    | Request time                  |

---

# RELATIONSHIP SUMMARY

### Geographic Hierarchy

- One province → many districts
- One district → many sectors
- One sector → many cells
- One cell → many villages

### Incident Taxonomy

- One incident_category → many incident_types

### Core Reporting

- One device → many reports
- One device → many device_trust_history records
- One report → many evidence files
- One report → one ML prediction result
- One report → many rule_execution_logs
- One report → zero or one police review

### Hotspot Detection

- One hotspot → many reports
- One report → zero or many hotspots
- hotspot_reports links reports to hotspots (many-to-many)
- One hotspot → many hotspot_history records
- One clustering_run → many hotspots

### Machine Learning

- One ml_model → many ml_predictions
- One ml_model → many ml_training_data records

### Police & Authentication

- One police_user → many police_sessions
- One police_user → many notifications
- One police_user → many activity_logs
- One police_user → many api_keys
- One api_key → many api_request_logs

### Feedback

- One app_feedback → many feedback_attachments

### Verification Rules

- One verification_rule → many rule_execution_logs

---

# DATA FLOWS

## 1. Incident Report Submission Flow

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Mobile    │────►│   Report    │────►│   Evidence  │────►│   Rule      │
│    App      │     │   Created   │     │   Uploaded  │     │   Check     │
└─────────────┘     └─────────────┘     └─────────────┘     └──────┬──────┘
                                                                    │
                    ┌─────────────┐     ┌─────────────┐     ┌──────▼──────┐
                    │   Ready     │◄────│   Trust     │◄────│    ML       │
                    │ for Review  │     │Classification│    │  Scoring    │
                    └──────┬──────┘     └─────────────┘     └─────────────┘
                           │
                    ┌──────▼──────┐     ┌─────────────┐     ┌─────────────┐
                    │   Police    │────►│   Device    │────►│  Training   │
                    │   Review    │     │Score Update │     │    Data     │
                    └─────────────┘     └─────────────┘     └─────────────┘
```

## 2. Hybrid Verification Pipeline

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         HYBRID VERIFICATION                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────┐                                                        │
│  │   Report    │                                                        │
│  │  Received   │                                                        │
│  └──────┬──────┘                                                        │
│         │                                                               │
│         ▼                                                               │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                    STAGE 1: RULE-BASED                          │   │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐            │   │
│  │  │  GPS    │  │ Motion  │  │  Time   │  │Evidence │            │   │
│  │  │ Check   │  │ Check   │  │ Check   │  │ Check   │            │   │
│  │  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘            │   │
│  │       └──────────┬─┴──────────┬─┴──────────┬─┘                 │   │
│  │                  ▼            ▼            ▼                    │   │
│  │              rule_execution_logs                                │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│         │                                                               │
│         ▼ (if passed)                                                   │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                    STAGE 2: ML SCORING                           │   │
│  │  ┌─────────────────┐    ┌─────────────────┐                     │   │
│  │  │ Feature Vector  │───►│   ML Model      │                     │   │
│  │  │   Extraction    │    │  (RandomForest) │                     │   │
│  │  └─────────────────┘    └────────┬────────┘                     │   │
│  │                                  ▼                               │   │
│  │                         ml_predictions                           │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│         │                                                               │
│         ▼                                                               │
│  ┌─────────────┐                                                        │
│  │Trust Class: │  Trusted (≥70) │ Suspicious (40-69) │ False (<40)     │
│  └─────────────┘                                                        │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## 3. Hotspot Detection Flow

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Scheduled  │────►│   Filter    │────►│   DBSCAN    │
│   Trigger   │     │  by Trust   │     │  Clustering │
└─────────────┘     └─────────────┘     └──────┬──────┘
                                               │
┌─────────────┐     ┌─────────────┐     ┌──────▼──────┐
│  Dashboard  │◄────│   Assign    │◄────│   Create    │
│   Display   │     │   Priority  │     │   Hotspots  │
└─────────────┘     └─────────────┘     └─────────────┘
```

## 4. ML Model Retraining Cycle

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Police    │────►│  Training   │────►│   Train     │
│   Reviews   │     │    Data     │     │  New Model  │
└─────────────┘     └─────────────┘     └──────┬──────┘
                                               │
┌─────────────┐     ┌─────────────┐     ┌──────▼──────┐
│   Deploy    │◄────│   A/B Test  │◄────│   Evaluate  │
│   Champion  │     │   Compare   │     │   Metrics   │
└─────────────┘     └─────────────┘     └─────────────┘
```

## 5. Device Trust Score Evolution

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   New       │────►│   Report    │────►│   Police    │
│  Device (50)│     │  Submitted  │     │   Review    │
└─────────────┘     └─────────────┘     └──────┬──────┘
                                               │
                    ┌──────────────────────────┼──────────────────────────┐
                    │                          │                          │
              ┌─────▼─────┐            ┌───────▼───────┐           ┌─────▼─────┐
              │ Confirmed │            │    Needs      │           │  Rejected │
              │   (+10)   │            │Investigation  │           │   (-15)   │
              └─────┬─────┘            └───────┬───────┘           └─────┬─────┘
                    │                          │                          │
                    └──────────────────────────┼──────────────────────────┘
                                               │
                                        ┌──────▼──────┐
                                        │device_trust_│
                                        │  history    │
                                        └─────────────┘
```

## 6. Police Dashboard Notification Flow

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Event     │────►│   Check     │────►│   Create    │
│  Triggered  │     │Jurisdiction │     │Notification │
└─────────────┘     └─────────────┘     └──────┬──────┘
                                               │
┌─────────────┐     ┌─────────────┐     ┌──────▼──────┐
│   Email     │◄────│   Push      │◄────│   Deliver   │
│ (optional)  │     │   Notify    │     │   Channels  │
└─────────────┘     └─────────────┘     └─────────────┘
```

## 7. Public Safety Map Generation

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Trusted    │────►│  Aggregate  │────►│  Apply      │
│  Reports    │     │   by Zone   │     │  Threshold  │
└─────────────┘     └─────────────┘     └──────┬──────┘
                                               │
┌─────────────┐     ┌─────────────┐     ┌──────▼──────┐
│   Public    │◄────│  Calculate  │◄────│  Anonymize  │
│    Map      │     │Safety Score │     │    Data     │
└─────────────┘     └─────────────┘     └─────────────┘

Privacy Rules:
• Minimum 3 incidents per zone
• No individual report details
• Category counts only
• No device information
```

---

# TABLE COUNT SUMMARY

| Section       | Tables | Purpose                         |
| ------------- | ------ | ------------------------------- |
| Core Device   | 2      | Device tracking and history     |
| Geography     | 5      | Rwanda administrative hierarchy |
| Taxonomy      | 2      | Incident classification         |
| Reports       | 1      | Core transaction table          |
| Evidence      | 1      | File metadata storage           |
| Rules Engine  | 2      | Rule-based verification         |
| ML System     | 3      | Machine learning pipeline       |
| Hotspots      | 4      | Cluster detection and tracking  |
| Police Users  | 2      | Authentication and sessions     |
| Notifications | 1      | Alert system                    |
| Analytics     | 2      | Pre-aggregated statistics       |
| Public Map    | 1      | Privacy-preserving public data  |
| System Config | 1      | Application settings            |
| Audit         | 2      | Complete audit trail            |
| Feedback      | 2      | User feedback collection        |
| API           | 2      | External integrations           |

**Total: 33 Tables**
---
