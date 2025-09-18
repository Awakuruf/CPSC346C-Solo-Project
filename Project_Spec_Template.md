# Project Spec - CatchChance

## 1) User & Decision
- Who: recreational anglers
- Decision: should I fish now or later?

## 2) Target & Horizon
- Target: `caught_any` = 1 if ≥1 fish, else 0
- Horizon: one session (≈1 hour)

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