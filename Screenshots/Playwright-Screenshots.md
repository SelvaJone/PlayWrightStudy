# Playwright Screenshots

## Overview

Playwright provides built-in APIs for capturing screenshots of web pages and individual elements.

Screenshots are useful for:

* Debugging failed tests
* Visual validation
* Test evidence
* CI/CD artifacts
* Regression testing
* Capturing full-page screenshots
* Capturing specific elements
* Comparing screenshots
* Documenting failures

---

## 1. Basic Screenshot

```javascript
import { test } from '@playwright/test';

test('capture screenshot', async ({ page }) => {
  await page.goto('https://example.com');

  await page.screenshot({
    path: 'screenshots/homepage.png'
  });
});
```

The screenshot is saved to:

```text
screenshots/homepage.png
```

---

## 2. Screenshot with a Simple File Name

```javascript
await page.screenshot({
  path: 'homepage.png'
});
```

---

## 3. Screenshot of the Entire Page

By default, Playwright captures the visible viewport.

To capture the entire scrollable page:

```javascript
await page.screenshot({
  path: 'screenshots/full-page.png',
  fullPage: true
});
```

Example:

```javascript
import { test } from '@playwright/test';

test('full page screenshot', async ({ page }) => {
  await page.goto('https://example.com');

  await page.screenshot({
    path: 'screenshots/full-page.png',
    fullPage: true
  });
});
```

---

## 4. Screenshot of a Specific Element

Use a locator's `screenshot()` method.

```javascript
const logo = page.locator('.logo');

await logo.screenshot({
  path: 'screenshots/logo.png'
});
```

Complete example:

```javascript
import { test } from '@playwright/test';

test('element screenshot', async ({ page }) => {
  await page.goto('https://example.com');

  const header = page.locator('header');

  await header.screenshot({
    path: 'screenshots/header.png'
  });
});
```

---

## 5. Screenshot of a Button

```javascript
const button = page.getByRole('button', {
  name: 'Login'
});

await button.screenshot({
  path: 'screenshots/login-button.png'
});
```

---

## 6. Screenshot After an Action

Screenshots can be captured after performing an action.

```javascript
await page.getByRole('button', {
  name: 'Login'
}).click();

await page.screenshot({
  path: 'screenshots/after-login.png'
});
```

---

## 7. Screenshot After Filling a Form

```javascript
await page.getByLabel('Username').fill('selva');

await page.getByLabel('Password').fill('password123');

await page.screenshot({
  path: 'screenshots/login-form-filled.png'
});
```

---

## 8. Screenshot Before and After an Action

```javascript
await page.screenshot({
  path: 'screenshots/before-click.png'
});

await page.getByRole('button', {
  name: 'Submit'
}).click();

await page.screenshot({
  path: 'screenshots/after-click.png'
});
```

This is useful for debugging UI behavior.

---

# 9. Screenshot with Custom Image Type

Playwright supports PNG and JPEG screenshots.

### PNG

```javascript
await page.screenshot({
  path: 'screenshots/page.png',
  type: 'png'
});
```

### JPEG

```javascript
await page.screenshot({
  path: 'screenshots/page.jpg',
  type: 'jpeg'
});
```

---

# 10. JPEG Quality

JPEG screenshots can specify a quality level.

```javascript
await page.screenshot({
  path: 'screenshots/page.jpg',
  type: 'jpeg',
  quality: 80
});
```

Quality must be between:

```text
0 - 100
```

PNG does not support the `quality` option.

---

# 11. Screenshot with Omitted Background

Playwright can capture screenshots with a transparent background for supported page content.

```javascript
await page.screenshot({
  path: 'screenshots/page.png',
  omitBackground: true
});
```

This is particularly useful for certain UI components and visual testing scenarios.

---

# 12. Screenshot After Waiting for a Page

Always make sure the required page state has been reached before capturing the screenshot.

```javascript
await page.goto('https://example.com');

await page.waitForLoadState('networkidle');

await page.screenshot({
  path: 'screenshots/page.png'
});
```

For application-specific states, prefer waiting for the relevant element:

```javascript
await page.getByRole('heading', {
  name: 'Dashboard'
}).waitFor();

await page.screenshot({
  path: 'screenshots/dashboard.png'
});
```

---

# 13. Screenshot with Specific Viewport

Viewport size can be configured in the test.

```javascript
import { test } from '@playwright/test';

test.use({
  viewport: {
    width: 1280,
    height: 720
  }
});

test('desktop screenshot', async ({ page }) => {
  await page.goto('https://example.com');

  await page.screenshot({
    path: 'screenshots/desktop.png'
  });
});
```

---

# 14. Mobile Screenshot

```javascript
import { test, devices } from '@playwright/test';

test.use({
  ...devices['iPhone 13']
});

test('mobile screenshot', async ({ page }) => {
  await page.goto('https://example.com');

  await page.screenshot({
    path: 'screenshots/mobile.png',
    fullPage: true
  });
});
```

---

# 15. Screenshot of a Specific Area

The `clip` option allows you to capture a specific rectangular region.

```javascript
await page.screenshot({
  path: 'screenshots/area.png',
  clip: {
    x: 100,
    y: 100,
    width: 500,
    height: 300
  }
});
```

The coordinates are measured in CSS pixels.

---

# 16. Screenshot of a Locator vs Clip

### Locator screenshot

Preferred when you want an actual UI element:

```javascript
await page.locator('.profile-card').screenshot({
  path: 'screenshots/profile-card.png'
});
```

### Clip screenshot

Useful for capturing an arbitrary rectangular area:

```javascript
await page.screenshot({
  path: 'screenshots/area.png',
  clip: {
    x: 50,
    y: 100,
    width: 600,
    height: 400
  }
});
```

Generally, prefer locator screenshots when the target is a known element.

---

# 17. Screenshot Animations

Animations can make screenshots inconsistent.

For visual testing, animations can be disabled.

```javascript
await page.screenshot({
  path: 'screenshots/page.png',
  animations: 'disabled'
});
```

This helps make screenshots more deterministic.

---

# 18. Screenshot with Caret Hidden

For text inputs, the blinking cursor can create inconsistent screenshots.

```javascript
await page.screenshot({
  path: 'screenshots/form.png',
  caret: 'hide'
});
```

This is useful for visual regression testing.

---

# 19. Mask Dynamic Elements

Playwright supports masking elements during screenshot capture.

```javascript
await page.screenshot({
  path: 'screenshots/dashboard.png',
  mask: [
    page.locator('.timestamp'),
    page.locator('.user-id')
  ]
});
```

Dynamic elements such as:

* Timestamps
* Random IDs
* User-specific data
* Live counters
* Random advertisements

can otherwise cause visual differences.

---

# 20. Mask a Specific Element

```javascript
const timestamp = page.locator('.timestamp');

await page.screenshot({
  path: 'screenshots/dashboard.png',
  mask: [timestamp]
});
```

---

# 21. Multiple Masked Elements

```javascript
await page.screenshot({
  path: 'screenshots/dashboard.png',
  mask: [
    page.locator('.timestamp'),
    page.locator('.notification-count'),
    page.locator('.random-value')
  ]
});
```

---

# 22. Mask Color

A mask can be customized with a color.

```javascript
await page.screenshot({
  path: 'screenshots/dashboard.png',
  mask: [
    page.locator('.timestamp')
  ],
  maskColor: '#FF00FF'
});
```

---

# 23. Screenshot with Test Information

A useful approach is to generate a unique screenshot filename.

```javascript
import { test } from '@playwright/test';

test('login test', async ({ page }, testInfo) => {
  await page.goto('https://example.com');

  await page.screenshot({
    path: testInfo.outputPath('login.png')
  });
});
```

Using `testInfo.outputPath()` is useful because Playwright manages the test output directory.

---

# 24. Screenshot on Test Failure

Instead of manually capturing screenshots everywhere, configure Playwright to capture screenshots when tests fail.

In `playwright.config.js`:

```javascript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  use: {
    screenshot: 'only-on-failure'
  }
});
```

Possible values include:

```text
'off'
'on'
'only-on-failure'
```

---

# 25. Screenshot for Every Test

```javascript
export default defineConfig({
  use: {
    screenshot: 'on'
  }
});
```

This captures screenshots during tests according to Playwright's configured screenshot behavior.

However, capturing screenshots for every test can increase artifact size.

For most CI pipelines:

```javascript
screenshot: 'only-on-failure'
```

is a practical choice.

---

# 26. Disable Automatic Screenshots

```javascript
export default defineConfig({
  use: {
    screenshot: 'off'
  }
});
```

---

# 27. Recommended CI Configuration

A common configuration is:

```javascript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  use: {
    screenshot: 'only-on-failure',
    trace: 'retain-on-failure',
    video: 'retain-on-failure'
  }
});
```

This provides useful debugging artifacts when a test fails.

---

# 28. Screenshot with Trace

Screenshots and traces work well together.

```javascript
export default defineConfig({
  use: {
    trace: 'retain-on-failure',
    screenshot: 'only-on-failure'
  }
});
```

When a test fails, you can inspect the trace and screenshot to understand what happened.

---

# 29. Screenshot During a Test

```javascript
test('checkout flow', async ({ page }) => {
  await page.goto('https://example.com');

  await page.screenshot({
    path: 'screenshots/checkout-start.png'
  });

  await page.getByRole('button', {
    name: 'Add to Cart'
  }).click();

  await page.screenshot({
    path: 'screenshots/cart.png'
  });

  await page.getByRole('button', {
    name: 'Checkout'
  }).click();

  await page.screenshot({
    path: 'screenshots/checkout.png'
  });
});
```

---

# 30. Screenshot in a Page Object Model

A Page Object can expose a screenshot method.

```javascript
export class LoginPage {
  constructor(page) {
    this.page = page;
    this.username = page.getByLabel('Username');
    this.password = page.getByLabel('Password');
    this.loginButton = page.getByRole('button', {
      name: 'Login'
    });
  }

  async captureScreenshot(path) {
    await this.page.screenshot({
      path
    });
  }

  async login(username, password) {
    await this.username.fill(username);
    await this.password.fill(password);
    await this.loginButton.click();
  }
}
```

Test:

```javascript
import { test } from '@playwright/test';
import { LoginPage } from './pages/LoginPage';

test('login screenshot', async ({ page }) => {
  const loginPage = new LoginPage(page);

  await page.goto('https://example.com/login');

  await loginPage.captureScreenshot(
    'screenshots/login-page.png'
  );
});
```

---

# 31. Element Screenshot in Page Object

```javascript
export class DashboardPage {
  constructor(page) {
    this.page = page;
    this.dashboard = page.locator('.dashboard');
  }

  async captureDashboard(path) {
    await this.dashboard.screenshot({
      path
    });
  }
}
```

---

# 32. Screenshot Naming Convention

A consistent naming convention makes test artifacts easier to understand.

Example:

```text
screenshots/
├── login-before.png
├── login-after.png
├── dashboard.png
├── profile.png
├── checkout.png
└── order-confirmation.png
```

For larger projects:

```text
screenshots/
├── login/
│   ├── login-page.png
│   └── login-error.png
├── checkout/
│   ├── cart.png
│   ├── payment.png
│   └── confirmation.png
└── profile/
    └── profile-page.png
```

---

# 33. Screenshot Naming with Test Data

```javascript
const username = 'selva';

await page.screenshot({
  path: `screenshots/${username}-dashboard.png`
});
```

For CI and parallel execution, avoid filenames that can collide.

A safer approach is:

```javascript
await page.screenshot({
  path: testInfo.outputPath('dashboard.png')
});
```

---

# 34. Screenshot After an Assertion

```javascript
await expect(
  page.getByRole('heading', {
    name: 'Dashboard'
  })
).toBeVisible();

await page.screenshot({
  path: 'screenshots/dashboard-verified.png'
});
```

---

# 35. Screenshot Before an Assertion

```javascript
await page.screenshot({
  path: 'screenshots/before-assertion.png'
});

await expect(
  page.getByText('Success')
).toBeVisible();
```

For most tests, screenshots are more useful after meaningful state changes or when diagnosing failures.

---

# 36. Screenshot and Visual Regression

Playwright supports screenshot assertions through `toHaveScreenshot()`.

```javascript
import { test, expect } from '@playwright/test';

test('visual regression', async ({ page }) => {
  await page.goto('https://example.com');

  await expect(page).toHaveScreenshot('homepage.png');
});
```

Playwright compares the current screenshot against the stored baseline.

---

# 37. Element Visual Regression

```javascript
const header = page.locator('header');

await expect(header).toHaveScreenshot('header.png');
```

This is useful when only one component needs visual validation.

---

# 38. Screenshot Baselines

A visual regression test may produce a structure similar to:

```text
tests/
├── visual.spec.js
└── visual.spec.js-snapshots/
    └── homepage.png
```

The baseline screenshot is compared with the screenshot generated during the test.

---

# 39. Update Visual Baselines

When the UI intentionally changes, update the baseline:

```bash
npx playwright test --update-snapshots
```

Short form:

```bash
npx playwright test -u
```

Do not update snapshots blindly.

Always review the visual changes first.

---

# 40. Screenshot Difference

If the actual screenshot differs from the expected screenshot, Playwright can report a visual mismatch.

The test output can include:

```text
Expected
Actual
Diff
```

This makes it easier to identify visual changes.

---

# 41. Screenshot Threshold

Visual comparisons can use a configurable threshold.

Example:

```javascript
await expect(page).toHaveScreenshot('homepage.png', {
  maxDiffPixels: 100
});
```

Another option is a ratio-based threshold:

```javascript
await expect(page).toHaveScreenshot('homepage.png', {
  maxDiffPixelRatio: 0.01
});
```

Use thresholds carefully.

A large threshold can hide real UI defects.

---

# 42. Screenshot with a Custom Timeout

```javascript
await expect(page).toHaveScreenshot('homepage.png', {
  timeout: 10000
});
```

---

# 43. Screenshot After Waiting for Fonts

Fonts can affect visual comparisons.

```javascript
await page.evaluate(() => document.fonts.ready);

await expect(page).toHaveScreenshot('homepage.png');
```

This can help reduce font-related screenshot differences.

---

# 44. Hide Dynamic Content

For example:

```javascript
await page.locator('.live-clock').evaluate(
  element => element.style.visibility = 'hidden'
);

await page.screenshot({
  path: 'screenshots/dashboard.png'
});
```

Masking is often preferable for visual tests:

```javascript
await expect(page).toHaveScreenshot('dashboard.png', {
  mask: [
    page.locator('.live-clock')
  ]
});
```

---

# 45. Screenshot Helper Utility

A reusable screenshot utility can reduce duplicate code.

```javascript
export async function takeScreenshot(
  page,
  name
) {
  await page.screenshot({
    path: `screenshots/${name}.png`,
    fullPage: true
  });
}
```

Use it:

```javascript
import { takeScreenshot } from './utils/screenshot.js';

await takeScreenshot(page, 'dashboard');
```

---

# 46. Screenshot Helper with TestInfo

A better approach for parallel execution is to use `testInfo`.

```javascript
export async function takeScreenshot(
  page,
  testInfo,
  name
) {
  await page.screenshot({
    path: testInfo.outputPath(`${name}.png`),
    fullPage: true
  });
}
```

Usage:

```javascript
await takeScreenshot(
  page,
  testInfo,
  'dashboard'
);
```

---

# 47. Complete Screenshot Test

```javascript
import { test, expect } from '@playwright/test';

test('complete screenshot example', async ({
  page
}, testInfo) => {

  await page.goto('https://example.com');

  await page.screenshot({
    path: testInfo.outputPath('homepage.png'),
    fullPage: true
  });

  const heading = page.getByRole('heading', {
    name: 'Example Domain'
  });

  await heading.screenshot({
    path: testInfo.outputPath('heading.png')
  });

  await expect(page).toHaveScreenshot(
    'homepage-visual.png'
  );
});
```

---

# 48. Screenshot Configuration Example

A production-style configuration might look like:

```javascript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './tests',

  use: {
    baseURL: 'https://example.com',

    screenshot: 'only-on-failure',

    trace: 'retain-on-failure',

    video: 'retain-on-failure'
  },

  reporter: [
    ['html']
  ]
});
```

---

# 49. Screenshots in CI/CD

Screenshots are particularly valuable in CI/CD.

For example:

```text
Test Failure
     |
     v
Playwright
     |
     +---- Screenshot
     |
     +---- Trace
     |
     +---- Video
     |
     v
CI Artifact
```

When a test fails on a remote CI machine, screenshots provide immediate visual evidence.

---

# 50. Screenshot Best Practices

### 1. Use screenshots strategically

Do not capture screenshots after every line of code.

Capture them at meaningful states:

```text
Login page
      ↓
After login
      ↓
Dashboard
      ↓
Checkout
      ↓
Confirmation
```

### 2. Prefer `only-on-failure` in CI

```javascript
screenshot: 'only-on-failure'
```

### 3. Use `testInfo.outputPath()`

This helps avoid filename conflicts during parallel execution.

### 4. Mask dynamic data

Use:

```javascript
mask: [...]
```

for timestamps and other dynamic elements.

### 5. Disable animations for visual testing

```javascript
animations: 'disabled'
```

### 6. Use stable locators

Prefer:

```javascript
page.getByRole(...)
```

or:

```javascript
page.getByTestId(...)
```

### 7. Do not blindly update snapshots

Always review visual changes before running:

```bash
npx playwright test -u
```

---

# 51. Common Screenshot Options

| Option           | Purpose                        |
| ---------------- | ------------------------------ |
| `path`           | Output file location           |
| `fullPage`       | Capture entire scrollable page |
| `type`           | PNG or JPEG                    |
| `quality`        | JPEG quality                   |
| `clip`           | Capture a specific area        |
| `animations`     | Control animations             |
| `caret`          | Show/hide text caret           |
| `mask`           | Hide dynamic elements          |
| `maskColor`      | Customize mask color           |
| `omitBackground` | Capture transparent background |

---

# 52. Page Screenshot vs Locator Screenshot

| Feature           | Page Screenshot | Locator Screenshot |
| ----------------- | --------------- | ------------------ |
| Entire viewport   | Yes             | No                 |
| Full page         | Yes             | No                 |
| Specific element  | Using `clip`    | Yes                |
| Easy to use       | Yes             | Yes                |
| Component testing | Good            | Excellent          |
| Visual regression | Excellent       | Excellent          |

---

# 53. Screenshot vs Visual Assertion

### Screenshot

```javascript
await page.screenshot({
  path: 'homepage.png'
});
```

This simply captures an image.

### Visual assertion

```javascript
await expect(page).toHaveScreenshot(
  'homepage.png'
);
```

This captures and compares the image against a baseline.

Use screenshots for **evidence/debugging**.

Use `toHaveScreenshot()` for **visual regression testing**.

---

# 54. Interview Questions

## Q1. How do you take a screenshot in Playwright?

```javascript
await page.screenshot({
  path: 'screenshot.png'
});
```

---

## Q2. How do you capture the entire page?

```javascript
await page.screenshot({
  path: 'full-page.png',
  fullPage: true
});
```

---

## Q3. How do you capture a specific element?

```javascript
await page.locator('.header').screenshot({
  path: 'header.png'
});
```

---

## Q4. How do you capture screenshots only when tests fail?

Configure:

```javascript
use: {
  screenshot: 'only-on-failure'
}
```

---

## Q5. How do you perform visual regression testing?

```javascript
await expect(page).toHaveScreenshot(
  'homepage.png'
);
```

---

## Q6. How do you update screenshot baselines?

```bash
npx playwright test -u
```

---

## Q7. How do you handle dynamic content?

Use masking:

```javascript
await expect(page).toHaveScreenshot('page.png', {
  mask: [
    page.locator('.timestamp')
  ]
});
```

---

## Q8. How do you capture a screenshot in a parallel test safely?

Use:

```javascript
await page.screenshot({
  path: testInfo.outputPath('screenshot.png')
});
```

---

## Q9. What is the difference between `page.screenshot()` and `toHaveScreenshot()`?

`page.screenshot()` captures an image.

`toHaveScreenshot()` captures an image and compares it with a baseline for visual regression testing.

---

## Q10. Why can screenshots be flaky?

Common reasons include:

* Animations
* Dynamic data
* Different fonts
* Different browser versions
* Different viewport sizes
* Loading delays
* Responsive layouts
* Time-dependent content

Use stable test data, wait for the correct UI state, disable animations when appropriate, and mask dynamic content.

---

# 55. Recommended Enterprise Structure

```text
playwright-project/
│
├── tests/
│   ├── login.spec.js
│   ├── dashboard.spec.js
│   └── checkout.spec.js
│
├── pages/
│   ├── LoginPage.js
│   ├── DashboardPage.js
│   └── CheckoutPage.js
│
├── utils/
│   └── screenshot.js
│
├── screenshots/
│
├── test-results/
│
├── playwright-report/
│
├── playwright.config.js
└── package.json
```

For CI execution, it is usually better to let Playwright manage test artifacts under its output directories rather than committing generated screenshots into Git.

---

# 56. Final Recommended Configuration

```javascript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './tests',

  use: {
    screenshot: 'only-on-failure',
    trace: 'retain-on-failure',
    video: 'retain-on-failure'
  },

  reporter: [
    ['html']
  ]
});
```

This gives a strong balance between:

* Debugging
* Storage usage
* CI performance
* Test evidence
* Failure investigation

---

# Summary

Playwright provides powerful screenshot capabilities for both debugging and visual testing.

The most important APIs are:

```javascript
await page.screenshot({
  path: 'page.png'
});
```

For full-page screenshots:

```javascript
await page.screenshot({
  path: 'page.png',
  fullPage: true
});
```

For element screenshots:

```javascript
await page.locator('.element').screenshot({
  path: 'element.png'
});
```

For failure screenshots:

```javascript
use: {
  screenshot: 'only-on-failure'
}
```

For visual regression:

```javascript
await expect(page).toHaveScreenshot(
  'homepage.png'
);
```

For dynamic content:

```javascript
await expect(page).toHaveScreenshot(
  'homepage.png',
  {
    mask: [
      page.locator('.timestamp')
    ]
  }
);
```

For parallel-safe artifact paths:

```javascript
await page.screenshot({
  path: testInfo.outputPath('page.png')
});
```

Screenshots are especially valuable when combined with **Playwright Test, traces, videos, HTML reports, CI/CD pipelines, and visual regression testing**.
