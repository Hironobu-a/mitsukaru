# 技術選定・アーキテクチャ設計書

**作成日**: 2026年2月18日
**バージョン**: 1.0

---

## 1. 技術選定の方針

### 1.1 選定基準

| 基準 | 重み | 説明 |
|------|------|------|
| 開発生産性 | ★★★★★ | 少人数チームでの高速開発 |
| パフォーマンス | ★★★★☆ | Core Web Vitals 達成 |
| SEO対応 | ★★★★★ | 求人サイトの生命線 |
| エコシステム | ★★★★☆ | ライブラリ・コミュニティの充実度 |
| 採用容易性 | ★★★☆☆ | エンジニア採用のしやすさ |
| 運用コスト | ★★★★☆ | スタートアップフェーズのコスト最適化 |
| 拡張性 | ★★★★☆ | Phase 2/3 への対応力 |

### 1.2 既存設計書からの変更点

既存のシステム設計書では `Laravel 11 + Blade` を想定していたが、以下の理由でフロントエンドを分離する。

| 観点 | Blade（従来案） | Next.js + Laravel API（採用案） |
|------|-----------------|-------------------------------|
| SEO | SSR不可（Blade自体はSSRだが動的UI困難） | App Router による完全SSR/SSG対応 |
| UX | ページ遷移ごとにリロード | SPA的な高速遷移 + プリフェッチ |
| 開発速度 | フロント・バック混在で非効率 | 完全分離で並行開発可能 |
| 型安全性 | なし | TypeScript による完全な型安全 |
| コンポーネント再利用 | 困難 | React コンポーネントの高い再利用性 |
| モバイル対応 | 別途開発が必要 | React Native でコード共有可能 |

---

## 2. 技術スタック

### 2.1 全体構成図

```
┌─────────────────────────────────────────────────────────────────┐
│                        Client Layer                              │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  Next.js 15 (App Router)                                  │   │
│  │  React 19 + TypeScript 5.x                                │   │
│  │  Tailwind CSS 4 + shadcn/ui                               │   │
│  │  TanStack Query v5 (Server State)                         │   │
│  │  Zustand (Client State)                                   │   │
│  └──────────────────────────────────────────────────────────┘   │
│                              │                                    │
│                         HTTPS / REST                              │
│                              │                                    │
├──────────────────────────────┼────────────────────────────────────┤
│                        API Layer                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  Laravel 11 (PHP 8.3)                                     │   │
│  │  Laravel Sanctum (SPA Authentication)                     │   │
│  │  FormRequest + Service Layer                              │   │
│  │  Laravel Horizon (Queue Dashboard)                        │   │
│  │  Laravel Telescope (Debug - dev only)                     │   │
│  └──────────────────────────────────────────────────────────┘   │
│                              │                                    │
├──────────────────────────────┼────────────────────────────────────┤
│                       Data Layer                                  │
│  ┌────────────┐  ┌──────────┐  ┌────────────┐  ┌───────────┐   │
│  │ MySQL 8.0  │  │ Redis 7  │  │ Meilisearch│  │ S3 / MinIO│   │
│  │ (Primary)  │  │ (Cache/  │  │ (全文検索)  │  │ (Storage) │   │
│  │            │  │  Session/ │  │            │  │           │   │
│  │            │  │  Queue)   │  │            │  │           │   │
│  └────────────┘  └──────────┘  └────────────┘  └───────────┘   │
│                                                                   │
├───────────────────────────────────────────────────────────────────┤
│                    Infrastructure Layer                            │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  Docker / Docker Compose (Local Dev)                      │   │
│  │  AWS ECS Fargate / EC2 (Production)                       │   │
│  │  CloudFront + S3 (Frontend CDN)                           │   │
│  │  RDS MySQL (Managed DB)                                   │   │
│  │  ElastiCache Redis (Managed Cache)                        │   │
│  │  GitHub Actions (CI/CD)                                   │   │
│  │  Sentry (Error Tracking)                                  │   │
│  └──────────────────────────────────────────────────────────┘   │
└───────────────────────────────────────────────────────────────────┘
```

### 2.2 フロントエンド

| カテゴリ | 技術 | バージョン | 選定理由 |
|---------|------|-----------|---------|
| フレームワーク | **Next.js** | 15.x | App Router によるSSR/SSG、SEO最適化、React Server Components |
| UI ライブラリ | **React** | 19.x | Server Components、Suspense、Concurrent Features |
| 言語 | **TypeScript** | 5.x | 型安全性による品質向上・開発効率 |
| CSS | **Tailwind CSS** | 4.x | ユーティリティファースト、高速スタイリング |
| UIコンポーネント | **shadcn/ui** | latest | カスタマイズ可能・アクセシブル・モダンデザイン |
| サーバー状態管理 | **TanStack Query** | v5 | キャッシュ・再検証・楽観的更新 |
| クライアント状態 | **Zustand** | v5 | 軽量・シンプル・TypeScript親和性 |
| フォーム | **React Hook Form** | v7 | パフォーマンス・バリデーション統合 |
| バリデーション | **Zod** | v3 | TypeScript ファーストなスキーマバリデーション |
| アイコン | **Lucide React** | latest | 軽量・一貫性のあるアイコンセット |
| 日付操作 | **date-fns** | v3 | Tree-shakeable・軽量 |
| テスト | **Vitest** + **Testing Library** | latest | 高速・React Testing Library 統合 |
| E2E テスト | **Playwright** | latest | クロスブラウザ・高信頼性 |
| リンター | **ESLint** + **Prettier** | latest | コード品質・フォーマット統一 |

### 2.3 バックエンド

| カテゴリ | 技術 | バージョン | 選定理由 |
|---------|------|-----------|---------|
| フレームワーク | **Laravel** | 11.x | 既存チームの知見活用・充実したエコシステム |
| 言語 | **PHP** | 8.3 | Typed Properties、Enum、Fibers |
| DB | **MySQL** | 8.0 | 実績・安定性・RDS対応 |
| キャッシュ/セッション | **Redis** | 7.x | 高速・多機能（キャッシュ・セッション・キュー） |
| 全文検索 | **Meilisearch** | 1.x | 高速・日本語対応・Laravel Scout 統合 |
| ファイルストレージ | **S3** (MinIO for dev) | - | スケーラブル・低コスト |
| 認証 | **Laravel Sanctum** | - | SPA認証に最適・シンプル |
| キュー | **Laravel Queue** (Redis driver) | - | メール送信・通知の非同期処理 |
| メール | **Laravel Mail** + **Amazon SES** | - | 高配信率・低コスト |
| テスト | **Pest** | v3 | モダンな記法・Laravel統合 |
| 静的解析 | **PHPStan** (Level 8) | latest | 型安全性の担保 |
| コードスタイル | **Laravel Pint** | latest | PSR-12準拠・自動修正 |
| API ドキュメント | **Scramble** | latest | Laravel API の自動ドキュメント生成 |

### 2.4 インフラ・DevOps

| カテゴリ | 技術 | 選定理由 |
|---------|------|---------|
| コンテナ | **Docker** + **Docker Compose** | ローカル開発環境の統一 |
| CI/CD | **GitHub Actions** | GitHub統合・無料枠充実 |
| ホスティング (Frontend) | **Vercel** or **AWS CloudFront + S3** | Next.js最適化・CDN |
| ホスティング (Backend) | **AWS ECS Fargate** or **EC2** | 既存AWS環境との親和性 |
| DB | **AWS RDS MySQL** | マネージド・自動バックアップ |
| キャッシュ | **AWS ElastiCache Redis** | マネージド・高可用性 |
| DNS | **AWS Route 53** | 既存管理との統合 |
| SSL | **AWS ACM** | 自動更新・無料 |
| 監視 | **Sentry** + **UptimeRobot** + **CloudWatch** | エラー追跡・死活監視・メトリクス |
| ログ | **CloudWatch Logs** | AWS統合・検索・アラート |

---

## 3. アーキテクチャ設計

### 3.1 全体アーキテクチャ: API分離型モノリス

```
                    ┌─────────────────┐
                    │   CloudFront    │
                    │   (CDN)         │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
    ┌─────────▼──────┐  ┌───▼────────┐  ┌──▼──────────┐
    │  Vercel / S3   │  │  ALB       │  │  S3         │
    │  (Frontend)    │  │  (API LB)  │  │  (Assets)   │
    │  Next.js SSR   │  │            │  │             │
    └────────────────┘  └───┬────────┘  └─────────────┘
                            │
                   ┌────────▼────────┐
                   │  ECS Fargate    │
                   │  Laravel API    │
                   │  (Auto Scaling) │
                   └───┬────┬───┬───┘
                       │    │   │
              ┌────────┘    │   └────────┐
              │             │            │
    ┌─────────▼──┐  ┌──────▼──┐  ┌──────▼──────┐
    │  RDS MySQL │  │  Redis  │  │ Meilisearch │
    │  (Primary  │  │  Cache  │  │  (Search)   │
    │  + Replica)│  │  Queue  │  │             │
    └────────────┘  └─────────┘  └─────────────┘
```

### 3.2 バックエンドアーキテクチャ（レイヤード + Service）

```
HTTP Request
    │
    ▼
Middleware（認証・CORS・レート制限・ログ）
    │
    ▼
Route（api.php / web.php）
    │
    ▼
FormRequest（バリデーション + 認可）
    │
    ▼
Controller（薄く保つ：リクエスト受付 → Service呼出 → レスポンス整形）
    │
    ▼
Service（ビジネスロジック / トランザクション / イベント発火）
    │
    ├──▶ Model / Eloquent（DB操作・リレーション・スコープ）
    ├──▶ Notification（メール・Slack通知）
    ├──▶ Event / Listener（非同期処理のトリガー）
    └──▶ Job（キュー処理：メール送信・検索インデックス更新）
```

### 3.3 ディレクトリ構成

#### バックエンド（Laravel）

```
backend/
├── app/
│   ├── Console/
│   │   └── Commands/           # Artisan カスタムコマンド
│   ├── Enums/                  # PHP 8.1 Enum（ステータス定数等）
│   │   ├── JobStatus.php
│   │   ├── ApplicationStatus.php
│   │   ├── CompanyStatus.php
│   │   └── UserStatus.php
│   ├── Events/                 # ドメインイベント
│   │   ├── ApplicationCreated.php
│   │   └── JobPublished.php
│   ├── Exceptions/             # カスタム例外
│   │   └── BusinessException.php
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── Api/
│   │   │   │   ├── V1/        # API v1
│   │   │   │   │   ├── AuthController.php
│   │   │   │   │   ├── JobController.php
│   │   │   │   │   ├── ApplicationController.php
│   │   │   │   │   ├── FavoriteController.php
│   │   │   │   │   ├── UserProfileController.php
│   │   │   │   │   ├── CompanyController.php
│   │   │   │   │   └── ArticleController.php
│   │   │   │   └── Company/   # 企業側API
│   │   │   │       ├── JobManageController.php
│   │   │   │       └── ApplicationManageController.php
│   │   │   └── Admin/         # 管理者API
│   │   │       ├── DashboardController.php
│   │   │       ├── CompanyVerificationController.php
│   │   │       └── UserManageController.php
│   │   ├── Middleware/
│   │   │   ├── ForceJsonResponse.php
│   │   │   └── EnsureCompanyVerified.php
│   │   ├── Requests/           # FormRequest
│   │   │   ├── Auth/
│   │   │   ├── Job/
│   │   │   ├── Application/
│   │   │   ├── Company/
│   │   │   └── Article/
│   │   └── Resources/          # API Resource（レスポンス整形）
│   │       ├── JobResource.php
│   │       ├── JobCollection.php
│   │       ├── CompanyResource.php
│   │       ├── ApplicationResource.php
│   │       └── ArticleResource.php
│   ├── Jobs/                   # Queue Jobs
│   │   ├── SendApplicationNotification.php
│   │   └── SyncSearchIndex.php
│   ├── Listeners/
│   │   ├── NotifyCompanyOnApplication.php
│   │   └── UpdateSearchIndexOnJobChange.php
│   ├── Models/
│   │   ├── User.php
│   │   ├── UserProfile.php
│   │   ├── Company.php
│   │   ├── CompanyProfile.php
│   │   ├── CompanyUser.php
│   │   ├── Job.php
│   │   ├── JobApplication.php
│   │   ├── JobFavorite.php
│   │   ├── Article.php
│   │   ├── Profession.php
│   │   ├── Prefecture.php
│   │   ├── EmploymentType.php
│   │   ├── SalaryType.php
│   │   ├── Skill.php
│   │   └── AdminUser.php
│   ├── Notifications/
│   │   ├── ApplicationReceivedNotification.php
│   │   ├── ApplicationStatusChangedNotification.php
│   │   └── WelcomeNotification.php
│   ├── Policies/
│   │   ├── JobPolicy.php
│   │   ├── ApplicationPolicy.php
│   │   └── CompanyPolicy.php
│   ├── Providers/
│   │   ├── AppServiceProvider.php
│   │   └── EventServiceProvider.php
│   ├── Rules/                  # カスタムバリデーションルール
│   │   ├── UniqueApplicationRule.php
│   │   └── ValidLicenseRule.php
│   └── Services/
│       ├── Job/
│       │   ├── JobCreateService.php
│       │   ├── JobSearchService.php
│       │   └── JobStatusService.php
│       ├── Application/
│       │   └── ApplicationService.php
│       ├── User/
│       │   └── UserRegistrationService.php
│       └── Company/
│           └── CompanyVerificationService.php
├── config/
├── database/
│   ├── factories/
│   ├── migrations/
│   └── seeders/
├── routes/
│   ├── api.php                 # v1 API ルート
│   ├── api_company.php         # 企業側 API ルート
│   └── api_admin.php           # 管理者 API ルート
├── tests/
│   ├── Feature/
│   │   ├── Api/
│   │   │   ├── AuthTest.php
│   │   │   ├── JobTest.php
│   │   │   ├── ApplicationTest.php
│   │   │   └── FavoriteTest.php
│   │   └── Company/
│   │       ├── JobManageTest.php
│   │       └── ApplicationManageTest.php
│   └── Unit/
│       ├── Services/
│       │   ├── JobSearchServiceTest.php
│       │   └── ApplicationServiceTest.php
│       └── Models/
│           ├── JobTest.php
│           └── UserTest.php
├── docker-compose.yml
├── Dockerfile
├── phpstan.neon
├── pint.json
└── phpunit.xml
```

#### フロントエンド（Next.js）

```
frontend/
├── src/
│   ├── app/                    # App Router
│   │   ├── (public)/           # 公開ページ（レイアウトグループ）
│   │   │   ├── page.tsx        # トップページ
│   │   │   ├── jobs/
│   │   │   │   ├── page.tsx    # 求人一覧
│   │   │   │   └── [id]/
│   │   │   │       └── page.tsx # 求人詳細
│   │   │   ├── companies/
│   │   │   │   └── [id]/
│   │   │   │       └── page.tsx # 企業詳細
│   │   │   ├── articles/
│   │   │   │   ├── page.tsx    # 記事一覧
│   │   │   │   └── [slug]/
│   │   │   │       └── page.tsx # 記事詳細
│   │   │   └── [profession]/   # 職種別LP (/tax-accountant, /lawyer)
│   │   │       └── page.tsx
│   │   ├── (auth)/             # 認証ページ
│   │   │   ├── login/
│   │   │   │   └── page.tsx
│   │   │   └── register/
│   │   │       └── page.tsx
│   │   ├── (dashboard)/        # 求職者ダッシュボード（認証必須）
│   │   │   ├── layout.tsx
│   │   │   ├── profile/
│   │   │   │   └── page.tsx
│   │   │   ├── applications/
│   │   │   │   └── page.tsx
│   │   │   └── favorites/
│   │   │       └── page.tsx
│   │   ├── company/            # 企業管理画面（認証必須）
│   │   │   ├── layout.tsx
│   │   │   ├── dashboard/
│   │   │   │   └── page.tsx
│   │   │   ├── jobs/
│   │   │   │   ├── page.tsx    # 求人管理一覧
│   │   │   │   ├── new/
│   │   │   │   │   └── page.tsx
│   │   │   │   └── [id]/
│   │   │   │       └── edit/
│   │   │   │           └── page.tsx
│   │   │   └── applications/
│   │   │       └── page.tsx
│   │   ├── admin/              # 管理画面（認証必須）
│   │   │   ├── layout.tsx
│   │   │   ├── dashboard/
│   │   │   │   └── page.tsx
│   │   │   ├── companies/
│   │   │   │   └── page.tsx
│   │   │   ├── jobs/
│   │   │   │   └── page.tsx
│   │   │   └── users/
│   │   │       └── page.tsx
│   │   ├── layout.tsx          # ルートレイアウト
│   │   ├── not-found.tsx
│   │   └── error.tsx
│   ├── components/
│   │   ├── ui/                 # shadcn/ui ベースコンポーネント
│   │   │   ├── button.tsx
│   │   │   ├── input.tsx
│   │   │   ├── card.tsx
│   │   │   ├── dialog.tsx
│   │   │   ├── select.tsx
│   │   │   └── ...
│   │   ├── layout/             # レイアウトコンポーネント
│   │   │   ├── header.tsx
│   │   │   ├── footer.tsx
│   │   │   ├── sidebar.tsx
│   │   │   └── mobile-nav.tsx
│   │   ├── job/                # 求人関連コンポーネント
│   │   │   ├── job-card.tsx
│   │   │   ├── job-list.tsx
│   │   │   ├── job-search-form.tsx
│   │   │   ├── job-detail.tsx
│   │   │   ├── job-apply-dialog.tsx
│   │   │   └── job-favorite-button.tsx
│   │   ├── company/            # 企業関連コンポーネント
│   │   │   ├── company-card.tsx
│   │   │   └── company-profile.tsx
│   │   ├── article/            # 記事関連コンポーネント
│   │   │   ├── article-card.tsx
│   │   │   └── article-list.tsx
│   │   └── shared/             # 共通コンポーネント
│   │       ├── pagination.tsx
│   │       ├── search-filters.tsx
│   │       ├── loading-skeleton.tsx
│   │       ├── empty-state.tsx
│   │       └── seo-head.tsx
│   ├── hooks/                  # カスタムフック
│   │   ├── use-auth.ts
│   │   ├── use-jobs.ts
│   │   ├── use-applications.ts
│   │   ├── use-favorites.ts
│   │   └── use-debounce.ts
│   ├── lib/                    # ユーティリティ
│   │   ├── api-client.ts       # Axios / fetch ラッパー
│   │   ├── auth.ts             # 認証ヘルパー
│   │   ├── constants.ts        # 定数
│   │   ├── utils.ts            # 汎用ユーティリティ
│   │   └── validations.ts      # Zod スキーマ
│   ├── stores/                 # Zustand ストア
│   │   ├── auth-store.ts
│   │   └── ui-store.ts
│   └── types/                  # TypeScript 型定義
│       ├── api.ts              # API レスポンス型
│       ├── job.ts
│       ├── user.ts
│       ├── company.ts
│       └── article.ts
├── public/
│   ├── images/
│   └── favicon.ico
├── tailwind.config.ts
├── next.config.ts
├── tsconfig.json
├── vitest.config.ts
├── playwright.config.ts
└── package.json
```

---

## 4. 認証・認可アーキテクチャ

### 4.1 認証フロー（Laravel Sanctum SPA認証）

```
┌──────────┐     GET /sanctum/csrf-cookie     ┌──────────┐
│          │ ──────────────────────────────▶  │          │
│  Next.js │     Set-Cookie: XSRF-TOKEN       │  Laravel  │
│  (SPA)   │ ◀──────────────────────────────  │  API     │
│          │                                   │          │
│          │     POST /api/v1/login            │          │
│          │     Cookie: XSRF-TOKEN            │          │
│          │ ──────────────────────────────▶  │          │
│          │     Set-Cookie: session           │          │
│          │ ◀──────────────────────────────  │          │
│          │                                   │          │
│          │     GET /api/v1/user              │          │
│          │     Cookie: session               │          │
│          │ ──────────────────────────────▶  │          │
│          │     { user data }                 │          │
│          │ ◀──────────────────────────────  │          │
└──────────┘                                   └──────────┘
```

### 4.2 マルチガード構成

```php
// config/auth.php
'guards' => [
    'web' => [          // 求職者
        'driver' => 'session',
        'provider' => 'users',
    ],
    'company' => [      // 企業担当者
        'driver' => 'session',
        'provider' => 'company_users',
    ],
    'admin' => [        // 管理者
        'driver' => 'session',
        'provider' => 'admin_users',
    ],
],
```

### 4.3 RBAC（Role-Based Access Control）

```
求職者 (User)
├── 求人検索・閲覧
├── 求人応募
├── お気に入り管理
├── プロフィール管理
└── 応募履歴閲覧

企業担当者 (CompanyUser)
├── 自社求人CRUD
├── 自社応募者管理
├── 企業プロフィール管理
└── ステータス更新

管理者 (AdminUser)
├── 全求人管理
├── 全ユーザー管理
├── 企業審査
├── コンテンツ管理
├── KPIダッシュボード
└── システム設定
```

---

## 5. キャッシュ戦略

### 5.1 キャッシュレイヤー

| レイヤー | 対象 | TTL | 無効化タイミング |
|---------|------|-----|-----------------|
| CDN (CloudFront) | 静的アセット・SSGページ | 24h | デプロイ時 |
| Redis | 求人一覧・マスターデータ | 1h | 求人更新時 |
| Redis | 検索結果 | 15min | 求人更新時 |
| Redis | ユーザーセッション | 2h | ログアウト時 |
| Next.js ISR | 求人詳細ページ | 1h | On-demand Revalidation |
| Next.js ISR | 記事ページ | 24h | On-demand Revalidation |
| Browser | API レスポンス (TanStack Query) | 5min | staleTime 設定 |

### 5.2 キャッシュ無効化パターン

```php
// 求人更新時のキャッシュ無効化
class JobCreateService
{
    public function create(Company $company, array $data): Job
    {
        return DB::transaction(function () use ($company, $data) {
            $job = $company->jobs()->create($data);

            // Redis キャッシュ無効化
            Cache::tags(['jobs', "company:{$company->id}"])->flush();

            // Next.js ISR の On-demand Revalidation
            Http::post(config('app.frontend_url') . '/api/revalidate', [
                'secret' => config('app.revalidation_secret'),
                'paths' => ['/jobs', "/jobs/{$job->id}"],
            ]);

            return $job;
        });
    }
}
```

---

## 6. 検索アーキテクチャ（Meilisearch）

### 6.1 選定理由

| 比較項目 | MySQL LIKE | Elasticsearch | Meilisearch |
|---------|------------|---------------|-------------|
| セットアップ | 不要 | 複雑 | 簡単 |
| 日本語対応 | △（FULLTEXT） | ○（kuromoji） | ○（組込み） |
| メモリ使用量 | - | 大 | 小 |
| Laravel統合 | Eloquent | Scout + Driver | Scout + 公式Driver |
| 運用コスト | 低 | 高 | 中 |
| タイポ耐性 | なし | あり | あり |
| ファセット検索 | 不可 | 可能 | 可能 |

### 6.2 インデックス設計

```php
// Job Model
class Job extends Model
{
    use Searchable;

    public function toSearchableArray(): array
    {
        return [
            'id' => $this->id,
            'title' => $this->title,
            'description' => $this->description,
            'company_name' => $this->company->name,
            'prefecture' => $this->prefecture?->name,
            'professions' => $this->professions->pluck('name')->toArray(),
            'skills' => $this->skills->pluck('name')->toArray(),
            'employment_type' => $this->employmentType->name,
            'salary_min' => $this->salary_min,
            'salary_max' => $this->salary_max,
            'published_at' => $this->published_at?->timestamp,
        ];
    }

    public function searchableAs(): string
    {
        return 'jobs';
    }
}
```

---

## 7. 非同期処理設計

### 7.1 キュー対象

| ジョブ | トリガー | 優先度 |
|--------|---------|--------|
| 応募通知メール | 応募作成 | high |
| ステータス変更通知 | ステータス更新 | high |
| ウェルカムメール | 会員登録 | default |
| 検索インデックス更新 | 求人CRUD | default |
| 企業審査完了通知 | 審査ステータス変更 | high |
| 日次レポート生成 | スケジュール（毎朝9時） | low |

### 7.2 キュー構成

```php
// config/queue.php
'connections' => [
    'redis' => [
        'driver' => 'redis',
        'connection' => 'queue',
        'queue' => 'default',
        'retry_after' => 90,
        'block_for' => 5,
    ],
],

// Queue名の分離
// high: 通知系（即時性が必要）
// default: 検索インデックス更新等
// low: レポート生成等
```

---

## 8. エラーハンドリング

### 8.1 API エラーレスポンス統一フォーマット

```json
{
    "message": "バリデーションエラーが発生しました。",
    "errors": {
        "title": ["求人タイトルは必須です。"],
        "salary_max": ["上限年収は下限年収以上の値を入力してください。"]
    },
    "error_code": "VALIDATION_ERROR",
    "status": 422
}
```

### 8.2 例外ハンドリング階層

```
BusinessException（ビジネスロジックエラー → 422）
├── DuplicateApplicationException
├── CompanyNotVerifiedException
└── JobNotPublishedException

AuthenticationException（認証エラー → 401）
AuthorizationException（認可エラー → 403）
ModelNotFoundException（リソース不在 → 404）
ValidationException（バリデーション → 422）
ThrottleRequestsException（レート制限 → 429）
```

---

## 9. ログ・監視設計

### 9.1 ログレベル

| レベル | 用途 | 出力先 |
|--------|------|--------|
| ERROR | 例外・障害 | CloudWatch + Sentry |
| WARNING | 想定外の状態（二重応募試行等） | CloudWatch |
| INFO | 重要操作（応募・求人公開等） | CloudWatch |
| DEBUG | 開発用詳細ログ | ローカルのみ |

### 9.2 監査ログ

```php
// 重要操作の監査ログ
activity()
    ->performedOn($job)
    ->causedBy($user)
    ->withProperties(['status' => 'published'])
    ->log('求人を公開しました');
```
