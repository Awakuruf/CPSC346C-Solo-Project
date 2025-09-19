# Project Spec - CatchChance

## 1) User & Decision
**User:** Recreational anglers using mobile/web app.  
**Decision:** Whether to fish during a given session or which hour to target based on predicted probability of catching at least one fish.  
**Rationale:** Helps anglers optimize time on water and manage expectations; reduces frustration and wasted effort.  

## 2) Target & Horizon
**Target:** Binary — catch ≥1 fish during the next fishing session.  
**Horizon:** Short-term: next session (~2–4 hours) with optional hourly breakdown.  
**Output:** Probability (0–1) per coarse bucket (morning / afternoon / evening / night); optional per hour.  

## 3) Features (No Leakage)
- Input features: time-of-day, location (coarse grid), historical catch rates for user/region, weather (wind, temp, precipitation), lunar phase.  
- Excluded: future weather not yet observed, other users’ private history (unless aggregated), real-time fish stock data (not available).  
- Reasoning: Avoid leakage by using only data available at prediction time.  

## 4) Baseline → Model Plan
**Baseline Rule/Heuristic:**  
- Predict probability using historical average catch rate for that coarse time-of-day in the same location.  

**Simple Model:**  
- Logistic regression / Gradient Boosted Tree using historical session features.  
- Hypothesis: model will capture interactions (e.g., morning + high wind = lower probability) better than baseline.  

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