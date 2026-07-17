---
name: playwright-e2e
description: This skill guides writing Playwright end-to-end tests with authentication token management, page object models, parallel test execution, and API testing patterns. Use when creating E2E tests, writing Playwright test suites, implementing auth token managers, testing React admin dashboards, or setting up Playwright configurations.
---

# Playwright E2E Testing

## Core Principle

Authenticate ONCE, test MANY times. Use `AuthTokenManager` for all authenticated tests. Never hardcode tokens or login repeatedly in each test.

## Project Setup

```bash
npm init playwright@latest
```

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  timeout: 30_000,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [['html'], ['list']],
  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:4000',
    screenshot: 'only-on-failure',
    trace: 'on-first-retry',
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
  ],
  webServer: {
    command: 'pnpm dev',
    port: 4000,
    reuseExistingServer: !process.env.CI,
  },
});
```

## Auth Token Manager Pattern

```typescript
interface TokenState {
  accessToken: string;
  refreshToken: string;
  expiresAt: number;
}

class AuthTokenManager {
  private state: TokenState | null = null;
  private baseUrl: string;
  private storagePath: string;

  constructor(options: { baseUrl: string; storagePath: string }) {
    this.baseUrl = options.baseUrl;
    this.storagePath = options.storagePath;
  }

  async initialize(): Promise<void> {
    try {
      const saved = JSON.parse(fs.readFileSync(this.storagePath, 'utf-8'));
      if (saved.expiresAt > Date.now()) {
        this.state = saved;
        return;
      }
    } catch {}
    await this.login();
  }

  async getValidToken(): Promise<string> {
    if (!this.state || this.state.expiresAt < Date.now() + 60_000) {
      await this.refresh();
    }
    return this.state!.accessToken;
  }

  getAuthHeaders(): Record<string, string> {
    return { Authorization: `Bearer ${this.state!.accessToken}` };
  }

  async saveTokenState(): Promise<void> {
    fs.writeFileSync(this.storagePath, JSON.stringify(this.state));
  }

  private async login(): Promise<void> {
    const res = await fetch(`${this.baseUrl}/api/v1/auth/login`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        email: process.env.TEST_USER_EMAIL,
        password: process.env.TEST_USER_PASSWORD,
      }),
    });
    const data = await res.json();
    this.state = {
      accessToken: data.access_token,
      refreshToken: data.refresh_token,
      expiresAt: Date.now() + data.expires_in * 1000,
    };
  }

  private async refresh(): Promise<void> {
    // Refresh token logic
  }
}
```

## Test Structure with Auth

```typescript
import { test, expect } from '@playwright/test';

let authManager: AuthTokenManager;

test.describe.serial('Camera Management', () => {
  test.beforeAll(async () => {
    authManager = new AuthTokenManager({
      baseUrl: process.env.BASE_URL || 'http://localhost:8000',
      storagePath: '.playwright/auth-state.json',
    });
    await authManager.initialize();
  });

  test('should list cameras', async ({ page }) => {
    await page.goto('/cameras');
    await expect(page.getByRole('heading', { name: 'Cameras' })).toBeVisible();
    const rows = page.getByTestId('camera-row');
    await expect(rows.first()).toBeVisible();
  });

  test('should create camera', async ({ page }) => {
    await page.goto('/cameras/new');
    await page.getByLabel('Name').fill('Lobby Camera');
    await page.getByLabel('RTSP URL').fill('rtsp://192.168.1.100:554/stream');
    await page.getByRole('button', { name: 'Save' }).click();
    await expect(page.getByText('Camera created')).toBeVisible();
  });

  test.afterAll(async () => {
    await authManager.saveTokenState();
  });
});
```

## Page Object Model

```typescript
class CamerasPage {
  constructor(private page: Page) {}

  async goto() {
    await this.page.goto('/cameras');
  }

  async waitForLoad() {
    await this.page.waitForSelector('[data-testid="camera-list"]');
  }

  getCameraRow(name: string) {
    return this.page.getByRole('row', { name });
  }

  async createCamera(data: { name: string; rtspUrl: string }) {
    await this.page.getByRole('button', { name: 'Add Camera' }).click();
    await this.page.getByLabel('Name').fill(data.name);
    await this.page.getByLabel('RTSP URL').fill(data.rtspUrl);
    await this.page.getByRole('button', { name: 'Save' }).click();
  }

  async deleteCamera(name: string) {
    const row = this.getCameraRow(name);
    await row.getByRole('button', { name: 'Delete' }).click();
    await this.page.getByRole('button', { name: 'Confirm' }).click();
  }
}
```

## API Testing Pattern

```typescript
test('API: create and fetch camera', async ({ request }) => {
  const token = await authManager.getValidToken();
  const headers = authManager.getAuthHeaders();

  const createRes = await request.post('/api/v1/cameras', {
    headers,
    data: { name: 'Test Camera', rtsp_url: 'rtsp://test:554/stream' },
  });
  expect(createRes.status()).toBe(201);
  const created = await createRes.json();

  const getRes = await request.get(`/api/v1/cameras/${created.data.id}`, { headers });
  expect(getRes.status()).toBe(200);
  const fetched = await getRes.json();
  expect(fetched.data.name).toBe('Test Camera');
});
```

## Visual Regression Testing

```typescript
test('dashboard matches screenshot', async ({ page }) => {
  await page.goto('/dashboard');
  await page.waitForLoadState('networkidle');
  await expect(page).toHaveScreenshot('dashboard.png', {
    maxDiffPixelRatio: 0.01,
  });
});
```

## Best Practices

- Use `data-testid` attributes for stable selectors
- Prefer user-visible text and ARIA roles over CSS selectors
- Use `test.describe.serial` for tests that share state
- Run tests in parallel by default, serial only when needed
- Use `expect.toBeVisible()` over `expect.toHaveCount()`
- Always clean up test data in `afterAll` or `afterEach`
- Use environment variables for URLs and credentials
- Enable trace recording on first retry for debugging
