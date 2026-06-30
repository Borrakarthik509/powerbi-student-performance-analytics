# Dashboard Guide

This is a **single-page dashboard** with a **3-state Explore navigator** and six visual zones.

## Overview Layout
```text
┌─────┬──────────────────────────────────────────────────────────────┬──────────┐
│LOGO │  BEYOND GRADES  (HTML title + animated subtitle)            │ SLICER   │
│badge│                                                             │ Roll No. │
├─────┴────┬─────────────────────────────────┬──────────────────────┴──────────┤
│ PROFILE  │         CGPA CARD               │       ATTENDANCE CARD           │
│          │  Badge (Novice/Silver/Gold)      │  Badge (Novice/Silver/Gold)     │
│  Photo   │  CGPA value                     │  Attendance %                   │
│  Name    │  MAX: semester ↑  MIN: sem ↓    │  MAX: semester ↑  MIN: sem ↓    │
│          │  — OR (Academics view) —         │  — OR (Academics view) —        │
│  Gender  │  Line chart (4 semesters)       │  Line chart (4 semesters)       │
│  College ├─────────────────────────────────┼─────────────────────────────────┤
│  Branch  │         CREDITS CARD            │    STUDENTS PERFORMANCE         │
│          │  Gauge (subjects cleared/total)  │  Medal (Novice/Silver/Gold)     │
│          │  Credits Earned · Backlogs       │  Student score                  │
├──────────┴─────────────────────────────────┴─────────────────────────────────┤
│  CONTACT SECTION  ·  Phone number  ·  Email ID                               │
└──────────────────────────────────────────────────────────────────────────────┘
```

## Explore States (bookmark-driven)

| State | What Changes | Key Visuals |
|-------|-------------|-------------|
| **Performance** | CGPA & Attendance cards show achievement badges + gauge | Badges, medals, gauge arc |
| **Attendance** | Same layout with attendance-focused highlight | Badges, min/max indicators |
| **Academics** | CGPA & Attendance cards switch to semester line charts | Line charts (Sem 1–4 trends) |

## Quick Start
1. **Enter a Roll Number** in the slicer (top-right) to load a student's data.
2. **Use the Explore navigator** (top-left) to switch between Performance, Attendance, and Academics views.
