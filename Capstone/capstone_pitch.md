# Capstone Pitch Report - **MindMirror: A Privacy-Preserving Mental Health Companion**

## 1. Core Pitch Idea

**Problem:**
Most AI wellness apps (e.g., Replika, Wysa) collect deeply personal reflections and store them on U.S. servers, exposing users to data resale, surveillance, and opaque model behavior. This violates local data-protection norms like **PIPEDA**, **GDPR**, and **LGPD**, undermining user trust and autonomy.

**Proposed Solution:**
*MindMirror* is a **decentralized AI wellness chatbot** that processes all conversations **locally or in region-locked clouds**. It blends **mindfulness and Daoist reflection** to encourage calm, culturally inclusive self-inquiry without exporting sensitive data.
The app transparently signals when AI is used, encrypts all messages end-to-end, and allows users to delete or export data at will.

**Monetization / Impact Model:**
Freemium subscription:

* **Free:** Local inference, journaling, daily reflection mode
* **Premium:** Encrypted cloud sync, streak tracking, and personalized summaries
  Impact goal: Demonstrate that emotional AI can be both private and ethically grounded.

## 2. Problem & Stakeholders
**Scenario Summary:**
Digital wellness tools increasingly cross jurisdictions without respecting **data protection** laws. Under **PIPEDA (Canada)** and **GDPR (EU)**, emotional data may be considered sensitive personal information depending on context—particularly if it relates to health conditions or uses biometric processing. Both frameworks require appropriate consent and safeguards for international data transfers.


**Primary Stakeholders:**

* Everyday users seeking safe reflective tools
* Mental-health practitioners exploring digital adjuncts
* Regulators ensuring cross-border compliance

**Empty Chair:**
Users uncomfortable with Western mental-health framings—such as **culturally diverse or spiritually oriented populations**—who desire reflective dialogue without clinical or data-extractive overtones.

**Pain Points:**

* Lack of visibility into where data goes
* Distrust of AI tone or bias
* Need for calm, private reflection rather than “therapy replacement”

## 3. Architecture Snapshot
```mermaid
flowchart TB

%% === Define Styles ===
classDef user fill:#FFEBE5,stroke:#E67E22,stroke-width:2px,color:#000
classDef local fill:#E5F9E0,stroke:#27AE60,stroke-width:2px,color:#000
classDef cloud fill:#E3F2FD,stroke:#1E88E5,stroke-width:2px,color:#000
classDef boundary stroke-dasharray: 5 5,stroke:#555,stroke-width:1.5px

%% === User Layer ===
U["User (MindMirror App)"]:::user

%% === Local Processing ===
subgraph L["Local Device Boundary"]
direction TB
    FE["React-Native Frontend<br/>• AES-256 Encryption<br/>• IndexedDB Storage"]:::local
    TK["Tokenizer & Sentiment Filter<br/>• Pre-process text<br/>• Bias moderation"]:::local
    LM["Quantized LLM (Mistral 7B)<br/>• On-device inference<br/>• Reflection tone filter"]:::local
    KS["Local Key Store<br/>• Data deletion & export controls"]:::local
end
class L boundary

%% === Regional Cloud ===
subgraph C["Region-Locked Cloud (ca-central-1 / eu-west-1)"]
direction TB
    API["API Gateway<br/>• Signed & encrypted requests"]:::cloud
    LLM["Regional LLM Endpoint<br/>• Cloud Run container<br/>• Audit logs (metadata only)"]:::cloud
    S3["Encrypted S3 Bucket<br/>• Region-tagged storage<br/>• Sync only with user consent"]:::cloud
end
class C boundary

%% === Governance / Oversight ===
GOV["Model Governance Layer<br/>• Prompt audit scripts<br/>• Compliance tests<br/>• Red-bar verification"]:::cloud

%% === Data Flows ===
U --> FE
FE --> TK
TK --> LM
LM -->|Offline Mode| FE
LM -.->|Opt-in Cloud Inference| API
API --> LLM --> S3
S3 --> FE
FE --> KS
GOV -.-> LLM
GOV -.-> S3

%% === Trust Boundaries ===
L -.->|Trust Boundary<br/>End-to-End Encrypted| C
```

**Trade-offs:**

| Factor          | Decision                          | Rationale                         |
| --------------- | --------------------------------- | --------------------------------- |
| **Cost**        | Local inference increases compute | Prioritizes user sovereignty      |
| **Ethics**      | Region isolation + explainability | Aligns with privacy and fairness  |
| **Reliability** | Offline fallback                  | Works even without network access |

## 4. Clause → Control → Test
| Clause (Promise)                                                 | Control (Implementation)                                           | Test (Red Bar)                                                              |
| ---------------------------------------------------------------- | ------------------------------------------------------------------ | --------------------------------------------------------------------------- |
| 1. “User reflections never leave their jurisdiction.”            | VPC isolation + region-tagged S3 bucket (ca-central-1 / eu-west-1) | `test_region_lock()` asserts all writes reference correct regional endpoint |
| 2. “AI responses are always disclosed and logged transparently.” | UI badge + metadata tag (`source: AI`) + audit ledger              | `test_ai_disclosure()` checks every chat entry includes `source` metadata   |
| 3. “User data remains deletable and exportable.”                 | Local key-store + GDPR “Right to be Forgotten” API                 | `test_data_erasure()` deletes all user rows and verifies null return        |

## 5. AI / Automation Usage Plan
* **Model:** Mistral 7B (quantized for on-device use)
* **Automation:** nightly prompt-audit scripts ensure tone alignment; CI tests verify data-handling promises.
* **Assistive AI in Development:** GPT-4 used for code refactoring and architecture documentation (not user data).

**Failure Modes & Mitigation:**

* *Mode collapse / hallucination:* sentiment + toxicity filter before display.
* *Cultural bias:* community-sourced reflective corpus integrating Daoist, Stoic, and Indigenous perspectives.
* *Performance drift:* automatic regression tests on prompt clarity and latency.

**Documentation Strategy:**
Public **Model Card** + **Data Flow Diagram** outlining privacy guarantees, red-bar test results, and known limitations.

## 6. Risks & Mitigations
- Risk 1 (ethical, operational, cost, etc.) + mitigation + evidence/test
- Risk 2 + mitigation + evidence/test
- Success metrics / acceptance tests (tie back to Clause→Control→Test)

## Appendix (Optional)
- Supporting data (links to repo, ledger entry, etc.)
- Assumptions & open questions