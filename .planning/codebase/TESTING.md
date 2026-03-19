# Testing Patterns

**Analysis Date:** 2026-03-19

## Test Framework

**Backend (NodeJS) - Jest:**
- Runner: Jest 29+ with @swc/jest for TS transformation
- Config locations:
  - `packages/server/jest.config.ts`
  - `packages/backend-core/jest.config.ts`
  - `packages/worker/jest.config.ts`
  - `packages/shared-core/jest.config.ts`
  - `packages/upgrade-tests/jest.config.ts`
- Module mapper: Path aliases configured for `@budibase/*` imports

**Frontend (Browser) - Vitest:**
- Runner: Vitest with jsdom environment
- Config location: `packages/builder/vite.config.mjs` (test section in Vite config)
- Setup file: `packages/builder/vitest.setup.js`
- Environment: jsdom with custom matchers and ResizeObserver polyfill

**Assertion Library:**
- Backend: Jest matchers + jest-extended (via `expect.extend(matchers)`)
- Frontend: Vitest matchers + @testing-library/jest-dom/vitest

## Run Commands

**Backend tests:**
```bash
cd packages/server
yarn test                    # Run all tests in server
yarn test <filename>         # Run specific test file
yarn test <pattern>          # Run tests matching pattern
```

**Frontend tests:**
```bash
cd packages/builder
yarn test                    # Run all tests (vitest)
yarn test --watch           # Watch mode
```

**With environment variables:**
```bash
DATASOURCE=postgres yarn test filename.spec.ts
```
Available databases (from `packages/server/src/integrations/tests/utils/index.ts`): postgres, mysql, mariadb, etc.

## Test File Organization

**Location patterns:**
- Backend: Collocated test directories: `middleware/tests/`, `sdk/workspace/*/tests/`
- Backend: Separate API tests in `packages/server/src/tests/api/`
- Backend: Integration tests in `packages/server/src/integration-test/`
- Frontend: Collocated with source files

**Naming:**
- Backend: `.spec.ts` extension (e.g., `authorized.spec.ts`)
- Frontend: `.spec.ts` or `.test.js/ts` (e.g., `CreateEditGroupModal.spec.ts`, `searchFields.test.js`)

**Directory structure example:**
```
packages/server/src/
├── middleware/
│   ├── authorized.ts
│   └── tests/
│       ├── authorized.spec.ts
│       ├── resourceId.spec.ts
│       └── trimViewRowInfo.spec.ts
├── tests/
│   ├── api/
│   │   ├── chatApps.spec.ts
│   │   ├── chatConversations.spec.ts
│   │   └── rowExternal.spec.ts
│   ├── utilities/
│   │   ├── TestConfiguration.ts
│   │   ├── structures.ts
│   │   └── api.ts
│   ├── jestEnv.ts
│   └── jestSetup.ts
```

## Test Structure

**Backend Jest pattern:**
```typescript
import { generator, mocks } from "@budibase/backend-core/tests"
import TestConfiguration from "../utilities/TestConfiguration"

describe("authorization middleware", () => {
  let config: TestConfiguration

  beforeAll(async () => {
    config = new TestConfiguration()
    await config.init("test-name")
  })

  beforeEach(() => {
    jest.clearAllMocks()
    mocks.licenses.useCloudFree()
  })

  afterEach(() => {
    config.afterEach()
  })

  afterAll(() => {
    config.end()
  })

  it("describes behavior clearly", async () => {
    // Arrange
    config.setUser({ _id: "user", role: { _id: "ADMIN" } })

    // Act
    await config.executeMiddleware()

    // Assert
    expect(config.next).toHaveBeenCalled()
  })

  describe("nested describe for related tests", () => {
    it("another test", () => {
      expect(result).toEqual(expected)
    })
  })
})
```

**Frontend Vitest pattern:**
```typescript
import { render, screen } from "@testing-library/svelte"
import { describe, expect, it, vi } from "vitest"
import Component from "./Component.svelte"

describe("Component", () => {
  it("renders expected content", () => {
    render(Component, {
      props: { prop: "value" },
    })

    expect(screen.getByText("Expected")).toBeInTheDocument()
  })

  it("calls handlers on user action", () => {
    const handler = vi.fn()
    render(Component, {
      props: { onClick: handler },
    })

    screen.getByRole("button").click()
    expect(handler).toHaveBeenCalled()
  })
})
```

**Jest Setup Files:**

Backend-core (`packages/backend-core/tests/jestSetup.ts`):
```typescript
import env from "../environment"
import * as matchers from "jest-extended"
import { env as coreEnv, timers } from "@budibase/backend-core"

expect.extend(matchers)
jest.setTimeout(process.env.CI ? 30 * 1000 : 100 * 1000)
process.env.DISABLE_PINO_LOGGER = "1"
```

Server (`packages/server/src/tests/jestSetup.ts`):
```typescript
import env from "../environment"
import * as matchers from "jest-extended"
import nock from "nock"

expect.extend(matchers)
nock.disableNetConnect()
nock.enableNetConnect(host =>
  host.includes("localhost") || host.includes("127.0.0.1")
)
```

## Mocking

**Jest Mocking (Backend):**
```typescript
// Mock at top of file before imports
jest.mock("../../sdk/workspace/permissions", () => ({
  ...jest.requireActual("../../sdk/workspace/permissions"),
  getResourcePerms: jest.fn().mockResolvedValue({}),
}))

// Import after mock declaration
import sdk from "../../sdk"

// Type the mock
const mockedGetPerms = sdk.permissions.getResourcePerms as jest.MockedFunction<
  typeof sdk.permissions.getResourcePerms
>

// In tests
beforeEach(() => {
  jest.clearAllMocks()
  mockedGetPerms.mockResolvedValue({ READ: { role: "PUBLIC" } })
})

afterEach(() => {
  jest.clearAllMocks()
})
```

**Vitest Mocking (Frontend):**
```typescript
import { vi } from "vitest"

vi.mock("@budibase/bbui", () => ({
  Button: MockButton,
  Modal: MockModal,
}))

// In tests
const handler = vi.fn()
const result = handler()
expect(handler).toHaveBeenCalledWith(expectedArg)
```

**Network Mocking:**
- Backend uses `nock` for HTTP mocking (see jestSetup.ts)
- Pattern: `nock.disableNetConnect()` prevents accidental external calls
- Allow localhost: `nock.enableNetConnect(host => host.includes("localhost"))`

**What to Mock:**
- External HTTP calls (REST APIs, webhooks)
- Database access (when testing logic, not integration)
- Time-dependent functions (use Jest fake timers)
- Heavy dependencies
- SDK methods in isolation tests

**What NOT to Mock:**
- Database in integration tests
- Internal service-to-service calls in end-to-end scenarios
- Standard library functions
- Type definitions

## Fixtures and Factories

**Test Data Builders (Backend):**

Location: `packages/server/src/tests/utilities/structures.ts`

Example builder functions:
```typescript
export function basicTable(
  datasource?: Datasource,
  ...extra: Partial<Table>[]
): Table {
  return tableForDatasource(
    datasource,
    {
      name: "TestTable",
      schema: {
        name: { type: FieldType.STRING, name: "name" },
        description: { type: FieldType.STRING, name: "description" },
      },
    },
    ...extra
  )
}
```

Usage pattern - merge with extra config:
```typescript
import { basicTable } from "../utilities/structures"

const table = basicTable(datasource, {
  name: "CustomTable",
  schema: {
    ...
  }
})
```

**Automation Test Builder (Backend):**

Location: `packages/server/src/automations/tests/utilities/AutomationTestBuilder.ts`

Pattern: Fluent API for building automation tests
```typescript
const builder = new createAutomationBuilder().onRowSaved(...)
  .step("someAction")({ input: value })
  .branch("condition", filters, branchSteps)
  .loop(LoopStepType.ARRAY, binding, loopSteps)
```

**Random Data Generation (Backend):**

Use `generator` from `@budibase/backend-core/tests`:
```typescript
import { generator } from "@budibase/backend-core/tests"

const userId = generator.guid()
const email = generator.email()
const array = generator.arrayOf(() => generator.guid(), { min: 5, max: 5 })
```

**Helper Functions (Frontend):**

Example from `packages/builder/src/settings/pages/people/users/roleUtils.spec.ts`:
```typescript
const buildGroup = (overrides = {}) => ({
  _id: "group-1",
  _rev: "rev-1",
  name: "Actions",
  ...overrides,
})

describe("Component", () => {
  it("test", () => {
    const group = buildGroup({ name: "Custom" })
    expect(group.name).toBe("Custom")
  })
})
```

## Coverage

**Requirements:**
- No enforced minimum coverage globally
- Specific areas may have targets (check `.nycrc` or coverage config)
- Coverage reports generated: lcov, json, clover formats

**View Coverage (Backend):**
```bash
cd packages/server
yarn test
# Coverage output in coverage/ directory
open coverage/lcov-report/index.html
```

**Excluded from Coverage:**
- `src/**/*.spec.ts` (test files)
- `src/tests/**/*` (test utilities)
- `src/db/views/staticViews.*` (coverage breaks view functions)
- `src/jsRunner/**/*` (coverage interferes with isolated VM)

## Test Types

**Unit Tests:**
- Scope: Single function or class method
- Isolation: Heavy mocking of dependencies
- Location: Collocated with source in `tests/` subdirectories
- Example: `packages/backend-core/src/cache/tests/user.spec.ts`
  - Tests `getUsers()` function with cache behavior
  - Mocks database layer
  - Focuses on caching logic

**Integration Tests:**
- Scope: Multiple components working together
- Database: Real database interactions
- Location: `packages/server/src/integration-test/` for DB tests
- Setup: Uses `DBTestConfiguration` for database context
- Example pattern:
  ```typescript
  const config = new DBTestConfiguration()

  beforeAll(async () => {
    const userCount = 10
    await config.doInTenant(async () => {
      const db = getGlobalDB()
      // Create test data
    })
  })
  ```

**API Tests:**
- Scope: Full HTTP endpoints
- Framework: Jest with TestConfiguration
- Setup: `TestConfiguration` manages server instance
- Location: `packages/server/src/tests/api/`
- Example: `packages/server/src/tests/api/chatApps.spec.ts`
  - Creates real database documents
  - Makes HTTP requests via supertest
  - Validates response status and body

**E2E Tests:**
- Framework: Not standard in codebase
- Automation testing: Uses AutomationTestBuilder fluent API
- Example: `packages/server/src/automations/tests/*.spec.ts`

## Common Patterns

**Async Testing:**
```typescript
// Jest pattern
it("awaits async function", async () => {
  const result = await asyncFunction()
  expect(result).toEqual(expected)
})

// Frontend pattern
it("handles async operations", async () => {
  render(Component)
  await userEvent.click(screen.getByRole("button"))
  await expect(screen.findByText("Loaded")).resolves.toBeInTheDocument()
})
```

**Error Testing:**
```typescript
// Backend pattern - expect thrown
it("throws on invalid input", async () => {
  expect(() => {
    ctx.throw(400, "Invalid request")
  }).toThrow(HTTPError)
})

// Backend pattern - expect rejected promise
it("rejects with error", async () => {
  await expect(asyncFunction(invalid)).rejects.toThrow("Error message")
})

// Frontend pattern
it("shows error message", () => {
  render(Component, { props: { shouldError: true } })
  expect(screen.getByText("Error occurred")).toBeInTheDocument()
})
```

**Mocking Results:**
```typescript
// Test final outcome, not intermediate states
it("caches results after first fetch", async () => {
  jest.spyOn(UserDB, "bulkGet")

  // First call
  await getUsers(userIds)
  // Second call
  const results = await getUsers(userIds)

  // Assert final state - only one DB call made
  expect(UserDB.bulkGet).toHaveBeenCalledTimes(1)
  expect(results.users).toHaveLength(5)
})
```

**Conditional Test Execution:**
```typescript
it.each([
  [Constants.BudibaseRoles.Admin, expectedAdminFlags],
  [Constants.BudibaseRoles.Developer, expectedDevFlags],
])("returns flags for %s", (role, expected) => {
  expect(getRoleFlags(role)).toEqual(expected)
})

it.skip("skip this temporarily", () => {})

it.todo("implement this test")
```

**Timeout Configuration:**
- Development: 100 seconds (for debugging)
- CI: 30 seconds (faster feedback)
- Set in jestSetup.ts: `jest.setTimeout(process.env.CI ? 30 * 1000 : 100 * 1000)`
- Override per-test: `jest.setTimeout(5000)` at test start

## Test Utilities

**Backend Test Configuration:**

Location: `packages/server/src/tests/utilities/TestConfiguration.ts`

Provides:
- Server instance management (beforeAll/afterAll)
- User/workspace setup
- API request helpers
- Database cleanup
- License mocking

Usage:
```typescript
const config = new TestConfiguration()

beforeAll(async () => {
  await config.init()
})

afterAll(() => {
  config.end()
})

const headers = await config.defaultHeaders()
const response = await config
  .getRequest()!
  .get("/api/tables")
  .set(headers)
```

**Database Test Configuration:**

For tests needing real database:
```typescript
import { DBTestConfiguration } from "@budibase/backend-core/tests"

const config = new DBTestConfiguration()

await config.doInTenant(async () => {
  // Run code in tenant context with database
  const db = getGlobalDB()
  await db.put(doc)
})
```

**Mocking Pro Features:**

Location: `packages/server/src/tests/utilities/mocks/pro.ts`

```typescript
import { initProMocks } from "../utilities/mocks/pro"

beforeAll(() => {
  initProMocks()
})

// Or use mocks directly
import { mocks } from "@budibase/backend-core/tests"

beforeEach(() => {
  mocks.licenses.useCloudFree()
  // or
  mocks.licenses.useUnlimited()
})
```

---

*Testing analysis: 2026-03-19*
