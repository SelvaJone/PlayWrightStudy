# Playwright Locators

## 1. What is a Locator?

A **Locator** in Playwright is a way to identify and interact with elements on a web page.

Locators are one of the most important concepts in Playwright because they provide:

* Reliable element identification
* Auto-waiting
* Retry-ability
* Better readability
* Better handling of dynamic pages

### Basic Example

```javascript
const { test, expect } = require('@playwright/test');

test('Locator example', async ({ page }) => {
    await page.goto('https://example.com');

    const heading = page.locator('h1');

    await expect(heading).toBeVisible();
});
```

---

# 2. Common Locator Types

Playwright provides several recommended locator methods.

| Locator     | Example              |
| ----------- | -------------------- |
| Role        | `getByRole()`        |
| Text        | `getByText()`        |
| Label       | `getByLabel()`       |
| Placeholder | `getByPlaceholder()` |
| Alt Text    | `getByAltText()`     |
| Title       | `getByTitle()`       |
| Test ID     | `getByTestId()`      |
| CSS         | `locator()`          |
| XPath       | `locator()`          |

---

# 3. getByRole()

`getByRole()` is one of the most recommended ways to locate elements.

```javascript
await page.getByRole('button', { name: 'Login' }).click();
```

Example:

```html
<button>Login</button>
```

Playwright:

```javascript
await page.getByRole('button', { name: 'Login' }).click();
```

### Common Roles

```text
button
link
textbox
checkbox
radio
heading
combobox
listbox
option
tab
row
cell
```

Example:

```javascript
await page.getByRole('textbox', { name: 'Username' }).fill('admin');

await page.getByRole('textbox', { name: 'Password' }).fill('password');

await page.getByRole('button', { name: 'Login' }).click();
```

---

# 4. getByText()

Use `getByText()` to locate an element based on visible text.

```javascript
await page.getByText('Welcome').click();
```

Example:

```html
<div>Welcome</div>
```

```javascript
await page.getByText('Welcome').click();
```

### Exact Text

```javascript
await page.getByText('Welcome', { exact: true }).click();
```

Without `exact: true`, Playwright can match text containing the specified value.

---

# 5. getByLabel()

`getByLabel()` is useful for form fields associated with a label.

HTML:

```html
<label for="username">Username</label>
<input id="username">
```

Playwright:

```javascript
await page.getByLabel('Username').fill('admin');
```

Another example:

```javascript
await page.getByLabel('Password').fill('secret123');
```

This is usually better than relying on complex CSS selectors.

---

# 6. getByPlaceholder()

If an input contains a placeholder:

```html
<input placeholder="Enter username">
```

Use:

```javascript
await page.getByPlaceholder('Enter username').fill('admin');
```

Another example:

```javascript
await page.getByPlaceholder('Enter password').fill('password123');
```

---

# 7. getByAltText()

Useful for images that contain meaningful `alt` text.

HTML:

```html
<img src="logo.png" alt="Company Logo">
```

Playwright:

```javascript
await page.getByAltText('Company Logo').click();
```

---

# 8. getByTitle()

If an element has a `title` attribute:

```html
<button title="Settings">⚙</button>
```

Use:

```javascript
await page.getByTitle('Settings').click();
```

---

# 9. getByTestId()

A test ID is commonly used specifically for automation.

HTML:

```html
<button data-testid="login-button">
    Login
</button>
```

Playwright:

```javascript
await page.getByTestId('login-button').click();
```

### Recommended Practice

For applications where developers can add automation-friendly attributes:

```html
<button data-testid="login-button">
    Login
</button>
```

```javascript
await page.getByTestId('login-button').click();
```

Test IDs are generally more stable than CSS classes that may change because of UI styling.

---

# 10. CSS Locators

Playwright supports CSS selectors through `locator()`.

```javascript
await page.locator('#username').fill('admin');
```

Class:

```javascript
await page.locator('.login-button').click();
```

Attribute:

```javascript
await page.locator('[name="username"]').fill('admin');
```

Multiple attributes:

```javascript
await page.locator('input[name="username"]').fill('admin');
```

---

# 11. XPath Locators

Playwright also supports XPath.

```javascript
await page.locator('//input[@id="username"]').fill('admin');
```

Another example:

```javascript
await page.locator('//button[text()="Login"]').click();
```

### Recommendation

Prefer Playwright's user-facing locators:

```javascript
getByRole()
getByLabel()
getByText()
getByPlaceholder()
getByTestId()
```

Use XPath when there is a specific reason to do so.

---

# 12. Locator Chaining

Locators can be chained.

HTML:

```html
<div class="login">
    <input name="username">
    <button>Login</button>
</div>
```

Playwright:

```javascript
const login = page.locator('.login');

await login.locator('input[name="username"]').fill('admin');

await login.getByRole('button', { name: 'Login' }).click();
```

This is useful when the same element type appears multiple times on a page.

---

# 13. Filtering Locators

Playwright allows filtering locators.

Example:

```javascript
const product = page.getByRole('listitem')
    .filter({ hasText: 'Laptop' });

await product.getByRole('button', { name: 'Add to cart' }).click();
```

This is much more readable than creating a complicated XPath.

---

# 14. filter({ hasText })

Example:

```javascript
const product = page.locator('.product')
    .filter({ hasText: 'Laptop' });

await product.click();
```

---

# 15. filter({ has })

You can filter based on another locator.

```javascript
const product = page.locator('.product')
    .filter({
        has: page.getByRole('button', { name: 'Add to cart' })
    });
```

---

# 16. Locating Parent Elements

Example HTML:

```html
<div class="product">
    <span>Laptop</span>
    <button>Add to cart</button>
</div>
```

Playwright:

```javascript
const product = page.locator('.product')
    .filter({ hasText: 'Laptop' });

await product.getByRole('button', { name: 'Add to cart' }).click();
```

This approach is generally preferred over complex XPath parent traversal.

---

# 17. nth()

If multiple elements match a locator, `nth()` can select a specific element.

```javascript
await page.locator('.product').nth(0).click();
```

Second element:

```javascript
await page.locator('.product').nth(1).click();
```

### Important

Indexes start from `0`.

```text
nth(0) → First
nth(1) → Second
nth(2) → Third
```

Avoid `nth()` when a more specific locator is available.

---

# 18. first()

Select the first matching element.

```javascript
await page.locator('.product').first().click();
```

---

# 19. last()

Select the last matching element.

```javascript
await page.locator('.product').last().click();
```

---

# 20. Checking Locator Count

Use `count()` to determine how many elements match.

```javascript
const count = await page.locator('.product').count();

console.log(count);
```

Example:

```javascript
const buttons = page.getByRole('button');

console.log(await buttons.count());
```

---

# 21. Checking Visibility

```javascript
const button = page.getByRole('button', { name: 'Login' });

await expect(button).toBeVisible();
```

---

# 22. Checking Enabled State

```javascript
await expect(
    page.getByRole('button', { name: 'Login' })
).toBeEnabled();
```

---

# 23. Checking Disabled State

```javascript
await expect(
    page.getByRole('button', { name: 'Submit' })
).toBeDisabled();
```

---

# 24. Checking Text

```javascript
await expect(
    page.getByRole('heading')
).toHaveText('Welcome');
```

For partial text:

```javascript
await expect(
    page.getByRole('heading')
).toContainText('Welcome');
```

---

# 25. Checking Attribute

```javascript
await expect(
    page.locator('#username')
).toHaveAttribute('type', 'text');
```

---

# 26. Locator vs ElementHandle

Playwright strongly encourages using **Locator** instead of `ElementHandle`.

### Locator

```javascript
const button = page.getByRole('button', { name: 'Login' });

await button.click();
```

### ElementHandle

```javascript
const button = await page.$('button');

await button.click();
```

Locators are preferred because they provide Playwright's auto-waiting and retry behavior.

---

# 27. Locator Auto-Waiting

Consider:

```javascript
await page.getByRole('button', { name: 'Login' }).click();
```

Playwright automatically waits for the element to become actionable.

It can wait for conditions such as:

* Element exists
* Element is visible
* Element is enabled
* Element is stable
* Element can receive events

Therefore, unnecessary hard waits should normally be avoided.

### Avoid

```javascript
await page.waitForTimeout(5000);

await page.getByRole('button', { name: 'Login' }).click();
```

### Prefer

```javascript
await page.getByRole('button', { name: 'Login' }).click();
```

---

# 28. Locator Reuse

Create a locator once and reuse it.

```javascript
const username = page.getByLabel('Username');
const password = page.getByLabel('Password');
const loginButton = page.getByRole('button', { name: 'Login' });

await username.fill('admin');
await password.fill('password123');
await loginButton.click();
```

This improves readability.

---

# 29. Locator Assertions

Use Playwright's `expect()` for validation.

```javascript
await expect(
    page.getByRole('heading', { name: 'Dashboard' })
).toBeVisible();
```

More examples:

```javascript
await expect(locator).toBeVisible();

await expect(locator).toBeHidden();

await expect(locator).toBeEnabled();

await expect(locator).toBeDisabled();

await expect(locator).toHaveText('Hello');

await expect(locator).toContainText('Hello');

await expect(locator).toHaveValue('admin');

await expect(locator).toHaveAttribute('type', 'text');
```

---

# 30. Example Login Test

```javascript
const { test, expect } = require('@playwright/test');

test('Login Test', async ({ page }) => {

    await page.goto('https://example.com/login');

    const username = page.getByLabel('Username');
    const password = page.getByLabel('Password');
    const loginButton = page.getByRole('button', {
        name: 'Login'
    });

    await username.fill('admin');
    await password.fill('password123');

    await loginButton.click();

    await expect(
        page.getByRole('heading', { name: 'Dashboard' })
    ).toBeVisible();
});
```

---

# 31. Locator Best Practices

## Prefer

```javascript
page.getByRole('button', { name: 'Login' });
```

over:

```javascript
page.locator('#root > div:nth-child(2) > button');
```

Prefer:

```javascript
page.getByLabel('Username');
```

over:

```javascript
page.locator('input.form-control:nth-child(1)');
```

Prefer:

```javascript
page.getByTestId('login-button');
```

over:

```javascript
page.locator('.btn.btn-primary.login');
```

---

# 32. Recommended Locator Priority

A practical priority is:

```text
1. getByRole()
2. getByLabel()
3. getByPlaceholder()
4. getByText()
5. getByTestId()
6. getByAltText()
7. getByTitle()
8. CSS locator
9. XPath
```

The exact choice depends on the application's HTML and accessibility attributes.

---

# 33. Selenium vs Playwright Locators

| Selenium               | Playwright                         |
| ---------------------- | ---------------------------------- |
| `By.id()`              | `getByTestId()` / `locator('#id')` |
| `By.name()`            | `locator('[name="..."]')`          |
| `By.className()`       | `locator('.class')`                |
| `By.cssSelector()`     | `locator('css')`                   |
| `By.xpath()`           | `locator('xpath')`                 |
| `By.linkText()`        | `getByRole('link', {name: '...'})` |
| `By.partialLinkText()` | `getByRole()` / `getByText()`      |

### Selenium

```java
driver.findElement(By.id("username")).sendKeys("admin");
```

### Playwright

```javascript
await page.locator('#username').fill('admin');
```

Or preferably:

```javascript
await page.getByLabel('Username').fill('admin');
```

---

# 34. Important Interview Questions

### Q1. What is a Locator in Playwright?

A Locator identifies an element and provides Playwright's auto-waiting and retry behavior for interactions and assertions.

### Q2. Which locator is recommended?

User-facing locators such as:

```javascript
getByRole()
getByLabel()
getByText()
getByPlaceholder()
```

are generally preferred.

### Q3. Does Playwright support XPath?

Yes.

```javascript
page.locator('//button[@id="login"]');
```

### Q4. What is `getByRole()`?

It locates elements based on their accessible role and optionally their accessible name.

```javascript
page.getByRole('button', { name: 'Login' });
```

### Q5. What is the difference between Locator and ElementHandle?

A Locator is a reusable, lazy representation of an element and supports auto-waiting and retry behavior. An ElementHandle represents a specific DOM element and is generally less preferred for normal test interactions.

### Q6. Does Playwright automatically wait before clicking?

Yes. Playwright performs actionability checks before actions such as `click()`.

### Q7. What is `nth()`?

It selects an element by zero-based index.

```javascript
page.locator('.item').nth(0);
```

### Q8. How do you filter a locator?

```javascript
page.locator('.product')
    .filter({ hasText: 'Laptop' });
```

---

# 35. Key Takeaways

```text
Locator
   ↓
Identify element
   ↓
Interact with element
   ↓
Playwright auto-waits
   ↓
Perform assertion
```

The most important Playwright locator methods to remember are:

```javascript
getByRole()
getByText()
getByLabel()
getByPlaceholder()
getByAltText()
getByTitle()
getByTestId()
locator()
```

**Next topic:** `Actions/Playwright-Actions.md`
