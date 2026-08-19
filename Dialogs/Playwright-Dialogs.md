# Playwright Dialogs

## 1. Introduction

Browser dialogs are native browser popups that require special handling in Playwright.

Common browser dialogs are:

* `alert()`
* `confirm()`
* `prompt()`
* `beforeunload`

Playwright provides the `dialog` event to handle these dialogs.

---

## 2. Types of Browser Dialogs

### 2.1 Alert

An alert displays a message and provides an **OK** button.

```javascript
alert("This is an alert");
```

Example:

```html
<button onclick="alert('Hello Playwright')">Click Me</button>
```

---

### 2.2 Confirm

A confirm dialog provides:

* OK
* Cancel

```javascript
confirm("Are you sure?");
```

The result is:

* `true` when OK is selected
* `false` when Cancel is selected

---

### 2.3 Prompt

A prompt dialog asks the user to enter a value.

```javascript
prompt("Enter your name:");
```

The user can:

* Enter a value and click OK
* Click Cancel

---

### 2.4 BeforeUnload

A `beforeunload` dialog can appear when a page is being closed or navigated away from.

Playwright can also handle this type of dialog.

---

# 3. Playwright Dialog Event

Playwright exposes the `dialog` event:

```javascript
page.on('dialog', async dialog => {
    await dialog.accept();
});
```

The dialog object provides useful methods and properties.

---

# 4. Important Dialog Methods

| Method                  | Purpose                  |
| ----------------------- | ------------------------ |
| `dialog.accept()`       | Accept the dialog        |
| `dialog.dismiss()`      | Dismiss the dialog       |
| `dialog.message()`      | Get dialog message       |
| `dialog.type()`         | Get dialog type          |
| `dialog.defaultValue()` | Get default prompt value |

---

# 5. Accept an Alert

Example:

```javascript
page.on('dialog', async dialog => {
    console.log(dialog.message());
    await dialog.accept();
});

await page.getByRole('button', { name: 'Show Alert' }).click();
```

When the alert appears:

1. Playwright detects the dialog.
2. The event handler executes.
3. The message is printed.
4. The dialog is accepted.

---

# 6. Dismiss an Alert

A dialog can be dismissed using:

```javascript
await dialog.dismiss();
```

Example:

```javascript
page.on('dialog', async dialog => {
    console.log(dialog.message());
    await dialog.dismiss();
});

await page.getByRole('button', { name: 'Show Alert' }).click();
```

---

# 7. Handle Confirm Dialog

Example:

```javascript
page.on('dialog', async dialog => {
    console.log(dialog.type());
    console.log(dialog.message());

    await dialog.accept();
});

await page.getByRole('button', { name: 'Delete' }).click();
```

This clicks **OK** on the confirmation dialog.

---

# 8. Dismiss Confirm Dialog

To click **Cancel**:

```javascript
page.on('dialog', async dialog => {
    await dialog.dismiss();
});

await page.getByRole('button', { name: 'Delete' }).click();
```

---

# 9. Handle Prompt Dialog

Prompt dialogs can be accepted with text.

```javascript
page.on('dialog', async dialog => {
    await dialog.accept('Selva');
});

await page.getByRole('button', { name: 'Enter Name' }).click();
```

The value `Selva` is entered into the prompt.

---

# 10. Get Dialog Message

Use:

```javascript
dialog.message()
```

Example:

```javascript
page.on('dialog', async dialog => {
    console.log(`Dialog message: ${dialog.message()}`);

    await dialog.accept();
});
```

---

# 11. Get Dialog Type

Use:

```javascript
dialog.type()
```

Example:

```javascript
page.on('dialog', async dialog => {
    console.log(`Dialog type: ${dialog.type()}`);

    await dialog.accept();
});
```

Possible values include:

```text
alert
confirm
prompt
beforeunload
```

---

# 12. Get Prompt Default Value

For a prompt dialog:

```javascript
page.on('dialog', async dialog => {
    console.log(`Default value: ${dialog.defaultValue()}`);

    await dialog.accept('New Value');
});
```

---

# 13. Handle Dialog with `page.once()`

If the dialog needs to be handled only once, `page.once()` is useful.

```javascript
page.once('dialog', async dialog => {
    await dialog.accept();
});

await page.getByRole('button', { name: 'Show Alert' }).click();
```

After the first dialog, the handler is removed.

---

# 14. `page.on()` vs `page.once()`

### `page.on()`

Handles every matching dialog event.

```javascript
page.on('dialog', async dialog => {
    await dialog.accept();
});
```

Use this when multiple dialogs may appear.

### `page.once()`

Handles only the next dialog.

```javascript
page.once('dialog', async dialog => {
    await dialog.accept();
});
```

Use this when only one dialog is expected.

---

# 15. Important: Register Dialog Handler Before the Action

The dialog handler should normally be registered **before** performing the action that triggers the dialog.

Correct:

```javascript
page.once('dialog', async dialog => {
    await dialog.accept();
});

await page.getByRole('button', { name: 'Delete' }).click();
```

Avoid:

```javascript
await page.getByRole('button', { name: 'Delete' }).click();

page.once('dialog', async dialog => {
    await dialog.accept();
});
```

The second example can cause the test to hang because the dialog may already be waiting for handling.

---

# 16. Handle Dialog and Validate Message

Example:

```javascript
page.once('dialog', async dialog => {
    expect(dialog.message()).toBe('Are you sure you want to delete?');

    await dialog.accept();
});

await page.getByRole('button', { name: 'Delete' }).click();
```

This verifies the dialog message before accepting it.

---

# 17. Validate Dialog Type

```javascript
page.once('dialog', async dialog => {
    expect(dialog.type()).toBe('confirm');

    await dialog.accept();
});

await page.getByRole('button', { name: 'Delete' }).click();
```

---

# 18. Validate Prompt Default Value

```javascript
page.once('dialog', async dialog => {
    expect(dialog.type()).toBe('prompt');
    expect(dialog.defaultValue()).toBe('Guest');

    await dialog.accept('Selva');
});

await page.getByRole('button', { name: 'Enter Name' }).click();
```

---

# 19. Complete Alert Example

```javascript
import { test, expect } from '@playwright/test';

test('Handle alert dialog', async ({ page }) => {

    await page.goto('https://example.com');

    page.once('dialog', async dialog => {

        console.log(`Type: ${dialog.type()}`);
        console.log(`Message: ${dialog.message()}`);

        expect(dialog.type()).toBe('alert');
        expect(dialog.message()).toBe('Hello Playwright');

        await dialog.accept();
    });

    await page.getByRole('button', { name: 'Show Alert' }).click();
});
```

---

# 20. Complete Confirm Example

```javascript
import { test, expect } from '@playwright/test';

test('Accept confirm dialog', async ({ page }) => {

    await page.goto('https://example.com');

    page.once('dialog', async dialog => {

        expect(dialog.type()).toBe('confirm');
        expect(dialog.message()).toBe('Are you sure?');

        await dialog.accept();
    });

    await page.getByRole('button', { name: 'Delete' }).click();
});
```

---

# 21. Confirm Dialog - Cancel

```javascript
import { test } from '@playwright/test';

test('Dismiss confirm dialog', async ({ page }) => {

    await page.goto('https://example.com');

    page.once('dialog', async dialog => {
        await dialog.dismiss();
    });

    await page.getByRole('button', { name: 'Delete' }).click();
});
```

---

# 22. Complete Prompt Example

```javascript
import { test, expect } from '@playwright/test';

test('Handle prompt dialog', async ({ page }) => {

    await page.goto('https://example.com');

    page.once('dialog', async dialog => {

        expect(dialog.type()).toBe('prompt');

        console.log(`Message: ${dialog.message()}`);
        console.log(`Default: ${dialog.defaultValue()}`);

        await dialog.accept('Selva');
    });

    await page.getByRole('button', { name: 'Enter Name' }).click();
});
```

---

# 23. Dialog Handling with `waitForEvent`

Playwright can also explicitly wait for the dialog event.

```javascript
const dialogPromise = page.waitForEvent('dialog');

await page.getByRole('button', { name: 'Show Alert' }).click();

const dialog = await dialogPromise;

console.log(dialog.message());

await dialog.accept();
```

This approach is useful when you want direct access to the dialog object in the test flow.

---

# 24. `waitForEvent('dialog')` with Validation

```javascript
const dialogPromise = page.waitForEvent('dialog');

await page.getByRole('button', { name: 'Delete' }).click();

const dialog = await dialogPromise;

expect(dialog.type()).toBe('confirm');
expect(dialog.message()).toContain('delete');

await dialog.accept();
```

---

# 25. Dialog Handling with Timeout

You can specify a timeout when waiting for a dialog:

```javascript
const dialogPromise = page.waitForEvent('dialog', {
    timeout: 5000
});

await page.getByRole('button', { name: 'Show Alert' }).click();

const dialog = await dialogPromise;

await dialog.accept();
```

If the dialog does not appear within the timeout, the test fails.

---

# 26. Multiple Dialogs

If multiple dialogs are expected, `page.on()` can be used.

```javascript
page.on('dialog', async dialog => {

    console.log(dialog.message());

    await dialog.accept();
});

await page.getByRole('button', { name: 'Start Process' }).click();
```

The handler remains active for subsequent dialogs.

---

# 27. Handling Different Dialog Types

You can inspect the dialog type and handle each type differently.

```javascript
page.on('dialog', async dialog => {

    switch (dialog.type()) {

        case 'alert':
            await dialog.accept();
            break;

        case 'confirm':
            await dialog.dismiss();
            break;

        case 'prompt':
            await dialog.accept('Selva');
            break;

        default:
            await dialog.dismiss();
    }
});
```

---

# 28. Conditional Dialog Handling

Example:

```javascript
page.on('dialog', async dialog => {

    if (dialog.type() === 'confirm') {

        if (dialog.message().includes('Delete')) {
            await dialog.accept();
        } else {
            await dialog.dismiss();
        }

    } else {
        await dialog.accept();
    }
});
```

---

# 29. Dialog Handler in `beforeEach`

If many tests need the same dialog behavior, you can configure it in a hook.

```javascript
import { test } from '@playwright/test';

test.beforeEach(async ({ page }) => {

    page.on('dialog', async dialog => {
        await dialog.accept();
    });

});

test('Test 1', async ({ page }) => {

    await page.goto('https://example.com');

    // Test steps
});
```

---

# 30. Dialog Handling in a Fixture

A reusable fixture can handle common dialogs.

```javascript
import { test as base } from '@playwright/test';

export const test = base.extend({

    dialogHandler: async ({ page }, use) => {

        page.on('dialog', async dialog => {
            await dialog.accept();
        });

        await use();
    }

});
```

Test:

```javascript
import { test } from './fixtures';

test('Dialog test', async ({ page }) => {

    await page.goto('https://example.com');

    // Dialog is automatically handled
});
```

---

# 31. Dialog Handling with Page Object Model

A page object can contain the action that triggers the dialog.

```javascript
export class DeletePage {

    constructor(page) {
        this.page = page;
        this.deleteButton = page.getByRole('button', {
            name: 'Delete'
        });
    }

    async deleteItem() {

        this.page.once('dialog', async dialog => {
            await dialog.accept();
        });

        await this.deleteButton.click();
    }
}
```

Test:

```javascript
import { test } from '@playwright/test';
import { DeletePage } from './pages/DeletePage';

test('Delete item', async ({ page }) => {

    const deletePage = new DeletePage(page);

    await deletePage.deleteItem();
});
```

---

# 32. Dialog Handling in TypeScript

```typescript
import { test, expect } from '@playwright/test';

test('Handle alert', async ({ page }) => {

    page.once('dialog', async dialog => {

        console.log(dialog.type());
        console.log(dialog.message());

        expect(dialog.type()).toBe('alert');

        await dialog.accept();
    });

    await page.getByRole('button', { name: 'Show Alert' }).click();
});
```

---

# 33. TypeScript Dialog Handler

You can explicitly type the dialog:

```typescript
import { Dialog } from '@playwright/test';

page.once('dialog', async (dialog: Dialog) => {
    await dialog.accept();
});
```

---

# 34. Handling Dialogs Without a Handler

Playwright automatically dismisses dialogs when there is no listener registered.

However, relying on this behavior is generally not recommended when the dialog is part of the expected application flow.

For important dialog validation, explicitly register a handler:

```javascript
page.once('dialog', async dialog => {
    expect(dialog.message()).toBe('Expected message');
    await dialog.accept();
});
```

---

# 35. Common Mistake - Handler Registered Too Late

Incorrect:

```javascript
await page.getByText('Delete').click();

page.on('dialog', async dialog => {
    await dialog.accept();
});
```

Better:

```javascript
page.once('dialog', async dialog => {
    await dialog.accept();
});

await page.getByText('Delete').click();
```

---

# 36. Common Mistake - Forgetting `accept()` or `dismiss()`

Incorrect:

```javascript
page.on('dialog', async dialog => {
    console.log(dialog.message());
});
```

The dialog must be handled.

Correct:

```javascript
page.on('dialog', async dialog => {
    console.log(dialog.message());
    await dialog.accept();
});
```

---

# 37. Common Mistake - Using Locator APIs for Native Dialogs

Native browser dialogs are not normal DOM elements.

Do not try:

```javascript
await page.locator('.alert').click();
```

Instead:

```javascript
page.once('dialog', async dialog => {
    await dialog.accept();
});
```

---

# 38. Common Mistake - Handling a Prompt Like an Alert

Incorrect:

```javascript
page.once('dialog', async dialog => {
    await dialog.accept();
});
```

This accepts the prompt without providing a value.

To enter text:

```javascript
page.once('dialog', async dialog => {
    await dialog.accept('Selva');
});
```

---

# 39. Dialog Message Validation

A strong automation test should validate the message when the message is part of the requirement.

```javascript
page.once('dialog', async dialog => {

    const message = dialog.message();

    expect(message).toContain('Are you sure');

    await dialog.accept();
});
```

---

# 40. Dialog Type Validation

```javascript
page.once('dialog', async dialog => {

    expect(dialog.type()).toBe('confirm');

    await dialog.accept();
});
```

---

# 41. Reusable Dialog Utility

Create a utility:

```javascript
export async function acceptDialog(page) {

    page.once('dialog', async dialog => {
        await dialog.accept();
    });
}
```

Usage:

```javascript
await acceptDialog(page);

await page.getByRole('button', { name: 'Delete' }).click();
```

---

# 42. Reusable Prompt Utility

```javascript
export async function handlePrompt(page, value) {

    page.once('dialog', async dialog => {
        await dialog.accept(value);
    });
}
```

Usage:

```javascript
await handlePrompt(page, 'Selva');

await page.getByRole('button', { name: 'Enter Name' }).click();
```

---

# 43. Reusable Dialog Utility with Expected Message

```javascript
import { expect } from '@playwright/test';

export async function acceptDialogWithMessage(page, expectedMessage) {

    page.once('dialog', async dialog => {

        expect(dialog.message()).toBe(expectedMessage);

        await dialog.accept();
    });
}
```

Usage:

```javascript
await acceptDialogWithMessage(
    page,
    'Are you sure you want to delete?'
);

await page.getByRole('button', { name: 'Delete' }).click();
```

---

# 44. Dialogs and Parallel Execution

Dialog handlers are attached to a specific `Page` instance.

Example:

```javascript
test('Dialog test', async ({ page }) => {

    page.once('dialog', async dialog => {
        await dialog.accept();
    });

    await page.getByRole('button', { name: 'Show Alert' }).click();
});
```

Each Playwright test receives its own page context, so dialog handling remains isolated between tests.

---

# 45. Dialogs and Screenshots

If needed, you can take a screenshot after handling the dialog.

```javascript
page.once('dialog', async dialog => {
    console.log(dialog.message());
    await dialog.accept();
});

await page.getByRole('button', { name: 'Delete' }).click();

await page.screenshot({
    path: 'screenshots/after-dialog.png'
});
```

---

# 46. Dialogs and Tracing

Dialog handling can also be captured as part of Playwright tracing.

```javascript
await context.tracing.start({
    screenshots: true,
    snapshots: true
});

page.once('dialog', async dialog => {
    await dialog.accept();
});

await page.getByRole('button', { name: 'Delete' }).click();

await context.tracing.stop({
    path: 'trace.zip'
});
```

---

# 47. Best Practices

### 1. Register the handler before triggering the dialog

```javascript
page.once('dialog', async dialog => {
    await dialog.accept();
});

await button.click();
```

### 2. Use `once()` when only one dialog is expected

```javascript
page.once('dialog', async dialog => {
    await dialog.accept();
});
```

### 3. Validate dialog messages when required

```javascript
expect(dialog.message()).toBe('Expected message');
```

### 4. Use `dismiss()` for Cancel scenarios

```javascript
await dialog.dismiss();
```

### 5. Provide text for prompts

```javascript
await dialog.accept('Selva');
```

### 6. Avoid unnecessary global dialog handlers

Keep dialog handling close to the action that triggers the dialog unless the behavior is intentionally global.

### 7. Do not treat native dialogs as DOM elements

Use the Playwright `dialog` API.

---

# 48. Interview Questions

## Q1. How does Playwright handle browser dialogs?

Playwright handles browser dialogs using the `dialog` event.

```javascript
page.on('dialog', async dialog => {
    await dialog.accept();
});
```

---

## Q2. What types of browser dialogs does Playwright support?

Common types are:

* `alert`
* `confirm`
* `prompt`
* `beforeunload`

---

## Q3. How do you accept an alert?

```javascript
page.once('dialog', async dialog => {
    await dialog.accept();
});
```

---

## Q4. How do you dismiss a confirmation dialog?

```javascript
page.once('dialog', async dialog => {
    await dialog.dismiss();
});
```

---

## Q5. How do you enter text into a prompt?

```javascript
page.once('dialog', async dialog => {
    await dialog.accept('Selva');
});
```

---

## Q6. How do you get the dialog message?

```javascript
dialog.message()
```

---

## Q7. How do you get the dialog type?

```javascript
dialog.type()
```

---

## Q8. What is the difference between `page.on()` and `page.once()`?

`page.on()` handles every matching event.

```javascript
page.on('dialog', async dialog => {
    await dialog.accept();
});
```

`page.once()` handles only the next matching event.

```javascript
page.once('dialog', async dialog => {
    await dialog.accept();
});
```

---

## Q9. What happens if you do not handle a dialog?

Playwright automatically dismisses dialogs when there is no dialog listener. However, if the dialog is important to the test, explicit handling is preferred.

---

## Q10. Can Playwright interact with native browser dialogs using locators?

No.

Native browser dialogs are not DOM elements.

Use:

```javascript
page.on('dialog', ...)
```

instead.

---

## Q11. How do you validate an alert message?

```javascript
page.once('dialog', async dialog => {

    expect(dialog.message()).toBe('Expected message');

    await dialog.accept();
});
```

---

## Q12. How do you handle multiple dialogs?

Use `page.on()`:

```javascript
page.on('dialog', async dialog => {
    await dialog.accept();
});
```

---

## Q13. How do you handle different dialog types?

```javascript
page.on('dialog', async dialog => {

    if (dialog.type() === 'alert') {
        await dialog.accept();
    }

    if (dialog.type() === 'confirm') {
        await dialog.dismiss();
    }

    if (dialog.type() === 'prompt') {
        await dialog.accept('Selva');
    }
});
```

---

## Q14. How do you wait for a dialog explicitly?

```javascript
const dialogPromise = page.waitForEvent('dialog');

await page.getByRole('button', { name: 'Delete' }).click();

const dialog = await dialogPromise;

await dialog.accept();
```

---

## Q15. Why should the dialog handler be registered before clicking the button?

Because the button click may immediately trigger a native dialog. Registering the handler first ensures Playwright can handle the dialog as soon as it appears.

---

# 49. Real-World Automation Example

Suppose an application displays:

```text
Are you sure you want to delete this vehicle?
```

The test should verify the message and click OK.

```javascript
import { test, expect } from '@playwright/test';

test('Delete vehicle confirmation', async ({ page }) => {

    await page.goto('https://example.com');

    page.once('dialog', async dialog => {

        expect(dialog.type()).toBe('confirm');

        expect(dialog.message())
            .toBe('Are you sure you want to delete this vehicle?');

        await dialog.accept();
    });

    await page.getByRole('button', {
        name: 'Delete Vehicle'
    }).click();
});
```

---

# 50. Real-World Cancel Example

```javascript
import { test } from '@playwright/test';

test('Cancel vehicle deletion', async ({ page }) => {

    await page.goto('https://example.com');

    page.once('dialog', async dialog => {

        console.log(dialog.message());

        await dialog.dismiss();
    });

    await page.getByRole('button', {
        name: 'Delete Vehicle'
    }).click();
});
```

---

# 51. Real-World Prompt Example

```javascript
import { test, expect } from '@playwright/test';

test('Enter vehicle nickname', async ({ page }) => {

    await page.goto('https://example.com');

    page.once('dialog', async dialog => {

        expect(dialog.type()).toBe('prompt');

        await dialog.accept('My Toyota');
    });

    await page.getByRole('button', {
        name: 'Enter Vehicle Name'
    }).click();
});
```

---

# 52. Recommended Pattern

For most one-time dialog scenarios, use:

```javascript
page.once('dialog', async dialog => {

    console.log(`Type: ${dialog.type()}`);
    console.log(`Message: ${dialog.message()}`);

    await dialog.accept();
});

await page.getByRole('button', {
    name: 'Trigger Dialog'
}).click();
```

For prompt:

```javascript
page.once('dialog', async dialog => {
    await dialog.accept('Input Value');
});

await page.getByRole('button', {
    name: 'Trigger Prompt'
}).click();
```

For Cancel:

```javascript
page.once('dialog', async dialog => {
    await dialog.dismiss();
});

await page.getByRole('button', {
    name: 'Trigger Dialog'
}).click();
```

---

# 53. Quick Reference

| Requirement              | Playwright                    |
| ------------------------ | ----------------------------- |
| Handle dialog            | `page.on('dialog', ...)`      |
| Handle next dialog       | `page.once('dialog', ...)`    |
| Accept                   | `dialog.accept()`             |
| Accept prompt with value | `dialog.accept('value')`      |
| Cancel/Dismiss           | `dialog.dismiss()`            |
| Get message              | `dialog.message()`            |
| Get type                 | `dialog.type()`               |
| Get prompt default       | `dialog.defaultValue()`       |
| Wait for dialog          | `page.waitForEvent('dialog')` |

---

# 54. Final Example

```javascript
import { test, expect } from '@playwright/test';

test('Complete dialog handling example', async ({ page }) => {

    await page.goto('https://example.com');

    const dialogPromise = page.waitForEvent('dialog');

    await page.getByRole('button', {
        name: 'Delete'
    }).click();

    const dialog = await dialogPromise;

    console.log(`Dialog type: ${dialog.type()}`);
    console.log(`Dialog message: ${dialog.message()}`);

    expect(dialog.type()).toBe('confirm');
    expect(dialog.message()).toContain('delete');

    await dialog.accept();
});
```

---

# 55. Summary

Playwright provides a simple API for handling native browser dialogs.

The most important APIs are:

```javascript
page.on('dialog', ...)
```

```javascript
page.once('dialog', ...)
```

```javascript
dialog.accept()
```

```javascript
dialog.dismiss()
```

```javascript
dialog.message()
```

```javascript
dialog.type()
```

```javascript
dialog.defaultValue()
```

```javascript
page.waitForEvent('dialog')
```

For senior-level Playwright automation, remember these key points:

1. Register the dialog handler before triggering the dialog.
2. Use `once()` when only one dialog is expected.
3. Use `on()` when multiple dialogs may occur.
4. Use `accept()` for OK.
5. Use `dismiss()` for Cancel.
6. Pass a value to `accept()` for prompts.
7. Validate dialog type and message when they are part of the requirement.
8. Native browser dialogs are handled through Playwright's `dialog` API, not locators.
9. Keep dialog handling close to the action that triggers it unless a global handler is intentional.
10. `waitForEvent('dialog')` is useful when you want to explicitly synchronize with the dialog event.

---

## File Path

```text
Dialogs/Playwright-Dialogs.md
```
