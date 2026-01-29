# AI Agent Tips

Claude Code、GitHub Copilot、Cursor などの AI エージェントでの開発を快適にするための包括的なスキル、コマンド、ガイドラインをまとめたリポジトリです。

## 📚 概要

このリポジトリには、モダンな Web 開発に必要な技術スタックとベストプラクティスが日本語でまとめられています。これらのファイルは、様々なプロジェクトで使用したり、PC の `.claude` ディレクトリに設定したりすることを想定しています。

## 🎯 対象技術スタック

- React
- TypeScript
- Node.js
- Nest.js
- Next.js
- Unit Test (Jest/Vitest)
- E2E Test (Playwright)
- Hono

## 📁 ファイル構成

```
ai-agent-tips/
├── skills/                    # 技術別スキルドキュメント
│   ├── react-typescript.md    # React + TypeScript のベストプラクティス
│   ├── nextjs.md              # Next.js 開発ガイド
│   ├── nestjs.md              # Nest.js バックエンド開発
│   ├── nodejs.md              # Node.js 開発基礎
│   ├── unit-test.md           # ユニットテストの書き方
│   ├── e2e-playwright.md      # Playwright E2E テスト
│   ├── hono.md                # Hono フレームワーク
│   ├── tdd.md                 # TDD & レガシーコード改善
│   └── ai-review.md           # AI PRレビュー & 人名プロンプト
├── commands/                  # よく使うコマンド集
│   ├── development.md         # 開発コマンド
│   ├── testing.md             # テスト実行コマンド
│   └── build-deploy.md        # ビルド・デプロイコマンド
├── AGENT.md                   # AI エージェント向けガイドライン
└── README.md                  # このファイル
```

## 🚀 使い方

### 1. プロジェクトで使用する

#### 方法A: リポジトリをクローンして参照
```bash
# プロジェクトのルートで
git clone https://github.com/zukki30/ai-agent-tips.git .ai-tips
```

#### 方法B: 必要なファイルをコピー
```bash
# 特定のスキルファイルをコピー
cp ai-agent-tips/skills/react-typescript.md docs/
cp ai-agent-tips/AGENT.md .
```

### 2. Claude Desktop で使用する

Claude Desktop の設定に追加して、すべてのプロジェクトで参照できるようにします。

#### macOS
```bash
# Claude の設定ディレクトリにコピー
cp -r skills ~/Library/Application\ Support/Claude/skills/
cp AGENT.md ~/Library/Application\ Support/Claude/
```

#### Windows
```powershell
# Claude の設定ディレクトリにコピー
Copy-Item -Recurse skills "$env:APPDATA\Claude\skills\"
Copy-Item AGENT.md "$env:APPDATA\Claude\"
```

#### Linux
```bash
# Claude の設定ディレクトリにコピー
cp -r skills ~/.config/Claude/skills/
cp AGENT.md ~/.config/Claude/
```

### 3. Cursor や GitHub Copilot での使用

プロジェクトルートに `.cursorrules` や `.github/copilot-instructions.md` として配置します。

```bash
# Cursor用
cp AGENT.md .cursorrules

# GitHub Copilot用
mkdir -p .github
cp AGENT.md .github/copilot-instructions.md
```

## 📖 各ファイルの説明

### Skills（スキルドキュメント）

各技術スタックの詳細なベストプラクティス、コーディング規約、サンプルコードを含んでいます。

- **react-typescript.md**: コンポーネント設計、Hooks の使用方法、パフォーマンス最適化
- **nextjs.md**: App Router、Server Components、データフェッチング、SEO対策
- **nestjs.md**: モジュール設計、DI、ガード、インターセプター、テスト
- **nodejs.md**: 非同期処理、ストリーム、エラーハンドリング、セキュリティ
- **unit-test.md**: Jest/Vitest、モック、React Testing Library
- **e2e-playwright.md**: Playwright の設定、ロケーター、API テスト、POM
- **hono.md**: ルーティング、ミドルウェア、バリデーション、RPC モード

### Commands（コマンド集）

開発でよく使うコマンドを体系的にまとめています。

- **development.md**: npm、Git、Docker、データベースなどの基本コマンド
- **testing.md**: Jest、Vitest、Playwright、Cypress のテスト実行コマンド
- **build-deploy.md**: ビルド、デプロイ、CI/CD、ロールバックのコマンド

### AGENT.md

AI エージェントが開発を行う際の包括的なガイドライン。

- 開発原則（型安全性、関数型プログラミング、SOLID原則）
- コーディング規約（命名規則、インポート順序、コメント）
- プロジェクト構造（Next.js、Nest.js）
- 開発フロー（ブランチ戦略、テスト、デプロイ）
- セキュリティガイドライン
- パフォーマンス最適化
- エラーハンドリング

## 💡 活用例

### 新規プロジェクトの開始時
```bash
# 1. プロジェクト作成
npx create-next-app@latest my-app --typescript

# 2. ai-agent-tips をクローン
cd my-app
git clone https://github.com/zukki30/ai-agent-tips.git .ai-tips

# 3. AGENT.md をプロジェクトルートにコピー
cp .ai-tips/AGENT.md .

# 4. AI エージェントに指示
# "AGENT.md のガイドラインに従って、ユーザー認証機能を実装してください"
```

### 既存プロジェクトへの適用
```bash
# 1. 現在のプロジェクト構造を確認
# 2. 該当する skills ファイルを参照
# 3. コーディング規約に従ってリファクタリング
```

### チーム開発での共有
```bash
# プロジェクトの docs/ にコピー
mkdir -p docs/ai-guidelines
cp -r ai-agent-tips/skills docs/ai-guidelines/
cp ai-agent-tips/AGENT.md docs/ai-guidelines/

# チームメンバーに周知
# AI エージェントを使用する際は docs/ai-guidelines/ を参照
```

## 🔧 カスタマイズ

プロジェクト固有のルールがある場合は、AGENT.md をカスタマイズして使用できます。

```markdown
# AGENT.md に追加

## プロジェクト固有のルール

### API命名規則
- すべてのAPIエンドポイントは `/api/v1/` で始める
- REST原則に従う（GET, POST, PUT, DELETE）

### データベース
- PostgreSQL を使用
- Prisma ORM を使用
- マイグレーションファイルは必ず git 管理
```

## 🤝 コントリビューション

改善提案やバグ報告は Issue や Pull Request でお願いします。

1. このリポジトリをフォーク
2. フィーチャーブランチを作成 (`git checkout -b feature/amazing-feature`)
3. 変更をコミット (`git commit -m 'Add some amazing feature'`)
4. ブランチにプッシュ (`git push origin feature/amazing-feature`)
5. Pull Request を作成

## 📝 ライセンス

MIT License - 自由に使用、改変、配布できます。

## 🙏 謝辞

このリポジトリは Claude の公式ドキュメント（[Agent Skills Best Practices](https://docs.anthropic.com/en/docs/build-with-claude/agent-skills/best-practices)）を参考に作成されています。

## 📞 サポート

質問や提案がある場合は、GitHub Issues をご利用ください。

---

**Happy Coding with AI Agents! 🚀**
