# Hono 開発スキル

## 概要
このスキルは、Hono フレームワークを使用した軽量で高速な Web API 開発のベストプラクティスを提供します。

## Hono の特徴
- **軽量**: 小さなバンドルサイズ
- **高速**: 超高速なルーティング
- **マルチランタイム**: Cloudflare Workers、Deno、Bun、Node.js で動作
- **TypeScript ファースト**: 完全な型安全性

## 基本的なセットアップ

### Node.js での基本アプリケーション
```typescript
import { Hono } from 'hono';
import { serve } from '@hono/node-server';

const app = new Hono();

app.get('/', (c) => {
  return c.text('Hello Hono!');
});

serve(app);
```

### Cloudflare Workers での基本アプリケーション
```typescript
import { Hono } from 'hono';

const app = new Hono();

app.get('/', (c) => {
  return c.text('Hello Hono on Cloudflare Workers!');
});

export default app;
```

## ルーティング

### 基本的なルート
```typescript
import { Hono } from 'hono';

const app = new Hono();

// GET リクエスト
app.get('/users', (c) => {
  return c.json({ users: [] });
});

// POST リクエスト
app.post('/users', async (c) => {
  const body = await c.req.json();
  return c.json({ user: body }, 201);
});

// PUT リクエスト
app.put('/users/:id', async (c) => {
  const id = c.req.param('id');
  const body = await c.req.json();
  return c.json({ id, ...body });
});

// DELETE リクエスト
app.delete('/users/:id', (c) => {
  const id = c.req.param('id');
  return c.json({ message: `User ${id} deleted` });
});

// PATCH リクエスト
app.patch('/users/:id', async (c) => {
  const id = c.req.param('id');
  const body = await c.req.json();
  return c.json({ id, ...body });
});
```

### パスパラメータとクエリパラメータ
```typescript
// パスパラメータ
app.get('/users/:id', (c) => {
  const id = c.req.param('id');
  return c.json({ id });
});

// 複数のパスパラメータ
app.get('/posts/:postId/comments/:commentId', (c) => {
  const postId = c.req.param('postId');
  const commentId = c.req.param('commentId');
  return c.json({ postId, commentId });
});

// クエリパラメータ
app.get('/search', (c) => {
  const query = c.req.query('q');
  const page = c.req.query('page') || '1';
  return c.json({ query, page });
});

// すべてのクエリパラメータ
app.get('/filter', (c) => {
  const queries = c.req.queries();
  return c.json(queries);
});
```

### ワイルドカード
```typescript
// ワイルドカードマッチング
app.get('/posts/*', (c) => {
  return c.text('Match any path after /posts/');
});

// 正規表現
app.get('/posts/:id{[0-9]+}', (c) => {
  const id = c.req.param('id'); // 数字のみ
  return c.json({ id });
});
```

## リクエストとレスポンス

### リクエストの処理
```typescript
app.post('/users', async (c) => {
  // JSON ボディの取得
  const body = await c.req.json();
  
  // フォームデータの取得
  const formData = await c.req.formData();
  
  // テキストの取得
  const text = await c.req.text();
  
  // ヘッダーの取得
  const contentType = c.req.header('Content-Type');
  
  // 認証トークンの取得
  const token = c.req.header('Authorization')?.replace('Bearer ', '');
  
  return c.json({ received: body });
});
```

### レスポンスの返却
```typescript
app.get('/example', (c) => {
  // JSON レスポンス
  return c.json({ message: 'Hello' });
  
  // テキストレスポンス
  return c.text('Hello');
  
  // HTML レスポンス
  return c.html('<h1>Hello</h1>');
  
  // リダイレクト
  return c.redirect('/new-path');
  
  // カスタムステータスコード
  return c.json({ error: 'Not Found' }, 404);
  
  // カスタムヘッダー
  return c.json(
    { data: 'example' },
    200,
    { 'X-Custom-Header': 'value' }
  );
});
```

## ミドルウェア

### ビルトインミドルウェア
```typescript
import { Hono } from 'hono';
import { logger } from 'hono/logger';
import { cors } from 'hono/cors';
import { bearerAuth } from 'hono/bearer-auth';
import { jwt } from 'hono/jwt';

const app = new Hono();

// ロガー
app.use('*', logger());

// CORS
app.use('*', cors({
  origin: 'https://example.com',
  allowMethods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowHeaders: ['Content-Type', 'Authorization'],
  maxAge: 600,
}));

// Bearer 認証
app.use('/api/*', bearerAuth({
  token: 'your-secret-token',
}));

// JWT 認証
app.use('/protected/*', jwt({
  secret: 'your-jwt-secret',
}));
```

### カスタムミドルウェア
```typescript
import { Context, Next } from 'hono';

// ロギングミドルウェア
const requestLogger = async (c: Context, next: Next) => {
  const start = Date.now();
  await next();
  const end = Date.now();
  console.log(`${c.req.method} ${c.req.url} - ${end - start}ms`);
};

// 認証ミドルウェア
const authMiddleware = async (c: Context, next: Next) => {
  const token = c.req.header('Authorization');
  
  if (!token) {
    return c.json({ error: 'Unauthorized' }, 401);
  }
  
  // トークンの検証
  const isValid = await verifyToken(token);
  
  if (!isValid) {
    return c.json({ error: 'Invalid token' }, 401);
  }
  
  await next();
};

// エラーハンドリングミドルウェア
const errorHandler = async (c: Context, next: Next) => {
  try {
    await next();
  } catch (error) {
    console.error('Error:', error);
    return c.json({ error: 'Internal Server Error' }, 500);
  }
};

// ミドルウェアの適用
app.use('*', requestLogger);
app.use('/api/*', authMiddleware);
app.use('*', errorHandler);
```

## バリデーション

### Zod を使用したバリデーション
```typescript
import { z } from 'zod';
import { zValidator } from '@hono/zod-validator';

// スキーマの定義
const userSchema = z.object({
  name: z.string().min(1).max(100),
  email: z.string().email(),
  age: z.number().int().positive().optional(),
});

// バリデーターの適用
app.post('/users', zValidator('json', userSchema), async (c) => {
  const user = c.req.valid('json');
  // user は型安全
  return c.json({ user }, 201);
});

// クエリパラメータのバリデーション
const searchSchema = z.object({
  q: z.string(),
  page: z.string().regex(/^\d+$/).transform(Number),
  limit: z.string().regex(/^\d+$/).transform(Number).default('10'),
});

app.get('/search', zValidator('query', searchSchema), (c) => {
  const { q, page, limit } = c.req.valid('query');
  return c.json({ q, page, limit });
});
```

### カスタムバリデーション
```typescript
const validateUser = async (c: Context, next: Next) => {
  const body = await c.req.json();
  
  if (!body.name || typeof body.name !== 'string') {
    return c.json({ error: 'Name is required' }, 400);
  }
  
  if (!body.email || !body.email.includes('@')) {
    return c.json({ error: 'Valid email is required' }, 400);
  }
  
  await next();
};

app.post('/users', validateUser, async (c) => {
  const body = await c.req.json();
  return c.json({ user: body }, 201);
});
```

## 型安全な API 開発

### RPC モード (hono/client)
```typescript
// サーバーサイド (api.ts)
import { Hono } from 'hono';
import { zValidator } from '@hono/zod-validator';
import { z } from 'zod';

const app = new Hono();

const route = app
  .get('/posts', (c) => {
    return c.json([
      { id: 1, title: 'Post 1' },
      { id: 2, title: 'Post 2' },
    ]);
  })
  .post(
    '/posts',
    zValidator('json', z.object({
      title: z.string(),
      body: z.string(),
    })),
    async (c) => {
      const { title, body } = c.req.valid('json');
      return c.json({ id: 3, title, body }, 201);
    }
  );

export type AppType = typeof route;

// クライアントサイド
import { hc } from 'hono/client';
import type { AppType } from './api';

const client = hc<AppType>('http://localhost:3000');

// 型安全なAPIコール
const posts = await client.posts.$get();
const data = await posts.json(); // 型推論される

const newPost = await client.posts.$post({
  json: {
    title: 'New Post',
    body: 'Content',
  },
});
```

## ファイルアップロード

### フォームデータの処理
```typescript
app.post('/upload', async (c) => {
  const formData = await c.req.formData();
  const file = formData.get('file') as File;
  
  if (!file) {
    return c.json({ error: 'No file uploaded' }, 400);
  }
  
  // ファイル情報の取得
  const fileName = file.name;
  const fileSize = file.size;
  const fileType = file.type;
  
  // ファイルの読み込み
  const arrayBuffer = await file.arrayBuffer();
  const buffer = Buffer.from(arrayBuffer);
  
  // ファイルの保存処理（実装依存）
  // await saveFile(buffer, fileName);
  
  return c.json({
    message: 'File uploaded successfully',
    fileName,
    fileSize,
    fileType,
  });
});
```

## エラーハンドリング

### グローバルエラーハンドラー
```typescript
import { HTTPException } from 'hono/http-exception';

// カスタムエラーの定義
class ValidationError extends HTTPException {
  constructor(message: string) {
    super(400, { message });
  }
}

// エラーハンドラーの設定
app.onError((err, c) => {
  if (err instanceof HTTPException) {
    return c.json(
      {
        error: err.message,
      },
      err.status
    );
  }
  
  console.error('Unexpected error:', err);
  return c.json(
    {
      error: 'Internal Server Error',
    },
    500
  );
});

// エラーの使用
app.get('/users/:id', async (c) => {
  const id = c.req.param('id');
  const user = await findUser(id);
  
  if (!user) {
    throw new HTTPException(404, { message: 'User not found' });
  }
  
  return c.json(user);
});
```

## データベース統合

### Prisma との統合
```typescript
import { PrismaClient } from '@prisma/client';
import { Hono } from 'hono';

const prisma = new PrismaClient();
const app = new Hono();

app.get('/users', async (c) => {
  const users = await prisma.user.findMany();
  return c.json(users);
});

app.post('/users', async (c) => {
  const { name, email } = await c.req.json();
  
  const user = await prisma.user.create({
    data: { name, email },
  });
  
  return c.json(user, 201);
});

app.get('/users/:id', async (c) => {
  const id = c.req.param('id');
  
  const user = await prisma.user.findUnique({
    where: { id },
  });
  
  if (!user) {
    return c.json({ error: 'User not found' }, 404);
  }
  
  return c.json(user);
});
```

## テスト

### Hono アプリケーションのテスト
```typescript
import { describe, it, expect } from 'vitest';
import { Hono } from 'hono';

const app = new Hono();

app.get('/hello', (c) => {
  return c.json({ message: 'Hello' });
});

describe('API Tests', () => {
  it('should return hello message', async () => {
    const res = await app.request('/hello');
    expect(res.status).toBe(200);
    
    const data = await res.json();
    expect(data).toEqual({ message: 'Hello' });
  });

  it('should return 404 for unknown route', async () => {
    const res = await app.request('/unknown');
    expect(res.status).toBe(404);
  });
});
```

## Cloudflare Workers 固有の機能

### KV ストレージの使用
```typescript
type Bindings = {
  MY_KV: KVNamespace;
};

const app = new Hono<{ Bindings: Bindings }>();

app.get('/cache/:key', async (c) => {
  const key = c.req.param('key');
  const value = await c.env.MY_KV.get(key);
  
  if (!value) {
    return c.json({ error: 'Not found' }, 404);
  }
  
  return c.json({ value });
});

app.post('/cache/:key', async (c) => {
  const key = c.req.param('key');
  const { value } = await c.req.json();
  
  await c.env.MY_KV.put(key, value);
  
  return c.json({ message: 'Saved' });
});
```

### D1 データベースの使用
```typescript
type Bindings = {
  DB: D1Database;
};

const app = new Hono<{ Bindings: Bindings }>();

app.get('/users', async (c) => {
  const { results } = await c.env.DB
    .prepare('SELECT * FROM users')
    .all();
  
  return c.json(results);
});

app.post('/users', async (c) => {
  const { name, email } = await c.req.json();
  
  await c.env.DB
    .prepare('INSERT INTO users (name, email) VALUES (?, ?)')
    .bind(name, email)
    .run();
  
  return c.json({ message: 'User created' }, 201);
});
```

## パフォーマンス最適化

### キャッシング
```typescript
import { cache } from 'hono/cache';

// キャッシュミドルウェア
app.use(
  '/api/*',
  cache({
    cacheName: 'api-cache',
    cacheControl: 'max-age=3600',
  })
);

// 手動キャッシュ制御
app.get('/data', async (c) => {
  const data = await fetchData();
  
  c.header('Cache-Control', 'public, max-age=3600');
  return c.json(data);
});
```

## ベストプラクティス

1. **型安全性の活用**: TypeScript の型を最大限活用
2. **ミドルウェアの活用**: 共通処理はミドルウェアに抽出
3. **バリデーション**: すべての入力を検証
4. **エラーハンドリング**: 適切なHTTPステータスコードとエラーメッセージ
5. **セキュリティ**: CORS、認証、認可を適切に実装
6. **テスト**: ユニットテストとインテグレーションテストを記述
7. **パフォーマンス**: 適切なキャッシング戦略を実装

## 参考資料
- Hono 公式ドキュメント: https://hono.dev/
- Cloudflare Workers ドキュメント: https://developers.cloudflare.com/workers/
- Hono Examples: https://github.com/honojs/examples
