# Privacy Impact Assessment (PIA) 

## 1. Overview

**Project purpose:** Short-horizon fishing probability API (CatchChance)  
**Scope:** Collect user fishing session data to predict short-term catch probability.  

**Data processing summary:**  
- Collection → Processing → Storage → Sharing → Retention  
- Primary data: user session start/end, catch outcomes, GPS (coarse), timestamp  
- Derived data: hour-of-day bucket, location grid aggregation, user 30-day history  
* One thing we need to be careful is how cloud provider access logs may capture IPs and referrers; mitigated by short retention and masking.

## Telemetry Decision Matrix

| Metric / Log | Purpose | Value (1–5) | Invasive (1–5) | Effort (1–5) | Retention | Access | Keep? |
|--------------|---------|--------------|----------------|---------------|----------|--------|-------|
| Session start | Timing analysis | 4 | 2 | 2 | 30d raw | Analytics | Yes |
| Session end | Duration metrics | 4 | 2 | 2 | 30d raw | Analytics | Yes |
| Catch outcome | Model labels | 5 | 1 | 2 | 30d raw | Model | Yes |
| GPS (coarse) | Location baseline | 5 | 3 | 2 | 30d raw | Analytics | Yes |
| User ID (hashed) | Personalization | 4 | 2 | 3 | 30d raw | Model | Yes |
| Device UA | Debug | 2 | 1 | 1 | 7d raw | DevOps | Yes |

## Guardrails

- **k-anonymity threshold:** k ≥ 10 for public aggregates  
- **Jitter / aggregation:** Hour-bucket + 1 km² grid; optional sampling  
- **Raw TTLs:** 7–30 days depending on field  
- **Least-privilege access:** Roles limited to analytics/model/devops needs  
- **Disclosure / transparency:** Onboarding guide, PIA link  
- **Opt-in for sensitive features:** Fine-grained location, personalization

## 2. Data Inventory

| Field | Purpose | Lawful Basis | Minimization | Retention | Access Roles |
|-------|---------|-------------|-------------|----------|-------------|
| Session timestamp | Calculate activity patterns | Consent | Only hour bucket used for analytics | 30 days raw, aggregated 2-years | Analytics team |
| GPS (coarse) | Location-based predictions | Consent | 1 km² grid | 30 days raw, aggregated 2-years | Analytics team |
| Catch outcome | Model labels | Consent | Only outcome stored | 30 days raw, aggregated 2-years | Analytics team |
| User ID (hashed) | Personalization | Consent | Hash with daily salt | 30 days raw, aggregated 2-years | Model team |
| Device info (UA) | Debugging / session grouping | Legitimate interest | Only general type retained | 7 days raw, anonymized indefinitely | DevOps |

### Data we collect (Summarized)
- user_id (hashed)
- coarse location grid
- aggregate history (30 days)
- optional environment, bait, target species

## 3. Linkability & Identifiability

| Risk | Quasi-identifiers | Mitigation |
|------|-----------------|-----------|
| Session linkability | IP + timestamp + coarse GPS | Hash IP, coarse grid, daily salt |
| Cross-session tracking | Hashed user ID | Hash with rotation; optional opt-in for fine personalization |
| Derived PII | Hour-bucket + location | Only store aggregated stats for public display |

## 4. Purpose Limitation & Secondary Use

- **Declared purpose:** Predict short-term catch probability for users.  
- **Secondary use approval process:** Any new purpose requires explicit team approval and update to PIA.  
- **Function creep protections:** Default pipelines ignore unused fields; no raw GPS exported; logs purged after TTL.

## 5. Minimization & Retention

| Data Type | Aggregation / Bucketing | Raw TTL | Aggregate Retention | Deletion Plan |
|-----------|------------------------|---------|------------------|---------------|
| Session timestamp | Hour bucket | 30 days | Aggregated indefinitely | Automated daily deletion |
| GPS | 1 km² grid | 30 days | Aggregated indefinitely | Automated daily deletion |
| Catch outcome | Outcome only | 30 days | Aggregated indefinitely | Automated daily deletion |
| User ID | Hashed, daily salt | 30 days | Aggregated statistics | Automated daily deletion |


#### Data Minimization Trade-offs
| Option                                 | Capability for Prediction                                      | Privacy Risk (Re-identification, linkage)                      | Kept / Rejected | Rationale                                                       |
| -------------------------------------- | -------------------------------------------------------------- | -------------------------------------------------------------- | --------------- | --------------------------------------------------------------- |
| **Raw GPS (lat/long, full precision)** | Highest spatial accuracy; supports fine-grained catch modeling | Very high: reveals fishing locations, can link across sessions |  Rejected      | Too invasive; not required for hourly probability               |
| **1 km² grid cell (rounded GPS)**      | Preserves useful signal (e.g. coastal vs open water)           | Moderate: location fuzzed, harder to link to individuals       |  Kept          | Balances predictive value with reduced risk                     |
| **No GPS (time-only model)**           | Simplest; low privacy risk                                     | Low accuracy: ignores environmental variation                  |  Rejected      | Fails project’s functional requirement of contextual prediction |
| **Pseudonymous user ID (salted hash)** | Allows opt-in personalization, A/B evaluation                  | Low: salts rotated daily, no cross-session linkage             |  Kept          | Provides utility while constraining re-ID risk                  |

## 6. Access Control & Governance

| Role | Access Level | Notes |
|------|-------------|------|
| Analytics team | Read aggregated data | No raw GPS unless opt-in |
| Model team | Read anonymized hashed user IDs | For personalization features |
| DevOps | Debug logs (masked) | Only general device info; auto-purged |

## 7. Transparency & Choice

- “What we collect & why” posted in onboarding guide.  
- Optional opt-in for fine-grained location personalization.  
- Users can request export / deletion of data.

## 8. Security

- Threats: scraping, doxxing, insider misuse.  
- Mitigations: WAF, rate limits, access auditing, hashing sensitive fields, daily TTL.  
- Secrets: encrypted at rest; HTTPS in transit.

## 9. Compliance & Policy Alignment

- Course & org policies followed.  
- GDPR & PIPEDA considerations for opt-in & deletion.  
- Internal review documented in commits & PRs.


## 10. Residual Risks & Trade-offs

| Risk | Trade-off | Contingency |
|------|-----------|------------|
| Cold-start users | Personalization optional; no full benefit | Default to grid×hour baseline |
| Spike events | Cached baseline may mislead | Documented degraded mode; optional weather input |
| Partial reporting | Sparse labels limit model | Baseline works independently of user submissions |

## 11. Risks & Mitigations
1. **Cost blow-up during viral spike**
    - *Mitigation:* Rate limits, tiered access, cache-first architecture, degrade to cached/baseline responses for free tier, hard budget cutoff (circuit-breaker) to prevent runaway cloud costs.
    - *Acceptance criterion:* under a 50k req/hr spike, system returns cached/baseline responses with p95 ≤ 500 ms for cached responses and no unbounded spend > preset budget.
2. **Privacy / re-identification via location/time combination**
    - *Mitigation:* k-anonymity (k≥10) for exposed aggregates, coarse grids by default, jitter/noise on published aggregates, opt-in for fine location.
3. **Poor model calibration / harm (misleading probability)**
    - *Mitigation:* evaluate calibration (Brier), recalibrate using Platt scaling/isotonic, expose uncertainty, slow roll updates, require minimal performance improvement over baseline before release.
4. **Data poisoning from adversarial labels (fake reports)**
    - *Mitigation:* weight opt-in labels by trust score, verify with heuristics, do not automatically retrain on unverified labels.