## **Chapter 10: Troubleshooting & Frequently Asked Questions**

Playwright is powerful, but like any automation tool, tests can fail due to **environment issues, flaky selectors, timing problems, or browser inconsistencies**. This chapter provides **practical troubleshooting strategies**, tips for debugging, and answers to frequently asked questions, making Playwright automation **more reliable and maintainable**.

---

### **10.1 Common Pitfalls**

Understanding common pitfalls helps prevent issues before they occur.

#### **1. Flaky Tests**

* **Cause:** Elements not yet loaded or dynamic content updates.
* **Solution:** Use **Playwright’s auto-waiting locators** instead of manual waits.

```ts
await page.locator('#submitButton').click(); // auto-waits
```

* Avoid `page.waitForTimeout()` unless necessary; prefer `waitForSelector`, `toBeVisible`, or `toHaveText`.

#### **2. Incorrect Selectors**

* **Cause:** Using brittle CSS selectors that change frequently.
* **Solution:** Use **robust locator strategies**:

  * `getByRole()`
  * `getByText()`
  * `getByLabel()`
  * Chaining locators for precision.

```ts
await page.getByRole('button', { name: 'Submit' }).click();
```

#### **3. Timing Issues**

* **Cause:** Tests run faster than the UI can update.
* **Solution:** Rely on **auto-waiting features** and `await` every action.
* Use `expect` with built-in timeout options:

```ts
await expect(page.locator('#message')).toHaveText('Success', { timeout: 5000 });
```

#### **4. Browser Incompatibility**

* **Cause:** Tests passing in Chromium but failing in Firefox/WebKit.
* **Solution:** Test across **all target browsers** regularly.
* Use `projects` in `playwright.config.ts` to run tests cross-browser.

#### **5. Resource Bottlenecks**

* **Cause:** Large test suites consuming memory or CPU.
* **Solution:** Limit **parallelism**:

```ts
workers: 2 // in playwright.config.ts
```

* Use **context isolation** to prevent session conflicts.

---

### **10.2 Debugging Tests**

Playwright provides several **tools and techniques** for debugging failures.

#### **1. Playwright Inspector**

* Launch tests in interactive mode:

```bash
npx playwright test --debug
```

* Features:

  * Pause/resume execution.
  * Inspect locators visually.
  * Step through each action.

#### **2. Trace Viewer**

* Record **screenshots, DOM snapshots, and network events**.

```ts
await page.tracing.start({ screenshots: true, snapshots: true });
await page.goto('https://example.com');
await page.tracing.stop({ path: 'trace.zip' });
```

* Visualize traces with:

```bash
npx playwright show-trace trace.zip
```

#### **3. Console Logging**

* Capture browser console messages for errors or warnings.

```ts
page.on('console', msg => console.log(msg.text()));
```

#### **4. Pause Test Execution**

```ts
await page.pause();
```

* Opens Playwright Inspector mid-test.
* Allows **manual interaction** to diagnose issues.

#### **5. Verbose API Debugging**

* Enable detailed logging:

```bash
DEBUG=pw:api npx playwright test
```

* Logs all Playwright API calls and network events.

---

### **10.3 Performance and Parallelism Issues**

Tests can fail or slow down due to **parallel execution, heavy pages, or resource constraints**.

#### **1. Optimizing Parallel Execution**

* Use **`workers`** in configuration to balance speed and resource usage:

```ts
workers: 4
```

* Use **isolated browser contexts** instead of launching multiple browsers for efficiency.

#### **2. Avoiding Heavy Assets**

* Block unnecessary resources:

```ts
await page.route('**/*.png', route => route.abort());
```

* Speeds up test execution and reduces flakiness.

#### **3. Headless vs Headed Mode**

* Headless mode is faster but sometimes behaves differently for rendering.
* Use headed mode for debugging visual issues:

```ts
browser = await chromium.launch({ headless: false });
```

---

### **10.4 Handling Flaky Tests**

Flakiness is one of the biggest challenges in automation.

#### **Common Causes**

* Dynamic UI content.
* Network delays or intermittent failures.
* Conflicts between parallel tests.

#### **Strategies to Reduce Flakiness**

1. **Use Auto-Waiting Locators**
2. **Test Isolation**

   ```ts
   test.use({ storageState: undefined });
   ```
3. **Retries**

   ```ts
   retries: 2 // in playwright.config.ts
   ```
4. **Network Mocking**

   * Prevent reliance on slow or unstable APIs.
5. **Selectors Best Practices**

   * Prefer **semantic selectors** (`getByRole`, `getByText`) over brittle CSS.

---

### **10.5 Cross-Browser Debugging**

* Tests may behave differently in Chromium, Firefox, or WebKit.
* **Strategies:**

  1. Regularly test on all browser targets.
  2. Use **projects** in `playwright.config.ts` to manage configurations.
  3. Capture **traces, screenshots, and videos** for each browser.
  4. Use **feature detection** for browser-specific behavior instead of hard-coded assumptions.

---

### **10.6 Common Errors and Solutions**

| Error                                   | Cause                               | Solution                                                           |
| --------------------------------------- | ----------------------------------- | ------------------------------------------------------------------ |
| `TimeoutError: waiting for selector`    | Element not rendered yet            | Use `await expect(locator).toBeVisible()` instead of fixed waits   |
| `Navigation failed`                     | Page load timeout or network issue  | Increase `timeout`, check network conditions, use retries          |
| `Context closed`                        | Browser context closed unexpectedly | Ensure `browser.newContext()` and `browser.close()` are balanced   |
| `Locator not found`                     | Wrong selector or dynamic content   | Use robust locators like `getByRole`, `nth()`, or `filter()`       |
| `Cannot find module '@playwright/test'` | Dependency missing                  | Run `npm install -D @playwright/test` and `npx playwright install` |

---

### **10.7 Best Practices for Debugging and Maintenance**

1. **Write Descriptive Test Names**

   * Makes debugging easier in reports and CI logs.

2. **Use Fixtures and Hooks**

   * Isolate setup/teardown to avoid side effects:

   ```ts
   test.beforeEach(async ({ page }) => { await page.goto('/login'); });
   ```

3. **Capture Artifacts**

   * Videos, screenshots, and traces help diagnose failures:

   ```ts
   use: { trace: 'retain-on-failure', video: 'retain-on-failure' }
   ```

4. **Test in Realistic Environments**

   * Match CI browsers, resolutions, and network conditions.

5. **Maintain Selectors**

   * Regularly review and update locators after UI changes.

---

### **10.8 Frequently Asked Questions**

**Q1: Why is my test passing locally but failing in CI?**

* Differences in browser versions, network speed, or resource availability. Capture **traces and videos** to diagnose.

**Q2: How can I reduce test flakiness?**

* Use robust locators, auto-waiting, retries, and isolated contexts. Avoid fixed timeouts unless necessary.

**Q3: Can I test mobile layouts?**

* Yes, use Playwright’s **device emulation**:

```ts
const context = await browser.newContext({ ...devices['iPhone 14'] });
```

**Q4: How to debug intermittent network issues?**

* Mock or intercept API responses, use tracing, and retry failed requests.

**Q5: Why are my screenshots different across runs?**

* Minor rendering differences may occur. Use **thresholds in visual regression testing**:

```ts
await expect(locator).toHaveScreenshot({ threshold: 0.1 });
```

**Q6: Can Playwright tests be parallelized safely?**

* Yes, with **isolated contexts** and `workers` configured in `playwright.config.ts`.

---

### **10.9 Summary**

Effective troubleshooting in Playwright involves understanding **common pitfalls, debugging tools, and performance optimization**:

* **Flakiness Prevention:** Auto-waiting, retries, robust locators.
* **Debugging Tools:** Inspector, tracing, screenshots, console logs.
* **Performance Optimization:** Parallelism, headless mode, blocking heavy assets.
* **Cross-Browser Awareness:** Test across Chromium, Firefox, WebKit, and mobile emulations.
* **CI/CD Integration:** Capture artifacts and run headless tests for reliability.
* **Documentation & FAQ:** Maintain internal documentation to quickly resolve recurring issues.

By mastering these techniques, teams can **dramatically reduce test failures, improve reliability, and maintain a scalable Playwright test suite**.

