# CreditGuard AI — Multi-Agent Loan Underwriting Assistant

A **CrewAI-based multi-agent AI system for automated loan pre-screening**, designed to simulate the workflow of a real credit underwriting team.

CreditGuard AI uses four specialized AI agents — **Financial Analyst, Credit Risk Assessor, Compliance Officer, and Underwriting Report Writer** — to evaluate loan applications, identify financial and KYC/AML concerns, assess risk, and generate an explainable underwriting memo.

> **Note:** This project is a portfolio/demo project using synthetic data. It is intended to demonstrate multi-agent orchestration and explainable AI concepts, not to make real-world lending decisions.

---

## 🚀 Project Overview

Traditional loan applications often require several manual review stages:

1. A financial analyst evaluates income, debt, and credit history.
2. A risk officer converts financial information into a risk tier.
3. A compliance officer checks for KYC/AML concerns.
4. An underwriter reviews the findings and prepares a final recommendation.

CreditGuard AI reproduces this workflow using a **sequential multi-agent architecture**.

Instead of asking one LLM to perform the entire analysis, each agent has a clearly defined responsibility. The agents exchange their outputs through CrewAI task context, creating a traceable underwriting pipeline.

The system produces recommendations such as:

* **Approve**
* **Refer for manual review**
* **Decline**

---

## 🏗️ Architecture

```text
                  Loan Application
                         │
                         ▼
              ┌─────────────────────┐
              │  Financial Analyst  │
              │                     │
              │ • Calculate DTI     │
              │ • Credit bracket    │
              └──────────┬──────────┘
                         │
                         ▼
              ┌─────────────────────┐
              │  Credit Risk        │
              │  Assessor           │
              │                     │
              │ • Risk tier         │
              │ • Risk reasoning    │
              └──────────┬──────────┘
                         │
                         │
        ┌────────────────▼────────────────┐
        │                                 │
        ▼                                 ▼
┌───────────────────┐          ┌────────────────────┐
│ Compliance Officer│          │ Verified financial │
│                   │          │ & risk findings    │
│ • KYC/AML screen  │          │                    │
│ • Red flags       │          │                    │
└─────────┬─────────┘          └─────────┬──────────┘
          │                              │
          └──────────────┬───────────────┘
                         ▼
              ┌─────────────────────┐
              │ Underwriting Report │
              │ Writer              │
              │                     │
              │ • Combine findings  │
              │ • Write memo        │
              │ • Recommendation    │
              └──────────┬──────────┘
                         │
                         ▼
                 Underwriting Memo
```

---

## 🤖 AI Agents

### 1. Financial Analyst

Responsible for evaluating the applicant's basic financial profile.

**Tools:**

* DTI Calculator
* Credit Score Bracket

The agent calculates the applicant's Debt-to-Income ratio and converts the credit score into a qualitative bracket.

### 2. Credit Risk Assessor

Evaluates the applicant's overall credit risk using:

* DTI
* Credit score
* Employment stability
* Loan amount
* Loan purpose

It produces a:

* Low
* Medium
* High

risk classification.

### 3. Compliance Officer

Reviews application notes for potential **KYC/AML red flags**.

The project uses a deterministic keyword-based screening tool to identify phrases such as:

* `p.o. box`
* `could not confirm`
* `mismatch`
* `unverified`
* `refused to provide`

The tool returns either detected red flags or a clean result.

### 4. Underwriting Report Writer

Combines the outputs from the other agents into a structured underwriting memo containing:

* Applicant
* Financial Summary
* Risk Tier
* Compliance Check
* Final Recommendation

---

## 🛠️ Technology Stack

| Technology        | Purpose                         |
| ----------------- | ------------------------------- |
| **Python**        | Core programming language       |
| **CrewAI**        | Multi-agent orchestration       |
| **Groq**          | LLM inference                   |
| **Llama 3.3 70B** | Large language model            |
| **Google Colab**  | Development/runtime environment |
| **CrewAI Tools**  | Tool integration                |
| **LiteLLM**       | LLM connectivity                |

The project is designed to run on the **free Groq tier and Google Colab**, without requiring a local GPU.

---

## 🔧 Deterministic Tools

One of the important design decisions in this project is that numerical calculations and KYC keyword detection are **not delegated to the LLM**.

### DTI Calculator

```text
DTI = Monthly Debt Payments / Monthly Income × 100
```

Risk bands:

|      DTI | Risk Band |
| -------: | --------- |
|  `< 36%` | Low       |
| `36–43%` | Medium    |
|  `> 43%` | High      |

### Credit Score Bracket

| Credit Score | Bracket   |
| -----------: | --------- |
|         740+ | Excellent |
|      670–739 | Good      |
|      580–669 | Fair      |
|         <580 | Poor      |

This tool-grounded approach makes the numerical portion of the workflow deterministic and auditable rather than relying on the LLM to perform calculations.

---

## 🔄 Agent Workflow

CrewAI uses a **sequential process**.

The workflow is:

```text
Loan Application
       │
       ├──────────────► Financial Analyst
       │                       │
       │                       ▼
       │                Financial Findings
       │                       │
       │                       ▼
       │                Risk Assessor
       │                       │
       │                       ▼
       │                  Risk Tier
       │
       └──────────────► Compliance Officer
                               │
                               ▼
                         KYC Findings
                               │
                               └──────────────┐
                                              ▼
                                    Report Writer
                                              │
                                              ▼
                                     Final Underwriting
                                           Memo
```

CrewAI's `context=[...]` mechanism allows downstream tasks to consume the outputs of upstream tasks without manually passing strings between agents.

---

## 📊 Example Applications

The project uses **synthetic loan application data**, not real customer information.

### Applicant A

* Monthly income: 3,200
* Monthly debt payments: 1,450
* Credit score: 612
* Employment: 2.5 years
* Loan: 15,000
* Purpose: Debt consolidation
* Country: Latvia

The system calculated:

```text
DTI = 45.3%
Risk Band = High
Credit Score = Fair
```

The compliance agent also detected KYC/AML concerns involving a P.O. box and an unverifiable employer phone number.

The final recommendation was:

**Refer for manual review.**

---

### Applicant B

* Monthly income: 5,400
* Monthly debt payments: 900
* Credit score: 748
* Employment: 7 years
* Loan: 8,000
* Purpose: Home renovation
* Country: Estonia

The system calculated:

```text
DTI = 16.7%
Risk Band = Low
Credit Score = Excellent
```

No KYC red flags were detected.

The final recommendation was:

**Approve.**

---

## 💡 Key Design Principle: Explainability

A major objective of CreditGuard AI is to keep the underwriting workflow **explainable and auditable**.

Instead of allowing the LLM to invent financial values, the system separates:

**Deterministic computation**

```text
Income
   ↓
Debt Payments
   ↓
DTI Calculator
   ↓
Verified DTI
```

from:

**LLM reasoning**

```text
Verified Financial Data
          ↓
   Risk Assessment
          ↓
   Human-readable
   explanation
```

Similarly, KYC screening is performed by a Python tool rather than asking the LLM to independently invent compliance findings.

This architecture is particularly relevant to financial AI because auditability and traceability are important when AI is used around credit-risk workflows.

---

## ⚙️ Installation

The project can be executed in Google Colab.

Install the required packages:

```bash
pip install -q crewai crewai-tools litellm
```

The original project uses a free Groq API key for LLM inference.

Create a Groq API key and provide it when prompted.

---

## ▶️ Running the Project

Once the dependencies and API key are configured, the system processes each sample application sequentially.

Conceptually:

```python
for app in loan_applications:
    crew = build_crew(app)
    output = await crew.kickoff_async()
    results[app["applicant_id"]] = output
```

Each application goes through the complete multi-agent underwriting workflow.

The generated reports are then saved to:

```text
underwriting_reports.md
```

The project demonstrates that multiple applications can be processed through the same reusable CrewAI workflow.

---

## 📁 Project Structure

```text
CreditGuard-AI/
│
├── README.md
├── CreditGuard_AI.ipynb
├── underwriting_reports.md
└── requirements.txt
```

If the project is maintained as a single Colab notebook, the notebook contains:

```text
1. Installation
2. API configuration
3. Sample applications
4. Deterministic tools
5. Agent definitions
6. Task definitions
7. Crew construction
8. Crew execution
9. Underwriting reports
```

---

## 🔐 Data & Responsible AI

This project uses **synthetic applicant information** and does not use real customer PII.

CreditGuard AI is a demonstration of AI-assisted **pre-screening**, not an autonomous lending system.

The generated recommendation should therefore be treated as:

> **Decision support for a human underwriter, not a final lending decision.**

Potential production considerations would include:

* Human-in-the-loop approval
* Model governance
* Bias and fairness testing
* Explainability requirements
* Regulatory compliance
* Secure handling of applicant data
* Audit logging
* Robust KYC/AML verification
* Validation against real underwriting policies

---

## 🎯 What This Project Demonstrates

This project demonstrates several practical AI engineering concepts:

* Multi-agent AI architecture
* Role-based agent design
* Sequential agent orchestration
* Tool-augmented LLMs
* Deterministic financial calculations
* KYC/AML rule-based screening
* Context passing between agents
* Explainable AI workflows
* Automated report generation
* Human-in-the-loop decision support

The key distinction is that this is a **real multi-agent workflow**, rather than one LLM prompt pretending to represent several agents.

---

## 🚀 Future Improvements

Possible extensions include:

### Data & ML

* Integrate a real credit-risk ML model
* Add probability-of-default estimation
* Add SHAP-based feature explanations
* Introduce historical loan datasets
* Add model performance monitoring

### Multi-Agent System

* Add a Fraud Detection Agent
* Add a Document Verification Agent
* Add a Loan Policy Agent
* Add a Fairness/Audit Agent
* Add a Human Underwriter Agent

### Engineering

* FastAPI backend
* Streamlit dashboard
* PostgreSQL database
* Docker deployment
* Azure deployment
* Agent observability and logging

### Responsible AI

* Bias detection
* Fair lending analysis
* Explainable risk scoring
* Decision audit trails
* Human approval gates

---

## 📌 Disclaimer

CreditGuard AI is an educational and portfolio project demonstrating multi-agent AI orchestration for loan underwriting.

It uses synthetic data and simplified financial/KYC rules. It **must not be used to make actual lending, credit, compliance, or financial decisions without appropriate validation, governance, regulatory review, and human oversight.**

---

## 👩‍💻 Author

**Anjali Shibu**

M.Sc. Computer Science — Data Analytics & AI

Focus areas:

* AI Engineering
* Data Analytics
* Explainable AI
* Credit Risk
* Responsible AI
* Multi-Agent Systems
* Financial Technology
