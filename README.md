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

**2. Appointments from today onwards**
```sql
SELECT *
FROM appointments
WHERE appointment_date >= CURDATE()
ORDER BY appointment_date;
```
*Concepts: Date filtering, `CURDATE()`, `ORDER BY`*

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

### Bonus Queries & Further Insights

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

**23. Monthly appointment trend**
```sql
SELECT
    DATE_FORMAT(appointment_date, '%Y-%m') AS month,
    COUNT(*) AS appointment_count
FROM appointments
GROUP BY DATE_FORMAT(appointment_date, '%Y-%m')
ORDER BY month;
```

**24. Most frequent appointment reasons**
```sql
SELECT
    reason,
    COUNT(*) AS appointment_count
FROM appointments
GROUP BY reason
ORDER BY appointment_count DESC;
```

---

## 🧠 SQL Concepts Used

`SELECT` `WHERE` `ORDER BY` `GROUP BY` `HAVING` `JOIN` `LEFT JOIN` `Subqueries` `CTEs` `Window Functions` `DENSE_RANK()` `ROW_NUMBER()` `LAG()` `CASE` `Date Functions` `Aggregate Functions`

---

## 📊 Business Intelligence Dashboard

The SQL results were translated into a Power BI dashboard concept.

**1. Appointment Volume Trend** 

**2. Appointments Reasons**

**3. Hospital with most Appointments**  

**4. Staffing VS Demand by Speciality** 

**5. Busiest Doctors by Appointments** 

**6. Most Prescribed Medications** 

**7. Most Patients by Age** 

**8. Largest Department by bed Capacity**
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

## ▶️ How to Run

**MySQL Workbench**
1. Open MySQL Workbench.
2. Create or select the hospital database.
3. Run the database setup SQL file.
4. Open `hospital_queries.sql`.
5. Execute each query individually.
6. Review the result sets.

**Power BI**
1. Import the prepared hospital dataset.
2. Create the required table relationships.
3. Add KPIs and charts.
4. Add interactive slicers.
5. Create the dashboard pages.
6. Use the SQL results to support business insights.

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

If you found this project useful, feel free to explore the repository.
