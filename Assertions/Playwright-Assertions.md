# Playwright Assertions

## 1. Introduction

Assertions are used to verify that the application behaves as expected.

In Playwright, assertions are primarily provided by:

```javascript
expect()
import { test, expect } from '@playwright/test';

test('Verify page title', async ({ page }) => {
    await page.goto('https://example.com');

    await expect(page).toHaveTitle('Example Domain');
});
2. Why Use Playwright Assertions?

Assertions help us validate:

Page title
URL
Element visibility
Element text
Input values
Element state
Element attributes
Element count
Page content
API responses
Custom conditions

Example:

await expect(page.locator('#loginButton')).toBeVisible();
3. Import expect

In Playwright Test:

import { test, expect } from '@playwright/test';

Example:

test('Assertion example', async ({ page }) => {
    await page.goto('https://example.com');


    await expect(page).toHaveTitle('Example Domain');
});
4. Auto-Retrying Assertions

One of the major advantages of Playwright is that many assertions automatically retry until the expected condition is met or the timeout is reached.

Example:

await expect(page.locator('#message')).toBeVisible();

Playwright does not immediately fail if the element is not visible.

It waits and retries.

This is especially useful for:

AJAX applications
React applications
Angular applications
Vue applications
Dynamic elements
API-driven UI updates
5. expect(locator).toBeVisible()

Verifies that an element is visible.

await expect(page.locator('#loginButton')).toBeVisible();

Example:

const loginButton = page.getByRole('button', {
    name: 'Login'
});


await expect(loginButton).toBeVisible();
6. toBeHidden()

Verifies that an element is hidden.

await expect(page.locator('#loading')).toBeHidden();

Example:

await page.getByRole('button', {
    name: 'Submit'
}).click();


await expect(page.locator('#loading')).toBeHidden();
7. toBeEnabled()

Verifies that an element is enabled.

await expect(page.locator('#submit')).toBeEnabled();

Example:

const submitButton = page.getByRole('button', {
    name: 'Submit'
});


await expect(submitButton).toBeEnabled();
8. toBeDisabled()

Verifies that an element is disabled.

await expect(page.locator('#submit')).toBeDisabled();
9. toBeEditable()

Verifies that an input or textarea can be edited.

await expect(page.locator('#username')).toBeEditable();

Example:

const username = page.getByLabel('Username');


await expect(username).toBeEditable();
10. toBeChecked()

Verifies that a checkbox or radio button is checked.

await expect(page.locator('#rememberMe')).toBeChecked();

Example:

const checkbox = page.getByLabel('Remember me');


await checkbox.check();


await expect(checkbox).toBeChecked();
11. toBeUnchecked()

Verifies that a checkbox is not checked.

await expect(page.locator('#rememberMe')).not.toBeChecked();

Example:

const checkbox = page.getByLabel('Remember me');


await checkbox.uncheck();


await expect(checkbox).not.toBeChecked();
12. toHaveText()

Verifies exact text.

await expect(page.locator('#message'))
    .toHaveText('Login successful');

Example:

const message = page.locator('#message');


await expect(message).toHaveText('Login successful');
13. toContainText()

Verifies that an element contains the specified text.

await expect(page.locator('#message'))
    .toContainText('successful');

Difference:

toHaveText()

checks the expected text more strictly.

toContainText()

checks whether the expected text exists within the element's text.

Example:

await expect(page.locator('#message'))
    .toContainText('Login');
14. toHaveValue()

Verifies the value of an input field.

await expect(page.locator('#username'))
    .toHaveValue('admin');

Example:

await page.getByLabel('Username').fill('admin');


await expect(page.getByLabel('Username'))
    .toHaveValue('admin');
15. toHaveAttribute()

Verifies an element attribute.

await expect(page.locator('#login'))
    .toHaveAttribute('type', 'submit');

Example:

await expect(page.locator('#home'))
    .toHaveAttribute('href', '/home');
16. toHaveClass()

Verifies the CSS class of an element.

await expect(page.locator('#message'))
    .toHaveClass('success');

Multiple classes:

await expect(page.locator('#message'))
    .toHaveClass(/success/);
17. toHaveId()

Verifies the ID of an element.

await expect(page.locator('button'))
    .toHaveId('loginButton');
18. toHaveCSS()

Verifies a CSS property.

await expect(page.locator('#message'))
    .toHaveCSS('display', 'block');

Example:

await expect(page.locator('#message'))
    .toHaveCSS('font-size', '16px');
19. toHaveCount()

Verifies the number of matching elements.

await expect(page.locator('.product'))
    .toHaveCount(10);

Example:

const products = page.locator('.product');


await expect(products).toHaveCount(5);

This is useful for:

Tables
Product lists
Menus
Search results
Repeated elements
20. toHaveURL()

Verifies the current page URL.

await expect(page)
    .toHaveURL('https://example.com/dashboard');

Regular expression:

await expect(page)
    .toHaveURL(/dashboard/);

Example:

await page.getByRole('button', {
    name: 'Login'
}).click();


await expect(page).toHaveURL(/dashboard/);
21. toHaveTitle()

Verifies the page title.

await expect(page)
    .toHaveTitle('Dashboard');

Regular expression:

await expect(page)
    .toHaveTitle(/Dashboard/);
22. toHaveScreenshot()

Playwright supports screenshot-based assertions.

Example:

await expect(page).toHaveScreenshot('homepage.png');

This compares the current screenshot with a previously stored baseline.

Useful for:

Visual regression testing
UI validation
Layout validation
Cross-browser visual testing
23. Negative Assertions

Use:

.not

to verify that something is NOT true.

Example:

await expect(page.locator('#error'))
    .not.toBeVisible();

Another example:

await expect(page.locator('#submit'))
    .not.toBeDisabled();

Text:

await expect(page.locator('#message'))
    .not.toContainText('Error');

URL:

await expect(page)
    .not.toHaveURL(/error/);
24. Soft Assertions

By default, a failed assertion causes the test to fail immediately.

A soft assertion allows the test to continue.

Use:

expect.soft()

Example:

await expect.soft(page.locator('#username'))
    .toBeVisible();


await expect.soft(page.locator('#password'))
    .toBeVisible();


await expect.soft(page.locator('#login'))
    .toBeVisible();

All assertions are evaluated before the test reports the failures.

25. Normal vs Soft Assertion

Normal assertion:

await expect(page.locator('#username'))
    .toBeVisible();


await expect(page.locator('#password'))
    .toBeVisible();

If the first assertion fails, the second assertion is not executed.

Soft assertion:

await expect.soft(page.locator('#username'))
    .toBeVisible();


await expect.soft(page.locator('#password'))
    .toBeVisible();

The test continues after the first failure.

26. Custom Assertion Timeout

Playwright assertions have their own timeout.

You can specify a timeout:

await expect(page.locator('#message'))
    .toBeVisible({
        timeout: 10000
    });

This waits up to 10 seconds.

Example:

await expect(page.getByText('Payment Successful'))
    .toBeVisible({
        timeout: 15000
    });
27. Assertion Timeout in Configuration

You can configure the default expectation timeout in playwright.config.js.

import { defineConfig } from '@playwright/test';


export default defineConfig({
    expect: {
        timeout: 10000
    }
});

Now assertions will wait up to 10 seconds by default.

28. Assertion Timeout vs Test Timeout

These are different concepts.

Assertion timeout:

expect: {
    timeout: 10000
}

Controls how long an assertion waits.

Test timeout:

timeout: 30000

Controls the maximum duration of the test.

Example:

export default defineConfig({
    timeout: 30000,


    expect: {
        timeout: 10000
    }
});
29. expect.poll()

expect.poll() is useful when you need to repeatedly evaluate a custom function until the expected result is reached.

Example:

await expect.poll(async () => {
    return await page.locator('#status').textContent();
}).toBe('Completed');

Another example:

await expect.poll(async () => {
    const response = await page.request.get('/api/status');


    return response.status();
}).toBe(200);
30. expect.poll() with Timeout
await expect.poll(
    async () => {
        return await getStatus();
    },
    {
        timeout: 15000
    }
).toBe('Completed');

Useful for:

Polling APIs
Background jobs
Asynchronous processing
Eventual consistency
Long-running operations
31. expect.toPass()

toPass() retries a block of code until all assertions inside it pass.

Example:

await expect(async () => {
    const status = await page.locator('#status').textContent();


    expect(status).toBe('Completed');
}).toPass();

With timeout:

await expect(async () => {
    const status = await page.locator('#status').textContent();


    expect(status).toBe('Completed');
}).toPass({
    timeout: 15000
});
32. Locator Assertions

The most common Playwright assertions are locator assertions.

Example:

const loginButton = page.getByRole('button', {
    name: 'Login'
});


await expect(loginButton).toBeVisible();
await expect(loginButton).toBeEnabled();
await expect(loginButton).toHaveText('Login');

This is preferred over manually retrieving values.

33. Avoid Manual Assertions When Possible

Avoid:

const text = await page.locator('#message').textContent();


if (text !== 'Success') {
    throw new Error('Message is incorrect');
}

Prefer:

await expect(page.locator('#message'))
    .toHaveText('Success');

Why?

Because Playwright's assertion:

Automatically waits
Provides better error messages
Handles retries
Integrates with Playwright Test
Produces useful reports
34. Avoid Unnecessary waitForTimeout()

Avoid:

await page.waitForTimeout(5000);


await expect(page.locator('#message'))
    .toBeVisible();

Prefer:

await expect(page.locator('#message'))
    .toBeVisible();

The assertion automatically waits for the condition.

35. Assertions with getByRole()

Example:

const loginButton = page.getByRole('button', {
    name: 'Login'
});


await expect(loginButton).toBeVisible();
await expect(loginButton).toBeEnabled();

This provides a readable and maintainable test.

36. Assertions with getByText()
const message = page.getByText('Login successful');


await expect(message).toBeVisible();

Example:

await expect(
    page.getByText('Welcome to Dashboard')
).toBeVisible();
37. Assertions with getByLabel()
const username = page.getByLabel('Username');


await username.fill('admin');


await expect(username)
    .toHaveValue('admin');
38. Assertions with getByPlaceholder()
const searchBox = page.getByPlaceholder('Search');


await searchBox.fill('Toyota');


await expect(searchBox)
    .toHaveValue('Toyota');
39. Assertions with Multiple Elements

Suppose the page contains:

<ul>
    <li>Apple</li>
    <li>Orange</li>
    <li>Banana</li>
</ul>

You can verify the count:

await expect(page.locator('li'))
    .toHaveCount(3);

You can verify text:

await expect(page.locator('li'))
    .toHaveText([
        'Apple',
        'Orange',
        'Banana'
    ]);
40. Assertions with Arrays of Text
const items = page.locator('.item');


await expect(items).toHaveText([
    'Item 1',
    'Item 2',
    'Item 3'
]);

This verifies the text of each matching element.

41. Verify Checkbox
const terms = page.getByLabel('I agree to the terms');


await terms.check();


await expect(terms).toBeChecked();
42. Verify Input
const email = page.getByLabel('Email');


await email.fill('test@example.com');


await expect(email)
    .toHaveValue('test@example.com');
43. Verify Button State
const submit = page.getByRole('button', {
    name: 'Submit'
});


await expect(submit).toBeEnabled();

Disabled:

await expect(submit).toBeDisabled();
44. Verify Element Attribute
const link = page.getByRole('link', {
    name: 'Home'
});


await expect(link)
    .toHaveAttribute('href', '/home');
45. Verify URL After Navigation
await page.getByRole('link', {
    name: 'Dashboard'
}).click();


await expect(page)
    .toHaveURL(/dashboard/);
46. Verify Page Title
await page.goto('https://example.com');


await expect(page)
    .toHaveTitle(/Example/);
47. Verify Modal
const modal = page.getByRole('dialog');


await expect(modal).toBeVisible();

After closing:

await page.getByRole('button', {
    name: 'Close'
}).click();


await expect(modal).toBeHidden();
48. Verify Error Message
await page.getByRole('button', {
    name: 'Login'
}).click();


await expect(page.getByText('Invalid username or password'))
    .toBeVisible();
49. Assertion Example - Login
import { test, expect } from '@playwright/test';


test('Verify successful login', async ({ page }) => {


    await page.goto('https://example.com/login');


    await page.getByLabel('Username').fill('admin');


    await page.getByLabel('Password').fill('password');


    await page.getByRole('button', {
        name: 'Login'
    }).click();


    await expect(page)
        .toHaveURL(/dashboard/);


    await expect(
        page.getByText('Welcome')
    ).toBeVisible();
});
50. Assertion Example - Form Validation
import { test, expect } from '@playwright/test';


test('Verify required field validation', async ({ page }) => {


    await page.goto('https://example.com/register');


    await page.getByRole('button', {
        name: 'Register'
    }).click();


    await expect(
        page.getByText('Username is required')
    ).toBeVisible();


    await expect(
        page.getByText('Password is required')
    ).toBeVisible();
});
51. Assertion Example - Search
import { test, expect } from '@playwright/test';


test('Verify search results', async ({ page }) => {


    await page.goto('https://example.com');


    const search = page.getByPlaceholder('Search');


    await search.fill('Toyota');


    await page.getByRole('button', {
        name: 'Search'
    }).click();


    const results = page.locator('.search-result');


    await expect(results).toHaveCount(5);


    await expect(results.first())
        .toContainText('Toyota');
});
52. Assertion Example - Table
const rows = page.locator('table tbody tr');


await expect(rows).toHaveCount(10);


await expect(rows.first())
    .toContainText('Toyota');
53. Assertion Example - API

Playwright can also perform API testing.

import { test, expect } from '@playwright/test';


test('Verify API response', async ({ request }) => {


    const response = await request.get('/api/users');


    expect(response.status()).toBe(200);


    const body = await response.json();


    expect(body.users.length).toBeGreaterThan(0);
});

Here:

expect(response.status()).toBe(200);

is a synchronous assertion because the status value has already been obtained.

54. Common Assertion Categories
Page Assertions
await expect(page).toHaveURL('/dashboard');


await expect(page).toHaveTitle(/Dashboard/);
Locator Assertions
await expect(locator).toBeVisible();


await expect(locator).toBeEnabled();


await expect(locator).toHaveText('Success');


await expect(locator).toHaveValue('admin');


await expect(locator).toHaveAttribute('type', 'button');


await expect(locator).toHaveCount(5);
Generic Assertions
expect(value).toBe(10);


expect(value).toEqual(10);


expect(value).toContain('Toyota');


expect(value).toBeTruthy();


expect(value).toBeFalsy();


expect(value).toBeNull();


expect(value).toBeDefined();
55. Generic expect()

Playwright's expect() can also be used with normal JavaScript values.

Example:

const actual = 10;


expect(actual).toBe(10);

String:

const name = 'Toyota';


expect(name).toBe('Toyota');

Contains:

expect(name).toContain('Toy');

Boolean:

const loggedIn = true;


expect(loggedIn).toBeTruthy();
56. toEqual()

toEqual() is commonly used for objects and arrays.

Example:

const actual = {
    name: 'Toyota',
    country: 'Japan'
};


expect(actual).toEqual({
    name: 'Toyota',
    country: 'Japan'
});

Array:

expect([1, 2, 3]).toEqual([1, 2, 3]);
57. toBe()

toBe() checks strict equality.

expect(10).toBe(10);


expect('Toyota').toBe('Toyota');


expect(true).toBe(true);

For objects, prefer:

toEqual()
58. toContain()
expect('Playwright Automation')
    .toContain('Playwright');

Array:

expect(['Java', 'JavaScript', 'Python'])
    .toContain('JavaScript');
59. toBeTruthy()
expect(true).toBeTruthy();

Example:

const responseReceived = true;


expect(responseReceived).toBeTruthy();
60. toBeFalsy()
expect(false).toBeFalsy();
61. toBeNull()
expect(null).toBeNull();
62. toBeDefined()
expect(value).toBeDefined();
63. Multiple Assertions

A single test can contain multiple assertions.

test('Verify dashboard', async ({ page }) => {


    await page.goto('/dashboard');


    await expect(page)
        .toHaveTitle(/Dashboard/);


    await expect(page)
        .toHaveURL(/dashboard/);


    await expect(
        page.getByText('Welcome')
    ).toBeVisible();


    await expect(
        page.getByRole('button', {
            name: 'Logout'
        })
    ).toBeVisible();
});
64. Assertions in Page Object Model

Assertions can be placed in test classes or page objects depending on framework design.

Example Page Object:

export class LoginPage {


    constructor(page) {
        this.page = page;


        this.username = page.getByLabel('Username');
        this.password = page.getByLabel('Password');


        this.loginButton = page.getByRole('button', {
            name: 'Login'
        });
    }


    async login(username, password) {
        await this.username.fill(username);
        await this.password.fill(password);
        await this.loginButton.click();
    }
}

Test:

import { test, expect } from '@playwright/test';
import { LoginPage } from './pages/LoginPage';


test('Login test', async ({ page }) => {


    const loginPage = new LoginPage(page);


    await page.goto('/login');


    await loginPage.login('admin', 'password');


    await expect(page)
        .toHaveURL(/dashboard/);
});

A common best practice is to keep most verification/assertion logic in the test layer while page objects focus on page behavior and locators.

65. Assertions vs Actions

Actions perform operations:

await page.getByLabel('Username').fill('admin');


await page.getByRole('button', {
    name: 'Login'
}).click();

Assertions verify results:

await expect(page)
    .toHaveURL(/dashboard/);


await expect(
    page.getByText('Welcome')
).toBeVisible();

Remember:

Action      → Do something
Assertion   → Verify something
66. Selenium vs Playwright Assertions
Selenium + Java
Assert.assertEquals(
    driver.getTitle(),
    "Dashboard"
);
Playwright
await expect(page)
    .toHaveTitle('Dashboard');
Selenium
Assert.assertTrue(
    element.isDisplayed()
);
Playwright
await expect(locator)
    .toBeVisible();
Selenium
Assert.assertEquals(
    element.getText(),
    "Login Successful"
);
Playwright
await expect(locator)
    .toHaveText('Login Successful');
67. Important Playwright Assertion Advantage

Selenium often requires explicit waits around dynamic conditions.

Example Selenium:

WebDriverWait wait = new WebDriverWait(
    driver,
    Duration.ofSeconds(10)
);


wait.until(
    ExpectedConditions.visibilityOfElementLocated(
        By.id("message")
    )
);


Assert.assertEquals(
    driver.findElement(By.id("message")).getText(),
    "Success"
);

Playwright:

await expect(page.locator('#message'))
    .toHaveText('Success');

The Playwright assertion automatically waits for the expected condition.

68. Common Mistakes
Mistake 1: Using waitForTimeout()

Avoid:

await page.waitForTimeout(5000);


await expect(locator).toBeVisible();

Prefer:

await expect(locator).toBeVisible();
Mistake 2: Manually checking visibility

Avoid:

const visible = await locator.isVisible();


expect(visible).toBe(true);

Prefer:

await expect(locator).toBeVisible();
Mistake 3: Manually retrieving text

Avoid:

const text = await locator.textContent();


expect(text).toBe('Success');

Prefer:

await expect(locator).toHaveText('Success');
Mistake 4: Using overly generic selectors

Avoid:

await expect(page.locator('div')).toBeVisible();

Prefer:

await expect(
    page.getByRole('button', {
        name: 'Submit'
    })
).toBeVisible();
69. Best Practices
Prefer Playwright's web-first assertions.
Use expect(locator) for UI verification.
Avoid unnecessary waitForTimeout().
Prefer role, label, text, and test-id locators.
Use toHaveText() instead of manually calling textContent().
Use toHaveValue() for input values.
Use toBeVisible() for visibility.
Use toBeEnabled() and toBeDisabled() for button state.
Use toHaveURL() for navigation validation.
Use toHaveTitle() for title validation.
Use soft assertions when multiple independent validations are useful.
Use expect.poll() for polling custom conditions.
Keep assertion messages meaningful through clear test names and locators.
Avoid unnecessary custom waits.
Keep assertions close to the behavior they validate.
70. Quick Reference
Assertion	Purpose
toBeVisible()	Element is visible
toBeHidden()	Element is hidden
toBeEnabled()	Element is enabled
toBeDisabled()	Element is disabled
toBeEditable()	Element can be edited
toBeChecked()	Checkbox/radio is checked
toHaveText()	Verify exact text
toContainText()	Verify partial text
toHaveValue()	Verify input value
toHaveAttribute()	Verify attribute
toHaveClass()	Verify CSS class
toHaveId()	Verify ID
toHaveCSS()	Verify CSS property
toHaveCount()	Verify element count
toHaveURL()	Verify URL
toHaveTitle()	Verify page title
toHaveScreenshot()	Visual comparison
expect.poll()	Retry custom condition
expect.soft()	Continue after assertion failure
toPass()	Retry assertion block
71. Interview Questions
Q1. What is assertion in Playwright?

An assertion verifies that the application is in the expected state.

Example:

await expect(locator).toBeVisible();
Q2. Are Playwright assertions automatically waiting?

Yes. Playwright's web-first assertions automatically retry until the expected condition is met or the assertion timeout is reached.

Q3. What is the difference between toHaveText() and toContainText()?

toHaveText() verifies the expected text more strictly.

await expect(locator).toHaveText('Login Successful');

toContainText() verifies that the expected text is contained within the element's text.

await expect(locator).toContainText('Successful');
Q4. How do you perform a negative assertion?

Use .not.

await expect(locator).not.toBeVisible();
Q5. What is a soft assertion?

A soft assertion allows the test to continue after an assertion failure.

await expect.soft(locator).toBeVisible();
Q6. How do you verify the current URL?
await expect(page).toHaveURL(/dashboard/);
Q7. How do you verify page title?
await expect(page).toHaveTitle('Dashboard');
Q8. How do you verify an input value?
await expect(locator).toHaveValue('admin');
Q9. How do you verify element count?
await expect(page.locator('.product'))
    .toHaveCount(10);
Q10. What is expect.poll()?

expect.poll() repeatedly evaluates a custom function until the expected result is achieved.

await expect.poll(async () => {
    return await getStatus();
}).toBe('Completed');
72. Final Example
import { test, expect } from '@playwright/test';


test('Complete assertion example', async ({ page }) => {


    await page.goto('https://example.com/login');


    // Page assertion
    await expect(page)
        .toHaveTitle(/Login/);


    // Input assertion
    const username = page.getByLabel('Username');


    await expect(username)
        .toBeVisible();


    await expect(username)
        .toBeEditable();


    await username.fill('admin');


    await expect(username)
        .toHaveValue('admin');


    // Password
    const password = page.getByLabel('Password');


    await password.fill('password');


    // Button assertion
    const loginButton = page.getByRole('button', {
        name: 'Login'
    });


    await expect(loginButton)
        .toBeVisible();


    await expect(loginButton)
        .toBeEnabled();


    // Action
    await loginButton.click();


    // Navigation assertion
    await expect(page)
        .toHaveURL(/dashboard/);


    // Content assertion
    await expect(
        page.getByText('Welcome')
    ).toBeVisible();
});
Summary

Playwright assertions are a core part of Playwright Test.

The most important assertions to remember are:

await expect(locator).toBeVisible();


await expect(locator).toBeHidden();


await expect(locator).toBeEnabled();


await expect(locator).toBeDisabled();


await expect(locator).toHaveText('text');


await expect(locator).toContainText('text');


await expect(locator).toHaveValue('value');


await expect(locator).toBeChecked();


await expect(locator).toHaveAttribute('href', '/home');


await expect(locator).toHaveCount(5);


await expect(page).toHaveURL(/dashboard/);


await expect(page).toHaveTitle(/Dashboard/);

For advanced scenarios:

await expect.soft(locator).toBeVisible();


await expect.poll(async () => {
    return await getStatus();
}).toBe('Completed');


await expect(async () => {
    // assertions
}).toPass();

The key Playwright principle is:

ACTION → ASSERTION


fill() / click() / selectOption()
              ↓
           expect()
              ↓
         Verify result

Playwright's auto-waiting + web-first assertions are one of the biggest advantages over traditional Selenium synchronization patterns.


