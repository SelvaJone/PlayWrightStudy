# Playwright Test Suites

## Overview

Playwright Test provides a powerful test runner for organizing, grouping, filtering, and executing automated tests.

A **test suite** is a collection of related tests organized around a feature, module, workflow, or functionality.

A well-designed Playwright test suite should be:

* Easy to understand
* Easy to maintain
* Independently executable
* Reusable
* Scalable
* Suitable for parallel execution
* Easy to run in CI/CD
* Easy to debug

---

## 1. Basic Test Suite

A Playwright test suite can be created using `test.describe()`.

```javascript
import { test, expect } from '@playwright/test';

test.describe('Login Test Suite', () => {

  test('Valid login', async ({ page }) => {
    await page.goto('https://example.com/login');

    await page.getByLabel('Username').fill('admin');
    await page.getByLabel('Password').fill('password');

    await page.getByRole('button', { name: 'Login' }).click();

    await expect(page).toHaveURL(/dashboard/);
  });

  test('Invalid login', async ({ page }) => {
    await page.goto('https://example.com/login');

    await page.getByLabel('Username').fill('invalid');
    await page.getByLabel('Password').fill('invalid');

    await page.getByRole('button', { name: 'Login' }).click();

    await expect(page.getByText('Invalid credentials')).toBeVisible();
  });

});
```

---

# 2. Why Use Test Suites?

Test suites help organize tests logically.

For example:

```text
tests/
├── login.spec.js
├── registration.spec.js
├── checkout.spec.js
└── profile.spec.js
```

Each file can contain a test suite.

Example:

```javascript
test.describe('Login Tests', () => {
  // Login tests
});
```

This makes the automation framework easier to maintain.

---

# 3. Grouping Tests with describe()

Use `test.describe()` to group related tests.

```javascript
import { test, expect } from '@playwright/test';

test.describe('User Management', () => {

  test('Create user', async ({ page }) => {
    // test
  });

  test('Update user', async ({ page }) => {
    // test
  });

  test('Delete user', async ({ page }) => {
    // test
  });

});
```

The resulting hierarchy is:

```text
User Management
├── Create user
├── Update user
└── Delete user
```

---

# 4. Nested Test Suites

Playwright supports nested `describe()` blocks.

```javascript
import { test } from '@playwright/test';

test.describe('E-Commerce Application', () => {

  test.describe('Login', () => {

    test('Valid login', async ({ page }) => {
      // test
    });

    test('Invalid login', async ({ page }) => {
      // test
    });

  });

  test.describe('Shopping Cart', () => {

    test('Add product to cart', async ({ page }) => {
      // test
    });

    test('Remove product from cart', async ({ page }) => {
      // test
    });

  });

});
```

Hierarchy:

```text
E-Commerce Application
│
├── Login
│   ├── Valid login
│   └── Invalid login
│
└── Shopping Cart
    ├── Add product to cart
    └── Remove product from cart
```

Nested suites are useful for large applications.

---

# 5. beforeEach()

`beforeEach()` executes before every test in the suite.

```javascript
import { test } from '@playwright/test';

test.describe('Login Tests', () => {

  test.beforeEach(async ({ page }) => {
    await page.goto('https://example.com/login');
  });

  test('Valid login', async ({ page }) => {
    // test
  });

  test('Invalid login', async ({ page }) => {
    // test
  });

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

# 6. afterEach()

`afterEach()` executes after every test.

```javascript
test.describe('Login Tests', () => {

  test.afterEach(async ({ page }) => {
    console.log('Test completed');
  });

  test('Valid login', async ({ page }) => {
    // test
  });

  test('Invalid login', async ({ page }) => {
    // test
  });

});
```

---

# 7. beforeAll()

`beforeAll()` runs once before all tests in the suite.

```javascript
test.describe('User Tests', () => {

  test.beforeAll(async () => {
    console.log('Starting User Test Suite');
  });

  test('Create user', async ({ page }) => {
    // test
  });

  test('Update user', async ({ page }) => {
    // test
  });

});
```

Conceptually:

```text
beforeAll
   ↓
Test 1
Test 2
Test 3
```

---

# 8. afterAll()

`afterAll()` runs once after all tests in the suite.

```javascript
test.describe('User Tests', () => {

  test.afterAll(async () => {
    console.log('User Test Suite completed');
  });

  test('Create user', async ({ page }) => {
    // test
  });

  test('Update user', async ({ page }) => {
    // test
  });

});
```

---

# 9. Complete Suite with Hooks

```javascript
import { test, expect } from '@playwright/test';

test.describe('Login Test Suite', () => {

  test.beforeAll(async () => {
    console.log('Starting Login Suite');
  });

  test.beforeEach(async ({ page }) => {
    await page.goto('https://example.com/login');
  });

  test('Valid login', async ({ page }) => {
    await page.getByLabel('Username').fill('admin');
    await page.getByLabel('Password').fill('password');

    await page.getByRole('button', { name: 'Login' }).click();

    await expect(page).toHaveURL(/dashboard/);
  });

  test('Invalid login', async ({ page }) => {
    await page.getByLabel('Username').fill('invalid');
    await page.getByLabel('Password').fill('invalid');

    await page.getByRole('button', { name: 'Login' }).click();

    await expect(page.getByText('Invalid credentials')).toBeVisible();
  });

  test.afterEach(async () => {
    console.log('Test completed');
  });

  test.afterAll(async () => {
    console.log('Login Suite completed');
  });

});
```

---

# 10. Test Suite Naming

Use meaningful suite names.

Good:

```javascript
test.describe('Customer Login', () => {});
```

```javascript
test.describe('Shopping Cart', () => {});
```

```javascript
test.describe('Payment Processing', () => {});
```

Avoid:

```javascript
test.describe('Tests', () => {});
```

```javascript
test.describe('Suite1', () => {});
```

---

# 11. Organizing Tests by Feature

A common enterprise structure is:

```text
tests/
├── authentication/
│   ├── login.spec.js
│   ├── logout.spec.js
│   └── registration.spec.js
│
├── customer/
│   ├── profile.spec.js
│   └── address.spec.js
│
├── shopping/
│   ├── cart.spec.js
│   └── checkout.spec.js
│
└── payment/
    ├── payment.spec.js
    └── refund.spec.js
```

This structure works well for large automation frameworks.

---

# 12. Organizing Tests by Application Module

Example:

```text
tests/
├── Login/
├── Dashboard/
├── Customer/
├── Orders/
├── Payments/
├── Reports/
└── Administration/
```

Each folder can contain multiple test specifications.

---

# 13. Organizing Tests by Test Type

Another approach is:

```text
tests/
├── smoke/
├── regression/
├── integration/
├── api/
├── ui/
└── e2e/
```

However, in large projects, organizing primarily by **business feature** is often easier to maintain.

---

# 14. Running a Complete Test Suite

Run all Playwright tests:

```bash
npx playwright test
```

---

# 15. Running a Specific Test File

```bash
npx playwright test tests/login.spec.js
```

---

# 16. Running Multiple Test Files

```bash
npx playwright test tests/login.spec.js tests/checkout.spec.js
```

---

# 17. Running Tests from a Directory

```bash
npx playwright test tests/authentication/
```

This executes tests within the specified directory.

---

# 18. Running Tests by Suite Name

Suppose:

```javascript
test.describe('Login Test Suite', () => {

  test('Valid login', async ({ page }) => {
    // test
  });

  test('Invalid login', async ({ page }) => {
    // test
  });

});
```

Run using the `--grep` option:

```bash
npx playwright test --grep "Login Test Suite"
```

---

# 19. Running Tests by Test Name

```bash
npx playwright test --grep "Valid login"
```

This allows you to execute tests matching a specific title.

---

# 20. Excluding Tests with grep-invert

Use `--grep-invert` to exclude matching tests.

```bash
npx playwright test --grep-invert "Login"
```

This runs tests that do not match `Login`.

---

# 21. Test Suite with Tags

Tags are useful for categorizing tests.

```javascript
import { test } from '@playwright/test';

test('Login test @smoke', async ({ page }) => {
  // test
});

test('Checkout test @regression', async ({ page }) => {
  // test
});
```

Run smoke tests:

```bash
npx playwright test --grep @smoke
```

Run regression tests:

```bash
npx playwright test --grep @regression
```

---

# 22. Combining Suite Names and Tags

```javascript
test.describe('Authentication', () => {

  test('Valid login @smoke', async ({ page }) => {
    // test
  });

  test('Password validation @regression', async ({ page }) => {
    // test
  });

});
```

Run:

```bash
npx playwright test --grep @smoke
```

---

# 23. Serial Test Suites

By default, Playwright tests are designed to be independent.

If tests genuinely need to run sequentially, use serial mode.

```javascript
test.describe.configure({ mode: 'serial' });

test.describe('Sequential Tests', () => {

  test('Create customer', async ({ page }) => {
    // test
  });

  test('Update customer', async ({ page }) => {
    // depends on previous test
  });

  test('Delete customer', async ({ page }) => {
    // depends on previous test
  });

});
```

Execution:

```text
Create customer
      ↓
Update customer
      ↓
Delete customer
```

Use serial execution carefully.

---

# 24. Parallel Test Suites

Playwright supports parallel execution.

```javascript
test.describe.configure({ mode: 'parallel' });

test.describe('Parallel Tests', () => {

  test('Test A', async ({ page }) => {
    // test
  });

  test('Test B', async ({ page }) => {
    // test
  });

  test('Test C', async ({ page }) => {
    // test
  });

});
```

Conceptually:

```text
Worker 1 → Test A
Worker 2 → Test B
Worker 3 → Test C
```

Tests should be independent when using parallel execution.

---

# 25. Fully Parallel Test Execution

Playwright configuration can enable full parallel execution.

```javascript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  fullyParallel: true,
});
```

This is useful for reducing execution time in large test suites.

---

# 26. Test Suite with Project Configuration

Playwright projects allow the same suite to run against different browsers or environments.

```javascript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  projects: [
    {
      name: 'chromium',
      use: {
        browserName: 'chromium'
      }
    },
    {
      name: 'firefox',
      use: {
        browserName: 'firefox'
      }
    },
    {
      name: 'webkit',
      use: {
        browserName: 'webkit'
      }
    }
  ]
});
```

Run a specific project:

```bash
npx playwright test --project=chromium
```

---

# 27. Suite-Level Annotations

You can configure tests within a suite.

```javascript
test.describe('Checkout Tests', () => {

  test.describe.configure({
    retries: 2
  });

  test('Checkout test', async ({ page }) => {
    // test
  });

});
```

This allows suite-level configuration.

---

# 28. Skip a Test

```javascript
test.skip('Feature not implemented', async ({ page }) => {
  // test
});
```

---

# 29. Skip an Entire Suite

```javascript
test.describe.skip('Disabled Suite', () => {

  test('Test 1', async ({ page }) => {
    // test
  });

  test('Test 2', async ({ page }) => {
    // test
  });

});
```

---

# 30. Run Only a Specific Suite

Use `describe.only()`:

```javascript
test.describe.only('Login Suite', () => {

  test('Valid login', async ({ page }) => {
    // test
  });

  test('Invalid login', async ({ page }) => {
    // test
  });

});
```

This is useful during local debugging.

Avoid committing `only()` to source control.

---

# 31. Test Hooks and Scope

Hooks can be defined at different levels.

```javascript
test.beforeEach(async ({ page }) => {
  // Applies to tests in this file
});

test.describe('Login', () => {

  test.beforeEach(async ({ page }) => {
    // Applies to tests inside Login suite
  });

  test('Login test', async ({ page }) => {
    // test
  });

});
```

The inner hook applies specifically to the nested suite.

---

# 32. Multiple Suites in One File

```javascript
import { test } from '@playwright/test';

test.describe('Login', () => {

  test('Valid login', async ({ page }) => {
    // test
  });

});

test.describe('Registration', () => {

  test('Create account', async ({ page }) => {
    // test
  });

});
```

This is valid, but separate files are usually preferable when the suites become large.

---

# 33. Example Enterprise Test Suite

```javascript
import { test, expect } from '@playwright/test';

test.describe('Customer Management', () => {

  test.beforeEach(async ({ page }) => {
    await page.goto('https://example.com');
  });

  test.describe('Customer Creation', () => {

    test('Create customer with valid data @smoke', async ({ page }) => {

      await page.getByRole('link', { name: 'Customers' }).click();

      await page.getByRole('button', { name: 'Add Customer' }).click();

      await page.getByLabel('First Name').fill('John');
      await page.getByLabel('Last Name').fill('Smith');
      await page.getByLabel('Email').fill('john@example.com');

      await page.getByRole('button', { name: 'Save' }).click();

      await expect(
        page.getByText('Customer created successfully')
      ).toBeVisible();
    });

  });

  test.describe('Customer Search', () => {

    test('Search customer @regression', async ({ page }) => {

      await page.getByRole('link', { name: 'Customers' }).click();

      await page.getByPlaceholder('Search customer').fill('John');

      await page.getByRole('button', { name: 'Search' }).click();

      await expect(page.getByText('John')).toBeVisible();
    });

  });

});
```

---

# 34. Recommended Enterprise Structure

For a large Playwright framework:

```text
playwright-project/
│
├── tests/
│   ├── authentication/
│   │   ├── login.spec.js
│   │   └── logout.spec.js
│   │
│   ├── customer/
│   │   ├── customer-create.spec.js
│   │   ├── customer-search.spec.js
│   │   └── customer-update.spec.js
│   │
│   ├── orders/
│   │   ├── order-create.spec.js
│   │   └── order-cancel.spec.js
│   │
│   └── payments/
│       ├── payment.spec.js
│       └── refund.spec.js
│
├── pages/
│   ├── LoginPage.js
│   ├── CustomerPage.js
│   └── CheckoutPage.js
│
├── fixtures/
├── utils/
├── test-data/
├── playwright.config.js
├── package.json
└── README.md
```

---

# 35. Test Suite Best Practices

### 1. Keep tests independent

Avoid:

```text
Test 1 creates customer
       ↓
Test 2 updates customer created by Test 1
```

Prefer:

```text
Test 1 → independent customer
Test 2 → independent customer
```

---

### 2. Use meaningful names

Good:

```javascript
test('Verify user can reset password with valid email', async ({ page }) => {
});
```

Poor:

```javascript
test('Test 1', async ({ page }) => {
});
```

---

### 3. Group related functionality

```javascript
test.describe('Authentication', () => {
});
```

---

### 4. Avoid oversized test files

Instead of:

```text
application.spec.js
```

with hundreds of tests, split by feature:

```text
login.spec.js
customer.spec.js
orders.spec.js
payments.spec.js
```

---

### 5. Use tags for execution categories

```text
@smoke
@regression
@sanity
@api
@e2e
```

Example:

```javascript
test('Login works @smoke', async ({ page }) => {
});
```

---

### 6. Avoid unnecessary serial execution

Do not use:

```javascript
test.describe.configure({ mode: 'serial' });
```

unless tests truly depend on each other.

Independent tests are easier to parallelize.

---

### 7. Keep test data manageable

Separate test data from test logic when possible.

```text
test-data/
├── users.json
├── customers.json
└── products.json
```

---

### 8. Use Page Object Model for complex applications

Test suites should focus on business behavior rather than low-level locator details.

```javascript
test('Create customer', async ({ customerPage }) => {

  await customerPage.createCustomer({
    firstName: 'John',
    lastName: 'Smith'
  });

});
```

---

# 36. Test Suite Execution Strategy

A typical CI/CD strategy can be:

```text
Pull Request
     ↓
Smoke Tests
     ↓
Build
     ↓
Regression Tests
     ↓
Cross-Browser Tests
     ↓
Reports
```

Example commands:

```bash
npx playwright test --grep @smoke
```

```bash
npx playwright test --grep @regression
```

```bash
npx playwright test --project=chromium
```

---

# 37. Interview Questions

### Q1. What is a test suite in Playwright?

A test suite is a logical collection of related tests, commonly created using `test.describe()`.

---

### Q2. How do you group tests in Playwright?

Using:

```javascript
test.describe('Suite Name', () => {
});
```

---

### Q3. Can Playwright have nested test suites?

Yes.

```javascript
test.describe('Application', () => {

  test.describe('Login', () => {
  });

});
```

---

### Q4. How do you run only one suite?

Use:

```bash
npx playwright test --grep "Suite Name"
```

or temporarily use:

```javascript
test.describe.only('Suite Name', () => {
});
```

---

### Q5. How do you skip a suite?

```javascript
test.describe.skip('Suite Name', () => {
});
```

---

### Q6. What is the difference between beforeAll and beforeEach?

`beforeAll()` runs once for the suite.

`beforeEach()` runs before every test.

---

### Q7. How do you run tests serially?

```javascript
test.describe.configure({ mode: 'serial' });
```

---

### Q8. How do you run tests in parallel?

Use Playwright's parallel execution capabilities, for example:

```javascript
test.describe.configure({ mode: 'parallel' });
```

or configure:

```javascript
fullyParallel: true
```

---

### Q9. How do you categorize Playwright tests?

Tags can be used:

```javascript
test('Login test @smoke', async ({ page }) => {
});
```

Then:

```bash
npx playwright test --grep @smoke
```

---

### Q10. What is the recommended way to organize large Playwright suites?

Organize tests primarily by **business feature/module**, keep tests independent, use reusable fixtures/page objects, and use tags for execution categories.

---

# 38. Quick Reference

| Requirement       | Playwright                                      |
| ----------------- | ----------------------------------------------- |
| Create suite      | `test.describe()`                               |
| Nested suite      | Nested `test.describe()`                        |
| Before every test | `test.beforeEach()`                             |
| After every test  | `test.afterEach()`                              |
| Before suite      | `test.beforeAll()`                              |
| After suite       | `test.afterAll()`                               |
| Skip test         | `test.skip()`                                   |
| Skip suite        | `test.describe.skip()`                          |
| Run only test     | `test.only()`                                   |
| Run only suite    | `test.describe.only()`                          |
| Serial suite      | `test.describe.configure({ mode: 'serial' })`   |
| Parallel suite    | `test.describe.configure({ mode: 'parallel' })` |
| Tag tests         | `@smoke`, `@regression`                         |
| Run by tag        | `--grep @smoke`                                 |
| Exclude tests     | `--grep-invert`                                 |
| Run all tests     | `npx playwright test`                           |
| Run project       | `--project=chromium`                            |

---

# 39. Recommended Playwright Test Suite Pattern

For a senior-level automation framework, a good pattern is:

```text
tests/
│
├── authentication/
│   ├── login.spec.js
│   └── logout.spec.js
│
├── customer/
│   ├── create-customer.spec.js
│   ├── search-customer.spec.js
│   └── update-customer.spec.js
│
├── orders/
│   ├── create-order.spec.js
│   └── cancel-order.spec.js
│
└── payments/
    ├── payment.spec.js
    └── refund.spec.js
```

Inside each file:

```javascript
test.describe('Feature Name', () => {

  test.beforeEach(async ({ page }) => {
    // common setup
  });

  test('Business scenario 1 @smoke', async ({ page }) => {
    // test
  });

  test('Business scenario 2 @regression', async ({ page }) => {
    // test
  });

});
```

This structure provides **clear organization, maintainability, parallel execution, CI/CD flexibility, and easy test filtering**.

---

## Summary

Playwright Test Suites are used to organize automated tests into logical groups.

The most important concepts are:

```text
test.describe()
      ↓
Group related tests
      ↓
beforeAll / afterAll
      ↓
beforeEach / afterEach
      ↓
Individual tests
      ↓
Tags / grep
      ↓
Parallel execution
      ↓
CI/CD execution
```

For enterprise automation, prefer:

```text
Feature-based folders
        +
Meaningful test names
        +
Independent tests
        +
Reusable fixtures
        +
Page Object Model
        +
Tags
        +
Parallel execution
```

This provides a scalable Playwright test-suite architecture suitable for large QA automation projects.
