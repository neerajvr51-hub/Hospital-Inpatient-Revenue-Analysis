Hospital Inpatient Revenue & Bed Efficiency Analysis
Project Overview
After working on a healthcare project from the payer (insurance) side to detect fraud, I wanted to understand how things work on the provider (hospital) side. In this project, I analyzed inpatient hospital data to help the hospital management optimize their internal revenue, check bed utilization, and track doctor performance metrics.

As a fresher data analyst, my goal was to look beyond basic patient counts and find real operational bottlenecks that impact a hospital's daily business and quality of care.

Data Cleaning & Schema Transformation
The raw dataset came with unorganized, merged column names (e.g., dischargedate, treatmentcost). To make the database easy to read and standard for analysis, my first step was to transform the schema using SQL data definition commands:

SQL
ALTER TABLE hcp 
RENAME COLUMN dischargedate TO discharge_date,
RENAME COLUMN treatmentcost TO treatment_cost,
RENAME COLUMN doctorname TO doctor_name,
RENAME COLUMN insuranceprovider TO insurance_provider,
RENAME COLUMN treatmenttype TO treatment_type,
RENAME COLUMN bedtype TO bed_type,
RENAME COLUMN paymentmethod TO payment_method,
RENAME COLUMN satisfactionscore TO satisfaction_score;
Core Business Problems & Analysis
1. Inpatient Bed Utilization Analysis
Problem Statement: Beds are a limited resource in a hospital. If certain departments occupy beds for too long, the hospital cannot take new patients, leading to a loss in potential revenue.

SQL Query:

SQL
SELECT 
    bed_type,
    department,
    ROUND(AVG(length_of_stay), 2) AS avg_stay
FROM hcp
GROUP BY bed_type, department
ORDER BY avg_stay DESC;
Key Insight:

General Surgery patients in General beds and Orthopedics patients in Semi-Private beds have the highest average length of stay (~9 days).

Recommendation: The management should investigate why surgery patients are occupying general beds for so long. Speeding up discharge paperwork or post-op checkups can free up these beds faster for new admissions.

2. Revenue Cycle Management (RCM) & Payer Analysis
Problem Statement: The finance team needs to track which payment paths generate the highest bills and how different insurance companies are performing against patient costs.

SQL Query:

SQL
-- Track average treatment cost by payment method
SELECT 
    payment_method,
    ROUND(AVG(treatment_cost), 2) AS avg_cost
FROM hcp
GROUP BY payment_method
ORDER BY avg_cost DESC;

-- Track average treatment cost by insurance provider
SELECT 
    insurance_provider,
    ROUND(AVG(treatment_cost), 2) AS avg_cost
FROM hcp
GROUP BY insurance_provider
ORDER BY avg_cost DESC;
Key Insight:

Patients using Insurance have the highest average bill amount (~2.10 Lakhs), whereas Online payments have the lowest (~1.93 Lakhs). Among insurance companies, Star Health handles the highest average claim amounts (~2.05 Lakhs).

Recommendation: Since insurance holds the highest ticket size, the billing team must ensure seamless pre-authorization workflows with Star Health to avoid cash flow delays.

3. Clinical Quality & Doctor Scorecard
Problem Statement: High patient readmissions hurt a hospital's reputation and clinical rating. We need to identify if any specific doctors have unusually high readmission rates.

SQL Query:

SQL
SELECT 
    doctor_name,
    COUNT(patient_id) AS total_patients_treated,
    SUM(CASE WHEN readmitted = 'Yes' THEN 1 ELSE 0 END) AS total_readmissions,
    ROUND((SUM(CASE WHEN readmitted = 'Yes' THEN 1 ELSE 0 END) * 100.0) / COUNT(patient_id), 2) AS readmission_rate_pct,
    ROUND(AVG(satisfaction_score), 1) AS avg_satisfaction_score
FROM hcp
GROUP BY doctor_name
ORDER BY readmission_rate_pct DESC;
Key Insight:

This analysis provides a clear view of which clinical teams face high return rates.

Recommendation: Doctors with high readmission rates and low satisfaction scores should be flagged for an internal clinical review to see if patients are being discharged prematurely without proper recovery guidelines.

4. Patient Demographics & Severity Risk Profiling
Problem Statement: Does patient risk (severity) directly correlate with higher resource usage (length of stay and costs)?

SQL Query:

SQL
-- Analysis by Patient Age Groups
SELECT
    age_group,
    ROUND(AVG(treatment_cost), 2) AS avg_treatment_cost,
    ROUND(AVG(length_of_stay), 2) AS avg_length_of_stay
FROM hcp
GROUP BY age_group
ORDER BY avg_treatment_cost DESC;

-- Analysis by Case Severity
SELECT
    severity,
    ROUND(AVG(treatment_cost), 2) AS avg_treatment_cost,
    ROUND(AVG(length_of_stay), 2) AS avg_length_of_stay
FROM hcp
GROUP BY severity
ORDER BY avg_treatment_cost DESC;
Key Insight:

Severe cases naturally cost the highest (~2.04 Lakhs) compared to Mild cases (~1.95 Lakhs). However, surprisingly, the average length of stay is almost identical across all levels (~8.2 to 8.5 days).

Recommendation: Mild cases are occupying hospital beds just as long as critical, severe cases. The hospital should look into setting up a fast-track discharge process or day-care options for mild cases to reduce unnecessary bed occupancy.

Final Project Summary (Conclusion)
By shifting the perspective from Payer to Provider, this project successfully highlights where the hospital is operationally efficient and where it is losing money. By fixing the discharge delays for mild severity patients and managing high-stay departments like General Surgery, the hospital can improve its bed turnover rate, take in more patients, and ultimately increase overall profitability.
