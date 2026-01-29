# AGENT.md - AI Agent 開発ガイドライン

このドキュメントは、AIエージェント（Claude Code、GitHub Copilot、Cursor等）がこのリポジトリやこれらのスキルを使用したプロジェクトで開発を行う際のガイドラインを提供します。

## 会話ガイドライン

- 常に日本語で会話する

## 目次

- [開発原則](#開発原則)
- [コーディング規約](#コーディング規約)
- [プロジェクト構造](#プロジェクト構造)
- [技術スタック](#技術スタック)
- [開発フロー](#開発フロー)
- [Spec-Driven Development](#spec-driven-development)
- [テスト戦略](#テスト戦略)
- [セキュリティガイドライン](#セキュリティガイドライン)
- [パフォーマンス最適化](#パフォーマンス最適化)
- [エラーハンドリング](#エラーハンドリング)
- [ドキュメンテーション](#ドキュメンテーション)
- [PR レビューガイドライン](#pr-レビューガイドライン)

## 開発原則

### 1. 型安全性を最優先
- **TypeScript を必ず使用**し、`any` 型の使用は避ける
- すべての関数、Props、State に明示的な型定義を行う
- `strict` モードを有効にし、型チェックを厳格に実行

```typescript
// Good
type UserData = {
  id: string;
  name: string;
  email: string;
};

function processUser(user: UserData): void {
  // 処理
}

// Bad
function processUser(user: any) {
  // 処理
}
```

### 2. 関数型プログラミングの活用
- イミュータブルなデータ構造を使用
- 純粋関数を優先
- 副作用は最小限に抑え、明示的に管理

```typescript
// Good: イミュータブルな更新
const updatedUser = { ...user, name: 'New Name' };

// Bad: ミュータブルな更新
user.name = 'New Name';
```

### 3. DRY原則 (Don't Repeat Yourself)
- コードの重複を避ける
- 共通ロジックは関数、カスタムフック、ユーティリティとして抽出
- コンポーネントは再利用可能な単位で設計

### 4. SOLID原則の遵守
- **単一責任の原則**: 1つのモジュール、クラス、関数は1つの責任のみ
- **開放閉鎖の原則**: 拡張には開いており、修正には閉じている
- **依存性逆転の原則**: 具象ではなく抽象に依存

## コーディング規約

### 命名規則

#### ファイル・ディレクトリ
```
コンポーネント: PascalCase (例: UserProfile.tsx)
フック: camelCase (例: useUserData.ts)
ユーティリティ: camelCase (例: formatDate.ts)
型定義: PascalCase (例: User.types.ts)
定数: UPPER_SNAKE_CASE (例: API_ENDPOINTS.ts)
```

#### 変数・関数
```typescript
// 変数: camelCase
const userName = 'John';
const isActive = true;

// 定数: UPPER_SNAKE_CASE
const MAX_RETRY_COUNT = 3;
const API_BASE_URL = 'https://api.example.com';

// 関数: camelCase（動詞で始める）
function fetchUser() {}
function createPost() {}
function validateEmail() {}

// Boolean: is/has/can で始める
const isValid = true;
const hasPermission = false;
const canEdit = true;

// イベントハンドラ: handle で始める
const handleClick = () => {};
const handleSubmit = () => {};
```

#### React コンポーネント
```typescript
// コンポーネント: PascalCase
// Props 型を先に定義し、関数宣言で記述
type UserProfileProps = {
  userId: string;
  onUpdate?: (user: User) => void;
};

export function UserProfile({ userId }: UserProfileProps) {
  return <div>{userId}</div>;
}
```

### インポート順序
```typescript
// 1. 外部ライブラリ
import React, { useState, useEffect } from 'react';
import { useRouter } from 'next/navigation';

// 2. 内部モジュール（絶対パス）
import { Button } from '@/components/common/Button';
import { useAuth } from '@/hooks/useAuth';

// 3. 相対パス
import { UserCard } from './UserCard';
import { formatDate } from '../utils/formatDate';

// 4. 型定義
import type { User } from '@/types/user';

// 5. スタイル
import styles from './UserProfile.module.css';
```

### コメント
```typescript
// 原則: コードは自己文書化すべき。コメントは「なぜ」を説明
// Good: 複雑な処理の理由を説明
// キャッシュを無効化してデータの整合性を保証
cache.clear();

// Bad: 処理内容を単に説明
// キャッシュをクリア
cache.clear();

// JSDoc コメント（公開API、複雑な関数）
/**
 * ユーザー情報を取得する
 * @param userId - ユーザーID
 * @param options - オプション設定
 * @returns ユーザー情報
 * @throws {NotFoundError} ユーザーが見つからない場合
 */
async function fetchUser(
  userId: string,
  options?: FetchOptions
): Promise<User> {
  // 実装
}
```

## プロジェクト構造

### Next.js (App Router)
```
my-app/
├── app/
│   ├── (auth)/              # ルートグループ
│   │   ├── login/
│   │   └── register/
│   ├── api/                 # API Routes
│   ├── components/          # ページ固有コンポーネント
│   ├── layout.tsx
│   └── page.tsx
├── src/
│   ├── components/          # 共有コンポーネント
│   │   ├── common/         # 汎用UI
│   │   └── features/       # 機能別
│   ├── hooks/              # カスタムフック
│   ├── lib/                # ユーティリティ
│   ├── types/              # 型定義
│   ├── utils/              # ヘルパー関数
│   └── constants/          # 定数
├── public/                 # 静的ファイル
├── tests/                  # テスト
├── .env.local
├── .env.example
├── next.config.js
├── tsconfig.json
└── package.json
```

### Nest.js
```
my-api/
├── src/
│   ├── modules/
│   │   ├── users/
│   │   │   ├── dto/
│   │   │   ├── entities/
│   │   │   ├── users.controller.ts
│   │   │   ├── users.service.ts
│   │   │   ├── users.module.ts
│   │   │   └── users.service.spec.ts
│   │   └── auth/
│   ├── common/
│   │   ├── decorators/
│   │   ├── filters/
│   │   ├── guards/
│   │   ├── interceptors/
│   │   └── pipes/
│   ├── config/
│   ├── database/
│   └── main.ts
├── test/
│   └── e2e/
├── nest-cli.json
├── tsconfig.json
└── package.json
```

## 技術スタック

### フロントエンド
- **React**: 関数コンポーネント + Hooks
- **Next.js**: App Router（Server Components優先）
- **TypeScript**: strict モード
- **CSS**: Tailwind CSS または CSS Modules
- **状態管理**: React Context / Zustand / Jotai
- **フォーム**: React Hook Form + Zod
- **データフェッチ**: SWR / TanStack Query

### バックエンド
- **Node.js**: v18以上
- **Nest.js**: モジュラーアーキテクチャ
- **Hono**: 軽量API（Cloudflare Workers等）
- **TypeScript**: strict モード
- **ORM**: Prisma / TypeORM
- **認証**: JWT / Passport.js
- **バリデーション**: class-validator / Zod

### テスト
- **ユニットテスト**: Jest / Vitest
- **E2E**: Playwright
- **コンポーネントテスト**: React Testing Library
- **カバレッジ**: 80%以上を目標

## 開発フロー

### 1. 開発開始前
```bash
# 最新のコードを取得
git pull origin main

# 依存パッケージのインストール
npm install

# 環境変数の設定
cp .env.example .env.local

# データベースのセットアップ
npm run db:setup

# 開発サーバーの起動
npm run dev
```

### 2. 機能開発
```bash
# フィーチャーブランチの作成
git checkout -b feature/user-profile

# コードの実装
# - 型定義から開始
# - テストを先に書く（TDD推奨）
# - 小さな単位でコミット

# テストの実行
npm test

# リンティング
npm run lint

# 型チェック
npx tsc --noEmit
```

### 3. コードレビュー前
```bash
# すべてのテストを実行
npm test -- --coverage

# E2Eテストの実行
npm run test:e2e

# ビルドの確認
npm run build

# セキュリティチェック
npm audit

# フォーマット
npm run format
```

### 4. マージ後
```bash
# デプロイの確認
# - CI/CDパイプラインの成功を確認
# - ステージング環境での動作確認
# - 本番デプロイ
```

## Spec-Driven Development

Claude Code を用いた spec-driven development を行います。
以下の5つのフェーズに従って開発を進めてください。

### 1. 事前準備フェーズ

- ユーザーが Claude Code に対して、実行したいタスクの概要を伝える
- このフェーズで `mkdir -p ./docs/specs` を実行する
- `./docs/specs` 内にタスクの概要から適切な spec 名を考えて、その名前のディレクトリを作成する
  - 例：「記事コンポーネントを作成する」→ `./docs/specs/create-article-component`
- 以下のフェーズで作成するファイルはこのディレクトリの中に配置する

### 2. 要件フェーズ

- Claude Code がユーザーから伝えられたタスクの概要に基づいて、タスクが満たすべき「要件ファイル」を作成する
- Claude Code がユーザーに対して「要件ファイル」を提示し、問題がないかを尋ねる
- ユーザーが「要件ファイル」を確認し、問題があれば Claude Code に対してフィードバックする
- ユーザーが「要件ファイル」を確認し、問題がないと答えるまで修正を繰り返す

### 3. 設計フェーズ

- Claude Code は、「要件ファイル」に記載されている要件を満たすような設計を記述した「設計ファイル」を作成する
- Claude Code がユーザーに対して「設計ファイル」を提示し、問題がないかを尋ねる
- ユーザーが「設計ファイル」を確認し、問題があれば Claude Code に対してフィードバックする
- ユーザーが「設計ファイル」を確認し、問題がないと答えるまで修正を繰り返す

### 4. 実装計画フェーズ

- Claude Code は、「設計ファイル」に記載されている設計を実装するための「実装計画ファイル」を作成する
- Claude Code がユーザーに対して「実装計画ファイル」を提示し、問題がないかを尋ねる
- ユーザーが「実装計画ファイル」を確認し、問題があれば Claude Code に対してフィードバックする
- ユーザーが「実装計画ファイル」を確認し、問題がないと答えるまで修正を繰り返す

### 5. 実装フェーズ

- Claude Code は、「実装計画ファイル」に基づいて実装を開始する
- 実装するときは「要件ファイル」「設計ファイル」に記載されている内容を守りながら実装する

## テスト戦略

### テストピラミッド
```
      /\
     /E2E\        少ない（重要なユーザーフロー）
    /------\
   /統合テスト\    中程度（API、DB統合）
  /----------\
 /ユニットテスト\   多い（関数、コンポーネント）
/--------------\
```

### ユニットテスト
- **対象**: 純粋関数、ユーティリティ、カスタムフック
- **ツール**: Jest / Vitest
- **カバレッジ**: 80%以上

```typescript
describe('formatDate', () => {
  it('should format date correctly', () => {
    const date = new Date('2024-01-01');
    expect(formatDate(date)).toBe('2024年1月1日');
  });
});
```

### コンポーネントテスト
- **対象**: Reactコンポーネント
- **ツール**: React Testing Library
- **原則**: ユーザーの視点でテスト

```typescript
import { render, screen, fireEvent } from '@testing-library/react';

test('should submit form with valid data', async () => {
  render(<LoginForm />);
  
  await userEvent.type(screen.getByLabelText(/email/i), 'user@example.com');
  await userEvent.type(screen.getByLabelText(/password/i), 'password123');
  await userEvent.click(screen.getByRole('button', { name: /login/i }));
  
  expect(screen.getByText(/welcome/i)).toBeInTheDocument();
});
```

### E2Eテスト
- **対象**: クリティカルなユーザーフロー
- **ツール**: Playwright
- **実行**: CI/CD + ローカル

```typescript
test('user can complete purchase flow', async ({ page }) => {
  await page.goto('/products');
  await page.getByText('商品A').click();
  await page.getByRole('button', { name: 'カートに追加' }).click();
  await page.getByRole('link', { name: '購入手続き' }).click();
  await page.getByLabel('カード番号').fill('4242424242424242');
  await page.getByRole('button', { name: '注文確定' }).click();

  await expect(page.getByText('注文完了')).toBeVisible();
});
```

## セキュリティガイドライン

### 1. 認証・認可
```typescript
// JWT の適切な使用
const secret = process.env.JWT_SECRET;
if (!secret) throw new Error('JWT_SECRET is not configured');

const token = jwt.sign(
  { userId: user.id, role: user.role },
  secret,
  { expiresIn: '1h' }
);

// パスワードのハッシュ化
import bcrypt from 'bcrypt';
const hashedPassword = await bcrypt.hash(password, 10);
```

### 2. 入力検証
```typescript
// Zod でバリデーション
import { z } from 'zod';

const userSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
  age: z.number().min(0).max(150),
});

// 入力のサニタイゼーション
import DOMPurify from 'isomorphic-dompurify';
const clean = DOMPurify.sanitize(userInput);
```

### 3. XSS対策
```typescript
// React は自動的にエスケープ
<div>{userInput}</div> // 安全

// dangerouslySetInnerHTML は避ける
// どうしても必要な場合はサニタイズ
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(html) }} />
```

### 4. CSRF対策
```typescript
// SameSite Cookie の使用
res.cookie('token', token, {
  httpOnly: true,
  secure: true,
  sameSite: 'strict',
});
```

### 5. 環境変数の管理
```typescript
// センシティブな情報は環境変数で管理
const config = {
  database: {
    url: process.env.DATABASE_URL,
  },
  jwt: {
    secret: process.env.JWT_SECRET,
  },
};

// .env.example を提供（実際の値は含めない）
```

## パフォーマンス最適化

### フロントエンド

#### 1. コード分割
```typescript
// 動的インポート
const HeavyComponent = dynamic(() => import('./HeavyComponent'), {
  loading: () => <div>Loading...</div>,
});

// React.lazy
const LazyComponent = React.lazy(() => import('./LazyComponent'));
```

#### 2. メモ化
```typescript
// React.memo
export const ExpensiveComponent = React.memo(({ data }) => {
  return <div>{/* レンダリング */}</div>;
});

// useMemo
const expensiveValue = useMemo(() => {
  return computeExpensiveValue(data);
}, [data]);

// useCallback
const handleClick = useCallback(() => {
  doSomething(id);
}, [id]);
```

#### 3. 画像最適化
```typescript
// Next.js Image コンポーネント
import Image from 'next/image';

<Image
  src="/photo.jpg"
  alt="Photo"
  width={500}
  height={300}
  priority // LCP 画像の場合
/>
```

### バックエンド

#### 1. データベースクエリ最適化
```typescript
// N+1問題の回避
const users = await prisma.user.findMany({
  include: {
    posts: true, // JOIN で一度に取得
  },
});

// インデックスの使用
// schema.prisma
model User {
  id    String @id
  email String @unique // インデックス
  @@index([createdAt]) // 複合インデックス
}
```

#### 2. キャッシング
```typescript
// Redis でキャッシュ
const cachedUser = await redis.get(`user:${userId}`);
if (cachedUser) {
  return JSON.parse(cachedUser);
}

const user = await fetchUser(userId);
await redis.set(`user:${userId}`, JSON.stringify(user), 'EX', 3600);
return user;
```

## エラーハンドリング

### フロントエンド

#### Error Boundary
```typescript
class ErrorBoundary extends React.Component<Props, State> {
  state = { hasError: false };

  static getDerivedStateFromError(error: Error) {
    return { hasError: true };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('Error caught:', error, errorInfo);
    // エラーログサービスに送信
  }

  render() {
    if (this.state.hasError) {
      return <div>エラーが発生しました</div>;
    }
    return this.props.children;
  }
}
```

### バックエンド

#### グローバルエラーハンドラ
```typescript
// Nest.js
@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();

    const status = exception instanceof HttpException
      ? exception.getStatus()
      : 500;

    const message = exception instanceof HttpException
      ? exception.message
      : 'Internal server error';

    // ログ記録
    console.error({
      timestamp: new Date().toISOString(),
      path: request.url,
      error: exception,
    });

    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      message,
    });
  }
}
```

## ドキュメンテーション

### コードドキュメント
```typescript
/**
 * ユーザー情報を更新する
 * 
 * @param userId - 更新対象のユーザーID
 * @param data - 更新データ
 * @returns 更新されたユーザー情報
 * @throws {NotFoundError} ユーザーが存在しない場合
 * @throws {ValidationError} バリデーションエラーの場合
 * 
 * @example
 * ```typescript
 * const user = await updateUser('123', {
 *   name: 'New Name',
 *   email: 'new@example.com',
 * });
 * ```
 */
async function updateUser(
  userId: string,
  data: UpdateUserData
): Promise<User> {
  // 実装
}
```

### README.md
各プロジェクトには以下を含むREADME.mdを作成：
- プロジェクト概要
- セットアップ手順
- 開発コマンド
- 技術スタック
- ディレクトリ構造
- 環境変数の説明

## AIエージェント向け指示

### コード生成時の注意点
1. **型安全性**: 必ず TypeScript の型を定義
2. **テスト**: コードとテストを同時に生成
3. **エラーハンドリング**: 適切な例外処理を含める
4. **パフォーマンス**: 不要な再レンダリングを避ける
5. **セキュリティ**: 入力検証を忘れない
6. **アクセシビリティ**: セマンティックHTML、ARIA属性
7. **ドキュメント**: 複雑なロジックにはコメントを追加

## PR レビューガイドライン

このチェックリストを使って **一貫性があり高品質** なコードレビューを行います。
該当しない項目は状況に応じて省略・調整してください。

### 1. 事前準備

1. **PR 説明を読む**
   - 課題、解決方針、関連 Issue が正しく説明されているか確認する
2. **差分とコミット履歴をざっと把握する**
   - 変更ファイルの種類、規模、影響範囲、リスク領域を掴む

### 2. 主要チェック項目

| カテゴリ | 確認ポイント | 典型的な問題例 |
| -------- | ------------ | -------------- |
| **正確性** | 要件を実装しているか、境界値を考慮しているか、バグがないか | オフバイワン、null/undefined 処理漏れ、競合状態、`await` 抜け |
| **コード品質** | 読みやすく保守しやすいか、DRY 原則を守れているか | 長大関数、深いネスト、重複コード |
| **スタイル/一貫性** | プロジェクトの規約・リンター・フォーマッタに従っているか | 命名不統一、タブとスペース混在、マジックナンバー |
| **パフォーマンス** | 想定スケールで効率的か | 無制限ループ、N x N クエリ、不必要な再レンダー |
| **セキュリティ** | 入力バリデーション、機密情報保護、OWASP への対応 | SQL インジェクション、XSS、平文保存、ハードコード秘密鍵 |
| **テスト** | 単体/結合テストが追加され失敗系も網羅、CI で合格 | テスト不足、フレークテスト、カバレッジ低 |
| **ドキュメント** | コメント、README、ADR、公開 API の記載が更新済み | 破壊的変更の未告知、古いドキュメント |
| **アクセシビリティ (UI)** | ARIA、キーボード操作、コントラスト等に準拠 | ラベル欠如、フォーカストラップ、色のみ伝達 |

### 3. レビューワークフロー

1. **全体フィードバック**
   - 目的と総評を簡潔にまとめる
2. **インラインコメント**
   - 該当行に具体的・建設的・行動指向で記述
   - 必要に応じてコード例を添える
3. **質問**
   - 意図が不明瞭な場合は変更依頼前にオープンクエスチョンで確認
4. **承認判断**
   - **Approve**: 問題なし
   - **Request Changes**: ブロッキング問題あり
   - **Comment Only**: 軽微な指摘や提案のみ

### 4. トーンと表現ガイド

- **敬意とポジティブさ** を忘れず、善意を前提に
- **コードにフォーカス** し、作者個人を批判しない
- **命令より提案** を優先：「〜の抽出を検討してみては？」
- **称賛と指摘のバランス**：良かった点も必ず示す

### 5. AI レビュー例テンプレート

```text
### サマリ
データレイヤーを簡潔にできていて良いです！
ただしエラーハンドリング周辺でブロッキング 1 件、軽微な提案 3 件があります。

---

#### Blocking
1. **`user` の null チェック**
   行 42: `user.id` が undefined になる可能性 → 早期 return またはデフォルト値を追加してください。

#### Suggestions（非ブロッキング）
* 行 18: 手動 fetch を共有 `apiClient` に置き換えると DRY。
* 行 67: `renderTable` を小さなコンポーネントに分割すると読みやすいです。

#### Questions
* この変更は `betaNewDashboard` フィーチャーフラグとどう連携しますか？

---

**Decision**: 変更をリクエスト
```

## まとめ

このガイドラインに従うことで：
- **保守性の高い**コードが書ける
- **バグの少ない**アプリケーションが作れる
- **セキュアな**システムが構築できる
- **パフォーマンスの良い**プロダクトが開発できる
- **チーム開発が円滑**になる

常に品質を意識し、ユーザーに価値を届けることを最優先に開発を進めましょう。
