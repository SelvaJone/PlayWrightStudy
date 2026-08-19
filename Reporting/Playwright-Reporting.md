# Playwright Reporting

## 1. Introduction

Playwright provides built-in test reporting capabilities that help us understand:

* Which tests passed or failed
* Test execution duration
* Error messages
* Screenshots
* Videos
* Traces
* Test steps
* Retry information
* Test results in CI/CD pipelines

The default Playwright reporter is the **HTML Reporter**.

---

# 2. Default HTML Reporter

Playwright automatically generates an HTML report after test execution.

Run:

```bash
npx playwright test
```

Then open the report:

```bash
npx playwright show-report
```

The report is usually generated under:

```text
playwright-report/
```

Typical structure:

```text
playwright-report/
├── index.html
├── data/
└── trace/
```

The HTML report provides a visual interface for reviewing test execution.

---

# 3. Running Tests and Opening the Report

Run all tests:

```bash
npx playwright test
```

Open the report:

```bash
npx playwright show-report
```

Run a specific test file:

```bash
npx playwright test tests/login.spec.ts
```

Open the report:

```bash
npx playwright show-report
```

---

# 4. Configure HTML Reporter

Reporter configuration is defined in:

```text
playwright.config.ts
```

Example:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  reporter: 'html',
});
```

This enables the HTML reporter.

---

# 5. HTML Reporter with Configuration

We can configure the HTML reporter.

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  reporter: [
    [
      'html',
      {
        outputFolder: 'playwright-report',
        open: 'never',
      },
    ],
  ],
});
```

### Important options

| Option         | Description                                  |
| -------------- | -------------------------------------------- |
| `outputFolder` | Directory where report is generated          |
| `open`         | Controls when the report automatically opens |

---

# 6. HTML Reporter `open` Options

The `open` option supports:

```text
always
never
on-failure
```

Example:

```typescript
reporter: [
  [
    'html',
    {
      open: 'on-failure',
    },
  ],
],
```

The report opens automatically when tests fail.

For CI/CD, normally use:

```typescript
open: 'never'
```

Example:

```typescript
reporter: [
  [
    'html',
    {
      outputFolder: 'playwright-report',
      open: 'never',
    },
  ],
],
```

---

# 7. Multiple Reporters

Playwright allows multiple reporters.

Example:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  reporter: [
    ['list'],
    ['html', { open: 'never' }],
  ],
});
```

This gives:

```text
Console output
      +
HTML report
```

---

# 8. Common Built-in Reporters

Playwright provides several built-in reporters.

Common reporters include:

```text
list
line
dot
html
json
junit
blob
github
```

Example:

```typescript
reporter: 'list'
```

---

# 9. List Reporter

The list reporter displays detailed test execution information in the terminal.

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  reporter: 'list',
});
```

Example output:

```text
Running 3 tests using 1 worker

✓ Login test
✓ Search test
✘ Checkout test

3 passed
1 failed
```

---

# 10. Line Reporter

The line reporter provides compact terminal output.

```typescript
reporter: 'line'
```

Useful when running many tests.

---

# 11. Dot Reporter

The dot reporter provides minimal output.

```typescript
reporter: 'dot'
```

Each test is represented by a symbol.

This is useful for CI environments where we want concise logs.

---

# 12. JSON Reporter

The JSON reporter generates machine-readable test results.

Configuration:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  reporter: [
    [
      'json',
      {
        outputFile: 'test-results/results.json',
      },
    ],
  ],
});
```

Output:

```text
test-results/
└── results.json
```

JSON reports are useful when another tool needs to process test results.

---

# 13. JUnit Reporter

JUnit format is commonly used by CI/CD systems.

Example:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  reporter: [
    [
      'junit',
      {
        outputFile: 'test-results/results.xml',
      },
    ],
  ],
});
```

Output:

```text
test-results/
└── results.xml
```

JUnit XML can be consumed by many CI/CD systems.

---

# 14. Multiple Reporters Example

A common configuration is:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  reporter: [
    ['list'],
    ['html', { open: 'never' }],
    [
      'junit',
      {
        outputFile: 'test-results/results.xml',
      },
    ],
  ],
});
```

This produces:

```text
Terminal
   +
HTML Report
   +
JUnit XML
```

---

# 15. Screenshots

Screenshots can be captured when tests fail.

Configure:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  use: {
    screenshot: 'only-on-failure',
  },
});
```

Possible values include:

```text
off
on
only-on-failure
```

Recommended configuration:

```typescript
use: {
  screenshot: 'only-on-failure',
}
```

---

# 16. Screenshot for Every Test

To capture screenshots for every test:

```typescript
use: {
  screenshot: 'on',
}
```

This can increase storage requirements, especially in large test suites.

---

# 17. Video Recording

Playwright can record videos.

Example:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  use: {
    video: 'on',
  },
});
```

Possible values include:

```text
off
on
retain-on-failure
on-first-retry
```

Recommended for many automation frameworks:

```typescript
use: {
  video: 'retain-on-failure',
}
```

This keeps videos primarily for failed tests.

---

# 18. Video on First Retry

For tests that fail, we can record video on the first retry.

```typescript
use: {
  video: 'on-first-retry',
},
```

This is useful when retries are enabled.

Example:

```typescript
retries: 2,

use: {
  video: 'on-first-retry',
},
```

---

# 19. Trace Viewer

Trace Viewer is one of Playwright's most powerful debugging features.

A trace can contain:

* Actions
* Screenshots
* DOM snapshots
* Network activity
* Console messages
* Source information
* Timing information

Configure:

```typescript
use: {
  trace: 'on-first-retry',
},
```

---

# 20. Trace Configuration

Common trace options include:

```text
off
on
retain-on-failure
on-first-retry
```

Recommended:

```typescript
use: {
  trace: 'on-first-retry',
},
```

---

# 21. Open Trace

After a trace is generated, use:

```bash
npx playwright show-trace path/to/trace.zip
```

Example:

```bash
npx playwright show-trace test-results/login-test/trace.zip
```

The Trace Viewer provides a detailed timeline of the test.

---

# 22. Recommended Debugging Configuration

A practical configuration is:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  retries: process.env.CI ? 2 : 0,

  reporter: [
    ['list'],
    ['html', { open: 'never' }],
  ],

  use: {
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    trace: 'on-first-retry',
  },
});
```

This is a good starting point for a professional automation framework.

---

# 23. Complete Reporting Configuration

Example:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './tests',

  retries: process.env.CI ? 2 : 0,

  reporter: [
    ['list'],
    [
      'html',
      {
        outputFolder: 'playwright-report',
        open: 'never',
      },
    ],
    [
      'junit',
      {
        outputFile: 'test-results/results.xml',
      },
    ],
  ],

  use: {
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    trace: 'on-first-retry',
  },
});
```

---

# 24. Test Annotations

Playwright allows tests to have metadata.

Example:

```typescript
import { test, expect } from '@playwright/test';

test(
  'Login Test',
  {
    tag: '@smoke',
  },
  async ({ page }) => {
    await page.goto('https://example.com');

    await expect(page).toHaveTitle(/Example/);
  }
);
```

Tags can help organize and filter tests.

Run tagged tests:

```bash
npx playwright test --grep @smoke
```

---

# 25. Test Steps in Reports

Use `test.step()` to create meaningful steps.

```typescript
import { test, expect } from '@playwright/test';

test('Login Test', async ({ page }) => {

  await test.step('Open Login Page', async () => {
    await page.goto('https://example.com/login');
  });

  await test.step('Enter Username', async () => {
    await page.getByLabel('Username').fill('testuser');
  });

  await test.step('Enter Password', async () => {
    await page.getByLabel('Password').fill('password');
  });

  await test.step('Click Login', async () => {
    await page.getByRole('button', { name: 'Login' }).click();
  });

});
```

These steps appear in the HTML report and make failures easier to understand.

---

# 26. Attachments

Playwright supports attachments to test results.

Example:

```typescript
import { test } from '@playwright/test';

test('Attachment Test', async ({}, testInfo) => {

  await testInfo.attach('Test Data', {
    body: JSON.stringify({
      username: 'testuser',
      environment: 'QA',
    }),
    contentType: 'application/json',
  });

});
```

Attachments can be viewed from the test report.

---

# 27. Attach a File

Example:

```typescript
await testInfo.attach('Screenshot', {
  path: 'screenshots/login.png',
  contentType: 'image/png',
});
```

Another example:

```typescript
await testInfo.attach('Response', {
  path: 'results/response.json',
  contentType: 'application/json',
});
```

---

# 28. Custom Screenshot Attachment

We can manually capture a screenshot and attach it.

```typescript
import { test } from '@playwright/test';

test('Screenshot Attachment', async ({ page }, testInfo) => {

  await page.goto('https://example.com');

  const screenshot = await page.screenshot();

  await testInfo.attach('Homepage Screenshot', {
    body: screenshot,
    contentType: 'image/png',
  });

});
```

---

# 29. Environment Information

We can include environment information in reports through test metadata, attachments, or CI configuration.

Example:

```typescript
const environment = process.env.TEST_ENV || 'QA';

console.log(`Running tests in ${environment}`);
```

Run:

```bash
TEST_ENV=QA npx playwright test
```

On Windows PowerShell:

```powershell
$env:TEST_ENV="QA"
npx playwright test
```

---

# 30. Filtering Tests for Reporting

Run a specific test:

```bash
npx playwright test login.spec.ts
```

Run tests by title:

```bash
npx playwright test -g "Login"
```

Run smoke tests using tags:

```bash
npx playwright test --grep @smoke
```

Run only failed tests from the previous run:

```bash
npx playwright test --last-failed
```

---

# 31. Report Directory

A typical project may contain:

```text
playwright-project/
│
├── tests/
│   ├── login.spec.ts
│   └── checkout.spec.ts
│
├── test-results/
│
├── playwright-report/
│
├── playwright.config.ts
├── package.json
└── package-lock.json
```

The `test-results` directory usually contains test artifacts such as:

* Screenshots
* Videos
* Traces
* Attachments

The `playwright-report` directory contains the HTML report.

---

# 32. Do Not Commit Reports to Git

Generated reports normally should not be committed to GitHub.

Add these directories to `.gitignore`:

```text
playwright-report/
test-results/
blob-report/
```

Example `.gitignore`:

```text
node_modules/
playwright-report/
test-results/
blob-report/
```

---

# 33. CI/CD Reporting

In CI/CD, reports should usually be generated without opening a browser.

Use:

```typescript
reporter: [
  ['html', { open: 'never' }],
  ['junit', { outputFile: 'test-results/results.xml' }],
],
```

CI pipeline:

```text
Checkout Code
      ↓
Install Dependencies
      ↓
Install Playwright Browsers
      ↓
Run Tests
      ↓
Generate Reports
      ↓
Upload Artifacts
      ↓
Review Results
```

---

# 34. GitHub Actions Example

Example workflow:

```yaml
name: Playwright Tests

on:
  push:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install Dependencies
        run: npm ci

      - name: Install Playwright Browsers
        run: npx playwright install --with-deps

      - name: Run Playwright Tests
        run: npx playwright test

      - name: Upload Playwright Report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

The report is uploaded as a CI artifact.

---

# 35. Jenkins Reporting

For Jenkins, generate JUnit results:

```typescript
reporter: [
  ['html', { open: 'never' }],
  [
    'junit',
    {
      outputFile: 'test-results/results.xml',
    },
  ],
],
```

Jenkins can process:

```text
test-results/results.xml
```

The HTML report can also be archived as a build artifact.

---

# 36. Allure Reporting

Allure is a popular third-party reporting solution.

A typical setup uses an Allure Playwright reporter.

Install:

```bash
npm install -D allure-playwright
```

Configure:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  reporter: [
    ['list'],
    ['allure-playwright'],
  ],
});
```

Run tests:

```bash
npx playwright test
```

Generate/open the Allure report using the Allure CLI.

The exact Allure CLI setup can vary depending on the project and operating system.

---

# 37. Playwright HTML vs Allure

| Feature               | Playwright HTML | Allure                  |
| --------------------- | --------------- | ----------------------- |
| Built-in              | Yes             | No                      |
| Setup                 | Very easy       | Additional setup        |
| Screenshots           | Yes             | Yes                     |
| Video                 | Yes             | Yes                     |
| Trace                 | Yes             | Can integrate artifacts |
| Test steps            | Yes             | Yes                     |
| History               | Basic           | Strong history features |
| Dashboard             | Good            | Advanced                |
| CI/CD                 | Excellent       | Excellent               |
| Additional dependency | No              | Yes                     |

For most projects, start with the built-in HTML reporter.

---

# 38. Reporter Selection for a Real Project

A practical enterprise setup can be:

```typescript
reporter: [
  ['list'],
  [
    'html',
    {
      outputFolder: 'playwright-report',
      open: 'never',
    },
  ],
  [
    'junit',
    {
      outputFile: 'test-results/results.xml',
    },
  ],
],
```

This provides:

```text
Developer
   ↓
Terminal Output
   ↓
HTML Report
   ↓
CI/CD
   ↓
JUnit Results
```

---

# 39. Recommended Configuration for QA Automation

For a Selenium/Playwright-style automation framework, this is a strong configuration:

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',

  fullyParallel: true,

  forbidOnly: !!process.env.CI,

  retries: process.env.CI ? 2 : 0,

  workers: process.env.CI ? 1 : undefined,

  reporter: [
    ['list'],
    [
      'html',
      {
        outputFolder: 'playwright-report',
        open: 'never',
      },
    ],
    [
      'junit',
      {
        outputFile: 'test-results/results.xml',
      },
    ],
  ],

  use: {
    baseURL: 'https://example.com',

    screenshot: 'only-on-failure',

    video: 'retain-on-failure',

    trace: 'on-first-retry',

    headless: true,
  },

  projects: [
    {
      name: 'chromium',
      use: {
        ...devices['Desktop Chrome'],
      },
    },
  ],
});
```

---

# 40. Example Test with Reporting Steps

```typescript
import { test, expect } from '@playwright/test';

test.describe('Login Tests', () => {

  test('Valid Login', async ({ page }) => {

    await test.step('Navigate to application', async () => {
      await page.goto('/login');
    });

    await test.step('Enter username', async () => {
      await page.getByLabel('Username').fill('testuser');
    });

    await test.step('Enter password', async () => {
      await page.getByLabel('Password').fill('password');
    });

    await test.step('Click login', async () => {
      await page.getByRole('button', {
        name: 'Login',
      }).click();
    });

    await test.step('Verify dashboard', async () => {
      await expect(
        page.getByRole('heading', {
          name: 'Dashboard',
        })
      ).toBeVisible();
    });

  });

});
```

This produces a report with clear logical steps.

---

# 41. Failed Test Investigation

When a test fails, investigate in this order:

```text
1. Open HTML Report
        ↓
2. Identify failed test
        ↓
3. Read error message
        ↓
4. Review test steps
        ↓
5. Check screenshot
        ↓
6. Check video
        ↓
7. Open trace
        ↓
8. Investigate network/console/DOM
        ↓
9. Fix the problem
        ↓
10. Re-run test
```

---

# 42. Trace + Screenshot + Video

A powerful configuration is:

```typescript
use: {
  screenshot: 'only-on-failure',
  video: 'retain-on-failure',
  trace: 'on-first-retry',
},
```

This gives the framework enough diagnostic information without generating excessive artifacts for every successful test.

---

# 43. Report Commands Cheat Sheet

Run all tests:

```bash
npx playwright test
```

Open HTML report:

```bash
npx playwright show-report
```

Run headed:

```bash
npx playwright test --headed
```

Run a specific browser:

```bash
npx playwright test --project=chromium
```

Run a specific file:

```bash
npx playwright test tests/login.spec.ts
```

Run by test name:

```bash
npx playwright test -g "Login"
```

Run smoke tests:

```bash
npx playwright test --grep @smoke
```

Run only previously failed tests:

```bash
npx playwright test --last-failed
```

Open trace:

```bash
npx playwright show-trace trace.zip
```

---

# 44. Best Practices

## 44.1 Use HTML Reporter

Use the built-in HTML reporter for everyday test execution.

```typescript
reporter: [
  ['html', { open: 'never' }],
],
```

## 44.2 Capture Screenshots on Failure

```typescript
screenshot: 'only-on-failure'
```

## 44.3 Retain Videos for Failures

```typescript
video: 'retain-on-failure'
```

## 44.4 Capture Trace During Retries

```typescript
trace: 'on-first-retry'
```

## 44.5 Use JUnit in CI/CD

```typescript
[
  'junit',
  {
    outputFile: 'test-results/results.xml',
  },
]
```

## 44.6 Use Test Steps

```typescript
await test.step('Login', async () => {
  // test actions
});
```

## 44.7 Avoid Excessive Artifacts

Do not automatically capture screenshots, videos, and traces for every successful test unless there is a specific need.

## 44.8 Keep Reports Out of Source Control

Add generated directories to `.gitignore`.

---

# 45. Interview Questions

## Q1. What is the default Playwright reporter?

The default reporter is the **List reporter** in standard Playwright Test execution, while the HTML reporter is available as a built-in reporter and is commonly configured for detailed visual reporting.

---

## Q2. How do you generate an HTML report?

Run:

```bash
npx playwright test
```

Then:

```bash
npx playwright show-report
```

---

## Q3. How do you configure HTML reporting?

```typescript
reporter: [
  [
    'html',
    {
      outputFolder: 'playwright-report',
      open: 'never',
    },
  ],
],
```

---

## Q4. How do you generate a JUnit report?

```typescript
reporter: [
  [
    'junit',
    {
      outputFile: 'test-results/results.xml',
    },
  ],
],
```

---

## Q5. How do you capture screenshots only when tests fail?

```typescript
use: {
  screenshot: 'only-on-failure',
},
```

---

## Q6. How do you retain videos for failed tests?

```typescript
use: {
  video: 'retain-on-failure',
},
```

---

## Q7. How do you capture a trace during retries?

```typescript
use: {
  trace: 'on-first-retry',
},
```

---

## Q8. How do you open a trace?

```bash
npx playwright show-trace trace.zip
```

---

## Q9. Can Playwright use multiple reporters?

Yes.

Example:

```typescript
reporter: [
  ['list'],
  ['html', { open: 'never' }],
  ['junit', { outputFile: 'results.xml' }],
],
```

---

## Q10. Why use JUnit reporting?

JUnit XML is widely supported by CI/CD systems and allows test results to be displayed and processed by CI tools.

---

## Q11. What is `test.step()`?

`test.step()` creates logical test steps that improve readability and debugging in reports.

Example:

```typescript
await test.step('Login', async () => {
  await page.getByLabel('Username').fill('testuser');
});
```

---

## Q12. What is Trace Viewer?

Trace Viewer is Playwright's debugging tool that allows us to inspect test actions, screenshots, DOM snapshots, network activity, console messages, and timing information.

---

## Q13. Should reports be committed to Git?

Normally, no.

Generated artifacts such as:

```text
playwright-report/
test-results/
```

should generally be added to `.gitignore`.

---

## Q14. What reporting setup would you recommend for CI/CD?

A common setup is:

```typescript
reporter: [
  ['list'],
  ['html', { open: 'never' }],
  [
    'junit',
    {
      outputFile: 'test-results/results.xml',
    },
  ],
],

use: {
  screenshot: 'only-on-failure',
  video: 'retain-on-failure',
  trace: 'on-first-retry',
},
```

---

# 46. Quick Reference

| Requirement           | Configuration                         |
| --------------------- | ------------------------------------- |
| HTML report           | `html`                                |
| Terminal report       | `list`                                |
| Minimal output        | `dot`                                 |
| Compact output        | `line`                                |
| JSON report           | `json`                                |
| CI/CD XML             | `junit`                               |
| Screenshot on failure | `screenshot: 'only-on-failure'`       |
| Video on failure      | `video: 'retain-on-failure'`          |
| Trace on retry        | `trace: 'on-first-retry'`             |
| Open HTML report      | `npx playwright show-report`          |
| Open trace            | `npx playwright show-trace trace.zip` |
| Test steps            | `test.step()`                         |
| Attach artifact       | `testInfo.attach()`                   |

---

# 47. Key Takeaways

```text
Playwright Reporting
        |
        +-- HTML Reporter
        |
        +-- List Reporter
        |
        +-- JSON Reporter
        |
        +-- JUnit Reporter
        |
        +-- Screenshots
        |
        +-- Videos
        |
        +-- Trace Viewer
        |
        +-- Test Steps
        |
        +-- Attachments
        |
        +-- CI/CD Integration
        |
        +-- Allure
```

For a professional Playwright automation framework, a strong default is:

```typescript
reporter: [
  ['list'],
  ['html', { open: 'never' }],
  ['junit', { outputFile: 'test-results/results.xml' }],
],

use: {
  screenshot: 'only-on-failure',
  video: 'retain-on-failure',
  trace: 'on-first-retry',
},
```

This gives developers readable terminal output, QA engineers a detailed HTML report, and CI/CD systems machine-readable JUnit results.
