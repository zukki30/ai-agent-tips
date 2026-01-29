# TDD & レガシーコード改善スキル

## 概要

このスキルは、テスト駆動開発（TDD）とレガシーコード改善のベストプラクティスを提供します。
AI エージェントが TDD サイクルに従って開発を進め、テストのないコードを安全に改善するための指針です。

参考: [t_wada「Working with Legacy Code - the true record」](https://speakerdeck.com/twada/working-with-legacy-code-the-true-record)

## TDD の基本サイクル（Red → Green → Refactor）

AI エージェントは以下のサイクルを **厳密に守って** 開発を進めてください。

### 1. Red: 失敗するテストを書く

```typescript
// まずテストを書く（この時点では実装がないので失敗する）
describe('calculateTotal', () => {
  it('商品の合計金額を計算する', () => {
    const items = [
      { name: '商品A', price: 1000, quantity: 2 },
      { name: '商品B', price: 500, quantity: 1 },
    ];
    expect(calculateTotal(items)).toBe(2500);
  });
});
```

- テストを実行し、**失敗すること（Red）を確認する**
- 失敗を確認せずに実装に進まない

### 2. Green: テストを通す最小限の実装を書く

```typescript
// テストを通すための最小限の実装
function calculateTotal(items: CartItem[]): number {
  return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
}
```

- テストを通す **最小限のコード** だけを書く
- この段階では美しいコードである必要はない
- テストが Green になることを確認する

### 3. Refactor: テストを保ちながら改善する

```typescript
// リファクタリング例: 型の明確化とヘルパーの抽出
type CartItem = {
  name: string;
  price: number;
  quantity: number;
};

function itemSubtotal(item: CartItem): number {
  return item.price * item.quantity;
}

function calculateTotal(items: CartItem[]): number {
  return items.reduce((sum, item) => sum + itemSubtotal(item), 0);
}
```

- テストが通ったまま（Green を維持）リファクタリングする
- テストが Red になったらリファクタリングを巻き戻す

## レガシーコード改善のアプローチ

テストのないコード（レガシーコード）を改善する際の優先順位:

### ステップ 1: バージョン管理の確認

- コードがバージョン管理されていることを確認する
- バージョン管理なしでは安全なコード変更は不可能

### ステップ 2: 特性テスト（Characterization Test）を書く

既存の動作を記録するテストを最初に書く。

```typescript
// 既存の動作をそのまま記録する特性テスト
describe('既存の processOrder 関数', () => {
  it('正常な注文を処理できる（Happy Path）', async () => {
    const order = createTestOrder();
    const result = await processOrder(order);

    // 現在の動作をそのまま記録
    expect(result.status).toBe('completed');
    expect(result.totalAmount).toBe(3000);
  });
});
```

- まずは **Happy Path**（正常系）のテストだけで十分
- 完全なテストカバレッジは求めない
- 実装の詳細に依存しない、入出力レベルのテストを書く

### ステップ 3: Humble Object パターンでロジックを分離

フレームワーク依存のコードから純粋なロジックを抽出する。

```typescript
// Before: フレームワークに密結合
export const handler = async (event: APIGatewayEvent) => {
  const body = JSON.parse(event.body || '{}');
  const user = await db.findUser(body.userId);
  const total = user.cart.reduce((s, i) => s + i.price * i.qty, 0);
  const tax = total * 0.1;
  await db.saveOrder({ userId: body.userId, total: total + tax });
  return { statusCode: 200, body: JSON.stringify({ total: total + tax }) };
};

// After: Humble Object パターンで分離
// ドメインロジック（テスト容易）
function calculateOrderTotal(cart: CartItem[]): number {
  return cart.reduce((sum, item) => sum + item.price * item.qty, 0);
}

function calculateTax(amount: number, rate: number = 0.1): number {
  return amount * rate;
}

// ハンドラは薄いアダプター（Humble Object）
export const handler = async (event: APIGatewayEvent) => {
  const body = JSON.parse(event.body || '{}');
  const user = await db.findUser(body.userId);
  const subtotal = calculateOrderTotal(user.cart);
  const total = subtotal + calculateTax(subtotal);
  await db.saveOrder({ userId: body.userId, total });
  return { statusCode: 200, body: JSON.stringify({ total }) };
};
```

### ステップ 4: アーキテクチャの層分離

段階的にコードを以下の層に分離する:

```
┌─────────────────────────────┐
│   基盤層（Infrastructure）    │  API Gateway, Express, etc.
├─────────────────────────────┤
│   情報層（Information）       │  ビジネスルール、計算ロジック
├─────────────────────────────┤
│   事実層（Facts）             │  イベント、データ記録
├─────────────────────────────┤
│   永続化層（Persistence）     │  DB, ストレージ
└─────────────────────────────┘
```

- **基盤層**: フレームワーク固有の処理（薄く保つ）
- **情報層**: 事実から計算される意味のあるデータ（テスト重点）
- **事実層**: 実際に発生したイベントやデータの記録
- **永続化層**: データベースやストレージへの読み書き

## AI エージェントが TDD を行う際のルール

### 必ず守ること

1. **テストを先に書く**: 実装コードの前にテストを書く
2. **Red を確認する**: テストが失敗することを実行して確認する
3. **最小限の実装**: テストを通すための最小コードだけ書く
4. **Green を確認する**: テストが通ることを実行して確認する
5. **リファクタリング**: Green を維持しながらコードを改善する
6. **サイクルを繰り返す**: 次の機能も同じサイクルで進める

### やってはいけないこと

1. テストを書かずに実装を始める
2. Red を確認せずに実装に入る
3. 一度に大量のテストと実装を書く
4. テストが Red のままリファクタリングする
5. テストの実行をスキップする

## テストの優先順位

```
高優先度:
├── ビジネスロジックのユニットテスト
├── API エンドポイントの統合テスト
├── 境界値・エッジケースのテスト
└── エラーハンドリングのテスト

中優先度:
├── コンポーネントテスト
├── カスタムフックのテスト
└── ユーティリティ関数のテスト

低優先度（ただし重要なフローは必須）:
└── E2E テスト
```

## 参考資料

- Michael Feathers『レガシーコード改善ガイド』
- Kent Beck『テスト駆動開発』
- Martin Fowler『リファクタリング』
