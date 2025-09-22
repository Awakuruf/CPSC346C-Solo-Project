# Assumption Log — CatchChance

**Project:** CatchChance — short-horizon fishing probability API
**Sprint window:** Sept 18–22, 2025 (Due to joining from waitlist on 16th.)

---  

## Assumptions

| Assumption | Why it might fail | Test you ran | Result | Impact on conclusions |
|------------|------------------|--------------|--------|-----------------------|
| Time-of-day (hour bucket) is a strong baseline predictor | Fish behavior may depend more on weather/location than hour | **Statement:** Hour bucket must outperform majority class baseline. **Risk:** If false, baseline adds no value. **Evidence so far:** Fishing guides cite morning/evening peaks. **Test method:** Offline baseline check with synthetic CSV (100 rows, 30% positive). **Acceptance criterion:** ≥5% precision gain over majority. **Next action:** If fails, prioritize weather/location as baseline. | ~8% precision lift over majority | Keep as viable baseline |
| Coarse location grid (1 km²) is sufficient for aggregation | Grid may be too coarse → mixes rivers/lakes/ocean | **Statement:** 1 km² grid must represent homogeneous water body. **Risk:** If false, predictions are misleading. **Evidence so far:** Map samples show mixed environments. **Test method:** Schema inspection + coastal grid spot check. **Acceptance criterion:** ≥80% of sampled cells = single water type. **Next action:** If fails, add water_body_type feature. | Some grids span water types | Medium risk; stretch goal feature added |
| Users will report outcomes (labels) | Low reporting due to effort/privacy concerns | **Statement:** Sufficient labels (>10% sessions) must be reported. **Risk:** If false, model cannot improve beyond baseline. **Evidence so far:** Comparable apps get ~5–15%. **Test method:** Synthetic probe plan (assume 5%, 10%, 15% reporting). **Acceptance criterion:** ≥10% sustainable. **Next action:** If lower, ensure baseline works without labels. | 5% scenario = very slow growth | Must design API for sparse labels |
| Baseline probability can be cached at grid×hour | Cache hit rate may be low if queries are scattered | **Statement:** ≥60% of queries should hit cache. **Risk:** If false, DB load increases significantly. **Evidence so far:** Usage logs from similar APIs suggest clustering. **Test method:** Back-of-envelope: 10k queries/day, assume 70% overlap. **Acceptance criterion:** ≥60% cache hit rate. **Next action:** If fails, add user-specific cache. | ~70% overlap estimate → ~7k queries saved/day | High payoff; caching justified |
| Laplace smoothing (α=10) stabilizes rare buckets | Too much smoothing makes predictions unresponsive | **Statement:** Smoothing must prevent 0%/100% estimates. **Risk:** If false, unstable predictions erode trust. **Evidence so far:** Synthetic CSV shows extreme values with α=1. **Test method:** Compare α=1 vs α=10 on small dataset. **Acceptance criterion:** No bucket = 0% or 100%. **Next action:** If fails, tune α adaptively. | α=10 avoided extremes, α=1 was noisy | Keep α=10 default | SLA: p95 latency ≤ 300 ms at 100 RPS | Cold starts or network hops may exceed target | **Statement:** Service must meet SLA target under load. **Risk:** If false, UX degrades. **Evidence so far:** AWS calculator + queueing model. **Test method:** Synthetic probe plan for 100 RPS. **Acceptance criterion:** p95 ≤ 300 ms. **Next action:** If fails, explore pre-warming or lower RPS guarantee. | Est. 200 ms p95 achievable | SLA feasible but needs later validation | Logistic regression improves calibration vs baseline | Small dataset may overfit | **Statement:** Logistic regression should lower Brier score vs baseline. **Risk:** If false, no gain from added complexity. **Evidence so far:** Synthetic 500-row sample. **Test method:** Offline baseline vs logistic regression. **Acceptance criterion:** ≥10% Brier reduction. **Next action:** If fails, stick with baseline. | Brier ↓15% with logistic regression | Kept logistic regression as candidate model beyond baseline. |
| Privacy guardrail: k ≥ 10 for public aggregates | Many grid×hour cells have <10 sessions | **Statement:** All published aggregates must meet k ≥ 10. **Risk:** If false, privacy leakage. **Evidence so far:** Mock dataset probe. **Test method:** Synthetic suppression at k=10. **Acceptance criterion:** 0 cells < k. **Next action:** If too many suppressed, relax granularity. | ~40% suppressed at k=10 | Acceptable trade-off; must explain to users |
| Cost envelope ≤ $1 / 10k predictions | Egress or model load underestimated | **Statement:** Cost per 10k must stay under \$1. **Risk:** If false, system unsustainable. **Evidence so far:** AWS calculator (API Gateway + Lambda + Dynamo). **Test method:** Back-of-envelope cloud cost estimate. **Acceptance criterion:** ≤ $1 per 10k. **Next action:** If fails, optimize cache + cheaper storage. | ~$0.40 / 10k | Holds for baseline; risk under spikes | Assumption | Statement | Risk if false | Evidence so far | Test method (design-only) | Acceptance criterion | Next action| Consent model for fine location | Users understand opt-in | If false, non-compliance | Copy review | Onboarding wording review | ≥80% correct comprehension | Pilot test wording before rollout |

## Sensitivity / Spec Grid

| Spec | Description | Brier Score |
| --- | --- | --- |
| A | Hour-bucket only baseline | 0.24 (synthetic) |
| B | Add user 30-day history | 0.20 |
| C | Add location grid + weather | 0.18 |

**Conclusion:** Improvements are incremental; baseline is already useful.

## Actions Taken

- **Kept:** hour-bucket baseline, Laplace smoothing α=10
- **Changed:** location grid flagged as coarse → added `water_body_type` as stretch goal
- **Dropped:** reliance on high label reporting; model plan assumes sparse labels
- **Added:** privacy guardrail (k=10 suppression) after red-team prompt