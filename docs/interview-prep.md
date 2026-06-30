# 🎤 Interview Preparation — Technical Q&A

[← Back to README](../README.md)

> 16 prepared answers covering data modelling, DAX, visualisation, interactivity, and deployment for the Beyond Grades student performance dashboard.

---

## Table of Contents

- [Section 1: Data Modelling](#section-1-data-modelling) — Q1–Q4
- [Section 2: DAX](#section-2-dax) — Q5–Q7
- [Section 3: Visualisation & UX](#section-3-visualisation--ux) — Q8–Q11
- [Section 4: Interactivity & Performance](#section-4-interactivity--performance) — Q12–Q14
- [Section 5: Deployment & Governance](#section-5-deployment--governance) — Q15–Q16
- [Quick-Reference Cheat Sheet](#quick-reference-cheat-sheet)

---

## Section 1: Data Modelling

### Q1. Walk me through the data model. Why did you choose this structure?

> The model follows a **star-like schema** with **12 tables** organised around a central student data set. Dimension tables handle lookup data for colleges, branches, genders, and semester-level metrics, while a disconnected measure island called `table` houses all DAX business logic. I chose this structure because it cleanly separates raw data from computed logic, keeps relationships simple (single-direction, many-to-one), and lets Power BI's VertiPaq engine resolve cross-filter queries efficiently — even when the roll number slicer, bookmark navigator, and profile card are all interacting simultaneously on a single 1280×720 canvas.

---

### Q2. Why did you use a disconnected measure island?

> Storing all measures in a disconnected table called `table` is a widely adopted Power BI best practice. It has three concrete benefits:
>
> 1. The source data tables' field lists stay clean — **only raw columns, no mixed measures**
> 2. All business logic is in **one findable location** when you need to edit or audit formulas
> 3. The measure island can be hidden from report view while still being accessible to visuals
>
> Because this dashboard pulls from a **remote semantic model** (Power BI Service dataset), the measure island also provides a single place to define calculated fields without modifying the shared dataset schema — any report connected to the same dataset can define its own measures independently.

---

### Q3. How do the unpivoted semester tables work?

> Individual semester data — both CGPA and attendance — is stored across columns like `sem1_cgpa`, `sem2_cgpa`, `sem3_cgpa`, `sem4_cgpa` (and the corresponding attendance columns). To plot these on a **line chart with semesters on the x-axis**, the data needs to be unpivoted into a long format: one row per student per semester, with columns for `Semester`, `CGPA`, and `Attendance`. This transformation converts the wide-format source data into a shape that Power BI's line chart visual can naturally consume — each semester becomes a categorical axis point, and CGPA or attendance becomes the y-axis value. Without unpivoting, you would need four separate measures manually positioned on the axis, which breaks dynamic filtering and makes the model brittle.

---

### Q4. How does the StatusSteps table drive the medal system?

> The `StatusSteps` table is a **lookup table** that defines three achievement tiers — **Novice**, **Silver**, and **Gold** — along with their corresponding thresholds and visual properties (badge colours, medal icons). When a student is selected via the roll number slicer, DAX measures evaluate the student's `performance_score`, `cgpa`, and `attendance` against the thresholds in `StatusSteps` to determine which tier they fall into. The resulting tier label is then passed to the **HTML Content custom visual**, which renders the appropriate badge or medal icon with tier-specific styling. This design is fully data-driven — adding a new tier (e.g., Platinum) requires only a new row in `StatusSteps`, not a DAX or visual redesign.

---

## Section 2: DAX

### Q5. Explain the Min/Max semester measures.

> The dashboard highlights each student's **best and worst semesters** using conditional indicators with green up-arrows (▲) and red down-arrows (▼). The DAX logic works in two steps:
>
> 1. **Identify extremes** — `MAXX` and `MINX` iterate over the unpivoted semester rows for the selected student and return the highest and lowest CGPA (or attendance) values:
>
> ```dax
> Max Semester CGPA =
> MAXX(
>     FILTER('SemesterData', 'SemesterData'[roll_number] = SELECTEDVALUE('Students'[roll_number])),
>     'SemesterData'[cgpa]
> )
> ```
>
> 2. **Conditional formatting** — a companion measure compares each semester's value to the max/min and returns an icon or colour code. The green arrow appears next to the semester matching the `MAXX` result; the red arrow appears next to the `MINX` result. If a student has identical values across all semesters, no arrows are displayed, preventing misleading emphasis.

---

### Q6. How do the HTML renderer measures work?

> The **HTML Content custom visual** (v1.6.0 by Daniel Marsh-Patrick) accepts a DAX measure that returns a string of valid HTML. I use this pattern to build several dashboard components dynamically:
>
> 1. **Dashboard title** — returns an `<h1>` tag with inline CSS for the "Beyond Grades" heading, styled with custom font, colour, and letter-spacing
> 2. **Animated subtitle** — returns HTML with CSS `@keyframes` to create a subtle fade-in or typing animation on the subtitle text
> 3. **Profile card** — constructs a card layout with the student's photo, name, gender, college, and branch, using `SELECTEDVALUE()` to pull each field
> 4. **Achievement badges/medals** — returns SVG or styled `<div>` elements showing the Novice/Silver/Gold badge based on the tier determined by `StatusSteps`
>
> ```dax
> HTML_ProfileCard =
> VAR _name = SELECTEDVALUE('Students'[name], "--")
> VAR _college = SELECTEDVALUE('Students'[college], "--")
> RETURN
>     "<div style='font-family:Segoe UI; color:white;'>"
>     & "<h2>" & _name & "</h2>"
>     & "<p>" & _college & "</p>"
>     & "</div>"
> ```
>
> The key advantage is **complete CSS control** — fonts, gradients, animations, and responsive layouts that native Power BI visuals cannot achieve.

---

### Q7. How is the Student Performance Score calculated?

> The `performance_score` is a **composite metric** that blends multiple academic indicators into a single 0–100 scale. The calculation weights three factors:
>
> 1. **CGPA** — normalised to a 0–100 range (CGPA ÷ 10 × 100), weighted highest as the primary academic indicator
> 2. **Attendance percentage** — used directly as a 0–100 value, reflecting engagement and consistency
> 3. **Subjects cleared ratio** — `subjects_cleared` ÷ total subjects (24), scaled to 0–100, penalising students with backlogs
>
> The weighted formula produces a single number that the `StatusSteps` table then maps to a tier (Novice / Silver / Gold). This approach avoids the pitfall of using CGPA alone — a student with 9.0 CGPA but 50% attendance and 3 backlogs would score lower than their GPA suggests, which is the intended design: the dashboard is called "Beyond Grades" because it evaluates the **whole student**, not just marks.

---

## Section 3: Visualisation & UX

### Q8. How did you achieve the dark-themed, premium look?

> The visual identity is built on four layers:
>
> 1. **Canvas background** — a **deep navy fill** (`#12239E`) applied as the page background colour, establishing the dominant tone
> 2. **SVG overlay** — a scalable vector graphic layered on top of the canvas background, adding subtle texture, gradients, or geometric patterns without rasterisation artifacts at any zoom level
> 3. **CY25SU11 theme** — a custom Power BI theme JSON that standardises font families, text colours (white/light grey), accent colours, and visual border styles across all native visuals
> 4. **HTML Content visuals** — for elements requiring styling beyond what the theme supports (gradients, animations, custom fonts), the HTML renderer provides full CSS control
>
> The result is a **cohesive, portfolio-ready aesthetic** where every element — from KPI cards to the profile section — shares the same visual language. The dark background also increases the perceptual contrast of coloured data points on charts, making trends easier to read at a glance.

---

### Q9. Why use the HTML Content custom visual?

> The **HTML Content visual** (v1.6.0 by Daniel Marsh-Patrick) fills a critical gap: Power BI's native visuals offer limited text styling — no custom fonts beyond the theme set, no CSS animations, no inline SVG, no responsive HTML layouts. I use it for five components:
>
> 1. **Dashboard title** — custom font sizing and letter-spacing that the native text box cannot achieve
> 2. **Animated subtitle** — CSS keyframe animations (fade-in, slide-up) for visual polish
> 3. **Profile card** — structured layout with student photo, name, gender, college, and branch in a styled card
> 4. **Achievement badges** — Novice/Silver/Gold badges with tier-specific colours, icons, and conditional rendering
> 5. **Contact section** — phone and email displayed with icons and consistent formatting
>
> The trade-off is a **dependency on a third-party visual** — if the organisation's Power BI admin blocks custom visuals, these components would need to be rebuilt with native text boxes and images, losing the animation and layout precision.

---

### Q10. How does the gauge visual work?

> The gauge shows **subjects cleared vs total subjects** — for example, 20 out of 24 (83%). The configuration is:
>
> - **Value** — `SELECTEDVALUE('Students'[subjects_cleared])`, which responds to the roll number slicer
> - **Maximum** — hardcoded to `24` (the total number of subjects in the curriculum)
> - **Target** — also set to `24`, drawing a target line at the full-clear mark
>
> The gauge's fill colour shifts based on the ratio: green tones for 80%+, amber for 60–80%, and red below 60%. This gives an **instant visual read** on whether a student is on track to complete all subjects. When no student is selected, the gauge displays `0`, reinforcing the "select a student" prompt.

---

### Q11. What design decisions make this dashboard portfolio-ready?

> Five deliberate choices elevate this from a functional report to a portfolio piece:
>
> 1. **Single-page constraint** — forces information hierarchy discipline; everything fits on 1280×720 without scrolling
> 2. **Consistent colour language** — deep navy background, white text, accent colours only for data and interactive elements
> 3. **Data-driven components** — badges, profile cards, and titles are all generated by DAX measures, not static images, so they respond to slicer selections
> 4. **Bookmark navigator** — three exploration states (Performance, Attendance, Academics) give the viewer a guided path through the data without overwhelming them with all visuals at once
> 5. **Empty-state handling** — when no student is selected, all fields show `"--"` placeholders, Novice-tier badges appear, and the gauge shows 0 — the dashboard never looks broken or half-loaded

---

## Section 4: Interactivity & Performance

### Q12. How does the 3-state Explore navigator work?

> The navigator uses **4 bookmarks** mapped to **3 states** via a single Explore control — this is not two independent toggles, but one unified navigator with three mutually exclusive views:
>
> 1. **Performance** — shows the **achievement badges/medals** (Novice/Silver/Gold) and the **gauge visual** (subjects cleared vs total). This is the default landing state, giving an at-a-glance summary of the selected student's standing
> 2. **Attendance** — shows the **achievement badges** with an attendance-focused layout, emphasising attendance metrics and the student's engagement data
> 3. **Academics** — shows the **two line charts**: semester CGPA trend and semester attendance trend across all 4 semesters, with Min/Max indicators (green ▲ / red ▼ arrows)
>
> Each bookmark stores the **visibility state** of specific visual groups — when "Academics" is selected, the gauge and badge visuals are hidden and the line charts are shown; when "Performance" is selected, the reverse occurs. The 4th bookmark serves as the **default/reset state**. The profile section, contact section, and roll number slicer remain **persistent across all three states**, providing continuity as the user navigates. This pattern avoids multi-page navigation, keeping the entire experience on a single canvas.

---

### Q13. How does the roll number slicer drive the entire dashboard?

> The roll number slicer is a **text slicer** bound to the `roll_number` column. When a user types or selects a roll number (e.g., `23A91A0501`), it applies a filter that propagates through **every relationship** in the model:
>
> 1. The **profile card** updates — `SELECTEDVALUE()` measures pull the student's name, gender, college, branch, and photo
> 2. The **contact section** updates — phone and email fields resolve to the selected student
> 3. The **gauge visual** updates — `subjects_cleared` reflects the selected student's progress
> 4. The **line charts** update — semester CGPA and attendance trends filter to the selected student's 4 semesters
> 5. The **badges/medals** update — the `StatusSteps` tier is recalculated based on the selected student's performance score
> 6. The **Min/Max indicators** update — arrows reposition to the selected student's best and worst semesters
>
> This **single slicer drives 100% of the dashboard content**, making it a true student-level drill-down tool. There are no other slicers or filters — the roll number is the sole interaction point.

---

### Q14. What happens when no student is selected?

> The dashboard is designed with a **deliberate empty state** to avoid confusion or visual artifacts:
>
> 1. **All text fields show `"--"`** — name, college, branch, gender, phone, email all display the placeholder string via `SELECTEDVALUE('Students'[name], "--")` with the alternate result parameter
> 2. **Novice-tier badges appear** — the `StatusSteps` logic defaults to the lowest tier when no performance score can be calculated, so the badges render in their base/default style
> 3. **The gauge shows 0** — `SELECTEDVALUE('Students'[subjects_cleared])` returns BLANK, which the gauge interprets as zero, displaying an empty arc
> 4. **Line charts show no data points** — with no student filter, the semester data returns empty, and the charts display a clean empty state
>
> This design ensures the dashboard **never looks broken** — a viewer who lands on the page without a selection immediately understands they need to enter a roll number. It is a common portfolio-quality pattern: design the empty state as carefully as the populated state.

---

## Section 5: Deployment & Governance

### Q15. Why does this report use a remote semantic model instead of embedded data?

> The report connects to a **remote semantic model** hosted on Power BI Service (DatasetId: `38109f7e-969f-4f01-9ac0-d1273ed20835`) rather than embedding data inside the `.pbix` file. This architecture has four advantages:
>
> | Benefit | Detail |
> |---------|--------|
> | **Single source of truth** | Multiple reports can connect to the same dataset; a schema change propagates once |
> | **Smaller `.pbix` file** | The report file contains only visuals and layout — no data — making it lightweight for version control |
> | **Server-side refresh** | Data refresh happens on Power BI Service on a schedule; report consumers always see current data without manual action |
> | **Governance separation** | Dataset owners control the schema and refresh; report authors control the visual layer — clean separation of concerns |
>
> The trade-off is that the report **cannot be opened offline** — it requires a live connection to Power BI Service. For a portfolio project, this is acceptable because the intent is to demonstrate architecture skills, not offline usability.

---

### Q16. How would you add Row-Level Security?

> RLS is **not currently configured** because this is a single-owner portfolio project. To add it for a production deployment (e.g., restricting faculty to their department's students):
>
> 1. Add a `department` or `branch` column to the student table (already present as `branch` — CSE, ECE, EEE, MECH)
> 2. In Power BI Desktop → **Modelling → Manage Roles**, create a role per department:
>
> ```dax
> [branch] = "CSE"
> ```
>
> 3. In Power BI Service, assign each faculty member's email to their corresponding role
> 4. When they access the report, the RLS filter silently applies **before any visual query runs** — the report URL and layout are identical across all faculty, but each sees only their department's students
>
> For a more scalable approach, replace the static role filter with a **dynamic RLS** pattern using `USERPRINCIPALNAME()`:
>
> ```dax
> [faculty_email] = USERPRINCIPALNAME()
> ```
>
> This requires a mapping table linking faculty emails to branches, but eliminates the need to create a separate role for each department.

---

## Quick-Reference Cheat Sheet

| Topic | Key Point |
|-------|-----------|
| Model type | Star-like schema — 12 tables with single-direction relationships |
| Tables | Student data, semester tables, StatusSteps, dimension lookups |
| Measures | Disconnected measure island (`table`) — all DAX logic centralised |
| Custom Visuals | HTML Content v1.6.0 (Daniel Marsh-Patrick) — titles, badges, profile |
| Bookmarks | 4 bookmarks → 3-state Explore navigator (Performance, Attendance, Academics) |
| Theme | CY25SU11 custom theme JSON — dark mode, white text, accent colours |
| Canvas | 1280 × 720, deep navy `#12239E` background with SVG overlay |
| Data Source | Remote semantic model on Power BI Service (no embedded data) |
| RLS | Not configured — single-owner portfolio scope; roadmap item |
| Key feature | Roll number slicer drives 100% of dashboard content — single interaction point |
