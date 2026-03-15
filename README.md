# EarnGuard AI
### AI-Powered Parametric Income Insurance for India's Gig Delivery Workers

> *"No claim forms. No waiting. Just automatic protection — every week."*

---

## The Problem: India's 15 Million Invisible Workers

India's gig delivery economy is one of the fastest-growing labour markets in the world. Over **15 million delivery workers** power platforms like Swiggy, Zomato, Amazon Flex, Blinkit, and Dunzo — yet every single one of them operates without a financial safety net.

These workers earn ₹3,000–₹7,000 per week, paid per delivery, with no fixed salary, no sick leave, and no employer-backed insurance. Their income is entirely at the mercy of forces outside their control:

- A week of heavy monsoon rain in Mumbai can cut deliveries by 60%
- A city-wide traffic blockage during a political rally wipes out an entire day's earnings
- A Swiggy platform outage lasting 3 hours means zero orders, zero income
- Extreme summer heat above 42°C forces workers off the road entirely

**A single bad week can mean a worker cannot pay rent, feed their family, or repay a loan instalment.**

---

## Why Existing Insurance Does Not Work for Gig Workers

The Indian insurance industry has largely ignored this segment. Here is why existing products fail:

| Problem | Why It Fails for Gig Workers |
|---|---|
| Annual premium cycles | Workers earn and spend weekly — annual premiums are unaffordable upfront |
| Claim-based payouts | Filing a claim requires documents, waiting periods, and agent visits — impractical for daily-wage earners |
| Health/accident focus | Covers hospitalisation, not the income lost while recovering or during disruptions |
| Fixed salary assumption | Actuarial models assume stable income — gig workers have high weekly variance |
| Single-event triggers | Traditional parametric insurance pays only if rainfall exceeds X mm — ignores the worker's actual income impact |
| Language and literacy barriers | Complex policy documents in English are inaccessible to most delivery workers |

**There is no product today that predicts what a worker should have earned and automatically pays the difference when disruptions occur.**

---

## Our Innovation: AI-Based Income Deviation Insurance

EarnGuard AI introduces a fundamentally new insurance model built on two layers:

### Layer 1 — AI Income Prediction
Before the week begins, our AI model predicts what a specific worker is expected to earn that week, based on:
- Their personal earnings history (last 4–8 weeks)
- Day-of-week delivery patterns
- Upcoming weather forecast for their city
- Seasonal demand signals (festivals, holidays)
- Historical disruption patterns for their delivery zone

### Layer 2 — Income Deviation Trigger
At the end of the week, the system compares actual earnings against the AI prediction. If earnings fall below the predicted amount by more than the plan's threshold **and** a confirmed disruption event occurred, a payout is automatically initiated — no claim required.

```
IF  actual_earnings < predicted_earnings × (1 − threshold)
AND disruption_confirmed = TRUE
THEN trigger_payout(worker_id, payout_amount)
```

This makes EarnGuard AI the **world's first income-prediction-based parametric insurance** product designed specifically for gig workers.

---

## Real-World Example: Ravi's Story

> Ravi is a Swiggy delivery partner in Bengaluru. He typically earns around ₹6,500 per week.

**Monday:** Ravi purchases the Standard Plan for ₹59. The AI predicts his income for the week at ₹6,300.

**Wednesday–Friday:** Heavy monsoon rain hits Bengaluru. Rainfall crosses 38mm/day. Traffic blockage scores spike to 0.8 in his zone. The system logs a confirmed `RAIN_DISRUPTION` + `TRAFFIC_DISRUPTION` event.

**Sunday night:** Ravi's actual earnings for the week: ₹4,200.

**The system calculates:**
```
Predicted income  : ₹6,300
Actual income     : ₹4,200
Deviation         : 33.3%  →  exceeds 25% threshold ✓
Disruption event  : Confirmed ✓
Payout amount     : min(₹6,300 − ₹4,200, ₹1,200 cap) = ₹1,200
```

**By 11:00 PM Sunday**, ₹1,200 is credited to Ravi's UPI account. No form. No agent. No wait.

---

## Weekly Premium Model

Insurance is priced on a **weekly cycle** to match the gig worker's earning rhythm. Workers buy a new policy every Monday and are covered for exactly seven days.

| Plan | Weekly Premium | Coverage Cap | Trigger Threshold | Best For |
|---|---|---|---|---|
| Basic | ₹29 | ₹500 | 30% income drop | Occasional coverage, low-risk zones |
| Standard | ₹59 | ₹1,200 | 25% income drop | Most workers — recommended default |
| Pro | ₹99 | ₹2,500 | 20% income drop | High-earning workers, monsoon season |

- Policy activates Monday 00:00 IST, expires Sunday 23:59 IST
- Premium collected via UPI or Razorpay at time of purchase
- Payout credited to worker's UPI ID within 2 hours of trigger
- Workers can skip any week — no lock-in, no cancellation penalty

---

## Parametric Trigger Logic

The trigger system is fully automated. No human intervention is required unless fraud is flagged.

**Disruption events monitored (hourly):**

| Event Type | Threshold | Data Source |
|---|---|---|
| Heavy rainfall | > 35mm/day | OpenWeatherMap API |
| Extreme heat | > 42°C | OpenWeatherMap API |
| Extreme cold | < 8°C | OpenWeatherMap API |
| Poor air quality | AQI > 300 | WAQI API |
| Traffic blockage | Severity score > 0.7 | Google Maps Traffic API |
| Platform outage | Downtime > 2 hours | Platform status monitoring |

**Payout formula:**
```
payout = min(predicted_income − actual_income, coverage_cap)
```

The payout is capped at the plan's coverage cap to keep the product actuarially sustainable.

---

## AI Components

### 1. Income Prediction Model
- Algorithm: Gradient Boosting Regressor
- Inputs: earnings history, delivery patterns, weather forecast, city zone, platform, seasonality
- Output: `predicted_weekly_income` (₹) with confidence interval
- Accuracy target: Mean Absolute Error < ₹400 (< 12% MAPE)

### 2. Risk Scoring Engine
- Algorithm: Random Forest Classifier
- Classifies each worker as Low / Medium / High income volatility risk
- Used to recommend the right plan and personalise future pricing
- Inputs: earnings variance, disruption history, city risk index

### 3. Fraud Detection Module
- Algorithm: Isolation Forest (unsupervised anomaly detection)
- Runs on every payout event before funds are released
- Detects: geographic claim clustering, earnings manipulation, new account abuse, repeated max claims
- Payouts with anomaly score > 0.75 are held for admin review

---

## Fraud Detection Strategy

EarnGuard AI's fraud risk is structurally lower than traditional insurance because:

1. **Disruption events are independently verified** — the system only pays when external API data confirms a real disruption in the worker's zone. A worker cannot fabricate rain or a platform outage.
2. **Earnings data cross-referenced** — reported actual income is compared against GPS delivery activity logs (production) or flagged for review if inconsistent.
3. **Isolation Forest anomaly detection** — identifies coordinated fraud rings where many workers in the same micro-zone claim simultaneously without a matching disruption event.
4. **New account cooling period** — workers registered less than 7 days ago cannot trigger a payout without manual admin approval.
5. **Consecutive cap claims** — workers hitting the coverage cap 3+ weeks in a row are automatically flagged for review.

---

## System Architecture

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

**Data flow summary:**
- Worker App → Backend API (all user actions)
- Backend API → AI Engine (income prediction, risk scoring, fraud check)
- Backend API → MongoDB (all persistence)
- Backend API → External APIs (disruption monitoring, payment processing)
- AI Engine has no direct external access — all data is passed by the backend

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React.js / Next.js (PWA, mobile-first) |
| Backend API | Node.js + Express |
| AI Engine | Python 3.10 + Flask + Scikit-learn + Pandas |
| Database | MongoDB Atlas |
| Auth | Firebase Auth (phone OTP) |
| Weather | OpenWeatherMap API |
| Air Quality | WAQI (World Air Quality Index) API |
| Traffic | Google Maps Traffic API |
| Payments | Razorpay Sandbox + UPI Simulator |
| Hosting | AWS EC2 (backend + AI) + Vercel (frontend) |
| Scheduling | node-cron (hourly monitoring + weekly trigger) |

---

## Platform Workflow (Summary)

```
Register + OTP Verify
        ↓
Profile Setup (city, platform, zone)
        ↓
AI Predicts Weekly Income
        ↓
Worker Buys Weekly Policy (UPI / Razorpay)
        ↓
System Monitors Disruptions Hourly (Weather + AQI + Traffic)
        ↓
Sunday 23:00 — Compare Actual vs Predicted Earnings
        ↓
Deviation ≥ Threshold AND Disruption Confirmed?
     ↓ YES                          ↓ NO
Fraud Check → Payout Triggered    Policy Closes (no payout)
     ↓
UPI Credit Within 2 Hours → Worker Notified via SMS + Push
```

---

## Impact for Workers and Insurers

### For Workers
- First-ever weekly income protection product designed for gig workers
- Zero paperwork — fully automated payout in under 2 hours
- Affordable entry point at ₹29/week (less than a cup of chai per day)
- Available in Hindi, Tamil, Telugu, Kannada, and English
- Builds financial resilience for India's most vulnerable working class

### For Insurers and Partners
- New addressable market: 15 million gig workers, largely uninsured
- Low claim friction reduces operational cost vs traditional insurance
- AI-driven risk scoring enables actuarially sound pricing
- Disruption-verified triggers dramatically reduce fraudulent claims
- Weekly premium cycle creates high-frequency, recurring revenue

### Year 1 Targets

| Metric | Target |
|---|---|
| Workers onboarded | 50,000 |
| Cities covered | 10 (Mumbai, Delhi, Bengaluru, Chennai, Hyderabad, Pune, Kolkata, Ahmedabad, Jaipur, Lucknow) |
| Average weekly premium | ₹59 |
| Payout trigger rate | ~18% of active policies/week during monsoon season |
| Max income protection | ₹2,500/week per worker |

EarnGuard AI directly addresses **SDG 8** (Decent Work and Economic Growth) and **SDG 10** (Reduced Inequalities) by delivering affordable, automated income protection to workers who have never had access to any financial safety net.

---

*Phase-1 Hackathon Submission — EarnGuard AI Team*
