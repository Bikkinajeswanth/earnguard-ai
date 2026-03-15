# EarnGuard AI — Platform Workflow

---

## Design Philosophy

The EarnGuard AI workflow is built around three principles:

1. **Zero friction for workers** — Registration takes under 3 minutes. Payouts require zero action from the worker.
2. **Weekly rhythm alignment** — Every step maps to the gig worker's natural weekly earning cycle.
3. **Automation by default** — Disruption monitoring, deviation detection, and payout triggering are fully automated. Human intervention only occurs when fraud is flagged.

---

## Complete User Journey

### Step 1: Worker Registration and Verification

**Actor:** Delivery worker (first-time user)
**Time required:** ~3 minutes
**Channel:** Mobile web app (Android, low-bandwidth optimised)

1. Worker visits EarnGuard AI on their phone browser (no app install required — PWA).
2. Selects preferred language: Hindi / Tamil / Telugu / Kannada / English.
3. Enters mobile number → taps "Send OTP".
4. Firebase Auth sends a 6-digit OTP via SMS.
5. Worker enters OTP → identity verified → JWT issued and stored in browser.
6. Worker completes profile:
   - Full name
   - City (dropdown: Mumbai, Delhi, Bengaluru, Chennai, Hyderabad, Pune, Kolkata, Ahmedabad, Jaipur, Lucknow)
   - Primary delivery platform (Swiggy / Zomato / Amazon Flex / Blinkit / Dunzo / Other)
   - Delivery zone within city (e.g., Koramangala, Indiranagar)
   - UPI ID (for payout credit)
   - Approximate weekly earnings (self-reported, used as cold-start baseline)
7. Worker profile saved to MongoDB `workers` collection.
8. System immediately queues an AI income prediction job for this worker.

---

### Step 2: AI Income Prediction

**Actor:** System (automated — runs post-registration and every Monday 06:00 IST)
**Engine:** Python AI model (`/predict/income`)

1. Backend collects input features for the worker:
   - Historical weekly earnings (last 4–8 weeks, or self-reported baseline for new workers)
   - Day-of-week delivery pattern
   - City, zone, platform
   - Upcoming week's weather forecast (OpenWeatherMap 7-day)
   - Upcoming week's AQI forecast (WAQI)
   - `is_festival_week` flag (static Indian festival calendar)
   - `city_disruption_index` (computed from historical disruptions in MongoDB)

2. Backend sends feature payload to AI engine `POST /predict/income`.

3. AI engine returns:
   ```json
   {
     "predicted_weekly_income": 6300,
     "confidence_interval": [5800, 6800],
     "risk_tier": "Medium",
     "model_version": "v1.3"
   }
   ```

4. Result stored in `predictions` collection.

5. Worker's dashboard now shows:
   - "Your predicted earnings this week: ₹6,300"
   - Risk tier badge (Low / Medium / High)
   - Recommended plan based on risk tier

**Cold start handling:** New workers with no earnings history are assigned the median prediction of their city + zone + platform cohort, with a 10% conservative discount applied.

---

### Step 3: Weekly Policy Purchase

**Actor:** Delivery worker
**Purchase window:** Saturday 18:00 – Sunday 23:59 IST (for the following week)
**Also available:** Monday 00:00 – Monday 10:00 IST (grace window)

1. Worker navigates to `/buy-policy`.
2. Sees three plan cards (Basic / Standard / Pro) with:
   - Weekly premium amount
   - Coverage cap
   - Trigger threshold
   - Recommended badge on the plan matching their risk tier
3. Worker selects a plan and taps "Buy Now".
4. Razorpay checkout opens — worker pays via UPI, wallet, or card.
5. On payment success:
   - Razorpay sends webhook to `/api/policy/confirm`
   - Backend verifies webhook signature
   - Policy record created in `policies` collection:
     ```
     status: "active"
     weekStart: Monday 00:00 IST
     weekEnd: Sunday 23:59 IST
     ```
   - Confirmation SMS sent: *"EarnGuard policy active. Covered up to ₹1,200 this week."*
   - Policy card appears on worker's dashboard with countdown timer.

---

### Step 4: Real-Time Disruption Monitoring

**Actor:** System (automated, continuous)
**Frequency:** Every hour via node-cron scheduled job

1. Backend fetches the list of all cities with at least one active policy.
2. For each city, polls external APIs:
   - **OpenWeatherMap:** Current rainfall (mm), temperature (°C), hourly forecast
   - **WAQI:** Current AQI index
   - **Google Maps Traffic:** Traffic severity score for each active delivery zone
   - **Platform status monitor:** HTTP health check on Swiggy and Zomato order APIs
3. Each reading is evaluated against disruption thresholds:

   | Condition | Event Type Logged |
   |---|---|
   | Rainfall > 35mm in past 24h | `RAIN_DISRUPTION` |
   | Temperature > 42°C | `HEAT_DISRUPTION` |
   | Temperature < 8°C | `COLD_DISRUPTION` |
   | AQI > 300 | `AQI_DISRUPTION` |
   | Traffic severity > 0.7 | `TRAFFIC_DISRUPTION` |
   | Platform downtime > 2 hours | `PLATFORM_DISRUPTION` |

4. Confirmed disruption events are written to the `disruptions` collection with:
   - `city`, `zone`, `type`, `severity`, `startTime`, `source`
5. Workers in affected zones see a live alert banner on their dashboard:
   *"⚠️ Heavy rain disruption detected in Koramangala. Your policy is active."*

---

### Step 5: End-of-Week Earnings Submission

**Actor:** Delivery worker (manual for prototype; platform API in production)
**Deadline:** Sunday 22:00 IST

1. Worker receives a push notification + SMS on Sunday evening:
   *"Submit your earnings for this week to check if you qualify for a payout."*
2. Worker opens the app and enters their actual weekly earnings (₹).
   - In production: earnings are pulled automatically from platform APIs.
3. Submission stored in `earnings` collection:
   ```
   workerId, weekStart, actualIncome, submittedAt
   ```

---

### Step 6: Income Deviation Detection

**Actor:** System (automated, runs every Sunday at 23:00 IST)

1. System queries all `active` policies for the current week.
2. For each policy, fetches:
   - `predicted_weekly_income` from `predictions`
   - `actual_weekly_income` from `earnings`
3. Calculates deviation:
   ```
   deviation = (predicted − actual) / predicted
   ```
4. Checks if a disruption event was logged for the worker's city/zone during the policy week.
5. Evaluates trigger condition:
   ```
   IF deviation >= plan_threshold AND disruption_confirmed = TRUE
   THEN → proceed to fraud check and payout
   ELSE → close policy with status "no_payout"
   ```
6. Workers whose policies close without a payout receive an SMS:
   *"Your EarnGuard policy for this week has closed. No income deviation detected. Stay protected — renew for next week."*

---

### Step 7: Fraud Check

**Actor:** System (automated, runs before every payout)
**Engine:** Python AI model (`/detect/fraud`)

1. Backend sends payout event features to AI engine `POST /detect/fraud`:
   - Worker ID, zone, policy age, deviation amount, disruption type
   - Number of simultaneous claims in the same micro-zone
   - Worker's claim history (consecutive max claims, etc.)
2. Isolation Forest model returns:
   ```json
   {
     "anomaly_score": 0.42,
     "flag": false,
     "action": "proceed"
   }
   ```
3. Decision logic:
   - `anomaly_score < 0.75` → proceed to payout automatically
   - `anomaly_score ≥ 0.75` → hold payout, flag for admin review
4. Flagged payouts appear in the admin fraud review queue with the reason code.

---

### Step 8: Automatic Payout

**Actor:** System (automated for clean payouts; admin-approved for flagged ones)
**SLA:** Within 2 hours of Sunday 23:00 IST trigger

1. Payout amount calculated:
   ```
   payout = min(predicted_income − actual_income, coverage_cap)
   ```
   **Example:** Predicted ₹6,300 | Actual ₹4,200 | Cap ₹1,200 → **Payout = ₹1,200**

2. Backend calls Razorpay Payout API with:
   - Worker's UPI ID
   - Payout amount
   - Reference: `policyId`

3. Payout record created in `payouts` collection with status `processing`.

4. Razorpay sandbox simulates UPI transfer and sends webhook to `/api/payout/confirm`.

5. On webhook receipt:
   - Payout status updated to `completed`
   - Policy status updated to `closed_with_payout`
   - Worker receives push notification + SMS:
     *"✅ ₹1,200 has been credited to your UPI account from EarnGuard AI. Stay protected next week!"*

---

### Step 9: Dashboard Update and Renewal Prompt

**Actor:** Worker (passive view)

1. Worker opens app and sees the weekly summary card:
   - Predicted income vs actual income (bar chart)
   - Disruption events that occurred during the week
   - Payout amount received (or "No payout this week")
   - Total savings protected since joining EarnGuard AI
2. Renewal prompt displayed:
   *"Next week's policy window opens Saturday 18:00. Your predicted income: ₹6,100."*
3. Full payout history accessible under the worker's profile.

---

## Admin Workflow (Parallel to Worker Journey)

| Time | Admin Action |
|---|---|
| Continuous | Monitor live disruption map across all cities |
| Sunday 23:00 | Review fraud-flagged payout queue |
| Monday 00:00 | Approve or reject held payouts |
| Monday 06:00 | Review weekly analytics report (policies sold, payouts triggered, payout rate by city) |
| On demand | Export payout data for actuarial review |

---

## Complete Workflow Diagram

```
[Worker] Opens App → Selects Language
         ↓
[Worker] Enters Phone → OTP Verified (Firebase)
         ↓
[Worker] Completes Profile (city, platform, zone, UPI ID)
         ↓
[System] AI Predicts Weekly Income → Stored in DB
         ↓
[Worker] Views Prediction → Selects Plan → Pays via UPI
         ↓
[System] Policy Activated (Mon 00:00 – Sun 23:59)
         ↓
[System] Hourly: Poll Weather + AQI + Traffic APIs
         ↓ (disruption detected?)
[System] Log Disruption Event → Alert Worker on Dashboard
         ↓
[Worker] Sunday: Submits Actual Weekly Earnings
         ↓
[System] Sunday 23:00: Calculate Deviation
         ↓
    Deviation ≥ Threshold?
    AND Disruption Confirmed?
         ↓ YES                    ↓ NO
[System] Fraud Check         Policy Closes → No Payout
         ↓
    Score < 0.75?
    ↓ YES          ↓ NO
Payout Triggered  Admin Review Queue
         ↓
[Razorpay] UPI Credit → Worker SMS + Push Notification
         ↓
[Worker] Views Payout Summary → Prompted to Renew
```

---

*EarnGuard AI — Workflow Document v2.0 | Phase-1 Hackathon*
