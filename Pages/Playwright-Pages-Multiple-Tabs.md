# Playwright Pages & Multiple Tabs

## Overview

In Playwright, a **Page** represents a single browser tab or window.

A browser context can contain multiple pages:

```text
Browser
└── BrowserContext
    ├── Page 1
    ├── Page 2
    └── Page 3
```

Playwright makes it easy to:

* Open new tabs
* Work with multiple pages
* Handle popup windows
* Switch between tabs
* Wait for new pages
* Transfer data between pages
* Close pages
* Work with multiple browser contexts

---

## 1. Creating a Page

A page is normally created from a browser context.

### JavaScript

```javascript
const page = await context.newPage();
```

### TypeScript

```typescript
const page = await context.newPage();
```

### Python

```python
page = await context.new_page()
```

---

## 2. Using the Default Page

With Playwright Test, the `page` fixture is already available.

```typescript
import { test, expect } from '@playwright/test';

test('open page', async ({ page }) => {
  await page.goto('https://example.com');

  await expect(page).toHaveTitle(/Example/);
});
```

There is no need to manually create the first page.

---

# 3. Opening a New Tab

Create another page from the same browser context.

```typescript
const newPage = await context.newPage();

await newPage.goto('https://example.com');
```

Both pages belong to the same browser context.

```typescript
const page1 = await context.newPage();
const page2 = await context.newPage();

await page1.goto('https://example.com');
await page2.goto('https://playwright.dev');
```

---

# 4. Multiple Pages in One Context

You can have multiple pages inside the same context.

```typescript
const page1 = await context.newPage();
const page2 = await context.newPage();
const page3 = await context.newPage();

await page1.goto('https://example.com');
await page2.goto('https://playwright.dev');
await page3.goto('https://google.com');
```

You can access all pages using:

```typescript
const pages = context.pages();

console.log(pages.length);
```

---

# 5. Get All Open Pages

```typescript
const pages = context.pages();

for (const page of pages) {
  console.log(await page.title());
}
```

Example output:

```text
Example Domain
Playwright
Google
```

---

# 6. Get the First Page

```typescript
const pages = context.pages();

const firstPage = pages[0];
```

---

# 7. Get the Last Page

```typescript
const pages = context.pages();

const lastPage = pages[pages.length - 1];
```

---

# 8. Handling a New Tab

A common scenario is clicking a link that opens a new tab.

For example:

```html
<a href="https://playwright.dev" target="_blank">
    Open Playwright
</a>
```

The recommended Playwright approach is to wait for the new page while performing the click.

```typescript
const newPagePromise = context.waitForEvent('page');

await page.getByText('Open Playwright').click();

const newPage = await newPagePromise;

await newPage.waitForLoadState();

console.log(await newPage.title());
```

---

# 9. Using Promise.all()

A cleaner approach is:

```typescript
const [newPage] = await Promise.all([
  context.waitForEvent('page'),
  page.getByText('Open Playwright').click()
]);

await newPage.waitForLoadState();
```

This is one of the most important patterns for handling new tabs.

---

# 10. Why Promise.all() Is Important

Avoid this pattern:

```typescript
await page.getByText('Open Playwright').click();

const newPage = await context.waitForEvent('page');
```

The new page event could occur before `waitForEvent()` starts listening.

Instead:

```typescript
const [newPage] = await Promise.all([
  context.waitForEvent('page'),
  page.getByText('Open Playwright').click()
]);
```

The listener is registered before the click happens.

---

# 11. Waiting for the New Page

After capturing the page, wait for it to load.

```typescript
const [newPage] = await Promise.all([
  context.waitForEvent('page'),
  page.locator('a[target="_blank"]').click()
]);

await newPage.waitForLoadState('domcontentloaded');

console.log(await newPage.url());
```

Possible load states:

```typescript
await newPage.waitForLoadState('domcontentloaded');
```

```typescript
await newPage.waitForLoadState('load');
```

```typescript
await newPage.waitForLoadState('networkidle');
```

Use `domcontentloaded` or `load` when possible rather than relying unnecessarily on `networkidle`.

---

# 12. Handling Popup Windows

A popup is another page opened as a result of an action.

Example:

```typescript
const [popup] = await Promise.all([
  page.waitForEvent('popup'),
  page.getByRole('button', { name: 'Open Popup' }).click()
]);

await popup.waitForLoadState();

console.log(await popup.title());
```

`page.waitForEvent('popup')` is useful when the popup is specifically opened by a page action.

---

# 13. Page vs Context Page Event

There are two important patterns.

### Context-level new page

```typescript
const [newPage] = await Promise.all([
  context.waitForEvent('page'),
  page.getByText('Open Tab').click()
]);
```

### Page-level popup

```typescript
const [popup] = await Promise.all([
  page.waitForEvent('popup'),
  page.getByText('Open Popup').click()
]);
```

Generally:

* Use `context.waitForEvent('page')` for a new page/tab in the context.
* Use `page.waitForEvent('popup')` when the page action specifically opens a popup.

---

# 14. Complete Multiple-Tab Example

```typescript
import { test, expect } from '@playwright/test';

test('handle multiple tabs', async ({ page, context }) => {

  await page.goto('https://example.com');

  const [newPage] = await Promise.all([
    context.waitForEvent('page'),
    page.getByText('Open New Tab').click()
  ]);

  await newPage.waitForLoadState('domcontentloaded');

  console.log('Original Page:', await page.url());
  console.log('New Page:', await newPage.url());

  await expect(newPage).toHaveTitle(/Example/);
});
```

---

# 15. Working With the Original Page

You don't need to manually switch back to the original page.

Keep references to both pages.

```typescript
const originalPage = page;

const [newPage] = await Promise.all([
  context.waitForEvent('page'),
  page.getByText('Open New Tab').click()
]);

await newPage.waitForLoadState();

await originalPage.bringToFront();

console.log(await originalPage.title());
```

---

# 16. Bring a Page to the Front

Use:

```typescript
await page.bringToFront();
```

Example:

```typescript
await page2.bringToFront();

await page2.getByRole('button', { name: 'Submit' }).click();
```

However, in most automated tests, you don't need to manually switch tabs. You can interact directly with the appropriate `Page` object.

---

# 17. Playwright Does Not Require Selenium-Style Switching

In Selenium, you commonly see:

```java
driver.switchTo().window(windowHandle);
```

Playwright works differently.

You keep a reference to the page:

```typescript
const [newPage] = await Promise.all([
  context.waitForEvent('page'),
  page.click('a[target="_blank"]')
]);

await newPage.getByRole('button', { name: 'Submit' }).click();
```

There is no equivalent `switchTo().window()` requirement.

---

# 18. Getting the Page URL

```typescript
console.log(page.url());
```

For another tab:

```typescript
console.log(newPage.url());
```

---

# 19. Getting the Page Title

```typescript
const title = await page.title();

console.log(title);
```

For a second page:

```typescript
const title = await newPage.title();

console.log(title);
```

---

# 20. Navigating a New Page

```typescript
const newPage = await context.newPage();

await newPage.goto('https://playwright.dev');

console.log(await newPage.url());
```

---

# 21. Closing a Page

Close a specific page:

```typescript
await newPage.close();
```

Check whether a page is closed:

```typescript
console.log(newPage.isClosed());
```

Example:

```typescript
await newPage.close();

console.log(newPage.isClosed());
```

Output:

```text
true
```

---

# 22. Closing All Pages

```typescript
for (const page of context.pages()) {
  await page.close();
}
```

Usually, you don't need to do this manually because Playwright Test handles browser cleanup.

---

# 23. Detecting New Pages

You can listen for new pages:

```typescript
context.on('page', async page => {
  console.log('New page opened:', page.url());
});
```

This is useful for monitoring behavior.

For tests where you need the specific new page, prefer:

```typescript
const [newPage] = await Promise.all([
  context.waitForEvent('page'),
  page.click('a[target="_blank"]')
]);
```

---

# 24. Multiple Tabs Example

```typescript
import { test } from '@playwright/test';

test('multiple tabs', async ({ page, context }) => {

  await page.goto('https://example.com');

  const page2 = await context.newPage();
  const page3 = await context.newPage();

  await page2.goto('https://playwright.dev');
  await page3.goto('https://google.com');

  console.log('Page 1:', await page.url());
  console.log('Page 2:', await page2.url());
  console.log('Page 3:', await page3.url());

});
```

---

# 25. Iterating Through Multiple Pages

```typescript
const pages = context.pages();

for (const currentPage of pages) {
  console.log('URL:', currentPage.url());
  console.log('Title:', await currentPage.title());
}
```

---

# 26. Finding a Specific Page

You can identify a page by URL.

```typescript
const targetPage = context
  .pages()
  .find(p => p.url().includes('playwright.dev'));

if (targetPage) {
  console.log(await targetPage.title());
}
```

---

# 27. Waiting for a Specific URL

After opening a page:

```typescript
await newPage.waitForURL('**/dashboard');

console.log(await newPage.url());
```

You can also use a regular expression:

```typescript
await newPage.waitForURL(/dashboard/);
```

---

# 28. New Tab With URL Validation

```typescript
const [newPage] = await Promise.all([
  context.waitForEvent('page'),
  page.getByRole('link', { name: 'Dashboard' }).click()
]);

await newPage.waitForLoadState('domcontentloaded');

await newPage.waitForURL('**/dashboard');

console.log('Dashboard opened successfully');
```

---

# 29. New Tab With Assertion

```typescript
const [newPage] = await Promise.all([
  context.waitForEvent('page'),
  page.getByRole('link', { name: 'Help' }).click()
]);

await newPage.waitForLoadState();

await expect(newPage).toHaveURL(/help/);
```

---

# 30. Handling Authentication in a New Tab

Suppose the original page opens an authentication page.

```typescript
const [authPage] = await Promise.all([
  context.waitForEvent('page'),
  page.getByText('Login').click()
]);

await authPage.waitForLoadState();

await authPage.getByLabel('Username').fill('testuser');
await authPage.getByLabel('Password').fill('password');

await authPage.getByRole('button', { name: 'Login' }).click();
```

Because both pages belong to the same context, cookies and other context-level state can be shared according to the context configuration.

---

# 31. Sharing Data Between Pages

You can read data from one page and use it on another.

```typescript
const value = await page.locator('#accountNumber').textContent();

const newPage = await context.newPage();

await newPage.goto('https://example.com/search');

await newPage.locator('#search').fill(value ?? '');
```

---

# 32. Sharing Cookies Between Pages

Pages created from the same browser context share the context's cookie state.

```typescript
const page1 = await context.newPage();

await page1.goto('https://example.com');

const page2 = await context.newPage();

await page2.goto('https://example.com/profile');
```

If authentication cookies were established in the context, the second page can use the same context state.

---

# 33. Pages in Different Contexts

Pages in different browser contexts are isolated.

```typescript
const context1 = await browser.newContext();
const context2 = await browser.newContext();

const page1 = await context1.newPage();
const page2 = await context2.newPage();
```

The pages belong to separate sessions.

```text
Browser
├── Context 1
│   └── Page 1
│
└── Context 2
    └── Page 2
```

---

# 34. Multi-User Testing

Separate contexts are useful when testing multiple users.

```typescript
const user1Context = await browser.newContext();
const user2Context = await browser.newContext();

const user1Page = await user1Context.newPage();
const user2Page = await user2Context.newPage();

await user1Page.goto('https://example.com');
await user2Page.goto('https://example.com');
```

Each context has its own browser session.

---

# 35. New Tab From a Button

```typescript
const [newPage] = await Promise.all([
  context.waitForEvent('page'),
  page.getByRole('button', { name: 'Open Report' }).click()
]);

await newPage.waitForLoadState();

await expect(newPage).toHaveURL(/report/);
```

---

# 36. New Tab From a Link

```typescript
const [newPage] = await Promise.all([
  context.waitForEvent('page'),
  page.getByRole('link', { name: 'Documentation' }).click()
]);

await newPage.waitForLoadState();

console.log(await newPage.url());
```

---

# 37. New Tab With `target="_blank"`

```typescript
const [newPage] = await Promise.all([
  context.waitForEvent('page'),
  page.locator('a[target="_blank"]').click()
]);

await newPage.waitForLoadState();

console.log(await newPage.title());
```

---

# 38. Popup Example

```typescript
const [popup] = await Promise.all([
  page.waitForEvent('popup'),
  page.getByRole('button', { name: 'Open Window' }).click()
]);

await popup.waitForLoadState();

await popup.getByRole('textbox').fill('Playwright');

await popup.getByRole('button', { name: 'Search' }).click();
```

---

# 39. Waiting for a Page Without Immediately Using It

```typescript
const newPagePromise = context.waitForEvent('page');

await page.getByText('Open Tab').click();

const newPage = await newPagePromise;

await newPage.waitForLoadState();
```

This is valid, but the `Promise.all()` pattern is usually easier to read and protects against timing issues.

---

# 40. Multiple New Tabs

If an action opens multiple pages, you can listen for page events.

```typescript
const pagesBefore = context.pages().length;

await page.getByRole('button', { name: 'Open Multiple' }).click();

await page.waitForTimeout(1000);

const pagesAfter = context.pages();

console.log('Pages before:', pagesBefore);
console.log('Pages after:', pagesAfter.length);
```

For deterministic tests, prefer event-based waiting rather than arbitrary sleeps.

---

# 41. Avoid Hard Waits

Avoid:

```typescript
await page.waitForTimeout(5000);
```

when waiting for a new tab.

Prefer:

```typescript
const [newPage] = await Promise.all([
  context.waitForEvent('page'),
  page.getByText('Open Tab').click()
]);
```

This is faster and more reliable.

---

# 42. Page Event With a URL Filter

You can capture a page and then validate its URL.

```typescript
const [newPage] = await Promise.all([
  context.waitForEvent('page'),
  page.getByText('Open Account').click()
]);

await newPage.waitForURL(/account/);

console.log('Account page opened');
```

---

# 43. Page Lifecycle

A typical page lifecycle is:

```text
Create Page
     ↓
Navigate
     ↓
Interact
     ↓
Wait for New Page/Popup
     ↓
Interact With New Page
     ↓
Close Page
```

Example:

```typescript
const newPage = await context.newPage();

await newPage.goto('https://example.com');

await newPage.getByRole('button', { name: 'Submit' }).click();

await newPage.close();
```

---

# 44. Best Practice: Store Page References

Instead of repeatedly doing:

```typescript
context.pages()[1]
```

store the page:

```typescript
const [newPage] = await Promise.all([
  context.waitForEvent('page'),
  page.click('a[target="_blank"]')
]);
```

Then use:

```typescript
await newPage.getByRole('button', { name: 'Continue' }).click();
```

This makes tests much easier to understand.

---

# 45. Best Practice: Use Meaningful Names

Avoid:

```typescript
const p1 = page;
const p2 = newPage;
```

Prefer:

```typescript
const loginPage = page;
const dashboardPage = newPage;
```

Or:

```typescript
const mainPage = page;
const paymentPage = newPage;
```

Meaningful names improve test readability.

---

# 46. Best Practice: Don't Manually Switch Tabs

Selenium-style:

```java
driver.switchTo().window(windowHandle);
```

is generally unnecessary in Playwright.

Instead:

```typescript
const [paymentPage] = await Promise.all([
  context.waitForEvent('page'),
  page.getByText('Payment').click()
]);

await paymentPage.getByLabel('Card Number').fill('4111111111111111');
```

---

# 47. Best Practice: Use Event-Based Synchronization

Preferred:

```typescript
const [newPage] = await Promise.all([
  context.waitForEvent('page'),
  page.getByRole('link', { name: 'Open' }).click()
]);
```

Avoid:

```typescript
await page.click('a');
await page.waitForTimeout(3000);
const pages = context.pages();
```

Event-based synchronization is more reliable.

---

# 48. Best Practice: Validate the New Page

Don't just capture the new page.

Validate it:

```typescript
const [newPage] = await Promise.all([
  context.waitForEvent('page'),
  page.getByText('Reports').click()
]);

await newPage.waitForLoadState();

await expect(newPage).toHaveURL(/reports/);
await expect(newPage.getByRole('heading', { name: 'Reports' })).toBeVisible();
```

---

# 49. Best Practice: Close Temporary Pages

If you create temporary pages manually:

```typescript
const tempPage = await context.newPage();

try {
  await tempPage.goto('https://example.com');
  // Test logic
} finally {
  await tempPage.close();
}
```

---

# 50. Complete Interview Example

### Scenario

A user clicks a link that opens a new tab. Validate the new tab.

```typescript
import { test, expect } from '@playwright/test';

test('validate new tab', async ({ page, context }) => {

  await page.goto('https://example.com');

  const [newPage] = await Promise.all([
    context.waitForEvent('page'),
    page.getByRole('link', { name: 'Open New Tab' }).click()
  ]);

  await newPage.waitForLoadState('domcontentloaded');

  await expect(newPage).toHaveURL(/example/);

  console.log('New tab URL:', newPage.url());

  await expect(
    newPage.getByRole('heading')
  ).toBeVisible();
});
```

---

# 51. Playwright vs Selenium

| Feature           | Selenium              | Playwright                     |
| ----------------- | --------------------- | ------------------------------ |
| Browser tab       | Window handle         | `Page`                         |
| Switch tab        | `switchTo().window()` | Use page reference             |
| New window        | Window handle         | `Page`                         |
| New tab event     | Manual handling       | `context.waitForEvent('page')` |
| Popup             | Window handling       | `page.waitForEvent('popup')`   |
| Multiple tabs     | Window handles        | Multiple `Page` objects        |
| Session isolation | Driver/session        | Browser contexts               |
| Cookie isolation  | Driver/session        | Browser context                |
| Tab interaction   | Switch required       | Direct page reference          |

---

# 52. Common Mistakes

## Mistake 1: Clicking Before Waiting for the Page

Incorrect:

```typescript
await page.click('a[target="_blank"]');

const newPage = await context.waitForEvent('page');
```

Correct:

```typescript
const [newPage] = await Promise.all([
  context.waitForEvent('page'),
  page.click('a[target="_blank"]')
]);
```

---

## Mistake 2: Using Hard Waits

Avoid:

```typescript
await page.waitForTimeout(5000);
```

Prefer:

```typescript
const [newPage] = await Promise.all([
  context.waitForEvent('page'),
  page.click('a[target="_blank"]')
]);
```

---

## Mistake 3: Using Page Indexes Everywhere

Avoid:

```typescript
const newPage = context.pages()[1];
```

Prefer:

```typescript
const [newPage] = await Promise.all([
  context.waitForEvent('page'),
  page.click('a[target="_blank"]')
]);
```

---

## Mistake 4: Forgetting to Wait for Navigation

Avoid immediately interacting when navigation may still be happening.

Prefer:

```typescript
await newPage.waitForLoadState('domcontentloaded');

await newPage.getByRole('button', { name: 'Continue' }).click();
```

---

# 53. JavaScript Example

```javascript
const [newPage] = await Promise.all([
  context.waitForEvent('page'),
  page.getByText('Open Tab').click()
]);

await newPage.waitForLoadState();

console.log(await newPage.url());
```

---

# 54. TypeScript Example

```typescript
const [newPage] = await Promise.all([
  context.waitForEvent('page'),
  page.getByText('Open Tab').click()
]);

await newPage.waitForLoadState('domcontentloaded');

await expect(newPage).toHaveURL(/dashboard/);
```

---

# 55. Python Example

```python
async with context.expect_page() as new_page_info:
    await page.get_by_text("Open Tab").click()

new_page = await new_page_info.value

await new_page.wait_for_load_state()

print(await new_page.url())
```

For a popup:

```python
async with page.expect_popup() as popup_info:
    await page.get_by_text("Open Popup").click()

popup = await popup_info.value

await popup.wait_for_load_state()
```

---

# 56. Java Example

```java
Page newPage;

try (Page.WaitForEventContext event = page.waitForEvent("popup")) {
    page.getByText("Open Popup").click();
    newPage = event.page();
}

newPage.waitForLoadState();

System.out.println(newPage.url());
```

---

# 57. Real-World Test Scenario

### Scenario: Open Terms and Conditions in a New Tab

```typescript
import { test, expect } from '@playwright/test';

test('validate terms and conditions in new tab', async ({ page, context }) => {

  await page.goto('https://example.com');

  const [termsPage] = await Promise.all([
    context.waitForEvent('page'),
    page.getByRole('link', { name: 'Terms and Conditions' }).click()
  ]);

  await termsPage.waitForLoadState('domcontentloaded');

  await expect(termsPage).toHaveURL(/terms/);

  await expect(
    termsPage.getByRole('heading', { name: /Terms/i })
  ).toBeVisible();
});
```

---

# 58. Real-World Test Scenario: Payment Provider

```typescript
test('payment provider opens in new tab', async ({ page, context }) => {

  await page.goto('/checkout');

  const [paymentPage] = await Promise.all([
    context.waitForEvent('page'),
    page.getByRole('button', { name: 'Pay Now' }).click()
  ]);

  await paymentPage.waitForLoadState('domcontentloaded');

  await expect(paymentPage).toHaveURL(/payment/);

  await paymentPage.getByLabel('Card Number').fill('4111111111111111');

  await paymentPage.getByRole('button', { name: 'Continue' }).click();
});
```

---

# 59. Real-World Test Scenario: Multiple User Sessions

```typescript
test('multiple users using separate contexts', async ({ browser }) => {

  const adminContext = await browser.newContext();
  const userContext = await browser.newContext();

  const adminPage = await adminContext.newPage();
  const userPage = await userContext.newPage();

  await adminPage.goto('https://example.com');
  await userPage.goto('https://example.com');

  // Admin and user have isolated sessions.

  await adminContext.close();
  await userContext.close();
});
```

---

# 60. Key Methods to Remember

### Create page

```typescript
const page = await context.newPage();
```

### Get all pages

```typescript
context.pages();
```

### Wait for new page

```typescript
context.waitForEvent('page');
```

### Wait for popup

```typescript
page.waitForEvent('popup');
```

### Bring page to front

```typescript
page.bringToFront();
```

### Get URL

```typescript
page.url();
```

### Get title

```typescript
await page.title();
```

### Close page

```typescript
await page.close();
```

### Check closed

```typescript
page.isClosed();
```

### Wait for URL

```typescript
await page.waitForURL('**/dashboard');
```

---

# 61. Most Important Pattern

For interviews and real automation projects, remember this pattern:

```typescript
const [newPage] = await Promise.all([
  context.waitForEvent('page'),
  page.getByRole('link', { name: 'Open New Tab' }).click()
]);

await newPage.waitForLoadState('domcontentloaded');

await expect(newPage).toHaveURL(/expected-url/);
```

For a popup:

```typescript
const [popup] = await Promise.all([
  page.waitForEvent('popup'),
  page.getByRole('button', { name: 'Open Popup' }).click()
]);

await popup.waitForLoadState('domcontentloaded');
```

---

# 62. Summary

Playwright represents browser tabs and windows as **Page objects**.

The key concepts are:

1. One `Page` represents one browser tab/window.
2. Multiple pages can exist inside one `BrowserContext`.
3. Use `context.newPage()` to create a page manually.
4. Use `context.pages()` to retrieve open pages.
5. Use `context.waitForEvent('page')` to capture a new tab.
6. Use `page.waitForEvent('popup')` for popups.
7. Use `Promise.all()` when an action triggers a new page.
8. You normally don't need Selenium-style window switching.
9. Keep references to pages instead of relying on page indexes.
10. Use event-based synchronization instead of hard waits.
11. Browser contexts provide session isolation.
12. Multiple pages in the same context can share context-level browser state.
13. Validate the new page using URL, title, and element assertions.
14. Close manually created pages when they are no longer needed.

## Quick Reference

```typescript
// Create a page
const newPage = await context.newPage();

// Get all pages
const pages = context.pages();

// New tab
const [newPage] = await Promise.all([
  context.waitForEvent('page'),
  page.getByText('Open Tab').click()
]);

// Popup
const [popup] = await Promise.all([
  page.waitForEvent('popup'),
  page.getByText('Open Popup').click()
]);

// Wait for loading
await newPage.waitForLoadState('domcontentloaded');

// URL
console.log(newPage.url());

// Title
console.log(await newPage.title());

// Bring to front
await newPage.bringToFront();

// Close
await newPage.close();
```

---

## Interview Questions

### Q1. What is a Page in Playwright?

A `Page` represents a single browser tab or window.

### Q2. How do you handle a new tab?

```typescript
const [newPage] = await Promise.all([
  context.waitForEvent('page'),
  page.click('a[target="_blank"]')
]);
```

### Q3. How do you handle a popup?

```typescript
const [popup] = await Promise.all([
  page.waitForEvent('popup'),
  page.click('#open-popup')
]);
```

### Q4. Does Playwright require `switchTo().window()`?

No. Playwright allows you to keep a reference to the desired `Page` and interact with it directly.

### Q5. How do you get all open pages?

```typescript
const pages = context.pages();
```

### Q6. How do you create a new page?

```typescript
const page = await context.newPage();
```

### Q7. How do you close a page?

```typescript
await page.close();
```

### Q8. How are multiple users tested independently?

Create separate browser contexts:

```typescript
const user1 = await browser.newContext();
const user2 = await browser.newContext();
```

### Q9. Why use `Promise.all()` for a new tab?

It ensures Playwright starts listening for the page event before the action that creates the new page occurs.

### Q10. What is the difference between a BrowserContext and Page?

A `BrowserContext` represents an isolated browser session, while a `Page` represents a tab/window inside that session.
