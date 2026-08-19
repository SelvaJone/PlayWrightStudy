# Playwright API Testing

## 1. Introduction

Playwright is primarily known for end-to-end browser automation, but it also provides powerful API testing capabilities through `APIRequestContext`.

With Playwright API testing, you can:

* Send HTTP requests without opening a browser
* Test REST APIs
* Validate status codes
* Validate response bodies
* Send JSON request payloads
* Add authentication headers
* Chain API requests
* Combine API testing with UI testing
* Create reusable API utilities
* Prepare test data through APIs
* Clean up test data through APIs

Playwright API testing works especially well when API and UI testing are part of the same automation framework.

---

## 2. API Testing Architecture

A typical Playwright API test flow looks like:

```text
Test
 |
 +-- APIRequestContext
       |
       +-- GET
       +-- POST
       +-- PUT
       +-- PATCH
       +-- DELETE
       |
       +-- Response
              |
              +-- Status Code
              +-- Headers
              +-- JSON Body
              +-- Text Body
```

---

# 3. APIRequestContext

`APIRequestContext` is Playwright's main API testing interface.

It allows you to make HTTP requests directly.

Example:

```typescript
import { test, expect } from '@playwright/test';

test('GET users API', async ({ request }) => {
  const response = await request.get('https://example.com/api/users');

  expect(response.status()).toBe(200);
});
```

Here:

```typescript
request
```

is an `APIRequestContext` provided by Playwright.

---

# 4. GET Request

A GET request retrieves data from a server.

```typescript
import { test, expect } from '@playwright/test';

test('GET user', async ({ request }) => {
  const response = await request.get(
    'https://example.com/api/users/101'
  );

  expect(response.status()).toBe(200);
});
```

---

# 5. Validate GET Response

You can validate the response body.

```typescript
test('Validate GET response', async ({ request }) => {
  const response = await request.get(
    'https://example.com/api/users/101'
  );

  expect(response.status()).toBe(200);

  const body = await response.json();

  expect(body.id).toBe(101);
  expect(body.name).toBeTruthy();
});
```

---

# 6. Response Status

Use:

```typescript
response.status()
```

Example:

```typescript
const response = await request.get(
  'https://example.com/api/users/101'
);

expect(response.status()).toBe(200);
```

Common HTTP status codes:

| Status | Meaning               |
| ------ | --------------------- |
| 200    | OK                    |
| 201    | Created               |
| 202    | Accepted              |
| 204    | No Content            |
| 400    | Bad Request           |
| 401    | Unauthorized          |
| 403    | Forbidden             |
| 404    | Not Found             |
| 409    | Conflict              |
| 500    | Internal Server Error |
| 502    | Bad Gateway           |
| 503    | Service Unavailable   |

---

# 7. Response OK

Playwright provides:

```typescript
response.ok()
```

Example:

```typescript
expect(response.ok()).toBeTruthy();
```

`ok()` returns `true` when the HTTP status is in the successful range.

Example:

```typescript
test('API response should be successful', async ({ request }) => {
  const response = await request.get(
    'https://example.com/api/users'
  );

  expect(response.ok()).toBeTruthy();
});
```

---

# 8. Response JSON

Use:

```typescript
await response.json()
```

Example:

```typescript
const response = await request.get(
  'https://example.com/api/users/101'
);

const data = await response.json();

console.log(data);
```

Example response:

```json
{
  "id": 101,
  "name": "John",
  "email": "john@example.com"
}
```

Validation:

```typescript
expect(data.id).toBe(101);
expect(data.name).toBe('John');
expect(data.email).toBe('john@example.com');
```

---

# 9. Response Text

Use:

```typescript
await response.text()
```

Example:

```typescript
const response = await request.get(
  'https://example.com/api/message'
);

const body = await response.text();

console.log(body);
```

---

# 10. Response Headers

You can retrieve headers using:

```typescript
response.headers()
```

Example:

```typescript
const headers = response.headers();

console.log(headers);
```

Validate a header:

```typescript
expect(response.headers()['content-type'])
  .toContain('application/json');
```

---

# 11. POST Request

POST is commonly used to create a new resource.

Example:

```typescript
test('Create user', async ({ request }) => {
  const response = await request.post(
    'https://example.com/api/users',
    {
      data: {
        name: 'John',
        email: 'john@example.com'
      }
    }
  );

  expect(response.status()).toBe(201);
});
```

---

# 12. POST Request with JSON

Playwright automatically serializes JavaScript objects supplied through `data`.

```typescript
const response = await request.post(
  'https://example.com/api/users',
  {
    data: {
      name: 'John',
      email: 'john@example.com',
      role: 'QA Engineer'
    }
  }
);
```

---

# 13. POST Request with Headers

```typescript
const response = await request.post(
  'https://example.com/api/users',
  {
    headers: {
      'Content-Type': 'application/json',
      'Authorization': 'Bearer YOUR_TOKEN'
    },
    data: {
      name: 'John',
      email: 'john@example.com'
    }
  }
);
```

---

# 14. PUT Request

PUT is generally used to completely update a resource.

```typescript
test('Update user', async ({ request }) => {
  const response = await request.put(
    'https://example.com/api/users/101',
    {
      data: {
        name: 'John Updated',
        email: 'john.updated@example.com'
      }
    }
  );

  expect(response.status()).toBe(200);
});
```

---

# 15. PATCH Request

PATCH is generally used for a partial update.

```typescript
test('Partially update user', async ({ request }) => {
  const response = await request.patch(
    'https://example.com/api/users/101',
    {
      data: {
        email: 'newemail@example.com'
      }
    }
  );

  expect(response.status()).toBe(200);
});
```

---

# 16. DELETE Request

DELETE removes a resource.

```typescript
test('Delete user', async ({ request }) => {
  const response = await request.delete(
    'https://example.com/api/users/101'
  );

  expect(response.status()).toBe(204);
});
```

---

# 17. Query Parameters

Query parameters can be passed using `params`.

Example:

```typescript
const response = await request.get(
  'https://example.com/api/users',
  {
    params: {
      page: 2,
      limit: 10
    }
  }
);
```

This generates a request similar to:

```text
GET /api/users?page=2&limit=10
```

---

# 18. Multiple Query Parameters

```typescript
const response = await request.get(
  'https://example.com/api/vehicles',
  {
    params: {
      region: 'US',
      modelYear: '2026',
      status: 'ACTIVE'
    }
  }
);
```

---

# 19. Path Parameters

Path parameters are normally inserted directly into the URL.

```typescript
const userId = 101;

const response = await request.get(
  `https://example.com/api/users/${userId}`
);
```

---

# 20. Authentication with Bearer Token

```typescript
const token = 'YOUR_ACCESS_TOKEN';

const response = await request.get(
  'https://example.com/api/profile',
  {
    headers: {
      Authorization: `Bearer ${token}`
    }
  }
);
```

Validate:

```typescript
expect(response.status()).toBe(200);
```

---

# 21. Basic Authentication

Playwright supports HTTP credentials through the request context.

```typescript
const context = await request.newContext({
  httpCredentials: {
    username: 'username',
    password: 'password'
  }
});
```

Then:

```typescript
const response = await context.get(
  'https://example.com/api/users'
);
```

---

# 22. Creating a Custom APIRequestContext

Instead of using the built-in `request` fixture, you can create your own request context.

```typescript
import { test, expect, request } from '@playwright/test';

test('Custom API context', async () => {
  const apiContext = await request.newContext({
    baseURL: 'https://example.com'
  });

  const response = await apiContext.get('/api/users');

  expect(response.status()).toBe(200);

  await apiContext.dispose();
});
```

---

# 23. Base URL

A base URL can be configured in `playwright.config.ts`.

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  use: {
    baseURL: 'https://example.com'
  }
});
```

Then the test becomes:

```typescript
test('GET users', async ({ request }) => {
  const response = await request.get('/api/users');

  expect(response.status()).toBe(200);
});
```

This avoids repeating the domain in every test.

---

# 24. API Request Headers Globally

You can configure common headers.

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  use: {
    baseURL: 'https://example.com',
    extraHTTPHeaders: {
      'Accept': 'application/json'
    }
  }
});
```

---

# 25. Environment Variables

Never hard-code secrets in test code.

Instead of:

```typescript
const token = 'my-secret-token';
```

use:

```typescript
const token = process.env.API_TOKEN;
```

Example:

```typescript
const response = await request.get(
  '/api/profile',
  {
    headers: {
      Authorization: `Bearer ${process.env.API_TOKEN}`
    }
  }
);
```

---

# 26. Environment-Based Configuration

A common framework design is:

```text
.env
.env.qa
.env.stage
.env.prod
```

Example:

```text
BASE_URL=https://qa.example.com
API_TOKEN=xxxxx
```

Test:

```typescript
const response = await request.get('/api/users');
```

Configuration:

```typescript
use: {
  baseURL: process.env.BASE_URL
}
```

---

# 27. API Response Validation

API testing should validate more than just the status code.

Validate:

1. Status code
2. Response headers
3. Response body
4. Required fields
5. Field values
6. Data types
7. Business rules

Example:

```typescript
test('Validate user API', async ({ request }) => {
  const response = await request.get('/api/users/101');

  expect(response.status()).toBe(200);

  const body = await response.json();

  expect(body.id).toBe(101);
  expect(body.name).toBeTruthy();
  expect(body.email).toContain('@');
});
```

---

# 28. Validate JSON Structure

Example:

```typescript
const body = await response.json();

expect(body).toHaveProperty('id');
expect(body).toHaveProperty('name');
expect(body).toHaveProperty('email');
```

---

# 29. Validate Array Response

Suppose the response is:

```json
{
  "users": [
    {
      "id": 1,
      "name": "John"
    },
    {
      "id": 2,
      "name": "David"
    }
  ]
}
```

Test:

```typescript
const body = await response.json();

expect(body.users).toBeInstanceOf(Array);
expect(body.users.length).toBeGreaterThan(0);
```

Validate an item:

```typescript
expect(body.users[0]).toHaveProperty('id');
expect(body.users[0]).toHaveProperty('name');
```

---

# 30. POST Response Validation

```typescript
test('Create user and validate response', async ({ request }) => {
  const payload = {
    name: 'John',
    email: 'john@example.com'
  };

  const response = await request.post(
    '/api/users',
    {
      data: payload
    }
  );

  expect(response.status()).toBe(201);

  const body = await response.json();

  expect(body.name).toBe(payload.name);
  expect(body.email).toBe(payload.email);
  expect(body.id).toBeTruthy();
});
```

---

# 31. API Chaining

API chaining means using the output of one API request in another request.

Example:

```text
POST Create User
       |
       v
Get User ID
       |
       v
GET User
       |
       v
PUT User
       |
       v
DELETE User
```

---

# 32. API Chaining Example

```typescript
test('API chaining', async ({ request }) => {

  const createResponse = await request.post(
    '/api/users',
    {
      data: {
        name: 'John',
        email: 'john@example.com'
      }
    }
  );

  expect(createResponse.status()).toBe(201);

  const createdUser = await createResponse.json();

  const userId = createdUser.id;

  const getResponse = await request.get(
    `/api/users/${userId}`
  );

  expect(getResponse.status()).toBe(200);

  const user = await getResponse.json();

  expect(user.id).toBe(userId);
});
```

---

# 33. API Chaining with Update

```typescript
test('Create and update user', async ({ request }) => {

  const createResponse = await request.post(
    '/api/users',
    {
      data: {
        name: 'John',
        email: 'john@example.com'
      }
    }
  );

  expect(createResponse.status()).toBe(201);

  const createdUser = await createResponse.json();

  const userId = createdUser.id;

  const updateResponse = await request.put(
    `/api/users/${userId}`,
    {
      data: {
        name: 'John Updated',
        email: 'john.updated@example.com'
      }
    }
  );

  expect(updateResponse.status()).toBe(200);

  const updatedUser = await updateResponse.json();

  expect(updatedUser.name).toBe('John Updated');
});
```

---

# 34. API + UI Testing

One of the biggest advantages of Playwright is combining API and UI testing.

Example:

```text
API
 |
 +-- Create test user
 |
 v
UI
 |
 +-- Login
 +-- Validate user
 |
 v
API
 |
 +-- Delete test user
```

This can make tests faster because test data can be created through APIs instead of through the UI.

---

# 35. Create Test Data Through API

Example:

```typescript
test('Create test data through API', async ({ request, page }) => {

  const response = await request.post(
    '/api/users',
    {
      data: {
        username: 'testuser',
        password: 'Password123'
      }
    }
  );

  expect(response.status()).toBe(201);

  await page.goto('/login');

  await page.getByLabel('Username').fill('testuser');
  await page.getByLabel('Password').fill('Password123');

  await page.getByRole('button', {
    name: 'Login'
  }).click();

  await expect(page).toHaveURL(/dashboard/);
});
```

---

# 36. API Cleanup After UI Test

API can also be used for cleanup.

```typescript
test('UI test with API cleanup', async ({ request, page }) => {

  const createResponse = await request.post(
    '/api/users',
    {
      data: {
        username: 'testuser',
        password: 'Password123'
      }
    }
  );

  const user = await createResponse.json();

  await page.goto('/login');

  // UI test steps

  const deleteResponse = await request.delete(
    `/api/users/${user.id}`
  );

  expect(deleteResponse.status()).toBe(204);
});
```

---

# 37. API Fixtures

For a framework, API operations can be placed inside fixtures.

Example:

```typescript
import {
  test as base,
  APIRequestContext
} from '@playwright/test';

type Fixtures = {
  api: APIRequestContext;
};

export const test = base.extend<Fixtures>({
  api: async ({ request }, use) => {
    await use(request);
  }
});
```

Test:

```typescript
import { expect } from '@playwright/test';
import { test } from './fixtures';

test('API test using fixture', async ({ api }) => {

  const response = await api.get('/api/users');

  expect(response.status()).toBe(200);
});
```

---

# 38. API Utility Class

For larger frameworks, API operations can be placed inside a reusable class.

Example:

```typescript
import { APIRequestContext } from '@playwright/test';

export class UserApi {

  constructor(
    private request: APIRequestContext
  ) {}

  async getUser(userId: number) {
    return await this.request.get(
      `/api/users/${userId}`
    );
  }

  async createUser(data: object) {
    return await this.request.post(
      '/api/users',
      {
        data
      }
    );
  }

  async updateUser(userId: number, data: object) {
    return await this.request.put(
      `/api/users/${userId}`,
      {
        data
      }
    );
  }

  async deleteUser(userId: number) {
    return await this.request.delete(
      `/api/users/${userId}`
    );
  }
}
```

---

# 39. Using the API Utility Class

```typescript
import { test, expect } from '@playwright/test';
import { UserApi } from './UserApi';

test('User API using utility class', async ({ request }) => {

  const userApi = new UserApi(request);

  const response = await userApi.getUser(101);

  expect(response.status()).toBe(200);
});
```

---

# 40. API Client Pattern

A better framework structure can separate:

```text
API Tests
   |
   v
API Client
   |
   v
API Endpoints
   |
   v
APIRequestContext
```

Example:

```text
api/
  clients/
    UserApi.ts
    VehicleApi.ts
    AuthApi.ts

  models/
    User.ts
    Vehicle.ts

  tests/
    UserApi.spec.ts
    VehicleApi.spec.ts
```

---

# 41. Request Payload Model

Instead of defining large payloads directly inside tests, create TypeScript interfaces.

```typescript
export interface UserRequest {
  name: string;
  email: string;
  role: string;
}
```

Then:

```typescript
const user: UserRequest = {
  name: 'John',
  email: 'john@example.com',
  role: 'QA'
};
```

Use it:

```typescript
const response = await request.post(
  '/api/users',
  {
    data: user
  }
);
```

---

# 42. Response Model

Create an interface:

```typescript
export interface UserResponse {
  id: number;
  name: string;
  email: string;
  role: string;
}
```

Then:

```typescript
const body = await response.json() as UserResponse;

expect(body.id).toBeTruthy();
expect(body.name).toBe('John');
```

---

# 43. Authentication API

A common automation framework first authenticates using an API.

Example:

```typescript
const loginResponse = await request.post(
  '/api/login',
  {
    data: {
      username: 'testuser',
      password: 'Password123'
    }
  }
);

expect(loginResponse.status()).toBe(200);

const loginBody = await loginResponse.json();

const token = loginBody.token;
```

Then:

```typescript
const response = await request.get(
  '/api/profile',
  {
    headers: {
      Authorization: `Bearer ${token}`
    }
  }
);
```

---

# 44. Reusable Authentication Helper

```typescript
import { APIRequestContext } from '@playwright/test';

export async function getAuthToken(
  request: APIRequestContext
): Promise<string> {

  const response = await request.post(
    '/api/login',
    {
      data: {
        username: process.env.API_USERNAME,
        password: process.env.API_PASSWORD
      }
    }
  );

  if (!response.ok()) {
    throw new Error(
      `Authentication failed: ${response.status()}`
    );
  }

  const body = await response.json();

  return body.token;
}
```

Usage:

```typescript
test('Authenticated API test', async ({ request }) => {

  const token = await getAuthToken(request);

  const response = await request.get(
    '/api/profile',
    {
      headers: {
        Authorization: `Bearer ${token}`
      }
    }
  );

  expect(response.status()).toBe(200);
});
```

---

# 45. API Request Context with Default Headers

You can create a request context with default headers.

```typescript
const apiContext = await request.newContext({
  baseURL: 'https://example.com',
  extraHTTPHeaders: {
    Accept: 'application/json',
    Authorization: `Bearer ${process.env.API_TOKEN}`
  }
});
```

Now requests automatically use those headers.

```typescript
const response = await apiContext.get('/api/users');
```

---

# 46. Ignore HTTPS Errors

For test environments with certificates that are not trusted by the machine, you may configure:

```typescript
const apiContext = await request.newContext({
  baseURL: 'https://qa.example.com',
  ignoreHTTPSErrors: true
});
```

Use this carefully.

It is generally better to fix certificate configuration rather than disable certificate verification in production-like environments.

---

# 47. API Request Timeout

You can configure timeout:

```typescript
const apiContext = await request.newContext({
  timeout: 30000
});
```

Or configure it in the Playwright configuration:

```typescript
export default defineConfig({
  use: {
    actionTimeout: 30000
  }
});
```

For API-specific contexts, prefer configuring the request context appropriately for the API behavior being tested.

---

# 48. Handling API Errors

Do not only test successful scenarios.

Test:

```text
200 - Successful request
201 - Resource created
400 - Invalid request
401 - Missing/invalid authentication
403 - Forbidden
404 - Resource not found
409 - Conflict
500 - Server error
```

Example:

```typescript
test('Invalid user should return 404', async ({ request }) => {

  const response = await request.get(
    '/api/users/999999'
  );

  expect(response.status()).toBe(404);
});
```

---

# 49. Negative API Testing

Example:

```typescript
test('Create user with invalid payload', async ({ request }) => {

  const response = await request.post(
    '/api/users',
    {
      data: {
        name: ''
      }
    }
  );

  expect(response.status()).toBe(400);

  const body = await response.json();

  expect(body.message).toBeTruthy();
});
```

---

# 50. API Contract Validation

Contract testing verifies that an API response follows the expected structure.

Example:

```typescript
const body = await response.json();

expect(body).toHaveProperty('id');
expect(body).toHaveProperty('name');
expect(body).toHaveProperty('email');
```

For more complex APIs, schema validation can be added using libraries such as Zod or Ajv.

Example with Zod:

```typescript
import { z } from 'zod';

const UserSchema = z.object({
  id: z.number(),
  name: z.string(),
  email: z.string().email()
});

const body = await response.json();

UserSchema.parse(body);
```

---

# 51. API Test Hooks

You can use `beforeEach` to create common data.

```typescript
test.beforeEach(async ({ request }) => {

  const response = await request.post(
    '/api/users',
    {
      data: {
        name: 'Test User',
        email: 'test@example.com'
      }
    }
  );

  expect(response.status()).toBe(201);
});
```

---

# 52. beforeAll for Authentication

Authentication can sometimes be performed once before a group of tests.

```typescript
let token: string;

test.beforeAll(async ({ request }) => {

  const response = await request.post(
    '/api/login',
    {
      data: {
        username: process.env.API_USERNAME,
        password: process.env.API_PASSWORD
      }
    }
  );

  const body = await response.json();

  token = body.token;
});
```

Then:

```typescript
test('Get profile', async ({ request }) => {

  const response = await request.get(
    '/api/profile',
    {
      headers: {
        Authorization: `Bearer ${token}`
      }
    }
  );

  expect(response.status()).toBe(200);
});
```

When using shared authentication state, make sure tests do not accidentally become dependent on each other.

---

# 53. API Tests with Test Data

Test data should ideally be separated from test logic.

Example:

```text
test-data/
  users.json
  vehicles.json
  authentication.json
```

Example `users.json`:

```json
{
  "validUser": {
    "name": "John",
    "email": "john@example.com",
    "role": "QA"
  }
}
```

Test:

```typescript
import users from '../test-data/users.json';

test('Create user using test data', async ({ request }) => {

  const response = await request.post(
    '/api/users',
    {
      data: users.validUser
    }
  );

  expect(response.status()).toBe(201);
});
```

---

# 54. API Testing with Data-Driven Tests

Example:

```typescript
const users = [
  {
    name: 'John',
    email: 'john@example.com'
  },
  {
    name: 'David',
    email: 'david@example.com'
  },
  {
    name: 'Robert',
    email: 'robert@example.com'
  }
];

for (const user of users) {

  test(`Create user ${user.name}`, async ({ request }) => {

    const response = await request.post(
      '/api/users',
      {
        data: user
      }
    );

    expect(response.status()).toBe(201);
  });
}
```

---

# 55. API Testing and Page Object Model

Page Object Model is primarily for UI interactions.

API operations should generally use a separate API client/service layer.

Recommended architecture:

```text
Playwright Framework
|
+-- pages/
|     LoginPage.ts
|     HomePage.ts
|     VehiclePage.ts
|
+-- api/
|     AuthApi.ts
|     UserApi.ts
|     VehicleApi.ts
|
+-- tests/
|     login.spec.ts
|     user-api.spec.ts
|     vehicle-api.spec.ts
|
+-- fixtures/
|     testFixtures.ts
|
+-- test-data/
|     users.json
|     vehicles.json
|
+-- utils/
      Logger.ts
      TestData.ts
```

---

# 56. API + Page Object Model

A UI test can combine API utilities and Page Objects.

```typescript
test('API setup + UI validation', async ({
  request,
  page
}) => {

  const userApi = new UserApi(request);

  const response = await userApi.createUser({
    name: 'John',
    email: 'john@example.com',
    role: 'QA'
  });

  expect(response.status()).toBe(201);

  const user = await response.json();

  await page.goto('/users');

  await expect(
    page.getByText(user.name)
  ).toBeVisible();
});
```

---

# 57. API Test Organization

A professional project can use:

```text
playwright-api/
|
+-- tests/
|    +-- api/
|         +-- users.spec.ts
|         +-- vehicles.spec.ts
|         +-- auth.spec.ts
|
+-- api/
|    +-- clients/
|    |     +-- UserApi.ts
|    |     +-- VehicleApi.ts
|    |     +-- AuthApi.ts
|    |
|    +-- models/
|          +-- User.ts
|          +-- Vehicle.ts
|
+-- fixtures/
|
+-- test-data/
|
+-- utils/
|
+-- playwright.config.ts
+-- package.json
```

---

# 58. Complete API Test Example

```typescript
import { test, expect } from '@playwright/test';

test.describe('User API Tests', () => {

  let userId: number;

  test('Create user', async ({ request }) => {

    const response = await request.post(
      '/api/users',
      {
        data: {
          name: 'John',
          email: 'john@example.com',
          role: 'QA'
        }
      }
    );

    expect(response.status()).toBe(201);

    const body = await response.json();

    expect(body.name).toBe('John');
    expect(body.email).toBe('john@example.com');

    userId = body.id;
  });

  test('Get user', async ({ request }) => {

    const response = await request.get(
      `/api/users/${userId}`
    );

    expect(response.status()).toBe(200);

    const body = await response.json();

    expect(body.id).toBe(userId);
  });

  test('Update user', async ({ request }) => {

    const response = await request.put(
      `/api/users/${userId}`,
      {
        data: {
          name: 'John Updated',
          email: 'john.updated@example.com',
          role: 'Senior QA'
        }
      }
    );

    expect(response.status()).toBe(200);

    const body = await response.json();

    expect(body.name).toBe('John Updated');
  });

  test('Delete user', async ({ request }) => {

    const response = await request.delete(
      `/api/users/${userId}`
    );

    expect(response.status()).toBe(204);
  });

});
```

Important:

The above tests share `userId`, so they are dependent on execution order. In a real framework, avoid this design when tests need to run independently.

A better approach is to create and clean up test data within each test or use fixtures.

---

# 59. Independent API Test Example

```typescript
test('Get existing user', async ({ request }) => {

  const userId = 101;

  const response = await request.get(
    `/api/users/${userId}`
  );

  expect(response.status()).toBe(200);

  const body = await response.json();

  expect(body.id).toBe(userId);
});
```

This test can run independently.

---

# 60. API Test with Setup and Cleanup

```typescript
test('User lifecycle test', async ({ request }) => {

  // Setup
  const createResponse = await request.post(
    '/api/users',
    {
      data: {
        name: 'Automation User',
        email: 'automation@example.com'
      }
    }
  );

  expect(createResponse.status()).toBe(201);

  const user = await createResponse.json();

  try {

    // Test
    const getResponse = await request.get(
      `/api/users/${user.id}`
    );

    expect(getResponse.status()).toBe(200);

    const getBody = await getResponse.json();

    expect(getBody.name).toBe('Automation User');

  } finally {

    // Cleanup
    const deleteResponse = await request.delete(
      `/api/users/${user.id}`
    );

    expect([200, 204]).toContain(
      deleteResponse.status()
    );
  }
});
```

---

# 61. API Testing Best Practices

## Use a Base URL

Prefer:

```typescript
baseURL: 'https://qa.example.com'
```

instead of repeating:

```typescript
https://qa.example.com/api/users
```

---

## Do Not Hard-Code Credentials

Avoid:

```typescript
username: 'admin'
password: 'password123'
```

Use environment variables:

```typescript
process.env.API_USERNAME
process.env.API_PASSWORD
```

---

## Validate the Complete Response

Do not only validate:

```typescript
expect(response.status()).toBe(200);
```

Also validate:

```typescript
const body = await response.json();

expect(body.id).toBeTruthy();
expect(body.name).toBeTruthy();
expect(body.email).toContain('@');
```

---

## Keep API Logic Reusable

Avoid duplicating:

```typescript
request.post(...)
request.get(...)
request.put(...)
```

throughout hundreds of tests.

Create API client classes.

---

## Keep Tests Independent

Avoid:

```text
Test 1 creates user
        |
Test 2 uses user
        |
Test 3 deletes user
```

Prefer:

```text
Test 1 -> setup -> test -> cleanup

Test 2 -> setup -> test -> cleanup

Test 3 -> setup -> test -> cleanup
```

---

## Clean Up Test Data

Delete users, vehicles, subscriptions, or other records created by automation whenever possible.

---

# 62. API Testing vs UI Testing

| API Testing              | UI Testing                            |
| ------------------------ | ------------------------------------- |
| Faster                   | Slower                                |
| No browser required      | Browser required                      |
| Tests backend            | Tests user interface                  |
| Easy test data setup     | UI setup may be slower                |
| Easy response validation | Visual/UI validation                  |
| Less flaky               | Can be more susceptible to UI changes |
| HTTP-level validation    | End-to-end validation                 |

A strong automation framework should use both.

---

# 63. Playwright API Testing vs Postman

| Feature              | Playwright API  | Postman                          |
| -------------------- | --------------- | -------------------------------- |
| API requests         | Yes             | Yes                              |
| Automated assertions | Yes             | Yes                              |
| UI testing           | Yes             | Limited                          |
| API + UI flow        | Excellent       | Limited                          |
| TypeScript           | Excellent       | Scripts                          |
| Test runner          | Playwright Test | Postman/Newman                   |
| Parallel execution   | Yes             | Limited compared with Playwright |
| CI/CD                | Excellent       | Excellent                        |
| Page Object Model    | Yes             | No                               |
| Browser automation   | Yes             | No                               |

Playwright is especially useful when the project needs both API and browser automation in one framework.

---

# 64. API Testing in CI/CD

Playwright API tests can run in CI/CD just like UI tests.

Example:

```bash
npx playwright test tests/api
```

Run a specific API test:

```bash
npx playwright test tests/api/users.spec.ts
```

Run in headed mode when browser testing is involved:

```bash
npx playwright test --headed
```

Run with a specific project:

```bash
npx playwright test --project=chromium
```

API-only tests do not require a visible browser.

---

# 65. API Test Reports

Playwright automatically provides test reporting.

Example:

```bash
npx playwright test tests/api
```

HTML report:

```bash
npx playwright show-report
```

API test results can therefore be included with the same automation reports used for UI tests.

---

# 66. Debugging API Tests

Run with debugging:

```bash
npx playwright test tests/api/users.spec.ts --debug
```

Add logging:

```typescript
console.log('Status:', response.status());

console.log(
  'Response:',
  await response.text()
);
```

You can also log request payloads:

```typescript
const payload = {
  name: 'John',
  email: 'john@example.com'
};

console.log('Request:', payload);

const response = await request.post(
  '/api/users',
  {
    data: payload
  }
);
```

Avoid logging passwords, access tokens, or other secrets.

---

# 67. Common Mistakes

## Mistake 1: Forgetting await

Incorrect:

```typescript
const response = request.get('/api/users');
```

Correct:

```typescript
const response = await request.get('/api/users');
```

---

## Mistake 2: Not Checking Status

Incorrect:

```typescript
const body = await response.json();
```

Better:

```typescript
expect(response.status()).toBe(200);

const body = await response.json();
```

---

## Mistake 3: Hard-Coding Tokens

Avoid:

```typescript
Authorization: 'Bearer abc123'
```

Use:

```typescript
Authorization: `Bearer ${process.env.API_TOKEN}`
```

---

## Mistake 4: Making Tests Dependent

Avoid:

```text
Create test
   ↓
Get test
   ↓
Update test
   ↓
Delete test
```

unless the entire flow is intentionally one lifecycle test.

---

## Mistake 5: Putting Everything in the Test

Avoid large tests containing:

```typescript
request.post(...)
request.get(...)
request.put(...)
request.delete(...)
```

repeated across many files.

Use API client classes.

---

# 68. Recommended Senior-Level Framework

A scalable Playwright API framework can look like:

```text
Playwright Framework
│
├── api
│   ├── clients
│   │   ├── AuthApi.ts
│   │   ├── UserApi.ts
│   │   ├── VehicleApi.ts
│   │   └── SubscriptionApi.ts
│   │
│   └── models
│       ├── User.ts
│       ├── Vehicle.ts
│       └── Subscription.ts
│
├── pages
│   ├── LoginPage.ts
│   ├── HomePage.ts
│   └── VehiclePage.ts
│
├── fixtures
│   └── testFixtures.ts
│
├── tests
│   ├── api
│   │   ├── auth.spec.ts
│   │   ├── users.spec.ts
│   │   └── vehicles.spec.ts
│   │
│   └── ui
│       ├── login.spec.ts
│       └── vehicle.spec.ts
│
├── test-data
│   ├── users.json
│   └── vehicles.json
│
├── utils
│   ├── Logger.ts
│   └── TestData.ts
│
├── playwright.config.ts
├── package.json
└── tsconfig.json
```

---

# 69. Important Playwright API Methods

| Method                 | Purpose                    |
| ---------------------- | -------------------------- |
| `request.get()`        | GET request                |
| `request.post()`       | POST request               |
| `request.put()`        | PUT request                |
| `request.patch()`      | PATCH request              |
| `request.delete()`     | DELETE request             |
| `response.status()`    | HTTP status                |
| `response.ok()`        | Successful response check  |
| `response.json()`      | Parse JSON                 |
| `response.text()`      | Get text                   |
| `response.headers()`   | Get response headers       |
| `request.newContext()` | Create API request context |
| `context.dispose()`    | Dispose API context        |

---

# 70. Senior QA Interview Questions

### 1. What is APIRequestContext?

`APIRequestContext` is Playwright's API request interface used to send HTTP requests directly without browser interaction.

### 2. Can Playwright test REST APIs?

Yes.

Playwright supports:

```text
GET
POST
PUT
PATCH
DELETE
```

### 3. Can Playwright combine API and UI testing?

Yes. API calls can prepare data, UI can validate behavior, and API calls can clean up test data.

### 4. How do you send a POST request?

```typescript
await request.post('/api/users', {
  data: {
    name: 'John'
  }
});
```

### 5. How do you validate response JSON?

```typescript
const body = await response.json();

expect(body.id).toBeTruthy();
```

### 6. How do you pass query parameters?

```typescript
await request.get('/api/users', {
  params: {
    page: 1,
    limit: 10
  }
});
```

### 7. How do you pass authentication?

```typescript
headers: {
  Authorization: `Bearer ${token}`
}
```

### 8. How do you avoid hard-coded credentials?

Use environment variables or a secure CI/CD secret store.

### 9. How do you create reusable API methods?

Create API client/service classes.

### 10. How do you perform API chaining?

Capture the response from one API and use required values in subsequent requests.

### 11. How do you test negative scenarios?

Validate expected error codes and response messages for invalid requests.

### 12. How do you clean up API test data?

Use API DELETE operations or teardown/fixture logic.

### 13. Why use API testing for test data creation?

It is usually faster and less fragile than creating data through multiple UI steps.

### 14. Should API tests depend on each other?

Generally no. Independent tests are easier to debug and safely parallelize.

### 15. How would you design a senior-level Playwright API framework?

Use:

```text
API Clients
+
Request/Response Models
+
Fixtures
+
Environment Configuration
+
Test Data
+
Authentication Utilities
+
API Tests
+
CI/CD
+
Reporting
```

---

# 71. Final Recommended Pattern

For a professional Playwright automation framework:

```text
                 Playwright
                     |
          +----------+----------+
          |                     |
         UI                    API
          |                     |
    Page Objects          API Clients
          |                     |
          +----------+----------+
                     |
                  Fixtures
                     |
                Test Data
                     |
              Environment
                     |
                  CI/CD
```

The goal is not to replace UI automation with API automation.

The strongest framework uses the right layer for the right purpose:

```text
API -> Fast setup, backend validation, test data

UI -> User journey and frontend validation

API + UI -> Complete end-to-end validation
```

---

# 72. Key Takeaways

* Playwright supports API testing through `APIRequestContext`.
* API tests do not require a browser.
* Use `GET`, `POST`, `PUT`, `PATCH`, and `DELETE`.
* Validate status codes and response bodies.
* Use `params` for query parameters.
* Use `headers` for authentication and other HTTP headers.
* Use `data` for request payloads.
* Use environment variables for secrets.
* Create reusable API client classes.
* Use fixtures for shared API functionality.
* Use TypeScript interfaces/models for request and response data.
* Use API calls to create and clean up test data.
* Combine API and UI testing when appropriate.
* Keep tests independent whenever possible.
* Design API automation as a separate service/client layer rather than putting API logic directly into every test.
* API testing is an important part of a senior-level Playwright automation framework.
