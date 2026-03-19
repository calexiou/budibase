# Architecture

**Analysis Date:** 2026-03-19

## Pattern Overview

**Overall:** Monorepo with layered architecture (Presentation → API → SDK → Backend-Core) + Event-driven automation system

**Key Characteristics:**
- Lerna monorepo with 15 packages using `@budibase/` scoped imports
- Clear separation between backend services (Node.js), frontend UIs (Svelte), and shared utilities
- Endpoint/EndpointGroup abstraction for declarative route registration
- SDK layer provides domain-specific operations (tables, rows, queries, automations, etc.)
- Event-driven automation system with job queue (Bull)
- Tenancy and multi-workspace support at backend-core level

## Layers

**Frontend Layer:**
- Purpose: User interfaces for building apps and running them
- Location: `packages/builder/` (Vite/Svelte), `packages/client/` (app runtime), `packages/frontend-core/` (shared frontend utilities)
- Contains: Svelte components, stores (Svelte), pages, helpers
- Depends on: `@budibase/frontend-core`, `@budibase/shared-core`, `@budibase/types`
- Used by: End users and app builders

**API Layer:**
- Purpose: HTTP route handlers and request/response validation
- Location: `packages/server/src/api/` (routes and controllers), `packages/worker/src/api/`
- Contains: Route definitions (via EndpointGroup), controllers/handlers, middleware
- Depends on: SDK layer, backend-core, services
- Used by: Frontend and public API clients

**SDK Layer:**
- Purpose: Domain-specific operations (tables, rows, datasources, automations, workspaces, etc.)
- Location: `packages/server/src/sdk/`
- Contains: `workspace/` (tables, rows, queries, automations), `users/`, `dev/`, `plugins/`
- Depends on: Backend-core, database layer, utilities
- Used by: Controllers, automations, events

**Service Layer:**
- Purpose: Cross-cutting domain logic
- Location: `packages/server/src/services/`
- Contains: Business logic that doesn't fit controller pattern
- Depends on: SDK, backend-core
- Used by: Controllers and SDK modules

**Database Layer:**
- Purpose: Database abstraction and persistence
- Location: `packages/backend-core/src/db/`, `packages/server/src/db/`
- Contains: CouchDB wrappers, migrations, query builders
- Depends on: Backend-core infrastructure (cache, tenancy, context)
- Used by: SDK, services, controllers

**Backend-Core Layer:**
- Purpose: Shared infrastructure and utilities for all backend packages
- Location: `packages/backend-core/src/`
- Contains: Auth, cache, queues, events, tenancy, logging, middleware, context
- Depends on: External services (Redis, CouchDB)
- Used by: All backend packages

**Shared Utilities:**
- Purpose: Logic usable across backend and frontend
- Location: `packages/shared-core/` (automations, helpers, SDK docs), `packages/types/` (TypeScript types)
- Contains: Automation definitions, utility functions, type definitions
- Depends on: None (minimal external deps)
- Used by: All packages

## Data Flow

**App Rendering Request (HTTP GET):**

1. Request hits `packages/server/src/koa.ts` middleware stack
2. Auth middleware validates token (via `backend-core/auth`)
3. Tenancy middleware determines workspace (via `backend-core/context`)
4. Route resolves via Endpoint registry → controller
5. Controller calls SDK methods (e.g., `sdk.tables.getTable()`)
6. SDK queries database (CouchDB) → cached in Redis
7. Response serialized and sent via HTTP

**Data Mutation (POST/PUT):**

1. Request hits Koa middleware → validation via Joi/Zod
2. Controller calls SDK method with change (e.g., `sdk.rows.save()`)
3. SDK validates, updates database (with event logging)
4. Event emitted to `backend-core/events` system
5. Event triggers automation workflows (if configured)
6. Websockets notify connected clients of change
7. Response returned with updated data

**Automation Execution:**

1. User creates automation in builder UI
2. Automation definition stored in CouchDB (document type `automation`)
3. Trigger event (row created, schedule fires, etc.) emitted to event system
4. `packages/worker/src/` processes event → queues job in Bull queue
5. JS execution via `packages/server/src/jsRunner/` (isolated-vm)
6. Automation steps execute in sequence (queries, scripts, webhooks)
7. Results logged, notifications sent via events

**State Management (Frontend):**

1. Builder stores defined in `packages/builder/src/stores/builder/`
2. Portal stores defined in `packages/builder/src/stores/portal/`
3. Stores fetch data via `@budibase/frontend-core/api` client
4. Subscriptions to websockets update stores in real-time
5. Components reactively render based on store state

## Key Abstractions

**Endpoint/EndpointGroup:**
- Purpose: Declarative HTTP route registration with middleware composition
- Examples: `packages/backend-core/src/Endpoint/Endpoint.ts`, `packages/backend-core/src/Endpoint/EndpointGroup.ts`
- Pattern: Routes define themselves in `packages/server/src/api/routes/[domain].ts`, auto-register into groups (builder-only, authenticated, public)

**SDK Modules:**
- Purpose: Provide domain-specific facades for complex operations
- Examples: `packages/server/src/sdk/workspace/tables/`, `packages/server/src/sdk/workspace/rows/`
- Pattern: Each SDK module (tables, rows, queries) exports CRUD + domain-specific operations

**Context & Tenancy:**
- Purpose: Request-scoped access to workspace/user/app data
- Examples: `packages/backend-core/src/context/`, `packages/backend-core/src/tenancy/`
- Pattern: Async local storage tracks current workspace/app during request lifecycle

**Event System:**
- Purpose: Decouple state changes from side effects (automations, webhooks, notifications)
- Examples: `packages/backend-core/src/events/`, `packages/server/src/events/`
- Pattern: Domain logic emits events → subscribers consume in worker process

**Integration Classes:**
- Purpose: Database driver abstraction layer
- Examples: `packages/server/src/integrations/postgres.ts`, `packages/server/src/integrations/mongodb.ts`
- Pattern: Each datasource type extends BaseIntegration class, implements `query()`, `create()`, etc.

## Entry Points

**Server (API):**
- Location: `packages/server/src/index.ts`
- Triggers: Application startup (`node dist/index.js`)
- Responsibilities: Initializes database, starts Koa HTTP server on port 4001, registers all routes

**Worker (Background Jobs):**
- Location: `packages/worker/src/index.ts`
- Triggers: Application startup (separate process)
- Responsibilities: Initializes Bull queue, listens for automation events, executes jobs on port 4002

**Builder (Frontend):**
- Location: `packages/builder/src/main.js`
- Triggers: Browser loads app
- Responsibilities: Mounts Svelte App component, initializes router, loads initial stores

**Client (App Runtime):**
- Location: `packages/client/src/index.ts`
- Triggers: Published app loads in user browser
- Responsibilities: Initializes component tree, manages data bindings, handles user interactions

## Error Handling

**Strategy:** Try/catch at controller level, HTTPError exceptions for API responses

**Patterns:**
- Controllers wrap SDK calls in try/catch, catch exceptions become HTTP error responses
- `backend-core/errors` provides HTTPError class with status codes
- Validation errors (Joi/Zod) caught by middleware, return 400
- Uncaught exceptions logged via `backend-core/logging`, process exits gracefully (worker/server handle via `http-graceful-shutdown`)

## Cross-Cutting Concerns

**Logging:** Console.log statements redirected to pino (backend-core/logging) in production
**Validation:** Joi schemas for request bodies (handled by middleware), Zod for internal validation
**Authentication:** Passport.js + JWT tokens via `backend-core/auth`
**Tenancy:** Async local storage tracks workspace/app context (`backend-core/context`)
**Caching:** Redis-backed via `backend-core/cache` (DocWritethrough for data, session store)
**Queuing:** Bull queue for automations, Redis backend via `backend-core/queue`

---

*Architecture analysis: 2026-03-19*
