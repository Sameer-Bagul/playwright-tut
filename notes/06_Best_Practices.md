## **Chapter 6: Best Practices**

---

### **6.1 Writing Resilient Tests (Avoid Flakiness)**

Flaky tests are one of the most common problems in automation. Playwright’s architecture and auto-waiting reduce flakiness, but following **best practices** ensures tests are **stable, predictable, and reliable**.

#### **Key Principles**

1. **Avoid Hard Waits**

   ```ts
   // Bad practice
   await page.waitForTimeout(5000);

   // Better: Auto-wait or explicit wait
   await page.locator('button#submit').click();
   ```

   * Playwright auto-waits for elements to be **visible, enabled, and stable**.
   * Hard-coded delays increase test time and are unreliable.

2. **Use Locators, Not Raw Selectors**

   ```ts
   // Less resilient
   await page.click('button:nth-child(3)');

   // More resilient
   await page.getByRole('button', { name: 'Submit' }).click();
   ```

   * Prefer **semantic locators** (`getByRole`, `getByLabel`, `getByText`) for long-term maintainability.
   * Avoid brittle CSS selectors that change frequently.

3. **Avoid Global State Dependencies**

   * Each test should be **isolated** and not rely on previous tests.
   * Use `browser.newContext()` for clean sessions.

4. **Handle Asynchronous Behavior Properly**

   * Always use `await` with actions.
   * Avoid multiple chained async actions without waiting.

---

### **6.2 Use of Locators Over Brittle Selectors**

**Locators** are the cornerstone of Playwright’s stability. They automatically:

* Wait for elements to appear
* Ensure the element is actionable
* Reduce the risk of flakiness

#### **Locator Types**

| Type           | Usage              | Notes                                      |
| -------------- | ------------------ | ------------------------------------------ |
| `getByRole()`  | Semantic ARIA role | Best for accessibility and stability       |
| `getByText()`  | Visible text       | Works for buttons, links, labels           |
| `getByLabel()` | Form labels        | Matches input elements reliably            |
| `locator()`    | CSS/XPath          | Only if semantic locators aren’t available |

**Chaining & Filtering:**

```ts
await page.locator('.product').filter({ hasText: 'Laptop' }).first().click();
```

* Targets specific elements within dynamic content.
* Reduces reliance on index-based selectors.

---

### **6.3 Isolating Tests and Avoiding Third-Party Dependencies**

#### **Isolate Tests**

* Each test should **create its own context**:

  ```ts
  const context = await browser.newContext();
  const page = await context.newPage();
  ```
* This ensures **no shared cookies, local storage, or cache** between tests.

#### **Avoid External Dependencies**

* Minimize reliance on:

  * Live APIs
  * Third-party services
  * External databases
* Instead, **mock responses** using `page.route()`:

  ```ts
  await page.route('**/api/users', route =>
    route.fulfill({ status: 200, body: JSON.stringify([{ id: 1, name: 'Sam' }]) })
  );
  ```

#### **Use Fixtures for Shared Setup**

* Use fixtures to **reuse code** and manage test data.
* Example: authenticated sessions, mock APIs, or pre-populated forms.

---

### **6.4 Naming, Structure, and Maintainability**

Well-structured tests are easier to **read, debug, and extend**.

#### **File and Test Naming**

* Use meaningful test names that describe behavior:

  ```ts
  test('Login with valid credentials redirects to dashboard', async ({ page }) => { ... });
  ```
* Test files:

  * `login.spec.ts`, `dashboard.spec.ts`, `checkout.spec.ts`
* Test folders:

  ```
  tests/
    auth/
    dashboard/
    checkout/
  ```

#### **Describe Blocks**

* Group related tests using `test.describe()`:

  ```ts
  test.describe('User Authentication', () => {
    test('Login success', async ({ page }) => {});
    test('Login failure', async ({ page }) => {});
  });
  ```

#### **Reusable Functions**

* Extract repeated logic into **helper functions**:

  ```ts
  export async function login(page: Page, username: string, password: string) {
    await page.fill('#username', username);
    await page.fill('#password', password);
    await page.click('#loginBtn');
  }
  ```
* Avoid duplication — helps maintain large test suites.

---

### **6.5 Managing Test Data Effectively**

**Best Practices for Test Data:**

* Use **fixtures or factories** to create test data dynamically.
* Avoid hardcoding credentials, IDs, or URLs in tests.
* Store environment-specific data in `.env` files:

  ```ts
  BASE_URL=https://staging.example.com
  USERNAME=testuser
  PASSWORD=secret
  ```

**Example: Using Fixtures for Test Data**

```ts
test.use({
  storageState: 'state.json',  // pre-authenticated session
});
```

* Simplifies tests that require login or pre-existing data.
* Reduces test execution time.

---

### **6.6 Screenshots, Videos, and Tracing for Debugging**

Playwright provides **built-in tools** to debug failures:

#### **Screenshots**

* Capture screenshots on failure:

```ts
await page.screenshot({ path: 'error.png', fullPage: true });
```

* Or configure globally in `playwright.config.ts`:

```ts
use: { screenshot: 'only-on-failure' }
```

#### **Videos**

* Record full test runs:

```ts
use: { video: 'retain-on-failure' }
```

* Useful for CI/CD pipelines to understand failures.

#### **Tracing**

* Trace tests step-by-step for detailed analysis:

```ts
await page.tracing.start({ screenshots: true, snapshots: true });
await page.goto('https://example.com');
await page.tracing.stop({ path: 'trace.zip' });
```

* Open in Playwright Inspector:

```bash
npx playwright show-trace trace.zip
```

---

### **6.7 Performance and Parallel Execution Best Practices**

#### **Parallelism**

* Enable parallel test execution using multiple workers:

```ts
workers: 4  // in playwright.config.ts
```

* Each test runs in its **own context** — isolated and faster.

#### **Avoid Overloading Resources**

* Limit CPU/memory-heavy tests in parallel to avoid flakiness.
* Consider splitting large test suites into **multiple projects**.

#### **Retries**

* Configure `retries: 2` for network-dependent or flaky tests.
* Combine with tracing for debugging failures.

---

### **6.8 CI/CD Recommendations**

#### **Headless Mode**

* Run tests headlessly in CI to save resources:

```ts
use: { headless: true }
```

#### **Artifacts**

* Upload screenshots, videos, or trace files as CI artifacts for analysis.

#### **Environment Management**

* Use environment variables for URLs, credentials, and API endpoints.

#### **Parallel Builds**

* Split tests across multiple CI jobs for faster execution.

**Example GitHub Actions Snippet:**

```yaml
- run: npx playwright test --workers=4 --reporter=html
- uses: actions/upload-artifact@v3
  with:
    name: playwright-report
    path: playwright-report/
```

---

### **6.9 Accessibility and Semantic Practices**

* Prefer **ARIA-based locators**:

```ts
page.getByRole('button', { name: 'Submit' });
```

* Ensures accessibility and test reliability.
* Avoid brittle selectors like `.btn-primary` that are prone to UI changes.

---

### **6.10 Versioning and Dependency Management**

* Lock Playwright version in `package.json` to prevent breaking changes:

```json
"@playwright/test": "1.49.0"
```

* Use `npx playwright install` to ensure **browser binaries match the version**.
* Update dependencies cautiously; always run regression tests.

---

### **6.11 Common Pitfalls to Avoid**

| Pitfall                    | Why It's a Problem                         | Solution                               |
| -------------------------- | ------------------------------------------ | -------------------------------------- |
| Hard-coded timeouts        | Slows tests and introduces flakiness       | Use auto-waiting locators              |
| Shared state               | Tests fail due to leftover cookies/storage | Use isolated `BrowserContext` per test |
| Nested async without await | Actions execute out of order               | Always `await` asynchronous calls      |
| Excessive screenshots      | Slows CI/CD pipeline                       | Capture only on failure or selectively |
| Ignoring traces            | Makes debugging failures difficult         | Use tracing for failed tests           |

---

### **6.12 Summary**

* **Write resilient tests**: auto-waits, semantic locators, no hard waits.
* **Isolate tests**: separate browser contexts for independence.
* **Use fixtures and reusable helpers** to reduce duplication.
* **Debugging tools**: screenshots, video, tracing.
* **Organize tests properly**: naming, folders, describe blocks.
* **CI/CD practices**: headless execution, environment variables, parallelism.
* **Maintainability**: version locking, avoid brittle selectors, semantic locators.

Following these best practices ensures **scalable, maintainable, and reliable** Playwright test suites suitable for enterprise-grade automation.

