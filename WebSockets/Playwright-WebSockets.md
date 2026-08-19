# Playwright WebSockets

## Overview

WebSockets provide a persistent, two-way communication channel between a client and a server over a single connection.

Unlike traditional HTTP request/response communication, WebSockets allow the server to push messages to the browser without the browser continuously polling the server.

Playwright provides WebSocket monitoring capabilities through the `page.on('websocket')` event.

---

## Table of Contents

1. [What Are WebSockets?](#what-are-websockets)
2. [WebSocket Communication Flow](#websocket-communication-flow)
3. [Why Test WebSockets?](#why-test-websockets)
4. [Playwright WebSocket Support](#playwright-websocket-support)
5. [Listening for WebSocket Connections](#listening-for-websocket-connections)
6. [Monitoring WebSocket Events](#monitoring-websocket-events)
7. [Monitoring Sent Messages](#monitoring-sent-messages)
8. [Monitoring Received Messages](#monitoring-received-messages)
9. [Waiting for a WebSocket](#waiting-for-a-websocket)
10. [Capturing WebSocket Messages](#capturing-websocket-messages)
11. [Filtering WebSocket Connections](#filtering-websocket-connections)
12. [Complete WebSocket Monitoring Example](#complete-websocket-monitoring-example)
13. [Testing Real-Time Applications](#testing-real-time-applications)
14. [WebSocket Authentication](#websocket-authentication)
15. [WebSocket Testing with Fixtures](#websocket-testing-with-fixtures)
16. [Using WebSocket Data in Assertions](#using-websocket-data-in-assertions)
17. [Handling JSON Messages](#handling-json-messages)
18. [Handling Binary Messages](#handling-binary-messages)
19. [WebSocket Lifecycle](#websocket-lifecycle)
20. [Multiple WebSocket Connections](#multiple-websocket-connections)
21. [WebSocket Debugging](#websocket-debugging)
22. [WebSocket Testing Best Practices](#websocket-testing-best-practices)
23. [Common Mistakes](#common-mistakes)
24. [Interview Questions](#interview-questions)
25. [Summary](#summary)

---

# What Are WebSockets?

WebSocket is a communication protocol that provides full-duplex communication between a client and server.

The connection starts as an HTTP request and is then upgraded to a WebSocket connection.

Typical WebSocket URL:

```text
ws://example.com/socket
```

Secure WebSocket:

```text
wss://example.com/socket
```

### HTTP vs WebSocket

Traditional HTTP:

```text
Browser ---> Request ---> Server
Browser <--- Response --- Server
```

WebSocket:

```text
Browser <=================> Server
        Persistent Connection
```

Both sides can send messages independently.

---

# WebSocket Communication Flow

A typical WebSocket connection works like this:

```text
Browser
   |
   | HTTP Upgrade Request
   v
Server
   |
   | 101 Switching Protocols
   v
WebSocket Connection
   |
   +---- Client Message ---->
   |
   <---- Server Message ----+
   |
   +---- Client Message ---->
   |
   <---- Server Message ----+
   |
   | Close
   v
Connection Closed
```

---

# Why Test WebSockets?

WebSockets are commonly used in:

* Chat applications
* Trading applications
* Stock price dashboards
* Live sports applications
* Notifications
* Multiplayer games
* Collaboration tools
* Real-time dashboards
* Delivery tracking
* IoT applications
* Customer support applications

A UI may look correct while the underlying WebSocket communication is broken.

Therefore, testing WebSocket traffic can help verify:

* Connection establishment
* Server messages
* Client messages
* Message format
* Authentication
* Connection closure
* Reconnection
* Real-time updates

---

# Playwright WebSocket Support

Playwright exposes WebSocket connections through the `page.on('websocket')` event.

Basic syntax:

```javascript
page.on('websocket', websocket => {
    console.log(`WebSocket URL: ${websocket.url()}`);
});
```

A WebSocket object can expose events such as:

```javascript
websocket.on('framesent', event => {
    // Client sent a message
});

websocket.on('framereceived', event => {
    // Server sent a message
});

websocket.on('close', () => {
    // WebSocket closed
});

websocket.on('socketerror', error => {
    // WebSocket error
});
```

---

# Listening for WebSocket Connections

The simplest approach is to register a listener before navigating to the application.

```javascript
import { test } from '@playwright/test';

test('monitor WebSocket connection', async ({ page }) => {

    page.on('websocket', websocket => {
        console.log('WebSocket connected:');
        console.log(websocket.url());
    });

    await page.goto('https://example.com');

});
```

### Important

Register the listener before the action that creates the WebSocket connection.

Correct:

```javascript
page.on('websocket', websocket => {
    console.log(websocket.url());
});

await page.goto('https://example.com');
```

Potentially incorrect:

```javascript
await page.goto('https://example.com');

page.on('websocket', websocket => {
    console.log(websocket.url());
});
```

The WebSocket may already have been created before the listener was registered.

---

# Monitoring WebSocket Events

You can monitor the WebSocket lifecycle.

```javascript
page.on('websocket', websocket => {

    console.log(`Connected: ${websocket.url()}`);

    websocket.on('framesent', event => {
        console.log('Frame sent:', event.payload);
    });

    websocket.on('framereceived', event => {
        console.log('Frame received:', event.payload);
    });

    websocket.on('close', () => {
        console.log('WebSocket closed');
    });

    websocket.on('socketerror', error => {
        console.log('WebSocket error:', error);
    });

});
```

This is useful for debugging real-time applications.

---

# Monitoring Sent Messages

The `framesent` event can be used to monitor messages sent from the browser to the server.

```javascript
page.on('websocket', websocket => {

    websocket.on('framesent', event => {
        console.log('Sent:', event.payload);
    });

});
```

Example output:

```text
Sent: {"type":"subscribe","channel":"orders"}
```

---

# Monitoring Received Messages

The `framereceived` event can be used to monitor messages received from the server.

```javascript
page.on('websocket', websocket => {

    websocket.on('framereceived', event => {
        console.log('Received:', event.payload);
    });

});
```

Example:

```text
Received: {"type":"orderUpdate","status":"SHIPPED"}
```

---

# Waiting for a WebSocket

Sometimes the test needs to wait until a WebSocket connection is created.

Playwright supports waiting for events using `waitForEvent()`.

```javascript
const websocketPromise = page.waitForEvent('websocket');

await page.goto('https://example.com');

const websocket = await websocketPromise;

console.log('Connected:', websocket.url());
```

This is useful when the WebSocket is created during page initialization.

---

# Waiting for a WebSocket During a User Action

Suppose clicking a button creates a WebSocket connection.

```javascript
const websocketPromise = page.waitForEvent('websocket');

await page.getByRole('button', { name: 'Connect' }).click();

const websocket = await websocketPromise;

console.log(websocket.url());
```

This is a reliable synchronization pattern.

---

# Waiting for a Specific WebSocket

You can use a predicate to wait for a particular WebSocket.

```javascript
const websocketPromise = page.waitForEvent('websocket', {
    predicate: websocket =>
        websocket.url().includes('/notifications')
});

await page.goto('https://example.com');

const websocket = await websocketPromise;

console.log('Notification WebSocket connected');
```

This is useful when an application opens several WebSocket connections.

---

# Capturing WebSocket Messages

You can store received messages in an array.

```javascript
const messages = [];

page.on('websocket', websocket => {

    websocket.on('framereceived', event => {
        messages.push(event.payload);
    });

});

await page.goto('https://example.com');
```

Later:

```javascript
console.log(messages);
```

You can then validate the messages.

```javascript
expect(messages.length).toBeGreaterThan(0);
```

---

# Filtering WebSocket Connections

Applications may use multiple WebSockets.

For example:

```text
wss://example.com/chat
wss://example.com/notifications
wss://example.com/analytics
```

You can filter the desired connection.

```javascript
page.on('websocket', websocket => {

    if (websocket.url().includes('/notifications')) {

        websocket.on('framereceived', event => {
            console.log('Notification:', event.payload);
        });

    }

});
```

---

# Complete WebSocket Monitoring Example

```javascript
import { test, expect } from '@playwright/test';

test('monitor WebSocket traffic', async ({ page }) => {

    const receivedMessages = [];

    page.on('websocket', websocket => {

        console.log(`WebSocket connected: ${websocket.url()}`);

        websocket.on('framesent', event => {
            console.log('Sent:', event.payload);
        });

        websocket.on('framereceived', event => {

            console.log('Received:', event.payload);

            receivedMessages.push(event.payload);
        });

        websocket.on('close', () => {
            console.log('WebSocket closed');
        });

        websocket.on('socketerror', error => {
            console.log('WebSocket error:', error);
        });

    });

    await page.goto('https://example.com');

    await page.waitForTimeout(3000);

    expect(receivedMessages.length).toBeGreaterThan(0);
});
```

---

# Testing Real-Time Applications

Consider a chat application.

The user sends:

```text
Hello
```

The application may send:

```json
{
    "type": "message",
    "text": "Hello"
}
```

The server may return:

```json
{
    "type": "message",
    "text": "Hello",
    "status": "delivered"
}
```

Playwright can monitor the underlying traffic.

```javascript
test('verify chat WebSocket message', async ({ page }) => {

    let receivedMessage;

    page.on('websocket', websocket => {

        websocket.on('framereceived', event => {

            try {
                const data = JSON.parse(event.payload);

                if (data.type === 'message') {
                    receivedMessage = data;
                }

            } catch {
                // Ignore non-JSON messages
            }

        });

    });

    await page.goto('https://example.com/chat');

    await page.getByPlaceholder('Message').fill('Hello');

    await page.getByRole('button', { name: 'Send' }).click();

    await page.waitForTimeout(2000);

    expect(receivedMessage.text).toBe('Hello');
});
```

---

# WebSocket Authentication

WebSockets may require authentication.

Common approaches include:

* Cookies
* Authorization tokens
* Session tokens
* Query parameters
* Headers
* Authentication performed during the initial HTTP handshake

For example, the application might establish authentication using a browser session.

Playwright can reuse authenticated browser state.

```javascript
import { test } from '@playwright/test';

test.use({
    storageState: 'playwright/.auth/user.json'
});

test('authenticated WebSocket', async ({ page }) => {

    page.on('websocket', websocket => {

        console.log('Authenticated WebSocket:', websocket.url());

        websocket.on('framereceived', event => {
            console.log(event.payload);
        });

    });

    await page.goto('https://example.com/dashboard');

});
```

---

# WebSocket Testing with Fixtures

A custom fixture can be useful when multiple tests need WebSocket monitoring.

```javascript
import { test as base } from '@playwright/test';

export const test = base.extend({

    websocketMessages: async ({ page }, use) => {

        const messages = [];

        page.on('websocket', websocket => {

            websocket.on('framereceived', event => {
                messages.push(event.payload);
            });

        });

        await use(messages);
    }

});
```

Test:

```javascript
import { test, expect } from './fixtures';

test('verify WebSocket message', async ({
    page,
    websocketMessages
}) => {

    await page.goto('https://example.com');

    await page.waitForTimeout(2000);

    expect(websocketMessages.length).toBeGreaterThan(0);
});
```

---

# Using WebSocket Data in Assertions

You can assert specific message content.

```javascript
test('validate notification message', async ({ page }) => {

    let notification;

    page.on('websocket', websocket => {

        websocket.on('framereceived', event => {

            try {

                const data = JSON.parse(event.payload);

                if (data.type === 'notification') {
                    notification = data;
                }

            } catch {
                // Ignore invalid JSON
            }

        });

    });

    await page.goto('https://example.com');

    await page.waitForTimeout(3000);

    expect(notification).toBeDefined();
    expect(notification.type).toBe('notification');
});
```

---

# Handling JSON Messages

Most WebSocket applications exchange JSON.

Example:

```json
{
    "event": "vehicleStatus",
    "vin": "123456789",
    "status": "ONLINE"
}
```

Parse the message:

```javascript
websocket.on('framereceived', event => {

    try {

        const data = JSON.parse(event.payload);

        console.log(data.event);
        console.log(data.vin);
        console.log(data.status);

    } catch (error) {

        console.log('Message is not JSON');

    }

});
```

---

# Validating JSON Fields

```javascript
websocket.on('framereceived', event => {

    const data = JSON.parse(event.payload);

    expect(data.event).toBe('vehicleStatus');
    expect(data.status).toBe('ONLINE');

});
```

You can also use partial validation:

```javascript
expect(data).toEqual(
    expect.objectContaining({
        event: 'vehicleStatus',
        status: 'ONLINE'
    })
);
```

---

# Handling Binary Messages

WebSocket frames can contain binary data.

Depending on the application, binary messages may represent:

* Images
* Audio
* Video
* Compressed data
* Protocol buffers
* Custom binary protocols

When working with binary payloads, inspect the payload type and convert it appropriately before validating it.

For example, a binary payload may need to be decoded using:

```javascript
const buffer = Buffer.from(payload);
```

The exact decoding approach depends on the application's WebSocket protocol.

---

# WebSocket Lifecycle

A WebSocket generally follows this lifecycle:

```text
CONNECTING
     |
     v
 OPEN
     |
     +--------> MESSAGE
     |
     +--------> MESSAGE
     |
     v
 CLOSING
     |
     v
 CLOSED
```

Playwright can monitor important lifecycle events.

```javascript
page.on('websocket', websocket => {

    console.log('WebSocket opened');

    websocket.on('framereceived', event => {
        console.log('Message received');
    });

    websocket.on('framesent', event => {
        console.log('Message sent');
    });

    websocket.on('close', () => {
        console.log('WebSocket closed');
    });

});
```

---

# Multiple WebSocket Connections

A modern application may have several WebSocket connections.

```javascript
const sockets = [];

page.on('websocket', websocket => {

    sockets.push(websocket);

    console.log('WebSocket:', websocket.url());

});
```

You can identify them:

```javascript
for (const websocket of sockets) {

    console.log(websocket.url());

}
```

---

# Monitoring Specific Connections

```javascript
page.on('websocket', websocket => {

    const url = websocket.url();

    if (url.includes('/chat')) {

        websocket.on('framereceived', event => {
            console.log('Chat:', event.payload);
        });

    }

    if (url.includes('/notifications')) {

        websocket.on('framereceived', event => {
            console.log('Notification:', event.payload);
        });

    }

});
```

---

# WebSocket Debugging

WebSocket monitoring is especially useful during debugging.

Example:

```javascript
page.on('websocket', websocket => {

    console.log('URL:', websocket.url());

    websocket.on('framesent', event => {
        console.log('[SEND]', event.payload);
    });

    websocket.on('framereceived', event => {
        console.log('[RECEIVE]', event.payload);
    });

    websocket.on('socketerror', error => {
        console.log('[ERROR]', error);
    });

    websocket.on('close', () => {
        console.log('[CLOSE]');
    });

});
```

Example output:

```text
URL: wss://example.com/socket
[SEND] {"type":"subscribe","channel":"orders"}
[RECEIVE] {"type":"connected"}
[RECEIVE] {"type":"orderUpdate","status":"SHIPPED"}
[CLOSE]
```

This can quickly identify whether the problem is:

* Connection
* Client message
* Server response
* Authentication
* Server error
* Unexpected disconnect

---

# Using Playwright Tracing

Tracing can be combined with WebSocket debugging.

```javascript
await context.tracing.start({
    screenshots: true,
    snapshots: true
});
```

Perform the test:

```javascript
await page.goto('https://example.com');

await page.getByRole('button', { name: 'Connect' }).click();
```

Stop tracing:

```javascript
await context.tracing.stop({
    path: 'trace.zip'
});
```

Tracing helps correlate UI actions with network activity and application behavior.

---

# WebSocket Testing with Network Events

Playwright's network APIs and WebSocket events can be used together.

```javascript
page.on('request', request => {
    console.log('Request:', request.url());
});

page.on('response', response => {
    console.log('Response:', response.status(), response.url());
});

page.on('websocket', websocket => {
    console.log('WebSocket:', websocket.url());
});
```

This gives visibility into:

```text
HTTP Requests
      |
      v
Authentication
      |
      v
WebSocket Connection
      |
      v
Real-Time Messages
```

---

# WebSocket Connection Failure

You can monitor socket errors.

```javascript
page.on('websocket', websocket => {

    websocket.on('socketerror', error => {

        console.error(
            `WebSocket error for ${websocket.url()}:`,
            error
        );

    });

});
```

This is useful for diagnosing:

* Server unavailable
* Authentication failure
* Network problems
* TLS problems
* Unexpected server shutdown

---

# Testing WebSocket Reconnection

Real-time applications often reconnect automatically.

A basic test strategy:

```javascript
test('verify WebSocket reconnection', async ({ page }) => {

    const connections = [];

    page.on('websocket', websocket => {

        connections.push(websocket.url());

        console.log(
            'WebSocket connection:',
            websocket.url()
        );

        websocket.on('close', () => {
            console.log('WebSocket closed');
        });

    });

    await page.goto('https://example.com');

    await page.waitForTimeout(5000);

    console.log('Connections:', connections);

    expect(connections.length).toBeGreaterThan(0);
});
```

For a production-grade test, the test should deliberately trigger or simulate the relevant failure condition and then verify that a new connection is established.

---

# WebSocket Message Filtering

Applications often send many different event types.

Example:

```javascript
websocket.on('framereceived', event => {

    try {

        const data = JSON.parse(event.payload);

        switch (data.type) {

            case 'notification':
                console.log('Notification:', data);
                break;

            case 'orderUpdate':
                console.log('Order:', data);
                break;

            case 'heartbeat':
                console.log('Heartbeat received');
                break;

            default:
                console.log('Unknown event:', data);
        }

    } catch {
        console.log('Non-JSON frame');
    }

});
```

---

# Waiting for a Specific WebSocket Message

A reusable helper can make tests cleaner.

```javascript
async function waitForWebSocketMessage(
    page,
    predicate,
    timeout = 10000
) {

    return new Promise((resolve, reject) => {

        const timer = setTimeout(() => {
            reject(
                new Error(
                    'Timed out waiting for WebSocket message'
                )
            );
        }, timeout);

        const handler = websocket => {

            websocket.on('framereceived', event => {

                try {

                    const data = JSON.parse(event.payload);

                    if (predicate(data)) {

                        clearTimeout(timer);
                        resolve(data);
                    }

                } catch {
                    // Ignore non-JSON frames
                }

            });

        };

        page.on('websocket', handler);
    });
}
```

Usage:

```javascript
const messagePromise = waitForWebSocketMessage(
    page,
    data => data.type === 'orderUpdate'
);

await page.goto('https://example.com');

const message = await messagePromise;

console.log(message);
```

---

# WebSocket Test Helper

A reusable helper class can centralize WebSocket monitoring.

```javascript
export class WebSocketMonitor {

    constructor(page) {

        this.page = page;
        this.messages = [];
        this.connections = [];

        this.page.on('websocket', websocket => {

            this.connections.push(websocket);

            websocket.on('framereceived', event => {

                this.messages.push({
                    url: websocket.url(),
                    payload: event.payload
                });

            });

        });

    }

    getMessages() {
        return this.messages;
    }

    getConnections() {
        return this.connections;
    }

}
```

Test:

```javascript
import { test, expect } from '@playwright/test';
import { WebSocketMonitor } from './WebSocketMonitor';

test('monitor application WebSocket', async ({ page }) => {

    const monitor = new WebSocketMonitor(page);

    await page.goto('https://example.com');

    await page.waitForTimeout(3000);

    expect(monitor.getConnections().length)
        .toBeGreaterThan(0);

    console.log(monitor.getMessages());
});
```

---

# Page Object Model with WebSocket Monitoring

WebSocket monitoring can also be separated into a utility or service class instead of placing network logic directly in page objects.

Example:

```javascript
export class DashboardPage {

    constructor(page) {

        this.page = page;

        this.refreshButton =
            page.getByRole('button', {
                name: 'Refresh'
            });
    }

    async open() {

        await this.page.goto(
            'https://example.com/dashboard'
        );

    }

    async refresh() {

        await this.refreshButton.click();

    }

}
```

WebSocket monitoring can remain in a dedicated utility:

```javascript
export class WebSocketMonitor {

    constructor(page) {

        this.messages = [];

        page.on('websocket', websocket => {

            websocket.on('framereceived', event => {

                this.messages.push(event.payload);

            });

        });

    }

    messagesReceived() {
        return this.messages;
    }

}
```

This keeps responsibilities separated.

---

# WebSocket Test Data

WebSocket tests may require test data such as:

```javascript
const expectedEvent = {
    type: 'vehicleStatus',
    status: 'ONLINE'
};
```

Then:

```javascript
const actual = JSON.parse(event.payload);

expect(actual).toEqual(
    expect.objectContaining(expectedEvent)
);
```

For larger projects, keep WebSocket test data separate:

```text
test-data/
    websocket/
        vehicle-status.json
        notifications.json
        orders.json
```

---

# WebSocket and API Chaining

A common enterprise scenario is:

```text
API
 |
 | Create Order
 v
Order ID
 |
 v
WebSocket
 |
 | Order Status Update
 v
UI
```

The test can:

1. Create an entity through an API.
2. Capture the generated ID.
3. Open the UI.
4. Monitor WebSocket traffic.
5. Wait for an event containing that ID.
6. Validate the event.
7. Validate the UI.

Example:

```javascript
const orderId = 'ORDER-12345';

let orderUpdate;

page.on('websocket', websocket => {

    websocket.on('framereceived', event => {

        try {

            const data = JSON.parse(event.payload);

            if (
                data.type === 'orderUpdate' &&
                data.orderId === orderId
            ) {
                orderUpdate = data;
            }

        } catch {
            // Ignore invalid messages
        }

    });

});
```

Then:

```javascript
expect(orderUpdate).toBeDefined();
expect(orderUpdate.orderId).toBe(orderId);
```

---

# Real-World Example: Vehicle Status

A connected vehicle application may receive real-time vehicle events.

Example:

```json
{
    "event": "vehicleStatus",
    "vin": "123456789",
    "status": "ONLINE",
    "timestamp": "2026-08-19T19:30:00Z"
}
```

Test:

```javascript
test('verify vehicle online event', async ({ page }) => {

    let vehicleEvent;

    page.on('websocket', websocket => {

        if (!websocket.url().includes('/vehicle')) {
            return;
        }

        websocket.on('framereceived', event => {

            try {

                const data = JSON.parse(event.payload);

                if (
                    data.event === 'vehicleStatus' &&
                    data.status === 'ONLINE'
                ) {
                    vehicleEvent = data;
                }

            } catch {
                // Ignore invalid payload
            }

        });

    });

    await page.goto(
        'https://example.com/vehicle'
    );

    await page.waitForTimeout(5000);

    expect(vehicleEvent).toBeDefined();
    expect(vehicleEvent.status).toBe('ONLINE');

});
```

---

# WebSocket vs HTTP API Testing

WebSocket testing and API testing solve different problems.

| Feature            | HTTP API            | WebSocket              |
| ------------------ | ------------------- | ---------------------- |
| Communication      | Request/response    | Full duplex            |
| Connection         | Usually per request | Persistent             |
| Server push        | No                  | Yes                    |
| Real-time          | Limited             | Excellent              |
| Typical use        | CRUD APIs           | Live updates           |
| Playwright support | `request`           | `page.on('websocket')` |
| Typical validation | Status/body         | Frames/events          |

A complete automation framework may need both.

---

# WebSocket vs Playwright APIRequest

Playwright API testing:

```javascript
const response = await request.get(
    'https://example.com/api/orders'
);

expect(response.ok()).toBeTruthy();
```

WebSocket monitoring:

```javascript
page.on('websocket', websocket => {

    websocket.on('framereceived', event => {

        console.log(event.payload);

    });

});
```

Use API testing for:

* REST endpoints
* CRUD operations
* Request/response validation

Use WebSocket monitoring for:

* Real-time messages
* Server push
* Live updates
* Connection behavior

---

# Best Practices

## 1. Register listeners before navigation

```javascript
page.on('websocket', websocket => {
    // Monitor
});

await page.goto(url);
```

---

## 2. Filter WebSocket URLs

Do not process every WebSocket if the application has many.

```javascript
if (!websocket.url().includes('/orders')) {
    return;
}
```

---

## 3. Avoid arbitrary waits when possible

Instead of:

```javascript
await page.waitForTimeout(5000);
```

Prefer event-based synchronization where practical.

For example:

```javascript
const websocketPromise =
    page.waitForEvent('websocket');

await page.goto(url);

const websocket =
    await websocketPromise;
```

---

## 4. Validate message structure

Do not only check that a message exists.

Weak:

```javascript
expect(message).toBeDefined();
```

Better:

```javascript
expect(message.type).toBe('orderUpdate');
expect(message.status).toBe('SHIPPED');
```

---

## 5. Parse JSON safely

```javascript
try {

    const data = JSON.parse(event.payload);

} catch {

    // Handle non-JSON message
}
```

---

## 6. Keep WebSocket utilities reusable

Instead of repeating:

```javascript
page.on('websocket', ...)
```

in every test, create:

* WebSocket monitor
* WebSocket helper
* Fixture
* Utility class

---

## 7. Correlate messages with test data

For example:

```javascript
data.orderId === expectedOrderId
```

This prevents a test from accidentally validating another event.

---

## 8. Validate both backend event and UI

For real-time applications:

```text
WebSocket Event
      |
      v
Application Processing
      |
      v
UI Update
```

A strong end-to-end test can validate both.

---

# Common Mistakes

## Mistake 1: Registering listener too late

```javascript
await page.goto(url);

page.on('websocket', websocket => {
    // May miss connection
});
```

Better:

```javascript
page.on('websocket', websocket => {
    // Monitor
});

await page.goto(url);
```

---

## Mistake 2: Monitoring every WebSocket

Large applications can generate many WebSocket connections.

Filter by URL:

```javascript
if (websocket.url().includes('/orders')) {
    // Monitor
}
```

---

## Mistake 3: Assuming every frame is JSON

Not every WebSocket frame is JSON.

Use:

```javascript
try {
    const data = JSON.parse(event.payload);
} catch {
    // Non-JSON frame
}
```

---

## Mistake 4: Using long fixed waits

Avoid:

```javascript
await page.waitForTimeout(10000);
```

when an event-based approach is available.

---

## Mistake 5: Validating only UI

If the UI shows:

```text
Order shipped
```

that does not necessarily prove that the correct WebSocket event was received.

Validate the underlying event when WebSocket behavior is part of the requirement.

---

## Mistake 6: Ignoring reconnection

Real-world WebSocket applications can disconnect.

Important scenarios include:

* Network interruption
* Server restart
* Authentication expiration
* Mobile network transition
* Idle timeout

Test reconnection behavior when it is a business requirement.

---

# Recommended WebSocket Test Structure

A scalable project can use:

```text
PlaywrightProject/
│
├── tests/
│   └── websocket/
│       ├── connection.spec.js
│       ├── messages.spec.js
│       ├── notifications.spec.js
│       └── reconnection.spec.js
│
├── utils/
│   └── WebSocketMonitor.js
│
├── fixtures/
│   └── websocket.fixture.js
│
├── test-data/
│   └── websocket/
│       ├── notifications.json
│       └── vehicle-status.json
│
├── pages/
│   └── DashboardPage.js
│
└── playwright.config.js
```

---

# Example Enterprise WebSocket Test

```javascript
import { test, expect } from '@playwright/test';

test.describe('WebSocket Tests', () => {

    test('verify real-time order update', async ({ page }) => {

        let orderUpdate;

        const websocketPromise =
            page.waitForEvent('websocket', {
                predicate: websocket =>
                    websocket.url().includes('/orders')
            });

        await page.goto(
            'https://example.com/orders'
        );

        const websocket =
            await websocketPromise;

        websocket.on('framereceived', event => {

            try {

                const data =
                    JSON.parse(event.payload);

                if (data.type === 'orderUpdate') {
                    orderUpdate = data;
                }

            } catch {
                // Ignore non-JSON frames
            }

        });

        await page.getByRole(
            'button',
            { name: 'Refresh Orders' }
        ).click();

        await expect
            .poll(() => orderUpdate)
            .toBeDefined();

        expect(orderUpdate.type)
            .toBe('orderUpdate');

        expect(orderUpdate.status)
            .toBe('SHIPPED');

    });

});
```

---

# WebSocket Testing Checklist

* Register WebSocket listeners before the connection is created.
* Identify the correct WebSocket URL.
* Monitor `framesent`.
* Monitor `framereceived`.
* Monitor `close`.
* Monitor `socketerror`.
* Parse JSON safely.
* Filter irrelevant messages.
* Correlate events with test data.
* Prefer event-based synchronization.
* Test authentication.
* Test connection failures when required.
* Test reconnection when required.
* Validate important message fields.
* Validate UI updates triggered by WebSocket events.
* Keep WebSocket utilities reusable.
* Avoid unnecessary fixed waits.
* Capture useful logs during failures.
* Combine API, UI, and WebSocket testing for end-to-end scenarios.

---

# Interview Questions

## 1. What is a WebSocket?

A WebSocket is a protocol that provides persistent, full-duplex communication between a client and server.

---

## 2. How does Playwright detect WebSocket connections?

Using:

```javascript
page.on('websocket', websocket => {
    // ...
});
```

---

## 3. How do you monitor messages sent by the browser?

Use:

```javascript
websocket.on('framesent', event => {
    console.log(event.payload);
});
```

---

## 4. How do you monitor messages received from the server?

Use:

```javascript
websocket.on('framereceived', event => {
    console.log(event.payload);
});
```

---

## 5. How do you detect WebSocket closure?

```javascript
websocket.on('close', () => {
    console.log('Closed');
});
```

---

## 6. How do you detect WebSocket errors?

```javascript
websocket.on('socketerror', error => {
    console.log(error);
});
```

---

## 7. How do you wait for a WebSocket connection?

```javascript
const websocketPromise =
    page.waitForEvent('websocket');

await page.goto(url);

const websocket =
    await websocketPromise;
```

---

## 8. How do you wait for a specific WebSocket?

```javascript
const websocket =
    await page.waitForEvent('websocket', {
        predicate: ws =>
            ws.url().includes('/notifications')
    });
```

---

## 9. How do you parse a WebSocket JSON message?

```javascript
const data = JSON.parse(event.payload);
```

---

## 10. What if the WebSocket payload is not JSON?

Handle it safely:

```javascript
try {
    const data = JSON.parse(event.payload);
} catch {
    // Handle non-JSON payload
}
```

---

## 11. Why should the WebSocket listener be registered before navigation?

Because the application may establish the WebSocket during page initialization. Registering the listener first prevents missing the connection.

---

## 12. Can Playwright validate WebSocket messages?

Yes. WebSocket frames can be captured and validated using Playwright's WebSocket events.

---

## 13. How do you test WebSocket reconnection?

Monitor WebSocket creation and closure, trigger the relevant failure condition, and verify that a new WebSocket connection is established.

---

## 14. Can WebSocket testing be combined with API testing?

Yes.

A common enterprise flow is:

```text
API creates data
      ↓
WebSocket receives event
      ↓
UI displays update
      ↓
Playwright validates UI
```

---

## 15. Why is WebSocket testing important for real-time applications?

Because UI validation alone may not prove that the correct real-time event was received, processed, and displayed.

---

# Senior-Level Interview Scenario

### Question

Your application displays vehicle status updates in real time. The UI sometimes shows the wrong status. How would you debug it using Playwright?

### Answer

I would first monitor WebSocket connections:

```javascript
page.on('websocket', websocket => {

    console.log('WebSocket:', websocket.url());

    websocket.on('framesent', event => {
        console.log('Sent:', event.payload);
    });

    websocket.on('framereceived', event => {
        console.log('Received:', event.payload);
    });

    websocket.on('socketerror', error => {
        console.log('Error:', error);
    });

    websocket.on('close', () => {
        console.log('Closed');
    });

});
```

Then I would:

1. Identify the correct WebSocket endpoint.
2. Capture the received vehicle-status event.
3. Parse the payload.
4. Correlate the VIN with the test data.
5. Verify the status received from the server.
6. Verify the UI displays the same status.
7. Check for duplicate or stale events.
8. Check whether the connection was closed or re-established.
9. Review tracing and logs if necessary.

This helps determine whether the problem is in:

```text
Server
   ↓
WebSocket
   ↓
Application State
   ↓
UI
```

---

# WebSocket Automation Architecture

A mature Playwright framework can separate responsibilities:

```text
                 Playwright Test
                       |
        +--------------+--------------+
        |              |              |
        v              v              v
   Page Objects    API Helpers   WebSocket Monitor
        |              |              |
        +--------------+--------------+
                       |
                       v
                Test Assertions
```

This architecture makes real-time tests easier to maintain.

---

# Summary

Playwright provides useful WebSocket monitoring capabilities for real-time application testing.

The most important API is:

```javascript
page.on('websocket', websocket => {
    // WebSocket connection
});
```

Important events include:

```javascript
websocket.on('framesent', ...);
websocket.on('framereceived', ...);
websocket.on('close', ...);
websocket.on('socketerror', ...);
```

A reliable WebSocket test should:

```text
1. Register listener
       ↓
2. Establish connection
       ↓
3. Identify correct WebSocket
       ↓
4. Capture messages
       ↓
5. Parse payload
       ↓
6. Correlate test data
       ↓
7. Assert message
       ↓
8. Assert UI behavior
```

For senior-level automation, WebSocket testing becomes especially powerful when combined with:

* Playwright UI automation
* API testing
* Authentication
* Test fixtures
* Page Object Model
* Network monitoring
* Tracing
* Test data management
* CI/CD execution

The goal is not simply to confirm that a WebSocket exists, but to verify that the **correct real-time event flows from the server through the application and results in the expected user-visible behavior**.
