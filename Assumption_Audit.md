# Assumption Log — CatchChance

**Project:** CatchChance — short-horizon fishing probability API  
**Sprint window:** Week 1 (Day 0–7)

---

## Assumptions (Day 0–1)

| Assumption | Why it might fail | Test you ran | Result | Impact on conclusions |
|------------|------------------|--------------|--------|-----------------------|
| Time-of-day (hour bucket) is a strong baseline predictor | Fish behavior may depend more on weather/location than hour | Offline baseline check with synthetic CSV (100 sessions, labels ~30% pos, stratified by hour bucket) | Majority class baseline = 30%. Hour bucket rule improves precision by ~8% | Keep assumption as viable baseline |
| Coarse location grid (1 km²) is sufficient for aggregation | Grid may be too coarse → mix of rivers/lakes/ocean | Schema inspection + map check (example coastal grid) | Some grids span different water types | Flagged as medium-risk; may need water-body type feature |
| Users will report outcomes (labels) | Low reporting due to effort or privacy concerns | Synthetic probe plan: assume 5%–15% report rate | If 5%, training set grows very slowly | Must design API to work with very sparse labels; baseline not reliant on user reports |



## Initial Guess
Baseline target = time-of-day (morning, afternoon, evening, night).  
Stretch goals = add weather, location, user history.  