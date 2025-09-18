# PIA scratch


**Project purpose:** Short-horizon fishing probability API (CatchChance)  
**Scope:** Collect user fishing session data to predict short-term catch probability.  

**Data processing summary:**  
- Collection → Processing → Storage → Sharing → Retention  
- Primary data: user session start/end, catch outcomes, GPS (coarse), timestamp  
- Derived data: hour-of-day bucket, location grid aggregation, user 30-day history  

## Data we collect
- user_id (hashed)
- coarse location grid
- aggregate history (30 days)
- optional environment, bait, target species

## Risks
- Re-identification if grid too fine
- Misuse of location history
- Data poisoning (fake reports)

## Mitigations
- k-anonymity for aggregates
- No raw GPS unless opt-in
- Raw logs deleted after 30 days