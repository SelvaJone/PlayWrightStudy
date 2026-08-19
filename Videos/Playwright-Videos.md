# Playwright Videos

## Overview

Playwright can record a video of each test execution. Video recording is especially useful for:

* Debugging failed tests
* Investigating unexpected UI behavior
* Reviewing test execution
* CI/CD troubleshooting
* Capturing evidence for failed tests
* Understanding flaky test failures

Playwright records the video at the browser-context level.

---

## 1. Enable Video Recording

Video recording can be enabled when creating a browser context.

```javascript
const { chromium } = require('@playwright/test');

(async () => {
  const browser = await chromium.launch();

  const context = await browser.newContext({
    recordVideo: {
      dir: 'videos/'
    }
  });

  const page = await context.newPage();

  await page.goto('https://example.com');

  await page.waitForTimeout(3000);

  await context.close();
  await browser.close();
})();
```

The recorded video will be saved inside the `videos/` directory.

---

## 2. Video Recording with Playwright Test

In Playwright Test, video recording is normally configured through `playwright.config.js`.

```javascript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  use: {
    video: 'on'
  }
});
```

Now every test execution will record a video.

---

## 3. Video Modes

Playwright supports several video recording modes.

### Record Every Test

```javascript
export default defineConfig({
  use: {
    video: 'on'
  }
});
```

This records a video for every test.

---

### Record Only Failed Tests

```javascript
export default defineConfig({
  use: {
    video: 'retain-on-failure'
  }
});
```

This is one of the most useful configurations for CI/CD.

A video is recorded during the test, but it is retained only when the test fails.

---

### Record Retry Attempts

```javascript
export default defineConfig({
  use: {
    video: 'on-first-retry'
  }
});
```

This records video when a test is retried for the first time.

This is useful when debugging flaky tests.

---

## 4. Recommended CI Configuration

A common configuration is:

```javascript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  retries: process.env.CI ? 2 : 0,

  use: {
    video: 'retain-on-failure'
  }
});
```

This gives you:

* No unnecessary videos for successful tests
* Videos for failed tests
* Useful evidence in CI
* Reduced storage usage

---

## 5. Complete Example

```javascript
import { test, expect } from '@playwright/test';

test('Login test', async ({ page }) => {
  await page.goto('https://example.com/login');

  await page.getByLabel('Username').fill('testuser');

  await page.getByLabel('Password').fill('password');

  await page.getByRole('button', { name: 'Login' }).click();

  await expect(page).toHaveURL(/dashboard/);
});
```

Configuration:

```javascript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  use: {
    video: 'retain-on-failure'
  }
});
```

If the test fails, Playwright retains the video.

---

## 6. Specify Video Directory

When using the browser context API:

```javascript
const context = await browser.newContext({
  recordVideo: {
    dir: 'test-results/videos/'
  }
});
```

This keeps videos organized separately from other test artifacts.

---

## 7. Set Video Size

You can specify the video dimensions.

```javascript
const context = await browser.newContext({
  recordVideo: {
    dir: 'videos/',
    size: {
      width: 1280,
      height: 720
    }
  }
});
```

Example:

```javascript
recordVideo: {
  dir: 'videos/',
  size: {
    width: 1920,
    height: 1080
  }
}
```

---

## 8. Video Resolution

The video size can affect:

* Video quality
* File size
* Storage requirements
* CI performance

A common choice is:

```javascript
size: {
  width: 1280,
  height: 720
}
```

This provides good quality without creating unnecessarily large files.

---

## 9. Access the Video Object

When using the browser context API, the page provides access to its video.

```javascript
const video = page.video();
```

Example:

```javascript
const video = page.video();

console.log(video);
```

The video object provides methods for working with the recorded video.

---

## 10. Get Video Path

After the browser context is closed:

```javascript
const videoPath = await page.video().path();

console.log(videoPath);
```

Important:

The browser context should be closed before relying on the final video file.

Example:

```javascript
const browser = await chromium.launch();

const context = await browser.newContext({
  recordVideo: {
    dir: 'videos/'
  }
});

const page = await context.newPage();

await page.goto('https://example.com');

await context.close();

const videoPath = await page.video().path();

console.log(videoPath);

await browser.close();
```

---

## 11. Save Video to a Specific Location

Playwright provides the `saveAs()` method.

```javascript
await page.video().saveAs('videos/login-test.webm');
```

Example:

```javascript
const video = page.video();

await context.close();

await video.saveAs('videos/login-test.webm');
```

---

## 12. Delete a Video

You can delete a recorded video using:

```javascript
await page.video().delete();
```

Example:

```javascript
const video = page.video();

await context.close();

await video.delete();
```

---

## 13. Video Format

Playwright records browser videos in WebM format.

Typical output:

```text
test-results/
└── videos/
    └── test-video.webm
```

WebM files can be opened using many modern media players and browsers.

---

## 14. Video with Screenshots and Traces

Videos become especially useful when combined with:

* Screenshots
* Traces
* Console logs
* Network logs
* Test reports

Example:

```javascript
export default defineConfig({
  use: {
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    trace: 'retain-on-failure'
  }
});
```

This provides a strong debugging package for failed tests.

---

## 15. Recommended Debugging Configuration

For local debugging:

```javascript
export default defineConfig({
  use: {
    screenshot: 'on',
    video: 'on',
    trace: 'on'
  }
});
```

For CI:

```javascript
export default defineConfig({
  use: {
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    trace: 'retain-on-failure'
  }
});
```

---

## 16. Video with Retries

Consider this configuration:

```javascript
export default defineConfig({
  retries: 2,

  use: {
    video: 'on-first-retry'
  }
});
```

Test flow:

```text
Test starts
   |
   v
Test fails
   |
   v
Retry #1
   |
   v
Video recorded
   |
   v
Test passes/fails
```

This is useful for diagnosing flaky tests without recording every test.

---

## 17. Video with Parallel Execution

Videos work with parallel tests.

Example:

```javascript
export default defineConfig({
  workers: 4,

  use: {
    video: 'retain-on-failure'
  }
});
```

Each test gets its own video artifact.

Example:

```text
test-results/
├── test-1/
│   └── video.webm
├── test-2/
│   └── video.webm
├── test-3/
│   └── video.webm
└── test-4/
    └── video.webm
```

---

## 18. Video in CI/CD

Videos are particularly useful in CI/CD pipelines.

Example configuration:

```javascript
export default defineConfig({
  use: {
    video: 'retain-on-failure'
  }
});
```

When a test fails:

```text
CI Pipeline
     |
     v
Playwright Test
     |
     v
Test Failure
     |
     +---- Screenshot
     |
     +---- Video
     |
     +---- Trace
     |
     v
Test Report
```

The artifacts can then be uploaded by the CI system.

---

## 19. GitHub Actions Example

A typical GitHub Actions workflow can preserve Playwright test results.

```yaml
- name: Run Playwright tests
  run: npx playwright test

- name: Upload Playwright report
  if: always()
  uses: actions/upload-artifact@v4
  with:
    name: playwright-report
    path: playwright-report/
    retention-days: 30

- name: Upload test results
  if: always()
  uses: actions/upload-artifact@v4
  with:
    name: test-results
    path: test-results/
    retention-days: 30
```

Failed-test videos stored in `test-results/` can therefore be retained as CI artifacts.

---

## 20. Video and Test Reports

Videos are automatically associated with test results when Playwright Test manages the recording.

Example:

```javascript
import { test, expect } from '@playwright/test';

test('Checkout flow', async ({ page }) => {
  await page.goto('https://example.com');

  await page.getByRole('button', { name: 'Checkout' }).click();

  await expect(page).toHaveURL(/checkout/);
});
```

Configuration:

```javascript
export default defineConfig({
  use: {
    video: 'retain-on-failure'
  }
});
```

When the test fails, the report can provide access to the recorded video artifact.

---

## 21. Video vs Screenshot vs Trace

| Feature                     | Screenshot      | Video            | Trace          |
| --------------------------- | --------------- | ---------------- | -------------- |
| Captures single moment      | Yes             | No               | No             |
| Captures complete execution | No              | Yes              | Yes            |
| Shows UI movement           | No              | Yes              | Yes            |
| Network information         | No              | No               | Yes            |
| DOM information             | Limited         | Visual           | Yes            |
| Debugging capability        | Basic           | Good             | Excellent      |
| File size                   | Small           | Medium/Large     | Large          |
| Best use                    | Visual evidence | Execution replay | Deep debugging |

---

## 22. When Should You Use Video?

Use videos when:

* A test is difficult to reproduce
* UI behavior needs to be observed
* Animations cause failures
* Timing issues occur
* Tests are flaky
* CI failures need visual evidence
* You need evidence for a defect

---

## 23. When Should You Avoid Recording Every Video?

Recording every test can create large amounts of data.

For a large test suite:

```text
10,000 tests
     |
     v
10,000 videos
     |
     v
Large storage usage
```

Instead, prefer:

```javascript
video: 'retain-on-failure'
```

or:

```javascript
video: 'on-first-retry'
```

---

## 24. Best Practice Configuration

A practical enterprise configuration is:

```javascript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  retries: process.env.CI ? 2 : 0,

  use: {
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    trace: 'retain-on-failure'
  }
});
```

This gives you:

```text
Successful Test
    |
    +---- No screenshot
    +---- No retained video
    +---- No retained trace


Failed Test
    |
    +---- Screenshot
    +---- Video
    +---- Trace
```

---

## 25. Enterprise Automation Example

```javascript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './tests',

  retries: process.env.CI ? 2 : 0,

  workers: process.env.CI ? 4 : undefined,

  use: {
    baseURL: 'https://example.com',

    screenshot: 'only-on-failure',

    video: 'retain-on-failure',

    trace: 'retain-on-failure'
  },

  reporter: [
    ['html'],
    ['list']
  ]
});
```

This is a good starting point for a professional Playwright automation framework.

---

## 26. Interview Questions

### Q1. How do you enable video recording in Playwright?

```javascript
use: {
  video: 'on'
}
```

---

### Q2. How do you record video only for failed tests?

```javascript
use: {
  video: 'retain-on-failure'
}
```

---

### Q3. What is `on-first-retry`?

It records video when a test is retried for the first time.

```javascript
video: 'on-first-retry'
```

---

### Q4. Why use `retain-on-failure` in CI?

It avoids storing videos for successful tests while preserving videos that help investigate failures.

---

### Q5. What video format does Playwright use?

Playwright records videos in WebM format.

---

### Q6. How do you specify the video directory?

```javascript
recordVideo: {
  dir: 'videos/'
}
```

---

### Q7. How do you specify video dimensions?

```javascript
recordVideo: {
  dir: 'videos/',
  size: {
    width: 1280,
    height: 720
  }
}
```

---

### Q8. How do videos help with flaky tests?

They allow engineers to visually review what happened during the failed or retried execution.

---

### Q9. Should you record videos for every test in a large test suite?

Usually no. `retain-on-failure` or `on-first-retry` is more appropriate to control storage and execution overhead.

---

### Q10. What is better for deep debugging: video or trace?

A Playwright trace generally provides more debugging information because it includes execution details, DOM snapshots, network information, screenshots, and other test metadata. Video is primarily visual evidence.

---

## 27. Best Practices

### Recommended

```javascript
use: {
  screenshot: 'only-on-failure',
  video: 'retain-on-failure',
  trace: 'retain-on-failure'
}
```

### Avoid unnecessarily

```javascript
use: {
  video: 'on'
}
```

for very large CI test suites unless there is a specific reason.

### Additional recommendations

1. Record videos for failures in CI.
2. Use `on-first-retry` for flaky-test investigation.
3. Combine videos with traces and screenshots.
4. Store test artifacts separately from source code.
5. Configure CI artifact retention.
6. Avoid committing generated video files to Git.
7. Use appropriate video dimensions.
8. Clean old artifacts regularly.
9. Use videos as debugging evidence rather than the only debugging mechanism.
10. Prefer traces for detailed root-cause analysis.

---

## 28. Quick Reference

```javascript
// Every test
video: 'on'

// Failed tests
video: 'retain-on-failure'

// First retry
video: 'on-first-retry'

// Browser context
recordVideo: {
  dir: 'videos/'
}

// Video size
recordVideo: {
  dir: 'videos/',
  size: {
    width: 1280,
    height: 720
  }
}
```

---

## 29. Recommended Folder Structure

```text
playwright-project/
│
├── tests/
│   ├── login.spec.js
│   ├── checkout.spec.js
│   └── profile.spec.js
│
├── test-results/
│   ├── screenshots/
│   ├── videos/
│   └── traces/
│
├── playwright-report/
│
├── playwright.config.js
├── package.json
└── README.md
```

---

## 30. Summary

Playwright video recording is an important debugging capability for modern UI automation.

The most important configurations are:

```javascript
video: 'on'
```

Record every test.

```javascript
video: 'retain-on-failure'
```

Retain videos only for failed tests.

```javascript
video: 'on-first-retry'
```

Record video during the first retry.

For enterprise automation and CI/CD, a strong default is:

```javascript
use: {
  screenshot: 'only-on-failure',
  video: 'retain-on-failure',
  trace: 'retain-on-failure'
}
```

This provides screenshots, videos, and traces when something goes wrong while avoiding unnecessary artifacts for successful tests.
