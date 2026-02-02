# 🏥 Hospital Emergency Room Dashboard — Power BI

An interactive Power BI dashboard that analyzes **9,216 patient emergency room visits** spanning April 2019 to October 2020. The report provides hospital administrators with actionable insights into patient wait times, satisfaction scores, demographic breakdowns, and department referral patterns.

<img width="1429" height="798" alt="Hospital_Dashboard_PowerBI" src="https://github.com/user-attachments/assets/1b17f4e3-8d84-49b4-ba61-cac41c044df4" />

---

## 📌 Table of Contents

- [Project Overview](#-project-overview)
- [Objectives](#-objectives)
- [Dataset](#-dataset)
- [Data Model](#-data-model)
- [DAX Measures & Calculated Columns](#-dax-measures--calculated-columns)
- [Dashboard Walkthrough](#-dashboard-walkthrough)
- [Key Insights](#-key-insights)
- [Project Structure](#-project-structure)
- [Tools Used](#-tools-used)
- [How to Use](#-how-to-use)
- [Future Improvements](#-future-improvements)

---

## 🔎 Project Overview

Hospital emergency rooms face constant pressure to reduce patient wait times while maintaining care quality. This project takes a raw CSV dataset of ER visits and transforms it into a single-page Power BI dashboard that enables stakeholders to quickly identify bottlenecks, track trends, and compare outcomes across demographics.

The entire workflow covers data cleaning, data modeling, DAX calculations, custom background design, and interactive report building.

---

## 🎯 Objectives

The dashboard was built to answer six core questions defined in the project requirements document:

1. **Average Wait Time** — Evaluate the average waiting time of patients across the ER.
2. **Monthly Visit Trends** — Track patient visits on a monthly basis to identify seasonal patterns.
3. **Department Referral Volume** — Measure total visits by department referral to understand downstream demand.
4. **Age Group Breakdown** — Segment patient visits by age group (Infant → Adult).
5. **Satisfaction by Demographics** — Determine average patient satisfaction by age group and race.
6. **Wait Time by Demographics** — Determine average wait time by age group and race.

---

## 📊 Dataset

**Source file:** `Hospital_ER.csv`
**Records:** 9,216 rows
**Time span:** April 1, 2019 — October 30, 2020

### Raw Columns

| Column | Data Type | Description |
|---|---|---|
| `date` | Datetime | Timestamp of the patient's ER visit |
| `patient_id` | String | Unique patient identifier (format: `XXX-XX-XXXX`) |
| `patient_gender` | String | Gender — `M` (Male), `F` (Female), `NC` (Not Confirmed) |
| `patient_age` | Integer | Patient age in years (range: 1–79) |
| `patient_sat_score` | Float | Satisfaction score on a 0–10 scale (nullable — 75.10% unrated) |
| `patient_first_inital` | String | First initial of the patient's first name |
| `patient_last_name` | String | Patient's last name |
| `patient_race` | String | Self-reported race/ethnicity |
| `patient_admin_flag` | Boolean | `true` = Administrative appointment, `false` = Non-administrative |
| `patient_waittime` | Integer | Wait time in minutes (range: 10–60 min) |
| `department_referral` | String | Department the patient was referred to (or `None`) |

### Key Data Statistics

| Metric | Value |
|---|---|
| Total Patients | 9,216 |
| Avg. Satisfaction Score | 5.47 (of those who rated) |
| Service Not Rated | 75.10% |
| Avg. Wait Time | 35.26 minutes |
| Administrative Appointments | 50.04% |
| Non-Administrative Appointments | 49.96% |
| Male / Female / Unknown | 51.1% / 48.7% / 0.3% |

### Race Distribution

African American, Asian, White, Native American/Alaska Native, Pacific Islander, Two or More Races, and Declined to Identify.

### Department Referrals

| Department | Patient Count |
|---|---|
| None (Walk-in) | 5,400 |
| General Practice | 1,840 |
| Orthopedics | 995 |
| Physiotherapy | 276 |
| Cardiology | 248 |
| Neurology | 193 |
| Gastroenterology | 178 |
| Renal | 86 |

---

## 🗂 Data Model

The Power BI data model consists of **five tables** connected in a star-schema-style layout:

<img width="1433" height="741" alt="Screenshot 2025-09-18 002356" src="https://github.com/user-attachments/assets/4f1f4dc5-bbe1-4377-b067-74c61736525c" />

### Tables & Relationships

```
┌──────────────────┐     ┌─────────────────────────┐     ┌────────────┐
│   Calculations    │     │    Patients Dataset      │────▶│    Date    │
│                  │     │                         │ *:1  │            │
│  • Column1       │     │  • Age Buckets (calc)   │     │  • Date    │
└──────────────────┘     │  • Age Groups (calc)    │     │  • Month   │
                         │  • date                 │     │  • MonthNum│
┌──────────────────┐     │  • department_referral   │     │  • Weekday │
│    Parameter      │     │  • Full Name (calc)     │     │  • Weektype│
│                  │     │  • Momemt (calc)         │     └────────────┘
│  • Parameter     │     │  • patient_admin_flag    │
│  • Parameter     │     │  • patient_age           │     ┌────────────┐
│    Fields        │     │  • patient_gender        │     │   Table    │
│  • Parameter     │     │  • patient_id            │     │            │
│    Order         │     │  • patient_race          │     │ • @Total_  │
└──────────────────┘     │  • patient_sat_score     │     │   Patients │
                         │  • patient_waittime      │     │ • Month    │
                         └─────────────────────────┘     └────────────┘
```

**Patients Dataset → Date** — Many-to-one relationship on the date field, enabling time intelligence calculations.

**Patients Dataset** — The central fact table, enriched with several calculated columns (Age Buckets, Age Groups, Full Name, Momemt).

**Date** — A dedicated date dimension table with derived fields for Month, MonthNum, Weekday, and Weektype (Weekday vs Weekend).

**Parameter** — A disconnected parameter table used to drive the dynamic heatmap toggle (switch between Avg. Satisfaction and Avg. Wait Time).

**Table** — A helper table containing `@Total_Patients` measure and Month for specific visual calculations.

**Calculations** — A utility table used for supplementary calculations.

---

## 📐 DAX Measures & Calculated Columns

### Age Buckets (Calculated Column)

Segments patient ages into 10-year bucket ranges for the heatmap matrix:

<img width="570" height="214" alt="Screenshot 2025-09-18 002605" src="https://github.com/user-attachments/assets/907e1f91-fba2-4d6e-8761-4ecd984c7b9d" />


```dax
Age Buckets =
    SWITCH(
        TRUE(),
        'Patients Dataset'[patient_age] <= 10, "0-10",
        'Patients Dataset'[patient_age] <= 20, "11-20",
        'Patients Dataset'[patient_age] <= 30, "21-30",
        'Patients Dataset'[patient_age] <= 40, "31-40",
        'Patients Dataset'[patient_age] <= 50, "41-50",
        'Patients Dataset'[patient_age] <= 60, "51-60",
        'Patients Dataset'[patient_age] <= 70, "61-70",
        "70+"
    )
```

### Age Groups (Calculated Column)

Categorizes patients into life-stage groups for the horizontal bar chart:

<img width="527" height="305" alt="Screenshot 2025-09-18 002629" src="https://github.com/user-attachments/assets/28b1c5fd-7e47-4fbe-9505-2d7bf4db1e94" />

```dax
Age Groups =
    VAR _PatientAge = 'Patients Dataset'[patient_age]
    RETURN
    IF(
        _PatientAge <= 2, "Infant",
        IF(
            _PatientAge <= 6, "Early Childhood",
            IF(
                _PatientAge <= 12, "Middle Childhood",
                IF(
                    _PatientAge <= 18, "Teenager",
                    "Adult"
                )
            )
        )
    )
```

### Values Max Point (Measure)

Identifies the minimum and maximum data points on the yearly visits line chart for conditional formatting (marker highlighting):

<img width="724" height="322" alt="Screenshot 2025-09-18 002442" src="https://github.com/user-attachments/assets/1e0f637d-d77b-4344-a6b7-6c03a44c0338" />

```dax
Values Max Point (year) =
    VAR _PatientTable =
        CALCULATETABLE(
            ADDCOLUMNS(
                SUMMARIZE('Date', 'Date'[Year]),
                "@Total_Patients", [Total Patients]
            ),
            ALLSELECTED()
        )
    VAR _MinValu = MINX(_PatientTable, [@Total_Patients])
    VAR _MaxValu = MAXX(_PatientTable, [@Total_Patients])
    VAR _TotalPatients = CALCULATE([Total Patients], _PatientTable)
    RETURN
    SWITCH(
        TRUE(),
        _TotalPatients = _MinValu, [Total Patients],
        _TotalPatients = _MaxValu, [Total Patients]
    )
```

### Other Key Measures

- **Total Patients** — `COUNTROWS('Patients Dataset')`
- **Avg. Satisfaction** — `AVERAGE('Patients Dataset'[patient_sat_score])`
- **Avg. Wait Time** — `AVERAGE('Patients Dataset'[patient_waittime])`
- **Momemt** — Extracts AM/PM from the visit timestamp for the time-of-day slicer.
- **Full Name** — Concatenates `patient_first_initial` and `patient_last_name`.

---

## 📋 Dashboard Walkthrough

The dashboard is organized into a **3-row grid layout** built on a custom-designed background template:

![Editable Background_New](https://github.com/user-attachments/assets/d1eef764-0b71-46c5-8549-0adb209ec2dd)

### Row 1 — KPI Cards & Filters

| Visual | Description |
|---|---|
| **Total Patients Visits** | Big number card showing 9,216 with admin vs non-admin split (50.04% / 49.96%) |
| **Avg Satisfaction** | Score card — 5.47 out of 10, with sparkline trend |
| **Service Not Rated** | Percentage card — 75.10% of patients did not leave a satisfaction rating |
| **Avg Wait Time** | Score card — 35.26 minutes average, with sparkline trend |
| **Momemt Slicer** | AM / PM toggle to filter visits by time of day |
| **Referred vs Walk-in** | Shows referred patients (41.41%) vs walk-in patients (58.59%) |

### Row 2 — Trends & Distribution

| Visual | Description |
|---|---|
| **Patients by Weektype** | Clustered bar chart comparing Weekday vs Weekend visit volume |
| **Total Patients by Age Groups** | Horizontal bar chart — Adult, Middle Childhood, Teenager, Early Childhood, Infant |
| **Total Patients Visits by Year** | Line chart with conditional data-point markers (min/max highlighted) — 4,338 (2019) → 4,878 (2020) |
| **Total Patients by department_referral** | Horizontal bar chart showing referral distribution across 8 departments |

### Row 3 — Demographics & Heatmap

| Visual | Description |
|---|---|
| **Total Patients Visits** | Area chart showing monthly patient volume (Jan: 431 → peak Jul: 1,024) |
| **Gender Breakdown** | Donut/icon display — Male 51.1%, Female 48.7%, Unknown 0.3% |
| **Dynamic Heatmap** | Matrix with patient_race as rows and Age Buckets as columns, colored by a **Parameter slicer** that toggles between Avg. Satisfaction and Avg. Wait Time. Darkest green = lowest value. |

---

## 💡 Key Insights

1. **Patient volume grew 12.4% year-over-year** — from 4,338 visits in 2019 to 4,878 in 2020, suggesting increasing ER demand.

2. **75% of patients did not rate the service** — this massive gap in feedback data limits satisfaction analysis and signals a need for better survey collection processes.

3. **Walk-in patients dominate at 58.59%** — the majority of ER visits are not referred from other departments, indicating the ER serves as a primary entry point.

4. **Wait times are nearly uniform** across demographics — the 35.26-minute average shows little variation by race or age, which may suggest consistent (but potentially improvable) triage processes.

5. **July is the peak month** with 1,024 visits, while January is the lowest at 431 — this seasonality could inform staffing and resource allocation.

6. **Adults account for the largest age group** by a significant margin, followed by Middle Childhood and Teenagers.

7. **Weekend vs weekday volume is roughly balanced**, with weekdays showing slightly higher volume — this suggests the ER sees consistent demand regardless of the day.

---

## 📂 Project Structure

```
Hospital-ER-Dashboard/
│
├── Hospital_ER.csv                        # Raw dataset (9,216 records)
├── Hosp_data_dashboard.pbix               # Power BI dashboard file
├── Patients_Emergency_Room_Report.docx    # Project requirements document
│
├── Hospital_Dashboard_PowerBI.png         # Final dashboard screenshot
├── Dashboard_Rough.png                    # Dashboard draft/iteration
│
├── Editable_Background.jpg               # Custom dashboard background (v1)
├── Editable_Background_New.jpg            # Custom dashboard background (v2)
│
├── Screenshot_20250918_002356.png         # Data model view
├── Screenshot_20250918_002442.png         # DAX: Values Max Point measure
├── Screenshot_20250918_002605.png         # DAX: Age Buckets calculated column
├── Screenshot_20250918_002629.png         # DAX: Age Groups calculated column
│
└── README.md                              # This file
```

---

## 🛠 Tools Used

| Tool | Purpose |
|---|---|
| **Microsoft Power BI Desktop** | Data modeling, DAX calculations, report building, and interactive visualizations |
| **Power Query (M Language)** | Data import, cleaning, and transformation within Power BI |
| **DAX (Data Analysis Expressions)** | Calculated columns (Age Buckets, Age Groups), measures (Total Patients, Avg Satisfaction, Max Point), and parameter-driven dynamic visuals |
| **Custom Background Design** | Dashboard canvas background designed externally and imported as a wallpaper image to create a polished, structured layout |
| **Microsoft Word** | Project requirements documentation |

---

## 🚀 How to Use

1. **Clone the repository**
   ```bash
   git clone https://github.com/<your-username>/Hospital-ER-Dashboard.git
   ```

2. **Open the dashboard**
   - Launch **Power BI Desktop** (free download from [Microsoft](https://powerbi.microsoft.com/desktop/)).
   - Open `Hosp_data_dashboard.pbix`.

3. **Refresh the data**
   - If the CSV path has changed, go to **Home → Transform Data** and update the source file path to point to `Hospital_ER.csv`.
   - Click **Close & Apply**.

4. **Interact with the dashboard**
   - Use the **AM / PM slicer** to filter by time of day.
   - Use the **Parameter dropdown** on the heatmap to toggle between Avg. Satisfaction and Avg. Wait Time views.
   - Click on any bar, data point, or chart element to cross-filter the entire report.

---

## 🔮 Future Improvements

- **Improve satisfaction survey capture** — the 75% unrated gap is the biggest data quality issue to address.
- **Add a second page** for detailed drill-through by individual department referral.
- **Integrate real-time data** via Power BI Dataflows or a SQL database connection.
- **Add predictive analytics** using Python/R visuals to forecast monthly visit volumes.
- **Include patient outcome data** to correlate wait times with clinical results.

---

> **Built with** Power BI | **Data:** 9,216 ER patient records | **Period:** 2019–2020
