# 🛡 FraudShield SMB

**AI-powered fraud detection built for small and medium businesses.**

SMBs lose an estimated **$42 billion annually** to fraud, yet enterprise detection tools price out 90% of small businesses. FraudShield SMB closes that gap — a real-time, ML-driven fraud scoring engine that deploys in minutes, not months.

> Score a transaction in **<50ms**. Block fraud before it clears. Save your clients thousands.

---

## The Problem

Small businesses are disproportionately targeted by fraud but lack the tools to fight it:

- **67% of SMBs** experienced fraud in 2024 (AFP Payments Fraud Survey)
- Average loss per incident: **$67,000** — enough to shutter a small business
- Enterprise solutions (Featurespace, Feedzai) start at **$100K+/year**
- Most SMBs rely on manual review or nothing at all

FraudShield SMB provides enterprise-grade detection at SMB-friendly pricing — starting at $49/month.

---

## Architecture

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
                  │ (PostgreSQL)│              │ (SNS/Twilio) │  │ Feed         │
                  └──────┬──────┘              └──────────────┘  └──────────────┘
                         │
              ┌──────────┴──────────┐
              │                     │
       ┌──────▼──────┐      ┌──────▼──────┐
       │ PM Dashboard│      │ SMB Dashboard│
       │ (Portfolio) │      │ (Client)     │
       └─────────────┘      └──────────────┘
```

### AWS Production Architecture

For production deployment, the system maps to AWS services:

| Component | AWS Service | Purpose |
|---|---|---|
| API Gateway | **API Gateway** + **Lambda** | Rate-limited REST API with auto-scaling |
| ML Scoring | **SageMaker** | Real-time inference endpoint for fraud model |
| Rules Engine | **Lambda** | Configurable business rules per client |
| Transaction DB | **DynamoDB** | Low-latency reads/writes at scale |
| Event Stream | **Kinesis** | Real-time transaction event pipeline |
| Alert Service | **SNS** + **Lambda** | Multi-channel notifications (email, SMS, webhook) |
| Cache | **ElastiCache (Redis)** | Score caching + rate limiting |
| Auth | **Cognito** | JWT-based auth with MFA support |
| Dashboard | **CloudFront** + **S3** | Global CDN for React frontend |

---

## Tech Stack

**Backend:** Node.js · Express · PostgreSQL · Redis · JWT  
**ML Engine:** 7-signal scoring model (amount anomaly, velocity, time-of-day, vendor risk, geo-mismatch, device fingerprint, category anomaly)  
**Frontend:** React · Vanilla JS · CSS (single-page app)  
**Infrastructure:** Docker · AWS (SageMaker, Lambda, DynamoDB, Kinesis, SNS, API Gateway)  
**Integrations:** Stripe (billing) · Twilio (SMS) · SendGrid (email) · Square/PayPal (webhooks)

---

## Key Features

### Real-Time ML Scoring Engine
Every transaction is evaluated across **7 fraud signal dimensions** in under 50ms:

| Signal | What It Detects | Weight |
|---|---|---|
| Amount Anomaly | Transactions deviating from client's historical average (z-score analysis) | 20% |
| Velocity Check | Unusual spikes in transaction frequency | 18% |
| Vendor Risk | First-time or high-risk payment vendors | 15% |
| Geo-location Mismatch | Transactions from unexpected locations | 15% |
| Time-of-Day Anomaly | Activity outside business hours | 12% |
| Device Fingerprint | Unrecognized devices initiating payments | 10% |
| Category Anomaly | Unusual transaction categories (e.g., refunds, gift cards) | 10% |

### Configurable Risk Thresholds
Each client gets tunable sensitivity:

| Profile | Block ≥ | Flag ≥ | Review ≥ |
|---|---|---|---|
| Conservative | 75 | 55 | 35 |
| Balanced (default) | 85 | 65 | 45 |
| Permissive | 92 | 78 | 55 |

### Multi-Tenant Architecture
- **PM Dashboard** — portfolio-level view across all SMB clients
- **Client Dashboard** — individual business view with live transaction feed
- **API Key Auth** — programmatic access for POS/payment integrations
- **JWT Auth** — dashboard login with role-based access (admin, PM, client)

### Instant Alerts
- Critical alerts → **Email + SMS** within 60 seconds
- High alerts → **Email** notification
- Auto-block → Transactions above threshold are halted automatically

---

## Performance Targets

| Metric | Target | Notes |
|---|---|---|
| Scoring latency | < 50ms | P95, with Redis cache |
| API throughput | 300 req/min per client | Configurable per plan |
| Uptime SLA | 99.9% | Production target |
| False positive rate | < 5% | Tunable via risk profiles |
| Detection rate | > 92% | Based on backtesting |

---

## API Reference

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
    "location": "Seattle, WA",
    "ip_address": "203.0.113.42"
  }'
```

**Response:**
```json
{
  "transactionId": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
  "score": 78,
  "status": "flagged",
  "signals": [
    {
      "name": "Amount Anomaly",
      "score": 65,
      "severity": "high",
      "explanation": "Amount $3500 is 2.3 standard deviations from average ($580)."
    },
    {
      "name": "Time-of-Day Anomaly",
      "score": 80,
      "severity": "high",
      "explanation": "Transaction at 2:00 — outside normal business hours."
    }
  ],
  "recommendation": "High risk detected. Alert sent to business owner.",
  "modelVersion": "1.0.0",
  "latencyMs": 34
}
```

### Other Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/v1/auth/register` | Create account |
| `POST` | `/api/v1/auth/login` | Get JWT tokens |
| `POST` | `/api/v1/score` | **Score a transaction** |
| `POST` | `/api/v1/score/batch` | Score up to 100 transactions |
| `GET` | `/api/v1/clients` | List SMB clients (PM/Admin) |
| `POST` | `/api/v1/clients` | Onboard new SMB client |
| `GET` | `/api/v1/transactions` | Query transaction history |
| `PATCH` | `/api/v1/transactions/:id/status` | Approve/reject flagged txns |
| `GET` | `/api/v1/alerts` | List alerts |
| `POST` | `/api/v1/alerts/:id/resolve` | Resolve an alert |
| `GET` | `/api/v1/dashboard/overview` | Portfolio analytics |
| `GET` | `/api/v1/dashboard/trends` | Transaction trends over time |
| `GET` | `/api/v1/billing/plans` | Subscription pricing |

Full interactive docs available at `/api/v1/docs` (Swagger UI).

---

## Run Locally

### Prerequisites

- **Node.js** ≥ 18
- **Docker** + **Docker Compose** (for PostgreSQL + Redis)
- **Git**

### Quick Start

```bash
# 1. Clone the repo
git clone https://github.com/adedejiayobami0-ux/fraudshieldsmb.git
cd fraudshieldsmb

# 2. Install dependencies
npm install

# 3. Start PostgreSQL + Redis
docker-compose up -d postgres redis

# 4. Configure environment
cp .env.example .env
# Edit .env with your settings (defaults work for local dev)

# 5. Run database migrations
node scripts/migrate.js

# 6. Seed demo data
node scripts/seed.js

# 7. Start the API server
npm run dev
```

The API will be running at `http://localhost:3001`. Open `http://localhost:3001/api/v1/docs` for interactive API documentation.

### Demo Credentials

| Role | Email | Password |
|---|---|---|
| Admin | `admin@fraudshieldsmb.com` | `demo1234` |
| Program Manager | `pm@fraudshieldsmb.com` | `demo1234` |

### Full Docker Setup (everything containerized)

```bash
docker-compose up --build
```

This starts the API, PostgreSQL, and Redis in containers. The API is available at `http://localhost:3001`.

### Run Tests

```bash
npm test
```

---

## Project Structure

```
fraudshieldsmb/
├── src/
│   ├── app.js                  # Express app with middleware
│   ├── server.js               # Entry point
│   ├── config/
│   │   ├── database.js         # PostgreSQL (Knex)
│   │   ├── redis.js            # Redis connection
│   │   └── swagger.js          # OpenAPI spec
│   ├── middleware/
│   │   ├── auth.js             # JWT + API key authentication
│   │   ├── errorHandler.js     # Global error handling
│   │   └── validate.js         # Joi request validation
│   ├── ml/
│   │   └── scoringEngine.js    # 🧠 Core fraud scoring engine
│   ├── routes/
│   │   ├── auth.js             # Register / login / refresh
│   │   ├── clients.js          # SMB client CRUD + onboarding
│   │   ├── scoring.js          # Transaction scoring (core API)
│   │   ├── transactions.js     # Transaction history + review
│   │   ├── alerts.js           # Alert management
│   │   ├── dashboard.js        # Portfolio analytics
│   │   ├── billing.js          # Subscription plans + usage
│   │   └── webhooks.js         # Payment processor webhooks
│   ├── services/
│   │   └── alertService.js     # Email + SMS notifications
│   └── utils/
│       └── logger.js           # Winston structured logging
├── scripts/
│   ├── migrate.js              # Database schema setup
│   └── seed.js                 # Demo data generator
├── index.html                  # Frontend dashboard (React SPA)
├── docker-compose.yml          # Local dev infrastructure
├── Dockerfile                  # Production container
├── .env.example                # Environment config template
└── package.json
```

---

## Roadmap

- [ ] **v1.1** — Trained ML model on SageMaker (replace rule-based scoring)
- [ ] **v1.2** — Square and Stripe webhook auto-scoring
- [ ] **v1.3** — Client self-serve onboarding portal
- [ ] **v1.4** — Batch file upload (CSV transaction import)
- [ ] **v2.0** — Multi-region deployment + SOC 2 compliance

---

## License

Proprietary. © 2025 A. Adedeji. All rights reserved.
