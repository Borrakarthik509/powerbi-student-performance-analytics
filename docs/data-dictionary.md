# 📖 Data Dictionary

[← Back to README](../README.md)

> Complete column definitions, data types, and table descriptions for every table in the Beyond Grades — Student Performance Dashboard data model.

---

## Table of Contents

- [roll numbers (Dimension)](#table-roll-numbers-dimension)
- [Bio (Dimension)](#table-bio-dimension)
- [sem wise cgpa data (Fact — Unpivoted)](#table-sem-wise-cgpa-data-fact--unpivoted)
- [sem wise attendance data (Fact — Unpivoted)](#table-sem-wise-attendance-data-fact--unpivoted)
- [sem wise credits (Fact — Unpivoted)](#table-sem-wise-credits-fact--unpivoted)
- [sem wise avg credits (Reference/Calculated)](#table-sem-wise-avg-credits-referencecalculated)
- [sem_attendance (Staging/Lookup)](#table-sem_attendance-staginglookup)
- [sem_credits (Staging/Lookup)](#table-sem_credits-staginglookup)
- [Sem cgpa perf (Staging/Lookup)](#table-sem-cgpa-perf-staginglookup)
- [Student score (Calculated)](#table-student-score-calculated)
- [StatusSteps (Config/Lookup)](#table-statussteps-configlookup)
- [table (Measure Island)](#table-table-measure-island)

---

## Table: roll numbers (Dimension)

Slicer source for student roll number lookup. Powers the primary student selector — when a user picks a roll number, all other visuals filter to that student's data.

| Column | Data Type | Description | Example Values |
|--------|-----------|-------------|----------------|
| `Roll number` | Text | Unique student roll number identifier | `"23A91A0509"`, `"23A91A0512"` |

---

## Table: Bio (Dimension)

Student biographical and identity data. Provides the profile card content (photo, name, contact) that appears when a student is selected.

| Column | Data Type | Description | Example Values |
|--------|-----------|-------------|----------------|
| `Roll number` | Text | Foreign key — joins to `roll numbers[Roll number]` | `"23A91A0509"` |
| `Name` | Text | Full name of the student | `"Karthik Reddy"` |
| `Gender` | Text | Student gender | `"Male"`, `"Female"` |
| `College` | Text | College or institution name | `"VNRVJIET"` |
| `Branch` | Text | Academic branch / department | `"CSE"`, `"ECE"`, `"IT"` |
| `Email` | Text | Student email address | `"student@college.edu"` |
| `Phone` | Text | Contact phone number | `"9876543210"` |
| `Photo URL` | Text | URL to student profile photo (rendered via Image visual) | `"https://..."` |

---

## Table: sem wise cgpa data (Fact — Unpivoted)

Semester CGPA values in unpivoted (tall) format, powering the CGPA line chart. Each row represents one student-semester combination.

| Column | Data Type | Description | Example Values |
|--------|-----------|-------------|----------------|
| `sem name` | Text | Semester label used as x-axis category | `"Sem 1"`, `"Sem 2"`, … `"Sem 8"` |
| `Value` | Decimal | CGPA value for the semester (scale 0–10) | `7.68`, `8.12`, `9.01` |

> **Why unpivoted?** The raw source data stores CGPA as wide columns (`Sem1_CGPA`, `Sem2_CGPA`, …). Unpivoting into `sem name` / `Value` pairs enables a single line chart visual to plot all semesters on a shared x-axis without hard-coding column references — this is more maintainable and adapts automatically if future semesters are added.

---

## Table: sem wise attendance data (Fact — Unpivoted)

Semester attendance percentages in unpivoted (tall) format, powering the attendance trend line chart.

| Column | Data Type | Description | Example Values |
|--------|-----------|-------------|----------------|
| `Attribute` | Text | Semester label used as x-axis category | `"Sem 1"`, `"Sem 2"`, … `"Sem 8"` |
| `Value` | Decimal | Attendance percentage for the semester | `89.5`, `92.3`, `75.0` |

---

## Table: sem wise credits (Fact — Unpivoted)

Semester credit values in unpivoted (tall) format, used for credit accumulation analysis.

| Column | Data Type | Description | Example Values |
|--------|-----------|-------------|----------------|
| `Attribute` | Text | Semester identifier | `"Sem 1"`, `"Sem 2"`, … `"Sem 8"` |
| `Value` | Decimal | Credit value earned in the semester | `18`, `20`, `22` |

---

## Table: sem wise avg credits (Reference/Calculated)

Pre-calculated average credits per semester across the student population. Contains 8 columns (one per semester) used as benchmark reference values for comparing individual credit performance against cohort averages.

| Column | Data Type | Description |
|--------|-----------|-------------|
| `Sem 1` – `Sem 8` | Decimal | Average credits earned by the cohort in each semester |

---

## Table: sem_attendance (Staging/Lookup)

Attendance reference table with 8 columns. Serves as a staging layer between raw data and the unpivoted `sem wise attendance data` table, providing source attendance values per semester.

| Column | Data Type | Description |
|--------|-----------|-------------|
| 8 semester columns | Decimal | Raw attendance percentages per semester (pre-unpivot staging) |

---

## Table: sem_credits (Staging/Lookup)

Credits reference table with 11 columns. Provides the source data for credit analysis including per-semester credit counts and derived credit metrics.

| Column | Data Type | Description |
|--------|-----------|-------------|
| 11 columns | Various | Semester-level credit data and derived credit metrics (pre-unpivot staging) |

---

## Table: Sem cgpa perf (Staging/Lookup)

CGPA performance classification table with 7 columns. Categorises students into performance tiers based on their semester CGPA values.

| Column | Data Type | Description |
|--------|-----------|-------------|
| 7 columns | Various | CGPA thresholds and corresponding performance category labels |

---

## Table: Student score (Calculated)

Performance score computation table with 7 columns. Combines CGPA, attendance, credits, and backlogs into a single weighted performance score used to assign medal/status tiers.

| Column | Data Type | Description |
|--------|-----------|-------------|
| 7 columns | Various | Component scores, weights, and the final composite performance score |

---

## Table: StatusSteps (Config/Lookup)

Defines the thresholds for student status/medal assignments. Each row maps a score range to a status tier.

| Column | Data Type | Description | Example Values |
|--------|-----------|-------------|----------------|
| Status tier | Text | Medal / tier label | `"Novice"`, `"Silver"`, `"Gold"` |
| Threshold columns | Decimal | Min/max score boundaries for each tier | `0`–`60`, `60`–`80`, `80`–`100` |

> **Design note:** Externalising tier thresholds into a config table (rather than hard-coding them in DAX) allows non-technical stakeholders to adjust classification boundaries without modifying measure logic.

---

## Table: table (Measure Island)

A disconnected table containing **all 25+ DAX measures** — the single source of truth for every calculation displayed in the dashboard.

| Measure | Return Type | Description |
|---------|-------------|-------------|
| See [DAX Measure Reference](dax-reference.md) for the complete list | — | — |

> **Why a measure island?** Storing all measures in a single disconnected table keeps each source table's field list clean (only raw columns), centralises all business logic in one location, and simplifies future refactoring. With 25+ measures spanning identity, performance, semester analysis, and HTML renderers, this pattern is essential for maintaining an organised model.

---

## Relationships Overview

| From (Dimension/Source) | To (Target) | Join Column | Direction |
|-------------------------|-------------|-------------|-----------|
| `roll numbers` | `Bio` | `Roll number` | Single |
| `roll numbers` | `sem wise cgpa data` | Roll number key | Single |
| `roll numbers` | `sem wise attendance data` | Roll number key | Single |
| `roll numbers` | `sem wise credits` | Roll number key | Single |
| `Student score` | `StatusSteps` | Score / threshold | Single |
| `table` | *(disconnected)* | — | — |

> All dimension-to-fact joins use **single-direction** filtering. The `roll numbers` table acts as the primary slicer dimension — selecting a roll number cascades filters to all fact tables, ensuring every visual updates to reflect the selected student's data.
