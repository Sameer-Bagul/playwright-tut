## **Chapter 7: Advanced Topics**

---

### **7.1 Code Generation and Inspector**

Playwright provides tools to **generate code automatically** and **inspect page behavior** in real-time. These are invaluable for learning, prototyping, or debugging complex interactions.

#### **Codegen (Automatic Test Generation)**

`playwright codegen` allows you to **record user interactions** in a browser and generate Playwright scripts automatically.

**Usage:**

```bash
npx playwright codegen https://example.com
```

**Features:**

* Automatically generates `click`, `fill`, `select`, and navigation commands.
* Generates in **TypeScript, JavaScript, Python, or Java**.
* Supports **locator selectors**, reducing brittle CSS usage.

**Example Output:**

```ts
await page.goto('https://example.com');
await page.getByRole('textbox', { name: 'Username' }).fill('sameer');
await page.getByRole('button', { name: 'Login' }).click();
```

**Benefits:**

* Speeds up test creation.
* Reduces manual coding errors.
* Helps beginners understand Playwright APIs.

---

#### **Inspector**

The **Playwright Inspector** provides a visual interface to **debug tests step by step**.

**Launching Inspector:**

```bash
npx playwright test --debug
```

**Inspector Features:**

* Pause and resume test execution.
* Highlight elements on hover.
* Step through commands interactively.
* Modify locators or test logic on the fly.

**Use Cases:**

* Debugging flaky tests.
* Fine-tuning locators for dynamic pages.
* Learning Playwright by observing browser behavior.

---

### **7.2 Tracing, Video, and Screenshot Capture**

Playwright’s built-in tools for **recording execution** are essential for diagnosing failures and creating reliable tests.

#### **Tracing**

Tracing captures **all actions, network events, DOM snapshots, and screenshots** during a test.

**Enable Tracing:**

```ts
await page.tracing.start({ screenshots: true, snapshots: true });
await page.goto('https://example.com');
await page.click('text=Login');
await page.tracing.stop({ path: 'trace.zip' });
```

**View Trace:**

```bash
npx playwright show-trace trace.zip
```

**Use Cases:**

* Identify why a test failed.
* Review step-by-step browser interactions.
* Debug flakiness in CI pipelines.

---

#### **Video Recording**

* Playwright can record **full-page video** per test or per context.
* Configurable in `playwright.config.ts`:

```ts
use: {
  video: 'retain-on-failure',
}
```

* Videos are stored in the test output folder and can be reviewed when tests fail.

---

#### **Screenshots**

* Capture **entire pages or specific elements**.

```ts
await page.screenshot({ path: 'homepage.png', fullPage: true });
await page.locator('#loginForm').screenshot({ path: 'login.png' });
```

* Combine with `expect` for **visual regression** comparisons.

---

### **7.3 Visual Regression Testing**

Playwright supports **automated visual testing** to detect UI changes over time.

**Steps:**

1. Capture baseline screenshot:

```ts
await expect(page.locator('#header')).toHaveScreenshot('header.png');
```

2. Compare in future runs:

* Playwright detects pixel differences.
* Alerts developers to unintended UI changes.

**Options for Comparison:**

* `threshold` – Allow minor pixel differences.
* `fullPage` – Compare entire page or only specific elements.

**Example Config:**

```ts
await expect(page).toHaveScreenshot({
  path: 'homepage.png',
  fullPage: true,
  threshold: 0.1,
});
```

**Benefits:**

* Detects regressions that functional tests cannot catch.
* Ensures consistent visual quality across browsers.

---

### **7.4 Multi-Browser & Cross-Platform Testing**

Playwright’s **multi-browser support** allows testing across:

* Chromium (Chrome, Edge)
* Firefox
* WebKit (Safari)
* Mobile emulation (iPhone, Android)

#### **Projects in Config**

```ts
projects: [
  { name: 'Chrome', use: { ...devices['Desktop Chrome'] } },
  { name: 'Firefox', use: { ...devices['Desktop Firefox'] } },
  { name: 'WebKit', use: { ...devices['Desktop Safari'] } },
  { name: 'iPhone 14', use: { ...devices['iPhone 14'] } },
]
```

**Use Cases:**

* Validate cross-browser compatibility.
* Test responsiveness on multiple devices.
* Ensure consistent behavior across environments.

**Parallel Execution:**

* Projects can run **simultaneously**, leveraging Playwright’s **parallel worker system**.

---

### **7.5 Integration with CI/CD**

Playwright integrates seamlessly with **Continuous Integration / Continuous Deployment** pipelines to automate testing as part of the development workflow.

#### **Common CI/CD Integrations**

* GitHub Actions
* GitLab CI
* Jenkins
* CircleCI
* Azure DevOps

#### **CI Best Practices**

1. **Headless Mode** – Run browsers without GUI.
2. **Artifacts** – Save screenshots, videos, and traces for failed tests.
3. **Environment Variables** – Configure URLs, credentials, and API endpoints dynamically.
4. **Retries** – Enable retries to reduce false negatives due to network or timing issues.
5. **Parallelism** – Split tests across multiple workers to reduce build times.

**GitHub Actions Example:**

```yaml
name: Playwright Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 18
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npx playwright test --workers=4 --reporter=html
      - uses: actions/upload-artifact@v3
        with:
          name: playwright-report
          path: playwright-report/
```

---

### **7.6 Mobile and Geolocation Testing**

Playwright supports **device emulation**, including touch events, screen size, and geolocation.

```ts
const context = await browser.newContext({
  ...devices['Pixel 5'],
  geolocation: { latitude: 37.7749, longitude: -122.4194 },
  permissions: ['geolocation'],
});
const page = await context.newPage();
await page.goto('https://maps.example.com');
```

**Benefits:**

* Test location-based features.
* Simulate mobile interactions (touch, gestures, viewport sizes).
* Run parallel tests on multiple device emulations.

---

### **7.7 Network Throttling and Mocking**

Playwright allows advanced network control:

* **Throttle network conditions** for slow connections.
* **Mock API responses** for stable test environments.

**Network Throttling Example:**

```ts
await page.emulateNetworkConditions({
  offline: false,
  downloadThroughput: 500 * 1024 / 8,
  uploadThroughput: 500 * 1024 / 8,
  latency: 100
});
```

**API Mocking:**

```ts
await page.route('**/api/users', route =>
  route.fulfill({ status: 200, body: JSON.stringify([{ id:1, name:'Sam' }]) })
);
```

* Reduces test flakiness caused by external services.
* Simulates edge cases and error handling scenarios.

---

### **7.8 Debugging Advanced Failures**

#### **Using Traces**

* Inspect failed tests step-by-step.
* Review screenshots, DOM snapshots, and network activity.

#### **Pausing Tests**

```ts
await page.pause();
```

* Opens Playwright Inspector mid-test.
* Allows manual interaction with the page to diagnose issues.

#### **Verbose Logging**

```ts
DEBUG=pw:api npx playwright test
```

* Logs detailed Playwright API calls.
* Useful for debugging complex interactions.

---

### **7.9 Combining Features for Enterprise-Grade Automation**

**Scenario:** Test a cross-browser, mobile-responsive checkout flow with API mocking, tracing, and video capture.

```ts
const context = await browser.newContext({
  ...devices['iPhone 14'],
  storageState: 'state.json',
  video: 'retain-on-failure'
});

await page.route('**/api/cart', route => route.fulfill({ status: 200, body: '[]' }));
await page.goto('/checkout');
await page.tracing.start({ screenshots: true, snapshots: true });
await page.fill('#card', '4111 1111 1111 1111');
await page.click('button#pay');
await page.tracing.stop({ path: 'checkout-trace.zip' });
```

* Ensures **cross-browser, multi-device coverage**.
* Reduces dependency on live APIs.
* Provides **trace + video for debugging**.

---

### **7.10 Summary**

**Advanced Playwright Features Include:**

* **Code Generation** – Rapid test creation and learning.
* **Inspector** – Interactive debugging of tests.
* **Tracing, Video, Screenshots** – Diagnose flaky tests or failures.
* **Visual Regression** – Detect unintended UI changes.
* **Multi-browser & device testing** – Ensure cross-platform compatibility.
* **CI/CD Integration** – Automated testing for continuous delivery.
* **Network control & mocking** – Simulate edge cases and offline scenarios.
* **Enterprise-grade workflows** – Combine features for scalable, reliable automation.

