# External Integrations

**Analysis Date:** 2026-03-19

## APIs & External Services

**AI & Large Language Models:**
- OpenAI - ChatGPT and related models
  - SDK/Client: `@ai-sdk/openai` 3.0.41, `openai` 6.32.0
  - Auth: `OPENAI_API_KEY` (via LiteLLM)
  - Location: `packages/server/src/` (AI SDK integration in pro features)
  - Env vars: `LITELLM_URL`, `LITELLM_MASTER_KEY`

- Azure OpenAI - Azure-hosted OpenAI models
  - SDK/Client: `@ai-sdk/azure` 3.0.42
  - Auth: Azure credentials (via LiteLLM)
  - Location: `packages/server/src/tests/utilities/mocks/ai/`

- Anthropic Claude - Claude AI models
  - SDK/Client: `@anthropic-ai/sdk` 0.79.0
  - Location: `packages/pro/` (enterprise feature)
  - Requires: Pro/enterprise package

- LiteLLM Service - Unified LLM API proxy
  - Docker service on port 4000 (localhost:4000 in dev)
  - Env vars: `LITELLM_URL`, `LITELLM_MASTER_KEY`, `BBAI_LITELLM_KEY`
  - Config: `hosting/litellm_config.yaml`
  - Supports: OpenAI, Azure OpenAI, Anthropic, and other providers

**Chat Platforms:**
- Slack
  - SDK/Client: `@chat-adapter/slack` 4.20.0
  - Location: `packages/server/src/api/controllers/webhook/`
  - Features: Chat webhook handlers, message routing

- Discord
  - SDK/Client: `@chat-adapter/discord` 4.20.0
  - Location: `packages/server/src/api/controllers/webhook/`
  - Features: Chat webhook handlers

- Microsoft Teams
  - SDK/Client: `@chat-adapter/teams` 4.20.0
  - Location: `packages/server/src/api/controllers/webhook/`
  - Features: Chat webhook handlers

- Real-time Communication
  - Socket.IO 4.8.1 - Bidirectional WebSocket communication
  - Socket.IO Redis Adapter 8.2.1 - Clustered Socket.IO support
  - Location: `packages/server/src/api/routes/chat.ts`, `packages/frontend-core/`

**GitHub:**
- GitHub API
  - SDK/Client: `@octokit/rest` 20.0.0
  - Auth: `GITHUB_TOKEN` environment variable
  - Location: `packages/server/src/sdk/plugins/update.ts`
  - Purpose: Plugin installation and management from GitHub

**Google Services:**
- Google Sheets
  - SDK/Client: `google-spreadsheet` (@budibase/google-spreadsheet 4.1.5)
  - Auth: Google OAuth credentials
  - Location: `packages/server/src/integrations/googlesheets.ts`
  - Purpose: Query and update Google Sheets

- Google Firestore
  - SDK/Client: `@google-cloud/firestore` 7.8.0
  - Auth: Google Cloud service account credentials
  - Location: `packages/server/src/integrations/firebase.ts`
  - Purpose: NoSQL database integration

- Google Authentication
  - SDK/Client: `google-auth-library` 10.5.0, `passport-google-oauth` 2.0.0
  - Auth: Google OAuth 2.0
  - Location: `packages/backend-core/src/` (Passport strategies)
  - Purpose: SSO and user authentication

**Email & Communication:**
- Email via IMAP/SMTP
  - SDK/Client: `imapflow` 1.1.0, `nodemailer` 7.0.11
  - Location: `packages/worker/` (email automations)
  - Purpose: Email trigger detection and message sending
  - Auth: Email provider credentials (IMAP username/password, SMTP config)

- Email Parsing
  - SDK/Client: `mailparser` 3.7.2
  - Location: `packages/server/src/integrations/`
  - Purpose: Parse email content for automation inputs

**Microsoft & Enterprise:**
- Azure Active Directory
  - SDK/Client: `@azure/msal-node` 2.5.1
  - Auth: Azure AD client credentials
  - Location: `packages/backend-core/src/`
  - Purpose: Enterprise SSO and user provisioning

- Atlassian (Jira/Confluence)
  - Auth via API token
  - Env vars: `ATLASSIAN_API_TOKEN`, `ATLASSIAN_EMAIL`, `ATLASSIAN_BASE_URL`
  - Location: `packages/server/src/` (datasource integration)
  - Purpose: Query Jira/Confluence data

- BambooHR
  - Auth via API key
  - Env vars: `BAMBOOHR_API_KEY`, `BAMBOOHR_SUBDOMAIN`
  - Location: `packages/server/src/` (datasource integration)
  - Purpose: HR data integration

**Third-party SaaS:**
- Airtable
  - SDK/Client: `airtable` 0.12.2
  - Location: `packages/server/src/integrations/airtable.ts`
  - Purpose: Query and update Airtable bases

## Data Storage

**Databases:**
- CouchDB (Primary)
  - Connection: `COUCH_DB_URL` (default: http://localhost:5984)
  - Client: `@budibase/nano` 10.1.5 (wrapper around PouchDB)
  - Location: `packages/backend-core/src/db/`
  - Purpose: Main application database, app definitions, workspace data
  - Authentication: `COUCH_DB_USER`, `COUCH_DB_PASSWORD`
  - Docker service: `couchdb-service` on port 5984

- PostgreSQL
  - Client: `pg` 8.10.0
  - Connection: Configured via datasource UI
  - Query builder: `knex` 2.4.2
  - Location: `packages/server/src/integrations/postgres.ts`
  - Purpose: External SQL database integration

- MySQL
  - Client: `mysql2` 3.9.8
  - Connection: Configured via datasource UI
  - Query builder: `knex` 2.4.2
  - Location: `packages/server/src/integrations/mysql.ts`
  - Purpose: External SQL database integration

- MongoDB
  - Client: `mongodb` 6.7.0
  - Connection: Configured via datasource UI
  - Location: `packages/server/src/integrations/mongodb.ts`
  - Purpose: External MongoDB database integration

- Microsoft SQL Server
  - Client: `mssql` 11.0.1
  - Query builder: `knex` 2.4.2
  - Location: `packages/server/src/integrations/microsoftSqlServer.ts`
  - Purpose: External MSSQL database integration

- Oracle Database
  - Client: `oracledb` 6.5.1
  - Query builder: `knex` 2.4.2
  - Location: `packages/server/src/integrations/oracle.ts`
  - Purpose: External Oracle DB integration

- ArangoDB
  - Client: `arangojs` 7.2.0
  - Location: `packages/server/src/integrations/arangodb.ts`
  - Purpose: Graph database integration

- Snowflake
  - Client: `snowflake-sdk` 1.15.0
  - Location: `packages/server/src/integrations/snowflake.ts`
  - Purpose: Data warehouse integration

- Elasticsearch
  - Client: `@elastic/elasticsearch` 7.10.0
  - Location: `packages/server/src/integrations/elasticsearch.ts`
  - Purpose: Search and analytics integration

**File Storage:**
- MinIO (S3-compatible, self-hosted)
  - Connection: `MINIO_URL` (default: http://localhost:9000)
  - Credentials: `MINIO_ACCESS_KEY`, `MINIO_SECRET_KEY`
  - Docker service: `minio-service` on port 9000
  - SDK: `@aws-sdk/client-s3` 3.709.0 (used for S3-compatible access)
  - Location: `packages/server/src/` and `packages/backend-core/src/`
  - Purpose: Store app assets, file uploads, backups

- AWS S3 (for cloud deployments)
  - SDK: `@aws-sdk/*` packages (S3, DynamoDB, CloudFront)
  - Auth: AWS IAM credentials
  - Env vars: `AWS_REGION`, `AWS_SESSION_TOKEN`
  - CloudFront signing: `aws-cloudfront-sign` 3.0.2
  - Purpose: Cloud object storage and distribution

**Caching:**
- Redis
  - Connection: `REDIS_URL` (default: localhost:6379)
  - Credentials: `REDIS_PASSWORD`, `REDIS_USERNAME` (optional)
  - Clients: `redis` 4.x, `ioredis` 5.3.2
  - Docker service: `redis-service` on port 6379
  - Clustered mode: `REDIS_CLUSTERED` flag
  - Adapters: `@socket.io/redis-adapter` 8.2.1 (for Socket.IO), `@chat-adapter/state-ioredis` 4.20.0
  - Location: Used throughout `packages/backend-core/` and `packages/server/src/`
  - Purpose: Session storage, job queue, real-time state, caching

## Authentication & Identity

**Auth Provider:**
- Custom/Internal
  - Implementation: Passport-based authentication
  - Location: `packages/backend-core/src/` (auth utilities and strategies)
  - Supports multiple strategies:
    - Local (username/password)
    - Google OAuth
    - Azure AD / OpenID Connect
    - SAML (via OpenID Connect)
  - Session management via cookies and JWT

**JWT & Tokens:**
- JSON Web Tokens
  - Library: `jsonwebtoken` 9.0.2
  - Secret: `JWT_SECRET` environment variable
  - Password hashing: `bcrypt` 6.0.0
  - Location: `packages/backend-core/src/auth/`

**OAuth2 Management:**
- OAuth2 configuration and CRUD
  - Location: `packages/server/src/sdk/workspace/oauth2/`
  - Files: `packages/server/src/sdk/workspace/oauth2/crud.ts`
  - Purpose: Store and manage OAuth2 provider configurations for external integrations

**OpenID Connect:**
- Generic OpenID Connect support
  - Library: `@govtechsg/passport-openidconnect` 1.0.3
  - Location: `packages/backend-core/src/`
  - Purpose: Support enterprise SSO via any OIDC provider

## Monitoring & Observability

**Error Tracking:**
- Datadog APM
  - Library: `dd-trace` 5.63.0
  - Configuration: Environment variables (Datadog API keys)
  - Location: `packages/server/src/`, `packages/worker/`, `packages/backend-core/src/`
  - Purpose: Application performance monitoring and error tracking

**Logs:**
- Pino (structured logging)
  - Libraries: `pino` 8.11.0, `pino-http` 8.3.3, `koa-pino-logger` 4.0.0
  - Location: `packages/backend-core/src/middleware/`
  - Console.log is redirected to Pino in production
  - Log level: `LOG_LEVEL` env var (default: info)

- File-based log rotation
  - Library: `rotating-file-stream` 3.1.0
  - Purpose: Rotate logs to prevent disk overflow

**Analytics:**
- PostHog
  - Libraries: `posthog-js` 1.118.0 (frontend), `posthog-node` 4.0.1 (backend)
  - Location: `packages/builder/` (frontend analytics), backend components
  - Purpose: Product analytics and user behavior tracking

## CI/CD & Deployment

**Hosting:**
- Self-hosted: Docker container deployment
- Cloud: Account portal (https://budibase.com or custom domain)
- Docker images provided for:
  - App server (`budibase/apps`)
  - Worker (`budibase/worker`)
  - Proxy (`budibase/proxy`)
  - Database (`budibase/database` - CouchDB variant)
  - Dependencies (`budibase/dependencies`)

**CI Pipeline:**
- GitHub Actions (inferred from `@octokit/rest` usage)
- Lerna-managed monorepo builds
- Yarn workspaces for dependency management

**Build Artifacts:**
- Docker Compose files: `hosting/docker-compose.yaml`, `hosting/docker-compose.dev.yaml`
- Nginx configuration: `hosting/nginx.dev.conf`
- Single-image deployment: `hosting/single/Dockerfile`

## Environment Configuration

**Required env vars (core infrastructure):**
- `COUCH_DB_URL` - CouchDB connection (default: http://couchdb-service:5984)
- `COUCH_DB_USER`, `COUCH_DB_PASSWORD` - CouchDB credentials
- `REDIS_URL` - Redis connection (default: redis-service:6379)
- `REDIS_PASSWORD`, `REDIS_USERNAME` - Redis credentials
- `MINIO_URL` - MinIO object storage (default: http://minio-service:9000)
- `MINIO_ACCESS_KEY`, `MINIO_SECRET_KEY` - MinIO credentials
- `WORKER_URL` - Background worker service URL
- `JWT_SECRET` - Secret key for JWT signing
- `API_ENCRYPTION_KEY` - Encryption key for API credentials

**AI/LLM env vars:**
- `LITELLM_URL` - LiteLLM service endpoint
- `LITELLM_MASTER_KEY` - LiteLLM authentication key
- `BBAI_LITELLM_KEY` - Budibase AI specific LiteLLM key
- Model-specific keys configured via LiteLLM config file

**SSO & Identity env vars:**
- `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET` - Google OAuth
- Cloud deployment sets additional OAuth variables

**Third-party integrations:**
- `GITHUB_TOKEN` - GitHub API access
- `ATLASSIAN_API_TOKEN`, `ATLASSIAN_EMAIL`, `ATLASSIAN_BASE_URL` - Jira/Confluence
- `BAMBOOHR_API_KEY`, `BAMBOOHR_SUBDOMAIN` - BambooHR
- Database-specific credentials (Snowflake, etc.) - Stored per datasource config

**Feature flags & modes:**
- `SELF_HOSTED` - Deployment mode
- `MULTI_TENANCY` - Enable multi-tenant support
- `BUDIBASE_ENVIRONMENT` - dev, staging, production
- `ENABLE_ANALYTICS` - Enable PostHog analytics
- `DISABLE_ACCOUNT_PORTAL` - Disable account portal
- `OFFLINE_MODE` - Run without external services

**Secrets location:**
- Development: `.env` file at project root (git-ignored)
- Cloud deployment: Environment variables or secrets manager
- Docker: Environment variables or `.env` files mounted in containers
- Production: Use cloud provider secrets management (AWS Secrets Manager, Azure Key Vault, etc.)

## Webhooks & Callbacks

**Incoming:**
- Automation webhooks: `packages/server/src/api/routes/webhook.ts`
  - Webhook URL format: `/api/v1/webhooks/automationId`
  - Supports: HTTP POST payloads, JSON body parsing
  - Authentication: Webhook token-based

- Chat webhooks: `packages/server/src/api/controllers/webhook/chatHandler.ts`
  - Handles incoming messages from Slack, Discord, Teams
  - Routes to chat apps via Socket.IO
  - State management: Redis or in-memory

**Outgoing:**
- Automation triggers to external services via REST queries
  - Location: `packages/server/src/api/controllers/query/`
  - Supports: HTTP GET, POST, PUT, DELETE, PATCH
  - Integration: Custom REST connector

- Email notifications via SMTP
  - Library: `nodemailer` 7.0.11
  - Location: `packages/worker/src/` (email automations)
  - Supported providers: SMTP, Gmail, Outlook, etc.

- Slack notifications
  - Via Slack webhook URLs (configured per datasource)
  - Chat adapter integration for two-way chat

- Chat app message routing
  - Via Socket.IO to chat platform handlers
  - Supported: Slack, Discord, Teams

---

*Integration audit: 2026-03-19*
