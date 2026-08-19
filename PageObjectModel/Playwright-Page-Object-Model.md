# Playwright Page Object Model (POM)

## 1. Introduction

Page Object Model (POM) is a design pattern used in test automation to separate:

- Test logic
- Page locators
- Page actions
- Reusable page functionality

Instead of writing locators and UI actions directly inside every test, we create a separate class for each application page.

This makes Playwright tests:

- Reusable
- Maintainable
- Readable
- Scalable
- Easier to debug

---

# 2. Why Use Page Object Model?

Without POM:

```typescript
import { test, expect } from '@playwright/test';

test('Login test', async ({ page }) => {
    await page.goto('https://example.com');

    await page.locator('#username').fill('admin');
    await page.locator('#password').fill('password');
    await page.locator('#login').click();

    await expect(page.locator('#dashboard')).toBeVisible();
});
If the login locator changes, every test using the login functionality may need to be updated.

With POM:

await loginPage.login('admin', 'password');

The locator and implementation are maintained in one place.

3. Basic POM Structure

A typical Playwright project can be organized as:

PlaywrightProject/
│
├── tests/
│   ├── login.spec.ts
│   └── dashboard.spec.ts
│
├── pages/
│   ├── LoginPage.ts
│   └── DashboardPage.ts
│
├── fixtures/
│   └── test-fixtures.ts
│
├── test-data/
│   └── users.json
│
├── playwright.config.ts
├── package.json
└── tsconfig.json
4. Creating a Page Object

Example:

import { Page, Locator } from '@playwright/test';


export class LoginPage {


    readonly page: Page;
    readonly username: Locator;
    readonly password: Locator;
    readonly loginButton: Locator;


    constructor(page: Page) {
        this.page = page;


        this.username = page.locator('#username');
        this.password = page.locator('#password');
        this.loginButton = page.locator('#login');
    }


    async login(username: string, password: string) {
        await this.username.fill(username);
        await this.password.fill(password);
        await this.loginButton.click();
    }
}
5. Using the Page Object in a Test
import { test, expect } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';


test('Login test', async ({ page }) => {


    const loginPage = new LoginPage(page);


    await page.goto('https://example.com');


    await loginPage.login('admin', 'password');


    await expect(page.locator('#dashboard')).toBeVisible();
});

The test is now easier to read.

6. Page Object Constructor

The constructor receives the Playwright Page object.

constructor(page: Page) {
    this.page = page;
}

Example:

const loginPage = new LoginPage(page);

Here:

Playwright Page
      ↓
LoginPage
      ↓
Locators + Actions
7. Defining Locators

Locators should normally be defined as class properties.

readonly username: Locator;
readonly password: Locator;
readonly loginButton: Locator;

Then initialize them in the constructor:

this.username = page.locator('#username');
this.password = page.locator('#password');
this.loginButton = page.locator('#login');
8. Using getByRole()

Playwright recommends user-facing locators where possible.

Example:

this.loginButton = page.getByRole('button', {
    name: 'Login'
});

Instead of:

this.loginButton = page.locator('#login');
9. Using getByLabel()

For form fields:

this.username = page.getByLabel('Username');
this.password = page.getByLabel('Password');

This is usually more readable than CSS selectors.

10. Using getByText()

Example:

this.welcomeMessage = page.getByText('Welcome');
11. Using getByTestId()

If the application provides test IDs:

this.loginButton = page.getByTestId('login-button');

HTML:

<button data-testid="login-button">
    Login
</button>
12. Complete LoginPage Example
import { Page, Locator } from '@playwright/test';


export class LoginPage {


    readonly page: Page;


    readonly usernameInput: Locator;
    readonly passwordInput: Locator;
    readonly loginButton: Locator;
    readonly errorMessage: Locator;


    constructor(page: Page) {


        this.page = page;


        this.usernameInput = page.getByLabel('Username');
        this.passwordInput = page.getByLabel('Password');


        this.loginButton = page.getByRole('button', {
            name: 'Login'
        });


        this.errorMessage = page.getByText('Invalid username or password');
    }


    async enterUsername(username: string) {
        await this.usernameInput.fill(username);
    }


    async enterPassword(password: string) {
        await this.passwordInput.fill(password);
    }


    async clickLogin() {
        await this.loginButton.click();
    }


    async login(username: string, password: string) {


        await this.enterUsername(username);
        await this.enterPassword(password);
        await this.clickLogin();
    }


    async getErrorMessage() {
        return await this.errorMessage.textContent();
    }
}
13. Test Using LoginPage
import { test, expect } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';


test('Valid Login', async ({ page }) => {


    const loginPage = new LoginPage(page);


    await page.goto('https://example.com/login');


    await loginPage.login(
        'admin',
        'password'
    );


    await expect(
        page.getByText('Dashboard')
    ).toBeVisible();
});
14. Page Object Should Contain Actions

A Page Object should contain operations that belong to that page.

For example:

async login(username: string, password: string) {
    await this.usernameInput.fill(username);
    await this.passwordInput.fill(password);
    await this.loginButton.click();
}

Another example:

async logout() {
    await this.logoutButton.click();
}
15. Page Object Should Not Contain Test Data

Avoid:

async login() {


    await this.username.fill('admin');
    await this.password.fill('password');


}

Better:

async login(username: string, password: string) {


    await this.username.fill(username);
    await this.password.fill(password);


}

Test data should come from:

Test files
JSON
CSV
Environment variables
Fixtures
API responses
Data providers or equivalent mechanisms
16. Page Object Should Not Contain Too Much Assertion Logic

Avoid putting every test assertion inside the Page Object.

Instead of:

async verifyDashboard() {


    await expect(
        this.dashboard
    ).toBeVisible();


}

You can keep the Page Object responsible for page behavior and let the test own scenario-specific assertions:

await loginPage.login('admin', 'password');


await expect(
    dashboardPage.dashboard
).toBeVisible();

However, reusable page-level verification methods can be appropriate when they represent meaningful page behavior.

17. Dashboard Page

Example:

import { Page, Locator } from '@playwright/test';


export class DashboardPage {


    readonly page: Page;
    readonly dashboardTitle: Locator;
    readonly profileButton: Locator;
    readonly logoutButton: Locator;


    constructor(page: Page) {


        this.page = page;


        this.dashboardTitle =
            page.getByRole('heading', {
                name: 'Dashboard'
            });


        this.profileButton =
            page.getByRole('button', {
                name: 'Profile'
            });


        this.logoutButton =
            page.getByRole('button', {
                name: 'Logout'
            });
    }


    async openProfile() {
        await this.profileButton.click();
    }


    async logout() {
        await this.logoutButton.click();
    }
}
18. Multiple Page Objects in One Test
import { test, expect } from '@playwright/test';


import { LoginPage } from '../pages/LoginPage';
import { DashboardPage } from '../pages/DashboardPage';


test('Login and Logout', async ({ page }) => {


    const loginPage = new LoginPage(page);
    const dashboardPage = new DashboardPage(page);


    await page.goto('https://example.com/login');


    await loginPage.login(
        'admin',
        'password'
    );


    await expect(
        dashboardPage.dashboardTitle
    ).toBeVisible();


    await dashboardPage.logout();
});
19. Page Navigation Inside Page Objects

You can create a navigation method:

async navigate() {


    await this.page.goto(
        'https://example.com/login'
    );
}

Then:

await loginPage.navigate();

However, keeping environment-specific URLs in configuration or environment variables is often better for larger frameworks.

20. Using Base URL

playwright.config.ts:

import { defineConfig } from '@playwright/test';


export default defineConfig({


    use: {
        baseURL: 'https://example.com'
    }


});

Then:

await page.goto('/login');

This avoids hardcoding the full URL in every test.

21. POM With BasePage

For larger frameworks, common functionality can be moved into a BasePage.

import { Page } from '@playwright/test';


export class BasePage {


    constructor(protected page: Page) {}


    async navigate(path: string) {
        await this.page.goto(path);
    }


    async reload() {
        await this.page.reload();
    }


    async goBack() {
        await this.page.goBack();
    }


}
22. Extending BasePage
import { Page, Locator } from '@playwright/test';
import { BasePage } from './BasePage';


export class LoginPage extends BasePage {


    readonly username: Locator;
    readonly password: Locator;
    readonly loginButton: Locator;


    constructor(page: Page) {


        super(page);


        this.username =
            page.getByLabel('Username');


        this.password =
            page.getByLabel('Password');


        this.loginButton =
            page.getByRole('button', {
                name: 'Login'
            });
    }


    async login(username: string, password: string) {


        await this.username.fill(username);
        await this.password.fill(password);
        await this.loginButton.click();
    }
}
23. POM With Fixtures

Playwright fixtures can provide Page Objects automatically.

test-fixtures.ts:

import {
    test as base
} from '@playwright/test';


import { LoginPage } from '../pages/LoginPage';
import { DashboardPage } from '../pages/DashboardPage';


type Fixtures = {
    loginPage: LoginPage;
    dashboardPage: DashboardPage;
};


export const test = base.extend<Fixtures>({


    loginPage: async ({ page }, use) => {


        await use(
            new LoginPage(page)
        );
    },


    dashboardPage: async ({ page }, use) => {


        await use(
            new DashboardPage(page)
        );
    }


});


export { expect } from '@playwright/test';
24. Test Using Fixtures
import {
    test,
    expect
} from '../fixtures/test-fixtures';


test('Login test', async ({
    page,
    loginPage,
    dashboardPage
}) => {


    await page.goto('/login');


    await loginPage.login(
        'admin',
        'password'
    );


    await expect(
        dashboardPage.dashboardTitle
    ).toBeVisible();
});

This removes repetitive Page Object creation from individual tests.

25. Recommended Framework Structure

A scalable Playwright framework can look like:

PlaywrightProject/
│
├── tests/
│   ├── login.spec.ts
│   ├── dashboard.spec.ts
│   └── checkout.spec.ts
│
├── pages/
│   ├── BasePage.ts
│   ├── LoginPage.ts
│   ├── DashboardPage.ts
│   ├── ProductPage.ts
│   └── CheckoutPage.ts
│
├── fixtures/
│   └── test-fixtures.ts
│
├── test-data/
│   ├── users.json
│   └── products.json
│
├── utils/
│   ├── ApiUtils.ts
│   ├── DateUtils.ts
│   └── FileUtils.ts
│
├── config/
│   └── test-config.ts
│
├── playwright.config.ts
├── package.json
└── tsconfig.json
26. POM vs Test File
Test File

Responsible for:

Test scenario
Test data
Business flow
Assertions
Page Object

Responsible for:

Locators
Page actions
Navigation
Reusable page functionality

Example:

login.spec.ts
       |
       v
LoginPage.ts
       |
       +-- username locator
       +-- password locator
       +-- login button
       +-- login()
27. Good POM Design

Good:

await loginPage.login(
    username,
    password
);

Avoid:

await page.locator('#username').fill(username);
await page.locator('#password').fill(password);
await page.locator('#login').click();

The second approach exposes implementation details in every test.

28. Method Design

Prefer business-level methods.

Instead of:

await loginPage.enterUsername('admin');
await loginPage.enterPassword('password');
await loginPage.clickLogin();

If these operations are always performed together, create:

await loginPage.login(
    'admin',
    'password'
);

This makes the test easier to understand.

29. Reusable Page Methods

Example:

async searchProduct(productName: string) {


    await this.searchBox.fill(productName);
    await this.searchButton.click();
}

Test:

await productPage.searchProduct(
    'iPhone'
);
30. Dynamic Locators

POM can accept parameters for dynamic elements.

product(productName: string) {


    return this.page.getByRole(
        'link',
        {
            name: productName
        }
    );
}

Usage:

await productPage
    .product('Laptop')
    .click();
31. Locator Factory Methods

Another example:

getProductCard(productName: string) {


    return this.page.locator(
        `[data-product="${productName}"]`
    );
}

Usage:

await productPage
    .getProductCard('Laptop')
    .click();
32. POM With API Testing

Page Objects can also be combined with API setup.

Example:

const response = await request.post(
    '/api/users',
    {
        data: {
            username: 'admin'
        }
    }
);

Then UI testing can use the created data.

This is useful for:

Test setup
Test data creation
Cleanup
Reducing UI setup time
33. POM and Authentication

Authentication can be handled using:

Login Page
Storage state
Authentication fixtures
API login

Example:

await page.goto('/dashboard');

If authentication state is already configured, tests don't need to perform the login UI flow every time.

34. POM and Environment Configuration

Avoid:

await page.goto(
    'https://qa.example.com/login'
);

Prefer:

await page.goto('/login');

with:

use: {
    baseURL: process.env.BASE_URL
}

Then environment-specific execution can use different URLs.

35. POM Best Practices
1. Keep locators inside Page Objects
readonly loginButton = page.getByRole(
    'button',
    { name: 'Login' }
);
2. Use stable locators

Prefer:

getByRole()
getByLabel()
getByTestId()

over fragile XPath expressions.

3. Keep tests readable

Good:

await loginPage.login(
    username,
    password
);
4. Avoid duplicated actions

If multiple tests perform the same workflow, create a reusable method.

5. Keep Page Objects focused

One Page Object should generally represent one page or meaningful page component.

6. Avoid hardcoded test data

Pass data as parameters.

7. Use fixtures for dependency management

Fixtures can create Page Objects automatically.

36. Component Objects

Not everything has to be a full page.

Reusable components can have their own classes.

Examples:

Header
Footer
Navigation Menu
Date Picker
Modal
Search Box
Table

Example:

import { Page, Locator } from '@playwright/test';


export class Header {


    readonly page: Page;
    readonly profileButton: Locator;
    readonly logoutButton: Locator;


    constructor(page: Page) {


        this.page = page;


        this.profileButton =
            page.getByRole('button', {
                name: 'Profile'
            });


        this.logoutButton =
            page.getByRole('button', {
                name: 'Logout'
            });
    }


    async logout() {
        await this.profileButton.click();
        await this.logoutButton.click();
    }
}
37. Page Object Composition

A page can contain reusable components.

export class DashboardPage {


    readonly header: Header;


    constructor(page: Page) {


        this.header = new Header(page);
    }
}

Then:

await dashboardPage.header.logout();

This is useful for large applications.

38. POM With TypeScript Types

Using TypeScript gives compile-time checking.

async login(
    username: string,
    password: string
) {
    await this.username.fill(username);
    await this.password.fill(password);
}

If an incorrect type is passed:

await loginPage.login(
    123,
    true
);

TypeScript can identify the problem.

39. Example End-to-End POM
LoginPage.ts
import {
    Page,
    Locator
} from '@playwright/test';


export class LoginPage {


    readonly page: Page;


    readonly username: Locator;
    readonly password: Locator;
    readonly loginButton: Locator;


    constructor(page: Page) {


        this.page = page;


        this.username =
            page.getByLabel('Username');


        this.password =
            page.getByLabel('Password');


        this.loginButton =
            page.getByRole(
                'button',
                { name: 'Login' }
            );
    }


    async navigate() {


        await this.page.goto('/login');
    }


    async login(
        username: string,
        password: string
    ) {


        await this.username.fill(username);


        await this.password.fill(password);


        await this.loginButton.click();
    }
}
DashboardPage.ts
import {
    Page,
    Locator
} from '@playwright/test';


export class DashboardPage {


    readonly page: Page;
    readonly title: Locator;


    constructor(page: Page) {


        this.page = page;


        this.title =
            page.getByRole(
                'heading',
                { name: 'Dashboard' }
            );
    }
}
login.spec.ts
import {
    test,
    expect
} from '@playwright/test';


import { LoginPage }
    from '../pages/LoginPage';


import { DashboardPage }
    from '../pages/DashboardPage';


test('Valid Login', async ({ page }) => {


    const loginPage =
        new LoginPage(page);


    const dashboardPage =
        new DashboardPage(page);


    await loginPage.navigate();


    await loginPage.login(
        'admin',
        'password'
    );


    await expect(
        dashboardPage.title
    ).toBeVisible();
});
40. POM Execution Flow
Test
 |
 |-- Create LoginPage
 |
 |-- Navigate to Login
 |
 |-- login(username, password)
 |       |
 |       |-- Fill username
 |       |-- Fill password
 |       |-- Click Login
 |
 |-- Create/Use DashboardPage
 |
 |-- Verify Dashboard
41. Common POM Mistakes
Mistake 1: Putting everything in one Page Object

Avoid creating:

ApplicationPage.ts

with hundreds of locators and methods.

Instead:

LoginPage.ts
DashboardPage.ts
ProductPage.ts
CheckoutPage.ts
Mistake 2: Hardcoded test data

Avoid:

async login() {


    await this.username.fill('admin');
    await this.password.fill('password');
}

Prefer:

async login(
    username: string,
    password: string
) {
    await this.username.fill(username);
    await this.password.fill(password);
}
Mistake 3: Fragile XPath

Avoid:

page.locator(
    '/html/body/div[1]/div[2]/button'
);

Prefer:

page.getByRole(
    'button',
    { name: 'Login' }
);
Mistake 4: Duplicate methods

Avoid creating multiple methods that do the same thing.

Bad:

loginUser()
performLogin()
doLogin()
loginToApplication()

Choose one clear method:

login()
42. POM Interview Questions
Beginner
1. What is Page Object Model?

POM is a design pattern that separates page locators and page actions from test scenarios.

2. Why use POM?

To improve:

Maintainability
Reusability
Readability
Scalability
3. What does a Page Object contain?

Usually:

Page reference
Locators
Page actions
Reusable methods
4. Should test data be stored in Page Objects?

Generally no. Test data should be supplied by tests, fixtures, configuration, or external data sources.

43. Intermediate Interview Questions
5. How do you pass Playwright Page into a Page Object?
constructor(page: Page) {
    this.page = page;
}
6. How do you create reusable login functionality?
async login(
    username: string,
    password: string
) {
    await this.username.fill(username);
    await this.password.fill(password);
    await this.loginButton.click();
}
7. Can Page Objects use Playwright fixtures?

Yes.

Fixtures can instantiate and inject Page Objects into tests.

8. Can one Page Object use another Page Object?

Yes.

This is useful for reusable components and shared navigation.

44. Advanced Interview Questions
9. POM vs Fixtures?

POM organizes page behavior.

Fixtures manage test dependencies and lifecycle.

They are complementary.

Fixture
   |
   +-- Page
   |
   +-- LoginPage
   |
   +-- DashboardPage
10. Should assertions be inside Page Objects?

Not necessarily.

Scenario-specific assertions are generally clearer in test files.

However, reusable page-level verification methods can be appropriate.

11. How do you handle dynamic elements?

Use parameterized locator methods.

getProduct(name: string) {
    return this.page.getByRole(
        'link',
        { name }
    );
}
12. How do you design POM for a large application?

Use:

BasePage
Page Objects
Component Objects
Fixtures
Test Data
Utilities
Configuration
45. POM and Selenium Comparison
Selenium POM
public class LoginPage {


    WebDriver driver;


    By username =
        By.id("username");


    By password =
        By.id("password");


    By loginButton =
        By.id("login");


    public void login(
        String user,
        String pass) {


        driver.findElement(username)
              .sendKeys(user);


        driver.findElement(password)
              .sendKeys(pass);


        driver.findElement(loginButton)
              .click();
    }
}
Playwright POM
export class LoginPage {


    constructor(private page: Page) {}


    username =
        this.page.getByLabel('Username');


    password =
        this.page.getByLabel('Password');


    loginButton =
        this.page.getByRole(
            'button',
            { name: 'Login' }
        );


    async login(
        user: string,
        pass: string
    ) {


        await this.username.fill(user);


        await this.password.fill(pass);


        await this.loginButton.click();
    }
}

The design concept is very similar.

Playwright provides additional features such as:

Auto-waiting
Built-in locators
Fixtures
Browser contexts
Trace viewer
Parallel execution
46. Recommended Playwright POM Pattern

For a professional framework:

tests/
    login.spec.ts
    dashboard.spec.ts


pages/
    BasePage.ts
    LoginPage.ts
    DashboardPage.ts


components/
    Header.ts
    Footer.ts
    Navigation.ts


fixtures/
    test-fixtures.ts


test-data/
    users.json
    products.json


utils/
    ApiUtils.ts
    DateUtils.ts


config/
    environment.ts


playwright.config.ts
47. Key Takeaways
POM
 |
 +-- Separates UI implementation from tests
 |
 +-- Locators belong in Page Objects
 |
 +-- Actions belong in Page Objects
 |
 +-- Test scenarios belong in test files
 |
 +-- Test data should be externalized
 |
 +-- Fixtures can create Page Objects
 |
 +-- Components can be modeled separately
 |
 +-- BasePage can contain common functionality
 |
 +-- Use stable Playwright locators
 |
 +-- Keep Page Objects focused and reusable
48. Final Example
Page Object
export class LoginPage {


    constructor(private page: Page) {}


    private username =
        this.page.getByLabel('Username');


    private password =
        this.page.getByLabel('Password');


    private loginButton =
        this.page.getByRole(
            'button',
            { name: 'Login' }
        );


    async login(
        username: string,
        password: string
    ) {


        await this.username.fill(username);


        await this.password.fill(password);


        await this.loginButton.click();
    }
}
Test
test('User can login', async ({ page }) => {


    const loginPage =
        new LoginPage(page);


    await page.goto('/login');


    await loginPage.login(
        'admin',
        'password'
    );


    await expect(
        page.getByRole(
            'heading',
            { name: 'Dashboard' }
        )
    ).toBeVisible();
});

The test now describes what the user does, while the Page Object handles how the application is interacted with.

That separation is the main purpose of the Playwright Page Object Model.



