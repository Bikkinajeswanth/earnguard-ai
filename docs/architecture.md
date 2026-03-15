# EarnGuard AI — System Architecture

---

## Overview

EarnGuard AI is built on a **four-layer modular architecture**: a mobile-first frontend, a Node.js backend API, a Python AI engine, and a MongoDB database — all connected to external data and payment APIs. Each layer is independently deployable and communicates over well-defined interfaces.

The architecture is designed for a hackathon prototype that can realistically scale to production with minimal structural changes.

---

## Full Architecture Diagram

```
┌──────────────────────────────────────────────────────────────────┐
│                         FRONTEND LAYER                           │
│                    React / Next.js (PWA)                         │
│                                                                  │
│   /onboard    /dashboard    /buy-policy    /notify    /admin     │
└─────────────────────────────┬────────────────────────────────────┘
                              │ HTTPS / REST API + JWT Auth
┌─────────────────────────────▼────────────────────────────────────┐
│                        BACKEND SERVICES                          │
│                      Node.js + Express                           │
│                                                                  │
│  /api/auth   /api/worker   /api/policy   /api/predict            │
│  /api/monitor   /api/payout   /api/admin                         │
│                                                                  │
│  Scheduled Jobs (node-cron):                                     │
│    • Hourly  → Poll Weather, AQI, Traffic APIs                   │
│    • Sunday 23:00 → Run end-of-week deviation check              │
└──────┬──────────────────────┬──────────────────┬─────────────────┘
       │ Internal HTTP        │ Mongoose ODM     │ HTTPS
┌──────▼──────────┐  ┌────────▼──────────────┐  ┌▼────────────────────────┐
│  AI PREDICTION  │  │      DATABASE         │  │   EXTERNAL DATA APIs    │
│     ENGINE      │  │    MongoDB Atlas      │  │                         │
│                 │  │                       │  │  OpenWeatherMap         │
│  Python + Flask │  │  workers              │  │  → Rainfall, Temp       │
│  Scikit-learn   │  │  policies             │  │                         │
│  Pandas / NumPy │  │  predictions          │  │  WAQI                   │
│                 │  │  disruptions          │  │  → AQI Index            │
│  /predict/income│  │  payouts              │  │                         │
│  /score/risk    │  │  earnings             │  │  Google Maps Traffic    │
│  /detect/fraud  │  │                       │  │  → Blockage Score       │
└─────────────────┘  └───────────────────────┘  │                         │
                                                │  Razorpay Sandbox       │
                                                │  → Premium Collection   │
                                                │  → UPI Payout           │
                                                │                         │
                                                │  Platform Status Monitor│
                                                │  → Swiggy/Zomato Uptime │
                                                └─────────────────────────┘
```

---

## Layer 1: Frontend

**Technology:** React.js / Next.js, deployed as a Progressive Web App (PWA)

The frontend is mobile-first and optimised for low-bandwidth Android devices. It serves two distinct user types: delivery workers and platform admins.

**Worker-facing routes:**

| Route | Purpose |
|---|---|
| `/onboard` | Phone OTP registration, profile setup |
| `/dashboard` | Predicted income, active policy, disruption alerts, payout history |
| `/buy-policy` | Plan selection, premium payment via UPI |
| `/notifications` | Real-time payout and disruption alerts |

**Admin-facing routes:**

| Route | Purpose |
|---|---|
| `/admin/overview` | Live policy count, payout queue, disruption map |
| `/admin/fraud` | Fraud-flagged payouts pending manual review |
| `/admin/analytics` | Weekly earnings, payout rates, city-level breakdown |

**Key implementation details:**
- State management: Zustand (lightweight, suitable for mobile)
- API calls: Axios with JWT Bearer token on all authenticated routes
- Localisation: i18next supporting Hindi, Tamil, Telugu, Kannada, English
- Offline support: Service worker caches dashboard data for low-connectivity use

---

## Layer 2: Backend Services

**Technology:** Node.js + Express, hosted on AWS EC2

The backend is the central orchestrator. It handles all business logic, coordinates between the AI engine, database, and external APIs, and runs scheduled jobs for monitoring and payout triggering.

**API modules:**

| Module | Prefix | Responsibility |
|---|---|---|
| Auth | `/api/auth` | Firebase OTP verification, JWT issuance and refresh |
| Worker | `/api/worker` | Profile CRUD, zone and platform management |
| Policy | `/api/policy` | Plan purchase, policy lifecycle (active → closed) |
| Prediction | `/api/predict` | Calls AI engine, stores and serves prediction results |
| Monitoring | `/api/monitor` | Polls external APIs, logs disruption events |
| Payout | `/api/payout` | Deviation check, fraud scoring, Razorpay payout initiation |
| Admin | `/api/admin` | Aggregated analytics, fraud review queue |

**Scheduled jobs (node-cron):**

| Schedule | Job |
|---|---|
| Every hour | Poll Weather, AQI, Traffic APIs for all active-policy cities |
| Every Monday 06:00 IST | Trigger AI income prediction for all workers with new policies |
| Every Sunday 23:00 IST | Run end-of-week earnings comparison for all active policies |
| On deviation detected | Invoke fraud check → initiate payout if clean |

**Security:**
- All routes except `/api/auth` require a valid JWT
- Rate limiting on `/api/auth` (max 5 OTP requests per phone per hour)
- Razorpay webhook signature verified on every payment callback
- Environment variables for all API keys — never hardcoded

---

## Layer 3: AI Prediction Engine

**Technology:** Python 3.10 + Flask + Scikit-learn + Pandas, containerised with Docker

The AI engine runs as a separate internal microservice. It is not publicly accessible — all requests come exclusively from the backend API.

**Exposed endpoints:**

| Endpoint | Method | Input | Output |
|---|---|---|---|
| `/predict/income` | POST | Worker features + weather forecast | `predicted_weekly_income`, confidence interval |
| `/score/risk` | POST | Earnings variance + disruption history | `risk_tier` (Low/Medium/High), `risk_score` |
| `/detect/fraud` | POST | Payout event features | `anomaly_score`, `flag`, `reason` |

**Models loaded at startup via `joblib`:**
- `income_predictor.pkl` — Gradient Boosting Regressor
- `risk_classifier.pkl` — Random Forest Classifier
- `fraud_detector.pkl` — Isolation Forest

**Retraining pipeline:** Runs every Monday as an offline batch job. New model files replace old ones after validation. Previous versions are archived for rollback.

---

## Layer 4: Database

**Technology:** MongoDB Atlas (M0 Free Tier for prototype, M10 for production)

MongoDB is chosen for its flexible document schema, which accommodates the variable structure of disruption events and earnings records across different platforms and cities.

**Collections and key fields:**

| Collection | Key Fields |
|---|---|
| `workers` | `workerId`, `phone`, `city`, `zone`, `platform`, `upiId`, `registeredAt` |
| `policies` | `policyId`, `workerId`, `plan`, `premium`, `weekStart`, `weekEnd`, `status` |
| `predictions` | `predictionId`, `workerId`, `weekStart`, `predictedIncome`, `confidenceInterval`, `riskTier`, `modelVersion` |
| `disruptions` | `eventId`, `city`, `zone`, `type`, `severity`, `startTime`, `endTime`, `source` |
| `payouts` | `payoutId`, `workerId`, `policyId`, `amount`, `triggeredAt`, `status`, `razorpayRef` |
| `earnings` | `workerId`, `weekStart`, `actualIncome`, `source`, `submittedAt` |

**Indexes:**
- Compound index on `workerId + weekStart` across `policies`, `predictions`, `earnings`
- Index on `city + weekStart` on `disruptions` for fast zone lookups during payout checks

---

## External Data Integrations

All external API calls originate from the backend. API keys are stored in server-side environment variables and are never sent to the frontend.

| Integration | Provider | Data Used | Trigger Threshold |
|---|---|---|---|
| Weather | OpenWeatherMap | Rainfall (mm/day), temperature (°C) | Rain > 35mm, Temp > 42°C or < 8°C |
| Air Quality | WAQI | AQI index per city | AQI > 300 |
| Traffic | Google Maps Platform | Traffic severity score per zone | Score > 0.7 |
| Platform Status | Custom HTTP monitor | HTTP status of Swiggy/Zomato order APIs | Downtime > 2 hours |

---

## Payment Simulation Layer

**Technology:** Razorpay Sandbox + UPI Simulator

For the hackathon prototype, all payments are simulated using Razorpay's sandbox environment. No real money moves.

**Premium collection flow:**
1. Backend creates a Razorpay Order via API
2. Frontend renders Razorpay checkout (UPI / card / wallet)
3. On payment success, Razorpay sends a webhook to `/api/policy/confirm`
4. Backend verifies webhook signature → activates policy

**Payout disbursement flow:**
1. Backend calls Razorpay Payout API with worker's UPI ID and amount
2. Razorpay sandbox simulates UPI transfer
3. Razorpay sends webhook to `/api/payout/confirm`
4. Backend updates payout status to `completed` → notifies worker

---

## Deployment Topology (Prototype)

```
Vercel
└── Next.js frontend (PWA)

AWS EC2 t3.small
├── Node.js + Express backend (port 3001)
└── Python + Flask AI engine (port 5000, internal only)

MongoDB Atlas M0
└── earnguard-db (free tier, 512MB)
```

---

*EarnGuard AI — Architecture Document v2.0 | Phase-1 Hackathon*
