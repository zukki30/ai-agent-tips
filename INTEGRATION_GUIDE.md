# 使用例・統合ガイド

このファイルでは、ai-agent-tips を実際のプロジェクトに統合する具体例を示します。

## 1. Next.js + TypeScript プロジェクトでの使用

### セットアップ
```bash
# Next.js プロジェクトを作成
npx create-next-app@latest my-app --typescript --tailwind --app

cd my-app

# ai-agent-tips をクローン
git clone https://github.com/zukki30/ai-agent-tips.git .ai-tips

# AGENT.md をプロジェクトルートにコピー
cp .ai-tips/AGENT.md .

# Cursor用の設定
cp AGENT.md .cursorrules
```

### AI エージェントへの指示例

#### 例1: 新機能の実装
```
AGENT.md とスキルファイル（skills/nextjs.md, skills/react-typescript.md）を参照して、
以下の要件でユーザープロフィールページを実装してください：

- App Router を使用
- Server Components を優先
- 型安全性を確保
- React Testing Library でテストを作成
- アクセシビリティに配慮

パス: app/profile/[id]/page.tsx
```

#### 例2: API エンドポイントの作成
```
skills/nextjs.md の API Routes セクションを参照して、
ユーザー情報を取得する API エンドポイントを作成してください：

- Route Handlers (App Router) を使用
- Zod でバリデーション
- エラーハンドリングを適切に実装
- TypeScript で型安全に

パス: app/api/users/[id]/route.ts
```

## 2. Nest.js バックエンドプロジェクトでの使用

### セットアップ
```bash
# Nest.js プロジェクトを作成
npm install -g @nestjs/cli
nest new my-api

cd my-api

# ai-agent-tips をクローン
git clone https://github.com/zukki30/ai-agent-tips.git .ai-tips

# AGENT.md をプロジェクトルートにコピー
cp .ai-tips/AGENT.md .
```

### AI エージェントへの指示例

#### 例: CRUD リソースの実装
```
AGENT.md と skills/nestjs.md を参照して、
商品（Product）の CRUD 機能を実装してください：

要件：
- モジュール、コントローラー、サービスの適切な分離
- DTO でバリデーション
- Prisma ORM を使用
- ユニットテストと E2E テストを含める
- JWT 認証ガードを適用
- Swagger ドキュメントを生成

実装するファイル：
- src/modules/products/products.module.ts
- src/modules/products/products.controller.ts
- src/modules/products/products.service.ts
- src/modules/products/dto/create-product.dto.ts
- src/modules/products/entities/product.entity.ts
```

## 3. Hono で軽量 API を構築

### セットアップ
```bash
# プロジェクトを作成
mkdir my-hono-api && cd my-hono-api
npm init -y

# 依存関係をインストール
npm install hono
npm install -D typescript @types/node tsx

# ai-agent-tips をクローン
git clone https://github.com/zukki30/ai-agent-tips.git .ai-tips
cp .ai-tips/AGENT.md .
```

### AI エージェントへの指示例
```
skills/hono.md を参照して、以下の機能を持つ REST API を実装してください：

エンドポイント：
- GET /api/tasks - タスク一覧
- GET /api/tasks/:id - タスク詳細
- POST /api/tasks - タスク作成
- PUT /api/tasks/:id - タスク更新
- DELETE /api/tasks/:id - タスク削除

要件：
- Zod でバリデーション
- JWT 認証ミドルウェア
- エラーハンドリング
- CORS 設定
- 型安全な RPC モード対応
```

## 4. テスト駆動開発（TDD）での使用

### AI エージェントへの指示例
```
skills/unit-test.md を参照して、TDD でユーザー認証機能を実装してください：

手順：
1. まず、以下のテストケースを作成
   - メールアドレスとパスワードで認証できる
   - 無効な認証情報でエラーが返る
   - JWT トークンが正しく生成される
   - トークンの有効期限が設定される

2. テストをパスする最小限の実装を作成

3. リファクタリング
```

## 5. E2E テストの実装

### セットアップ
```bash
# Playwright をインストール
npm init playwright@latest

# ai-agent-tips を参照
cp .ai-tips/skills/e2e-playwright.md docs/
```

### AI エージェントへの指示例
```
skills/e2e-playwright.md を参照して、ログインから商品購入までの
E2E テストを実装してください：

シナリオ：
1. トップページにアクセス
2. ログインボタンをクリック
3. 認証情報を入力してログイン
4. 商品一覧ページに遷移
5. 商品を選択してカートに追加
6. チェックアウトページで注文確定
7. 注文完了ページが表示されることを確認

テストファイル: tests/e2e/purchase-flow.spec.ts
```

## 6. プロジェクト全体での統合

### プロジェクト構成に組み込む
```bash
# ドキュメントディレクトリに配置
mkdir -p docs/ai-guidelines
cp -r .ai-tips/skills docs/ai-guidelines/
cp -r .ai-tips/commands docs/ai-guidelines/
cp .ai-tips/AGENT.md docs/ai-guidelines/

# package.json に参照を追加
```

```json
{
  "scripts": {
    "docs": "open docs/ai-guidelines/README.md",
    "dev:guide": "echo 'See docs/ai-guidelines/ for AI Agent guidelines'"
  }
}
```

### チーム開発での活用

#### プルリクエストテンプレート
```markdown
## チェックリスト

コードレビュー前に以下を確認してください：

- [ ] AGENT.md のコーディング規約に準拠
- [ ] 適切な型定義がされている
- [ ] テストが含まれている（カバレッジ80%以上）
- [ ] セキュリティチェックをパス
- [ ] パフォーマンスの考慮がされている

参考：`docs/ai-guidelines/AGENT.md`
```

## 7. Claude Desktop でグローバル設定

### macOS
```bash
# すべてのプロジェクトで使用できるように設定
mkdir -p ~/Library/Application\ Support/Claude/skills
cp -r .ai-tips/skills/* ~/Library/Application\ Support/Claude/skills/
cp .ai-tips/AGENT.md ~/Library/Application\ Support/Claude/
```

### プロジェクトを開いた時の指示例
```
このプロジェクトは React + Next.js を使用しています。
~/Library/Application Support/Claude/skills/ にある
nextjs.md と react-typescript.md のガイドラインに従って開発してください。
```

## 8. カスタマイズ例

### プロジェクト固有のルールを追加

```markdown
# AGENT.md に以下を追加

## プロジェクト固有のルール

### スタイリング
- Tailwind CSS を使用
- カラーパレット: primary (#3B82F6), secondary (#10B981)
- ダークモードに対応

### API仕様
- すべてのAPIは `/api/v1/` プレフィックス
- レスポンスは統一フォーマット:
  ```json
  {
    "success": true,
    "data": {},
    "error": null
  }
  ```

### コミットメッセージ
- Conventional Commits に従う
- 例: `feat: add user authentication`
```

## 9. CI/CD での活用

### GitHub Actions での例
```yaml
name: Code Quality Check

on: [pull_request]

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Check Coding Standards
        run: |
          # AGENT.md のガイドラインに従っているかチェック
          npm run lint
          npx tsc --noEmit
          
      - name: Run Tests
        run: |
          # skills/unit-test.md の基準を満たしているか
          npm test -- --coverage
          npm run test:e2e
```

## 10. トラブルシューティング

### よくある問題と解決策

#### 問題: AI エージェントがガイドラインを無視する
```
解決策：
プロンプトで明示的に参照を指示する

例：
"必ず AGENT.md の 'セキュリティガイドライン' セクションを参照し、
入力バリデーションとサニタイゼーションを実装してください"
```

#### 問題: コマンドが見つからない
```
解決策：
commands/ ディレクトリのファイルを参照

例：
"commands/testing.md を確認して、Playwright の E2E テストを実行してください"
```

## まとめ

ai-agent-tips は以下の場面で活用できます：

1. **新規プロジェクト**: 一貫した設計思想で開始
2. **既存プロジェクト**: 段階的なリファクタリング
3. **チーム開発**: 共通のコーディング規約
4. **学習**: ベストプラクティスの参照
5. **コードレビュー**: レビュー基準として使用

AI エージェントと協力して、高品質なコードを効率的に開発しましょう！
