# Playwright Installation

## 1. Overview

Playwright can be installed and configured using Node.js and npm.

For a modern Playwright automation project, the recommended setup is:

```text
Node.js
   ↓
npm
   ↓
Playwright
   ↓
Playwright Test
   ↓
Chromium / Firefox / WebKit
```

The most common setup for automation is:

```text
TypeScript + Playwright + Playwright Test
```

JavaScript is also fully supported.

---

# 2. Prerequisites

Before installing Playwright, install:

* Node.js
* npm
* VS Code or another IDE
* Git
* A browser is optional because Playwright can install its own browser binaries

Check Node.js:

```bash
node --version
```

Check npm:

```bash
npm --version
```

Example:

```text
node --version
v22.x.x

npm --version
10.x.x
```

The exact versions may differ depending on your environment.

---

# 3. Install Node.js

Go to the official Node.js website and install the current LTS release.

After installation, open a new terminal and verify:

```bash
node --version
npm --version
```

If both commands return versions, Node.js and npm are available.

---

# 4. Create a Playwright Project

Create a project directory:

```bash
mkdir PlaywrightStudy
```

Move into it:

```bash
cd PlaywrightStudy
```

Initialize npm:

```bash
npm init -y
```

This creates:

```text
package.json
```

Example:

```json
{
  "name": "playwrightstudy",
  "version": "1.0.0",
  "scripts": {},
  "dependencies": {}
}
```

---

# 5. Install Playwright Test

Install Playwright:

```bash
npm init playwright@latest
```

This command creates a Playwright project and asks several configuration questions.

Typical prompts include:

```text
✔ Do you want to use TypeScript or JavaScript?
✔ Where to put your end-to-end tests?
✔ Add a GitHub Actions workflow?
✔ Install Playwright browsers?
```

For a new automation project, a typical choice is:

```text
TypeScript
tests
Yes/No depending on CI requirement
Yes
```

---

# 6. Alternative Installation

You can install the Playwright Test package directly:

```bash
npm install -D @playwright/test
```

Then install the browser binaries:

```bash
npx playwright install
```

This approach is useful when you want more control over project creation.

---

# 7. Install Specific Browsers

Install all supported browsers:

```bash
npx playwright install
```

Install Chromium:

```bash
npx playwright install chromium
```

Install Firefox:

```bash
npx playwright install firefox
```

Install WebKit:

```bash
npx playwright install webkit
```

---

# 8. Install System Dependencies

On Linux CI environments, you may need browser system dependencies.

You can use:

```bash
npx playwright install --with-deps
```

This installs Playwright browsers and required operating-system dependencies.

This is particularly useful for CI/CD environments such as Linux-based build agents.

---

# 9. Verify Installation

After installation, run:

```bash
npx playwright test
```

If the installation is successful, Playwright should discover and execute the sample tests.

You can also verify the Playwright version:

```bash
npx playwright --version
```

Example:

```text
Version 1.x.x
```

---

# 10. Project Structure

A newly created Playwright project commonly looks like:

```text
PlaywrightStudy/
│
├── tests/
│   └── example.spec.ts
│
├── tests-examples/
│   └── demo-todo-app.spec.ts
│
├── playwright.config.ts
├── package.json
├── package-lock.json
└── node_modules/
```

Depending on the Playwright version and setup choices, the generated structure may vary.

---

# 11. Important Files

## package.json

Contains:

* Project information
* Dependencies
* Scripts

Example:

```json
{
  "name": "playwrightstudy",
  "version": "1.0.0",
  "scripts": {
    "test": "playwright test"
  },
  "devDependencies": {
    "@playwright/test": "^1.x.x"
  }
}
```

---

# 12. package-lock.json

`package-lock.json` records the exact dependency tree installed by npm.

It helps maintain consistent installations across environments.

You should normally commit it to Git.

---

# 13. node_modules

`node_modules` contains installed npm packages.

Example:

```text
node_modules/
    |
    +── @playwright/
    +── @playwright
    +── other dependencies
```

Do not normally commit `node_modules` to Git.

Add it to `.gitignore`:

```text
node_modules/
```

---

# 14. playwright.config.ts

This is the central Playwright configuration file.

Example:

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({

    testDir: './tests',

    use: {
        baseURL: 'https://example.com',
        headless: true
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

Configuration can control:

* Test directory
* Browser
* Base URL
* Headless/headed mode
* Timeouts
* Screenshots
* Videos
* Traces
* Retries
* Workers
* Projects
* Reporter
* Parallel execution

---

# 15. Create Your First Test

Create:

```text
tests/login.spec.ts
```

Example:

```typescript
import { test, expect } from '@playwright/test';

test('verify Example page', async ({ page }) => {

    await page.goto('https://example.com');

    await expect(page).toHaveTitle(/Example/);

});
```

---

# 16. Run the Test

Run all tests:

```bash
npx playwright test
```

Run a specific file:

```bash
npx playwright test tests/login.spec.ts
```

Run a specific test:

```bash
npx playwright test -g "verify Example page"
```

---

# 17. Run in Headed Mode

By default, tests normally run headlessly.

To see the browser:

```bash
npx playwright test --headed
```

This is useful when learning and debugging.

---

# 18. Run in Debug Mode

Use:

```bash
npx playwright test --debug
```

Debug mode opens Playwright Inspector and allows you to step through the test.

---

# 19. Run Codegen

Playwright includes a code generator.

Run:

```bash
npx playwright codegen https://example.com
```

A browser opens and Playwright generates automation code as you interact with the application.

Codegen is useful for learning:

* Locators
* Click actions
* Fill actions
* Navigation
* Assertions

Example generated code:

```typescript
await page.getByRole('button', { name: 'Login' }).click();
```

---

# 20. Install Playwright in an Existing Project

If you already have a Node.js project:

```bash
npm install -D @playwright/test
```

Then:

```bash
npx playwright install
```

Create:

```text
playwright.config.ts
```

Create:

```text
tests/
```

Then add your test files.

---

# 21. Installing Playwright in VS Code

VS Code works very well with Playwright.

Typical workflow:

```text
VS Code
   ↓
Open Playwright project
   ↓
Open Terminal
   ↓
npm install
   ↓
npx playwright install
   ↓
Create tests
   ↓
Run tests
```

You can execute commands from:

```text
Terminal → New Terminal
```

---

# 22. Recommended VS Code Extensions

Useful extensions include:

* Playwright Test for VS Code
* ESLint
* Prettier
* GitLens
* GitHub Pull Requests and Issues

The Playwright VS Code extension can help you:

* Run tests
* Debug tests
* View test results
* Pick tests
* Use Playwright tooling from the editor

---

# 23. Windows PowerShell

On Windows, you may run Playwright commands from:

```text
PowerShell
```

or:

```text
Command Prompt
```

or:

```text
VS Code Terminal
```

Example:

```powershell
npx playwright test
```

If PowerShell reports an execution-policy error when running npm-related commands, use an appropriate terminal such as Command Prompt, or adjust your PowerShell execution policy according to your organization's security rules.

Do not disable security protections unnecessarily.

---

# 24. Verify Browser Installation

Run:

```bash
npx playwright install --dry-run
```

This can show the browser installation information without performing the installation.

To install the browsers:

```bash
npx playwright install
```

---

# 25. Browser Cache Location

Playwright stores browser binaries separately from your project.

You generally do not need to manually manage these browser executables.

This is different from Selenium, where you may have historically needed to think about:

```text
ChromeDriver
GeckoDriver
EdgeDriver
```

Modern Selenium Manager reduces much of that manual driver management.

Playwright manages its supported browser binaries through its installation process.

---

# 26. Playwright vs Selenium Installation

## Selenium

A Java Selenium project commonly requires:

```text
Java
   ↓
Maven
   ↓
Selenium dependency
   ↓
TestNG/JUnit
   ↓
Browser
```

Historically, WebDriver binaries were also managed separately, although Selenium Manager now automates driver management in modern Selenium versions.

## Playwright

A Node.js Playwright project commonly uses:

```text
Node.js
   ↓
npm
   ↓
@playwright/test
   ↓
Playwright browsers
```

This makes the initial browser automation setup straightforward.

---

# 27. npm Scripts

You can add convenient commands to `package.json`.

Example:

```json
{
  "scripts": {
    "test": "playwright test",
    "test:headed": "playwright test --headed",
    "test:debug": "playwright test --debug",
    "test:chromium": "playwright test --project=chromium",
    "report": "playwright show-report"
  }
}
```

Then run:

```bash
npm test
```

or:

```bash
npm run test:headed
```

or:

```bash
npm run test:debug
```

---

# 28. Test Report

After running tests, Playwright can generate an HTML report.

Run:

```bash
npx playwright show-report
```

The report can provide:

* Passed tests
* Failed tests
* Test duration
* Errors
* Attachments
* Screenshots
* Traces
* Videos when configured

Reporting will be covered in detail in the Reporting topic.

---

# 29. Clean Installation

If dependencies become corrupted, you can remove:

```text
node_modules
package-lock.json
```

Then reinstall:

```bash
npm install
```

For browser binaries:

```bash
npx playwright install
```

However, don't delete `package-lock.json` routinely. In team projects, keeping the lock file provides reproducible dependency installation.

---

# 30. Updating Playwright

Check the installed version:

```bash
npx playwright --version
```

Update the package:

```bash
npm install -D @playwright/test@latest
```

Then install/update browser binaries:

```bash
npx playwright install
```

Always review release notes and test the framework after upgrading a major version.

---

# 31. Uninstall Playwright

Remove the npm package:

```bash
npm uninstall @playwright/test
```

If the project is no longer needed, you can also remove the project directory.

---

# 32. Git Setup

A typical `.gitignore` should include:

```text
node_modules/
test-results/
playwright-report/
blob-report/
```

Depending on your framework, you may also exclude:

```text
*.log
.env
```

Do not commit secrets such as:

```text
username
password
API keys
tokens
client secrets
```

Use environment variables or a secure secret-management solution instead.

---

# 33. Recommended Project Setup

For your PlaywrightStudy repository, a clean initial setup can be:

```text
PlaywrightStudy/
│
├── tests/
├── pages/
├── fixtures/
├── utils/
├── test-data/
├── config/
│
├── playwright.config.ts
├── package.json
├── package-lock.json
└── .gitignore
```

As the framework grows, this can evolve into:

```text
PlaywrightStudy/
│
├── tests/
├── pages/
├── components/
├── fixtures/
├── utils/
├── api/
├── test-data/
├── auth/
├── config/
├── reports/
│
├── playwright.config.ts
├── package.json
├── package-lock.json
└── .gitignore
```

---

# 34. Installation Checklist

```text
Node.js
    ↓
npm
    ↓
Create project
    ↓
npm init playwright@latest
    ↓
Install browsers
    ↓
npx playwright install
    ↓
Create tests
    ↓
npx playwright test
    ↓
Verify report
```

---

# 35. Common Installation Problems

## Problem 1: `node` is not recognized

Verify Node.js installation:

```bash
node --version
```

If it fails, Node.js may not be installed or may not be available in PATH.

---

## Problem 2: `npm` is not recognized

Check:

```bash
npm --version
```

If it fails, verify Node.js installation and PATH configuration.

---

## Problem 3: Browser executable is missing

Run:

```bash
npx playwright install
```

---

## Problem 4: Installation fails on Linux

Try:

```bash
npx playwright install --with-deps
```

---

## Problem 5: Tests cannot find the test file

Verify:

```text
testDir
```

in:

```text
playwright.config.ts
```

For example:

```typescript
testDir: './tests'
```

---

# 36. Recommended Installation Commands

For a new project:

```bash
mkdir PlaywrightStudy
cd PlaywrightStudy
npm init playwright@latest
```

Then:

```bash
npx playwright install
```

Run:

```bash
npx playwright test
```

Run headed:

```bash
npx playwright test --headed
```

Run debug:

```bash
npx playwright test --debug
```

Open report:

```bash
npx playwright show-report
```

---

# 37. Interview Questions

### Beginner

1. How do you install Playwright?
2. What is `npm init playwright@latest`?
3. What is `@playwright/test`?
4. How do you install Playwright browsers?
5. How do you verify Playwright installation?
6. What is `playwright.config.ts`?
7. How do you run all Playwright tests?
8. How do you run a specific test file?
9. How do you run tests in headed mode?
10. How do you run Playwright in debug mode?

### Intermediate

11. How do you install only Chromium?
12. How do you install browser dependencies on Linux?
13. Where are Playwright browser binaries managed?
14. How do you configure a base URL?
15. How do you configure multiple browsers?
16. How do you configure retries?
17. How do you configure workers?
18. How do you configure screenshots?
19. How do you configure video recording?
20. How do you generate an HTML report?

### Senior-Level

21. How would you structure a Playwright framework from scratch?
22. How would you manage Playwright dependencies in CI/CD?
23. How would you handle browser version changes?
24. How would you manage environment-specific configuration?
25. How would you securely manage credentials?
26. How would you optimize Playwright installation in CI?
27. How would you cache Playwright browsers in CI/CD?
28. How would you design a multi-browser project configuration?
29. How would you troubleshoot browser installation failures?
30. How would you migrate an existing Selenium framework to Playwright?

---

# 38. Key Takeaways

Remember these commands:

```bash
# Create Playwright project
npm init playwright@latest

# Install Playwright package
npm install -D @playwright/test

# Install browsers
npx playwright install

# Install browsers + Linux dependencies
npx playwright install --with-deps

# Check version
npx playwright --version

# Run tests
npx playwright test

# Run headed
npx playwright test --headed

# Debug
npx playwright test --debug

# Generate tests
npx playwright codegen https://example.com

# Open report
npx playwright show-report
```

The most important files are:

```text
package.json
package-lock.json
playwright.config.ts
tests/
.gitignore
```

The basic Playwright installation flow is:

```text
Node.js
   ↓
npm
   ↓
@playwright/test
   ↓
Playwright browsers
   ↓
playwright.config.ts
   ↓
Tests
   ↓
Playwright Test Runner
```

This completes the **Playwright Installation** foundation. The next topic is:

**`Browsers/Playwright-Browsers.md`**
