# テスト実行コマンド集

## Jest テストコマンド

### 基本実行
```bash
# すべてのテストを実行
npm test

# または
npx jest

# ウォッチモード（変更を監視して自動実行）
npm test -- --watch

# または
npx jest --watch
```

### カバレッジ
```bash
# カバレッジを取得
npm test -- --coverage

# カバレッジをブラウザで開く
npm test -- --coverage && open coverage/lcov-report/index.html
```

### フィルタリング
```bash
# 特定のファイルのみテスト
npx jest path/to/test-file.test.ts

# ファイル名パターンでフィルタ
npx jest --testNamePattern=user

# テスト名でフィルタ
npx jest -t "should create user"

# 変更されたファイルのみテスト
npx jest --onlyChanged

# 特定のディレクトリのテスト
npx jest src/users/
```

### その他のオプション
```bash
# 詳細な出力
npx jest --verbose

# 失敗したテストのみ再実行
npx jest --onlyFailures

# 並列実行の制御
npx jest --maxWorkers=4

# デバッグモード
node --inspect-brk node_modules/.bin/jest --runInBand

# ログ出力を抑制
npx jest --silent
```

### CI環境での実行
```bash
# CI モード（キャッシュなし、ウォッチなし）
npx jest --ci

# CI でカバレッジも取得
npx jest --ci --coverage --maxWorkers=2
```

## Vitest テストコマンド

### 基本実行
```bash
# すべてのテストを実行
npx vitest

# 単発実行（ウォッチなし）
npx vitest run

# UI モード
npx vitest --ui
```

### カバレッジ
```bash
# カバレッジを取得
npx vitest --coverage

# カバレッジプロバイダーを指定
npx vitest --coverage --coverage.provider=v8
```

### フィルタリング
```bash
# 特定のファイルのみテスト
npx vitest path/to/test.test.ts

# テスト名でフィルタ
npx vitest -t "user creation"

# 変更されたファイルのみ
npx vitest --changed
```

## Playwright テストコマンド

### 基本実行
```bash
# すべてのテストを実行
npx playwright test

# ヘッドモード（ブラウザを表示）
npx playwright test --headed

# UIモード（インタラクティブ）
npx playwright test --ui

# デバッグモード
npx playwright test --debug
```

### ブラウザ指定
```bash
# 特定のブラウザのみ
npx playwright test --project=chromium
npx playwright test --project=firefox
npx playwright test --project=webkit

# 複数ブラウザ
npx playwright test --project=chromium --project=firefox
```

### フィルタリング
```bash
# 特定のファイルのみテスト
npx playwright test tests/login.spec.ts

# テスト名でフィルタ
npx playwright test -g "login flow"

# タグでフィルタ
npx playwright test --grep @smoke
```

### レポート
```bash
# HTML レポートを生成して開く
npx playwright show-report

# テスト結果を JSON で出力
npx playwright test --reporter=json

# 複数のレポーター
npx playwright test --reporter=html --reporter=json
```

### トレース
```bash
# トレースを有効化
npx playwright test --trace on

# トレースビューアーを開く
npx playwright show-trace trace.zip
```

### その他
```bash
# 失敗したテストのみ再実行
npx playwright test --last-failed

# 並列実行の制御
npx playwright test --workers=4

# シャーディング（複数マシンでの実行）
npx playwright test --shard=1/3
```

## React Testing Library

### コンポーネントテスト
```bash
# React コンポーネントのテスト
npm test -- --testPathPattern=components

# 特定のコンポーネントのテスト
npm test -- Button.test.tsx
```

## E2E テスト (Cypress)

### 基本実行
```bash
# Cypress を開く
npx cypress open

# ヘッドレスで実行
npx cypress run

# 特定のブラウザで実行
npx cypress run --browser chrome
npx cypress run --browser firefox

# 特定のスペックファイルを実行
npx cypress run --spec "cypress/e2e/login.cy.ts"
```

### 設定
```bash
# 特定の設定ファイルを使用
npx cypress run --config-file cypress.config.production.ts

# ベースURLを指定
npx cypress run --config baseUrl=https://staging.example.com
```

## テストデータベースのセットアップ

### Prisma でのテストDB
```bash
# テスト用データベースのマイグレーション
DATABASE_URL="postgresql://localhost/test_db" npx prisma migrate deploy

# テスト用データベースのリセット
DATABASE_URL="postgresql://localhost/test_db" npx prisma migrate reset --skip-seed

# テストデータのシード
DATABASE_URL="postgresql://localhost/test_db" npx prisma db seed
```

## 統合テスト

### API テスト
```bash
# Supertest を使用したAPIテスト
npm test -- --testPathPattern=api

# 特定のエンドポイントのテスト
npm test -- api/users.test.ts
```

## パフォーマンステスト

### Lighthouse CI
```bash
# Lighthouse でパフォーマンステスト
npx lighthouse-ci autorun

# 特定のURLをテスト
npx lighthouse https://example.com --view
```

## テストのデバッグ

### Node.js デバッガー
```bash
# Jest をデバッグ
node --inspect-brk node_modules/.bin/jest --runInBand

# VSCode でデバッグ設定（launch.json）
{
  "type": "node",
  "request": "launch",
  "name": "Jest Debug",
  "program": "${workspaceFolder}/node_modules/.bin/jest",
  "args": ["--runInBand", "--no-cache"],
  "console": "integratedTerminal",
  "internalConsoleOptions": "neverOpen"
}
```

## CI/CD でのテスト実行

### GitHub Actions での例
```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 18
      - run: npm ci
      - run: npm test -- --ci --coverage
      - uses: codecov/codecov-action@v3
```

## 便利なテストスクリプト（package.json）

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "test:unit": "jest --testPathPattern=unit",
    "test:integration": "jest --testPathPattern=integration",
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "test:debug": "node --inspect-brk node_modules/.bin/jest --runInBand",
    "test:ci": "jest --ci --coverage --maxWorkers=2"
  }
}
```

## テスト前の準備コマンド

```bash
# データベースのセットアップとテスト実行
npm run db:test:setup && npm test

# Docker でテスト用DBを起動してテスト
docker compose -f docker-compose.test.yml up -d && npm test

# テスト後のクリーンアップ
npm test; docker compose -f docker-compose.test.yml down
```

## まとめ：よく使うテストコマンド

```bash
# 開発中（ウォッチモード）
npm test -- --watch

# コミット前（カバレッジチェック）
npm test -- --coverage

# CI/CD
npm test -- --ci --coverage

# デバッグ
npm test -- --debug

# E2E（ローカル）
npx playwright test --ui

# E2E（CI）
npx playwright test --workers=2
```
