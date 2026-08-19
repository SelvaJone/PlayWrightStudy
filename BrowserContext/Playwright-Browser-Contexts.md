# Playwright Browser Contexts

## Overview

A **BrowserContext** in Playwright is an isolated browser session within a browser instance.

A BrowserContext is similar to an **incognito browser profile**. Each context has its own:

* Cookies
* Local storage
* Session storage
* Cache
* Permissions
* Authentication state
* Browser-level settings

Browser contexts are extremely useful for:

* Running tests independently
* Testing multiple users
* Testing different roles
* Parallel execution
* Authentication scenarios
* Multi-user workflows
* Session isolation

---

## 1. Browser vs BrowserContext vs Page

The Playwright hierarchy is:

```text
Browser
   |
   +-- BrowserContext
   |       |
   |       +-- Page
   |       +-- Page
   |
   +-- BrowserContext
           |
           +-- Page
           +-- Page
```

### Browser

Represents the actual browser instance.

```javascript
const browser = await chromium.launch();
```

### BrowserContext

Represents an isolated browser session.

```javascript
const context = await browser.newContext();
```

### Page

Represents a browser tab.

```javascript
const page = await context.newPage();
```

---

# 2. Basic BrowserContext Example

```javascript
const { chromium } = require('@playwright/test');

(async () => {
  const browser = await chromium.launch();

  const context = await browser.newContext();

  const page = await context.newPage();

  await page.goto('https://example.com');

  console.log(await page.title());

  await context.close();
  await browser.close();
})();
```

---

# 3. Creating Multiple Browser Contexts

You can create multiple isolated contexts from the same browser.

```javascript
const { chromium } = require('@playwright/test');

(async () => {
  const browser = await chromium.launch();

  const context1 = await browser.newContext();
  const context2 = await browser.newContext();

  const page1 = await context1.newPage();
  const page2 = await context2.newPage();

  await page1.goto('https://example.com');
  await page2.goto('https://example.com');

  await context1.close();
  await context2.close();

  await browser.close();
})();
```

Each context has independent browser state.

---

# 4. BrowserContext Isolation

Suppose User 1 logs into an application.

```javascript
const user1Context = await browser.newContext();
const user1Page = await user1Context.newPage();

await user1Page.goto('https://example.com/login');

await user1Page.fill('#username', 'user1');
await user1Page.fill('#password', 'password1');

await user1Page.click('#login');
```

Create another context for User 2:

```javascript
const user2Context = await browser.newContext();
const user2Page = await user2Context.newPage();

await user2Page.goto('https://example.com');
```

User 2 does not inherit User 1's cookies or session.

This is one of the most important advantages of BrowserContext.

---

# 5. BrowserContext and Cookies

Each BrowserContext has its own cookies.

```javascript
const context = await browser.newContext();

await context.addCookies([
  {
    name: 'sessionId',
    value: '12345',
    domain: 'example.com',
    path: '/'
  }
]);

const cookies = await context.cookies();

console.log(cookies);
```

---

# 6. Getting Cookies

```javascript
const cookies = await context.cookies();

for (const cookie of cookies) {
  console.log(cookie.name);
  console.log(cookie.value);
}
```

You can also retrieve cookies for a specific URL:

```javascript
const cookies = await context.cookies('https://example.com');

console.log(cookies);
```

---

# 7. Clearing Cookies

```javascript
await context.clearCookies();
```

This clears cookies belonging to the BrowserContext.

---

# 8. Local Storage and Session Storage

BrowserContext provides isolation for:

* Cookies
* Local storage
* Session storage
* IndexedDB

For example:

```javascript
await page.evaluate(() => {
  localStorage.setItem('username', 'testuser');
});
```

Another BrowserContext will not automatically see this value.

---

# 9. Multiple Users in One Test

BrowserContext is very useful when testing workflows involving multiple users.

For example:

```text
Admin
   |
   +-- BrowserContext 1

Customer
   |
   +-- BrowserContext 2
```

Example:

```javascript
const adminContext = await browser.newContext();
const customerContext = await browser.newContext();

const adminPage = await adminContext.newPage();
const customerPage = await customerContext.newPage();

await adminPage.goto('https://example.com');
await customerPage.goto('https://example.com');
```

The two users remain isolated.

---

# 10. Multi-User Testing Example

```javascript
const { test, expect } = require('@playwright/test');

test('Admin and customer workflow', async ({ browser }) => {

  const adminContext = await browser.newContext();
  const customerContext = await browser.newContext();

  const adminPage = await adminContext.newPage();
  const customerPage = await customerContext.newPage();

  // Admin login
  await adminPage.goto('https://example.com/login');

  await adminPage.fill('#username', 'admin');
  await adminPage.fill('#password', 'admin123');
  await adminPage.click('#login');

  // Customer login
  await customerPage.goto('https://example.com/login');

  await customerPage.fill('#username', 'customer');
  await customerPage.fill('#password', 'customer123');
  await customerPage.click('#login');

  // Perform workflow

  await adminContext.close();
  await customerContext.close();
});
```

---

# 11. BrowserContext with Playwright Test

In Playwright Test, a BrowserContext is normally provided automatically through the `context` fixture.

```javascript
import { test } from '@playwright/test';

test('BrowserContext example', async ({ context, page }) => {

  console.log(context);
  console.log(page);

  await page.goto('https://example.com');
});
```

Playwright Test creates an isolated context for each test.

---

# 12. Using the Context Fixture

```javascript
import { test, expect } from '@playwright/test';

test('Context example', async ({ context }) => {

  const page = await context.newPage();

  await page.goto('https://example.com');

  await expect(page).toHaveTitle(/Example/);
});
```

---

# 13. Creating a New Page in Existing Context

```javascript
const page1 = await context.newPage();
const page2 = await context.newPage();

await page1.goto('https://example.com');
await page2.goto('https://google.com');
```

Both pages belong to the same BrowserContext.

Therefore, they share the same:

* Cookies
* Local storage
* Session
* Permissions

---

# 14. Pages vs Contexts

Consider:

```javascript
const context = await browser.newContext();

const page1 = await context.newPage();
const page2 = await context.newPage();
```

Here:

```text
Context
 |
 +-- Page 1
 |
 +-- Page 2
```

Both pages share browser state.

But:

```javascript
const context1 = await browser.newContext();
const context2 = await browser.newContext();

const page1 = await context1.newPage();
const page2 = await context2.newPage();
```

Now:

```text
Context 1
 |
 +-- Page 1

Context 2
 |
 +-- Page 2
```

The sessions are isolated.

---

# 15. BrowserContext Options

BrowserContext can be configured when created.

```javascript
const context = await browser.newContext({
  viewport: {
    width: 1280,
    height: 720
  }
});
```

---

# 16. Setting User Agent

```javascript
const context = await browser.newContext({
  userAgent: 'My Custom User Agent'
});
```

This can be useful when testing applications that behave differently based on the user agent.

---

# 17. Setting Viewport

```javascript
const context = await browser.newContext({
  viewport: {
    width: 1920,
    height: 1080
  }
});
```

---

# 18. Setting Locale

```javascript
const context = await browser.newContext({
  locale: 'en-US'
});
```

Another example:

```javascript
const context = await browser.newContext({
  locale: 'fr-FR'
});
```

This is useful for localization testing.

---

# 19. Setting Timezone

```javascript
const context = await browser.newContext({
  timezoneId: 'America/New_York'
});
```

Another example:

```javascript
const context = await browser.newContext({
  timezoneId: 'Asia/Kolkata'
});
```

Useful for applications where date/time behavior depends on timezone.

---

# 20. Permissions

You can grant permissions to a BrowserContext.

```javascript
const context = await browser.newContext({
  permissions: ['geolocation']
});
```

For example:

```javascript
const context = await browser.newContext({
  permissions: ['notifications']
});
```

---

# 21. Geolocation

BrowserContext can simulate a geographic location.

```javascript
const context = await browser.newContext({
  geolocation: {
    latitude: 40.7128,
    longitude: -74.0060
  },
  permissions: ['geolocation']
});
```

This is useful for location-based application testing.

---

# 22. HTTP Authentication

BrowserContext can configure HTTP credentials.

```javascript
const context = await browser.newContext({
  httpCredentials: {
    username: 'admin',
    password: 'password'
  }
});
```

---

# 23. Extra HTTP Headers

You can configure headers for requests from the context.

```javascript
const context = await browser.newContext({
  extraHTTPHeaders: {
    'X-Test-Environment': 'QA'
  }
});
```

---

# 24. Ignore HTTPS Errors

For test environments with certificate issues:

```javascript
const context = await browser.newContext({
  ignoreHTTPSErrors: true
});
```

Use this carefully and generally only for appropriate test environments.

---

# 25. Color Scheme

You can emulate a color scheme.

```javascript
const context = await browser.newContext({
  colorScheme: 'dark'
});
```

Light mode:

```javascript
const context = await browser.newContext({
  colorScheme: 'light'
});
```

This is useful for testing dark/light mode functionality.

---

# 26. Device Emulation with BrowserContext

Playwright provides device descriptors.

```javascript
import { chromium, devices } from '@playwright/test';

const iPhone = devices['iPhone 13'];

const browser = await chromium.launch();

const context = await browser.newContext({
  ...iPhone
});

const page = await context.newPage();

await page.goto('https://example.com');
```

---

# 27. Authentication State

One of the most powerful BrowserContext features is authentication state.

You can save authentication state:

```javascript
await context.storageState({
  path: 'playwright/.auth/user.json'
});
```

The saved state can include:

* Cookies
* Local storage
* IndexedDB state where supported

---

# 28. Reusing Authentication State

Create a context using an existing authentication state:

```javascript
const context = await browser.newContext({
  storageState: 'playwright/.auth/user.json'
});
```

This allows tests to start already authenticated.

---

# 29. Authentication Example

```javascript
const context = await browser.newContext();

const page = await context.newPage();

await page.goto('https://example.com/login');

await page.fill('#username', 'testuser');
await page.fill('#password', 'password');

await page.click('#login');

await context.storageState({
  path: 'playwright/.auth/user.json'
});

await context.close();
```

Later:

```javascript
const context = await browser.newContext({
  storageState: 'playwright/.auth/user.json'
});

const page = await context.newPage();

await page.goto('https://example.com/dashboard');
```

---

# 30. Context-Level Routing

You can intercept network requests at the BrowserContext level.

```javascript
await context.route('**/*.png', async route => {
  await route.abort();
});
```

This applies to pages within that context.

---

# 31. Context-Level Request Handling

Example:

```javascript
await context.route('**/api/users', async route => {

  await route.fulfill({
    status: 200,
    contentType: 'application/json',
    body: JSON.stringify({
      users: [
        {
          id: 1,
          name: 'John'
        }
      ]
    })
  });

});
```

---

# 32. Context-Level Events

You can listen for context events.

```javascript
context.on('page', page => {
  console.log('New page created:', page.url());
});
```

For example:

```javascript
context.on('request', request => {
  console.log('Request:', request.url());
});
```

And:

```javascript
context.on('response', response => {
  console.log('Response:', response.url());
});
```

---

# 33. BrowserContext Pages

You can get all pages belonging to a context.

```javascript
const pages = context.pages();

console.log(pages.length);
```

Example:

```javascript
for (const page of context.pages()) {
  console.log(page.url());
}
```

---

# 34. BrowserContext Close

Always close manually created contexts when finished.

```javascript
await context.close();
```

Then close the browser:

```javascript
await browser.close();
```

Complete example:

```javascript
const browser = await chromium.launch();

const context = await browser.newContext();

const page = await context.newPage();

await page.goto('https://example.com');

await context.close();
await browser.close();
```

---

# 35. BrowserContext vs New Browser

A common question is:

**Why create a new BrowserContext instead of launching another browser?**

Creating a new context is much cheaper and faster.

### New Browser

```javascript
const browser1 = await chromium.launch();
const browser2 = await chromium.launch();
```

### New Context

```javascript
const browser = await chromium.launch();

const context1 = await browser.newContext();
const context2 = await browser.newContext();
```

Usually, multiple contexts within one browser are preferred when you need isolated sessions.

---

# 36. BrowserContext for Parallel Users

Example:

```javascript
const adminContext = await browser.newContext();
const userContext = await browser.newContext();
const managerContext = await browser.newContext();

const adminPage = await adminContext.newPage();
const userPage = await userContext.newPage();
const managerPage = await managerContext.newPage();
```

Each role has a separate session.

---

# 37. BrowserContext for Role-Based Testing

A useful test architecture is:

```text
Browser
 |
 +-- Admin Context
 |     |
 |     +-- Admin Page
 |
 +-- Manager Context
 |     |
 |     +-- Manager Page
 |
 +-- Customer Context
       |
       +-- Customer Page
```

This allows you to test workflows involving multiple roles.

---

# 38. BrowserContext for Multi-Tab Testing

Multiple tabs within the same context:

```javascript
const page1 = await context.newPage();
const page2 = await context.newPage();

await page1.goto('https://example.com');
await page2.goto('https://example.com/dashboard');
```

Because they share the same context, they share authentication/session state.

---

# 39. BrowserContext for Session Isolation

Example:

```javascript
const context1 = await browser.newContext();
const context2 = await browser.newContext();

const page1 = await context1.newPage();
const page2 = await context2.newPage();

await page1.goto('https://example.com/login');
await page2.goto('https://example.com/login');
```

Login performed in `context1` does not affect `context2`.

---

# 40. BrowserContext with Test Fixtures

A recommended Playwright Test pattern is to use fixtures:

```javascript
import { test, expect } from '@playwright/test';

test('Login test', async ({ page, context }) => {

  await page.goto('https://example.com/login');

  await context.storageState({
    path: 'auth.json'
  });

});
```

For most normal tests, you do not need to manually create a BrowserContext because Playwright Test manages it for you.

---

# 41. When Should You Create a BrowserContext Manually?

Use a manually created context when you need:

* Multiple users in one test
* Multiple isolated sessions
* Different authentication states
* Different locales
* Different permissions
* Different geolocations
* Custom browser settings
* Session isolation
* Multi-user collaboration testing

Example:

```javascript
const user1 = await browser.newContext({
  storageState: 'user1.json'
});

const user2 = await browser.newContext({
  storageState: 'user2.json'
});
```

---

# 42. Best Practices

### 1. Prefer BrowserContext over launching multiple browsers

```javascript
const context1 = await browser.newContext();
const context2 = await browser.newContext();
```

### 2. Keep contexts isolated

Do not intentionally share cookies or storage unless required.

### 3. Use authentication state

```javascript
storageState: 'playwright/.auth/user.json'
```

### 4. Close manually created contexts

```javascript
await context.close();
```

### 5. Use meaningful names

```javascript
const adminContext = await browser.newContext();
const customerContext = await browser.newContext();
```

### 6. Use Playwright fixtures when possible

```javascript
test('Example', async ({ page }) => {
});
```

Playwright Test handles context creation and cleanup automatically.

---

# 43. Common Mistakes

## Mistake 1: Sharing a Page Between Users

Incorrect:

```javascript
const page = await browser.newPage();
```

Then trying to use the same page for multiple authenticated users.

Better:

```javascript
const user1Context = await browser.newContext();
const user2Context = await browser.newContext();

const user1Page = await user1Context.newPage();
const user2Page = await user2Context.newPage();
```

---

## Mistake 2: Forgetting to Close Contexts

Incorrect:

```javascript
const context = await browser.newContext();
const page = await context.newPage();
```

Better:

```javascript
await context.close();
```

---

## Mistake 3: Expecting Contexts to Share Login State

```javascript
const context1 = await browser.newContext();
const context2 = await browser.newContext();
```

These contexts are isolated.

If shared authentication is required, explicitly use `storageState`.

---

# 44. Interview Questions

### Q1. What is a BrowserContext?

A BrowserContext is an isolated browser session that provides independent cookies, storage, permissions, and authentication state.

### Q2. Why is BrowserContext useful?

It allows multiple isolated sessions to run inside the same browser instance.

### Q3. Can multiple BrowserContexts exist in one browser?

Yes.

```javascript
const context1 = await browser.newContext();
const context2 = await browser.newContext();
```

### Q4. Do BrowserContexts share cookies?

No. Each context has its own cookies.

### Q5. Can multiple pages exist in one BrowserContext?

Yes.

```javascript
const page1 = await context.newPage();
const page2 = await context.newPage();
```

### Q6. Do pages in the same context share authentication?

Yes, because they share the same browser session.

### Q7. How do you create an authenticated context?

```javascript
const context = await browser.newContext({
  storageState: 'playwright/.auth/user.json'
});
```

### Q8. How do you close a BrowserContext?

```javascript
await context.close();
```

### Q9. How can you test two users simultaneously?

Create two BrowserContexts:

```javascript
const adminContext = await browser.newContext();
const customerContext = await browser.newContext();
```

### Q10. Is BrowserContext the same as a browser?

No.

```text
Browser
   |
   +-- BrowserContext
          |
          +-- Page
```

A browser is the browser process, while a BrowserContext is an isolated session within that browser.

---

# 45. Selenium Comparison

| Selenium                   | Playwright                        |
| -------------------------- | --------------------------------- |
| WebDriver                  | Browser                           |
| Browser session            | BrowserContext                    |
| Browser tab/window         | Page                              |
| Cookies                    | Context cookies                   |
| Separate WebDriver session | Separate BrowserContext           |
| Incognito-like profile     | BrowserContext                    |
| Driver-based               | Browser/Context/Page architecture |

A useful Selenium-to-Playwright mental model is:

```text
Selenium WebDriver
        ↓
Playwright BrowserContext

Selenium WebElement
        ↓
Playwright Locator

Selenium Window/Tab
        ↓
Playwright Page
```

---

# 46. Real-World Example

Suppose an application requires:

* Admin login
* Customer login
* Admin creates an order
* Customer verifies the order

BrowserContext is ideal for this workflow.

```javascript
import { test, expect } from '@playwright/test';

test('Admin creates order and customer verifies it', async ({ browser }) => {

  const adminContext = await browser.newContext();
  const customerContext = await browser.newContext();

  const adminPage = await adminContext.newPage();
  const customerPage = await customerContext.newPage();

  // Admin
  await adminPage.goto('https://example.com/login');

  await adminPage.fill('#username', 'admin');
  await adminPage.fill('#password', 'admin123');
  await adminPage.click('#login');

  await adminPage.goto('https://example.com/orders');
  await adminPage.click('#create-order');

  // Customer
  await customerPage.goto('https://example.com/login');

  await customerPage.fill('#username', 'customer');
  await customerPage.fill('#password', 'customer123');
  await customerPage.click('#login');

  await customerPage.goto('https://example.com/orders');

  await expect(
    customerPage.locator('.order')
  ).toBeVisible();

  await adminContext.close();
  await customerContext.close();
});
```

---

# 47. Key Takeaways

```text
Browser
   |
   +-- BrowserContext 1
   |       |
   |       +-- Page
   |       +-- Page
   |
   +-- BrowserContext 2
           |
           +-- Page
           +-- Page
```

Remember:

1. **Browser** = browser process.
2. **BrowserContext** = isolated browser session.
3. **Page** = browser tab.
4. Pages in the same context share session state.
5. Different contexts have isolated sessions.
6. BrowserContexts are excellent for multi-user testing.
7. `storageState` can be used to reuse authentication.
8. Context-level settings can control locale, timezone, permissions, geolocation, headers, and more.
9. Playwright Test automatically creates and manages contexts for normal tests.
10. Manually create contexts when you need multiple isolated sessions in the same test.

## Quick Reference

```javascript
// Create browser
const browser = await chromium.launch();

// Create context
const context = await browser.newContext();

// Create page
const page = await context.newPage();

// Create another isolated context
const secondContext = await browser.newContext();

// Create page in second context
const secondPage = await secondContext.newPage();

// Save authentication
await context.storageState({
  path: 'playwright/.auth/user.json'
});

// Reuse authentication
const authenticatedContext = await browser.newContext({
  storageState: 'playwright/.auth/user.json'
});

// Get pages
const pages = context.pages();

// Clear cookies
await context.clearCookies();

// Close contexts
await context.close();
await secondContext.close();
await authenticatedContext.close();

// Close browser
await browser.close();
```

**BrowserContext is one of the most important Playwright concepts because it provides fast, isolated, and configurable browser sessions without requiring a separate browser process for every user or test scenario.**
