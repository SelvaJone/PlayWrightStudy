# Playwright Authentication — Advanced

**File:** `Authentication/Playwright-Authentication-Advanced.md`

## 1. Overview

Authentication is one of the most important parts of modern Playwright test automation.

A real application may require:

* Username and password
* MFA or OTP
* SSO
* OAuth
* Access tokens
* Refresh tokens
* Session cookies
* Role-based access
* Multiple users
* API authentication

Logging in through the UI before every test can make a test suite slow and unreliable.

Playwright provides several approaches to authenticate efficiently and reuse authenticated browser state.

The most commonly used approach is:

```text
Login once
   ↓
Save authentication state
   ↓
Reuse storageState
   ↓
Run tests without logging in again
```

---

# 2. What Is Authentication State?

Authentication state represents information that tells an application that a user is already logged in.

It can include:

* Cookies
* Local storage
* Session storage-related application state
* Tokens stored by the application

Playwright commonly saves this state using:

```text
storageState
```

Example:

```typescript
await page.context().storageState({
  path: 'playwright/.auth/user.json'
});
```

Later, another browser context can reuse it:

```typescript
const context = await browser.newContext({
  storageState: 'playwright/.auth/user.json'
});
```

---

# 3. Why Reuse Authentication?

Without authentication reuse:

```text
Test 1 → Login → Test
Test 2 → Login → Test
Test 3 → Login → Test
Test 4 → Login → Test
```

With authentication reuse:

```text
Login once
   ↓
Save authentication state
   ↓
Test 1
Test 2
Test 3
Test 4
```

Benefits:

* Faster execution
* Less duplicated code
* Reduced dependency on login UI
* Fewer authentication-related failures
* Easier parallel execution
* Cleaner test architecture

---

# 4. Recommended Authentication Folder Structure

A common project structure is:

```text
playwright-project/
│
├── tests/
│   ├── login.spec.ts
│   ├── dashboard.spec.ts
│   └── orders.spec.ts
│
├── tests-examples/
│
├── playwright/
│   └── .auth/
│       ├── user.json
│       ├── admin.json
│       └── manager.json
│
├── fixtures/
│   └── auth.fixture.ts
│
├── pages/
│   ├── LoginPage.ts
│   └── DashboardPage.ts
│
├── playwright.config.ts
├── package.json
└── tsconfig.json
```

The authentication files should normally be excluded from Git.

Add:

```text
playwright/.auth/
```

to `.gitignore`.

---

# 5. Basic Login and Save Storage State

Example:

```typescript
import { test } from '@playwright/test';

test('authenticate user', async ({ page }) => {
  await page.goto('https://example.com/login');

  await page.getByLabel('Username').fill('testuser');
  await page.getByLabel('Password').fill('Password123');

  await page.getByRole('button', { name: 'Login' }).click();

  await page.waitForURL('**/dashboard');

  await page.context().storageState({
    path: 'playwright/.auth/user.json'
  });
});
```

The resulting file contains authentication information required to recreate the logged-in state.

---

# 6. Reuse storageState

Once authentication state has been created:

```typescript
import { test } from '@playwright/test';

test.use({
  storageState: 'playwright/.auth/user.json'
});

test('open dashboard', async ({ page }) => {
  await page.goto('https://example.com/dashboard');

  await page.getByText('Dashboard').isVisible();
});
```

The test starts with the saved authentication state.

---

# 7. Configure Authentication Globally

Authentication can be configured in `playwright.config.ts`.

Example:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  use: {
    baseURL: 'https://example.com',
    storageState: 'playwright/.auth/user.json'
  }
});
```

Now all tests using this configuration will start with the authenticated state.

Example:

```typescript
import { test, expect } from '@playwright/test';

test('authenticated dashboard', async ({ page }) => {
  await page.goto('/dashboard');

  await expect(page.getByRole('heading', {
    name: 'Dashboard'
  })).toBeVisible();
});
```

---

# 8. Authentication Setup Project

A better approach for larger projects is to create a dedicated setup project.

Example:

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  projects: [
    {
      name: 'setup',
      testMatch: /.*\.setup\.ts/
    },

    {
      name: 'chromium',
      use: {
        browserName: 'chromium',
        storageState: 'playwright/.auth/user.json'
      },
      dependencies: ['setup']
    }
  ]
});
```

The setup project runs before the dependent test project.

---

# 9. Authentication Setup File

Example:

```typescript
import { test as setup } from '@playwright/test';

const authFile = 'playwright/.auth/user.json';

setup('authenticate', async ({ page }) => {
  await page.goto('/login');

  await page.getByLabel('Username').fill(
    process.env.TEST_USERNAME!
  );

  await page.getByLabel('Password').fill(
    process.env.TEST_PASSWORD!
  );

  await page.getByRole('button', {
    name: 'Login'
  }).click();

  await page.waitForURL('**/dashboard');

  await page.context().storageState({
    path: authFile
  });
});
```

---

# 10. Why Use a Setup Project?

A setup project provides:

```text
Authentication Setup
        ↓
Chromium Tests
        ↓
Firefox Tests
        ↓
WebKit Tests
```

Advantages:

* Centralized authentication
* Clear project dependencies
* Reusable state
* Better scalability
* Easy CI/CD integration

---

# 11. Authentication with Environment Variables

Never hard-code credentials in test code.

Bad:

```typescript
await page.getByLabel('Username').fill('myusername');
await page.getByLabel('Password').fill('mypassword');
```

Better:

```typescript
await page.getByLabel('Username').fill(
  process.env.TEST_USERNAME!
);

await page.getByLabel('Password').fill(
  process.env.TEST_PASSWORD!
);
```

Example `.env`:

```text
TEST_USERNAME=testuser
TEST_PASSWORD=secretpassword
```

Do not commit `.env` if it contains real credentials.

---

# 12. Using dotenv

Install:

```bash
npm install dotenv
```

Then:

```typescript
import dotenv from 'dotenv';

dotenv.config();
```

Or load environment variables through your test configuration.

Example:

```typescript
import { defineConfig } from '@playwright/test';
import dotenv from 'dotenv';

dotenv.config();

export default defineConfig({
  use: {
    baseURL: process.env.BASE_URL
  }
});
```

---

# 13. Multiple User Roles

Enterprise applications commonly have different roles:

```text
Admin
Manager
Employee
Customer
Read-only User
```

Create separate authentication states:

```text
playwright/.auth/
├── admin.json
├── manager.json
└── user.json
```

Admin:

```typescript
test.use({
  storageState: 'playwright/.auth/admin.json'
});
```

Manager:

```typescript
test.use({
  storageState: 'playwright/.auth/manager.json'
});
```

Regular user:

```typescript
test.use({
  storageState: 'playwright/.auth/user.json'
});
```

---

# 14. Role-Based Test Suites

Example:

```typescript
import { test, expect } from '@playwright/test';

test.describe('Admin tests', () => {

  test.use({
    storageState: 'playwright/.auth/admin.json'
  });

  test('admin can access administration page', async ({ page }) => {
    await page.goto('/admin');

    await expect(
      page.getByRole('heading', {
        name: 'Administration'
      })
    ).toBeVisible();
  });

});
```

---

# 15. Multiple Authentication Setup Files

Example structure:

```text
tests/
├── auth.setup.ts
├── admin-auth.setup.ts
└── manager-auth.setup.ts
```

You can generate different storage states:

```typescript
await page.context().storageState({
  path: 'playwright/.auth/admin.json'
});
```

and:

```typescript
await page.context().storageState({
  path: 'playwright/.auth/manager.json'
});
```

---

# 16. API-Based Authentication

UI login is not always necessary.

If the application provides an authentication API, you can authenticate using `request`.

Example:

```typescript
import { request } from '@playwright/test';

const apiContext = await request.newContext();

const response = await apiContext.post(
  'https://example.com/api/login',
  {
    data: {
      username: process.env.TEST_USERNAME,
      password: process.env.TEST_PASSWORD
    }
  }
);

console.log(await response.json());
```

This is often faster than performing the login through the browser.

---

# 17. Save API Authentication State

Example:

```typescript
import { request } from '@playwright/test';

const requestContext = await request.newContext();

await requestContext.post('/api/login', {
  data: {
    username: process.env.TEST_USERNAME,
    password: process.env.TEST_PASSWORD
  }
});

await requestContext.storageState({
  path: 'playwright/.auth/api-user.json'
});
```

Then:

```typescript
test.use({
  storageState: 'playwright/.auth/api-user.json'
});
```

---

# 18. UI Login vs API Login

| Feature              | UI Login    | API Login                |
| -------------------- | ----------- | ------------------------ |
| Speed                | Slower      | Faster                   |
| UI dependency        | Yes         | No                       |
| Login page testing   | Yes         | No                       |
| Authentication setup | Simple      | Requires API             |
| Stability            | Can vary    | Usually higher           |
| Best for             | Login tests | Most authenticated tests |

A common strategy is:

```text
Login test
    ↓
UI authentication

Other tests
    ↓
API authentication
    ↓
storageState
```

---

# 19. Testing the Login Page Separately

Do not completely eliminate UI login testing.

Create dedicated tests:

```typescript
test('valid user can login', async ({ page }) => {
  await page.goto('/login');

  await page.getByLabel('Username').fill('user');
  await page.getByLabel('Password').fill('password');

  await page.getByRole('button', {
    name: 'Login'
  }).click();

  await expect(page).toHaveURL(/dashboard/);
});
```

For the rest of the suite, reuse authentication state.

---

# 20. Authentication Fixtures

Fixtures provide a clean way to manage authentication.

Example:

```typescript
import {
  test as base
} from '@playwright/test';

type Fixtures = {
  authenticatedPage: void;
};

export const test = base.extend<Fixtures>({
  authenticatedPage: async ({ page }, use) => {

    await page.goto('/login');

    await page.getByLabel('Username').fill(
      process.env.TEST_USERNAME!
    );

    await page.getByLabel('Password').fill(
      process.env.TEST_PASSWORD!
    );

    await page.getByRole('button', {
      name: 'Login'
    }).click();

    await page.waitForURL('**/dashboard');

    await use();
  }
});
```

Tests:

```typescript
test('authenticated test', async ({
  page,
  authenticatedPage
}) => {
  await page.goto('/dashboard');
});
```

---

# 21. Worker-Scoped Authentication

For large test suites, worker-scoped authentication can improve performance.

Concept:

```text
Worker 1
   ↓
Authenticate
   ↓
Tests

Worker 2
   ↓
Authenticate
   ↓
Tests

Worker 3
   ↓
Authenticate
   ↓
Tests
```

Each worker can have its own authentication state.

This is especially useful when:

* Tests run in parallel
* Users cannot share sessions
* Tests modify server-side state
* Each worker needs an isolated account

---

# 22. Authentication and Parallel Execution

Be careful when multiple tests use the same account.

Example:

```text
Test A → Admin account
Test B → Admin account
Test C → Admin account
```

If tests modify the same data, they may interfere with each other.

Better:

```text
Worker 1 → Admin User 1
Worker 2 → Admin User 2
Worker 3 → Admin User 3
```

Use separate accounts when test isolation is important.

---

# 23. Authentication with Projects

Different projects can use different users.

Example:

```typescript
projects: [
  {
    name: 'admin',
    use: {
      storageState: 'playwright/.auth/admin.json'
    }
  },

  {
    name: 'user',
    use: {
      storageState: 'playwright/.auth/user.json'
    }
  }
]
```

Run:

```bash
npx playwright test --project=admin
```

or:

```bash
npx playwright test --project=user
```

---

# 24. Session Cookies

Authentication may be stored in cookies.

Example:

```typescript
const cookies = await context.cookies();

console.log(cookies);
```

You can inspect cookies:

```typescript
for (const cookie of cookies) {
  console.log(cookie.name);
  console.log(cookie.domain);
  console.log(cookie.path);
});
```

Avoid printing sensitive cookie values in CI logs.

---

# 25. Adding Cookies Manually

For certain testing scenarios:

```typescript
await context.addCookies([
  {
    name: 'sessionId',
    value: 'abc123',
    domain: 'example.com',
    path: '/'
  }
]);
```

Then:

```typescript
await page.goto('https://example.com/dashboard');
```

The application may recognize the session.

Use this only when it matches the application's actual authentication design.

---

# 26. Local Storage Authentication

Some applications store tokens in local storage.

Example:

```typescript
await page.evaluate(() => {
  localStorage.setItem(
    'accessToken',
    'test-token'
  );
});
```

Then navigate:

```typescript
await page.goto('/dashboard');
```

However, this approach should only be used when the application genuinely uses local storage for authentication.

---

# 27. Session Storage Consideration

`storageState` primarily handles cookies and local storage.

Session storage is not automatically persisted in the same way.

If an application relies heavily on session storage, you may need a custom setup strategy.

For example:

```typescript
await page.addInitScript(() => {
  window.sessionStorage.setItem(
    'token',
    'test-token'
  );
});
```

---

# 28. OAuth Authentication

OAuth flows may involve:

```text
Application
    ↓
Authorization Server
    ↓
Login
    ↓
Authorization Code
    ↓
Access Token
    ↓
Application
```

Testing the complete OAuth flow through the UI can be slow.

Where appropriate, authenticate through an API or prepare the authenticated browser state.

However, maintain dedicated tests for the actual OAuth integration.

---

# 29. SSO Authentication

Enterprise applications often use:

```text
SAML
OIDC
OAuth 2.0
Azure AD
Okta
Google Identity
```

SSO can introduce:

* Redirects
* MFA
* External domains
* Identity-provider sessions
* Security policies

For normal application tests, a reusable authenticated state is usually preferable to performing SSO repeatedly.

Keep dedicated SSO tests for validating the authentication integration itself.

---

# 30. MFA and OTP

MFA can make automated authentication more complicated.

Examples:

```text
Password
   ↓
OTP
   ↓
Dashboard
```

or:

```text
Password
   ↓
Authenticator
   ↓
Approval
   ↓
Dashboard
```

Avoid disabling real security controls in production.

For test environments, teams commonly provide controlled authentication mechanisms such as:

* Test users
* Test OTP services
* API authentication
* Authentication bypass specifically designed for testing
* Dedicated identity-provider test environments

---

# 31. Authentication Expiration

Authentication state can expire.

Possible causes:

```text
Session timeout
Token expiration
Cookie expiration
Password change
Server-side logout
Identity-provider expiration
```

If tests suddenly redirect to:

```text
/login
```

the saved authentication state may be stale.

Regenerate:

```text
playwright/.auth/user.json
```

or rerun the authentication setup.

---

# 32. Detecting Authentication Failure

Example:

```typescript
await page.goto('/dashboard');

if (page.url().includes('/login')) {
  throw new Error(
    'Authentication state is invalid or expired.'
  );
}
```

A better approach is to assert expected authenticated content:

```typescript
await expect(
  page.getByRole('heading', {
    name: 'Dashboard'
  })
).toBeVisible();
```

---

# 33. Authentication State Should Not Be Committed

Authentication files may contain sensitive information.

Do not commit:

```text
playwright/.auth/user.json
playwright/.auth/admin.json
```

Add:

```text
playwright/.auth/
```

to `.gitignore`.

Example:

```text
node_modules/
playwright/.auth/
.env
test-results/
playwright-report/
```

---

# 34. Authentication in CI/CD

CI/CD pipelines should provide credentials securely.

Example environment variables:

```text
TEST_USERNAME
TEST_PASSWORD
BASE_URL
```

Do not write:

```yaml
TEST_PASSWORD: MyRealPassword123
```

Instead, use the CI platform's secret-management mechanism.

Concept:

```text
CI Secret
   ↓
Environment Variable
   ↓
Playwright Authentication
   ↓
storageState
   ↓
Tests
```

---

# 35. GitHub Actions Example

Example:

```yaml
name: Playwright Tests

on:
  push:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - run: npm ci

      - run: npx playwright install --with-deps

      - run: npx playwright test
        env:
          BASE_URL: ${{ secrets.BASE_URL }}
          TEST_USERNAME: ${{ secrets.TEST_USERNAME }}
          TEST_PASSWORD: ${{ secrets.TEST_PASSWORD }}
```

The exact syntax may vary depending on the CI platform.

---

# 36. Reauthentication Strategy

A mature framework should have a strategy for expired sessions.

Example:

```text
Start Test Run
      ↓
Check Authentication
      ↓
Valid?
 ┌────┴────┐
Yes        No
 ↓          ↓
Tests    Authenticate
             ↓
       Save storageState
             ↓
           Tests
```

---

# 37. Authentication Helper

You can create a reusable helper:

```typescript
import {
  Page
} from '@playwright/test';

export async function login(
  page: Page,
  username: string,
  password: string
) {
  await page.goto('/login');

  await page.getByLabel('Username').fill(username);

  await page.getByLabel('Password').fill(password);

  await page.getByRole('button', {
    name: 'Login'
  }).click();

  await page.waitForURL('**/dashboard');
}
```

Then:

```typescript
await login(
  page,
  process.env.TEST_USERNAME!,
  process.env.TEST_PASSWORD!
);
```

---

# 38. Authentication Page Object

Example:

```typescript
import { Page } from '@playwright/test';

export class LoginPage {

  constructor(private page: Page) {}

  username = this.page.getByLabel('Username');

  password = this.page.getByLabel('Password');

  loginButton = this.page.getByRole(
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

    await this.page.waitForURL(
      '**/dashboard'
    );
  }
}
```

---

# 39. Authentication with Page Object

Example:

```typescript
import { test } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';

test('login', async ({ page }) => {

  const loginPage = new LoginPage(page);

  await page.goto('/login');

  await loginPage.login(
    process.env.TEST_USERNAME!,
    process.env.TEST_PASSWORD!
  );
});
```

---

# 40. API Login + Browser Context

One powerful approach is:

```text
API Login
    ↓
Receive Session
    ↓
Save Authentication State
    ↓
Create Browser Context
    ↓
Run UI Tests
```

This avoids repeatedly navigating through the login UI.

---

# 41. Authentication State Per Role

A scalable enterprise framework might use:

```text
playwright/.auth/
│
├── admin.json
├── manager.json
├── customer.json
├── support.json
└── read-only.json
```

Tests select the appropriate state:

```typescript
test.use({
  storageState: 'playwright/.auth/customer.json'
});
```

---

# 42. Authentication State Per Environment

You may also need:

```text
playwright/.auth/
├── qa-user.json
├── stage-user.json
└── prod-user.json
```

Avoid using production credentials for automated testing unless explicitly authorized and properly secured.

---

# 43. Authentication Best Practices

Follow these principles:

1. Do not hard-code credentials.
2. Store secrets in secure environment variables.
3. Do not commit authentication state.
4. Use `storageState` for reusable sessions.
5. Use API login where appropriate.
6. Keep dedicated UI login tests.
7. Use separate users when tests modify shared data.
8. Regenerate expired authentication state.
9. Protect CI logs from sensitive values.
10. Use test-specific identity-provider environments.
11. Keep authentication setup separate from business tests.
12. Use fixtures for reusable authentication behavior.
13. Use project dependencies for centralized setup.
14. Avoid unnecessary UI login for every test.
15. Validate that the authentication state is actually valid.

---

# 44. Common Authentication Mistakes

## Mistake 1 — Hard-coded passwords

Bad:

```typescript
password = 'Password123';
```

Better:

```typescript
password = process.env.TEST_PASSWORD!;
```

---

## Mistake 2 — Committing auth files

Bad:

```text
git add playwright/.auth/user.json
```

Better:

```text
playwright/.auth/
```

in `.gitignore`.

---

## Mistake 3 — Logging sensitive information

Avoid:

```typescript
console.log(await context.cookies());
```

because cookies may contain session information.

---

## Mistake 4 — One account for everything

This can cause test interference.

Use separate accounts when required.

---

## Mistake 5 — Logging in before every test

This increases execution time and UI dependency.

Use reusable authentication state.

---

## Mistake 6 — Assuming storageState never expires

Authentication can expire.

Your framework should handle stale sessions.

---

# 45. Authentication Architecture

A strong Playwright framework can look like:

```text
                    Playwright
                         │
              ┌──────────┴──────────┐
              │                     │
        Authentication          Tests
              │                     │
       ┌──────┼──────┐       ┌──────┼──────┐
       │      │      │       │      │      │
      UI     API    SSO    Admin   User   Manager
       │      │
       └──────┼──────┘
              │
        storageState
              │
       ┌──────┼──────┐
       │      │      │
    Browser  Context  Tests
```

---

# 46. Recommended Enterprise Approach

For a large automation framework:

```text
Login Page
    ↓
Dedicated Authentication Test
    ↓
API/UI Authentication
    ↓
storageState
    ↓
Authentication Fixture
    ↓
Role-Based Tests
    ↓
Parallel Execution
    ↓
CI/CD
```

This provides good separation between authentication and functional testing.

---

# 47. Interview Question

### Q1. How do you handle authentication in Playwright?

A strong answer:

> I generally authenticate once and save the browser context state using Playwright's `storageState`. The authentication state can then be reused across tests, avoiding repeated UI logins. For larger frameworks, I use a dedicated setup project or authentication fixture. If the application provides an authentication API, I prefer API-based authentication because it is faster and less dependent on the UI. For applications with multiple roles, I maintain separate authentication states for each role and avoid sharing accounts when tests modify common data.

---

# 48. Interview Question

### Q2. What is `storageState`?

Answer:

> `storageState` allows Playwright to save and restore browser authentication state, primarily cookies and local storage. It allows tests to start with an already authenticated browser context instead of logging in through the UI for every test.

Example:

```typescript
await context.storageState({
  path: 'playwright/.auth/user.json'
});
```

Reuse:

```typescript
test.use({
  storageState: 'playwright/.auth/user.json'
});
```

---

# 49. Interview Question

### Q3. How do you handle multiple users?

Answer:

> I create separate authentication states for different roles or users, such as `admin.json`, `manager.json`, and `user.json`. Each test suite selects the appropriate `storageState`. For parallel tests that modify shared data, I prefer separate accounts per worker to prevent test interference.

---

# 50. Interview Question

### Q4. UI Login vs API Login?

Answer:

> I use UI login when I specifically need to validate the login functionality. For most authenticated functional tests, I prefer API-based authentication or a previously generated `storageState`, because it is faster, more stable, and reduces dependency on the login UI.

---

# 51. Interview Question

### Q5. How do you secure authentication credentials?

Answer:

> I never hard-code credentials in the test code or commit authentication state files. Credentials are supplied through environment variables or CI/CD secret management. Authentication state files are added to `.gitignore`, and sensitive values are never printed to test logs.

---

# 52. Interview Question

### Q6. How do you handle expired authentication?

Answer:

> I detect authentication failures by checking for the login page or expected authenticated content. If the stored state has expired, the authentication setup is rerun to generate a fresh `storageState`.

---

# 53. Interview Question

### Q7. How do you handle authentication in parallel execution?

Answer:

> I avoid sharing the same account when tests modify common data. Depending on the application, I use separate authentication states or worker-specific users. This prevents race conditions and test-data conflicts.

---

# 54. Interview Question

### Q8. Can Playwright test applications using SSO?

Answer:

> Yes. Playwright can automate SSO flows, but enterprise SSO often involves redirects, MFA, and external identity providers. For regular application testing, I prefer a controlled test identity environment and reusable authentication state. Dedicated tests can validate the actual SSO integration.

---

# 55. Interview Question

### Q9. Why shouldn't authentication files be committed to Git?

Answer:

> Authentication state may contain session cookies or tokens that can provide access to the application. Committing those files can expose credentials or active sessions. They should be ignored by Git and generated securely during local setup or CI execution.

---

# 56. Interview Question

### Q10. How would you design authentication for a large Playwright framework?

Answer:

> I would create a dedicated authentication layer using setup projects, API authentication where possible, and `storageState`. I would keep credentials in secure environment variables, maintain separate authentication states for different roles, use fixtures for reusable behavior, and create worker-specific users when parallel tests require data isolation.

---

# 57. Complete Example

## `auth.setup.ts`

```typescript
import { test as setup } from '@playwright/test';

const authFile =
  'playwright/.auth/user.json';

setup('authenticate', async ({ page }) => {

  await page.goto('/login');

  await page.getByLabel('Username').fill(
    process.env.TEST_USERNAME!
  );

  await page.getByLabel('Password').fill(
    process.env.TEST_PASSWORD!
  );

  await page.getByRole('button', {
    name: 'Login'
  }).click();

  await page.waitForURL('**/dashboard');

  await page.context().storageState({
    path: authFile
  });
});
```

---

## `playwright.config.ts`

```typescript
import {
  defineConfig
} from '@playwright/test';

export default defineConfig({

  use: {
    baseURL: process.env.BASE_URL
  },

  projects: [

    {
      name: 'setup',

      testMatch:
        /.*\.setup\.ts/
    },

    {
      name: 'chromium',

      use: {
        browserName: 'chromium',

        storageState:
          'playwright/.auth/user.json'
      },

      dependencies: ['setup']
    }

  ]
});
```

---

## `dashboard.spec.ts`

```typescript
import {
  test,
  expect
} from '@playwright/test';

test('authenticated dashboard', async ({
  page
}) => {

  await page.goto('/dashboard');

  await expect(
    page.getByRole('heading', {
      name: 'Dashboard'
    })
  ).toBeVisible();

});
```

---

# 58. Final Authentication Flow

The recommended flow is:

```text
                START
                  │
                  ▼
          Authentication Setup
                  │
          ┌───────┴────────┐
          │                │
       UI Login         API Login
          │                │
          └───────┬────────┘
                  │
                  ▼
            storageState
                  │
                  ▼
         Authentication Fixture
                  │
          ┌───────┼────────┐
          │       │        │
        Admin   Manager   User
          │       │        │
          └───────┼────────┘
                  │
                  ▼
           Parallel Tests
                  │
                  ▼
                CI/CD
```

---

# 59. Key Takeaways

```text
storageState
    ↓
Reuse authentication

Setup Project
    ↓
Centralized authentication

API Login
    ↓
Faster authentication

Fixtures
    ↓
Reusable authentication logic

Multiple Users
    ↓
Role-based testing

Worker Users
    ↓
Parallel isolation

Environment Variables
    ↓
Secure credentials

.gitignore
    ↓
Protect authentication files
```

The most important Playwright authentication concepts to remember are:

**`storageState` + Authentication Setup + API Login + Fixtures + Multiple Users + Secure Secrets + Parallel Isolation**
