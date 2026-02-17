# Docker テスト実行環境ガイド

このドキュメントでは、Dockerを使用したテスト実行環境の構築と使用方法について説明します。

## 📋 概要

Nemieluプロジェクトのテストは、Dockerコンテナ内で実行されます。これにより、ローカル環境を汚すことなく、一貫した環境でテストを実行できます。

---

## 🐳 Docker環境の構成

### サービス構成

`docker-compose.yml` には以下のサービスが定義されています：

1. **app** - アプリケーション実行用（開発サーバー）
2. **test** - テスト実行専用（SQLiteメモリDB使用）
3. **db** - MySQL データベース（開発用）

### テストサービスの特徴

- **データベース**: SQLiteメモリDB（高速、依存なし）
- **メール**: 配列ドライバー（実際の送信なし）
- **キャッシュ**: 配列ドライバー（永続化なし）
- **セッション**: 配列ドライバー
- **プロファイル**: `test` プロファイルで起動

---

## 🚀 クイックスタート

### 1. 初回セットアップ

```bash
# Dockerイメージをビルド
./docker-test.sh build

# Composer依存関係をインストール
./docker-test.sh install
```

### 2. テストの実行

```bash
# 全てのテストを実行
./docker-test.sh

# または
docker-compose run --rm test php artisan test
```

---

## 📝 テスト実行コマンド

### 基本的なテスト実行

```bash
# 全てのテストを実行
./docker-test.sh

# ユニットテストのみ
./docker-test.sh unit

# 機能テストのみ
./docker-test.sh feature
```

### フィルタリング

```bash
# 特定のテスト名でフィルタ
./docker-test.sh filter test_user_can_be_created

# 特定のテストファイルを実行
./docker-test.sh file tests/Unit/Models/UserTest.php
```

### メンテナンス

```bash
# Dockerイメージを再ビルド
./docker-test.sh build

# Composer依存関係をインストール
./docker-test.sh install

# Composer依存関係を更新
./docker-test.sh update

# テストコンテナのシェルに入る
./docker-test.sh shell

# Dockerリソースをクリーンアップ
./docker-test.sh clean
```

### ヘルプの表示

```bash
./docker-test.sh help
```

---

## 🔧 詳細なコマンド

### docker-compose コマンドを直接使用

便利スクリプトを使わずに、直接 `docker-compose` コマンドを使うこともできます：

```bash
# 全てのテストを実行
docker-compose run --rm test php artisan test

# ユニットテストのみ
docker-compose run --rm test php artisan test --testsuite=Unit

# 機能テストのみ
docker-compose run --rm test php artisan test --testsuite=Feature

# 特定のテストファイル
docker-compose run --rm test php artisan test tests/Unit/Models/UserTest.php

# フィルタ
docker-compose run --rm test php artisan test --filter test_user_can_be_created

# コンテナのシェルに入る
docker-compose run --rm test bash
```

---

## 📊 テスト結果の例

### 成功した場合

```
  PASS  Tests\Unit\Models\UserTest
  ✓ user can be created
  ✓ user fillable attributes
  ✓ user hidden attributes
  ✓ user password is hidden in json
  ✓ email verified at is cast to datetime

  Tests:  89 passed
  Time:   1.47s
```

### 失敗した場合

失敗したテストは赤く表示され、詳細なエラーメッセージが表示されます：

```
  FAIL  Tests\Unit\Models\UserTest
  ⨯ user can be created

  FAILED  Tests\Unit\Models\UserTest > user can be created
  
  Expected status code 200 but received 500.
  ...
```

---

## 🔍 トラブルシューティング

### 1. Dockerイメージがビルドできない

```bash
# キャッシュをクリアして再ビルド
docker-compose build --no-cache app
```

### 2. 依存関係のエラー

```bash
# Composerキャッシュをクリアして再インストール
docker-compose run --rm test composer clear-cache
docker-compose run --rm test composer install
```

### 3. テストが失敗する

```bash
# テストコンテナに入って調査
docker-compose run --rm test bash

# コンテナ内で
php artisan config:clear
php artisan cache:clear
php artisan test
```

### 4. ポートがすでに使用されている

```bash
# 既存のコンテナを停止
docker-compose down

# または、docker-compose.yml のポート番号を変更
```

### 5. Dockerリソースが不足

```bash
# 不要なDockerリソースをクリーンアップ
./docker-test.sh clean

# または
docker system prune -a --volumes
```

---

## 📂 ファイル構成

### Docker関連ファイル

```
nemielu-dev/
├── Dockerfile              # PHPアプリケーション用イメージ定義
├── docker-compose.yml      # サービス構成定義
├── docker-test.sh          # 便利なテスト実行スクリプト
├── .env                    # 本番/開発環境設定
├── .env.testing            # テスト環境設定
└── phpunit.xml             # PHPUnit設定
```

### 設定ファイルの役割

#### `Dockerfile`
- PHP 8.0-fpm ベース
- 必要なPHP拡張をインストール
- SQLite サポート
- Composer インストール済み

#### `docker-compose.yml`
- `app`: 開発サーバー用
- `test`: テスト実行専用（プロファイル: test）
- `db`: MySQL 8.0

#### `.env.testing`
- テスト専用の環境変数
- SQLiteメモリDB設定
- メール配列ドライバー

---

## 🎯 ベストプラクティス

### 1. テストの実行頻度

```bash
# 開発中は頻繁に実行
./docker-test.sh unit  # 高速

# コミット前には全テストを実行
./docker-test.sh
```

### 2. CI/CD統合

GitHub Actionsなどで自動実行する場合：

```yaml
- name: Run Tests
  run: |
    docker-compose build app
    docker-compose run --rm test composer install
    docker-compose run --rm test php artisan test
```

### 3. ローカル開発との使い分け

**Dockerを使う場合:**
- ✅ 環境の一貫性
- ✅ 本番環境に近い
- ✅ クリーンな環境

**ローカル実行の場合:**
- ✅ 高速
- ✅ デバッグしやすい
- ⚠️ 環境依存のリスク

---

## 📈 パフォーマンス

### 実行時間の目安

| テストスイート | テスト数 | 実行時間 |
|---|---|---|
| 全テスト | 89 | 1.47s |
| ユニットテスト | 27 | 0.42s |
| 機能テスト | 62 | 1.21s |

### 高速化のヒント

1. **ユニットテストから実行**
   ```bash
   ./docker-test.sh unit  # 0.42秒で完了
   ```

2. **並列実行**（Laravel 8以降）
   ```bash
   docker-compose run --rm test php artisan test --parallel
   ```

3. **特定のテストのみ**
   ```bash
   ./docker-test.sh filter test_specific_feature
   ```

---

## 🔐 セキュリティ

### 環境変数の管理

- `.env` ファイルはGitにコミットしない（`.gitignore`に追加済み）
- `.env.testing` には本番の認証情報を含めない
- Dockerコンテナは本番データベースに接続しない

### テストデータ

- テストはSQLiteメモリDBを使用
- 本番データベースには影響しない
- 各テスト後に自動的にロールバック

---

## 📚 参考リソース

### 公式ドキュメント

- [Laravel Testing](https://laravel.com/docs/8.x/testing)
- [PHPUnit](https://phpunit.de/documentation.html)
- [Docker Compose](https://docs.docker.com/compose/)

### プロジェクト内ドキュメント

- `tests/README.md` - テスト全般のガイド
- `TEST_COMPLETION_REPORT.md` - テスト整備報告書
- `run-tests.sh` - ローカル実行スクリプト

---

## ✅ チェックリスト

### 初回セットアップ

- [ ] Dockerがインストールされている
- [ ] `docker-compose`が使用可能
- [ ] `./docker-test.sh build` でイメージをビルド
- [ ] `./docker-test.sh install` で依存関係をインストール
- [ ] `./docker-test.sh` でテストが実行できる

### 日常的な使用

- [ ] コード変更後にテストを実行
- [ ] 新機能追加時にテストを追加
- [ ] コミット前に全テストを実行
- [ ] 定期的に `./docker-test.sh update` で依存関係を更新

---

## 🎉 まとめ

Docker環境を使用することで、以下のメリットがあります：

1. ✅ **環境の一貫性** - 全員が同じ環境でテスト
2. ✅ **クリーンな環境** - ローカル環境を汚さない
3. ✅ **簡単なセットアップ** - `./docker-test.sh build` だけ
4. ✅ **高速なテスト実行** - SQLiteメモリDB使用
5. ✅ **本番に近い環境** - Dockerコンテナで実行

---

**質問や問題がある場合は、`./docker-test.sh help` を確認してください！**
