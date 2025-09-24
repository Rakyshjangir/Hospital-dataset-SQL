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



