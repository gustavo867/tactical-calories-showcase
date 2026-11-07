# Engineering case studies

[Back to the showcase](../README.en.md) · [Português](engineering.md)

These cases explain implementation decisions in the private application. Test references identify existing coverage; they are not a claim that those suites were rerun while preparing this documentation.

## 1. Retrying a successful request without recording a meal twice

**Failure scenario.** PostgreSQL commits an offline food entry, but the connection drops before the browser receives the response. The browser still has the pending operation and cannot safely infer whether it succeeded.

**Decision.** Keep a stable `operationId` and preserve the original payload across retries. The database stores a receipt scoped to the authenticated account in the same transaction as the mutation. Repeating the same operation returns a replay acknowledgment and current state. Reusing its identifier with a different payload returns an idempotency conflict.

Batch and meal-registration flows use stable `attemptId` values. When a response may have been lost, the interface retains the original attempt for replay before permitting a new edited submission. Batch confirmation is transactional: all items are recorded together or none are.

**Why it matters.** A timeout becomes recoverable without requiring the user to guess whether another click will double their intake. This provides idempotent effects on replay; it does not assume the network delivers requests exactly once.

**Existing coverage.** `frontend/e2e/offline-sync.spec.ts`, backend HTTP flows, and SQL receipt tests exercise lost responses and replay. Batch/meal flows also test repeated attempts and changed payloads.

## 2. Resolving conflicts without rewriting nutritional history

**Failure scenario.** A user changes goals offline while another device updates them online. Separately, an offline food entry may refer to nutritional values that have changed before synchronization.

**Decision.** Goal operations carry a base revision; catalog-based consumption carries a nutritional hash. The server reports `goals_conflict` or `food_changed` with current state. The client preserves the pending operation and pauses sending for a decision. Goal recovery offers discard or application from today; nutritional recovery compares the values originally used with the current reference.

Recorded consumption also retains its nutritional snapshot and, where available, an immutable calculation basis. Later catalog edits do not recalculate past intake. Missing macronutrients remain unknown rather than being converted to zero.

**Trade-off.** Explicit recovery adds UI and domain logic. It preserves the meaning of the user's record instead of hiding a change behind a successful synchronization indicator.

**Existing coverage.** `frontend/e2e/offline-conflicts.spec.ts` exercises goal recovery, changed references, and unavailable foods. Quantity and nutrition-precision suites cover calculations and preserved historical values.

## 3. Keeping browser state correct across tabs and delayed responses

**Failure scenario.** Two tabs send pending work simultaneously, or an old bootstrap request finishes after a newer mutation has already been acknowledged. A user may also create another local action while a request is in flight.

**Decision.** Serialize senders with Web Locks or an IndexedDB lease, and notify peers with BroadcastChannel. Store a confirmed-state version and reject stale bootstrap writes. Apply acknowledgments transactionally and rebuild the visible projection from the current pending queue, not the queue that existed when the request began.

**Trade-off.** The local store needs explicit state transitions, version checks, and recovery logic. The benefit is preserving both newer confirmed data and newly created pending work.

**Existing coverage.** `frontend/e2e/offline-bootstrap-race.spec.ts` covers stale bootstrap responses. `frontend/e2e/offline-sync.spec.ts` covers the lease fallback and actions created during a request. Unit tests target the lock, sync loop, and projection.

## 4. Preserving pending work without crossing account boundaries

**Failure scenario.** Account A has pending changes when its session expires. The next person signs in as account B on the same browser.

**Decision.** Partition local data by account, revalidate the session before sending, and include the expected account on requests. Keep account A's pending queue for account A rather than submitting it through account B's session. A `401` requests reauthentication without deleting pending actions. Local sign-out deactivates local access without claiming to have revoked the remote session.

**Trade-off.** Identity, local access, and network session state require separate handling. Browser storage is still subject to clearing and eviction; retaining a queue is not a permanent backup guarantee.

**Existing coverage.** `frontend/e2e/offline-accounts.spec.ts` covers session expiry, account switching, and resuming work with the original account. HTTP and SQL tests cover authorization boundaries.

## Validation strategy

| Layer                                   | What it verifies                                                                                          |
| --------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| Shared-contract and frontend unit tests | Schema validation, queue transitions, projection, locking, error classification, and calculation behavior |
| Backend HTTP integration                | NestJS, verified identity, authorization, Prisma, and PostgreSQL behavior together                        |
| SQL integration                         | Domain invariants, account boundaries, transaction behavior, and receipts                                 |
| Playwright                              | User-visible flows through the static frontend and API, including offline and recovery paths              |
| GitHub Actions                          | Lint, types, unit tests, and build for `staging`; additional integration and browser suites for `main`    |

Integration tests use an isolated PostgreSQL database. The local Auth fixture replaces the external identity provider, not the food catalog or application database. Browser tests include desktop, mobile Chromium, and mobile WebKit configurations; individual cases may be browser-specific.

The showcase intentionally avoids unverified latency claims, coverage percentages, production availability claims, and fixed passing-test counts. Those need dated execution evidence from the relevant application revision.
