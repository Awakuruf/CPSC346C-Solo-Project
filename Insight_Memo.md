# Insights

**Notes / Initial Observations (Specifically to fishing data statistics):**  
- Hypothesis: Catch probability varies by time-of-day; morning may have higher catch rates.  
- Question: Does hourly granularity provide significantly more predictive power than coarse buckets?  
- Placeholder: No data analyzed yet.  

1. Hour-of-day seems predictive even without environment data.
2. Data sparsity will be an issue → smoothing + aggregation needed.
3. Privacy matters more than accuracy at first → better to underfit than leak data.

### 1. Free Tier Insights:  
**Serverless free tiers fail *sooner* than expected under viral load**  
I assumed AWS Lambda or GCP Functions free tiers would survive a 10× spike without cost. In practice, cold-start overhead + per-invocation billing eats through free quotas quickly at ~50k req/hr.  

**Evidence:** Cost model simulation (`analysis/cost_model.ipynb`, cell 12) shows Lambda’s 1M free requests/month burn out in <1 day at viral load.  
* TODO - Evidence based on AWS insights 

**Why it matters:** My design must either (a) throttle non-essential requests, or (b) pre-compute/coarsen predictions (by hour, not minute) to reduce calls. This changes architecture from naive “predict on every hit” to caching results.  

**Limits:** Based on synthetic request trace, not real traffic; assumes uniform geographic distribution.  

**Next question:** How do caching + rate-limits affect fairness for new vs. repeat users?  

### 2. Privacy vs Accuracy Insight
**Privacy matters more than accuracy at first → better to underfit than leak data.**  

**Evidence:** The core principle of privacy-preserving machine learning, particularly differential privacy, involves deliberately trading accuracy for privacy. This is a critical countermeasure against attacks like "membership inference," where a complex, overfit model can leak information about its training data. Overfitting, by definition, involves the model memorizing specific data points, making it a direct privacy risk. By starting with a simpler model (that may underfit initially), we build a more generalized and robust solution that is inherently less susceptible to these vulnerabilities, protecting user trust from the outset.

**Why it matters:** This insight addresses the project's long-term Survival by prioritizing user trust and ethical responsibility. A data breach or a perception of privacy violations could be a catastrophic business failure, regardless of the app's predictive accuracy. A privacy-first approach ensures the long-term viability of the product, as it builds a foundation of trust with the user base.

**Limits:** The degree to which a model is underfit must be carefully balanced; too simple a model may have no practical utility. Furthermore, while underfitting can mitigate some privacy risks, it is not a complete solution. Other robust techniques like k-anonymity or federated learning may be necessary for comprehensive data protection.

**Next question:** What specific privacy-preserving techniques (e.g., differential privacy, k-anonymity, data de-identification) are most suitable for our application, and what is the minimum acceptable accuracy loss for each? How will we test for and prove that our model does not leak private information?
