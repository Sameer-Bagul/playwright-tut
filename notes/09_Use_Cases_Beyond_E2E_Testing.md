## **Chapter 9: Use Cases Beyond E2E Testing**

Playwright is commonly known as an end-to-end testing framework, but its capabilities extend far beyond traditional testing. With full **browser automation, network control, and multi-browser support**, it can be leveraged for **web scraping, component testing, API testing, workflow automation, and server-side automation**. This chapter explores these use cases in depth.

---

### **9.1 Web Scraping and Data Extraction**

Modern websites often rely heavily on **JavaScript for rendering content dynamically**, which makes traditional scraping methods (like `requests` or `BeautifulSoup`) insufficient. Playwright excels in this area by **rendering the page fully in a real browser context**, ensuring accurate extraction.

#### **Why Playwright for Scraping?**

* **Handles dynamic content:** Waits for elements to render before scraping.
* **Simulates real user behavior:** Handles infinite scroll, pagination, and form submissions.
* **Network control:** Can intercept requests, throttle bandwidth, or simulate offline conditions.
* **Multi-browser scraping:** Scrape across Chrome, Firefox, WebKit to avoid detection or verify cross-browser data integrity.

#### **Example: Scraping E-commerce Products**

```ts
import { chromium } from 'playwright';

(async () => {
  const browser = await chromium.launch({ headless: true });
  const page = await browser.newPage();
  await page.goto('https://example-ecommerce.com');

  // Wait for products to load
  await page.waitForSelector('.product-card');

  // Extract product names and prices
  const products = await page.locator('.product-card').evaluateAll(cards =>
    cards.map(card => ({
      name: card.querySelector('.product-title')?.textContent,
      price: card.querySelector('.product-price')?.textContent
    }))
  );

  console.log(products);
  await browser.close();
})();
```

#### **Advanced Scraping Techniques**

1. **Infinite scroll handling**

   ```ts
   let previousHeight;
   while (true) {
     previousHeight = await page.evaluate('document.body.scrollHeight');
     await page.evaluate('window.scrollTo(0, document.body.scrollHeight)');
     await page.waitForTimeout(1000);
     const newHeight = await page.evaluate('document.body.scrollHeight');
     if (newHeight === previousHeight) break;
   }
   ```

2. **Pagination**

   * Navigate multiple pages using `for` loops and next-page buttons.
   * Combine with **locator filters** to extract specific content.

3. **Throttling & Human-like Interactions**

   * Random delays and scrolling mimic real user activity.
   * Reduces risk of anti-scraping measures blocking your bot.

---

### **9.2 Component Testing in Isolation**

Playwright is increasingly used for **component-level testing**, particularly with modern frameworks like **React, Vue, Angular, or Svelte**.

#### **Benefits**

* Runs components in a **real browser environment**, unlike Jest or jsdom.
* Supports **user interactions, events, and accessibility testing**.
* Can be combined with **visual regression** to detect UI anomalies per component.

#### **Example: Testing a React Button Component**

```ts
import { test, expect } from '@playwright/react';
import Button from './Button';

test('Button click triggers callback', async ({ mount }) => {
  const component = await mount(<Button label="Submit" />);
  await component.click();
  await expect(component).toHaveText('Clicked!');
});
```

**Advanced Features for Component Testing**

* **Hook support**: Test React hooks and lifecycle events.
* **Prop variation testing**: Automatically test different prop combinations.
* **Visual regression integration**: Capture snapshots for each component state.

---

### **9.3 API Testing and Request Interception**

Playwright can handle **API testing**, even though it’s a UI-focused framework. Its **network interception and request manipulation** capabilities enable robust testing of back-end services.

#### **Network Interception**

```ts
await page.route('**/api/users', route =>
  route.fulfill({ status: 200, body: JSON.stringify([{ id: 1, name: 'Sameer' }]) })
);
```

* Mock backend responses to **simulate failures or edge cases**.
* Test front-end behavior without hitting live APIs.
* Capture request/response data for validation.

#### **Direct API Requests**

* Playwright can issue **HTTP requests directly** using `APIRequestContext`:

```ts
import { test, request } from '@playwright/test';

test('Validate user API', async () => {
  const apiContext = await request.newContext();
  const response = await apiContext.get('https://api.example.com/users');
  const data = await response.json();
  expect(data).toHaveLengthGreaterThan(0);
});
```

**Advantages**

* Integrates UI and API testing in a single framework.
* Ensures front-end reacts correctly to API responses.
* Allows **error simulation** (500, 404, timeout) without backend changes.

---

### **9.4 Automating Complex Workflows**

Playwright allows full **automation of multi-step web processes**, not limited to testing.

#### **Example: Automating an E-commerce Checkout Flow**

```ts
await page.goto('https://example.com');
await page.fill('#search', 'Laptop');
await page.click('text=Search');
await page.click('.product-card:first-child button');
await page.click('#checkout');
await page.fill('#cardNumber', '4111111111111111');
await page.click('#pay');
```

**Key Techniques**

* **Multi-page navigation**: Handle multiple tabs, pop-ups, and redirects.
* **Form automation**: Fill, validate, and submit forms programmatically.
* **Error handling**: Retry or fallback in case of network or UI failures.

---

### **9.5 Headless and Server-Side Automation**

Headless execution allows **server-side automation**, such as:

* Scheduled scraping tasks.
* Batch report generation.
* Monitoring dashboards.
* Automating repetitive tasks like bulk form submissions.

#### **Example: Scheduled Screenshot Capture**

```ts
import { chromium } from 'playwright';

(async () => {
  const browser = await chromium.launch({ headless: true });
  const page = await browser.newPage();
  await page.goto('https://example.com/dashboard');
  await page.screenshot({ path: 'dashboard.png', fullPage: true });
  await browser.close();
})();
```

**Benefits**

* Faster execution than headed mode.
* Can run on CI/CD servers, Docker containers, or cloud functions.
* Enables **full automation pipelines** without manual intervention.

---

### **9.6 Integrating E2E, Component, API, and Automation**

Playwright allows teams to **combine different testing strategies** in a single project:

1. **E2E testing** for full user flows.
2. **Component testing** for isolated UI behaviors.
3. **API testing** for backend validations.
4. **Workflow automation** for tasks like scraping, reporting, or alerts.

#### **Example: Combined Workflow**

* Test login E2E.
* Validate login API response.
* Capture screenshot and video.
* Scrape dashboard stats for reporting.
* Mock API responses to simulate error conditions.

```ts
await page.tracing.start({ screenshots: true });
await page.goto('/login');
await page.fill('#username', 'sameer');
await page.fill('#password', 'password');
await page.click('#loginBtn');

const response = await page.request.get('/api/dashboard');
console.log(await response.json());

await page.screenshot({ path: 'dashboard.png' });
await page.tracing.stop({ path: 'trace.zip' });
```

**Outcome**

* Unified testing, automation, and reporting.
* Reduced framework complexity (no need for multiple tools).
* High reliability and maintainability.

---

### **9.7 Advanced Automation Practices**

* **Context Isolation**: Use `browser.newContext()` for independent sessions.
* **Parallel Execution**: Run multiple automation tasks simultaneously for efficiency.
* **Retry Mechanism**: Automate retries for flaky actions or network issues.
* **Data-driven Automation**: Pull test or scraping data from CSV, JSON, or databases.

**Example: Parallel Scraping**

```ts
const browsers = await Promise.all([
  chromium.launch({ headless: true }),
  firefox.launch({ headless: true }),
  webkit.launch({ headless: true })
]);
```

---

### **9.8 Combining Playwright with Other Tools**

* **Database verification**: Validate scraped data or form submissions in the database.
* **CI/CD integration**: Schedule scraping or automation tasks in GitHub Actions, GitLab CI, Jenkins.
* **Reporting**: Generate automated reports combining E2E results, screenshots, and API logs.
* **Visual Regression**: Combine scraping with visual regression to detect UI anomalies.

---

### **9.9 Summary**

Playwright’s capabilities extend far beyond traditional E2E testing. Its **full browser automation, network control, and multi-browser support** allow it to be used for:

* **Web Scraping**: Extract structured data from dynamic web pages.
* **Component Testing**: Validate individual UI components in real browser environments.
* **API Testing**: Intercept requests, mock APIs, and validate backend responses.
* **Complex Workflow Automation**: Automate multi-step processes, forms, and user interactions.
* **Headless Server-side Automation**: Scheduled tasks, reports, or dashboards.
* **Integrated Strategies**: Combine E2E, API, component, and automation for enterprise-grade pipelines.

By leveraging these use cases, teams can **maximize the value of Playwright**, reducing tool fragmentation while improving reliability, maintainability, and automation coverage.

