# Smart Automation of Life Insurance Using AI and Multimodal Systems

>This final-year research project, conducted at the Sri Lanka Institute of Information Technology, explores AI-driven automation across four core life insurance operations: claims processing, regulatory compliance, risk assessment, and policy recommendation. The research investigates how artificial intelligence techniques, including machine learning, natural language processing, and predictive analytics, can improve operational efficiency, decision-making accuracy, customer experience, and regulatory adherence within the life insurance industry. The proposed framework aims to demonstrate the potential of AI to streamline insurance workflows while supporting data-driven business outcomes.
This project was developed in collaboration with **Union Assurance PLC**, whose domain expertise, business context, and data insights shaped the problem statements for each module. The technical research, architecture, and implementation are entirely the work of the project team.

**Live platform:** [smart-automation-of-life-insurance-us.netlify.app](https://smart-automation-of-life-insurance-us.netlify.app/)

---

## Overview

The platform consists of four independently deployed AI modules, each targeting a different bottleneck in the insurance lifecycle:

| # | Module | Core Problem |
|---|--------|-------------|
| 1 | Claims Automation | Manual, slow, fraud-prone claims processing |
| 2 | Sanction Detection | Incomplete AML/CFT screening at onboarding |
| 3 | Risk Assessment & Policy Optimization | Subjective underwriting and portfolio management |
| 4 | Policy Recommendation | Generic, one-size-fits-all policy suggestions |

---

## Module 01 — Claims Automation

**Live:** [insuranceclaimdashboard.pythonanywhere.com](https://insuranceclaimdashboard.pythonanywhere.com/)  
**Stack:** Python · Flask · XGBoost · OCR (Tesseract/LLAMA) · NLP  
**Frontend:** HTML · CSS · JavaScript

An end-to-end claims processing pipeline that combines document intelligence, fraud detection, and email triage.

**How it works:**

1. **Document ingestion** — OCR (with an LLM assist for extraction) pulls key fields from submitted PDFs, invoices, and receipts
2. **Fraud scoring** — an XGBoost classifier calculates a decision confidence score based on historical claim patterns
3. **Auto-approval / routing** — low-risk claims are settled straight-through; flagged ones are routed to an adjuster
4. **Email triage** — an NLP parser reads incoming emails, extracts urgency, sentiment, and intent, and feeds an email prioritisation modal for staff

Two portals are served: one for the customer (claim submission and status) and one for company staff (dashboard and case management).

**Results:**
- ~94% reduction in average settlement time vs. manual baseline
- ~60% lower operational cost per claim
- >91% fraud detection accuracy on test data

---

## Module 02 — Sanction Detection

**Live:** [chamindudenuwan.pythonanywhere.com](https://chamindudenuwan.pythonanywhere.com/)  
**Stack:** Python · Flask · RapidFuzz · spaCy · OpenCV  
**Frontend:** HTML · CSS · JavaScript

A multimodal screening engine that checks new policyholders against international and local sanctions watchlists in real time.

**Watchlists covered:**
- OFAC (US Specially Designated Nationals)
- UN Security Council Consolidated List
- EU Consolidated Sanctions List
- Local Sri Lanka watchlist

**Screening pipeline:**

1. **Data cleaning & normalisation** — standardises names, IDs, and images across list formats (CSV / XML / JSON)
2. **Staging store** — cleaned data is held in an intermediate staging layer before matching
3. **Name screening** — fuzzy matching catches transliterations, aliases, and typographic variants
4. **Attribute screening** — date of birth, nationality, and address used to disambiguate common names
5. **Facial recognition** — OpenCV-based biometric check against watchlist passport photos and selfies
6. **Fusion engine** — combines name score, attribute score, and biometric score into a single risk output (`Low / Medium / High / Blocked`)

Results are surfaced in a compliance dashboard for company staff.

**Results:**
- 99.8% screening accuracy
- Sub-second response per request
- Zero false negatives on known positive test cases

---

## Module 03 — Risk Assessment & Policy Optimization

**Live:** [taktuk-policyy-optimization-frontend.hf.space](https://taktuk-policyy-optimization-frontend.hf.space) — models, inference API, and UI all hosted on HF Spaces  
**Stack:** Python · Rule-based risk engine · Stable-Baselines3 (PPO) · SHAP · XGBoost · KMeans · FAISS · Sentence Transformers · Groq  
**Frontend:** HTML · CSS · JavaScript (served from HF Spaces)

The most technically layered module. It has two distinct components serving different users and purposes.

---

### Component 3A — Individual Customer Risk Assessment (Customer-facing)

A customer self-onboarding flow where each applicant is assessed in real time, held for human review, and only activated once approved.

**Flow:**

1. **Customer data entry** — the applicant fills in demographic, marital, financial, and lifestyle details (smoking, alcohol consumption, etc.) through the UI and clicks Proceed
2. **Pending state** — the customer is added to the system's pending list; their account is locked (no PDF download, no ticketing) until a company reviewer acts
3. **Risk scoring** — the system runs a **rule-based risk engine** to calculate a risk score and category for the customer based on their inputs
4. **PPO weight adjustment** — a PPO agent proposes updated risk weight adjustments tailored to this customer's feature profile
5. **Textual SHAP + GenAI reasoning** — SHAP values are translated into plain-language explanations of which features drove the risk score, supplemented by LLM-generated reasoning per customer
6. **Human-in-the-Loop (HIL) review** — a company staff member reviews the proposed weight adjustments and the customer's risk profile in the same interface. They can:
   - **Approve weights** → approved weights are versioned and used as the baseline for the next customer's calculation
   - **Reject weights** → a rejection reason must be provided
   - **Approve the customer** → account is activated; the customer can download their risk PDF and use ticketing
   - **Reject the customer** → applicant is notified with reasoning
7. **Post-approval** — the customer takes the downloaded PDF to the company and can raise queries or exchange messages through a **bidirectional ticketing system**

Weight versioning is tracked throughout — each approved set of weights becomes the new starting point for subsequent calculations, creating an adaptive, continuously improving underwriting baseline.

---

### Component 3B — Portfolio-Level Policy Optimization (Company-facing)

A batch pipeline where the company uploads a CSV of existing customers and policies, and the system generates optimisation actions and derived policy variants at scale.

**Flow:**

1. **CSV upload** — company uploads customer details, policies, riders, and related attributes
2. **Incremental processing** — data is processed in an incremental methodology (not all-at-once) through a preprocessing and feature engineering pipeline
3. **Policy aggregates** — features are aggregated at the policy level across the customer base
4. **Per-policy clustering** — KMeans groups policies into cohorts based on risk and behavioural similarity
5. **XGBoost modelling** — a gradient-boosted model scores each policy cluster
6. **SHAP / LIME + XAI** — explainability layer generates feature attribution for each policy group
7. **PPO action generation** — the PPO agent recommends one of four actions per policy: `SPLIT`, `MERGE`, `NEW`, or `MAINTAIN`
8. **GenAI reasoning** — a Groq-hosted LLM generates natural-language rationale for each recommended action
9. **RAG-derived rider assignment** — a RAG pipeline (FAISS + Sentence Transformers) retrieves relevant policy and rider documents to predict what riders would apply to each derived policy variant, and what the downstream effects of each action would be
10. **Target group profiling** — the system identifies which customer segments from the uploaded base should be mapped to each derived policy variant

**Results:**
- 31% reduction in loss ratio vs. baseline
- 69% improvement in underwriting efficiency
- 200+ variables assessed per customer in real time

---

## Module 04 — Policy Recommendation

**Live:** [insurematch.vercel.app](https://insurematch.vercel.app/)  
**Stack:** Python · scikit-learn · XGBoost · SHAP · Sentence Transformers · FastAPI  
**Frontend:** Next.js (deployed on Vercel)

A personalised recommendation engine that surfaces the most relevant policies and riders for each customer profile, with white-box explanations powered by SHAP.

**Input profile dimensions:**
- Demographics: age, gender, marital status, dependents
- Financial: income, savings, liabilities, investment behaviour
- Health: medical history, occupation risk, lifestyle habits
- Behavioural: interaction patterns, preferred coverage, renewal history

**ML pipeline:**

1. **Feature engineering** — demographic, financial, occupational, medical, and risk features are mapped and embedded using Sentence Transformers
2. **Ensemble policy scoring** — collaborative filtering identifies similar customer cohorts; gradient-boosted trees rank policies by predicted fit
3. **Coverage gap analysis** — the model identifies what a customer currently lacks vs. their risk profile
4. **Rider selection and prioritisation** — filtering and scoring determines the most relevant add-ons
5. **XAI layer (SHAP)** — every recommendation ships with a plain-language explanation of the driving factors

User queries are handled by a conversational agent: intent classification routes between pre-retrieval and post-retrieval RAG paths, with context managed across turns and a final LLM response grounded in the policy vector store.

**Results:**
- 3× improvement in cross-sell conversion
- 87% recommendation acceptance rate in pilot cohorts
- 50+ features assessed per customer

---

## Architecture Summary

```
Frontend (HTML/CSS/JS or Next.js)
          │
          ▼
  REST APIs per module (Flask / FastAPI)
          │
    ┌─────┴──────┬──────────────┬──────────────┐
    │            │              │              │
 Claims      Sanctions      Risk/PPO     Recommendation
 Pipeline    Screening      (HF Spaces)   Engine
    │            │              │              │
    └─────┬──────┴──────────────┴──────────────┘
          │
   Policy DB · Claims DB · Watchlist DB · Vector Store
```

Each module is independently deployed and stateless at the API layer. Shared state (approved weights, risk scores, watchlist data) is persisted in SQL or staging stores depending on the module.

---

## Tech Stack

| Layer | Technologies |
|-------|-------------|
| Frontend (M1–M3) | HTML5, CSS3, JavaScript |
| Frontend (M4) | Next.js, React |
| Backend | Python 3.9+, Flask, FastAPI |
| ML / Supervised | XGBoost, scikit-learn, TensorFlow/Keras |
| Reinforcement Learning | Stable-Baselines3 (PPO) |
| Explainability | SHAP |
| NLP / Semantic Search | spaCy, NLTK, Sentence Transformers, FAISS |
| Fuzzy Matching | RapidFuzz |
| Computer Vision | OpenCV |
| GenAI / LLM | Groq-hosted LLM, RAG pipeline |
| Deployment | Netlify, PythonAnywhere, Vercel, Hugging Face Spaces |
| Data | Pandas, NumPy, SQLite, PostgreSQL |

---

## Compliance Context

All modules were designed with IRCSL regulatory requirements in mind. AI decisions are logged and auditable. The HIL mechanism in Module 03 ensures no automated weight changes are applied to live policies without human sign-off. Sanction screening is aligned to FATF recommendations and Central Bank of Sri Lanka AML/CFT directives.

---

## Getting Started

```bash
git clone https://github.com/your-org/smart-insurance-ai.git
cd smart-insurance-ai

pip install -r requirements.txt
```

```bash
# Module 01 — Claims
cd claims && python app.py

# Module 02 — Sanctions
cd sanctions && python app.py

# Module 03 — Risk (see HF Spaces for hosted version)
cd risk-assessment && python app.py

# Module 04 — Policy Recommendation
cd policy-ai && npm install && npm run dev
```

---

## Repository Structure

```
smart-insurance-ai/
│
├── index.html                    # Platform homepage
│
├── claims/                       # Module 01
│   ├── app.py
│   ├── models/
│   └── templates/
│
├── sanctions/                    # Module 02
│   ├── app.py
│   ├── watchlists/
│   └── models/
│
├── risk-assessment/              # Module 03
│   ├── app.py
│   ├── ppo_agent/
│   │   ├── train.py
│   │   └── saved_model/
│   └── rag/
│
├── policy-ai/                    # Module 04 (Next.js)
│   ├── src/
│   └── public/
│
├── requirements.txt
└── README.md
```

---

## Contact

Questions about the research: raise an issue or reach out via the repository.

For production inquiries regarding the platform, contact the team directly at [InsureMatch Research Project Website](https://insure-math.vercel.app/#domain).
