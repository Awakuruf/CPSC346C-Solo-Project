# Insights

**Notes / Initial Observations (Specifically to fishing data statistics):**  
- Hypothesis: Catch probability varies by time-of-day; morning may have higher catch rates.  
- Question: Does hourly granularity provide significantly more predictive power than coarse buckets?  
- Placeholder: No data analyzed yet.  

1. Free-tier infra means model must be tiny.
2. Privacy matters more than accuracy at first → better to underfit than leak data.
3. Data sparsity will be an issue → smoothing + aggregation needed.

### 1. Free Tier Insights:  
**Serverless free tiers fail *sooner* than expected under viral load**  
I assumed AWS Lambda or GCP Functions free tiers would survive a 10× spike without cost. In practice, cold-start overhead + per-invocation billing eats through free quotas quickly at ~50k req/hr.  

**Evidence:** Cost model simulation (`analysis/cost_model.ipynb`, cell 12) shows Lambda’s 1M free requests/month burn out in <1 day at viral load.  
* According to AWS’s official pricing documentation, the Lambda free tier includes 1 million free requests and 400,000 GB-seconds of compute time per month [AWS Lambda Pricing](https://aws.amazon.com/lambda/pricing/). At viral traffic rates, this allotment can be consumed in well under a day: for example, at just 1,000 requests per second—a modest “viral” load—an application would process 3.6 million requests in a single hour, exhausting the 1 million free requests in less than 20 minutes. This shows that while the Lambda free tier may suffice for low-traffic workloads, it is quickly exceeded once a function experiences rapid adoption or viral growth.

**Why it matters:** My design must either (a) throttle non-essential requests, or (b) pre-compute/coarsen predictions (by hour, not minute) to reduce calls. This changes architecture from naive “predict on every hit” to caching results.  

**Limits:** Based on synthetic request trace, not real traffic; assumes uniform geographic distribution.  

**Next question:** How do caching + rate-limits affect fairness for new vs. repeat users?  

### 2. Privacy vs Accuracy Insight
**Privacy matters more than accuracy at first → better to underfit than leak data.**  

**Evidence:** The core principle of privacy-preserving machine learning, particularly differential privacy, involves deliberately trading accuracy for privacy. This is a critical countermeasure against attacks like "membership inference," where a complex, overfit model can leak information about its training data. Overfitting, by definition, involves the model memorizing specific data points, making it a direct privacy risk. By starting with a simpler model (that may underfit initially), we build a more generalized and robust solution that is inherently less susceptible to these vulnerabilities, protecting user trust from the outset.

**Why it matters:** This insight addresses the project's long-term Survival by prioritizing user trust and ethical responsibility. A data breach or a perception of privacy violations could be a catastrophic business failure, regardless of the app's predictive accuracy. A privacy-first approach ensures the long-term viability of the product, as it builds a foundation of trust with the user base.

**Limits:** The degree to which a model is underfit must be carefully balanced; too simple a model may have no practical utility. Furthermore, while underfitting can mitigate some privacy risks, it is not a complete solution. Other robust techniques like k-anonymity or federated learning may be necessary for comprehensive data protection.

**Next question:** What specific privacy-preserving techniques (e.g., differential privacy, k-anonymity, data de-identification) are most suitable for our application, and what is the minimum acceptable accuracy loss for each? How will we test for and prove that our model does not leak private information?

### 3. Telemetry Feedback

**The biggest challenge for long-term model improvement is "data drift," not the initial model's accuracy. A robust telemetry feedback loop is the only way to solve this.**

**Evidence:** Machine learning models are trained on historical data, but the real world is constantly changing. A phenomenon known as data drift occurs when the statistical properties of the incoming data in production shift over time, causing the model's accuracy to degrade. This could happen if, for example, new fishing regulations are introduced, or a new type of lure becomes popular, changing the underlying data patterns that our model was trained on. Sources on telemetry in machine learning and feedback loops confirm that continuous monitoring is the primary way to detect and adapt to this drift. The central idea is to create a closed-loop system where our predictions are compared to real-world outcomes (user-logged catches) and that new, corrected data is fed back into the training pipeline. Without this loop, our model's initial high accuracy would be a "one-and-done" event, destined to decay as the world changes around it.

**Why it matters:** This insight addresses the project's long-term Survival. The project's success isn't just about a good MVP; it's about building a product that gets better over time. Our initial model performance is less important than our ability to adapt. This means that a key component of our MVP must be the logging and feedback system, not just the prediction algorithm. This telemetry system will be the engine of our long-term growth and success.

**Limits:** Designing a robust telemetry system requires careful planning to avoid capturing irrelevant or biased data. We must ensure that our feedback loop doesn't inadvertently create its own biases (e.g., if a model recommends fishing spots that are already popular, it might get more data from those spots and amplify the bias).

**Next question:** What are the most critical data points we must capture in our telemetry, and how can we design a feedback loop that is both cost-effective and resilient to bias?