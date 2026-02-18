# ミツプロ (税理士転職・求人サイト) - はじめに (Getting Started)

ミツプロ（https://tax.mitsukaru-pro.co.jp/）プロジェクトへようこそ。このドキュメントは、プロジェクトに参加する最初のステップとして、環境構築からドキュメントの歩き方までを案内します。

---

## 📚 ドキュメント構成 (Directory Map)

プロジェクトのドキュメントは、目的別に以下のフォルダに整理されています。

- **1_Management_Strategy/**: 経営方針、技術ロードマップ、チーム体制など
  - `CTO_MANAGEMENT_PLAN.md`: 技術戦略・開発方針書
  - `TECH_DEBT_ROADMAP.md`: 技術的負債の解消計画
  - `TEAM_STRUCTURE.md`: チーム体制と役割分担
- **2_Development_Environment/**: 開発環境構築、ツールガイド
  - `GETTING_STARTED.md`: (本書) 環境構築の第一歩
  - `DEVELOPMENT_GUIDE.md`: 詳細な開発フロー
  - `DATABASE_TOOLS_GUIDE.md`: DBツールの使い方
- **3_Testing_Quality/**: テスト方針、品質管理
  - `EFFICIENT_TEAM_TEST_STRATEGY.md`: テスト戦略の全体像
  - `DOCKER_TESTING_GUIDE.md`: Dockerでのテスト実行方法
- **4_Infrastructure_Operations/**: インフラ、デプロイ、運用
  - `DOMAIN_CICD_POLICY_AND_STEPS.md`: CI/CDパイプラインの方針

---

## 🚀 最速で起動する（3ステップ）

### ステップ1: Dockerイメージをビルド

```bash
docker-compose build app
```

### ステップ2: 初期セットアップ（初回のみ）

```bash
# 依存関係をインストール
docker-compose run --rm app composer install

# データベースをセットアップ
docker-compose up -d db
sleep 10
docker-compose run --rm app php artisan migrate:fresh
```

### ステップ3: アプリケーションを起動

```bash
# 起動スクリプトを使用
./docker-start.sh

# または、直接起動
docker-compose up -d
```

### ステップ4: ブラウザでアクセス

ブラウザで以下のURLを開きます：

🌐 **http://localhost:8000**

---

## 📋 システム要件

- **Docker**: 20.10 以上
- **Docker Compose**: 2.0 以上
- **ディスク空き容量**: 2GB以上
- **メモリ**: 4GB以上推奨

---

## 🎯 主な機能

Nemieluは睡眠・眠気管理システムです：

- 👤 **ユーザー管理** - 一般ユーザーと管理者
- 📊 **PVTテスト** - 反応時間測定
- 😴 **眠気評価** - 眠気スコアの記録
- 📈 **レポート** - データの可視化
- ⏰ **クロノタイプ評価** - 睡眠タイプの判定

---

## 📚 ドキュメント

詳しい使い方は以下のドキュメントを参照してください：

| ドキュメント | 説明 |
|---|---|
| **`DEVELOPMENT_GUIDE.md`** | 開発環境の詳細ガイド |
| **`DOCKER_TESTING_GUIDE.md`** | テスト実行ガイド |
| **`tests/README.md`** | テストの書き方 |
| **`TEST_COMPLETION_REPORT.md`** | テスト整備報告 |

---

## 🔧 便利なコマンド

### アプリケーションの管理

```bash
# 起動
./docker-start.sh

# 停止
./docker-stop.sh

# ログを表示
docker-compose logs -f app

# コンテナに入る
docker-compose exec app bash
```

### テストの実行

```bash
# 全てのテストを実行
./docker-test.sh

# ユニットテストのみ
./docker-test.sh unit

# 機能テストのみ
./docker-test.sh feature
```

### データベース操作

```bash
# マイグレーションを実行
docker-compose exec app php artisan migrate

# データベースを初期化
docker-compose exec app php artisan migrate:fresh

# シーダーを実行
docker-compose exec app php artisan db:seed
```

---

## 🏗️ プロジェクト構成

```
mitsupro-tax/
├── app/                    # アプリケーションコード
│   ├── Http/Controllers/  # コントローラー
│   └── Models/            # Eloquentモデル
├── database/
│   ├── migrations/        # データベース構造
│   ├── seeders/          # 初期データ
│   └── factories/        # テストデータ生成
├── resources/
│   └── views/            # Bladeテンプレート
├── routes/
│   ├── web.php           # Webルート
│   ├── api.php           # APIルート
│   └── admin.php         # 管理者ルート
├── tests/
│   ├── Unit/             # ユニットテスト
│   └── Feature/          # 機能テスト
├── docker-start.sh       # 起動スクリプト ⭐
├── docker-stop.sh        # 停止スクリプト ⭐
└── docker-test.sh        # テスト実行スクリプト ⭐
```

---

## 🌐 アクセスURL

起動後、以下のURLでアクセスできます：

| サービス | URL | 説明 |
|---|---|---|
| アプリケーション | http://localhost:8000 | メインアプリ |
| 一般ユーザーログイン | http://localhost:8000/login | ユーザー画面 |
| 管理者ログイン | http://localhost:8000/admin/login | 管理画面 |
| ユーザー登録 | http://localhost:8000/register | 新規登録 |

---

## 👥 初期アカウント

初回セットアップ後、以下のアカウントを作成してください：

### 管理者アカウントの作成

```bash
docker-compose exec app php artisan tinker
```

Tinkerで以下を実行：

```php
$admin = new App\Models\Admin();
$admin->name = '管理者';
$admin->email = 'admin@example.com';
$admin->password = bcrypt('password');
$admin->client_code = 'ADMIN001';
$admin->save();
```

### 一般ユーザーは登録画面から作成

http://localhost:8000/register にアクセスして新規登録

---

## 🔍 トラブルシューティング

### アプリケーションが起動しない

```bash
# ログを確認
docker-compose logs app

# コンテナを再起動
docker-compose restart app
```

### データベース接続エラー

```bash
# データベースが起動しているか確認
docker-compose ps

# データベースを再起動
docker-compose restart db

# 15秒待ってから再度アクセス
```

### ポートがすでに使用されている

```bash
# 既存のコンテナを停止
docker-compose down

# または、別のポートを使用
# docker-compose.yml で "8001:8000" に変更
```

### パーミッションエラー

```bash
# 権限を修正
docker-compose exec app chmod -R 775 storage bootstrap/cache
docker-compose exec app chown -R www-data:www-data storage bootstrap/cache
```

---

## 📝 開発の流れ

### 1. 朝の開発開始

```bash
# アプリケーションを起動
./docker-start.sh

# ブラウザでアクセス
open http://localhost:8000
```

### 2. コード編集

お好みのエディタでコードを編集します。変更は自動的にコンテナに反映されます。

### 3. テストの実行

```bash
# コード変更後にテストを実行
./docker-test.sh
```

### 4. 夜の作業終了

```bash
# アプリケーションを停止
./docker-stop.sh

# 変更をコミット
git add .
git commit -m "機能追加"
```

---

## 🎓 次のステップ

プロジェクトに慣れたら、以下のドキュメントを読んでください：

1. **`DEVELOPMENT_GUIDE.md`** - より詳しい開発方法
2. **`DOCKER_TESTING_GUIDE.md`** - テストの書き方と実行方法
3. **Laravel 8 ドキュメント** - https://laravel.com/docs/8.x

---

## 💡 ヒント

### エイリアスを設定すると便利

`.bashrc` や `.zshrc` に追加：

```bash
alias ms="cd /path/to/mitsupro-tax && ./docker-start.sh"
alias mt="cd /path/to/mitsupro-tax && ./docker-test.sh"
```

### VSCode拡張機能

- Docker
- PHP Intelephense
- Laravel Extension Pack
- Tailwind CSS IntelliSense

---

## 🎉 準備完了！

これで開発を始める準備が整いました。

**ブラウザで http://localhost:8000 を開いて、ミツプロを体験してください！**

質問や問題がある場合は、各ドキュメントを参照するか、ログを確認してください：

```bash
docker-compose logs -f app
```

Happy Coding! 🚀
