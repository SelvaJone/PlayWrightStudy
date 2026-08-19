# Playwright Visual Testing

## 1. Introduction

Playwright provides built-in support for **visual regression testing** through screenshot comparison.

Visual testing verifies that the application's UI looks the same as an approved baseline image.

It can detect unexpected changes such as:

* Layout changes
* Missing elements
* Incorrect colors
* Font changes
* Spacing changes
* Broken responsive layouts
* Unexpected CSS changes
* Incorrect images or icons
* Component rendering differences

The primary Playwright API used for visual testing is:

```javascript
await expect(page).toHaveScreenshot();
```

---

# 2. Why Visual Testing?

Traditional functional tests verify behavior:

```javascript
await expect(page.getByRole('button', { name: 'Login' })).toBeVisible();
```

This confirms that the button exists.

However, it does not verify:

* Button position
* Button size
* Font
* Color
* Padding
* Alignment
* Overall page appearance

Visual testing can detect these differences.

Example:

```javascript
await expect(page).toHaveScreenshot();
```

Playwright compares the current screenshot against a stored baseline.

---

# 3. Visual Regression Testing

Visual regression testing follows this process:

```text
Run Test
   |
   v
Open Application
   |
   v
Capture Screenshot
   |
   v
Compare With Baseline
   |
   +---- Match ----> PASS
   |
   +---- Difference -> FAIL
```

Example:

```javascript
test('homepage visual test', async ({ page }) => {
  await page.goto('https://example.com');

  await expect(page).toHaveScreenshot();
});
```

On the first execution, Playwright creates a baseline screenshot.

On subsequent executions, Playwright compares the new screenshot with the baseline.

---

# 4. Basic Screenshot Assertion

```javascript
import { test, expect } from '@playwright/test';

test('homepage visual test', async ({ page }) => {
  await page.goto('https://example.com');

  await expect(page).toHaveScreenshot();
});
```

The screenshot becomes the expected visual state.

---

# 5. Naming Screenshots

You can provide a screenshot name.

```javascript
await expect(page).toHaveScreenshot('homepage.png');
```

Example:

```javascript
test('homepage screenshot', async ({ page }) => {
  await page.goto('https://example.com');

  await expect(page).toHaveScreenshot('homepage.png');
});
```

This makes the snapshot easier to identify.

---

# 6. Screenshot Folder Structure

Playwright stores screenshots in snapshot directories.

A typical project may look like:

```text
playwright-project/
│
├── tests/
│   └── visual.spec.js
│
├── tests/visual.spec.js-snapshots/
│   └── homepage-chromium-linux.png
│
├── playwright.config.js
├── package.json
└── node_modules/
```

The exact snapshot path and filename can vary based on:

* Test file
* Project
* Browser
* Operating system
* Screenshot name

---

# 7. Full Page Screenshot

You can capture the entire page.

```javascript
await expect(page).toHaveScreenshot({
  fullPage: true
});
```

Example:

```javascript
test('full page visual test', async ({ page }) => {
  await page.goto('https://example.com');

  await expect(page).toHaveScreenshot({
    fullPage: true
  });
});
```

This is useful when the page extends beyond the viewport.

---

# 8. Element Screenshot

Visual testing does not have to cover the entire page.

You can compare an individual element.

```javascript
const loginForm = page.locator('#login-form');

await expect(loginForm).toHaveScreenshot();
```

Example:

```javascript
test('login form visual test', async ({ page }) => {
  await page.goto('https://example.com/login');

  const loginForm = page.locator('#login-form');

  await expect(loginForm).toHaveScreenshot();
});
```

This is particularly useful for component-level visual testing.

---

# 9. Screenshot With a Specific Name

```javascript
const loginForm = page.locator('#login-form');

await expect(loginForm).toHaveScreenshot('login-form.png');
```

This creates a clearly named baseline.

---

# 10. Screenshot Options

Playwright supports screenshot comparison options.

Example:

```javascript
await expect(page).toHaveScreenshot({
  fullPage: true,
  animations: 'disabled'
});
```

Common options include:

* `fullPage`
* `animations`
* `caret`
* `clip`
* `mask`
* `maskColor`
* `scale`
* `stylePath`
* `threshold`
* `maxDiffPixels`
* `maxDiffPixelRatio`

---

# 11. Disabling Animations

Animations can cause visual test instability.

Example:

```javascript
await expect(page).toHaveScreenshot({
  animations: 'disabled'
});
```

This is useful when elements are:

* Moving
* Fading
* Sliding
* Expanding
* Rotating

Example:

```javascript
test('stable visual test', async ({ page }) => {
  await page.goto('https://example.com');

  await expect(page).toHaveScreenshot({
    animations: 'disabled'
  });
});
```

---

# 12. Hiding the Caret

Text input fields can display a blinking cursor.

You can control the caret:

```javascript
await expect(page).toHaveScreenshot({
  caret: 'hide'
});
```

Example:

```javascript
await expect(page).toHaveScreenshot({
  caret: 'hide'
});
```

This helps prevent unnecessary visual differences.

---

# 13. Masking Dynamic Elements

Dynamic content is one of the biggest challenges in visual testing.

Examples:

* Current date
* Time
* Random numbers
* User names
* Advertisements
* Live counters
* Transaction IDs

You can mask an element.

```javascript
const timestamp = page.locator('.timestamp');

await expect(page).toHaveScreenshot({
  mask: [timestamp]
});
```

Example:

```javascript
test('mask dynamic content', async ({ page }) => {
  await page.goto('https://example.com');

  const timestamp = page.locator('.timestamp');

  await expect(page).toHaveScreenshot({
    mask: [timestamp]
  });
});
```

---

# 14. Mask Multiple Elements

You can mask multiple locators.

```javascript
const date = page.locator('.date');
const username = page.locator('.username');

await expect(page).toHaveScreenshot({
  mask: [date, username]
});
```

This prevents dynamic content from causing false failures.

---

# 15. Mask Color

You can customize the mask color.

```javascript
await expect(page).toHaveScreenshot({
  mask: [page.locator('.timestamp')],
  maskColor: '#FF00FF'
});
```

The default masking behavior can be customized using `maskColor`.

---

# 16. Pixel Difference

Visual comparison is based on pixel differences.

Even a small rendering difference can cause a failure.

For example:

```text
Baseline Screenshot
        |
        v
Pixel Comparison
        |
        v
Current Screenshot
```

Playwright calculates the visual differences and determines whether the difference is acceptable.

---

# 17. maxDiffPixels

You can allow a specific number of different pixels.

```javascript
await expect(page).toHaveScreenshot({
  maxDiffPixels: 100
});
```

This means a small number of pixel differences is acceptable.

Example:

```javascript
test('visual test with pixel tolerance', async ({ page }) => {
  await page.goto('https://example.com');

  await expect(page).toHaveScreenshot({
    maxDiffPixels: 100
  });
});
```

Use this carefully.

A large value can hide real UI defects.

---

# 18. maxDiffPixelRatio

You can also specify the allowed percentage of differing pixels.

```javascript
await expect(page).toHaveScreenshot({
  maxDiffPixelRatio: 0.01
});
```

For example:

```text
0.01 = approximately 1%
```

This can be useful when screenshot dimensions vary.

---

# 19. threshold

You can configure the color difference threshold.

```javascript
await expect(page).toHaveScreenshot({
  threshold: 0.2
});
```

The threshold controls how much individual pixel color variation is tolerated.

Do not increase the threshold unnecessarily.

---

# 20. Combining Visual Options

Example:

```javascript
await expect(page).toHaveScreenshot({
  fullPage: true,
  animations: 'disabled',
  caret: 'hide',
  maxDiffPixels: 100,
  threshold: 0.2
});
```

---

# 21. Using a Locator

Component-level visual testing is often more maintainable than full-page visual testing.

Example:

```javascript
test('product card visual test', async ({ page }) => {
  await page.goto('https://example.com/products');

  const productCard = page.locator('.product-card').first();

  await expect(productCard).toHaveScreenshot('product-card.png');
});
```

This focuses the test on a specific UI component.

---

# 22. Multiple Component Screenshots

You can test multiple components.

```javascript
test('components visual test', async ({ page }) => {
  await page.goto('https://example.com');

  await expect(
    page.locator('.header')
  ).toHaveScreenshot('header.png');

  await expect(
    page.locator('.navigation')
  ).toHaveScreenshot('navigation.png');

  await expect(
    page.locator('.footer')
  ).toHaveScreenshot('footer.png');
});
```

---

# 23. Visual Testing With Page States

Visual testing is especially useful for different application states.

Examples:

```text
Login Page
Home Page
Empty Cart
Cart With Items
Error Page
Success Page
Modal Open
Dropdown Open
Mobile Menu Open
```

Example:

```javascript
test('shopping cart visual test', async ({ page }) => {
  await page.goto('https://example.com/cart');

  await expect(page).toHaveScreenshot('empty-cart.png');
});
```

---

# 24. Visual Testing After User Interaction

Visual assertions can be performed after actions.

```javascript
test('menu visual test', async ({ page }) => {
  await page.goto('https://example.com');

  await page.getByRole('button', { name: 'Menu' }).click();

  await expect(page).toHaveScreenshot('menu-open.png');
});
```

This verifies the visual state after interaction.

---

# 25. Visual Testing for Modals

```javascript
test('modal visual test', async ({ page }) => {
  await page.goto('https://example.com');

  await page.getByRole('button', { name: 'Open Modal' }).click();

  const modal = page.locator('[role="dialog"]');

  await expect(modal).toHaveScreenshot('modal.png');
});
```

This is an excellent use case for component-level visual testing.

---

# 26. Visual Testing for Dropdowns

```javascript
test('dropdown visual test', async ({ page }) => {
  await page.goto('https://example.com');

  await page.getByRole('button', { name: 'Options' }).click();

  await expect(page).toHaveScreenshot('dropdown-open.png');
});
```

---

# 27. Visual Testing for Error Messages

```javascript
test('error message visual test', async ({ page }) => {
  await page.goto('https://example.com/login');

  await page.getByRole('button', { name: 'Login' }).click();

  const errorMessage = page.locator('.error-message');

  await expect(errorMessage).toHaveScreenshot('login-error.png');
});
```

---

# 28. Visual Testing for Responsive Design

Playwright projects can define multiple browser/device projects.

Example:

```javascript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  projects: [
    {
      name: 'chromium-desktop',
      use: {
        ...devices['Desktop Chrome']
      }
    },
    {
      name: 'mobile-chrome',
      use: {
        ...devices['Pixel 5']
      }
    }
  ]
});
```

The same visual test can then run against different environments.

```javascript
test('responsive visual test', async ({ page }) => {
  await page.goto('https://example.com');

  await expect(page).toHaveScreenshot('responsive-page.png');
});
```

---

# 29. Browser-Specific Snapshots

Visual output can differ between browsers.

For example:

```text
Chromium
Firefox
WebKit
```

Playwright can maintain separate snapshots for different projects.

This is important because rendering engines can produce small differences.

---

# 30. Operating System Differences

Visual tests can also differ between operating systems because of:

* Fonts
* Font rendering
* Anti-aliasing
* Browser versions
* System libraries

For reliable visual testing, run baselines and comparisons in a consistent environment.

A common CI strategy is:

```text
Developer
   |
   v
Create/Update Baseline
   |
   v
CI Environment
   |
   v
Run Visual Tests
   |
   v
Compare Against Baseline
```

---

# 31. Updating Baselines

When an intentional UI change occurs, update the screenshot baseline.

Example command:

```bash
npx playwright test --update-snapshots
```

Short form:

```bash
npx playwright test -u
```

Use this only when the UI change is expected.

Do not blindly update snapshots after every failure.

---

# 32. Important Rule for Updating Snapshots

When a visual test fails:

```text
Visual Failure
     |
     v
Investigate Difference
     |
     +---- Unexpected -> Fix Application
     |
     +---- Expected ----> Update Baseline
```

Never immediately run:

```bash
npx playwright test -u
```

without investigating the difference.

Otherwise, you may accidentally approve a real defect.

---

# 33. Running Visual Tests

Run all tests:

```bash
npx playwright test
```

Run a specific test:

```bash
npx playwright test visual.spec.js
```

Run headed:

```bash
npx playwright test --headed
```

Run with a specific project:

```bash
npx playwright test --project=chromium
```

---

# 34. Debugging Visual Failures

When a visual test fails, Playwright provides information about the difference.

Typically you can inspect:

```text
Expected
Actual
Diff
```

Conceptually:

```text
Expected Screenshot
        |
        v
Current Screenshot
        |
        v
Difference Image
```

The difference image helps identify the changed area.

---

# 35. Example Visual Test Project

Recommended structure:

```text
playwright-project/
│
├── tests/
│   ├── functional/
│   │   └── login.spec.js
│   │
│   └── visual/
│       └── visual.spec.js
│
├── tests/visual/visual.spec.js-snapshots/
│   ├── homepage-chromium-linux.png
│   ├── login-form-chromium-linux.png
│   └── menu-open-chromium-linux.png
│
├── playwright.config.js
├── package.json
└── package-lock.json
```

---

# 36. Complete Visual Testing Example

```javascript
import { test, expect } from '@playwright/test';

test.describe('Visual Regression Tests', () => {

  test('homepage visual test', async ({ page }) => {
    await page.goto('https://example.com');

    await expect(page).toHaveScreenshot('homepage.png', {
      fullPage: true,
      animations: 'disabled',
      caret: 'hide'
    });
  });

  test('login form visual test', async ({ page }) => {
    await page.goto('https://example.com/login');

    const loginForm = page.locator('#login-form');

    await expect(loginForm).toHaveScreenshot('login-form.png');
  });

  test('menu visual test', async ({ page }) => {
    await page.goto('https://example.com');

    await page.getByRole('button', { name: 'Menu' }).click();

    await expect(page).toHaveScreenshot('menu-open.png');
  });

});
```

---

# 37. Handling Dynamic Dates

Suppose the application displays:

```text
August 19, 2026
```

The value changes every day.

Instead of allowing the changing date to affect the snapshot, mask it.

```javascript
const date = page.locator('.current-date');

await expect(page).toHaveScreenshot({
  mask: [date]
});
```

---

# 38. Handling Dynamic User Data

```javascript
const username = page.locator('.username');

await expect(page).toHaveScreenshot({
  mask: [username]
});
```

This prevents user-specific data from causing failures.

---

# 39. Handling Animations

Bad visual test:

```javascript
await expect(page).toHaveScreenshot();
```

when the page contains active animations.

Better:

```javascript
await expect(page).toHaveScreenshot({
  animations: 'disabled'
});
```

---

# 40. Handling Random Data

Suppose a page contains:

```text
Order ID: 83746291
```

If the ID changes every run, the screenshot can fail.

Possible solution:

```javascript
const orderId = page.locator('.order-id');

await expect(page).toHaveScreenshot({
  mask: [orderId]
});
```

---

# 41. Visual Testing Best Practices

## 41.1 Keep Visual Tests Focused

Prefer:

```javascript
await expect(page.locator('.login-form'))
  .toHaveScreenshot();
```

when you only need to verify a component.

---

## 41.2 Avoid Excessive Full-Page Snapshots

Full-page screenshots can become difficult to maintain.

Use them for important page-level validation.

---

## 41.3 Stabilize Dynamic Content

Mask or otherwise stabilize:

* Dates
* Times
* Random values
* User-specific content
* Animations
* Ads
* Live data

---

## 41.4 Use Consistent Environments

Keep the following consistent where possible:

* Browser
* Browser version
* Operating system
* Fonts
* Viewport
* Device configuration

---

## 41.5 Review Snapshot Changes

Do not automatically approve every screenshot change.

Ask:

```text
Was the UI change intentional?
```

If yes:

```text
Update baseline.
```

If no:

```text
Fix the application.
```

---

# 42. Visual Testing in CI/CD

Visual testing works well in CI/CD pipelines.

Example:

```text
Developer Push
      |
      v
CI Pipeline
      |
      v
Install Dependencies
      |
      v
Install Playwright Browsers
      |
      v
Run Tests
      |
      v
Visual Comparison
      |
   +--+--+
   |     |
 PASS   FAIL
   |     |
   v     v
 Build  Investigate
```

Example CI command:

```bash
npx playwright test
```

---

# 43. Git and Visual Snapshots

Baseline screenshots should normally be stored in source control.

Example:

```text
tests/
└── visual.spec.js-snapshots/
    ├── homepage-chromium-linux.png
    ├── login-chromium-linux.png
    └── menu-chromium-linux.png
```

When the UI intentionally changes:

```bash
npx playwright test -u
```

Then review the changed image files before committing them.

---

# 44. Visual Testing and Git Pull Requests

A good workflow is:

```text
Developer Changes UI
        |
        v
Run Visual Tests
        |
        v
Review Differences
        |
        v
Update Baselines if Expected
        |
        v
Commit Code + Snapshots
        |
        v
Create Pull Request
        |
        v
CI Runs Visual Tests
```

This helps catch accidental UI regressions before production.

---

# 45. Visual Testing vs Functional Testing

| Testing Type          | Purpose                              |
| --------------------- | ------------------------------------ |
| Functional Testing    | Verifies behavior                    |
| API Testing           | Verifies backend/API behavior        |
| Accessibility Testing | Verifies accessibility rules         |
| Visual Testing        | Verifies UI appearance               |
| Performance Testing   | Verifies speed and resource behavior |

Visual testing should complement functional testing rather than replace it.

---

# 46. Visual Testing vs Screenshot Capture

These are different concepts.

### Screenshot Capture

```javascript
await page.screenshot({
  path: 'homepage.png'
});
```

This simply creates an image.

### Visual Assertion

```javascript
await expect(page).toHaveScreenshot();
```

This captures and compares against a baseline.

Therefore:

```text
page.screenshot()
        |
        v
Capture Image
```

while:

```text
toHaveScreenshot()
        |
        v
Capture + Compare + Assert
```

---

# 47. Common Mistakes

## Mistake 1: Updating snapshots without investigation

Bad:

```bash
npx playwright test -u
```

immediately after every failure.

Correct approach:

```text
Failure
  |
  v
Review Diff
  |
  v
Determine Cause
  |
  +--> Bug
  |
  +--> Expected UI Change
```

---

## Mistake 2: Ignoring dynamic content

Dynamic content can create unstable tests.

Use:

```javascript
mask: [locator]
```

when appropriate.

---

## Mistake 3: Using large tolerances

Avoid:

```javascript
maxDiffPixels: 10000
```

unless you have a specific reason.

Large tolerances can hide genuine regressions.

---

## Mistake 4: Different environments for baseline and execution

A screenshot created on one environment may not perfectly match another.

Keep the visual testing environment consistent.

---

## Mistake 5: Testing everything visually

Not every element needs a screenshot assertion.

Focus on:

* Critical pages
* Important components
* Major user flows
* High-risk UI areas
* Responsive layouts

---

# 48. Advanced Visual Testing Strategy

A mature automation framework can use multiple levels.

```text
Level 1
Component Visual Testing
        |
        v
Level 2
Page Visual Testing
        |
        v
Level 3
Responsive Visual Testing
        |
        v
Level 4
Cross-Browser Visual Testing
        |
        v
Level 5
CI/CD Visual Regression
```

---

# 49. Recommended Visual Test Categories

A large application can organize visual tests as:

```text
tests/
└── visual/
    ├── login/
    │   └── login-visual.spec.js
    │
    ├── dashboard/
    │   └── dashboard-visual.spec.js
    │
    ├── navigation/
    │   └── navigation-visual.spec.js
    │
    ├── components/
    │   ├── button-visual.spec.js
    │   ├── modal-visual.spec.js
    │   └── card-visual.spec.js
    │
    └── responsive/
        └── responsive-visual.spec.js
```

---

# 50. Interview Questions

## Q1. What is visual regression testing?

Visual regression testing compares the current UI against an approved baseline screenshot to detect unintended visual changes.

---

## Q2. How do you perform visual testing in Playwright?

Use:

```javascript
await expect(page).toHaveScreenshot();
```

---

## Q3. How do you test a specific element visually?

Use a locator:

```javascript
await expect(
  page.locator('.login-form')
).toHaveScreenshot();
```

---

## Q4. How do you capture the entire page?

```javascript
await expect(page).toHaveScreenshot({
  fullPage: true
});
```

---

## Q5. How do you handle dynamic content?

Use masking or stabilize the content.

Example:

```javascript
await expect(page).toHaveScreenshot({
  mask: [page.locator('.timestamp')]
});
```

---

## Q6. How do you disable animations?

```javascript
await expect(page).toHaveScreenshot({
  animations: 'disabled'
});
```

---

## Q7. How do you update screenshots?

```bash
npx playwright test -u
```

or:

```bash
npx playwright test --update-snapshots
```

---

## Q8. Should you update snapshots whenever a test fails?

No.

First determine whether the difference is:

* An actual defect
* An expected UI change
* An environment issue
* Dynamic content
* A test synchronization problem

Only expected changes should result in updated baselines.

---

## Q9. What is the difference between `page.screenshot()` and `toHaveScreenshot()`?

`page.screenshot()` captures an image.

`toHaveScreenshot()` captures and compares the image against a baseline and creates an assertion.

---

## Q10. What is `maxDiffPixels`?

It specifies the maximum number of pixels that may differ before the visual assertion fails.

Example:

```javascript
await expect(page).toHaveScreenshot({
  maxDiffPixels: 100
});
```

---

## Q11. What is `maxDiffPixelRatio`?

It defines the allowed ratio of different pixels.

Example:

```javascript
await expect(page).toHaveScreenshot({
  maxDiffPixelRatio: 0.01
});
```

---

## Q12. What is `threshold`?

It controls the acceptable pixel-level color difference during screenshot comparison.

Example:

```javascript
await expect(page).toHaveScreenshot({
  threshold: 0.2
});
```

---

## Q13. Why can visual tests be flaky?

Common reasons include:

* Animations
* Dynamic data
* Different fonts
* Different operating systems
* Different browser versions
* Network-dependent content
* Random data
* Time-dependent content

---

## Q14. How can you reduce visual test flakiness?

Use:

* Stable test data
* Consistent environments
* Disabled animations
* Masked dynamic elements
* Stable viewport sizes
* Appropriate synchronization
* Reasonable screenshot tolerances

---

## Q15. Should visual tests replace functional tests?

No.

They complement functional tests.

Functional tests verify behavior, while visual tests verify appearance.

---

# 51. Senior-Level Interview Scenario

### Question

A visual test fails in CI but passes locally. How would you investigate?

### Answer

I would first compare the environments.

I would check:

1. Browser version
2. Operating system
3. Playwright version
4. Fonts
5. Viewport size
6. Device configuration
7. Screenshot baseline
8. Dynamic content
9. Animations
10. Network-dependent content

I would inspect the expected, actual, and diff screenshots.

If the difference is caused by environment inconsistency, I would standardize the environment.

If it is caused by dynamic content, I would stabilize or mask it.

If it is an actual UI regression, I would fix the application rather than update the snapshot.

---

# 52. Recommended Enterprise Approach

For an enterprise Playwright framework:

```text
Visual Testing
│
├── Component Screenshots
│
├── Page Screenshots
│
├── Responsive Screenshots
│
├── Cross-Browser Screenshots
│
├── Dynamic Content Handling
│
├── Snapshot Management
│
├── CI/CD Integration
│
└── Pull Request Review
```

Recommended principles:

1. Keep baselines under version control.
2. Keep test environments consistent.
3. Mask dynamic content.
4. Disable unnecessary animations.
5. Keep tolerances small and justified.
6. Review screenshot changes.
7. Prefer focused component screenshots where appropriate.
8. Use full-page screenshots for important page-level validation.
9. Run visual tests in CI.
10. Never blindly update snapshots.

---

# 53. Quick Reference

### Basic visual assertion

```javascript
await expect(page).toHaveScreenshot();
```

### Named screenshot

```javascript
await expect(page).toHaveScreenshot('homepage.png');
```

### Full page

```javascript
await expect(page).toHaveScreenshot({
  fullPage: true
});
```

### Element screenshot

```javascript
await expect(
  page.locator('.login-form')
).toHaveScreenshot();
```

### Disable animations

```javascript
await expect(page).toHaveScreenshot({
  animations: 'disabled'
});
```

### Hide caret

```javascript
await expect(page).toHaveScreenshot({
  caret: 'hide'
});
```

### Mask dynamic content

```javascript
await expect(page).toHaveScreenshot({
  mask: [page.locator('.timestamp')]
});
```

### Pixel tolerance

```javascript
await expect(page).toHaveScreenshot({
  maxDiffPixels: 100
});
```

### Pixel ratio tolerance

```javascript
await expect(page).toHaveScreenshot({
  maxDiffPixelRatio: 0.01
});
```

### Update snapshots

```bash
npx playwright test -u
```

### Run a specific test

```bash
npx playwright test visual.spec.js
```

---

# 54. Final Summary

Playwright visual testing provides a powerful way to detect unintended UI changes.

The most important API is:

```javascript
await expect(page).toHaveScreenshot();
```

Remember the core workflow:

```text
Create Baseline
      |
      v
Run Test
      |
      v
Capture Current Screenshot
      |
      v
Compare With Baseline
      |
   +--+--+
   |     |
 Match  Difference
   |     |
 PASS   Investigate
         |
      +--+--+
      |     |
     Bug  Expected
      |     |
     Fix  Update
```

The key skills for real-world Playwright visual testing are:

* Screenshot assertions
* Element screenshots
* Full-page screenshots
* Snapshot management
* Dynamic content masking
* Animation handling
* Pixel tolerance
* Responsive visual testing
* Cross-browser testing
* CI/CD integration
* Visual failure analysis
* Baseline maintenance

Visual testing should be treated as an additional quality layer on top of functional, API, accessibility, and integration testing.
