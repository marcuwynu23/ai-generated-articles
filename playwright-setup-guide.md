# How to Set Up Playwright for Automated Testing (Full-Screen Mode)
**By [Your Name] | March 2025**

## Introduction
**Playwright** is a powerful automation framework that allows developers to **test web applications** across multiple browsers (**Chromium, Firefox, WebKit**). It is widely used for **end-to-end testing, UI automation, and RPA (Robotic Process Automation).**

In this guide, we’ll cover how to **install, configure, and run Playwright tests** in a web project using **Bun, Node.js, or npm**, and configure it to **run in full-screen mode** for better UI testing visibility.

---

## Step 1: Install Playwright
Navigate to your project folder and run the following command:

### Using Bun
```sh
bun add -d @playwright/test
```

### Using npm or Yarn
```sh
npm install -D @playwright/test
# or
yarn add -D @playwright/test
```

After installation, run the Playwright **setup command** to install the necessary browser binaries:

```sh
npx playwright install
```
This will download **Chromium, Firefox, and WebKit**, enabling you to test across different browsers.

---

## Step 2: Configure Playwright (Full-Screen Mode)
Playwright uses a `playwright.config.ts` file for configuration. Let's create or update it to **run in full-screen mode**.

### ✅ Updated `playwright.config.ts` (Full-Screen Mode)
```ts
import { defineConfig } from "@playwright/test";

export default defineConfig({
  testDir: "./tests/e2e", // Directory where test files are located
  fullyParallel: true,
  use: {
    baseURL: "http://localhost:3000", // Your local dev server
    browserName: "chromium",
    headless: false, // Run with a visible browser
    viewport: null, // Allows the browser to take full screen size
    launchOptions: {
      args: ["--start-maximized"], // Maximizes the window
    },
    screenshot: "only-on-failure",
    video: "retain-on-failure",
  },
});
```

### 🛠 Key Enhancements
✅ **`viewport: null`** → Ensures the browser takes up the full screen dynamically.  
✅ **`args: ["--start-maximized"]`** → Forces the browser to launch **maximized**.  
✅ **Runs in `headless: false` mode** → Ensures visibility during testing.  
✅ **Captures screenshots & videos on failure** for debugging.

---

## Step 3: Create a Playwright Test
Once Playwright is installed, create a new test file inside a `tests/` directory:

```sh
mkdir tests && touch tests/example.spec.ts
```

Now, add the following Playwright test in `tests/example.spec.ts`:

```ts
import { test, expect } from "@playwright/test";

test("Check homepage title", async ({ page }) => {
  await page.goto("https://example.com");
  await expect(page).toHaveTitle("Example Domain");
});
```

This script:
- **Opens** `https://example.com`
- **Checks** if the page title is `"Example Domain"`
- **Verifies** that the website is loading correctly

---

## Step 4: Run Playwright Tests
To execute your test, use the following command:

```sh
bun run playwright test
```
or, if using npm:
```sh
npx playwright test
```

---

## Step 5: Run Playwright in Full-Screen Mode
By default, Playwright runs in **headless mode** (no visible browser window). To **run tests in full-screen mode**, use:

```sh
bun run playwright test --headed
```

or in **debug mode**:
```sh
bun run playwright test --debug
```

✅ **The browser will now launch in full-screen mode**, making UI interactions more visible.

---

## Step 6: Running Tests in CI/CD
To run Playwright tests in a **CI/CD pipeline** (GitHub Actions), create a `.github/workflows/playwright.yml` file:

```yml
name: Playwright Tests

on:
  push:
    branches:
      - main
      - feature/*
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 18

      - name: Install dependencies
        run: bun install

      - name: Install Playwright Browsers
        run: npx playwright install --with-deps

      - name: Run Playwright tests
        run: bun run playwright test
```

This workflow:
✅ Runs tests on **every commit**  
✅ Supports **feature branches**  
✅ Runs on **Ubuntu (Linux CI)**  

---

## Bonus: Automating a Login Test
Playwright can also **test user authentication**. Here's how to automate a login test:

```ts
import { test, expect } from "@playwright/test";

test("Login test", async ({ page }) => {
  await page.goto("https://example.com/login");

  await page.fill("input[name='username']", "testuser");
  await page.fill("input[name='password']", "password123");
  await page.click("button[type='submit']");

  await expect(page).toHaveURL("https://example.com/dashboard");
});
```

---

## Conclusion
🎭 **Playwright** is a powerful tool for **end-to-end testing, automation, and RPA**.  
🚀 With Playwright, you can test across **multiple browsers**, debug easily, and integrate with **CI/CD pipelines**.  

Now you're ready to **automate web applications with Playwright in full-screen mode!** 🎉

---

💡 **Need help setting up Playwright for advanced testing?** Let me know in the comments! 🚀
