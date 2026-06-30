# Data Model

The data model follows a **star-like schema** — unpivoted semester fact tables, staging/lookup dimensions, a Bio dimension, and one isolated measure island.

```mermaid
graph TD
    subgraph Dimensions
        RN[Roll Numbers]
        BIO[Bio]
    end

    subgraph "Semester Facts (Unpivoted)"
        SWC[sem wise cgpa data]
        SWA[sem wise attendance data]
        SWCR[sem wise credits]
    end

    subgraph "Staging / Lookup"
        SA[sem_attendance]
        SC[sem_credits]
        SCP[Sem cgpa perf]
        SS[StatusSteps]
        SWAC[sem wise avg credits]
    end

    subgraph "Calculated"
        SSC[Student score]
    end

    subgraph "Measure Island"
        MT[table<br/><i>25+ DAX measures</i>]
    end

    RN -->|Roll number| SWC
    RN -->|Roll number| SWA
    RN -->|Roll number| SWCR
    BIO -->|Roll number| RN
    SA -.->|lookup| SWA
    SC -.->|lookup| SWCR
    SCP -.->|classification| SSC
    SS -.->|medal tiers| MT
    SWAC -.->|averages| SWCR
    MT -.- |DAX references| SWC
    MT -.- |DAX references| SWA
    MT -.- |DAX references| SSC

    style RN fill:#118DFF,stroke:#0D6EBF,color:#fff
    style BIO fill:#118DFF,stroke:#0D6EBF,color:#fff
    style SWC fill:#12239E,stroke:#0D1A7A,color:#fff
    style SWA fill:#12239E,stroke:#0D1A7A,color:#fff
    style SWCR fill:#12239E,stroke:#0D1A7A,color:#fff
    style SA fill:#1D1E23,stroke:#B3B0AD,color:#fff
    style SC fill:#1D1E23,stroke:#B3B0AD,color:#fff
    style SCP fill:#1D1E23,stroke:#B3B0AD,color:#fff
    style SS fill:#1D1E23,stroke:#B3B0AD,color:#fff
    style SWAC fill:#1D1E23,stroke:#B3B0AD,color:#fff
    style SSC fill:#E66C37,stroke:#C45A2E,color:#fff
    style MT fill:#D9B300,stroke:#B39700,color:#000
```

> **Why a disconnected Measure Island?** All 25+ DAX measures are stored in an isolated `table` — a best practice that separates business logic from raw data columns and keeps the field list clean. Every card, badge, gauge, and HTML renderer pulls from this single measure container.

> **Why unpivoted semester tables?** The `sem wise cgpa data` and `sem wise attendance data` tables store semester data in a long (Attribute/Value) format, which is the optimal structure for Power BI line charts to show semester-by-semester trends without needing separate columns per semester.
