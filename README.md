# 🛡️ Fraud Detection System — Real-Time Fraud Detection & Investigation Dashboard

*An end-to-end ML & AI-powered real-time fraud detection dashboard for financial transactions.*

## 📌 Overview

Fraud Detection System is a complete fraud detection and investigation system that simulates how financial institutions monitor, investigate, and respond to suspicious transactions in real time.

**Workflow:** Transaction Generation → Feature Engineering → Anomaly Detection → Risk Scoring → Alert Generation → Investigation Report → Dashboard Visualization

> **Dashboard status:** The Next.js dashboard is **not included in this repository yet**. The `dashboard/` entry is an empty submodule link with no files. You can run the FastAPI backend and the ML pipeline from this repo; the frontend setup, demo, and Vercel steps below apply once the dashboard code is published here.

## 🛠️ Tech Stack

| Layer | Technology |
| --- | --- |
| **Backend** | FastAPI, Python 3.10+, Uvicorn |
| **Frontend** | Next.js 14, TypeScript, Tailwind CSS, Lucide Icons *(not yet published in this repo)* |
| **ML Model** | Isolation Forest (Scikit-learn) + Rule-based hybrid |
| **AI/LLM** | Ollama — Llama 3.2 (for investigation report generation) |
| **Data** | Pandas, NumPy, SHAP, Matplotlib, Faker |
| **Storage** | In-memory (Python dictionaries) |
| **DevOps** | Git, GitHub, Vercel (Frontend), Render (Backend) |

## ✨ Key Features

- Real-time transaction scoring (0–100 risk score per transaction)
- ML anomaly detection — Isolation Forest + Rule-based hybrid algorithm
- LLM-generated risk reports — What happened / Why risky / Recommended action
- Interactive Admin Dashboard built with Next.js *(not yet published in this repo)*
- Alert Review/Dismiss workflow for full alert management
- SHAP explainability for feature importance analysis
- Live feed simulation for real-time transaction monitoring

## ⚙️ Setup & Installation

### Prerequisites

| Tool | Version |
| --- | --- |
| Python | 3.10+ |
| Node.js | 18+ (dashboard only) |
| npm | Latest (dashboard only) |
| Git | Latest |

### Step 1: Clone Repository

```bash
git clone https://github.com/mdibnalabid/fraud-detection-system.git
cd fraud-detection-system
cd fraud-detection
```

### Step 2: Backend Setup

```bash
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
uvicorn src.api.main:app --reload
```

### Step 3: Ollama Setup (for LLM reports)

```bash
ollama pull llama3.2
ollama run llama3.2
```

### Step 4: Frontend Setup *(once the dashboard code is published)*

The `dashboard/` folder is currently empty in this repo, so these commands will not work yet:

```bash
cd dashboard
npm install
npm run dev
```

Now open:

- **Frontend:** http://localhost:3000 *(once the dashboard is published)*
- **Backend API:** http://localhost:8000
- **API Docs:** http://localhost:8000/docs

## 📁 Project Structure

```
fraud-detection/
│
├── dashboard/                         # Frontend (Next.js) — not yet published (empty)
│
├── data/
│   ├── processed/                     # Evaluation artifacts
│   │   ├── alerts.json
│   │   ├── confusion_matrix.png
│   │   ├── risk_distribution.png
│   │   ├── scored_transactions.csv
│   │   ├── shap_summary.png
│   │   └── test_set.csv
│   └── raw/
│       └── synthetic_transactions.csv
│
├── models/
│   └── anomaly_model.pkl              # Trained ML model
│
├── notebooks/
│   └── exploration.ipynb
│
├── src/
│   ├── api/                           # FastAPI backend
│   │   ├── alerts.py                  # Alert generation / retrieval logic
│   │   ├── main.py                    # Entry point
│   │   ├── reporting.py               # Investigation report generation (LLM)
│   │   └── scoring.py                 # Risk scoring engine
│   ├── data_generator.py              # Synthetic data creator
│   ├── feature_engineering.py         # Feature extraction
│   ├── model_evaluate.py              # Model evaluation
│   └── model_train.py                 # Model training pipeline
│
├── .gitignore
├── README.md
└── requirements.txt
```

### Folder Descriptions

| Folder | Purpose |
| --- | --- |
| `dashboard/` | Next.js frontend — not yet published; currently an empty submodule link with no files |
| `data/processed/` | Evaluation outputs (charts, scored data, alerts) |
| `models/` | Trained ML model artifact |
| `notebooks/` | Jupyter notebook for exploration |
| `src/api/` | FastAPI backend — all API logic |
| `src/` | ML pipeline (data gen, training, evaluation) |

## 📡 API Endpoints

With the backend running, interactive API docs are available at http://localhost:8000/docs.

### 1. `POST /score`

Scores a single transaction with the Isolation Forest model, explains the score with SHAP, and builds an investigation report. If the transaction is suspicious (`risk_score >= 60`), an alert is created automatically.

**Request body:**

```json
{
  "transaction_id": "TXN_001",
  "user_id": "U_1023",
  "amount": 25000,
  "timestamp": "2026-10-03T02:30:00",
  "location": "Dhaka",
  "device": "android",
  "txn_type": "send_money",
  "recipient_new": 1
}
```

| Field | Type | Description |
| --- | --- | --- |
| `transaction_id` | string | Transaction identifier |
| `user_id` | string | Customer identifier |
| `amount` | number | Transaction amount |
| `timestamp` | string | ISO 8601 date-time; its hour is used as a feature |
| `location` | string | City, e.g. `Dhaka`, `Chittagong`, `Sylhet` |
| `device` | string | Device type, e.g. `android`, `ios`, `web` |
| `txn_type` | string | e.g. `send_money`, `mobile_recharge`, `bill_pay`, `cash_out`, `merchant_pay` |
| `recipient_new` | integer | `1` if the recipient is new, otherwise `0` (default `0`) |

**Response** (values are illustrative):

```json
{
  "transaction_id": "TXN_001",
  "risk_score": 72.5,
  "is_suspicious": true,
  "top_features": [
    { "feature": "is_night", "value": -0.4123 },
    { "feature": "log_amount", "value": -0.2871 },
    { "feature": "recipient_new", "value": -0.1534 },
    { "feature": "hour", "value": -0.0912 },
    { "feature": "is_off_hours", "value": -0.0458 }
  ],
  "report": {
    "what_happened": "User U_1023 made a $25000.00 send_money from Dhaka using android at 2026-10-03T02:30:00.",
    "why_is_risky": "Risk score 72.5/100. Key risk factors: is_night, log_amount, recipient_new.",
    "next_actions": "1. Review transaction. 2. Contact customer. 3. Block if suspicious."
  }
}
```

- `risk_score`: 0–100, computed from the Isolation Forest anomaly score (higher means more anomalous).
- `is_suspicious`: `true` when `risk_score >= 60`.
- `top_features`: the 5 features with the largest absolute SHAP values for this transaction.
- `report`: what happened, why it is risky, and recommended next actions.
- Errors: returns `500` with an error message if scoring fails (for example, an invalid `timestamp`).

### 2. `GET /alerts`

Returns all alerts, newest first. Alerts are created automatically by `POST /score` for suspicious transactions. Use the optional `status` query parameter to filter by `flagged`, `reviewed`, or `dismissed`, for example `GET /alerts?status=flagged`.

Each alert contains `alert_id`, `transaction_id`, `user_id`, `amount`, `risk_score`, `status`, `top_features`, `report`, `created_at`, and `updated_at`.

### 3. `POST /alerts/{alert_id}/review`

Marks an alert as `reviewed`. Returns `404` if the alert does not exist.

### 4. `POST /alerts/{alert_id}/dismiss`

Marks an alert as `dismissed`. Returns `404` if the alert does not exist.

### 5. `GET /`

Health check. Returns `{"message": "Fraud Detection API is running"}`.

## 🧠 Detection Approach

**Hybrid Strategy:** Isolation Forest (60%) + Rule-based checks (40%)

**Risk Score Formula:**

```
Risk Score = (Anomaly Score × 0.6) + (Rule Score × 0.4)
```

**Features Used:** `amount_ratio`, `hour_of_day`, `is_new_recipient`, `is_new_device`, `is_new_location`, `txn_frequency`, `amount_zscore`

## 📊 Evaluation Artifacts (`data/processed/`)

| File | Description |
| --- | --- |
| `confusion_matrix.png` | Model performance |
| `risk_distribution.png` | Score distribution |
| `shap_summary.png` | Feature importance |
| `scored_transactions.csv` | All transactions scored |
| `alerts.json` | Generated alerts |

## 🎮 Demo Instructions

> These steps require the dashboard, which is not yet published in this repo. To try the backend on its own, start it (Step 2) and send a request to `POST /score` from the API docs at http://localhost:8000/docs.

1. Open http://localhost:3000
2. Click **Run Demo Scenario**
3. Watch transactions appear gradually (Low → Medium → High risk)
4. Click any high-risk transaction to view its investigation report

### Demo Risk Distribution

| Risk Level | Count |
| --- | --- |
| Low | 13 |
| Medium | 4 |
| High | 3 |
| **Total** | **20** |

## 🌐 Deployment

### Backend → Render

- **Root:** `src/api`
- **Build:** `pip install -r requirements.txt`
- **Start:** `uvicorn main:app --host 0.0.0.0 --port $PORT`

### Frontend → Vercel *(once the dashboard code is published)*

- **Root:** `dashboard`
- **Framework:** Next.js
- **Env:** `NEXT_PUBLIC_API_URL`

## 👥 Team Members

- Shihab Ar Rafi
- Raiyan Jahan
- Md. Ibna Labid


-

**Fraud Detection System** — Detecting suspicious transactions with intelligence, explainability, and action-ready reporting.