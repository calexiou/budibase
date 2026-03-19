# Coding Conventions

**Analysis Date:** 2026-03-19

## Naming Patterns

**Files:**
- TypeScript/JavaScript files: camelCase or PascalCase (e.g., `authorized.ts`, `TestConfiguration.ts`)
- Test files: Suffix with `.spec.ts` or `.test.ts` (e.g., `authorized.spec.ts`, `searchFields.test.js`)
- Svelte components: PascalCase (e.g., `CreateEditGroupModal.svelte`)
- Config files: camelCase or dash-separated (e.g., `vite.config.mjs`, `.prettierrc.json`)

**Functions and Arrow Functions:**
- camelCase naming
- Prefer arrow functions: `const myFunction = () => { ... }`
- Use async/await, not Promise chains
- Example from `packages/server/src/sdk/plugins/index.ts`: `export async function fetch(type?: PluginType): Promise<Plugin[]>`

**Variables:**
- camelCase for all variables and constants
- Prefix unused variables with underscore: `const _unused = value`
- Example pattern: `const userIds`, `const datasources`, `let isAuthed`

**Types and Interfaces:**
- Use `interface` for object types (exported)
- Use `type` for unions and primitives
- PascalCase naming (e.g., `interface User`, `type PermissionLevel`)
- Example from `packages/server/src/tests/utilities/TestConfiguration.ts`: `export interface TableToBuild extends Omit<Table, "sourceId" | "sourceType">`

**Constants:**
- SCREAMING_SNAKE_CASE for compile-time constants
- camelCase for runtime constants
- Example: `const ENV_VAR_PREFIX = "env."` in `packages/server/src/sdk/workspace/datasources/datasources.ts`

## Code Style

**Formatting:**
- Tool: Prettier (configured in `.prettierrc.json`)
- Tab width: 2 spaces
- No semicolons
- Double quotes (not single)
- Trailing commas: ES5 style
- Arrow parentheses: avoid when possible

**Linting:**
- Tool: ESLint (configured in `eslint.config.mjs`)
- No var declarations (enforced error)
- No unused variables (except those prefixed with `_`)
- No implicit eval, extend native, labels, lone blocks, new wrappers
- Strict mode enabled for TypeScript files

**TypeScript Configuration:**
- Strict mode enabled
- Use `consistent-type-imports` rule (enforce `import type`)
- No casting to `any` - use proper types
- All TypeScript files enforce strict checking

## Import Organization

**Order in Backend/NodeJS files (packages/server, packages/backend-core, packages/worker):**
1. External imports: `import { ... } from "external-lib"`
2. Backend-core imports: `import { ... } from "@budibase/backend-core"`
3. Types imports: `import type { ... } from "@budibase/types"`
4. Shared-core imports: `import { ... } from "@budibase/shared-core"`
5. Internal absolute imports: `import sdk from "../../sdk"`
6. Relative imports: `import { something } from "../relative/path"`

**Order in Frontend/Browser files (packages/builder, packages/client, packages/frontend-core):**
1. External imports: `import { ... } from "library"`
2. Budibase types: `import type { ... } from "@budibase/types"`
3. Budibase shared-core: `import { ... } from "@budibase/shared-core"`
4. Budibase bbui: `import { ... } from "@budibase/bbui"`
5. Internal absolute imports: `import { Component } from "@/components"`
6. Relative imports: `import { helper } from "../utils"`

**Path Aliases:**
- Backend: `@budibase/backend-core`, `@budibase/shared-core`, `@budibase/types`, `@budibase/pro`, `@budibase/string-templates`
- Frontend: `@budibase/types`, `@budibase/shared-core`, `@budibase/bbui`, `@/` (src root), `assets/`

**Avoid:**
- Barrel imports (rule: `local-rules/no-barrel-imports` = error)
- Importing from wrong packages (rule: `local-rules/no-budibase-imports` = error)
- Console.error in production code (rule: `local-rules/no-console-error` = error)

## Error Handling

**Patterns:**
- Use try/catch blocks with error handling
- Check error type: `err instanceof Error ? err.message : String(err)`
- Log errors with context: `console.log("Operation failed:", context, err.message)`
- Example from `packages/server/src/sdk/plugins/index.ts`:
  ```typescript
  try {
    // operation
  } catch (err) {
    console.log("Failed operation:", url, err instanceof Error ? err.message : String(err))
    return
  }
  ```
- Backend API: Use `ctx.throw(statusCode, "message")` for HTTP errors
- Return early on error instead of nested conditions

## Logging

**Framework:**
- Backend (NodeJS): Use `console.log` and `console.error` (redirected to pino logger)
- Frontend: Use `console.log`, `console.warn`, `console.error` (allow list varies by package)
- Never use `console.log` in tests - output not visible in STDOUT

**Patterns:**
- `console.log("Message", context, details)` for informational logs
- `console.error("Error message", err)` for errors in production code
- Example from `packages/server/src/middleware/workspaceMigrations.ts`: `console.log("Skipping migration redirect")`
- Backend packages restrict console output in tests via setup file

## Comments

**When to Comment:**
- Only when behavior is unclear or non-obvious
- Explain the "why", not the "what"
- Avoid obvious comments
- Example pattern (minimal and purposeful)

**JSDoc/TSDoc:**
- Used sparingly in the codebase
- Function signatures should be self-documenting via TypeScript
- Document complex algorithms or gotchas, not simple functions

## Function Design

**Size:**
- Keep functions focused and small
- Avoid long parameter lists (use object destructuring for multiple related params)
- Extract complex logic into named helper functions

**Parameters:**
- Use destructuring for object parameters
- Optional parameters are clearly typed
- Example: `function middleware(opts?: { appType?: AppType } = {})`

**Return Values:**
- Always type return values explicitly
- Use union types for multiple return scenarios
- Prefer returning typed objects over arrays where meaning is important
- Async functions: return `Promise<T>`

## Module Design

**Exports:**
- Named exports preferred for multiple exports
- Default export only when exporting a single primary thing (e.g., default middleware function)
- Re-export pattern: `export * from "./module"` with barrel files used in types/shared-core
- Example from `packages/shared-core/src/index.ts`:
  ```typescript
  export * from "./constants"
  export * as dataFilters from "./filters"
  export * as helpers from "./helpers"
  ```

**Barrel Files:**
- Used in `packages/shared-core` for namespaced exports
- Avoided in regular packages due to `local-rules/no-barrel-imports` rule
- Enables organized public APIs

## Test-Specific Conventions

**Test Files:**
- Collocated with source: `feature.ts` → `feature.spec.ts` (middleware, tests directory)
- Separate tests directory in packages/server for API and integration tests
- Example: `packages/server/src/middleware/tests/authorized.spec.ts`

**Test Structure:**
- Describe blocks for grouping related tests
- Clear test names describing behavior: `it("throws when user lacks permissions", ...)`
- Avoid intermediate state assertions - test final outcomes
- No console.log in tests

**Mocking:**
- Jest mocks: `jest.mock()` with actual imports at top
- TypeScript mocking: Cast with `jest.MockedFunction<typeof originalFunction>`
- Vitest mocks: `vi.mock()` and `vi.fn()`
- Clear mock setup in beforeEach
- Example pattern from `packages/server/src/middleware/tests/authorized.spec.ts`:
  ```typescript
  jest.mock("../../sdk/workspace/permissions", () => ({
    ...jest.requireActual("../../sdk/workspace/permissions"),
    getResourcePerms: jest.fn().mockResolvedValue({}),
  }))
  ```

---

*Convention analysis: 2026-03-19*
