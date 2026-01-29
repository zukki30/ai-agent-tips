# 開発コマンド集

このファイルには、よく使用する開発コマンドをまとめています。

## Node.js / npm コマンド

### プロジェクトの初期化
```bash
# 新規プロジェクト作成（対話式）
npm init

# デフォルト設定で作成
npm init -y

# TypeScript プロジェクトの初期化
npm init -y && npm install -D typescript @types/node
npx tsc --init
```

### パッケージ管理
```bash
# 依存パッケージのインストール
npm install

# 特定のパッケージをインストール
npm install <package-name>

# 開発用依存パッケージをインストール
npm install -D <package-name>

# グローバルインストール
npm install -g <package-name>

# パッケージのアンインストール
npm uninstall <package-name>

# パッケージの更新
npm update

# パッケージの脆弱性チェック
npm audit

# 脆弱性の自動修正
npm audit fix

# 古いパッケージの確認
npm outdated
```

### スクリプト実行
```bash
# package.json の scripts を実行
npm run <script-name>

# 開発サーバー起動
npm run dev

# ビルド
npm run build

# 本番環境での起動
npm start
```

## React / Next.js コマンド

### プロジェクト作成
```bash
# Vite で React プロジェクト作成（推奨）
npm create vite@latest my-app -- --template react-ts

# Next.js プロジェクト作成
npx create-next-app@latest my-app --typescript --tailwind --app
```

### 開発・ビルド
```bash
# Next.js 開発サーバー起動
npm run dev

# Next.js 本番ビルド
npm run build

# Next.js 本番起動
npm start

# Next.js 静的エクスポート（next.config.js に output: 'export' を設定後）
npm run build
```

## Nest.js コマンド

### プロジェクト作成
```bash
# Nest.js CLI のインストール
npm install -g @nestjs/cli

# 新規プロジェクト作成
nest new my-project

# TypeScript strict mode で作成
nest new my-project --strict
```

### リソース生成
```bash
# モジュール作成
nest generate module users

# コントローラー作成
nest generate controller users

# サービス作成
nest generate service users

# CRUD リソース一式作成
nest generate resource users

# ガード作成
nest generate guard auth

# インターセプター作成
nest generate interceptor logging

# パイプ作成
nest generate pipe validation

# フィルター作成
nest generate filter http-exception
```

### 開発・ビルド
```bash
# 開発サーバー起動（ホットリロード）
npm run start:dev

# デバッグモード
npm run start:debug

# 本番ビルド
npm run build

# 本番起動
npm run start:prod
```

## TypeScript コマンド

### コンパイル
```bash
# TypeScript のコンパイル
npx tsc

# ウォッチモード
npx tsc --watch

# 特定のファイルをコンパイル
npx tsc src/index.ts

# tsconfig.json を指定
npx tsc --project tsconfig.production.json
```

### 開発ツール
```bash
# ts-node でTypeScriptファイルを直接実行
npx ts-node src/index.ts

# tsx でTypeScriptファイルを実行（高速）
npx tsx src/index.ts

# tsx のウォッチモード
npx tsx watch src/index.ts
```

## Git コマンド

### 基本操作
```bash
# 初期化
git init

# リポジトリのクローン
git clone <repository-url>

# 変更の確認
git status

# 差分の確認
git diff

# ファイルをステージング
git add <file>

# すべての変更をステージング
git add .

# コミット
git commit -m "commit message"

# プッシュ
git push origin <branch-name>

# プル
git pull origin <branch-name>
```

### ブランチ操作
```bash
# ブランチ一覧
git branch

# 新規ブランチ作成
git branch <branch-name>

# ブランチ切り替え
git checkout <branch-name>

# ブランチ作成と切り替えを同時に
git checkout -b <branch-name>

# ブランチの削除
git branch -d <branch-name>

# ブランチのマージ
git merge <branch-name>
```

### 履歴確認
```bash
# コミット履歴
git log

# コミット履歴（簡潔表示）
git log --oneline

# グラフ表示
git log --graph --oneline --all

# 特定ファイルの履歴
git log -- <file>
```

## Docker コマンド

### イメージ操作
```bash
# イメージのビルド
docker build -t <image-name> .

# イメージ一覧
docker images

# イメージの削除
docker rmi <image-id>
```

### コンテナ操作
```bash
# コンテナの起動
docker run -d -p 3000:3000 <image-name>

# コンテナ一覧（実行中）
docker ps

# コンテナ一覧（すべて）
docker ps -a

# コンテナの停止
docker stop <container-id>

# コンテナの削除
docker rm <container-id>

# コンテナのログ確認
docker logs <container-id>

# コンテナ内でコマンド実行
docker exec -it <container-id> /bin/bash
```

### Docker Compose
```bash
# サービスの起動
docker compose up

# バックグラウンドで起動
docker compose up -d

# サービスの停止
docker compose down

# ビルドして起動
docker compose up --build

# ログ確認
docker compose logs

# 特定サービスのログ
docker compose logs <service-name>
```

## データベースコマンド

### PostgreSQL
```bash
# データベースに接続
psql -U <username> -d <database>

# データベース一覧
\l

# テーブル一覧
\dt

# テーブル構造確認
\d <table-name>

# SQL実行
\i <sql-file>

# 終了
\q
```

### MySQL
```bash
# データベースに接続
mysql -u <username> -p

# データベース一覧
SHOW DATABASES;

# データベース選択
USE <database>;

# テーブル一覧
SHOW TABLES;

# テーブル構造確認
DESCRIBE <table>;
```

## Prisma コマンド

### 初期化とマイグレーション
```bash
# Prisma の初期化
npx prisma init

# マイグレーション作成
npx prisma migrate dev --name <migration-name>

# マイグレーション実行
npx prisma migrate deploy

# データベースのリセット
npx prisma migrate reset

# Prisma Client の生成
npx prisma generate
```

### その他
```bash
# Prisma Studio 起動（GUI）
npx prisma studio

# スキーマのフォーマット
npx prisma format

# スキーマの検証
npx prisma validate
```

## 環境変数管理

### .env ファイル
```bash
# .env.example から .env を作成
cp .env.example .env

# 環境変数を指定してコマンド実行
NODE_ENV=production npm start

# dotenv-cli を使用
npx dotenv -e .env.production npm start
```

## その他の便利コマンド

### パッケージ情報
```bash
# パッケージの情報を表示
npm info <package-name>

# パッケージのバージョン一覧
npm view <package-name> versions

# インストールされているパッケージのバージョン確認
npm list

# グローバルパッケージの確認
npm list -g --depth=0
```

### キャッシュ管理
```bash
# npm キャッシュのクリア
npm cache clean --force

# node_modules の削除と再インストール
rm -rf node_modules package-lock.json && npm install
```

### ポート確認・プロセス終了
```bash
# ポートを使用しているプロセスを確認（macOS/Linux）
lsof -i :<port-number>

# プロセスの終了
kill -9 <process-id>

# Windows でポート確認
netstat -ano | findstr :<port-number>

# Windows でプロセス終了
taskkill /PID <process-id> /F
```
