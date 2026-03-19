# Codebase Structure

**Analysis Date:** 2026-03-19

## Directory Layout

```
budibase/
├── packages/                    # Lerna monorepo workspaces
│   ├── server/                 # Main API server (Node.js, Koa)
│   ├── worker/                 # Background job processor
│   ├── builder/                # Builder UI (Svelte/Vite)
│   ├── client/                 # Published app runtime
│   ├── backend-core/           # Shared backend infrastructure
│   ├── frontend-core/          # Shared frontend utilities
│   ├── shared-core/            # Shared cross-platform logic
│   ├── types/                  # TypeScript types
│   ├── pro/                    # Pro features (licensing, audit)
│   ├── bbui/                   # Budibase UI component library
│   ├── sdk/                    # JavaScript SDK for external integrations
│   ├── cli/                    # Command-line interface
│   ├── string-templates/       # String interpolation engine
│   └── upgrade-tests/          # Version upgrade test suite
├── hosting/                     # Deployment configs
│   ├── single/                 # Single container deploy
│   ├── couchdb/                # Database docker setup
│   └── scripts/                # Deployment automation
├── scripts/                     # Build and dev scripts
│   └── dev/                    # Development setup (Docker, env)
├── charts/                      # Kubernetes Helm charts
├── i18n/                        # Internationalization files
├── docs/                        # Documentation
├── eslint-local-rules/          # Custom eslint rules
└── .planning/                   # GSD analysis documents
```

## Directory Purposes

**packages/server:**
- Purpose: Main HTTP API server for builder and app functionality
- Contains: Route handlers, controllers, SDK, integrations, automations
- Key files: `src/index.ts` (entry), `src/app.ts` (Koa init), `src/koa.ts` (middleware setup)
- Runs on: Port 4001

**packages/worker:**
- Purpose: Background job processor for automations and async tasks
- Contains: Automation queue processing, event subscriptions, job handlers
- Key files: `src/index.ts` (entry), `src/api/` (health endpoints)
- Runs on: Port 4002

**packages/builder:**
- Purpose: Low-code builder UI for creating applications
- Contains: Svelte components, stores, pages, design system integration
- Key files: `src/main.js` (entry), `src/App.svelte` (root), `src/stores/builder/` (state)
- Runs on: Port 3000 (dev), compiled into `server/dist/client/`

**packages/client:**
- Purpose: Published app runtime - renders user-built apps in browsers
- Contains: Component renderer, data binding engine, event handlers
- Key files: `src/index.ts` (entry), `src/components/` (component implementations)
- Output: Bundled into `server/dist/client/`

**packages/backend-core:**
- Purpose: Shared infrastructure for all backend services
- Contains: Auth, context/tenancy, cache, queue, events, logging, middleware
- Key files: `src/index.ts` (barrel export), `src/Endpoint/` (route abstraction)
- Used by: server, worker, cli, pro

**packages/frontend-core:**
- Purpose: Shared utilities and components for all frontend packages
- Contains: API client, stores, theme system, fetch utilities, icons
- Key files: `src/index.ts`, `src/api/` (HTTP client)
- Used by: builder, client

**packages/shared-core:**
- Purpose: Cross-platform utilities usable in Node.js and browsers
- Contains: Automation definitions, helper utilities, SDK documentation
- Key files: `src/automations/` (automation step/trigger definitions), `src/helpers/`
- Used by: server, worker, builder, client

**packages/types:**
- Purpose: Centralized TypeScript type definitions
- Contains: All domain types (Table, Row, User, Automation, etc.)
- Key files: `src/` (organized by domain)
- Used by: All packages

**packages/pro:**
- Purpose: Enterprise/Pro features (licensing, audit logs, SSO)
- Contains: Middleware for licensing enforcement, audit event processors
- Key files: `src/` (organized by feature area)
- Used by: server, worker, backend-core

**packages/bbui:**
- Purpose: Budibase UI component library (buttons, modals, inputs, etc.)
- Contains: Svelte components with accessibility and theming
- Key files: `src/` (component definitions)
- Used by: builder, frontend-core

**packages/sdk:**
- Purpose: JavaScript SDK for external integrations with Budibase
- Contains: Client library for programmatic API access
- Key files: `src/` (public API surface)
- Used by: External developers

**packages/string-templates:**
- Purpose: Handlebars-based template engine for string interpolation
- Contains: Template parsing and compilation
- Key files: `src/` (template engine)
- Used by: server (automations, workflows), client (data bindings)

## Key File Locations

**Entry Points:**

- `packages/server/src/index.ts` - Initializes development environment, requires app.ts
- `packages/server/src/app.ts` - Initializes database, creates Koa app, calls startup
- `packages/server/src/startup/index.ts` - Initialization of features, queues, websockets
- `packages/worker/src/index.ts` - Initializes job processing and event listeners
- `packages/builder/src/main.js` - Mounts root Svelte app to DOM
- `packages/client/src/index.ts` - Initializes app component renderer

**Configuration:**

- `packages/server/src/environment.ts` - Server environment variables
- `packages/worker/src/environment.ts` - Worker environment variables
- `packages/backend-core/src/environment.ts` - Shared core env vars
- `tsconfig.build.json` - Root TypeScript build configuration
- `lerna.json` - Monorepo workspace configuration
- `.prettierrc.json` - Code formatting rules
- `eslint.config.mjs` - Linting configuration

**Core Logic:**

- `packages/server/src/sdk/` - All domain operations (tables, rows, queries, automations)
- `packages/server/src/api/` - HTTP routes and controllers
- `packages/server/src/integrations/` - Database driver implementations
- `packages/server/src/automations/` - Automation execution engine
- `packages/backend-core/src/db/` - Database abstraction layer
- `packages/backend-core/src/auth/` - Authentication and authorization
- `packages/backend-core/src/context/` - Request-scoped context management

**Testing:**

- `packages/server/src/tests/` - Test fixtures and utilities
- `packages/server/src/tests/TestConfiguration.ts` - Main test setup class
- `packages/server/src/tests/utilities/api.ts` - API request builders
- `packages/server/src/tests/utilities/structures.ts` - Domain object factories
- `packages/server/src/automations/tests/utilities/AutomationTestBuilder.ts` - Automation test helpers
- `packages/backend-core/tests/` - Backend-core integration tests

## Naming Conventions

**Files:**

- `.ts` - TypeScript source files
- `.svelte` - Svelte components
- `.test.ts`, `.spec.ts` - Jest test files
- `index.ts` - Barrel export files (export all from directory)
- Controllers: `[domain].ts` (e.g., `table.ts`, `row.ts`)
- Routes: `[domain].ts` in `api/routes/`

**Directories:**

- `src/` - Source code
- `src/api/routes/` - Route definitions
- `src/api/controllers/` - Request handlers
- `src/sdk/workspace/` - Domain operations
- `src/integrations/` - External data source drivers
- `src/middleware/` - Koa middleware
- `src/stores/` - Svelte stores (state)
- `src/components/` - Svelte/UI components
- `__tests__/` or `tests/` - Test files (co-located or separate)
- `__mocks__/` - Jest mock files
- `dist/` - Compiled output (not committed)

## Where to Add New Code

**New API Endpoint:**

1. Create route in `packages/server/src/api/routes/[domain].ts`
2. Create controller in `packages/server/src/api/controllers/[domain].ts` or subdirectory
3. Use EndpointGroup pattern: `routes.get("/api/...", controllerFn)`
4. Add to `packages/server/src/api/routes/index.ts` imports
5. Test in `packages/server/src/api/routes/tests/[domain].test.ts`

**New SDK Operation:**

1. Create module in `packages/server/src/sdk/workspace/[domain]/index.ts`
2. Export public functions (getters, CRUD, domain-specific)
3. Import in `packages/server/src/sdk/index.ts` barrel export
4. Use in controllers: `sdk.[domain].[operation]()`
5. Test in `packages/server/src/sdk/tests/[domain].spec.ts`

**New Domain Type:**

1. Define in `packages/types/src/[domain]/` directory
2. Export from `packages/types/src/index.ts`
3. Import as `import { TypeName } from "@budibase/types"`
4. Use in controllers, SDK, and frontend code

**New Automation Step/Trigger:**

1. Define in `packages/shared-core/src/automations/steps/` or `triggers/`
2. Export from `packages/shared-core/src/automations/index.ts`
3. Implement handler in `packages/server/src/automations/steps.ts` or `triggers.ts`
4. Test in `packages/server/src/automations/tests/`

**New Frontend Component:**

1. Create in `packages/builder/src/components/[category]/ComponentName.svelte`
2. Export from category index if needed
3. Use in pages or other components
4. Test in `packages/builder/src/test/` or co-located `.spec.ts`

**New Shared Store (Builder):**

1. Create in `packages/builder/src/stores/builder/[domain].ts` (builder features) or `packages/builder/src/stores/portal/` (workspace/portal features)
2. Export from `packages/builder/src/stores/builder/index.ts`
3. Use in components: `import { storeVar } from "stores"`

**New Middleware:**

1. Create in `packages/server/src/middleware/[name].ts`
2. Use in `packages/server/src/api/index.ts` via `router.use(middleware.[name])`
3. Or add to route-specific EndpointGroup if only for one domain

**New Integration (Database Driver):**

1. Create in `packages/server/src/integrations/[dbname].ts`
2. Extend BaseIntegration class from `packages/server/src/integrations/base/`
3. Register in `packages/server/src/integrations/index.ts`
4. Add tests in `packages/server/src/integrations/tests/`

## Special Directories

**packages/server/build/:**
- Purpose: Build artifacts and output (compiled JavaScript)
- Generated: Yes (by `yarn build`)
- Committed: No

**packages/server/dist/:**
- Purpose: Compiled output from TypeScript
- Generated: Yes (by build process)
- Committed: No
- Contains: Compiled backend-core, SDK, API code; compiled builder (client/)

**packages/server/client/:**
- Purpose: Compiled builder and client apps (copied from builder/client packages)
- Generated: Yes (postbuild step)
- Committed: No
- Generated from: `packages/builder/dist/` and `packages/client/dist/`

**packages/pro/**:
- Purpose: Enterprise feature implementations
- Committed: Yes (part of monorepo)
- Loaded conditionally: Via `packages/server/src/initPro.ts` check

**hosting/:**
- Purpose: Docker and deployment configurations
- Contains: Dockerfile variants, docker-compose, Kubernetes charts
- Committed: Yes
- Key: `hosting/single/Dockerfile` (single image build)

**.planning/codebase/:**
- Purpose: GSD analysis documents (ARCHITECTURE.md, CONVENTIONS.md, etc.)
- Generated: Yes (by `/gsd:map-codebase`)
- Committed: Yes
- Used by: `/gsd:plan-phase`, `/gsd:execute-phase`

---

*Structure analysis: 2026-03-19*
