# Codebase Concerns

**Analysis Date:** 2026-03-19

## Tech Debt

**Loose Typing with `as any` Casts:**
- Issue: 319+ instances of `as any` type casting throughout the codebase bypassing TypeScript's type system. Common in data handling, API responses, and grid components.
- Files: `packages/frontend-core/src/components/grid/stores/`, `packages/client/src/`, `packages/server/src/automations/steps/`, `packages/string-templates/src/helpers/javascript.ts` (line 67-68), `packages/backend-core/src/sql/sqlTable.ts`
- Impact: Reduces type safety, makes refactoring risky, allows bugs to slip through. Particularly dangerous in critical paths like search filters and data validation.
- Fix approach: Gradually replace `as any` with proper union types and generics. Start with high-impact files like grid stores and validation. Use `@ts-expect-error` with comments only as interim solution.

**Bulk User Insert Loops:**
- Issue: At `packages/backend-core/src/users/db.ts:412`, bulk user creation loops through individual platform user additions instead of batching. Each user triggers separate database write.
- Files: `packages/backend-core/src/users/db.ts` (lines 410-416)
- Impact: Performance degradation with large user imports. N+1 pattern creates unnecessary database round-trips.
- Fix approach: Implement bulk insert into info database. Group `platform.users.addUser()` calls into single batched operation.

**SQL Datastore-Specific Null Handling:**
- Issue: At `packages/backend-core/src/sql/sql.ts:1311`, sort operations have conditional logic for null handling that relies on database-specific defaults (PostgreSQL vs Oracle vs others).
- Files: `packages/backend-core/src/sql/sql.ts` (lines 1311-1320)
- Impact: Fragile code that breaks when switching databases. Behavior inconsistency across platforms.
- Fix approach: Abstract null ordering into a database-agnostic layer or establish explicit defaults for all supported databases.

**User Search Limitations:**
- Issue: At `packages/worker/src/api/controllers/global/users.ts:308`, only two search keys supported (email string and _id equality). Search uses in-memory filtering fallback.
- Files: `packages/worker/src/api/controllers/global/users.ts` (line 308), `packages/backend-core/src/users/users.ts` (line 275)
- Impact: User search is limited and inefficient. Complex queries require loading all users into memory. Doesn't scale to large tenant populations.
- Fix approach: Migrate search logic from in-memory filtering to SQS-based querying. Requires moving search implementation from server package to core.

**Unrefactored OAuth Token Update:**
- Issue: At `packages/backend-core/src/auth/auth.ts:153`, manual user save and cache invalidation required instead of using centralized save function.
- Files: `packages/backend-core/src/auth/auth.ts` (lines 153-170)
- Impact: Duplicate code, inconsistent user update patterns, risk of missing cache invalidation in other OAuth flows.
- Fix approach: Consolidate OAuth token updates to use the `UserDB.save()` function, ensuring consistent cache invalidation.

**HACK - SCIM Events Silently Dropped:**
- Issue: At `packages/backend-core/src/cache/docWritethrough.ts:60`, SCIM log events are dropped from the doc writethrough queue without error or logging.
- Files: `packages/backend-core/src/cache/docWritethrough.ts` (lines 60-62)
- Impact: SCIM operations may appear to succeed but events never persist. Silently losing data makes debugging extremely difficult.
- Fix approach: Either support SCIM events in the writethrough queue or return explicit error/rejection to caller so handling is intentional.

---

## Security Concerns

**Unsafe JavaScript Evaluation in String Templates:**
- Risk: `eval()` used directly in string template processing. At `packages/string-templates/src/helpers/javascript.ts:115`, snippets are evaluated with `eval(iifeWrapper())`.
- Files: `packages/string-templates/src/helpers/javascript.ts` (line 115), `packages/pro/src/sdk/plugins/index.ts` (line 48)
- Current mitigation: Context sandbox via custom `runJS` implementation; variables wrapped in function scope. Admin-only feature in production.
- Recommendations: Document security model clearly. Ensure user-supplied code never passes through this path. Consider VM2 or similar if accepting untrusted code.

**eval() in Plugin Validation:**
- Risk: Plugin JS files validated using direct `eval()` call at `packages/pro/src/sdk/plugins/index.ts:48`. If validation passes, the plugin is trusted.
- Files: `packages/pro/src/sdk/plugins/index.ts` (lines 42-52)
- Current mitigation: Error messages caught and wrapped, but code still executed during validation.
- Recommendations: Use static analysis instead of execution for plugin validation. Run plugin code in isolated VM with resource limits.

**unsafe-eval in Content Security Policy:**
- Risk: CSP explicitly allows `'unsafe-eval'` for script-src at `packages/backend-core/src/middleware/contentSecurityPolicy.ts:11`.
- Files: `packages/backend-core/src/middleware/contentSecurityPolicy.ts` (line 11)
- Current mitigation: Required for string template JS execution. Feature flagged behind licensing check.
- Recommendations: Implement nonce-based script whitelisting for JavaScript execution instead of blanket unsafe-eval. Generate nonces per request and inject dynamically.

**unsafe-inline in Style CSP:**
- Risk: CSP allows `'unsafe-inline'` for styles at `packages/backend-core/src/middleware/contentSecurityPolicy.ts:22`.
- Files: `packages/backend-core/src/middleware/contentSecurityPolicy.ts` (line 22)
- Impact: Can be exploited for style-based attacks, exfiltration.
- Recommendations: Generate style nonces per request similar to script nonces.

**Empty Catch Blocks Hiding Errors:**
- Issue: Multiple empty catch blocks silently swallow errors, particularly in quota and credit operations.
- Files: `packages/pro/src/sdk/quotas/helpers/ai.ts` (catch with empty swallow), `packages/server/src/sdk/workspace/ai/llm/bbai.ts`, `packages/server/src/api/controllers/ai/chatIdentityLinks.ts`, `packages/server/src/api/controllers/ai/budibaseai-v2.ts`
- Impact: Quota failures, credit updates, and billing operations fail silently. Users may not be charged, or charging may be missed.
- Fix approach: Log all caught errors with context. At minimum, emit metrics for retry/failure analysis. Implement explicit retry logic.

---

## Known Architectural Issues

**Monolithic Test Files:**
- Issue: Several test files exceed 4000+ lines (viewV2.spec.ts: 5094 lines, row.spec.ts: 4429 lines, search.spec.ts: 4205 lines).
- Files: `packages/server/src/api/routes/tests/viewV2.spec.ts`, `packages/server/src/api/routes/tests/row.spec.ts`, `packages/server/src/api/routes/tests/search.spec.ts`
- Impact: Slow test execution, difficult to locate relevant tests, high chance of test interdependencies. 377 test files total, many sprawling.
- Fix approach: Split monolithic test suites by feature/endpoint. Extract common test utilities to reduce duplication. Run tests in parallel.

**String Throws Instead of Error Objects:**
- Issue: Code throws strings instead of Error objects at `packages/backend-core/src/users/db.ts:150`, `packages/string-templates/src/index.ts`.
- Files: `packages/backend-core/src/users/db.ts` (line 150: `throw "Password must be specified"`), `packages/string-templates/src/index.ts`
- Impact: Stack traces are lost, error handling inconsistent, impossible to programmatically catch specific errors.
- Fix approach: Replace all `throw "string"` with `throw new Error("message")` or custom error classes.

**Type Coercion in Function Returns:**
- Issue: Multiple functions return values cast to `any` instead of properly typed.
- Files: `packages/pro/src/sdk/groups/groups.ts:322` returns `resp as any`, `packages/backend-core/src/queue/inMemoryQueue.ts` returns `this as any`
- Impact: Downstream code loses type information, making API contracts unclear.
- Fix approach: Define proper return types for all public functions.

---

## Performance Bottlenecks

**Large Component Store Files:**
- Problem: `packages/builder/src/stores/builder/automations.ts` is 3171 lines, `restTemplates.ts` is 3097 lines, `components.ts` is 1372 lines. Single stores managing complex state.
- Cause: Monolithic stores with multiple concerns (creation, deletion, updates, serialization).
- Impact: Slow hot-reloads during development, large bundle size, difficult state reasoning.
- Improvement path: Break into smaller composable stores, implement lazy evaluation, memoize computed properties.

**SQL Conditional Complexity:**
- Problem: SQL query builder has multiple database-specific conditionals for null handling, type conversion, etc.
- Cause: Supporting PostgreSQL, MySQL, Oracle, SQLite simultaneously without abstraction layer.
- Impact: Query generation is complex, each new feature requires testing across all databases.
- Improvement path: Build explicit database abstraction layer for null ordering, date handling, type casting.

**In-Memory User Search:**
- Problem: User searches fall back to loading all users into memory when using complex filters.
- Cause: Search logic is in 'server' package but core only has basic email/ID lookups.
- Impact: O(N) memory usage, request hangs with large user bases.
- Improvement path: Move search to SQS layer in backend-core with proper indexing.

---

## Fragile Areas

**Grid Column Store (`packages/frontend-core/src/components/grid/stores/columns.ts`):**
- Files: `packages/frontend-core/src/components/grid/stores/columns.ts` (lines 192-193)
- Why fragile: TODO comments mark undefined `__left` and `__idx` fields with type `any`. These fields drive column positioning and logic. Type is unsafe.
- Safe modification: Add proper types for these fields, understand their role in layout system before refactoring.
- Test coverage: Grid column tests exist but are tightly coupled to current implementation.

**Validation Schema Utilities (`packages/frontend-core/src/utils/validation/`):**
- Files: `packages/frontend-core/src/utils/validation/validators.js` (line 1: TODO Convert to yup validators)
- Why fragile: Mix of old custom validators and new yup-based system. Code path unclear for which validations use which system.
- Safe modification: Document the current state fully before attempting migration. Add feature flag to selectively enable yup validators.
- Test coverage: Validation tests are scattered; need consolidated test suite.

**AI Quota/Credit System:**
- Files: `packages/pro/src/sdk/quotas/quotas.ts` (line 146: TODO about results), `packages/server/src/api/controllers/ai/chatConversations.ts` (line 526: TODO logging not implemented)
- Why fragile: Silent failure points in credit deduction. No comprehensive logging. Edge cases around concurrent requests not handled.
- Safe modification: Add comprehensive logging before any changes. Add circuit breaker for quota checks.
- Test coverage: Limited test coverage for failure scenarios.

**Automation Data Rehydration:**
- Files: `packages/server/src/automations/rehydrate.ts` (line 32: comment about Redis being ephemeral)
- Why fragile: Assumes job queue has persisted data after Redis restarts. Self-hosted setups lose job state.
- Safe modification: Make no assumptions about Redis persistence. Add explicit state storage mechanism.
- Test coverage: Need tests with Redis restart scenarios.

**Data Binding Logic (`packages/builder/src/dataBinding.js`):**
- Files: `packages/builder/src/dataBinding.js` (lines 351, 666, 1381: multiple TODO comments about removal and whitespace handling)
- Why fragile: Large complex file (1000+ lines) with deferred cleanup and poorly understood edge cases around whitespace.
- Safe modification: Write comprehensive tests for current behavior before touching. Extract individual concerns into separate modules.
- Test coverage: Limited test coverage for edge cases.

---

## Scaling Limits

**CouchDB View Queries in Bulk Operations:**
- Current capacity: Database query limit of 5000 rows (SQL_MAX_ROWS environment variable) or 500 related rows (SQL_MAX_RELATED_ROWS).
- Limit: User/group operations that scan all users hit memory limits. Pagination not always implemented.
- Scaling path: Implement cursor-based pagination for all bulk operations. Add batch processing for large datasets.

**Column State in Grid:**
- Current capacity: Grid components store full column metadata in state, leading to large Svelte store objects.
- Limit: Large tables (500+ columns) cause memory pressure and slowdowns.
- Scaling path: Virtualize column rendering, lazy-load column metadata.

**Automation Job Persistence:**
- Current capacity: Bull queue retries up to 100 times (PERSIST_MAX_ATTEMPTS in docWritethrough.ts).
- Limit: Doesn't account for job data loss in ephemeral Redis setups.
- Scaling path: Use persistent job storage backend, implement multi-queue failover.

---

## Dependencies at Risk

**Eval-Based Execution:**
- Risk: Core functionality depends on JavaScript evaluation (eval). This is difficult to secure, hard to sandbox properly.
- Impact: If eval sandbox is compromised, attacker gains code execution in app context.
- Migration plan: Implement WebAssembly-based expression evaluator, or strictly limit JS to specific safe operations.

**Plugin Loading via eval:**
- Risk: Plugin system validates datasource plugins via eval. Malicious plugin could inject code.
- Impact: All plugins loaded after a malicious one could be compromised.
- Migration plan: Implement static JS analysis for plugins, move validation to build time.

---

## Test Coverage Gaps

**AI/LLM Integration Tests:**
- Untested area: Quota enforcement, credit deduction, token counting across different LLM providers.
- Files: `packages/server/src/api/routes/tests/ai.spec.ts`, `packages/server/src/api/routes/tests/ai/aiConfig.spec.ts` - focus on configuration, not quota/credit flows.
- Risk: Credit deduction bugs could lead to platform charges without corresponding usage. Quota bypasses possible.
- Priority: High - financial impact

**Complex SQL Query Generation:**
- Untested area: Database-specific null ordering, complex filter combinations across PostgreSQL/MySQL/Oracle/SQLite.
- Files: `packages/backend-core/src/sql/sql.ts` has complex conditional logic but limited edge case testing.
- Risk: Query generation bugs only caught when running specific database. Inconsistent results across databases.
- Priority: High - data correctness

**User Search Edge Cases:**
- Untested area: Large dataset search, special characters in email, concurrent search requests.
- Files: `packages/worker/src/api/routes/global/tests/users.spec.ts` tests basic CRUD but not search performance/consistency.
- Risk: Search failures could prevent user lookups in critical paths like login.
- Priority: High - feature blocking

**SCIM Event Handling:**
- Untested area: SCIM log persistence in doc writethrough. Currently silently dropped.
- Files: No tests for SCIM event persistence.
- Risk: SCIM sync operations fail silently, users never actually synced.
- Priority: High - data consistency

**Bulk Delete Transaction Consistency:**
- Untested area: Partial failures during bulk user deletion. What happens if one delete fails mid-operation?
- Files: `packages/backend-core/src/users/db.ts` (lines 443+) has bulk delete logic but limited error scenario testing.
- Risk: Inconsistent state where some users deleted, others not, leaving orphaned references.
- Priority: Medium - data consistency

**OAuth Token Refresh Failure Paths:**
- Untested area: What happens when refresh token is invalid, expired, or provider unreachable?
- Files: `packages/backend-core/src/auth/auth.ts` (lines 55-116) has refresh logic but error handling unclear.
- Risk: Users locked out if token refresh fails silently.
- Priority: Medium - authentication blocking

---

*Concerns audit: 2026-03-19*
