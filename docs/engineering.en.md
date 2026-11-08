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

## 5. Adding sources and author editing without losing historical references

**Problem.** Adding another catalog source can cause code collisions or change existing IDs. Allowing authors to edit shared foods also requires separating the current reference from the values actually consumed in the past.

**Decision.** Preserve all 597 TACO IDs and add 19 USDA references through new migrations, retaining stable global identity, textual source-specific codes, and provenance. Checking batches against the official archive, including checksum and nutrients, makes verification reproducible without a runtime external lookup. TBCA was not incorporated.

Editing owned private and public foods retains ID and visibility, with author authorization. The current reference serves new entries; historical snapshots and calculation bases remain intact. Public deletion remains blocked. If an offline consumption refers to a food that changed, the conflict requires review rather than silent replacement.

**Existing coverage.** `additional-catalog.spec.ts` and `catalog-expansion.spec.ts` verify tilapia, mustard, attribution, proportional calculations, and the updated offline catalog. HTTP and SQL integration verify authorship, IDs, and history; `public-food-management*.spec.ts` cover editing, recovery, conflict, and synchronization.

## 6. Selecting CI without relying solely on the import graph

**Problem.** Compiled schemas, dynamic imports, and migrations enumerated through the file system create dependencies that import-based selection may miss.

**Decision.** Select whole package suites for `staging`, expanding to all unit suites for shared changes, deletions, renames, unknown paths, or comparison failures. The selection job always tests its own scripts. Documentation-only changes can skip application jobs. `main` and manual runs retain database/HTTP integration and the full Playwright matrix.

The project also enables Bun 1.4.4’s global dependency store with `linker = "isolated"` and `globalStore = true`, sharing eligible packages across projects and worktrees. This is distinct from the download cache restored by CI. Docker retains `--linker hoisted` to copy local dependencies between stages.

The global Bun download cache (`~/.bun/install/cache`) does not replace `bun install --frozen-lockfile`. The intermediate Next.js cache is separate from the static artifact produced in the current run. The gate accepts skipped jobs only when the plan permits them and rejects failures, cancellations, or missing jobs. No measured time or cost savings are published here.

**Promotion quality.** Refresh and catalog-restoration fixtures were corrected, along with waiting for an aborted request in the batch test. The E2E limit increased to 45 minutes without reducing coverage. The draft-concurrency test now controls notifications between two tabs while retaining the stale-revision rejection assertion; that last fix did not change the application.

## Validation strategy

| Layer                                   | What it verifies                                                                                          |
| --------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| Shared-contract and frontend unit tests | Schema validation, queue transitions, projection, locking, error classification, and calculation behavior |
| Backend HTTP integration                | NestJS, verified identity, authorization, Prisma, and PostgreSQL behavior together                        |
| SQL integration                         | Domain invariants, account boundaries, transaction behavior, and receipts                                 |
| Playwright                              | User-visible flows through the static frontend and API, including offline and recovery paths              |
| GitHub Actions                          | Conservative package selection for `staging`; full validation for `main` and manual runs                  |

Integration tests use an isolated PostgreSQL database. The local Auth fixture replaces the external identity provider, not the food catalog or application database. Browser tests include desktop, mobile Chromium, and mobile WebKit configurations; individual cases may be browser-specific.

The showcase intentionally avoids unverified latency claims, coverage percentages, production availability claims, and fixed passing-test counts. Those need dated execution evidence from the relevant application revision.

## v0.7.0 reference

Verified through `gh` on November 8, 2026: v0.7.0 was merged into `main` as `8f5a586480bca3ac2c049ad40b0c7d7531c4a734`. The successful full run is [42449310809](https://github.com/gustavo867/tactical-calories/actions/runs/42449310809), on pre-merge HEAD `f9768bb44837b078e95757bc13ab4fbca05adb87`: base, desktop, mobile, mobile Safari, and gate passed. Supabase Preview was skipped by configuration. This is dated evidence, not CI rerun in the showcase or manual validation of a hosted deployment.

The references below belong to the private product repository; the public showcase content and images do not require access to it.

- Features: PRs [#135](https://github.com/gustavo867/tactical-calories/pull/135), [#136](https://github.com/gustavo867/tactical-calories/pull/136), and [#137](https://github.com/gustavo867/tactical-calories/pull/137).
- CI: PRs [#139](https://github.com/gustavo867/tactical-calories/pull/139) and [#140](https://github.com/gustavo867/tactical-calories/pull/140).
- Promotion and fixes: PRs [#142](https://github.com/gustavo867/tactical-calories/pull/142), [#143](https://github.com/gustavo867/tactical-calories/pull/143), [#144](https://github.com/gustavo867/tactical-calories/pull/144), and [#145](https://github.com/gustavo867/tactical-calories/pull/145).
- Documentation consulted at the confirmed commit: `README.md`, `docs/backend-architecture.md`, `docs/ci.md`, `docs/validation/issue-133-ampliacao.md`, `docs/validation/issue-134.md`, `docs/validation/pr-142.md`, and `docs/validation/pr-142-draft-concurrency.md`.
