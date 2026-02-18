# 詳細データベース設計書

**作成日**: 2026年2月18日
**バージョン**: 1.0

---

## 1. ER図（Mermaid）

```mermaid
erDiagram
    %% ===== マスター系 =====
    professions {
        bigint id PK
        varchar name
        varchar slug UK
        varchar icon
        text description
        int sort_order
        timestamp created_at
        timestamp updated_at
    }

    prefectures {
        bigint id PK
        varchar name
        varchar region
        int sort_order
        timestamp created_at
        timestamp updated_at
    }

    employment_types {
        bigint id PK
        varchar name
        int sort_order
        timestamp created_at
        timestamp updated_at
    }

    salary_types {
        bigint id PK
        varchar name
        int sort_order
        timestamp created_at
        timestamp updated_at
    }

    skills {
        bigint id PK
        varchar name
        bigint profession_id FK
        int sort_order
        timestamp created_at
        timestamp updated_at
    }

    licenses {
        bigint id PK
        varchar name
        bigint profession_id FK
        int sort_order
        timestamp created_at
        timestamp updated_at
    }

    %% ===== ユーザー系 =====
    users {
        bigint id PK
        varchar email UK
        varchar password
        varchar last_name
        varchar first_name
        varchar last_name_kana
        varchar first_name_kana
        date birthday
        tinyint gender
        bigint prefecture_id FK
        tinyint status
        varchar avatar_path
        timestamp email_verified_at
        varchar remember_token
        timestamp deleted_at
        timestamp created_at
        timestamp updated_at
    }

    user_profiles {
        bigint id PK
        bigint user_id FK-UK
        bigint profession_id FK
        text self_introduction
        int experience_years
        int desired_salary_min
        int desired_salary_max
        bigint desired_prefecture_id FK
        text career_history
        tinyint job_search_status
        timestamp created_at
        timestamp updated_at
    }

    user_skills {
        bigint id PK
        bigint user_id FK
        bigint skill_id FK
        timestamp created_at
    }

    user_licenses {
        bigint id PK
        bigint user_id FK
        bigint license_id FK
        date acquired_at
        timestamp created_at
    }

    %% ===== 企業系 =====
    companies {
        bigint id PK
        varchar name
        varchar name_kana
        bigint prefecture_id FK
        varchar address
        varchar phone
        varchar email UK
        varchar website
        tinyint status
        tinyint is_verified
        timestamp verified_at
        timestamp deleted_at
        timestamp created_at
        timestamp updated_at
    }

    company_profiles {
        bigint id PK
        bigint company_id FK-UK
        text description
        varchar industry
        varchar employee_count
        varchar founded_year
        varchar capital
        text benefits
        text culture
        varchar logo_path
        varchar cover_image_path
        timestamp created_at
        timestamp updated_at
    }

    company_users {
        bigint id PK
        bigint company_id FK
        varchar email UK
        varchar password
        varchar name
        varchar role
        varchar remember_token
        timestamp deleted_at
        timestamp created_at
        timestamp updated_at
    }

    %% ===== 求人系 =====
    jobs {
        bigint id PK
        bigint company_id FK
        varchar title
        text description
        text requirement
        text welcome_skill
        bigint employment_type_id FK
        bigint salary_type_id FK
        int salary_min
        int salary_max
        bigint prefecture_id FK
        varchar work_location
        varchar nearest_station
        varchar work_hours
        text benefits
        text selection_process
        tinyint remote_type
        tinyint status
        date published_at
        date closed_at
        int view_count
        timestamp deleted_at
        timestamp created_at
        timestamp updated_at
    }

    job_professions {
        bigint id PK
        bigint job_id FK
        bigint profession_id FK
    }

    job_skills {
        bigint id PK
        bigint job_id FK
        bigint skill_id FK
    }

    job_favorites {
        bigint id PK
        bigint user_id FK
        bigint job_id FK
        timestamp created_at
    }

    job_applications {
        bigint id PK
        bigint user_id FK
        bigint job_id FK
        tinyint status
        text message
        text company_memo
        timestamp applied_at
        timestamp created_at
        timestamp updated_at
    }

    application_status_histories {
        bigint id PK
        bigint application_id FK
        tinyint from_status
        tinyint to_status
        bigint changed_by_id
        varchar changed_by_type
        text note
        timestamp created_at
    }

    %% ===== コンテンツ系 =====
    categories {
        bigint id PK
        varchar name
        varchar slug UK
        int sort_order
        timestamp created_at
        timestamp updated_at
    }

    tags {
        bigint id PK
        varchar name
        varchar slug UK
        timestamp created_at
        timestamp updated_at
    }

    articles {
        bigint id PK
        varchar title
        varchar slug UK
        text body
        varchar meta_description
        varchar featured_image_path
        bigint category_id FK
        bigint profession_id FK
        bigint author_id FK
        tinyint status
        timestamp published_at
        int view_count
        timestamp created_at
        timestamp updated_at
    }

    article_tags {
        bigint id PK
        bigint article_id FK
        bigint tag_id FK
    }

    %% ===== 管理者系 =====
    admin_users {
        bigint id PK
        varchar name
        varchar email UK
        varchar password
        tinyint role
        varchar remember_token
        timestamp created_at
        timestamp updated_at
    }

    %% ===== 通知系 =====
    notifications {
        char id PK
        varchar type
        varchar notifiable_type
        bigint notifiable_id
        text data
        timestamp read_at
        timestamp created_at
        timestamp updated_at
    }

    %% ===== リレーション =====
    users ||--o| user_profiles : "has one"
    users ||--o{ user_skills : "has many"
    users ||--o{ user_licenses : "has many"
    users ||--o{ job_favorites : "has many"
    users ||--o{ job_applications : "has many"
    users }o--o| prefectures : "belongs to"

    user_profiles }o--o| professions : "belongs to"
    user_profiles }o--o| prefectures : "desired"
    user_skills }o--|| skills : "belongs to"
    user_licenses }o--|| licenses : "belongs to"

    companies ||--o| company_profiles : "has one"
    companies ||--o{ company_users : "has many"
    companies ||--o{ jobs : "has many"
    companies }o--o| prefectures : "belongs to"

    jobs }o--|| companies : "belongs to"
    jobs }o--|| employment_types : "belongs to"
    jobs }o--|| salary_types : "belongs to"
    jobs }o--o| prefectures : "belongs to"
    jobs ||--o{ job_professions : "has many"
    jobs ||--o{ job_skills : "has many"
    jobs ||--o{ job_favorites : "has many"
    jobs ||--o{ job_applications : "has many"

    job_professions }o--|| professions : "belongs to"
    job_skills }o--|| skills : "belongs to"

    job_applications ||--o{ application_status_histories : "has many"

    skills }o--o| professions : "belongs to"
    licenses }o--o| professions : "belongs to"

    articles }o--o| categories : "belongs to"
    articles }o--o| professions : "belongs to"
    articles }o--o| admin_users : "authored by"
    articles ||--o{ article_tags : "has many"
    article_tags }o--|| tags : "belongs to"
```

---

## 2. 全テーブル詳細定義

### 2.1 マスター系テーブル

#### `professions`（士業種別マスター）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO_INCREMENT | PK |
| name | VARCHAR(50) | NO | - | 士業名（税理士, 弁護士等） |
| slug | VARCHAR(50) | NO | - | URL用スラッグ（tax-accountant, lawyer等）UK |
| icon | VARCHAR(100) | YES | NULL | アイコンクラス名 |
| description | TEXT | YES | NULL | 士業の説明文 |
| sort_order | INT | NO | 0 | 表示順 |
| created_at | TIMESTAMP | YES | NULL | |
| updated_at | TIMESTAMP | YES | NULL | |

**初期データ**:
```
税理士 (tax-accountant), 公認会計士 (certified-accountant),
弁護士 (lawyer), 司法書士 (judicial-scrivener),
社会保険労務士 (labor-consultant), 行政書士 (administrative-scrivener),
弁理士 (patent-attorney), 土地家屋調査士 (land-surveyor),
中小企業診断士 (sme-consultant), 不動産鑑定士 (real-estate-appraiser)
```

#### `prefectures`（都道府県マスター）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO_INCREMENT | PK |
| name | VARCHAR(10) | NO | - | 都道府県名 |
| region | VARCHAR(10) | NO | - | 地方名（北海道, 東北, 関東等） |
| sort_order | INT | NO | 0 | JISコード順 |
| created_at | TIMESTAMP | YES | NULL | |
| updated_at | TIMESTAMP | YES | NULL | |

#### `employment_types`（雇用形態マスター）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO_INCREMENT | PK |
| name | VARCHAR(30) | NO | - | 雇用形態名 |
| sort_order | INT | NO | 0 | 表示順 |
| created_at | TIMESTAMP | YES | NULL | |
| updated_at | TIMESTAMP | YES | NULL | |

**初期データ**: 正社員, 契約社員, パート・アルバイト, 業務委託, 派遣社員

#### `salary_types`（給与形態マスター）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO_INCREMENT | PK |
| name | VARCHAR(20) | NO | - | 給与形態名 |
| sort_order | INT | NO | 0 | 表示順 |
| created_at | TIMESTAMP | YES | NULL | |
| updated_at | TIMESTAMP | YES | NULL | |

**初期データ**: 月給, 年俸, 時給, 日給

#### `skills`（スキルマスター）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO_INCREMENT | PK |
| name | VARCHAR(100) | NO | - | スキル名 |
| profession_id | BIGINT UNSIGNED | YES | NULL | 関連士業（NULLは共通スキル）FK |
| sort_order | INT | NO | 0 | 表示順 |
| created_at | TIMESTAMP | YES | NULL | |
| updated_at | TIMESTAMP | YES | NULL | |

**INDEX**: `(profession_id)`

#### `licenses`（資格マスター）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO_INCREMENT | PK |
| name | VARCHAR(100) | NO | - | 資格名 |
| profession_id | BIGINT UNSIGNED | YES | NULL | 関連士業 FK |
| sort_order | INT | NO | 0 | 表示順 |
| created_at | TIMESTAMP | YES | NULL | |
| updated_at | TIMESTAMP | YES | NULL | |

---

### 2.2 ユーザー系テーブル

#### `users`（求職者）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO_INCREMENT | PK |
| email | VARCHAR(255) | NO | - | メールアドレス UK |
| password | VARCHAR(255) | NO | - | ハッシュ化パスワード |
| last_name | VARCHAR(50) | NO | - | 姓 |
| first_name | VARCHAR(50) | NO | - | 名 |
| last_name_kana | VARCHAR(50) | YES | NULL | 姓（カナ） |
| first_name_kana | VARCHAR(50) | YES | NULL | 名（カナ） |
| birthday | DATE | YES | NULL | 生年月日 |
| gender | TINYINT | YES | NULL | 0:未回答 1:男性 2:女性 3:その他 |
| prefecture_id | BIGINT UNSIGNED | YES | NULL | 居住都道府県 FK |
| status | TINYINT | NO | 1 | 1:active 2:pause 9:withdrawn |
| avatar_path | VARCHAR(500) | YES | NULL | プロフィール画像パス |
| email_verified_at | TIMESTAMP | YES | NULL | メール認証日時 |
| remember_token | VARCHAR(100) | YES | NULL | |
| deleted_at | TIMESTAMP | YES | NULL | ソフトデリート |
| created_at | TIMESTAMP | YES | NULL | |
| updated_at | TIMESTAMP | YES | NULL | |

**INDEX**: `(status)`, `(prefecture_id)`, `(email_verified_at)`

#### `user_profiles`（求職者プロフィール）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO_INCREMENT | PK |
| user_id | BIGINT UNSIGNED | NO | - | FK UK (1:1) |
| profession_id | BIGINT UNSIGNED | YES | NULL | 主たる士業 FK |
| self_introduction | TEXT | YES | NULL | 自己紹介文 |
| experience_years | INT | YES | NULL | 実務経験年数 |
| desired_salary_min | INT UNSIGNED | YES | NULL | 希望年収下限 |
| desired_salary_max | INT UNSIGNED | YES | NULL | 希望年収上限 |
| desired_prefecture_id | BIGINT UNSIGNED | YES | NULL | 希望勤務地 FK |
| career_history | TEXT | YES | NULL | 職務経歴（JSON or テキスト） |
| job_search_status | TINYINT | NO | 1 | 1:積極的 2:良い案件あれば 3:情報収集中 |
| created_at | TIMESTAMP | YES | NULL | |
| updated_at | TIMESTAMP | YES | NULL | |

#### `user_skills`（求職者スキル中間テーブル）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO_INCREMENT | PK |
| user_id | BIGINT UNSIGNED | NO | - | FK |
| skill_id | BIGINT UNSIGNED | NO | - | FK |
| created_at | TIMESTAMP | YES | NULL | |

**UNIQUE**: `(user_id, skill_id)`

#### `user_licenses`（保有資格中間テーブル）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO_INCREMENT | PK |
| user_id | BIGINT UNSIGNED | NO | - | FK |
| license_id | BIGINT UNSIGNED | NO | - | FK |
| acquired_at | DATE | YES | NULL | 取得年月日 |
| created_at | TIMESTAMP | YES | NULL | |

**UNIQUE**: `(user_id, license_id)`

---

### 2.3 企業系テーブル

#### `companies`（採用企業・事務所）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO_INCREMENT | PK |
| name | VARCHAR(200) | NO | - | 企業名・事務所名 |
| name_kana | VARCHAR(200) | YES | NULL | 企業名（カナ） |
| prefecture_id | BIGINT UNSIGNED | YES | NULL | 所在地都道府県 FK |
| address | VARCHAR(500) | YES | NULL | 住所 |
| phone | VARCHAR(20) | YES | NULL | 電話番号 |
| email | VARCHAR(255) | NO | - | 代表メールアドレス UK |
| website | VARCHAR(500) | YES | NULL | 企業サイトURL |
| status | TINYINT | NO | 1 | 1:審査中 2:掲載中 9:停止 |
| is_verified | TINYINT | NO | 0 | 審査通過フラグ |
| verified_at | TIMESTAMP | YES | NULL | 審査通過日時 |
| deleted_at | TIMESTAMP | YES | NULL | ソフトデリート |
| created_at | TIMESTAMP | YES | NULL | |
| updated_at | TIMESTAMP | YES | NULL | |

**INDEX**: `(status, is_verified)`, `(prefecture_id)`

#### `company_profiles`（企業プロフィール）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO_INCREMENT | PK |
| company_id | BIGINT UNSIGNED | NO | - | FK UK (1:1) |
| description | TEXT | YES | NULL | 企業紹介文 |
| industry | VARCHAR(100) | YES | NULL | 業種 |
| employee_count | VARCHAR(50) | YES | NULL | 従業員数（1-10名, 11-50名等） |
| founded_year | VARCHAR(10) | YES | NULL | 設立年 |
| capital | VARCHAR(50) | YES | NULL | 資本金 |
| benefits | TEXT | YES | NULL | 福利厚生（JSON） |
| culture | TEXT | YES | NULL | 社風・カルチャー |
| logo_path | VARCHAR(500) | YES | NULL | ロゴ画像パス |
| cover_image_path | VARCHAR(500) | YES | NULL | カバー画像パス |
| created_at | TIMESTAMP | YES | NULL | |
| updated_at | TIMESTAMP | YES | NULL | |

#### `company_users`（企業担当者アカウント）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO_INCREMENT | PK |
| company_id | BIGINT UNSIGNED | NO | - | FK |
| email | VARCHAR(255) | NO | - | UK |
| password | VARCHAR(255) | NO | - | ハッシュ化パスワード |
| name | VARCHAR(100) | NO | - | 担当者名 |
| role | VARCHAR(20) | NO | 'member' | owner / admin / member |
| remember_token | VARCHAR(100) | YES | NULL | |
| deleted_at | TIMESTAMP | YES | NULL | ソフトデリート |
| created_at | TIMESTAMP | YES | NULL | |
| updated_at | TIMESTAMP | YES | NULL | |

**INDEX**: `(company_id)`

---

### 2.4 求人系テーブル

#### `jobs`（求人票）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO_INCREMENT | PK |
| company_id | BIGINT UNSIGNED | NO | - | FK (CASCADE DELETE) |
| title | VARCHAR(200) | NO | - | 求人タイトル |
| description | TEXT | NO | - | 仕事内容 |
| requirement | TEXT | YES | NULL | 応募要件 |
| welcome_skill | TEXT | YES | NULL | 歓迎スキル |
| employment_type_id | BIGINT UNSIGNED | NO | - | 雇用形態 FK |
| salary_type_id | BIGINT UNSIGNED | NO | - | 給与形態 FK |
| salary_min | INT UNSIGNED | YES | NULL | 給与下限 |
| salary_max | INT UNSIGNED | YES | NULL | 給与上限 |
| prefecture_id | BIGINT UNSIGNED | YES | NULL | 勤務都道府県 FK |
| work_location | VARCHAR(300) | YES | NULL | 詳細勤務地 |
| nearest_station | VARCHAR(200) | YES | NULL | 最寄り駅 |
| work_hours | VARCHAR(200) | YES | NULL | 勤務時間 |
| benefits | TEXT | YES | NULL | 待遇・福利厚生 |
| selection_process | TEXT | YES | NULL | 選考プロセス |
| remote_type | TINYINT | NO | 0 | 0:出社 1:一部リモート 2:フルリモート |
| status | TINYINT | NO | 1 | 1:下書き 2:公開 3:非公開 9:削除 |
| published_at | DATE | YES | NULL | 公開日 |
| closed_at | DATE | YES | NULL | 掲載終了日 |
| view_count | INT UNSIGNED | NO | 0 | 閲覧数 |
| deleted_at | TIMESTAMP | YES | NULL | ソフトデリート |
| created_at | TIMESTAMP | YES | NULL | |
| updated_at | TIMESTAMP | YES | NULL | |

**INDEX**:
- `(status, published_at)` - 公開求人の新着順表示
- `(company_id, status)` - 企業別求人管理
- `(prefecture_id)` - 勤務地検索
- `(salary_min, salary_max)` - 年収検索
- `(employment_type_id)` - 雇用形態フィルタ
- `(remote_type)` - リモート検索

#### `job_professions`（求人対象職種中間テーブル）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO_INCREMENT | PK |
| job_id | BIGINT UNSIGNED | NO | - | FK (CASCADE DELETE) |
| profession_id | BIGINT UNSIGNED | NO | - | FK |

**UNIQUE**: `(job_id, profession_id)`

#### `job_skills`（求人スキルタグ中間テーブル）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO_INCREMENT | PK |
| job_id | BIGINT UNSIGNED | NO | - | FK (CASCADE DELETE) |
| skill_id | BIGINT UNSIGNED | NO | - | FK |

**UNIQUE**: `(job_id, skill_id)`

#### `job_favorites`（お気に入り）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO_INCREMENT | PK |
| user_id | BIGINT UNSIGNED | NO | - | FK (CASCADE DELETE) |
| job_id | BIGINT UNSIGNED | NO | - | FK (CASCADE DELETE) |
| created_at | TIMESTAMP | YES | NULL | |

**UNIQUE**: `(user_id, job_id)`
**INDEX**: `(user_id)`

#### `job_applications`（応募）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO_INCREMENT | PK |
| user_id | BIGINT UNSIGNED | NO | - | FK (RESTRICT DELETE) |
| job_id | BIGINT UNSIGNED | NO | - | FK (RESTRICT DELETE) |
| status | TINYINT | NO | 1 | 1:応募済 2:書類選考中 3:面接調整 4:内定 5:辞退 9:不採用 |
| message | TEXT | YES | NULL | 応募メッセージ |
| company_memo | TEXT | YES | NULL | 企業側メモ（求職者非公開） |
| applied_at | TIMESTAMP | NO | CURRENT_TIMESTAMP | 応募日時 |
| created_at | TIMESTAMP | YES | NULL | |
| updated_at | TIMESTAMP | YES | NULL | |

**UNIQUE**: `(user_id, job_id)` - 二重応募防止
**INDEX**: `(user_id, status)`, `(job_id, status)`

#### `application_status_histories`（選考ステータス履歴）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO_INCREMENT | PK |
| application_id | BIGINT UNSIGNED | NO | - | FK (CASCADE DELETE) |
| from_status | TINYINT | YES | NULL | 変更前ステータス |
| to_status | TINYINT | NO | - | 変更後ステータス |
| changed_by_id | BIGINT UNSIGNED | NO | - | 変更者ID |
| changed_by_type | VARCHAR(50) | NO | - | 変更者タイプ（CompanyUser / AdminUser） |
| note | TEXT | YES | NULL | 変更理由メモ |
| created_at | TIMESTAMP | YES | NULL | |

**INDEX**: `(application_id, created_at)`

---

### 2.5 コンテンツ系テーブル

#### `categories`（記事カテゴリ）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO_INCREMENT | PK |
| name | VARCHAR(100) | NO | - | カテゴリ名 |
| slug | VARCHAR(100) | NO | - | URL用スラッグ UK |
| sort_order | INT | NO | 0 | 表示順 |
| created_at | TIMESTAMP | YES | NULL | |
| updated_at | TIMESTAMP | YES | NULL | |

#### `tags`（タグ）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO_INCREMENT | PK |
| name | VARCHAR(100) | NO | - | タグ名 |
| slug | VARCHAR(100) | NO | - | URL用スラッグ UK |
| created_at | TIMESTAMP | YES | NULL | |
| updated_at | TIMESTAMP | YES | NULL | |

#### `articles`（転職ノウハウ記事）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO_INCREMENT | PK |
| title | VARCHAR(200) | NO | - | 記事タイトル |
| slug | VARCHAR(200) | NO | - | URL用スラッグ UK |
| body | LONGTEXT | NO | - | 記事本文（Markdown or HTML） |
| meta_description | VARCHAR(300) | YES | NULL | SEO メタディスクリプション |
| featured_image_path | VARCHAR(500) | YES | NULL | アイキャッチ画像パス |
| category_id | BIGINT UNSIGNED | YES | NULL | カテゴリ FK |
| profession_id | BIGINT UNSIGNED | YES | NULL | 対象士業 FK |
| author_id | BIGINT UNSIGNED | YES | NULL | 著者（管理者）FK |
| status | TINYINT | NO | 1 | 1:下書き 2:公開 |
| published_at | TIMESTAMP | YES | NULL | 公開日時 |
| view_count | INT UNSIGNED | NO | 0 | 閲覧数 |
| created_at | TIMESTAMP | YES | NULL | |
| updated_at | TIMESTAMP | YES | NULL | |

**INDEX**: `(status, published_at)`, `(category_id)`, `(profession_id)`

#### `article_tags`（記事タグ中間テーブル）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO_INCREMENT | PK |
| article_id | BIGINT UNSIGNED | NO | - | FK (CASCADE DELETE) |
| tag_id | BIGINT UNSIGNED | NO | - | FK |

**UNIQUE**: `(article_id, tag_id)`

---

### 2.6 管理者系テーブル

#### `admin_users`（管理者）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | BIGINT UNSIGNED | NO | AUTO_INCREMENT | PK |
| name | VARCHAR(100) | NO | - | 管理者名 |
| email | VARCHAR(255) | NO | - | UK |
| password | VARCHAR(255) | NO | - | ハッシュ化パスワード |
| role | TINYINT | NO | 1 | 1:editor 2:admin 9:super_admin |
| remember_token | VARCHAR(100) | YES | NULL | |
| created_at | TIMESTAMP | YES | NULL | |
| updated_at | TIMESTAMP | YES | NULL | |

---

## 3. Enum 定義（PHP 8.1 Backed Enum）

```php
// app/Enums/UserStatus.php
enum UserStatus: int
{
    case Active = 1;
    case Paused = 2;
    case Withdrawn = 9;
}

// app/Enums/CompanyStatus.php
enum CompanyStatus: int
{
    case UnderReview = 1;
    case Active = 2;
    case Suspended = 9;
}

// app/Enums/JobStatus.php
enum JobStatus: int
{
    case Draft = 1;
    case Published = 2;
    case Unpublished = 3;
    case Deleted = 9;
}

// app/Enums/ApplicationStatus.php
enum ApplicationStatus: int
{
    case Applied = 1;
    case Screening = 2;
    case Interview = 3;
    case Offered = 4;
    case Declined = 5;
    case Rejected = 9;
}

// app/Enums/Gender.php
enum Gender: int
{
    case Unspecified = 0;
    case Male = 1;
    case Female = 2;
    case Other = 3;
}

// app/Enums/RemoteType.php
enum RemoteType: int
{
    case OnSite = 0;
    case Hybrid = 1;
    case FullRemote = 2;
}

// app/Enums/JobSearchStatus.php
enum JobSearchStatus: int
{
    case Active = 1;        // 積極的に転職活動中
    case Passive = 2;       // 良い案件があれば
    case Exploring = 3;     // 情報収集中
}

// app/Enums/AdminRole.php
enum AdminRole: int
{
    case Editor = 1;
    case Admin = 2;
    case SuperAdmin = 9;
}
```

---

## 4. Migration 実行順序

```
01 - create_professions_table
02 - create_prefectures_table
03 - create_employment_types_table
04 - create_salary_types_table
05 - create_skills_table
06 - create_licenses_table
07 - create_users_table
08 - create_user_profiles_table
09 - create_user_skills_table
10 - create_user_licenses_table
11 - create_companies_table
12 - create_company_profiles_table
13 - create_company_users_table
14 - create_jobs_table
15 - create_job_professions_table
16 - create_job_skills_table
17 - create_job_favorites_table
18 - create_job_applications_table
19 - create_application_status_histories_table
20 - create_categories_table
21 - create_tags_table
22 - create_articles_table
23 - create_article_tags_table
24 - create_admin_users_table
25 - create_notifications_table
```

---

## 5. Seeder 設計

```php
// DatabaseSeeder.php の実行順序
class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        $this->call([
            // マスターデータ（本番でも使用）
            ProfessionSeeder::class,
            PrefectureSeeder::class,
            EmploymentTypeSeeder::class,
            SalaryTypeSeeder::class,
            SkillSeeder::class,
            LicenseSeeder::class,
            CategorySeeder::class,

            // 開発用ダミーデータ（本番では実行しない）
            AdminUserSeeder::class,
            UserSeeder::class,
            CompanySeeder::class,
            JobSeeder::class,
            ApplicationSeeder::class,
            ArticleSeeder::class,
        ]);
    }
}
```
