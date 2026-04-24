# 🧠 Explainable & Fair AI for Credit Risk Assessment

## 📌 Overview

This project presents an **Explainable and Fair AI** system for credit risk (default) prediction, designed for real-world financial decision-making environments.

The system combines:
- ✅ Interpretable machine learning
- ✅ SHAP-based explainability
- ✅ Fairness evaluation using AIF360
- ✅ API deployment with FastAPI
- ✅ End-user friendly UI for risk assessment

The goal is to move beyond black-box models and provide **transparent, auditable, and regulator-friendly** AI decisions.

---

## ✨ Key Features

| Feature | Description |
|---------|-------------|
| 📊 **Accurate Default Prediction** | Threshold-tuned Logistic Regression with class weighting |
| 🔍 **SHAP Explainability** | Global feature importance + individual (customer-level) explanations |
| ⚖️ **Fairness-Aware Design** | Bias audit using AIF360; sensitive attributes excluded from explanations |
| 🌐 **End-to-End Deployment** | FastAPI backend + HTML-based frontend UI |
| 🧠 **Human-Readable Explanations** | Converts SHAP values into simple business-friendly reasoning |

---

## 🧱 Project Architecture

```
User Input (UI / API)
        ↓
Schema Validation (Pydantic)
        ↓
Feature Engineering
        ↓
ML Model (Logistic Regression)
        ↓
SHAP Explainability
        ↓
Text-based Reasoning
        ↓
Final Output (Prediction + Explanation + Fairness Note)
```

---

## 📂 Project Structure

```
├── Fraud_Risk_notebook.ipynb              # Model development & analysis
├── predict.py                             # Prediction + SHAP logic
├── main.py                                # FastAPI app
├── schema.py                              # Input/Output Pydantic schemas
├── model.joblib                           # Trained model artifact
├── shap_explainer.joblib                  # SHAP explainer artifact
├── feature_names.joblib                   # Feature ordering artifact
├── templates/
│   └── index.html                         # Frontend UI
├── default of credit card clients.xls     # Dataset (UCI)
├── requirements.txt                       # Python dependencies
└── README.md                              # This file
```

---

## 🚀 Getting Started

### Prerequisites

- Python 3.9+
- pip

### Installation

```bash
git clone <repo-url>
cd Project
pip install -r requirements.txt
```

### Run the Application

```bash
uvicorn main:app --reload
```

Then open [http://localhost:8000](http://localhost:8000) in your browser.

---

## 📊 Dataset

| Property | Details |
|----------|---------|
| **Source** | [UCI Credit Card Default Dataset](https://archive.ics.uci.edu/ml/datasets/default+of+credit+card+clients) |
| **Records** | 30,000 customers |
| **Target** | Default payment next month (Yes = 1, No = 0) |
| **Feature Categories** | Demographic (excluded from explanations), Financial (credit limit, bills, payments), Behavioral (repayment history) |

---

## ⚙️ Model Details

### 🧠 Model Used

**Logistic Regression** with class weighting (`class_weight='balanced'`) and threshold tuning via Precision-Recall curve analysis.

### 📈 Model Performance (Logistic Regression — Threshold-Tuned)

| Metric | Class 0 (No Default) | Class 1 (Default) |
|--------|---------------------|--------------------|
| **Precision** | 0.87 | 0.44 |
| **Recall** | 0.79 | 0.59 |
| **F1-Score** | 0.83 | 0.50 |

| Aggregate Metric | Value |
|-----------------|-------|
| **Accuracy** | 0.74 |
| **ROC-AUC** | 0.745 |
| **Macro Avg F1** | 0.66 |
| **Weighted Avg F1** | 0.75 |

> **Note**: The model is optimized for **recall on defaulters** (Class 1) rather than raw accuracy, as missing a defaulter is more costly than a false positive in credit risk scenarios.

### Models Compared

| Model | ROC-AUC |
|-------|---------|
| Logistic Regression (balanced) | 0.745 |
| Random Forest | 0.758 |
| XGBoost | 0.766 |
| Balanced Random Forest | 0.766 |
| LR with Reweighing (AIF360) | 0.742 |

Logistic Regression was selected as the final model for its **interpretability** and compatibility with SHAP's linear explainer, aligning with the project's goal of explainability.

---

## 🔍 Explainability (SHAP)

### Global Insights

Top contributing features:
- `LIMIT_BAL` — Credit limit relative to usage
- `MAX_BILL_AMT` — Maximum outstanding bill
- `PAY_TO_BILL_RATIO` — Repayment ratio
- `NUM_DELAYED_MONTHS` — History of payment delays

### Local Insights

Per-customer explanations using SHAP values, translated into human-readable text:

> *"Outstanding bill amount is high, which increased the default risk"*
> *"Credit limit is low relative to usage, which increased the default risk"*

Sensitive demographic attributes (`SEX`, `AGE`, `EDUCATION`, `MARRIAGE`, `AGE_GROUP`) are **excluded from explanations** to ensure fairness.

---

## ⚖️ Fairness Analysis

| Property | Details |
|----------|---------|
| **Framework** | IBM AIF360 |
| **Technique** | Reweighing (pre-processing fairness mitigation) |
| **Sensitive Attributes** | Gender, Age, Education, Marital Status |

### Key Findings

- Minimal demographic bias observed across protected groups
- Reweighing had limited impact on ROC-AUC (0.745 → 0.742) but affected the recall/precision trade-off
- Demographics are used **only for fairness evaluation**, not for prediction or explanation

---

## 🌐 API Usage

### Start the Server

```bash
uvicorn main:app --reload
```

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/` | Web UI for risk assessment |
| `POST` | `/predict` | JSON API for programmatic access |

### Sample Request

```json
POST /predict
Content-Type: application/json

{
  "LIMIT_BAL": 50000,
  "BILL_SEPT": 2000,
  "BILL_AUG": 3000,
  "BILL_JUL": 2500,
  "BILL_JUN": 2200,
  "BILL_MAY": 2100,
  "BILL_APR": 1800,
  "PAY_AMT_SEPT": 1000,
  "PAY_AMT_AUG": 1200,
  "PAY_AMT_JUL": 1100,
  "PAY_AMT_JUN": 900,
  "PAY_AMT_MAY": 800,
  "PAY_AMT_APR": 700,
  "PAY_SEPT": 0,
  "PAY_AUG": 1,
  "PAY_JUL": 0,
  "PAY_JUN": 0,
  "PAY_MAY": 1,
  "PAY_APR": 0
}
```

### Sample Response

```json
{
  "prediction": "Default",
  "risk_probability": 0.72,
  "risk_level": "Medium Risk",
  "key_reasons": [
    "Credit limit is low relative to usage, which increased the default risk",
    "Outstanding bill amount is high, which increased the default risk"
  ],
  "fairness_note": "This decision is based on financial and behavioral attributes. Sensitive demographic attributes such as gender and age did not materially influence the prediction."
}
```

---

## 🖥️ UI Demo

The web interface provides:
- Input form for customer financial data
- One-click risk assessment
- Color-coded risk level display (🟢 Low / 🟡 Medium / 🔴 High)
- Human-readable explanation of key risk factors
- Fairness disclosure note

---

## 🔮 Future Work

- 📈 Model drift monitoring in production
- 💳 Transaction-level fraud detection
- 🔬 Extended fairness analysis (intersectional bias across age × education × gender)
- 🐳 Docker containerization for deployment
- 📊 SHAP visualization in the frontend UI

---

## 📚 References

- Lundberg & Lee (2017) — [SHAP: A Unified Approach to Interpreting Model Predictions](https://arxiv.org/abs/1705.07874)
- Ribeiro et al. (2016) — [LIME: Local Interpretable Model-agnostic Explanations](https://arxiv.org/abs/1602.04938)
- Bellamy et al. (2018) — [AI Fairness 360: An Extensible Toolkit](https://arxiv.org/abs/1810.01943)
- Dal Pozzolo et al. (2018) — Fraud Detection in Credit Card Transactions
- Zhang & Sheng (2023) — Fairness in Machine Learning

---

## ⭐ Support

If you find this project useful, give it a ⭐ on GitHub!
