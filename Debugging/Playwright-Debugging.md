# Playwright Debugging

## 1. Introduction

Debugging is an important part of Playwright test automation. Playwright provides several built-in tools that make it easier to understand why a test is failing.

Common debugging tools include:

* Playwright Inspector
* `page.pause()`
* Headed browser execution
* VS Code debugger
* Trace Viewer
* Screenshots
* Videos
* Console logs
* Network logging
* Locator inspection
* Verbose Playwright logs

---

# 2. Debug Mode

Playwright provides a built-in debug mode using the `PWDEBUG` environment variable.

### Windows PowerShell

```powershell
$env:PWDEBUG="1"
npx playwright test
```

### Windows CMD

```cmd
set PWDEBUG=1
npx playwright test
```

### macOS/Linux

```bash
PWDEBUG=1 npx playwright test
```

Debug mode:

* Opens the browser in headed mode
* Launches Playwright Inspector
* Disables normal timeout behavior for easier debugging
* Allows stepping through actions
* Shows locator information

---

# 3. Playwright Inspector

Playwright Inspector allows you to debug tests interactively.

Run:

```bash
npx playwright test --debug
```

This opens:

1. Browser
2. Playwright Inspector

The Inspector provides controls such as:

* Resume
* Step over
* Pick locator
* Locator preview
* Action details

---

# 4. Using `page.pause()`

`page.pause()` pauses test execution at a specific point.

Example:

```java
page.navigate("https://example.com");

page.pause();

page.locator("#username").fill("admin");
page.locator("#password").fill("password");
```

When execution reaches `page.pause()`, Playwright Inspector opens.

This is useful when you want to inspect the page before continuing.

---

# 5. JavaScript Example

```javascript
const { test, expect } = require('@playwright/test');

test('Debug Login Test', async ({ page }) => {

    await page.goto('https://example.com');

    await page.pause();

    await page.locator('#username').fill('admin');
    await page.locator('#password').fill('password');

});
```

Run:

```bash
npx playwright test --debug
```

---

# 6. TypeScript Example

```typescript
import { test, expect } from '@playwright/test';

test('Debug Login Test', async ({ page }) => {

    await page.goto('https://example.com');

    await page.pause();

    await page.locator('#username').fill('admin');
    await page.locator('#password').fill('password');

});
```

---

# 7. Java Example

Playwright also supports debugging in Java through the IDE debugger.

Example:

```java
@Test
public void debugLoginTest() {

    page.navigate("https://example.com");

    page.pause();

    page.locator("#username").fill("admin");
    page.locator("#password").fill("password");
}
```

You can place an IDE breakpoint before or after an action and run the test in Debug mode.

---

# 8. Headed Mode

By default, Playwright tests normally run in headless mode.

To see the browser:

```bash
npx playwright test --headed
```

Example:

```bash
npx playwright test --headed
```

This is useful when you want to visually observe:

* Navigation
* Clicking
* Form filling
* Popups
* Dialogs
* Page transitions

---

# 9. Slow Motion

You can slow down browser operations using `slowMo`.

Example:

```javascript
const { chromium } = require('playwright');

(async () => {

    const browser = await chromium.launch({
        headless: false,
        slowMo: 500
    });

    const page = await browser.newPage();

    await page.goto('https://example.com');

    await page.locator('#username').fill('admin');

    await page.locator('#password').fill('password');

    await browser.close();
})();
```

`slowMo: 500` adds approximately 500 milliseconds between operations.

This can make visual debugging easier.

---

# 10. VS Code Debugging

Playwright works very well with Visual Studio Code.

Typical workflow:

1. Open the Playwright project.
2. Open the test file.
3. Set a breakpoint.
4. Start the test in Debug mode.
5. Execution stops at the breakpoint.
6. Inspect variables and application state.
7. Step through the test.

Example:

```javascript
test('Login Test', async ({ page }) => {

    await page.goto('https://example.com');

    const username = page.locator('#username');

    await username.fill('admin');

    // Set breakpoint here

    await page.locator('#password').fill('password');

});
```

---

# 11. Using `test.only()`

When debugging one test, you can use `test.only()`.

```javascript
test.only('Login Test', async ({ page }) => {

    await page.goto('https://example.com');

});
```

Only this test will execute.

### Important

Do not commit `test.only()` to your repository.

Remove it after debugging.

---

# 12. Using `describe.only()`

You can also debug an entire test suite.

```javascript
test.describe.only('Login Tests', () => {

    test('Valid Login', async ({ page }) => {
        await page.goto('https://example.com');
    });

    test('Invalid Login', async ({ page }) => {
        await page.goto('https://example.com');
    });

});
```

Only the tests inside this `describe` block will run.

---

# 13. Locator Debugging

One of the most common Playwright failures is an incorrect locator.

Example:

```javascript
await page.locator('#userName').fill('admin');
```

If the actual element is:

```html
<input id="username">
```

the test fails.

Use Playwright's locator tools to verify the locator.

A better approach is often:

```javascript
page.getByRole('textbox', { name: 'Username' });
```

or:

```javascript
page.getByLabel('Username');
```

---

# 14. Checking Locator Count

You can determine how many elements match a locator.

```javascript
const count = await page.locator('.login-button').count();

console.log(`Matching elements: ${count}`);
```

If the expected count is one:

```javascript
expect(await page.locator('.login-button').count()).toBe(1);
```

This can help identify:

* Duplicate elements
* Wrong selectors
* Hidden elements
* Unexpected DOM changes

---

# 15. Inspecting Element Text

You can print text during debugging.

```javascript
const text = await page.locator('.message').textContent();

console.log(`Message: ${text}`);
```

Or:

```javascript
console.log(await page.locator('.message').innerText());
```

---

# 16. Inspecting Input Values

For an input field:

```javascript
const value = await page.locator('#username').inputValue();

console.log(`Username: ${value}`);
```

This helps verify whether the expected value was entered.

---

# 17. Checking Element Visibility

```javascript
const visible = await page.locator('#submit').isVisible();

console.log(`Submit visible: ${visible}`);
```

You can also assert:

```javascript
await expect(page.locator('#submit')).toBeVisible();
```

---

# 18. Checking Element State

### Enabled

```javascript
console.log(
    await page.locator('#submit').isEnabled()
);
```

### Checked

```javascript
console.log(
    await page.locator('#rememberMe').isChecked()
);
```

### Editable

```javascript
console.log(
    await page.locator('#username').isEditable()
);
```

---

# 19. Console Logging

Use `console.log()` to understand test execution.

```javascript
console.log('Opening login page');

await page.goto('https://example.com');

console.log('Entering username');

await page.locator('#username').fill('admin');

console.log('Entering password');

await page.locator('#password').fill('password');

console.log('Clicking login');

await page.locator('#login').click();
```

This helps identify the last successful operation before a failure.

---

# 20. Browser Console Logs

You can listen for browser console messages.

```javascript
page.on('console', message => {
    console.log(`Browser console: ${message.text()}`);
});
```

This is useful for identifying:

* JavaScript errors
* Warnings
* Application logs
* Unexpected browser messages

---

# 21. Browser Errors

Listen for page errors:

```javascript
page.on('pageerror', error => {
    console.log(`Page error: ${error.message}`);
});
```

This can help identify JavaScript errors in the application.

---

# 22. Network Debugging

Playwright can monitor network requests.

```javascript
page.on('request', request => {
    console.log(`REQUEST: ${request.method()} ${request.url()}`);
});
```

Monitor responses:

```javascript
page.on('response', response => {
    console.log(
        `RESPONSE: ${response.status()} ${response.url()}`
    );
});
```

---

# 23. Debugging API Failures

You can identify failed HTTP responses.

```javascript
page.on('response', response => {

    if (response.status() >= 400) {
        console.log(
            `HTTP ERROR: ${response.status()} ${response.url()}`
        );
    }

});
```

This is useful for identifying:

* 400 Bad Request
* 401 Unauthorized
* 403 Forbidden
* 404 Not Found
* 500 Internal Server Error
* Other API failures

---

# 24. Debugging Requests

You can inspect request details.

```javascript
page.on('request', request => {

    console.log(`URL: ${request.url()}`);
    console.log(`Method: ${request.method()}`);

});
```

For POST requests:

```javascript
page.on('request', request => {

    if (request.method() === 'POST') {
        console.log(request.postData());
    }

});
```

---

# 25. Screenshots for Debugging

Take a screenshot when needed.

```javascript
await page.screenshot({
    path: 'debug.png',
    fullPage: true
});
```

You can also take a screenshot of a specific element:

```javascript
await page.locator('.login-form').screenshot({
    path: 'login-form.png'
});
```

---

# 26. Screenshot on Failure

Configure screenshots in `playwright.config.ts`.

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({

    use: {
        screenshot: 'only-on-failure'
    }

});
```

Options include:

```text
'off'
'on'
'only-on-failure'
```

Recommended:

```typescript
screenshot: 'only-on-failure'
```

---

# 27. Video Recording

Playwright can record videos.

```typescript
export default defineConfig({

    use: {
        video: 'retain-on-failure'
    }

});
```

Common options:

```text
'off'
'on'
'retain-on-failure'
'on-first-retry'
```

For debugging failures:

```typescript
video: 'retain-on-failure'
```

is useful.

---

# 28. Trace Viewer

Trace Viewer is one of the most powerful Playwright debugging tools.

It can capture:

* Actions
* Screenshots
* DOM snapshots
* Network requests
* Console messages
* Source information
* Timing information

---

# 29. Enable Trace

In `playwright.config.ts`:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({

    use: {
        trace: 'retain-on-failure'
    }

});
```

This records a trace when a test fails.

---

# 30. Trace Options

Common trace settings:

```typescript
trace: 'off'
```

No trace.

```typescript
trace: 'on'
```

Record every test.

```typescript
trace: 'retain-on-failure'
```

Keep traces for failed tests.

```typescript
trace: 'on-first-retry'
```

Record the first retry.

For CI/CD, a good option is:

```typescript
trace: 'on-first-retry'
```

---

# 31. Open Trace Viewer

After a test failure, you may have a `.zip` trace file.

Open it with:

```bash
npx playwright show-trace path/to/trace.zip
```

Example:

```bash
npx playwright show-trace test-results/login-test/trace.zip
```

---

# 32. Trace Viewer Timeline

The Trace Viewer provides a timeline of test execution.

You can inspect:

```text
Test Start
    ↓
Navigate
    ↓
Fill Username
    ↓
Fill Password
    ↓
Click Login
    ↓
Wait for Response
    ↓
Assertion
    ↓
Failure
```

Selecting an action allows you to investigate what happened at that point.

---

# 33. Trace Viewer and DOM Snapshot

Trace Viewer can show a snapshot of the page at different points during execution.

This is extremely useful when an element:

* Was present earlier
* Disappeared later
* Changed text
* Changed attributes
* Became hidden
* Was replaced by another element

---

# 34. Debugging Timeouts

Example failure:

```text
Timeout 30000ms exceeded.
```

Do not immediately increase the timeout.

First investigate:

1. Is the locator correct?
2. Is the element visible?
3. Is the page loaded?
4. Is an API request failing?
5. Is another element blocking the action?
6. Is the application slow?
7. Is there an unexpected popup?

Use:

```javascript
await page.pause();
```

and inspect the page.

---

# 35. Avoid Arbitrary Waits

Avoid:

```javascript
await page.waitForTimeout(5000);
```

This is usually not a good solution for synchronization problems.

Prefer:

```javascript
await expect(page.locator('#message')).toBeVisible();
```

or:

```javascript
await page.locator('#submit').click();
await page.waitForURL('**/dashboard');
```

Playwright automatically waits for many conditions.

---

# 36. Debugging Flaky Tests

A flaky test passes sometimes and fails sometimes.

Common causes:

* Incorrect waits
* Unstable locators
* Race conditions
* Network delays
* Application timing issues
* Shared test data
* Test dependency
* Parallel execution problems
* Environment instability

---

# 37. Debugging Flaky Tests with Retries

You can configure retries.

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({

    retries: 2,

});
```

However, retries should not be used to hide real problems.

A test that fails intermittently should be investigated.

---

# 38. Debugging with Retry Trace

A useful configuration:

```typescript
export default defineConfig({

    retries: 2,

    use: {
        trace: 'on-first-retry',
        screenshot: 'only-on-failure',
        video: 'retain-on-failure'
    }

});
```

This gives useful debugging evidence without recording everything.

---

# 39. Debugging Authentication Problems

When authentication fails, check:

* Login URL
* Username
* Password
* Authentication API
* Cookies
* Local storage
* Session storage
* Redirect URL
* Authentication state
* Token expiration

You can inspect cookies:

```javascript
const cookies = await page.context().cookies();

console.log(cookies);
```

---

# 40. Debugging Local Storage

```javascript
const value = await page.evaluate(() => {
    return localStorage.getItem('token');
});

console.log(value);
```

You can inspect all local storage entries:

```javascript
const storage = await page.evaluate(() => {
    return Object.entries(localStorage);
});

console.log(storage);
```

---

# 41. Debugging Page URL

```javascript
console.log(await page.url());
```

You can assert:

```javascript
await expect(page).toHaveURL(/dashboard/);
```

This is useful when redirects are unexpected.

---

# 42. Debugging Page Title

```javascript
console.log(await page.title());
```

Assertion:

```javascript
await expect(page).toHaveTitle(/Dashboard/);
```

---

# 43. Debugging Popups

If a popup is expected:

```javascript
const popupPromise = page.waitForEvent('popup');

await page.locator('#open-window').click();

const popup = await popupPromise;

console.log(await popup.url());
```

If a popup is causing unexpected behavior, inspect the page events and trace.

---

# 44. Debugging Dialogs

Monitor dialogs:

```javascript
page.on('dialog', async dialog => {

    console.log(`Dialog type: ${dialog.type()}`);
    console.log(`Dialog message: ${dialog.message()}`);

    await dialog.dismiss();

});
```

---

# 45. Debugging Downloads

```javascript
const downloadPromise = page.waitForEvent('download');

await page.locator('#download').click();

const download = await downloadPromise;

console.log(download.suggestedFilename());
```

---

# 46. Debugging File Uploads

```javascript
const fileInput = page.locator('input[type="file"]');

await fileInput.setInputFiles('test-data/sample.pdf');
```

If upload fails, verify:

* File exists
* Correct file path
* Input element is correct
* File type is supported
* Application accepts the file

---

# 47. Debugging Frames

If an element is inside an iframe, the locator must use the frame.

Example:

```javascript
const frame = page.frameLocator('#payment-frame');

await frame.locator('#cardNumber').fill('4111111111111111');
```

If the locator cannot find the element, verify that the element is actually inside the iframe.

---

# 48. Debugging Shadow DOM

Playwright locators can work with open Shadow DOM.

Example:

```javascript
await page.locator('my-component')
    .locator('button')
    .click();
```

If an element is not found, verify:

* Shadow root is open
* Correct host element
* Correct locator
* Component has finished rendering

---

# 49. Verbose Playwright Logging

You can enable verbose API logging.

### Windows PowerShell

```powershell
$env:DEBUG="pw:api"
npx playwright test
```

### Windows CMD

```cmd
set DEBUG=pw:api
npx playwright test
```

### macOS/Linux

```bash
DEBUG=pw:api npx playwright test
```

This can provide detailed information about Playwright actions.

---

# 50. Debugging with Environment Variables

You can configure debugging behavior through environment variables.

Example:

```powershell
$env:PWDEBUG="1"
npx playwright test
```

For API logs:

```powershell
$env:DEBUG="pw:api"
npx playwright test
```

Remember to remove temporary debugging environment variables when they are no longer needed.

---

# 51. Complete Debug Configuration

A practical configuration:

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({

    testDir: './tests',

    timeout: 30 * 1000,

    retries: 2,

    use: {
        baseURL: 'https://example.com',

        trace: 'on-first-retry',

        screenshot: 'only-on-failure',

        video: 'retain-on-failure',

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

---

# 52. Recommended Debugging Workflow

When a Playwright test fails:

```text
Test Failure
     ↓
Read Error Message
     ↓
Check Locator
     ↓
Run Test in Headed Mode
     ↓
Use page.pause()
     ↓
Inspect Page
     ↓
Check Console Errors
     ↓
Check Network Requests
     ↓
Check Screenshot
     ↓
Open Trace Viewer
     ↓
Identify Root Cause
     ↓
Fix Test/Application Issue
     ↓
Run Test Again
```

---

# 53. Debugging Checklist

When debugging a failed Playwright test, check:

* Is the URL correct?
* Is the locator correct?
* Is the element visible?
* Is the element enabled?
* Is the element inside an iframe?
* Is the element inside Shadow DOM?
* Is there a popup?
* Is there a browser dialog?
* Did authentication succeed?
* Did the API return an error?
* Did the page redirect?
* Is the test data correct?
* Is the environment available?
* Is the test dependent on another test?
* Is there a timing issue?
* Is the test flaky?
* Is the browser version correct?
* Is the trace available?
* Is the screenshot useful?
* Are browser console errors present?

---

# 54. Common Debugging Commands

### Run all tests

```bash
npx playwright test
```

### Run a specific test

```bash
npx playwright test tests/login.spec.ts
```

### Run headed

```bash
npx playwright test --headed
```

### Run debug mode

```bash
npx playwright test --debug
```

### Run a specific test in debug mode

```bash
npx playwright test tests/login.spec.ts --debug
```

### Open HTML report

```bash
npx playwright show-report
```

### Open trace

```bash
npx playwright show-trace trace.zip
```

---

# 55. Example: Complete Debugging Test

```javascript
const { test, expect } = require('@playwright/test');

test('Complete Debugging Example', async ({ page }) => {

    page.on('console', message => {
        console.log(`BROWSER: ${message.text()}`);
    });

    page.on('pageerror', error => {
        console.log(`PAGE ERROR: ${error.message}`);
    });

    page.on('response', response => {

        if (response.status() >= 400) {
            console.log(
                `HTTP ERROR: ${response.status()} ${response.url()}`
            );
        }

    });

    await page.goto('https://example.com');

    console.log(`Current URL: ${await page.url()}`);

    console.log(`Page title: ${await page.title()}`);

    await page.pause();

    const username = page.locator('#username');

    console.log(
        `Username locator count: ${await username.count()}`
    );

    await username.fill('admin');

    console.log(
        `Username value: ${await username.inputValue()}`
    );

    await page.screenshot({
        path: 'debug-login.png',
        fullPage: true
    });

    await page.locator('#login').click();

    await expect(page).toHaveURL(/dashboard/);

});
```

---

# 56. Best Practices

## 1. Use `page.pause()` during local debugging

```javascript
await page.pause();
```

---

## 2. Use Trace Viewer for difficult failures

```typescript
trace: 'on-first-retry'
```

---

## 3. Capture screenshots on failures

```typescript
screenshot: 'only-on-failure'
```

---

## 4. Capture videos when failures need visual investigation

```typescript
video: 'retain-on-failure'
```

---

## 5. Use stable locators

Prefer:

```javascript
page.getByRole('button', { name: 'Login' });
```

over fragile CSS selectors such as:

```javascript
page.locator('div:nth-child(4) > button');
```

---

## 6. Avoid unnecessary `waitForTimeout()`

Do not solve synchronization problems with arbitrary sleeps.

---

## 7. Investigate the root cause

Do not simply increase the timeout:

```javascript
timeout: 120000
```

unless the application genuinely needs that amount of time.

---

## 8. Keep debugging code out of production tests

Remove temporary code such as:

```javascript
await page.pause();
```

and:

```javascript
test.only(...)
```

before committing.

---

# 57. Playwright Debugging Tools Summary

| Tool             | Purpose                      |
| ---------------- | ---------------------------- |
| `page.pause()`   | Pause execution              |
| `--debug`        | Start Inspector              |
| `--headed`       | Show browser                 |
| `slowMo`         | Slow browser actions         |
| VS Code Debugger | Step through code            |
| Trace Viewer     | Detailed test investigation  |
| Screenshot       | Capture page state           |
| Video            | Replay test execution        |
| Console events   | Inspect browser logs         |
| `pageerror`      | Detect JavaScript errors     |
| Request events   | Inspect outgoing requests    |
| Response events  | Inspect HTTP responses       |
| `PWDEBUG=1`      | Enable Playwright debug mode |
| `DEBUG=pw:api`   | Detailed Playwright API logs |

---

# 58. Interview Questions

### Q1. How do you debug a Playwright test?

Use:

```bash
npx playwright test --debug
```

or:

```javascript
await page.pause();
```

You can also use screenshots, videos, trace viewer, console logs, and IDE debugging.

---

### Q2. What is Playwright Inspector?

Playwright Inspector is an interactive debugging tool that allows you to pause, resume, step through actions, inspect locators, and examine the page.

---

### Q3. What is `page.pause()`?

`page.pause()` pauses execution and allows the tester to inspect the current browser state through Playwright Inspector.

---

### Q4. How do you debug a locator?

Verify the locator using Playwright Inspector and check:

```javascript
await page.locator('selector').count();
```

You can also use role-, label-, text-, or test-id-based locators.

---

### Q5. How do you debug API failures?

Monitor responses:

```javascript
page.on('response', response => {
    console.log(response.status(), response.url());
});
```

---

### Q6. How do you debug flaky tests?

Use:

* Trace Viewer
* Screenshots
* Videos
* Retries
* Console logs
* Network monitoring
* Stable locators
* Proper synchronization

Then identify the actual root cause.

---

### Q7. What is Trace Viewer?

Trace Viewer is a Playwright debugging tool that provides a detailed record of test execution, including actions, screenshots, DOM snapshots, network activity, and timing.

---

### Q8. How do you open a trace?

```bash
npx playwright show-trace trace.zip
```

---

### Q9. How do you capture screenshots only when tests fail?

```typescript
use: {
    screenshot: 'only-on-failure'
}
```

---

### Q10. How do you record videos only for failed tests?

```typescript
use: {
    video: 'retain-on-failure'
}
```

---

# 59. Senior-Level Debugging Strategy

For a senior automation engineer, debugging should follow a structured approach.

```text
Failure
   ↓
Understand Error
   ↓
Reproduce Locally
   ↓
Check Locator
   ↓
Check Application State
   ↓
Check API/Network
   ↓
Check Authentication
   ↓
Check Test Data
   ↓
Check Timing/Synchronization
   ↓
Review Trace
   ↓
Determine Root Cause
   ↓
Fix
   ↓
Run Regression
```

The goal is not simply to make the test pass.

The goal is to determine **why the test failed** and whether the problem is in:

* Automation code
* Application code
* Test data
* Environment
* Network
* Authentication
* Timing
* Configuration

---

# 60. Final Summary

Playwright provides powerful debugging capabilities without requiring many third-party tools.

The most important tools to know are:

```text
npx playwright test --debug
```

```javascript
await page.pause();
```

```bash
npx playwright test --headed
```

```bash
npx playwright show-trace trace.zip
```

And the most useful failure configuration is:

```typescript
use: {
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure'
}
```

For day-to-day Playwright automation, the recommended debugging combination is:

**Playwright Inspector + VS Code Debugger + Trace Viewer + Screenshots + Console/Network logs.**

---

## Suggested GitHub Location

```text
PlaywrightStudy/
└── Debugging/
    └── Playwright-Debugging.md
```
