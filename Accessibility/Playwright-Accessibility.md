# Playwright Accessibility Testing

## Overview

Playwright provides built-in support for testing web accessibility through locators, ARIA roles, accessible names, keyboard interactions, and integration with accessibility testing tools such as Axe.

Accessibility testing helps ensure that applications can be used by people with disabilities, including users who rely on:

* Screen readers
* Keyboard navigation
* Voice control
* High-contrast modes
* Assistive technologies

Playwright does not replace a complete accessibility audit, but it provides excellent tools for automated accessibility checks.

---

## 1. Why Accessibility Testing Matters

A web application should be usable by as many people as possible.

Common accessibility requirements include:

* Proper semantic HTML
* Correct ARIA attributes
* Keyboard accessibility
* Meaningful labels
* Accessible buttons and links
* Form field labels
* Correct heading structure
* Alternative text for images
* Visible focus indicators
* Appropriate color contrast

Accessibility testing should be part of the overall test automation strategy.

---

# 2. Playwright and Accessibility

Playwright understands accessibility concepts through its locator engine.

For example:

```java
page.getByRole(AriaRole.BUTTON, new Page.GetByRoleOptions().setName("Login"));
```

This locator searches for a button based on its accessible role and name.

In JavaScript/TypeScript:

```javascript
await page.getByRole('button', { name: 'Login' }).click();
```

The role-based approach is generally preferred over fragile CSS or XPath selectors.

---

# 3. Common Accessibility Roles

Some commonly used ARIA roles are:

| Role       | Example                    |
| ---------- | -------------------------- |
| button     | `<button>Login</button>`   |
| link       | `<a href="/home">Home</a>` |
| textbox    | `<input>`                  |
| checkbox   | `<input type="checkbox">`  |
| radio      | `<input type="radio">`     |
| heading    | `<h1>Welcome</h1>`         |
| combobox   | `<select>`                 |
| list       | `<ul>`                     |
| listitem   | `<li>`                     |
| dialog     | Modal dialog               |
| navigation | Navigation section         |
| alert      | Alert message              |
| tab        | Tab control                |
| table      | HTML table                 |

---

# 4. Using getByRole()

`getByRole()` is one of the most important Playwright accessibility-oriented locators.

Example:

```javascript
await page.getByRole('button', { name: 'Login' }).click();
```

Another example:

```javascript
await page.getByRole('link', { name: 'Products' }).click();
```

Checkbox:

```javascript
await page.getByRole('checkbox', { name: 'Remember me' }).check();
```

Radio button:

```javascript
await page.getByRole('radio', { name: 'Male' }).check();
```

---

# 5. Accessible Names

An accessible name is the name that assistive technologies use to identify an element.

Example:

```html
<button aria-label="Close">X</button>
```

The accessible name is:

```text
Close
```

Playwright can locate it using:

```javascript
await page.getByRole('button', { name: 'Close' }).click();
```

---

# 6. Accessible Name from Visible Text

Example:

```html
<button>Submit</button>
```

Playwright:

```javascript
await page.getByRole('button', { name: 'Submit' }).click();
```

The button's visible text becomes its accessible name.

---

# 7. Accessible Name Using aria-label

HTML:

```html
<button aria-label="Search">🔍</button>
```

Playwright:

```javascript
await page.getByRole('button', { name: 'Search' }).click();
```

This is preferable to relying on the icon itself.

---

# 8. Accessible Forms

Forms should have meaningful labels.

Example:

```html
<label for="username">Username</label>
<input id="username" type="text">
```

Playwright:

```javascript
await page.getByLabel('Username').fill('selva');
```

Password:

```javascript
await page.getByLabel('Password').fill('Password123');
```

Login:

```javascript
await page.getByRole('button', { name: 'Login' }).click();
```

---

# 9. getByLabel()

`getByLabel()` is especially useful for accessible form testing.

Example:

```javascript
await page.getByLabel('Email').fill('test@example.com');
```

Checkbox:

```javascript
await page.getByLabel('Accept Terms').check();
```

Radio:

```javascript
await page.getByLabel('Male').check();
```

---

# 10. Heading Accessibility

Applications should have meaningful heading structures.

Example:

```html
<h1>Account Settings</h1>
<h2>Profile</h2>
<h2>Security</h2>
```

Playwright:

```javascript
await expect(
  page.getByRole('heading', { name: 'Account Settings' })
).toBeVisible();
```

Specific heading level:

```javascript
await expect(
  page.getByRole('heading', {
    name: 'Account Settings',
    level: 1
  })
).toBeVisible();
```

---

# 11. Testing Links

Example:

```javascript
await expect(
  page.getByRole('link', { name: 'Privacy Policy' })
).toBeVisible();
```

Click:

```javascript
await page.getByRole('link', { name: 'Privacy Policy' }).click();
```

---

# 12. Testing Buttons

Example:

```javascript
await expect(
  page.getByRole('button', { name: 'Submit' })
).toBeEnabled();
```

Click:

```javascript
await page.getByRole('button', { name: 'Submit' }).click();
```

Disabled button:

```javascript
await expect(
  page.getByRole('button', { name: 'Submit' })
).toBeDisabled();
```

---

# 13. Testing Checkboxes

Example:

```javascript
const checkbox = page.getByRole('checkbox', {
  name: 'Accept Terms'
});

await checkbox.check();

await expect(checkbox).toBeChecked();
```

Uncheck:

```javascript
await checkbox.uncheck();
```

---

# 14. Testing Radio Buttons

Example:

```javascript
const radio = page.getByRole('radio', {
  name: 'Credit Card'
});

await radio.check();

await expect(radio).toBeChecked();
```

---

# 15. Testing Keyboard Accessibility

Many users navigate applications using only a keyboard.

Playwright supports keyboard operations.

Example:

```javascript
await page.keyboard.press('Tab');
```

Next element:

```javascript
await page.keyboard.press('Tab');
```

Previous element:

```javascript
await page.keyboard.press('Shift+Tab');
```

Activate:

```javascript
await page.keyboard.press('Enter');
```

Escape:

```javascript
await page.keyboard.press('Escape');
```

---

# 16. Keyboard Navigation Test

Example:

```javascript
test('keyboard navigation', async ({ page }) => {
  await page.goto('https://example.com');

  await page.keyboard.press('Tab');

  await expect(
    page.getByRole('link', { name: 'Home' })
  ).toBeFocused();
});
```

The exact element that receives focus depends on the application's tab order.

---

# 17. Testing Focus

Playwright provides `toBeFocused()`.

Example:

```javascript
await page.getByLabel('Username').focus();

await expect(
  page.getByLabel('Username')
).toBeFocused();
```

This is useful for testing keyboard accessibility.

---

# 18. Dialog Accessibility

Example HTML:

```html
<div role="dialog" aria-label="Confirmation">
    <h2>Delete Account?</h2>
    <button>Cancel</button>
    <button>Delete</button>
</div>
```

Playwright:

```javascript
const dialog = page.getByRole('dialog', {
  name: 'Confirmation'
});

await expect(dialog).toBeVisible();
```

Buttons:

```javascript
await dialog.getByRole('button', {
  name: 'Delete'
}).click();
```

---

# 19. Testing Alerts

Example:

```html
<div role="alert">
    Invalid username or password.
</div>
```

Playwright:

```javascript
await expect(
  page.getByRole('alert')
).toHaveText('Invalid username or password.');
```

---

# 20. Testing Navigation

Example:

```html
<nav aria-label="Main Navigation">
```

Playwright:

```javascript
await expect(
  page.getByRole('navigation', {
    name: 'Main Navigation'
  })
).toBeVisible();
```

---

# 21. Testing Images

Images should have meaningful alternative text when they convey information.

Example:

```html
<img src="car.png" alt="Toyota vehicle">
```

Playwright:

```javascript
await expect(
  page.getByAltText('Toyota vehicle')
).toBeVisible();
```

Decorative images can appropriately use:

```html
<img src="decorative.png" alt="">
```

---

# 22. Testing Accessible Textboxes

Example:

```javascript
const username = page.getByRole('textbox', {
  name: 'Username'
});

await username.fill('selva');

await expect(username).toHaveValue('selva');
```

---

# 23. Accessibility-Friendly Locators

Preferred locator order generally starts with user-facing and semantic locators:

```text
getByRole()
getByLabel()
getByPlaceholder()
getByText()
getByTestId()
CSS
XPath
```

Example:

```javascript
page.getByRole('button', { name: 'Login' })
```

is usually preferable to:

```javascript
page.locator('#loginButton')
```

because the role-based locator reflects how a user or assistive technology identifies the element.

---

# 24. Avoid Overusing CSS and XPath

Avoid:

```javascript
page.locator('div.header > button:nth-child(2)')
```

Prefer:

```javascript
page.getByRole('button', { name: 'Login' })
```

Avoid:

```javascript
page.locator('//button[@id="login"]')
```

Prefer:

```javascript
page.getByRole('button', { name: 'Login' })
```

---

# 25. Accessibility Snapshot

Playwright can inspect an element's accessibility information using accessibility-related browser evaluation approaches.

For modern Playwright testing, role-based locators are generally the preferred way to validate accessible behavior.

For deeper accessibility-tree inspection, browser-level accessibility tooling or dedicated accessibility testing libraries can be used.

---

# 26. Axe Accessibility Testing

Playwright can be integrated with `axe-core` for automated accessibility scanning.

Install:

```bash
npm install -D @axe-core/playwright
```

---

# 27. Basic Axe Test

Example:

```javascript
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test('accessibility scan', async ({ page }) => {
  await page.goto('https://example.com');

  const accessibilityScanResults =
    await new AxeBuilder({ page }).analyze();

  expect(accessibilityScanResults.violations).toEqual([]);
});
```

This scans the page for accessibility violations detected by Axe.

---

# 28. Report Accessibility Violations

Instead of immediately asserting zero violations, you can inspect the results.

```javascript
const results =
  await new AxeBuilder({ page }).analyze();

console.log(results.violations);
```

Each violation can contain information such as:

* Rule ID
* Description
* Impact
* Affected HTML
* Help information

---

# 29. Scan a Specific Area

Axe can be limited to a particular portion of the page.

Example:

```javascript
const results =
  await new AxeBuilder({ page })
    .include('#main-content')
    .analyze();
```

This is useful when testing a specific component.

---

# 30. Excluding an Area

Example:

```javascript
const results =
  await new AxeBuilder({ page })
    .exclude('.third-party-widget')
    .analyze();
```

Use exclusions carefully.

A third-party component should not automatically be excluded just to make a test pass.

---

# 31. Accessibility Test for Login Page

```javascript
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test('login page accessibility', async ({ page }) => {
  await page.goto('/login');

  const results =
    await new AxeBuilder({ page }).analyze();

  expect(results.violations).toEqual([]);
});
```

---

# 32. Accessibility Test for a Component

```javascript
test('login form accessibility', async ({ page }) => {
  await page.goto('/login');

  const results =
    await new AxeBuilder({ page })
      .include('#login-form')
      .analyze();

  expect(results.violations).toEqual([]);
});
```

---

# 33. Testing Missing Form Labels

Bad HTML:

```html
<input type="text" id="username">
```

There is no associated label.

Better:

```html
<label for="username">Username</label>
<input type="text" id="username">
```

Playwright:

```javascript
await page.getByLabel('Username').fill('selva');
```

If the label relationship is missing, this locator can help expose the problem.

---

# 34. Testing ARIA Labels

HTML:

```html
<button aria-label="Open menu">
    ☰
</button>
```

Playwright:

```javascript
await page.getByRole('button', {
  name: 'Open menu'
}).click();
```

---

# 35. Testing ARIA Expanded

Example:

```html
<button
  aria-expanded="false"
  aria-controls="menu">
  Menu
</button>
```

After clicking:

```javascript
await page.getByRole('button', {
  name: 'Menu'
}).click();
```

You can validate the state:

```javascript
await expect(
  page.getByRole('button', { name: 'Menu' })
).toHaveAttribute('aria-expanded', 'true');
```

---

# 36. Testing ARIA Checked

Example:

```html
<div role="checkbox"
     aria-checked="true">
</div>
```

You can inspect the attribute:

```javascript
await expect(
  page.locator('[role="checkbox"]')
).toHaveAttribute('aria-checked', 'true');
```

Prefer native HTML controls whenever possible.

---

# 37. Testing Accessible Tabs

Example:

```javascript
const tab = page.getByRole('tab', {
  name: 'Profile'
});

await tab.click();

await expect(tab).toHaveAttribute(
  'aria-selected',
  'true'
);
```

---

# 38. Accessibility Testing with Test Fixtures

Accessibility scanning can be added to a reusable helper.

```javascript
async function checkAccessibility(page) {
  const results =
    await new AxeBuilder({ page }).analyze();

  expect(results.violations).toEqual([]);
}
```

Use it:

```javascript
test('home page accessibility', async ({ page }) => {
  await page.goto('/');

  await checkAccessibility(page);
});
```

---

# 39. Accessibility Helper Class

Example:

```javascript
import AxeBuilder from '@axe-core/playwright';

export async function runAccessibilityScan(page) {
  return await new AxeBuilder({ page }).analyze();
}
```

Test:

```javascript
const results = await runAccessibilityScan(page);

expect(results.violations).toEqual([]);
```

---

# 40. Accessibility Testing in Page Object Model

Example:

```javascript
import AxeBuilder from '@axe-core/playwright';

export class AccessibilityPage {
  constructor(page) {
    this.page = page;
  }

  async runAccessibilityScan() {
    return await new AxeBuilder({
      page: this.page
    }).analyze();
  }
}
```

Test:

```javascript
const accessibilityPage =
  new AccessibilityPage(page);

const results =
  await accessibilityPage.runAccessibilityScan();

expect(results.violations).toEqual([]);
```

---

# 41. Accessibility Test Strategy

A good accessibility automation strategy can include:

```text
             Accessibility Testing
                     |
       +-------------+-------------+
       |             |             |
   Locators      Keyboard       Axe Scan
       |             |             |
    ARIA Roles    Tab Order     WCAG Rules
    Labels        Focus         Violations
    Names         Enter         Impact
```

---

# 42. What Playwright Can Test

Playwright can help validate:

* Accessible roles
* Accessible names
* Form labels
* Keyboard interaction
* Focus
* Buttons
* Links
* Checkboxes
* Radio buttons
* Dialogs
* Alerts
* Headings
* Navigation
* Images
* ARIA states
* Accessibility rules through Axe integration

---

# 43. What Automated Accessibility Testing Cannot Fully Guarantee

Automated accessibility testing cannot guarantee that an application is completely accessible.

Human testing is still important for:

* Screen-reader usability
* Logical content flow
* Meaningful instructions
* Complex keyboard workflows
* Cognitive accessibility
* Real-world assistive technology behavior
* Overall usability

Automation should complement manual accessibility testing.

---

# 44. WCAG

WCAG stands for:

**Web Content Accessibility Guidelines**

Common principles are represented by POUR:

```text
P - Perceivable
O - Operable
U - Understandable
R - Robust
```

Accessibility testing should consider applicable WCAG requirements for the application.

---

# 45. Example End-to-End Accessibility Test

```javascript
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test.describe('Accessibility Tests', () => {

  test('login page should be accessible', async ({ page }) => {

    await page.goto('/login');

    // Verify heading
    await expect(
      page.getByRole('heading', {
        name: 'Login',
        level: 1
      })
    ).toBeVisible();

    // Verify username
    await expect(
      page.getByLabel('Username')
    ).toBeVisible();

    // Verify password
    await expect(
      page.getByLabel('Password')
    ).toBeVisible();

    // Verify login button
    await expect(
      page.getByRole('button', {
        name: 'Login'
      })
    ).toBeVisible();

    // Accessibility scan
    const results =
      await new AxeBuilder({ page }).analyze();

    expect(results.violations).toEqual([]);
  });

});
```

---

# 46. Best Practices

## Use semantic locators

Prefer:

```javascript
page.getByRole('button', { name: 'Save' })
```

over:

```javascript
page.locator('.save-button')
```

---

## Use labels for forms

Prefer:

```javascript
page.getByLabel('Email')
```

over:

```javascript
page.locator('#email')
```

---

## Test keyboard navigation

Verify important workflows can be completed using:

```text
Tab
Shift+Tab
Enter
Space
Escape
Arrow keys
```

where appropriate.

---

## Test focus

Verify that focus moves to the correct element after:

* Opening dialogs
* Closing dialogs
* Submitting forms
* Showing validation errors
* Navigating with the keyboard

---

## Run automated accessibility scans

Use Axe or another accessibility engine to catch common violations.

---

## Do not hide accessibility problems

Avoid excessive Axe exclusions.

If a violation is found:

1. Understand the violation.
2. Determine whether it is a real issue.
3. Fix the application.
4. Re-run the test.
5. Document legitimate exceptions.

---

# 47. Common Mistakes

### Mistake 1: Using only CSS selectors

```javascript
page.locator('#login')
```

Better:

```javascript
page.getByRole('button', {
  name: 'Login'
})
```

---

### Mistake 2: Ignoring keyboard navigation

A page can look correct visually while still being difficult to operate using a keyboard.

---

### Mistake 3: Missing form labels

Every user-facing form field should have an appropriate accessible name.

---

### Mistake 4: Using incorrect ARIA

ARIA should enhance semantic HTML rather than replace native controls unnecessarily.

---

### Mistake 5: Treating Axe as complete accessibility testing

Axe detects many automated accessibility issues, but it cannot replace manual accessibility testing.

---

# 48. Interview Questions

### Q1. What is accessibility testing?

Accessibility testing verifies that an application can be used effectively by people with disabilities and assistive technologies.

### Q2. How does Playwright support accessibility testing?

Through semantic locators such as `getByRole()`, `getByLabel()`, keyboard APIs, focus assertions, ARIA-aware locators, and integration with accessibility engines such as Axe.

### Q3. Why is getByRole() useful?

It identifies elements using their semantic role and accessible name, making tests closer to how users and assistive technologies interact with the application.

### Q4. What is an accessible name?

It is the name exposed to assistive technologies to identify an element.

### Q5. How do you test a button?

```javascript
await expect(
  page.getByRole('button', { name: 'Login' })
).toBeVisible();
```

### Q6. How do you test keyboard accessibility?

Use Playwright keyboard APIs:

```javascript
await page.keyboard.press('Tab');
await page.keyboard.press('Enter');
```

and verify focus and resulting behavior.

### Q7. How do you integrate Axe with Playwright?

Install:

```bash
npm install -D @axe-core/playwright
```

Then:

```javascript
const results =
  await new AxeBuilder({ page }).analyze();

expect(results.violations).toEqual([]);
```

### Q8. Can automated accessibility testing find every accessibility problem?

No. Automated tools detect many common issues, but manual testing and assistive-technology testing are still required.

### Q9. What is WCAG?

WCAG is the Web Content Accessibility Guidelines standard for making web content accessible.

### Q10. What are the four WCAG principles?

```text
Perceivable
Operable
Understandable
Robust
```

---

# 49. Quick Reference

| Requirement    | Playwright Approach     |
| -------------- | ----------------------- |
| Button         | `getByRole('button')`   |
| Link           | `getByRole('link')`     |
| Heading        | `getByRole('heading')`  |
| Form field     | `getByLabel()`          |
| Checkbox       | `getByRole('checkbox')` |
| Radio          | `getByRole('radio')`    |
| Dialog         | `getByRole('dialog')`   |
| Alert          | `getByRole('alert')`    |
| Image          | `getByAltText()`        |
| Keyboard       | `page.keyboard`         |
| Focus          | `toBeFocused()`         |
| ARIA state     | `toHaveAttribute()`     |
| Automated scan | `@axe-core/playwright`  |

---

# 50. Recommended Accessibility Test Checklist

* [ ] Verify meaningful page headings.
* [ ] Verify buttons have accessible names.
* [ ] Verify links have accessible names.
* [ ] Verify form fields have labels.
* [ ] Verify images have appropriate alternative text.
* [ ] Verify keyboard navigation.
* [ ] Verify focus behavior.
* [ ] Verify dialogs are accessible.
* [ ] Verify alerts are accessible.
* [ ] Verify important ARIA states.
* [ ] Run Axe accessibility scans.
* [ ] Review accessibility violations.
* [ ] Avoid unnecessary Axe exclusions.
* [ ] Perform manual keyboard testing.
* [ ] Perform screen-reader testing where required.
* [ ] Validate against the application's applicable WCAG requirements.

---

# Conclusion

Playwright provides strong accessibility-testing capabilities through semantic locators, ARIA-aware element identification, keyboard interactions, focus assertions, and integration with accessibility engines.

A strong Playwright accessibility strategy should combine:

```text
Playwright Semantic Locators
          +
Keyboard Testing
          +
Focus Testing
          +
ARIA Validation
          +
Axe Automated Scanning
          +
Manual Accessibility Testing
```

This combination helps create reliable automated tests while improving the accessibility and usability of the application.
