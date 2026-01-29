# Unit Test 開発スキル

## 概要
このスキルは、JavaScript/TypeScript プロジェクトにおけるユニットテストのベストプラクティスを提供します。

## テストフレームワーク

### Jest の基本設定
```typescript
// jest.config.ts
import type { Config } from 'jest';

const config: Config = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  roots: ['<rootDir>/src'],
  testMatch: ['**/__tests__/**/*.ts', '**/?(*.)+(spec|test).ts'],
  collectCoverageFrom: [
    'src/**/*.ts',
    '!src/**/*.d.ts',
    '!src/**/*.test.ts',
    '!src/**/*.spec.ts',
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
};

export default config;
```

### Vitest の設定（モダンな代替）
```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'node',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: ['node_modules/', 'dist/'],
    },
  },
});
```

## テストの構造

### AAA パターン (Arrange-Act-Assert)
```typescript
import { describe, it, expect, beforeEach } from '@jest/globals';
import { Calculator } from './calculator';

describe('Calculator', () => {
  let calculator: Calculator;

  beforeEach(() => {
    // Arrange: テスト対象の準備
    calculator = new Calculator();
  });

  it('should add two numbers correctly', () => {
    // Arrange: テストデータの準備
    const a = 5;
    const b = 3;

    // Act: テスト対象のメソッドを実行
    const result = calculator.add(a, b);

    // Assert: 結果の検証
    expect(result).toBe(8);
  });
});
```

## モック (Mocking)

### 関数のモック
```typescript
import { jest, describe, it, expect } from '@jest/globals';
import { UserService } from './user-service';
import { UserRepository } from './user-repository';

// モジュール全体のモック
jest.mock('./user-repository');

describe('UserService', () => {
  let userService: UserService;
  let mockUserRepository: jest.Mocked<UserRepository>;

  beforeEach(() => {
    mockUserRepository = new UserRepository() as jest.Mocked<UserRepository>;
    userService = new UserService(mockUserRepository);
  });

  it('should return user by id', async () => {
    // モックの戻り値を設定
    const mockUser = { id: '1', name: 'Test User' };
    mockUserRepository.findById.mockResolvedValue(mockUser);

    // テスト実行
    const user = await userService.getUser('1');

    // 検証
    expect(user).toEqual(mockUser);
    expect(mockUserRepository.findById).toHaveBeenCalledWith('1');
    expect(mockUserRepository.findById).toHaveBeenCalledTimes(1);
  });
});
```

### スパイ (Spy)
```typescript
import { jest, describe, it, expect } from '@jest/globals';

describe('Logger', () => {
  it('should log messages', () => {
    const consoleSpy = jest.spyOn(console, 'log').mockImplementation();
    
    console.log('Test message');
    
    expect(consoleSpy).toHaveBeenCalledWith('Test message');
    
    consoleSpy.mockRestore();
  });
});
```

## 非同期テスト

### Promise のテスト
```typescript
describe('Async operations', () => {
  it('should resolve with data', async () => {
    const data = await fetchData();
    expect(data).toBeDefined();
  });

  it('should reject with error', async () => {
    await expect(fetchInvalidData()).rejects.toThrow('Invalid data');
  });

  // タイムアウトの設定
  it('should complete within timeout', async () => {
    await expect(slowOperation()).resolves.toBe('done');
  }, 10000); // 10秒のタイムアウト
});
```

### タイマーのモック
```typescript
describe('Timer tests', () => {
  beforeEach(() => {
    jest.useFakeTimers();
  });

  afterEach(() => {
    jest.useRealTimers();
  });

  it('should execute callback after delay', () => {
    const callback = jest.fn();
    
    setTimeout(callback, 1000);
    
    // 時間を進める
    jest.advanceTimersByTime(1000);
    
    expect(callback).toHaveBeenCalled();
  });

  it('should execute all timers', () => {
    const callback = jest.fn();
    
    setTimeout(callback, 1000);
    setTimeout(callback, 2000);
    
    jest.runAllTimers();
    
    expect(callback).toHaveBeenCalledTimes(2);
  });
});
```

## React コンポーネントのテスト

### React Testing Library
```typescript
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { UserProfile } from './UserProfile';

describe('UserProfile', () => {
  it('should render user information', () => {
    const user = { id: '1', name: 'John Doe', email: 'john@example.com' };
    
    render(<UserProfile user={user} />);
    
    expect(screen.getByText('John Doe')).toBeInTheDocument();
    expect(screen.getByText('john@example.com')).toBeInTheDocument();
  });

  it('should handle button click', async () => {
    const onUpdate = jest.fn();
    const user = { id: '1', name: 'John Doe' };
    
    render(<UserProfile user={user} onUpdate={onUpdate} />);
    
    const button = screen.getByRole('button', { name: /update/i });
    await userEvent.click(button);
    
    expect(onUpdate).toHaveBeenCalledWith(user);
  });

  it('should update input value', async () => {
    render(<UserProfile />);
    
    const input = screen.getByLabelText(/name/i);
    await userEvent.type(input, 'New Name');
    
    expect(input).toHaveValue('New Name');
  });

  it('should handle async operations', async () => {
    render(<UserProfile userId="1" />);
    
    expect(screen.getByText(/loading/i)).toBeInTheDocument();
    
    await waitFor(() => {
      expect(screen.getByText('John Doe')).toBeInTheDocument();
    });
  });
});
```

### カスタムフックのテスト
```typescript
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

describe('useCounter', () => {
  it('should initialize with default value', () => {
    const { result } = renderHook(() => useCounter());
    expect(result.current.count).toBe(0);
  });

  it('should increment counter', () => {
    const { result } = renderHook(() => useCounter());
    
    act(() => {
      result.current.increment();
    });
    
    expect(result.current.count).toBe(1);
  });

  it('should initialize with custom value', () => {
    const { result } = renderHook(() => useCounter(10));
    expect(result.current.count).toBe(10);
  });
});
```

## テストカバレッジ

### カバレッジレポートの生成
```bash
# Jest
npm test -- --coverage

# Vitest
npm test -- --coverage
```

### カバレッジ目標の設定
```typescript
// jest.config.ts
{
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
    './src/critical/': {
      branches: 100,
      functions: 100,
      lines: 100,
      statements: 100,
    },
  }
}
```

## テストデータの管理

### ファクトリーパターン
```typescript
// test-helpers/factories.ts
export const createUser = (overrides?: Partial<User>): User => ({
  id: '1',
  name: 'Test User',
  email: 'test@example.com',
  createdAt: new Date(),
  ...overrides,
});

export const createPost = (overrides?: Partial<Post>): Post => ({
  id: '1',
  title: 'Test Post',
  content: 'Test content',
  authorId: '1',
  ...overrides,
});

// 使用例
it('should update user', async () => {
  const user = createUser({ name: 'John Doe' });
  const result = await updateUser(user);
  expect(result.name).toBe('John Doe');
});
```

### フィクスチャ
```typescript
// __fixtures__/users.ts
export const validUser = {
  id: '1',
  name: 'Valid User',
  email: 'valid@example.com',
};

export const invalidUser = {
  id: '2',
  name: '',
  email: 'invalid-email',
};

// テストでの使用
import { validUser, invalidUser } from './__fixtures__/users';

it('should accept valid user', () => {
  expect(validateUser(validUser)).toBe(true);
});

it('should reject invalid user', () => {
  expect(validateUser(invalidUser)).toBe(false);
});
```

## スナップショットテスト

### コンポーネントのスナップショット
```typescript
import { render } from '@testing-library/react';
import { UserCard } from './UserCard';

describe('UserCard', () => {
  it('should match snapshot', () => {
    const user = { id: '1', name: 'John Doe', email: 'john@example.com' };
    const { container } = render(<UserCard user={user} />);
    
    expect(container).toMatchSnapshot();
  });
});
```

### インラインスナップショット
```typescript
it('should generate correct output', () => {
  const result = formatUser({ name: 'John', age: 30 });
  
  expect(result).toMatchInlineSnapshot(`
    {
      "age": 30,
      "name": "John",
    }
  `);
});
```

## テストのベストプラクティス

### 1. テストは独立させる
```typescript
// Bad: テスト間で状態を共有
let sharedUser: User;

it('creates user', () => {
  sharedUser = createUser();
});

it('updates user', () => {
  updateUser(sharedUser); // 前のテストに依存
});

// Good: 各テストで独立したセットアップ
it('creates user', () => {
  const user = createUser();
  expect(user).toBeDefined();
});

it('updates user', () => {
  const user = createUser();
  const updated = updateUser(user);
  expect(updated).toBeDefined();
});
```

### 2. 明確な命名
```typescript
// Bad: 不明確な名前
it('test1', () => {});

// Good: 明確で説明的な名前
it('should throw error when email is invalid', () => {});
it('should return user list sorted by creation date', () => {});
```

### 3. 1つのテストで1つの概念をテスト
```typescript
// Bad: 複数のことをテスト
it('should create and update user', () => {
  const user = createUser();
  expect(user).toBeDefined();
  
  const updated = updateUser(user);
  expect(updated.name).toBe('Updated');
});

// Good: 分離されたテスト
it('should create user', () => {
  const user = createUser();
  expect(user).toBeDefined();
});

it('should update user name', () => {
  const user = createUser();
  const updated = updateUser(user, { name: 'Updated' });
  expect(updated.name).toBe('Updated');
});
```

### 4. テストの可読性
```typescript
// Good: Given-When-Then コメント
it('should calculate total price with tax', () => {
  // Given: 商品と税率
  const items = [
    { price: 100, quantity: 2 },
    { price: 50, quantity: 1 },
  ];
  const taxRate = 0.1;

  // When: 合計を計算
  const total = calculateTotal(items, taxRate);

  // Then: 正しい金額が返される
  expect(total).toBe(275); // (100*2 + 50) * 1.1
});
```

## デバッグ

### テストのデバッグ
```typescript
import { screen, debug } from '@testing-library/react';

it('should render component', () => {
  render(<MyComponent />);
  
  // DOMの構造を確認
  screen.debug();
  
  // 特定の要素を確認
  const element = screen.getByRole('button');
  screen.debug(element);
});
```

### Only/Skip
```typescript
// 特定のテストのみ実行
it.only('should run only this test', () => {});

// テストをスキップ
it.skip('should skip this test', () => {});

// describe レベルでも使用可能
describe.only('Only this suite', () => {});
describe.skip('Skip this suite', () => {});
```

## ベストプラクティスまとめ
1. **高速なテスト**: ユニットテストは高速に実行されるべき
2. **独立性**: テスト間で状態を共有しない
3. **明確な失敗メッセージ**: 何が失敗したか明確にわかる
4. **実装詳細ではなく動作をテスト**: 内部実装に依存しない
5. **適切なカバレッジ**: 100%を目指すのではなく、重要な部分を確実にカバー

## 参考資料
- Jest 公式ドキュメント: https://jestjs.io/
- Vitest 公式ドキュメント: https://vitest.dev/
- React Testing Library: https://testing-library.com/react
