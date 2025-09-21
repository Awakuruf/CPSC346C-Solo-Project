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

## 5) Metrics, SLA, and Cost

**Primary metric (harms-aware):**

- **AUC-PR** for `caught_any` (good for imbalanced positive events) **and** calibration (Brier score / calibration curve). We care more about well-calibrated probabilities (so users can make rational decisions).

**Secondary metrics:**

- p95 latency, error rate, cache hit rate, cost per 10k predictions.

**SLA (product):**

- **p95 latency ≤ 300 ms** for personalized predictions (server-side inference).
- **99.5% availability** for health endpoints.
- **Cost envelope:** aim to keep **normal load (100 req/day)** within typical free tier (zero or near-zero). For budget planning pick: **max cost per 10k personalized predictions ≤ $1** (design goal). Under spike, system must survive (degrade gracefully) rather than incur unbounded costs.

**Rationale:** a 300 ms p95 keeps UX snappy on mobile; lightweight model inference and caching should make this achievable.

## 6) API Sketch

**6.1 Endpoints**

| Method | Path | Purpose | Auth? |
| --- | --- | --- | --- |
| POST | /v1/predict | Get probability for user session | Bearer |
| GET | /v1/health | Liveness & readiness | none |
| POST | /v1/report_session | (Opt-in) user reports session outcome (label) | Bearer |
| GET | /v1/aggregate/:grid/:hour | Public aggregated probability for grid/hour | none (cacheable) |

**6.2 Request example (POST /v1/predict)**

```json
{
  "user_id": "user-123",
  "location_grid": "g-53.123_-123.123",
  "hour": 14,
  "bait_type": "worm",
  "target_species": "trout",
  "opt_in_fine_location": false
  }

```
**Request fields:**

* `user_id` *(string, hashed)* → pseudonymous identifier for personalization.
* `location_grid` *(string)* → coarse lat/long grid cell (1 km²).
* `hour` *(int, 0–23)* → local hour of day (bucket).
* `bait_type` *(string, optional)* → user input; can inform model features.
* `target_species` *(string, optional)* → filter probabilities for target species.
* `opt_in_fine_location` *(boolean)* → whether user consented to higher-resolution GPS use.

**6.2 Response (200)**

```json
{
  "score": 0.37,
  "bucket": "afternoon",
  "explanation": "based on 30-day catch rate and current water temp",
  "model_version": "v0.3"
}

```
**Response fields:**

* `score` *(float, 0–1)* → predicted probability of at least one catch.
* `bucket` *(string)* → coarse time-of-day category (morning/afternoon/evening/night).
* `explanation` *(string)* → short rationale for transparency (features, data used).
* `model_version` *(string)* → current model version for reproducibility.

---
**6.3 Auth scheme:**

* Header: `Authorization: Bearer <API_KEY>`
* Free-tier API keys → access only to cached aggregate endpoints.
* Registered (opt-in) users → access to personalized predictions.
* OAuth flow reserved for 3rd-party partner integrations.

**Rate limits:**

* Default: 60 req/min per registered API key.
* Free-tier aggregate keys: 600 req/min (served from cache).
* Spike handling: token bucket prioritizes higher-tier keys; cache ensures graceful degradation.

## 7) Privacy, Ethics, Reciprocity (PIA excerpt)

**Data inventory**

- **Personal data:** `user_id`, last-30d aggregated history (`n_sessions`, `n_catches`) — stored pseudonymized.
- **Location:** coarse grid by default; fine location only with explicit opt-in.
- **Telemetry:** latency, error rates, anonymized counters.

**Purpose limitation:** only used to predict `caught_any` for next session and improve model with opt-in labels. No advertising, no sale of data.

**Retention**

- **Raw session records:** retained **30 days** then aggregated to counts.
- **Aggregates / model training dataset:** retained **2 years** as anonymized aggregates.
- **User opt-out:** user can request deletion of raw history; we keep aggregates needed for global models (with k-anonymity).

**Access:** only engineering & model pipeline service accounts; least privilege enforced by IAM; no manual exports without audit.

**Telemetry decision matrix (value vs invasiveness vs effort)**

| Telemetry item | Value | Invasiveness | Store? | TTL |
| --- | --- | --- | --- | --- |
| request latency | high | low | yes | 90 days |
| error traces (no PII) | high | low | yes | 180 days |
| request metadata (user_id hashed, grid) | medium | medium | yes | 30 days |
| raw GPS traces | low | high | **no** (unless opt-in) | n/a |

**Guardrails**

- **k-anonymity threshold**: do not display or expose aggregates for grid-hour combos with `n_sessions < k` (`k=10`) — return “insufficient data” or broaden the grid/time window.
- **Jitter / aggregation:** when serving public aggregate endpoints, add small Laplace noise to published rates to reduce re-identification risk.
- **Retention:** raw session logs => 30 days then aggregated; deletion on user request.
- **Consent:** fine-grained location or identifiable sharing requires explicit opt-in and clear disclosure.
- **Disclosure text:** short, user-facing privacy notice on first-use + link to full PIA explaining aggregate retention and opt-out.

**Reciprocity:** users receive direct value (personalized probabilities), and optionally an anonymized community dashboard (e.g., top catch-hours) to increase perceived reciprocity.

## 8) Architectural Diagram
TODO

## 9) Risks & Mitigations

1. **Cost blow-up during viral spike**
    - *Mitigation:* Rate limits, tiered access, cache-first architecture, degrade to cached/baseline responses for free tier, hard budget cutoff (circuit-breaker) to prevent runaway cloud costs.
    - *Acceptance criterion:* under a 50k req/hr spike, system returns cached/baseline responses with p95 ≤ 500 ms for cached responses and no unbounded spend > preset budget.
2. **Privacy / re-identification via location/time combination**
    - *Mitigation:* k-anonymity (k≥10) for exposed aggregates, coarse grids by default, jitter/noise on published aggregates, opt-in for fine location.
3. **Poor model calibration / harm (misleading probability)**
    - *Mitigation:* evaluate calibration (Brier), recalibrate using Platt scaling/isotonic, expose uncertainty, slow roll updates, require minimal performance improvement over baseline before release.
4. **Data poisoning from adversarial labels (fake reports)**
    - *Mitigation:* weight opt-in labels by trust score, verify with heuristics, do not automatically retrain on unverified labels.