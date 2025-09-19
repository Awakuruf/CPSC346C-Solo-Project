# Insights

**Notes / Initial Observations:**  
- Hypothesis: Catch probability varies by time-of-day; morning may have higher catch rates.  
- Question: Does hourly granularity provide significantly more predictive power than coarse buckets?  
- Placeholder: No data analyzed yet.  

1. Hour-of-day seems predictive even without environment data.
2. Data sparsity will be an issue → smoothing + aggregation needed.
3. Privacy matters more than accuracy at first → better to underfit than leak data.

### 1. Insight:  
**Serverless free tiers fail *sooner* than expected under viral load**  
I assumed AWS Lambda or GCP Functions free tiers would survive a 10× spike without cost. In practice, cold-start overhead + per-invocation billing eats through free quotas quickly at ~50k req/hr.  

**Evidence:** Cost model simulation (`analysis/cost_model.ipynb`, cell 12) shows Lambda’s 1M free requests/month burn out in <1 day at viral load.  
* TODO - Evidence based on AWS insights 

**Why it matters:** My design must either (a) throttle non-essential requests, or (b) pre-compute/coarsen predictions (by hour, not minute) to reduce calls. This changes architecture from naive “predict on every hit” to caching results.  

**Limits:** Based on synthetic request trace, not real traffic; assumes uniform geographic distribution.  

**Next question:** How do caching + rate-limits affect fairness for new vs. repeat users?  
