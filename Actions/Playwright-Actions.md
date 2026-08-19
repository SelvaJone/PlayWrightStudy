# Playwright Actions

## 1. Introduction

Playwright provides built-in methods to interact with web elements.

Common actions include:

* Click
* Double-click
* Right-click
* Fill text
* Type text
* Press keyboard keys
* Check and uncheck checkboxes
* Select radio buttons
* Select dropdown options
* Hover
* Focus
* Clear input fields
* Upload files
* Drag and drop
* Scroll
* Tap on mobile devices

Playwright automatically performs actionability checks before most actions.

---

# 2. Basic Click

Use `click()` to click an element.

```javascript
await page.getByRole('button', { name: 'Login' }).click();
```

Using a locator:

```javascript
const loginButton = page.getByRole('button', {
    name: 'Login'
});

await loginButton.click();
```

---

# 3. Click Using CSS

```javascript
await page.locator('#login').click();
```

```javascript
await page.locator('.login-button').click();
```

---

# 4. Click Using Text

```javascript
await page.getByText('Login').click();
```

For exact text:

```javascript
await page.getByText('Login', { exact: true }).click();
```

---

# 5. Double Click

Use `dblclick()`.

```javascript
await page.getByText('Open').dblclick();
```

Example:

```javascript
await page.locator('#file').dblclick();
```

---

# 6. Right Click

Use the `button` option.

```javascript
await page.getByText('File').click({
    button: 'right'
});
```

Or:

```javascript
await page.locator('#file').click({
    button: 'right'
});
```

This is useful for opening context menus.

---

# 7. Middle Mouse Click

```javascript
await page.locator('#link').click({
    button: 'middle'
});
```

Supported mouse buttons include:

```text
left
right
middle
```

---

# 8. Force Click

Normally Playwright checks whether the element is actionable before clicking.

You can bypass some actionability checks using `force`.

```javascript
await page.locator('#login').click({
    force: true
});
```

### Important

Do not use `force: true` as the default solution for failing tests.

First determine why the element is not actionable.

---

# 9. Click with Modifiers

You can hold keyboard modifiers while clicking.

### Control + Click

```javascript
await page.getByText('Item').click({
    modifiers: ['Control']
});
```

### Shift + Click

```javascript
await page.getByText('Item').click({
    modifiers: ['Shift']
});
```

### Alt + Click

```javascript
await page.getByText('Item').click({
    modifiers: ['Alt']
});
```

### Meta + Click

```javascript
await page.getByText('Item').click({
    modifiers: ['Meta']
});
```

Multiple modifiers can be used:

```javascript
await page.getByText('Item').click({
    modifiers: ['Control', 'Shift']
});
```

---

# 10. Click Position

You can click at a specific position inside an element.

```javascript
await page.locator('#canvas').click({
    position: {
        x: 100,
        y: 50
    }
});
```

This can be useful for:

* Canvas applications
* Maps
* Custom UI controls
* Graphical interfaces

---

# 11. Fill Text

`fill()` is the preferred way to enter text into an input.

```javascript
await page.getByLabel('Username').fill('admin');
```

Password:

```javascript
await page.getByLabel('Password').fill('Password123');
```

---

# 12. Fill Replaces Existing Text

Suppose the input already contains:

```text
oldValue
```

Then:

```javascript
await page.locator('#username').fill('newValue');
```

will replace the existing value.

This is one reason `fill()` is commonly preferred over manually typing characters.

---

# 13. Clear an Input

You can clear an input using `fill('')`.

```javascript
await page.getByLabel('Username').fill('');
```

This is generally simpler than selecting and deleting the existing text.

---

# 14. Press a Keyboard Key

Use `press()`.

```javascript
await page.getByLabel('Username').press('Enter');
```

Escape:

```javascript
await page.locator('#search').press('Escape');
```

Tab:

```javascript
await page.locator('#username').press('Tab');
```

---

# 15. Common Keyboard Keys

Examples:

```javascript
await locator.press('Enter');

await locator.press('Tab');

await locator.press('Escape');

await locator.press('Backspace');

await locator.press('Delete');

await locator.press('ArrowUp');

await locator.press('ArrowDown');

await locator.press('ArrowLeft');

await locator.press('ArrowRight');

await locator.press('Home');

await locator.press('End');

await locator.press('PageUp');

await locator.press('PageDown');
```

---

# 16. Keyboard Shortcuts

Playwright supports keyboard combinations.

### Control + A

```javascript
await page.locator('#username').press('Control+A');
```

### Control + C

```javascript
await page.locator('#username').press('Control+C');
```

### Control + V

```javascript
await page.locator('#username').press('Control+V');
```

### Control + X

```javascript
await page.locator('#username').press('Control+X');
```

On macOS, `Meta` may be used instead of `Control`.

---

# 17. Keyboard API

Playwright also provides a keyboard API.

```javascript
await page.keyboard.press('Enter');
```

Type:

```javascript
await page.keyboard.type('Hello');
```

Insert text:

```javascript
await page.keyboard.insertText('Hello World');
```

---

# 18. Keyboard Down and Up

You can press and release a key manually.

```javascript
await page.keyboard.down('Shift');

await page.keyboard.press('ArrowDown');

await page.keyboard.up('Shift');
```

This is useful for advanced keyboard interactions.

---

# 19. Type Text

Playwright provides `pressSequentially()` when you specifically want to simulate typing characters sequentially.

```javascript
await page.getByLabel('Username')
    .pressSequentially('admin');
```

You can specify a delay:

```javascript
await page.getByLabel('Username')
    .pressSequentially('admin', {
        delay: 100
    });
```

For normal form input, prefer:

```javascript
await page.getByLabel('Username').fill('admin');
```

Use sequential typing when the application specifically depends on keyboard events.

---

# 20. Checkbox

Use `check()`.

```javascript
await page.getByLabel('Remember me').check();
```

Example:

```javascript
const checkbox = page.getByLabel('Remember me');

await checkbox.check();
```

---

# 21. Uncheck Checkbox

Use `uncheck()`.

```javascript
await page.getByLabel('Remember me').uncheck();
```

---

# 22. Check Checkbox State

```javascript
await expect(
    page.getByLabel('Remember me')
).toBeChecked();
```

---

# 23. Check If Checkbox Is Not Selected

```javascript
await expect(
    page.getByLabel('Remember me')
).not.toBeChecked();
```

---

# 24. Radio Button

Use `check()` for radio buttons.

```javascript
await page.getByLabel('Male').check();
```

Another example:

```javascript
await page.getByLabel('Premium').check();
```

---

# 25. Radio Button Validation

```javascript
await expect(
    page.getByLabel('Premium')
).toBeChecked();
```

---

# 26. Select Dropdown Option

For a native `<select>` element, use `selectOption()`.

HTML:

```html
<select id="country">
    <option value="us">United States</option>
    <option value="ca">Canada</option>
    <option value="mx">Mexico</option>
</select>
```

Select by value:

```javascript
await page.locator('#country').selectOption('ca');
```

---

# 27. Select Option by Label

```javascript
await page.locator('#country').selectOption({
    label: 'Canada'
});
```

---

# 28. Select Option by Value

```javascript
await page.locator('#country').selectOption({
    value: 'ca'
});
```

---

# 29. Select Option by Index

```javascript
await page.locator('#country').selectOption({
    index: 1
});
```

Remember that indexes start at `0`.

---

# 30. Multiple Select Options

For a multi-select dropdown:

```javascript
await page.locator('#countries').selectOption([
    'us',
    'ca'
]);
```

---

# 31. Validate Selected Option

```javascript
await expect(
    page.locator('#country')
).toHaveValue('ca');
```

---

# 32. Hover

Use `hover()`.

```javascript
await page.getByText('Products').hover();
```

Example:

```javascript
await page.locator('.menu').hover();
```

Hover is commonly used for:

* Menus
* Tooltips
* Dropdown menus
* Hidden navigation options

---

# 33. Hover Example

```javascript
await page.getByRole('button', {
    name: 'Products'
}).hover();

await page.getByRole('link', {
    name: 'Laptops'
}).click();
```

---

# 34. Focus

Use `focus()`.

```javascript
await page.getByLabel('Username').focus();
```

This is useful when testing:

* Focus behavior
* Keyboard interactions
* Accessibility
* Validation triggered by focus

---

# 35. Blur

Playwright does not require a dedicated `blur()` locator action for most scenarios.

You can move focus to another element:

```javascript
await page.getByLabel('Username').focus();

await page.getByLabel('Password').focus();
```

Or use JavaScript when testing a specific application behavior.

---

# 36. Drag and Drop

Playwright supports `dragTo()`.

```javascript
const source = page.locator('#source');
const target = page.locator('#target');

await source.dragTo(target);
```

This is the simplest approach for standard drag-and-drop interfaces.

---

# 37. Drag and Drop Example

```javascript
await page.locator('.drag-item')
    .dragTo(page.locator('.drop-zone'));
```

---

# 38. Mouse API

Playwright provides a low-level mouse API.

Move the mouse:

```javascript
await page.mouse.move(100, 200);
```

Mouse down:

```javascript
await page.mouse.down();
```

Mouse up:

```javascript
await page.mouse.up();
```

Click:

```javascript
await page.mouse.click(100, 200);
```

Double-click:

```javascript
await page.mouse.dblclick(100, 200);
```

---

# 39. Mouse Drag

For advanced mouse interactions:

```javascript
await page.mouse.move(100, 100);

await page.mouse.down();

await page.mouse.move(300, 300);

await page.mouse.up();
```

This can be useful for:

* Canvas
* Drawing applications
* Sliders
* Custom drag operations

---

# 40. Scrolling

Playwright normally scrolls an element into view automatically when performing actions.

Example:

```javascript
await page.getByRole('button', {
    name: 'Submit'
}).click();
```

Playwright can automatically scroll the button into view if necessary.

---

# 41. Scroll an Element into View

You can explicitly call:

```javascript
await page.locator('#footer')
    .scrollIntoViewIfNeeded();
```

---

# 42. Mouse Wheel

For manual scrolling:

```javascript
await page.mouse.wheel(0, 500);
```

Scroll down:

```javascript
await page.mouse.wheel(0, 1000);
```

Scroll up:

```javascript
await page.mouse.wheel(0, -500);
```

---

# 43. File Upload

Use `setInputFiles()`.

HTML:

```html
<input type="file" id="upload">
```

Playwright:

```javascript
await page.locator('#upload')
    .setInputFiles('files/test.pdf');
```

---

# 44. Upload Multiple Files

```javascript
await page.locator('#upload').setInputFiles([
    'files/file1.pdf',
    'files/file2.pdf'
]);
```

---

# 45. Remove Selected File

```javascript
await page.locator('#upload').setInputFiles([]);
```

---

# 46. FileChooser

If clicking a button opens the operating-system file chooser:

```javascript
const fileChooserPromise = page.waitForEvent('filechooser');

await page.getByRole('button', {
    name: 'Upload'
}).click();

const fileChooser = await fileChooserPromise;

await fileChooser.setFiles('files/test.pdf');
```

---

# 47. Touch Actions

For mobile device emulation, Playwright supports touch interactions when configured for touch.

Example:

```javascript
await page.getByText('Submit').tap();
```

---

# 48. Tap

Use `tap()` for touch interaction.

```javascript
await page.getByRole('button', {
    name: 'Submit'
}).tap();
```

This is particularly useful when testing mobile-oriented interfaces.

---

# 49. Select Text

You can select text in an input or textarea.

```javascript
await page.locator('#username').selectText();
```

This selects the entire text.

---

# 50. Input Value

To retrieve an input value:

```javascript
const value = await page.locator('#username').inputValue();

console.log(value);
```

---

# 51. Text Content

Retrieve text:

```javascript
const text = await page.locator('.message').textContent();

console.log(text);
```

For visible text, `innerText()` can also be used:

```javascript
const text = await page.locator('.message').innerText();

console.log(text);
```

---

# 52. Inner HTML

```javascript
const html = await page.locator('.container').innerHTML();

console.log(html);
```

Use this when inspecting the DOM structure during a test.

---

# 53. Complete Actions Example

```javascript
const { test, expect } = require('@playwright/test');

test('Playwright Actions Demo', async ({ page }) => {

    await page.goto('https://example.com');

    // Fill
    await page.getByLabel('Username').fill('admin');

    // Keyboard
    await page.getByLabel('Username').press('Tab');

    // Fill password
    await page.getByLabel('Password').fill('Password123');

    // Checkbox
    await page.getByLabel('Remember me').check();

    // Validate checkbox
    await expect(
        page.getByLabel('Remember me')
    ).toBeChecked();

    // Click
    await page.getByRole('button', {
        name: 'Login'
    }).click();

    // Validate result
    await expect(
        page.getByRole('heading', {
            name: 'Dashboard'
        })
    ).toBeVisible();
});
```

---

# 54. Actions vs Selenium

## Selenium

```java
driver.findElement(By.id("username"))
      .sendKeys("admin");

driver.findElement(By.id("login"))
      .click();
```

## Playwright

```javascript
await page.locator('#username').fill('admin');

await page.locator('#login').click();
```

---

# 55. Selenium Checkbox vs Playwright

### Selenium

```java
driver.findElement(By.id("remember")).click();
```

### Playwright

```javascript
await page.getByLabel('Remember me').check();
```

Playwright provides an action specifically designed for checkbox behavior.

---

# 56. Selenium Dropdown vs Playwright

### Selenium

```java
Select dropdown =
    new Select(driver.findElement(By.id("country")));

dropdown.selectByValue("ca");
```

### Playwright

```javascript
await page.locator('#country').selectOption('ca');
```

---

# 57. Selenium Mouse Actions vs Playwright

### Selenium

```java
Actions actions = new Actions(driver);

actions.moveToElement(element)
       .click()
       .perform();
```

### Playwright

```javascript
await element.hover();

await element.click();
```

Or:

```javascript
await element.hover();
await element.click();
```

Playwright generally requires less explicit synchronization and action setup.

---

# 58. Playwright Actionability

Before performing actions, Playwright checks whether the element is actionable.

Typical checks include:

```text
Element exists
      ↓
Element is visible
      ↓
Element is stable
      ↓
Element receives events
      ↓
Element is enabled
      ↓
Action is performed
```

This is one of the major differences between Playwright and traditional Selenium automation.

---

# 59. Avoid Hard Waits

Avoid:

```javascript
await page.waitForTimeout(5000);

await page.getByRole('button', {
    name: 'Submit'
}).click();
```

Prefer:

```javascript
await page.getByRole('button', {
    name: 'Submit'
}).click();
```

Or use an explicit assertion when appropriate:

```javascript
await expect(
    page.getByRole('button', {
        name: 'Submit'
    })
).toBeVisible();

await page.getByRole('button', {
    name: 'Submit'
}).click();
```

---

# 60. Best Practices

## Use User-Facing Locators

Prefer:

```javascript
await page.getByRole('button', {
    name: 'Login'
}).click();
```

over fragile selectors.

## Use `fill()` for Normal Input

```javascript
await page.getByLabel('Username').fill('admin');
```

## Use `press()` for Keyboard Actions

```javascript
await page.getByLabel('Username').press('Enter');
```

## Use `check()` and `uncheck()` for Checkboxes

```javascript
await checkbox.check();
await checkbox.uncheck();
```

## Use `selectOption()` for Native Selects

```javascript
await page.locator('#country').selectOption('ca');
```

## Use `dragTo()` for Standard Drag-and-Drop

```javascript
await source.dragTo(target);
```

## Avoid Unnecessary `force`

```javascript
await locator.click({ force: true });
```

should not be the first solution to an action failure.

## Avoid Hard Waits

```javascript
await page.waitForTimeout(5000);
```

should generally be avoided in normal automation.

---

# 61. Important Playwright Actions Cheat Sheet

```javascript
// Click
await locator.click();

// Double click
await locator.dblclick();

// Fill
await locator.fill('text');

// Type sequentially
await locator.pressSequentially('text');

// Keyboard
await locator.press('Enter');

// Check
await locator.check();

// Uncheck
await locator.uncheck();

// Select dropdown
await locator.selectOption('value');

// Hover
await locator.hover();

// Focus
await locator.focus();

// Drag and drop
await source.dragTo(target);

// Upload
await locator.setInputFiles('file.pdf');

// Select text
await locator.selectText();

// Tap
await locator.tap();

// Scroll
await locator.scrollIntoViewIfNeeded();
```

---

# 62. Interview Questions

### Q1. What is the difference between `fill()` and `pressSequentially()`?

`fill()` sets the input value efficiently and is preferred for normal form entry.

`pressSequentially()` simulates typing characters one by one and can be useful when an application depends on keyboard events.

---

### Q2. How do you select a checkbox?

```javascript
await page.getByLabel('Remember me').check();
```

---

### Q3. How do you uncheck a checkbox?

```javascript
await page.getByLabel('Remember me').uncheck();
```

---

### Q4. How do you select a dropdown value?

```javascript
await page.locator('#country').selectOption('ca');
```

---

### Q5. How do you perform a right-click?

```javascript
await locator.click({
    button: 'right'
});
```

---

### Q6. How do you double-click?

```javascript
await locator.dblclick();
```

---

### Q7. How do you hover over an element?

```javascript
await locator.hover();
```

---

### Q8. How do you perform drag and drop?

```javascript
await source.dragTo(target);
```

---

### Q9. How do you upload a file?

```javascript
await locator.setInputFiles('files/test.pdf');
```

---

### Q10. Does Playwright automatically wait before actions?

Yes. Playwright performs actionability checks and waits for the element to become actionable before performing most actions.

---

# 63. Key Takeaways

The most important Playwright actions are:

```text
click()
dblclick()
fill()
press()
pressSequentially()
check()
uncheck()
selectOption()
hover()
focus()
dragTo()
setInputFiles()
tap()
scrollIntoViewIfNeeded()
```

For most automation tests, the pattern is:

```text
Locate
   ↓
Interact
   ↓
Auto-wait
   ↓
Validate
```

Example:

```javascript
const loginButton = page.getByRole('button', {
    name: 'Login'
});

await expect(loginButton).toBeVisible();

await loginButton.click();
```

**Next topic:** `Waits/Playwright-Waits.md`
