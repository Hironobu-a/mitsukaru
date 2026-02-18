# 士業向け求人サイト リポジトリ・Git管理戦略

**作成日**: 2026年2月18日  
**作成者**: 正社員エンジニア（Tech Lead / CTO補佐）  
**対象**: 開発チーム全員  
**ステータス**: 策定中 → 開発開始次第、本文書がソースコード管理の基準となる

---

## 0. なぜ新規リポジトリが必要か

| 観点 | 結論 |
|------|------|
| **技術スタック** | 既存（Laravel 8 / PHP 8.0）と新規（Laravel 11 / PHP 8.3）は別物。同一リポジトリに混在させるとアップグレード・Composerの依存が衝突する |
| **設計思想** | 既存は Fat Controller + テスト不足の技術的負債を抱えている。新規は最初からクリーンアーキテクチャで構築するため分離する必要がある |
| **デプロイ独立性** | 新旧でCI/CDパイプライン・インフラが異なる。同一リポジトリでは分岐管理が複雑になる |
| **チーム可読性** | 業務委託エンジニアが参画しやすいよう、1リポジトリ = 1プロダクトを原則とする |

**→ 結論：新規リポジトリを作成する。既存との共有は行わない。**

---

## 1. リポジトリ設計

### 1.1 命名・設定

| 項目 | 値 |
|------|---|
| **リポジトリ名** | `mitsupro-v2`（または `shigyo-pro`） |
| **GitHub Organization** | ミツカル組織アカウント（既存 org に追加） |
| **可視性** | `Private` |
| **デフォルトブランチ** | `main` |
| **Description** | 士業向け転職・求人プラットフォーム（Laravel 11 / PHP 8.3） |
| **Topics** | `laravel`, `php`, `mysql`, `docker`, `pest` |

### 1.2 初期ディレクトリ構成（Laravel 標準 + 追加）

```
mitsupro-v2/
├── .github/
│   ├── workflows/
│   │   ├── ci.yml          # PR時: lint + test
│   │   └── deploy.yml      # main merge時: 本番デプロイ
│   ├── PULL_REQUEST_TEMPLATE.md
│   └── ISSUE_TEMPLATE/
│       ├── feature.md
│       └── bugfix.md
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   ├── Requests/        # FormRequest（職域別サブディレクトリ）
│   │   └── Resources/       # API Resource
│   ├── Models/
│   ├── Services/            # ビジネスロジック層
│   ├── Policies/            # 認可ポリシー
│   └── Notifications/
├── database/
│   ├── migrations/
│   ├── factories/
│   └── seeders/
├── tests/
│   ├── Feature/             # Featureテスト（エンドポイントシナリオ）
│   └── Unit/                # Unitテスト（Service / Model）
├── docker/
│   ├── php/
│   │   └── Dockerfile
│   └── nginx/
│       └── default.conf
├── docker-compose.yml
├── docker-compose.testing.yml  # テスト専用（DB分離）
├── .env.example
├── .env.testing.example
├── .gitignore
├── phpstan.neon
├── pint.json                # Laravel Pint（コード整形）
└── README.md
```

---

## 2. ブランチ戦略

### 2.1 現行からの変更点と理由

現行の `tax.mitsukaru-pro` をベースにした **GitHub Flow（mainのみ）** から、  
新規開発では **develop ブランチを追加した 2 段階フロー** に変更する。

| 項目 | 現行 | 新規採用 | 変更理由 |
|------|------|----------|---------|
| ブランチ構成 | `main` のみ | `main` + `develop` | ステージング環境を設け、本番deploy前に統合テストを挟む |
| デプロイ方法 | 手動 SSH pull | GitHub Actions 自動 | 人的ミスの排除・デプロイ履歴の可視化 |
| テスト | 任意 | CI で必須化 | PRマージ前にテスト通過を強制 |
| マージ方式 | Squash and merge | **Squash and merge** 維持 | 履歴が読みやすくなる（変更なし） |

---

### 2.2 ブランチ構成

```
main ──────────────────────────────── 本番環境（★ 常にリリース可能な状態）
  ↑ Squash merge（PR必須・CI通過必須）
develop ───────────────────────────── ステージング環境（統合・動作確認）
  ↑ Squash merge（PR必須・CI通過必須）
feature/xxx   ← 開発者が切る作業ブランチ（develop から分岐）
bugfix/xxx
hotfix/xxx    ← 本番緊急対応のみ main から直接分岐（事後レビュー必須）
```

#### 各ブランチの役割

| ブランチ | 保護 | デプロイ先 | 説明 |
|---------|------|----------|------|
| `main` | ✅ 保護 | 本番 | 直接 push 禁止。`develop` からのPRのみ |
| `develop` | ✅ 保護 | ステージング | 直接 push 禁止。`feature/*` 等からのPRのみ |
| `feature/*` | ❌ | ローカル | 新機能開発。`develop` から分岐 |
| `bugfix/*` | ❌ | ローカル | バグ修正。`develop` から分岐 |
| `hotfix/*` | ❌ | ローカル | 本番緊急対応のみ `main` から分岐。修正後 `main` と `develop` の両方にマージ |

---

### 2.3 ブランチ命名規則

現行ルールを継承しつつ、プレフィックスを拡張：

```
type/IssueNumber-short-description
```

| type | 用途 |
|------|------|
| `feature` | 新機能 |
| `bugfix` | バグ修正 |
| `hotfix` | 本番緊急対応 |
| `refactor` | リファクタリング（動作変更なし） |
| `test` | テスト追加・修正のみ |
| `chore` | 依存関係更新・設定変更など |
| `docs` | ドキュメント変更のみ |

**命名例**:
```
feature/12-job-search-filter
bugfix/34-application-duplicate-error
refactor/56-job-service-extraction
test/78-application-feature-test
chore/90-update-laravel-11-2
```

---

### 2.4 Branch Protection Rules（GitHub 設定）

#### `main` ブランチ

```yaml
- Require a pull request before merging: true
  - Required approvals: 1
- Require status checks to pass before merging:
    - ci / lint
    - ci / test
- Require branches to be up to date before merging: true
- Do not allow bypassing the above settings: true
- Restrict who can push: 正社員エンジニアのみ
```

#### `develop` ブランチ

```yaml
- Require a pull request before merging: true
  - Required approvals: 1
- Require status checks to pass before merging:
    - ci / lint
    - ci / test
- Require branches to be up to date before merging: true
```

---

## 3. CI/CD パイプライン設計

### 3.1 全体フロー

```
feature/* → (PR) → develop → (PR) → main
               ↓                ↓
             CI 実行           CI 実行
          [lint][test]      [lint][test]
                                ↓
                          自動デプロイ（本番）
```

### 3.2 CI（`.github/workflows/ci.yml`）

```yaml
name: CI

on:
  pull_request:
    branches: [main, develop]

jobs:
  lint:
    name: Lint（コード整形チェック）
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: shivammathur/setup-php@v2
        with:
          php-version: '8.3'
          coverage: none
      - run: composer install --no-interaction --prefer-dist
      - name: Laravel Pint（整形チェック）
        run: ./vendor/bin/pint --test
      - name: PHPStan（静的解析）
        run: ./vendor/bin/phpstan analyse --memory-limit=512M

  test:
    name: Test（自動テスト）
    runs-on: ubuntu-latest
    services:
      mysql:
        image: mysql:8.0
        env:
          MYSQL_ROOT_PASSWORD: root
          MYSQL_DATABASE: testing
        options: --health-cmd="mysqladmin ping" --health-interval=10s
    steps:
      - uses: actions/checkout@v4
      - uses: shivammathur/setup-php@v2
        with:
          php-version: '8.3'
          coverage: xdebug
      - run: composer install --no-interaction --prefer-dist
      - run: cp .env.testing.example .env.testing
      - run: php artisan key:generate --env=testing
      - name: Pest（Feature + Unit テスト）
        run: ./vendor/bin/pest --coverage --min=60
```

> **カバレッジ最低ラインの推移計画**  
> 開発初期: 60% → 機能完成時: 70% → 安定稼働後: 80%

---

### 3.3 Deploy（`.github/workflows/deploy.yml`）

```yaml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  deploy:
    name: 本番デプロイ
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: SSH Deploy
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.PROD_HOST }}
          username: ${{ secrets.PROD_USER }}
          key: ${{ secrets.PROD_SSH_KEY }}
          script: |
            cd /var/www/mitsupro-v2
            git pull origin main
            composer install --no-dev --optimize-autoloader
            php artisan migrate --force
            php artisan config:cache
            php artisan route:cache
            php artisan view:cache
```

> **Secrets 管理（GitHub Actions）**:  
> `PROD_HOST`, `PROD_USER`, `PROD_SSH_KEY` は GitHub リポジトリの  
> Settings → Secrets and variables → Actions に登録する。  
> `.env` の本番値はサーバー上に直接配置し、コードに含めない。

---

## 4. GitHub Issues / Project 運用

### 4.1 Issue の種別

```
.github/ISSUE_TEMPLATE/
├── feature.md    # 新機能
├── bugfix.md     # バグ
└── chore.md      # 依存更新・設定変更
```

#### `feature.md` テンプレート

```markdown
## 概要
（何を実現したいか 1-2 文で）

## 受け入れ条件
- [ ] ...
- [ ] ...

## 技術的な考慮事項
（任意）

## 関連 Issue / デザイン
（任意）
```

### 4.2 GitHub Projects でのタスク管理

```
Board View のカラム構成:
│ Backlog │ Todo │ In Progress │ In Review │ Done │
```

- Issue 作成時は `Backlog` に入れる
- スプリント開始時に `Todo` に移動（週単位が目安）
- 着手時に `In Progress`、PR 作成時に `In Review` に移動
- マージ後に `Done` へ（GitHub が自動でクローズ）

---

## 5. コミットメッセージ規約（現行から継承・強化）

現行の Conventional Commits 準拠を維持した上で、以下を追加・明確化する：

```
<type>(<scope>): <subject>

[body - 任意: なぜこの変更をするのか]
[footer - 任意: Closes #IssueNumber]
```

### スコープの例（新プロジェクト用）

| scope | 対象 |
|-------|------|
| `job` | 求人機能 |
| `application` | 応募機能 |
| `user` | 求職者アカウント |
| `company` | 企業アカウント |
| `auth` | 認証 |
| `article` | 記事・コンテンツ |
| `infra` | Docker / CI / デプロイ設定 |
| `db` | Migration / Seeder |

**例**:
```
feat(job): 求人検索に年収範囲フィルタを追加

salary_min / salary_max パラメータを受け付け、
JobSearchService にフィルタロジックを実装。

Closes #42
```

---

## 6. 開発フロー（日次）

```
1. Issue 確認 → Todo から着手するものを選ぶ
2. develop から feature ブランチを切る
   git switch develop && git pull
   git switch -c feature/42-salary-filter

3. 実装 + テスト追加（Pest）
4. ローカルで CI と同等チェックを実施
   ./vendor/bin/pint              # 整形
   ./vendor/bin/phpstan analyse   # 静的解析
   ./vendor/bin/pest              # テスト

5. develop への PR を作成（テンプレート使用）
6. CI が自動実行される（lint + test）
7. レビュワーが Approve → Squash and merge
8. develop のステージングで動作確認
9. リリース準備完了 → develop → main への PR
10. Approve → Squash and merge → 自動で本番デプロイ
```

---

## 7. フェーズ別 Git 運用ロードマップ

### Phase 0: リポジトリ初期構築（今週中）

- [ ] GitHub に `mitsupro-v2` プライベートリポジトリを作成
- [ ] `main` / `develop` の Branch Protection Rules を設定
- [ ] `composer create-project laravel/laravel` で Laravel 11 初期化
- [ ] Docker 環境（php:8.3 + nginx + mysql:8.0）を構築
- [ ] `.github/workflows/ci.yml` を作成・動作確認
- [ ] `.env.example` / `.env.testing.example` を作成（本番値は含めない）
- [ ] PHPStan + Laravel Pint を導入・設定
- [ ] Github Secrets に SSH 鍵等を登録
- [ ] `README.md` に環境構築手順を記載

### Phase 1: 基盤 Migration + モデル（来週）

- [ ] マスターテーブル（professions / prefectures / employment_types）を Migration 作成
- [ ] users / companies / jobs の Migration + Factory 作成
- [ ] 中間テーブル（job_skills / job_professions / job_favorites）を作成
- [ ] `database/seeders/DevelopmentSeeder.php` で開発用データを投入
- [ ] 全 Migration のテストをローカルで通す（`migrate:refresh --seed`）

### Phase 2: 認証・求人 CRUD（以降順次）

- [ ] Laravel Breeze / Fortify で認証基盤を構築
- [ ] 求人一覧・詳細・検索の実装
- [ ] 応募・お気に入り機能の実装
- [ ] Feature テストを各機能と並行で作成

---

## 8. 既存プロジェクト（tax.mitsukaru-pro）との関係

| 項目 | 既存 (`tax.mitsukaru-pro`) | 新規 (`mitsupro-v2`) |
|------|--------------------------|---------------------|
| リポジトリ | 既存のまま維持 | 新規作成 |
| ブランチ戦略 | GitHub Flow（main のみ）のまま | develop 追加の 2 段階フロー |
| デプロイ | 手動 SSH pull のまま | GitHub Actions 自動デプロイ |
| テスト | 現状のテストを継続拡充 | 初期から Pest + CI 必須 |
| 並走期間 | 新規が本番稼動するまで維持 | 段階的に機能を移行 |

---

## 参考：.gitignore（新規プロジェクト用）

```gitignore
/node_modules
/public/hot
/public/storage
/storage/*.key
/vendor
.env
.env.testing       # ← 絶対に追跡しない（既存の教訓）
.env.backup
.env.production
.phpunit.result.cache
Homestead.json
Homestead.yaml
npm-debug.log
yarn-error.log
/.idea
/.vscode
.DS_Store
/storage/test-reports/
/public/test-reports/
```

> **⚠️ 最重要ルール（既存プロジェクトの反省から）**  
> `.env` と `.env.testing` は **絶対にコミットしない**。  
> 初回 `git init` の直後に `.gitignore` を必ずコミットすること。  
> `git status` で tracked になっていないか毎回確認する。

---

**次のアクション（Tech Lead として優先で実施）**:

1. [ ] GitHub Organization に `mitsupro-v2` リポジトリを作成
2. [ ] 本ドキュメントをチームに共有・合意を得る
3. [ ] Phase 0 の初期構築タスクを GitHub Issues に起票
4. [ ] 設計書（`SHIGYO_JOB_SITE_SYSTEM_DESIGN.md`）のMigration を実装ブランチで書き起こす
