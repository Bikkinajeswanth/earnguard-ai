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

---

## Adversarial Defense & Anti-Spoofing Strategy

A critical vulnerability in any location-aware insurance system is GPS spoofing — where a malicious user fakes their device location to appear inside a disruption zone and trigger a false payout. EarnGuard AI addresses this through a multi-signal verification layer that goes well beyond GPS coordinates.

---

### How the System Tells a Genuine Worker from a Spoofer

A real delivery worker affected by heavy rain behaves in a specific, consistent way: their order acceptance rate drops, completed deliveries fall, movement patterns become erratic or stationary, and their earnings decline in proportion to the disruption severity. A spoofer, by contrast, fakes a location but cannot simultaneously fake all the correlated signals that accompany genuine disruption impact.

The system cross-validates four dimensions before treating a disruption as genuine for a specific worker:

| Dimension | Genuine Worker Signal | Spoofer Red Flag |
|---|---|---|
| Delivery activity | Orders accepted and completed drop sharply during disruption hours | Normal or high order activity despite claimed disruption |
| Movement pattern | GPS trace shows slow, disrupted, or stationary movement consistent with rain/traffic | GPS coordinates jump unnaturally or remain perfectly static |
| Weather-behaviour consistency | Earnings drop correlates with the exact hours the disruption was active | Earnings drop does not align with disruption window timing |
| Historical behaviour | Pattern matches the worker's own baseline from prior disruption weeks | First-time disruption claim with no prior history of income variance |

---

### Data Signals Used Beyond GPS

The system collects and cross-references the following signals at payout evaluation time:

**Device and Motion Signals**
- Accelerometer and gyroscope data from the worker's phone — a stationary spoof device shows no motion variance; a real worker navigating rain shows consistent movement noise
- GPS trace continuity — real movement produces smooth coordinate transitions; spoofed locations often show teleportation jumps or perfectly looped paths
- Device mock location flag — Android exposes a `mock_location` API flag that is checked at session start and logged

**Delivery Activity Logs**
- Orders accepted per hour during the disruption window vs the worker's historical hourly average
- Orders completed vs orders accepted (completion ratio) — genuine disruption causes both to fall; a spoofer may show normal completion ratios
- Time-on-platform during disruption hours — a worker genuinely unable to deliver goes offline; a spoofer may remain online with normal session activity

**Weather-Behaviour Consistency**
- Earnings drop is compared against the hourly disruption severity curve — if rainfall peaked between 14:00–18:00 and the worker's earnings dropped in a different window, the claim is flagged
- Zone-level disruption confirmation requires correlated income drops across multiple workers in the same zone before the disruption is treated as zone-confirmed (crowd-sourced validation)

**Time-Based Activity Patterns**
- Login and session timestamps during the disruption window are compared against the worker's historical activity schedule
- A worker who is never active on Wednesday afternoons but suddenly submits a claim for a Wednesday disruption is flagged for review

**Network and Device Patterns**
- IP geolocation is cross-checked against the claimed delivery zone — a worker claiming disruption in Koramangala but connecting from a Delhi IP is flagged
- VPN usage detected via IP reputation databases is logged as a risk signal
- Device fingerprint consistency — a sudden change in device ID mid-week is flagged

**Claim Frequency and History**
- Workers claiming payouts more than 3 consecutive weeks are reviewed regardless of anomaly score
- Workers whose claim amounts consistently hit the exact coverage cap are flagged for earnings manipulation review
- Claim rate is compared against the worker's city-zone cohort — outliers above 2 standard deviations from the cohort claim rate are escalated

**Fraud Ring and Cluster Detection**
- If more than 15% of workers in a micro-zone (radius < 500m) submit claims in the same week without a zone-confirmed disruption event, the entire cluster is held for admin review
- New account clusters — multiple accounts registered from the same device or IP within 48 hours are flagged as potential synthetic identity fraud
- Coordinated UPI ID patterns — payouts going to a small set of UPI accounts across many worker IDs are flagged as potential mule account rings

---

### UX Balance: Protecting Honest Workers

The anti-spoofing layer is designed to be invisible to legitimate workers. The system's default posture is trust, not suspicion.

**Auto-approval for low-risk claims:**
- Claims with an anomaly score below 0.75 are approved and paid automatically within 2 hours — no friction, no delay
- Workers with 4+ weeks of clean claim history and a consistent delivery pattern receive a **Trusted Worker** status that raises their auto-approval threshold to 0.85
- Zone-confirmed disruptions (where multiple workers in the same area show correlated income drops) receive a lower scrutiny threshold — the disruption itself is the primary evidence

**Transparent flagging for suspicious claims:**
- Workers whose claims are held for review receive an immediate SMS: *"Your payout is being reviewed. We will update you within 4 hours."*
- The review SLA is 4 hours — workers are never left waiting without a status update
- If a held claim is approved after review, the payout is credited with a brief explanation: *"Your claim was verified. ₹1,200 has been credited."*
- If a claim is rejected, the worker receives a plain-language reason and a link to a one-tap appeal form

**Trust score system:**
- Every worker has a rolling trust score (0.0–1.0) updated weekly based on claim history, delivery consistency, and device signal integrity
- Trust score operates silently in the background — it is never surfaced to the worker as a number
- A high trust score reduces the anomaly threshold required for auto-approval
- A low trust score does not block payouts — it routes them to faster human review rather than automatic rejection

**Fallback checks before rejection:**
- No claim is rejected by the automated system alone — all rejections require a human admin action
- Before a claim is rejected, the system runs a secondary check: if the worker's zone had a confirmed disruption and their delivery logs show a genuine income drop, the claim is escalated for approval even if other signals are ambiguous
- Workers can submit a one-tap appeal with optional supporting evidence (e.g., a photo of flooded roads) — this is optional and never required for standard claims

---

*Phase-1 Hackathon Submission — EarnGuard AI Team*
