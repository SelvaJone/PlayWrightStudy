# Playwright Retries

Retries allow Playwright Test to automatically re-run a failed test. They are useful for handling temporary or intermittent failures, especially in CI/CD environments.

---

## 1. What Are Retries?

A retry means Playwright runs a failed test again.

For example:

```text
Test Attempt 1 → Failed
Test Attempt 2 → Passed
```

Playwright will report the test as **flaky** when it fails initially but passes on retry.

Retries are configured using the `retries` option.

---

## 2. Configure Retries Globally

Retries can be configured in `playwright.config.ts`.

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  retries: 2,
});
```

This means:

```text
Initial attempt + up to 2 retries
```

So a test can run a maximum of **3 times**.

---

## 3. Configure Retries for CI Only

A common practice is to enable retries in CI but not during local development.

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  retries: process.env.CI ? 2 : 0,
});
```

Behavior:

```text
Local machine → 0 retries
CI/CD          → 2 retries
```

This is a recommended approach for many automation frameworks.

---

## 4. Configure Retries for a Project

Retries can also be configured for an individual Playwright project.

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  projects: [
    {
      name: 'chromium',
      use: {
        browserName: 'chromium',
      },
      retries: 2,
    },

    {
      name: 'firefox',
      use: {
        browserName: 'firefox',
      },
      retries: 1,
    },
  ],
});
```

Here:

```text
Chromium → 2 retries
Firefox  → 1 retry
```

---

## 5. Configure Retries for a Test

Retries can be configured for a specific test using `test.describe.configure()`.

```typescript
import { test, expect } from '@playwright/test';

test.describe.configure({ retries: 2 });

test('Login test', async ({ page }) => {
  await page.goto('https://example.com');

  await expect(page).toHaveTitle(/Example/);
});
```

The tests inside the `describe` block can be retried according to the configured value.

---

## 6. Configure Retries for a Test Group

```typescript
import { test, expect } from '@playwright/test';

test.describe('Login Tests', () => {
  test.describe.configure({ retries: 2 });

  test('Valid login', async ({ page }) => {
    await page.goto('https://example.com/login');

    await expect(page.locator('#username')).toBeVisible();
  });

  test('Invalid login', async ({ page }) => {
    await page.goto('https://example.com/login');

    await expect(page.locator('#error')).toBeVisible();
  });
});
```

Both tests can be retried up to two times.

---

## 7. Retry Behavior

Suppose:

```typescript
retries: 2
```

The execution can look like:

```text
Attempt 1 → Failed
Attempt 2 → Failed
Attempt 3 → Passed
```

Playwright stops retrying as soon as the test passes.

Another example:

```text
Attempt 1 → Failed
Attempt 2 → Passed
```

There is no third execution.

---

## 8. Retry Status

Playwright provides information about the current retry using `testInfo.retry`.

```typescript
import { test } from '@playwright/test';

test('Retry example', async ({ page }, testInfo) => {
  console.log('Retry number:', testInfo.retry);

  await page.goto('https://example.com');
});
```

Possible values:

```text
0 → Initial test execution
1 → First retry
2 → Second retry
```

---

## 9. Detecting a Retry

You can use `testInfo.retry` to determine whether the test is being retried.

```typescript
import { test } from '@playwright/test';

test('Detect retry', async ({ page }, testInfo) => {
  if (testInfo.retry > 0) {
    console.log('This test is being retried');
  }

  await page.goto('https://example.com');
});
```

---

## 10. Retry-Specific Logic

Sometimes you may want to perform additional diagnostics when a test is retried.

```typescript
import { test } from '@playwright/test';

test('Retry diagnostics', async ({ page }, testInfo) => {
  if (testInfo.retry > 0) {
    console.log('Retry detected');
    console.log('Retry number:', testInfo.retry);
  }

  await page.goto('https://example.com');
});
```

This can be useful for debugging flaky tests.

---

## 11. Retries and Hooks

Retries affect the entire test execution lifecycle.

For example:

```typescript
import { test } from '@playwright/test';

test.beforeEach(async ({ page }) => {
  console.log('Before each');
});

test.afterEach(async () => {
  console.log('After each');
});

test('Example test', async ({ page }) => {
  await page.goto('https://example.com');
});
```

If the test is retried, the test's setup and teardown lifecycle is executed again for the retry.

This helps ensure that each retry gets a clean test environment.

---

## 12. Retries and Fixtures

Fixtures are also created again as required for each retry.

Example:

```typescript
import { test, expect } from '@playwright/test';

test('Fixture retry example', async ({ page }) => {
  await page.goto('https://example.com');

  await expect(page.locator('h1')).toBeVisible();
});
```

The Playwright test runner manages fixture setup and cleanup during each test attempt.

---

## 13. Retries and Workers

Playwright workers may be restarted after failures.

This is important when running tests in parallel.

For example:

```text
Worker 1
  Test A → Passed
  Test B → Failed

Retry
  Worker restarted if required
  Test B → Retried
```

Therefore, tests should not depend on state left behind by another test.

---

## 14. Retries in CI/CD

Retries are especially useful in CI/CD environments.

Example:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  retries: process.env.CI ? 2 : 0,
});
```

This gives developers faster local feedback while allowing CI to tolerate temporary failures.

---

## 15. Running Tests with Retries

You can run Playwright tests normally:

```bash
npx playwright test
```

If retries are configured:

```text
Test failed
↓
Retry 1
↓
Retry 2
↓
Final result
```

---

## 16. Retry with Workers

Retries can be combined with parallel execution.

```bash
npx playwright test --workers=4
```

Configuration:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  workers: process.env.CI ? 4 : undefined,
  retries: process.env.CI ? 2 : 0,
});
```

---

## 17. Retries with Sharding

Retries can also be used with Playwright sharding.

Example:

```bash
npx playwright test --shard=1/4
```

Configuration:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  retries: process.env.CI ? 2 : 0,
});
```

Each shard executes its assigned tests, and failed tests can be retried according to the retry configuration.

---

## 18. Retries and Flaky Tests

Retries are useful for identifying flaky tests.

Example:

```text
Initial run → Failed
Retry        → Passed
```

Playwright reports the test as:

```text
flaky
```

This is different from a consistently failing test.

### Flaky

```text
Failed → Passed
```

### Failed

```text
Failed → Failed → Failed
```

### Passed

```text
Passed
```

---

## 19. Why Flaky Tests Are Dangerous

Retries should not be used to hide real defects.

For example:

```typescript
retries: 5
```

may make a test appear more reliable, but it can also hide:

* Application defects
* Timing problems
* Poor synchronization
* Incorrect test data
* Environment instability
* Race conditions
* Test isolation problems

Retries should be a safety mechanism, not a replacement for fixing flaky tests.

---

## 20. Recommended Retry Configuration

A common configuration is:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  retries: process.env.CI ? 2 : 0,

  workers: process.env.CI ? 4 : undefined,

  use: {
    trace: 'on-first-retry',
  },
});
```

This is useful because tracing can automatically capture additional information when a retry occurs.

---

## 21. Trace on First Retry

One of the most useful Playwright configurations is:

```typescript
use: {
  trace: 'on-first-retry',
}
```

Example:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  retries: process.env.CI ? 2 : 0,

  use: {
    trace: 'on-first-retry',
  },
});
```

Behavior:

```text
Initial attempt → Failed
First retry     → Trace collected
Second retry    → No additional trace
```

This provides useful debugging information without generating traces for every test.

---

## 22. Screenshot on Retry

Screenshots can also be configured.

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  retries: process.env.CI ? 2 : 0,

  use: {
    screenshot: 'only-on-failure',
  },
});
```

This captures screenshots when tests fail.

---

## 23. Video on Retry

Video recording can also be configured.

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  retries: process.env.CI ? 2 : 0,

  use: {
    video: 'on-first-retry',
  },
});
```

This is useful for diagnosing intermittent UI failures.

---

## 24. Recommended CI Configuration

A practical configuration is:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  retries: process.env.CI ? 2 : 0,

  workers: process.env.CI ? 4 : undefined,

  use: {
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'on-first-retry',
  },
});
```

This provides:

```text
Local:
  Retries = 0

CI:
  Retries = 2
  Parallel workers = 4
  Trace = First retry
  Screenshot = Failure
  Video = First retry
```

---

## 25. Retry and Test Data

Tests using retries should be independent.

Avoid relying on shared mutable data.

Bad example:

```text
Test 1 creates user
Test 2 modifies user
Test 3 deletes user
```

If Test 2 is retried, the user may already have been modified.

A better approach is to create test data independently for each test.

---

## 26. Retry and Database State

If a test modifies database data, retry behavior can create unexpected results.

For example:

```text
Attempt 1:
Create customer
Update customer
Test fails

Retry:
Create customer again
```

The second attempt may fail because the customer already exists.

Use unique test data where appropriate.

Example:

```typescript
const customerId = `customer-${Date.now()}`;
```

---

## 27. Retry and API Tests

Retries are also applicable to Playwright API tests.

```typescript
import { test, expect } from '@playwright/test';

test('API retry example', async ({ request }) => {
  const response = await request.get('/api/users');

  expect(response.ok()).toBeTruthy();
});
```

If the test fails and retries are configured, Playwright can rerun the test.

However, be careful with APIs that perform state-changing operations such as:

```text
POST
PUT
PATCH
DELETE
```

Repeating these operations may have side effects.

---

## 28. Retry and Authentication

Authentication setup should be designed carefully when retries are enabled.

For example:

```typescript
test.beforeEach(async ({ page }) => {
  await page.goto('/login');

  await page.fill('#username', 'user');
  await page.fill('#password', 'password');

  await page.click('#login');
});
```

Each retry should be able to establish the required authenticated state independently.

---

## 29. Retry and Timeouts

Retries do not replace proper timeout configuration.

Bad approach:

```typescript
retries: 5
```

instead of fixing a synchronization issue.

Better approach:

```typescript
await expect(page.locator('#result')).toBeVisible();
```

instead of:

```typescript
await page.waitForTimeout(5000);
```

Use Playwright's auto-waiting and web-first assertions whenever possible.

---

## 30. Retry and Explicit Waits

Avoid using retries to compensate for unnecessary hard waits.

Bad:

```typescript
await page.waitForTimeout(10000);
```

Better:

```typescript
await expect(page.locator('#dashboard')).toBeVisible();
```

Retries should handle occasional environmental failures, not poor synchronization.

---

## 31. Retry Reporting

Playwright's HTML report provides information about test attempts.

Run:

```bash
npx playwright show-report
```

The report can help identify:

* Failed tests
* Passed tests
* Flaky tests
* Retry attempts
* Errors
* Traces
* Screenshots
* Videos

---

## 32. Retry Status in Custom Reporting

You can access retry information using `testInfo`.

```typescript
import { test } from '@playwright/test';

test('Custom retry information', async ({ page }, testInfo) => {
  console.log(`Test: ${testInfo.title}`);
  console.log(`Retry: ${testInfo.retry}`);
});
```

Example output:

```text
Test: Login test
Retry: 0
```

On the first retry:

```text
Test: Login test
Retry: 1
```

---

## 33. Maximum Retry Count

Suppose:

```typescript
retries: 3
```

The maximum number of executions is:

```text
1 initial attempt
+
3 retries
=
4 total attempts
```

Therefore:

```text
retries: 0 → Maximum 1 execution
retries: 1 → Maximum 2 executions
retries: 2 → Maximum 3 executions
retries: 3 → Maximum 4 executions
```

---

## 34. Retry Strategy for Automation Frameworks

A good enterprise strategy is:

```text
Local Development
        ↓
Retries = 0
        ↓
Fix failures immediately

CI/CD
        ↓
Retries = 1 or 2
        ↓
Identify flaky tests
        ↓
Investigate failures
        ↓
Fix root cause
```

---

## 35. Recommended Best Practices

### Use retries carefully

```typescript
retries: 2
```

is generally more reasonable than:

```typescript
retries: 10
```

### Enable retries primarily in CI

```typescript
retries: process.env.CI ? 2 : 0
```

### Capture diagnostics

```typescript
use: {
  trace: 'on-first-retry',
  screenshot: 'only-on-failure',
  video: 'on-first-retry',
}
```

### Fix flaky tests

Do not continuously increase the retry count.

### Keep tests independent

Each test should be able to run independently.

### Use unique test data

This prevents retry attempts from conflicting with previous attempts.

### Avoid hard waits

Prefer Playwright auto-waiting and assertions.

---

# Interview Questions

## 1. What are retries in Playwright?

Retries allow Playwright Test to rerun a failed test automatically.

---

## 2. How do you configure retries?

```typescript
export default defineConfig({
  retries: 2,
});
```

---

## 3. How do you enable retries only in CI?

```typescript
retries: process.env.CI ? 2 : 0
```

---

## 4. What does `retries: 2` mean?

It means the test gets:

```text
1 initial attempt
+
2 retry attempts
=
3 maximum executions
```

---

## 5. How can you identify the current retry?

Use:

```typescript
testInfo.retry
```

---

## 6. What is the value of `testInfo.retry` on the first execution?

```text
0
```

---

## 7. What is the value on the first retry?

```text
1
```

---

## 8. What is a flaky test?

A test that initially fails but passes when retried is considered flaky.

Example:

```text
Failed → Passed
```

---

## 9. Should retries be used to hide test failures?

No.

Retries should help identify intermittent failures, while the root cause of flaky tests should still be fixed.

---

## 10. How do you capture a trace when a test is retried?

```typescript
use: {
  trace: 'on-first-retry',
}
```

---

## 11. How do you capture video on the first retry?

```typescript
use: {
  video: 'on-first-retry',
}
```

---

## 12. How do you capture screenshots when tests fail?

```typescript
use: {
  screenshot: 'only-on-failure',
}
```

---

## 13. Can retries be configured for a project?

Yes.

```typescript
projects: [
  {
    name: 'chromium',
    retries: 2,
  },
]
```

---

## 14. Can retries be configured for a test group?

Yes.

```typescript
test.describe.configure({ retries: 2 });
```

---

## 15. Why are retries useful in CI/CD?

CI environments can experience temporary issues such as:

* Network instability
* Browser startup problems
* Environment delays
* Service availability issues
* Infrastructure instability

Retries can reduce false failures caused by temporary problems.

---

# Example Enterprise Configuration

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './tests',

  retries: process.env.CI ? 2 : 0,

  workers: process.env.CI ? 4 : undefined,

  reporter: [
    ['html'],
    ['list'],
  ],

  use: {
    baseURL: 'https://example.com',

    trace: 'on-first-retry',

    screenshot: 'only-on-failure',

    video: 'on-first-retry',
  },
});
```

This is a strong starting point for an enterprise Playwright framework.

---

# Key Takeaways

```text
Retries
   ↓
Automatically rerun failed tests
   ↓
Useful for CI/CD
   ↓
Use 1–2 retries in most cases
   ↓
Use testInfo.retry to identify attempts
   ↓
Capture trace/video on retry
   ↓
Investigate flaky tests
   ↓
Do not use retries to hide defects
```

The most commonly recommended configuration is:

```typescript
retries: process.env.CI ? 2 : 0
```

combined with:

```typescript
use: {
  trace: 'on-first-retry',
  screenshot: 'only-on-failure',
  video: 'on-first-retry',
}
```

This provides a good balance between **test reliability, CI stability, execution time, and debugging capability**.
