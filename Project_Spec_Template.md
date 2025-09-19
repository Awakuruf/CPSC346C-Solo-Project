# Project Spec - CatchChance

## 1) User & Decision
**User:** Recreational anglers using mobile/web app.  
**Decision:** Whether to fish during a given session or which hour to target based on predicted probability of catching at least one fish.  
**Rationale:** Helps anglers optimize time on water and manage expectations; reduces frustration and wasted effort.  

## 2) Target & Horizon
**Target:** Binary — catch ≥1 fish during the next fishing session.  
**Horizon:** Short-term: next session (~2–4 hours) with optional hourly breakdown.  
**Output:** Probability (0–1) per coarse bucket (morning / afternoon / evening / night); optional per hour.  

## 3) Features
- hour, day, month
- coarse location grid
- user past 30-day stats
- optional: bait, target species, environment if available

## 4) Baseline
- Historical catch rate by hour + grid
- Laplace smoothing

## 5) Possible Model
- Logistic regression with small feature set

## 6) Metrics
- Care about calibration (Brier)
- AUC-PR for imbalanced positives

## 7) API Sketch
- `POST /predict` → probability
- `GET /health`
- maybe `POST /report_session`
- maybe `GET /aggregate/:grid/:hour`

## 8) Privacy
- Coarse grids by default
- Raw logs → keep 30d max
- Opt-in for fine GPS