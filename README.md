# 🛡️ EarnGuard AI
### Parametric Income Insurance for India's Gig Delivery Workers — Powered by AI

**Traditional parametric insurance asks: "Did it rain?" We ask: "Did this worker lose income — and can we prove why?"**
**Every Sunday at 23:00, the system predicts, verifies, and pays — zero human action required.**

[▶ Watch 3-min Demo](YOUR_DEMO_LINK)

---

## ⚡ The Problem

- **15 million gig delivery workers** in India earn ₹3,000–₹7,000/week with no salary, no sick leave, no safety net
- A single week of heavy rain, a traffic blockage, or a platform outage can wipe out 40–70% of weekly income
- Existing insurance products require annual premiums, claim forms, and waiting periods — none of which work for daily-wage earners

---

## 🤖 Our Solution

- **AI predicts each worker's expected weekly income** using their personal earnings history, weather forecasts, and disruption patterns
- If actual income drops below the predicted amount **and** a disruption is confirmed, a payout is automatically triggered — no claim, no form, no wait
- Every Sunday at 23:00, the automation engine runs end-to-end:

```
1. Compare predicted vs actual income
2. Confirm disruption event in worker's zone (Weather + AQI + Traffic APIs)
3. Run fraud check (Isolation Forest AI model)
4. Trigger UPI payout within 2 hours — zero human action needed
```

---

## 📊 Real Example

Ravi is a Swiggy delivery partner in Bengaluru. The AI predicts his income at **₹6,300** for the week.
Heavy rain hits — 38mm/day. His actual earnings: **₹4,200**. Deviation: **33%**, exceeding his 25% threshold.
Disruption confirmed via OpenWeatherMap. Fraud check passes.
**₹1,200 credited to Ravi's UPI by 11 PM Sunday.**
Ravi did nothing. No form. No call. No wait.

---

## ⚡ How It Works

1. **Register** — Phone OTP, city, delivery platform, UPI ID (under 3 minutes)
2. **Predict** — AI generates a personalised weekly income prediction every Monday
3. **Monitor** — System polls Weather, AQI, and Traffic APIs every hour during the policy week
4. **Auto-Payout** — Sunday 23:00: deviation check → fraud check → UPI credit within 2 hours

---

## 📊 Plans

| Plan | Weekly Premium | Coverage Cap | Triggers At |
|---|---|---|---|
| Basic | ₹29 | ₹500 | 30% income drop |
| Standard | ₹59 | ₹1,200 | 25% income drop |
| Pro | ₹99 | ₹2,500 | 20% income drop |

Policy runs Monday 00:00 – Sunday 23:59 IST. No lock-in. Skip any week.

---

## 🛡️ Guidewire Platform Mapping

| Guidewire Product | EarnGuard AI Usage |
|---|---|
| **PolicyCenter** | Weekly policy lifecycle — purchase, activate, close every 7 days |
| **ClaimCenter** | Automated payout trigger engine — deviation check replaces manual claim filing |
| **DataHub** | Real-time disruption data ingestion and AI feature pipeline for income prediction |

---

## ⚡ Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React / Next.js PWA — mobile-first, 5 languages, works on low-end Android |
| Backend | Node.js + Express on AWS EC2 |
| AI Engine | Python + Flask + Scikit-learn (internal microservice) |
| Database | MongoDB Atlas |
| Payments | Razorpay Sandbox + UPI |
| External Data | OpenWeatherMap, WAQI, Google Maps Traffic APIs |
| Auth | Firebase OTP |

---

## 🤖 Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                        WORKER APP                            │
│                  React / Next.js (Mobile Web)                │
│         Onboarding │ Dashboard │ Policy │ Notifications      │
└───────────────────────────┬──────────────────────────────────┘
                            │ HTTPS / REST + JWT
┌───────────────────────────▼──────────────────────────────────┐
│                      BACKEND API                             │
│                   Node.js + Express                          │
│    Auth │ Worker │ Policy │ Predict │ Monitor │ Payout       │
└────┬──────────────────────┬──────────────────┬───────────────┘
     │ Internal HTTP        │ Mongoose ODM     │ HTTPS
┌────▼──────────┐  ┌────────▼───────────┐  ┌──▼────────────────────────┐
│  AI ENGINE    │  │     DATABASE       │  │     EXTERNAL APIs         │
│ Python+Flask  │  │   MongoDB Atlas    │  │  OpenWeatherMap (Weather)  │
│ Scikit-learn  │  │ Workers │ Policies │  │  WAQI (Air Quality)       │
│ Pandas/NumPy  │  │ Payouts │ Disrupts │  │  Google Maps (Traffic)    │
└───────────────┘  └────────────────────┘  │  Razorpay Sandbox (Pay)   │
                                           └───────────────────────────┘
```

---

## 🤖 AI Models

| Model | Algorithm | Purpose |
|---|---|---|
| Income Prediction | Gradient Boosting Regressor | Predicts personalised weekly income per worker |
| Risk Scoring | Random Forest Classifier | Classifies Low / Medium / High risk for plan recommendation |
| Fraud Detection | Isolation Forest | Unsupervised anomaly detection before every payout |

---

## 📊 Docs

- [AI Model — prediction logic, features, fraud detection](docs/ai-model.md)
- [Architecture — full system design, API contracts, database schema](docs/architecture.md)
- [Workflow — end-to-end user journey, automation engine, admin flow](docs/workflow.md)

---

*Guidewire DevTrails 2026 — EarnGuard AI Team*
