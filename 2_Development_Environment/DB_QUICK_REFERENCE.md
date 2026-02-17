# データベース管理 - クイックリファレンス

このドキュメントは、データベース管理ツールの使い方を素早く確認するためのリファレンスです。

---

## 🚀 アクセス方法

| ツール | URL | ログイン情報 |
|---|---|---|
| **phpMyAdmin** | http://localhost:8000 | 自動ログイン済み |
| **Adminer** | http://localhost:8081 | ユーザー: `laravel`<br>パスワード: `secret` |

---

## 📊 データベース構成

### 主要テーブル

| テーブル名 | 説明 |
|---|---|
| `users` | 一般ユーザー |
| `admins` | 管理者 |
| `pvt_reports` | PVTテスト結果 |
| `pvt_reaction_times` | PVT反応時間データ |
| `reports` | テストレポート |
| `user_datas` | ユーザー詳細データ |
| `clients` | クライアント情報 |

### 接続情報

```
ホスト: db (Docker内) または localhost (外部)
ポート: 3306
ユーザー: laravel
パスワード: secret
データベース: laravel
```

---

## ⚡ よく使うSQLクエリ

### ユーザー管理

```sql
-- ユーザー一覧
SELECT id, name, email, client_code FROM users;

-- 管理者一覧
SELECT id, name, email FROM admins;

-- 特定ユーザーの検索
SELECT * FROM users WHERE email = 'user@example.com';

-- 新しいユーザーを追加
INSERT INTO users (name, email, password, client_code) 
VALUES ('テストユーザー', 'test@example.com', '$2y$10$...', 'TEST001');
```

### レポート管理

```sql
-- 最新のPVTレポート
SELECT * FROM pvt_reports ORDER BY created_at DESC LIMIT 10;

-- ユーザー別レポート数
SELECT u.name, COUNT(pr.id) as count
FROM users u
LEFT JOIN pvt_reports pr ON u.id = pr.user_id
GROUP BY u.id;

-- 平均睡眠時間
SELECT AVG(ln_sleep_time) FROM pvt_reports;
```

### データ編集

```sql
-- ユーザー情報を更新
UPDATE users SET name = '新しい名前' WHERE id = 1;

-- データを削除
DELETE FROM users WHERE id = 999;

-- パスワードをリセット
UPDATE users SET password = '$2y$10$...' WHERE email = 'user@example.com';
```

---

## 🔍 データの確認

### テーブル構造

```sql
-- テーブル一覧
SHOW TABLES;

-- テーブル構造
DESCRIBE users;

-- インデックス確認
SHOW INDEX FROM users;
```

### データ統計

```sql
-- レコード数
SELECT COUNT(*) FROM users;

-- 日別ユーザー登録数
SELECT DATE(created_at) as date, COUNT(*) as count
FROM users
GROUP BY date
ORDER BY date DESC;
```

---

## 🛠️ 便利なコマンド

### phpMyAdmin

| 操作 | 手順 |
|---|---|
| テーブル表示 | 左メニュー → テーブル名 → 「表示」タブ |
| データ検索 | テーブル選択 → 「検索」タブ |
| SQLクエリ | 「SQL」タブ → クエリ入力 → 「実行」 |
| エクスポート | テーブル選択 → 「エクスポート」タブ |
| インポート | 「インポート」タブ → ファイル選択 |

### Adminer

| 操作 | 手順 |
|---|---|
| テーブル表示 | テーブル名をクリック |
| データ編集 | 行の「edit」リンク → 編集 → 「Save」 |
| SQLクエリ | 「SQL command」→ クエリ入力 → 「Execute」 |
| エクスポート | 「Export」→ オプション選択 |

---

## 🔧 Docker コマンド

### サービス管理

```bash
# データベース管理ツールを起動
docker-compose up -d phpmyadmin adminer

# 停止
docker-compose stop phpmyadmin adminer

# 再起動
docker-compose restart phpmyadmin adminer

# ログ確認
docker-compose logs phpmyadmin
docker-compose logs adminer
```

### データベース直接操作

```bash
# MySQLクライアントで接続
docker-compose exec db mysql -ularavel -psecret laravel

# バックアップ
docker-compose exec db mysqldump -ularavel -psecret laravel > backup.sql

# リストア
docker-compose exec -T db mysql -ularavel -psecret laravel < backup.sql
```

---

## 🎯 トラブルシューティング

| 問題 | 解決方法 |
|---|---|
| アクセスできない | `docker-compose ps` で状態確認 |
| ログインできない | サーバ: `db`, ユーザー: `laravel`, パスワード: `secret` |
| データが表示されない | データベース `laravel` を選択 |
| 動作が遅い | `docker-compose restart phpmyadmin` |

---

## 📚 関連ドキュメント

- **DATABASE_TOOLS_GUIDE.md** - 詳細ガイド
- **DEVELOPMENT_GUIDE.md** - 開発環境ガイド
- **GETTING_STARTED.md** - はじめに

---

## ⚠️ 重要な注意事項

1. **バックアップ**: 重要なデータ変更前に必ずバックアップ
2. **確認**: UPDATE/DELETE前にSELECTで確認
3. **トランザクション**: 大きな変更はトランザクションを使用
4. **本番環境**: これらのツールは開発専用

---

## 🎨 おすすめのワークフロー

### データ確認

1. phpMyAdmin または Adminer にアクセス
2. テーブルを選択してデータを表示
3. 必要に応じてフィルタや検索を使用

### データ編集

1. まずSELECTでデータを確認
2. バックアップを作成（重要な場合）
3. UPDATEまたはDELETEを実行
4. 再度SELECTで結果を確認
5. アプリケーションで動作確認

### トラブル調査

1. ログでエラーメッセージを確認
2. SQLクエリでデータの状態を調査
3. テーブル構造やインデックスを確認
4. 必要に応じてデータを修正

---

## 💡 ヒント

- **SQLエディタ**: 複雑なクエリはVSCodeなどで書いてからコピペ
- **履歴**: phpMyAdminはクエリ履歴を保存
- **テーマ**: Adminerはテーマ変更可能
- **ショートカット**: Ctrl+Enter でクエリ実行（phpMyAdmin）

---

**詳細は DATABASE_TOOLS_GUIDE.md を参照してください！**
