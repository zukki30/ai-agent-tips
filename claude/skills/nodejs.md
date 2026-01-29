# Node.js 開発スキル

## 概要
このスキルは、Node.js を使用したサーバーサイド JavaScript/TypeScript 開発のベストプラクティスを提供します。

## プロジェクト構成

### 推奨ディレクトリ構造
```
project/
├── src/
│   ├── controllers/
│   ├── services/
│   ├── models/
│   ├── routes/
│   ├── middleware/
│   ├── utils/
│   ├── config/
│   └── index.ts
├── tests/
├── dist/              # ビルド出力
├── node_modules/
├── package.json
├── tsconfig.json
└── .env
```

## パッケージ管理

### package.json の基本
```json
{
  "name": "my-node-app",
  "version": "1.0.0",
  "type": "module",
  "engines": {
    "node": ">=18.0.0",
    "npm": ">=9.0.0"
  },
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js",
    "test": "jest",
    "lint": "eslint src/**/*.ts",
    "format": "prettier --write src/**/*.ts"
  },
  "dependencies": {
    "express": "^4.18.0",
    "dotenv": "^16.0.0"
  },
  "devDependencies": {
    "@types/express": "^4.17.0",
    "@types/node": "^20.0.0",
    "typescript": "^5.0.0",
    "tsx": "^4.0.0"
  }
}
```

## TypeScript 設定

### tsconfig.json
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "lib": ["ES2022"],
    "moduleResolution": "bundler",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "tests"]
}
```

## 非同期処理

### async/await の使用
```typescript
// Good: エラーハンドリング付き
async function fetchUser(userId: string): Promise<User> {
  try {
    const response = await fetch(`/api/users/${userId}`);
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const user = await response.json();
    return user;
  } catch (error) {
    console.error('Failed to fetch user:', error);
    throw error;
  }
}

// 並列実行
async function fetchMultipleUsers(userIds: string[]): Promise<User[]> {
  const promises = userIds.map(id => fetchUser(id));
  return await Promise.all(promises);
}
```

### Promise のエラーハンドリング
```typescript
// Good: 適切なエラーハンドリング
fetchUser('123')
  .then(user => processUser(user))
  .catch(error => handleError(error))
  .finally(() => cleanup());

// Better: async/await
async function main() {
  try {
    const user = await fetchUser('123');
    await processUser(user);
  } catch (error) {
    handleError(error);
  } finally {
    cleanup();
  }
}
```

## ストリーム処理

### ファイルストリーム
```typescript
import { createReadStream, createWriteStream } from 'fs';
import { pipeline } from 'stream/promises';

// ファイルのコピー
async function copyFile(source: string, destination: string): Promise<void> {
  await pipeline(
    createReadStream(source),
    createWriteStream(destination)
  );
}

// データ変換ストリーム
import { Transform } from 'stream';

const upperCaseTransform = new Transform({
  transform(chunk, encoding, callback) {
    const upperCased = chunk.toString().toUpperCase();
    callback(null, upperCased);
  }
});

await pipeline(
  createReadStream('input.txt'),
  upperCaseTransform,
  createWriteStream('output.txt')
);
```

## エラーハンドリング

### カスタムエラークラス
```typescript
// カスタムエラーの定義
export class AppError extends Error {
  constructor(
    public message: string,
    public statusCode: number = 500,
    public isOperational: boolean = true
  ) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }
}

export class NotFoundError extends AppError {
  constructor(message: string = 'Resource not found') {
    super(message, 404);
  }
}

export class ValidationError extends AppError {
  constructor(message: string = 'Validation failed') {
    super(message, 400);
  }
}

// 使用例
if (!user) {
  throw new NotFoundError('User not found');
}
```

### グローバルエラーハンドラー
```typescript
import { Request, Response, NextFunction } from 'express';

export function errorHandler(
  error: Error,
  req: Request,
  res: Response,
  next: NextFunction
) {
  if (error instanceof AppError) {
    return res.status(error.statusCode).json({
      status: 'error',
      message: error.message,
    });
  }

  // 予期しないエラー
  console.error('Unexpected error:', error);
  return res.status(500).json({
    status: 'error',
    message: 'Internal server error',
  });
}
```

## 環境変数管理

### dotenv の使用
```typescript
// config/env.ts
import { config } from 'dotenv';
import { z } from 'zod';

config();

const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'production', 'test']).default('development'),
  PORT: z.string().default('3000'),
  DATABASE_URL: z.string(),
  JWT_SECRET: z.string(),
  LOG_LEVEL: z.enum(['error', 'warn', 'info', 'debug']).default('info'),
});

export const env = envSchema.parse(process.env);
```

## ロギング

### 構造化ロギング
```typescript
import pino from 'pino';

export const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  transport: {
    target: 'pino-pretty',
    options: {
      colorize: true,
      translateTime: 'SYS:standard',
      ignore: 'pid,hostname',
    },
  },
});

// 使用例
logger.info({ userId: '123' }, 'User logged in');
logger.error({ err: error }, 'Failed to process request');
logger.debug({ data: response }, 'API response');
```

## データベース接続

### 接続プール管理
```typescript
import { Pool } from 'pg';

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

// クエリの実行
export async function query<T>(
  text: string,
  params?: any[]
): Promise<T[]> {
  const client = await pool.connect();
  try {
    const result = await client.query(text, params);
    return result.rows;
  } finally {
    client.release();
  }
}

// トランザクション
export async function transaction<T>(
  callback: (client: any) => Promise<T>
): Promise<T> {
  const client = await pool.connect();
  try {
    await client.query('BEGIN');
    const result = await callback(client);
    await client.query('COMMIT');
    return result;
  } catch (error) {
    await client.query('ROLLBACK');
    throw error;
  } finally {
    client.release();
  }
}
```

## セキュリティ

### 入力の検証とサニタイゼーション
```typescript
import { z } from 'zod';

// スキーマ定義
const userSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
  name: z.string().min(1).max(100),
});

// バリデーション
function validateUser(data: unknown) {
  return userSchema.parse(data);
}

// 使用例
try {
  const validUser = validateUser(requestBody);
  // 処理続行
} catch (error) {
  if (error instanceof z.ZodError) {
    // バリデーションエラーの処理
    return res.status(400).json({ errors: error.errors });
  }
}
```

### セキュリティヘッダー
```typescript
import helmet from 'helmet';
import express from 'express';

const app = express();

// セキュリティヘッダーの設定
app.use(helmet());

// CORS設定
app.use((req, res, next) => {
  res.header('Access-Control-Allow-Origin', process.env.ALLOWED_ORIGIN);
  res.header('Access-Control-Allow-Methods', 'GET,POST,PUT,DELETE');
  res.header('Access-Control-Allow-Headers', 'Content-Type,Authorization');
  next();
});
```

## パフォーマンス最適化

### クラスタリング
```typescript
import cluster from 'cluster';
import os from 'os';

if (cluster.isPrimary) {
  const numCPUs = os.cpus().length;
  console.log(`Primary ${process.pid} is running`);

  // ワーカープロセスの生成
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died`);
    cluster.fork(); // 再起動
  });
} else {
  // ワーカープロセスでアプリケーションを起動
  startApp();
  console.log(`Worker ${process.pid} started`);
}
```

## テスト

### Jest でのユニットテスト
```typescript
import { describe, it, expect, beforeEach, jest } from '@jest/globals';
import { fetchUser } from './user-service';

describe('fetchUser', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('should fetch user successfully', async () => {
    const userId = '123';
    const user = await fetchUser(userId);
    
    expect(user).toBeDefined();
    expect(user.id).toBe(userId);
  });

  it('should throw error when user not found', async () => {
    await expect(fetchUser('invalid')).rejects.toThrow('User not found');
  });
});
```

## ベストプラクティス
1. **TypeScript の使用**: 型安全性を確保
2. **async/await の使用**: コールバック地獄を避ける
3. **エラーハンドリング**: すべての async 関数に try-catch を実装
4. **環境変数の検証**: 起動時に必須の環境変数を確認
5. **ロギング**: 適切なログレベルで構造化ログを出力
6. **セキュリティ**: helmet などのセキュリティミドルウェアを使用

## 参考資料
- Node.js 公式ドキュメント: https://nodejs.org/docs/
- TypeScript 公式ドキュメント: https://www.typescriptlang.org/
