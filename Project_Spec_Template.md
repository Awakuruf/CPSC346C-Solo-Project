# Project Spec - CatchChance

## 1) User & Decision
**User:** Recreational anglers using the mobile app (or website) to plan a session.
**Decision:** Whether to fish during a given session or which hour to target based on predicted probability of catching at least one fish. The product returns a short, actionable probability (e.g., “37% chance this afternoon”), and a suggested hour bucket if the user wants to maximize catch chance.
**Rationale:** Helps anglers optimize time on water and manage expectations; reduces frustration and wasted effort. Privacy-sensitive: location + behavioral history matter, so guardrails required.

## 2) Target & Horizon
**Target:** Binary outcome `caught_any` for a session (1 if the session yields ≥1 catch, else 0). The API returns a probability `p ∈ [0,1]` for that target.

**Horizon:** The next single fishing session starting within a specified hour bucket (request includes intended start time or chooses by hour). If no start time provided, the model scores the current hour bucket.

Granularity: hourly buckets by local hour (or coarse: morning/afternoon/evening/night). No leakage: only use data available **before** session start.

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