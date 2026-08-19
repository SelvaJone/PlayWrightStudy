# Playwright Docker

## 1. Introduction

Docker allows us to run Playwright tests inside a consistent, isolated environment.

Using Docker with Playwright helps ensure that:

* Tests run with the same browser versions.
* Node.js and dependencies are consistent.
* Local and CI environments behave similarly.
* Tests can run without installing Playwright directly on the host machine.
* CI/CD pipelines become easier to standardize.

Typical architecture:

```text
Developer / CI Server
        |
        v
   Docker Container
        |
        +-- Node.js
        +-- Playwright
        +-- Chromium
        +-- Firefox
        +-- WebKit
        |
        v
   Playwright Tests
```

---

# 2. Why Use Docker with Playwright?

Without Docker:

```text
Developer Machine
    |
    +-- Node.js version
    +-- npm packages
    +-- Browser versions
    +-- OS dependencies
    |
    +-- Playwright Tests
```

Different machines may have different configurations.

With Docker:

```text
Docker Image
    |
    +-- Node.js
    +-- Playwright
    +-- Browsers
    +-- Required OS dependencies
    |
    v
Same Test Environment
```

### Benefits

1. Consistent test environment
2. Easier CI/CD integration
3. Browser dependencies are managed inside the container
4. Easy environment cleanup
5. Reproducible test execution
6. Easy parallel execution
7. Useful for GitHub Actions, Jenkins, GitLab CI and Azure DevOps

---

# 3. Prerequisites

Install:

* Docker
* Node.js
* npm
* Playwright project

Verify Docker:

```bash
docker --version
```

Example:

```text
Docker version 27.x
```

Verify Node.js:

```bash
node --version
```

Verify npm:

```bash
npm --version
```

Verify Playwright:

```bash
npx playwright --version
```

---

# 4. Sample Playwright Project

Example project:

```text
playwright-docker/
│
├── tests/
│   └── login.spec.ts
│
├── playwright.config.ts
├── package.json
├── package-lock.json
├── Dockerfile
├── .dockerignore
└── docker-compose.yml
```

---

# 5. package.json

Example:

```json
{
  "name": "playwright-docker",
  "version": "1.0.0",
  "scripts": {
    "test": "playwright test",
    "test:headed": "playwright test --headed",
    "report": "playwright show-report"
  },
  "devDependencies": {
    "@playwright/test": "^1.55.0"
  }
}
```

Install dependencies:

```bash
npm install
```

---

# 6. Example Test

Create:

```text
tests/login.spec.ts
```

Example:

```typescript
import { test, expect } from '@playwright/test';

test('Verify Playwright Docker execution', async ({ page }) => {
  await page.goto('https://example.com');

  await expect(page).toHaveTitle(/Example/);
});
```

---

# 7. Playwright Configuration

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
    ['html', { outputFolder: 'playwright-report' }],
    ['list']
  ],

  use: {
    baseURL: process.env.BASE_URL || 'https://example.com',

    headless: true,

    trace: 'retain-on-failure',

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

# 8. Official Playwright Docker Image

Playwright provides Docker images containing the required browser dependencies.

A typical base image looks like:

```dockerfile
FROM mcr.microsoft.com/playwright:<version>-noble
```

Example:

```dockerfile
FROM mcr.microsoft.com/playwright:1.55.0-noble
```

The exact Playwright image version should normally match the Playwright version used by the project.

---

# 9. Basic Dockerfile

Create:

```text
Dockerfile
```

Example:

```dockerfile
FROM mcr.microsoft.com/playwright:1.55.0-noble

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

CMD ["npx", "playwright", "test"]
```

---

# 10. Understanding the Dockerfile

### FROM

```dockerfile
FROM mcr.microsoft.com/playwright:1.55.0-noble
```

Specifies the base Docker image.

The Playwright image includes:

* Node.js
* Playwright dependencies
* Supported browsers
* Linux dependencies required by browsers

---

### WORKDIR

```dockerfile
WORKDIR /app
```

Sets the working directory inside the container.

---

### COPY package files

```dockerfile
COPY package*.json ./
```

Copies:

```text
package.json
package-lock.json
```

into the container.

---

### npm ci

```dockerfile
RUN npm ci
```

Installs dependencies using the lock file.

`npm ci` is preferred in CI/CD environments because it provides reproducible installations.

---

### COPY project

```dockerfile
COPY . .
```

Copies the Playwright project into the container.

---

### CMD

```dockerfile
CMD ["npx", "playwright", "test"]
```

Runs the Playwright tests when the container starts.

---

# 11. Build Docker Image

From the project directory:

```bash
docker build -t playwright-tests .
```

Check images:

```bash
docker images
```

Example:

```text
REPOSITORY          TAG       IMAGE ID
playwright-tests    latest    abc123
```

---

# 12. Run Playwright Tests

Run:

```bash
docker run --rm playwright-tests
```

Explanation:

```text
docker run
```

Starts a container.

```text
--rm
```

Automatically removes the container after execution.

```text
playwright-tests
```

Specifies the Docker image.

---

# 13. Run Specific Test

You can override the Docker command:

```bash
docker run --rm playwright-tests npx playwright test tests/login.spec.ts
```

---

# 14. Run Specific Browser

Chromium:

```bash
docker run --rm playwright-tests npx playwright test --project=chromium
```

Firefox:

```bash
docker run --rm playwright-tests npx playwright test --project=firefox
```

WebKit:

```bash
docker run --rm playwright-tests npx playwright test --project=webkit
```

The selected browser must also be configured/available in the project.

---

# 15. Run Tests with Environment Variables

Example:

```bash
docker run --rm \
  -e BASE_URL=https://staging.example.com \
  playwright-tests
```

In Playwright:

```typescript
baseURL: process.env.BASE_URL
```

This allows the same Docker image to run against:

```text
DEV
QA
STAGE
PROD
```

without changing the test code.

---

# 16. Multiple Environment Example

Development:

```bash
docker run --rm \
  -e BASE_URL=https://dev.example.com \
  playwright-tests
```

QA:

```bash
docker run --rm \
  -e BASE_URL=https://qa.example.com \
  playwright-tests
```

Stage:

```bash
docker run --rm \
  -e BASE_URL=https://stage.example.com \
  playwright-tests
```

---

# 17. .dockerignore

Create:

```text
.dockerignore
```

Example:

```text
node_modules
playwright-report
test-results
.git
.gitignore
Dockerfile
docker-compose.yml
README.md
.env
```

This prevents unnecessary files from being copied into the Docker image.

---

# 18. Why Exclude node_modules?

Do not copy the local:

```text
node_modules/
```

into the Docker container.

Instead:

```dockerfile
COPY package*.json ./
RUN npm ci
```

This ensures dependencies are installed inside the container for the correct environment.

---

# 19. Dockerfile Optimization

A better Dockerfile:

```dockerfile
FROM mcr.microsoft.com/playwright:1.55.0-noble

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY playwright.config.ts ./
COPY tests ./tests

CMD ["npx", "playwright", "test"]
```

This keeps the image focused on the files required by the tests.

---

# 20. Docker Compose

Docker Compose makes it easier to manage test execution.

Create:

```text
docker-compose.yml
```

Example:

```yaml
services:
  playwright:
    build:
      context: .
      dockerfile: Dockerfile

    environment:
      BASE_URL: https://example.com

    command: npx playwright test

    volumes:
      - ./playwright-report:/app/playwright-report
      - ./test-results:/app/test-results
```

Run:

```bash
docker compose up --build
```

---

# 21. Stop Docker Compose

Run:

```bash
docker compose down
```

---

# 22. Docker Compose with QA Environment

Example:

```yaml
services:
  playwright:
    build:
      context: .

    environment:
      BASE_URL: https://qa.example.com

    command: npx playwright test

    volumes:
      - ./playwright-report:/app/playwright-report
      - ./test-results:/app/test-results
```

---

# 23. Persist Playwright Reports

Containers are temporary.

If the container is deleted, files created inside it can also disappear.

Therefore, mount the report directory:

```yaml
volumes:
  - ./playwright-report:/app/playwright-report
```

This means:

```text
Host
 |
 +-- playwright-report/
 |
 Docker Container
 |
 +-- /app/playwright-report/
```

Test reports remain available on the host machine.

---

# 24. Persist Test Results

Use:

```yaml
volumes:
  - ./test-results:/app/test-results
```

This preserves:

* Screenshots
* Videos
* Traces
* Other test artifacts

---

# 25. Playwright Trace in Docker

Configuration:

```typescript
use: {
  trace: 'retain-on-failure'
}
```

After a failure, trace files can be generated under:

```text
test-results/
```

You can inspect a trace with:

```bash
npx playwright show-trace test-results/path-to-trace.zip
```

If the trace is generated inside Docker, mount the results directory so the file is available outside the container.

---

# 26. Screenshots in Docker

Configuration:

```typescript
use: {
  screenshot: 'only-on-failure'
}
```

Screenshots are stored in:

```text
test-results/
```

Mount the directory:

```yaml
volumes:
  - ./test-results:/app/test-results
```

---

# 27. Videos in Docker

Configuration:

```typescript
use: {
  video: 'retain-on-failure'
}
```

Videos are stored as test artifacts.

Mount:

```yaml
volumes:
  - ./test-results:/app/test-results
```

---

# 28. Running Headless Tests

Docker environments normally execute Playwright in headless mode.

Configuration:

```typescript
use: {
  headless: true
}
```

Run:

```bash
docker run --rm playwright-tests
```

No graphical desktop is required.

---

# 29. Headed Mode

Headed browser execution requires a display environment.

For CI/CD Docker execution, headless mode is generally recommended.

For normal automated CI tests:

```typescript
headless: true
```

---

# 30. Running Tests in Parallel

Playwright supports parallel execution.

Configuration:

```typescript
fullyParallel: true
```

Workers:

```typescript
workers: 4
```

Example:

```typescript
workers: process.env.CI ? 4 : undefined
```

Docker can also be scaled using multiple containers.

---

# 31. Parallel Containers

Instead of one container:

```text
Container 1
   |
 Tests
```

you can run:

```text
Container 1 --> Tests
Container 2 --> Tests
Container 3 --> Tests
Container 4 --> Tests
```

This is useful for large test suites.

However, the total parallelism should be controlled to avoid overloading the CI runner or test environment.

---

# 32. Docker + CI/CD Architecture

Typical architecture:

```text
GitHub / GitLab / Bitbucket
          |
          v
      CI Pipeline
          |
          v
     Docker Build
          |
          v
 Playwright Container
          |
          v
    Run Test Suite
          |
          +----> HTML Report
          |
          +----> Screenshots
          |
          +----> Videos
          |
          +----> Traces
```

---

# 33. Docker with Jenkins

Typical Jenkins pipeline:

```text
Jenkins
   |
   v
Checkout Code
   |
   v
Build Docker Image
   |
   v
Run Playwright Tests
   |
   v
Publish Reports
```

Example Jenkins shell commands:

```bash
docker build -t playwright-tests .

docker run --rm \
  -e BASE_URL=https://qa.example.com \
  -v "$WORKSPACE/playwright-report:/app/playwright-report" \
  -v "$WORKSPACE/test-results:/app/test-results" \
  playwright-tests
```

---

# 34. Docker with GitHub Actions

Example workflow:

```yaml
name: Playwright Docker Tests

on:
  push:
    branches:
      - main

jobs:
  playwright:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Build Docker Image
        run: docker build -t playwright-tests .

      - name: Run Playwright Tests
        run: |
          docker run --rm \
            -e BASE_URL=https://qa.example.com \
            -v ${{ github.workspace }}/playwright-report:/app/playwright-report \
            -v ${{ github.workspace }}/test-results:/app/test-results \
            playwright-tests
```

---

# 35. Environment-Specific Docker Execution

A single image can be reused.

Example:

```text
playwright-tests:latest
```

DEV:

```bash
docker run --rm \
  -e BASE_URL=https://dev.example.com \
  playwright-tests
```

QA:

```bash
docker run --rm \
  -e BASE_URL=https://qa.example.com \
  playwright-tests
```

STAGE:

```bash
docker run --rm \
  -e BASE_URL=https://stage.example.com \
  playwright-tests
```

This is a good CI/CD design because the test image does not need to change between environments.

---

# 36. Secrets and Credentials

Do not hardcode credentials in:

```text
Dockerfile
playwright.config.ts
test files
docker-compose.yml
```

Bad:

```typescript
const username = "admin";
const password = "Password123";
```

Instead use environment variables:

```typescript
const username = process.env.TEST_USERNAME;
const password = process.env.TEST_PASSWORD;
```

Run:

```bash
docker run --rm \
  -e TEST_USERNAME="$TEST_USERNAME" \
  -e TEST_PASSWORD="$TEST_PASSWORD" \
  playwright-tests
```

In CI/CD, store credentials in the CI system's secret manager.

---

# 37. Docker Image Versioning

Avoid relying blindly on:

```dockerfile
latest
```

Prefer a specific Playwright version:

```dockerfile
FROM mcr.microsoft.com/playwright:1.55.0-noble
```

Your:

```text
package.json
```

should use a compatible Playwright version.

For example:

```json
"@playwright/test": "^1.55.0"
```

Keeping the Docker image and Playwright package aligned helps avoid browser/version mismatches.

---

# 38. Browser Version Compatibility

A common problem is:

```text
Playwright package version
        !=
Browser version
```

This can cause unexpected failures.

Best practice:

```text
Playwright package
       |
       +---- Compatible Docker image
                    |
                    +---- Browser binaries
```

Keep versions synchronized.

---

# 39. Common Docker Commands

Build:

```bash
docker build -t playwright-tests .
```

List images:

```bash
docker images
```

Run:

```bash
docker run --rm playwright-tests
```

List containers:

```bash
docker ps
```

List all containers:

```bash
docker ps -a
```

Remove container:

```bash
docker rm <container-id>
```

Remove image:

```bash
docker rmi playwright-tests
```

View logs:

```bash
docker logs <container-id>
```

---

# 40. Debugging Docker Test Failures

Run the container interactively:

```bash
docker run --rm -it playwright-tests /bin/bash
```

Inside the container:

```bash
node --version
```

Check Playwright:

```bash
npx playwright --version
```

Run tests manually:

```bash
npx playwright test
```

List files:

```bash
ls
```

This is useful for troubleshooting.

---

# 41. Common Problem: Browser Missing

Error may look similar to:

```text
Executable doesn't exist
```

Solution:

Use the official Playwright Docker image or install browsers:

```bash
npx playwright install --with-deps
```

When using the official Playwright Docker image, browsers and required dependencies are already provided.

---

# 42. Common Problem: Permission Errors

Some Linux environments may produce permission problems.

For example:

```text
Permission denied
```

Check:

```bash
docker logs <container-id>
```

Also verify:

* File ownership
* Mounted directory permissions
* CI runner permissions
* Docker user configuration

---

# 43. Common Problem: Report Missing

If the report disappears after the container exits, the report was probably created only inside the container.

Use a volume:

```bash
-v "$PWD/playwright-report:/app/playwright-report"
```

Or Docker Compose:

```yaml
volumes:
  - ./playwright-report:/app/playwright-report
```

---

# 44. Common Problem: Tests Cannot Access Application

Possible causes:

* Incorrect `BASE_URL`
* DNS issue
* Network restrictions
* Proxy configuration
* Application unavailable
* Certificate issue
* Container network configuration

Debug with:

```bash
docker run --rm playwright-tests
```

and verify the URL from inside the container.

---

# 45. Common Problem: TLS/Certificate Errors

Some test environments use internal certificates.

For a test-only environment, Playwright can be configured to ignore HTTPS errors:

```typescript
use: {
  ignoreHTTPSErrors: true
}
```

Use this carefully.

It is generally better to configure the correct trusted certificate chain rather than disabling certificate validation.

---

# 46. Dockerfile Best Practices

### 1. Use a specific image version

```dockerfile
FROM mcr.microsoft.com/playwright:1.55.0-noble
```

### 2. Use npm ci

```dockerfile
RUN npm ci
```

### 3. Use `.dockerignore`

```text
node_modules
test-results
playwright-report
.git
```

### 4. Do not hardcode secrets

Use environment variables or CI/CD secrets.

### 5. Persist reports

Use Docker volumes.

### 6. Keep images reproducible

Pin important versions.

### 7. Run headless in CI

```typescript
headless: true
```

---

# 47. Recommended Project Structure

```text
playwright-docker/
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
├── utils/
│   └── testData.ts
│
├── playwright.config.ts
├── package.json
├── package-lock.json
│
├── Dockerfile
├── .dockerignore
├── docker-compose.yml
│
├── playwright-report/
└── test-results/
```

---

# 48. Complete Dockerfile Example

```dockerfile
FROM mcr.microsoft.com/playwright:1.55.0-noble

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

ENV CI=true

CMD ["npx", "playwright", "test"]
```

---

# 49. Complete docker-compose.yml Example

```yaml
services:
  playwright:
    build:
      context: .
      dockerfile: Dockerfile

    environment:
      BASE_URL: https://qa.example.com

    command: npx playwright test

    volumes:
      - ./playwright-report:/app/playwright-report
      - ./test-results:/app/test-results
```

Run:

```bash
docker compose up --build
```

---

# 50. Complete Playwright Configuration

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
    ['html', {
      outputFolder: 'playwright-report',
      open: 'never'
    }]
  ],

  use: {
    baseURL: process.env.BASE_URL || 'https://example.com',

    headless: true,

    screenshot: 'only-on-failure',

    video: 'retain-on-failure',

    trace: 'retain-on-failure',

    ignoreHTTPSErrors: false
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

# 51. End-to-End Docker Execution

Complete flow:

```text
1. Developer writes Playwright tests
              |
              v
2. package.json created
              |
              v
3. Dockerfile created
              |
              v
4. Docker image built
              |
              v
5. Container started
              |
              v
6. Playwright tests execute
              |
              v
7. Browser launches
              |
              v
8. Tests complete
              |
              +----> HTML Report
              |
              +----> Screenshots
              |
              +----> Videos
              |
              +----> Traces
              |
              v
9. Container exits
```

---

# 52. Docker vs Local Execution

| Feature                 | Local        | Docker          |
| ----------------------- | ------------ | --------------- |
| Node.js                 | Host machine | Container       |
| Browser dependencies    | Host machine | Container       |
| Environment consistency | Medium       | High            |
| CI/CD                   | Good         | Excellent       |
| Isolation               | Low          | High            |
| Reproducibility         | Medium       | High            |
| Setup                   | Easy         | Requires Docker |
| Scalability             | Good         | Excellent       |

---

# 53. Docker vs Selenium Grid

Docker and Selenium Grid solve different problems.

### Docker

Provides:

```text
Containerized Environment
```

### Selenium Grid

Provides:

```text
Distributed Browser Execution
```

Playwright can use Docker without Selenium Grid.

Example:

```text
Docker
  |
  +-- Playwright
  +-- Chromium
  +-- Tests
```

You do not need Selenium Grid simply because you are using Docker.

---

# 54. Docker + Playwright + Jenkins

A common enterprise architecture:

```text
Git
 |
 v
Jenkins
 |
 +-- Checkout
 |
 +-- Build Docker Image
 |
 +-- Run Playwright
 |
 +-- Generate HTML Report
 |
 +-- Archive Artifacts
 |
 v
Test Result
```

This provides a clean and reproducible automation pipeline.

---

# 55. Docker + Playwright + GitHub Actions

Another common architecture:

```text
GitHub
   |
   v
GitHub Actions
   |
   v
Ubuntu Runner
   |
   v
Docker
   |
   v
Playwright
   |
   v
Browser
   |
   v
Tests
   |
   +----> Report
   +----> Screenshot
   +----> Video
   +----> Trace
```

---

# 56. Interview Questions

## Beginner

### 1. Why use Docker with Playwright?

Docker provides a consistent and reproducible environment containing the required Node.js, browser dependencies and Playwright runtime.

### 2. What is a Dockerfile?

A Dockerfile contains instructions for building a Docker image.

### 3. How do you build a Playwright Docker image?

```bash
docker build -t playwright-tests .
```

### 4. How do you run the image?

```bash
docker run --rm playwright-tests
```

### 5. Why use the official Playwright Docker image?

It provides the required browser dependencies and system libraries needed by Playwright.

---

# 57. Intermediate Interview Questions

### 6. Why use `npm ci` instead of `npm install`?

`npm ci` installs dependencies based on the lock file and is designed for clean, reproducible CI installations.

### 7. How do you pass environment variables into Docker?

```bash
docker run --rm \
  -e BASE_URL=https://qa.example.com \
  playwright-tests
```

### 8. How do you preserve Playwright reports?

Use a Docker volume:

```bash
-v "$PWD/playwright-report:/app/playwright-report"
```

### 9. How do you debug a Playwright Docker container?

Run interactively:

```bash
docker run --rm -it playwright-tests /bin/bash
```

### 10. Why use `.dockerignore`?

To prevent unnecessary files such as `node_modules`, reports and Git files from being copied into the image.

---

# 58. Advanced Interview Questions

### 11. How would you run Playwright tests against multiple environments?

Use environment variables:

```bash
-e BASE_URL=https://qa.example.com
```

and read them through:

```typescript
process.env.BASE_URL
```

### 12. How would you store credentials?

Use CI/CD secret management and environment variables instead of hardcoding credentials.

### 13. How would you collect traces from Docker?

Configure:

```typescript
trace: 'retain-on-failure'
```

and mount:

```text
test-results/
```

outside the container.

### 14. How can Docker improve CI/CD reliability?

It standardizes:

* Node.js
* Playwright
* Browsers
* Linux dependencies
* Test execution environment

### 15. How would you scale Playwright tests using Docker?

Use multiple containers or CI workers to distribute test execution while controlling total parallelism.

---

# 59. Senior-Level Interview Question

### How would you design an enterprise Playwright Docker framework?

A good architecture would include:

```text
Source Control
     |
     v
CI/CD Pipeline
     |
     v
Docker Build
     |
     v
Versioned Playwright Image
     |
     +-------------------+
     |                   |
     v                   v
Worker 1              Worker 2
     |                   |
     v                   v
Playwright            Playwright
Tests                 Tests
     |                   |
     +---------+---------+
               |
               v
       Test Results
               |
       +-------+-------+
       |       |       |
       v       v       v
     HTML   Trace   Screenshots
     Report
```

Important considerations:

* Version pinning
* Reproducible builds
* Environment variables
* Secret management
* Parallel execution
* Test sharding
* Artifact collection
* Failure diagnostics
* Docker image caching
* CI resource management
* Browser/version compatibility

---

# 60. Best Practices Summary

```text
Use official Playwright Docker images
              +
Pin compatible Playwright versions
              +
Use npm ci
              +
Use .dockerignore
              +
Keep secrets outside source code
              +
Use environment variables
              +
Run headless in CI
              +
Persist reports and artifacts
              +
Enable trace on failures
              +
Control parallelism
              +
Use Docker in CI/CD
```

---

# 61. Quick Reference

### Build

```bash
docker build -t playwright-tests .
```

### Run

```bash
docker run --rm playwright-tests
```

### Run with environment

```bash
docker run --rm \
  -e BASE_URL=https://qa.example.com \
  playwright-tests
```

### Interactive container

```bash
docker run --rm -it playwright-tests /bin/bash
```

### Docker Compose

```bash
docker compose up --build
```

### Stop Compose

```bash
docker compose down
```

### View containers

```bash
docker ps -a
```

### View logs

```bash
docker logs <container-id>
```

### Remove image

```bash
docker rmi playwright-tests
```

---

# 62. Final Takeaway

Docker + Playwright is especially useful for enterprise automation because it creates a repeatable test environment.

The key concept is:

```text
Playwright Tests
       |
       v
Docker Container
       |
       +-- Node.js
       +-- Playwright
       +-- Browsers
       +-- OS Dependencies
       |
       v
CI/CD Pipeline
       |
       +-- Test Results
       +-- HTML Reports
       +-- Screenshots
       +-- Videos
       +-- Traces
```

For a senior QA automation engineer, the important skills are not only writing Playwright tests but also understanding how to package, execute, scale, debug and report those tests in Docker-based CI/CD environments.
