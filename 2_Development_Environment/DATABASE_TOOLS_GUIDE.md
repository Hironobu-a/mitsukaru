# データベース管理ツールガイド

このドキュメントでは、Nemieluプロジェクトのデータベースを管理・確認するためのツールについて説明します。

---

## 🗄️ 利用可能なツール

Docker環境で2つのデータベース管理ツールを利用できます：

| ツール | URL | 特徴 |
|---|---|---|
| **phpMyAdmin** | http://localhost:8080 | 多機能、使いやすい、人気のツール |
| **Adminer** | http://localhost:8081 | 軽量、シンプル、高速 |

両方とも同じMySQLデータベースに接続しているので、お好みの方を使ってください。

---

## 🚀 クイックスタート

### 1. ツールを起動

```bash
# phpMyAdminとAdminerを起動
docker-compose up -d phpmyadmin adminer

# または、docker-start.sh を使用すると自動的に起動します
./docker-start.sh
```

### 2. ブラウザでアクセス

#### phpMyAdminの場合

1. ブラウザで **http://localhost:8080** を開く
2. 自動的にログイン画面が表示される
3. 以下の情報で **既にログイン済み** の場合があります

#### Adminerの場合

1. ブラウザで **http://localhost:8081** を開く
2. ログイン情報を入力：
   - **システム**: MySQL
   - **サーバ**: `db`
   - **ユーザ名**: `laravel`
   - **パスワード**: `secret`
   - **データベース**: `laravel`
3. 「ログイン」をクリック

---

## 📊 phpMyAdmin の使い方

### 接続情報

phpMyAdminは自動的に接続されます。もし手動でログインする必要がある場合：

- **サーバ**: `db`
- **ユーザ名**: `laravel`
- **パスワード**: `secret`

### 基本的な使い方

#### 1. データベースの選択

左サイドバーから `laravel` データベースを選択

#### 2. テーブルの表示

- 左サイドバーのテーブル一覧からテーブルを選択
- 「表示」タブでデータを確認

#### 3. データの検索

- テーブルを選択
- 「検索」タブで条件を指定

#### 4. データの編集

- テーブルを選択
- 「表示」タブで行の「編集」をクリック
- 値を変更して「実行」

#### 5. SQLクエリの実行

- 「SQL」タブを選択
- クエリを入力して「実行」

```sql
-- ユーザー一覧を表示
SELECT * FROM users;

-- 最新の10件のPVTレポートを表示
SELECT * FROM pvt_reports ORDER BY created_at DESC LIMIT 10;

-- 特定のユーザーのレポート数をカウント
SELECT user_id, COUNT(*) as report_count 
FROM reports 
GROUP BY user_id;
```

#### 6. データのエクスポート

- テーブルまたはデータベースを選択
- 「エクスポート」タブ
- フォーマットを選択（SQL、CSV、JSON等）
- 「実行」

#### 7. データのインポート

- 「インポート」タブ
- ファイルを選択
- 「実行」

---

## ⚡ Adminer の使い方

### 接続情報

- **システム**: MySQL
- **サーバ**: `db`
- **ユーザ名**: `laravel`
- **パスワード**: `secret`
- **データベース**: `laravel`

### 基本的な使い方

#### 1. データベースの選択

トップページから `laravel` を選択

#### 2. テーブルの表示

- テーブル一覧からテーブル名をクリック
- データが表示される

#### 3. データの編集

- テーブルを表示
- 行の「edit」リンクをクリック
- 値を変更して「Save」

#### 4. SQLクエリの実行

- 左メニューの「SQL command」をクリック
- クエリを入力して「Execute」

#### 5. データのエクスポート

- 左メニューの「Export」をクリック
- オプションを選択して「Export」

---

## 🎯 よく使う操作

### ユーザー管理

#### ユーザー一覧を表示

```sql
SELECT id, name, email, client_code, created_at 
FROM users 
ORDER BY created_at DESC;
```

#### 管理者一覧を表示

```sql
SELECT id, name, email, client_code, created_at 
FROM admins 
ORDER BY created_at DESC;
```

#### 特定のユーザーを検索

```sql
SELECT * FROM users WHERE email LIKE '%example%';
```

### レポート管理

#### 最新のPVTレポートを表示

```sql
SELECT 
    pr.id,
    u.name as user_name,
    pr.average,
    pr.sleepiness,
    pr.created_at
FROM pvt_reports pr
LEFT JOIN users u ON pr.user_id = u.id
ORDER BY pr.created_at DESC
LIMIT 20;
```

#### ユーザーごとのレポート数

```sql
SELECT 
    u.name,
    COUNT(r.id) as report_count
FROM users u
LEFT JOIN reports r ON u.id = r.user_id
GROUP BY u.id, u.name
ORDER BY report_count DESC;
```

### テーブル構造の確認

#### テーブル一覧を表示

```sql
SHOW TABLES;
```

#### テーブルの構造を表示

```sql
DESCRIBE users;
-- または
SHOW CREATE TABLE users;
```

#### インデックスを表示

```sql
SHOW INDEX FROM users;
```

---

## 📱 各ツールの特徴比較

### phpMyAdmin

**メリット:**
- ✅ 多機能で高度な操作が可能
- ✅ 日本語対応
- ✅ UIが直感的
- ✅ データのビジュアル表示
- ✅ リレーションの表示
- ✅ 大規模なインポート/エクスポート

**デメリット:**
- ⚠️ やや重い
- ⚠️ メモリ使用量が多い

**おすすめの用途:**
- 複雑なクエリの実行
- 大量のデータのエクスポート
- データベース構造の変更
- 初心者向け

### Adminer

**メリット:**
- ✅ 軽量で高速
- ✅ シンプルなUI
- ✅ 1つのPHPファイル
- ✅ 複数のDBシステムに対応

**デメリット:**
- ⚠️ 機能が限定的
- ⚠️ UIがシンプル

**おすすめの用途:**
- 素早いデータ確認
- 簡単な編集
- 軽い動作が必要な場合
- 上級者向け

---

## 🔧 高度な使い方

### バックアップとリストア

#### phpMyAdminでバックアップ

1. データベース `laravel` を選択
2. 「エクスポート」タブ
3. 「簡易」または「詳細」を選択
4. フォーマット: SQL
5. 「実行」

#### コマンドラインでバックアップ

```bash
# バックアップを作成
docker-compose exec db mysqldump -ularavel -psecret laravel > backup_$(date +%Y%m%d_%H%M%S).sql

# リストア
docker-compose exec -T db mysql -ularavel -psecret laravel < backup_20260210_000000.sql
```

### データの一括更新

```sql
-- 全ユーザーの特定フィールドを更新
UPDATE users SET first_flg = 0 WHERE first_flg IS NULL;

-- 条件付き更新
UPDATE reports SET sleepiness = 5 WHERE sleepiness > 5;
```

### パフォーマンスチューニング

#### インデックスの追加

```sql
-- 頻繁に検索されるカラムにインデックスを追加
CREATE INDEX idx_user_email ON users(email);
CREATE INDEX idx_report_user_id ON reports(user_id);
```

#### スロークエリの確認

```sql
-- 実行計画を表示
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';
```

---

## 🛡️ セキュリティとベストプラクティス

### 本番環境での注意

⚠️ **重要**: これらのツールは開発環境専用です。本番環境では以下の対策が必要：

1. **アクセス制限**
   - IPアドレス制限
   - VPN経由のみアクセス可能に

2. **認証強化**
   - 強力なパスワード
   - 2要素認証

3. **無効化**
   - 本番環境ではphpMyAdmin/Adminerを無効化
   - 必要な時だけ一時的に有効化

### 開発環境でのベストプラクティス

1. **本番データを使わない**
   - テストデータのみを使用
   - 個人情報は含めない

2. **定期的なバックアップ**
   - 重要な変更前にバックアップ
   - 自動バックアップスクリプトの活用

3. **クエリのテスト**
   - SELECT文で動作確認してからUPDATE/DELETE
   - トランザクションの活用

```sql
-- トランザクションの例
START TRANSACTION;
UPDATE users SET name = 'テスト' WHERE id = 1;
-- 確認
SELECT * FROM users WHERE id = 1;
-- 問題なければコミット
COMMIT;
-- 問題があればロールバック
-- ROLLBACK;
```

---

## 🔍 トラブルシューティング

### phpMyAdminにアクセスできない

```bash
# コンテナの状態を確認
docker-compose ps

# ログを確認
docker-compose logs phpmyadmin

# 再起動
docker-compose restart phpmyadmin
```

### Adminerにアクセスできない

```bash
# コンテナの状態を確認
docker-compose ps

# ログを確認
docker-compose logs adminer

# 再起動
docker-compose restart adminer
```

### データベースに接続できない

```bash
# データベースが起動しているか確認
docker-compose ps db

# データベースのログを確認
docker-compose logs db

# データベースを再起動
docker-compose restart db

# 接続情報を確認
# サーバ: db (localhost ではない)
# ユーザー: laravel
# パスワード: secret
# データベース: laravel
```

### ポートが使用中

```bash
# ポート8080が使用中の場合
# docker-compose.yml を編集
ports:
  - "8082:80"  # 8080 -> 8082 に変更

# ポート8081が使用中の場合
ports:
  - "8083:8080"  # 8081 -> 8083 に変更
```

---

## 📚 よく使うSQLクエリ集

### テーブル情報

```sql
-- 全テーブルのレコード数
SELECT 
    table_name,
    table_rows
FROM information_schema.tables
WHERE table_schema = 'laravel'
ORDER BY table_rows DESC;

-- テーブルサイズ
SELECT 
    table_name,
    ROUND(((data_length + index_length) / 1024 / 1024), 2) AS size_mb
FROM information_schema.tables
WHERE table_schema = 'laravel'
ORDER BY size_mb DESC;
```

### データ分析

```sql
-- ユーザー登録数の推移（月別）
SELECT 
    DATE_FORMAT(created_at, '%Y-%m') as month,
    COUNT(*) as user_count
FROM users
GROUP BY month
ORDER BY month DESC;

-- 平均睡眠時間
SELECT 
    AVG(ln_sleep_time) as avg_sleep_hours,
    MIN(ln_sleep_time) as min_sleep_hours,
    MAX(ln_sleep_time) as max_sleep_hours
FROM pvt_reports;

-- ユーザーごとのテスト実施回数
SELECT 
    u.name,
    COUNT(pr.id) as pvt_count
FROM users u
LEFT JOIN pvt_reports pr ON u.id = pr.user_id
GROUP BY u.id, u.name
HAVING pvt_count > 0
ORDER BY pvt_count DESC;
```

---

## 🎨 UI カスタマイズ

### phpMyAdmin

設定ファイルで外観をカスタマイズできます（高度な使用者向け）。

### Adminer

テーマを変更できます：
- ログイン画面の下部にテーマ選択あり

---

## 🚀 その他のツール

Docker環境以外で使えるデータベースツール：

### デスクトップアプリ

| ツール | OS | 特徴 |
|---|---|---|
| **TablePlus** | Mac, Windows, Linux | モダンなUI、高速 |
| **DBeaver** | Mac, Windows, Linux | 無料、多機能 |
| **Sequel Pro** | Mac | シンプル、軽量 |
| **HeidiSQL** | Windows | 無料、人気 |
| **DataGrip** | Mac, Windows, Linux | JetBrains製、有料 |

### VSCode拡張機能

- **MySQL** by Jun Han
- **Database Client** by Weijan Chen

### 接続情報（デスクトップツール用）

```
ホスト: 127.0.0.1 または localhost
ポート: 3306
ユーザー: laravel
パスワード: secret
データベース: laravel
```

---

## ✅ チェックリスト

### 日常的な使用

- [ ] データベースツールを起動
- [ ] ブラウザでアクセス
- [ ] データを確認・編集
- [ ] 変更をコミット前にテスト

### データの変更前

- [ ] バックアップを作成
- [ ] SELECT文で対象データを確認
- [ ] トランザクションを使用
- [ ] 変更後にアプリケーションで動作確認

---

## 🎉 まとめ

データベース管理ツールが利用可能になりました：

| ツール | URL | 用途 |
|---|---|---|
| **phpMyAdmin** | http://localhost:8080 | 多機能、初心者向け |
| **Adminer** | http://localhost:8081 | 軽量、上級者向け |

どちらも同じデータベースに接続しているので、お好みの方をご利用ください！

---

**質問や問題がある場合は、ログを確認してください:**

```bash
docker-compose logs phpmyadmin
docker-compose logs adminer
```

Happy Database Management! 🗄️
