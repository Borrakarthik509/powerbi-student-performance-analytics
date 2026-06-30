# 📐 DAX Measure Reference

[← Back to README](../README.md)

> Complete reference for all DAX measures in the Beyond Grades — Student Performance Dashboard, including business context, HTML renderers, and recommended enhancements.

---

## Table of Contents

- [Identity / Bio Measures](#identity--bio-measures)
- [Performance Measures](#performance-measures)
- [Semester Min/Max Measures](#semester-minmax-measures)
- [HTML Renderer Measures](#html-renderer-measures)
- [Recommended Next Measures](#recommended-next-measures)

---

All explicit measures are stored in the `table` — a disconnected measure island that separates business logic from raw data columns and keeps the field list clean in Report view.

---

## Identity / Bio Measures

These measures extract the selected student's biographical data using the `SELECTEDVALUE` pattern, enabling profile card visuals to display context-aware information.

---

### `Student_Name`

| Property | Value |
|----------|-------|
| **Table** | `table` |
| **Return Type** | Text |
| **Used In** | Profile card — student name display |

```dax
Student_Name = SELECTEDVALUE(Bio[Name])
```

**Business purpose:** Returns the full name of the currently selected student. Drives the profile card heading and personalises the entire dashboard experience for the selected roll number.

---

### `Student_email`

| Property | Value |
|----------|-------|
| **Table** | `table` |
| **Return Type** | Text |
| **Used In** | Profile card — contact details |

```dax
Student_email = SELECTEDVALUE(Bio[Email])
```

**Business purpose:** Returns the email address of the selected student, displayed in the profile contact section.

---

### `Student_phonenum`

| Property | Value |
|----------|-------|
| **Table** | `table` |
| **Return Type** | Text |
| **Used In** | Profile card — contact details |

```dax
Student_phonenum = SELECTEDVALUE(Bio[Phone])
```

**Business purpose:** Returns the phone number of the selected student for quick contact reference.

---

### `Student_gender`

| Property | Value |
|----------|-------|
| **Table** | `table` |
| **Return Type** | Text |
| **Used In** | Profile card — demographic display |

```dax
Student_gender = SELECTEDVALUE(Bio[Gender])
```

**Business purpose:** Returns the gender of the selected student. Used in the profile card and potentially for demographic filtering.

---

### `Student_college`

| Property | Value |
|----------|-------|
| **Table** | `table` |
| **Return Type** | Text |
| **Used In** | Profile card — institution display |

```dax
Student_college = SELECTEDVALUE(Bio[College])
```

**Business purpose:** Returns the college/institution name. Useful in multi-institution datasets to identify the student's academic home.

---

### `Student_branch`

| Property | Value |
|----------|-------|
| **Table** | `table` |
| **Return Type** | Text |
| **Used In** | Profile card — department display |

```dax
Student_branch = SELECTEDVALUE(Bio[Branch])
```

**Business purpose:** Returns the academic branch (CSE, ECE, IT, etc.) of the selected student.

---

## Performance Measures

Core academic performance metrics that power the KPI cards and score computations.

---

### `Student_cgpa`

| Property | Value |
|----------|-------|
| **Table** | `table` |
| **Return Type** | Decimal |
| **Used In** | CGPA KPI card, CGPA badge |
| **Example Value** | `7.68` |

```dax
Student_cgpa = 
VAR SelectedStudent = SELECTEDVALUE('roll numbers'[Roll number])
RETURN
    IF(
        ISBLANK(SelectedStudent),
        BLANK(),
        AVERAGE('sem wise cgpa data'[Value])
    )
```

**Business purpose:** Returns the cumulative/current CGPA of the selected student. This is the primary academic performance indicator, displayed prominently in the KPI strip and used as an input to the composite performance score.

---

### `Student_attendance`

| Property | Value |
|----------|-------|
| **Table** | `table` |
| **Return Type** | Decimal |
| **Used In** | Attendance KPI card, Attendance badge |
| **Example Value** | `89.5` |

```dax
Student_attendance = 
VAR SelectedStudent = SELECTEDVALUE('roll numbers'[Roll number])
RETURN
    IF(
        ISBLANK(SelectedStudent),
        BLANK(),
        AVERAGE('sem wise attendance data'[Value])
    )
```

**Business purpose:** Returns the overall attendance percentage. Attendance below institutional thresholds (typically 75%) triggers visual warnings via the Attendance Badge HTML measure.

---

### `Student_performance_score`

| Property | Value |
|----------|-------|
| **Table** | `table` |
| **Return Type** | Decimal |
| **Used In** | Performance score card, Medal assignment |
| **Example Value** | `88.77` |

```dax
Student_performance_score = 
VAR SelectedStudent = SELECTEDVALUE('roll numbers'[Roll number])
RETURN
    IF(
        ISBLANK(SelectedStudent),
        BLANK(),
        // Weighted composite calculation: 
        // 50% CGPA normalized score, 30% Attendance percentage, 20% Subject clearance percentage.
        // Penalized by 5 points for every active backlog.
        VAR CGPAScore = ( [Student_cgpa] / 10 ) * 100
        VAR AttendanceScore = [Student_attendance]
        VAR TotalSub = [Total Subjects]
        VAR ClearedSub = [Student_subjectscleared]
        VAR ClearanceScore = DIVIDE(ClearedSub, TotalSub, 0) * 100
        VAR RawScore = (CGPAScore * 0.50) + (AttendanceScore * 0.30) + (ClearanceScore * 0.20)
        VAR BacklogPenalty = [Student_backlogs] * 5
        VAR FinalScore = RawScore - BacklogPenalty
        RETURN
            MAX(0, MIN(100, FinalScore))
    )
```

**Business purpose:** The headline metric of the dashboard — a weighted composite score (0–100) combining CGPA, attendance, credits earned, and backlogs. This score maps to the `StatusSteps` table to assign Novice/Silver/Gold medal tiers.

---

### `Student_subjectscleared`

| Property | Value |
|----------|-------|
| **Table** | `table` |
| **Return Type** | Integer |
| **Used In** | Subjects cleared KPI card |
| **Example Value** | `20` |

```dax
Student_subjectscleared = 
VAR SelectedStudent = SELECTEDVALUE('roll numbers'[Roll number])
RETURN
    IF(
        ISBLANK(SelectedStudent),
        BLANK(),
        // Standard curriculum clear count: total subjects minus active backlogs
        [Total Subjects] - [Student_backlogs]
    )
```

**Business purpose:** Returns the number of subjects the student has successfully cleared. Compared against `Total Subjects` to calculate completion rate.

---

### `Student_credits`

| Property | Value |
|----------|-------|
| **Table** | `table` |
| **Return Type** | Integer |
| **Used In** | Credits KPI card |
| **Example Value** | `82` |

```dax
Student_credits = 
VAR SelectedStudent = SELECTEDVALUE('roll numbers'[Roll number])
RETURN
    IF(
        ISBLANK(SelectedStudent),
        BLANK(),
        SUM('sem wise credits'[Value])
    )
```

**Business purpose:** Returns the total credits earned by the selected student across all semesters.

---

### `Student_backlogs`

| Property | Value |
|----------|-------|
| **Table** | `table` |
| **Return Type** | Integer |
| **Used In** | Backlogs KPI card |
| **Example Value** | `0` |

```dax
Student_backlogs = 
VAR SelectedStudent = SELECTEDVALUE('roll numbers'[Roll number])
RETURN
    IF(
        ISBLANK(SelectedStudent),
        BLANK(),
        // Active backlogs are fetched from the Student score table
        LOOKUPVALUE('Student score'[Backlogs], 'Student score'[Roll number], SelectedStudent)
    )
```

**Business purpose:** Returns the number of active backlogs (failed/pending subjects). Zero backlogs is a positive signal; any non-zero value reduces the composite performance score and may change the medal tier.

---

### `Total Subjects`

| Property | Value |
|----------|-------|
| **Table** | `table` |
| **Return Type** | Integer |
| **Used In** | Subject completion context |
| **Example Value** | `24` |

```dax
Total Subjects = 24  // Standard curriculum benchmark count
```

**Business purpose:** Returns the total number of subjects in the curriculum. Used as the denominator when calculating subject clearance rate (`Student_subjectscleared / Total Subjects`).

---

## Semester Min/Max Measures

These measures identify the best and worst performing semesters using `MINX`, `MAXX`, and `TOPN` patterns — enabling at-a-glance identification of performance peaks and troughs.

---

### `Min Sem CGPA Value`

| Property | Value |
|----------|-------|
| **Table** | `table` |
| **Return Type** | Decimal |
| **Pattern** | `MINX` |

```dax
Min Sem CGPA Value =
MINX('sem wise cgpa data', 'sem wise cgpa data'[Value])
```

**Business purpose:** Returns the lowest semester CGPA achieved by the selected student — highlights the weakest academic semester.

---

### `Min Sem CGPA Name`

| Property | Value |
|----------|-------|
| **Table** | `table` |
| **Return Type** | Text |
| **Pattern** | `TOPN` + `FIRSTNONBLANK` |

```dax
Min Sem CGPA Name =
SELECTCOLUMNS(
    TOPN(1, 'sem wise cgpa data', 'sem wise cgpa data'[Value], ASC),
    "SemName", 'sem wise cgpa data'[sem name]
)
```

**Business purpose:** Returns the semester label (e.g., "Sem 3") corresponding to the lowest CGPA — provides context for the minimum value.

---

### `Max Sem CGPA Value`

| Property | Value |
|----------|-------|
| **Table** | `table` |
| **Return Type** | Decimal |
| **Pattern** | `MAXX` |

```dax
Max Sem CGPA Value =
MAXX('sem wise cgpa data', 'sem wise cgpa data'[Value])
```

**Business purpose:** Returns the highest semester CGPA — identifies the student's peak academic performance.

---

### `Max Sem CGPA Name`

| Property | Value |
|----------|-------|
| **Table** | `table` |
| **Return Type** | Text |
| **Pattern** | `TOPN` + `FIRSTNONBLANK` |

```dax
Max Sem CGPA Name =
SELECTCOLUMNS(
    TOPN(1, 'sem wise cgpa data', 'sem wise cgpa data'[Value], DESC),
    "SemName", 'sem wise cgpa data'[sem name]
)
```

**Business purpose:** Returns the semester label corresponding to the highest CGPA.

---

### `Min Sem Attendance Value`

| Property | Value |
|----------|-------|
| **Table** | `table` |
| **Return Type** | Decimal |
| **Pattern** | `MINX` |

```dax
Min Sem Attendance Value =
MINX('sem wise attendance data', 'sem wise attendance data'[Value])
```

**Business purpose:** Returns the lowest semester attendance percentage — flags the semester with the worst attendance record.

---

### `Min Sem Attendance Name`

| Property | Value |
|----------|-------|
| **Table** | `table` |
| **Return Type** | Text |
| **Pattern** | `TOPN` |

```dax
Min Sem Attendance Name =
SELECTCOLUMNS(
    TOPN(1, 'sem wise attendance data', 'sem wise attendance data'[Value], ASC),
    "SemName", 'sem wise attendance data'[Attribute]
)
```

**Business purpose:** Returns the semester label corresponding to the lowest attendance.

---

### `Max Sem Attendance Value`

| Property | Value |
|----------|-------|
| **Table** | `table` |
| **Return Type** | Decimal |
| **Pattern** | `MAXX` |

```dax
Max Sem Attendance Value =
MAXX('sem wise attendance data', 'sem wise attendance data'[Value])
```

**Business purpose:** Returns the highest semester attendance percentage.

---

### `Max Sem Attendance Name`

| Property | Value |
|----------|-------|
| **Table** | `table` |
| **Return Type** | Text |
| **Pattern** | `TOPN` |

```dax
Max Sem Attendance Name =
SELECTCOLUMNS(
    TOPN(1, 'sem wise attendance data', 'sem wise attendance data'[Value], DESC),
    "SemName", 'sem wise attendance data'[Attribute]
)
```

**Business purpose:** Returns the semester label corresponding to the highest attendance.

---

## HTML Renderer Measures

These measures return raw HTML strings rendered by the **HTML Content** custom visual. They enable rich, styled content inside Power BI that standard visuals cannot produce.

---

### `LDashboardTitleHTML`

| Property | Value |
|----------|-------|
| **Table** | `table` |
| **Return Type** | Text (HTML) |
| **Used In** | Dashboard header — title bar |

```dax
LDashboardTitleHTML = 
"
<div style='text-align: left; padding: 10px 0;'>
    <h1 style='font-family: \"Outfit\", \"Segoe UI\", sans-serif; font-size: 28px; font-weight: 700; color: #FFFFFF; margin: 0; letter-spacing: 1.5px; text-transform: uppercase;'>
        🎓 Beyond Grades
    </h1>
</div>
"
```

**Business purpose:** Generates a styled HTML title block for the dashboard header area, incorporating custom fonts, gradients, and responsive sizing that exceed standard text visual capabilities.

---

### `AnimatedSubtitleHTML`

| Property | Value |
|----------|-------|
| **Table** | `table` |
| **Return Type** | Text (HTML) |
| **Used In** | Dashboard header — animated subtitle |

```dax
AnimatedSubtitleHTML = 
"
<div style='text-align: left; font-family: \"Inter\", \"Segoe UI\", sans-serif; font-size: 13px; color: #B3B0AD; font-weight: 400; margin-top: 4px;'>
    <span style='animation: pulseGlow 3s ease-in-out infinite alternate;'>
        Student Performance Analytics & Academic Achievement Tracker
    </span>
    <style>
        @keyframes pulseGlow {
            0% { opacity: 0.6; color: #B3B0AD; }
            100% { opacity: 1; color: #FFFFFF; text-shadow: 0 0 8px rgba(255, 255, 255, 0.3); }
        }
    </style>
</div>
"
```

**Business purpose:** Produces an animated subtitle with CSS keyframe animations (e.g., fade-in, typing effect) displayed beneath the main title — adds visual polish and a premium feel to the dashboard header.

---

### `L_Card_HTML_Measure`

| Property | Value |
|----------|-------|
| **Table** | `table` |
| **Return Type** | Text (HTML) |
| **Used In** | Student profile card |

```dax
L_Card_HTML_Measure = 
VAR _Name = [Student_Name]
VAR _Gender = [Student_gender]
VAR _College = [Student_college]
VAR _Branch = [Student_branch]
VAR _Email = [Student_email]
VAR _Phone = [Student_phonenum]
VAR _Photo = SELECTEDVALUE(Bio[Photo URL])
RETURN
"
<div style='background: rgba(255, 255, 255, 0.03); border: 1px solid rgba(255, 255, 255, 0.08); border-radius: 12px; padding: 16px; font-family: \"Outfit\", \"Segoe UI\", sans-serif; display: flex; align-items: center; gap: 16px; box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15); backdrop-filter: blur(8px); color: #FFFFFF; max-width: 400px;'>
    <div style='width: 65px; height: 65px; border-radius: 50%; overflow: hidden; border: 2px solid #118DFF; flex-shrink: 0;'>
        <img src='\"" & _Photo & "\"' style='width: 100%; height: 100%; object-fit: cover;' alt='Photo' onerror='this.src=\"https://via.placeholder.com/150/118DFF/FFFFFF?text=Student\"' />
    </div>
    <div style='flex: 1; min-width: 0;'>
        <div style='font-size: 16px; font-weight: 600; margin-bottom: 2px; color: #FFFFFF; white-space: nowrap; overflow: hidden; text-overflow: ellipsis;'>" & _Name & "</div>
        <div style='font-size: 11px; color: #B3B0AD; margin-bottom: 6px; text-transform: uppercase; letter-spacing: 0.5px;'>" & _College & " &middot; " & _Branch & " (" & _Gender & ")</div>
        <div style='font-size: 10px; color: #8F8C8A; display: flex; flex-direction: column; gap: 2px;'>
            <div>📧 " & _Email & "</div>
            <div>📞 " & _Phone & "</div>
        </div>
    </div>
</div>
"
```

**Business purpose:** Constructs a complete student profile card in HTML — including photo, name, roll number, college, branch, and contact details — rendered as a single rich visual. This approach allows pixel-perfect layout control beyond Power BI's native card visuals.

---

### `Student Score Medal HTML`

| Property | Value |
|----------|-------|
| **Table** | `table` |
| **Return Type** | Text (HTML) |
| **Used In** | Medal / status badge display |

```dax
Student Score Medal HTML = 
VAR _Score = [Student_performance_score]
VAR _Tier = 
    CALCULATE(
        SELECTEDVALUE(StatusSteps[Status tier]),
        FILTER(
            StatusSteps,
            _Score >= StatusSteps[Min Score] && _Score < StatusSteps[Max Score]
        )
    )
VAR _Color = 
    SWITCH(
        _Tier,
        "Gold", "#FFD700",
        "Silver", "#C0C0C0",
        "Novice", "#CD7F32",
        "#B3B0AD"
    )
VAR _Emoji = 
    SWITCH(
        _Tier,
        "Gold", "🥇",
        "Silver", "🥈",
        "Novice", "🥉",
        "🎓"
    )
RETURN
"
<div style='display: flex; align-items: center; justify-content: center; flex-direction: column; font-family: \"Outfit\", \"Segoe UI\", sans-serif; color: #FFFFFF; text-align: center; padding: 10px; background: rgba(255, 255, 255, 0.02); border-radius: 8px; border: 1px solid rgba(255, 255, 255, 0.05);'>
    <div style='font-size: 40px; margin-bottom: 2px; filter: drop-shadow(0 0 6px " & _Color & "80);'>" & _Emoji & "</div>
    <div style='font-size: 13px; font-weight: 700; text-transform: uppercase; letter-spacing: 1px; color: " & _Color & "; margin-top: 4px;'>" & _Tier & " Tier</div>
    <div style='font-size: 11px; color: #8F8C8A; margin-top: 2px;'>Score: " & FORMAT(_Score, "0.0") & " / 100</div>
</div>
"
```

**Business purpose:** Renders the student's medal tier (Novice 🥉 / Silver 🥈 / Gold 🥇) as a styled HTML badge, with dynamic colours and icons determined by the `Student_performance_score` and `StatusSteps` thresholds.

---

### `Attendance Badge HTML`

| Property | Value |
|----------|-------|
| **Table** | `table` |
| **Return Type** | Text (HTML) |
| **Used In** | Attendance status indicator |

```dax
Attendance Badge HTML = 
VAR _Attendance = [Student_attendance]
VAR _Color = 
    IF(
        _Attendance >= 85, "#2ECC71",  // Safe - Green
        IF(_Attendance >= 75, "#F1C40F", // Borderline - Amber
        "#E74C3C"                      // Critical - Red
        )
    )
VAR _Status = 
    IF(
        _Attendance >= 85, "Healthy", 
        IF(_Attendance >= 75, "Borderline", 
        "Critical"
        )
    )
RETURN
"
<div style='display: inline-block; background: " & _Color & "15; border: 1px solid " & _Color & "; color: " & _Color & "; border-radius: 20px; padding: 4px 12px; font-family: \"Outfit\", \"Segoe UI\", sans-serif; font-size: 11px; font-weight: 600; text-align: center; text-transform: uppercase; letter-spacing: 0.5px;'>
    " & _Status & " (" & FORMAT(_Attendance, "0.0%") & ")
</div>
"
```

**Business purpose:** Generates a colour-coded attendance badge — green for healthy attendance (≥ 75%), amber for borderline, red for critical. Provides an instant visual signal without requiring the user to interpret raw percentage values.

---

### `cgpa Badge HTML`

| Property | Value |
|----------|-------|
| **Table** | `table` |
| **Return Type** | Text (HTML) |
| **Used In** | CGPA status indicator |

```dax
cgpa Badge HTML = 
VAR _CGPA = [Student_cgpa]
VAR _Color = 
    IF(
        _CGPA >= 8.0, "#2ECC71",  // Outstanding - Green
        IF(_CGPA >= 6.5, "#F1C40F", // Average - Amber
        "#E74C3C"                 // Needs Imp. - Red
        )
    )
VAR _Status = 
    IF(
        _CGPA >= 8.0, "Outstanding", 
        IF(_CGPA >= 6.5, "Good", 
        "Needs Imp."
        )
    )
RETURN
"
<div style='display: inline-block; background: " & _Color & "15; border: 1px solid " & _Color & "; color: " & _Color & "; border-radius: 20px; padding: 4px 12px; font-family: \"Outfit\", \"Segoe UI\", sans-serif; font-size: 11px; font-weight: 600; text-align: center; text-transform: uppercase; letter-spacing: 0.5px;'>
    " & _Status & " (" & FORMAT(_CGPA, "0.00") & ")
</div>
"
```

---

## Recommended Next Measures

These measures would strengthen the dashboard's analytical value and are straightforward to implement:

### Semester_GPA_Change

```dax
Semester_GPA_Change =
VAR CurrentSem =
    MAXX('sem wise cgpa data', 'sem wise cgpa data'[sem name])
VAR PreviousSem =
    CALCULATE(
        MAXX('sem wise cgpa data', 'sem wise cgpa data'[sem name]),
        'sem wise cgpa data'[sem name] < CurrentSem
    )
VAR CurrentGPA =
    CALCULATE(
        SELECTEDVALUE('sem wise cgpa data'[Value]),
        'sem wise cgpa data'[sem name] = CurrentSem
    )
VAR PreviousGPA =
    CALCULATE(
        SELECTEDVALUE('sem wise cgpa data'[Value]),
        'sem wise cgpa data'[sem name] = PreviousSem
    )
RETURN
    CurrentGPA - PreviousGPA
```

**Use case:** Shows semester-over-semester CGPA change (positive = improvement, negative = decline). Could power a KPI card with conditional up/down arrows, giving faculty an instant trend signal.

---

### Attendance_Risk_Flag

```dax
Attendance_Risk_Flag =
IF(
    [Student_attendance] < 75,
    "⚠️ At Risk",
    IF(
        [Student_attendance] < 85,
        "⚡ Borderline",
        "✅ Safe"
    )
)
```

**Use case:** Text-based risk classification for attendance. Can drive conditional formatting or alert visuals — particularly useful for academic advisors monitoring large student cohorts.

---

### Credit_Completion_Pct

```dax
Credit_Completion_Pct =
DIVIDE(
    [Student_credits],
    <total_required_credits>,
    0
)
```

**Use case:** Percentage of total required credits earned to date. A progress-bar visual driven by this measure would show how close a student is to graduation requirements — valuable for academic planning conversations.

---

> **Note:** The measures above reflect the functional intent inferred from the data model. The exact DAX expressions are embedded in the `.pbix` file. Open the file in Power BI Desktop → **Model view → table (measure island)** to inspect or modify them.
