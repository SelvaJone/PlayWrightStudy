# Playwright Tracing

## Overview

Playwright Tracing is a powerful debugging feature that records detailed information about test execution.

A Playwright trace can contain:

* Actions performed during the test
* Screenshots
* DOM snapshots
* Network requests
* Console messages
* Source code
* Test timing
* Browser information
* Errors
* Before/after snapshots of actions

The recorded trace can be opened using **Playwright Trace Viewer**.

---

# 1. Why Use Playwright Tracing?

Tracing is especially useful when a test:

* Passes locally but fails in CI
* Fails intermittently
* Has timing issues
* Has locator problems
* Has unexpected navigation
* Has network-related failures
* Is difficult to reproduce
* Runs in a remote CI environment

Instead of trying to reproduce the failure manually, you can inspect the recorded trace.

---

# 2. Basic Trace Concepts

A trace records a sequence of test activities.

For example:

```text
Test starts
   |
   v
Navigate to application
   |
   v
Fill username
   |
   v
Fill password
   |
   v
Click Login
   |
   v
Navigate to Dashboard
   |
   v
Assertion
   |
   v
Test fails
   |
   v
Trace saved
```

The trace can then be opened in Trace Viewer.

---

# 3. Starting a Trace Manually

Tracing is controlled through the browser context.

```java
context.tracing().start(new Tracing.StartOptions()
        .setScreenshots(true)
        .setSnapshots(true)
        .setSources(true));
```

Example:

```java
Browser browser = playwright.chromium().launch(
        new BrowserType.LaunchOptions().setHeadless(false)
);

BrowserContext context = browser.newContext();

context.tracing().start(new Tracing.StartOptions()
        .setScreenshots(true)
        .setSnapshots(true)
        .setSources(true));

Page page = context.newPage();

page.navigate("https://example.com");

page.locator("#username").fill("admin");

page.locator("#password").fill("password");

page.locator("button[type='submit']").click();

context.tracing().stop(
        new Tracing.StopOptions()
                .setPath(Paths.get("trace.zip"))
);

browser.close();
```

---

# 4. Tracing Options

Important tracing options include:

```java
new Tracing.StartOptions()
        .setScreenshots(true)
        .setSnapshots(true)
        .setSources(true);
```

### `screenshots`

Records screenshots during actions.

```java
.setScreenshots(true)
```

### `snapshots`

Records DOM snapshots.

```java
.setSnapshots(true)
```

### `sources`

Includes source files in the trace.

```java
.setSources(true)
```

Using all three provides a detailed debugging experience.

---

# 5. Stopping a Trace

Stop the trace using:

```java
context.tracing().stop(
        new Tracing.StopOptions()
                .setPath(Paths.get("trace.zip"))
);
```

The trace is saved as:

```text
trace.zip
```

---

# 6. Complete Manual Tracing Example

```java
import com.microsoft.playwright.*;

import java.nio.file.Paths;

public class TracingExample {

    public static void main(String[] args) {

        try (Playwright playwright = Playwright.create()) {

            Browser browser = playwright.chromium().launch(
                    new BrowserType.LaunchOptions()
                            .setHeadless(false)
            );

            BrowserContext context = browser.newContext();

            context.tracing().start(
                    new Tracing.StartOptions()
                            .setScreenshots(true)
                            .setSnapshots(true)
                            .setSources(true)
            );

            Page page = context.newPage();

            page.navigate("https://example.com");

            System.out.println(page.title());

            context.tracing().stop(
                    new Tracing.StopOptions()
                            .setPath(Paths.get("trace.zip"))
            );

            browser.close();
        }
    }
}
```

---

# 7. Trace Viewer

Playwright provides a Trace Viewer for inspecting trace files.

Open a trace using:

```bash
npx playwright show-trace trace.zip
```

Example:

```bash
npx playwright show-trace trace.zip
```

This opens the trace in the browser.

---

# 8. What Can You See in Trace Viewer?

Trace Viewer provides detailed information about each action.

Typical information includes:

```text
Test
 |
 +-- Navigate
 |
 +-- Fill username
 |
 +-- Fill password
 |
 +-- Click Login
 |
 +-- Expect Dashboard
```

Selecting an action allows you to inspect:

* Action details
* Locator
* Before snapshot
* After snapshot
* Screenshot
* DOM
* Network activity
* Console output
* Source code
* Timing

---

# 9. Screenshots in Tracing

Enable screenshots:

```java
context.tracing().start(
        new Tracing.StartOptions()
                .setScreenshots(true)
);
```

Screenshots help determine what the browser looked like at a particular point.

For example:

```text
Click Login
     |
     +-- Before Screenshot
     |
     +-- Click
     |
     +-- After Screenshot
```

This is useful for debugging UI problems.

---

# 10. DOM Snapshots

Enable snapshots:

```java
.setSnapshots(true)
```

DOM snapshots allow you to inspect the page structure during test execution.

This is especially helpful when:

* Elements are dynamically generated
* Elements disappear
* The page changes after an action
* A locator cannot find an element
* A modal appears unexpectedly

---

# 11. Source Files

Enable source files:

```java
.setSources(true)
```

This allows source information to be included in the trace.

It makes debugging easier because you can identify the source code associated with the test action.

---

# 12. Tracing with Playwright Test

In Playwright Test, tracing can be configured in `playwright.config.ts`.

Example:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
    use: {
        trace: 'on-first-retry'
    }
});
```

This is one of the most commonly recommended configurations.

---

# 13. Trace Modes

Playwright Test supports different trace configurations.

### `off`

Tracing is disabled.

```typescript
use: {
    trace: 'off'
}
```

### `on`

Trace every test.

```typescript
use: {
    trace: 'on'
}
```

### `on-first-retry`

Record a trace when a test is retried for the first time.

```typescript
use: {
    trace: 'on-first-retry'
}
```

This is very useful in CI because it avoids creating traces for every successful test.

### `retain-on-failure`

Retain traces for failed tests.

```typescript
use: {
    trace: 'retain-on-failure'
}
```

### `retain-on-first-failure`

Retain a trace for the first failure.

```typescript
use: {
    trace: 'retain-on-first-failure'
}
```

---

# 14. Recommended Configuration

For CI/CD pipelines, a common configuration is:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({

    use: {
        trace: 'on-first-retry'
    }

});
```

Why?

Successful tests normally do not need detailed traces.

If a test fails and is retried, the trace can provide valuable debugging information.

---

# 15. Tracing with Retries

Example:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
    retries: 2,

    use: {
        trace: 'on-first-retry'
    }
});
```

Execution:

```text
Attempt 1
   |
   +-- FAIL
   |
   v
Retry
   |
   +-- Trace recorded
   |
   v
Attempt 2
```

This is particularly useful for flaky tests.

---

# 16. Trace Configuration by Environment

You can configure tracing differently for local and CI execution.

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({

    use: {
        trace: process.env.CI
            ? 'on-first-retry'
            : 'off'
    }

});
```

Local:

```text
Trace = off
```

CI:

```text
Trace = on-first-retry
```

This reduces unnecessary trace generation during local development.

---

# 17. Trace Directory

You can configure where traces and other artifacts are stored through test configuration.

Example:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({

    outputDir: 'test-results',

    use: {
        trace: 'on-first-retry'
    }

});
```

Typical project structure:

```text
project/
│
├── tests/
│   └── login.spec.ts
│
├── test-results/
│   └── ...
│
├── playwright.config.ts
│
└── package.json
```

---

# 18. Viewing a Trace from Test Results

After a failed test, Playwright may generate trace artifacts under the test results directory.

You can inspect the generated trace with:

```bash
npx playwright show-trace path/to/trace.zip
```

Example:

```bash
npx playwright show-trace test-results/login-test/trace.zip
```

---

# 19. Trace Viewer Workflow

A typical debugging workflow is:

```text
1. Test fails
       |
       v
2. Locate trace.zip
       |
       v
3. Open Trace Viewer
       |
       v
4. Select failed action
       |
       v
5. Inspect screenshot
       |
       v
6. Inspect DOM snapshot
       |
       v
7. Inspect network
       |
       v
8. Inspect console
       |
       v
9. Identify root cause
       |
       v
10. Fix test/application
```

---

# 20. Trace vs Screenshot

A screenshot captures a single point in time.

Tracing captures the complete execution history.

### Screenshot

```text
Test
 |
 +-- Action
 |
 +-- Screenshot
```

### Trace

```text
Test
 |
 +-- Navigation
 +-- Screenshot
 +-- DOM
 +-- Network
 +-- Console
 +-- Action
 +-- Timing
 +-- Source
```

Therefore, tracing is much more useful for complex failures.

---

# 21. Trace vs Video

### Screenshot

Shows one point in time.

### Video

Shows visual browser execution.

### Trace

Provides interactive debugging information.

Comparison:

| Feature               | Screenshot | Video   | Trace |
| --------------------- | ---------- | ------- | ----- |
| Visual state          | Yes        | Yes     | Yes   |
| DOM snapshot          | No         | No      | Yes   |
| Network               | No         | No      | Yes   |
| Action details        | No         | Limited | Yes   |
| Source                | No         | No      | Yes   |
| Timing                | Limited    | Yes     | Yes   |
| Interactive debugging | No         | Limited | Yes   |

---

# 22. Tracing and Network Debugging

Tracing can help identify network-related problems.

For example:

```text
Click Login
     |
     v
POST /api/login
     |
     +-- 500 Internal Server Error
     |
     v
Login page remains open
```

The trace can help determine that the failure was caused by an API response rather than a locator problem.

---

# 23. Tracing and Locator Debugging

Suppose the test contains:

```typescript
await page.locator('#loginButton').click();
```

The test fails.

Using Trace Viewer, you can inspect:

* Locator
* DOM snapshot
* Element state
* Screenshot
* Action timing

You may discover that the actual element is:

```html
<button id="signInButton">
    Sign In
</button>
```

The trace helps identify the incorrect locator.

---

# 24. Tracing and Timing Problems

Consider:

```typescript
await page.locator('#submit').click();

await expect(page.locator('#dashboard'))
    .toBeVisible();
```

If the test fails intermittently, the trace can show:

```text
Click Submit
    |
    v
API request
    |
    v
Page navigation
    |
    v
Dashboard rendering
    |
    v
Assertion
```

This can help determine whether the issue is:

* Slow network
* Application delay
* Incorrect wait
* Navigation problem
* Rendering issue

---

# 25. Trace with Authentication

Tracing can also help debug authentication flows.

Example:

```text
Open Login
     |
     v
Enter Username
     |
     v
Enter Password
     |
     v
Click Login
     |
     v
Authentication Request
     |
     v
Dashboard
```

You can inspect the sequence if authentication fails.

Be careful with sensitive information when sharing traces.

---

# 26. Security Considerations

Traces can contain sensitive information.

They may include:

* Usernames
* Form data
* URLs
* Cookies
* Network information
* Application data
* Screenshots

Therefore:

**Do not upload traces containing sensitive data to public repositories.**

For example, avoid committing:

```text
trace.zip
```

to GitHub.

Add generated test artifacts to `.gitignore` when appropriate.

Example:

```gitignore
test-results/
playwright-report/
trace.zip
```

---

# 27. Tracing in CI/CD

Tracing is especially valuable in CI/CD.

Example:

```text
Developer
   |
   v
Git Push
   |
   v
CI Pipeline
   |
   v
Playwright Tests
   |
   +---- PASS
   |
   +---- FAIL
           |
           v
       Retry Test
           |
           v
       Trace Created
           |
           v
      CI Artifact
           |
           v
       Debug Failure
```

Instead of reproducing the failure locally, the QA engineer can inspect the trace artifact.

---

# 28. GitHub Actions Example

Example Playwright configuration:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({

    retries: process.env.CI ? 2 : 0,

    use: {
        trace: 'on-first-retry'
    }

});
```

GitHub Actions can then preserve test artifacts.

Example:

```yaml
- name: Run Playwright tests
  run: npx playwright test

- name: Upload Playwright report
  if: always()
  uses: actions/upload-artifact@v4
  with:
    name: playwright-report
    path: playwright-report/
```

You can also preserve:

```text
test-results/
```

when trace files are generated there.

---

# 29. Best Practice for Senior QA Automation

A practical configuration is:

```typescript
use: {
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure'
}
```

Example:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({

    retries: process.env.CI ? 2 : 0,

    use: {
        trace: 'on-first-retry',
        screenshot: 'only-on-failure',
        video: 'retain-on-failure'
    }

});
```

This provides useful failure diagnostics without generating unnecessary artifacts for every successful test.

---

# 30. Example Project Structure

A Playwright project may look like:

```text
PlaywrightProject/
│
├── tests/
│   ├── login.spec.ts
│   └── checkout.spec.ts
│
├── pages/
│   ├── LoginPage.ts
│   └── CheckoutPage.ts
│
├── fixtures/
│   └── test-fixtures.ts
│
├── test-data/
│   └── users.json
│
├── test-results/
│
├── playwright-report/
│
├── playwright.config.ts
│
├── package.json
│
└── .gitignore
```

---

# 31. Common Trace Commands

Run tests:

```bash
npx playwright test
```

Run a specific test:

```bash
npx playwright test tests/login.spec.ts
```

Open HTML report:

```bash
npx playwright show-report
```

Open trace:

```bash
npx playwright show-trace trace.zip
```

---

# 32. Common Mistakes

## Mistake 1: Recording traces for every test

```typescript
trace: 'on'
```

This can generate a large amount of data.

For CI, consider:

```typescript
trace: 'on-first-retry'
```

---

## Mistake 2: Committing traces to Git

Avoid committing generated artifacts:

```text
trace.zip
test-results/
```

Use `.gitignore`.

---

## Mistake 3: Sharing traces containing sensitive information

Check traces before sharing them.

They may contain application data and credentials entered during tests.

---

## Mistake 4: Using tracing as a replacement for proper assertions

Tracing is a debugging tool.

You still need meaningful assertions:

```typescript
await expect(page.locator('#dashboard'))
    .toBeVisible();
```

---

# 33. Recommended CI Configuration

For a senior automation framework:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({

    retries: process.env.CI ? 2 : 0,

    use: {
        baseURL: 'https://example.com',

        trace: 'on-first-retry',

        screenshot: 'only-on-failure',

        video: 'retain-on-failure'
    },

    reporter: [
        ['html'],
        ['list']
    ]

});
```

This provides:

```text
Failure
   |
   +-- Screenshot
   |
   +-- Video
   |
   +-- Trace
   |
   +-- HTML Report
```

---

# 34. Interview Questions

## Q1. What is Playwright Tracing?

Playwright Tracing records detailed test execution information such as actions, screenshots, DOM snapshots, network activity, and timing for debugging.

---

## Q2. What is Trace Viewer?

Trace Viewer is Playwright's interactive UI for inspecting trace files.

Command:

```bash
npx playwright show-trace trace.zip
```

---

## Q3. How do you enable tracing?

Using Playwright Test:

```typescript
use: {
    trace: 'on'
}
```

Or manually using the BrowserContext tracing API.

---

## Q4. What is `on-first-retry`?

It records a trace when a test is retried for the first time.

```typescript
use: {
    trace: 'on-first-retry'
}
```

It is commonly used in CI environments.

---

## Q5. Why is `on-first-retry` useful?

It avoids generating traces for every successful test while still providing detailed debugging information for failures.

---

## Q6. How do you open a trace?

```bash
npx playwright show-trace trace.zip
```

---

## Q7. What information can a trace contain?

A trace can contain:

* Screenshots
* DOM snapshots
* Actions
* Network requests
* Console information
* Source code
* Timing
* Browser state

---

## Q8. How is tracing different from screenshots?

A screenshot captures a single visual state, while a trace provides an interactive record of the test execution.

---

## Q9. How is tracing different from video?

Video shows visual execution, whereas tracing provides deeper debugging information such as DOM snapshots, actions, network information, and timing.

---

## Q10. Should traces be committed to Git?

Generally, no.

Trace files are generated artifacts and may contain sensitive application data.

---

# 35. Senior-Level Interview Scenario

### Question

A Playwright test passes locally but fails intermittently in Jenkins. How would you debug it?

### Answer

I would first enable tracing for retries:

```typescript
use: {
    trace: 'on-first-retry'
}
```

Then configure retries:

```typescript
retries: 2
```

When the test fails and is retried, Playwright generates a trace.

I would open it using:

```bash
npx playwright show-trace trace.zip
```

Then inspect:

1. Failed action
2. Locator
3. DOM snapshot
4. Screenshot
5. Network requests
6. Console messages
7. Navigation
8. Timing
9. Application state

This helps determine whether the root cause is a test synchronization problem, locator issue, network problem, application defect, or genuine flaky behavior.

---

# 36. Best Practices

Use the following practices:

1. Use `on-first-retry` in CI.
2. Avoid tracing every successful test unnecessarily.
3. Use screenshots for failed tests.
4. Use video selectively.
5. Preserve traces as CI artifacts.
6. Do not commit trace files to Git.
7. Be careful with credentials and sensitive data.
8. Use Trace Viewer to investigate flaky tests.
9. Combine tracing with meaningful assertions.
10. Use traces to identify root cause rather than simply retrying failures.

---

# 37. Quick Reference

### Enable tracing

```typescript
use: {
    trace: 'on'
}
```

### Trace on retry

```typescript
use: {
    trace: 'on-first-retry'
}
```

### Trace after failure

```typescript
use: {
    trace: 'retain-on-failure'
}
```

### Screenshot on failure

```typescript
use: {
    screenshot: 'only-on-failure'
}
```

### Video on failure

```typescript
use: {
    video: 'retain-on-failure'
}
```

### Open trace

```bash
npx playwright show-trace trace.zip
```

### Start manual tracing

```java
context.tracing().start(
        new Tracing.StartOptions()
                .setScreenshots(true)
                .setSnapshots(true)
                .setSources(true)
);
```

### Stop manual tracing

```java
context.tracing().stop(
        new Tracing.StopOptions()
                .setPath(Paths.get("trace.zip"))
);
```

---

# 38. Key Takeaway

Playwright Tracing is one of the most valuable debugging features in a professional Playwright automation framework.

A strong CI configuration is:

```typescript
use: {
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure'
}
```

When a test fails:

```text
Test Failure
     |
     v
Retry
     |
     v
Trace
     |
     +---- Screenshot
     |
     +---- DOM Snapshot
     |
     +---- Network
     |
     +---- Console
     |
     +---- Timing
     |
     v
Trace Viewer
     |
     v
Root Cause
```

**Tracing + Screenshot + Video + HTML Report** provides a strong debugging and reporting strategy for enterprise Playwright automation.
