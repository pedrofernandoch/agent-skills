# Code Quality Checklist

Quick reference for architectural integrity and implementation hygiene. Use alongside the
`code-review-and-quality` and `code-simplification` skills.

This file covers the quality concerns that don't have a home elsewhere in `references/`. Naming,
readability, over-abstraction, dead code, review process, dependency vetting, performance, testing,
observability, API design, and migration safety each live in their own skill or checklist — see
[See Also](#see-also) rather than duplicating them here.

## Table of Contents

- [Architecture Principles](#architecture-principles)
- [Separation of Concerns](#separation-of-concerns)
- [Modularity](#modularity)
- [Type Safety](#type-safety)
- [Lint and Suppression Discipline](#lint-and-suppression-discipline)
- [Error Handling](#error-handling)
- [Side Effects](#side-effects)
- [State and Immutability](#state-and-immutability)
- [Constants and Configuration](#constants-and-configuration)
- [Data Access](#data-access)
- [Async and Concurrency](#async-and-concurrency)
- [See Also](#see-also)

## Architecture Principles

Apply SOLID pragmatically. These are diagnostics for code that is already hard to change, not a
checklist to satisfy up front — an abstraction added to honor a principle nothing is pressing on is
just [overengineering](#see-also).

| Principle | What it asks | Smell it catches |
|---|---|---|
| **Single Responsibility** | Does this module have one reason to change? | A component that renders, fetches, validates, formats, and persists |
| **Open/Closed** | Can behavior be extended without editing complex existing logic each time? | A `switch` that grows a branch with every new case, touched by every feature |
| **Liskov Substitution** | Does every implementation honor the contract callers rely on? | A subtype that throws where the base returns, or narrows accepted input |
| **Interface Segregation** | Are consumers forced to depend on methods they don't use? | A 20-method interface where implementers stub half of it |
| **Dependency Inversion** | Does business logic depend on an abstraction rather than a concrete infrastructure detail? | A domain service that imports the database client directly |

- [ ] Each module, class, function, component, and hook has a responsibility you can state in one
      sentence without "and"
- [ ] High-level logic depends on abstractions, following the project's established DI pattern —
      not on a new one you introduced
- [ ] No abstraction, interface, factory, or wrapper added for extensibility nothing is asking for
- [ ] No new architectural pattern introduced where an established project pattern already solves
      the problem

## Separation of Concerns

Keep these apart where the project's architecture separates them: presentation, business logic,
domain logic, data access, infrastructure, validation, serialization, authentication,
authorization, external integrations, configuration.

- [ ] UI components are not a dumping ground for unrelated business logic
- [ ] Infrastructure details (SQL, HTTP clients, SDK types) don't leak into domain logic
- [ ] Business rules are testable without spinning up the database, the network, or the UI
- [ ] Validation lives at the boundary, not scattered through the call chain
- [ ] The separation follows the project's existing architecture rather than a cleaner one you'd
      prefer

## Modularity

A module should have a clear purpose, expose only what consumers need, minimize coupling, and have
predictable inputs and outputs.

- [ ] Public surface is the minimum consumers need; internals aren't exported "just in case"
- [ ] No circular dependencies between modules
- [ ] No hidden side effects at import time (module-level I/O, network calls, singleton
      construction, global registration)
- [ ] Boundaries represent responsibilities, not file-size targets — splitting a long file into
      `utils-1` and `utils-2` is not modularity
- [ ] Related code is colocated; a change to one concept doesn't require edits in six directories

## Type Safety

Where the project uses a type system:

- [ ] Types are precise; `any` appears only where genuinely unavoidable, and with a comment saying why
- [ ] No unsafe assertions (`as Foo`, `as unknown as Foo`) papering over a real mismatch
- [ ] No unnecessary non-null assertions (`!`) — narrow instead
- [ ] Domain states modeled explicitly; impossible states are unrepresentable
- [ ] Discriminated unions used where a value has distinct shapes per variant
- [ ] Types match actual runtime behavior (an optional field typed as required is a lie)

**Types are not runtime validation.** A compile-time type says nothing about what actually arrived
over the wire.

```typescript
// WRONG: the type is an assertion about data you haven't checked
const user = await res.json() as User;

// RIGHT: validate at the boundary, then the type is earned
const user = UserSchema.parse(await res.json());
```

- [ ] Every external input — API responses, request bodies, query params, env vars, webhook
      payloads, parsed files, database JSON columns — is validated at the boundary, not just typed

## Lint and Suppression Discipline

Respect the project's existing ESLint, Prettier, Stylelint, TypeScript, compiler, and EditorConfig
configuration, plus its import and naming rules.

- [ ] No lint rule disabled, config loosened, or check removed to make an implementation pass
- [ ] No new `eslint-disable`, `@ts-ignore`, `@ts-expect-error`, `# noqa`, `# type: ignore`, or
      formatter exception without investigating what the rule is actually reporting
- [ ] A necessary suppression is line-scoped (never file- or project-wide), names the specific
      rule, and carries a comment explaining why

```typescript
// WRONG: blanket, unexplained, whole file
/* eslint-disable */

// RIGHT: narrow, specific, justified
// eslint-disable-next-line no-await-in-loop -- migrations must run sequentially
await runMigration(migration);
```

- [ ] `@ts-expect-error` preferred over `@ts-ignore` where both apply — it fails when the underlying
      problem is fixed, so it can't rot silently

## Error Handling

Handle errors intentionally, following the project's existing error architecture.

Distinguish these — they need different handling, logging, and status codes:

| Kind | Typical handling |
|---|---|
| Expected business error | Return a typed result; not exceptional |
| Validation error | Reject at the boundary with field-level detail |
| Authentication error | 401, generic message, logged |
| Authorization error | 403, generic message, logged with principal |
| Infrastructure failure | Retry where safe, timeout, circuit-break, alert |
| Programming error | Fail loudly; don't catch to hide it |

- [ ] No empty `catch` blocks and no silently swallowed exceptions
- [ ] Errors are not caught, logged, and rethrown at every layer — log once, at the boundary that
      can act on it
- [ ] Error shape is consistent across the codebase (one error type/response contract, not five)
- [ ] Exceptions are not used for ordinary control flow without a reason
- [ ] Context preserved for debugging (`cause`, operation name, identifiers) without leaking
      sensitive data into the message
- [ ] Catch blocks narrow to what they can actually handle; unexpected errors propagate

```typescript
// WRONG: intent erased, cause lost, caller can't distinguish anything
try { await save(order); } catch { return null; }

// RIGHT: expected failure typed, unexpected failure propagates with context
try {
  await save(order);
} catch (err) {
  if (err instanceof UniqueConstraintError) return { ok: false, reason: 'duplicate' };
  throw new Error(`saving order ${order.id}`, { cause: err });
}
```

## Side Effects

Make side effects explicit and predictable.

- [ ] Functions that mutate, write, emit, or call out say so in their name and signature — a
      function called `getUser` does not create one
- [ ] No hidden dependence on module-level mutable state or globals
- [ ] Pure functions preferred where practical; logic that can be pure is separated from the I/O
      around it
- [ ] These are deliberate and visible at the call site: global state, hidden mutations, network
      requests, database writes, filesystem operations, event emission, timers, background jobs
- [ ] Timers, subscriptions, listeners, watchers, and connections are cleaned up on teardown

## State and Immutability

- [ ] The project's established state-management paradigm is followed; no second competing
      mechanism introduced without an architectural reason
- [ ] State is kept as local as possible — lifted only when something else genuinely needs it
- [ ] No mutation of function arguments, shared objects, cached values, or props
- [ ] Where the project uses immutable patterns, they're preserved consistently
- [ ] Derived values are computed, not stored and kept in sync by hand

## Constants and Configuration

- [ ] No magic numbers or magic strings where the value carries meaning — business rules, limits,
      timeouts, retry counts, status values, feature keys, repeated literals
- [ ] Constants are named for what they mean, not what they are (`MAX_UPLOAD_BYTES`, not `LIMIT_5MB`)
- [ ] Not every literal is extracted — `0`, `1`, and a one-off array index are clearer inline
- [ ] No hardcoded environment-specific values, URLs, endpoints, or credentials
- [ ] New configuration goes through the project's existing mechanism, is documented, and has a
      sensible default
- [ ] Configuration is read at a boundary and passed down, not read from `process.env` deep inside
      business logic
- [ ] Existing defaults are not changed without understanding what depends on them

## Data Access

Keep data access consistent with the project's architecture.

- [ ] No N+1 queries — a loop that queries per item is batched, joined, or preloaded
- [ ] Every query is bounded: explicit limits, pagination, and no unbounded `SELECT *` over a
      growing table
- [ ] Only the columns and relations actually used are fetched
- [ ] No duplicate queries for the same data within one request
- [ ] Persistence logic stays in the data layer rather than spreading through business logic
- [ ] Transactions used where multiple writes must be atomic — and not wrapped around unrelated
      work or network calls
- [ ] Query patterns match the available indexes
- [ ] Dynamic query construction is parameterized (see
      [security-checklist.md](security-checklist.md#database-security))

## Async and Concurrency

- [ ] No unhandled promises — every async call is awaited, returned, or explicitly handled
- [ ] Independent operations run concurrently rather than in a sequential `await` chain
- [ ] Concurrency is *not* introduced where ordering or transactional guarantees are required
- [ ] Every outbound call has a timeout; retries are bounded and backed off
- [ ] Cancellation is supported and honored (`AbortSignal`, cancellation token, cleanup on unmount)
- [ ] Stale responses can't overwrite fresh state — out-of-order responses are discarded
- [ ] Duplicate submissions are prevented or made idempotent
- [ ] Resources are released on every path, including error paths

```typescript
// WRONG: sequential for no reason; a slow response can overwrite a newer one
const user = await fetchUser(id);
const prefs = await fetchPrefs(id);
setResults(await search(query));

// RIGHT: parallel where independent, and the stale response is dropped
const [user, prefs] = await Promise.all([fetchUser(id), fetchPrefs(id)]);

useEffect(() => {
  const controller = new AbortController();
  search(query, { signal: controller.signal }).then(setResults).catch(ignoreAbort);
  return () => controller.abort();
}, [query]);
```

## See Also

The rest of what "clean code" covers lives closer to its subject:

| Concern | Where |
|---|---|
| Naming, readability, clarity over cleverness, over-simplification | `code-simplification` |
| Following existing conventions, scoping refactors to what changed | `code-simplification` |
| Review process, five-axis review, dead code hygiene, change sizing | `code-review-and-quality` |
| Whether to add a dependency at all | `code-review-and-quality` (Dependency Discipline) |
| Supply-chain vetting, lockfiles, install scripts | [security-checklist.md](security-checklist.md#dependency-security) |
| Comments, documentation, architectural decision records | `documentation-and-adrs` |
| API naming, status codes, pagination, contracts | `api-and-interface-design` |
| Component structure, prop drilling, re-renders | `frontend-ui-engineering` |
| Accessibility | [accessibility-checklist.md](accessibility-checklist.md) |
| Security | [security-checklist.md](security-checklist.md) |
| Algorithmic complexity, caching, Core Web Vitals | [performance-checklist.md](performance-checklist.md) |
| Test structure, coverage of edge cases, mocking limits | [testing-patterns.md](testing-patterns.md) |
| Logging, metrics, tracing for new functionality | [observability-checklist.md](observability-checklist.md) |
| Backward compatibility, expand/contract schema migrations | `deprecation-and-migration` |
| The completion bar for any change | [definition-of-done.md](definition-of-done.md) |
