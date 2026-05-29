# FinTrack

> 🚧 **Status: Early Development**

Personal finance tracking app — connect bank accounts via Plaid, categorize transactions automatically with GPT-4o, track spending trends, set budgets, and receive AI-powered insights and recommendations for improving financial health.

[![TypeScript](https://img.shields.io/badge/TypeScript-5.x-blue)](https://www.typescriptlang.org/)
[![React](https://img.shields.io/badge/React-18.x-61DAFB)](https://react.dev/)
[![Node.js](https://img.shields.io/badge/Node.js-20.x-green)](https://nodejs.org/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16.x-336791)](https://www.postgresql.org/)
[![OpenAI](https://img.shields.io/badge/OpenAI-GPT--4o-412991)](https://platform.openai.com/)
[![Plaid](https://img.shields.io/badge/Plaid-API-00B0B9)](https://plaid.com/)

---

## Features

- 🏦 **Bank Connectivity** — OAuth bank account linking via Plaid Link
- 🤖 **AI Categorization** — GPT-4o auto-classifies transactions with custom category rules
- 📊 **Spending Trends** — Visual dashboards for monthly/weekly/category breakdowns
- 💰 **Budget Tracking** — Set per-category budgets with real-time alerts
- 💡 **AI Insights** — Personalized recommendations to improve financial health
- 🔔 **Smart Alerts** — Unusual spending, bill predictions, overdraft warnings

---

## Tech Stack

| Layer         | Technology                          |
|---------------|-------------------------------------|
| Frontend      | React 18, TypeScript, Vite, TailwindCSS |
| Backend       | Node.js 20, Express, TypeScript     |
| Database      | PostgreSQL 16, Drizzle ORM          |
| AI            | OpenAI GPT-4o (categorization + insights) |
| Banking       | Plaid API (transactions, balances)  |
| Auth          | JWT + refresh tokens                |
| Testing       | Vitest (unit), Supertest (API), Playwright (e2e) |
| CI/CD         | GitHub Actions                      |

---

## Getting Started

### Prerequisites

- Node.js 20+
- PostgreSQL 16+
- Plaid developer account (sandbox keys)
- OpenAI API key

### Installation

```bash
git clone https://github.com/joshiujjwal/fintrack.git
cd fintrack
npm install
```

### Environment Setup

```bash
cp .env.example .env
# Fill in: DATABASE_URL, PLAID_CLIENT_ID, PLAID_SECRET, OPENAI_API_KEY, JWT_SECRET
```

### Database Setup

```bash
npm run db:migrate
npm run db:seed        # optional: load demo data
```

### Development

```bash
# Start API server (port 3001) + React dev server (port 5173) concurrently
npm run dev
```

### Testing

```bash
npm test               # unit + integration tests (Vitest)
npm run test:e2e       # Playwright end-to-end tests
npm run test:coverage  # coverage report
```

### Build

```bash
npm run build          # compiles API + bundles frontend
```

---

## Project Structure

```
fintrack/
├── src/
│   ├── api/                  # Express backend
│   │   ├── routes/           # Route handlers (accounts, transactions, budgets, insights)
│   │   ├── middleware/        # Auth, validation, error handling
│   │   ├── services/         # Business logic (plaid, openai, budget engine)
│   │   └── models/           # Drizzle ORM schemas
│   ├── web/                  # React frontend
│   │   ├── components/       # Reusable UI components
│   │   ├── pages/            # Route-level page components
│   │   ├── hooks/            # Custom React hooks
│   │   └── lib/              # API client, utilities
│   └── shared/               # Types and utils shared between API and web
│       ├── types/            # Shared TypeScript interfaces
│       └── utils/            # Shared helpers (formatting, validation)
├── tests/
│   ├── api/                  # Supertest API integration tests
│   ├── web/                  # Vitest component tests
│   └── e2e/                  # Playwright end-to-end tests
├── docs/
│   ├── spec.md               # Feature specification
│   └── adr/                  # Architecture Decision Records
├── .github/
│   ├── copilot-instructions.md
│   ├── workflows/            # GitHub Actions CI
│   └── instructions/         # Path-specific Copilot instructions
├── CLAUDE.md                 # AI agent context & workflow
├── AGENTS.md                 # OpenAI Codex agent instructions
└── TODO.md                   # Evidence-gated task breakdown
```

---

## Contributing

- **Red/Green TDD**: Write a failing test before any implementation
- **Evidence in PRs**: Include `npm test` output in PR description
- **Small focused PRs**: One feature or fix per PR — no refactors bundled
- **Update context files**: If you learn something non-obvious, update `CLAUDE.md` or `AGENTS.md`
- **No secrets in code**: All credentials go in `.env` (never committed)

---

## Security

- All financial data encrypted at rest (PostgreSQL TDE)
- Plaid tokens stored encrypted, never raw credentials
- JWT access tokens (15 min) + refresh tokens (7 days, httpOnly cookie)
- Rate limiting on all API endpoints
- OWASP Top 10 mitigations in place

---

## License

Private — All rights reserved.
