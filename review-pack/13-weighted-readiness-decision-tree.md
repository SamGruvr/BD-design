# Weighted Readiness Decision Tree

This artifact documents the current Step 2/3 weighted scoring logic used by the readiness API and review UI as non-blocking decision guidance.

## User-Friendly Graphic
![Weighted readiness decision tree](/Users/sama/Documents/GitHub/BD_ITA/docs/review-pack/assets/weighted-readiness-decision-tree.svg)

## Mermaid (Source Diagram)
```mermaid
flowchart TD
    A["Start: Evaluate Deal Readiness (Step 2 or Step 3)"] --> B["Build Section Checks"]
    B --> C1["Opportunity Profile<br/>Weight: 35<br/>Min: 60%"]
    B --> C2["Schedule Milestones<br/>Weight: 20<br/>Min: 66%"]
    B --> C3["Incumbent Baseline<br/>Weight: 20<br/>Min: 50%"]
    B --> C4["Market Evidence<br/>Weight: 15<br/>Min: 50%"]
    B --> C5{"Step == 3?"}

    C5 -->|No (Step 2)| C6["Reserve Section<br/>Weight: 10<br/>No min rule"]
    C5 -->|Yes (Step 3)| C7["Step 3 Status Update<br/>Weight: 5<br/>Min: 100%"]
    C5 -->|Yes (Step 3)| C8["Step 3 Source Links (FPDS)<br/>Weight: 5<br/>Min: 100%"]
    C5 -->|Yes (Step 3)| C9["Step 3 Checklist Complete<br/>Weight: 10<br/>Min: 100%"]

    C1 --> D["For each section:<br/>percent = completed checks / total checks"]
    C2 --> D
    C3 --> D
    C4 --> D
    C6 --> D
    C7 --> D
    C8 --> D
    C9 --> D

    D --> E["Section weighted score = percent * weight / 100"]
    E --> F["Total score = sum(all section weighted scores)<br/>Range: 0-100"]
    F --> G["Find minimumFailures:<br/>sections where percent < minimumPercent"]

    G --> H{"Step threshold met?"}
    H -->|Step 2: score >= 70<br/>Step 3: score >= 80| I{"Any minimumFailures?"}
    H -->|No| J["Guidance: readiness below threshold"]
    I -->|No| K["Guidance: readiness on-track"]
    I -->|Yes| J

    F --> L{"Recommendation by score"}
    L -->|>= 85| M["go"]
    L -->|>= 70 and < 85| N["conditional_go"]
    L -->|>= 50 and < 70| O["hold_recycle"]
    L -->|< 50| P["no_go"]
```

## Notes
- Readiness scores and section minimums drive recommendation quality, but do not block save/share state transitions.
- Step 3 adds stricter guidance sections (status update, FPDS link, checklist completion).

## Current Market Evidence (Option 2)
The Market Evidence section is currently evaluated with 6 checks (weight `15`, minimum `60%`):
1. At least one competitor exists.
2. At least one competitor has rationale or confidence.
3. At least one stakeholder exists.
4. Customer knowledge is captured (`customer_familiarity` or `familiarity_details`).
5. At least one voice-of-customer item has a response strategy.
6. At least one key meeting has notes.
