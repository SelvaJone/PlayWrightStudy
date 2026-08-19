# Playwright Test Sharding

## Overview

Playwright Test Sharding allows you to split a large test suite into multiple independent groups, called **shards**, and execute those shards in parallel.

Sharding is especially useful in CI/CD pipelines where you want to reduce the total execution time of a large Playwright test suite.

For example, if you have 1,000 tests, you can divide them into 4 shards:

```text
Shard 1 → Tests 1–250
Shard 2 → Tests 251–500
Shard 3 → Tests 501–750
Shard 4 → Tests 751–1000
```

Each shard can run on a separate CI machine or worker.

---

## Why Use Sharding?

Without sharding:

```text
CI Machine
   |
   +-- Run all tests
          |
          +-- 1,000 tests
          |
          +-- Long execution time
```

With sharding:

```text
                 Test Suite
                     |
          +----------+----------+
          |          |          |
       Shard 1    Shard 2    Shard 3    Shard 4
          |          |          |          |
       CI Job 1   CI Job 2   CI Job 3   CI Job 4
```

Benefits:

* Faster CI execution
* Better utilization of CI machines
* Scales large test suites
* Works well with GitHub Actions, Jenkins, GitLab CI, Azure DevOps, etc.
* Independent CI jobs can execute different shards

---

# Basic Sharding

Playwright provides the `--shard` option.

```bash
npx playwright test --shard=1/4
```

This means:

```text
Run shard 1 out of 4 total shards
```

For four shards:

```bash
npx playwright test --shard=1/4
npx playwright test --shard=2/4
npx playwright test --shard=3/4
npx playwright test --shard=4/4
```

Each command runs a different portion of the test suite.

---

# Sharding Syntax

The syntax is:

```bash
--shard=<current-shard>/<total-shards>
```

Example:

```bash
npx playwright test --shard=2/5
```

Meaning:

```text
Current shard = 2
Total shards = 5
```

Another example:

```bash
npx playwright test --shard=3/10
```

Meaning:

```text
Current shard = 3
Total shards = 10
```

---

# Sharding with Playwright Configuration

You normally do not need to add sharding configuration to `playwright.config.ts`.

For example:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './tests',

  timeout: 30 * 1000,

  fullyParallel: true,

  retries: process.env.CI ? 2 : 0,

  workers: process.env.CI ? 2 : undefined,

  reporter: 'html',

  use: {
    baseURL: 'https://example.com',
    headless: true,
  },
});
```

The shard is normally specified from the command line:

```bash
npx playwright test --shard=1/4
```

---

# Sharding vs Parallel Execution

Sharding and parallel execution are related but different.

## Parallel Execution

Parallel execution runs tests simultaneously using multiple workers on the same machine.

Example:

```text
One CI Machine

Worker 1 → Test A
Worker 2 → Test B
Worker 3 → Test C
Worker 4 → Test D
```

Configured with:

```typescript
workers: 4
```

---

## Sharding

Sharding distributes tests across multiple CI jobs or machines.

```text
CI Job 1 → Shard 1
CI Job 2 → Shard 2
CI Job 3 → Shard 3
CI Job 4 → Shard 4
```

Each shard can also use multiple Playwright workers.

Therefore, you can combine:

```text
Sharding
    +
Parallel Workers
```

---

# Sharding with Multiple Workers

For example:

```bash
npx playwright test --shard=1/4 --workers=4
```

This means:

```text
Shard 1 of 4
     |
     +-- Worker 1
     +-- Worker 2
     +-- Worker 3
     +-- Worker 4
```

Four CI jobs could run:

```bash
npx playwright test --shard=1/4 --workers=4
npx playwright test --shard=2/4 --workers=4
npx playwright test --shard=3/4 --workers=4
npx playwright test --shard=4/4 --workers=4
```

This can significantly reduce execution time for large suites.

---

# Sharding in GitHub Actions

A common approach is to use a matrix strategy.

Example:

```yaml
name: Playwright Tests

on:
  push:
  pull_request:

jobs:
  test:
    timeout-minutes: 60
    runs-on: ubuntu-latest

    strategy:
      fail-fast: false
      matrix:
        shard: [1/4, 2/4, 3/4, 4/4]

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright
        run: npx playwright install --with-deps

      - name: Run Playwright tests
        run: npx playwright test --shard=${{ matrix.shard }}
```

This creates four CI jobs:

```text
Job 1 → 1/4
Job 2 → 2/4
Job 3 → 3/4
Job 4 → 4/4
```

---

# GitHub Actions Matrix

The important part is:

```yaml
strategy:
  matrix:
    shard: [1/4, 2/4, 3/4, 4/4]
```

And:

```yaml
run: npx playwright test --shard=${{ matrix.shard }}
```

GitHub Actions automatically creates separate jobs.

---

# Sharding with Environment Variables

Instead of hardcoding the shard number, you can use environment variables.

Example:

```bash
npx playwright test --shard=$SHARD
```

Environment variable:

```bash
SHARD=1/4
```

For CI:

```yaml
env:
  SHARD: 1/4
```

Then:

```bash
npx playwright test --shard=$SHARD
```

---

# Sharding with npm Scripts

You can also define an npm script.

```json
{
  "scripts": {
    "test": "playwright test",
    "test:shard": "playwright test --shard"
  }
}
```

Run:

```bash
npm run test:shard -- 1/4
```

Another example:

```bash
npm run test:shard -- 2/4
```

---

# Sharding with Projects

Playwright projects can also be combined with sharding.

Example:

```typescript
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
]
```

Run:

```bash
npx playwright test --project=chromium --shard=1/4
```

Or:

```bash
npx playwright test --project=firefox --shard=1/4
```

This allows you to control both:

```text
Browser
+
Shard
```

---

# Sharding and Retries

Sharding works with retries.

Example:

```typescript
retries: process.env.CI ? 2 : 0
```

Run:

```bash
npx playwright test --shard=1/4
```

If a test fails, Playwright can retry it according to the configured retry count.

Example:

```text
Shard 1
   |
   +-- Test A → Pass
   +-- Test B → Fail → Retry → Pass
   +-- Test C → Pass
```

---

# Sharding and Workers

You can combine:

```text
Shards
Workers
Projects
Retries
```

Example:

```bash
npx playwright test \
  --shard=1/4 \
  --workers=4
```

Conceptually:

```text
             Test Suite
                 |
       +---------+---------+
       |         |         |
    Shard 1   Shard 2   Shard 3   Shard 4
       |
   +---+---+---+---+
   |   |   |   |   |
  W1  W2  W3  W4
```

---

# Important Consideration: Test Independence

Sharding works best when tests are independent.

Avoid tests like:

```text
Test 1 creates user
Test 2 depends on Test 1
Test 3 depends on Test 2
```

Because tests may execute in different shards.

Prefer:

```text
Test 1 → Creates its own test data
Test 2 → Creates its own test data
Test 3 → Creates its own test data
```

Each test should be independently executable.

---

# Test Data and Sharding

When using sharding, shared test data can become a problem.

Bad example:

```text
Shard 1 → Updates User A
Shard 2 → Deletes User A
```

This can create race conditions.

Better:

```text
Shard 1 → User A
Shard 2 → User B
Shard 3 → User C
Shard 4 → User D
```

Use unique test data whenever possible.

---

# Database Considerations

If tests use a shared database, sharding can cause conflicts.

For example:

```text
Shard 1 → UPDATE customer SET status='ACTIVE'
Shard 2 → UPDATE customer SET status='INACTIVE'
```

Both shards may modify the same record.

Better approaches include:

* Unique test records
* Test-data factories
* Database cleanup
* Isolated test environments
* Unique IDs
* API-based test-data creation

---

# Authentication and Sharding

Authentication state can usually be reused across tests.

Example:

```typescript
use: {
  storageState: 'playwright/.auth/user.json',
}
```

However, be careful when multiple shards modify the same authenticated account.

For tests that change user-specific data, consider separate accounts or independent test data.

---

# Reporting with Sharding

Each shard generates its own test results.

Example:

```text
Shard 1 → report
Shard 2 → report
Shard 3 → report
Shard 4 → report
```

In CI/CD, you may want to upload each shard's results as an artifact.

Example:

```yaml
- name: Upload Playwright report
  if: always()
  uses: actions/upload-artifact@v4
  with:
    name: playwright-report-${{ matrix.shard }}
    path: playwright-report/
```

This produces separate reports:

```text
playwright-report-1/4
playwright-report-2/4
playwright-report-3/4
playwright-report-4/4
```

---

# Blob Reporter for Sharding

For CI environments, Playwright's blob reporter is useful when you want to merge reports from multiple shards.

Configure:

```typescript
reporter: process.env.CI
  ? [['blob']]
  : [['html']]
```

Each shard produces a blob report.

Example:

```text
Shard 1 → blob report
Shard 2 → blob report
Shard 3 → blob report
Shard 4 → blob report
```

These can then be merged.

---

# Merging Sharded Reports

After all shards finish, merge the blob reports.

Example:

```bash
npx playwright merge-reports blob-report
```

Depending on the CI setup, you can merge the reports into an HTML report.

Example configuration:

```bash
npx playwright merge-reports --reporter html ./all-blob-reports
```

This provides a consolidated report.

---

# Recommended GitHub Actions Pattern

A practical setup is:

```yaml
name: Playwright Tests

on:
  push:
  pull_request:

jobs:
  test:
    timeout-minutes: 60
    runs-on: ubuntu-latest

    strategy:
      fail-fast: false
      matrix:
        shard: [1/4, 2/4, 3/4, 4/4]

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - run: npm ci

      - run: npx playwright install --with-deps

      - name: Run tests
        run: npx playwright test --shard=${{ matrix.shard }}

      - name: Upload blob report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: blob-report-${{ matrix.shard }}
          path: blob-report
```

---

# Choosing the Number of Shards

There is no universal best number.

Consider:

```text
Total tests
Test execution time
CI machine availability
CI cost
Number of workers
Test stability
```

For example:

```text
500 tests
     |
     +-- 2 shards → moderate parallelism
     |
     +-- 4 shards → faster
     |
     +-- 8 shards → potentially faster
```

More shards do not always mean proportionally faster execution.

There is CI startup overhead and infrastructure cost.

---

# Sharding vs Workers vs Projects

| Feature        | Purpose                                             |
| -------------- | --------------------------------------------------- |
| Workers        | Parallel execution on one machine                   |
| Sharding       | Split tests across CI jobs/machines                 |
| Projects       | Run tests against different configurations/browsers |
| Retries        | Re-run failed tests                                 |
| Fully Parallel | Allow tests to run independently in parallel        |

Example:

```text
4 Shards
   +
4 Workers per shard
   +
3 Browser Projects
```

This can create a very high level of parallel execution.

Use it carefully because CI resource consumption can increase significantly.

---

# Common Problems with Sharding

## 1. Tests Depend on Execution Order

Problem:

```text
Test A → creates data
Test B → uses data from Test A
```

Solution:

Make each test independent.

---

## 2. Shared Test Data

Problem:

```text
Shard 1 → modifies record
Shard 2 → modifies same record
```

Solution:

Use unique test data.

---

## 3. Shared Accounts

Problem:

```text
Shard 1 → User logs in
Shard 2 → User changes profile
```

Solution:

Use separate accounts or isolate test data.

---

## 4. Reports Are Split

Problem:

```text
Shard 1 → Report 1
Shard 2 → Report 2
Shard 3 → Report 3
Shard 4 → Report 4
```

Solution:

Use blob reports and merge them after all shards complete.

---

## 5. CI Job Fails

If one shard fails:

```text
Shard 1 → PASS
Shard 2 → PASS
Shard 3 → FAIL
Shard 4 → PASS
```

The overall pipeline should normally be considered failed.

Investigate the failed shard and its report.

---

# Interview Questions

### 1. What is Playwright sharding?

Sharding divides a Playwright test suite into multiple groups and executes those groups independently, usually across multiple CI jobs.

---

### 2. How do you run the first shard out of four?

```bash
npx playwright test --shard=1/4
```

---

### 3. What does `--shard=3/5` mean?

It means:

```text
Current shard = 3
Total shards = 5
```

---

### 4. What is the difference between sharding and workers?

**Workers** execute tests in parallel on the same machine.

**Sharding** distributes the test suite across multiple CI jobs or machines.

---

### 5. Can sharding and workers be used together?

Yes.

Example:

```bash
npx playwright test --shard=1/4 --workers=4
```

---

### 6. Can sharding be used in GitHub Actions?

Yes. A matrix strategy is commonly used:

```yaml
matrix:
  shard: [1/4, 2/4, 3/4, 4/4]
```

---

### 7. What is the biggest concern when using sharding?

Tests should be independent and should not rely on shared mutable data or execution order.

---

### 8. How do you merge reports from multiple shards?

Use Playwright's blob reporter and merge the generated blob reports after the shard jobs complete.

---

### 9. Does sharding always make tests faster?

Not necessarily.

The result depends on:

* Test suite size
* CI resources
* Number of shards
* Test distribution
* Startup overhead
* External dependencies

---

### 10. What happens if one shard fails?

The CI pipeline should normally fail, even if the other shards pass.

Example:

```text
Shard 1 → PASS
Shard 2 → PASS
Shard 3 → FAIL
Shard 4 → PASS

Overall → FAIL
```

---

# Best Practices

1. Keep tests independent.
2. Avoid shared mutable test data.
3. Use unique test records.
4. Combine sharding with workers carefully.
5. Use CI matrix strategies.
6. Use blob reports for consolidated reporting.
7. Upload reports from every shard.
8. Use `fail-fast: false` so all shards can finish.
9. Monitor CI execution time before and after introducing sharding.
10. Do not create more shards than your CI infrastructure can efficiently support.

---

# Example Project Structure

```text
playwright-project/
│
├── tests/
│   ├── login.spec.ts
│   ├── dashboard.spec.ts
│   ├── profile.spec.ts
│   └── checkout.spec.ts
│
├── playwright.config.ts
├── package.json
│
├── playwright-report/
│
├── blob-report/
│
└── .github/
    └── workflows/
        └── playwright.yml
```

---

# Quick Reference

Run all tests:

```bash
npx playwright test
```

Run first shard of four:

```bash
npx playwright test --shard=1/4
```

Run second shard:

```bash
npx playwright test --shard=2/4
```

Run third shard:

```bash
npx playwright test --shard=3/4
```

Run fourth shard:

```bash
npx playwright test --shard=4/4
```

Shard with workers:

```bash
npx playwright test --shard=1/4 --workers=4
```

Run a specific project with sharding:

```bash
npx playwright test --project=chromium --shard=1/4
```

---

# Summary

Playwright Test Sharding is a powerful technique for speeding up large test suites in CI/CD.

The key command is:

```bash
npx playwright test --shard=1/4
```

The general model is:

```text
                Playwright Test Suite
                         |
          +--------------+--------------+
          |              |              |
       Shard 1        Shard 2        Shard 3        Shard 4
          |              |              |              |
       CI Job 1       CI Job 2       CI Job 3       CI Job 4
          |              |              |              |
       Workers        Workers        Workers        Workers
```

The most important concepts to remember are:

```text
Workers  → Parallel execution on one machine

Sharding → Distribution across CI jobs/machines

Projects → Different browsers/configurations

Retries  → Re-run failed tests

Blob Reporter → Combine results from shards
```

For a large Playwright automation framework, **sharding + workers + CI matrix execution** can dramatically reduce overall test execution time while maintaining scalable test automation.
