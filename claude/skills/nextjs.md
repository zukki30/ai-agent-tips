# Next.js 開発スキル

## 概要
このスキルは、Next.js フレームワークを使用したフルスタック Web アプリケーション開発のベストプラクティスを提供します。

## App Router vs Pages Router
- **App Router 優先**: 新規プロジェクトでは App Router (app ディレクトリ) を使用
- **Server Components**: デフォルトで Server Components を使用し、必要な場合のみ Client Components を使用

## ディレクトリ構造 (App Router)
```
app/
├── (auth)/              # ルートグループ
│   ├── login/
│   └── register/
├── api/                 # API Routes
│   └── users/
│       └── route.ts
├── components/          # 共有コンポーネント
├── lib/                 # ユーティリティとヘルパー
├── types/               # 型定義
├── layout.tsx           # ルートレイアウト
└── page.tsx             # ホームページ
```

## Server Components と Client Components

### Server Components (デフォルト)
```typescript
// app/users/page.tsx
// デフォルトで Server Component
async function UsersPage() {
  const users = await fetchUsers(); // 直接データフェッチ可能
  
  return (
    <div>
      {users.map(user => (
        <UserCard key={user.id} user={user} />
      ))}
    </div>
  );
}

export default UsersPage;
```

### Client Components
```typescript
// app/components/Counter.tsx
'use client'; // Client Component として明示

import { useState } from 'react';

export function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}
```

## データフェッチング

### Server Components でのデータフェッチ
```typescript
// キャッシュ制御
async function getData() {
  const res = await fetch('https://api.example.com/data', {
    cache: 'force-cache', // デフォルト: 静的生成
    // cache: 'no-store', // 動的: 毎回フェッチ
    // next: { revalidate: 60 }, // ISR: 60秒ごとに再検証
  });
  
  if (!res.ok) {
    throw new Error('Failed to fetch data');
  }
  
  return res.json();
}
```

### クライアントサイドでのデータフェッチ
```typescript
'use client';

import useSWR from 'swr';

export function UserProfile({ userId }: { userId: string }) {
  const { data, error, isLoading } = useSWR(
    `/api/users/${userId}`,
    fetcher
  );
  
  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error loading user</div>;
  
  return <div>{data.name}</div>;
}
```

## API Routes

### Route Handlers (App Router)
```typescript
// app/api/users/route.ts
import { NextResponse } from 'next/server';

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  const id = searchParams.get('id');
  
  const users = await fetchUsers(id);
  
  return NextResponse.json(users);
}

export async function POST(request: Request) {
  const body = await request.json();
  const user = await createUser(body);
  
  return NextResponse.json(user, { status: 201 });
}
```

### Dynamic Route Handlers
```typescript
// app/api/users/[id]/route.ts
import { NextResponse } from 'next/server';

export async function GET(
  request: Request,
  { params }: { params: { id: string } }
) {
  const user = await fetchUser(params.id);
  
  if (!user) {
    return NextResponse.json(
      { error: 'User not found' },
      { status: 404 }
    );
  }
  
  return NextResponse.json(user);
}
```

## メタデータとSEO

### 静的メタデータ
```typescript
// app/page.tsx
import { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'ホーム',
  description: 'アプリケーションのホームページ',
};

export default function HomePage() {
  return <div>ホームページ</div>;
}
```

### 動的メタデータ
```typescript
// app/users/[id]/page.tsx
import { Metadata } from 'next';

export async function generateMetadata(
  { params }: { params: { id: string } }
): Promise<Metadata> {
  const user = await fetchUser(params.id);
  
  return {
    title: user.name,
    description: `${user.name}のプロフィール`,
  };
}
```

## 画像最適化
```typescript
import Image from 'next/image';

// 最適化された画像コンポーネント
<Image
  src="/profile.jpg"
  alt="Profile"
  width={500}
  height={500}
  priority // LCPイメージの場合
/>
```

## 環境変数
```typescript
// .env.local
DATABASE_URL=postgresql://...
NEXT_PUBLIC_API_URL=https://api.example.com

// 使用方法
const dbUrl = process.env.DATABASE_URL; // サーバーサイドのみ
const apiUrl = process.env.NEXT_PUBLIC_API_URL; // クライアントでも使用可能
```

## パフォーマンス最適化

### Loading UI
```typescript
// app/users/loading.tsx
export default function Loading() {
  return <div>Loading users...</div>;
}
```

### Error Handling
```typescript
// app/users/error.tsx
'use client';

export default function Error({
  error,
  reset,
}: {
  error: Error;
  reset: () => void;
}) {
  return (
    <div>
      <h2>エラーが発生しました</h2>
      <button onClick={reset}>再試行</button>
    </div>
  );
}
```

### Streaming と Suspense
```typescript
import { Suspense } from 'react';

export default function Page() {
  return (
    <div>
      <h1>ユーザー一覧</h1>
      <Suspense fallback={<div>Loading...</div>}>
        <UserList />
      </Suspense>
    </div>
  );
}
```

## ベストプラクティス
1. **Server Components を優先**: インタラクションが必要な場合のみ Client Components を使用
2. **適切なキャッシュ戦略**: データの特性に応じて cache と revalidate を設定
3. **型安全性の確保**: TypeScript を最大限活用し、型定義を徹底
4. **環境変数の管理**: センシティブな情報は NEXT_PUBLIC_ プレフィックスをつけない
5. **画像最適化**: next/image を必ず使用

## テスト
```typescript
// __tests__/page.test.tsx
import { render, screen } from '@testing-library/react';
import HomePage from '@/app/page';

describe('HomePage', () => {
  it('正しくレンダリングされる', () => {
    render(<HomePage />);
    expect(screen.getByText('ホームページ')).toBeInTheDocument();
  });
});
```

## 参考資料
- Next.js 公式ドキュメント: https://nextjs.org/docs
- App Router 移行ガイド: https://nextjs.org/docs/app/building-your-application/upgrading
