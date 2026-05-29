# AGENTS.md — FinTrack

> Instructions for AI coding agents (OpenAI Codex, Claude, GitHub Copilot agent mode).

## Setup

```bash
# 1. Install dependencies
npm install

# 2. Start database
docker compose up -d postgres

# 3. Run migrations
npm run db:migrate

# 4. Copy and fill environment variables
cp .env.example .env

# 5. Verify everything works
npm test
```

## Before You Write Any Code

1. **Read `TODO.md`** — identify the current unchecked task in the current phase
2. **Run `npm test`** — all tests must pass before you start
3. **Read `docs/spec.md`** — understand the requirement before implementing it
4. **Read `CLAUDE.md`** — check for relevant gotchas in the "Non-Obvious Conventions" section

## Testing Rules (Non-Negotiable)

- **Write tests FIRST** — no implementation without a failing test
- **Never delete or skip tests** — if a test is failing, fix the code, not the test
- **Mock all external services** — Plaid and OpenAI must be mocked in all tests
- **Test files mirror src structure**: `tests/api/routes/auth.test.ts` ↔ `src/api/routes/auth.ts`
- **Run before committing**: `npm test` must be green before any commit

```bash
npm test                  # run all tests
npm run test:watch        # watch mode during development
npm run test:coverage     # coverage (target: >80% on services/)
npm run test:e2e          # Playwright e2e (requires dev server)
```

## Code Style

### TypeScript
- **Strict mode** — `strict: true` in tsconfig; no `any` without explicit `// eslint-disable` comment and justification
- **Named exports only** — no default exports (makes refactoring easier with IDEs)
- **Interfaces over types** for object shapes; `type` for unions and mapped types
- **Explicit return types** on all functions in `src/api/` — not required in React components

### API (Express)
- Route handler signature: `(req: Request, res: Response, next: NextFunction) => Promise<void>`
- All routes must call `next(error)` on failure — never `res.status(500).json({ error: ... })` inline
- Use `validateBody(schema)` middleware (Zod) before every mutating route handler
- Services are injected, not imported directly in routes — enables mocking in tests

### React / Frontend
- Functional components only — no class components
- `useQuery`-style hooks for all data fetching (React Query)
- Co-locate component files: `ComponentName/index.tsx`, `ComponentName.test.tsx`, `ComponentName.stories.tsx`
- Tailwind classes only — no inline `style={{}}` except for dynamic chart dimensions

### Database (Drizzle)
- All schema changes via migration files — never `ALTER TABLE` manually
- Use `db.transaction()` wrapping any multi-table writes
- Drizzle `select()` over raw SQL for all queries — exception: complex analytics aggregations

### Error Handling
- API errors: throw typed `AppError` instances — caught by `errorHandler` middleware
- Frontend errors: all `api.*` calls wrapped in try/catch or React Query error states
- Never swallow errors silently (`catch (e) {}` is forbidden)

## Commit Message Format

```
<type>: <short description>

Types: feat | fix | test | refactor | docs | chore | perf
```

Examples:
- `test: add failing tests for Plaid token exchange`
- `feat: implement Plaid token exchange endpoint`
- `fix: handle Plaid cursor pagination correctly`
- `docs: update CLAUDE.md with Plaid webhook quirk`

## PR Instructions

Every PR description must include:
1. **What**: one-sentence summary of the change
2. **Why**: links to the TODO.md task or spec section this implements
3. **Evidence**: paste output of `npm test` showing all tests pass
4. **Screenshots**: UI changes require before/after screenshots
5. **Checklist**: confirm no secrets committed, no tests deleted, types strict-clean

## What NOT to Do

- ❌ Do not call Plaid or OpenAI APIs in tests — always mock them
- ❌ Do not commit `.env` or any file containing API keys
- ❌ Do not use `any` type without a comment explaining why
- ❌ Do not delete or modify existing passing tests
- ❌ Do not refactor unrelated code while implementing a feature
- ❌ Do not use `console.log` in `src/` — use the `logger` (pino) instance
- ❌ Do not add npm packages without checking the bundle size impact for frontend packages
- ❌ Do not call `fetch()` directly in React components — use `src/web/lib/api.ts`
