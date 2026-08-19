# Playwright Basics

## 1. What is Playwright?

Playwright is an open-source end-to-end testing and browser automation framework developed by Microsoft.

It supports modern web applications and provides automation APIs for:

* Chromium
* Google Chrome
* Microsoft Edge
* Firefox
* WebKit

Playwright can be used for:

* UI automation
* End-to-end testing
* API testing
* Cross-browser testing
* Mobile browser testing
* Network interception
* Authentication testing
* Screenshot and video capture
* Parallel test execution
* Test reporting

Playwright is commonly used with:

* JavaScript
* TypeScript
* Java
* Python
* .NET

For modern automation projects, **Playwright with TypeScript** is one of the most commonly used combinations.

---

# 2. Playwright vs Selenium

| Feature              | Selenium                                 | Playwright                                 |
| -------------------- | ---------------------------------------- | ------------------------------------------ |
| Developer            | Selenium project                         | Microsoft                                  |
| Main languages       | Java, Python, C#, JavaScript, etc.       | TypeScript, JavaScript, Java, Python, .NET |
| Browser automation   | Yes                                      | Yes                                        |
| Chromium             | Yes                                      | Yes                                        |
| Firefox              | Yes                                      | Yes                                        |
| WebKit               | No                                       | Yes                                        |
| Auto-waiting         | Limited/manual                           | Built-in                                   |
| Browser contexts     | No direct equivalent                     | Yes                                        |
| Network interception | Yes                                      | Powerful built-in support                  |
| API testing          | Usually separate tools                   | Built-in                                   |
| Parallel execution   | Test framework dependent                 | Built-in                                   |
| Trace viewer         | No native equivalent                     | Yes                                        |
| Codegen              | Available through Selenium IDE/ecosystem | Built-in                                   |
| Screenshots          | Yes                                      | Yes                                        |
| Video recording      | Framework/grid dependent                 | Built-in                                   |
| Multiple tabs        | Yes                                      | Yes                                        |
| Multiple browsers    | Yes                                      | Yes                                        |

### Important interview point

Selenium communicates with browsers primarily through the WebDriver protocol.

Playwright communicates with supported browsers using its own automation architecture and provides tighter integration with browser contexts, pages, network interception, auto-waiting, tracing, and test execution.

---

# 3. Why Playwright is Popular

Modern web applications frequently use:

* React
* Angular
* Vue
* Next.js
* AJAX
* REST APIs
* WebSockets
* Single-page applications

Playwright provides features that make these applications easier to automate.

Important features include:

1. Auto-waiting
2. Web-first assertions
3. Browser contexts
4. Multiple browser support
5. Network interception
6. API testing
7. Trace Viewer
8. Screenshots
9. Video recording
10. Parallel execution
11. Test fixtures
12. Authentication state
13. Built-in test runner

---

# 4. Playwright Architecture

A simplified architecture is:

```text
Test Script
    |
    v
Playwright API
    |
    +----------------+
    |                |
    v                v
Browser         APIRequest
    |
    +-----------------------------+
    |              |              |
    v              v              v
Chromium        Firefox         WebKit
    |
    v
Browser Context
    |
    v
Page
    |
    v
Web Application
```

A typical hierarchy is:

```text
Playwright
    |
    └── Browser
          |
          └── BrowserContext
                |
                └── Page
                      |
                      ├── Locator
                      └── Web Application
```

---

# 5. Important Playwright Objects

The most important objects are:

```text
Playwright
Browser
BrowserContext
Page
Locator
```

## Playwright

The top-level Playwright object used to access browser types and other Playwright functionality.

Example:

```typescript
import { chromium } from '@playwright/test';

const browser = await chromium.launch();
```

---

# 6. Browser

A Browser represents a running browser instance.

Example:

```typescript
const browser = await chromium.launch();
```

You can launch:

```typescript
chromium.launch();
firefox.launch();
webkit.launch();
```

Example:

```typescript
import { chromium } from '@playwright/test';

const browser = await chromium.launch({
    headless: false
});
```

---

# 7. BrowserContext

A BrowserContext is an isolated browser session.

It is similar to a fresh browser profile.

Example:

```typescript
const context = await browser.newContext();
```

Each context can have its own:

* Cookies
* Local storage
* Session storage
* Authentication state
* Permissions

Example:

```typescript
const context1 = await browser.newContext();
const context2 = await browser.newContext();
```

The two contexts are isolated from each other.

This is extremely useful for parallel testing and testing multiple users.

---

# 8. Page

A Page represents a browser tab.

Example:

```typescript
const page = await context.newPage();
```

Navigate to a URL:

```typescript
await page.goto('https://example.com');
```

A context can contain multiple pages:

```typescript
const page1 = await context.newPage();
const page2 = await context.newPage();
```

---

# 9. Locator

A Locator identifies elements on a web page.

Example:

```typescript
const username = page.locator('#username');
```

Other examples:

```typescript
page.getByRole('button', { name: 'Login' });

page.getByText('Welcome');

page.getByLabel('Username');

page.getByPlaceholder('Enter username');

page.locator('.username');
```

Locators are one of the most important Playwright concepts.

---

# 10. Basic Playwright Test

A simple Playwright test:

```typescript
import { test, expect } from '@playwright/test';

test('verify title', async ({ page }) => {

    await page.goto('https://example.com');

    await expect(page).toHaveTitle(/Example/);

});
```

Explanation:

```text
test()
   |
   └── Test definition

page
   |
   └── Browser tab

page.goto()
   |
   └── Navigate to URL

expect()
   |
   └── Validate application behavior
```

---

# 11. Basic Navigation

Navigate to a URL:

```typescript
await page.goto('https://example.com');
```

Go back:

```typescript
await page.goBack();
```

Go forward:

```typescript
await page.goForward();
```

Reload:

```typescript
await page.reload();
```

---

# 12. Basic Element Interaction

## Click

```typescript
await page.getByRole('button', { name: 'Login' }).click();
```

## Fill

```typescript
await page.getByLabel('Username').fill('selva');
```

## Password

```typescript
await page.getByLabel('Password').fill('Password123');
```

## Check checkbox

```typescript
await page.getByLabel('Remember me').check();
```

## Uncheck checkbox

```typescript
await page.getByLabel('Remember me').uncheck();
```

---

# 13. Assertions

Playwright provides web-first assertions.

Example:

```typescript
await expect(page).toHaveTitle('Login Page');
```

Check visibility:

```typescript
await expect(page.getByRole('button', { name: 'Login' }))
    .toBeVisible();
```

Check text:

```typescript
await expect(page.locator('.message'))
    .toHaveText('Login successful');
```

Check URL:

```typescript
await expect(page).toHaveURL(/dashboard/);
```

Check value:

```typescript
await expect(page.getByLabel('Username'))
    .toHaveValue('selva');
```

---

# 14. Auto-Waiting

One of the biggest advantages of Playwright is automatic waiting.

For example:

```typescript
await page.getByRole('button', { name: 'Login' }).click();
```

Playwright automatically waits for the element to be ready for the action.

It can wait for conditions such as:

* Element exists
* Element is visible
* Element is stable
* Element can receive events
* Element is enabled

Therefore, unnecessary code such as:

```typescript
await page.waitForTimeout(5000);
```

should generally be avoided.

---

# 15. Why Avoid waitForTimeout?

This is generally discouraged:

```typescript
await page.waitForTimeout(5000);
```

It introduces a fixed delay.

For example, if the application becomes ready after one second, the test still waits five seconds.

Instead, use an appropriate Playwright action or assertion:

```typescript
await expect(page.getByText('Dashboard')).toBeVisible();
```

This allows Playwright to wait until the expected condition is satisfied.

---

# 16. Headless and Headed Mode

By default, Playwright tests normally run in headless mode.

Headless:

```typescript
const browser = await chromium.launch({
    headless: true
});
```

Headed:

```typescript
const browser = await chromium.launch({
    headless: false
});
```

Headed mode is useful during debugging because you can see the browser.

---

# 17. Browser Types

Playwright supports:

### Chromium

```typescript
import { chromium } from '@playwright/test';
```

### Firefox

```typescript
import { firefox } from '@playwright/test';
```

### WebKit

```typescript
import { webkit } from '@playwright/test';
```

Example:

```typescript
const browser = await firefox.launch();
```

---

# 18. Complete Basic Example

```typescript
import { test, expect } from '@playwright/test';

test('Login test', async ({ page }) => {

    await page.goto('https://example.com/login');

    await page.getByLabel('Username').fill('selva');

    await page.getByLabel('Password').fill('Password123');

    await page.getByRole('button', { name: 'Login' }).click();

    await expect(page).toHaveURL(/dashboard/);

    await expect(page.getByText('Welcome')).toBeVisible();

});
```

This demonstrates:

* Test creation
* Page fixture
* Navigation
* Locators
* Text input
* Button click
* URL assertion
* Visibility assertion

---

# 19. Using Browser and Context Manually

Although the Playwright Test framework provides the `page` fixture automatically, understanding the underlying objects is important.

```typescript
import { chromium } from '@playwright/test';

(async () => {

    const browser = await chromium.launch({
        headless: false
    });

    const context = await browser.newContext();

    const page = await context.newPage();

    await page.goto('https://example.com');

    console.log(await page.title());

    await browser.close();

})();
```

The flow is:

```text
Launch Browser
      ↓
Create Context
      ↓
Create Page
      ↓
Navigate
      ↓
Interact
      ↓
Validate
      ↓
Close Browser
```

---

# 20. Multiple Browser Contexts

One browser can contain multiple isolated contexts.

```typescript
const browser = await chromium.launch();

const user1 = await browser.newContext();
const user2 = await browser.newContext();

const page1 = await user1.newPage();
const page2 = await user2.newPage();

await page1.goto('https://example.com');
await page2.goto('https://example.com');
```

This is useful for testing scenarios such as:

```text
User A
   |
   └── BrowserContext A

User B
   |
   └── BrowserContext B
```

Each user can have separate cookies and authentication.

---

# 21. Multiple Pages / Tabs

A BrowserContext can contain multiple pages.

```typescript
const page1 = await context.newPage();

const page2 = await context.newPage();
```

You can access existing pages:

```typescript
const pages = context.pages();

console.log(pages.length);
```

---

# 22. Playwright Test Fixtures

Playwright Test provides built-in fixtures.

The most commonly used fixture is:

```typescript
page
```

Example:

```typescript
test('Example test', async ({ page }) => {

    await page.goto('https://example.com');

});
```

Other important fixtures include:

```text
browser
context
page
request
```

Fixtures are covered in detail in the **Fixtures** topic.

---

# 23. Playwright Configuration

Playwright uses a configuration file:

```text
playwright.config.ts
```

Example:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({

    testDir: './tests',

    timeout: 30000,

    use: {
        baseURL: 'https://example.com',
        headless: true,
        screenshot: 'only-on-failure',
        video: 'retain-on-failure',
        trace: 'on-first-retry'
    },

    workers: 4

});
```

This allows common settings to be centralized.

---

# 24. Running Tests

Run all tests:

```bash
npx playwright test
```

Run a specific file:

```bash
npx playwright test tests/login.spec.ts
```

Run in headed mode:

```bash
npx playwright test --headed
```

Run a specific browser project:

```bash
npx playwright test --project=chromium
```

Run a specific test:

```bash
npx playwright test -g "Login test"
```

---

# 25. Debugging

Playwright provides several debugging capabilities.

## Headed mode

```bash
npx playwright test --headed
```

## Debug mode

```bash
npx playwright test --debug
```

## Inspector

Playwright Inspector allows you to:

* Pause execution
* Inspect locators
* Step through tests
* Debug actions

---

# 26. Codegen

Playwright provides code generation.

Run:

```bash
npx playwright codegen https://example.com
```

Codegen can help generate:

* Locators
* Click actions
* Fill actions
* Navigation commands

Example generated style:

```typescript
await page.getByRole('textbox', { name: 'Username' }).fill('selva');

await page.getByRole('button', { name: 'Login' }).click();
```

Codegen is useful for learning and quickly creating an initial test, but generated code should be reviewed and improved before becoming production automation.

---

# 27. Screenshot

Take a screenshot:

```typescript
await page.screenshot({
    path: 'screenshots/home.png'
});
```

Full-page screenshot:

```typescript
await page.screenshot({
    path: 'screenshots/home.png',
    fullPage: true
});
```

---

# 28. Video Recording

Playwright can record videos through browser context configuration.

```typescript
const context = await browser.newContext({
    recordVideo: {
        dir: 'videos/'
    }
});
```

This can be useful when analyzing failed tests.

---

# 29. Trace Viewer

Trace Viewer is one of Playwright's powerful debugging features.

A trace can capture:

* Actions
* Screenshots
* DOM snapshots
* Network activity
* Console information
* Timing

Configuration example:

```typescript
use: {
    trace: 'on-first-retry'
}
```

After a test failure, the trace can be opened with:

```bash
npx playwright show-trace trace.zip
```

---

# 30. Network Interception

Playwright allows you to intercept network requests.

Example:

```typescript
await page.route('**/api/users', async route => {

    await route.fulfill({
        status: 200,
        body: JSON.stringify({
            name: 'Selva'
        })
    });

});
```

This is useful for:

* Mocking APIs
* Testing error scenarios
* Controlling backend responses
* Testing frontend behavior independently

---

# 31. API Testing

Playwright can also perform API requests.

Example:

```typescript
import { test, expect } from '@playwright/test';

test('API test', async ({ request }) => {

    const response = await request.get('/api/users');

    expect(response.ok()).toBeTruthy();

});
```

This allows UI and API testing to be combined within the same Playwright project.

---

# 32. Authentication

Playwright supports saving authentication state.

A common approach is:

```text
Login
  ↓
Save authentication state
  ↓
Reuse state in tests
```

Example:

```typescript
await context.storageState({
    path: 'auth.json'
});
```

This can eliminate repeated login steps in every test.

---

# 33. Parallel Execution

Playwright Test supports parallel execution.

Example configuration:

```typescript
export default defineConfig({

    workers: 4

});
```

Tests can run across multiple workers.

This can significantly reduce overall execution time.

---

# 34. Recommended Locator Priority

For reliable Playwright tests, prefer user-facing locators.

Recommended order:

```text
1. getByRole()
2. getByLabel()
3. getByPlaceholder()
4. getByText()
5. getByTestId()
6. locator()
```

Example:

```typescript
await page.getByRole('button', { name: 'Login' }).click();
```

is generally preferable to:

```typescript
await page.locator('#loginButton').click();
```

when a meaningful role-based locator is available.

---

# 35. Playwright Best Practices

## Use Locators

Prefer:

```typescript
page.getByRole('button', { name: 'Submit' })
```

over fragile XPath expressions.

## Avoid Hard Waits

Avoid:

```typescript
await page.waitForTimeout(5000);
```

Prefer:

```typescript
await expect(page.getByText('Success')).toBeVisible();
```

## Use Web-First Assertions

Prefer:

```typescript
await expect(page.locator('.message')).toHaveText('Success');
```

## Use Page Objects for Large Frameworks

Keep test logic separate from page interaction logic.

```text
Tests
  ↓
Page Objects
  ↓
Playwright
  ↓
Application
```

## Keep Configuration Centralized

Use:

```text
playwright.config.ts
```

for common configuration.

---

# 36. Common Mistakes

### Mistake 1: Using fixed waits

```typescript
await page.waitForTimeout(5000);
```

### Mistake 2: Using fragile XPath everywhere

```typescript
page.locator('/html/body/div[2]/button')
```

### Mistake 3: Repeating login in every test

Use authentication state instead.

### Mistake 4: Creating a new browser for every action

Prefer appropriate browser/context/page lifecycle management.

### Mistake 5: Putting all logic into test files

For large projects, use:

* Page Objects
* Fixtures
* Utilities
* Test data
* Configuration

### Mistake 6: Ignoring trace information

Trace Viewer can provide valuable information for diagnosing failures.

---

# 37. Selenium to Playwright Mapping

| Selenium           | Playwright                                                      |
| ------------------ | --------------------------------------------------------------- |
| WebDriver          | Browser                                                         |
| Browser session    | BrowserContext                                                  |
| WebDriver tab      | Page                                                            |
| WebElement         | Locator                                                         |
| `driver.get()`     | `page.goto()`                                                   |
| `findElement()`    | `locator()` / `getByRole()`                                     |
| `click()`          | `click()`                                                       |
| `sendKeys()`       | `fill()` / `press()`                                            |
| `getTitle()`       | `page.title()`                                                  |
| `getCurrentUrl()`  | `page.url()`                                                    |
| Explicit Wait      | Auto-waiting + assertions                                       |
| ExpectedConditions | Web-first assertions                                            |
| Screenshot         | `page.screenshot()`                                             |
| Selenium Grid      | Playwright workers / browser projects / external infrastructure |
| TestNG             | Playwright Test                                                 |
| `@BeforeMethod`    | `beforeEach()`                                                  |
| `@AfterMethod`     | `afterEach()`                                                   |
| DataProvider       | Parameterized/custom test data patterns                         |
| PageFactory        | Page Object + Locator                                           |
| RemoteWebDriver    | Remote browser/connect APIs                                     |

---

# 38. Important Interview Questions

### Beginner

1. What is Playwright?
2. Who developed Playwright?
3. Which browsers does Playwright support?
4. Which programming languages are supported?
5. What is Playwright Test?
6. What is a Browser?
7. What is BrowserContext?
8. What is a Page?
9. What is a Locator?
10. How do you navigate to a URL?

### Intermediate

11. What is auto-waiting?
12. What are web-first assertions?
13. How do you handle multiple tabs?
14. How do you handle frames?
15. How do you handle dialogs?
16. How do you upload a file?
17. How do you take screenshots?
18. How do you record videos?
19. How do you intercept network requests?
20. How do you run tests in parallel?

### Advanced

21. What is BrowserContext and why is it important?
22. How does Playwright achieve test isolation?
23. How do you implement authentication state?
24. How do you mock API responses?
25. How do you perform API testing?
26. How do fixtures work?
27. How do you implement Page Object Model?
28. How do you configure multiple browsers?
29. How do you debug failed tests using Trace Viewer?
30. How would you design a Playwright automation framework for a large enterprise application?

---

# 39. Senior Automation Engineer Perspective

For a senior QA automation engineer, knowing individual Playwright commands is not enough.

You should understand how to build a maintainable framework:

```text
Playwright
    |
    +── Test Runner
    |
    +── Configuration
    |
    +── Fixtures
    |
    +── Page Objects
    |
    +── Utilities
    |
    +── Test Data
    |
    +── API Layer
    |
    +── Authentication
    |
    +── Reporting
    |
    +── Trace / Debugging
    |
    +── Parallel Execution
    |
    +── CI/CD
```

A strong Playwright framework should provide:

* Maintainability
* Reusability
* Scalability
* Parallel execution
* Cross-browser testing
* Reliable synchronization
* Good reporting
* Easy debugging
* CI/CD integration

---

# 40. Key Takeaways

Remember these core concepts:

```text
Playwright
    ↓
Browser
    ↓
BrowserContext
    ↓
Page
    ↓
Locator
    ↓
Action
    ↓
Assertion
```

The most important Playwright concepts to learn are:

1. Browser
2. BrowserContext
3. Page
4. Locator
5. Auto-waiting
6. Web-first assertions
7. Playwright Test
8. Fixtures
9. Page Object Model
10. Authentication
11. Network interception
12. API testing
13. Parallel execution
14. Trace Viewer
15. CI/CD

For someone coming from Selenium, the **Browser → BrowserContext → Page → Locator** hierarchy and **auto-waiting** are especially important.

---

# Quick Reference

```typescript
import { test, expect } from '@playwright/test';

test('Playwright basic test', async ({ page }) => {

    // Navigate
    await page.goto('https://example.com');

    // Get title
    console.log(await page.title());

    // Locator
    const button = page.getByRole('button', {
        name: 'Login'
    });

    // Click
    await button.click();

    // Assertion
    await expect(page).toHaveURL(/dashboard/);

});
```

This is the foundation for the rest of the **PlaywrightStudy** series.
