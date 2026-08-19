# Playwright Test Data

## Overview

Test data is the information used by automated tests to validate application behavior.

Examples:

* Usernames and passwords
* Customer information
* VINs
* Product details
* Search values
* API request payloads
* Expected results
* Environment-specific URLs
* Configuration values

Good test-data management makes Playwright tests:

* Reusable
* Maintainable
* Readable
* Data-driven
* Easy to execute across environments
* Less dependent on hardcoded values

---

# 1. Hardcoded Test Data

The simplest approach is to keep test data directly inside the test.

```javascript
import { test, expect } from '@playwright/test';

test('Login test', async ({ page }) => {

    await page.goto('https://example.com/login');

    await page.locator('#username').fill('testuser');
    await page.locator('#password').fill('Password123');

    await page.locator('#login').click();

    await expect(page).toHaveTitle(/Dashboard/);
});
```

### Problem

Hardcoded values become difficult to maintain when:

* Multiple tests use the same data
* Credentials change
* Different environments require different data
* Multiple datasets are required

For larger frameworks, externalizing test data is preferred.

---

# 2. Test Data in a JavaScript Object

Test data can be stored in a JavaScript object.

```javascript
const loginData = {
    username: 'testuser',
    password: 'Password123'
};

test('Login test', async ({ page }) => {

    await page.goto('https://example.com/login');

    await page.locator('#username').fill(loginData.username);
    await page.locator('#password').fill(loginData.password);

    await page.locator('#login').click();
});
```

This is better than repeating the same values throughout the test.

---

# 3. Separate Test Data File

Create a separate file:

```text
TestData/
    loginData.js
```

### loginData.js

```javascript
const loginData = {
    validUser: {
        username: 'testuser',
        password: 'Password123'
    },

    invalidUser: {
        username: 'invaliduser',
        password: 'WrongPassword'
    }
};

module.exports = loginData;
```

Use it in the test:

```javascript
const { test, expect } = require('@playwright/test');
const loginData = require('../TestData/loginData');

test('Valid login', async ({ page }) => {

    await page.goto('https://example.com/login');

    await page.locator('#username').fill(loginData.validUser.username);
    await page.locator('#password').fill(loginData.validUser.password);

    await page.locator('#login').click();

    await expect(page).toHaveTitle(/Dashboard/);
});
```

---

# 4. Using ES Modules

If the project uses ES modules, use `export` and `import`.

### loginData.js

```javascript
export const loginData = {
    validUser: {
        username: 'testuser',
        password: 'Password123'
    },

    invalidUser: {
        username: 'invaliduser',
        password: 'WrongPassword'
    }
};
```

### Test

```javascript
import { test, expect } from '@playwright/test';
import { loginData } from '../TestData/loginData';

test('Valid login', async ({ page }) => {

    await page.goto('https://example.com/login');

    await page.locator('#username').fill(loginData.validUser.username);
    await page.locator('#password').fill(loginData.validUser.password);

    await page.locator('#login').click();
});
```

---

# 5. JSON Test Data

JSON is commonly used for test data because it separates data from test logic.

Example:

```text
TestData/
    loginData.json
```

### loginData.json

```json
{
    "validUser": {
        "username": "testuser",
        "password": "Password123"
    },
    "invalidUser": {
        "username": "invaliduser",
        "password": "WrongPassword"
    }
}
```

---

# 6. Reading JSON Data

### CommonJS

```javascript
const loginData = require('../TestData/loginData.json');
```

Example:

```javascript
const { test } = require('@playwright/test');
const loginData = require('../TestData/loginData.json');

test('Login using JSON data', async ({ page }) => {

    await page.goto('https://example.com/login');

    await page.locator('#username').fill(loginData.validUser.username);
    await page.locator('#password').fill(loginData.validUser.password);

    await page.locator('#login').click();
});
```

---

# 7. Multiple Test Data Sets

JSON can contain multiple datasets.

```json
[
    {
        "username": "user1",
        "password": "Password1",
        "expected": "success"
    },
    {
        "username": "user2",
        "password": "Password2",
        "expected": "success"
    },
    {
        "username": "invalid",
        "password": "WrongPassword",
        "expected": "failure"
    }
]
```

---

# 8. Data-Driven Testing

Playwright tests can iterate through multiple datasets.

```javascript
import { test, expect } from '@playwright/test';

const loginData = [
    {
        username: 'user1',
        password: 'Password1'
    },
    {
        username: 'user2',
        password: 'Password2'
    },
    {
        username: 'user3',
        password: 'Password3'
    }
];

for (const data of loginData) {

    test(`Login test - ${data.username}`, async ({ page }) => {

        await page.goto('https://example.com/login');

        await page.locator('#username').fill(data.username);
        await page.locator('#password').fill(data.password);

        await page.locator('#login').click();

        await expect(page).toHaveTitle(/Dashboard/);
    });
}
```

Each dataset creates a separate test.

---

# 9. Using Test Titles with Test Data

Including test-data information in the test title makes reports easier to understand.

```javascript
const users = [
    {
        username: 'admin',
        role: 'Administrator'
    },
    {
        username: 'manager',
        role: 'Manager'
    },
    {
        username: 'user',
        role: 'Standard User'
    }
];

for (const user of users) {

    test(`Verify login for ${user.role}`, async ({ page }) => {

        console.log(`Testing user: ${user.username}`);

        await page.goto('https://example.com/login');

        // Login steps
    });
}
```

---

# 10. Parameterized Tests

Parameterization allows the same test logic to execute with different values.

```javascript
const searchData = [
    'Toyota',
    'Lexus',
    'Honda'
];

for (const searchText of searchData) {

    test(`Search for ${searchText}`, async ({ page }) => {

        await page.goto('https://example.com');

        await page.locator('#search').fill(searchText);
        await page.locator('#searchButton').click();

        await expect(page.locator('.results')).toBeVisible();
    });
}
```

---

# 11. Test Data with Objects

Objects are useful when a test requires multiple related values.

```javascript
const customer = {
    firstName: 'John',
    lastName: 'Smith',
    email: 'john.smith@example.com',
    phone: '1234567890',
    city: 'Dallas'
};
```

Use the data:

```javascript
await page.locator('#firstName').fill(customer.firstName);
await page.locator('#lastName').fill(customer.lastName);
await page.locator('#email').fill(customer.email);
await page.locator('#phone').fill(customer.phone);
await page.locator('#city').fill(customer.city);
```

---

# 12. Environment Variables

Environment variables are useful for values that change between environments.

Examples:

```text
BASE_URL
USERNAME
PASSWORD
API_URL
ENVIRONMENT
```

Instead of hardcoding:

```javascript
await page.goto('https://stage.example.com');
```

Use:

```javascript
await page.goto(process.env.BASE_URL);
```

---

# 13. `.env` File

A `.env` file can contain environment-specific values.

```text
BASE_URL=https://stage.example.com
USERNAME=testuser
PASSWORD=Password123
```

Install the `dotenv` package if needed:

```bash
npm install dotenv
```

Load the variables:

```javascript
import 'dotenv/config';
```

Then:

```javascript
console.log(process.env.BASE_URL);
console.log(process.env.USERNAME);
```

---

# 14. Using Environment Variables in Playwright

Example:

```javascript
import 'dotenv/config';
import { test } from '@playwright/test';

test('Login using environment variables', async ({ page }) => {

    await page.goto(process.env.BASE_URL);

    await page.locator('#username').fill(process.env.USERNAME);
    await page.locator('#password').fill(process.env.PASSWORD);

    await page.locator('#login').click();
});
```

---

# 15. Never Commit Sensitive Data

Do not commit passwords, tokens, API keys, or other secrets into Git.

Bad:

```javascript
const password = 'MyRealPassword123';
```

Better:

```javascript
const password = process.env.PASSWORD;
```

Add `.env` to `.gitignore`:

```text
.env
```

---

# 16. Playwright Configuration with Base URL

Instead of storing the base URL in every test, configure it in `playwright.config.js`.

```javascript
import { defineConfig } from '@playwright/test';

export default defineConfig({
    use: {
        baseURL: 'https://example.com'
    }
});
```

Then the test can use:

```javascript
await page.goto('/login');
```

instead of:

```javascript
await page.goto('https://example.com/login');
```

This makes tests easier to maintain.

---

# 17. Environment-Specific Configuration

Different environments may use different URLs.

```javascript
import { defineConfig } from '@playwright/test';

const environment = process.env.TEST_ENV || 'stage';

const urls = {
    dev: 'https://dev.example.com',
    stage: 'https://stage.example.com',
    prod: 'https://prod.example.com'
};

export default defineConfig({
    use: {
        baseURL: urls[environment]
    }
});
```

Run:

```bash
$env:TEST_ENV="dev"
npx playwright test
```

For stage:

```bash
$env:TEST_ENV="stage"
npx playwright test
```

For production:

```bash
$env:TEST_ENV="prod"
npx playwright test
```

---

# 18. Test Data with Fixtures

Fixtures can provide reusable test data.

```javascript
import { test as base } from '@playwright/test';

export const test = base.extend({
    customer: async ({}, use) => {

        const customer = {
            firstName: 'John',
            lastName: 'Smith',
            email: 'john.smith@example.com'
        };

        await use(customer);
    }
});
```

Use the fixture:

```javascript
import { test } from './fixtures';

test('Create customer', async ({ page, customer }) => {

    await page.goto('/customer');

    await page.locator('#firstName').fill(customer.firstName);
    await page.locator('#lastName').fill(customer.lastName);
    await page.locator('#email').fill(customer.email);
});
```

Fixtures are especially useful when the same data is required by many tests.

---

# 19. Dynamic Test Data

Some test data should be generated dynamically.

Example:

```javascript
function generateEmail() {
    return `user_${Date.now()}@example.com`;
}

test('Create user', async ({ page }) => {

    const email = generateEmail();

    await page.goto('/register');

    await page.locator('#email').fill(email);
});
```

Every test execution gets a unique email.

---

# 20. Dynamic Customer Data

```javascript
function generateCustomer() {

    const timestamp = Date.now();

    return {
        firstName: `John${timestamp}`,
        lastName: 'Smith',
        email: `john${timestamp}@example.com`
    };
}
```

Use:

```javascript
const customer = generateCustomer();

console.log(customer.email);
```

---

# 21. Date Test Data

JavaScript can generate dates dynamically.

```javascript
const today = new Date();

console.log(today);
```

Formatted date:

```javascript
const today = new Date()
    .toISOString()
    .split('T')[0];

console.log(today);
```

Example output:

```text
2026-08-19
```

---

# 22. Test Data Utility

Create a utility:

```text
utils/
    testDataUtils.js
```

Example:

```javascript
export function generateEmail() {

    return `user_${Date.now()}@example.com`;
}

export function generateCustomer() {

    const timestamp = Date.now();

    return {
        firstName: `John${timestamp}`,
        lastName: 'Smith',
        email: `john${timestamp}@example.com'
    };
}
```

Import it:

```javascript
import {
    generateEmail,
    generateCustomer
} from '../utils/testDataUtils';
```

---

# 23. API Test Data

Playwright can also be used for API testing.

Example payload:

```javascript
const userData = {
    name: 'John Smith',
    email: 'john@example.com',
    role: 'user'
};
```

Use it with `request`:

```javascript
import { test, expect } from '@playwright/test';

test('Create user through API', async ({ request }) => {

    const userData = {
        name: 'John Smith',
        email: 'john@example.com',
        role: 'user'
    };

    const response = await request.post('/users', {
        data: userData
    });

    expect(response.ok()).toBeTruthy();
});
```

---

# 24. API Response as Test Data

Test data can also come from an API.

```javascript
const response = await request.get('/users');

const users = await response.json();

console.log(users);
```

A value from the response can then be used in another request or UI test.

This is useful for API chaining and end-to-end testing.

---

# 25. Test Data Flow

A common automation flow is:

```text
Create Test Data
       ↓
Create API Data
       ↓
Retrieve Generated ID
       ↓
Open UI
       ↓
Search Using ID
       ↓
Validate UI
       ↓
Clean Up Test Data
```

This approach reduces dependency on static data.

---

# 26. Test Data Cleanup

Tests should clean up data when appropriate.

Example:

```javascript
test.afterEach(async ({ request }) => {

    await request.delete(`/users/${userId}`);
});
```

Cleanup is especially important when tests create:

* Customers
* Vehicles
* Appointments
* Orders
* Accounts
* Database records

---

# 27. Before and After Hooks with Test Data

```javascript
let userId;

test.beforeEach(async ({ request }) => {

    const response = await request.post('/users', {
        data: {
            name: 'Test User',
            email: `user_${Date.now()}@example.com`
        }
    });

    const data = await response.json();

    userId = data.id;
});

test.afterEach(async ({ request }) => {

    if (userId) {
        await request.delete(`/users/${userId}`);
    }
});
```

---

# 28. Recommended Folder Structure

A scalable Playwright project can use:

```text
PlaywrightProject/
│
├── tests/
│   ├── login/
│   │   └── Login.spec.js
│   │
│   └── customer/
│       └── Customer.spec.js
│
├── TestData/
│   ├── loginData.json
│   ├── customerData.json
│   └── searchData.json
│
├── fixtures/
│   └── testFixtures.js
│
├── utils/
│   └── testDataUtils.js
│
├── pages/
│   ├── LoginPage.js
│   └── CustomerPage.js
│
├── playwright.config.js
├── package.json
├── .env
└── .gitignore
```

---

# 29. Static vs Dynamic Test Data

| Type                 | Example                  | Best Use                  |
| -------------------- | ------------------------ | ------------------------- |
| Hardcoded            | `admin`                  | Simple tests              |
| JavaScript object    | `{ username, password }` | Small projects            |
| JSON                 | `loginData.json`         | Data-driven testing       |
| Environment variable | `process.env.BASE_URL`   | Environment configuration |
| Fixture              | Reusable customer        | Shared test data          |
| Dynamic              | `Date.now()`             | Unique records            |
| API-generated        | API response             | End-to-end workflows      |
| Database-generated   | DB record                | Complex integration tests |

---

# 30. Recommended Approach

For a professional Playwright framework:

```text
                    Test Data
                       │
        ┌──────────────┼──────────────┐
        │              │              │
       JSON          .env          Fixtures
        │              │              │
 Static datasets   Environment    Reusable data
        │              │              │
        └──────────────┼──────────────┘
                       │
                 Test Data Utils
                       │
                       ▼
                    Tests
```

Use:

* JSON for static datasets
* `.env` for environment-specific values
* Fixtures for reusable test data
* Utilities for dynamic data generation
* API/database setup for complex test data
* Cleanup after tests where necessary

---

# 31. Best Practices

### 1. Avoid unnecessary hardcoding

Prefer:

```javascript
const user = testData.validUser;
```

over repeatedly writing:

```javascript
await page.locator('#username').fill('testuser');
```

### 2. Keep test logic separate from test data

Test:

```javascript
await loginPage.login(user.username, user.password);
```

Data:

```javascript
{
    "username": "testuser",
    "password": "Password123"
}
```

### 3. Do not store secrets in source control

Use:

```javascript
process.env.PASSWORD
```

instead of committing real credentials.

### 4. Generate unique data when required

```javascript
const email = `user_${Date.now()}@example.com`;
```

### 5. Reuse common test data

Use fixtures or utilities instead of duplicating data.

### 6. Keep datasets readable

Avoid extremely large JSON files containing unrelated data.

### 7. Clean up generated data

Delete records created during testing whenever possible.

### 8. Make environment configuration independent

Tests should not need code changes when switching between:

```text
DEV
STAGE
QA
PROD
```

---

# 32. Interview Questions

## Q1. How do you manage test data in Playwright?

Use a combination of:

* JavaScript/TypeScript objects
* JSON files
* Environment variables
* Fixtures
* Utility functions
* API/database setup

---

## Q2. How do you perform data-driven testing in Playwright?

Store multiple datasets and iterate through them.

```javascript
for (const data of testData) {

    test(`Test ${data.username}`, async ({ page }) => {

        // Test logic
    });
}
```

---

## Q3. How do you manage passwords?

Use environment variables or a secret-management solution.

```javascript
process.env.PASSWORD
```

Never commit real passwords to Git.

---

## Q4. How do you generate unique test data?

Use a timestamp, UUID, or another unique-data generator.

```javascript
const email = `user_${Date.now()}@example.com`;
```

---

## Q5. Why use fixtures for test data?

Fixtures provide reusable, centralized test data and setup logic.

They reduce duplication and make tests easier to maintain.

---

## Q6. What is the difference between test data and configuration?

**Test data** is information used by the test.

Example:

```text
username
password
customerName
VIN
```

**Configuration** controls how and where the test runs.

Example:

```text
baseURL
browser
timeout
environment
```

---

## Q7. Should credentials be stored in JSON?

Generally, no.

Static non-sensitive data can be stored in JSON, but credentials and secrets should be supplied through environment variables or a secure secret-management system.

---

# 33. Complete Example

### Test Data

```javascript
const users = [
    {
        username: 'admin',
        password: 'Admin123',
        role: 'Administrator'
    },
    {
        username: 'manager',
        password: 'Manager123',
        role: 'Manager'
    }
];

module.exports = users;
```

### Test

```javascript
const { test, expect } = require('@playwright/test');
const users = require('../TestData/users');

for (const user of users) {

    test(`Login as ${user.role}`, async ({ page }) => {

        await page.goto('/login');

        await page.locator('#username').fill(user.username);
        await page.locator('#password').fill(user.password);

        await page.locator('#login').click();

        await expect(page.locator('#dashboard')).toBeVisible();
    });
}
```

This provides a clean separation:

```text
Test Data
    ↓
Test Logic
    ↓
Page Objects
    ↓
Playwright
    ↓
Application
```

---

# 34. Final Recommendation

For a senior-level Playwright framework, use the following approach:

```text
                    Playwright Tests
                           │
            ┌──────────────┼──────────────┐
            │              │              │
        Test Data       Fixtures       Utilities
            │              │              │
        JSON / TS      Reusable Data    Dynamic Data
            │              │              │
            └──────────────┼──────────────┘
                           │
                    Environment
                           │
                         .env
                           │
                    playwright.config
```

### Key principles

```text
Separate data from test logic
Avoid hardcoded secrets
Use JSON for static datasets
Use environment variables for configuration/secrets
Use fixtures for reusable data
Generate unique data when necessary
Use APIs/databases for complex setup
Clean up test-generated data
Keep the framework scalable and maintainable
```

This structure works well with **Playwright + JavaScript/TypeScript + Page Object Model + Fixtures + API testing + CI/CD**.
