# **Playwright Cheat Sheet & Notes**

---

## **1. Browser & Context Management**

| Function / Method                     | Usage                           | Example                                                       |
| ------------------------------------- | ------------------------------- | ------------------------------------------------------------- |
| `chromium.launch({ headless: true })` | Launch Chromium browser         | `const browser = await chromium.launch({ headless: false });` |
| `firefox.launch()`                    | Launch Firefox                  | `const browser = await firefox.launch();`                     |
| `webkit.launch()`                     | Launch WebKit                   | `const browser = await webkit.launch();`                      |
| `browser.newContext()`                | Create isolated browser context | `const context = await browser.newContext();`                 |
| `context.newPage()`                   | Open a new page/tab             | `const page = await context.newPage();`                       |
| `browser.close()`                     | Close browser                   | `await browser.close();`                                      |
| `context.close()`                     | Close context                   | `await context.close();`                                      |

**Notes:** Use **contexts** to isolate sessions and test multi-user scenarios.

---

## **2. Navigation & Page Methods**

| Function / Method           | Usage                 | Example                                                        |
| --------------------------- | --------------------- | -------------------------------------------------------------- |
| `page.goto(url)`            | Navigate to a URL     | `await page.goto('https://example.com');`                      |
| `page.reload()`             | Reload current page   | `await page.reload();`                                         |
| `page.waitForURL(url)`      | Wait for specific URL | `await page.waitForURL('**/dashboard');`                       |
| `page.title()`              | Get page title        | `const title = await page.title();`                            |
| `page.url()`                | Get current URL       | `console.log(await page.url());`                               |
| `page.screenshot({ path })` | Take screenshot       | `await page.screenshot({ path: 'home.png', fullPage: true });` |
| `page.pdf({ path })`        | Generate PDF          | `await page.pdf({ path: 'page.pdf' });`                        |

---

## **3. Locators & Actions**

| Function / Method                  | Usage                    | Example                                                   |
| ---------------------------------- | ------------------------ | --------------------------------------------------------- |
| `page.locator(selector)`           | Select element(s)        | `const btn = page.locator('button#submit');`              |
| `locator.click()`                  | Click element            | `await btn.click();`                                      |
| `locator.dblclick()`               | Double click             | `await btn.dblclick();`                                   |
| `locator.fill(text)`               | Fill input               | `await page.locator('#name').fill('Sam');`                |
| `locator.type(text)`               | Type with delay          | `await page.locator('#search').type('Playwright');`       |
| `locator.check()`                  | Check checkbox           | `await page.locator('#agree').check();`                   |
| `locator.uncheck()`                | Uncheck checkbox         | `await page.locator('#agree').uncheck();`                 |
| `locator.hover()`                  | Hover over element       | `await page.locator('#menu').hover();`                    |
| `locator.press(key)`               | Press key                | `await page.locator('#search').press('Enter');`           |
| `locator.selectOption(value)`      | Select dropdown          | `await page.locator('#country').selectOption('IN');`      |
| `locator.scrollIntoViewIfNeeded()` | Scroll element into view | `await page.locator('#footer').scrollIntoViewIfNeeded();` |

**Advanced Locator Features:**

* `locator.first()`, `locator.nth(index)`
* `locator.filter({ hasText: 'text' })`
* Semantic locators: `getByRole()`, `getByText()`, `getByLabel()`, `getByPlaceholder()`

---

## **4. Assertions**

| Function / Method                              | Usage                    | Example                                                                             |
| ---------------------------------------------- | ------------------------ | ----------------------------------------------------------------------------------- |
| `expect(locator).toBeVisible()`                | Check element is visible | `await expect(page.locator('#submit')).toBeVisible();`                              |
| `expect(locator).toHaveText(text)`             | Check element text       | `await expect(page.locator('#message')).toHaveText('Success');`                     |
| `expect(locator).toHaveCount(n)`               | Check number of elements | `await expect(page.locator('.item')).toHaveCount(5);`                               |
| `expect(page).toHaveTitle(title)`              | Page title               | `await expect(page).toHaveTitle(/Example/);`                                        |
| `expect(response.status()).toBe(200)`          | API status code          | `const res = await page.request.get('/api'); await expect(res.status()).toBe(200);` |
| `expect(locator).toHaveAttribute(attr, value)` | Attribute check          | `await expect(page.locator('img')).toHaveAttribute('alt', 'Logo');`                 |

---

## **5. Frames, Popups & Multiple Tabs**

| Function / Method            | Usage                   | Example                                                                                             |
| ---------------------------- | ----------------------- | --------------------------------------------------------------------------------------------------- |
| `page.frame(nameOrUrl)`      | Access iframe           | `const frame = page.frame('login-frame');`                                                          |
| `page.waitForEvent('popup')` | Wait for new tab/window | `const [popup] = await Promise.all([ page.waitForEvent('popup'), page.locator('#open').click() ]);` |
| `popup.goto(url)`            | Navigate popup          | `await popup.goto('https://example.com');`                                                          |
| `page.context().pages()`     | List all tabs           | `const pages = page.context().pages();`                                                             |

---

## **6. Network Interception & API Testing**

| Function / Method                 | Usage              | Example                                                                            |
| --------------------------------- | ------------------ | ---------------------------------------------------------------------------------- |
| `page.route(url, handler)`        | Intercept requests | `await page.route('**/api', route => route.fulfill({ status: 200, body: '{}' }));` |
| `page.request.get(url)`           | GET API call       | `const res = await page.request.get('/api/users');`                                |
| `page.request.post(url, options)` | POST API call      | `await page.request.post('/api/login', { data: { user, pass } });`                 |
| `route.abort()`                   | Abort request      | `route.abort();`                                                                   |
| `route.continue()`                | Continue request   | `route.continue();`                                                                |
| `route.fulfill()`                 | Mock response      | `route.fulfill({ status: 200, body: '{"ok":true}' });`                             |

---

## **7. Hooks & Fixtures**

| Hook              | Usage                 | Example                                                                |
| ----------------- | --------------------- | ---------------------------------------------------------------------- |
| `test.beforeEach` | Run before each test  | `test.beforeEach(async ({ page }) => { await page.goto('/login'); });` |
| `test.afterEach`  | Run after each test   | `test.afterEach(async ({ page }) => { await page.close(); });`         |
| `test.beforeAll`  | Run before all tests  | `test.beforeAll(async () => { /* setup */ });`                         |
| `test.afterAll`   | Run after all tests   | `test.afterAll(async () => { /* cleanup */ });`                        |
| `fixtures`        | Custom shared objects | `test.use({ storageState: 'state.json' });`                            |

---

## **8. Debugging & Tracing**

| Method                                           | Usage           | Example                                                             |
| ------------------------------------------------ | --------------- | ------------------------------------------------------------------- |
| `page.pause()`                                   | Pause test      | `await page.pause();`                                               |
| `page.tracing.start({ screenshots, snapshots })` | Start tracing   | `await page.tracing.start({ screenshots: true, snapshots: true });` |
| `page.tracing.stop({ path })`                    | Stop tracing    | `await page.tracing.stop({ path: 'trace.zip' });`                   |
| `page.screenshot({ path })`                      | Take screenshot | `await page.screenshot({ path: 'screen.png' });`                    |
| `page.video().path()`                            | Capture video   | `await page.video().saveAs('video.mp4');`                           |

---

## **9. Visual Regression Testing**

| Method                               | Usage                    | Example                                                    |
| ------------------------------------ | ------------------------ | ---------------------------------------------------------- |
| `expect(locator).toHaveScreenshot()` | Compare element snapshot | `await expect(page.locator('#logo')).toHaveScreenshot();`  |
| `expect(page).toHaveScreenshot()`    | Compare full page        | `await expect(page).toHaveScreenshot({ fullPage: true });` |

---

## **10. CI/CD & Parallel Testing**

* Configure `playwright.config.ts` for **parallelism**:

```ts
workers: 4,
projects: [
  { name: 'Chromium', use: { browserName: 'chromium' } },
  { name: 'Firefox', use: { browserName: 'firefox' } },
  { name: 'WebKit', use: { browserName: 'webkit' } },
]
```

* Enable **trace, video, screenshots** for debugging in CI:

```ts
use: { trace: 'retain-on-failure', video: 'retain-on-failure' }
```

---

## **11. Common Use Cases**

1. **Login Flow Test**

```ts
await page.goto('/login');
await page.fill('#username', 'sam');
await page.fill('#password', 'pass');
await page.click('#submit');
await expect(page.locator('#welcome')).toHaveText('Welcome Sam');
```

2. **Web Scraping**

```ts
const items = await page.locator('.product').allTextContents();
```

3. **API Validation**

```ts
const response = await page.request.get('/api/data');
await expect(response.status()).toBe(200);
```

4. **Multi-browser Testing**

```ts
await chromium.launch(); await firefox.launch(); await webkit.launch();
```

5. **Component Testing**

```ts
await mount(<Button label="Click" />);
await expect(locator).toHaveText('Click');
```

---

This cheat sheet gives you **all essential Playwright methods, functions, and typical use cases**, so you can **quickly write tests, automate workflows, scrape data, and debug effectively**.

