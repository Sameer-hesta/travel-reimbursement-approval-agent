# Travel Reimbursement Approval Agent

A lightweight GenAI prototype that evaluates travel reimbursement claims against a mock company policy and returns structured decisions. Built as a single Jupyter Notebook using Python, LangChain, Groq, and deterministic policy-checking tools.

## Project Overview

The agent evaluates travel reimbursement claims and recommends one of four outcomes:

- `APPROVE`
- `PARTIAL_APPROVE`
- `REJECT`
- `MANUAL_REVIEW`

It uses the five sample claims and the policy provided in the candidate assignment. No real employee or company data is used.

## Tech Stack

- Python
- Jupyter Notebook / Google Colab
- LangChain
- LangChain-Groq
- Groq-hosted LLM (`openai/gpt-oss-20b` in the notebook)
- Pydantic for structured decision validation
- Pandas and Matplotlib for results/dashboard

## Architecture

```text
Claim Input
    |
    v
LangChain Agent
    |
    +--> Policy Lookup
    +--> Receipt Completeness Check
    +--> Reimbursement Limit Checker
    +--> Approval Threshold Checker
    |
    v
Policy-grounded Decision
    |
    v
Structured JSON Output (Pydantic)
    |
    v
Dashboard (decision counts and finalized amounts)
```

## Tools

### 1. Policy Lookup
Returns the full policy or selected policy IDs so decisions can be grounded in the policy rules.

### 2. Receipt Completeness Check
Checks whether an expense requires a receipt and flags missing required receipts. Airfare and lodging always require receipts; other line items require receipts when the amount is over $25.

### 3. Reimbursement Limit Checker
Calculates eligible amounts and deductions for:
- Meals: $75 per day
- Lodging: $200 per night
- Ground transport: $50 per day

It also flags ineligible expenses, unknown categories, and non-economy airfare exceptions.

### 4. Approval Threshold Checker
Determines the approval tier:
- Up to $500: auto-approval tier
- Above $500 and up to $2,000: manager-approvable tier
- Above $2,000: director/manual review

## Policy Rules Covered

- Eligible business expenses: economy airfare, lodging, meals, ground transportation, conference/registration fees.
- Ineligible expenses: alcohol/minibar, spa/gym/personal entertainment, in-room movies, personal shopping, gifts, traffic fines, penalties, late fees, and personal expenses.
- Receipts: required for airfare and lodging, and for individual expenses above $25.
- Non-economy airfare: routed to manual review.
- Claims submitted more than 30 days after the expense date: manual review.
- Reimbursable amounts above $2,000: manual review.

Policy IDs (`POL-*`) are retained in the notebook and included in decision outputs.

## Getting Started

### Option A — Google Colab (recommended)

The notebook is designed to be run top-to-bottom. Google Colab is recommended because it provides a regular hosted Python runtime and supports the compiled dependencies used by LangChain.

1. Open [Google Colab](https://colab.research.google.com/).
2. Choose **File → Upload notebook**.
3. Upload `SameerKhan.ipynb` (or the notebook filename in this repository).
4. Run the dependency-installation cell.
5. Configure your Groq API key (see below).
6. Select **Runtime → Run all**.

### Option B — Local Jupyter

Use Python 3.10+ in a virtual environment, then open the notebook in Jupyter Notebook or JupyterLab.

Install dependencies:

```bash
python -m pip install -U jupyter langchain langchain-groq pydantic pandas matplotlib
```

Start Jupyter:

```bash
jupyter notebook
```

> Avoid JupyterLite/browser-only WASM environments for this notebook: some compiled Python dependencies required by the stack may not install there.

## Groq API Configuration

The notebook uses Groq for LLM-powered evaluation when a `GROQ_API_KEY` is available. Groq free-tier availability and limits may change.

1. Create an API key at [Groq Console](https://console.groq.com/keys).
2. In Colab, open the **Secrets** panel (key icon) and add a secret named `GROQ_API_KEY`.
3. Allow the notebook to access that secret.

For a local environment, set the environment variable before launching Jupyter:

**macOS / Linux**
```bash
export GROQ_API_KEY="your_groq_api_key"
jupyter notebook
```

**Windows PowerShell**
```powershell
$env:GROQ_API_KEY="your_groq_api_key"
jupyter notebook
```

Do not commit API keys, tokens, or other credentials to GitHub.

If no Groq key is configured, the notebook uses its deterministic fallback so the example claims can still be evaluated without an LLM call.

## Running the Demo

Run all notebook cells from top to bottom. The notebook:

1. Loads the reimbursement policy.
2. Loads the five sample claims from the assignment.
3. Defines policy and validation tools.
4. Sets up the LangChain agent (when Groq is configured).
5. Evaluates claims.
6. Prints a JSON array of decision objects.
7. Displays a dashboard summarizing decisions and finalized amounts.

## Output Schema

Each claim result contains the following fields:

```json
{
  "claim_id": "CLM-001",
  "decision": "APPROVE",
  "approved_amount": 0.0,
  "deducted_amount": 0.0,
  "missing_docs": [],
  "policy_refs": [],
  "confidence": 0.95,
  "explanation": "Short policy-grounded explanation.",
  "tools_used": []
}
```

The decision field is one of `APPROVE`, `PARTIAL_APPROVE`, `REJECT`, or `MANUAL_REVIEW`.

## Design Choices

- **Deterministic business checks:** amounts, receipt requirements, category eligibility, caps, and approval thresholds are handled by Python tools.
- **Policy grounding:** policy rules have stable IDs so the output can reference the rule behind a decision.
- **LLM for interpretation:** the LLM is intended to reason over the claim and tool/policy context, not replace deterministic financial calculations.
- **Manual review:** ambiguous or exception cases are routed to human review instead of forcing a decision.
- **Fallback:** deterministic logic makes the demo runnable when the LLM is unavailable or not configured.

## Limitations

- Receipts are represented as metadata; the notebook does not perform OCR or inspect receipt images/PDFs.
- The prototype does not connect to an expense-management platform or approval system.
- The LLM can still produce inaccurate interpretations or tool-use decisions; production use would need strict validation, monitoring, audit logging, and a human approval workflow.
- The provided sample claims and dates are fixed assignment data.
- This is a prototype for demonstration and evaluation, not a production reimbursement system.

## Future Improvements

- Receipt OCR and extraction with amount/date/vendor validation.
- Duplicate receipt detection.
- Automated unit and policy regression tests.
- Persistent audit logs for tool calls and agent decisions.
- Reviewer UI for Manual Review cases.
- Authentication, authorization, and integration with an enterprise expense system.

## Repository Structure

```text
.
├── README.md
└── SameerKhan.ipynb
```

## License

No license is currently specified. Add a license file if you intend to distribute or reuse this project publicly.
