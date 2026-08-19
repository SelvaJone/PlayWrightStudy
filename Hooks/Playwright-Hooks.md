# Playwright Hooks

## 1. What Are Hooks?

Hooks are special methods that run automatically before or after test execution.

Playwright Test provides hooks that allow us to perform setup and cleanup activities around tests.

Common Playwright hooks:

* `test.beforeAll()`
* `test.afterAll()`
* `test.beforeEach()`
* `test.afterEach()`

Hooks are useful for:

* Test setup
* Test cleanup
* Login/setup actions
* Creating test data
* Closing resources
* Resetting application state
* Preparing common test conditions

---

## 2. Import Playwright Test

```javascript
import { test, expect } from '@playwright/test';
```

---

# 3. beforeEach()

`beforeEach()` runs before every test in the scope where it is defined.

### Example

```javascript
import { test, expect } from '@playwright/test';

test.beforeEach(async ({ page }) => {
    await page.goto('https://example.com');
});

test('Test 1', async ({ page }) => {
    await expect(page).toHaveTitle(/Example/);
});

test('Test 2', async ({ page }) => {
    await expect(page.locator('h1')).toHaveText('Example Domain');
});
```

Execution:

```text
beforeEach
   ↓
Test 1

beforeEach
   ↓
Test 2
```

---

# 4. afterEach()

`afterEach()` runs after every test.

### Example

```javascript
import { test, expect } from '@playwright/test';

test.afterEach(async ({ page }) => {
    console.log('Test completed');
});

test('Login test', async ({ page }) => {
    await page.goto('https://example.com');

    await expect(page).toHaveTitle(/Example/);
});
```

Execution:

```text
Test
 ↓
afterEach
```

---

# 5. beforeAll()

`beforeAll()` runs once before all tests in a file or test group.

### Example

```javascript
import { test, expect } from '@playwright/test';

test.beforeAll(async () => {
    console.log('Runs once before all tests');
});

test('Test 1', async ({ page }) => {
    await page.goto('https://example.com');
});

test('Test 2', async ({ page }) => {
    await page.goto('https://example.com');
});
```

Execution:

```text
beforeAll
   ↓
Test 1
   ↓
Test 2
```

Unlike `beforeEach()`, `beforeAll()` does not run before every test.

---

# 6. afterAll()

`afterAll()` runs once after all tests in the scope have completed.

### Example

```javascript
import { test } from '@playwright/test';

test.afterAll(async () => {
    console.log('Runs once after all tests');
});

test('Test 1', async ({ page }) => {
    await page.goto('https://example.com');
});

test('Test 2', async ({ page }) => {
    await page.goto('https://example.com');
});
```

Execution:

```text
Test 1
   ↓
Test 2
   ↓
afterAll
```

---

# 7. Using All Four Hooks

```javascript
import { test, expect } from '@playwright/test';

test.beforeAll(async () => {
    console.log('Before all tests');
});

test.beforeEach(async ({ page }) => {
    console.log('Before each test');

    await page.goto('https://example.com');
});

test.afterEach(async () => {
    console.log('After each test');
});

test.afterAll(async () => {
    console.log('After all tests');
});

test('Test 1', async ({ page }) => {
    await expect(page).toHaveTitle(/Example/);
});

test('Test 2', async ({ page }) => {
    await expect(page.locator('h1')).toHaveText('Example Domain');
});
```

Execution order:

```text
beforeAll

beforeEach
Test 1
afterEach

beforeEach
Test 2
afterEach

afterAll
```

---

# 8. Hooks Inside test.describe()

Hooks can be scoped to a specific test group.

```javascript
import { test, expect } from '@playwright/test';

test.describe('Login Tests', () => {

    test.beforeEach(async ({ page }) => {
        await page.goto('https://example.com/login');
    });

    test.afterEach(async () => {
        console.log('Login test completed');
    });

    test('Valid Login', async ({ page }) => {
        // Login test
    });

    test('Invalid Login', async ({ page }) => {
        // Invalid login test
    });
});
```

The hooks apply only to tests inside the `describe` block.

---

# 9. beforeAll() Inside describe()

```javascript
test.describe('User Tests', () => {

    test.beforeAll(async () => {
        console.log('Create test data');
    });

    test('Create User', async ({ page }) => {
        // Test
    });

    test('Update User', async ({ page }) => {
        // Test
    });

});
```

`beforeAll()` runs once for this test group.

---

# 10. afterAll() Inside describe()

```javascript
test.describe('User Tests', () => {

    test.afterAll(async () => {
        console.log('Clean up test data');
    });

    test('Create User', async ({ page }) => {
        // Test
    });

    test('Update User', async ({ page }) => {
        // Test
    });

});
```

This is useful for cleanup activities.

---

# 11. Nested Hooks

Hooks can be used with nested `describe()` blocks.

```javascript
import { test } from '@playwright/test';

test.beforeEach(async () => {
    console.log('Outer beforeEach');
});

test.describe('Login Tests', () => {

    test.beforeEach(async () => {
        console.log('Inner beforeEach');
    });

    test('Login Test', async ({ page }) => {
        console.log('Test');
    });

});
```

The outer hook applies to the nested test as well.

Execution:

```text
Outer beforeEach
       ↓
Inner beforeEach
       ↓
Test
```

---

# 12. Hook Execution Order

For a simple test:

```javascript
test.beforeAll()
test.beforeEach()
test()
test.afterEach()
test.afterAll()
```

Example:

```javascript
test.beforeAll(async () => {
    console.log('1 - beforeAll');
});

test.beforeEach(async () => {
    console.log('2 - beforeEach');
});

test('Test', async () => {
    console.log('3 - Test');
});

test.afterEach(async () => {
    console.log('4 - afterEach');
});

test.afterAll(async () => {
    console.log('5 - afterAll');
});
```

Output:

```text
1 - beforeAll
2 - beforeEach
3 - Test
4 - afterEach
5 - afterAll
```

---

# 13. Hooks and Fixtures

Playwright hooks can use fixtures.

For example, `page` can be used inside `beforeEach()`.

```javascript
test.beforeEach(async ({ page }) => {
    await page.goto('https://example.com');
});
```

The `page` fixture is automatically provided by Playwright.

Other commonly used fixtures include:

```text
page
browser
context
request
```

---

# 14. Login Using beforeEach()

A common automation pattern is to perform login before every test.

```javascript
import { test, expect } from '@playwright/test';

test.beforeEach(async ({ page }) => {

    await page.goto('https://example.com/login');

    await page.locator('#username').fill('admin');
    await page.locator('#password').fill('password');

    await page.locator('#login').click();
});

test('Dashboard Test', async ({ page }) => {

    await expect(page.locator('#dashboard')).toBeVisible();
});

test('Profile Test', async ({ page }) => {

    await page.locator('#profile').click();

    await expect(page.locator('#profilePage')).toBeVisible();
});
```

This prevents duplicate login code in every test.

---

# 15. Cleanup Using afterEach()

```javascript
test.afterEach(async ({ page }) => {

    await page.evaluate(() => {
        localStorage.clear();
        sessionStorage.clear();
    });

});
```

However, prefer Playwright's isolated browser contexts and fixtures instead of manually clearing application state when possible.

---

# 16. Creating Test Data with beforeAll()

```javascript
let userId;

test.beforeAll(async ({ request }) => {

    const response = await request.post('/api/users', {
        data: {
            name: 'Test User',
            email: 'test@example.com'
        }
    });

    const data = await response.json();

    userId = data.id;
});
```

The created data can then be used by tests.

---

# 17. Cleanup Test Data with afterAll()

```javascript
test.afterAll(async ({ request }) => {

    if (userId) {
        await request.delete(`/api/users/${userId}`);
    }

});
```

This is useful for API-based test data cleanup.

---

# 18. Hooks with API Testing

Playwright can use the `request` fixture inside hooks.

```javascript
import { test, expect } from '@playwright/test';

let userId;

test.beforeAll(async ({ request }) => {

    const response = await request.post('/api/users', {
        data: {
            name: 'Selva',
            email: 'selva@example.com'
        }
    });

    const body = await response.json();

    userId = body.id;
});

test('Get User', async ({ request }) => {

    const response = await request.get(`/api/users/${userId}`);

    expect(response.ok()).toBeTruthy();
});

test.afterAll(async ({ request }) => {

    await request.delete(`/api/users/${userId}`);
});
```

---

# 19. Hook Failure

If a `beforeEach()` hook fails, the test normally cannot continue successfully.

Example:

```javascript
test.beforeEach(async ({ page }) => {

    await page.goto('https://example.com');

    await expect(page.locator('#login')).toBeVisible();
});
```

If the login element does not exist, the hook fails and the test will be marked accordingly.

---

# 20. Hook Timeout

Hooks can also be affected by Playwright timeouts.

```javascript
test.beforeEach(async ({ page }) => {

    test.setTimeout(60000);

    await page.goto('https://example.com');
});
```

It is generally better to configure appropriate timeouts centrally rather than increasing them unnecessarily inside individual hooks.

---

# 21. Hooks vs Fixtures

Hooks:

```javascript
test.beforeEach(async ({ page }) => {
    await page.goto('/login');
});
```

Fixtures:

```javascript
const test = base.extend({
    loggedInPage: async ({ page }, use) => {

        await page.goto('/login');

        // Login

        await use(page);
    }
});
```

### When to use hooks

Use hooks for:

* Simple setup
* Simple cleanup
* Common initialization
* Test-specific preparation

### When to use fixtures

Use fixtures for:

* Reusable test dependencies
* Complex setup
* Authentication
* Page objects
* Test data
* Custom browser contexts
* Dependency injection

---

# 22. Hooks vs Page Object Model

Avoid putting too much business logic inside hooks.

### Avoid

```javascript
test.beforeEach(async ({ page }) => {

    await page.goto('/login');

    await page.locator('#username').fill('admin');
    await page.locator('#password').fill('password');
    await page.locator('#login').click();

    await page.locator('#products').click();
    await page.locator('#search').fill('Laptop');
});
```

This makes the hook difficult to maintain.

### Better

Use a Page Object.

```javascript
test.beforeEach(async ({ loginPage }) => {

    await loginPage.login('admin', 'password');

});
```

---

# 23. Global Setup vs Hooks

Playwright also supports `globalSetup`.

### Hooks

```javascript
test.beforeAll(async () => {
    // Setup for this test file/group
});
```

### Global setup

```javascript
// global-setup.js

async function globalSetup() {
    console.log('Global setup');
}

export default globalSetup;
```

Configured in:

```javascript
// playwright.config.js

export default {
    globalSetup: './global-setup.js'
};
```

### Difference

| Feature                        | Hooks           | Global Setup        |
| ------------------------------ | --------------- | ------------------- |
| Scope                          | Test/file/group | Entire test run     |
| beforeEach available           | Yes             | No                  |
| afterEach available            | Yes             | No                  |
| Test fixtures                  | Yes             | Different mechanism |
| Common test setup              | Yes             | Yes                 |
| One-time global initialization | Sometimes       | Yes                 |

---

# 24. Best Practices

### 1. Keep hooks small

```javascript
test.beforeEach(async ({ page }) => {
    await page.goto('/login');
});
```

Avoid large amounts of business logic.

### 2. Don't hide important test steps

A test should remain understandable.

### 3. Use fixtures for reusable complex setup

If many tests require the same dependency, a fixture is often better than a large hook.

### 4. Use beforeAll carefully

Do not create shared mutable state that causes tests to depend on one another.

### 5. Prefer test isolation

Each test should ideally be independent.

### 6. Keep cleanup reliable

Use `afterEach()` or `afterAll()` when cleanup is required.

### 7. Avoid unnecessary hooks

Do not create hooks just because Playwright supports them.

---

# 25. Complete Example

```javascript
import { test, expect } from '@playwright/test';

test.describe('Product Tests', () => {

    test.beforeAll(async () => {
        console.log('Starting Product Test Suite');
    });

    test.beforeEach(async ({ page }) => {

        await page.goto('https://example.com');

        console.log('Application opened');
    });

    test('Verify Page Title', async ({ page }) => {

        await expect(page).toHaveTitle(/Example/);
    });

    test('Verify Heading', async ({ page }) => {

        await expect(page.locator('h1'))
            .toHaveText('Example Domain');
    });

    test.afterEach(async () => {

        console.log('Test completed');
    });

    test.afterAll(async () => {

        console.log('Product Test Suite completed');
    });

});
```

Execution:

```text
beforeAll
    ↓
beforeEach
    ↓
Verify Page Title
    ↓
afterEach
    ↓
beforeEach
    ↓
Verify Heading
    ↓
afterEach
    ↓
afterAll
```

---

# 26. Interview Questions

### Q1. What are Playwright hooks?

Hooks are lifecycle methods used to execute setup and cleanup code before or after tests.

---

### Q2. What is the difference between beforeEach and beforeAll?

`beforeEach()` runs before every test.

`beforeAll()` runs once before all tests in the applicable scope.

---

### Q3. What is the difference between afterEach and afterAll?

`afterEach()` runs after every test.

`afterAll()` runs once after all tests in the applicable scope.

---

### Q4. Can hooks be used inside describe blocks?

Yes.

```javascript
test.describe('Login Tests', () => {

    test.beforeEach(async ({ page }) => {
        await page.goto('/login');
    });

});
```

The hook applies to tests inside that scope.

---

### Q5. Can Playwright fixtures be used in hooks?

Yes.

```javascript
test.beforeEach(async ({ page }) => {
    await page.goto('/login');
});
```

---

### Q6. Should we put login logic in beforeEach?

It depends.

For simple tests, `beforeEach()` can be appropriate.

For reusable or complex authentication, Playwright fixtures or authentication state are generally better.

---

### Q7. What happens if beforeEach fails?

The test setup fails, so the test cannot proceed normally.

---

### Q8. Can we have multiple beforeEach hooks?

Yes, hooks can be defined at different scopes, such as outer and nested `describe()` blocks.

---

### Q9. What is the difference between hooks and fixtures?

Hooks provide lifecycle setup and cleanup.

Fixtures provide reusable test dependencies and are particularly useful for complex setup, authentication, page objects, and test data.

---

### Q10. What is the difference between globalSetup and beforeAll?

`globalSetup` is intended for setup for the overall test run.

`beforeAll()` is scoped to a test file or test group.

---

# 27. Quick Reference

```text
Playwright Hooks
│
├── beforeAll()
│   └── Runs once before tests
│
├── beforeEach()
│   └── Runs before every test
│
├── Test
│
├── afterEach()
│   └── Runs after every test
│
└── afterAll()
    └── Runs once after tests
```

## Hook Syntax

```javascript
test.beforeAll(async () => {
    // Setup once
});

test.beforeEach(async ({ page }) => {
    // Setup before each test
});

test.afterEach(async ({ page }) => {
    // Cleanup after each test
});

test.afterAll(async () => {
    // Cleanup once
});
```

## Recommended Strategy

```text
Simple setup/cleanup
        ↓
      Hooks
        ↓
Complex reusable setup
        ↓
     Fixtures
        ↓
Entire test-run setup
        ↓
   globalSetup
```

---

# Summary

Playwright hooks provide a clean way to manage test lifecycle activities.

The four primary hooks are:

```javascript
test.beforeAll()
test.beforeEach()
test.afterEach()
test.afterAll()
```

Remember:

```text
beforeAll  → once before tests
beforeEach → before every test
afterEach  → after every test
afterAll   → once after tests
```

For simple setup and cleanup, use hooks.

For complex reusable setup such as authentication, Page Objects, test data, or custom contexts, prefer Playwright fixtures.
