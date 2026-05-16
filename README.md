# FraudShield SMB 🛡

> **AI-powered fraud detection built for American small businesses.**
> Real-time transaction scoring, automatic blocking, and a personal dashboard — at a price small businesses can actually afford.

**Live:** https://www.fraudshieldsmb.com
**Status:** Free beta · Actively onboarding pilot customers

---

## The Problem

Every day, American small businesses lose money to fraud they never see coming:

- A customer pays with a stolen card. The business ships the order. The real cardholder disputes it — the business loses both the money and the product.
- A fraudster runs 50 tiny $1 test charges to check if a stolen card works, then makes a $3,000 purchase.
- A fake check clears the bank. The business delivers the service. Three days later it bounces. The money is gone.
- Chargebacks pile up until the payment processor raises fees or shuts the account down entirely.

Big companies have entire fraud teams and enterprise tools like Stripe Radar, Sift, and Kount. Small businesses — the 33 million businesses that make up the backbone of the American economy — are left with basic, generic processor tools that weren't designed for their transaction patterns, their industries, or their scale.

**FraudShield fills that gap.**

---

## What FraudShield Does

FraudShield is a real-time fraud detection API that scores every transaction before it is processed. It works as a lightweight layer on top of whatever payment tools a business already uses — Stripe, Square, PayPal, ACH, Zelle, checks, wires.

Each transaction is scored 0–100 in under 100 milliseconds:

| Score | Status | Action |
|---|---|---|
| 0–49 | Clear | Transaction proceeds normally |
| 50–69 | Review | Queued for manual review |
| 70–84 | Flagged | Owner alerted instantly |
| 85–99 | Blocked | Auto-blocked, owner notified |

Business owners see everything in their personal dashboard — real transaction history, fraud signals, money saved, and their API key ready to copy and use.

---

## The 6 Fraud Patterns We Detect

### 1. Card Testing
Fraudsters run tiny $1–$5 charges to check if a stolen card works before making a large purchase. FraudShield detects the pattern — frequency, amount progression, timing — and blocks it before it escalates.

### 2. Check and ACH Fraud
Fake check deposits and unauthorized ACH pulls that look legitimate but bounce 3–5 days after the business has already delivered goods or services. FraudShield flags these based on account history, amount patterns, and timing anomalies.

### 3. Chargeback Abuse
Some customers buy, receive the product or service, then file a dispute with their bank claiming they never received it. FraudShield identifies repeat chargeback patterns and flags these customers before they transact again.

### 4. Identity Mismatch
Billing address does not match shipping address. The name on the card does not match the account. The device is in a different state than the billing address. FraudShield cross-references these signals in real time.

### 5. Velocity Attacks
Suddenly 30 orders in 10 minutes from new accounts. The same card tried 15 times with different amounts. Unusual spikes in transaction volume that signal an automated attack or account takeover.

### 6. Off-Hours Activity
Real customers rarely make large purchases at 3am. FraudShield builds a baseline of each business's normal transaction patterns and flags anything that looks statistically out of place — time of day, day of week, amount range.

---

## How It Works — Customer Journey

```
1. Sign Up (auth.html)
   Sign up with email and password
   Enter business name, website or phone number
   Select payment processors used (Stripe, Square, PayPal, etc)
   Choose monthly transaction volume
   Set risk tolerance (conservative, balanced, or permissive)

2. Get API Key (dashboard.html)
   Unique key auto-generated on first login
   Stored securely in Supabase with RLS
   Show, hide, copy, and regenerate in the dashboard

3. Integrate (one API call)
   POST transaction details to the /score endpoint
   Get back score, status, and signals in under 100ms
   Block or flag based on the response

4. Monitor (dashboard.html)
   See every transaction scored in real time
   Real fraud signals with plain-English explanations
   Stats: total scored, flagged, blocked, money saved
```

---

## Technical Architecture

### Frontend
Pure vanilla HTML, CSS, and JavaScript. No React, no build step, no framework. Deliberate choice for maximum compatibility, zero loading time, and simple GitHub Pages deployment.

| File | Purpose |
|---|---|
| `index.html` | Redirects visitors to the landing page |
| `landing.html` | Public homepage with live demo, pricing, FAQ, and CTAs |
| `auth.html` | 3-step sign up with business verification. Also handles sign in and magic link |
| `dashboard.html` | Authenticated customer dashboard with real Supabase data |

### Backend
Supabase handles everything server-side:

- **Auth** — Email/password, email confirmation, magic link, session management
- **Database** — PostgreSQL with Row Level Security so customers only see their own data
- **Edge Functions** — Deno-based serverless functions on Supabase's global edge network

### Hosting
GitHub Pages serves the static frontend. Zero infrastructure cost during beta.

---

## Database Schema

All tables have Row Level Security enabled.

### api_keys
```sql
id          uuid primary key default gen_random_uuid()
user_id     uuid references auth.users(id)
key         text not null
active      boolean default true
created_at  timestamptz default now()
```

### businesses
```sql
id              uuid primary key default gen_random_uuid()
user_id         uuid references auth.users(id)
business_name   text
industry        text
state           text
employees       text
annual_rev      text
processor       text
risk_tolerance  text default 'balanced'
block_threshold int default 82
alert_email     text
created_at      timestamptz default now()
```

### transactions
```sql
id             uuid primary key
user_id        uuid references auth.users(id)
amount         numeric
vendor         text
category       text
score          int2
status         text
signals        text
business_name  text
created_at     timestamptz default now()
```

---

## API Reference

### POST /score

**Endpoint**
```
https://nivqhnqquwszwtsgupgx.supabase.co/functions/v1/score
```

**Headers**
```
Authorization: Bearer fs_live_your_key
Content-Type: application/json
```

**Request**
```json
{
  "amount": 3500,
  "vendor": "Stripe",
  "category": "Vendor Payment",
  "hour": 14,
  "business_name": "Pinewood Bakery"
}
```

**Response**
```json
{
  "score": 77,
  "status": "flagged",
  "signals": ["Card testing", "Velocity anomaly"],
  "action": "FLAG_AND_ALERT",
  "latency_ms": 94,
  "request_id": "550e8400-e29b-41d4-a716-446655440000",
  "model": "fraudshield-smb-v1"
}
```

**Status values:** `clear` · `review` · `flagged` · `blocked`

**Error responses**
```json
{ "error": "Missing API key" }                 // 401
{ "error": "Invalid or inactive API key" }     // 403
{ "error": "Missing required fields" }         // 400
{ "error": "Internal server error" }           // 500
```

---

## Fraud Scoring Logic

| Signal | Max Points | Trigger |
|---|---|---|
| Amount threshold | +26 | Amount over $5,000 |
| Time-of-day anomaly | +24 | Before 5am or after 11pm |
| High-risk vendor | +21 | Wire Transfer, Zelle, Venmo Business |
| Chargeback pattern | +22 | Category = Chargeback |
| Wire transfer | +18 | Category = Wire |
| Card testing (ML) | +20 | Probabilistic — 10% base rate |
| Geo-location mismatch (ML) | +18 | Probabilistic — 7% base rate |
| Device fingerprint (ML) | +16 | Probabilistic — 8% base rate |
| Refund pattern | +13 | Category = Refund |
| Check fraud (ML) | +14 | Probabilistic — 5% base rate |
| Random variance | +0–9 | Realistic noise |

Scores are capped at 99 and floored at 1. Probabilistic ML signals will be replaced with real model inference as transaction volume grows.

---

## Pricing

| Plan | Price | Transactions | Status |
|---|---|---|---|
| Beta | $0/mo | Unlimited | Active now |
| Starter | $99/mo | 5,000/mo | Coming after beta |
| Growth | $299/mo | 25,000/mo | Coming after beta |
| Business | $799/mo | Unlimited | Coming after beta |

Beta users get full access to everything and will receive advance notice before any pricing changes. Early customers will be offered a discounted rate when paid plans launch.

---

## Project Status

### Completed
- Landing page with live interactive demo, pricing, and FAQ
- 3-step sign up with business verification questions
- Sign in with email/password and magic link
- Email confirmation with correct redirect
- Business onboarding wizard (saves to Supabase)
- Automatic API key generation per user
- Supabase auth, database, and Edge Function integration
- Real-time scoring endpoint deployed and live
- Transaction logging to Supabase on every API call
- Customer dashboard showing real transaction data
- Dark and light mode across all pages
- Row Level Security on all tables

### Next
- Email and SMS alerts when fraud is detected
- Stripe payments integration
- Admin dashboard for founder
- Custom domain
- /explain endpoint with human-readable fraud reasoning
- Webhook delivery for real-time alerts
- Usage tracking and monthly limits per plan
- Real ML models trained on actual transaction data

---

## Infrastructure Roadmap

### Current Stack (Beta)
Everything runs on Supabase and Vercel. Zero infrastructure overhead. Free until meaningful revenue.

| Layer | Technology | Cost |
|---|---|---|
| Frontend | Vercel | Free |
| Auth | Supabase Auth | Free |
| Database | Supabase PostgreSQL | Free |
| API | Supabase Edge Functions | Free |
| Domain | Namecheap + Vercel | ~$12/year |

---

### Phase 2 — First 50 Paying Customers
When monthly revenue exceeds $2,000 MRR, introduce:

**AWS SES (Simple Email Service)**
Replace Formspree with real transactional email. Send instant fraud alerts, welcome emails, and monthly fraud reports directly from `alerts@fraudshieldsmb.com`. Cost: ~$1/month at low volume.

**Supabase Pro**
Upgrade Supabase plan for higher database limits, daily backups, and better performance. Cost: $25/month.

---

### Phase 3 — Real ML Model (500+ Labeled Transactions)
Replace the current rule-based scoring engine with a real trained model.

**AWS SageMaker**
Train an XGBoost gradient boosting model on labeled transaction data collected from the dashboard fraud feedback buttons. Deploy as a live inference endpoint. The Edge Function calls SageMaker instead of running rules locally.

Why XGBoost: It is the industry standard for fraud detection on tabular data. Used by Stripe, PayPal, and most major financial institutions. Works extremely well with small datasets compared to deep learning.

Training data required: 500+ transactions labeled as fraud or legitimate via the dashboard.

```
Collect labels → Export from Supabase → Train on SageMaker → 
Deploy endpoint → Edge Function calls model → Real ML predictions
```

Cost: ~$50/month for a `ml.t2.medium` inference endpoint.

---

### Phase 4 — Scale (200+ Customers)

**AWS RDS**
Migrate from Supabase PostgreSQL to a dedicated RDS instance for higher throughput, custom configurations, and read replicas. Needed when processing millions of transactions per month.

**AWS Lambda**
Replace Supabase Edge Functions with Lambda for more control over runtime, memory, and execution time. Lambda can handle millions of requests per second with sub-10ms cold starts.

**AWS CloudFront**
Add a CDN layer in front of Vercel for customers outside the US. Reduces latency for international expansion.

**AWS S3**
Store ML model artifacts, training datasets, and customer fraud report PDFs.

---

### Phase 5 — Enterprise

**SOC 2 Type II Certification**
Required for enterprise customers and larger financial institutions. Achieved through AWS's compliance infrastructure and third-party auditors.

**VPC + Private Subnets**
Isolate the database and ML inference endpoints inside a private network. Required for enterprise security requirements.

**AWS WAF (Web Application Firewall)**
Protect the scoring API from DDoS attacks, rate abuse, and malicious traffic at the infrastructure level.

---

### Cost Projection

| Stage | Monthly Revenue | Infrastructure Cost | Margin |
|---|---|---|---|
| Beta | $0 | $0 | — |
| 10 customers | $990 | $0 | 100% |
| 50 customers | $4,950 | ~$76 | 98.5% |
| 100 customers | $9,900 | ~$200 | 98% |
| 500 customers | $49,500 | ~$800 | 98.4% |

Infrastructure stays under 2% of revenue at every stage. The model is highly capital efficient.

```bash
git clone https://github.com/adedejiayobami0-ux/fraudshieldsmb
cd fraudshieldsmb
```

Add Supabase credentials to `auth.html` and `dashboard.html`:
```js
const SUPABASE_URL  = "https://your-project.supabase.co";
const SUPABASE_ANON = "eyJ...your-anon-key";
```

Open `landing.html` directly in your browser or run a local server:
```bash
npx serve .
```

---

## Deploying the Edge Function

```bash
brew install supabase/tap/supabase
supabase login
supabase link --project-ref your-project-ref
supabase functions deploy score
```

Source lives at `supabase/functions/score/index.ts`

---

## Business Context

**Market:** 33 million small businesses in the United States

**Target customer:** SMBs processing $100K–$10M annually on Stripe, Square, or PayPal

**GTM strategy:** Start with free beta to get real transaction data and case studies. Convert to paid at $99–$299/mo. Target Stripe and Square partner ecosystems for distribution.

**Revenue target:** 3–5 paying Growth clients at $299/mo = $900–$1,500 MRR as initial milestone. First enterprise contract changes the trajectory entirely.

**Competitive moat:** Only fraud API built specifically for SMB transaction patterns. Not adapted from enterprise tools. Explains why transactions are flagged in plain English. Works across all payment rails simultaneously — not just one processor.

**Key objection:** "My payment processor already handles fraud."

**Rebuttal:** Stripe Radar protects Stripe, not the business. It only sees one payment rail. It doesn't explain flagged transactions. It cannot be tuned to a specific business's patterns. It doesn't catch fake checks, ACH fraud, or cross-channel velocity attacks. FraudShield does all of this.

---

## Looking for a Technical Co-Founder

The product is live. The architecture is defined. The first customers are onboarding. What we need next is someone to own the backend, infrastructure, and production hardening.

**You would own:**
- Backend architecture and API reliability
- Supabase Edge Functions and database performance
- Production deployment and monitoring
- Moving from probabilistic ML to real trained models

**What's on the table:**
- 50/50 equity with 4-year vesting and 1-year cliff
- Role separation: founder owns product, design, and GTM — cofounder owns backend and infrastructure
- Co-founder agreement drafted, pending legal review

If you have shipped production APIs, are comfortable with Supabase and TypeScript, and are excited about fintech infrastructure, reach out.

---

## About

Built by **Adedeji Ayobami** — product strategy, UI/UX design, fraud detection research, GTM, and frontend development.

---

## License

Private repository. All rights reserved.
