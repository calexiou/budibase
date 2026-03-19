# Technology Stack

**Analysis Date:** 2026-03-19

## Languages

**Primary:**
- TypeScript 5.9.2 - Backend (server, worker, backend-core), frontend (builder, client, frontend-core, bbui), and shared packages
- JavaScript - Bundle outputs and legacy code
- Svelte 5.40.2 - Frontend UI components and pages

**Secondary:**
- YAML - Configuration files (docker-compose, nginx, letsencrypt)
- JSON - Configuration and build manifests

## Runtime

**Environment:**
- Node.js 22.x (>= 22.0.0 < 23.0.0) - Required for all backend services
- Browser (modern ES6+) - Frontend client-side execution

**Package Manager:**
- Yarn (Yarn Berry with workspaces)
- Lockfile: `yarn.lock` present at root

## Frameworks

**Backend:**
- Koa 3.1.2 - HTTP server framework (server, worker, backend-core)
- Koa Router 15.3.0 - URL routing for API endpoints
- Bull 4.10.1 - Job queue for background tasks and automations
- PouchDB 9.0.0 - NoSQL database client (CouchDB and local adapters)

**Frontend:**
- Vite 7.1.11 - Build tool and dev server (builder, client, bbui)
- Routify 2.18.18 - File-based routing (builder only)
- CodeMirror 5.65.16 - Code editor component (builder)
- ApexCharts 3.48.0 - Charting library (client)
- Socket.IO 4.8.1 - Real-time bidirectional communication (server and frontend-core)

**UI Component Libraries:**
- Adobe Spectrum CSS - Design system components across packages
- BBUI (@budibase/bbui) - Custom Budibase component library (Svelte-based)
- Svelte Spectrum CSS - Wrapped Spectrum components

**Testing:**
- Jest 30.0.5 - Unit and integration tests (root devDependency)
- Vitest 3.2.4 - Alternative test runner for frontend packages
- Supertest 6.3.3 - HTTP assertion library for API tests
- Nock 13.5.4 - HTTP mocking for tests
- Testcontainers 11.7.2 - Docker container management for test databases

**Build/Dev Tools:**
- ESBuild 0.18.17 - Fast JavaScript bundler
- SWC 1.13.5 - Fast TypeScript/JavaScript compiler alternative to Babel
- Lerna 9.0.3 - Monorepo management (root-level)
- NX (Nx) - Task orchestration for monorepo (via lerna integration)
- Rollup - Module bundler for certain packages
- Prettier 3.5.3 - Code formatter (no semicolons, 2-space tabs, double quotes)
- ESLint 9.26.0 - Linting and code quality
- Babel 7.22.5 - JavaScript transpilation for compatibility

## Key Dependencies

**Critical:**
- `@budibase/nano` 10.1.5 - CouchDB client wrapper
- `@budibase/backend-core` * - Shared backend utilities (types, auth, storage, caching)
- `@budibase/shared-core` * - Shared data structures for frontend and backend
- `@budibase/types` * - TypeScript type definitions for entire platform
- `@budibase/string-templates` * - Expression and templating engine
- `@budibase/frontend-core` * - Shared frontend utilities
- `@budibase/bbui` * - Reusable UI component library

**Databases & Storage:**
- `pg` 8.10.0 - PostgreSQL client
- `mysql2` 3.9.8 - MySQL client
- `mongodb` 6.7.0 - MongoDB client
- `mssql` 11.0.1 - Microsoft SQL Server client
- `oracledb` 6.5.1 - Oracle Database client
- `arangojs` 7.2.0 - ArangoDB client
- `snowflake-sdk` 1.15.0 - Snowflake data warehouse client
- `@elastic/elasticsearch` 7.10.0 - Elasticsearch client
- `@google-cloud/firestore` 7.8.0 - Google Firestore client
- `redis` 4.x - Redis client
- `ioredis` 5.3.2 - Alternative Redis client with better API
- `airtable` 0.12.2 - Airtable API client

**Cloud & Infrastructure:**
- `@aws-sdk/*` - AWS SDK packages (S3, DynamoDB, CloudFront, etc.)
- `aws-sdk` 2.1692.0 - Older AWS SDK version (still used for some services)
- `aws-cloudfront-sign` 3.0.2 - CloudFront URL signing
- `@smithy/node-http-handler` 4.1.1 - HTTP handler for AWS SDK v3
- MinIO (Docker) - S3-compatible object storage

**AI & LLM:**
- `ai` 6.0.116 - Vercel AI SDK (both backend and frontend-core)
- `@ai-sdk/openai` 3.0.41 - OpenAI provider for AI SDK
- `@ai-sdk/azure` 3.0.42 - Azure OpenAI provider
- `@ai-sdk/provider` 3.0.8 - AI SDK provider interface
- `@ai-sdk/svelte` 4.0.116 - Svelte bindings for AI SDK
- `openai` 6.32.0 - Direct OpenAI client
- `@anthropic-ai/sdk` 0.79.0 - Anthropic Claude client (in @budibase/pro)
- LiteLLM (Docker service) - Unified LLM API proxy supporting multiple providers

**Chat & Communication:**
- `@chat-adapter/slack` 4.20.0 - Slack chat integration
- `@chat-adapter/discord` 4.20.0 - Discord chat integration
- `@chat-adapter/teams` 4.20.0 - Microsoft Teams chat integration
- `@chat-adapter/state-ioredis` 4.20.0 - Redis state store for chat
- `@chat-adapter/state-memory` 4.20.0 - In-memory state store for chat
- `socket.io-client` 4.7.5 - Real-time client for frontend
- `@socket.io/redis-adapter` 8.2.1 - Redis adapter for Socket.IO
- `nodemailer` 7.0.11 - Email sending (worker)

**Authentication & Identity:**
- `jsonwebtoken` 9.0.2 - JWT creation and verification
- `bcrypt` 6.0.0 - Password hashing
- `koa-passport` 4.1.4/6.0.0 - Koa authentication middleware
- `passport-local` 1.0.0 - Username/password strategy
- `passport-google-oauth` 2.0.0 - Google OAuth strategy
- `@azure/msal-node` 2.5.1 - Azure AD authentication
- `@govtechsg/passport-openidconnect` 1.0.3 - OpenID Connect strategy

**External APIs & Services:**
- `@octokit/rest` 20.0.0 - GitHub API client (plugin management)
- `google-spreadsheet` (@budibase/google-spreadsheet 4.1.5) - Google Sheets API
- `google-auth-library` 10.5.0 - Google authentication
- `imapflow` 1.1.0 - IMAP email client for email automations
- `mailparser` 3.7.2 - Email parsing
- `ical-generator` 4.1.0 - iCalendar event generation
- `pdf-parse` 2.4.5 - PDF text extraction
- `jimp` 1.1.4 - Image manipulation
- `curlconverter` 3.21.0 - cURL to request conversion

**Data Processing & Utilities:**
- `knex` 2.4.2 - SQL query builder (multiple database support)
- `csvtojson` 2.0.13 - CSV parsing and conversion
- `joi` 17.6.0 - Data validation schemas
- `jsonschema` 1.4.0 - JSON Schema validation
- `bson` 6.9.0 - BSON serialization
- `xml2js` 0.6.2 - XML parsing
- `js-yaml` 4.1.1 - YAML parsing and generation
- `isolated-vm` 6.0.1 - Sandboxed JavaScript execution
- `lodash` 4.17.23 - Utility functions
- `dayjs` 1.10.8 - Date/time manipulation
- `uuid` 8.3.2 - UUID generation
- `archiver` 7.0.1 - ZIP and archive creation
- `extract-zip` 2.0.1 - ZIP extraction
- `tar` 7.5.11 - TAR archive handling

**Observability & Monitoring:**
- `dd-trace` 5.63.0 - Datadog APM integration
- `pino` 8.11.0 - Logging framework (backend-core)
- `pino-http` 8.3.3 - Pino HTTP middleware
- `pino-pretty` 10.0.0 - Pretty-print Pino logs (dev)
- `posthog-js` 1.118.0 - Analytics in builder frontend
- `posthog-node` 4.0.1 - Analytics in backend

**Security:**
- `sanitize-html` 2.13.0 - HTML sanitization
- `global-agent` 3.0.0 - Global HTTP/HTTPS agent for proxy support

## Configuration

**Environment:**
- `.env` file at project root (loaded by dotenv in development)
- Environment variables configured per deployment mode (cloud, self-hosted, account)
- Key configs: `COUCH_DB_URL`, `REDIS_URL`, `MINIO_URL`, `WORKER_URL`, `LITELLM_URL`
- Feature flags and mode toggles via env vars (MULTI_TENANCY, SELF_HOSTED, BUDIBASE_ENVIRONMENT, etc.)

**Build:**
- `tsconfig.build.json` - Root TypeScript build configuration (ES6 target, commonjs module, strict mode)
- `tsconfig.json` - Project-specific TypeScript configs in each package
- `babel.config.json` - Babel transpilation config
- `vite.config.ts` - Vite build configuration (frontend packages)
- `rollup.config.js` - Rollup bundling config (specific packages)

**Linting & Formatting:**
- `.prettierrc.json` - Prettier config (2 spaces, no semicolons, double quotes)
- ESLint config - Integrated with Prettier, enforced via `yarn lint`
- `husky` - Git hooks (pre-commit, pre-push)

## Platform Requirements

**Development:**
- Node.js 22.x (required, enforced in package.json `engines` field)
- Docker & Docker Compose - For local infrastructure (CouchDB, Redis, MinIO, nginx)
- Git with LFS (Large File Storage) - Enforced by husky hooks

**Production:**
- Node.js 22.x runtime
- Docker container runtime
- External services:
  - CouchDB (database)
  - Redis (caching, queues, sessions)
  - MinIO or AWS S3 (object storage)
  - Optional: PostgreSQL, MySQL, MongoDB, etc. (for external datasources)
  - Optional: LiteLLM service (for unified LLM access)
  - Optional: Elasticsearch (for search functionality)

**Infrastructure Components (Docker Compose):**
- CouchDB service - Primary database
- Redis service - Cache, queues, sessions
- MinIO service - S3-compatible object storage
- Nginx proxy - Reverse proxy and load balancing
- LiteLLM service - LLM provider abstraction layer
- App service (Server) - REST API server on port 4002
- Worker service - Background job processing on port 4003

---

*Stack analysis: 2026-03-19*
