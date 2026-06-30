# Performance Optimization

To ensure a smooth, interactive experience with instant visual rendering, the following optimization strategies are implemented:

## Data Model Optimization
- **Star Schema Design**: Complex queries are reduced by organizing data into unpivoted semester facts and small lookup tables.
- **Disconnected Measure Island**: All 25+ DAX calculations are isolated, which prevents implicit filtering overhead from dimensions on unused tables.

## Visualization Optimization
- **Bookmark-Driven Navigation**: By toggling visibility states using bookmarks instead of page navigation, visual rendering is accelerated, resulting in a snappy user experience.
- **HTML Content Visual Constraints**: Custom visuals are kept lightweight to prevent long processing times in the JavaScript renderer.

## DAX Best Practices
- **Variables**: DAX measures leverage `VAR` extensively to calculate expressions only once per context.
- **Iterator Functions**: Minimized usage of slow iterators (`FILTER`, `SUMX`) over large tables in favor of context transition functions where applicable.
