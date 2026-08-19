This is the natural next topic after Basics → Installation → Browsers → Actions → Assertions.
# Playwright Fixtures

## 1. What are Fixtures?

Fixtures in Playwright provide a way to set up and tear down the environment required by tests.

They help us:

- Create reusable test setup
- Create reusable test teardown
- Share objects between tests
- Configure browsers and pages
- Create custom test dependencies
- Reduce duplicate code
- Build scalable automation frameworks

Playwright provides several built-in fixtures such as:

- `browser`
- `context`
- `page`
- `request`

---

# 2. Basic Fixture Example

The `page` fixture is one of the most commonly used fixtures.

```javascript
import { test, expect } from '@playwright/test';

test('Verify page title', async ({ page }) => {

    await page.goto('https://www.google.com');

    await expect(page).toHaveTitle(/Google/);

});

Here:

async ({ page })

means Playwright automatically provides the page fixture.

We do not need to manually create the browser or page.

3. Built-in Playwright Fixtures
Common Fixtures
Fixture	Purpose
browser	Browser instance
context	Browser context
page	Page/tab
request	API request context
browserName	Browser name
isMobile	Mobile configuration
baseURL	Configured base URL
4. Page Fixture

The page fixture represents a browser tab.

import { test } from '@playwright/test';


test('Page fixture example', async ({ page }) => {


    await page.goto('https://example.com');


    console.log(await page.title());


});

Playwright automatically:

Creates the browser
Creates a browser context
Creates a page
Executes the test
Cleans up the resources
5. Browser Fixture

The browser fixture represents the browser instance.

import { test } from '@playwright/test';


test('Browser fixture example', async ({ browser }) => {


    const context = await browser.newContext();


    const page = await context.newPage();


    await page.goto('https://example.com');


    await context.close();


});

Normally, you should prefer the page fixture unless you specifically need custom contexts.

6. Context Fixture

A browser context provides an isolated browser session.

import { test } from '@playwright/test';


test('Context fixture example', async ({ context }) => {


    const page = await context.newPage();


    await page.goto('https://example.com');


});

Each test can have its own isolated context.

This prevents:

Cookies leaking between tests
Local storage leaking between tests
Session data leaking between tests
7. Multiple Fixtures

A test can use multiple fixtures.

import { test, expect } from '@playwright/test';


test('Multiple fixtures', async ({ browser, context, page }) => {


    console.log(browser);
    console.log(context);
    console.log(page);


    await page.goto('https://example.com');


});

However, usually you only need:

async ({ page })
8. Fixture Lifecycle

A simplified Playwright fixture lifecycle looks like:

Test starts
    |
    v
Fixture setup
    |
    v
Test execution
    |
    v
Fixture teardown
    |
    v
Test ends

Playwright manages the lifecycle automatically for built-in fixtures.

9. Custom Fixtures

Custom fixtures allow us to create reusable test setup.

Example:

import { test as base } from '@playwright/test';


export const test = base.extend({


    loginPage: async ({ page }, use) => {


        await page.goto('https://example.com/login');


        await use(page);


    }


});

Now we can use the custom fixture:

import { test } from './fixtures';


test('Login test', async ({ loginPage }) => {


    console.log('Login page is ready');


});
10. Understanding use

The use function separates fixture setup from test execution.

Example:

loginPage: async ({ page }, use) => {


    await page.goto('https://example.com/login');


    await use(page);


}

The sequence is:

Create page
   |
   v
Navigate to login
   |
   v
use(page)
   |
   v
Test executes
   |
   v
Fixture teardown
11. Custom Login Fixture

A common real-world example is creating a reusable login fixture.

fixtures.js
import { test as base } from '@playwright/test';


export const test = base.extend({


    authenticatedPage: async ({ page }, use) => {


        await page.goto('https://example.com/login');


        await page.locator('#username').fill('admin');


        await page.locator('#password').fill('password');


        await page.locator('#login').click();


        await use(page);


    }


});


export { expect } from '@playwright/test';

Test:

import { test, expect } from './fixtures';


test('Verify dashboard', async ({ authenticatedPage }) => {


    await expect(authenticatedPage).toHaveURL(/dashboard/);


});

Now every test using authenticatedPage gets the login setup automatically.

12. Fixture with Page Object

Fixtures work very well with Page Object Model.

LoginPage.js
export class LoginPage {


    constructor(page) {


        this.page = page;
        this.username = page.locator('#username');
        this.password = page.locator('#password');
        this.loginButton = page.locator('#login');


    }


    async login(username, password) {


        await this.username.fill(username);
        await this.password.fill(password);
        await this.loginButton.click();


    }


}
fixtures.js
import { test as base } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';


export const test = base.extend({


    loginPage: async ({ page }, use) => {


        const loginPage = new LoginPage(page);


        await use(loginPage);


    }


});


export { expect } from '@playwright/test';
Test
import { test, expect } from '../fixtures/fixtures';


test('Login test', async ({ page, loginPage }) => {


    await page.goto('https://example.com/login');


    await loginPage.login('admin', 'password');


    await expect(page).toHaveURL(/dashboard/);


});
13. Worker-Scoped Fixtures

Fixtures can have different scopes.

The two important concepts are:

Test scope
Worker scope

Example:

import { test as base } from '@playwright/test';


export const test = base.extend({


    workerData: [async ({}, use) => {


        console.log('Worker setup');


        await use('test data');


        console.log('Worker teardown');


    }, { scope: 'worker' }]


});

A worker-scoped fixture is created once per worker.

14. Test-Scoped Fixture

The default fixture scope is test scope.

import { test as base } from '@playwright/test';


export const test = base.extend({


    testData: async ({}, use) => {


        const data = {
            username: 'admin',
            password: 'password'
        };


        await use(data);


    }


});

The fixture is created for each test.

15. Test Scope vs Worker Scope
Scope	Lifecycle
Test	Created for each test
Worker	Created once for each worker
Test Scope
Test 1 → Fixture
Test 2 → Fixture
Test 3 → Fixture
Worker Scope
Worker
   |
   +-- Test 1
   +-- Test 2
   +-- Test 3

The worker fixture is shared by tests running in that worker.

16. Fixture Dependencies

Fixtures can depend on other fixtures.

import { test as base } from '@playwright/test';


export const test = base.extend({


    user: async ({}, use) => {


        const user = {
            username: 'admin',
            password: 'password'
        };


        await use(user);


    },


    loggedInPage: async ({ page, user }, use) => {


        await page.goto('https://example.com/login');


        await page.locator('#username').fill(user.username);


        await page.locator('#password').fill(user.password);


        await page.locator('#login').click();


        await use(page);


    }


});

Here:

user
  |
  v
loggedInPage
  |
  v
test
17. Fixture Teardown

Code after await use() is executed during teardown.

testData: async ({}, use) => {


    console.log('Setup');


    const data = {
        name: 'Selva'
    };


    await use(data);


    console.log('Teardown');


}

Execution:

Setup
   |
   v
Test
   |
   v
Teardown
18. Fixture for Test Data

Fixtures can provide reusable test data.

import { test as base } from '@playwright/test';


export const test = base.extend({


    testUser: async ({}, use) => {


        const user = {
            firstName: 'John',
            lastName: 'Doe',
            email: 'john@example.com'
        };


        await use(user);


    }


});

Test:

import { test } from './fixtures';


test('Create user', async ({ testUser }) => {


    console.log(testUser.firstName);
    console.log(testUser.email);


});
19. Fixture for API Testing

Playwright's request fixture can be used for API testing.

import { test, expect } from '@playwright/test';


test('GET API request', async ({ request }) => {


    const response = await request.get('https://api.example.com/users');


    expect(response.status()).toBe(200);


});
20. Fixture with API Login

A custom fixture can perform API authentication.

import { test as base } from '@playwright/test';


export const test = base.extend({


    authToken: async ({ request }, use) => {


        const response = await request.post(
            'https://api.example.com/login',
            {
                data: {
                    username: 'admin',
                    password: 'password'
                }
            }
        );


        const body = await response.json();


        await use(body.token);


    }


});

Test:

import { test } from './fixtures';


test('API test', async ({ authToken }) => {


    console.log(authToken);


});
21. Fixtures vs Before Hooks

Both fixtures and hooks can perform setup.

Before Hook
test.beforeEach(async ({ page }) => {


    await page.goto('https://example.com');


});
Fixture
const test = base.extend({


    homePage: async ({ page }, use) => {


        await page.goto('https://example.com');


        await use(page);


    }


});

Fixtures are generally better when the setup represents a reusable dependency that tests explicitly consume.

22. Fixtures vs Hooks
Feature	Fixtures	Hooks
Reusable dependency	Excellent	Limited
Setup	Yes	Yes
Teardown	Yes	Yes
Dependency injection	Yes	No
Page Object integration	Excellent	Good
Test data injection	Excellent	Limited
Framework scalability	Excellent	Good
23. Recommended Framework Structure

A scalable Playwright project can use:

playwright-project/
│
├── tests/
│   ├── login.spec.js
│   ├── home.spec.js
│   └── user.spec.js
│
├── pages/
│   ├── LoginPage.js
│   ├── HomePage.js
│   └── UserPage.js
│
├── fixtures/
│   └── fixtures.js
│
├── utils/
│   ├── testData.js
│   └── helpers.js
│
├── playwright.config.js
└── package.json
24. Complete Fixture Example
fixtures/fixtures.js
import { test as base, expect } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';


export const test = base.extend({


    loginPage: async ({ page }, use) => {


        const loginPage = new LoginPage(page);


        await use(loginPage);


    },


    testUser: async ({}, use) => {


        const user = {
            username: 'admin',
            password: 'password'
        };


        await use(user);


    }


});


export { expect };
pages/LoginPage.js
export class LoginPage {


    constructor(page) {


        this.page = page;


        this.username = page.locator('#username');
        this.password = page.locator('#password');
        this.loginButton = page.locator('#login');


    }


    async navigate() {


        await this.page.goto('https://example.com/login');


    }


    async login(username, password) {


        await this.username.fill(username);


        await this.password.fill(password);


        await this.loginButton.click();


    }


}
tests/login.spec.js
import { test, expect } from '../fixtures/fixtures';


test('Login with valid user', async ({ loginPage, testUser }) => {


    await loginPage.navigate();


    await loginPage.login(
        testUser.username,
        testUser.password
    );


    await expect(loginPage.page).toHaveURL(/dashboard/);


});
25. Important Interview Questions
Q1. What is a Playwright fixture?

A fixture provides reusable setup and teardown functionality to tests.

Q2. What is the most commonly used Playwright fixture?

The page fixture.

test('Example', async ({ page }) => {


});
Q3. What does use() do?

use() passes the prepared fixture value to the test.

await use(page);

Code before use() is generally setup.

Code after use() is generally teardown.

Q4. What is the difference between test-scoped and worker-scoped fixtures?

Test-scoped fixtures are created for each test.

Worker-scoped fixtures are created once per worker and can be reused by tests within that worker.

Q5. Can fixtures depend on other fixtures?

Yes.

loggedInPage: async ({ page, user }, use) => {


});

Here loggedInPage depends on both page and user.

Q6. Can fixtures be used with Page Object Model?

Yes.

Fixtures are commonly used to instantiate Page Objects and inject them into tests.

Q7. Why use custom fixtures?

Custom fixtures help create:

Reusable setup
Authentication
Test data
Page Objects
API clients
Database setup
Environment configuration
26. Best Practices
Use built-in fixtures whenever possible.
Use custom fixtures for reusable dependencies.
Keep fixture logic separate from test logic.
Use Page Objects with fixtures for large frameworks.
Avoid putting unrelated setup into one large fixture.
Keep fixtures small and focused.
Use teardown when resources need cleanup.
Prefer dependency injection over global variables.
Use worker-scoped fixtures carefully.
Avoid unnecessary browser/context creation.
27. Key Takeaway

Playwright fixtures are one of the most important concepts for building a professional automation framework.

The basic pattern is:

fixture: async ({ dependencies }, use) => {


    // Setup


    await use(fixtureValue);


    // Teardown


}

The overall architecture becomes:

Test
  |
  v
Fixture
  |
  +---- Page Object
  |
  +---- Test Data
  |
  +---- Authentication
  |
  +---- API Client
  |
  +---- Utilities

For senior-level Playwright automation, understand:

Built-in fixtures
Custom fixtures
test.extend()
use()
Fixture dependencies
Test scope
Worker scope
Setup and teardown
Page Object + fixtures
Authentication fixtures
API fixtures


**Next topic:** `Hooks/Playwright-Hooks.md`
