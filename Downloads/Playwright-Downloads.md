# Playwright Downloads

## Overview

Playwright provides built-in support for handling file downloads initiated by a web application.

A download can be triggered by:

* Clicking a download link
* Clicking a download button
* Exporting data
* Downloading a PDF
* Downloading CSV/Excel files
* Generating reports
* Downloading images or documents
* Browser-generated files

Playwright allows you to:

* Detect downloads
* Wait for a download
* Save downloaded files
* Rename downloaded files
* Verify downloaded files
* Read downloaded file paths
* Check file names
* Check MIME types
* Verify file existence
* Handle multiple downloads
* Configure download behavior
* Delete downloaded files
* Download files in parallel
* Validate downloaded content

---

# 1. Basic Download

Suppose the application has:

```html
<a href="/files/report.pdf">Download Report</a>
```

Playwright can handle the download using `page.waitForEvent('download')`.

### JavaScript

```javascript
const downloadPromise = page.waitForEvent('download');

await page.getByText('Download Report').click();

const download = await downloadPromise;
```

The `download` object represents the downloaded file.

---

# 2. Recommended Download Pattern

The most important rule is:

> Start waiting for the download before clicking the download button.

```javascript
const downloadPromise = page.waitForEvent('download');

await page.getByRole('button', { name: 'Download' }).click();

const download = await downloadPromise;
```

Then save the file:

```javascript
await download.saveAs('downloads/report.pdf');
```

Complete example:

```javascript
import { test, expect } from '@playwright/test';

test('Download report', async ({ page }) => {
    await page.goto('https://example.com');

    const downloadPromise = page.waitForEvent('download');

    await page.getByRole('button', { name: 'Download Report' }).click();

    const download = await downloadPromise;

    await download.saveAs('downloads/report.pdf');
});
```

---

# 3. Why `waitForEvent()` Must Come Before the Click

Incorrect:

```javascript
await page.getByRole('button', { name: 'Download' }).click();

const download = await page.waitForEvent('download');
```

The download event may already have happened before Playwright starts listening.

Correct:

```javascript
const downloadPromise = page.waitForEvent('download');

await page.getByRole('button', { name: 'Download' }).click();

const download = await downloadPromise;
```

This pattern should be used whenever an action triggers an asynchronous Playwright event.

---

# 4. Save the Download

Use:

```javascript
await download.saveAs('downloads/report.pdf');
```

Example:

```javascript
const downloadPromise = page.waitForEvent('download');

await page.getByText('Download PDF').click();

const download = await downloadPromise;

await download.saveAs('downloads/report.pdf');
```

---

# 5. Get the Suggested File Name

Playwright provides:

```javascript
download.suggestedFilename()
```

Example:

```javascript
const filename = download.suggestedFilename();

console.log(filename);
```

If the server specifies:

```text
customer-report.pdf
```

the result will be:

```text
customer-report.pdf
```

---

# 6. Save Using the Suggested File Name

```javascript
const filename = download.suggestedFilename();

await download.saveAs(`downloads/${filename}`);
```

Complete example:

```javascript
const downloadPromise = page.waitForEvent('download');

await page.getByRole('button', { name: 'Download' }).click();

const download = await downloadPromise;

const filename = download.suggestedFilename();

await download.saveAs(`downloads/${filename}`);
```

---

# 7. Get the Download Path

Playwright provides:

```javascript
download.path()
```

Example:

```javascript
const path = await download.path();

console.log(path);
```

Complete example:

```javascript
const downloadPromise = page.waitForEvent('download');

await page.getByRole('button', { name: 'Download' }).click();

const download = await downloadPromise;

const path = await download.path();

console.log(path);
```

The path is the temporary location where Playwright stores the downloaded file.

---

# 8. Important Difference Between `path()` and `saveAs()`

### `path()`

Returns the temporary file location:

```javascript
const path = await download.path();
```

### `saveAs()`

Copies the downloaded file to a location you specify:

```javascript
await download.saveAs('downloads/report.pdf');
```

For test automation, `saveAs()` is usually more useful when you want to keep the downloaded file.

---

# 9. Verify File Name

Use Playwright's `expect()`:

```javascript
const filename = download.suggestedFilename();

expect(filename).toBe('report.pdf');
```

Complete example:

```javascript
import { test, expect } from '@playwright/test';

test('Verify downloaded file name', async ({ page }) => {
    await page.goto('https://example.com');

    const downloadPromise = page.waitForEvent('download');

    await page.getByRole('button', { name: 'Download Report' }).click();

    const download = await downloadPromise;

    expect(download.suggestedFilename()).toBe('report.pdf');
});
```

---

# 10. Verify File Extension

```javascript
const filename = download.suggestedFilename();

expect(filename).toMatch(/\.pdf$/);
```

For CSV:

```javascript
expect(filename).toMatch(/\.csv$/);
```

For Excel:

```javascript
expect(filename).toMatch(/\.(xlsx|xls)$/);
```

---

# 11. Verify Downloaded File Exists

Node.js `fs` can be used to verify the saved file.

```javascript
import fs from 'fs';

const filePath = 'downloads/report.pdf';

expect(fs.existsSync(filePath)).toBeTruthy();
```

Complete example:

```javascript
import { test, expect } from '@playwright/test';
import fs from 'fs';

test('Verify downloaded file exists', async ({ page }) => {
    await page.goto('https://example.com');

    const downloadPromise = page.waitForEvent('download');

    await page.getByRole('button', { name: 'Download Report' }).click();

    const download = await downloadPromise;

    const filePath = 'downloads/report.pdf';

    await download.saveAs(filePath);

    expect(fs.existsSync(filePath)).toBeTruthy();
});
```

---

# 12. Verify File Size

Node.js can be used to check the downloaded file size.

```javascript
import fs from 'fs';

const stats = fs.statSync('downloads/report.pdf');

expect(stats.size).toBeGreaterThan(0);
```

Complete example:

```javascript
const filePath = 'downloads/report.pdf';

await download.saveAs(filePath);

const stats = fs.statSync(filePath);

expect(stats.size).toBeGreaterThan(0);
```

This verifies that the file is not empty.

---

# 13. Verify PDF Download

```javascript
test('Download PDF', async ({ page }) => {
    await page.goto('https://example.com');

    const downloadPromise = page.waitForEvent('download');

    await page.getByRole('button', { name: 'Download PDF' }).click();

    const download = await downloadPromise;

    expect(download.suggestedFilename()).toMatch(/\.pdf$/);

    await download.saveAs('downloads/report.pdf');
});
```

---

# 14. Verify CSV Download

```javascript
test('Download CSV', async ({ page }) => {
    await page.goto('https://example.com');

    const downloadPromise = page.waitForEvent('download');

    await page.getByRole('button', { name: 'Export CSV' }).click();

    const download = await downloadPromise;

    expect(download.suggestedFilename()).toMatch(/\.csv$/);

    await download.saveAs('downloads/report.csv');
});
```

---

# 15. Verify Excel Download

```javascript
test('Download Excel file', async ({ page }) => {
    await page.goto('https://example.com');

    const downloadPromise = page.waitForEvent('download');

    await page.getByRole('button', { name: 'Export Excel' }).click();

    const download = await downloadPromise;

    expect(download.suggestedFilename()).toMatch(/\.(xlsx|xls)$/);

    await download.saveAs('downloads/report.xlsx');
});
```

---

# 16. Verify Download Content

After saving the file, Node.js can be used to read the file.

For a text file:

```javascript
import fs from 'fs';

const content = fs.readFileSync(
    'downloads/report.txt',
    'utf-8'
);

expect(content).toContain('Test Report');
```

---

# 17. Validate CSV Content

```javascript
import fs from 'fs';

const filePath = 'downloads/report.csv';

await download.saveAs(filePath);

const content = fs.readFileSync(filePath, 'utf-8');

expect(content).toContain('Customer');
expect(content).toContain('Order');
```

---

# 18. Validate Downloaded JSON

If the downloaded file is JSON:

```javascript
import fs from 'fs';

const filePath = 'downloads/data.json';

await download.saveAs(filePath);

const content = fs.readFileSync(filePath, 'utf-8');

const data = JSON.parse(content);

expect(data.status).toBe('success');
```

---

# 19. Download Event with Timeout

You can provide a timeout:

```javascript
const downloadPromise = page.waitForEvent('download', {
    timeout: 30000
});
```

Example:

```javascript
const downloadPromise = page.waitForEvent('download', {
    timeout: 30000
});

await page.getByRole('button', { name: 'Download' }).click();

const download = await downloadPromise;
```

---

# 20. Handling Download Failure

Playwright provides:

```javascript
download.failure()
```

Example:

```javascript
const failure = await download.failure();

expect(failure).toBeNull();
```

Complete example:

```javascript
const downloadPromise = page.waitForEvent('download');

await page.getByRole('button', { name: 'Download' }).click();

const download = await downloadPromise;

const failure = await download.failure();

expect(failure).toBeNull();
```

---

# 21. Check Download Failure

```javascript
const failure = await download.failure();

if (failure) {
    console.log(`Download failed: ${failure}`);
}
```

---

# 22. Complete Download Validation

```javascript
import { test, expect } from '@playwright/test';
import fs from 'fs';

test('Validate downloaded report', async ({ page }) => {
    await page.goto('https://example.com');

    const downloadPromise = page.waitForEvent('download');

    await page.getByRole('button', { name: 'Download Report' }).click();

    const download = await downloadPromise;

    const failure = await download.failure();

    expect(failure).toBeNull();

    const filename = download.suggestedFilename();

    expect(filename).toBe('report.pdf');

    const filePath = `downloads/${filename}`;

    await download.saveAs(filePath);

    expect(fs.existsSync(filePath)).toBeTruthy();

    const stats = fs.statSync(filePath);

    expect(stats.size).toBeGreaterThan(0);
});
```

---

# 23. Download Using a Custom File Name

```javascript
const downloadPromise = page.waitForEvent('download');

await page.getByRole('button', { name: 'Download' }).click();

const download = await downloadPromise;

await download.saveAs('downloads/customer-report.pdf');
```

The original server file name does not have to be used.

---

# 24. Create a Download Directory

Node.js can create the directory:

```javascript
import fs from 'fs';

fs.mkdirSync('downloads', {
    recursive: true
});
```

Example:

```javascript
import fs from 'fs';

fs.mkdirSync('downloads', {
    recursive: true
});

const downloadPromise = page.waitForEvent('download');

await page.getByRole('button', { name: 'Download' }).click();

const download = await downloadPromise;

await download.saveAs('downloads/report.pdf');
```

---

# 25. Using `path.join()`

For cross-platform file paths, use Node.js `path`.

```javascript
import path from 'path';

const filePath = path.join(
    'downloads',
    'report.pdf'
);

await download.saveAs(filePath);
```

This is preferable to manually concatenating paths.

---

# 26. Recommended Cross-Platform Example

```javascript
import { test, expect } from '@playwright/test';
import fs from 'fs';
import path from 'path';

test('Download report', async ({ page }) => {
    await page.goto('https://example.com');

    const downloadPromise = page.waitForEvent('download');

    await page.getByRole('button', { name: 'Download Report' }).click();

    const download = await downloadPromise;

    const filename = download.suggestedFilename();

    const downloadDir = path.join(
        process.cwd(),
        'downloads'
    );

    fs.mkdirSync(downloadDir, {
        recursive: true
    });

    const filePath = path.join(
        downloadDir,
        filename
    );

    await download.saveAs(filePath);

    expect(fs.existsSync(filePath)).toBeTruthy();
});
```

---

# 27. Handling Multiple Downloads

If clicking one button triggers multiple downloads, listen for multiple download events.

```javascript
const downloads = [];

page.on('download', download => {
    downloads.push(download);
});

await page.getByRole('button', {
    name: 'Download All'
}).click();
```

However, if the application intentionally triggers multiple downloads, it is usually better to use `waitForEvent()` for each expected download.

---

# 28. Multiple Downloads with `Promise.all`

Example:

```javascript
const download1Promise = page.waitForEvent('download');

await page.getByText('Download File 1').click();

const download1 = await download1Promise;

const download2Promise = page.waitForEvent('download');

await page.getByText('Download File 2').click();

const download2 = await download2Promise;
```

Save both:

```javascript
await download1.saveAs('downloads/file1.pdf');
await download2.saveAs('downloads/file2.pdf');
```

---

# 29. Download Triggered by a Link

```javascript
const downloadPromise = page.waitForEvent('download');

await page.locator('a.download-link').click();

const download = await downloadPromise;

await download.saveAs('downloads/file.pdf');
```

---

# 30. Download Triggered by a Button

```javascript
const downloadPromise = page.waitForEvent('download');

await page.getByRole('button', {
    name: 'Download'
}).click();

const download = await downloadPromise;

await download.saveAs('downloads/file.pdf');
```

---

# 31. Download Triggered by Text

```javascript
const downloadPromise = page.waitForEvent('download');

await page.getByText('Download Report').click();

const download = await downloadPromise;
```

---

# 32. Download Triggered by Locator

```javascript
const downloadPromise = page.waitForEvent('download');

await page.locator('#downloadButton').click();

const download = await downloadPromise;
```

---

# 33. Download from a New Page

If clicking a link opens a new page and that page initiates the download, wait for both events as appropriate.

```javascript
const pagePromise = context.waitForEvent('page');

const downloadPromise = page.waitForEvent('download');

await page.getByRole('link', {
    name: 'Download'
}).click();

const newPage = await pagePromise;

const download = await downloadPromise;
```

---

# 34. Browser Context Download Behavior

Playwright browser contexts support download handling.

Example:

```javascript
const context = await browser.newContext({
    acceptDownloads: true
});
```

Then:

```javascript
const page = await context.newPage();
```

In modern Playwright usage, downloads are supported by default in normal test contexts, but explicitly configuring `acceptDownloads` can be useful in custom browser-context setups.

---

# 35. Download in a Custom Browser Context

```javascript
import { chromium } from 'playwright';

const browser = await chromium.launch();

const context = await browser.newContext({
    acceptDownloads: true
});

const page = await context.newPage();

await page.goto('https://example.com');

const downloadPromise = page.waitForEvent('download');

await page.getByText('Download').click();

const download = await downloadPromise;

await download.saveAs('downloads/report.pdf');

await browser.close();
```

---

# 36. Download with `test` Fixture

Recommended Playwright Test approach:

```javascript
import { test, expect } from '@playwright/test';

test('download file', async ({ page }) => {
    await page.goto('https://example.com');

    const downloadPromise = page.waitForEvent('download');

    await page.getByRole('button', {
        name: 'Download'
    }).click();

    const download = await downloadPromise;

    expect(download.suggestedFilename())
        .toBe('report.pdf');

    await download.saveAs(
        'downloads/report.pdf'
    );
});
```

---

# 37. Download Path Using `path()`

```javascript
const downloadPromise = page.waitForEvent('download');

await page.getByRole('button', {
    name: 'Download'
}).click();

const download = await downloadPromise;

const temporaryPath = await download.path();

console.log(temporaryPath);
```

---

# 38. Read File Directly from Download Path

```javascript
import fs from 'fs';

const downloadPath = await download.path();

const content = fs.readFileSync(
    downloadPath,
    'utf-8'
);

console.log(content);
```

For long-term test reliability, saving the file to a controlled location is often preferable.

---

# 39. Delete Downloaded File

After the test:

```javascript
fs.unlinkSync(filePath);
```

Example:

```javascript
await download.saveAs(filePath);

expect(fs.existsSync(filePath)).toBeTruthy();

fs.unlinkSync(filePath);

expect(fs.existsSync(filePath)).toBeFalsy();
```

---

# 40. Using `finally` for Cleanup

A good practice is to clean up downloaded files even if an assertion fails.

```javascript
try {
    await download.saveAs(filePath);

    expect(fs.existsSync(filePath)).toBeTruthy();
} finally {
    if (fs.existsSync(filePath)) {
        fs.unlinkSync(filePath);
    }
}
```

---

# 41. Download Test with Cleanup

```javascript
import { test, expect } from '@playwright/test';
import fs from 'fs';
import path from 'path';

test('Download and validate report', async ({ page }) => {
    const filePath = path.join(
        process.cwd(),
        'downloads',
        'report.pdf'
    );

    try {
        await page.goto('https://example.com');

        const downloadPromise =
            page.waitForEvent('download');

        await page.getByRole('button', {
            name: 'Download Report'
        }).click();

        const download = await downloadPromise;

        expect(await download.failure()).toBeNull();

        await download.saveAs(filePath);

        expect(fs.existsSync(filePath)).toBeTruthy();

        const stats = fs.statSync(filePath);

        expect(stats.size).toBeGreaterThan(0);
    } finally {
        if (fs.existsSync(filePath)) {
            fs.unlinkSync(filePath);
        }
    }
});
```

---

# 42. Downloaded PDF Validation

For basic validation:

```javascript
const filename = download.suggestedFilename();

expect(filename).toMatch(/\.pdf$/);

await download.saveAs(filePath);

const stats = fs.statSync(filePath);

expect(stats.size).toBeGreaterThan(0);
```

For deeper PDF content validation, a PDF parsing library can be used.

---

# 43. Downloaded CSV Validation

```javascript
await download.saveAs(filePath);

const csv = fs.readFileSync(
    filePath,
    'utf-8'
);

expect(csv).toContain('Customer Name');
expect(csv).toContain('Order ID');
expect(csv).toContain('Status');
```

---

# 44. Downloaded JSON Validation

```javascript
await download.saveAs(filePath);

const jsonText = fs.readFileSync(
    filePath,
    'utf-8'
);

const json = JSON.parse(jsonText);

expect(json.status).toBe('success');
expect(json.data).toBeDefined();
```

---

# 45. Downloaded XML Validation

```javascript
await download.saveAs(filePath);

const xml = fs.readFileSync(
    filePath,
    'utf-8'
);

expect(xml).toContain('<Customer>');
```

---

# 46. Verify File Name Pattern

For dynamically generated names:

```javascript
const filename = download.suggestedFilename();

expect(filename).toMatch(
    /^report-\d{8}\.pdf$/
);
```

Example expected name:

```text
report-20260819.pdf
```

---

# 47. Verify Timestamped File

```javascript
const filename = download.suggestedFilename();

expect(filename).toContain('report');
expect(filename).toMatch(/\.pdf$/);
```

This is useful when the application generates a unique file name.

---

# 48. Verify File Type

You can verify the extension:

```javascript
expect(download.suggestedFilename())
    .toMatch(/\.pdf$/);
```

You can also inspect the file contents or MIME information when deeper validation is required.

---

# 49. Download Test with Page Object Model

### Page Object

```javascript
export class ReportsPage {
    constructor(page) {
        this.page = page;
        this.downloadButton = page.getByRole(
            'button',
            { name: 'Download Report' }
        );
    }

    async downloadReport() {
        const downloadPromise =
            this.page.waitForEvent('download');

        await this.downloadButton.click();

        return await downloadPromise;
    }
}
```

### Test

```javascript
import { test, expect } from '@playwright/test';
import { ReportsPage } from './pages/ReportsPage';

test('Download report', async ({ page }) => {
    const reportsPage = new ReportsPage(page);

    await page.goto('https://example.com');

    const download =
        await reportsPage.downloadReport();

    expect(download.suggestedFilename())
        .toBe('report.pdf');

    await download.saveAs(
        'downloads/report.pdf'
    );
});
```

---

# 50. Reusable Download Helper

Create a utility:

```javascript
export async function downloadFile(
    page,
    locator,
    filePath
) {
    const downloadPromise =
        page.waitForEvent('download');

    await locator.click();

    const download =
        await downloadPromise;

    await download.saveAs(filePath);

    return download;
}
```

Use it:

```javascript
const download = await downloadFile(
    page,
    page.getByRole('button', {
        name: 'Download'
    }),
    'downloads/report.pdf'
);
```

---

# 51. Reusable Download Validation Helper

```javascript
import fs from 'fs';

export async function validateDownload(
    download,
    filePath
) {
    expect(await download.failure()).toBeNull();

    await download.saveAs(filePath);

    expect(fs.existsSync(filePath)).toBeTruthy();

    const stats = fs.statSync(filePath);

    expect(stats.size).toBeGreaterThan(0);
}
```

---

# 52. Download Utility Example

```javascript
import fs from 'fs';
import path from 'path';

export async function saveDownload(
    page,
    locator,
    directory
) {
    fs.mkdirSync(directory, {
        recursive: true
    });

    const downloadPromise =
        page.waitForEvent('download');

    await locator.click();

    const download =
        await downloadPromise;

    const filename =
        download.suggestedFilename();

    const filePath =
        path.join(directory, filename);

    await download.saveAs(filePath);

    return {
        download,
        filename,
        filePath
    };
}
```

Usage:

```javascript
const result = await saveDownload(
    page,
    page.getByRole('button', {
        name: 'Download'
    }),
    'downloads'
);

console.log(result.filename);
console.log(result.filePath);
```

---

# 53. Downloads in CI/CD

Downloaded files can be useful for:

* Test evidence
* Report validation
* API response verification
* Export validation
* Regression testing

However, avoid storing unnecessary downloaded files in the repository.

Recommended:

```text
downloads/
```

Add temporary downloads to `.gitignore`:

```text
downloads/
```

---

# 54. Download Directory Structure

A project can use:

```text
playwright-project/
│
├── tests/
│   └── downloads.spec.js
│
├── pages/
│   └── ReportsPage.js
│
├── utils/
│   └── downloadHelper.js
│
├── downloads/
│
├── playwright.config.js
└── package.json
```

---

# 55. Using `process.cwd()`

Instead of relying on a relative path:

```javascript
const filePath = path.join(
    process.cwd(),
    'downloads',
    'report.pdf'
);
```

This makes the location relative to the project root from which the test is executed.

---

# 56. Download Test with Dynamic File Name

```javascript
test('Download dynamic report', async ({ page }) => {
    await page.goto('https://example.com');

    const downloadPromise =
        page.waitForEvent('download');

    await page.getByRole('button', {
        name: 'Download'
    }).click();

    const download =
        await downloadPromise;

    const filename =
        download.suggestedFilename();

    expect(filename).toMatch(
        /^report-.*\.pdf$/
    );

    const filePath = path.join(
        process.cwd(),
        'downloads',
        filename
    );

    await download.saveAs(filePath);

    expect(fs.existsSync(filePath))
        .toBeTruthy();
});
```

---

# 57. Download from a Menu

```javascript
await page.getByRole('button', {
    name: 'Export'
}).click();

const downloadPromise =
    page.waitForEvent('download');

await page.getByText('PDF').click();

const download =
    await downloadPromise;

await download.saveAs(
    'downloads/report.pdf'
);
```

---

# 58. Download from a Dropdown

```javascript
await page.getByLabel('Export Format')
    .selectOption('pdf');

const downloadPromise =
    page.waitForEvent('download');

await page.getByRole('button', {
    name: 'Export'
}).click();

const download =
    await downloadPromise;

await download.saveAs(
    'downloads/report.pdf'
);
```

---

# 59. Download from an Export Workflow

```javascript
test('Export customer report', async ({ page }) => {
    await page.goto('https://example.com');

    await page.getByRole('button', {
        name: 'Export'
    }).click();

    await page.getByText('CSV').click();

    const downloadPromise =
        page.waitForEvent('download');

    await page.getByRole('button', {
        name: 'Generate'
    }).click();

    const download =
        await downloadPromise;

    expect(download.suggestedFilename())
        .toMatch(/\.csv$/);

    await download.saveAs(
        'downloads/customers.csv'
    );
});
```

---

# 60. Common Mistake: Waiting After Click

Incorrect:

```javascript
await page.getByText('Download').click();

const download =
    await page.waitForEvent('download');
```

Correct:

```javascript
const downloadPromise =
    page.waitForEvent('download');

await page.getByText('Download').click();

const download =
    await downloadPromise;
```

---

# 61. Common Mistake: Not Saving the File

This only waits for the download:

```javascript
const download =
    await downloadPromise;
```

If you need the file for validation, save it:

```javascript
await download.saveAs(
    'downloads/report.pdf'
);
```

---

# 62. Common Mistake: Hard-Coding Temporary Download Paths

Avoid relying on:

```javascript
const path = await download.path();
```

for long-term test storage.

Prefer:

```javascript
await download.saveAs(
    'downloads/report.pdf'
);
```

This gives your test a predictable location.

---

# 63. Common Mistake: Not Checking Download Failure

Add:

```javascript
expect(await download.failure()).toBeNull();
```

This ensures the download did not fail.

---

# 64. Common Mistake: Not Validating File Size

A file can exist but still be empty.

Use:

```javascript
const stats = fs.statSync(filePath);

expect(stats.size).toBeGreaterThan(0);
```

---

# 65. Common Mistake: Using Arbitrary Sleeps

Avoid:

```javascript
await page.waitForTimeout(5000);
```

before checking the download.

Instead:

```javascript
const downloadPromise =
    page.waitForEvent('download');

await page.getByText('Download').click();

const download =
    await downloadPromise;
```

Playwright waits for the actual download event.

---

# 66. Senior-Level Download Test

```javascript
import { test, expect } from '@playwright/test';
import fs from 'fs';
import path from 'path';

test('Validate customer report download', async ({ page }) => {
    const downloadDir = path.join(
        process.cwd(),
        'downloads'
    );

    fs.mkdirSync(downloadDir, {
        recursive: true
    });

    await page.goto(
        'https://example.com/reports'
    );

    const downloadPromise =
        page.waitForEvent('download', {
            timeout: 30000
        });

    await page.getByRole('button', {
        name: 'Export Customer Report'
    }).click();

    const download =
        await downloadPromise;

    expect(await download.failure())
        .toBeNull();

    const filename =
        download.suggestedFilename();

    expect(filename).toMatch(
        /^customer-report-.*\.csv$/
    );

    const filePath =
        path.join(
            downloadDir,
            filename
        );

    await download.saveAs(filePath);

    expect(fs.existsSync(filePath))
        .toBeTruthy();

    const stats =
        fs.statSync(filePath);

    expect(stats.size)
        .toBeGreaterThan(0);

    const content =
        fs.readFileSync(
            filePath,
            'utf-8'
        );

    expect(content)
        .toContain('Customer');

    expect(content)
        .toContain('Status');
});
```

---

# 67. Download vs Upload

Downloads and uploads are different operations.

### Upload

Use:

```javascript
setInputFiles()
```

Example:

```javascript
await page.locator(
    'input[type="file"]'
).setInputFiles(
    'files/test.pdf'
);
```

### Download

Use:

```javascript
page.waitForEvent('download')
```

Example:

```javascript
const downloadPromise =
    page.waitForEvent('download');

await page.getByText('Download').click();

const download =
    await downloadPromise;
```

---

# 68. Download vs API Download

Sometimes a file is downloaded directly through an API endpoint.

For browser-triggered downloads:

```javascript
page.waitForEvent('download')
```

For direct HTTP/API testing, Playwright's `request` fixture can be used.

Example:

```javascript
const response =
    await request.get(
        'https://example.com/report.pdf'
    );

expect(response.ok()).toBeTruthy();

const body =
    await response.body();
```

Browser download testing verifies the user's UI workflow.

API download testing verifies the backend endpoint directly.

---

# 69. Download Testing Strategy

A strong download test should verify:

```text
1. User can trigger download
2. Download event occurs
3. Download does not fail
4. Correct file name is generated
5. Correct extension is used
6. File is saved successfully
7. File exists
8. File is not empty
9. File contents are correct
10. Temporary test data is cleaned up
```

---

# 70. Recommended Download Test Pattern

```javascript
const downloadPromise =
    page.waitForEvent('download');

await downloadButton.click();

const download =
    await downloadPromise;

expect(await download.failure())
    .toBeNull();

const filename =
    download.suggestedFilename();

const filePath =
    path.join(
        downloadDirectory,
        filename
    );

await download.saveAs(filePath);

expect(fs.existsSync(filePath))
    .toBeTruthy();

expect(
    fs.statSync(filePath).size
).toBeGreaterThan(0);
```

---

# 71. Playwright Download API Cheat Sheet

| API                             | Purpose                     |
| ------------------------------- | --------------------------- |
| `page.waitForEvent('download')` | Wait for download           |
| `download.suggestedFilename()`  | Get suggested file name     |
| `download.path()`               | Get temporary download path |
| `download.saveAs()`             | Save download               |
| `download.failure()`            | Check download failure      |
| `download.delete()`             | Delete downloaded file      |
| `download.createReadStream()`   | Read download as stream     |

---

# 72. Important Download Methods

### `suggestedFilename()`

```javascript
download.suggestedFilename();
```

Returns the suggested file name.

### `path()`

```javascript
await download.path();
```

Returns the temporary path.

### `saveAs()`

```javascript
await download.saveAs(
    'downloads/report.pdf'
);
```

Saves the file.

### `failure()`

```javascript
await download.failure();
```

Returns download failure information.

### `delete()`

```javascript
await download.delete();
```

Deletes the downloaded file.

### `createReadStream()`

```javascript
const stream =
    await download.createReadStream();
```

Provides access to the downloaded data as a stream.

---

# 73. Delete Download Using Playwright

Instead of Node.js:

```javascript
await download.delete();
```

Example:

```javascript
const downloadPromise =
    page.waitForEvent('download');

await page.getByText('Download').click();

const download =
    await downloadPromise;

await download.saveAs(
    'downloads/report.pdf'
);

await download.delete();
```

---

# 74. Download as Stream

```javascript
const stream =
    await download.createReadStream();
```

You can use Node.js stream APIs to process the downloaded data.

This is useful when you need to inspect downloaded content without necessarily relying on a permanent file.

---

# 75. Interview Questions

## Q1. How do you handle file downloads in Playwright?

Use the `download` event:

```javascript
const downloadPromise =
    page.waitForEvent('download');

await page.getByText('Download').click();

const download =
    await downloadPromise;
```

Then save:

```javascript
await download.saveAs(
    'downloads/file.pdf'
);
```

---

## Q2. Why should `waitForEvent('download')` be called before clicking?

Because the download event may occur immediately after the click. Starting the listener first prevents the test from missing the event.

---

## Q3. How do you get the downloaded file name?

```javascript
const filename =
    download.suggestedFilename();
```

---

## Q4. How do you save a downloaded file?

```javascript
await download.saveAs(
    'downloads/report.pdf'
);
```

---

## Q5. How do you check whether a download failed?

```javascript
const failure =
    await download.failure();

expect(failure).toBeNull();
```

---

## Q6. How do you get the temporary download path?

```javascript
const filePath =
    await download.path();
```

---

## Q7. How do you verify a downloaded file exists?

```javascript
expect(
    fs.existsSync(filePath)
).toBeTruthy();
```

---

## Q8. How do you verify the downloaded file is not empty?

```javascript
const stats =
    fs.statSync(filePath);

expect(stats.size)
    .toBeGreaterThan(0);
```

---

## Q9. How do you validate the downloaded file name?

```javascript
expect(
    download.suggestedFilename()
).toBe('report.pdf');
```

---

## Q10. How do you validate a dynamic file name?

```javascript
expect(
    download.suggestedFilename()
).toMatch(
    /^report-.*\.pdf$/
);
```

---

## Q11. How do you read a downloaded text file?

```javascript
const content =
    fs.readFileSync(
        filePath,
        'utf-8'
    );
```

---

## Q12. How do you validate a downloaded JSON file?

```javascript
const content =
    fs.readFileSync(
        filePath,
        'utf-8'
    );

const json =
    JSON.parse(content);

expect(json.status)
    .toBe('success');
```

---

## Q13. How do you download a PDF and validate it?

```javascript
const downloadPromise =
    page.waitForEvent('download');

await page.getByText('Download PDF').click();

const download =
    await downloadPromise;

expect(
    download.suggestedFilename()
).toMatch(/\.pdf$/);

await download.saveAs(
    'downloads/report.pdf'
);
```

Then validate file existence and size.

---

## Q14. How do you avoid arbitrary waits for downloads?

Do not use:

```javascript
await page.waitForTimeout(5000);
```

Use:

```javascript
const downloadPromise =
    page.waitForEvent('download');

await downloadButton.click();

const download =
    await downloadPromise;
```

---

## Q15. What is the difference between `path()` and `saveAs()`?

`path()` returns the temporary path managed by Playwright.

`saveAs()` saves the downloaded file to a location chosen by the test.

---

# 76. Best Practices

### 1. Start the event listener before the action

```javascript
const downloadPromise =
    page.waitForEvent('download');

await button.click();

const download =
    await downloadPromise;
```

### 2. Check download failure

```javascript
expect(
    await download.failure()
).toBeNull();
```

### 3. Validate the file name

```javascript
expect(
    download.suggestedFilename()
).toMatch(/\.pdf$/);
```

### 4. Save to a controlled directory

```javascript
await download.saveAs(
    'downloads/report.pdf'
);
```

### 5. Verify the file exists

```javascript
expect(
    fs.existsSync(filePath)
).toBeTruthy();
```

### 6. Verify file size

```javascript
expect(
    fs.statSync(filePath).size
).toBeGreaterThan(0);
```

### 7. Validate file content when required

```javascript
const content =
    fs.readFileSync(
        filePath,
        'utf-8'
    );

expect(content)
    .toContain('Customer');
```

### 8. Clean up test files

```javascript
fs.unlinkSync(filePath);
```

---

# 77. Real-World Automation Example

A typical enterprise automation test might look like this:

```javascript
import { test, expect } from '@playwright/test';
import fs from 'fs';
import path from 'path';

test('Verify customer report download', async ({ page }) => {

    const downloadDirectory = path.join(
        process.cwd(),
        'downloads'
    );

    fs.mkdirSync(downloadDirectory, {
        recursive: true
    });

    await page.goto(
        'https://example.com/customer-reports'
    );

    await page.getByRole('button', {
        name: 'Generate Report'
    }).click();

    const downloadPromise =
        page.waitForEvent('download');

    await page.getByRole('button', {
        name: 'Download CSV'
    }).click();

    const download =
        await downloadPromise;

    expect(await download.failure())
        .toBeNull();

    const filename =
        download.suggestedFilename();

    expect(filename)
        .toMatch(/\.csv$/);

    const filePath =
        path.join(
            downloadDirectory,
            filename
        );

    await download.saveAs(filePath);

    expect(fs.existsSync(filePath))
        .toBeTruthy();

    const stats =
        fs.statSync(filePath);

    expect(stats.size)
        .toBeGreaterThan(0);

    const content =
        fs.readFileSync(
            filePath,
            'utf-8'
        );

    expect(content)
        .toContain('Customer');

    expect(content)
        .toContain('Status');

    fs.unlinkSync(filePath);
});
```

---

# 78. Final Summary

Playwright provides a simple and reliable API for browser downloads.

The core pattern is:

```javascript
const downloadPromise =
    page.waitForEvent('download');

await page.getByRole('button', {
    name: 'Download'
}).click();

const download =
    await downloadPromise;
```

Then:

```javascript
await download.saveAs(
    'downloads/report.pdf'
);
```

Useful validation:

```javascript
expect(await download.failure())
    .toBeNull();

expect(download.suggestedFilename())
    .toMatch(/\.pdf$/);

expect(fs.existsSync(filePath))
    .toBeTruthy();

expect(
    fs.statSync(filePath).size
).toBeGreaterThan(0);
```

The key Playwright download APIs to remember are:

```text
page.waitForEvent('download')
download.suggestedFilename()
download.path()
download.saveAs()
download.failure()
download.delete()
download.createReadStream()
```

For senior-level Playwright automation, don't stop at verifying that a download event occurred. Validate the **file name, file type, file existence, file size, file content, and cleanup** where appropriate.
