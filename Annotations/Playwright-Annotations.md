# Playwright Annotations

Playwright annotations allow you to add metadata and control how tests are executed. They are useful for skipping tests, marking expected failures, increasing timeouts, organizing tests, adding tags, and providing additional information about test execution.

---

## 1. What Are Playwright Annotations?

Annotations provide additional information or instructions for a Playwright test.

Common annotations include:

| Annotation          | Purpose                           |
| ------------------- | --------------------------------- |
| `test.skip()`       | Skip a test                       |
| `test.fixme()`      | Mark a test as needing a fix      |
| `test.fail()`       | Mark a test as expected to fail   |
| `test.slow()`       | Triple the default timeout        |
| `test.only()`       | Run only selected tests           |
| `test.setTimeout()` | Set a custom timeout              |
| `test.describe()`   | Group tests                       |
| `test.info()`       | Access test execution information |

Example:

```java
// Not Java - Playwright uses JavaScript/TypeScript APIs
```

Playwright examples in this document use **TypeScript**.

---

# 2. test.skip()

`test.skip()` prevents a test from running.

```typescript
import { test, expect } from '@playwright/test';

test('Login Test', async ({ page }) => {
  test.skip();

  await page.goto('https://example.com');
});
```

The test is reported as **skipped**.

---

## 3. Conditional test.skip()

You can skip a test based on a condition.

```typescript
test('Mobile Test', async ({ page, browserName }) => {
  test.skip(browserName === 'chromium', 'This test is not supported on Chromium');

  await page.goto('https://example.com');
});
```

This is useful when a test should run only on specific browsers.

---

# 4. test.skip() at Describe Level

You can skip an entire group of tests.

```typescript
test.describe('Payment Tests', () => {

  test.skip();

  test('Credit Card Payment', async ({ page }) => {
    // Test code
  });

  test('PayPal Payment', async ({ page }) => {
    // Test code
  });

});
```

All tests inside the `describe` block are skipped.

---

# 5. Conditional Skip Based on Browser

Example:

```typescript
test('Safari-specific test', async ({ page, browserName }) => {
  test.skip(
    browserName !== 'webkit',
    'This test runs only on WebKit'
  );

  await page.goto('https://example.com');
});
```

This test runs only when the browser is WebKit.

---

# 6. Conditional Skip Based on Environment

You can skip tests depending on environment variables.

```typescript
test('Production-only Test', async ({ page }) => {
  test.skip(
    process.env.TEST_ENV !== 'production',
    'Runs only in production'
  );

  await page.goto('https://example.com');
});
```

Run with:

```bash
TEST_ENV=production npx playwright test
```

On Windows PowerShell:

```powershell
$env:TEST_ENV="production"
npx playwright test
```

---

# 7. test.fixme()

`test.fixme()` marks a test as needing to be fixed.

```typescript
test('Broken Login Test', async ({ page }) => {
  test.fixme('Application defect needs to be fixed');

  await page.goto('https://example.com');
});
```

The test will not execute.

This is useful for known broken functionality.

---

# 8. Conditional test.fixme()

```typescript
test('WebKit Login Test', async ({ page, browserName }) => {
  test.fixme(
    browserName === 'webkit',
    'Known WebKit issue'
  );

  await page.goto('https://example.com');
});
```

The test is marked as `fixme` only when running on WebKit.

---

# 9. test.fail()

`test.fail()` indicates that a test is expected to fail.

```typescript
test('Known Bug Test', async ({ page }) => {
  test.fail();

  await page.goto('https://example.com');

  await expect(page.getByText('Known Bug')).toBeVisible();
});
```

Playwright expects the test to fail.

If the test unexpectedly passes, Playwright reports an unexpected pass.

---

# 10. test.fail() With a Condition

```typescript
test('Known Browser Bug', async ({ page, browserName }) => {
  test.fail(
    browserName === 'webkit',
    'Known WebKit defect'
  );

  await page.goto('https://example.com');

  await expect(page.getByText('Some Element')).toBeVisible();
});
```

The test is expected to fail only on WebKit.

---

# 11. test.slow()

`test.slow()` marks a test as slow.

Playwright increases the test timeout.

```typescript
test('Slow API Test', async ({ page }) => {
  test.slow();

  await page.goto('https://example.com');

  // Long-running operations
});
```

By default, Playwright triples the timeout.

If the normal timeout is:

```text
30 seconds
```

The slow test gets approximately:

```text
90 seconds
```

---

# 12. test.slow() With a Condition

```typescript
test('Slow Test', async ({ page, browserName }) => {
  test.slow(
    browserName === 'webkit',
    'WebKit execution is slower'
  );

  await page.goto('https://example.com');
});
```

---

# 13. test.setTimeout()

Use `test.setTimeout()` when you need a specific timeout.

```typescript
test('Long Running Test', async ({ page }) => {
  test.setTimeout(120000);

  await page.goto('https://example.com');
});
```

Timeout:

```text
120000 ms = 120 seconds
```

---

# 14. test.setTimeout() Inside Describe

You can configure the timeout for all tests in a test group.

```typescript
test.describe('Checkout Tests', () => {

  test.setTimeout(120000);

  test('Add Product', async ({ page }) => {
    // Test
  });

  test('Complete Checkout', async ({ page }) => {
    // Test
  });

});
```

All tests inside the group receive the timeout.

---

# 15. test.only()

`test.only()` runs only the selected test.

```typescript
test.only('Login Test', async ({ page }) => {
  await page.goto('https://example.com');
});
```

Other tests in the file will not run.

### Important

Do not commit `test.only()` to your Git repository accidentally.

For example:

```typescript
test.only('Login Test', async ({ page }) => {
});
```

can cause the CI pipeline to execute only that test.

---

# 16. describe.only()

You can run only a particular group.

```typescript
test.describe.only('Login Tests', () => {

  test('Valid Login', async ({ page }) => {
  });

  test('Invalid Login', async ({ page }) => {
  });

});
```

Only tests inside this group execute.

---

# 17. test.describe()

`test.describe()` groups related tests.

```typescript
test.describe('Login Tests', () => {

  test('Valid Login', async ({ page }) => {
  });

  test('Invalid Login', async ({ page }) => {
  });

  test('Forgot Password', async ({ page }) => {
  });

});
```

This improves organization and reporting.

---

# 18. Nested describe()

You can create nested groups.

```typescript
test.describe('Authentication', () => {

  test.describe('Login', () => {

    test('Valid Login', async ({ page }) => {
    });

    test('Invalid Login', async ({ page }) => {
    });

  });

  test.describe('Password Reset', () => {

    test('Forgot Password', async ({ page }) => {
    });

  });

});
```

---

# 19. test.info()

`test.info()` provides information about the current test execution.

```typescript
test('Test Information', async ({ page }) => {

  const testInfo = test.info();

  console.log(testInfo.title);
  console.log(testInfo.status);
  console.log(testInfo.project.name);

});
```

Useful properties include:

```typescript
test.info().title
test.info().status
test.info().project.name
test.info().retry
test.info().workerIndex
test.info().parallelIndex
```

---

# 20. Adding Attachments With test.info()

You can attach files or data to the test report.

```typescript
test('Attach Data', async ({ page }) => {

  await test.info().attach('test-data', {
    body: JSON.stringify({
      username: 'testuser',
      environment: 'stage'
    }),
    contentType: 'application/json'
  });

});
```

Attachments can be useful for debugging and reporting.

---

# 21. Adding Tags

Playwright supports tags for test organization.

Example:

```typescript
test(
  'Login Test',
  {
    tag: '@smoke'
  },
  async ({ page }) => {

    await page.goto('https://example.com');

  }
);
```

Multiple tags:

```typescript
test(
  'Checkout Test',
  {
    tag: ['@smoke', '@regression']
  },
  async ({ page }) => {

    await page.goto('https://example.com');

  }
);
```

---

# 22. Running Tests by Tag

Run smoke tests:

```bash
npx playwright test --grep @smoke
```

Run regression tests:

```bash
npx playwright test --grep @regression
```

Exclude a tag:

```bash
npx playwright test --grep-invert @slow
```

---

# 23. Tags With Describe

You can also tag a group.

```typescript
test.describe(
  'Login Tests',
  {
    tag: '@smoke'
  },
  () => {

    test('Valid Login', async ({ page }) => {
    });

    test('Invalid Login', async ({ page }) => {
    });

  }
);
```

All tests in the group inherit the tag.

---

# 24. Annotations

Playwright also supports custom annotations.

```typescript
test(
  'Customer Login',
  {
    annotation: {
      type: 'JIRA',
      description: 'QA-1234'
    }
  },
  async ({ page }) => {

    await page.goto('https://example.com');

  }
);
```

Multiple annotations:

```typescript
test(
  'Customer Login',
  {
    annotation: [
      {
        type: 'JIRA',
        description: 'QA-1234'
      },
      {
        type: 'OWNER',
        description: 'QA Team'
      }
    ]
  },
  async ({ page }) => {

    await page.goto('https://example.com');

  }
);
```

---

# 25. Reading Annotations

You can access annotations through `test.info()`.

```typescript
test('Customer Login', async ({ page }) => {

  const annotations = test.info().annotations;

  console.log(annotations);

});
```

---

# 26. Combining Tags and Annotations

A real-world example:

```typescript
test(
  'Customer Login',
  {
    tag: ['@smoke', '@login'],
    annotation: [
      {
        type: 'JIRA',
        description: 'QA-1234'
      },
      {
        type: 'OWNER',
        description: 'Automation Team'
      }
    ]
  },
  async ({ page }) => {

    await page.goto('https://example.com');

  }
);
```

This gives the test:

```text
Tags:
@smoke
@login

Annotations:
JIRA = QA-1234
OWNER = Automation Team
```

---

# 27. Real-World Example

```typescript
import { test, expect } from '@playwright/test';

test.describe(
  'Login Tests',
  {
    tag: '@login'
  },
  () => {

    test(
      'Valid Login',
      {
        tag: '@smoke',
        annotation: {
          type: 'JIRA',
          description: 'QA-1001'
        }
      },
      async ({ page }) => {

        await page.goto('https://example.com');

        await expect(page).toHaveTitle(/Example/);
      }
    );

    test(
      'Known Login Bug',
      {
        tag: '@regression'
      },
      async ({ page }) => {

        test.fail('Known application defect');

        await page.goto('https://example.com');

      }
    );

    test(
      'Slow Login Test',
      async ({ page }) => {

        test.slow();

        await page.goto('https://example.com');

      }
    );

  }
);
```

---

# 28. Annotations Based on Project

You can apply annotations based on the Playwright project.

```typescript
test('Browser-specific Test', async ({ page, browserName }) => {

  test.skip(
    browserName === 'firefox',
    'Not supported on Firefox'
  );

  await page.goto('https://example.com');

});
```

This is particularly useful for:

```text
Chromium
Firefox
WebKit
Mobile Chrome
Mobile Safari
```

---

# 29. Annotations Based on Test Data

Example:

```typescript
const environment = process.env.TEST_ENV;

test('Environment Test', async ({ page }) => {

  test.skip(
    environment === 'production',
    'Not safe to execute in production'
  );

  await page.goto('https://example.com');

});
```

This helps prevent destructive tests from accidentally running against production.

---

# 30. Annotations and CI/CD

Annotations are very useful in CI/CD pipelines.

Example:

```typescript
test(
  'Smoke Test',
  {
    tag: '@smoke'
  },
  async ({ page }) => {

    await page.goto('https://example.com');

  }
);
```

CI command:

```bash
npx playwright test --grep @smoke
```

This allows the pipeline to execute only smoke tests.

---

# 31. Common Annotation Strategy

A professional Playwright framework can use tags like:

```text
@smoke
@regression
@sanity
@api
@ui
@login
@checkout
@mobile
@critical
```

Example:

```typescript
test(
  'Login Test',
  {
    tag: ['@smoke', '@regression', '@login']
  },
  async ({ page }) => {

    // Test implementation

  }
);
```

---

# 32. Skip vs Fixme vs Fail

These annotations are commonly confused.

### test.skip()

Use when:

```text
Do not execute this test.
```

Example:

```typescript
test.skip();
```

### test.fixme()

Use when:

```text
The test is currently broken and needs fixing.
```

Example:

```typescript
test.fixme('Application defect');
```

### test.fail()

Use when:

```text
The test is expected to fail.
```

Example:

```typescript
test.fail('Known bug');
```

---

# 33. Important Difference Between skip() and fixme()

Both prevent normal execution, but their purpose is different.

```text
skip()
    ↓
Test intentionally skipped

fixme()
    ↓
Test needs to be fixed
```

Use meaningful reasons:

```typescript
test.skip('Feature not available in this environment');
```

```typescript
test.fixme('Blocked by application defect QA-1234');
```

---

# 34. Best Practices

### 1. Do not leave test.only() in source control

Avoid:

```typescript
test.only('Login', async ({ page }) => {
});
```

before committing.

---

### 2. Always provide a reason

Prefer:

```typescript
test.skip(
  browserName === 'firefox',
  'Feature is not supported on Firefox'
);
```

instead of:

```typescript
test.skip(browserName === 'firefox');
```

---

### 3. Use tags consistently

Create a standard tagging strategy:

```text
@smoke
@regression
@sanity
@critical
```

---

### 4. Avoid unnecessary skipping

Do not skip tests just because they are temporarily failing.

Investigate the failure first.

---

### 5. Use fixme for known defects

```typescript
test.fixme('Blocked by QA-1234');
```

This makes the reason visible to the team.

---

### 6. Use fail for expected failures

```typescript
test.fail('Expected failure until defect is fixed');
```

This is better than allowing a known failing test to make the entire suite appear broken.

---

# 35. Interview Questions

### Q1. What is `test.skip()`?

It prevents a test from executing.

---

### Q2. What is `test.fixme()`?

It marks a test as needing a fix and prevents normal execution.

---

### Q3. What is `test.fail()`?

It tells Playwright that the test is expected to fail.

---

### Q4. What is `test.slow()`?

It marks a test as slow and increases its timeout.

---

### Q5. What is `test.only()`?

It executes only the selected test or test group.

---

### Q6. What is `test.describe()`?

It groups related tests together.

---

### Q7. How do you skip a test for Firefox?

```typescript
test.skip(
  browserName === 'firefox',
  'Not supported on Firefox'
);
```

---

### Q8. How do you run only smoke tests?

```bash
npx playwright test --grep @smoke
```

---

### Q9. How do you exclude regression tests?

```bash
npx playwright test --grep-invert @regression
```

---

### Q10. How do you add a Jira ID to a test?

```typescript
annotation: {
  type: 'JIRA',
  description: 'QA-1234'
}
```

---

# 36. Recommended Framework Usage

A senior-level Playwright framework can combine annotations like this:

```typescript
import { test, expect } from '@playwright/test';

test.describe(
  'Authentication',
  {
    tag: '@authentication'
  },
  () => {

    test(
      'Valid Login',
      {
        tag: ['@smoke', '@critical'],
        annotation: {
          type: 'JIRA',
          description: 'QA-1001'
        }
      },
      async ({ page }) => {

        await page.goto('https://example.com');

        await expect(page).toHaveTitle(/Example/);

      }
    );

    test(
      'Known Login Defect',
      {
        tag: '@regression'
      },
      async ({ page }) => {

        test.fail('Blocked by known application defect');

        await page.goto('https://example.com');

      }
    );

    test(
      'Unsupported Browser Test',
      async ({ page, browserName }) => {

        test.skip(
          browserName === 'webkit',
          'Feature is not supported on WebKit'
        );

        await page.goto('https://example.com');

      }
    );

  }
);
```

---

# 37. Summary

Playwright annotations are important for controlling and organizing test execution.

The most important ones to remember are:

```text
test.skip()
test.fixme()
test.fail()
test.slow()
test.only()
test.describe()
test.setTimeout()
test.info()
```

For framework-level test selection, tags are especially useful:

```typescript
tag: '@smoke'
```

Run:

```bash
npx playwright test --grep @smoke
```

For real-world automation frameworks, a combination of **tags + annotations + conditional execution** provides clean test organization and flexible CI/CD execution.

---

## Key Takeaway

```text
skip()     → Don't run the test
fixme()    → Test needs fixing
fail()     → Test is expected to fail
slow()     → Increase timeout
only()     → Run only this test
describe() → Group tests
setTimeout → Set custom timeout
info()     → Get test execution information
tags       → Categorize and filter tests
```
