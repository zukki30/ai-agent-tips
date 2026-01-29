# クイックリファレンス

開発中にすぐに参照できる重要なポイントをまとめたチートシートです。

## 📋 コーディング規約クイックリファレンス

### 命名規則
```typescript
// ファイル・コンポーネント: PascalCase
UserProfile.tsx
Button.tsx

// 変数・関数: camelCase
const userName = 'John';
function fetchUser() {}

// 定数: UPPER_SNAKE_CASE
const API_BASE_URL = 'https://api.example.com';
const MAX_RETRY_COUNT = 3;

// Boolean: is/has/can
const isValid = true;
const hasPermission = false;
const canEdit = true;

// イベントハンドラ: handle
const handleClick = () => {};
const handleSubmit = () => {};

// Props型: ComponentName + Props
interface UserProfileProps {}
```

### インポート順序
```typescript
// 1. 外部ライブラリ
import React from 'react';
import { useRouter } from 'next/router';

// 2. 内部モジュール（絶対パス）
import { Button } from '@/components/Button';

// 3. 相対パス
import { utils } from '../utils';

// 4. 型定義
import type { User } from '@/types';

// 5. スタイル
import styles from './styles.module.css';
```

## ⚡️ よく使うコマンド

### 開発
```bash
# 開発サーバー起動
npm run dev

# ビルド
npm run build

# テスト
npm test

# テスト（ウォッチモード）
npm test -- --watch

# E2E テスト
npx playwright test

# リント
npm run lint

# フォーマット
npm run format
```

### Git
```bash
# ブランチ作成と切り替え
git checkout -b feature/new-feature

# ステージング
git add .

# コミット
git commit -m "feat: add new feature"

# プッシュ
git push origin feature/new-feature
```

### トラブルシューティング
```bash
# node_modules を削除して再インストール
rm -rf node_modules package-lock.json && npm install

# ポート確認
lsof -i :3000

# プロセス終了
kill -9 <PID>
```

## 🎯 React/Next.js クイックリファレンス

### コンポーネント基本形
```typescript
interface Props {
  title: string;
  onSubmit?: () => void;
}

export const MyComponent: React.FC<Props> = ({ title, onSubmit }) => {
  const [state, setState] = useState('');
  
  useEffect(() => {
    // 副作用
  }, []);
  
  return <div>{title}</div>;
};
```

### Next.js Server Component
```typescript
// app/page.tsx
async function HomePage() {
  const data = await fetchData(); // 直接データフェッチ
  
  return <div>{data.title}</div>;
}
```

### Next.js Client Component
```typescript
// app/components/Counter.tsx
'use client';

import { useState } from 'react';

export function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

### API Route
```typescript
// app/api/users/route.ts
import { NextResponse } from 'next/server';

export async function GET() {
  const data = await fetchUsers();
  return NextResponse.json(data);
}

export async function POST(request: Request) {
  const body = await request.json();
  const result = await createUser(body);
  return NextResponse.json(result, { status: 201 });
}
```

## 🧪 テストクイックリファレンス

### ユニットテスト（Jest）
```typescript
describe('MyFunction', () => {
  it('should work correctly', () => {
    expect(myFunction(input)).toBe(expected);
  });
  
  it('should handle errors', () => {
    expect(() => myFunction(invalid)).toThrow();
  });
});
```

### React コンポーネントテスト
```typescript
import { render, screen, fireEvent } from '@testing-library/react';

test('button click', async () => {
  render(<MyComponent />);
  
  const button = screen.getByRole('button', { name: /submit/i });
  await fireEvent.click(button);
  
  expect(screen.getByText(/success/i)).toBeInTheDocument();
});
```

### E2E テスト（Playwright）
```typescript
test('user flow', async ({ page }) => {
  await page.goto('/');
  await page.fill('input[name="email"]', 'user@example.com');
  await page.click('button[type="submit"]');
  
  await expect(page).toHaveURL('/dashboard');
});
```

## 🔒 セキュリティチェックリスト

- [ ] 入力バリデーション（Zod等）
- [ ] 出力のサニタイゼーション
- [ ] パスワードのハッシュ化（bcrypt）
- [ ] JWT の適切な使用
- [ ] CORS の設定
- [ ] XSS 対策（React は基本的に自動）
- [ ] CSRF トークン（必要に応じて）
- [ ] 環境変数の適切な管理
- [ ] SQL インジェクション対策（ORM使用）
- [ ] レート制限

## 🚀 パフォーマンスチェックリスト

### フロントエンド
- [ ] 画像最適化（Next.js Image）
- [ ] コード分割（dynamic import）
- [ ] React.memo / useMemo / useCallback
- [ ] 仮想化（長いリスト）
- [ ] Lazy Loading
- [ ] Lighthouse スコア 90+

### バックエンド
- [ ] データベースインデックス
- [ ] N+1 問題の回避
- [ ] キャッシング（Redis等）
- [ ] 接続プール
- [ ] ページネーション
- [ ] 圧縮（gzip/brotli）

## 📝 型定義パターン

### Props 型
```typescript
interface ComponentProps {
  required: string;
  optional?: number;
  callback?: (value: string) => void;
  children?: React.ReactNode;
}
```

### API レスポンス型
```typescript
interface ApiResponse<T> {
  success: boolean;
  data: T;
  error?: string;
}

type User = {
  id: string;
  name: string;
  email: string;
};
```

### 汎用型
```typescript
type Nullable<T> = T | null;
type Optional<T> = T | undefined;
type AsyncFunction<T> = () => Promise<T>;
```

## 🎨 スタイリングパターン

### Tailwind CSS
```tsx
<div className="flex items-center justify-between p-4 bg-white rounded-lg shadow">
  <h1 className="text-2xl font-bold text-gray-900">Title</h1>
</div>
```

### CSS Modules
```tsx
import styles from './Component.module.css';

<div className={styles.container}>
  <h1 className={styles.title}>Title</h1>
</div>
```

## 🔧 環境変数パターン

### .env.local
```bash
# データベース
DATABASE_URL="postgresql://localhost/mydb"

# API
API_BASE_URL="https://api.example.com"
NEXT_PUBLIC_API_KEY="public-key"

# 認証
JWT_SECRET="super-secret-key"
```

### 使用方法
```typescript
// サーバーサイドのみ
const dbUrl = process.env.DATABASE_URL;

// クライアントでも使用可能（NEXT_PUBLIC_ プレフィックス）
const apiKey = process.env.NEXT_PUBLIC_API_KEY;
```

## 📦 よく使うパッケージ

### フロントエンド
```bash
npm install react react-dom next
npm install -D typescript @types/react @types/node
npm install zod react-hook-form @hookform/resolvers
npm install swr @tanstack/react-query
npm install date-fns lodash
```

### バックエンド
```bash
npm install @nestjs/core @nestjs/common
npm install hono
npm install prisma @prisma/client
npm install bcrypt jsonwebtoken
npm install -D @types/bcrypt @types/jsonwebtoken
```

### テスト
```bash
npm install -D jest @testing-library/react @testing-library/jest-dom
npm install -D @playwright/test
npm install -D vitest @vitest/ui
```

## 🐛 デバッグTips

### Console
```typescript
console.log('Debug:', variable);
console.table(arrayOfObjects);
console.time('operation');
// ... operation
console.timeEnd('operation');
```

### React DevTools
```typescript
// コンポーネント名を表示
MyComponent.displayName = 'MyComponent';
```

### Network
```typescript
// fetch のログ
console.log('Fetching:', url);
const response = await fetch(url);
console.log('Response:', response.status);
```

## 🎯 AI エージェントへの指示テンプレート

### 機能実装
```
AGENT.md と skills/<technology>.md を参照して、
<機能名> を実装してください。

要件：
- <要件1>
- <要件2>
- テストを含める
- 型安全性を確保
```

### バグ修正
```
以下のバグを修正してください：
<バグの説明>

AGENT.md のエラーハンドリングセクションを参照し、
適切なエラー処理を実装してください。
```

### リファクタリング
```
<ファイル名> をリファクタリングしてください。

AGENT.md のベストプラクティスに従い：
- 型安全性の向上
- パフォーマンスの改善
- コードの可読性向上
```

## 📚 詳細情報の参照先

- **詳細ガイド**: AGENT.md
- **技術別ガイド**: skills/ ディレクトリ
- **コマンド集**: commands/ ディレクトリ
- **統合例**: INTEGRATION_GUIDE.md

---

**このファイルをブックマークして、開発中にすぐ参照できるようにしましょう！**
