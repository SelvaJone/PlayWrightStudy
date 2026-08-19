# Playwright Browsers

## 1. Overview

Playwright supports modern browser automation across:

* Chromium
* Google Chrome
* Microsoft Edge
* Firefox
* WebKit

The three browser engines directly bundled and supported by Playwright are:

```text
Chromium
Firefox
WebKit
```

A simplified architecture is:

```text
Playwright
    |
    +── Chromium
    |     ├── Chrome
    |     └── Edge
    |
    +── Firefox
    |
    └── WebKit
```

Browser support is one of the major differences between Playwright and traditional Selenium automation.

---

# 2. Browser Engines

## Chromium

Chromium is the open-source browser engine used by browsers such as:

* Google Chrome
* Microsoft Edge
* Chromium

Playwright provides:

```typescript
import { chromium } from '@playwright/test';
```

Launch Chromium:

```typescript
const browser = await chromium.launch();
```

---

# 3. Firefox

Playwright supports Mozilla Firefox.

Import:

```typescript
import { firefox } from '@playwright/test';
```

Launch:

```typescript
const browser = await firefox.launch();
```

Firefox testing is important for cross-browser compatibility.

---

# 4. WebKit

Playwright also supports WebKit.

Import:

```typescript
import { webkit } from '@playwright/test';
```

Launch:

```typescript
const browser = await webkit.launch();
```

WebKit is especially important when testing applications that need Safari-like browser behavior.

---

# 5. Why WebKit Matters

Safari uses Apple's WebKit browser engine.

Playwright does not launch Safari itself through Playwright.

Instead, Playwright provides WebKit browser automation.

This allows teams to test important WebKit behavior without requiring Safari itself.

Interview point:

> Playwright supports Chromium, Firefox, and WebKit browser engines. Safari itself is not one of the Playwright browser binaries.

---

# 6. Browser Installation

Install all Playwright browsers:

```bash
npx playwright install
```

Install Chromium only:

```bash
npx playwright install chromium
```

Install Firefox only:

```bash
npx playwright install firefox
```

Install WebKit only:

```bash
npx playwright install webkit
```

Install browser dependencies on supported Linux environments:

```bash
npx playwright install --with-deps
```

---

# 7. Check Playwright Version

Check the installed Playwright version:

```bash
npx playwright --version
```

Example:

```text
Version 1.x.x
```

The exact version depends on the version installed in the project.

---

# 8. Launch Chromium

Basic example:

```typescript
import { chromium } from '@playwright/test';

const browser = await chromium.launch();

const context = await browser.newContext();

const page = await context.newPage();

await page.goto('https://example.com');

console.log(await page.title());

await browser.close();
```

The flow is:

```text
Chromium
   ↓
Browser
   ↓
BrowserContext
   ↓
Page
   ↓
Web Application
```

---

# 9. Launch Firefox

```typescript
import { firefox } from '@playwright/test';

const browser = await firefox.launch();

const context = await browser.newContext();

const page = await context.newPage();

await page.goto('https://example.com');

console.log(await page.title());

await browser.close();
```

---

# 10. Launch WebKit

```typescript
import { webkit } from '@playwright/test';

const browser = await webkit.launch();

const context = await browser.newContext();

const page = await context.newPage();

await page.goto('https://example.com');

console.log(await page.title());

await browser.close();
```

---

# 11. Headless Mode

Playwright normally runs tests in headless mode.

Example:

```typescript
const browser = await chromium.launch({
    headless: true
});
```

Explicitly use headed mode:

```typescript
const browser = await chromium.launch({
    headless: false
});
```

Headed mode is useful for:

* Debugging
* Learning
* Investigating failures
* Watching test execution

---

# 12. Browser Projects

In Playwright Test, browsers are commonly configured as projects.

Example:

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({

    projects: [

        {
            name: 'chromium',
            use: {
                ...devices['Desktop Chrome']
            }
        },

        {
            name: 'firefox',
            use: {
                ...devices['Desktop Firefox']
            }
        },

        {
            name: 'webkit',
            use: {
                ...devices['Desktop Safari']
            }
        }

    ]

});
```

Now the same tests can run against multiple browser configurations.

---

# 13. Run Chromium Tests

If the project is named:

```text
chromium
```

run:

```bash
npx playwright test --project=chromium
```

---

# 14. Run Firefox Tests

```bash
npx playwright test --project=firefox
```

---

# 15. Run WebKit Tests

```bash
npx playwright test --project=webkit
```

---

# 16. Run All Browsers

Simply run:

```bash
npx playwright test
```

If all three projects are configured, Playwright executes the tests against all configured projects.

Conceptually:

```text
Test
 |
 +── Chromium
 |
 +── Firefox
 |
 └── WebKit
```

---

# 17. Browser Configuration

A Playwright project can configure browser behavior through `use`.

Example:

```typescript
export default defineConfig({

    use: {
        headless: true,
        viewport: {
            width: 1280,
            height: 720
        }
    }

});
```

Common browser-related settings include:

* `headless`
* `viewport`
* `userAgent`
* `locale`
* `timezoneId`
* `permissions`
* `geolocation`
* `colorScheme`
* `storageState`
* `ignoreHTTPSErrors`

---

# 18. Viewport

Configure browser size:

```typescript
use: {
    viewport: {
        width: 1920,
        height: 1080
    }
}
```

This is useful when testing responsive web applications.

---

# 19. Mobile Browser Emulation

Playwright can emulate mobile devices.

Example:

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({

    projects: [

        {
            name: 'Mobile Chrome',
            use: {
                ...devices['Pixel 5']
            }
        },

        {
            name: 'Mobile Safari',
            use: {
                ...devices['iPhone 13']
            }
        }

    ]

});
```

This can emulate characteristics such as:

* Viewport
* User agent
* Device scale factor
* Touch support
* Mobile behavior

Important:

> Mobile emulation is not the same as testing on a real physical mobile device.

---

# 20. Desktop Device Configuration

Example:

```typescript
import { devices } from '@playwright/test';

{
    name: 'Chrome',
    use: {
        ...devices['Desktop Chrome']
    }
}
```

Firefox:

```typescript
{
    name: 'Firefox',
    use: {
        ...devices['Desktop Firefox']
    }
}
```

Safari/WebKit-style configuration:

```typescript
{
    name: 'Safari',
    use: {
        ...devices['Desktop Safari']
    }
}
```

---

# 21. Multiple Browser Projects

A typical enterprise configuration could be:

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({

    projects: [

        {
            name: 'chromium',
            use: {
                ...devices['Desktop Chrome']
            }
        },

        {
            name: 'firefox',
            use: {
                ...devices['Desktop Firefox']
            }
        },

        {
            name: 'webkit',
            use: {
                ...devices['Desktop Safari']
            }
        }

    ]

});
```

This creates a cross-browser test matrix.

---

# 22. Browser Matrix

A browser matrix might look like:

| Browser       | Engine   | Typical Use                |
| ------------- | -------- | -------------------------- |
| Chrome        | Chromium | Primary desktop testing    |
| Edge          | Chromium | Enterprise/Windows testing |
| Firefox       | Gecko    | Firefox compatibility      |
| Safari        | WebKit   | Safari compatibility       |
| Mobile Chrome | Chromium | Android-style testing      |
| Mobile Safari | WebKit   | iOS-style testing          |

---

# 23. Chrome vs Chromium

This is an important distinction.

**Chromium** is an open-source browser project/engine.

**Google Chrome** is Google's browser based on Chromium and includes Google's additional components.

Playwright's standard Chromium project uses Playwright's supported Chromium browser binary.

If you specifically need installed Google Chrome, you can configure a Chrome channel.

---

# 24. Using Google Chrome

Playwright supports browser channels.

Example:

```typescript
import { chromium } from '@playwright/test';

const browser = await chromium.launch({
    channel: 'chrome'
});
```

Or in configuration:

```typescript
use: {
    channel: 'chrome'
}
```

This allows testing with the installed Google Chrome browser.

---

# 25. Using Microsoft Edge

Microsoft Edge is Chromium-based.

Playwright can use the Edge channel:

```typescript
use: {
    channel: 'msedge'
}
```

Example:

```typescript
import { test } from '@playwright/test';

test.use({
    channel: 'msedge'
});

test('Edge test', async ({ page }) => {

    await page.goto('https://example.com');

});
```

---

# 26. Browser Channels

Common browser channels include:

```text
chrome
chrome-beta
chrome-dev
chrome-canary
msedge
msedge-beta
msedge-dev
msedge-canary
```

Availability depends on the installed browsers and environment.

---

# 27. Browser Launch Options

You can configure browser launch options.

Example:

```typescript
const browser = await chromium.launch({

    headless: false,

    slowMo: 100

});
```

`slowMo` slows down Playwright operations.

This can be useful for debugging.

Example:

```typescript
slowMo: 500
```

Each operation is slowed down by approximately 500 milliseconds.

Do not normally use `slowMo` in production test execution because it makes tests slower.

---

# 28. Browser Arguments

Playwright allows browser launch arguments.

Example:

```typescript
const browser = await chromium.launch({
    args: [
        '--start-maximized'
    ]
});
```

Be careful with custom browser arguments.

Incorrect or unsupported browser flags can cause unexpected behavior.

Use browser arguments only when there is a legitimate testing requirement.

---

# 29. Browser Context vs Browser

A common interview question is:

> What is the difference between Browser and BrowserContext?

### Browser

Represents the browser process/instance.

```typescript
const browser = await chromium.launch();
```

### BrowserContext

Represents an isolated browser session.

```typescript
const context = await browser.newContext();
```

Example:

```text
Browser
   |
   +── Context A → User A
   |
   +── Context B → User B
```

The contexts can have independent:

* Cookies
* Storage
* Authentication
* Permissions

---

# 30. Browser Context Isolation

Example:

```typescript
const context1 = await browser.newContext();

const context2 = await browser.newContext();
```

Create pages:

```typescript
const page1 = await context1.newPage();

const page2 = await context2.newPage();
```

These sessions are isolated.

This is one reason BrowserContext is extremely useful for testing multiple users.

---

# 31. Multiple Pages in One Context

A single context can contain multiple pages:

```typescript
const page1 = await context.newPage();

const page2 = await context.newPage();

const page3 = await context.newPage();
```

Conceptually:

```text
Browser
   |
   └── Context
         |
         ├── Page 1
         ├── Page 2
         └── Page 3
```

This is useful when testing:

* Multiple tabs
* Popups
* Multi-window workflows

---

# 32. Browser Context Storage

Contexts can maintain:

* Cookies
* Local storage
* Session storage
* Authentication state

Example:

```typescript
const context = await browser.newContext({
    storageState: 'auth.json'
});
```

This allows a test to start with an existing authenticated state.

---

# 33. Browser Permissions

You can configure permissions.

Example:

```typescript
const context = await browser.newContext({
    permissions: ['geolocation']
});
```

This can be useful when testing:

* Location services
* Notifications
* Camera
* Microphone
* Other browser permissions

Only grant permissions required by the test.

---

# 34. Geolocation

Example:

```typescript
const context = await browser.newContext({

    geolocation: {
        latitude: 32.7767,
        longitude: -96.7970
    },

    permissions: ['geolocation']

});
```

This allows testing location-dependent application behavior.

---

# 35. Locale

Configure browser locale:

```typescript
const context = await browser.newContext({
    locale: 'en-US'
});
```

Other examples:

```text
en-US
en-GB
fr-FR
es-ES
de-DE
```

This is useful for internationalization testing.

---

# 36. Time Zone

Configure timezone:

```typescript
const context = await browser.newContext({
    timezoneId: 'America/Chicago'
});
```

Other examples:

```text
America/New_York
America/Los_Angeles
Europe/London
Asia/Kolkata
```

This is useful for testing applications whose behavior depends on local time.

---

# 37. User Agent

A custom user agent can be configured:

```typescript
const context = await browser.newContext({
    userAgent: 'Custom-Test-Agent'
});
```

Use this carefully because changing the user agent can alter application behavior.

---

# 38. Color Scheme

Playwright can emulate light or dark mode.

Example:

```typescript
const context = await browser.newContext({
    colorScheme: 'dark'
});
```

Possible values include:

```text
light
dark
no-preference
```

This is useful for testing applications with dark mode.

---

# 39. Ignore HTTPS Errors

For certain test environments with invalid/self-signed certificates:

```typescript
const context = await browser.newContext({
    ignoreHTTPSErrors: true
});
```

Use this only in appropriate test environments.

Do not use it to hide certificate problems in production systems.

---

# 40. Browser Downloads

Playwright can handle downloads.

Example:

```typescript
const downloadPromise = page.waitForEvent('download');

await page.getByText('Download').click();

const download = await downloadPromise;

await download.saveAs('downloads/file.pdf');
```

Download handling is covered in more detail in the File Upload/Download topic.

---

# 41. Browser Screenshots

Take a screenshot:

```typescript
await page.screenshot({
    path: 'screenshots/home.png'
});
```

Full-page:

```typescript
await page.screenshot({
    path: 'screenshots/home.png',
    fullPage: true
});
```

---

# 42. Browser Video

Configure video recording:

```typescript
const context = await browser.newContext({

    recordVideo: {
        dir: 'videos/'
    }

});
```

This can be useful for debugging failures.

---

# 43. Browser Tracing

Playwright Trace Viewer can capture detailed test execution information.

Configuration:

```typescript
use: {
    trace: 'on-first-retry'
}
```

A trace can help analyze:

* Actions
* DOM snapshots
* Screenshots
* Network
* Timing
* Failures

---

# 44. Cross-Browser Testing

A common Playwright strategy is:

```text
                    Test Suite
                        |
       +----------------+----------------+
       |                |                |
       v                v                v
   Chromium          Firefox           WebKit
       |                |                |
       v                v                v
    Results          Results          Results
```

This allows the same automation suite to identify browser-specific issues.

---

# 45. Selenium vs Playwright Browser Management

## Selenium

Traditional Selenium architecture:

```text
Test
 ↓
Selenium WebDriver
 ↓
Browser Driver
 ↓
Browser
```

Examples:

```text
ChromeDriver → Chrome
GeckoDriver  → Firefox
EdgeDriver   → Edge
```

Modern Selenium includes Selenium Manager, which can automate much of driver management.

## Playwright

Playwright architecture:

```text
Test
 ↓
Playwright
 ↓
Browser
```

Playwright manages its supported browser binaries through its installation mechanism.

This simplifies browser setup for Playwright-managed browsers.

---

# 46. Browser Version Considerations

Playwright is designed and tested against particular browser versions bundled with each Playwright release.

Therefore, when upgrading Playwright:

```text
Upgrade Playwright
        ↓
Install/update browsers
        ↓
Run regression tests
        ↓
Validate framework
```

Use:

```bash
npx playwright install
```

after upgrading when browser binaries need to be updated.

---

# 47. Enterprise Browser Strategy

For an enterprise automation framework, you might define:

```typescript
projects: [

    {
        name: 'chrome',
        use: {
            ...devices['Desktop Chrome']
        }
    },

    {
        name: 'firefox',
        use: {
            ...devices['Desktop Firefox']
        }
    },

    {
        name: 'webkit',
        use: {
            ...devices['Desktop Safari']
        }
    },

    {
        name: 'edge',
        use: {
            channel: 'msedge'
        }
    }

]
```

Then execute:

```bash
npx playwright test
```

---

# 48. Selective Browser Execution

You do not always need to run every browser for every test.

For example:

```bash
npx playwright test --project=chromium
```

Smoke tests may run against:

```text
Chromium
```

while nightly regression may run against:

```text
Chromium
Firefox
WebKit
Edge
```

This can reduce CI execution time.

---

# 49. Browser Matrix in CI/CD

A CI pipeline might execute:

```text
Pull Request
    ↓
Chromium Smoke Tests
    ↓
Merge
    ↓
Cross-Browser Regression
    |
    +── Chromium
    +── Firefox
    +── WebKit
    └── Edge
```

This is often more efficient than running the complete browser matrix on every developer commit.

---

# 50. Common Browser Testing Mistakes

## Mistake 1: Testing only Chrome

A test passing in Chromium does not guarantee it will pass in Firefox or WebKit.

---

## Mistake 2: Assuming WebKit is Safari

WebKit provides Safari-like engine coverage, but it is not the same as running the exact Safari application on every Apple platform.

---

## Mistake 3: Using too many browser projects

Running every test against every browser can increase execution time significantly.

Use a test strategy based on risk and application requirements.

---

## Mistake 4: Ignoring mobile behavior

Desktop browser testing alone does not guarantee good mobile UX.

Use appropriate device emulation and, when necessary, real-device testing.

---

## Mistake 5: Overusing custom browser arguments

Unnecessary flags can make tests behave differently from real users.

---

# 51. Best Practices

### 1. Use Playwright-managed browsers for normal automation

```bash
npx playwright install
```

### 2. Configure browsers as projects

```typescript
projects: [
    // browser projects
]
```

### 3. Use Chromium for fast developer feedback

Run:

```bash
npx playwright test --project=chromium
```

### 4. Use cross-browser testing in regression

Run:

```text
Chromium
Firefox
WebKit
```

when required.

### 5. Use device emulation for responsive testing

Example:

```typescript
...devices['iPhone 13']
```

### 6. Keep browser configuration centralized

Use:

```text
playwright.config.ts
```

### 7. Avoid unnecessary hard-coded browser flags

### 8. Keep browser versions aligned with the Playwright version

---

# 52. Interview Questions

## Beginner

1. Which browsers does Playwright support?
2. What is Chromium?
3. Does Playwright support Firefox?
4. Does Playwright support WebKit?
5. Does Playwright directly automate Safari?
6. How do you install Playwright browsers?
7. How do you launch Chromium?
8. How do you launch Firefox?
9. How do you launch WebKit?
10. What is headless mode?

## Intermediate

11. What is a BrowserContext?
12. Why is BrowserContext important?
13. Can one browser have multiple contexts?
14. Can one context have multiple pages?
15. How do you configure multiple browser projects?
16. How do you run only Chromium tests?
17. How do you run only Firefox tests?
18. How do you use Microsoft Edge with Playwright?
19. What are browser channels?
20. How do you emulate mobile devices?

## Advanced

21. What is the difference between Chromium and Chrome?
22. How does Playwright manage browser binaries?
23. How would you design a cross-browser strategy?
24. How would you reduce cross-browser CI execution time?
25. How would you test Safari compatibility?
26. How would you test location-dependent functionality?
27. How would you test different locales?
28. How would you test timezone-dependent behavior?
29. How would you isolate two users using BrowserContext?
30. How would you design a browser matrix for an enterprise framework?

---

# 53. Quick Reference

## Browser imports

```typescript
import {
    chromium,
    firefox,
    webkit
} from '@playwright/test';
```

## Launch Chromium

```typescript
const browser = await chromium.launch();
```

## Launch Firefox

```typescript
const browser = await firefox.launch();
```

## Launch WebKit

```typescript
const browser = await webkit.launch();
```

## Headed

```typescript
const browser = await chromium.launch({
    headless: false
});
```

## Chrome channel

```typescript
const browser = await chromium.launch({
    channel: 'chrome'
});
```

## Edge channel

```typescript
const browser = await chromium.launch({
    channel: 'msedge'
});
```

## Create context

```typescript
const context = await browser.newContext();
```

## Create page

```typescript
const page = await context.newPage();
```

## Navigate

```typescript
await page.goto('https://example.com');
```

## Close browser

```typescript
await browser.close();
```

---

# 54. Key Takeaways

Remember this hierarchy:

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
```

Remember the main browser engines:

```text
Chromium
Firefox
WebKit
```

Remember common browser channels:

```text
Chrome
Edge
```

Remember the most important commands:

```bash
npx playwright install
npx playwright install chromium
npx playwright install firefox
npx playwright install webkit
npx playwright test
npx playwright test --project=chromium
```

For interviews, be particularly comfortable explaining:

* Chromium vs Chrome
* WebKit vs Safari
* Browser vs BrowserContext
* BrowserContext isolation
* Browser projects
* Cross-browser testing
* Mobile device emulation
* Browser channels
* Headless vs headed execution
* Browser management in CI/CD

The next topic is:

**`BrowserContext/Playwright-BrowserContext.md`**
