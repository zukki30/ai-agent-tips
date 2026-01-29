# E2E Test & Playwright スキル

## 概要
このスキルは、Playwright を使用したエンドツーエンド（E2E）テストのベストプラクティスを提供します。

## Playwright のセットアップ

### インストールと初期設定
```bash
npm init playwright@latest

# または既存プロジェクトに追加
npm install -D @playwright/test
npx playwright install
```

### playwright.config.ts
```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests/e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [
    ['html'],
    ['json', { outputFile: 'test-results.json' }],
    ['junit', { outputFile: 'test-results.xml' }],
  ],
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },

  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
    {
      name: 'Mobile Safari',
      use: { ...devices['iPhone 12'] },
    },
  ],

  webServer: {
    command: 'npm run start',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
    timeout: 120000,
  },
});
```

## 基本的なテストの書き方

### ページナビゲーションとインタラクション
```typescript
import { test, expect } from '@playwright/test';

test.describe('ログイン機能', () => {
  test('正常なログインフロー', async ({ page }) => {
    // ページに移動
    await page.goto('/login');

    // フォームに入力
    await page.fill('input[name="email"]', 'user@example.com');
    await page.fill('input[name="password"]', 'password123');

    // ボタンをクリック
    await page.click('button[type="submit"]');

    // ナビゲーションを待つ
    await page.waitForURL('/dashboard');

    // アサーション
    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('h1')).toHaveText('ダッシュボード');
  });

  test('無効な認証情報でのログイン失敗', async ({ page }) => {
    await page.goto('/login');

    await page.fill('input[name="email"]', 'invalid@example.com');
    await page.fill('input[name="password"]', 'wrongpassword');
    await page.click('button[type="submit"]');

    // エラーメッセージの確認
    await expect(page.locator('.error-message')).toBeVisible();
    await expect(page.locator('.error-message')).toHaveText(
      'メールアドレスまたはパスワードが正しくありません'
    );
  });
});
```

## ロケーター (Locators)

### 推奨されるロケーター戦略
```typescript
// 1. Role によるロケーター (最も推奨)
await page.getByRole('button', { name: '送信' });
await page.getByRole('textbox', { name: 'メールアドレス' });
await page.getByRole('checkbox', { name: '利用規約に同意' });

// 2. Label テキストによるロケーター
await page.getByLabel('パスワード');

// 3. Placeholder によるロケーター
await page.getByPlaceholder('メールアドレスを入力');

// 4. テキストによるロケーター
await page.getByText('ログイン');

// 5. Test ID によるロケーター (data-testid)
await page.getByTestId('submit-button');

// 6. CSS セレクター（最後の手段）
await page.locator('.submit-btn');
```

### ロケーターの連鎖
```typescript
// 特定のコンテナ内の要素を見つける
const userCard = page.locator('.user-card').filter({ hasText: 'John Doe' });
await userCard.getByRole('button', { name: '編集' }).click();

// 複数のロケーターを組み合わせる
await page
  .locator('form')
  .locator('input[name="username"]')
  .fill('testuser');
```

## アサーション

### ページとロケーターのアサーション
```typescript
import { test, expect } from '@playwright/test';

test('各種アサーション', async ({ page }) => {
  await page.goto('/products');

  // ページのアサーション
  await expect(page).toHaveURL(/products/);
  await expect(page).toHaveTitle(/商品一覧/);

  // 要素の表示確認
  await expect(page.getByRole('heading', { name: '商品一覧' })).toBeVisible();
  await expect(page.locator('.loading')).toBeHidden();

  // テキストの確認
  await expect(page.locator('.product-name')).toHaveText('商品A');
  await expect(page.locator('.product-name')).toContainText('商品');

  // 属性の確認
  await expect(page.locator('input')).toHaveAttribute('type', 'text');
  await expect(page.locator('input')).toHaveValue('初期値');

  // カウントの確認
  await expect(page.locator('.product-item')).toHaveCount(10);

  // CSS クラスの確認
  await expect(page.locator('button')).toHaveClass(/active/);

  // 有効/無効の確認
  await expect(page.locator('button')).toBeEnabled();
  await expect(page.locator('input')).toBeDisabled();
});
```

## フィクスチャとフック

### テストフィクスチャ
```typescript
import { test as base } from '@playwright/test';

// カスタムフィクスチャの定義
type MyFixtures = {
  authenticatedPage: Page;
};

const test = base.extend<MyFixtures>({
  authenticatedPage: async ({ page }, use) => {
    // ログイン処理
    await page.goto('/login');
    await page.fill('input[name="email"]', 'test@example.com');
    await page.fill('input[name="password"]', 'password');
    await page.click('button[type="submit"]');
    await page.waitForURL('/dashboard');

    // テストで使用
    await use(page);

    // クリーンアップ（必要な場合）
    await page.goto('/logout');
  },
});

// 使用例
test('認証が必要なページ', async ({ authenticatedPage }) => {
  await authenticatedPage.goto('/profile');
  await expect(authenticatedPage.locator('h1')).toHaveText('プロフィール');
});
```

### フック
```typescript
import { test, expect } from '@playwright/test';

test.describe('商品管理', () => {
  // すべてのテストの前に一度実行
  test.beforeAll(async ({ browser }) => {
    // データベースのセットアップなど
  });

  // 各テストの前に実行
  test.beforeEach(async ({ page }) => {
    await page.goto('/products');
  });

  // 各テストの後に実行
  test.afterEach(async ({ page }) => {
    // クリーンアップ
  });

  // すべてのテストの後に一度実行
  test.afterAll(async () => {
    // データベースのクリーンアップなど
  });

  test('商品一覧の表示', async ({ page }) => {
    // テスト内容
  });
});
```

## API テスト

### REST API のテスト
```typescript
import { test, expect } from '@playwright/test';

test('API エンドポイントのテスト', async ({ request }) => {
  // GET リクエスト
  const response = await request.get('/api/users');
  expect(response.ok()).toBeTruthy();
  expect(response.status()).toBe(200);
  
  const users = await response.json();
  expect(users).toHaveLength(10);

  // POST リクエスト
  const createResponse = await request.post('/api/users', {
    data: {
      name: 'New User',
      email: 'newuser@example.com',
    },
  });
  expect(createResponse.status()).toBe(201);
  
  const newUser = await createResponse.json();
  expect(newUser.name).toBe('New User');

  // PUT リクエスト
  await request.put(`/api/users/${newUser.id}`, {
    data: {
      name: 'Updated User',
    },
  });

  // DELETE リクエスト
  await request.delete(`/api/users/${newUser.id}`);
});
```

### API と UI の組み合わせ
```typescript
test('APIでデータ準備、UIで確認', async ({ page, request }) => {
  // API でテストデータを作成
  const response = await request.post('/api/products', {
    data: {
      name: 'テスト商品',
      price: 1000,
    },
  });
  const product = await response.json();

  // UI で確認
  await page.goto('/products');
  await expect(page.getByText('テスト商品')).toBeVisible();
  await expect(page.getByText('¥1,000')).toBeVisible();
});
```

## ネットワークとモック

### ネットワークリクエストのインターセプト
```typescript
test('ネットワークリクエストのモック', async ({ page }) => {
  // API レスポンスをモック
  await page.route('/api/users', async route => {
    await route.fulfill({
      status: 200,
      contentType: 'application/json',
      body: JSON.stringify([
        { id: 1, name: 'Mock User 1' },
        { id: 2, name: 'Mock User 2' },
      ]),
    });
  });

  await page.goto('/users');
  await expect(page.getByText('Mock User 1')).toBeVisible();
});

test('ネットワークリクエストの監視', async ({ page }) => {
  // リクエストをリッスン
  const requestPromise = page.waitForRequest('/api/users');
  await page.goto('/users');
  const request = await requestPromise;
  
  expect(request.method()).toBe('GET');
});
```

## スクリーンショットとビデオ

### スクリーンショットの取得
```typescript
test('ビジュアルリグレッションテスト', async ({ page }) => {
  await page.goto('/');
  
  // ページ全体のスクリーンショット
  await expect(page).toHaveScreenshot('homepage.png');
  
  // 特定の要素のスクリーンショット
  const element = page.locator('.product-card');
  await expect(element).toHaveScreenshot('product-card.png');
  
  // カスタムしきい値
  await expect(page).toHaveScreenshot({
    maxDiffPixels: 100,
  });
});
```

## モバイルテスト

### モバイルエミュレーション
```typescript
import { test, devices } from '@playwright/test';

test.use({
  ...devices['iPhone 13'],
});

test('モバイルビューのテスト', async ({ page }) => {
  await page.goto('/');
  
  // ハンバーガーメニューをクリック
  await page.getByRole('button', { name: 'メニュー' }).click();
  
  // メニューが表示されることを確認
  await expect(page.locator('.mobile-menu')).toBeVisible();
});

test('タッチイベント', async ({ page }) => {
  await page.goto('/');
  
  const element = page.locator('.swipeable');
  
  // スワイプジェスチャー
  await element.dispatchEvent('touchstart', { touches: [{ clientX: 200, clientY: 100 }] });
  await element.dispatchEvent('touchmove', { touches: [{ clientX: 50, clientY: 100 }] });
  await element.dispatchEvent('touchend');
});
```

## Page Object Model (POM)

### ページオブジェクトの作成
```typescript
// pages/LoginPage.ts
import { Page, Locator } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly submitButton: Locator;
  readonly errorMessage: Locator;

  constructor(page: Page) {
    this.page = page;
    this.emailInput = page.getByLabel('メールアドレス');
    this.passwordInput = page.getByLabel('パスワード');
    this.submitButton = page.getByRole('button', { name: 'ログイン' });
    this.errorMessage = page.locator('.error-message');
  }

  async goto() {
    await this.page.goto('/login');
  }

  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }

  async getErrorMessage() {
    return await this.errorMessage.textContent();
  }
}

// テストでの使用
import { test, expect } from '@playwright/test';
import { LoginPage } from './pages/LoginPage';

test('ログインページのテスト', async ({ page }) => {
  const loginPage = new LoginPage(page);
  
  await loginPage.goto();
  await loginPage.login('user@example.com', 'password123');
  
  await expect(page).toHaveURL('/dashboard');
});
```

## パフォーマンステスト

### パフォーマンスメトリクスの取得
```typescript
test('ページパフォーマンスの測定', async ({ page }) => {
  await page.goto('/');
  
  const performanceMetrics = await page.evaluate(() => {
    const navigation = performance.getEntriesByType('navigation')[0] as PerformanceNavigationTiming;
    return {
      domContentLoaded: navigation.domContentLoadedEventEnd - navigation.domContentLoadedEventStart,
      loadComplete: navigation.loadEventEnd - navigation.loadEventStart,
      firstPaint: performance.getEntriesByName('first-paint')[0]?.startTime,
      firstContentfulPaint: performance.getEntriesByName('first-contentful-paint')[0]?.startTime,
    };
  });
  
  console.log('Performance Metrics:', performanceMetrics);
  
  // パフォーマンスのアサーション
  expect(performanceMetrics.loadComplete).toBeLessThan(3000);
});
```

## ベストプラクティス

### 1. 適切な待機
```typescript
// Bad: 固定時間の待機
await page.waitForTimeout(5000);

// Good: 条件付き待機
await page.waitForSelector('.product-list');
await page.waitForLoadState('networkidle');
await expect(page.locator('.loading')).toBeHidden();
```

### 2. 安定したテスト
```typescript
// Bad: 不安定なセレクター
await page.click('div > div > button:nth-child(3)');

// Good: セマンティックなロケーター
await page.getByRole('button', { name: '送信' }).click();
```

### 3. テストの独立性
```typescript
// 各テストで必要な状態を明示的にセットアップ
test.beforeEach(async ({ page }) => {
  await page.goto('/');
  // 必要な初期状態のセットアップ
});
```

### 4. 並列実行
```typescript
// テストを並列実行可能に設計
test.describe.configure({ mode: 'parallel' });
```

## CI/CD での実行

### GitHub Actions の例
```yaml
name: Playwright Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 18
      - name: Install dependencies
        run: npm ci
      - name: Install Playwright Browsers
        run: npx playwright install --with-deps
      - name: Run Playwright tests
        run: npx playwright test
      - uses: actions/upload-artifact@v3
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

## 参考資料
- Playwright 公式ドキュメント: https://playwright.dev/
- Best Practices: https://playwright.dev/docs/best-practices
- API Reference: https://playwright.dev/docs/api/class-playwright
