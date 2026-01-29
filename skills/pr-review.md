# PR レビュースキル（Claude Code Skills 対応）

## 概要

このスキルは、Claude Code の Skills 機能を活用した自動 PR レビューの実践ガイドです。
`.claude/skills/` ディレクトリに配置することで、`/review` コマンドや GitHub Actions 経由で一貫性のある高品質なコードレビューを実行できます。

参考:
- [Claude Code Skills による自動コードレビュー実装ガイド](https://zenn.dev/neurostack_0001/articles/claude-code-skills-review-automation)

## Claude Code Skills とは

`.claude/skills/` ディレクトリにマークダウンファイルを配置するだけで、AI に判断ルールを教える仕組みです。

### 主な利点

- **レビュー基準の統一**: チーム全体で一貫したレビュー観点を適用できる
- **自動化**: GitHub Actions と連携し、PR 作成時に自動レビューを実行
- **即時フィードバック**: ローカルで `/review` コマンドを使えば即座にレビュー結果を取得
- **段階的な成長**: 使いながらルールを追加・改善できる

## スキルファイルの配置構造

```
.claude/
└── skills/
    └── code-review/
        ├── SKILL.md          # メインスキル定義（description が最重要）
        └── examples.md       # レビュー出力の具体例
```

## SKILL.md テンプレート

以下は `.claude/skills/code-review/SKILL.md` として配置するテンプレートです。
プロジェクトに合わせてカスタマイズして使用してください。

````markdown
---
description: "PR をレビューする・コードレビューする・このコード見て・差分を確認して・変更をチェックして - Pull Request のコード品質をセキュリティ、パフォーマンス、可読性、テストの4観点でレビューします"
---

# Code Review Skill

## レビュー実行手順

1. PR の説明（タイトル、本文、関連 Issue）を確認する
2. 変更ファイルの差分を取得し、全体の変更規模と影響範囲を把握する
3. 以下の4つの観点で各ファイルを精査する
4. 指定された出力フォーマットに従ってレビュー結果を出力する

## レビュー観点（4層構造）

### 1. セキュリティ

- SQL インジェクション・XSS・CSRF の脆弱性がないか
- 認証・認可の処理が適切か（トークン検証、権限チェック）
- 機密情報（API キー、パスワード、トークン）がハードコードされていないか
- 入力バリデーションが適切に行われているか（Zod 等のスキーマ検証）
- 依存パッケージに既知の脆弱性がないか

### 2. パフォーマンス

- N+1 クエリが発生していないか
- 不要なループ・再レンダリングがないか
- 適切なインデックスが設定されているか
- メモ化（useMemo, useCallback, React.memo）が必要な箇所で使われているか
- 大量データ処理にページネーションやストリーミングが使われているか

### 3. 可読性

- 変数名・関数名が意図を明確に表しているか
- 関数が単一責任になっているか（長大な関数がないか）
- ネストが深すぎないか（早期リターンで改善可能か）
- マジックナンバーが定数化されているか
- プロジェクトの命名規約・コーディング規約に従っているか

### 4. テスト

- 変更に対応するテストが追加・更新されているか
- エッジケース（境界値、null/undefined、空配列）が網羅されているか
- テストが実装の詳細に依存していないか（振る舞いベースか）
- モックが適切に使用されているか（過剰モックでないか）
- CI でテストが通っているか

## 出力フォーマット

以下のフォーマットでレビュー結果を出力してください：

```
### サマリ
[変更の概要と全体的な評価を 1-2 文で記述]

---

#### 🔴 Blocking（マージ前に修正必須）
1. **[問題の概要]**
   ファイル名:行番号 - [具体的な問題の説明]
   → [修正案またはコード例]

#### 🟡 Suggestions（非ブロッキングの改善提案）
* ファイル名:行番号 - [改善提案の説明]
* ファイル名:行番号 - [改善提案の説明]

#### 🟢 Good（良かった点）
* [良かった点の説明]

#### ❓ Questions（確認したい点）
* [意図が不明瞭な箇所への質問]

---

**Decision**: Approve / Request Changes / Comment Only
```

## トーンガイド

- 敬意とポジティブさを忘れず、善意を前提にする
- コードにフォーカスし、作者個人を批判しない
- 命令より提案を優先：「〜を検討してみてはいかがでしょうか？」
- 良かった点も必ず示し、称賛と指摘のバランスを取る
- 具体的な修正案やコード例を添える
````

## description の書き方（最重要）

記事によると、**description の書き方でスキルの呼び出し率の 9 割が決まります**。

### ポイント

1. **トリガーワードを明示的に含める**: AI がスキルを選択する際に description を参照するため、ユーザーが使いそうな言葉を網羅する
2. **50〜100 文字で具体的に**: 曖昧な記述ではスキルが呼び出されない
3. **何をするスキルかを明確に**: 入力と出力を示す

### 良い例

```yaml
description: "PR をレビューする・コードレビューする・このコード見て・差分を確認して・変更をチェックして - Pull Request のコード品質をセキュリティ、パフォーマンス、可読性、テストの4観点でレビューします"
```

### 悪い例

```yaml
# 曖昧すぎて呼び出されない
description: "コードの品質を確認します"

# トリガーワードがない
description: "Pull Request の品質チェックを行うスキル"
```

## GitHub Actions による自動レビュー

PR 作成時に自動でレビューを実行する GitHub Actions の設定例です。

### ワークフロー設定（.github/workflows/claude-review.yml）

```yaml
name: Claude Code Review

on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
  pull_request:
    types: [opened, synchronize]

permissions:
  contents: write
  pull-requests: write
  issues: write
  id-token: write

jobs:
  review:
    if: |
      (github.event_name == 'pull_request') ||
      (github.event_name == 'issue_comment' && contains(github.event.comment.body, '@claude'))
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

### 必要な設定

1. リポジトリの Settings → Secrets に `ANTHROPIC_API_KEY` を登録
2. GitHub App または PAT で適切な権限を付与
3. `.claude/skills/code-review/SKILL.md` をリポジトリに配置

## ローカルでの使い方

### /review コマンド

Claude Code のターミナルで直接レビューを実行できます。

```bash
# 現在のブランチの PR をレビュー
/review

# PR 番号を指定してレビュー
/review 123

# 特定のファイルだけレビュー
gh pr diff 123 -- src/components/ | claude "このコードをレビューしてください"
```

### PR 上での @claude メンション

PR のコメントで `@claude` をメンションすると、自動でレビューやコード修正を実行します。

```text
# レビューを依頼
@claude この PR をレビューしてください

# 修正を依頼
@claude セキュリティの問題を修正してください

# 特定の観点でレビュー
@claude パフォーマンスの観点でレビューしてください
```

## examples.md テンプレート

以下は `.claude/skills/code-review/examples.md` として配置する具体例です。

````markdown
# レビュー出力例

## 例1: API エンドポイントの追加

### サマリ
ユーザー検索 API の追加。基本的な実装は適切ですが、入力バリデーションに改善の余地があります。

---

#### 🔴 Blocking（マージ前に修正必須）
1. **SQL インジェクションのリスク**
   src/api/users.ts:24 - クエリパラメータが直接 SQL に埋め込まれている
   → Prisma のパラメータバインディングを使用してください：
   ```typescript
   const users = await prisma.user.findMany({
     where: { name: { contains: query } },
   });
   ```

#### 🟡 Suggestions（非ブロッキングの改善提案）
* src/api/users.ts:15 - リクエストパラメータに Zod スキーマを適用するとより安全です
* src/api/users.ts:30 - ページネーションの追加を検討してください（大量データ対応）

#### 🟢 Good（良かった点）
* エラーハンドリングが適切に実装されている
* レスポンスの型定義が明確

#### ❓ Questions（確認したい点）
* 検索結果の上限は設ける予定ですか？

---

**Decision**: Request Changes

## 例2: React コンポーネントのリファクタリング

### サマリ
UserProfile コンポーネントの分割リファクタリング。可読性が大幅に向上しています。

---

#### 🔴 Blocking（マージ前に修正必須）
なし

#### 🟡 Suggestions（非ブロッキングの改善提案）
* src/components/UserProfile/index.tsx:45 - `useCallback` でメモ化するとリスト再レンダリングを防げます
* src/components/UserProfile/Avatar.tsx:12 - alt 属性にユーザー名を含めるとアクセシビリティが向上します

#### 🟢 Good（良かった点）
* 単一責任の原則に従った適切なコンポーネント分割
* テストが変更に追従して更新されている
* 型定義が明確で、Props の責務が分かりやすい

#### ❓ Questions（確認したい点）
なし

---

**Decision**: Approve
````

## スキル運用のベストプラクティス

### 1. 小さく始めて育てる

最初から完璧なスキルファイルを目指さず、最低限の観点から始めて使いながら改善していきます。

```
Week 1: セキュリティとテストの基本チェックだけ
Week 2: パフォーマンス観点を追加
Week 3: プロジェクト固有のルールを追加
Week 4: 出力フォーマットを調整
```

### 2. ファイルサイズの管理

- **SKILL.md は 500 行以下** に抑える
- 詳細な例や補足情報は別ファイル（examples.md など）に分離
- 長いセッションでは `/compact` コマンドでコンテキストを圧縮

### 3. プロジェクト固有ルールの追加

SKILL.md のレビュー観点にプロジェクト固有のルールを追加できます。

```markdown
## プロジェクト固有のルール

- API エンドポイントは必ず `/api/v1/` プレフィックスを使用する
- データベースアクセスは Repository パターンで実装する
- 環境変数は Zod で型安全にバリデーションする
- コンポーネントは Storybook のストーリーを含める
```

### 4. チームでの共有

```bash
# スキルファイルをリポジトリに含める
git add .claude/skills/code-review/
git commit -m "feat: add Claude Code review skill"
git push

# チームメンバーは clone するだけで同じレビュー基準を共有
```

## 注意事項

- Claude Code の Skills 機能は **Pro / Max プラン以上** が必要（月 $20〜）
- GitHub Actions 経由の場合は **API キーの従量課金** が発生する
- 自動レビューの実行には **3〜4 分** かかるため、急ぎの場合はローカルの `/review` を使用
- AI レビューは補助ツールであり、**最終判断は必ず人間が行う**こと
- description のトリガーワードが不足していると **スキルが呼び出されない**（最もよくある問題）
