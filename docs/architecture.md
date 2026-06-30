# Architecture

The dashboard is built on a scalable and modular architecture leveraging Power BI best practices.

```mermaid
graph LR
    A[☁️ Power BI<br/>Service Dataset] -->|Live Connection<br/>Semantic Model| B[⚙️ Data Model]
    B --> C[3 Semester<br/>Fact Tables]
    B --> D[5 Staging /<br/>Lookup Tables]
    B --> E[Measure<br/>Island]
    B --> F[Bio +<br/>Roll Numbers]
    C --> G[📊 Report<br/>Canvas]
    D --> G
    E --> G
    F --> G
    G --> H[🖥️ Interactive<br/>Dashboard]
    H --> I[🔘 Explore<br/>Navigator]
    H --> J[📋 KPI Cards<br/>+ Badges]
    H --> K[📈 Charts<br/>+ Gauge]
    H --> L[🎨 HTML<br/>Content Visuals]

    style A fill:#118DFF,stroke:#0D6EBF,color:#fff
    style B fill:#4B275F,stroke:#361C44,color:#fff
    style G fill:#F2C811,stroke:#D4AD0E,color:#000
    style H fill:#12239E,stroke:#0D1A7A,color:#fff
```

## Tech Stack

| Technology | Usage |
|------------|-------|
| **Power BI Desktop** | Report authoring, data modelling, DAX measures |
| **Power BI Service** | Semantic model hosting, data refresh, live connection |
| **Power Query (M)** | Data extraction, transformation, and loading |
| **DAX** | Business logic — SELECTEDVALUE, MINX, MAXX, CALCULATE, DIVIDE, IF |
| **HTML Content Visual** | Custom HTML/CSS rendering for profile cards, badges, titles |
| **Star Schema** | Data model design pattern (unpivoted semester facts) |
