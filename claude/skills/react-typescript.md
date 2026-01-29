# React + TypeScript 開発スキル

## 概要
このスキルは、React と TypeScript を使用したフロントエンド開発のベストプラクティスとガイドラインを提供します。

## コーディング規約

### コンポーネント設計
- **関数コンポーネント**: クラスコンポーネントではなく、関数コンポーネントとHooksを使用してください
- **型定義**: すべてのPropsとStateに明示的な型定義を行ってください
- **ファイル命名**: コンポーネントファイルは PascalCase を使用 (例: `UserProfile.tsx`)
- **1ファイル1コンポーネント**: 原則として、1つのファイルには1つの主要コンポーネントのみを配置

### Props の型定義
```typescript
// Good
type UserProfileProps = {
  userId: string;
  userName: string;
  onUpdate?: (user: User) => void;
};

function UserProfile({ userId, userName, onUpdate }: UserProfileProps) {
  // ...
}
```

### Hooks の使用
- **カスタムフック**: ロジックの再利用には必ずカスタムフックを作成
- **useEffect の依存配列**: 必ず正確に指定し、ESLint の exhaustive-deps ルールに従う
- **useState の初期化**: 計算コストが高い初期値には関数形式を使用

```typescript
// Good: 複雑なロジックはカスタムフックに抽出
const useUserData = (userId: string) => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetchUser(userId).then(setUser).finally(() => setLoading(false));
  }, [userId]);
  
  return { user, loading };
};
```

### パフォーマンス最適化
- **React.memo**: 不要な再レンダリングを防ぐために適切に使用
- **useMemo / useCallback**: 計算コストが高い値や、子コンポーネントに渡す関数で使用
- **仮想化**: 大量のリストには react-window や react-virtualized を検討

### エラーハンドリング
- **Error Boundary**: エラー境界コンポーネントを適切に配置
- **エラー状態の管理**: ローディング、エラー、成功の状態を明確に管理

## ディレクトリ構造
```
src/
├── components/          # 再利用可能なUIコンポーネント
│   ├── common/         # 汎用コンポーネント
│   └── features/       # 機能固有のコンポーネント
├── hooks/              # カスタムフック
├── types/              # 型定義
├── utils/              # ユーティリティ関数
├── contexts/           # React Context
└── pages/              # ページコンポーネント
```

## テスト
- **React Testing Library**: Enzymeではなく、React Testing Libraryを使用
- **ユーザー視点**: 実装詳細ではなく、ユーザーの操作をテスト
- **型安全性**: テストコードにも型定義を適用

```typescript
import { render, screen, fireEvent } from '@testing-library/react';

test('ボタンクリックでカウントが増加する', () => {
  render(<Counter />);
  const button = screen.getByRole('button', { name: /increment/i });
  fireEvent.click(button);
  expect(screen.getByText('Count: 1')).toBeInTheDocument();
});
```

## よくある落とし穴
1. **useEffect の依存配列の不備**: 必要な依存関係を含めない
2. **不適切な状態管理**: prop drilling を避け、必要に応じて Context や状態管理ライブラリを使用
3. **キー属性の不適切な使用**: リストのインデックスをキーとして使用しない
4. **型の any 使用**: 型安全性を損なうため、any の使用は避ける

## 参考資料
- React 公式ドキュメント: https://react.dev/
- TypeScript 公式ドキュメント: https://www.typescriptlang.org/
- React TypeScript Cheatsheet: https://react-typescript-cheatsheet.netlify.app/
