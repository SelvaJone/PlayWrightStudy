# Playwright Configuration

## Overview

Playwright configuration is managed primarily through the `playwright.config.ts` file.

It allows you to centrally configure:

* Test directory
* Browser projects
* Base URL
* Timeouts
* Retries
* Workers
* Parallel execution
* Reporters
* Screenshots
* Videos
* Traces
* Browser context options
* Authentication state
* Environment-specific settings
* Web server
* Global setup and teardown

A typical Playwright project contains:

```text
playwright-project/
│
├── tests/
│   ├── login.spec.ts
│   └── home.spec.ts
│
├── playwright.config.ts
├── package.json
└── tsconfig.json
```

---

# 1. Basic Playwright Configuration

A simple configuration:

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',

  use: {
    browserName: 'chromium',
    headless: true,
  },
});
```

Run tests:

```bash
npx playwright test
```

---

# 2. `defineConfig()`

Playwright provides `defineConfig()` to define the configuration.

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
});
```

Benefits:

* Type safety
* Better IDE support
* Cleaner configuration
* Easier maintenance

---

# 3. Test Directory

Configure where Playwright should search for tests.

```typescript
export default defineConfig({
  testDir: './tests',
});
```

Example:

```text
project/
├── tests/
│   ├── login.spec.ts
│   └── checkout.spec.ts
│
└── playwright.config.ts
```

You can also use:

```typescript
testDir: './e2e'
```

---

# 4. Test File Matching

Playwright normally recognizes files such as:

```text
login.spec.ts
login.test.ts
```

You can customize matching with:

```typescript
testMatch: '**/*.spec.ts',
```

Example:

```typescript
export default defineConfig({
  testDir: './tests',
  testMatch: '**/*.spec.ts',
});
```

You can exclude files:

```typescript
testIgnore: '**/excluded/**',
```

---

# 5. Base URL

`baseURL` allows tests to use relative URLs.

Configuration:

```typescript
use: {
  baseURL: 'https://example.com',
},
```

Test:

```typescript
import { test } from '@playwright/test';

test('open application', async ({ page }) => {
  await page.goto('/');
});
```

Instead of:

```typescript
await page.goto('https://example.com/');
```

This is especially useful when switching between:

* DEV
* QA
* STAGE
* PROD

---

# 6. Browser Configuration

Playwright supports:

* Chromium
* Firefox
* WebKit

Example:

```typescript
use: {
  browserName: 'chromium',
},
```

Possible values:

```typescript
browserName: 'chromium'
browserName: 'firefox'
browserName: 'webkit'
```

---

# 7. Headless Mode

By default, Playwright runs tests in headless mode.

```typescript
use: {
  headless: true,
},
```

To run with a visible browser:

```typescript
use: {
  headless: false,
},
```

Command-line option:

```bash
npx playwright test --headed
```

---

# 8. Projects

Projects allow the same tests to run under different configurations.

For example:

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

  {
    name: 'webkit',
    use: {
      ...devices['Desktop Safari'],
    },
  },
],
```

Run all projects:

```bash
npx playwright test
```

Run only Chromium:

```bash
npx playwright test --project=chromium
```

---

# 9. Device Configuration

Playwright provides predefined device configurations.

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  projects: [
    {
      name: 'Chrome',
      use: {
        ...devices['Desktop Chrome'],
      },
    },

    {
      name: 'iPhone',
      use: {
        ...devices['iPhone 13'],
      },
    },
  ],
});
```

This configures settings such as:

* Viewport
* User agent
* Device scale factor
* Touch support
* Mobile behavior

---

# 10. Multiple Browsers

A common configuration:

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

This allows cross-browser testing.

---

# 11. Timeout

Configure the maximum time for a test.

```typescript
timeout: 30 * 1000,
```

This means:

```text
30 seconds
```

Example:

```typescript
export default defineConfig({
  timeout: 30 * 1000,
});
```

---

# 12. Expect Timeout

`expect` has its own timeout.

```typescript
expect: {
  timeout: 5000,
},
```

Example:

```typescript
export default defineConfig({
  expect: {
    timeout: 5000,
  },
});
```

This means Playwright assertions can retry for up to 5 seconds.

Example:

```typescript
await expect(page.getByText('Welcome')).toBeVisible();
```

---

# 13. Action Timeout

You can configure the timeout for actions.

```typescript
use: {
  actionTimeout: 10000,
},
```

This applies to operations such as:

```typescript
await page.click();
await page.fill();
await page.check();
```

---

# 14. Navigation Timeout

Configure navigation timeout:

```typescript
use: {
  navigationTimeout: 30000,
},
```

Example:

```typescript
await page.goto('/login');
```

The navigation can wait up to 30 seconds.

---

# 15. Retries

Retries allow failed tests to run again.

```typescript
retries: 2,
```

Example:

```typescript
export default defineConfig({
  retries: 2,
});
```

Behavior:

```text
First attempt → Failed
Retry 1        → Failed
Retry 2        → Passed
```

Retries are commonly enabled in CI environments.

---

# 16. Conditional Retries

You can configure retries differently for local and CI execution.

```typescript
retries: process.env.CI ? 2 : 0,
```

Meaning:

```text
Local → 0 retries
CI    → 2 retries
```

This is a common framework configuration.

---

# 17. Workers

Workers control how many test workers Playwright uses.

```typescript
workers: 4,
```

Example:

```typescript
export default defineConfig({
  workers: 4,
});
```

More workers can increase execution speed but also increase:

* CPU usage
* Memory usage
* Application load
* Test-data conflicts

---

# 18. CI Worker Configuration

A common configuration:

```typescript
workers: process.env.CI ? 2 : undefined,
```

This allows Playwright to use:

```text
CI     → 2 workers
Local  → default worker count
```

---

# 19. Fully Parallel Execution

Enable fully parallel execution:

```typescript
fullyParallel: true,
```

Example:

```typescript
export default defineConfig({
  fullyParallel: true,
});
```

This allows tests to run in parallel across files and tests where appropriate.

---

# 20. Reporter

Playwright supports different reporters.

Example:

```typescript
reporter: 'html',
```

Run:

```bash
npx playwright test
```

Open report:

```bash
npx playwright show-report
```

---

# 21. Multiple Reporters

You can configure multiple reporters:

```typescript
reporter: [
  ['list'],
  ['html', { open: 'never' }],
],
```

This provides:

```text
Terminal output
+
HTML report
```

---

# 22. HTML Reporter

Example:

```typescript
reporter: [
  ['html', { open: 'never' }],
],
```

After execution:

```bash
npx playwright show-report
```

The report can show:

* Passed tests
* Failed tests
* Test duration
* Screenshots
* Videos
* Traces
* Error messages

---

# 23. Screenshots

Configure screenshots:

```typescript
use: {
  screenshot: 'only-on-failure',
},
```

Available options include:

```text
off
on
only-on-failure
```

Recommended CI configuration:

```typescript
screenshot: 'only-on-failure',
```

---

# 24. Video

Configure video recording:

```typescript
use: {
  video: 'retain-on-failure',
},
```

Common options:

```text
off
on
retain-on-failure
on-first-retry
```

Example:

```typescript
video: 'retain-on-failure',
```

---

# 25. Trace

Trace recording is very useful for debugging.

```typescript
use: {
  trace: 'retain-on-failure',
},
```

A trace can provide:

* Screenshots
* DOM snapshots
* Network activity
* Actions
* Timing
* Console information

Recommended:

```typescript
trace: 'retain-on-failure',
```

---

# 26. Browser Context Options

Common browser context settings:

```typescript
use: {
  viewport: {
    width: 1280,
    height: 720,
  },

  locale: 'en-US',

  timezoneId: 'America/New_York',

  colorScheme: 'light',
},
```

---

# 27. Viewport

Set browser size:

```typescript
use: {
  viewport: {
    width: 1920,
    height: 1080,
  },
},
```

Or use a predefined device:

```typescript
use: {
  ...devices['Desktop Chrome'],
},
```

---

# 28. Locale

Set browser locale:

```typescript
use: {
  locale: 'en-US',
},
```

Other examples:

```typescript
locale: 'en-GB'
locale: 'fr-FR'
locale: 'es-MX'
```

Useful for localization testing.

---

# 29. Timezone

Configure timezone:

```typescript
use: {
  timezoneId: 'America/New_York',
},
```

Example:

```typescript
timezoneId: 'America/Los_Angeles'
```

Useful for applications that depend on:

* Date
* Time
* Appointments
* Scheduling
* Time zones

---

# 30. Color Scheme

Configure light or dark mode:

```typescript
use: {
  colorScheme: 'dark',
},
```

Possible values:

```text
light
dark
no-preference
```

---

# 31. Ignore HTTPS Errors

For test environments with certificate problems:

```typescript
use: {
  ignoreHTTPSErrors: true,
},
```

Example:

```typescript
export default defineConfig({
  use: {
    ignoreHTTPSErrors: true,
  },
});
```

Use this carefully and generally only in controlled test environments.

---

# 32. User Agent

You can configure a custom user agent:

```typescript
use: {
  userAgent: 'My-Test-Agent',
},
```

Useful for specialized testing scenarios.

---

# 33. Extra HTTP Headers

Configure additional HTTP headers:

```typescript
use: {
  extraHTTPHeaders: {
    'X-Test-Environment': 'QA',
  },
},
```

Multiple headers:

```typescript
use: {
  extraHTTPHeaders: {
    'Authorization': 'Bearer token',
    'X-Test-Environment': 'QA',
  },
},
```

---

# 34. Authentication State

Playwright can reuse authentication state.

Example:

```typescript
use: {
  storageState: 'playwright/.auth/user.json',
},
```

This can avoid logging in before every test.

Example structure:

```text
playwright/
└── .auth/
    └── user.json
```

Do not commit authentication files containing real credentials or sensitive session data.

---

# 35. Web Server

If your application must be started before testing:

```typescript
webServer: {
  command: 'npm run start',
  url: 'http://127.0.0.1:3000',
  reuseExistingServer: true,
},
```

Playwright can:

1. Start the application
2. Wait for the URL
3. Run tests
4. Stop the server

---

# 36. Environment Variables

Environment variables are useful for different environments.

Example:

```typescript
const baseURL = process.env.BASE_URL || 'https://qa.example.com';

export default defineConfig({
  use: {
    baseURL,
  },
});
```

Run:

```bash
BASE_URL=https://stage.example.com npx playwright test
```

On Windows PowerShell:

```powershell
$env:BASE_URL="https://stage.example.com"
npx playwright test
```

---

# 37. Environment-Based Configuration

A common approach:

```typescript
const environment = process.env.ENV || 'qa';

const urls = {
  dev: 'https://dev.example.com',
  qa: 'https://qa.example.com',
  stage: 'https://stage.example.com',
  prod: 'https://prod.example.com',
};

export default defineConfig({
  use: {
    baseURL: urls[environment as keyof typeof urls],
  },
});
```

Run QA:

```bash
ENV=qa npx playwright test
```

Run Stage:

```bash
ENV=stage npx playwright test
```

---

# 38. Global Setup

Global setup runs before the test suite.

Example:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  globalSetup: './global-setup.ts',
});
```

Example:

```typescript
import { FullConfig } from '@playwright/test';

async function globalSetup(config: FullConfig) {
  console.log('Global setup');
}

export default globalSetup;
```

Useful for:

* Preparing test data
* Authentication
* Environment initialization
* Database setup

---

# 39. Global Teardown

Global teardown runs after the test suite.

```typescript
export default defineConfig({
  globalTeardown: './global-teardown.ts',
});
```

Example:

```typescript
async function globalTeardown() {
  console.log('Global teardown');
}

export default globalTeardown;
```

Useful for:

* Cleaning test data
* Closing external resources
* Cleanup operations

---

# 40. Test Output Directory

Configure test artifacts:

```typescript
outputDir: 'test-results/',
```

Example:

```typescript
export default defineConfig({
  outputDir: 'test-results/',
});
```

Artifacts can include:

* Screenshots
* Videos
* Traces
* Other test attachments

---

# 41. Metadata

You can add metadata:

```typescript
metadata: {
  project: 'Automation',
  environment: 'QA',
},
```

This can be useful for reporting and framework information.

---

# 42. Complete Recommended Configuration

A practical configuration:

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',

  fullyParallel: true,

  forbidOnly: !!process.env.CI,

  retries: process.env.CI ? 2 : 0,

  workers: process.env.CI ? 2 : undefined,

  reporter: [
    ['list'],
    ['html', { open: 'never' }],
  ],

  timeout: 30 * 1000,

  expect: {
    timeout: 5000,
  },

  outputDir: 'test-results/',

  use: {
    baseURL: process.env.BASE_URL || 'https://qa.example.com',

    headless: true,

    screenshot: 'only-on-failure',

    video: 'retain-on-failure',

    trace: 'retain-on-failure',

    actionTimeout: 10000,

    navigationTimeout: 30000,

    viewport: {
      width: 1280,
      height: 720,
    },

    locale: 'en-US',

    timezoneId: 'America/New_York',
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

    {
      name: 'webkit',
      use: {
        ...devices['Desktop Safari'],
      },
    },
  ],
});
```

---

# 43. Running Tests With Different Configurations

Default:

```bash
npx playwright test
```

Specific project:

```bash
npx playwright test --project=chromium
```

Headed:

```bash
npx playwright test --headed
```

Debug mode:

```bash
npx playwright test --debug
```

Specific test:

```bash
npx playwright test tests/login.spec.ts
```

Specific test by title:

```bash
npx playwright test -g "login"
```

---

# 44. Configuration Precedence

Playwright configuration can be overridden at different levels.

Typical hierarchy:

```text
Global configuration
        ↓
Project configuration
        ↓
Test-level configuration
        ↓
Command-line options
```

Example project configuration:

```typescript
projects: [
  {
    name: 'chromium',
    use: {
      browserName: 'chromium',
    },
  },
],
```

---

# 45. Project-Specific Configuration

Different projects can have different settings.

```typescript
projects: [
  {
    name: 'Desktop Chrome',
    use: {
      ...devices['Desktop Chrome'],
      baseURL: 'https://qa.example.com',
    },
  },

  {
    name: 'Mobile Chrome',
    use: {
      ...devices['Pixel 5'],
      baseURL: 'https://qa.example.com',
    },
  },
],
```

This allows the same tests to run against multiple platforms.

---

# 46. Dependencies Between Projects

Projects can depend on another project.

Example:

```typescript
projects: [
  {
    name: 'setup',
    testMatch: /global.setup\.ts/,
  },

  {
    name: 'chromium',
    use: {
      ...devices['Desktop Chrome'],
      storageState: 'playwright/.auth/user.json',
    },
    dependencies: ['setup'],
  },
],
```

This is useful when:

```text
Setup
  ↓
Authentication
  ↓
Application tests
```

---

# 47. Common Enterprise Configuration

A typical enterprise framework may use:

```text
playwright.config.ts
        │
        ├── Environment
        │      ├── DEV
        │      ├── QA
        │      ├── STAGE
        │      └── PROD
        │
        ├── Browser Projects
        │      ├── Chromium
        │      ├── Firefox
        │      └── WebKit
        │
        ├── Reporting
        │
        ├── Retry
        │
        ├── Parallel Execution
        │
        ├── Screenshots
        │
        ├── Videos
        │
        └── Traces
```

---

# 48. Recommended Folder Structure

A scalable Playwright framework:

```text
playwright-project/
│
├── tests/
│   ├── login/
│   │   └── login.spec.ts
│   │
│   ├── checkout/
│   │   └── checkout.spec.ts
│   │
│   └── users/
│       └── users.spec.ts
│
├── pages/
│   ├── LoginPage.ts
│   └── HomePage.ts
│
├── fixtures/
│   └── test-fixtures.ts
│
├── utils/
│   └── test-data.ts
│
├── playwright/
│   └── .auth/
│
├── test-results/
│
├── playwright-report/
│
├── playwright.config.ts
├── package.json
└── tsconfig.json
```

---

# 49. Best Practices

## Use `defineConfig()`

```typescript
export default defineConfig({
});
```

## Keep environment URLs configurable

```typescript
baseURL: process.env.BASE_URL,
```

## Use projects for browser coverage

```typescript
projects: [
  { name: 'chromium' },
  { name: 'firefox' },
  { name: 'webkit' },
],
```

## Enable traces on failure

```typescript
trace: 'retain-on-failure',
```

## Capture screenshots on failure

```typescript
screenshot: 'only-on-failure',
```

## Use retries mainly in CI

```typescript
retries: process.env.CI ? 2 : 0,
```

## Avoid unnecessary fixed waits

Do not rely on:

```typescript
await page.waitForTimeout(5000);
```

Prefer Playwright's auto-waiting and web-first assertions.

## Do not hard-code credentials

Avoid:

```typescript
username: 'real-user'
password: 'real-password'
```

Use:

```text
Environment variables
Secrets management
Authentication state
```

---

# 50. Interview Questions

### Q1. What is `playwright.config.ts`?

It is the central configuration file used to configure Playwright test execution.

---

### Q2. What is `defineConfig()`?

It provides a typed and structured way to define Playwright configuration.

---

### Q3. How do you configure the base URL?

```typescript
use: {
  baseURL: 'https://qa.example.com',
},
```

---

### Q4. How do you configure retries?

```typescript
retries: 2,
```

---

### Q5. How do you configure parallel execution?

```typescript
fullyParallel: true,
workers: 4,
```

---

### Q6. How do you run tests in multiple browsers?

Use Playwright projects:

```typescript
projects: [
  {
    name: 'chromium',
    use: { ...devices['Desktop Chrome'] },
  },
  {
    name: 'firefox',
    use: { ...devices['Desktop Firefox'] },
  },
  {
    name: 'webkit',
    use: { ...devices['Desktop Safari'] },
  },
],
```

---

### Q7. How do you capture screenshots only when tests fail?

```typescript
screenshot: 'only-on-failure',
```

---

### Q8. How do you retain videos for failed tests?

```typescript
video: 'retain-on-failure',
```

---

### Q9. How do you capture traces for failed tests?

```typescript
trace: 'retain-on-failure',
```

---

### Q10. How do you configure different environments?

Use environment variables:

```typescript
baseURL: process.env.BASE_URL,
```

Then:

```bash
BASE_URL=https://stage.example.com npx playwright test
```

---

# 51. Selenium vs Playwright Configuration

| Selenium                        | Playwright                        |
| ------------------------------- | --------------------------------- |
| `WebDriver` configuration       | `playwright.config.ts`            |
| Browser setup in code           | Projects/configuration            |
| TestNG XML                      | Playwright config/projects        |
| Maven properties                | Environment variables/config      |
| TestNG parallel                 | Workers/parallel                  |
| TestNG retry analyzer           | `retries`                         |
| Extent/Allure configuration     | Built-in reporters + integrations |
| Screenshot utility              | Built-in screenshot option        |
| Video requires additional setup | Built-in video                    |
| Complex debugging setup         | Built-in trace viewer             |

---

# 52. Recommended `playwright.config.ts`

For a senior-level automation framework, a good starting point is:

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',

  fullyParallel: true,

  forbidOnly: !!process.env.CI,

  retries: process.env.CI ? 2 : 0,

  workers: process.env.CI ? 2 : undefined,

  reporter: [
    ['list'],
    ['html', { open: 'never' }],
  ],

  timeout: 30_000,

  expect: {
    timeout: 5_000,
  },

  use: {
    baseURL: process.env.BASE_URL || 'https://qa.example.com',

    headless: true,

    screenshot: 'only-on-failure',

    video: 'retain-on-failure',

    trace: 'retain-on-failure',

    actionTimeout: 10_000,

    navigationTimeout: 30_000,
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

    {
      name: 'webkit',
      use: {
        ...devices['Desktop Safari'],
      },
    },
  ],
});
```

---

# Summary

The Playwright configuration file is the **central control point** for the automation framework.

Important settings to remember:

```text
testDir
testMatch
baseURL
browserName
projects
devices
timeout
expect.timeout
actionTimeout
navigationTimeout
retries
workers
fullyParallel
reporter
screenshot
video
trace
viewport
locale
timezoneId
storageState
webServer
globalSetup
globalTeardown
outputDir
```

A strong Playwright framework should keep these settings centralized rather than scattering configuration throughout individual test files.
