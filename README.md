# 🏥 Hospital Management & Business Intelligence Dashboard

![SQL](https://img.shields.io/badge/SQL-Analysis-blue)
![MySQL](https://img.shields.io/badge/MySQL-Database-4479A1)
![Power BI](https://img.shields.io/badge/Power%20BI-Dashboard-F2C811)

## 📌 Project Overview

This project analyzes a hospital management database using **MySQL, SQL, and Business Intelligence techniques**, transforming raw operational data into insights that support better hospital management decisions.

The analysis covers patients, doctors, hospitals, appointments, departments, rooms, medications, and prescriptions, using SQL filtering, sorting, aggregation, joins, subqueries, CTEs, window functions, and conditional logic (`CASE`).

---

## 🎯 Objectives

The project was developed to:

- Analyze hospital appointment patterns
- Evaluate doctor workloads
- Compare hospital activity
- Analyze room and department capacity
- Identify emergency and consultation trends
- Analyze medication and prescription patterns
- Understand patient movement across specialties
- Generate actionable business recommendations
- Build a foundation for Power BI reporting

---

## 🛠️ Tech Stack

| Technology | Purpose |
|---|---|
| MySQL | Database management |
| MySQL Workbench | SQL development and testing |
| SQL | Data analysis |
| Power BI | Interactive visualization |
| HTML/CSS | Dashboard and business presentation |
| Excel | Data preparation |

---

## 🗃️ Database Entities

- `hospitals`
- `doctors`
- `patients`
- `appointments`
- `departments`
- `rooms`
- `medications`
- `prescriptions`

---

## 🔎 SQL Analysis

The project contains 24 SQL queries covering core and bonus business requirements.

### Core Queries

**1. Room numbers and capacities in the Neurology department**
```sql
SELECT r.room_no, r.capacity
FROM rooms r
JOIN departments d
    ON r.department_id = d.department_id
WHERE d.name = 'Neurology'
ORDER BY r.room_no;
```
*Concepts: `JOIN`, `WHERE`, `ORDER BY`*

**Result**

<img width="110" height="120" alt="Image" src="https://github.com/user-attachments/assets/d06cfaf3-8dc0-4cd4-b83e-ef5f7cbb3de5" />

There are **7 rooms** in the Neurology Department.

**Room 303** has the highest capacity, with a capacity of **4 patients**.

**Key Insight**

Room 303 represents the largest available room in the Neurology Department and may be useful when prioritizing room allocation for larger patient groups.

---

**2. Appointments from today onwards**
```sql
SELECT *
FROM appointments
WHERE appointment_date >= CURDATE()
ORDER BY appointment_date;
```
*Concepts: Date filtering, `CURDATE()`, `ORDER BY`*
**Result**

There is **only 1 appointment from today onwards**.

| Date | Appointment Type | Doctor | Specialty |
|---|---|---|---|
| 2026-09-03 | Consultation | Robert Chan | Emergency Medicine |

**Key Insight**

Only one upcoming appointment was identified from the current date, September 3, 2026.

---

**3. Patients with more than 3 appointments**
```sql
SELECT
    p.patient_id,
    p.name,
    COUNT(a.appointment_id) AS appointment_count
FROM patients p
JOIN appointments a
    ON p.patient_id = a.patient_id
GROUP BY p.patient_id, p.name
HAVING COUNT(a.appointment_id) > 3
ORDER BY appointment_count DESC;
```
*Concepts: `JOIN`, `COUNT()`, `GROUP BY`, `HAVING`*

**Result**

There are **no patients with more than 3 appointments**.

The maximum number of appointments for any patient is **3**.

**Key Insight**

The dataset does not contain patients with unusually high appointment frequency. The maximum recorded appointment count is three.

---

**4. Appointments with patient and doctor details**
```sql
SELECT
    a.appointment_id,
    a.appointment_date,
    a.reason,
    p.name AS patient_name,
    d.name AS doctor_name,
    d.specialty
FROM appointments a
JOIN patients p
    ON a.patient_id = p.patient_id
JOIN doctors d
    ON a.doctor_id = d.doctor_id
ORDER BY a.appointment_date;
```
*Concepts: Multiple `JOIN`s, aliases, sorting*

**Result**

<img width="364" height="224" alt="Image" src="https://github.com/user-attachments/assets/23b3da92-7237-4678-aba7-d9d66f856fb2" />

Further table results are present in .sql file.

---
**5. Patients with appointments in August 2026**
```sql
SELECT
    p.name,
    p.phone,
    a.appointment_date
FROM patients p
JOIN appointments a
    ON p.patient_id = a.patient_id
WHERE a.appointment_date >= '2026-08-01'
  AND a.appointment_date < '2026-09-01'
ORDER BY a.appointment_date;
```
*Concepts: Date range filtering, `JOIN`*

**Result**

<img width="303" height="179" alt="Image" src="https://github.com/user-attachments/assets/371202b7-0505-4784-a093-934e55571dd0" />

There are **13 patients** who have appointments in August.

**Key Insight**

August shows notable patient appointment activity, with 13 patients scheduled during the month.

---


**6. Medications related to pain or infection**
```sql
SELECT
    medication_id,
    name,
    description
FROM medications
WHERE LOWER(description) LIKE '%pain%'
   OR LOWER(description) LIKE '%infection%'
ORDER BY name;
```
*Concepts: `LIKE`, `LOWER()`, conditional filtering*

**Result**

<img width="346" height="167" alt="Image" src="https://github.com/user-attachments/assets/e95a1637-d817-4cef-ae14-c9e1b9721b82" />

There are **11 medications** containing either:

- `"pain"`
- `"infection"`

in their description.

**Key Insight**

Keyword-based medication analysis can help identify medications associated with common treatment categories such as pain management and infection treatment.

---

**7. Doctors who have not prescribed any medication**
```sql
SELECT
    d.doctor_id,
    d.name,
    d.specialty
FROM doctors d
LEFT JOIN prescriptions pr
    ON d.doctor_id = pr.doctor_id
WHERE pr.prescription_id IS NULL
ORDER BY d.name;
```
*Concepts: `LEFT JOIN`, `IS NULL`*

**Result**

<img width="224" height="78" alt="Image" src="https://github.com/user-attachments/assets/bc1737a1-aff5-4a88-93aa-5b11b0333d8f" />

There is **only 1 cardiologist** who has not prescribed any medication.

**Doctor:** Lisa Murphy

**Key Insight**

Dr. Lisa Murphy is the only cardiologist in the dataset without a recorded medication prescription.

---

**8. Patients prescribed more than one medication**
```sql
SELECT
    p.patient_id,
    p.name,
    COUNT(DISTINCT pr.medication_id) AS medication_count
FROM patients p
JOIN prescriptions pr
    ON p.patient_id = pr.patient_id
GROUP BY p.patient_id, p.name
HAVING COUNT(DISTINCT pr.medication_id) > 1
ORDER BY medication_count DESC;
```
*Concepts: `COUNT(DISTINCT)`, `GROUP BY`, `HAVING`*

**Result**

<img width="229" height="185" alt="Image" src="https://github.com/user-attachments/assets/90d43e64-4b6f-496b-b4f2-3d48bdb65579" />

There are **12 patients** who have been prescribed more than one medication.

**Taylor Johnson** has the highest count, with **4 medications**.

**Key Insight**

Taylor Johnson has the highest number of prescribed medications among patients with multiple prescriptions.

---


**9. Monthly appointments by hospital**
```sql
SELECT
    h.name AS hospital_name,
    DATE_FORMAT(a.appointment_date, '%Y-%m') AS appointment_month,
    COUNT(*) AS appointment_count
FROM appointments a
JOIN doctors d
    ON a.doctor_id = d.doctor_id
JOIN hospitals h
    ON d.hospital_id = h.hospital_id
GROUP BY
    h.hospital_id,
    h.name,
    DATE_FORMAT(a.appointment_date, '%Y-%m')
ORDER BY appointment_month, hospital_name;
```
*Concepts: Date functions, aggregation, multiple joins*

**Results**

**Green Valley Medical Center**
- Highest appointments: **February 2026**
- Monthly count: **3**
- Overall appointments: **12**

**Riverside General Hospital**
- Highest appointments: **February 2026 — 3**
- Highest appointments: **July 2026 — 3**
- Overall appointments: **12**

**Cedar Grove Community Hospital**
- Highest appointments: **May 2026 — 2**
- Overall appointments: **4**

**Northwind Regional Medical Center**
- Highest appointments: **March 2026 — 3**
- Overall appointments: **10**

**Lakeside Healthcare Institute**
There is **1 appointment** in each of the following months:
- January
- March
- April
- August
Overall appointments: **4**

**Harmony Hill Hospital**
- Highest appointments: **January 2026**
- Monthly count: **4**
- Overall appointments: **13**

**Silver Oak Medical Center**
- Highest appointments: **August 2026**
- Monthly count: **4**
- Overall appointments: **14**
⭐ **Highest overall appointment count among the hospitals analyzed.**

**Maplewood General Hospital**

Overall appointments from January 2026 to today:
**7 appointments**

**Pinecrest Medical Pavilion**
- Highest appointments: **April 2026**
- Monthly count: **3**
- Overall appointments: **11**

**Oceanview Health Center**

Overall appointments from January 2026 to today:
**12 appointments**

---


**10. Rank doctors by appointments within each hospital**
```sql
WITH doctor_counts AS (
    SELECT
        d.doctor_id,
        d.name AS doctor_name,
        d.hospital_id,
        h.name AS hospital_name,
        COUNT(a.appointment_id) AS appointment_count
    FROM doctors d
    JOIN hospitals h
        ON d.hospital_id = h.hospital_id
    LEFT JOIN appointments a
        ON d.doctor_id = a.doctor_id
    GROUP BY
        d.doctor_id,
        d.name,
        d.hospital_id,
        h.name
)
SELECT
    *,
    DENSE_RANK() OVER (
        PARTITION BY hospital_id
        ORDER BY appointment_count DESC
    ) AS hospital_rank
FROM doctor_counts
ORDER BY hospital_name, hospital_rank;
```
*Concepts: CTE, `DENSE_RANK()`, window functions*

**Result**

<img width="1125" height="1223" alt="Image" src="https://github.com/user-attachments/assets/fd4eebf4-692a-4857-8a2c-d90d5fc2db88" />

### Key Insight

The hospital-level analysis shows differences in appointment volume and monthly activity.

**Silver Oak Medical Center** has the highest overall number of appointments with **14**, followed by **Harmony Hill Hospital with 13**.

---

**11. Doctors with zero appointments**
```sql
SELECT
    d.doctor_id,
    d.name,
    d.specialty,
    h.name AS hospital_name
FROM doctors d
JOIN hospitals h
    ON d.hospital_id = h.hospital_id
LEFT JOIN appointments a
    ON d.doctor_id = a.doctor_id
WHERE a.appointment_id IS NULL
ORDER BY hospital_name, d.name;
```
*Concepts: `LEFT JOIN`, `IS NULL`*

**Result**

There are **no doctors with zero appointments**.

**Key Insight**

Every doctor in the dataset has at least one recorded appointment.

---

**12. Last two appointments for each patient**
```sql
WITH ranked_appointments AS (
    SELECT
        a.*,
        ROW_NUMBER() OVER (
            PARTITION BY patient_id
            ORDER BY appointment_date DESC, appointment_id DESC
        ) AS rn
    FROM appointments a
)
SELECT
    p.name AS patient_name,
    ra.appointment_date,
    ra.reason
FROM ranked_appointments ra
JOIN patients p
    ON ra.patient_id = p.patient_id
WHERE ra.rn <= 2
ORDER BY patient_name, ra.appointment_date DESC;
```
*Concepts: CTE, `ROW_NUMBER()`, `PARTITION BY`*

**Result**

The analysis compares the number of days between a patient's previous appointment and current appointment.

**Largest Difference**

**Erica Jimenez**

- Appointment gap: **195 days**

**Smallest Difference**

**Sean Green**

- Appointment gap: **3 days**

**Key Insight**

Erica Jimenez has the largest gap between appointments, while Sean Green has the shortest gap and all other patients fall in between

---

**13. Emergency appointments by age group**

Age criteria: 18 or below → Pediatric · 19–64 → Adult · 65+ → Geriatric
```sql
SELECT
    p.name AS patient_name,
    p.dob,
    a.appointment_date,
    CASE
        WHEN TIMESTAMPDIFF(YEAR, p.dob, a.appointment_date) <= 18 THEN 'Pediatric'
        WHEN TIMESTAMPDIFF(YEAR, p.dob, a.appointment_date) BETWEEN 19 AND 64 THEN 'Adult'
        ELSE 'Geriatric'
    END AS age_group
FROM appointments a
JOIN patients p
    ON a.patient_id = p.patient_id
WHERE a.reason = 'Emergency'
ORDER BY a.appointment_date;
```
*Concepts: `CASE`, `TIMESTAMPDIFF()`, joins*

**Result**

<img width="286" height="257" alt="Image" src="https://github.com/user-attachments/assets/ee35a3b1-a864-4bbf-b61f-29c10e864d73" />

---

**14. Cardiology consultations by age group**
```sql
SELECT
    CASE
        WHEN TIMESTAMPDIFF(YEAR, p.dob, a.appointment_date) <= 18 THEN 'Pediatric'
        WHEN TIMESTAMPDIFF(YEAR, p.dob, a.appointment_date) BETWEEN 19 AND 64 THEN 'Adult'
        ELSE 'Geriatric'
    END AS age_group,
    COUNT(*) AS consultation_count
FROM appointments a
JOIN patients p
    ON a.patient_id = p.patient_id
JOIN doctors d
    ON a.doctor_id = d.doctor_id
WHERE a.reason = 'Consultation'
  AND LOWER(d.specialty) = 'cardiology'
GROUP BY age_group;
```
*Concepts: `CASE`, joins, aggregation*

**Result**

<img width="137" height="61" alt="Image" src="https://github.com/user-attachments/assets/aceeeecb-f407-46fc-88eb-23d3b2d46d57" />

---

**15. Third most frequently prescribed medication(s)**
```sql
WITH medication_counts AS (
    SELECT
        m.medication_id,
        m.name,
        COUNT(pr.prescription_id) AS prescription_count
    FROM medications m
    JOIN prescriptions pr
        ON m.medication_id = pr.medication_id
    GROUP BY m.medication_id, m.name
),
ranked AS (
    SELECT
        *,
        DENSE_RANK() OVER (
            ORDER BY prescription_count DESC
        ) AS frequency_rank
    FROM medication_counts
)
SELECT
    name,
    prescription_count
FROM ranked
WHERE frequency_rank = 3;
```
*Concepts: CTEs, `DENSE_RANK()`, aggregation, ties*

**Result**
Most frequently prescribed medications are:

<img width="242" height="101" alt="Image" src="https://github.com/user-attachments/assets/79dcef40-65d0-445f-91c3-b2c5abf39c41" />

---

**16. Hospitals with the lowest doctor count**
```sql
WITH hospital_doctors AS (
    SELECT
        h.hospital_id,
        h.name AS hospital_name,
        COUNT(d.doctor_id) AS doctor_count
    FROM hospitals h
    LEFT JOIN doctors d
        ON h.hospital_id = d.hospital_id
    GROUP BY h.hospital_id, h.name
)
SELECT
    hospital_name,
    doctor_count
FROM hospital_doctors
WHERE doctor_count = (
    SELECT MIN(doctor_count)
    FROM hospital_doctors
);
```
*Concepts: CTE, subquery, `MIN()`*

**Result**
There are 3 hospitals with lowest doctor count.

<img width="272" height="64" alt="Image" src="https://github.com/user-attachments/assets/1682cff9-d76f-4a19-9c36-63aaeb3fe467" />

---

**17. Hospital(s) with the most Cardiology rooms**
```sql
WITH cardiology_rooms AS (
    SELECT
        h.hospital_id,
        h.name AS hospital_name,
        COUNT(r.room_id) AS room_count
    FROM hospitals h
    JOIN departments d
        ON h.hospital_id = d.hospital_id
    JOIN rooms r
        ON d.department_id = r.department_id
    WHERE LOWER(d.name) = 'cardiology'
    GROUP BY h.hospital_id, h.name
)
SELECT
    hospital_name,
    room_count
FROM cardiology_rooms
WHERE room_count = (
    SELECT MAX(room_count)
    FROM cardiology_rooms
);
```
*Concepts: CTE, `MAX()`, joins, subquery*

**Results**

<img width="252" height="53" alt="Image" src="https://github.com/user-attachments/assets/166b1f99-fcb5-4572-b8da-3d82237793d2" />

---

**18. Days between appointments for returning patients**
```sql
WITH ordered_appointments AS (
    SELECT
        patient_id,
        appointment_date,
        LAG(appointment_date) OVER (
            PARTITION BY patient_id
            ORDER BY appointment_date
        ) AS previous_appointment_date
    FROM appointments
)
SELECT
    p.name AS patient_name,
    previous_appointment_date,
    appointment_date,
    DATEDIFF(appointment_date, previous_appointment_date) AS days_between
FROM ordered_appointments o
JOIN patients p
    ON o.patient_id = p.patient_id
WHERE previous_appointment_date IS NOT NULL
ORDER BY patient_name, appointment_date;
```
*Concepts: `LAG()`, `DATEDIFF()`, window functions, CTE*

---

**19. Patients who visited multiple specialties**
```sql
SELECT
    p.patient_id,
    p.name,
    COUNT(DISTINCT d.specialty) AS specialty_count
FROM patients p
JOIN appointments a
    ON p.patient_id = a.patient_id
JOIN doctors d
    ON a.doctor_id = d.doctor_id
GROUP BY p.patient_id, p.name
HAVING COUNT(DISTINCT d.specialty) > 1
ORDER BY specialty_count DESC;
```
*Concepts: Multiple joins, `COUNT(DISTINCT)`, `HAVING`*

---

**20. Second-largest department room capacity in each hospital**
```sql
WITH department_capacity AS (
    SELECT
        h.hospital_id,
        h.name AS hospital_name,
        d.department_id,
        d.name AS department_name,
        COALESCE(SUM(r.capacity), 0) AS total_room_capacity
    FROM hospitals h
    JOIN departments d
        ON h.hospital_id = d.hospital_id
    LEFT JOIN rooms r
        ON d.department_id = r.department_id
    GROUP BY
        h.hospital_id,
        h.name,
        d.department_id,
        d.name
),
ranked_capacity AS (
    SELECT
        *,
        DENSE_RANK() OVER (
            PARTITION BY hospital_id
            ORDER BY total_room_capacity DESC
        ) AS capacity_rank
    FROM department_capacity
)
SELECT
    hospital_name,
    department_name,
    total_room_capacity
FROM ranked_capacity
WHERE capacity_rank = 2
ORDER BY hospital_name;
```
*Concepts: CTE, `SUM()`, `COALESCE()`, `DENSE_RANK()`, window functions*

---

**21. Specialist distribution by hospital**
```sql
SELECT
    h.name AS hospital_name,
    d.specialty,
    COUNT(*) AS specialist_count
FROM doctors d
JOIN hospitals h
    ON d.hospital_id = h.hospital_id
GROUP BY h.hospital_id, h.name, d.specialty
ORDER BY specialist_count DESC;
```

---

**22. Average room capacity per department and hospital**
```sql
SELECT
    h.name AS hospital_name,
    d.name AS department_name,
    AVG(r.capacity) AS average_room_capacity
FROM rooms r
JOIN departments d
    ON r.department_id = d.department_id
JOIN hospitals h
    ON d.hospital_id = h.hospital_id
GROUP BY h.name, d.name
ORDER BY average_room_capacity DESC;
```
---

**23. Monthly appointment trend**
```sql
SELECT
    DATE_FORMAT(appointment_date, '%Y-%m') AS month,
    COUNT(*) AS appointment_count
FROM appointments
GROUP BY DATE_FORMAT(appointment_date, '%Y-%m')
ORDER BY month;
```
**Result**
<img width="146" height="140" alt="Image" src="https://github.com/user-attachments/assets/426687a6-7019-48a8-8688-5e69ea4a9c28" />

<img width="1124" height="1225" alt="Image" src="https://github.com/user-attachments/assets/fb82df39-4a17-45b8-a73b-3133581b7ff8" />

---

**24. Most frequent appointment reasons**
```sql
SELECT
    reason,
    COUNT(*) AS appointment_count
FROM appointments
GROUP BY reason
ORDER BY appointment_count DESC;
```
**Results**
<img width="167" height="83" alt="Image" src="https://github.com/user-attachments/assets/b679d33f-29e4-434b-bfd8-6afa003f1bfe" />

---

## 🧠 SQL Concepts Used

`SELECT` `WHERE` `ORDER BY` `GROUP BY` `HAVING` `JOIN` `LEFT JOIN` `Subqueries` `CTEs` `Window Functions` `DENSE_RANK()` `ROW_NUMBER()` `LAG()` `CASE` `Date Functions` `Aggregate Functions`

---

## 📊 Data Visualisation 

The SQL results were translated into a Power BI dashboard concept.

**1. Appointment Volume Trend** 

<img width="1124" height="1225" alt="Image" src="https://github.com/user-attachments/assets/fb82df39-4a17-45b8-a73b-3133581b7ff8" />

**2. Appointments Reasons**

<img width="1133" height="1203" alt="Image" src="https://github.com/user-attachments/assets/de04fe6a-ba1e-423f-abd6-4f9db2c51906" />

**3. Hospital with most Appointments**  

<img width="1129" height="1203" alt="Image" src="https://github.com/user-attachments/assets/3ceb6635-30f0-48d9-8ba4-4bab7b36465b" />



**4. Staffing VS Demand by Speciality** 

<img width="1129" height="1201" alt="Image" src="https://github.com/user-attachments/assets/fa558697-a514-47cc-ba00-eca0653aefc8" />


**5. Busiest Doctors by Appointments** 

<img width="1125" height="1223" alt="Image" src="https://github.com/user-attachments/assets/689e201e-8689-405d-96fe-827f86717c18" />


**6. Most Prescribed Medications** 

<img width="1129" height="1207" alt="Image" src="https://github.com/user-attachments/assets/bd74f14e-a962-4db5-9c6e-f788f534c1f6" />


**7. Most Patients by Age** 

<img width="1108" height="1230" alt="Image" src="https://github.com/user-attachments/assets/d2f71d59-bed0-4b20-baa6-3d5fb29fffb0" />

**8. Largest Department by bed Capacity**

<img width="1131" height="1213" alt="Image" src="https://github.com/user-attachments/assets/dd3c6e5e-dc64-474e-bc9c-12947ed3a395" />

---

## 💡 Business Solutions

**Doctor Workload Management** — Monitor appointment distribution and identify doctors with unusually high or low workloads.

**Hospital Capacity Planning** — Compare appointment demand against available rooms and department capacity to identify resource gaps.

**Emergency Management** — Maintain dedicated capacity for emergency cases so urgent patients don't compete with routine appointments.

**Patient Care Coordination** — Identify patients visiting multiple specialties and support better coordination between departments.

**Medication Management** — Use prescription trends to improve pharmacy inventory planning and reduce shortages or unnecessary stock.

**Resource Allocation** — Allocate doctors, rooms, and other resources according to actual demand rather than distributing resources equally.

**Predictive Planning** — Use historical appointment trends to forecast future staffing, room, and medication requirements.

---

## 🔄 Project Workflow

```
Hospital Database
       ↓
MySQL Workbench
       ↓
20+ SQL Queries
       ↓
Data Analysis
       ↓
Power BI Visualization
       ↓
Business Insights
       ↓
Management Recommendations
```

---

## 🧠 Skills Demonstrated

**SQL:** Joins, CTEs, Subqueries, Window Functions, Aggregation, CASE Statements, Date Analysis

**Data Analytics:** KPI Development, Trend Analysis, Patient Segmentation, Workload Analysis, Capacity Analysis

**Business Intelligence:** Power BI, Interactive Dashboards, Slicers, Data Visualization, Data Storytelling

**Business Analysis:** Problem Solving, Resource Planning, Operational Analysis, Decision Support

---

## 📁 Repository Structure

```
hospital-management-bi/
│
├── README.md
├── hospital_queries.sql
│
├── powerbi/
│   ├── Hospital_BI_PowerBI_Ready.xlsx
│   ├── Hospital_BI_Clean_Pastel_Theme.json
│   └── Hospital_PowerBI_Page_Design_Guide.txt
│
└── dashboard/
    └── hospital_management_interactive_BI_dashboard.html
```

---

## ✅ Project Outcome

This project demonstrates an end-to-end analytics workflow:

**Database → SQL → Analysis → Visualization → Insights → Business Solutions**

The final solution transforms raw hospital data into a practical Business Intelligence and decision-support system.

---

## 🔑 Key Takeaway

> The goal was not only to write SQL queries, but to understand what the results mean and how they can help hospital management make better decisions.

---

## 👩‍💻 Author

**Fatima Malik**
Data Analyst
SQL | Data Analytics | Business Intelligence

[LinkedIn](https://linkedin.com/in/fatima-malik-5a8832418) · fatimamaliknaeem123@gmail.com

---

