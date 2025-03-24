
# Cardiovascular Risk Prediction Database: Tables and Annotations

This document outlines the schema of the cardiovascular disease prediction database, including all tables used for storing clinicians, patients, prediction models, risk scores, and output visualisations.

---

## 1. Clinician Table
```sql
-- Create the Clinician table
CREATE TABLE `Clinician` (
    `clinician_id` INT(11) NOT NULL AUTO_INCREMENT,  -- Unique identifier for each clinician
    `name` VARCHAR(255) NOT NULL,
    `email` VARCHAR(255) NOT NULL,  -- Unique email used for login or reference
    `password` VARCHAR(255) NOT NULL,  -- Password for authentication

    PRIMARY KEY (`clinician_id`),
    UNIQUE KEY (`email`)  -- Ensures no duplicate accounts
);
```
Each clinician is uniquely identified and secured. Email is enforced to be unique to prevent duplicate registrations.

---

## 2. Patient Table
```sql
-- Create the Patient table
CREATE TABLE `Patient` (
    `patient_id` INT(11) NOT NULL AUTO_INCREMENT,
    `name` VARCHAR(255) NOT NULL,
    `age` INT(11) NOT NULL,
    `clinician_id` INT(11) DEFAULT NULL,  -- Links the patient to their assigned clinician

    PRIMARY KEY (`patient_id`),
    KEY (`clinician_id`),  -- Index to improve JOIN performance
    FOREIGN KEY (`clinician_id`) REFERENCES `Clinician`(`clinician_id`)
);
```
Each patient record optionally links to a clinician, supporting multi-user environments.

---

## 3. Model Table
```sql
-- Create the Model table
CREATE TABLE `Model` (
    `model_id` INT(11) NOT NULL AUTO_INCREMENT,
    `model_name` VARCHAR(255) NOT NULL,  -- E.g. 'ResNet-50', 'DenseNet-121'
    `model_type` VARCHAR(100) NOT NULL,  -- E.g. 'CNN', 'XGBoost', 'Logistic Regression'

    PRIMARY KEY (`model_id`)
);
```
Stores metadata about ML models used in the risk prediction pipeline.

---

## 4. Risk Table
```sql
-- Create the Risk table
CREATE TABLE `Risk` (
    `risk_id` INT(11) NOT NULL AUTO_INCREMENT,
    `patient_id` INT(11) NOT NULL,
    `model_id` INT(11) NOT NULL,
    `risk_score` DECIMAL(5,2) NOT NULL,  -- Risk prediction output (0.00 to 100.00)
    `prediction_date` TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    PRIMARY KEY (`risk_id`),
    KEY (`patient_id`),
    KEY (`model_id`),
    FOREIGN KEY (`patient_id`) REFERENCES `Patient`(`patient_id`),
    FOREIGN KEY (`model_id`) REFERENCES `Model`(`model_id`)
);
```
Tracks all prediction outputs per patient/model combination. Scores are timestamped and stored with precision.

---

## 5. Output Table
```sql
-- Create the Output table
CREATE TABLE `Output` (
    `output_id` INT(11) NOT NULL AUTO_INCREMENT,
    `risk_id` INT(11) NOT NULL,  -- Links to a specific risk prediction
    `plot_type` VARCHAR(100) NOT NULL,  -- E.g. 'ROC Curve', 'Confusion Matrix'
    `generated_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    PRIMARY KEY (`output_id`),
    KEY (`risk_id`),
    FOREIGN KEY (`risk_id`) REFERENCES `Risk`(`risk_id`)
);
```
Stores visual output metadata generated after model inference — e.g., ROC curves, heatmaps etc.

---

## Notes
- All primary keys are `AUTO_INCREMENT` integers for easy reference.
- Timestamp fields (`prediction_date`, `generated_at`) allow for auditing model performance over time.
- Relationships are clearly defined using foreign keys — ensuring referential integrity across patients, clinicians, models, and predictions.
- The design supports future extensions such as uploading actual image outputs or prediction reports.
