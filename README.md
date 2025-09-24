Healthcare Analytics with SQL — Business Intelligence Case Study

Executive Summary:
This repository contains a healthcare analytics solution developed using SQL (MySQL) and Tableau. The project focuses on extracting actionable insights from patient, hospital, and insurance datasets to support business decisions in hospital management and healthcare operations.

Business Objectives
Improve patient care operations through data-driven insights
Identify revenue drivers across hospitals, doctors, and insurance providers
Track patient churn, readmission risk, and insurance coverage gaps
Support management in optimizing billing, admissions, and room utilization



-- number of patients admitted in the hospital
SELECT COUNT(*) AS TotalPatients
FROM Patients;

-- most common Blood Type.
SELECT BloodType, COUNT(*) AS CountBlood
FROM Patients
GROUP BY BloodType
ORDER BY CountBlood DESC
LIMIT 1;

-- SELECT AVG(BillingAmount) AS AvgBilling FROM Billing;
SELECT AVG(BillingAmount) AS AvgBilling
FROM Billing;

-- average age of patients per Medical Condition
SELECT m.MedicalCondition, AVG(p.Age) AS AvgAge
FROM Patients p
JOIN MedicalRecords m ON p.PatientID = m.PatientID
GROUP BY m.MedicalCondition;

-- Admission Type is most frequent
SELECT AdmissionType, COUNT(*) AS CountType
FROM MedicalRecords
GROUP BY AdmissionType
ORDER BY CountType DESC
LIMIT 1;

-- number of patients admitted per Doctor.
SELECT DoctorName, COUNT(*) AS PatientCount
FROM Doctors
GROUP BY DoctorName;
-- Room Number has the highest number of admitted patients
SELECT RoomNumber, COUNT(*) AS CountPatients
FROM Billing
GROUP BY RoomNumber
ORDER BY CountPatients DESC
LIMIT 1;

-- top 5 Medical Conditions leading to admission
SELECT MedicalCondition, COUNT(*) AS CountCondition
FROM MedicalRecords
GROUP BY MedicalCondition
ORDER BY CountCondition DESC
LIMIT 5;

-- How many patients were discharged on the same day they were admitted?
SELECT COUNT(*) AS SameDayDischarges
FROM MedicalRecords
WHERE DateOfAdmission = DischargeDate;

-- Doctor generated the highest total billing
SELECT d.DoctorName, SUM(b.BillingAmount) AS TotalBilling
FROM Doctors d
JOIN Billing b ON d.PatientID = b.PatientID
GROUP BY d.DoctorName
ORDER BY TotalBilling DESC
LIMIT 1;

-- detect duplicate patient admission records
SELECT PatientID, COUNT(*) AS CountRecords
FROM MedicalRecords
GROUP BY PatientID
HAVING COUNT(*) > 1;

-- total number of patients admitted each month
SELECT 
    DATE_FORMAT(date_of_admission, '%Y-%m') AS month,
    COUNT(patient_id) AS total_admissions
FROM admissions
GROUP BY month
ORDER BY month;

-- month with the highest admissions.
SELECT 
    DATE_FORMAT(date_of_admission, '%Y-%m') AS month,
    COUNT(patient_id) AS total_admissions
FROM admissions
GROUP BY month
ORDER BY total_admissions DESC
LIMIT 1;

-- average length of stay per patient.
SELECT 
    patient_id,
    AVG(DATEDIFF(discharge_date, date_of_admission)) AS avg_stay_days
FROM admissions
WHERE discharge_date IS NOT NULL
GROUP BY patient_id;

-- number of patients still admitted.
SELECT COUNT(*) AS currently_admitted
FROM admissions
WHERE discharge_date IS NULL;

-- average number of patients admitted per doctor each month.
SELECT 
    doctor,
    DATE_FORMAT(date_of_admission, '%Y-%m') AS month,
    COUNT(patient_id) / COUNT(DISTINCT DATE_FORMAT(date_of_admission, '%Y-%m')) AS avg_patients_per_month
FROM admissions
GROUP BY doctor, month;

-- hospital with the highest number of admissions in the last 6 months.
SELECT 
    hospital,
    COUNT(patient_id) AS total_admissions
FROM admissions
WHERE date_of_admission >= DATE_SUB(CURDATE(), INTERVAL 6 MONTH)
GROUP BY hospital
ORDER BY total_admissions DESC
LIMIT 1;

-- average billing amount per hospital per quarter.
SELECT 
    hospital,
    QUARTER(date_of_admission) AS quarter,
    AVG(billing_amount) AS avg_billing
FROM billing
JOIN admissions USING(patient_id)
GROUP BY hospital, quarter;

-- doctors who admitted more than 50 patients in a single year.
SELECT 
    doctor,
    YEAR(date_of_admission) AS year,
    COUNT(patient_id) AS total_patients
FROM admissions
GROUP BY doctor, year
HAVING total_patients > 50;

-- monthly total billing amount covered by each insurance provider.
SELECT 
    insurance_provider,
    DATE_FORMAT(date_of_admission, '%Y-%m') AS month,
    SUM(billing_amount) AS total_billing
FROM billing
JOIN admissions USING(patient_id)
GROUP BY insurance_provider, month
ORDER BY month;

-- insurance provider with the highest claim settlement in the last year.
SELECT 
    insurance_provider,
    SUM(billing_amount) AS total_claims
FROM billing
JOIN admissions USING(patient_id)
WHERE YEAR(date_of_admission) = YEAR(CURDATE()) - 1
GROUP BY insurance_provider
ORDER BY total_claims DESC
LIMIT 1;

-- average billing amount trend over time (month by month).
SELECT 
    DATE_FORMAT(date_of_admission, '%Y-%m') AS month,
    AVG(billing_amount) AS avg_billing
FROM billing
JOIN admissions USING(patient_id)
GROUP BY month
ORDER BY month;

-- patients whose billing amounts increased in every admission compared to the last.
SELECT patient_id
FROM (
    SELECT 
        patient_id,
        billing_amount,
        LAG(billing_amount) OVER (PARTITION BY patient_id ORDER BY date_of_admission) AS prev_bill
    FROM billing
    JOIN admissions USING(patient_id)
) t
WHERE prev_bill IS NOT NULL
GROUP BY patient_id
HAVING MIN(billing_amount > prev_bill) = 1;

-- medical condition with the fastest growing admission trend year-over-year.
SELECT 
    medical_condition,
    (COUNT(CASE WHEN YEAR(date_of_admission) = YEAR(CURDATE()) - 1 THEN patient_id END)) AS last_year,
    (COUNT(CASE WHEN YEAR(date_of_admission) = YEAR(CURDATE()) THEN patient_id END)) AS this_year,
    ((COUNT(CASE WHEN YEAR(date_of_admission) = YEAR(CURDATE()) THEN patient_id END)) -
     (COUNT(CASE WHEN YEAR(date_of_admission) = YEAR(CURDATE()) - 1 THEN patient_id END))) AS growth
FROM admissions
GROUP BY medical_condition
ORDER BY growth DESC
LIMIT 1;



