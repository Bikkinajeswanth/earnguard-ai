# EarnGuard AI — UI Concept Document
### Prototype Screen Designs for Phase-1 Hackathon Submission

---

## Design Principles

EarnGuard AI serves two very different users: a delivery worker on a low-end Android phone between shifts, and a platform admin on a desktop browser. Every design decision is made with that contrast in mind.

**Worker app (mobile-first):**
- Large tap targets — minimum 48px touch areas, no small links
- High contrast colours — readable in direct sunlight on a cracked screen
- Numbers over text — income figures are larger than any label
- Vernacular-first — app defaults to the worker's chosen language (Hindi, Tamil, Telugu, Kannada, English)
- Zero insurance jargon — "income drop" not "deviation threshold", "protection" not "indemnity"
- One action per screen — no decision fatigue

**Admin dashboard (desktop-first):**
- Data density over simplicity — admins need multiple metrics visible simultaneously
- Actionable tables — every row has a direct action button
- Real-time indicators — live disruption status, payout queue depth

---

## Screen 1: Worker Onboarding

**Route:** `/onboard`
**User:** New delivery worker, first visit
**Goal:** Collect the minimum information needed to generate an AI income prediction and activate the account in under 3 minutes

---

### 1a — Language Selection

The very first screen. No login, no form — just a language choice. This signals immediately that the app was built for the worker, not for an English-speaking office.

```
┌──────────────────────────────────────┐
│                                      │
│                                      │
│           🛡️ EarnGuard AI            │
│                                      │
│      Income Protection for           │
│      Delivery Workers                │
│                                      │
│      ─────────────────────           │
│                                      │
│      Choose your language            │
│                                      │
│      ┌────────────────────────┐      │
│      │  🇮🇳  हिंदी           │      │
│      └────────────────────────┘      │
│      ┌────────────────────────┐      │
│      │  🇬🇧  English          │      │
│      └────────────────────────┘      │
│      ┌────────────────────────┐      │
│      │  🇮🇳  தமிழ்           │      │
│      └────────────────────────┘      │
│      ┌────────────────────────┐      │
│      │  🇮🇳  తెలుగు          │      │
│      └────────────────────────┘      │
│      ┌────────────────────────┐      │
│      │  🇮🇳  ಕನ್ನಡ          │      │
│      └────────────────────────┘      │
│                                      │
└──────────────────────────────────────┘
```

**Behaviour:**
- Tapping a language saves the preference to local storage and navigates to the phone number screen
- All subsequent screens render entirely in the chosen language
- Language can be changed later from the profile settings menu

---

### 1b — Phone Number and OTP Verification

```
┌──────────────────────────────────────┐
│  ← Back                              │
│                                      │
│  Step 1 of 2                         │
│  ██████████░░░░░░░░░░  50%           │
│                                      │
│  Enter your mobile number            │
│                                      │
│  ┌──┬─────────────────────────────┐  │
│  │+91│  98765 43210               │  │
│  └──┴─────────────────────────────┘  │
│                                      │
│  ┌──────────────────────────────┐    │
│  │        Send OTP              │    │
│  └──────────────────────────────┘    │
│                                      │
│  ─────────────────────────────────   │
│                                      │
│  Enter the 6-digit OTP               │
│  sent to +91 98765 43210             │
│                                      │
│  ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐│
│  │ 4 │ │ 8 │ │ _ │ │ _ │ │ _ │ │ _ ││
│  └───┘ └───┘ └───┘ └───┘ └───┘ └───┘│
│                                      │
│  Resend OTP in  0:38                 │
│                                      │
│  ┌──────────────────────────────┐    │
│  │          Verify              │    │
│  └──────────────────────────────┘    │
│                                      │
└──────────────────────────────────────┘
```

**Fields:**
- Mobile number — numeric input, auto-prefixed with +91, 10-digit validation
- OTP — 6 individual digit boxes, auto-advances focus on each digit entry

**Behaviour:**
- OTP auto-reads from SMS on Android using the SMS Retriever API — worker may not need to type it
- Resend countdown (60 seconds) prevents OTP spam
- After 3 failed OTP attempts: account temporarily locked for 10 minutes with a clear message
- On success: JWT issued, stored in browser, worker proceeds to profile setup

---

### 1c — Profile Setup

```
┌──────────────────────────────────────┐
│  ← Back                              │
│                                      │
│  Step 2 of 2 — Your Profile          │
│  ████████████████████  100%          │
│                                      │
│  Your Name                           │
│  ┌──────────────────────────────┐    │
│  │  Ravi Kumar                  │    │
│  └──────────────────────────────┘    │
│                                      │
│  Your City                           │
│  ┌──────────────────────────────┐    │
│  │  Bengaluru                 ▼ │    │
│  └──────────────────────────────┘    │
│                                      │
│  Your Delivery Zone                  │
│  ┌──────────────────────────────┐    │
│  │  Koramangala               ▼ │    │
│  └──────────────────────────────┘    │
│                                      │
│  Your Delivery Platform              │
│  ┌──────────┐  ┌──────────┐         │
│  │ ● Swiggy │  │ ○ Zomato │         │
│  └──────────┘  └──────────┘         │
│  ┌──────────────┐  ┌────────┐       │
│  │ ○ Amazon Flex│  │○Blinkit│       │
│  └──────────────┘  └────────┘       │
│                                      │
│  Your UPI ID (for payouts)           │
│  ┌──────────────────────────────┐    │
│  │  ravi@upi                    │    │
│  └──────────────────────────────┘    │
│                                      │
│  Approximate weekly earnings (₹)     │
│  Used only as a starting estimate    │
│  ┌──────────────────────────────┐    │
│  │  6500                        │    │
│  └──────────────────────────────┘    │
│                                      │
│  ┌──────────────────────────────┐    │
│  │      Create My Account       │    │
│  └──────────────────────────────┘    │
│                                      │
└──────────────────────────────────────┘
```

**Fields and validation:**

| Field | Type | Validation |
|---|---|---|
| Name | Text | Required, 2–50 characters |
| City | Dropdown | 10 supported cities |
| Delivery Zone | Dropdown | Populated based on selected city |
| Platform | Radio buttons | One selection required |
| UPI ID | Text | Format: `name@bank` — inline validation |
| Weekly earnings | Number | ₹500–₹20,000 range, used as cold-start baseline only |

**Behaviour:**
- Delivery Zone dropdown is dynamically populated based on the selected City
- UPI ID shows a green tick on valid format, red border on invalid
- Weekly earnings field is clearly labelled as "approximate" — it is only used until the AI has 4 weeks of real data
- On submit: worker profile saved, AI income prediction triggered in the background, worker redirected to dashboard with a "Welcome" banner

---

## Screen 2: Weekly Insurance Plan Selection

**Route:** `/buy-policy`
**User:** Registered worker ready to purchase coverage for the upcoming week
**Goal:** Help the worker choose and pay for the right plan in under 60 seconds

```
┌──────────────────────────────────────┐
│  ← Dashboard                         │
│                                      │
│  Your AI-predicted income            │
│  this week:                          │
│                                      │
│         ₹6,300                       │
│         Mon 14 Jul – Sun 20 Jul      │
│                                      │
│  ─────────────────────────────────   │
│  Choose your protection plan         │
│                                      │
│  ┌────────────────────────────────┐  │
│  │  BASIC                ₹29/week │  │
│  │  ─────────────────────────     │  │
│  │  Coverage cap:        ₹500     │  │
│  │  Triggers when income          │  │
│  │  drops by 30% or more          │  │
│  │  Best for: Low-risk zones      │  │
│  └────────────────────────────────┘  │
│                                      │
│  ┌────────────────────────────────┐  │
│  │ ⭐ STANDARD           ₹59/week │  │
│  │  ─────────────────────────     │  │
│  │  Coverage cap:      ₹1,200     │  │
│  │  Triggers when income          │  │
│  │  drops by 25% or more          │  │
│  │  Best for: Most workers        │  │
│  │                                │  │
│  │  ✅ RECOMMENDED FOR YOU        │  │
│  └────────────────────────────────┘  │
│                                      │
│  ┌────────────────────────────────┐  │
│  │  PRO                  ₹99/week │  │
│  │  ─────────────────────────     │  │
│  │  Coverage cap:      ₹2,500     │  │
│  │  Triggers when income          │  │
│  │  drops by 20% or more          │  │
│  │  Best for: Monsoon season      │  │
│  └────────────────────────────────┘  │
│                                      │
│  Policy valid: Mon 00:00–Sun 23:59   │
│  Payout credited within 2 hours      │
│  No lock-in. Skip any week.          │
│                                      │
│  ┌──────────────────────────────┐    │
│  │      Pay ₹59 via UPI         │    │
│  └──────────────────────────────┘    │
│                                      │
└──────────────────────────────────────┘
```

**Plan card details:**

| Plan | Premium | Coverage Cap | Trigger Threshold |
|---|---|---|---|
| Basic | ₹29/week | ₹500 | Income drops ≥ 30% |
| Standard ⭐ | ₹59/week | ₹1,200 | Income drops ≥ 25% |
| Pro | ₹99/week | ₹2,500 | Income drops ≥ 20% |

**Behaviour:**
- The recommended plan badge is set by the AI risk scoring engine — Medium risk workers see Standard recommended, High risk workers see Pro
- Tapping a plan card selects it and updates the CTA button label and amount
- The predicted income figure at the top is pulled from the AI prediction for the current week
- Tapping "Pay via UPI" opens the Razorpay checkout bottom sheet
- On payment success: policy activated, confirmation SMS sent, worker redirected to dashboard with active policy card visible

---

## Screen 3: Worker Income Dashboard

**Route:** `/dashboard`
**User:** Worker with an active policy during the coverage week
**Goal:** Give the worker a clear, real-time view of their income status, active protection, and payout likelihood

```
┌──────────────────────────────────────┐
│  Hi Ravi 👋                 🔔  ☰   │
│                                      │
│  ┌────────────────────────────────┐  │
│  │  🛡️ ACTIVE POLICY             │  │
│  │  Standard Plan  |  ₹1,200 cap │  │
│  │  Mon 14 Jul – Sun 20 Jul       │  │
│  │  ████████████░░░░  5 days left │  │
│  └────────────────────────────────┘  │
│                                      │
│  This Week's Earnings                │
│  ─────────────────────────────────   │
│                                      │
│  Predicted Income    Actual So Far   │
│      ₹6,300              ₹4,200      │
│                           ⚠️ -33%    │
│                                      │
│  Daily Breakdown                     │
│  ┌──────────────────────────────┐    │
│  │  Mon  ████████████  ₹1,100   │    │
│  │  Tue  ██████████    ₹900     │    │
│  │  Wed  ████          ₹400  🌧️ │    │
│  │  Thu  ███           ₹350  🌧️ │    │
│  │  Fri  ████          ₹450  🌧️ │    │
│  │  Sat  ──            ₹0       │    │
│  │  Sun  ──            ₹0       │    │
│  └──────────────────────────────┘    │
│                                      │
│  ┌────────────────────────────────┐  │
│  │  ⚠️ DISRUPTION ACTIVE          │  │
│  │  🌧️ Heavy Rain — Koramangala   │  │
│  │  Wed 16 Jul – Fri 18 Jul       │  │
│  │  Confirmed via OpenWeatherMap  │  │
│  └────────────────────────────────┘  │
│                                      │
│  Income deviation:    33%            │
│  Your plan threshold: 25%            │
│  Payout status:  LIKELY ✅           │
│                                      │
│  ─────────────────────────────────   │
│  Claim History                       │
│                                      │
│  Jul 6   ₹800    ✅ Credited         │
│  Jun 29  ₹0      — No disruption     │
│  Jun 22  ₹1,200  ✅ Credited         │
│  Jun 15  ₹450    ✅ Credited         │
│                                      │
│  [ View Full History ]               │
│                                      │
└──────────────────────────────────────┘
```

**Dashboard sections:**

| Section | What It Shows |
|---|---|
| Active Policy Card | Plan name, coverage cap, policy validity dates, days remaining progress bar |
| Earnings Summary | AI-predicted income vs actual earnings so far, deviation percentage with colour coding |
| Daily Breakdown | Per-day earnings bar chart with disruption icons on affected days |
| Disruption Alert | Live banner when a disruption is active in the worker's zone, with source attribution |
| Deviation Status | Current deviation %, plan threshold, and payout likelihood indicator |
| Claim History | Last 4 payouts with date, amount, and status — scrollable |

**Colour coding for deviation:**
- 0–10% drop → green (on track)
- 10–24% drop → amber (watch)
- ≥ 25% drop → red with ⚠️ (payout likely)

**Behaviour:**
- Dashboard refreshes every 30 minutes automatically
- Disruption alert banner appears and disappears based on live API data
- "Payout Likely" status is an estimate only — final determination runs Sunday 23:00 IST
- Workers without an active policy see a "Buy This Week's Policy" CTA instead of the policy card

---

## Screen 4: Claim Notification Screen

**Route:** `/notifications/[payoutId]`
**User:** Worker after a payout has been triggered
**Goal:** Confirm the payout clearly, explain why it happened, and prompt renewal

---

### 4a — Push Notification (Lock Screen)

```
┌──────────────────────────────────────┐
│  🛡️ EarnGuard AI          11:47 PM  │
│                                      │
│  ₹1,200 credited to your UPI        │
│  account. Heavy rain disruption      │
│  confirmed. Tap to view details.     │
│                                      │
└──────────────────────────────────────┘
```

---

### 4b — In-App Payout Success Screen

```
┌──────────────────────────────────────┐
│  ← Back                              │
│                                      │
│                                      │
│              ✅                      │
│                                      │
│         Payout Successful            │
│                                      │
│              ₹1,200                  │
│       Credited to ravi@upi           │
│       Sunday 20 Jul, 11:47 PM        │
│                                      │
│  ─────────────────────────────────   │
│                                      │
│  Why you received this payout        │
│                                      │
│  Predicted income       ₹6,300       │
│  Actual income          ₹4,200       │
│  Income drop               33%       │
│  Your plan threshold       25%       │
│  Deviation above threshold  ✅       │
│                                      │
│  Disruption confirmed:               │
│  🌧️ Heavy Rain — Koramangala         │
│  Wed 16 Jul – Fri 18 Jul             │
│  Source: OpenWeatherMap              │
│                                      │
│  Payout calculation:                 │
│  ₹6,300 − ₹4,200 = ₹2,100           │
│  Capped at plan limit:    ₹1,200     │
│  Amount credited:         ₹1,200     │
│                                      │
│  ─────────────────────────────────   │
│                                      │
│  Protect yourself next week too      │
│                                      │
│  ┌──────────────────────────────┐    │
│  │   Buy Next Week's Policy     │    │
│  └──────────────────────────────┘    │
│                                      │
└──────────────────────────────────────┘
```

---

### 4c — No-Payout Notification (Policy Closed, No Trigger)

```
┌──────────────────────────────────────┐
│  ← Back                              │
│                                      │
│              📋                      │
│                                      │
│         Policy Closed                │
│       No payout this week            │
│                                      │
│  ─────────────────────────────────   │
│                                      │
│  Your earnings were within the       │
│  normal range this week.             │
│                                      │
│  Predicted income       ₹6,300       │
│  Actual income          ₹5,900       │
│  Income drop              6.3%       │
│  Your plan threshold      25.0%      │
│                                      │
│  No confirmed disruption event       │
│  in your zone this week.             │
│                                      │
│  ─────────────────────────────────   │
│                                      │
│  Great week, Ravi! 🎉                │
│  Stay protected for next week.       │
│                                      │
│  ┌──────────────────────────────┐    │
│  │     Renew for Next Week      │    │
│  └──────────────────────────────┘    │
│                                      │
└──────────────────────────────────────┘
```

**Key UX decisions across all notification states:**
- Payout amount is the largest text on the screen — the worker sees it before reading anything else
- The payout calculation is shown step-by-step so the worker understands exactly how ₹1,200 was arrived at
- Disruption source is cited (OpenWeatherMap) — builds trust that the system is objective, not arbitrary
- Renewal CTA appears on both payout and no-payout screens — the moment after a policy closes is the highest-intent moment for renewal

---

## Screen 5: Admin Analytics Dashboard

**Route:** `/admin`
**User:** EarnGuard AI platform administrator
**Device:** Desktop browser (1280px+ width)
**Goal:** Give the admin a complete real-time view of platform health, payout activity, fraud queue, and disruption events

---

### 5a — Top Navigation and KPI Cards

```
┌──────────────────────────────────────────────────────────────────────┐
│  🛡️ EarnGuard AI — Admin                          Week: Jul 14–20   │
│  Overview  │  Workers  │  Disruptions  │  Payouts  │  Fraud Queue   │
└──────────────────────────────────────────────────────────────────────┘

┌────────────────┐ ┌────────────────┐ ┌────────────────┐ ┌────────────┐
│ Active Workers │ │ Active Policies│ │ Premiums       │ │ Fraud      │
│                │ │                │ │ Collected      │ │ Flagged    │
│    12,480      │ │    4,821       │ │  ₹2,84,439     │ │    12      │
│  ▲ 11% WoW    │ │  ▲ 8% WoW     │ │  This week     │ │  Pending   │
└────────────────┘ └────────────────┘ └────────────────┘ └────────────┘

┌────────────────┐ ┌────────────────┐ ┌────────────────┐ ┌────────────┐
│ Payouts        │ │ Payout         │ │ Total Payout   │ │ Avg Payout │
│ Triggered      │ │ Rate           │ │ Amount         │ │ Amount     │
│                │ │                │ │                │ │            │
│     867        │ │    18.0%       │ │  ₹9,24,400     │ │  ₹1,066    │
│  of 4,821      │ │  of policies   │ │  This week     │ │  Per claim │
└────────────────┘ └────────────────┘ └────────────────┘ └────────────┘
```

---

### 5b — Disruption Analytics Panel

```
┌──────────────────────────────────────┐ ┌───────────────────────────────┐
│  Active Disruptions (Live)           │ │  Payout Rate by City          │
│                                      │ │                               │
│  🌧️ Mumbai — RAIN_DISRUPTION        │ │  Mumbai      ████████   24%   │
│     Severity: HIGH                   │ │  Bengaluru   ██████     18%   │
│     Since: Wed 14 Jul 14:00          │ │  Chennai     █████      15%   │
│     Workers affected: 1,240          │ │  Delhi       ████       12%   │
│     Policies at risk: 980            │ │  Hyderabad   ███         9%   │
│                          [View Map]  │ │  Pune        ██          7%   │
│                                      │ │  Others      ██          6%   │
│  🚗 Delhi — TRAFFIC_DISRUPTION      │ │                               │
│     Severity: MEDIUM                 │ └───────────────────────────────┘
│     Since: Thu 15 Jul 09:30          │
│     Workers affected: 380            │ ┌───────────────────────────────┐
│     Policies at risk: 290            │ │  Plan Distribution            │
│                          [View Map]  │ │                               │
│                                      │ │  Basic     ████████     32%   │
│  🌡️ Chennai — HEAT_DISRUPTION       │ │  Standard  ████████████ 51%   │
│     Severity: LOW                    │ │  Pro       █████        17%   │
│     Since: Fri 16 Jul 11:00          │ │                               │
│     Workers affected: 95             │ └───────────────────────────────┘
└──────────────────────────────────────┘
```

---

### 5c — Fraud Review Queue

```
┌──────────────────────────────────────────────────────────────────────┐
│  Fraud Review Queue  (12 pending)                                    │
│                                                                      │
│  [ Approve All Score < 0.80 ]   [ Export CSV ]   [ Refresh ]        │
│                                                                      │
│  Payout ID  Worker ID  City       Amount   Score  Reason            │
│  ─────────────────────────────────────────────────────────────────  │
│  P-98761    W-4421     Mumbai     ₹1,200   0.81   geo_clustering    │
│                                            [Approve]  [Reject]      │
│  P-98754    W-3301     Bengaluru  ₹2,500   0.88   new_account       │
│                                            [Approve]  [Reject]      │
│  P-98748    W-7821     Delhi      ₹1,200   0.76   repeated_cap      │
│                                            [Approve]  [Reject]      │
│  P-98731    W-2201     Chennai    ₹500     0.79   disruption_mismatch│
│                                            [Approve]  [Reject]      │
│                                                                      │
│  Showing 4 of 12  [ Load More ]                                     │
└──────────────────────────────────────────────────────────────────────┘
```

**Fraud reason codes displayed to admin:**

| Code | Meaning |
|---|---|
| `geo_clustering` | Unusually high number of claims from the same micro-zone simultaneously |
| `new_account` | Worker account is fewer than 7 days old |
| `repeated_cap` | Worker has hit the coverage cap 3 or more consecutive weeks |
| `disruption_mismatch` | Payout triggered but no disruption event logged for the worker's zone |
| `earnings_outlier` | Reported income is significantly lower than GPS delivery activity suggests |

---

### 5d — Weekly Earnings Comparison Chart

```
┌──────────────────────────────────────────────────────────────────────┐
│  Weekly Earnings Comparison — Platform Average (All Cities)          │
│                                                                      │
│  ₹7,000 ┤                                                            │
│  ₹6,500 ┤  ▓▓  ░░  ▓▓  ░░  ▓▓  ░░  ▓▓  ░░  ▓▓  ░░  ▓▓  ░░        │
│  ₹6,000 ┤  ▓▓  ░░  ▓▓  ░░  ▓▓  ░░  ▓▓  ░░  ▓▓  ░░  ▓▓  ░░        │
│  ₹5,500 ┤                                                            │
│  ₹5,000 ┤                    ░░        ░░                            │
│  ₹4,500 ┤                    ░░        ░░                            │
│  ₹4,000 ┤                                                            │
│         └────────────────────────────────────────────────────       │
│          W1   W2   W3   W4   W5   W6   W7   W8   W9   W10           │
│                                                                      │
│          ▓ Predicted Income    ░ Actual Income                       │
│                                                                      │
│  Weeks W4 and W6 show significant actual income drops                │
│  corresponding to monsoon disruption events.                         │
└──────────────────────────────────────────────────────────────────────┘
```

---

### 5e — Admin Actions Summary

| Action | Location | Description |
|---|---|---|
| Approve payout | Fraud Queue | Releases a held payout to the worker's UPI |
| Reject payout | Fraud Queue | Cancels the payout and flags the worker for review |
| Bulk approve | Fraud Queue header | Approves all queued payouts below a set anomaly score |
| Export payouts | Payouts tab | Downloads full payout ledger as CSV for actuarial review |
| View worker profile | Workers tab | Full history: earnings, policies, payouts, fraud flags |
| Trigger disruption (test) | Disruptions tab | Manually logs a disruption event for a city (testing only) |
| Download weekly report | Analytics tab | PDF summary of the week's KPIs, payout rates, and city breakdown |

---

*EarnGuard AI — UI Concept Document v3.0 | Phase-1 Hackathon Submission*
