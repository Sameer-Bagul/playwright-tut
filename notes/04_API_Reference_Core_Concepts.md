## **Chapter 4: API Reference & Core Concepts**

---

### **4.1 Browser, BrowserContext, Page**

#### **Concept Overview**

Playwright follows a **3-layer model** of browser automation:

1. **Browser** – Represents an entire browser instance (like Chrome, Firefox, WebKit).
2. **BrowserContext** – Represents an independent session or incognito window inside a browser.
3. **Page** – Represents a single browser tab or page within a context.

This architecture gives you **full test isolation**, control, and parallelization.

---

#### **Browser**

The `Browser` object controls the actual running instance of a browser.

**Example:**

```typescript
import { chromium } from 'playwright';

(async () => {
  const browser = await chromium.launch({ headless: false });
  const context = await browser.newContext();
  const page = await context.newPage();
  await page.goto('https://example.com');
  await browser.close();
})();
```

**Key Methods:**

| Method                      | Description                                                                  |
| --------------------------- | ---------------------------------------------------------------------------- |
| `launch()`                  | Starts a new browser instance                                                |
| `launchPersistentContext()` | Launches a browser with a persistent user profile (like a real user session) |
| `close()`                   | Closes the browser                                                           |

**Options for `launch()`:**

```typescript
await chromium.launch({
  headless: false,
  slowMo: 100,
  args: ['--start-maximized'],
});
```

* `headless`: run browser visibly
* `slowMo`: slows down actions (good for debugging)
* `args`: pass additional browser arguments

---

#### **BrowserContext**

Each context behaves like a separate browser profile — cookies, storage, and cache are **not shared**.

**Example:**

```typescript
const browser = await chromium.launch();
const context1 = await browser.newContext();
const context2 = await browser.newContext();

const page1 = await context1.newPage();
const page2 = await context2.newPage();
```

Each test in Playwright typically runs in its own isolated `BrowserContext`.

**Useful Options:**

```typescript
const context = await browser.newContext({
  viewport: { width: 1280, height: 720 },
  userAgent: 'custom-user-agent',
  permissions: ['geolocation'],
  storageState: 'state.json',
});
```

**Storage State:**
You can save and reuse session data like cookies and local storage:

```typescript
await context.storageState({ path: 'state.json' });
```

Later:

```typescript
await browser.newContext({ storageState: 'state.json' });
```

---

#### **Page**

A `Page` represents a browser tab.

**Basic Usage:**

```typescript
const page = await context.newPage();
await page.goto('https://example.com');
await page.click('text=Login');
await page.fill('#username', 'sameer');
await page.fill('#password', 'pass123');
await page.click('button:has-text("Submit")');
```

**Key Methods:**

| Method                  | Description                     |
| ----------------------- | ------------------------------- |
| `goto(url)`             | Navigates to a URL              |
| `click(selector)`       | Clicks an element               |
| `fill(selector, value)` | Fills input fields              |
| `title()`               | Returns page title              |
| `screenshot()`          | Captures a screenshot           |
| `waitForSelector()`     | Waits for an element to appear  |
| `evaluate()`            | Runs JavaScript inside the page |

**Screenshot Example:**

```typescript
await page.screenshot({ path: 'homepage.png', fullPage: true });
```

**Evaluate Example (Run JS in the DOM):**

```typescript
const headline = await page.evaluate(() => document.querySelector('h1').textContent);
```

---

### **4.2 Locator API (getByRole, filter, chaining)**

Playwright’s **Locator API** is one of its most powerful features — designed to make element selection **robust and resilient**.

#### **Locator Creation**

```typescript
const locator = page.getByRole('button', { name: 'Sign in' });
await locator.click();
```

Playwright waits automatically for:

* Element to be attached to DOM
* Element to be visible
* Element to be stable (no animation)
* Element to be enabled

#### **Locator Methods**

| Method              | Description                             |
| ------------------- | --------------------------------------- |
| `locator(selector)` | Generic CSS or XPath selector           |
| `getByRole()`       | Semantic locator using ARIA role        |
| `getByText()`       | Finds elements by visible text          |
| `getByLabel()`      | For input fields with associated labels |
| `filter()`          | Narrows down locators                   |
| `nth(index)`        | Picks a specific element among many     |
| `locator.first()`   | Selects the first matching element      |

**Example:**

```typescript
const products = page.locator('.product');
await products.nth(0).click(); // Click first product
```

#### **Locator Chaining**

```typescript
const menu = page.locator('.nav');
await menu.getByRole('link', { name: 'Docs' }).click();
```

#### **Locator Filters**

```typescript
const button = page.getByRole('button').filter({ hasText: 'Save' });
await button.click();
```

---

### **4.3 Page Methods (goto, reload, screenshot, evaluate)**

The `Page` API is the main interface for browser automation.

#### **Navigation**

```typescript
await page.goto('https://example.com');
await page.reload();
await page.goBack();
await page.goForward();
```

#### **Waiting**

Playwright auto-waits, but explicit waiting can be added:

```typescript
await page.waitForURL(/.*dashboard/);
await page.waitForSelector('#welcome');
```

#### **Screenshots & PDFs**

```typescript
await page.screenshot({ path: 'screenshot.png', fullPage: true });
await page.pdf({ path: 'page.pdf', format: 'A4' });
```

#### **JavaScript Execution**

Run arbitrary JavaScript in the page:

```typescript
const title = await page.evaluate(() => document.title);
```

Pass arguments:

```typescript
const headline = await page.evaluate((sel) => {
  return document.querySelector(sel).textContent;
}, 'h1');
```

#### **Dialogs**

Handle alerts, confirms, and prompts:

```typescript
page.on('dialog', async dialog => {
  console.log(dialog.message());
  await dialog.accept('OK');
});
await page.click('#triggerAlert');
```

---

### **4.4 Network/Request Interception & Mocking**

Playwright allows **complete network control** — intercepting requests, mocking responses, or blocking domains.

#### **Listening to Network Events**

```typescript
page.on('request', request => console.log('Request:', request.url()));
page.on('response', response => console.log('Response:', response.status()));
```

#### **Intercept & Modify Requests**

```typescript
await page.route('**/api/*', async route => {
  const request = route.request();
  if (request.url().includes('user')) {
    await route.fulfill({
      status: 200,
      contentType: 'application/json',
      body: JSON.stringify({ id: 1, name: 'Sameer' }),
    });
  } else {
    await route.continue();
  }
});
```

#### **Block Unwanted Content**

```typescript
await page.route('**/*.png', route => route.abort());
await page.route('**/*.jpg', route => route.abort());
```

#### **Network Assertions**

```typescript
const [response] = await Promise.all([
  page.waitForResponse('**/api/login'),
  page.click('#loginBtn'),
]);
expect(response.ok()).toBeTruthy();
```

---

### **4.5 Frames, iFrames, Multiple Tabs & Windows**

#### **Frames**

Access nested frames and iFrames using `frame()` or `frameLocator()`.

```typescript
const frame = page.frame({ name: 'loginFrame' });
await frame.fill('#username', 'sameer');
```

Or using frame locator (recommended):

```typescript
const frameLocator = page.frameLocator('#frameId');
await frameLocator.locator('button#submit').click();
```

**Advantages of frameLocator:**

* Auto-waits for the frame to load.
* More resilient and reliable.

---

#### **Multiple Tabs / Windows**

Playwright treats every new tab as a `Page` object.

**Example:**

```typescript
const [newPage] = await Promise.all([
  context.waitForEvent('page'),
  page.click('a[target="_blank"]'),
]);
await newPage.waitForLoadState();
console.log(await newPage.title());
```

**All Pages in a Context:**

```typescript
const pages = context.pages();
console.log('Open tabs:', pages.length);
```

**Popup Handling:**

```typescript
page.on('popup', async popup => {
  await popup.waitForLoadState();
  console.log(await popup.url());
});
```

---

### **4.6 Handling Downloads & Uploads**

**Downloads:**

```typescript
const [ download ] = await Promise.all([
  page.waitForEvent('download'),
  page.click('a#downloadLink'),
]);
await download.saveAs('file.zip');
```

**Uploads:**

```typescript
const input = page.locator('input[type="file"]');
await input.setInputFiles('resume.pdf');
```

---

### **4.7 Keyboard, Mouse, and Touch Actions**

**Keyboard:**

```typescript
await page.keyboard.type('Hello World!');
await page.keyboard.press('Enter');
```

**Mouse:**

```typescript
await page.mouse.move(100, 200);
await page.mouse.down();
await page.mouse.up();
```

**Touch (Mobile Simulation):**

```typescript
const context = await browser.newContext({ hasTouch: true });
const page = await context.newPage();
```

---

### **4.8 Events and Listeners**

Playwright’s event-driven model lets you observe browser behavior in real-time.

**Common Events:**

| Event         | Trigger                          |
| ------------- | -------------------------------- |
| `'console'`   | When a console message is logged |
| `'dialog'`    | Alert/Confirm/Prompt appears     |
| `'request'`   | A network request is made        |
| `'response'`  | Response received                |
| `'pageerror'` | JS error on page                 |
| `'popup'`     | New window/tab opened            |

**Example:**

```typescript
page.on('console', msg => console.log(msg.text()));
page.on('pageerror', err => console.error('Error:', err.message));
```

---

### **4.9 Context & Page Lifecycle**

Playwright has multiple lifecycle states:

| Object  | Lifecycle Method           | Description              |
| ------- | -------------------------- | ------------------------ |
| Browser | `launch()` / `close()`     | Start or stop browser    |
| Context | `newContext()` / `close()` | Manage isolated sessions |
| Page    | `newPage()` / `close()`    | Manage tabs/pages        |

Playwright ensures **graceful cleanup** after each test when using the built-in test runner.

---

### **Summary Table**

| Concept             | Description                        | Typical Usage          |
| ------------------- | ---------------------------------- | ---------------------- |
| **Browser**         | Launches actual browser instance   | `chromium.launch()`    |
| **BrowserContext**  | Isolated environment/session       | `browser.newContext()` |
| **Page**            | Represents a tab                   | `context.newPage()`    |
| **Locator**         | Resilient element selector         | `page.getByRole()`     |
| **Network Control** | Mock, intercept, or block requests | `page.route()`         |
| **Frames**          | Handle iframes                     | `page.frameLocator()`  |
| **Events**          | Observe browser/page actions       | `page.on('console')`   |

