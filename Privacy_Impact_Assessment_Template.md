# PIA scratch

## 1. Overview

**Project purpose:** Short-horizon fishing probability API (CatchChance)  
**Scope:** Collect user fishing session data to predict short-term catch probability.  

**Data processing summary:**  
- Collection → Processing → Storage → Sharing → Retention  
- Primary data: user session start/end, catch outcomes, GPS (coarse), timestamp  
- Derived data: hour-of-day bucket, location grid aggregation, user 30-day history  

## 2. Data Inventory

| Field | Purpose | Lawful Basis | Minimization | Retention | Access Roles |
|-------|---------|-------------|-------------|----------|-------------|
| Session timestamp | Calculate activity patterns | Consent | Only hour bucket used for analytics | 30 days raw, aggregated indefinite | Analytics team |
| GPS (coarse) | Location-based predictions | Consent | 1 km² grid | 30 days raw, aggregated indefinite | Analytics team |
| Catch outcome | Model labels | Consent | Only outcome stored | 30 days raw, aggregated indefinite | Analytics team |
| User ID (hashed) | Personalization | Consent | Hash with daily salt | 30 days raw, aggregated indefinite | Model team |
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

## Risks
- Re-identification if grid too fine
- Misuse of location history
- Data poisoning (fake reports)

## Mitigations
- k-anonymity for aggregates
- No raw GPS unless opt-in
- Raw logs deleted after 30 days