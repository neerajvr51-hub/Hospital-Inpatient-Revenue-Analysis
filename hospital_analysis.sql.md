-- ========================================================
-- PROJECT: HOSPITAL INPATIENT REVENUE & BED EFFICIENCY
-- AUTHOR: Niraj Vishwakarma
-- PURPOSE: Provider-side healthcare data analytics portfolio
-- ========================================================

CREATE DATABASE provider_db
USE provider_db

-- STEP 1: Data Cleaning & Column Renaming
-- The raw dataset had merged column names. 
-- I renamed them to standard snake_case for better readability.

ALTER TABLE hcp 
RENAME COLUMN dischargedate TO discharge_date,
RENAME COLUMN treatmentcost TO treatment_cost,
RENAME COLUMN doctorname TO doctor_name,
RENAME COLUMN insuranceprovider TO insurance_provider,
RENAME COLUMN treatmenttype TO treatment_type,
RENAME COLUMN bedtype TO bed_type,
RENAME COLUMN paymentmethod TO payment_method,
RENAME COLUMN satisfactionscore TO satisfaction_score;

ALTER TABLE hcp 
MODIFY COLUMN admission_date date;

ALTER TABLE hcp
MODIFY COLUMN discharge_date date;

SELECT * FROM hcp

-- STEP 2: CORE ANALYSIS

-- Query 1: Operational Efficiency - Bed & Department Stay Duration
SELECT 
    bed_type,
    department,
    ROUND(AVG(length_of_stay), 2) AS avg_stay
FROM hcp
GROUP BY bed_type, department
ORDER BY avg_stay DESC;

-- Query 2: Revenue Cycle - Performance of Different Payment Methods
SELECT 
    payment_method,
    ROUND(AVG(treatment_cost), 2) AS avg_cost
FROM hcp
GROUP BY payment_method
ORDER BY avg_cost DESC;

-- Query 3: Revenue Cycle - Performance of Insurance Providers
SELECT 
    insurance_provider,
    ROUND(AVG(treatment_cost), 2) AS avg_cost
FROM hcp
GROUP BY insurance_provider
ORDER BY avg_cost DESC;

-- Query 4: Clinical Quality - Doctor Scorecard (Readmission & Satisfaction)
SELECT 
    doctor_name,
    COUNT(patient_id) AS total_patients_treated,
    SUM(CASE WHEN readmitted = 'Yes' THEN 1 ELSE 0 END) AS total_readmissions,
    ROUND((SUM(CASE WHEN readmitted = 'Yes' THEN 1 ELSE 0 END) * 100.0) / COUNT(patient_id), 2) AS readmission_rate_pct,
    ROUND(AVG(satisfaction_score), 1) AS avg_satisfaction_score
FROM hcp
GROUP BY doctor_name
ORDER BY readmission_rate_pct DESC;

-- Query 5: Risk Adjustment - Analysis by Patient Age Groups
SELECT
    age_group,
    ROUND(AVG(treatment_cost), 2) AS avg_treatment_cost,
    ROUND(AVG(length_of_stay), 2) AS avg_length_of_stay
FROM hcp
GROUP BY age_group
ORDER BY avg_treatment_cost DESC;

-- Query 6: Risk Adjustment - Analysis by Case Severity
SELECT
    severity,
    ROUND(AVG(treatment_cost), 2) AS avg_treatment_cost,
    ROUND(AVG(length_of_stay), 2) AS avg_length_of_stay
FROM hcp
GROUP BY severity
ORDER BY avg_treatment_cost DESC;


    










