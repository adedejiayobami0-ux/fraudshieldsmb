🛡 FraudShield SMB
AI-powered fraud detection designed for small and medium businesses

SMBs lose an estimated **$42 billion annually** to fraud, yet enterprise detection tools price out 90% of small businesses. FraudShield SMB is built to close that gap — a real-time, ML-driven fraud scoring platform that deploys in minutes, not months.

Status: Interactive prototype · Architecture designed · Backend in development


 The Problem

Small businesses are disproportionately targeted by fraud but lack the tools to fight it:

- **67% of SMBs** experienced fraud in 2024 (AFP Payments Fraud Survey)
- Average loss per incident: **$67,000** — enough to shutter a small business
- Enterprise solutions (Featurespace, Feedzai) start at **$100K+/year**
- Most SMBs rely on manual review or nothing at all

FraudShield SMB aims to provide enterprise-grade detection at SMB-friendly pricing — starting at $49/month.

---

 Live Prototype

The interactive prototype demonstrates the full product vision:

🔗 [Launch Prototype →](https://adedejiayobami0-ux.github.io/fraudshieldsmb/)

### What the prototype includes

- **PM Dashboard** — Portfolio-level view across 6 simulated SMB clients with risk scoring, alert counts, and fraud prevention metrics
- **Client View** — Individual business dashboard with a live transaction feed that updates every 5 seconds, filterable by status (flagged, review, clear)
- **ML Scoring Modal** — Interactive fraud scoring engine where you can input transaction parameters (amount, vendor, category, time of day) and see real-time risk analysis with detected fraud signals
Onboarding Flow — 5-step client onboarding wizard with simulated AWS deployment sequence
Architecture Diagram Interactive system architecture showing the cloud-native detection pipeline

Scoring engine (prototype)

The prototype runs a client-side scoring model that evaluates transactions across 6 fraud signal dimensions:

| Signal | What It Detects |
|---|---|
| Amount Threshold | Unusually large transaction amounts |
| Velocity Anomaly | Abnormal transaction frequency patterns |
| Time-of-Day Anomaly | Activity outside business hours (10 PM – 5 AM) |
| Unusual Vendor | High-risk payment vendors (Wire Transfer, Zelle, Venmo) |
| Geo-location Mismatch | Transactions from unexpected locations |
| Device Fingerprint | Unrecognized devices initiating payments |

---

## Planned Architecture

The production system is designed around AWS managed services for scalability and low operational overhead:

```
┌──────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│              │     │                  │     │   DETECTION ENGINE   │
│   SMB Client │────▶│   API Gateway    │────▶│                     │
│  Browser/POS │     │  (Rate Limited)  │     │  ┌───────────────┐  │
│              │     │                  │     │  │ ML Scoring     │  │
└──────────────┘     └──────┬───────────┘     │  │ Engine         │  │
                            │                 │  │ (7 signals)    │  │
                     ┌──────▼───────────┐     │  └───────┬───────┘  │
                     │   Auth Layer     │     │          │          │
                     │  JWT + API Keys  │     │  ┌───────▼───────┐  │
                     └──────────────────┘     │  │ Rules Engine   │  │
                                              │  │ (configurable) │  │
                                              │  └───────┬───────┘  │
                                              └──────────┼──────────┘
                                                         │
                         ┌───────────────────────────────┼───────────────┐
                         │                               │               │
                  ┌──────▼──────┐              ┌─────────▼────┐  ┌───────▼──────┐
                  │ Transaction │              │ Alert Service│  │ Event Stream │
                  │ Database    │              │ Email + SMS  │  │ Real-time    │
                  │ (DynamoDB)  │              │ (SNS/Twilio) │  │ (Kinesis)    │
                  └──────┬──────┘              └──────────────┘  └──────────────┘
                         │
              ┌──────────┴──────────┐
              │                     │
       ┌──────▼──────┐      ┌──────▼──────┐
       │ PM Dashboard│      │ SMB Dashboard│
       │ (Portfolio) │      │ (Client)     │
       └─────────────┘      └──────────────┘
```

### AWS Service Mapping

| Component | AWS Service | Why |
|---|---|---|
| API Gateway | **API Gateway** + **Lambda** | Auto-scaling, pay-per-request, built-in throttling |
| ML Scoring | **SageMaker** | Managed ML inference with real-time endpoints |
| Rules Engine | **Lambda** | Configurable business rules per client |
| Transaction DB | **DynamoDB** | Single-digit ms reads at any scale |
| Event Stream | **Kinesis** | Real-time transaction pipeline for live dashboards |
| Alert Service | **SNS** + **Lambda** | Multi-channel notifications (email, SMS, webhook) |
| Cache | **ElastiCache (Redis)** | Score caching + session management |
| Auth | **Cognito** | JWT auth with MFA, no custom auth server needed |
| Dashboard | **CloudFront** + **S3** | Global CDN for the React frontend |

---

## Tech Stack

**Prototype (current):** React · Vanilla JS · CSS · Client-side scoring engine  
**Backend (planned):** Node.js · Express · PostgreSQL · Redis · JWT  
**ML Engine (planned):** 7-signal weighted scoring → SageMaker trained model  
**Infrastructure (planned):** Docker · AWS (SageMaker, Lambda, DynamoDB, Kinesis, SNS, API Gateway)  
**Integrations (planned):** Stripe (billing) · Twilio (SMS) · SendGrid (email) · Square/PayPal (webhooks)

---

## Planned Features

### Configurable Risk Thresholds (per client)

| Profile | Auto-Block ≥ | Flag ≥ | Review ≥ |
|---|---|---|---|
| Conservative | 75 | 55 | 35 |
| Balanced (default) | 85 | 65 | 45 |
| Permissive | 92 | 78 | 55 |

### Multi-Tenant Access

- **PM Dashboard** — portfolio view across all SMB clients
- **Client Dashboard** — individual business view with live transaction feed
- **API Key Auth** — programmatic access for POS/payment system integrations
- **JWT Auth** — dashboard login with role-based access (admin, PM, client)

### Instant Alerts

- Critical alerts → Email + SMS within 60 seconds
- High alerts → Email notification
- Auto-block → Transactions above threshold halted automatically

---

## Performance Targets

| Metric | Target |
|---|---|
| Scoring latency | < 50ms (P95) |
| API throughput | 300 req/min per client |
| Uptime | 99.9% |
| False positive rate | < 5% |
| Detection rate | > 92% |

---

## API Design (Planned)

### Score a Transaction

```bash
curl -X POST https://api.fraudshieldsmb.com/api/v1/score \
  -H "X-API-Key: fsk_your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 3500,
    "vendor": "Wire Transfer",
    "category": "Vendor Payment",
    "timestamp": "2025-03-15T02:30:00Z",
    "location": "Seattle, WA"
  }'
```

**Expected Response:**
```json
{
  "transactionId": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
  "score": 78,
  "status": "flagged",
  "signals": [
    {
      "name": "Amount Anomaly",
      "severity": "high",
      "explanation": "Amount $3500 is 2.3 std deviations from client average ($580)."
    },
    {
      "name": "Time-of-Day Anomaly",
      "severity": "high",
      "explanation": "Transaction at 2:00 AM — outside normal business hours."
    }
  ],
  "recommendation": "High risk detected. Alert sent to business owner.",
  "latencyMs": 34
}
```

### Planned Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/v1/score` | Score a transaction for fraud risk |
| `POST` | `/api/v1/score/batch` | Score up to 100 transactions |
| `POST` | `/api/v1/auth/register` | Create account |
| `POST` | `/api/v1/auth/login` | Get JWT tokens |
| `GET` | `/api/v1/clients` | List SMB clients |
| `POST` | `/api/v1/clients` | Onboard new SMB client |
| `GET` | `/api/v1/transactions` | Query transaction history |
| `GET` | `/api/v1/alerts` | List alerts |
| `GET` | `/api/v1/dashboard/overview` | Portfolio analytics |

---

## Run the Prototype Locally

```bash
# Clone the repo
git clone https://github.com/adedejiayobami0-ux/fraudshieldsmb.git
cd fraudshieldsmb

# Open in browser (no build step needed)
open index.html
```

The prototype is a single HTML file with no dependencies — just open it in any modern browser.

---

## Roadmap

- [x] Interactive prototype with simulated ML scoring
- [x] PM dashboard with portfolio management
- [x] Client onboarding wizard
- [x] Architecture design and AWS service mapping
- [ ] Backend API (Node.js + Express + PostgreSQL)
- [ ] Production ML scoring engine (7-signal model)
- [ ] JWT + API key authentication
- [ ] Stripe billing integration
- [ ] Twilio/SendGrid alert notifications
- [ ] Square and PayPal webhook integrations
- [ ] SageMaker model training on real fraud data
- [ ] SOC 2 compliance

---

## Author

**A. Adedeji**  
Built as a product management case study for AI-powered fraud detection in the SMB market.

---

## License

Proprietary. © 2025 A. Adedeji. All rights reserved.
