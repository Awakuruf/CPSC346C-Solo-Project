# Capstone Pitch Report Template (2 Pages)

> Keep your report under two pages. Use this outline to scaffold your pitch. Replace prompt text with your content.

## 1. Core Pitch Idea
- Describe the problem
- Describe your solution
- Identify the monetization model

## 2. Problem & Stakeholders
- Scenario summary (include jurisdictional constraints)
- Primary stakeholders + empty chair (who is left out?)
- Pain points / goals

## 3. Architecture Snapshot
- Diagram or textual description of target architecture
- Key services / data flows / trust boundaries
- Highlight trade-offs (cost / ethics / reliability)

## 4. Clause → Control → Test
| Clause (Promise) | Control (Implementation) | Test (Red Bar) |
| --- | --- | --- |
| Example: "Canadian health data stays in Canada." | Route 53 resolver in ca-central-1 + IAM policy restricting analysts to Canadian VPC | `test_canada_data_residency()` asserts no non-CA buckets referenced |
| Clause 1 | | |
| Clause 2 | | |

## 5. AI / Automation Usage Plan
- How you will use GenAI / automation tools
- Known failure modes + mitigation plan
- Documentation strategy (what will you publish?)

## 6. Risks & Mitigations
- Risk 1 (ethical, operational, cost, etc.) + mitigation + evidence/test
- Risk 2 + mitigation + evidence/test
- Success metrics / acceptance tests (tie back to Clause→Control→Test)

## Appendix (Optional)
- Supporting data (links to repo, ledger entry, etc.)
- Assumptions & open questions