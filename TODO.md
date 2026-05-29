# FinTrack — Task Breakdown

## How to Use This File

Workflow per task:
1. **Write tests FIRST** (red phase — tests must fail before you implement)
2. **Implement until tests pass** (green phase)
3. **Review diff manually** — no unreviewed code ships
4. **Commit with descriptive message** referencing the task
5. **Update CLAUDE.md / AGENTS.md** if you discovered anything non-obvious (compound loop)

Evidence gate between phases: all tests must pass (`npm test`) and a human must review before moving to the next phase.

---

## Phase 0: Foundation ⬜

- [ ] Initialize Node.js monorepo with npm workspaces (`api`, `web`, `shared`)
- [ ] Configure TypeScript (`tsconfig.json`) for all three packages with strict mode
- [ ] Set up ESLint + Prettier with TypeScript rules
- [ ] Install and configure Vitest for unit/integration tests
- [ ] Write smoke test: `true === true` — confirm test runner works
- [ ] Set up PostgreSQL locally with Docker Compose (`docker-compose.yml`)
- [ ] Install and configure Drizzle ORM; write first migration (users table)
- [ ] Configure GitHub Actions CI: lint → type-check → test on every push
- [ ] Set up Vite for React frontend with TailwindCSS
- [ ] Create `.env.example` with all required env var names (no values)
- [ ] Review all AI config files (CLAUDE.md, AGENTS.md, copilot-instructions.md)

**Gate:** CI green, smoke test passes, DB migrations run cleanly.

---

## Phase 1: Auth & User Management ⬜

- [ ] Write tests: user registration (happy path, duplicate email, weak password)
- [ ] Implement `POST /api/auth/register` — bcrypt password hash, return JWT
- [ ] Write tests: user login (valid creds, invalid creds, locked account)
- [ ] Implement `POST /api/auth/login` — validate creds, issue access + refresh tokens
- [ ] Write tests: token refresh (valid refresh, expired refresh, replay attack)
- [ ] Implement `POST /api/auth/refresh` — rotate refresh token, issue new access token
- [ ] Write tests: auth middleware (valid token, expired, missing, tampered)
- [ ] Implement `authMiddleware` — verify JWT, attach `req.user`
- [ ] Write tests: logout (invalidates refresh token)
- [ ] Implement `POST /api/auth/logout` — blacklist refresh token in DB
- [ ] Build React login + register pages (form validation, error states)
- [ ] Implement `useAuth` hook (login, logout, token auto-refresh)
- [ ] Manual test: register → login → refresh → logout flow end-to-end

**Gate:** Auth test suite green (unit + API integration). Manual flow verified.

---

## Phase 2: Plaid Bank Account Integration ⬜

- [ ] Write tests: Plaid Link token creation (mock Plaid client)
- [ ] Implement `POST /api/plaid/link-token` — create Plaid Link token for user
- [ ] Write tests: Plaid token exchange (mock public token → access token)
- [ ] Implement `POST /api/plaid/exchange-token` — exchange public token, store encrypted access token
- [ ] Write tests: account sync (mock Plaid accounts response, verify DB write)
- [ ] Implement `POST /api/plaid/sync-accounts` — fetch and store accounts with balances
- [ ] Write tests: transaction sync (mock Plaid transactions, handle pagination, deduplication)
- [ ] Implement `POST /api/plaid/sync-transactions` — paginated sync, upsert with `transaction_id` deduplication
- [ ] Write tests: webhook handler (transaction updates, item errors)
- [ ] Implement `POST /api/plaid/webhook` — handle Plaid webhook events, trigger sync jobs
- [ ] Build React `PlaidLinkButton` component using `react-plaid-link`
- [ ] Build `ConnectedAccounts` page — list linked accounts with balances
- [ ] Manual test: sandbox bank link → accounts appear → transactions sync

**Gate:** Plaid integration tests green (mocked). Manual sandbox flow verified with evidence screenshot.

---

## Phase 3: AI Transaction Categorization ⬜

- [ ] Write tests: category classifier (mock GPT-4o, verify prompt structure, parse response)
- [ ] Implement `CategoryService.classify(transactions[])` — batch GPT-4o calls with structured output
- [ ] Write tests: category overrides (user overrides AI category, persisted correctly)
- [ ] Implement `PATCH /api/transactions/:id/category` — manual category override
- [ ] Write tests: category learning (overrides improve future suggestions — store feedback)
- [ ] Implement feedback loop: store user corrections as few-shot examples per user
- [ ] Write tests: post-sync categorization trigger (new transactions get auto-categorized)
- [ ] Implement categorization job that runs after each Plaid sync
- [ ] Seed 20 standard categories: Food, Transport, Housing, Health, Entertainment, etc.
- [ ] Write tests: `GET /api/transactions` — filters by date, category, account, amount range
- [ ] Implement transactions list endpoint with filtering + pagination
- [ ] Build `TransactionList` component with category badge, override UI
- [ ] Manual test: sync transactions → verify AI categories → override one → verify persistence

**Gate:** Categorization accuracy >80% on test fixtures. All tests green.

---

## Phase 4: Budget Tracking ⬜

- [ ] Write tests: create budget (valid, duplicate category, negative amount)
- [ ] Implement `POST /api/budgets` — create per-category monthly budget
- [ ] Write tests: budget progress calculation (spent vs. limit, rollover logic)
- [ ] Implement `GET /api/budgets` — return budgets with current-period spending progress
- [ ] Write tests: budget alerts (triggers when 80% and 100% spent)
- [ ] Implement budget alert engine — runs after each transaction sync
- [ ] Write tests: update/delete budget
- [ ] Implement `PATCH /api/budgets/:id` and `DELETE /api/budgets/:id`
- [ ] Build `BudgetCard` component — progress bar, spent/remaining, alert state
- [ ] Build `BudgetDashboard` page — grid of all category budgets
- [ ] Manual test: create budget → sync transactions → verify progress → trigger alert

**Gate:** Budget engine tests green. Budget dashboard renders correctly with real data.

---

## Phase 5: Spending Trends & Analytics ⬜

- [ ] Write tests: monthly spending aggregation by category (edge: no transactions, partial month)
- [ ] Implement `GET /api/analytics/monthly` — spending by category for N months
- [ ] Write tests: trend detection (month-over-month change, spike detection)
- [ ] Implement `GET /api/analytics/trends` — MoM % change per category
- [ ] Write tests: net worth calculation (assets − liabilities across accounts)
- [ ] Implement `GET /api/analytics/net-worth` — historical net worth time series
- [ ] Build `SpendingChart` component (Recharts bar chart — monthly by category)
- [ ] Build `TrendIndicator` component (% change badges with up/down arrows)
- [ ] Build `NetWorthChart` component (Recharts area chart — rolling 12 months)
- [ ] Build `AnalyticsDashboard` page — assemble charts + summary cards

**Gate:** Analytics endpoints return correct values against seed data. Charts render in Storybook/browser.

---

## Phase 6: AI Insights & Recommendations ⬜

- [ ] Write tests: insight generator (mock GPT-4o, verify prompt includes spending context, parse structured JSON response)
- [ ] Implement `InsightService.generate(userId)` — build spending context, call GPT-4o, return typed insights
- [ ] Write tests: insight caching (insights cached for 24h, refreshed on significant spending change)
- [ ] Implement insight cache layer (PostgreSQL `insights` table with `generated_at`)
- [ ] Write tests: `GET /api/insights` — returns cached or freshly generated insights
- [ ] Implement insights endpoint
- [ ] Define insight types: `spending_spike`, `savings_opportunity`, `bill_prediction`, `budget_warning`, `positive_trend`
- [ ] Build `InsightCard` component — icon, headline, detail, action CTA
- [ ] Build `InsightsFeed` page — list of AI insights with refresh button
- [ ] Manual test: load test user data → generate insights → verify relevance

**Gate:** Insight generation tests green (mocked GPT-4o). Insights make sense on real sandbox data.

---

## Phase 7: Polish & Harden ⬜

- [ ] Add rate limiting (express-rate-limit) on all API endpoints
- [ ] Add request validation middleware (zod schemas on all inputs)
- [ ] Add structured logging (pino) with request IDs
- [ ] Write e2e tests (Playwright): register → link bank → view transactions → set budget
- [ ] Add database indexes on `transactions(user_id, date, category)` and `accounts(user_id)`
- [ ] Implement graceful shutdown (drain in-flight requests before process exit)
- [ ] Add health check endpoint `GET /api/health` — DB ping, Plaid ping
- [ ] Security audit: check OWASP Top 10, verify no raw SQL, verify all inputs sanitized
- [ ] Run `npm audit` — resolve any high/critical vulnerabilities
- [ ] Performance: verify transactions list API < 200ms with 10k row dataset
- [ ] Add error boundary to React app — friendly error pages
- [ ] Accessibility audit — keyboard navigation, ARIA labels on charts

**Gate:** E2e tests green. No high/critical npm audit findings. Performance benchmark passes.

---

## Phase 8: Ship ⬜

- [ ] Write deployment documentation (`docs/deployment.md`)
- [ ] Set up production environment variables in GitHub Secrets
- [ ] Add GitHub Actions deployment workflow (build → test → deploy)
- [ ] Configure production PostgreSQL (connection pooling via pgBouncer or Neon)
- [ ] Add Sentry error monitoring (frontend + backend)
- [ ] Set up uptime monitoring (Better Uptime or similar)
- [ ] Create demo Plaid sandbox account with seeded data for demos
- [ ] Final security review: rotate all sandbox keys, verify prod keys never in code
- [ ] Tag `v0.1.0` release

**Gate:** Production deploy succeeds. Health check passes. At least one full user flow works in production.

---

## Parking Lot 🅿️

- Mobile app (React Native)
- Investment account tracking (brokerage via Plaid Investments)
- Bill detection and auto-scheduling
- CSV export of transactions
- Multi-currency support
- Family/shared account view
- OpenBanking (UK) / European PSD2 support
- Recurring transaction detection

---

## Lessons Learned 📝

_Update this section as you discover non-obvious things about the stack, Plaid API quirks, GPT-4o prompt patterns, etc._

- 
