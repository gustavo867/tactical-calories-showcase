# Tactical Calories

[Português](README.md) · **English**

**A food and hydration tracker built to keep working when the connection does not.**

Tactical Calories is a personal full-stack project: a responsive, installable web app for tracking meals, calories, macronutrients, water intake, and daily goals. It combines a React interface with a NestJS API and PostgreSQL, with a local replica and a durable queue for supported offline actions.

This repository presents the product and its engineering decisions. The application source code is maintained privately.

**React · Next.js · TypeScript · NestJS · PostgreSQL · Prisma · Zod · Playwright**

![Desktop diary showing calorie balance, macronutrient progress, meal cards, and hydration](assets/screenshots/daily-dashboard-desktop.png)

[Product tour](#product-tour) · [Architecture](docs/architecture.en.md) · [Engineering challenges](docs/engineering.en.md) · [AI-assisted development](#ai-assisted-development)

## Why I built it

Recording a meal should remain possible through a connection failure. The engineering challenge is making local changes durable, reconciling them with an account later, and making the difference between “saved on this device” and “synced to your account” clear.

The project explores that problem end to end: browser persistence, API contracts, database transactions, conflict recovery, and automated browser tests.

## What the app does

- Tracks food intake, calories, macronutrients, hydration, meal notes, and goals that take effect from a chosen date.
- Searches an imported TACO food catalog and supports personal foods with private or explicitly shared visibility.
- Handles quantities in grams, servings, and custom household measures, with a review step for multi-food selections.
- Copies meals and saves reusable favorite combinations while preserving the original nutritional values.
- Shows historical intake and the goals applicable to each day.
- Supports Google sign-in and installation as a Progressive Web App (PWA).
- Saves supported individual diary actions offline and synchronizes them after reconnection and account verification.

## Product tour

The interface is in Brazilian Portuguese. These are actual development and validation captures, with illustrative entries; they represent different iterations of the product. See [screenshot notes](assets/screenshots/README.en.md) for provenance and context.

<table>
  <tr>
    <th>Food search</th>
    <th>Reusable meals</th>
    <th>Offline entry</th>
  </tr>
  <tr>
    <td valign="top"><img src="assets/screenshots/food-search-mobile.png" width="260" alt="Mobile food search with nutritional results and a multi-food selection summary"></td>
    <td valign="top"><img src="assets/screenshots/favorite-meals-mobile.png" width="260" alt="Saved breakfast combination with use, rename, and delete actions"></td>
    <td valign="top"><img src="assets/screenshots/offline-food-entry-mobile.png" width="260" alt="Food detail offering an individual offline entry while batch confirmation requires connection"></td>
  </tr>
  <tr>
    <td>Search the catalog and review a selection before recording it.</td>
    <td>Save meal combinations and review them before reuse.</td>
    <td>Keep individual offline entry available without silently submitting a batch.</td>
  </tr>
</table>

<details>
<summary>Desktop meal actions and hydration</summary>

![Desktop diary with copy meal and save as favorite actions, meal notes, and hydration progress](assets/screenshots/meal-actions-desktop.png)

</details>

## Engineering highlights

| Problem                                           | Design decision                                                           | Resulting behavior                                                          |
| ------------------------------------------------- | ------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| A request succeeds but its response is lost       | Stable operation IDs and transactional receipts scoped to the account     | Retrying the original operation does not duplicate consumption              |
| Goals or food references change on another device | Goal revisions and nutritional hashes, with explicit conflict handling    | Users review conflicts before blocked operations continue                   |
| A delayed response arrives after newer local work | Version checks on confirmed snapshots and reprojection of pending actions | Older responses cannot silently replace newer confirmed state               |
| Multiple tabs try to synchronize                  | Web Locks with an IndexedDB lease fallback                                | Tabs coordinate sending; server receipts remain the final duplication guard |
| A food's nutritional values change later          | Immutable nutritional snapshots on recorded consumption                   | Historical intake remains tied to the values used when recorded             |
| A user switches accounts with pending changes     | Separate local partitions and session revalidation before sending         | One account's queue is not submitted as another account                     |

Read the [engineering case studies](docs/engineering.en.md) for concrete failure scenarios, trade-offs, and test coverage.

## Architecture and stack

The frontend is a **static Next.js export**. NestJS owns authentication and the API on the same origin; PostgreSQL is accessed through Prisma. IndexedDB stores the local account replica and pending operations. A Service Worker caches the static application shell.

| Layer             | Technologies and responsibilities                                                      |
| ----------------- | -------------------------------------------------------------------------------------- |
| Interface         | React, Next.js, TypeScript; responsive diary and review flows                          |
| Local persistence | IndexedDB; confirmed snapshots, pending operations, and account isolation              |
| PWA               | Manifest and Service Worker; installable shell and offline startup after preparation   |
| API               | NestJS; session verification, authorization, validation, and synchronization endpoints |
| Contracts         | Shared Zod schemas and TypeScript types for frontend and backend                       |
| Data              | Prisma, PostgreSQL, versioned SQL migrations, transactions, and row-level security     |
| Identity          | Google OAuth through Supabase Auth, mediated by the backend                            |
| Quality           | Unit tests, HTTP integration tests, SQL tests, and Playwright                          |
| Delivery          | Bun workspace, GitHub Actions; deployment configuration for Vercel and Supabase        |

See the [architecture diagram and synchronization flow](docs/architecture.en.md).

## Testing and delivery

The private application includes tests for response loss and replay, account switching, expired sessions, goal and food-reference conflicts, IndexedDB write failures, concurrent tabs, and stale bootstrap responses.

HTTP integration tests run against the NestJS API and an isolated PostgreSQL database with a local Auth fixture. Playwright exercises the exported frontend and API, with desktop, mobile, and WebKit mobile configurations. Mobile browser emulation does not establish physical iPhone coverage.

GitHub Actions applies different checks by pull request target: `staging` runs lint, types, unit tests, and build; `main` adds database/HTTP integration tests and the Playwright matrix. These describe the configured validation strategy, not a live CI result or a coverage percentage.

## AI-assisted development

For this project, I used only Codex as a development assistant for planning, implementation, debugging, and code review. The project also contains an agent orchestration workflow with scoped task briefs, file ownership, model selection, and verification requirements.

My responsibility is to define the product behavior, evaluate proposed changes, and validate the result against the application's contracts and tests. AI-generated changes go through the same review and verification requirements as other changes.

AI assistance is part of the development workflow. The application's documented features do not include model-based nutrition analysis or AI-generated dietary recommendations.

## Offline scope and trade-offs

- Initial sign-in and account preparation require a connection. After preparation, the local diary and synchronized food catalog can be used offline.
- Supported individual actions use the persistent queue. Batch confirmation, household-measure registration, meal reuse, favorite-meal management, and deletion/undo require a connection and prior synchronization of pending individual actions.
- Synchronization runs in the foreground after session verification. Closed-window background synchronization is not guaranteed.
- Pending data depends on browser storage; clearing or eviction can remove it.

## About this repository

This is a documentation and screenshot showcase, not a runnable application distribution. It contains no application source, credentials, private account records, or original Git history. Deployment configuration in the private project is not presented here as a verified public demo.

Created by [Gustavo Santana](https://github.com/gustavo867).
