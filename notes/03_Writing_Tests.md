## **Chapter 3: Writing Tests**

---

### **3.1 First Test Example**

**Goal:** Learn the basic test syntax, structure, and how Playwright executes tests.

**Example:**

```typescript
import { test, expect } from '@playwright/test';

test('homepage has Playwright in title and links to docs', async ({ page }) => {
  await page.goto('https://playwright.dev/');
  await expect(page).toHaveTitle(/Playwright/);
  await page.getByRole('link', { name: 'Get started' }).click();
  await expect(page).toHaveURL(/.*docs/);
});
```

**Key Points:**

* `test()` defines a single test case.
* The `{ page }` object is automatically provided by the Playwright Test fixture system.
* `expect()` is used for assertions.
* Async/await is required — Playwright APIs are asynchronous.

**Running the Test:**

```bash
npx playwright test
```

**Report:**

```bash
npx playwright show-report
```

---

### **3.2 Actions & Locators**

#### **Locators**

Locators are Playwright’s way of finding and interacting with elements reliably.

**Creating Locators:**

```typescript
const loginButton = page.getByRole('button', { name: 'Login' });
await loginButton.click();
```

**Locator Methods:**

| Method         | Example                                    | Description                 |
| -------------- | ------------------------------------------ | --------------------------- |
| `getByRole()`  | `page.getByRole('link', { name: 'Docs' })` | Uses ARIA roles             |
| `getByText()`  | `page.getByText('Submit')`                 | Finds by visible text       |
| `getByLabel()` | `page.getByLabel('Username')`              | Finds form controls         |
| `locator()`    | `page.locator('.btn-primary')`             | Generic locator by selector |
| `filter()`     | `locator.filter({ hasText: 'Save' })`      | Filters matching elements   |

**Locator Chaining:**

```typescript
const row = page.getByRole('row').filter({ hasText: 'Item 1' });
await row.getByRole('button', { name: 'Edit' }).click();
```

#### **Common Actions**

| Action                  | Example                                | Description                 |
| ----------------------- | -------------------------------------- | --------------------------- |
| `click()`               | `await button.click();`                | Click element               |
| `fill()`                | `await input.fill('Sameer');`          | Fill input fields           |
| `check()` / `uncheck()` | `await checkbox.check();`              | Interact with checkboxes    |
| `selectOption()`        | `await select.selectOption('value1');` | Select from dropdown        |
| `hover()`               | `await button.hover();`                | Hover mouse over an element |
| `type()`                | `await input.type('Hello');`           | Type slowly (char-by-char)  |
| `press()`               | `await page.press('Enter');`           | Send keyboard keys          |

---

### **3.3 Assertions (Using expect)**

Playwright’s built-in assertion library (`@playwright/test`) provides auto-waiting and retry mechanisms.

**Examples:**

```typescript
await expect(page).toHaveTitle(/Playwright/);
await expect(page.locator('h1')).toHaveText('Getting Started');
await expect(page.locator('#submit')).toBeEnabled();
await expect(page.locator('button')).toHaveCount(3);
```

**Soft Assertions:**
Used when you want to continue test execution even if an assertion fails.

```typescript
await expect.soft(page.locator('h1')).toHaveText('Welcome');
```

**Negated Assertions:**

```typescript
await expect(page.locator('button')).not.toBeDisabled();
```

**Common Assertions:**

| Assertion         | Description           |
| ----------------- | --------------------- |
| `toHaveText()`    | Verifies visible text |
| `toHaveURL()`     | Verifies current URL  |
| `toBeVisible()`   | Element is visible    |
| `toBeHidden()`    | Element not visible   |
| `toBeChecked()`   | Checkbox is checked   |
| `toContainText()` | Partial text match    |

---

### **3.4 Test Isolation & Hooks (beforeEach, afterEach)**

Playwright creates a **new browser context per test** — ensuring full isolation (no cookies, local storage, etc. shared).

#### **Example:**

```typescript
import { test, expect } from '@playwright/test';

test.beforeEach(async ({ page }) => {
  await page.goto('https://example.com');
});

test.afterEach(async ({ page }) => {
  console.log('Test completed');
});

test('example test', async ({ page }) => {
  await expect(page).toHaveTitle(/Example/);
});
```

**Available Hooks:**

| Hook                           | Description                                   |
| ------------------------------ | --------------------------------------------- |
| `beforeAll()` / `afterAll()`   | Run once per file                             |
| `beforeEach()` / `afterEach()` | Run before/after each test                    |
| `test.step()`                  | For logical grouping and better trace reports |

**Example using test.step():**

```typescript
await test.step('Fill login form', async () => {
  await page.fill('#username', 'sam');
  await page.fill('#password', 'secret');
});
```

---

### **3.5 Using TypeScript vs JavaScript**

#### **TypeScript Advantages:**

* Built-in types and autocompletion.
* Early detection of errors.
* Supported directly by Playwright Test.

#### **TypeScript Config (tsconfig.json):**

```json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "CommonJS",
    "strict": true,
    "esModuleInterop": true,
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "outDir": "dist"
  }
}
```

#### **TypeScript Example:**

```typescript
import { test, expect } from '@playwright/test';

test('example TS test', async ({ page }) => {
  await page.goto('https://playwright.dev/');
  const title = await page.title();
  expect(title).toContain('Playwright');
});
```

#### **JavaScript Example:**

```javascript
const { test, expect } = require('@playwright/test');

test('example JS test', async ({ page }) => {
  await page.goto('https://playwright.dev/');
  await expect(page).toHaveTitle(/Playwright/);
});
```

#### **Running TypeScript Tests:**

Playwright automatically transpiles TypeScript without manual build steps:

```bash
npx playwright test
```

---

### **3.6 Organizing Tests**

**Test File Convention:**

* Files should end with `.spec.ts` or `.test.ts`.
* Place tests in the `tests/` directory.

**Example Folder Structure:**

```
tests/
  ├─ login.spec.ts
  ├─ dashboard.spec.ts
  └─ api.spec.ts
```

**Grouping Tests:**

```typescript
test.describe('Login Suite', () => {
  test('Valid login', async ({ page }) => {
    await page.goto('/login');
  });

  test('Invalid login', async ({ page }) => {
    await page.goto('/login');
  });
});
```

---

### **3.7 Parallel and Serial Execution**

By default, tests within a file run **in parallel**, but suites can be marked **serial** when needed.

```typescript
test.describe.serial('Account tests', () => {
  test('Step 1 - create', async () => { ... });
  test('Step 2 - update', async () => { ... });
});
```

To run a single test:

```bash
npx playwright test -g "Valid login"
```

---

### **Summary Table**

| Concept        | Key Takeaway                                     |
| -------------- | ------------------------------------------------ |
| **Tests**      | Use `test()` with async/await                    |
| **Locators**   | Use semantic locators (`getByRole`, `getByText`) |
| **Assertions** | Built-in `expect()` auto-waits                   |
| **Hooks**      | Control setup/teardown                           |
| **Isolation**  | New browser context per test                     |
| **TypeScript** | Fully supported, preferred for large projects    |

