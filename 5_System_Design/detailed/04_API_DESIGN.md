# API 設計書

**作成日**: 2026年2月18日
**バージョン**: 1.0
**ベースURL**: `https://api.mitsupro.jp/api/v1`

---

## 1. API 設計方針

### 1.1 基本原則

- RESTful API 設計に準拠
- JSON レスポンス統一
- API バージョニング（URL パス方式: `/api/v1/`）
- Laravel API Resource によるレスポンス整形
- ページネーションは cursor-based を基本、offset-based も対応
- 日本語エラーメッセージ
- レート制限: 認証済み 60req/min、未認証 30req/min

### 1.2 共通レスポンスフォーマット

#### 成功レスポンス（単一リソース）
```json
{
    "data": {
        "id": 1,
        "title": "税理士事務所スタッフ募集",
        ...
    }
}
```

#### 成功レスポンス（コレクション + ページネーション）
```json
{
    "data": [...],
    "meta": {
        "current_page": 1,
        "last_page": 10,
        "per_page": 20,
        "total": 195,
        "from": 1,
        "to": 20
    },
    "links": {
        "first": "...?page=1",
        "last": "...?page=10",
        "prev": null,
        "next": "...?page=2"
    }
}
```

#### エラーレスポンス
```json
{
    "message": "バリデーションエラーが発生しました。",
    "errors": {
        "title": ["求人タイトルは必須です。"]
    }
}
```

### 1.3 HTTP ステータスコード

| コード | 用途 |
|--------|------|
| 200 | 成功（取得・更新） |
| 201 | 成功（作成） |
| 204 | 成功（削除・コンテンツなし） |
| 400 | 不正なリクエスト |
| 401 | 未認証 |
| 403 | 権限なし |
| 404 | リソース不在 |
| 409 | 競合（二重応募等） |
| 422 | バリデーションエラー |
| 429 | レート制限超過 |
| 500 | サーバーエラー |

---

## 2. 認証 API

### 2.1 求職者認証

| メソッド | エンドポイント | 説明 | 認証 |
|---------|---------------|------|------|
| GET | `/sanctum/csrf-cookie` | CSRF トークン取得 | 不要 |
| POST | `/api/v1/auth/register` | 会員登録 | 不要 |
| POST | `/api/v1/auth/login` | ログイン | 不要 |
| POST | `/api/v1/auth/logout` | ログアウト | 必要 |
| GET | `/api/v1/auth/user` | 認証ユーザー取得 | 必要 |
| POST | `/api/v1/auth/forgot-password` | パスワードリセットメール送信 | 不要 |
| POST | `/api/v1/auth/reset-password` | パスワードリセット実行 | 不要 |
| POST | `/api/v1/auth/email/verify/{id}/{hash}` | メール認証 | 不要 |
| POST | `/api/v1/auth/email/resend` | 認証メール再送 | 必要 |

#### POST `/api/v1/auth/register` - 会員登録

**Request Body:**
```json
{
    "email": "user@example.com",
    "password": "securePassword123",
    "password_confirmation": "securePassword123",
    "last_name": "山田",
    "first_name": "太郎",
    "last_name_kana": "ヤマダ",
    "first_name_kana": "タロウ"
}
```

**Response (201):**
```json
{
    "data": {
        "id": 1,
        "email": "user@example.com",
        "last_name": "山田",
        "first_name": "太郎",
        "email_verified": false
    },
    "message": "登録が完了しました。認証メールを送信しました。"
}
```

#### POST `/api/v1/auth/login` - ログイン

**Request Body:**
```json
{
    "email": "user@example.com",
    "password": "securePassword123"
}
```

**Response (200):**
```json
{
    "data": {
        "id": 1,
        "email": "user@example.com",
        "last_name": "山田",
        "first_name": "太郎",
        "email_verified": true,
        "profile": { ... }
    }
}
```

### 2.2 企業担当者認証

| メソッド | エンドポイント | 説明 | 認証 |
|---------|---------------|------|------|
| POST | `/api/v1/company/auth/register` | 企業登録 | 不要 |
| POST | `/api/v1/company/auth/login` | ログイン | 不要 |
| POST | `/api/v1/company/auth/logout` | ログアウト | 必要 |
| GET | `/api/v1/company/auth/user` | 認証ユーザー取得 | 必要 |

### 2.3 管理者認証

| メソッド | エンドポイント | 説明 | 認証 |
|---------|---------------|------|------|
| POST | `/api/v1/admin/auth/login` | ログイン | 不要 |
| POST | `/api/v1/admin/auth/logout` | ログアウト | 必要 |
| GET | `/api/v1/admin/auth/user` | 認証ユーザー取得 | 必要 |

---

## 3. 求人 API（公開）

| メソッド | エンドポイント | 説明 | 認証 |
|---------|---------------|------|------|
| GET | `/api/v1/jobs` | 求人一覧（検索） | 不要 |
| GET | `/api/v1/jobs/{id}` | 求人詳細 | 不要 |
| GET | `/api/v1/jobs/{id}/related` | 関連求人 | 不要 |

### GET `/api/v1/jobs` - 求人一覧（検索）

**Query Parameters:**

| パラメータ | 型 | 必須 | 説明 |
|-----------|-----|------|------|
| keyword | string | No | フリーワード検索 |
| profession_ids[] | integer[] | No | 職種ID（複数可） |
| prefecture_id | integer | No | 勤務都道府県ID |
| employment_type_ids[] | integer[] | No | 雇用形態ID（複数可） |
| salary_min | integer | No | 年収下限 |
| salary_max | integer | No | 年収上限 |
| remote_type | integer | No | リモート区分 |
| skill_ids[] | integer[] | No | スキルID（複数可） |
| sort | string | No | newest / salary_high / salary_low (default: newest) |
| page | integer | No | ページ番号 (default: 1) |
| per_page | integer | No | 件数 (default: 20, max: 50) |

**Response (200):**
```json
{
    "data": [
        {
            "id": 1,
            "title": "【東京】税理士事務所スタッフ募集",
            "company": {
                "id": 10,
                "name": "山田税理士事務所",
                "logo_url": "https://..."
            },
            "professions": [
                { "id": 1, "name": "税理士", "slug": "tax-accountant" }
            ],
            "employment_type": { "id": 1, "name": "正社員" },
            "salary_type": { "id": 1, "name": "年俸" },
            "salary_min": 4000000,
            "salary_max": 6000000,
            "salary_display": "400万円〜600万円",
            "prefecture": { "id": 13, "name": "東京都" },
            "work_location": "東京都千代田区丸の内1-1-1",
            "remote_type": 1,
            "remote_type_label": "一部リモート",
            "skills": [
                { "id": 1, "name": "法人税申告" },
                { "id": 2, "name": "相続税" }
            ],
            "is_favorited": false,
            "published_at": "2026-02-15",
            "created_at": "2026-02-15T10:00:00Z"
        }
    ],
    "meta": {
        "current_page": 1,
        "last_page": 5,
        "per_page": 20,
        "total": 95
    },
    "links": { ... },
    "facets": {
        "professions": [
            { "id": 1, "name": "税理士", "count": 45 },
            { "id": 2, "name": "公認会計士", "count": 30 }
        ],
        "prefectures": [
            { "id": 13, "name": "東京都", "count": 40 },
            { "id": 27, "name": "大阪府", "count": 20 }
        ],
        "employment_types": [
            { "id": 1, "name": "正社員", "count": 70 }
        ]
    }
}
```

### GET `/api/v1/jobs/{id}` - 求人詳細

**Response (200):**
```json
{
    "data": {
        "id": 1,
        "title": "【東京】税理士事務所スタッフ募集",
        "description": "当事務所では...",
        "requirement": "税理士資格保有者...",
        "welcome_skill": "相続税の実務経験...",
        "company": {
            "id": 10,
            "name": "山田税理士事務所",
            "logo_url": "https://...",
            "profile": {
                "description": "創業30年の...",
                "industry": "税理士事務所",
                "employee_count": "11-50名",
                "founded_year": "1996"
            }
        },
        "professions": [...],
        "employment_type": { "id": 1, "name": "正社員" },
        "salary_type": { "id": 1, "name": "年俸" },
        "salary_min": 4000000,
        "salary_max": 6000000,
        "salary_display": "400万円〜600万円",
        "prefecture": { "id": 13, "name": "東京都" },
        "work_location": "東京都千代田区丸の内1-1-1",
        "nearest_station": "東京駅 徒歩5分",
        "work_hours": "9:00〜18:00（実働8時間）",
        "benefits": "社会保険完備、交通費全額支給...",
        "selection_process": "書類選考 → 一次面接 → 最終面接",
        "remote_type": 1,
        "remote_type_label": "一部リモート",
        "skills": [...],
        "is_favorited": false,
        "has_applied": false,
        "view_count": 150,
        "published_at": "2026-02-15",
        "closed_at": "2026-03-31"
    }
}
```

---

## 4. お気に入り API

| メソッド | エンドポイント | 説明 | 認証 |
|---------|---------------|------|------|
| GET | `/api/v1/favorites` | お気に入り一覧 | 必要 |
| POST | `/api/v1/jobs/{id}/favorite` | お気に入り追加 | 必要 |
| DELETE | `/api/v1/jobs/{id}/favorite` | お気に入り削除 | 必要 |

---

## 5. 応募 API

| メソッド | エンドポイント | 説明 | 認証 |
|---------|---------------|------|------|
| GET | `/api/v1/applications` | 応募一覧（求職者） | 必要 |
| POST | `/api/v1/jobs/{id}/apply` | 求人に応募 | 必要 |
| GET | `/api/v1/applications/{id}` | 応募詳細 | 必要 |

### POST `/api/v1/jobs/{id}/apply` - 求人に応募

**Request Body:**
```json
{
    "message": "貴事務所の理念に共感し、応募いたしました。"
}
```

**Response (201):**
```json
{
    "data": {
        "id": 1,
        "job": {
            "id": 1,
            "title": "【東京】税理士事務所スタッフ募集"
        },
        "status": 1,
        "status_label": "応募済み",
        "message": "貴事務所の理念に共感し、応募いたしました。",
        "applied_at": "2026-02-18T10:00:00Z"
    },
    "message": "応募が完了しました。"
}
```

**Error (409 - 二重応募):**
```json
{
    "message": "この求人にはすでに応募済みです。"
}
```

---

## 6. プロフィール API

| メソッド | エンドポイント | 説明 | 認証 |
|---------|---------------|------|------|
| GET | `/api/v1/profile` | プロフィール取得 | 必要 |
| PUT | `/api/v1/profile` | プロフィール更新 | 必要 |
| POST | `/api/v1/profile/avatar` | アバター画像アップロード | 必要 |
| DELETE | `/api/v1/profile/avatar` | アバター画像削除 | 必要 |
| PUT | `/api/v1/profile/skills` | スキル更新 | 必要 |
| PUT | `/api/v1/profile/licenses` | 保有資格更新 | 必要 |

### PUT `/api/v1/profile` - プロフィール更新

**Request Body:**
```json
{
    "last_name": "山田",
    "first_name": "太郎",
    "last_name_kana": "ヤマダ",
    "first_name_kana": "タロウ",
    "birthday": "1990-01-15",
    "gender": 1,
    "prefecture_id": 13,
    "profession_id": 1,
    "self_introduction": "税理士として5年の経験があります。",
    "experience_years": 5,
    "desired_salary_min": 5000000,
    "desired_salary_max": 7000000,
    "desired_prefecture_id": 13,
    "job_search_status": 1
}
```

---

## 7. 企業管理 API

### 7.1 求人管理

| メソッド | エンドポイント | 説明 | 認証 |
|---------|---------------|------|------|
| GET | `/api/v1/company/jobs` | 自社求人一覧 | 企業 |
| POST | `/api/v1/company/jobs` | 求人作成 | 企業 |
| GET | `/api/v1/company/jobs/{id}` | 求人詳細 | 企業 |
| PUT | `/api/v1/company/jobs/{id}` | 求人更新 | 企業 |
| PATCH | `/api/v1/company/jobs/{id}/status` | ステータス変更 | 企業 |
| DELETE | `/api/v1/company/jobs/{id}` | 求人削除（ソフト） | 企業 |

### POST `/api/v1/company/jobs` - 求人作成

**Request Body:**
```json
{
    "title": "【東京】税理士事務所スタッフ募集",
    "description": "当事務所では...",
    "requirement": "税理士資格保有者...",
    "welcome_skill": "相続税の実務経験...",
    "employment_type_id": 1,
    "salary_type_id": 1,
    "salary_min": 4000000,
    "salary_max": 6000000,
    "prefecture_id": 13,
    "work_location": "東京都千代田区丸の内1-1-1",
    "nearest_station": "東京駅 徒歩5分",
    "work_hours": "9:00〜18:00",
    "benefits": "社会保険完備...",
    "selection_process": "書類選考 → 面接",
    "remote_type": 1,
    "profession_ids": [1, 2],
    "skill_ids": [1, 5, 8],
    "status": 1,
    "closed_at": "2026-03-31"
}
```

### 7.2 応募者管理

| メソッド | エンドポイント | 説明 | 認証 |
|---------|---------------|------|------|
| GET | `/api/v1/company/applications` | 自社応募一覧 | 企業 |
| GET | `/api/v1/company/applications/{id}` | 応募詳細 | 企業 |
| PATCH | `/api/v1/company/applications/{id}/status` | ステータス更新 | 企業 |

### PATCH `/api/v1/company/applications/{id}/status` - ステータス更新

**Request Body:**
```json
{
    "status": 2,
    "note": "書類選考を開始します。"
}
```

### 7.3 企業プロフィール

| メソッド | エンドポイント | 説明 | 認証 |
|---------|---------------|------|------|
| GET | `/api/v1/company/profile` | 企業プロフィール取得 | 企業 |
| PUT | `/api/v1/company/profile` | 企業プロフィール更新 | 企業 |
| POST | `/api/v1/company/profile/logo` | ロゴアップロード | 企業 |

### 7.4 ダッシュボード

| メソッド | エンドポイント | 説明 | 認証 |
|---------|---------------|------|------|
| GET | `/api/v1/company/dashboard` | ダッシュボードデータ | 企業 |

**Response (200):**
```json
{
    "data": {
        "active_jobs_count": 5,
        "total_applications_count": 23,
        "new_applications_count": 3,
        "this_month_views": 450,
        "recent_applications": [...],
        "job_stats": [
            {
                "job_id": 1,
                "title": "税理士スタッフ募集",
                "applications_count": 8,
                "views_count": 120
            }
        ]
    }
}
```

---

## 8. 管理者 API

### 8.1 ダッシュボード

| メソッド | エンドポイント | 説明 | 認証 |
|---------|---------------|------|------|
| GET | `/api/v1/admin/dashboard` | KPIダッシュボード | 管理者 |

### 8.2 企業管理

| メソッド | エンドポイント | 説明 | 認証 |
|---------|---------------|------|------|
| GET | `/api/v1/admin/companies` | 企業一覧 | 管理者 |
| GET | `/api/v1/admin/companies/{id}` | 企業詳細 | 管理者 |
| PATCH | `/api/v1/admin/companies/{id}/verify` | 企業審査（承認/却下） | 管理者 |
| PATCH | `/api/v1/admin/companies/{id}/status` | ステータス変更 | 管理者 |

### 8.3 求人管理

| メソッド | エンドポイント | 説明 | 認証 |
|---------|---------------|------|------|
| GET | `/api/v1/admin/jobs` | 全求人一覧 | 管理者 |
| GET | `/api/v1/admin/jobs/{id}` | 求人詳細 | 管理者 |
| PATCH | `/api/v1/admin/jobs/{id}/status` | ステータス変更 | 管理者 |

### 8.4 ユーザー管理

| メソッド | エンドポイント | 説明 | 認証 |
|---------|---------------|------|------|
| GET | `/api/v1/admin/users` | ユーザー一覧 | 管理者 |
| GET | `/api/v1/admin/users/{id}` | ユーザー詳細 | 管理者 |
| PATCH | `/api/v1/admin/users/{id}/status` | ステータス変更 | 管理者 |

### 8.5 記事管理

| メソッド | エンドポイント | 説明 | 認証 |
|---------|---------------|------|------|
| GET | `/api/v1/admin/articles` | 記事一覧 | 管理者 |
| POST | `/api/v1/admin/articles` | 記事作成 | 管理者 |
| GET | `/api/v1/admin/articles/{id}` | 記事詳細 | 管理者 |
| PUT | `/api/v1/admin/articles/{id}` | 記事更新 | 管理者 |
| DELETE | `/api/v1/admin/articles/{id}` | 記事削除 | 管理者 |

---

## 9. マスターデータ API

| メソッド | エンドポイント | 説明 | 認証 |
|---------|---------------|------|------|
| GET | `/api/v1/master/professions` | 士業種別一覧 | 不要 |
| GET | `/api/v1/master/prefectures` | 都道府県一覧 | 不要 |
| GET | `/api/v1/master/employment-types` | 雇用形態一覧 | 不要 |
| GET | `/api/v1/master/salary-types` | 給与形態一覧 | 不要 |
| GET | `/api/v1/master/skills` | スキル一覧 | 不要 |
| GET | `/api/v1/master/skills?profession_id=1` | 職種別スキル | 不要 |
| GET | `/api/v1/master/licenses` | 資格一覧 | 不要 |

---

## 10. コンテンツ API（公開）

| メソッド | エンドポイント | 説明 | 認証 |
|---------|---------------|------|------|
| GET | `/api/v1/articles` | 記事一覧 | 不要 |
| GET | `/api/v1/articles/{slug}` | 記事詳細 | 不要 |
| GET | `/api/v1/articles/categories` | カテゴリ一覧 | 不要 |
| GET | `/api/v1/articles/tags` | タグ一覧 | 不要 |

---

## 11. 企業公開 API

| メソッド | エンドポイント | 説明 | 認証 |
|---------|---------------|------|------|
| GET | `/api/v1/companies/{id}` | 企業詳細（公開） | 不要 |
| GET | `/api/v1/companies/{id}/jobs` | 企業の公開求人一覧 | 不要 |

---

## 12. ルーティング実装

```php
// routes/api.php

use Illuminate\Support\Facades\Route;

// CSRF Cookie
Route::get('/sanctum/csrf-cookie', ...);

Route::prefix('v1')->group(function () {

    // === 公開 API ===
    Route::prefix('auth')->group(function () {
        Route::post('/register', [AuthController::class, 'register']);
        Route::post('/login', [AuthController::class, 'login']);
        Route::post('/forgot-password', [AuthController::class, 'forgotPassword']);
        Route::post('/reset-password', [AuthController::class, 'resetPassword']);
    });

    // マスターデータ
    Route::prefix('master')->group(function () {
        Route::get('/professions', [MasterController::class, 'professions']);
        Route::get('/prefectures', [MasterController::class, 'prefectures']);
        Route::get('/employment-types', [MasterController::class, 'employmentTypes']);
        Route::get('/salary-types', [MasterController::class, 'salaryTypes']);
        Route::get('/skills', [MasterController::class, 'skills']);
        Route::get('/licenses', [MasterController::class, 'licenses']);
    });

    // 求人（公開）
    Route::get('/jobs', [JobController::class, 'index']);
    Route::get('/jobs/{job}', [JobController::class, 'show']);
    Route::get('/jobs/{job}/related', [JobController::class, 'related']);

    // 企業（公開）
    Route::get('/companies/{company}', [CompanyController::class, 'show']);
    Route::get('/companies/{company}/jobs', [CompanyController::class, 'jobs']);

    // 記事（公開）
    Route::get('/articles', [ArticleController::class, 'index']);
    Route::get('/articles/categories', [ArticleController::class, 'categories']);
    Route::get('/articles/{article:slug}', [ArticleController::class, 'show']);

    // === 求職者認証済み API ===
    Route::middleware('auth:sanctum')->group(function () {
        Route::post('/auth/logout', [AuthController::class, 'logout']);
        Route::get('/auth/user', [AuthController::class, 'user']);

        // プロフィール
        Route::get('/profile', [UserProfileController::class, 'show']);
        Route::put('/profile', [UserProfileController::class, 'update']);
        Route::post('/profile/avatar', [UserProfileController::class, 'uploadAvatar']);
        Route::delete('/profile/avatar', [UserProfileController::class, 'deleteAvatar']);
        Route::put('/profile/skills', [UserProfileController::class, 'updateSkills']);
        Route::put('/profile/licenses', [UserProfileController::class, 'updateLicenses']);

        // お気に入り
        Route::get('/favorites', [FavoriteController::class, 'index']);
        Route::post('/jobs/{job}/favorite', [FavoriteController::class, 'store']);
        Route::delete('/jobs/{job}/favorite', [FavoriteController::class, 'destroy']);

        // 応募
        Route::get('/applications', [ApplicationController::class, 'index']);
        Route::post('/jobs/{job}/apply', [ApplicationController::class, 'store']);
        Route::get('/applications/{application}', [ApplicationController::class, 'show']);
    });

    // === 企業側 API ===
    Route::prefix('company')->group(function () {
        Route::post('/auth/register', [CompanyAuthController::class, 'register']);
        Route::post('/auth/login', [CompanyAuthController::class, 'login']);

        Route::middleware('auth:company')->group(function () {
            Route::post('/auth/logout', [CompanyAuthController::class, 'logout']);
            Route::get('/auth/user', [CompanyAuthController::class, 'user']);

            Route::get('/dashboard', [CompanyDashboardController::class, 'index']);

            Route::get('/profile', [CompanyProfileController::class, 'show']);
            Route::put('/profile', [CompanyProfileController::class, 'update']);
            Route::post('/profile/logo', [CompanyProfileController::class, 'uploadLogo']);

            // 求人管理（審査通過企業のみ）
            Route::middleware('company.verified')->group(function () {
                Route::apiResource('jobs', CompanyJobController::class);
                Route::patch('/jobs/{job}/status', [CompanyJobController::class, 'updateStatus']);
            });

            // 応募者管理
            Route::get('/applications', [CompanyApplicationController::class, 'index']);
            Route::get('/applications/{application}', [CompanyApplicationController::class, 'show']);
            Route::patch('/applications/{application}/status', [CompanyApplicationController::class, 'updateStatus']);
        });
    });

    // === 管理者 API ===
    Route::prefix('admin')->group(function () {
        Route::post('/auth/login', [AdminAuthController::class, 'login']);

        Route::middleware('auth:admin')->group(function () {
            Route::post('/auth/logout', [AdminAuthController::class, 'logout']);
            Route::get('/auth/user', [AdminAuthController::class, 'user']);

            Route::get('/dashboard', [AdminDashboardController::class, 'index']);

            Route::get('/companies', [AdminCompanyController::class, 'index']);
            Route::get('/companies/{company}', [AdminCompanyController::class, 'show']);
            Route::patch('/companies/{company}/verify', [AdminCompanyController::class, 'verify']);
            Route::patch('/companies/{company}/status', [AdminCompanyController::class, 'updateStatus']);

            Route::get('/jobs', [AdminJobController::class, 'index']);
            Route::get('/jobs/{job}', [AdminJobController::class, 'show']);
            Route::patch('/jobs/{job}/status', [AdminJobController::class, 'updateStatus']);

            Route::get('/users', [AdminUserController::class, 'index']);
            Route::get('/users/{user}', [AdminUserController::class, 'show']);
            Route::patch('/users/{user}/status', [AdminUserController::class, 'updateStatus']);

            Route::apiResource('articles', AdminArticleController::class);
        });
    });
});
```
