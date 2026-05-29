# ADR 0001: [Title]

**Date:** YYYY-MM-DD  
**Status:** Proposed | Accepted | Deprecated | Superseded by ADR-XXXX  
**Deciders:** [names or roles]

---

## Context

What is the issue that motivated this decision? What forces are at play (technical, business, organizational)?

## Decision

What is the change we're proposing and/or doing?

## Rationale

Why this option over the alternatives?

| Option | Pros | Cons |
|--------|------|------|
| Chosen option | ... | ... |
| Alternative A | ... | ... |
| Alternative B | ... | ... |

## Consequences

What becomes easier or harder after this decision? Are there any risks or trade-offs?

### Positive
- 

### Negative / Trade-offs
- 

### Risks
- 

---

## Examples in FinTrack

_Decisions made so far:_

- **ADR 0002**: Use Plaid `/transactions/sync` (cursor-based) over `/transactions/get` (date-range) — avoids duplicate transactions and handles modifications/removals natively.
- **ADR 0003**: Use Drizzle ORM over Prisma — lighter weight, closer to SQL, better TypeScript inference for complex queries.
- **ADR 0004**: Use Vitest over Jest — faster, native ESM support, no config gymnastics with TypeScript.
- **ADR 0005**: Store Plaid access tokens AES-256 encrypted — regulatory compliance + defense in depth.
