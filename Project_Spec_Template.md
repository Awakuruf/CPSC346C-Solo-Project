# Project Spec - CatchChance

## 1) User & Decision
**User:** Recreational anglers using the mobile app (or website) to plan a session.
**Decision:** Whether to fish during a given session or which hour to target based on predicted probability of catching at least one fish. The product returns a short, actionable probability (e.g., “37% chance this afternoon”), and a suggested hour bucket if the user wants to maximize catch chance.
**Rationale:** Helps anglers optimize time on water and manage expectations; reduces frustration and wasted effort. Privacy-sensitive: location + behavioral history matter, so guardrails required.

## 2) Target & Horizon
**Target:** Binary outcome `caught_any` for a session (1 if the session yields ≥1 catch, else 0). The API returns a probability `p ∈ [0,1]` for that target.

**Horizon:** The next single fishing session starting within a specified hour bucket (request includes intended start time or chooses by hour). If no start time provided, the model scores the current hour bucket.

Granularity: hourly buckets by local hour (or coarse: morning/afternoon/evening/night). No leakage: only use data available **before** session start.

## 3) Features (No leakage)

**Allowed at prediction time**

- `hour_of_day` (local hour)
- `day_of_week`, `month` (seasonality)
- `location_grid` (coarse — e.g., 1 km² grid cell or 0.1° lat/lon bucket)
- recent *user* history in past 30 days: `n_sessions_past_30d`, `catch_rate_past_30d` — aggregated counts only (no raw timestamps)
- environmental features if available at request time: `water_temp` (if user/device shares or public API), `weather_brief` (wind high/low), `tide_state` (optional)
- `bait_type` (optional enumerated), `target_species` (optional enumeration)

**Excluded (to avoid leakage)**

- Any post-session labels (future catches) for the session we predict.
- Future environmental forecasts beyond the prediction start time that would incorporate future observations.
- Exact GPS traces / raw timestamps — only aggregated counts or coarse grid.
- **Reasoning**: Avoids leakage by using only data available at prediction time.  

**Notes:** make `location_grid` coarse by default; require opt-in for finer location.

## 4) Baseline → Model Plan
- Predict probability using historical average catch rate for that coarse time-of-day in the same location.  

**Baseline (immediate, defensible):**

- `baseline_p(hour, grid) = historical_catch_rate_by_hour_in_grid` computed as smoothed rate across last 90 days:
    
    `baseline_p = (alpha * prior_rate + observed_catches) / (alpha + observed_sessions)` with Laplace smoothing (alpha=10).
    
    This is implementable with a small DB query and fits free-tier.
    

**Simple Model (one candidate):**

- **Logistic regression** (or small gradient-boosted tree) with features listed above, trained to predict `caught_any`.
    
    Hypothesis: personalization (user-level catch_rate) + environment + hour-of-day improves CALIBRATION and ranking vs baseline.
    

Why simple: models are small (fast inference), interpretable, low-cost inference, and robust to small datasets.

**Simple Model:**  
- Logistic regression / Gradient Boosted Tree using historical session features.  
- Hypothesis: model will capture interactions (e.g., morning + high wind = lower probability) better than baseline.  

## 5) Possible Model
- Logistic regression with small feature set

## 6) Metrics, SLA, and Cost

**Primary metric (harms-aware):**

- **AUC-PR** for `caught_any` (good for imbalanced positive events) **and** calibration (Brier score / calibration curve). We care more about well-calibrated probabilities (so users can make rational decisions).

**Secondary metrics:**

- p95 latency, error rate, cache hit rate, cost per 10k predictions.

**SLA (product):**

- **p95 latency ≤ 300 ms** for personalized predictions (server-side inference).
- **99.5% availability** for health endpoints.
- **Cost envelope:** aim to keep **normal load (100 req/day)** within typical free tier (zero or near-zero). For budget planning pick: **max cost per 10k personalized predictions ≤ $1** (design goal). Under spike, system must survive (degrade gracefully) rather than incur unbounded costs.

**Rationale:** a 300 ms p95 keeps UX snappy on mobile; lightweight model inference and caching should make this achievable.

## 7) API Sketch

**Endpoints**

| Method | Path | Purpose | Auth? |
| --- | --- | --- | --- |
| POST | /v1/predict | Get probability for user session | Bearer |
| GET | /v1/health | Liveness & readiness | none |
| POST | /v1/report_session | (Opt-in) user reports session outcome (label) | Bearer |
| GET | /v1/aggregate/:grid/:hour | Public aggregated probability for grid/hour | none (cacheable) |

**Request example (POST /v1/predict)**

```json
{
  "user_id": "user-123",
  "location_grid": "g-53.123_-123.123",
  "hour": 14,
  "bait_type": "worm",
  "target_species": "trout",
  "opt_in_fine_location": false}

```

**Response (200)**

```json
{
  "score": 0.37,
  "bucket": "afternoon",
  "explanation": "based on 30-day catch rate and current water temp",
  "model_version": "v0.3"
}

```

**Auth scheme:** Authorization: `Bearer <API_KEY>` (API keys for clients; OAuth for 3rd-party integrations). Free-tier API keys limited to cached aggregate responses; personalized predictions require registration and opt-in.

**Rate limits:** default 60 req/min per API key; free-tier aggregated-only keys: 600 req/min (cache-backed). Under viral spike, higher-tier keys prioritized via token bucket.

## 8) Privacy
- Coarse grids by default
- Raw logs → keep 30d max
- Opt-in for fine GPS