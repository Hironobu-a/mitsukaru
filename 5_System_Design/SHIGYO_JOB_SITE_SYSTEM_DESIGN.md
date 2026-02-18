# 士業向け求人サイト システム設計書

**作成日**: 2026年2月18日  
**対象**: 開発チーム全員  
**技術スタック想定**: Laravel 11 / PHP 8.3 / MySQL 8.0 / Docker

---

## 概要

本書は、ミツカルが展開する**士業向け求人サイト（ミツプロ拡張版）**の設計方針をまとめたものです。  
対象職種は税理士に限らず、弁護士・社会保険労務士・公認会計士・司法書士 等の士業全般への拡張を前提とした**モダンな Laravel アーキテクチャ**で設計します。

既存の `tax.mitsukaru-pro.co.jp`（Laravel 8）の技術的負債（Fat Controller、テスト不足、フロントend混在）を踏まえ、  
**初期設計の段階から責務分離・テスタビリティ・拡張性**を意識した構成を採用します。

---

## 1. Migration 設計

### 1.1 設計方針

- テーブル名は**複数形スネークケース**（Laravel 規約準拠）
- 外部キーには必ず `constrained()` + `cascadeOnDelete()` / `restrictOnDelete()` を明示
- `deleted_at` による**ソフトデリート**は、ユーザー・求人・企業など個人情報を持つテーブルに適用
- `ENUM` は使用禁止。代わりに**マスターテーブル**または `tinyint` + 定数クラスで管理（変更容易性のため）
- インデックスは「検索条件に使われるカラム」「外部キー」に必ず付与
- Seeder と Factory はテーブルと同時に必ず作成する

---

### 1.2 テーブル設計一覧

#### 【マスター系】

```
professions          - 士業種別マスター (税理士, 弁護士, 社労士, etc.)
prefectures          - 都道府県マスター
employment_types     - 雇用形態マスター (正社員, 契約社員, パート, etc.)
salary_types         - 給与形態マスター (月給, 年俸, 時給)
skills               - スキルマスター (各士業別スキルタグ)
```

#### 【ユーザー系】

```
users                - 求職者
user_profiles        - 求職者プロフィール (1:1)
user_skills          - 求職者スキル (中間)
user_licenses        - 保有資格 (中間)
```

#### 【企業 / 採用側】

```
companies            - 採用企業・事務所
company_profiles     - 企業プロフィール (1:1)
company_users        - 企業担当者アカウント
```

#### 【求人系】

```
jobs                 - 求人票
job_skills           - 求人に紐づくスキルタグ (中間)
job_professions      - 求人対象職種 (中間)
job_favorites        - お気に入り (中間)
job_applications     - 応募
application_statuses - 選考ステータス履歴
```

#### 【コンテンツ / SEO】

```
articles             - 転職ノウハウ記事
categories           - 記事カテゴリ
tags                 - タグ
article_tags         - 記事タグ (中間)
```

#### 【管理者】

```
admin_users          - 管理者
```

---

### 1.3 主要テーブル詳細

#### `professions`（士業種別マスター）

```php
Schema::create('professions', function (Blueprint $table) {
    $table->id();
    $table->string('name');           // 税理士, 弁護士, etc.
    $table->string('slug')->unique(); // tax-accountant, lawyer, etc.
    $table->integer('sort_order')->default(0);
    $table->timestamps();
});
```

---

#### `users`（求職者）

```php
Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('email')->unique();
    $table->string('password');
    $table->string('last_name');
    $table->string('first_name');
    $table->string('last_name_kana')->nullable();
    $table->string('first_name_kana')->nullable();
    $table->date('birthday')->nullable();
    $table->tinyInteger('gender')->nullable(); // 0:未回答 1:男性 2:女性
    $table->foreignId('prefecture_id')->nullable()->constrained()->nullOnDelete();
    $table->tinyInteger('status')->default(1); // 1:active 2:pause 9:退会
    $table->timestamp('email_verified_at')->nullable();
    $table->rememberToken();
    $table->softDeletes();
    $table->timestamps();

    $table->index('status');
    $table->index('prefecture_id');
});
```

---

#### `companies`（採用企業）

```php
Schema::create('companies', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('name_kana')->nullable();
    $table->foreignId('prefecture_id')->nullable()->constrained()->nullOnDelete();
    $table->string('address')->nullable();
    $table->string('phone')->nullable();
    $table->string('email')->unique();
    $table->tinyInteger('status')->default(1); // 1:審査中 2:掲載中 9:停止
    $table->tinyInteger('is_verified')->default(0); // 審査通過フラグ
    $table->softDeletes();
    $table->timestamps();

    $table->index(['status', 'is_verified']);
});
```

---

#### `jobs`（求人票）

```php
Schema::create('jobs', function (Blueprint $table) {
    $table->id();
    $table->foreignId('company_id')->constrained()->cascadeOnDelete();
    $table->string('title');
    $table->text('description');
    $table->text('requirement')->nullable();          // 応募要件
    $table->text('welcome_skill')->nullable();        // 歓迎スキル
    $table->foreignId('employment_type_id')->constrained()->restrictOnDelete();
    $table->foreignId('salary_type_id')->constrained()->restrictOnDelete();
    $table->unsignedInteger('salary_min')->nullable();
    $table->unsignedInteger('salary_max')->nullable();
    $table->foreignId('prefecture_id')->nullable()->constrained()->nullOnDelete();
    $table->string('work_location')->nullable();      // 詳細勤務地
    $table->tinyInteger('status')->default(1); // 1:下書き 2:公開 3:非公開 9:削除
    $table->date('published_at')->nullable();
    $table->date('closed_at')->nullable();
    $table->softDeletes();
    $table->timestamps();

    $table->index(['status', 'published_at']);
    $table->index(['company_id', 'status']);
    $table->index('prefecture_id');
    $table->index(['salary_min', 'salary_max']); // 年収検索用
});
```

---

#### `job_applications`（応募）

```php
Schema::create('job_applications', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained()->restrictOnDelete();
    $table->foreignId('job_id')->constrained()->restrictOnDelete();
    $table->tinyInteger('status')->default(1);
    // 1:応募済み 2:書類選考中 3:面接調整 4:内定 5:辞退 9:不採用
    $table->text('message')->nullable();    // 応募メッセージ
    $table->timestamp('applied_at')->useCurrent();
    $table->timestamps();

    // 同一求人への二重応募防止
    $table->unique(['user_id', 'job_id']);
    $table->index(['user_id', 'status']);
    $table->index(['job_id', 'status']);
});
```

---

#### `job_favorites`（お気に入り）

```php
Schema::create('job_favorites', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained()->cascadeOnDelete();
    $table->foreignId('job_id')->constrained()->cascadeOnDelete();
    $table->timestamps();

    $table->unique(['user_id', 'job_id']);
    $table->index('user_id');
});
```

---

#### `articles`（SEOコンテンツ記事）

```php
Schema::create('articles', function (Blueprint $table) {
    $table->id();
    $table->string('title');
    $table->string('slug')->unique();
    $table->text('body');
    $table->string('meta_description')->nullable();
    $table->foreignId('category_id')->nullable()->constrained()->nullOnDelete();
    $table->foreignId('profession_id')->nullable()->constrained()->nullOnDelete();
    $table->tinyInteger('status')->default(1); // 1:下書き 2:公開
    $table->timestamp('published_at')->nullable();
    $table->timestamps();

    $table->index(['status', 'published_at']);
    $table->index('slug');
});
```

---

### 1.4 Migration ファイルの命名規則

```
2026_02_18_000001_create_professions_table.php
2026_02_18_000002_create_prefectures_table.php
2026_02_18_000003_create_employment_types_table.php
2026_02_18_000004_create_salary_types_table.php
2026_02_18_000005_create_users_table.php
2026_02_18_000006_create_user_profiles_table.php
...
```

> マスターテーブル → 親テーブル → 子テーブル の順に作成し、外部キーエラーを防ぐ。

---

## 2. FormRequest での Validation 方針

### 2.1 基本方針

- **全ての入力はコントローラーに触れる前に FormRequest で検証する**
- Controller に `$request->validated()` 以外の入力参照を禁止する
- 1アクション = 1 FormRequest（`StoreJobRequest` / `UpdateJobRequest` のように分割）
- `authorize()` には **Policy** を使用し、認可もここで完結させる

---

### 2.2 ディレクトリ構成

```
app/Http/Requests/
├── Auth/
│   ├── LoginRequest.php
│   └── RegisterRequest.php
├── Job/
│   ├── StoreJobRequest.php
│   ├── UpdateJobRequest.php
│   └── SearchJobRequest.php
├── Application/
│   └── StoreApplicationRequest.php
├── Company/
│   ├── StoreCompanyRequest.php
│   └── UpdateCompanyRequest.php
└── Article/
    ├── StoreArticleRequest.php
    └── UpdateArticleRequest.php
```

---

### 2.3 実装例

#### `StoreJobRequest`（求人登録）

```php
<?php

namespace App\Http\Requests\Job;

use Illuminate\Foundation\Http\FormRequest;
use App\Models\Job;

class StoreJobRequest extends FormRequest
{
    public function authorize(): bool
    {
        // 認証済み企業ユーザーのみ投稿可能
        return $this->user() instanceof \App\Models\CompanyUser
            && $this->user()->company->is_verified;
    }

    public function rules(): array
    {
        return [
            'title'              => ['required', 'string', 'max:100'],
            'description'        => ['required', 'string', 'max:10000'],
            'requirement'        => ['nullable', 'string', 'max:5000'],
            'welcome_skill'      => ['nullable', 'string', 'max:5000'],
            'employment_type_id' => ['required', 'integer', 'exists:employment_types,id'],
            'salary_type_id'     => ['required', 'integer', 'exists:salary_types,id'],
            'salary_min'         => ['nullable', 'integer', 'min:0', 'max:99999999'],
            'salary_max'         => ['nullable', 'integer', 'min:0', 'max:99999999', 'gte:salary_min'],
            'prefecture_id'      => ['nullable', 'integer', 'exists:prefectures,id'],
            'work_location'      => ['nullable', 'string', 'max:200'],
            'profession_ids'     => ['required', 'array', 'min:1'],
            'profession_ids.*'   => ['integer', 'exists:professions,id'],
            'skill_ids'          => ['nullable', 'array'],
            'skill_ids.*'        => ['integer', 'exists:skills,id'],
            'status'             => ['required', 'integer', 'in:1,2'],  // 1:下書き 2:公開
            'closed_at'          => ['nullable', 'date', 'after:today'],
        ];
    }

    public function messages(): array
    {
        return [
            'salary_max.gte'       => '上限年収は下限年収以上の値を入力してください。',
            'profession_ids.min'   => '対象職種を1つ以上選択してください。',
            'closed_at.after'      => '掲載終了日は明日以降を指定してください。',
        ];
    }

    public function attributes(): array
    {
        return [
            'title'              => '求人タイトル',
            'description'        => '仕事内容',
            'employment_type_id' => '雇用形態',
            'salary_type_id'     => '給与形態',
            'salary_min'         => '下限年収',
            'salary_max'         => '上限年収',
            'prefecture_id'      => '勤務都道府県',
            'profession_ids'     => '対象職種',
        ];
    }
}
```

---

#### `SearchJobRequest`（求人検索）

```php
<?php

namespace App\Http\Requests\Job;

use Illuminate\Foundation\Http\FormRequest;

class SearchJobRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true; // 検索は誰でも可
    }

    public function rules(): array
    {
        return [
            'keyword'             => ['nullable', 'string', 'max:100'],
            'profession_ids'      => ['nullable', 'array'],
            'profession_ids.*'    => ['integer', 'exists:professions,id'],
            'prefecture_id'       => ['nullable', 'integer', 'exists:prefectures,id'],
            'employment_type_ids' => ['nullable', 'array'],
            'employment_type_ids.*'=> ['integer', 'exists:employment_types,id'],
            'salary_min'          => ['nullable', 'integer', 'min:0'],
            'salary_max'          => ['nullable', 'integer', 'min:0'],
            'sort'                => ['nullable', 'string', 'in:newest,salary_high,salary_low'],
            'page'                => ['nullable', 'integer', 'min:1'],
        ];
    }
}
```

---

### 2.4 カスタムルールの使い所

複数フィールドにまたがる検証や、DBを参照する複雑なルールは `app/Rules/` に独立させる。

| ルールクラス例 | 用途 |
|---|---|
| `UniqueApplicationRule` | 同一求人への二重応募チェック |
| `WithinSalaryRangeRule` | 求職者希望年収の妥当性チェック |
| `ValidLicenseRule` | 指定職種に対応した資格コードの検証 |

---

### 2.5 Controller 側の書き方

```php
// ❌ バリデーションをコントローラーで書かない
public function store(Request $request)
{
    $request->validate([...]);
}

// ✅ FormRequest に分離する
public function store(StoreJobRequest $request): RedirectResponse
{
    $job = $this->jobService->create(
        $request->user()->company,
        $request->validated()
    );

    return redirect()->route('jobs.show', $job);
}
```

---

## 3. Service 層は必要か？ その理由

### 3.1 結論：**必要。ただし薄く保つ**

本プロジェクトでは **Service 層を導入する**。  
ただし「すべてのビジネスロジックを Service に集める」のではなく、  
**Controller が肥大化しそうな箇所、または複数箇所から呼び出すロジック**に限定して使う。

---

### 3.2 導入する理由

#### ① 現行コードの反省（Fat Controller の教訓）

既存の `tax.mitsukaru-pro.co.jp` は Controller にビジネスロジックが書かれており、  
`CaffeineController` や `UserDataController` のテスト不足が今なお技術的負債になっている。  
Service に切り出すことで、**Controller を薄く保ちテストを書きやすくする**。

#### ② 複数のエントリーポイントからの再利用

例えば「応募作成」ロジックは以下の場所から呼ばれる可能性がある：

- `ApplicationController`（Web フォームからの応募）
- `Api\ApplicationController`（SPAやモバイルAPIからの応募）
- `Console\Commands\ImportApplicationCommand`（CSVインポート処理）

Controller に書いてしまうと複製が生まれるが、Service に切り出せば**一元管理できる**。

#### ③ テスタビリティの向上

Service クラスは HTTP レイヤー（Request / Response）に依存しないため、  
**単体テストで HTTP を介さずにビジネスロジックを検証できる**。

```php
// Controller を介さずに Service を直接テスト可能
it('応募を作成できる', function () {
    $user = User::factory()->create();
    $job  = Job::factory()->published()->create();

    $application = app(ApplicationService::class)->create($user, $job, ['message' => 'よろしくお願いします。']);

    expect($application)->toBeInstanceOf(JobApplication::class)
        ->and($application->user_id)->toBe($user->id);
});
```

---

### 3.3 Service を**使わない**ケース

| ケース | 理由 |
|---|---|
| シンプルな CRUD（一行で完結） | Controller + Eloquent で十分。Service に入れると過設計 |
| Model のスコープで完結する検索 | `Job::published()->search($params)` のようにスコープに書く |
| 単純なデータ取得 | Repository パターンは今フェーズでは過設計。Eloquent を直接使う |

---

### 3.4 ディレクトリ構成と命名規則

```
app/Services/
├── Job/
│   ├── JobCreateService.php     // 求人作成（複雑: 職種・スキルの紐付けを含む）
│   ├── JobSearchService.php     // 求人検索（複雑: 複数条件フィルタ + ソート）
│   └── JobStatusService.php     // 求人ステータス変更（公開/非公開/削除）
├── Application/
│   └── ApplicationService.php   // 応募作成・ステータス更新
├── User/
│   └── UserRegistrationService.php // 会員登録（メール送信含む）
└── Company/
    └── CompanyVerificationService.php // 企業審査フロー
```

---

### 3.5 Service の実装例

#### `ApplicationService`（応募作成）

```php
<?php

namespace App\Services\Application;

use App\Models\Job;
use App\Models\JobApplication;
use App\Models\User;
use App\Notifications\ApplicationReceivedNotification;
use Illuminate\Support\Facades\DB;

class ApplicationService
{
    /**
     * 求人に応募する
     *
     * @throws \RuntimeException 二重応募の場合
     */
    public function create(User $user, Job $job, array $data): JobApplication
    {
        return DB::transaction(function () use ($user, $job, $data) {
            if ($this->alreadyApplied($user, $job)) {
                throw new \RuntimeException('この求人にはすでに応募済みです。');
            }

            $application = JobApplication::create([
                'user_id' => $user->id,
                'job_id'  => $job->id,
                'message' => $data['message'] ?? null,
                'status'  => JobApplication::STATUS_APPLIED,
            ]);

            // 企業担当者へ通知
            $job->company->notifyUsers(new ApplicationReceivedNotification($application));

            return $application;
        });
    }

    private function alreadyApplied(User $user, Job $job): bool
    {
        return JobApplication::where('user_id', $user->id)
            ->where('job_id', $job->id)
            ->exists();
    }
}
```

---

### 3.6 アーキテクチャ全体像

```
HTTP Request
    │
    ▼
FormRequest（認可 + バリデーション）
    │
    ▼
Controller（ルーティング・レスポンス整形のみ）
    │
    ▼
Service（ビジネスロジック / トランザクション）
    │
    ▼
Model / Eloquent（DB操作）
```

---

## 4. Feature テストで何を検証するか

### 4.1 基本方針

Feature テストは**HTTPリクエストを起点にしたシナリオテスト**。  
ユーザーが実際に操作する流れを API レベルで検証し、**ユニットテストでは検証できない統合動作を担保**する。

- 使用フレームワーク: **PHPUnit（Pest）**
- DB: `RefreshDatabase` トレイト使用（テストごとにロールバック）
- テストデータ: **Factory** でのみ生成。DBシーダーに依存しない

---

### 4.2 検証する範囲（優先度別）

#### 🔴 優先度：HIGH（必ず書く）

| テスト対象 | 検証내容 |
|---|---|
| 会員登録 | 正常登録、メール重複エラー、バリデーションエラー |
| ログイン / ログアウト | 正常ログイン、誤パスワード、未認証リダイレクト |
| 求人一覧・検索 | キーワード検索、職種フィルタ、都道府県フィルタ、ソート切り替え |
| 求人詳細表示 | 公開中求人は表示、非公開求人は 404 |
| 求人応募 | 正常応募、二重応募エラー、未認証リダイレクト |
| 求人お気に入り | 追加・削除のトグル、認証必須 |
| 企業による求人投稿 | 正常投稿、審査未通過企業は 403 |
| 企業による求人ステータス変更 | 他社求人の変更は 403 |

---

#### 🟡 優先度：MEDIUM（できる限り書く）

| テスト対象 | 検証内容 |
|---|---|
| プロフィール編集 | 正常更新、他ユーザーのプロフィールは編集不可 |
| 応募一覧（求職者） | 自分の応募のみ表示される |
| 応募ステータス更新（企業側） | 自社求人のみ変更可能 |
| 記事一覧・詳細 | 公開記事のみ表示れる、下書きは 404 |
| ページネーション | 正しい件数・ページ切り替え |
| メール送信 | `Mail::fake()` で送信イベントを確認 |

---

#### 🟢 優先度：LOW（余力があれば）

| テスト対象 | 検証内容 |
|---|---|
| 管理者機能 | 求人・ユーザーの CRUD |
| 診断ツール | 入力 → 結果の計算ロジック |
| SEO メタタグ | レスポンスに `<meta>` タグが含まれるか |

---

### 4.3 実装例（Pest 記法）

#### 求人応募のテスト

```php
<?php

use App\Models\Job;
use App\Models\JobApplication;
use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;

uses(RefreshDatabase::class);

describe('求人応募', function () {

    it('ログイン済みユーザーは公開中の求人に応募できる', function () {
        $user = User::factory()->create();
        $job  = Job::factory()->published()->create();

        $response = $this->actingAs($user)
            ->postJson(route('jobs.applications.store', $job), [
                'message' => 'よろしくお願いします。',
            ]);

        $response->assertCreated();
        $this->assertDatabaseHas('job_applications', [
            'user_id' => $user->id,
            'job_id'  => $job->id,
        ]);
    });

    it('未ログインユーザーは応募できず 401 が返る', function () {
        $job = Job::factory()->published()->create();

        $this->postJson(route('jobs.applications.store', $job))
            ->assertUnauthorized();
    });

    it('同一求人への二重応募はエラーになる', function () {
        $user = User::factory()->create();
        $job  = Job::factory()->published()->create();

        JobApplication::factory()->create([
            'user_id' => $user->id,
            'job_id'  => $job->id,
        ]);

        $this->actingAs($user)
            ->postJson(route('jobs.applications.store', $job))
            ->assertUnprocessable(); // 422

        $this->assertDatabaseCount('job_applications', 1);
    });

    it('非公開の求人には応募できず 404 が返る', function () {
        $user = User::factory()->create();
        $job  = Job::factory()->closed()->create();

        $this->actingAs($user)
            ->postJson(route('jobs.applications.store', $job))
            ->assertNotFound();
    });
});
```

---

#### 求人検索のテスト

```php
describe('求人検索', function () {

    it('キーワードで求人を絞り込める', function () {
        Job::factory()->published()->create(['title' => '東京の税理士事務所求人']);
        Job::factory()->published()->create(['title' => '大阪の会計事務所求人']);
        Job::factory()->closed()->create(['title' => '東京の非公開求人']);

        $response = $this->getJson(route('jobs.index', ['keyword' => '東京']));

        $response->assertOk()
            ->assertJsonCount(1, 'data'); // 公開中の1件のみ
    });

    it('職種IDで求人を絞り込める', function () {
        $taxAccountant = Profession::factory()->create(['slug' => 'tax-accountant']);
        $lawyer        = Profession::factory()->create(['slug' => 'lawyer']);

        $taxJob    = Job::factory()->published()->hasAttached($taxAccountant, [], 'professions')->create();
        $lawyerJob = Job::factory()->published()->hasAttached($lawyer, [], 'professions')->create();

        $this->getJson(route('jobs.index', ['profession_ids' => [$taxAccountant->id]]))
            ->assertOk()
            ->assertJsonFragment(['id' => $taxJob->id])
            ->assertJsonMissing(['id' => $lawyerJob->id]);
    });

    it('年収範囲で求人を絞り込める', function () {
        Job::factory()->published()->create(['salary_min' => 4000000, 'salary_max' => 6000000]);
        Job::factory()->published()->create(['salary_min' => 7000000, 'salary_max' => 10000000]);

        $this->getJson(route('jobs.index', ['salary_min' => 6000000]))
            ->assertOk()
            ->assertJsonCount(1, 'data');
    });
});
```

---

#### 認可のテスト（企業による求人操作）

```php
describe('求人管理 - 企業担当者の認可', function () {

    it('自社の求人は編集できる', function () {
        $companyUser = CompanyUser::factory()->create();
        $job = Job::factory()->for($companyUser->company)->create();

        $this->actingAs($companyUser, 'company')
            ->putJson(route('company.jobs.update', $job), ['title' => '更新タイトル'])
            ->assertOk();
    });

    it('他社の求人は編集できず 403 が返る', function () {
        $companyUser = CompanyUser::factory()->create();
        $otherJob    = Job::factory()->create(); // 別会社の求人

        $this->actingAs($companyUser, 'company')
            ->putJson(route('company.jobs.update', $otherJob), ['title' => '不正更新'])
            ->assertForbidden();
    });

    it('審査未通過企業の担当者は求人を投稿できず 403 が返る', function () {
        $unverifiedUser = CompanyUser::factory()
            ->for(Company::factory()->unverified())
            ->create();

        $this->actingAs($unverifiedUser, 'company')
            ->postJson(route('company.jobs.store'), Job::factory()->make()->toArray())
            ->assertForbidden();
    });
});
```

---

### 4.4 テストの実行方法

```bash
# 全テスト実行
./vendor/bin/pest

# Feature テストのみ実行
./vendor/bin/pest tests/Feature

# 特定ファイルのみ
./vendor/bin/pest tests/Feature/Job/JobApplicationTest.php

# カバレッジレポート生成
./vendor/bin/pest --coverage --min=70
```

---

### 4.5 Feature テストで検証しないこと（ユニットテストの責務）

| 内容 | 理由 |
|---|---|
| Service の計算ロジック詳細 | `tests/Unit/Services/` で Service 単体テストを書く |
| Model のスコープ・リレーション | `tests/Unit/Models/` で Model テストを書く |
| メールテンプレートの HTML 内容 | `tests/Unit/Mail/` で Mailable 単体テストを書く |

> **Feature テストの責務は「エンドポイントへのリクエストが正しいレスポンスを返すか」の検証。詳細なロジック検証はUnblockedテストに委ねる。**

---

## まとめ

| 設計項目 | 採用方針 |
|---|---|
| Migration | マスター先行・ソフトデリート・ENUMなし・インデック明示 |
| FormRequest | 1アクション1クラス・authorize はポリシー委譲・validated()のみController に渡す |
| Service 層 | 必要（Fat Controller 防止・再利用・テスタビリティ向上）。ただし薄く作る |
| Feature テスト | HTTPシナリオを優先度別に整理。Pest 記法・RefreshDatabase・Factory で完結 |

---

**次のアクション**:
- [ ] `5_System_Design/` 配下にER図（Mermaid）を追加する
- [ ] 求人検索の複合クエリ設計を `JobSearchService` に落とし込む
- [ ] CI（GitHub Actions）にFeatureテスト自動実行を組み込む
- [ ] シーダー・Factoryの実装ガイドを `2_Development_Environment/GETTING_STARTED.md` に追記する
