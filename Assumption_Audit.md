# Assumption Log — CatchChance

**Project:** CatchChance — short-horizon fishing probability API

---

## Assumptions

| Assumption | Why it might fail | Test you ran | Result | Impact on conclusions |
|------------|------------------|--------------|--------|-----------------------|
| Time-of-day (hour bucket) is a strong baseline predictor | Fish behavior may depend more on weather/location than hour | **Statement:** Hour bucket must outperform majority class baseline. **Risk:** If false, baseline adds no value. **Evidence so far:** Fishing guides cite morning/evening peaks. **Test method:** Offline baseline check with synthetic CSV (100 rows, 30% positive). **Acceptance criterion:** ≥5% precision gain over majority. **Next action:** If fails, prioritize weather/location as baseline. | ~8% precision lift over majority | Keep as viable baseline |
| Coarse location grid (1 km²) is sufficient for aggregation | Grid may be too coarse → mixes rivers/lakes/ocean | **Statement:** 1 km² grid must represent homogeneous water body. **Risk:** If false, predictions are misleading. **Evidence so far:** Map samples show mixed environments. **Test method:** Schema inspection + coastal grid spot check. **Acceptance criterion:** ≥80% of sampled cells = single water type. **Next action:** If fails, add water_body_type feature. | Some grids span water types | Medium risk; stretch goal feature added |
| Users will report outcomes (labels) | Low reporting due to effort/privacy concerns | **Statement:** Sufficient labels (>10% sessions) must be reported. **Risk:** If false, model cannot improve beyond baseline. **Evidence so far:** Comparable apps get ~5–15%. **Test method:** Synthetic probe plan (assume 5%, 10%, 15% reporting). **Acceptance criterion:** ≥10% sustainable. **Next action:** If lower, ensure baseline works without labels. | 5% scenario = very slow growth | Must design API for sparse labels |


## Initial Guess
Baseline target = time-of-day (morning, afternoon, evening, night).  
Stretch goals = add weather, location, user history.  