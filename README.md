# Hospital Inpatient Revenue & Bed Efficiency Analysis

## Project Overview
After completing a healthcare data analytics project focused on 
the payer (insurance) side, this project shifts perspectives to 
the **provider (hospital) side**. The goal is to analyze inpatient 
operational data to help hospital management optimize internal 
revenue cycle workflows, reduce inpatient bed bottlenecks, 
and evaluate clinical quality. 

---

## Data Cleaning & Transformation
The raw incoming dataset contained unorganized column fields. 
The initial phase required executing a data definition script 
to standardize the table layout into clean `snake_case` formats:

```sql
ALTER TABLE hcp 
RENAME COLUMN dischargedate TO discharge_date,
RENAME COLUMN treatmentcost TO treatment_cost,
RENAME COLUMN doctorname TO doctor_name,
RENAME COLUMN insuranceprovider TO insurance_provider,
RENAME COLUMN treatmenttype TO treatment_type,
RENAME COLUMN bedtype TO bed_type,
RENAME COLUMN paymentmethod TO payment_method,
RENAME COLUMN satisfactionscore TO satisfaction_score;
Strategic Business Analyses
1. Inpatient Bed Utilization & Flow Optimization
Problem: Inpatient beds are fixed assets. Long recovery bottlenecks
in low-margin areas reduce a hospital's capability to take on
higher-revenue surgical or acute emergencies.

SQL
SELECT 
    bed_type,
    department,
    ROUND(AVG(length_of_stay), 2) AS avg_stay
FROM hcp
GROUP BY bed_type, department
ORDER BY avg_stay DESC;
Core Discovery: General Surgery cases using General beds
and Orthopedics patients inside Semi-Private housing experience
the longest stays (~9 days).

Strategic Recommendation: Hospital leadership should implement
standardized post-operative clinical paths to safely free up
these highly requested bed categories.

2. Revenue Cycle Management (RCM) & Payer Metrics
Problem: The finance department must track the highest ticket
billing sources and determine how various medical insurance companies
handle patient accounts to optimize cash flow stability.

SQL
-- Metric A: Cost distribution across payment modes
SELECT payment_method, ROUND(AVG(treatment_cost), 2) AS avg_cost 
FROM hcp GROUP BY payment_method ORDER BY avg_cost DESC;

-- Metric B: Claims profile across major insurers
SELECT insurance_provider, ROUND(AVG(treatment_cost), 2) AS avg_cost 
FROM hcp GROUP BY insurance_provider ORDER BY avg_cost DESC;
Core Discovery: Accounts cleared via corporate Insurance
settle at the highest average billing amount (~2.10 Lakhs).
Among active health carriers, Star Health commands the
largest average invoice size (~2.05 Lakhs).

Strategic Recommendation: Establish dedicated pre-authorization
billing desks for Star Health to minimize payment rejection rates.

3. Clinical Quality Monitoring & Doctor Scorecards
Problem: Unusually high patient readmissions create unnecessary
operational costs and lower a healthcare organization's regulatory
care scorecard.

SQL
SELECT 
    doctor_name,
    COUNT(patient_id) AS total_patients_treated,
    SUM(CASE WHEN readmitted = 'Yes' THEN 1 ELSE 0 END) AS total_readmissions,
    ROUND((SUM(CASE WHEN readmitted = 'Yes' THEN 1 ELSE 0 END) * 100.0) / COUNT(patient_id), 2) AS readmission_rate_pct,
    ROUND(AVG(satisfaction_score), 1) AS avg_satisfaction
FROM hcp
GROUP BY doctor_name
ORDER BY readmission_rate_pct DESC;
Core Discovery: This dynamic scorecard tracks doctor-specific
outliers showing high patient bounce-back rates paired with
low overall satisfaction ratings.

Strategic Recommendation: Flag extreme statistical outliers
for an internal clinical review board to ensure patients are
not being discharged prematurely.

4. Demographic Analysis & Severity Risk Profiling
Problem: Assessing if severe medical risks correlate directly
with prolonged bed occupancy and significantly higher treatment expenses.

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
Core Discovery: While Severe cases pull in the highest
billings (~2.04 Lakhs), Mild severity cases are unexpectedly
occupying beds for almost identical timeframes (~8.3 days).

Strategic Recommendation: Mild cases are causing unnecessary
bed blocks. The hospital should transition low-risk, mild-severity
cases toward outpatient daycare setups.

Executive Conclusion
By implementing operational improvements such as fast-tracking the
discharge pipelines for low-risk/mild cases and streamlining post-op
care bottlenecks in General Surgery, the facility can significantly
improve its bed turnover velocity. This directly increases patient
intake volume capacity and drives higher bottom-line profitability
for the hospital network.
