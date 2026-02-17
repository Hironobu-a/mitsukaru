# Nemielu 開発環境ガイド

このドキュメントでは、Dockerを使用したローカル開発環境の使い方を説明します。

## 📋 概要

Nemieluプロジェクトは、Dockerを使用してローカル開発環境を構築します。以下のサービスが含まれます：

- **app**: Laravel アプリケーション（PHP 8.0 + Laravel 8）
- **db**: MySQL 8.0 データベース
- **test**: テスト実行専用（SQLite使用）

---

## 🚀 クイックスタート

### 初回セットアップ

```bash
# 1. Dockerイメージをビルド
docker-compose build app

# 2. 依存関係をインストール
docker-compose run --rm app composer install

# 3. データベースをセットアップ
docker-compose up -d db
sleep 10  # データベースの起動を待つ
docker-compose run --rm app php artisan migrate:fresh

# 4. アプリケーションを起動
docker-compose up -d app
```

### 簡単な起動方法

```bash
# アプリケーションを起動
./docker-start.sh

# ブラウザで http://localhost:8000 にアクセス
```

---

## 🎯 基本的な使い方

### アプリケーションの起動

```bash
# 方法1: 便利スクリプトを使用（推奨）
./docker-start.sh

# 方法2: docker-composeを直接使用
docker-compose up -d
```

### アプリケーションの停止

```bash
# 方法1: 便利スクリプトを使用
./docker-stop.sh

# 方法2: docker-composeを直接使用
docker-compose down
```

### アプリケーションへのアクセス

起動後、以下のURLでアクセスできます：

- **アプリケーション**: http://localhost:8000
- **データベース**: localhost:3306
  - ユーザー: `laravel`
  - パスワード: `secret`
  - データベース: `laravel`

---

## 🔧 開発コマンド

### ログの確認

```bash
# 全てのログを表示
docker-compose logs -f

# アプリケーションのログのみ
docker-compose logs -f app

# データベースのログのみ
docker-compose logs -f db

# 最新50行のみ
docker-compose logs --tail=50 app
```

### コンテナに入る

```bash
# アプリケーションコンテナ
docker-compose exec app bash

# データベースコンテナ
docker-compose exec db bash
```

### Artisanコマンドの実行

```bash
# キャッシュクリア
docker-compose exec app php artisan cache:clear

# 設定キャッシュクリア
docker-compose exec app php artisan config:clear

# ルート一覧
docker-compose exec app php artisan route:list

# マイグレーション
docker-compose exec app php artisan migrate

# マイグレーションのロールバック
docker-compose exec app php artisan migrate:rollback

# データベースを初期化して再マイグレーション
docker-compose exec app php artisan migrate:fresh

# シーダー実行
docker-compose exec app php artisan db:seed
```

### Composerコマンド

```bash
# 依存関係をインストール
docker-compose run --rm app composer install

# 依存関係を更新
docker-compose run --rm app composer update

# パッケージを追加
docker-compose run --rm app composer require vendor/package

# オートロードを再生成
docker-compose run --rm app composer dump-autoload
```

### NPMコマンド

```bash
# 依存関係をインストール
docker-compose exec app npm install

# 開発ビルド
docker-compose exec app npm run dev

# ウォッチモード
docker-compose exec app npm run watch

# 本番ビルド
docker-compose exec app npm run prod
```

---

## 🧪 テストの実行

テストは専用のコンテナで実行します（SQLite使用）：

```bash
# 全てのテストを実行
./docker-test.sh

# ユニットテストのみ
./docker-test.sh unit

# 機能テストのみ
./docker-test.sh feature

# 詳細は DOCKER_TESTING_GUIDE.md を参照
```

---

## 🗄️ データベース管理

### マイグレーション

```bash
# マイグレーションを実行
docker-compose exec app php artisan migrate

# マイグレーションのステータス確認
docker-compose exec app php artisan migrate:status

# 最後のマイグレーションをロールバック
docker-compose exec app php artisan migrate:rollback

# 全てロールバックして再実行
docker-compose exec app php artisan migrate:fresh

# マイグレーション + シーダー
docker-compose exec app php artisan migrate:fresh --seed
```

### データベースへの直接接続

```bash
# MySQLクライアントで接続
docker-compose exec db mysql -ularavel -psecret laravel

# または、ホストマシンから
mysql -h127.0.0.1 -P3306 -ularavel -psecret laravel
```

### データベースのバックアップ

```bash
# ダンプを作成
docker-compose exec db mysqldump -ularavel -psecret laravel > backup.sql

# リストア
docker-compose exec -T db mysql -ularavel -psecret laravel < backup.sql
```

---

## 🛠️ トラブルシューティング

### ポートがすでに使用されている

```bash
# 既存のコンテナを停止
docker-compose down

# または、docker-compose.yml のポート番号を変更
# ports:
#   - "8001:8000"  # 8000 -> 8001 に変更
```

### データベース接続エラー

```bash
# データベースが完全に起動するまで待つ
docker-compose up -d db
sleep 15

# データベースのログを確認
docker-compose logs db

# 再起動
docker-compose restart db
```

### パーミッションエラー

```bash
# コンテナ内で権限を修正
docker-compose exec app chown -R www-data:www-data storage bootstrap/cache
docker-compose exec app chmod -R 775 storage bootstrap/cache
```

### キャッシュの問題

```bash
# 全てのキャッシュをクリア
docker-compose exec app php artisan cache:clear
docker-compose exec app php artisan config:clear
docker-compose exec app php artisan route:clear
docker-compose exec app php artisan view:clear
docker-compose exec app composer dump-autoload
```

### Dockerイメージの問題

```bash
# イメージを再ビルド
docker-compose build --no-cache app

# 古いイメージを削除
docker system prune -a
```

### コンテナが起動しない

```bash
# 詳細なログを確認
docker-compose logs app

# コンテナを削除して再作成
docker-compose down
docker-compose up -d
```

---

## 📂 ディレクトリ構成

```
nemielu-dev/
├── app/                    # アプリケーションコード
├── database/
│   ├── migrations/        # データベースマイグレーション
│   ├── seeders/          # データベースシーダー
│   └── factories/        # テストデータ生成
├── public/               # 公開ファイル（CSS, JS, 画像）
├── resources/            # ビュー、アセット
├── routes/               # ルート定義
├── tests/                # テストコード
├── storage/              # ログ、キャッシュ
├── .env                  # 環境変数（本番/開発）
├── .env.testing          # テスト環境変数
├── docker-compose.yml    # Docker構成
├── Dockerfile            # PHPコンテナ定義
├── docker-start.sh       # 起動スクリプト ⭐
├── docker-stop.sh        # 停止スクリプト ⭐
├── docker-test.sh        # テスト実行スクリプト
└── phpunit.xml           # テスト設定
```

---

## 🔐 環境変数

### 開発環境（.env）

```env
APP_NAME=Nemielu
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost:8000

DB_CONNECTION=mysql
DB_HOST=db
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=laravel
DB_PASSWORD=secret
```

### テスト環境（.env.testing）

```env
APP_ENV=testing
DB_CONNECTION=sqlite
DB_DATABASE=:memory:
MAIL_MAILER=array
```

---

## 🎨 フロントエンド開発

### アセットのビルド

```bash
# 開発ビルド
docker-compose exec app npm run dev

# ウォッチモード（自動再ビルド）
docker-compose exec app npm run watch

# 本番ビルド（最適化）
docker-compose exec app npm run prod
```

### Tailwind CSS

プロジェクトはTailwind CSS 2を使用しています。

```bash
# Tailwindの再ビルド
docker-compose exec app npm run dev
```

---

## 📊 パフォーマンス

### キャッシュの最適化

```bash
# 設定をキャッシュ
docker-compose exec app php artisan config:cache

# ルートをキャッシュ
docker-compose exec app php artisan route:cache

# ビューをキャッシュ
docker-compose exec app php artisan view:cache

# Composerの最適化
docker-compose exec app composer dump-autoload -o
```

### デバッグモードの切り替え

```bash
# .env ファイルを編集
APP_DEBUG=false  # 本番環境では false に

# 設定を再読み込み
docker-compose exec app php artisan config:cache
```

---

## 🔄 開発ワークフロー

### 日常的な開発

```bash
# 1. 朝、開発を開始
./docker-start.sh

# 2. コードを編集
# ... エディタでコードを変更 ...

# 3. 変更を確認
# ブラウザで http://localhost:8000 にアクセス

# 4. テストを実行
./docker-test.sh

# 5. 夜、作業を終了
./docker-stop.sh
```

### 新機能の追加

```bash
# 1. マイグレーションを作成
docker-compose exec app php artisan make:migration create_new_table

# 2. モデルを作成
docker-compose exec app php artisan make:model NewModel

# 3. コントローラーを作成
docker-compose exec app php artisan make:controller NewController

# 4. マイグレーションを実行
docker-compose exec app php artisan migrate

# 5. テストを作成
docker-compose exec app php artisan make:test NewFeatureTest

# 6. テストを実行
./docker-test.sh
```

---

## 📚 関連ドキュメント

- **`DOCKER_TESTING_GUIDE.md`** - テスト実行の詳細ガイド
- **`tests/README.md`** - テスト全般のガイド
- **`TEST_COMPLETION_REPORT.md`** - テスト整備報告書
- **[Laravel 8 Documentation](https://laravel.com/docs/8.x)** - Laravel公式ドキュメント

---

## 🎓 ヒントとコツ

### エイリアスの設定

`.bashrc` や `.zshrc` に以下を追加すると便利です：

```bash
# Nemielu開発エイリアス
alias nemielu-start="cd /path/to/nemielu-dev && ./docker-start.sh"
alias nemielu-stop="cd /path/to/nemielu-dev && ./docker-stop.sh"
alias nemielu-test="cd /path/to/nemielu-dev && ./docker-test.sh"
alias nemielu-logs="cd /path/to/nemielu-dev && docker-compose logs -f app"
alias nemielu-shell="cd /path/to/nemielu-dev && docker-compose exec app bash"
```

### VSCode統合

VSCodeを使用している場合、以下の拡張機能が便利です：

- Docker
- PHP Intelephense
- Laravel Extension Pack
- Tailwind CSS IntelliSense

---

## ✅ チェックリスト

### 開発開始前

- [ ] Dockerがインストールされている
- [ ] docker-composeが使用可能
- [ ] イメージがビルドされている
- [ ] 依存関係がインストールされている
- [ ] データベースがマイグレートされている

### 開発中

- [ ] コード変更後にテストを実行
- [ ] ログを定期的に確認
- [ ] キャッシュ問題が発生したらクリア

### 終了時

- [ ] 変更をコミット
- [ ] テストが全て成功
- [ ] コンテナを停止（./docker-stop.sh）

---

## 🎉 まとめ

Docker環境を使用することで、以下のメリットがあります：

1. ✅ **環境の一貫性** - 全員が同じ環境で開発
2. ✅ **簡単なセットアップ** - `./docker-start.sh` だけ
3. ✅ **クリーンな環境** - ローカル環境を汚さない
4. ✅ **本番に近い環境** - Dockerコンテナで実行
5. ✅ **効率的な開発** - ホットリロード、自動テスト

---

**質問や問題がある場合は、`docker-compose logs -f` でログを確認してください！**
