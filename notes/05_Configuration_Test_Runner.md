### **Chapter 5: Configuration & Test Runner**

#### **5.1 playwright.config.ts (or config.js)**

The `playwright.config.ts` (or `.js`) file serves as the central configuration point for your Playwright project. It defines how tests are executed, which browsers to use, timeouts, retries, parallelism, and other global test behaviors.
By maintaining configurations here, you ensure consistency across your entire test suite and avoid hardcoding repetitive setups in each test file.

**Key Configuration Options:**

* **testDir** – Directory where Playwright looks for test files (default: `tests`).
* **timeout** – Maximum time a test can take before being considered failed.
* **expect** – Configure assertion-related settings like `timeout` for `expect` checks.
* **fullyParallel** – Run all tests in parallel; speeds up large test suites.
* **forbidOnly** – Prevent accidentally committed `.only()` tests.
* **reporter** – Defines the output format for test results (e.g., `list`, `html`, `json`).
* **projects** – Define multiple browser configurations (Chromium, Firefox, WebKit).
* **use** – Default options for all tests such as `browserName`, `headless`, `viewport`, `screenshot`, and `video`.

**Example `playwright.config.ts`:**

```ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  timeout: 30 * 1000,
  retries: 2,
  reporter: [['list'], ['html', { open: 'never' }]],
  use: {
    headless: true,
    screenshot: 'on',
    video: 'retain-on-failure',
    baseURL: 'https://example.com',
  },
  projects: [
    {
      name: 'Chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'Firefox',
      use: { ...devices['Desktop Firefox'] },
    },
  ],
});
```

---

#### **5.2 Test Runner Options: Parallelism, Retries, Reporters**

Playwright comes with a built-in **test runner** designed to handle modern testing workflows efficiently.
It supports advanced configurations like **parallel execution**, **automatic retries**, **test isolation**, and **custom reporters** for detailed analysis.

**Parallelism**

* Playwright runs tests in **workers**, isolated environments that can execute tests concurrently.
* Parallelism improves test speed dramatically, especially in CI pipelines.
* You can control it using:

  ```bash
  npx playwright test --workers=4
  ```

  or via `workers` property in the config.

**Retries**

* The `retries` option automatically re-runs failed tests to handle flakiness caused by network delays or timing issues.
* Configure globally or per project:

  ```ts
  retries: 2
  ```

**Reporters**

* Reporters provide feedback on test results in various formats.
* Common reporters:

  * `list` – Minimal console output.
  * `line` – Compact single-line output.
  * `dot` – Dots for passed/failed tests.
  * `html` – Generates detailed HTML reports.
  * `json` – Structured output for integrations.
  * `junit` – For CI systems like Jenkins.

Example:

```bash
npx playwright test --reporter=html
```

Playwright also supports **custom reporters**, allowing you to define your own logic for logging, analytics, or external integrations.

---

#### **5.3 Fixtures & Custom Hooks**

**Fixtures** in Playwright are reusable setups that initialize test environments, shared data, or configurations. They promote **DRY (Don’t Repeat Yourself)** principles and enable powerful test isolation mechanisms.

A **fixture** can represent anything:

* A browser context or page.
* A mock API setup.
* A database connection.
* User authentication.

Playwright provides **built-in fixtures** such as `page`, `context`, and `browser`, but you can also **define custom fixtures**.

**Example: Creating a Custom Fixture**

```ts
import { test as base } from '@playwright/test';

type MyFixtures = {
  adminPage: Page;
};

const test = base.extend<MyFixtures>({
  adminPage: async ({ page }, use) => {
    await page.goto('/admin');
    await use(page);
  },
});

test('admin dashboard test', async ({ adminPage }) => {
  await adminPage.click('#settings');
});
```

This creates an isolated `adminPage` fixture available for specific tests, improving readability and maintainability.

---

**Hooks (Lifecycle Events)**
Playwright provides lifecycle hooks that let you define setup and teardown logic around your tests:

* `beforeAll` – Runs once before all tests in a file.
* `afterAll` – Runs once after all tests in a file.
* `beforeEach` – Runs before every test.
* `afterEach` – Runs after every test.

**Example:**

```ts
import { test } from '@playwright/test';

test.beforeAll(async () => {
  console.log('Setting up environment...');
});

test.afterAll(async () => {
  console.log('Cleaning up environment...');
});

test.beforeEach(async ({ page }) => {
  await page.goto('https://example.com/login');
});

test('User login test', async ({ page }) => {
  await page.fill('#username', 'sameer');
  await page.fill('#password', 'securePass');
  await page.click('button[type="submit"]');
});
```

Hooks, combined with fixtures, allow you to:

* Handle setup/cleanup efficiently.
* Reduce redundancy.
* Isolate each test context for maximum reliability.

---

#### **5.4 Environment Variables and Test Context Management**

Playwright tests often rely on dynamic configurations like environment URLs, credentials, or tokens.
These are best managed using **environment variables** or `.env` files.

**Using Environment Variables:**

```bash
BASE_URL=https://staging.example.com
```

Access in Playwright:

```ts
const baseUrl = process.env.BASE_URL;
await page.goto(`${baseUrl}/login`);
```

**Custom Test Contexts**

* You can define multiple contexts (e.g., staging vs production) in `projects`.
* Each context can have its own baseURL, user credentials, or device emulation.

---

#### **5.5 Command Line Interface (CLI) Options**

Playwright’s CLI allows developers to control tests, manage browsers, and debug failures efficiently.

**Common Commands:**

```bash
npx playwright test                  # Run all tests
npx playwright test example.spec.ts  # Run specific test
npx playwright show-report           # Open HTML report
npx playwright codegen https://example.com  # Launch inspector/codegen
npx playwright install               # Install browsers
```

**Flags:**

* `--project` – Run specific browser/project.
* `--headed` – Run in non-headless mode.
* `--grep "text"` – Filter tests by name or tag.
* `--ui` – Open Playwright test explorer interface.

These CLI utilities make Playwright flexible for both development and CI/CD environments.

---

#### **5.6 Continuous Integration (CI) Configurations**

Playwright integrates seamlessly with CI providers like **GitHub Actions**, **GitLab CI**, **Jenkins**, and **Azure DevOps**.

**Key CI Practices:**

* Run tests in **headless mode**.
* Use **retries** and **trace uploads** for debugging.
* Store HTML or trace reports as CI artifacts.

**Example (GitHub Actions):**

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
      - run: npx playwright test
      - uses: actions/upload-artifact@v3
        with:
          name: playwright-report
          path: playwright-report/
```

---

#### **Summary**

* `playwright.config.ts` defines global test behavior and environment setup.
* The built-in test runner manages **parallel execution**, **retries**, and **reporting**.
* **Fixtures** and **hooks** help modularize and isolate test logic.
* **Environment variables** enhance flexibility for multi-environment setups.
* Robust **CLI tools** and **CI integration** make Playwright suitable for both small teams and enterprise-scale testing pipelines.

