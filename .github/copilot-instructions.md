# GitHub Copilot Instructions — FinTrack

## Stack

- **Backend**: Node.js 20, Express, TypeScript (strict), Drizzle ORM, PostgreSQL 16
- **Frontend**: React 18, TypeScript, Vite, TailwindCSS, React Query, Recharts
- **AI**: OpenAI GPT-4o (via `openai` npm package, structured output mode)
- **Banking**: Plaid API (`plaid` npm package, `/transactions/sync` endpoint)
- **Auth**: JWT (access) + refresh tokens (httpOnly cookie, token rotation)
- **Testing**: Vitest + Supertest (API) + Playwright (e2e)

## TypeScript Conventions

- Strict mode always — no `any` without justification comment
- Named exports only — never `export default`
- Interfaces for object shapes, `type` for unions
- Explicit return types on all `src/api/` functions
- Zod for all runtime validation (request bodies, env vars, GPT-4o responses)

## API Conventions

```typescript
// Route handler pattern
export const createBudget = async (
  req: Request,
  res: Response,
  next: NextFunction
): Promise<void> => {
  try {
    const budget = await BudgetService.create(req.user.id, req.body);
    res.status(201).json({ data: budget });
  } catch (error) {
    next(error); // always delegate to errorHandler middleware
  }
};
```

- All routes use `validateBody(zodSchema)` middleware before handler
- Auth-protected routes use `authMiddleware` before handler
- Response envelope: `{ data: T }` for success, `{ error: { message, code } }` for errors

## React Conventions

```typescript
// Hook pattern for data fetching
export function useBudgets() {
  return useQuery({
    queryKey: ['budgets'],
    queryFn: () => api.get<Budget[]>('/budgets'),
  });
}

// Component pattern
export function BudgetCard({ budget }: { budget: Budget }) {
  // No direct fetch calls here — use hooks
}
```

- All data fetching via React Query hooks in `src/web/hooks/`
- Never call `fetch()` or `axios` directly in components
- All API calls through `src/web/lib/api.ts` (handles auth token injection + 401 refresh)

## OpenAI / GPT-4o Conventions

```typescript
// Always use structured output
const response = await openai.chat.completions.create({
  model: 'gpt-4o',
  response_format: { type: 'json_object' }, // requires "JSON" in system prompt
  messages: [
    { role: 'system', content: CATEGORIZATION_SYSTEM_PROMPT }, // must contain "JSON"
    { role: 'user', content: buildTransactionBatch(transactions) },
  ],
});
const result = CategoryResponseSchema.parse(JSON.parse(response.choices[0].message.content!));
```

- Always validate GPT-4o responses with Zod before using
- Never trust raw GPT-4o output without parsing and validating
- Mock OpenAI in all tests — never hit the real API in tests

## Plaid Conventions

```typescript
// Always use cursor-based sync
const syncResponse = await plaidClient.transactionsSync({
  access_token: decryptedToken,
  cursor: item.cursor ?? undefined,
});
// Process added, modified, removed arrays separately
```

- Encrypt/decrypt access tokens with `TokenService` — never store raw
- Handle Plaid `ITEM_LOGIN_REQUIRED` error by updating item status to `error`
- Verify webhook signatures before processing webhook payloads

## Database Conventions

```typescript
// Multi-table writes always in a transaction
await db.transaction(async (tx) => {
  const item = await tx.insert(plaidItems).values(itemData).returning();
  await tx.insert(accounts).values(accountsData);
});
```

- Use Drizzle's query builder, not raw SQL (exception: complex analytics)
- All UUIDs use PostgreSQL `gen_random_uuid()` default
- Never edit existing migration files

## Testing Conventions

```typescript
// Mock external services at module level
vi.mock('../services/PlaidService', () => ({
  PlaidService: {
    syncTransactions: vi.fn().mockResolvedValue(mockTransactions),
  },
}));

// Use fixtures from tests/fixtures/
import { mockUser, mockTransaction } from '../../fixtures';
```

- Write failing test first (red), then implement (green)
- Test file location: `tests/api/routes/budgets.test.ts` for `src/api/routes/budgets.ts`
- No `console.log` in tests — use `expect` assertions

## Boundaries

- Do NOT refactor working code unless explicitly asked
- Do NOT remove or skip existing tests
- Do NOT add `eslint-disable` comments without justification
- Do NOT introduce new npm dependencies without mentioning it
- Do NOT use `process.env.X` directly — always use the validated `config` object from `src/api/config.ts`
- Do NOT add `TODO` comments — add items to `TODO.md` instead
