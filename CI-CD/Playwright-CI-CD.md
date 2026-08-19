# Playwright CI/CD

## 1. Introduction

CI/CD stands for:

* **CI** – Continuous Integration
* **CD** – Continuous Delivery / Continuous Deployment

Playwright works very well with CI/CD tools because tests can run automatically whenever code is pushed, a pull request is created, or a deployment is triggered.

A typical Playwright CI/CD workflow looks like:

```text
Developer
   |
   v
Git Repository
   |
   v
CI/CD Pipeline
   |
   +---- Install dependencies
   |
   +---- Install Playwright browsers
   |
   +---- Run Playwright tests
   |
   +---- Generate test report
   |
   +---- Upload artifacts
   |
   v
Pass / Fail
```

---

# 2. Why Use Playwright in CI/CD?

Running Playwright tests in CI/CD provides:

* Automated test execution
* Early detection of defects
* Consistent test environments
* Headless browser execution
* Parallel test execution
* Automatic HTML reports
* Screenshots and videos for failures
* Integration with GitHub
* Integration with Jenkins
* Integration with Docker
* Test execution on every pull request

---

# 3. Basic Playwright CI Requirements

A CI machine generally needs:

```text
Node.js
npm
Playwright
Playwright browsers
Git
CI/CD tool
```

Example:

```bash
node --version
npm --version
npx playwright --version
```

---

# 4. Typical Playwright Project Structure

A Playwright project may look like:

```text
playwright-project/
│
├── tests/
│   ├── login.spec.ts
│   ├── home.spec.ts
│   └── checkout.spec.ts
│
├── pages/
│   ├── LoginPage.ts
│   └── HomePage.ts
│
├── test-data/
│   └── users.json
│
├── playwright.config.ts
├── package.json
├── package-lock.json
├── .gitignore
│
└── .github/
    └── workflows/
        └── playwright.yml
```

---

# 5. package.json

A typical `package.json`:

```json
{
  "name": "playwright-automation",
  "version": "1.0.0",
  "scripts": {
    "test": "playwright test",
    "test:headed": "playwright test --headed",
    "test:debug": "playwright test --debug",
    "test:chromium": "playwright test --project=chromium",
    "report": "playwright show-report"
  },
  "devDependencies": {
    "@playwright/test": "^1.55.0"
  }
}
```

The exact Playwright version should match the version used by your project.

---

# 6. Installing Dependencies in CI

CI should install dependencies from `package-lock.json`.

Recommended:

```bash
npm ci
```

Instead of:

```bash
npm install
```

### Why use `npm ci`?

`npm ci`:

* Installs exactly what is specified in `package-lock.json`
* Provides more consistent builds
* Is generally faster for CI
* Removes the existing `node_modules`
* Helps avoid unexpected dependency changes

---

# 7. Installing Playwright Browsers

After installing dependencies:

```bash
npx playwright install
```

For Linux CI environments, use:

```bash
npx playwright install --with-deps
```

This installs Playwright browsers and required operating-system dependencies.

---

# 8. Running Playwright Tests in CI

Basic command:

```bash
npx playwright test
```

Headless execution:

```bash
npx playwright test
```

Playwright runs headless by default.

Specific browser:

```bash
npx playwright test --project=chromium
```

Specific test:

```bash
npx playwright test tests/login.spec.ts
```

Specific test by title:

```bash
npx playwright test -g "valid login"
```

---

# 9. Headless vs Headed in CI

CI environments normally run browsers in headless mode.

Example:

```bash
npx playwright test
```

Avoid:

```bash
npx playwright test --headed
```

unless the CI environment specifically supports a graphical display.

For normal CI execution:

```text
CI Server
   |
   v
Playwright
   |
   v
Headless Browser
   |
   v
Tests
```

---

# 10. Playwright Configuration for CI

Example:

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',

  timeout: 30 * 1000,

  expect: {
    timeout: 5000
  },

  fullyParallel: true,

  forbidOnly: !!process.env.CI,

  retries: process.env.CI ? 2 : 0,

  workers: process.env.CI ? 2 : undefined,

  reporter: [
    ['html', { open: 'never' }],
    ['list']
  ],

  use: {
    baseURL: process.env.BASE_URL || 'https://example.com',

    trace: 'on-first-retry',

    screenshot: 'only-on-failure',

    video: 'retain-on-failure'
  },

  projects: [
    {
      name: 'chromium',
      use: {
        ...devices['Desktop Chrome']
      }
    }
  ]
});
```

---

# 11. Understanding CI Configuration

## `forbidOnly`

```typescript
forbidOnly: !!process.env.CI
```

Prevents accidentally committing:

```typescript
test.only(...)
```

When CI is running.

---

## `retries`

```typescript
retries: process.env.CI ? 2 : 0
```

Local:

```text
0 retries
```

CI:

```text
2 retries
```

---

## `workers`

```typescript
workers: process.env.CI ? 2 : undefined
```

This limits parallel workers in CI.

---

## `trace`

```typescript
trace: 'on-first-retry'
```

Playwright collects a trace when a failed test is retried.

---

## `screenshot`

```typescript
screenshot: 'only-on-failure'
```

Screenshot is captured when a test fails.

---

## `video`

```typescript
video: 'retain-on-failure'
```

Video is retained for failed tests.

---

# 12. Environment Variables

Do not hard-code environment-specific values.

Instead of:

```typescript
baseURL: 'https://stage.example.com'
```

Use:

```typescript
baseURL: process.env.BASE_URL
```

Example:

```bash
BASE_URL=https://stage.example.com
```

Or:

```bash
BASE_URL=https://prod.example.com
```

---

# 13. Using Environment Variables in Playwright

Example:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  use: {
    baseURL: process.env.BASE_URL
  }
});
```

Test:

```typescript
import { test, expect } from '@playwright/test';

test('open application', async ({ page }) => {
  await page.goto('/');
  await expect(page).toHaveTitle(/Example/);
});
```

---

# 14. Environment Selection

A common approach:

```bash
BASE_URL=https://stage.example.com npx playwright test
```

Another approach:

```bash
BASE_URL=https://qa.example.com npx playwright test
```

Another:

```bash
BASE_URL=https://prod.example.com npx playwright test
```

---

# 15. GitHub Actions

GitHub Actions is one of the most common ways to run Playwright tests in CI.

Project structure:

```text
.github/
└── workflows/
    └── playwright.yml
```

---

# 16. Basic GitHub Actions Workflow

Create:

```text
.github/workflows/playwright.yml
```

Example:

```yaml
name: Playwright Tests

on:
  push:
    branches:
      - main

  pull_request:
    branches:
      - main

jobs:
  test:
    timeout-minutes: 60

    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright browsers
        run: npx playwright install --with-deps

      - name: Run Playwright tests
        run: npx playwright test

      - name: Upload Playwright report
        if: ${{ !cancelled() }}
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

---

# 17. GitHub Actions Workflow Explained

## Checkout

```yaml
- name: Checkout repository
  uses: actions/checkout@v4
```

Downloads the repository code to the GitHub runner.

---

## Setup Node.js

```yaml
- name: Setup Node.js
  uses: actions/setup-node@v4
  with:
    node-version: 20
    cache: npm
```

Installs Node.js and enables npm caching.

---

## Install dependencies

```yaml
- name: Install dependencies
  run: npm ci
```

Installs dependencies from:

```text
package-lock.json
```

---

## Install Playwright

```yaml
- name: Install Playwright browsers
  run: npx playwright install --with-deps
```

Installs browsers and Linux dependencies.

---

## Execute tests

```yaml
- name: Run Playwright tests
  run: npx playwright test
```

Runs the test suite.

---

## Upload report

```yaml
- name: Upload Playwright report
  if: ${{ !cancelled() }}
  uses: actions/upload-artifact@v4
  with:
    name: playwright-report
    path: playwright-report/
```

Makes the Playwright report available as a GitHub Actions artifact.

---

# 18. GitHub Actions with Environment Variable

Example:

```yaml
- name: Run Playwright tests
  run: npx playwright test
  env:
    BASE_URL: https://stage.example.com
```

Then:

```typescript
use: {
  baseURL: process.env.BASE_URL
}
```

---

# 19. GitHub Secrets

Sensitive information should not be committed to GitHub.

Do not do:

```typescript
username: 'admin'
password: 'MyPassword123'
```

Instead use GitHub Secrets.

Example:

```text
USERNAME
PASSWORD
API_TOKEN
```

Then reference them:

```yaml
env:
  USERNAME: ${{ secrets.USERNAME }}
  PASSWORD: ${{ secrets.PASSWORD }}
```

---

# 20. Using Secrets in Playwright

Example:

```typescript
const username = process.env.USERNAME;
const password = process.env.PASSWORD;
```

Then:

```typescript
await page.getByLabel('Username').fill(username!);
await page.getByLabel('Password').fill(password!);
```

---

# 21. Do Not Commit Secrets

Never commit:

```text
.env
passwords
API keys
tokens
private certificates
credentials
```

Use:

```text
GitHub Secrets
Environment Variables
Secret Managers
```

---

# 22. .gitignore

Example:

```text
node_modules/
playwright-report/
test-results/
blob-report/
.env
.env.*
!.env.example
```

---

# 23. Playwright Reports

Playwright supports several reporters.

Common reporters:

```text
list
line
dot
html
json
junit
github
```

Example:

```typescript
reporter: [
  ['list'],
  ['html', { open: 'never' }]
]
```

---

# 24. HTML Report

Run:

```bash
npx playwright test
```

Then:

```bash
npx playwright show-report
```

The report can contain:

* Test status
* Test duration
* Steps
* Screenshots
* Videos
* Traces
* Errors

---

# 25. JUnit Reporter

JUnit reports are useful for CI systems.

Example:

```typescript
reporter: [
  ['html', { open: 'never' }],
  ['junit', { outputFile: 'test-results/results.xml' }]
]
```

Run:

```bash
npx playwright test
```

JUnit result:

```text
test-results/results.xml
```

---

# 26. JSON Reporter

Example:

```typescript
reporter: [
  ['json', { outputFile: 'test-results/results.json' }]
]
```

This can be consumed by other tools.

---

# 27. GitHub Reporter

Playwright provides a GitHub Actions reporter.

Example:

```typescript
reporter: process.env.CI
  ? [['github'], ['html', { open: 'never' }]]
  : [['list'], ['html', { open: 'never' }]]
```

This can provide GitHub-friendly annotations.

---

# 28. Uploading Test Results

Example:

```yaml
- name: Upload test results
  if: ${{ !cancelled() }}
  uses: actions/upload-artifact@v4
  with:
    name: test-results
    path: test-results/
```

---

# 29. Uploading Screenshots and Videos

If configured:

```typescript
use: {
  screenshot: 'only-on-failure',
  video: 'retain-on-failure'
}
```

Playwright stores failure artifacts under:

```text
test-results/
```

Upload them:

```yaml
- name: Upload test artifacts
  if: ${{ !cancelled() }}
  uses: actions/upload-artifact@v4
  with:
    name: playwright-test-results
    path: test-results/
```

---

# 30. Trace in CI

Configuration:

```typescript
trace: 'on-first-retry'
```

When a test fails and is retried, Playwright captures a trace.

Open a trace:

```bash
npx playwright show-trace path/to/trace.zip
```

Trace can help investigate:

* Actions
* Network requests
* Console logs
* DOM snapshots
* Screenshots
* Timing

---

# 31. CI Retry Strategy

Example:

```typescript
retries: process.env.CI ? 2 : 0
```

This means:

```text
Local:
Test fails -> stop

CI:
Test fails -> retry
Test fails -> retry again
Test fails -> final failure
```

Retries should not be used to hide real application defects.

---

# 32. Parallel Execution in CI

Playwright supports parallel execution.

Example:

```typescript
fullyParallel: true
```

Workers:

```typescript
workers: process.env.CI ? 2 : undefined
```

Example:

```text
Worker 1 -> Test A
Worker 2 -> Test B
Worker 3 -> Test C
Worker 4 -> Test D
```

Parallel execution can significantly reduce execution time.

---

# 33. Sharding

For large test suites, Playwright supports sharding.

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

This divides the test suite across multiple CI jobs.

---

# 34. GitHub Actions Sharding

Example:

```yaml
name: Playwright Tests

on:
  push:
    branches:
      - main

jobs:
  test:
    strategy:
      fail-fast: false
      matrix:
        shard: [1, 2, 3, 4]

    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm

      - run: npm ci

      - run: npx playwright install --with-deps

      - name: Run Playwright tests
        run: npx playwright test --shard=${{ matrix.shard }}/4
```

---

# 35. Jenkins CI/CD

Playwright can also run through Jenkins.

Typical Jenkins pipeline:

```text
Jenkins
   |
   +-- Checkout code
   |
   +-- Install Node
   |
   +-- npm ci
   |
   +-- Install Playwright
   |
   +-- Run tests
   |
   +-- Publish reports
   |
   v
Build Result
```

---

# 36. Jenkinsfile Example

Example:

```groovy
pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Install Playwright') {
            steps {
                sh 'npx playwright install --with-deps'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npx playwright test'
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'playwright-report/**', allowEmptyArchive: true
            archiveArtifacts artifacts: 'test-results/**', allowEmptyArchive: true
        }
    }
}
```

---

# 37. Jenkins on Windows

If Jenkins runs on Windows, use:

```groovy
stage('Install Dependencies') {
    steps {
        bat 'npm ci'
    }
}

stage('Install Playwright') {
    steps {
        bat 'npx playwright install'
    }
}

stage('Run Tests') {
    steps {
        bat 'npx playwright test'
    }
}
```

---

# 38. Docker and Playwright

Playwright can also run inside Docker.

Example:

```dockerfile
FROM mcr.microsoft.com/playwright:v1.55.0-noble

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

CMD ["npx", "playwright", "test"]
```

The Playwright Docker image provides browsers and their required dependencies.

---

# 39. Docker Test Execution

Build:

```bash
docker build -t playwright-tests .
```

Run:

```bash
docker run --rm playwright-tests
```

---

# 40. CI Pipeline with Docker

Typical architecture:

```text
GitHub
   |
   v
CI Pipeline
   |
   v
Docker Image
   |
   v
Playwright
   |
   +---- Chromium
   +---- Firefox
   +---- WebKit
   |
   v
Test Results
```

---

# 41. Multi-Browser CI Testing

Configuration:

```typescript
projects: [
  {
    name: 'chromium',
    use: {
      ...devices['Desktop Chrome']
    }
  },

  {
    name: 'firefox',
    use: {
      ...devices['Desktop Firefox']
    }
  },

  {
    name: 'webkit',
    use: {
      ...devices['Desktop Safari']
    }
  }
]
```

Run:

```bash
npx playwright test
```

This executes tests across configured browsers.

---

# 42. Run Only Chromium in CI

```bash
npx playwright test --project=chromium
```

Firefox:

```bash
npx playwright test --project=firefox
```

WebKit:

```bash
npx playwright test --project=webkit
```

---

# 43. CI-Specific Configuration

A useful pattern:

```typescript
const isCI = !!process.env.CI;

export default defineConfig({
  fullyParallel: true,

  forbidOnly: isCI,

  retries: isCI ? 2 : 0,

  workers: isCI ? 2 : undefined,

  reporter: isCI
    ? [['github'], ['html', { open: 'never' }]]
    : [['list'], ['html', { open: 'never' }]]
});
```

---

# 44. Example Complete Configuration

```typescript
import { defineConfig, devices } from '@playwright/test';

const isCI = !!process.env.CI;

export default defineConfig({
  testDir: './tests',

  fullyParallel: true,

  forbidOnly: isCI,

  retries: isCI ? 2 : 0,

  workers: isCI ? 2 : undefined,

  timeout: 30 * 1000,

  expect: {
    timeout: 5000
  },

  reporter: isCI
    ? [
        ['github'],
        ['html', { open: 'never' }],
        ['junit', { outputFile: 'test-results/results.xml' }]
      ]
    : [
        ['list'],
        ['html', { open: 'never' }]
      ],

  use: {
    baseURL: process.env.BASE_URL || 'https://example.com',

    trace: 'on-first-retry',

    screenshot: 'only-on-failure',

    video: 'retain-on-failure'
  },

  projects: [
    {
      name: 'chromium',
      use: {
        ...devices['Desktop Chrome']
      }
    }
  ]
});
```

---

# 45. Complete GitHub Actions Example

```yaml
name: Playwright Automation

on:
  push:
    branches:
      - main

  pull_request:
    branches:
      - main

  workflow_dispatch:

jobs:
  playwright-tests:

    timeout-minutes: 60

    runs-on: ubuntu-latest

    steps:

      - name: Checkout source code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm

      - name: Install npm dependencies
        run: npm ci

      - name: Install Playwright browsers
        run: npx playwright install --with-deps

      - name: Run Playwright tests
        run: npx playwright test
        env:
          BASE_URL: ${{ secrets.BASE_URL }}
          USERNAME: ${{ secrets.USERNAME }}
          PASSWORD: ${{ secrets.PASSWORD }}

      - name: Upload Playwright HTML report
        if: ${{ !cancelled() }}
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30

      - name: Upload test results
        if: ${{ !cancelled() }}
        uses: actions/upload-artifact@v4
        with:
          name: test-results
          path: test-results/
          retention-days: 30
```

---

# 46. Pull Request Workflow

A common professional workflow is:

```text
Developer creates branch
        |
        v
Write code
        |
        v
Push branch
        |
        v
Create Pull Request
        |
        v
GitHub Actions starts
        |
        +---- npm ci
        |
        +---- Install Playwright
        |
        +---- Run tests
        |
        +---- Generate report
        |
        v
Pass / Fail
        |
        v
Merge Pull Request
```

---

# 47. Run Tests on Every Pull Request

GitHub Actions:

```yaml
on:
  pull_request:
    branches:
      - main
```

This prevents broken code from being merged when tests fail.

---

# 48. Run Tests on Every Push

```yaml
on:
  push:
    branches:
      - main
```

Tests run after code is pushed to `main`.

---

# 49. Manual Execution

Add:

```yaml
workflow_dispatch:
```

Example:

```yaml
on:
  push:
    branches:
      - main

  pull_request:
    branches:
      - main

  workflow_dispatch:
```

This allows manually triggering the workflow from GitHub Actions.

---

# 50. Scheduled Execution

Playwright tests can run automatically on a schedule.

Example:

```yaml
on:
  schedule:
    - cron: '0 6 * * 1-5'
```

This runs approximately at:

```text
06:00 UTC
Monday-Friday
```

Cron schedules use UTC in GitHub Actions.

---

# 51. Smoke Tests in CI

Large regression suites can take a long time.

Use tags or project configuration for smoke tests.

Example:

```typescript
test('login smoke test', async ({ page }) => {
  await page.goto('/');
});
```

Run a specific group using grep:

```bash
npx playwright test --grep "@smoke"
```

Example:

```typescript
test('@smoke login test', async ({ page }) => {
  await page.goto('/');
});
```

---

# 52. Regression Tests

Example:

```typescript
test('@regression checkout test', async ({ page }) => {
  // test implementation
});
```

Run:

```bash
npx playwright test --grep "@regression"
```

---

# 53. Smoke vs Regression CI Strategy

A practical approach:

```text
Pull Request
     |
     v
Smoke Tests
     |
     v
Merge
     |
     v
Regression Suite
     |
     v
Nightly Full Regression
```

This keeps pull-request feedback fast.

---

# 54. CI Failure Investigation

When a test fails:

### Step 1

Check the test name.

### Step 2

Check the error message.

### Step 3

Check screenshot.

### Step 4

Check video.

### Step 5

Check trace.

### Step 6

Check console/network logs.

### Step 7

Reproduce locally.

---

# 55. Useful CI Commands

Run all tests:

```bash
npx playwright test
```

Run one file:

```bash
npx playwright test tests/login.spec.ts
```

Run Chromium:

```bash
npx playwright test --project=chromium
```

Run with grep:

```bash
npx playwright test --grep "@smoke"
```

Run with one worker:

```bash
npx playwright test --workers=1
```

Run headed locally:

```bash
npx playwright test --headed
```

Debug locally:

```bash
npx playwright test --debug
```

Open report:

```bash
npx playwright show-report
```

---

# 56. CI vs Local Execution

| Feature    | Local             | CI                 |
| ---------- | ----------------- | ------------------ |
| Browser    | Headed/Headless   | Usually Headless   |
| Retries    | Usually 0         | Usually 1–2        |
| Workers    | More              | Controlled         |
| Reports    | HTML              | HTML + CI reporter |
| Screenshot | Optional          | Failure            |
| Video      | Optional          | Failure            |
| Trace      | Optional          | Retry              |
| Secrets    | Local environment | CI secrets         |
| Execution  | Developer         | Automated          |

---

# 57. Common CI Problems

## Problem 1: Browser not installed

Error:

```text
Executable doesn't exist
```

Solution:

```bash
npx playwright install
```

Linux CI:

```bash
npx playwright install --with-deps
```

---

## Problem 2: Dependency mismatch

Use:

```bash
npm ci
```

instead of repeatedly changing dependencies with:

```bash
npm install
```

Commit:

```text
package-lock.json
```

---

## Problem 3: Test passes locally but fails in CI

Possible causes:

```text
Timing
Environment
Browser version
Operating system
Network
Missing environment variables
Authentication
Parallel execution
Test data
```

---

# 58. Avoid Hard Waits

Avoid:

```typescript
await page.waitForTimeout(5000);
```

Prefer Playwright's automatic waiting:

```typescript
await page.getByRole('button', {
  name: 'Login'
}).click();
```

Or:

```typescript
await expect(page.getByText('Dashboard')).toBeVisible();
```

This makes tests more reliable in CI.

---

# 59. Avoid Test Dependencies

Bad design:

```text
Test 1 must pass
     |
     v
Test 2 runs
     |
     v
Test 3 runs
```

Better:

```text
Test 1
Test 2
Test 3
Test 4
```

Each test should be independent whenever possible.

---

# 60. Test Data in CI

Do not depend on manually created local data.

Use:

```text
API setup
Database setup
Fixtures
Test data files
Environment-specific data
```

Example:

```text
test-data/users.json
```

---

# 61. Authentication in CI

For authenticated applications, Playwright can reuse authentication state.

Example:

```typescript
use: {
  storageState: 'playwright/.auth/user.json'
}
```

Do not commit sensitive authentication state if it contains real credentials or tokens.

Add:

```text
playwright/.auth/
```

to `.gitignore` when appropriate.

---

# 62. CI/CD Best Practices

### 1. Use `npm ci`

```bash
npm ci
```

### 2. Pin Node.js version

Example:

```yaml
node-version: 20
```

### 3. Install Playwright browsers

```bash
npx playwright install --with-deps
```

### 4. Use environment variables

```typescript
process.env.BASE_URL
```

### 5. Store secrets securely

Use CI secrets.

### 6. Enable retries carefully

```typescript
retries: process.env.CI ? 2 : 0
```

### 7. Capture failure artifacts

```text
Screenshots
Videos
Traces
Reports
```

### 8. Keep tests independent

Avoid test-to-test dependencies.

### 9. Use parallel execution

```typescript
fullyParallel: true
```

### 10. Use smoke tests for PR validation

Keep PR pipelines fast.

---

# 63. Recommended Playwright CI Architecture

```text
                    GitHub
                       |
             Push / Pull Request
                       |
                       v
              GitHub Actions
                       |
             +---------+---------+
             |                   |
             v                   v
        Install Node         Checkout Code
             |                   |
             +---------+---------+
                       |
                       v
                    npm ci
                       |
                       v
            Install Playwright
                       |
                       v
                Run Test Suite
                       |
          +------------+------------+
          |            |            |
          v            v            v
      Chromium      Firefox       WebKit
          |            |            |
          +------------+------------+
                       |
                       v
                 Test Results
                       |
          +------------+------------+
          |            |            |
          v            v            v
       HTML        Screenshot     Trace
       Report       /Video        Files
          |            |            |
          +------------+------------+
                       |
                       v
                GitHub Artifact
```

---

# 64. Example Enterprise CI/CD Flow

```text
Developer
   |
   v
Git Feature Branch
   |
   v
Pull Request
   |
   v
CI Pipeline
   |
   +---- Code Checkout
   |
   +---- npm ci
   |
   +---- Playwright Install
   |
   +---- Lint
   |
   +---- Unit Tests
   |
   +---- API Tests
   |
   +---- Playwright Smoke Tests
   |
   v
Pull Request Approval
   |
   v
Merge to Main
   |
   v
Full Playwright Regression
   |
   v
Deploy to QA
   |
   v
Regression
   |
   v
Deploy to Production
   |
   v
Production Smoke Tests
```

---

# 65. Interview Questions

## Q1. How do you run Playwright tests in CI?

Answer:

```bash
npm ci
npx playwright install --with-deps
npx playwright test
```

---

## Q2. Why use `npm ci` in CI?

Because it installs dependencies based exactly on the lock file and provides reproducible CI builds.

---

## Q3. How do you detect CI execution?

Use:

```typescript
process.env.CI
```

Example:

```typescript
const isCI = !!process.env.CI;
```

---

## Q4. How do you configure retries only in CI?

```typescript
retries: process.env.CI ? 2 : 0
```

---

## Q5. How do you upload Playwright reports in GitHub Actions?

Use:

```yaml
uses: actions/upload-artifact@v4
```

---

## Q6. How do you pass environment variables?

Example:

```yaml
env:
  BASE_URL: ${{ secrets.BASE_URL }}
```

Then:

```typescript
process.env.BASE_URL
```

---

## Q7. How do you handle secrets?

Use:

```text
GitHub Secrets
Jenkins Credentials
Cloud Secret Managers
```

Never commit passwords or tokens to Git.

---

## Q8. How do you capture screenshots on failure?

```typescript
use: {
  screenshot: 'only-on-failure'
}
```

---

## Q9. How do you capture traces?

```typescript
use: {
  trace: 'on-first-retry'
}
```

---

## Q10. How do you reduce execution time?

Use:

```text
Parallel execution
Workers
Sharding
Smoke suites
Independent tests
CI caching
```

---

# 66. Recommended Folder Structure for CI/CD

```text
playwright-project/
│
├── tests/
│   ├── smoke/
│   └── regression/
│
├── pages/
│
├── fixtures/
│
├── test-data/
│
├── playwright.config.ts
│
├── package.json
│
├── package-lock.json
│
├── .gitignore
│
├── Dockerfile
│
└── .github/
    └── workflows/
        └── playwright.yml
```

---

# 67. Important Commands Cheat Sheet

```bash
# Install dependencies
npm ci

# Install browsers
npx playwright install

# Install browsers + Linux dependencies
npx playwright install --with-deps

# Run all tests
npx playwright test

# Run Chromium
npx playwright test --project=chromium

# Run a specific file
npx playwright test tests/login.spec.ts

# Run tests matching text
npx playwright test -g "login"

# Run smoke tests
npx playwright test --grep "@smoke"

# Run with one worker
npx playwright test --workers=1

# Debug
npx playwright test --debug

# Headed
npx playwright test --headed

# Show report
npx playwright show-report

# Show trace
npx playwright show-trace trace.zip
```

---

# 68. Final Summary

Playwright is well suited for modern CI/CD pipelines.

The core CI process is:

```text
Checkout Code
     |
     v
npm ci
     |
     v
Install Playwright
     |
     v
Run Tests
     |
     v
Generate Reports
     |
     v
Upload Artifacts
     |
     v
Pass / Fail
```

For a professional Playwright automation framework, focus on:

```text
Playwright
   +
TypeScript
   +
Page Object Model
   +
Fixtures
   +
Test Data
   +
API Testing
   +
Reporting
   +
Parallel Execution
   +
GitHub Actions / Jenkins
   +
Docker
   +
CI/CD
```

A strong Playwright CI/CD implementation should provide:

* Reliable automated execution
* Headless browser testing
* Environment-specific configuration
* Secure credentials
* Parallel execution
* Retries where appropriate
* Screenshots on failure
* Videos on failure
* Trace collection
* HTML/JUnit reports
* GitHub/Jenkins integration
* Artifact storage
* Smoke and regression pipelines
* Scheduled regression execution

This gives you a complete foundation for running a **Playwright + TypeScript automation framework in CI/CD**.
