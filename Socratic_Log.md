# Socratic Log

### 1. Seed Idea

**Motive:** Capture design intent and see how to expand the seed idea.  

**Context:** Exploring initial design space for a prediction service.  

**Prompt A (design alternative):**  
> “I want to create a Prediction API for prediction target being how likely you are to catch a fish on different hours of the days (morning, afternoon, evening, etc).”  
→ Option considered: start with a coarse, time-of-day target (4 buckets).  

**Prompt B (red-team):**  
> “Isn’t this too simplistic — shouldn’t you include weather, location, or season instead of just time-of-day?”  
→ Risk checked: over-simplification may limit predictive power.  

**Inflection point:**  
- Realized starting too complex could stall progress, but starting too simplistic might undercut project value.  
- Decided to frame time-of-day as the *minimum viable baseline target* and note weather/location as stretch goals for later iterations.  

**Evidence:** No code yet; just project sketch notes.  

**Outcome:** Defined project identity as **CatchChance**, with prediction target = probability of catching at least one fish by coarse time-of-day.  

**Attribution:** Seed phrasing and example prompts came from me; AI helped structure the decision into baseline vs stretch-goal framing.  

### 2. Brainstorm Kickoff
**Context:** Initial brainstorming on predicting fish-catching probabilities.  

**Prompt A (Design Alternatives):**  
> “List 2–3 ways to model short-horizon catch probability using coarse time-of-day and optional hourly breakdown.”  
**Option Tested:** Baseline probability per hour bucket (majority-class + Laplace smoothing).  

**Prompt B (Red-Team):**  
> “What are the limitations or risks of relying on time-of-day alone?”  
**Risk Checked:** Ignores weather, location variability, and individual angler skill.  

**Inflection Point:**  
Decided baseline is viable for early testing but must later incorporate weather and location features.  

**Evidence:**  
Offline baseline check on synthetic CSV (100 rows, 30% positive).  

**Outcome:**  
Kept hour-bucket baseline; planned feature expansion for weather/location.  

**Attributions:**  
- AI: Suggested alternative baselines and red-team critique  
- You: Selected baseline approach and designed test  

### 3. Baseline Probabilities
**Context:** Early discussion on caching baseline probabilities.  

**Prompt A (Design Alternatives):**  
> “Suggest caching strategies for grid × hour probability tables to reduce API load.”  
**Option Tested:** Precompute hourly probabilities per coarse location grid.  

**Prompt B (Red-Team):**  
> “Could caching cause stale or misleading predictions?”  
**Risk Checked:** Weather/location changes could make cached predictions inaccurate.  

**Inflection Point:**  
Added plan for daily refresh and optional cache invalidation on weather update.  

**Evidence:**  
Back-of-envelope estimate: 10k queries/day → 70% cache hit rate.  

**Outcome:**  
Caching justified; daily refresh implemented to maintain accuracy.  

**Attributions:**  
- AI: Suggested caching alternatives and risks  
- You: Designed refresh strategy and simulated load  

### 4. Privacy
**Context:** Privacy and consent for precise location data.  

**Prompt A (Design Alternatives):**  
> “List ways to enforce k-anonymity for location grid data and obtain opt-in consent.”  
**Option Tested:** Require k ≥ 10 per public aggregate; pilot consent flow for fine-grained location.  

**Prompt B (Red-Team):**  
> “Could users be re-identified with small k or opt-in confusion?”  
**Risk Checked:** Potential leakage if grid cells are too small or opt-in is unclear.  

**Inflection Point:**  
Added consent pilot; enforced k-anonymity on all aggregates.  

**Evidence:**  
Mock dataset inspection; k≥10 threshold verified.  

**Outcome:**  
Privacy policy strengthened; opt-in pilot initiated.  

**Attributions:**  
- AI: Highlighted re-identification risks and options  
- You: Implemented pilot and enforced k-anonymity  

### 5. SLA and Latency
**Context:** SLA and latency evaluation for prediction API.  

**Prompt A (Design Alternatives):**  
> “Suggest strategies to meet p95 ≤ 300 ms at 100 RPS with minimal cost.”  
**Option Tested:** Precompute cache + lightweight probability lookup.  

**Prompt B (Red-Team):**  
> “What scenarios might break SLA or raise cost?”  
**Risk Checked:** Cold starts, high traffic bursts, and network variability.  

**Inflection Point:**  
Added pre-warming for Lambda and monitored queue lengths to maintain SLA.  

**Evidence:**  
Synthetic probe: estimated p95 ~200 ms at 100 RPS.  

**Outcome:**  
SLA achievable with pre-warming and caching strategy.  

**Attributions:**  
- AI: Suggested latency mitigation strategies  
- You: Implemented pre-warming and monitored API performance  