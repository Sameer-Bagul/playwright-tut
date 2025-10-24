## **Chapter 2: Getting Started / Installation**

### **2.1 Installation for Node / JavaScript / TypeScript**

**Overview:**
Playwright is primarily distributed as an NPM package. It supports JavaScript and TypeScript out of the box.
The Playwright Test runner (`@playwright/test`) provides everything you need — test structure, assertions, reporting, parallel execution, and configuration.

**Installation Command:**

```bash
# For a new project
npm init playwright@latest
```

This interactive CLI helps set up:

* Project structure
* Test examples
* Browsers (Chromium, Firefox, WebKit)
* Optional GitHub Actions CI workflow

**Manual Installation:**

```bash
npm install -D @playwright/test
npx playwright install
```

**Verify Installation:**

```bash
npx playwright --version
```

**TypeScript Support:**

* TypeScript is built-in — no need for additional typings.
* Default config `playwright.config.ts` uses TypeScript.

**File Structure Example:**

```
my-project/
│
├─ tests/
│   ├─ example.spec.ts
│
├─ playwright.config.ts
├─ package.json
└─ node_modules/
```

---

### **2.2 Setting Up Browsers (Chromium, Firefox, WebKit)**

Playwright supports **three browser engines**, each maintained by the Playwright team:

* **Chromium** (for Chrome, Edge, Opera)
* **Firefox**
* **WebKit** (for Safari)

**Browser Installation:**

```bash
npx playwright install
```

**Install specific browser:**

```bash
npx playwright install chromium
npx playwright install firefox
npx playwright install webkit
```

**Launch Browser Manually:**

```javascript
const { chromium } = require('playwright');

(async () => {
  const browser = await chromium.launch();
  const page = await browser.newPage();
  await page.goto('https://example.com');
  await browser.close();
})();
```

**Headed vs Headless:**

* `launch({ headless: false })` → opens visible browser window
* By default, Playwright runs in headless mode.

---

### **2.3 Project Setup (Folder Structure, Config)**

**Default Structure (Created by CLI):**

```
tests/
  └── example.spec.ts
playwright.config.ts
package.json
```

**playwright.config.ts Example:**

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  retries: 1,
  timeout: 30000,
  use: {
    browserName: 'chromium',
    headless: true,
    viewport: { width: 1280, height: 720 },
    screenshot: 'only-on-failure',
    trace: 'on-first-retry',
  },
});
```

**Key Config Options:**

* `testDir`: location of test files
* `retries`: retry failed tests
* `timeout`: per-test timeout
* `use`: global test options

**Running Tests:**

```bash
npx playwright test
```

**Generate a Report:**

```bash
npx playwright show-report
```

---

### **2.4 Running Tests in Different Modes**

**Run in UI Mode (interactive debugging):**

```bash
npx playwright test --ui
```

Opens a rich UI for debugging, filtering, and rerunning tests.

**Run in Specific Browser:**

```bash
npx playwright test --project=firefox
```

**Run Headed Mode:**

```bash
npx playwright test --headed
```

**Run Only One Test File:**

```bash
npx playwright test tests/example.spec.ts
```

---

### **2.5 Continuous Integration Setup**

**GitHub Actions Example (auto-generated):**

```yaml
name: Playwright Tests
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 18
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npx playwright test
```

**CI Best Practices:**

* Use `--with-deps` for Linux CI runners.
* Cache browser binaries to save build time.
* Use `npx playwright show-report` as an artifact step.

---

### **2.6 Environment Variables & Secrets**

Playwright allows environment-based configuration using `.env` or shell variables.

**Example:**

```bash
export BASE_URL=https://staging.example.com
npx playwright test
```

**Use in Tests:**

```typescript
test('visit base URL', async ({ page }) => {
  await page.goto(process.env.BASE_URL);
});
```

---

### **Summary:**

| Step             | Command                      | Purpose                           |
| ---------------- | ---------------------------- | --------------------------------- |
| Install          | `npm init playwright@latest` | Scaffold a new project            |
| Install browsers | `npx playwright install`     | Download browser binaries         |
| Run tests        | `npx playwright test`        | Execute test suite                |
| Debug            | `npx playwright test --ui`   | Visual test runner                |
| CI setup         | `--with-deps`                | Auto installs dependencies for CI |

