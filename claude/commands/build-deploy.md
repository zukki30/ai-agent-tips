# ビルド・デプロイコマンド集

## Node.js プロジェクトのビルド

### TypeScript のビルド
```bash
# 本番ビルド
npm run build

# TypeScript コンパイル
npx tsc

# ウォッチモード
npx tsc --watch

# クリーンビルド
rm -rf dist && npx tsc
```

### バンドラーを使用したビルド

#### Vite
```bash
# 開発ビルド
npm run dev

# 本番ビルド
npm run build

# プレビュー（ビルド結果の確認）
npm run preview

# ビルドの解析
npm run build -- --mode production --report
```

#### Webpack
```bash
# 開発ビルド
npm run build:dev

# 本番ビルド
npm run build:prod

# ウォッチモード
npm run build -- --watch

# バンドルアナライザー
npm run build -- --analyze
```

#### esbuild
```bash
# 高速ビルド
npx esbuild src/index.ts --bundle --outfile=dist/index.js --minify

# ウォッチモード
npx esbuild src/index.ts --bundle --outfile=dist/index.js --watch
```

## React / Next.js のビルド

### Next.js
```bash
# 本番ビルド
npm run build

# 本番起動
npm start

# 静的エクスポート（next.config.js に output: 'export' を設定）
npm run build

# スタンドアロンビルド
next build

# ビルド解析
ANALYZE=true npm run build
```

## Nest.js のビルド

```bash
# 本番ビルド
npm run build

# クリーンビルド
rm -rf dist && npm run build

# ウォッチビルド
npm run build -- --watch
```

## Docker を使用したビルド

### Dockerfile のビルド
```bash
# イメージのビルド
docker build -t my-app:latest .

# ビルド引数を指定
docker build -t my-app:latest --build-arg NODE_ENV=production .

# マルチステージビルドのターゲット指定
docker build --target production -t my-app:prod .

# ビルドキャッシュを使わない
docker build --no-cache -t my-app:latest .

# 別のDockerfileを使用
docker build -f Dockerfile.prod -t my-app:prod .
```

### Docker Compose でのビルド
```bash
# サービスのビルド
docker compose build

# キャッシュなしでビルド
docker compose build --no-cache

# 並列ビルド
docker compose build --parallel

# ビルドして起動
docker compose up --build
```

## デプロイコマンド

### Vercel
```bash
# Vercel CLI のインストール
npm install -g vercel

# ログイン
vercel login

# デプロイ（プレビュー）
vercel

# 本番デプロイ
vercel --prod

# 環境変数の設定
vercel env add VARIABLE_NAME

# プロジェクトのリンク
vercel link
```

### Netlify
```bash
# Netlify CLI のインストール
npm install -g netlify-cli

# ログイン
netlify login

# デプロイ
netlify deploy

# 本番デプロイ
netlify deploy --prod

# ビルドしてデプロイ
netlify deploy --build

# 環境変数の設定
netlify env:set VARIABLE_NAME value
```

### AWS (Amplify)
```bash
# Amplify CLI のインストール
npm install -g @aws-amplify/cli

# 初期化
amplify init

# バックエンドのプッシュ
amplify push

# 本番環境へのデプロイ
amplify publish

# ホスティングの追加
amplify add hosting
```

### Heroku
```bash
# Heroku CLI でログイン
heroku login

# アプリの作成
heroku create my-app

# デプロイ
git push heroku main

# ログの確認
heroku logs --tail

# 環境変数の設定
heroku config:set NODE_ENV=production

# スケールの設定
heroku ps:scale web=1
```

### Railway
```bash
# Railway CLI のインストール
npm install -g @railway/cli

# ログイン
railway login

# プロジェクトの初期化
railway init

# デプロイ
railway up

# ログの確認
railway logs
```

## Cloudflare Workers のデプロイ

### Wrangler
```bash
# Wrangler のインストール
npm install -g wrangler

# ログイン
wrangler login

# 新規プロジェクト作成
wrangler init

# 開発サーバー起動
wrangler dev

# デプロイ
wrangler deploy

# シークレットの設定
wrangler secret put SECRET_NAME

# ログの確認
wrangler tail
```

## デプロイ前のチェック

### リンティングとフォーマット
```bash
# ESLint でコードチェック
npm run lint

# 自動修正
npm run lint -- --fix

# Prettier でフォーマット
npm run format

# フォーマットチェック
npm run format:check
```

### テストの実行
```bash
# すべてのテストを実行
npm test

# カバレッジ付きでテスト
npm test -- --coverage

# E2Eテスト
npm run test:e2e
```

### 型チェック
```bash
# TypeScript の型チェック
npx tsc --noEmit

# 厳密な型チェック
npx tsc --noEmit --strict
```

### セキュリティチェック
```bash
# 脆弱性スキャン
npm audit

# 自動修正
npm audit fix

# 強制的に修正
npm audit fix --force

# Snyk でセキュリティチェック
npx snyk test
```

## ビルドサイズの最適化

### バンドルサイズの解析
```bash
# Next.js のバンドル解析
ANALYZE=true npm run build

# webpack-bundle-analyzer
npx webpack-bundle-analyzer stats.json

# source-map-explorer
npm run build && npx source-map-explorer 'dist/**/*.js'
```

### 未使用コードの削除
```bash
# depcheck で未使用の依存関係を確認
npx depcheck

# 未使用のエクスポートを検出
npx ts-prune
```

## 環境別のビルド

### 環境変数の管理
```bash
# 開発環境
NODE_ENV=development npm run build

# ステージング環境
NODE_ENV=staging npm run build

# 本番環境
NODE_ENV=production npm run build

# カスタム環境変数ファイルを使用
dotenv -e .env.production npm run build
```

## CI/CD パイプライン

### GitHub Actions の例
```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 18
      - run: npm ci
      - run: npm run build
      - run: npm test
      - name: Deploy to Vercel
        run: vercel --prod --token=${{ secrets.VERCEL_TOKEN }}
```

### GitLab CI の例
```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - deploy

build:
  stage: build
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/

test:
  stage: test
  script:
    - npm test

deploy:
  stage: deploy
  script:
    - npm run deploy
  only:
    - main
```

## ロールバック

### Vercel
```bash
# デプロイ履歴の確認
vercel ls

# 特定のデプロイメントにロールバック
vercel rollback <deployment-url>
```

### Heroku
```bash
# リリース履歴の確認
heroku releases

# 前のリリースにロールバック
heroku rollback

# 特定のバージョンにロールバック
heroku rollback v123
```

## デプロイ後の確認

### ヘルスチェック
```bash
# アプリケーションの起動確認
curl https://my-app.com/health

# レスポンスタイムの確認
curl -w "@curl-format.txt" -o /dev/null -s https://my-app.com

# SSL証明書の確認
openssl s_client -connect my-app.com:443 -servername my-app.com
```

### ログの確認
```bash
# Vercel のログ
vercel logs

# Heroku のログ
heroku logs --tail

# Netlify のログ
netlify logs

# AWS CloudWatch のログ
aws logs tail /aws/lambda/function-name --follow
```

## まとめ：デプロイフロー

```bash
# 1. コードの品質チェック
npm run lint && npm run format:check

# 2. 型チェック
npx tsc --noEmit

# 3. テストの実行
npm test -- --coverage

# 4. セキュリティチェック
npm audit

# 5. ビルド
npm run build

# 6. デプロイ
vercel --prod

# 7. デプロイ後の確認
curl https://my-app.com/health
```
