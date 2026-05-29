# FinTrack — Feature Specification

**Version:** 0.1.0-draft  
**Last Updated:** 2025  
**Status:** Pre-implementation

---

## 1. Overview

FinTrack is a personal finance tracking application that connects to users' bank accounts via the Plaid API, automatically categorizes transactions using OpenAI GPT-4o, and delivers AI-powered insights to help users improve their financial health.

### Problem Statement

Most people lack a clear, real-time picture of their finances. Manual spreadsheets are tedious and quickly fall out of date. Existing apps (Mint, YNAB) are either shutting down, expensive, or don't leverage modern AI for intelligent categorization and personalized advice.

### Goals

1. Reduce friction to < 5 minutes to get a complete financial overview after first setup
2. Categorization accuracy > 85% without user correction
3. AI insights users describe as "relevant" and "actionable" (qualitative bar)
4. Zero data breaches — all sensitive financial data encrypted at rest and in transit

---

## 2. Functional Requirements

### 2.1 Authentication

- [ ] Users can register with email + password
- [ ] Passwords hashed with bcrypt (cost factor ≥ 12)
- [ ] JWT access tokens (15-minute expiry)
- [ ] Refresh tokens (7-day expiry, httpOnly cookie, rotation on use)
- [ ] Refresh token replay attack protection (token families)
- [ ] Users can log out (invalidates refresh token family)

### 2.2 Bank Account Connectivity (Plaid)

- [ ] Users can link bank accounts via Plaid Link OAuth flow
- [ ] Support Plaid Sandbox, Development, and Production environments
- [ ] Linked accounts display name, type (checking/savings/credit), and current balance
- [ ] Users can link multiple institutions
- [ ] Users can unlink an account (revokes Plaid access, soft-deletes local data)
- [ ] Transaction sync triggered on: initial link, Plaid webhook, manual refresh
- [ ] Transactions synced using Plaid `/transactions/sync` (cursor-based, handles additions/modifications/removals)
- [ ] Plaid access tokens stored AES-256 encrypted in PostgreSQL

### 2.3 Transaction Categorization (AI)

- [ ] Every new transaction is categorized automatically within 30 seconds of sync
- [ ] GPT-4o receives: merchant name, amount, Plaid's raw category, account type
- [ ] Response is a structured JSON object: `{ category: string, confidence: number, reasoning: string }`
- [ ] Transactions batch-sent to GPT-4o (max 50 per call) to minimize API costs
- [ ] 20 standard categories: Food & Dining, Transport, Housing & Rent, Utilities, Health & Medical, Entertainment, Shopping, Travel, Education, Personal Care, Pets, Gifts & Donations, Business, Investments, Income, Transfers, Fees & Charges, Subscriptions, Insurance, Other
- [ ] Users can override AI category (stored as manual override, not re-classified)
- [ ] User overrides stored as few-shot examples to improve future classification for that user
- [ ] Transactions are searchable by: date range, category, account, merchant name, amount range

### 2.4 Budget Tracking

- [ ] Users can create a budget per category per month (e.g., Food: $500/month)
- [ ] Budget progress calculated in real-time as transactions sync
- [ ] Alert sent (in-app notification) when budget reaches 80% and 100%
- [ ] Budgets support: monthly (default), weekly, custom date range
- [ ] Dashboard shows all budgets with: spent, remaining, % used, status (on-track/warning/over)
- [ ] Historical budget performance (past months) accessible per budget

### 2.5 Spending Trends & Analytics

- [ ] Monthly spending breakdown by category (last 12 months default)
- [ ] Month-over-month % change per category with trend indicators (up/down/flat)
- [ ] Net worth tracker: sum of all asset accounts minus liability accounts over time
- [ ] Average monthly spend per category (rolling 3-month average)
- [ ] Top merchants by spend (last 30 days)
- [ ] Anomaly detection: flag any category that spikes > 50% vs. rolling average

### 2.6 AI Insights & Recommendations

- [ ] Insights generated on demand and cached for 24 hours
- [ ] Insights refreshed automatically when > $100 of new transactions sync
- [ ] 5 insight types:
  - `spending_spike` — "Your dining spend is 2x your 3-month average this month"
  - `savings_opportunity` — "You spent $89 on 4 different streaming services. Consider consolidating."
  - `bill_prediction` — "Your electricity bill (~$120) is due in ~5 days based on past patterns"
  - `budget_warning` — "At this rate, you'll exceed your Travel budget by $200 this month"
  - `positive_trend` — "You've reduced Food & Dining spend by 18% over the last 3 months. Great work!"
- [ ] Each insight includes: type, headline, detail, optional action CTA, priority score
- [ ] Insights sorted by priority score (higher = shown first)
- [ ] Users can dismiss insights (stored, not re-shown for 30 days)

---

## 3. Non-Functional Requirements

- [ ] API response time < 200ms (p95) for all read endpoints (excluding AI generation)
- [ ] AI categorization < 30 seconds for up to 50 transactions
- [ ] AI insights generation < 15 seconds
- [ ] 99.5% uptime target
- [ ] Supports up to 10,000 transactions per user without pagination degradation
- [ ] All PII and financial data encrypted at rest (AES-256)
- [ ] HTTPS-only in production (HSTS)
- [ ] OWASP Top 10 mitigations in place
- [ ] Frontend bundle < 500 KB gzipped initial load

---

## 4. Data Model

### Users
```
users
  id             UUID PK
  email          TEXT UNIQUE NOT NULL
  password_hash  TEXT NOT NULL
  created_at     TIMESTAMPTZ DEFAULT NOW()
  updated_at     TIMESTAMPTZ
```

### Plaid Items (linked institutions)
```
plaid_items
  id                UUID PK
  user_id           UUID FK → users.id
  plaid_item_id     TEXT UNIQUE NOT NULL
  access_token_enc  TEXT NOT NULL          -- AES-256 encrypted
  institution_name  TEXT
  institution_id    TEXT
  cursor            TEXT                   -- Plaid sync cursor
  status            TEXT                   -- active | error | disconnected
  last_synced_at    TIMESTAMPTZ
  created_at        TIMESTAMPTZ DEFAULT NOW()
```

### Accounts
```
accounts
  id               UUID PK
  user_id          UUID FK → users.id
  plaid_item_id    UUID FK → plaid_items.id
  plaid_account_id TEXT UNIQUE NOT NULL
  name             TEXT NOT NULL
  official_name    TEXT
  type             TEXT                   -- depository | credit | loan | investment
  subtype          TEXT                   -- checking | savings | credit card | etc.
  current_balance  NUMERIC(15,2)
  available_balance NUMERIC(15,2)
  currency         TEXT DEFAULT 'USD'
  is_active        BOOLEAN DEFAULT TRUE
  updated_at       TIMESTAMPTZ
```

### Transactions
```
transactions
  id                  UUID PK
  user_id             UUID FK → users.id
  account_id          UUID FK → accounts.id
  plaid_transaction_id TEXT UNIQUE NOT NULL
  amount              NUMERIC(15,2) NOT NULL  -- positive = debit, negative = credit
  merchant_name       TEXT
  description         TEXT NOT NULL
  date                DATE NOT NULL
  category            TEXT                    -- AI-assigned category name
  category_confidence FLOAT                   -- 0.0–1.0
  category_override   TEXT                    -- user manual override (nullable)
  is_pending          BOOLEAN DEFAULT FALSE
  plaid_category      TEXT[]                  -- Plaid's raw category array
  metadata            JSONB                   -- raw Plaid response for reference
  created_at          TIMESTAMPTZ DEFAULT NOW()
  updated_at          TIMESTAMPTZ
```

### Budgets
```
budgets
  id           UUID PK
  user_id      UUID FK → users.id
  category     TEXT NOT NULL
  amount_limit NUMERIC(15,2) NOT NULL
  period       TEXT DEFAULT 'monthly'      -- monthly | weekly | custom
  period_start DATE                        -- for custom periods
  period_end   DATE
  is_active    BOOLEAN DEFAULT TRUE
  created_at   TIMESTAMPTZ DEFAULT NOW()
  UNIQUE(user_id, category, period)
```

### Insights
```
insights
  id           UUID PK
  user_id      UUID FK → users.id
  type         TEXT NOT NULL
  headline     TEXT NOT NULL
  detail       TEXT NOT NULL
  action_cta   TEXT
  priority     INTEGER DEFAULT 50
  is_dismissed BOOLEAN DEFAULT FALSE
  dismissed_at TIMESTAMPTZ
  generated_at TIMESTAMPTZ DEFAULT NOW()
  expires_at   TIMESTAMPTZ
```

### Refresh Tokens
```
refresh_tokens
  id         UUID PK
  user_id    UUID FK → users.id
  token_hash TEXT UNIQUE NOT NULL
  family_id  UUID NOT NULL               -- for replay attack detection
  is_used    BOOLEAN DEFAULT FALSE
  expires_at TIMESTAMPTZ NOT NULL
  created_at TIMESTAMPTZ DEFAULT NOW()
```

---

## 5. API Design

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/api/auth/register` | — | Register new user |
| POST | `/api/auth/login` | — | Login, get tokens |
| POST | `/api/auth/refresh` | cookie | Rotate refresh token |
| POST | `/api/auth/logout` | JWT | Invalidate refresh token |
| POST | `/api/plaid/link-token` | JWT | Create Plaid Link token |
| POST | `/api/plaid/exchange-token` | JWT | Exchange public token |
| POST | `/api/plaid/sync` | JWT | Manual sync trigger |
| POST | `/api/plaid/webhook` | Plaid sig | Handle Plaid webhooks |
| GET | `/api/accounts` | JWT | List user's linked accounts |
| DELETE | `/api/accounts/:id` | JWT | Unlink account |
| GET | `/api/transactions` | JWT | List transactions (paginated, filterable) |
| PATCH | `/api/transactions/:id/category` | JWT | Override transaction category |
| GET | `/api/budgets` | JWT | List budgets with progress |
| POST | `/api/budgets` | JWT | Create budget |
| PATCH | `/api/budgets/:id` | JWT | Update budget |
| DELETE | `/api/budgets/:id` | JWT | Delete budget |
| GET | `/api/analytics/monthly` | JWT | Monthly spending by category |
| GET | `/api/analytics/trends` | JWT | MoM trend data |
| GET | `/api/analytics/net-worth` | JWT | Net worth history |
| GET | `/api/insights` | JWT | Get AI insights (cached) |
| POST | `/api/insights/refresh` | JWT | Force regenerate insights |
| PATCH | `/api/insights/:id/dismiss` | JWT | Dismiss an insight |
| GET | `/api/health` | — | Health check |

---

## 6. Test Plan

### Unit Tests (Vitest)
- `CategoryService.classify()` — mock GPT-4o, test prompt construction, JSON parsing, batch logic
- `InsightService.generate()` — mock GPT-4o, test context building, output parsing, all 5 insight types
- `BudgetEngine.calculateProgress()` — various spending scenarios, edge cases (no transactions, over limit)
- `PlaidService` — mock Plaid client, test all sync paths, cursor pagination, deduplication
- JWT helpers — sign, verify, refresh, expiry, tamper detection
- Zod schemas — all request body validation schemas

### Integration Tests (Supertest)
- Full auth flow: register → login → refresh → logout
- Plaid link flow: link-token → exchange → accounts appear
- Transaction sync: mock Plaid webhook → transactions categorized → budget updated
- Budget CRUD + progress calculation
- Analytics endpoints: verify correct aggregations against seed data
- Insight generation: mock GPT-4o → insights persisted → cached on second call

### E2E Tests (Playwright)
- Happy path: register → link Plaid sandbox → view dashboard → set budget → view insights
- Category override: click transaction → change category → verify persisted
- Budget alert: set low budget → sync transactions over limit → alert appears

### Edge Cases
- Duplicate transaction sync (same `plaid_transaction_id`)
- Plaid access token revoked (institution disconnected)
- GPT-4o API timeout during categorization
- User with 0 transactions (empty states)
- Budget for category with no transactions (0% progress)
- Refresh token replay (second use of same token)
- Large transaction volume (10,000+ transactions, paginated correctly)

---

## 7. Open Questions

- [ ] Should we support multiple currencies? (v0.1: USD only, defer)
- [ ] How do we handle Plaid's "pending" transactions in budgets? (count as spent or exclude?)
- [ ] GPT-4o rate limits — implement queue/backoff or use batch API?
- [ ] Should insights be user-personalized (per-user few-shot) or generic?
- [ ] What is the data retention policy for dismissed insights?
- [ ] Do we need GDPR/CCPA compliance features for v0.1? (data export, deletion)
