# Playwright Parallel Execution

## 1. Introduction

Playwright supports running tests in parallel to reduce overall execution time.

Parallel execution is especially important in large automation frameworks and CI/CD pipelines where hundreds or thousands of tests may need to execute efficiently.

Playwright achieves parallel execution primarily through **workers**.

```text
Without Parallel Execution

Test 1 → Test 2 → Test 3 → Test 4
                 ↓
             Long Runtime


With Parallel Execution

Worker 1 → Test 1
Worker 2 → Test 2
Worker 3 → Test 3
Worker 4 → Test 4
                 ↓
             Faster Runtime
```

---

# 2. Why Parallel Execution?

Suppose you have 100 tests and each test takes approximately 10 seconds.

Sequential execution:

```text
100 × 10 seconds = 1000 seconds
```

With 5 workers, execution can be significantly reduced because multiple tests run simultaneously.

Benefits:

* Faster test execution
* Faster CI/CD pipelines
* Better utilization of CPU resources
* Faster regression testing
* Better scalability
* Useful for large test suites

---

# 3. Playwright Workers

A **worker** is a separate process used by Playwright Test to execute tests.

Example:

```text
Workers = 4

Worker 1 → Test A → Test E → Test I
Worker 2 → Test B → Test F → Test J
Worker 3 → Test C → Test G → Test K
Worker 4 → Test D → Test H → Test L
```

Each worker has its own:

* Browser instances
* Browser contexts
* Test execution environment
* Worker-scoped fixtures

---

# 4. Configure Workers

Workers can be configured in `playwright.config.ts`.

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  workers: 4,
});
```

This allows up to four worker processes to execute tests concurrently.

---

# 5. Using Percentage of CPU

Playwright also supports specifying workers as a percentage.

```typescript
export default defineConfig({
  workers: '50%',
});
```

This means Playwright can use approximately half of the available CPU capacity for workers.

This can be useful in CI environments where the number of CPU cores may vary.

---

# 6. Command-Line Worker Configuration

Workers can also be configured from the command line.

```bash
npx playwright test --workers=4
```

Or:

```bash
npx playwright test --workers=50%
```

Command-line configuration is useful when different environments require different levels of parallelism.

For example:

```bash
# Local development
npx playwright test --workers=2

# CI
npx playwright test --workers=6
```

---

# 7. Default Parallel Execution

Playwright Test can execute tests in parallel across worker processes.

Example:

```typescript
import { test, expect } from '@playwright/test';

test('Test 1', async ({ page }) => {
  await page.goto('https://example.com');
});

test('Test 2', async ({ page }) => {
  await page.goto('https://example.com');
});

test('Test 3', async ({ page }) => {
  await page.goto('https://example.com');
});
```

With multiple workers, Playwright can distribute these tests across workers.

---

# 8. Fully Parallel Mode

Playwright supports fully parallel execution.

Configure:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  fullyParallel: true,
});
```

This tells Playwright that tests within files can also be executed in parallel.

---

# 9. Fully Parallel at Test Level

You can also enable fully parallel execution for a specific test file.

```typescript
import { test } from '@playwright/test';

test.describe.configure({
  mode: 'parallel',
});

test('Test 1', async ({ page }) => {
  // Test
});

test('Test 2', async ({ page }) => {
  // Test
});

test('Test 3', async ({ page }) => {
  // Test
});
```

The tests in the describe block can execute independently.

---

# 10. Parallel Mode

Playwright supports different execution modes.

## Default Mode

Tests are generally distributed between workers.

```typescript
test.describe('Login Tests', () => {
  test('Valid Login', async ({ page }) => {});
  test('Invalid Login', async ({ page }) => {});
  test('Logout', async ({ page }) => {});
});
```

## Parallel Mode

```typescript
test.describe.configure({
  mode: 'parallel',
});
```

Tests inside the block can run concurrently.

---

# 11. Serial Mode

Some tests depend on previous tests.

For example:

```text
Create User
     ↓
Update User
     ↓
Delete User
```

Running these tests in parallel would cause problems.

Use serial mode when tests have an unavoidable dependency.

```typescript
test.describe.configure({
  mode: 'serial',
});
```

Example:

```typescript
test.describe.configure({
  mode: 'serial',
});

test('Create User', async ({ page }) => {
  // Create user
});

test('Update User', async ({ page }) => {
  // Update user
});

test('Delete User', async ({ page }) => {
  // Delete user
});
```

However, serial mode should not be overused.

The preferred approach is to make tests independent whenever possible.

---

# 12. Parallel vs Serial

| Mode           | Behavior                                 |
| -------------- | ---------------------------------------- |
| Default        | Tests distributed across workers         |
| Parallel       | Tests can run concurrently               |
| Serial         | Tests execute in sequence                |
| Fully Parallel | Maximum independent test parallelization |

---

# 13. Test Isolation

Parallel execution requires good test isolation.

Bad design:

```text
Test A → creates user "Selva"
Test B → deletes user "Selva"
```

If both execute simultaneously, the tests can interfere with each other.

Better design:

```text
Test A → creates unique user A
Test B → creates unique user B
```

Each test should ideally have its own:

* Test data
* Browser context
* Authentication state
* Database records
* Temporary files

---

# 14. Avoid Shared Mutable Data

Avoid global mutable variables.

Bad:

```typescript
let userId: string;

test('Create User', async ({ request }) => {
  userId = '12345';
});

test('Update User', async ({ request }) => {
  console.log(userId);
});
```

Parallel tests can modify the same variable and produce unpredictable results.

Better:

```typescript
test('Update User', async ({ request }) => {
  const userId = `user-${Date.now()}`;

  // Use userId only within this test
});
```

---

# 15. Unique Test Data

Parallel tests often require unique test data.

Example:

```typescript
const userId = `user-${Date.now()}-${Math.random()}`;
```

A more controlled approach:

```typescript
import { randomUUID } from 'crypto';

const userId = `user-${randomUUID()}`;
```

Example:

```typescript
test('Create User', async ({ request }) => {
  const userId = `user-${randomUUID()}`;

  const response = await request.post('/users', {
    data: {
      id: userId,
      name: 'Test User',
    },
  });

  expect(response.ok()).toBeTruthy();
});
```

---

# 16. Browser Context Isolation

Playwright automatically provides isolated browser contexts for tests.

Example:

```typescript
test('Test A', async ({ page }) => {
  await page.goto('https://example.com');
});

test('Test B', async ({ page }) => {
  await page.goto('https://example.com');
});
```

The tests do not normally share the same browser context.

This helps prevent:

* Cookies leaking between tests
* Local storage conflicts
* Session conflicts
* Authentication conflicts

---

# 17. Worker-Scoped Fixtures

Fixtures can have worker scope.

Example:

```typescript
import { test as base } from '@playwright/test';

type Fixtures = {
  workerData: string;
};

export const test = base.extend<Fixtures>({
  workerData: [
    async ({}, use) => {
      const data = `worker-${process.pid}`;

      await use(data);
    },
    { scope: 'worker' },
  ],
});
```

Worker-scoped fixtures are created once per worker.

---

# 18. Test Scope vs Worker Scope

### Test-scoped fixture

Created for each test.

```text
Worker
 ├── Test 1 → Fixture
 ├── Test 2 → Fixture
 └── Test 3 → Fixture
```

### Worker-scoped fixture

Created once per worker.

```text
Worker
 └── Fixture
      ├── Test 1
      ├── Test 2
      └── Test 3
```

Worker scope can improve performance when initialization is expensive.

---

# 19. Worker Index

Playwright provides worker information through `testInfo`.

Example:

```typescript
import { test } from '@playwright/test';

test('Worker Information', async ({}, testInfo) => {
  console.log(testInfo.workerIndex);
});
```

Worker indexes are useful for:

* Creating unique data
* Assigning ports
* Creating worker-specific resources
* Debugging parallel failures

Example:

```typescript
const user = `user-${testInfo.workerIndex}`;
```

---

# 20. Avoid Fixed Ports

Parallel tests may fail if every worker tries to use the same port.

Bad:

```text
Worker 1 → Port 3000
Worker 2 → Port 3000
Worker 3 → Port 3000
```

Better:

```typescript
const port = 3000 + testInfo.workerIndex;
```

Result:

```text
Worker 0 → 3000
Worker 1 → 3001
Worker 2 → 3002
Worker 3 → 3003
```

---

# 21. Parallel API Testing

Playwright can also execute API tests in parallel.

```typescript
import { test, expect } from '@playwright/test';

test('GET Users', async ({ request }) => {
  const response = await request.get('/users');

  expect(response.ok()).toBeTruthy();
});

test('GET Products', async ({ request }) => {
  const response = await request.get('/products');

  expect(response.ok()).toBeTruthy();
});

test('GET Orders', async ({ request }) => {
  const response = await request.get('/orders');

  expect(response.ok()).toBeTruthy();
});
```

These tests can be distributed across workers.

---

# 22. Parallel UI + API Tests

A framework can execute UI and API tests concurrently.

```text
Worker 1 → UI Login Tests
Worker 2 → UI Vehicle Tests
Worker 3 → API User Tests
Worker 4 → API Vehicle Tests
```

This is useful for large end-to-end frameworks.

---

# 23. Parallel Projects

Playwright projects can represent different browsers or environments.

Example:

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  projects: [
    {
      name: 'chromium',
      use: {
        ...devices['Desktop Chrome'],
      },
    },
    {
      name: 'firefox',
      use: {
        ...devices['Desktop Firefox'],
      },
    },
    {
      name: 'webkit',
      use: {
        ...devices['Desktop Safari'],
      },
    },
  ],
});
```

Tests can run across multiple projects.

```text
              Test Suite
                  |
       ┌──────────┼──────────┐
       ↓          ↓          ↓
   Chromium    Firefox    WebKit
```

---

# 24. Browser Matrix Testing

Parallel projects are useful for cross-browser testing.

```text
                    Test Suite
                       |
       ┌───────────────┼───────────────┐
       ↓               ↓               ↓
   Chromium         Firefox          WebKit
       ↓               ↓               ↓
    Tests           Tests            Tests
```

This provides broad browser coverage while reducing total execution time.

---

# 25. Parallel Execution in CI/CD

Parallel execution is especially useful in CI/CD.

Example:

```yaml
steps:
  - run: npm ci
  - run: npx playwright install --with-deps
  - run: npx playwright test --workers=4
```

CI environments can use more workers than a developer laptop, depending on available resources.

---

# 26. Sharding

For very large test suites, Playwright supports sharding.

Example:

```bash
npx playwright test --shard=1/4
```

```bash
npx playwright test --shard=2/4
```

```bash
npx playwright test --shard=3/4
```

```bash
npx playwright test --shard=4/4
```

The suite is divided across four CI jobs.

```text
                    Test Suite
                       |
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
     Shard 1        Shard 2        Shard 3
        ↓              ↓              ↓
      Tests          Tests          Tests

                         +
                      Shard 4
                         ↓
                       Tests
```

Sharding and workers can be combined.

```text
CI Job 1
  ├── Worker 1
  ├── Worker 2
  └── Worker 3

CI Job 2
  ├── Worker 1
  ├── Worker 2
  └── Worker 3
```

---

# 27. Parallel Execution Strategy

A good strategy for a large automation framework is:

```text
Test Suite
    |
    ├── Independent UI Tests
    ├── Independent API Tests
    ├── Cross Browser Tests
    └── Integration Tests
             |
             ↓
        CI Sharding
             |
             ↓
        Multiple Workers
```

---

# 28. Common Parallel Execution Problems

## Problem 1: Shared Test Data

```text
Test A → User 100
Test B → User 100
```

Solution:

```text
Test A → User A
Test B → User B
```

---

## Problem 2: Shared Files

Bad:

```text
Worker 1 → test-results.json
Worker 2 → test-results.json
```

Both workers may attempt to write to the same file.

Use worker-specific filenames.

```typescript
const fileName = `result-${testInfo.workerIndex}.json`;
```

---

## Problem 3: Shared Database Records

Avoid:

```text
Worker 1 → Update Vehicle 123
Worker 2 → Delete Vehicle 123
```

Use independent records.

```text
Worker 1 → Vehicle A
Worker 2 → Vehicle B
```

---

## Problem 4: Tests Depending on Execution Order

Bad:

```text
Test 1 → Create Account
Test 2 → Login
Test 3 → Delete Account
```

Instead, make each test prepare its own required state.

```text
Test 1 → Create + Validate
Test 2 → Create + Login + Validate
Test 3 → Create + Delete + Validate
```

---

# 29. Parallel Test Design Best Practices

Follow these rules:

1. Keep tests independent.
2. Avoid global mutable variables.
3. Use unique test data.
4. Avoid execution-order dependencies.
5. Use isolated browser contexts.
6. Use worker-scoped fixtures carefully.
7. Avoid shared files.
8. Avoid shared database records.
9. Use worker indexes when necessary.
10. Choose worker count based on available resources.
11. Use sharding for very large suites.
12. Monitor CI resource utilization.

---

# 30. Recommended Configuration

A typical enterprise configuration could look like:

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',

  fullyParallel: true,

  workers: process.env.CI ? 4 : 2,

  retries: process.env.CI ? 2 : 0,

  reporter: [
    ['html'],
    ['junit', { outputFile: 'results/results.xml' }],
  ],

  use: {
    baseURL: 'https://example.com',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },

  projects: [
    {
      name: 'chromium',
      use: {
        ...devices['Desktop Chrome'],
      },
    },

    {
      name: 'firefox',
      use: {
        ...devices['Desktop Firefox'],
      },
    },
  ],
});
```

---

# 31. Local vs CI Configuration

A common strategy is:

```typescript
workers: process.env.CI ? 4 : 2,
```

Meaning:

```text
Local
   ↓
2 Workers

CI
   ↓
4 Workers
```

This prevents excessive CPU usage during local development while allowing faster CI execution.

---

# 32. Running Specific Tests with Workers

Run all tests:

```bash
npx playwright test
```

Run with two workers:

```bash
npx playwright test --workers=2
```

Run with four workers:

```bash
npx playwright test --workers=4
```

Run a specific file:

```bash
npx playwright test tests/login.spec.ts --workers=4
```

Run a specific project:

```bash
npx playwright test --project=chromium --workers=4
```

---

# 33. Debugging Parallel Tests

Parallel failures can sometimes be difficult to reproduce.

Useful options:

```bash
npx playwright test --workers=1
```

This disables parallelism and makes debugging easier.

You can also run:

```bash
npx playwright test --workers=1 --debug
```

This is useful when investigating:

* Race conditions
* Shared test data
* Timing problems
* Resource conflicts
* Environment-specific failures

---

# 34. Parallel Execution and Retries

Retries can be configured:

```typescript
export default defineConfig({
  retries: process.env.CI ? 2 : 0,
});
```

If a test fails in CI, Playwright can retry it.

However, retries should not be used to hide poorly isolated tests.

A flaky test should be investigated rather than simply relying on retries.

---

# 35. Parallel Execution and Reporting

Reports should clearly identify:

* Test name
* Worker
* Project
* Retry number
* Failure reason
* Trace
* Screenshot
* Video

Example:

```text
Login Test
Project: chromium
Worker: 2
Retry: 1
Status: Passed
```

This makes parallel failures easier to investigate.

---

# 36. Example Enterprise Framework

A scalable framework might look like:

```text
playwright-framework/
│
├── tests/
│   ├── ui/
│   ├── api/
│   ├── integration/
│   └── regression/
│
├── fixtures/
│   ├── auth.fixture.ts
│   ├── api.fixture.ts
│   └── worker.fixture.ts
│
├── pages/
│   ├── LoginPage.ts
│   ├── HomePage.ts
│   └── VehiclePage.ts
│
├── test-data/
│
├── utils/
│
├── playwright.config.ts
│
└── package.json
```

Execution:

```text
CI Pipeline
    |
    ├── Shard 1
    │     ├── Worker 1
    │     └── Worker 2
    |
    ├── Shard 2
    │     ├── Worker 1
    │     └── Worker 2
    |
    └── Shard 3
          ├── Worker 1
          └── Worker 2
```

---

# 37. Parallel Execution vs Selenium

### Selenium Grid

Selenium commonly uses:

```text
TestNG
   ↓
Parallel Tests
   ↓
ThreadLocal WebDriver
   ↓
Selenium Grid
   ↓
Multiple Nodes
```

### Playwright

Playwright provides:

```text
Playwright Test
   ↓
Workers
   ↓
Browser Context Isolation
   ↓
Parallel Execution
```

Playwright's built-in test runner makes parallel execution simpler because worker management and browser-context isolation are integrated into the framework.

---

# 38. Senior-Level Recommendation

Do not simply increase workers to the maximum.

For example:

```typescript
workers: 20
```

does not automatically mean the suite will be faster.

Too many workers can cause:

* CPU saturation
* Memory pressure
* Network congestion
* Database overload
* Application throttling
* CI instability

Start with a reasonable number and measure execution time.

Example:

```text
2 workers → 25 minutes
4 workers → 14 minutes
6 workers → 11 minutes
8 workers → 10 minutes
12 workers → 11 minutes
```

In this example, 8 workers may be a better choice than 12.

---

# 39. Interview Questions

## Beginner

### 1. What is parallel execution in Playwright?

Parallel execution means running multiple tests simultaneously using Playwright workers to reduce total execution time.

### 2. What is a Playwright worker?

A worker is a separate process used by Playwright Test to execute tests.

### 3. How do you configure workers?

```typescript
workers: 4
```

or:

```bash
npx playwright test --workers=4
```

---

## Intermediate

### 4. What is `fullyParallel`?

It enables Playwright to run tests in parallel more aggressively, including tests within the same file.

```typescript
fullyParallel: true
```

### 5. How do you run tests serially?

```typescript
test.describe.configure({
  mode: 'serial',
});
```

### 6. How do you run tests in parallel within a describe block?

```typescript
test.describe.configure({
  mode: 'parallel',
});
```

### 7. Why should tests be independent?

Independent tests can run safely in parallel without causing data or state conflicts.

---

## Advanced

### 8. What is the difference between workers and browser contexts?

A worker is a Playwright Test execution process, while a browser context is an isolated browser session within the browser.

### 9. How do you avoid test-data conflicts?

Use unique test data and ensure each test owns the data it modifies.

### 10. What is worker-scoped fixture?

A fixture with:

```typescript
{ scope: 'worker' }
```

is initialized once per worker rather than once per test.

### 11. What is sharding?

Sharding divides a test suite across multiple CI jobs.

Example:

```bash
npx playwright test --shard=1/4
```

### 12. Can workers and sharding be used together?

Yes.

Sharding distributes tests across CI jobs, while workers distribute tests within each job.

### 13. How do you debug a parallel execution issue?

First reduce concurrency:

```bash
npx playwright test --workers=1
```

Then investigate shared state, test data, resources, and timing dependencies.

### 14. Why can increasing workers make tests slower?

Because excessive workers can exhaust CPU, memory, network, database, or application resources.

### 15. How would you design a parallel-safe automation framework?

I would:

```text
Independent Tests
        ↓
Unique Test Data
        ↓
Isolated Browser Contexts
        ↓
Safe Fixtures
        ↓
Controlled Workers
        ↓
CI Sharding
        ↓
Reports + Traces
```

---

# 40. Key Takeaways

```text
Workers
   ↓
Parallel Test Execution
   ↓
Faster Regression
   ↓
Better CI/CD Performance
```

Remember:

* `workers` controls worker processes.
* `fullyParallel` enables aggressive parallel execution.
* `mode: 'parallel'` enables parallel execution inside a describe block.
* `mode: 'serial'` is for tests with unavoidable dependencies.
* Tests should be independent.
* Use unique test data.
* Avoid shared mutable state.
* Worker-scoped fixtures are initialized once per worker.
* Use `testInfo.workerIndex` when worker-specific resources are required.
* Use sharding for very large CI suites.
* Do not blindly maximize worker count.
* Measure performance and choose the optimal concurrency level.

## Recommended File

```text
Parallel-Execution/Playwright-Parallel-Execution.md
```

This topic is particularly important for **Senior QA Automation / SDET interviews** because interviewers often ask how you would design a Playwright framework that can execute a large regression suite efficiently in CI/CD.
