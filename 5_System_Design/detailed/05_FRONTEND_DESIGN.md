# フロントエンド設計書

**作成日**: 2026年2月18日
**バージョン**: 1.0

---

## 1. 設計方針

### 1.1 基本原則

- **Server Components First**: デフォルトはServer Component、インタラクティブな部分のみClient Component
- **Progressive Enhancement**: JavaScript無効でも基本機能が動作
- **Mobile First**: レスポンシブデザインはモバイルから設計
- **Accessibility**: WCAG 2.1 Level AA 準拠
- **Performance**: Core Web Vitals 全指標グリーン

### 1.2 コンポーネント設計原則

| 原則 | 説明 |
|------|------|
| 単一責任 | 1コンポーネント = 1つの責務 |
| コンポジション | 継承よりコンポジション |
| Colocation | 関連ファイルは近くに配置 |
| 型安全 | Props は全て TypeScript で型定義 |

---

## 2. ページ構成・画面遷移

### 2.1 サイトマップ

```
/ (トップページ)
├── /jobs (求人一覧・検索)
│   └── /jobs/[id] (求人詳細)
├── /companies/[id] (企業詳細)
├── /articles (記事一覧)
│   └── /articles/[slug] (記事詳細)
├── /[profession] (職種別LP: /tax-accountant, /lawyer, etc.)
│
├── /login (ログイン)
├── /register (会員登録)
├── /forgot-password (パスワードリセット)
│
├── /dashboard (求職者マイページ)
│   ├── /dashboard/profile (プロフィール編集)
│   ├── /dashboard/applications (応募管理)
│   └── /dashboard/favorites (お気に入り)
│
├── /company (企業管理画面)
│   ├── /company/dashboard (ダッシュボード)
│   ├── /company/jobs (求人管理)
│   │   ├── /company/jobs/new (求人作成)
│   │   └── /company/jobs/[id]/edit (求人編集)
│   ├── /company/applications (応募者管理)
│   └── /company/profile (企業プロフィール)
│
├── /admin (管理画面)
│   ├── /admin/dashboard (KPIダッシュボード)
│   ├── /admin/companies (企業管理)
│   ├── /admin/jobs (求人管理)
│   ├── /admin/users (ユーザー管理)
│   └── /admin/articles (記事管理)
│       └── /admin/articles/new (記事作成)
│       └── /admin/articles/[id]/edit (記事編集)
│
├── /privacy (プライバシーポリシー)
├── /terms (利用規約)
└── /contact (お問い合わせ)
```

### 2.2 レンダリング戦略

| ページ | 戦略 | 理由 |
|--------|------|------|
| トップページ | SSG + ISR (1h) | SEO重要・更新頻度低 |
| 求人一覧 | SSR | 検索パラメータ動的・SEO重要 |
| 求人詳細 | ISR (1h) + On-demand | SEO重要・更新時即時反映 |
| 企業詳細 | ISR (24h) | SEO重要・更新頻度低 |
| 記事一覧 | SSG + ISR (1h) | SEO最重要 |
| 記事詳細 | SSG + ISR (24h) | SEO最重要 |
| 職種別LP | SSG | SEO最重要・静的コンテンツ |
| ログイン/登録 | CSR | SEO不要・インタラクティブ |
| ダッシュボード | CSR | SEO不要・認証必須 |
| 企業管理画面 | CSR | SEO不要・認証必須 |
| 管理画面 | CSR | SEO不要・認証必須 |

---

## 3. コンポーネント設計

### 3.1 デザインシステム（shadcn/ui ベース）

#### カラーパレット

```css
/* Tailwind CSS 4 カスタムテーマ */
:root {
    /* Primary: 信頼感のあるブルー */
    --primary: oklch(0.55 0.18 250);
    --primary-foreground: oklch(0.98 0 0);

    /* Secondary: 温かみのあるアンバー */
    --secondary: oklch(0.75 0.12 75);
    --secondary-foreground: oklch(0.25 0 0);

    /* Accent: 士業の格式を表すダークネイビー */
    --accent: oklch(0.30 0.08 260);
    --accent-foreground: oklch(0.98 0 0);

    /* Semantic Colors */
    --success: oklch(0.60 0.15 145);
    --warning: oklch(0.75 0.15 85);
    --destructive: oklch(0.55 0.20 25);

    /* Background */
    --background: oklch(0.985 0.002 250);
    --card: oklch(1 0 0);
    --muted: oklch(0.96 0.005 250);

    /* Text */
    --foreground: oklch(0.20 0.02 260);
    --muted-foreground: oklch(0.55 0.01 260);

    /* Border */
    --border: oklch(0.90 0.005 250);
    --ring: oklch(0.55 0.18 250);
}
```

#### タイポグラフィ

```css
/* フォント設定 */
--font-sans: "Noto Sans JP", "Inter", system-ui, sans-serif;
--font-heading: "Noto Sans JP", "Inter", system-ui, sans-serif;

/* サイズスケール */
--text-xs: 0.75rem;    /* 12px */
--text-sm: 0.875rem;   /* 14px */
--text-base: 1rem;     /* 16px */
--text-lg: 1.125rem;   /* 18px */
--text-xl: 1.25rem;    /* 20px */
--text-2xl: 1.5rem;    /* 24px */
--text-3xl: 1.875rem;  /* 30px */
--text-4xl: 2.25rem;   /* 36px */
```

### 3.2 主要コンポーネント仕様

#### JobCard（求人カード）

```typescript
interface JobCardProps {
    job: {
        id: number;
        title: string;
        company: {
            id: number;
            name: string;
            logo_url: string | null;
        };
        professions: Array<{ id: number; name: string }>;
        employment_type: { name: string };
        salary_display: string;
        prefecture: { name: string } | null;
        remote_type_label: string;
        skills: Array<{ id: number; name: string }>;
        is_favorited: boolean;
        published_at: string;
    };
    variant?: 'default' | 'compact' | 'featured';
    onFavoriteToggle?: (jobId: number) => void;
}
```

**表示要素:**
- 企業ロゴ + 企業名
- 求人タイトル（最大2行、ellipsis）
- 職種バッジ（複数可）
- 雇用形態 / 年収 / 勤務地 / リモート
- スキルタグ（最大3つ + 「他N件」）
- お気に入りボタン（ハート）
- 掲載日

#### JobSearchForm（求人検索フォーム）

```typescript
interface JobSearchFormProps {
    initialValues?: JobSearchParams;
    onSearch: (params: JobSearchParams) => void;
    facets?: {
        professions: Array<{ id: number; name: string; count: number }>;
        prefectures: Array<{ id: number; name: string; count: number }>;
        employment_types: Array<{ id: number; name: string; count: number }>;
    };
}

interface JobSearchParams {
    keyword?: string;
    profession_ids?: number[];
    prefecture_id?: number;
    employment_type_ids?: number[];
    salary_min?: number;
    salary_max?: number;
    remote_type?: number;
    skill_ids?: number[];
    sort?: 'newest' | 'salary_high' | 'salary_low';
}
```

**UI構成:**
- キーワード入力（デバウンス300ms）
- 職種セレクト（マルチ選択 + チェックボックス）
- 勤務地セレクト（都道府県ドロップダウン）
- 雇用形態フィルタ（チェックボックス）
- 年収レンジスライダー
- リモート区分フィルタ
- ソート切り替え
- 検索結果件数表示
- フィルタリセットボタン

### 3.3 レイアウトコンポーネント

#### Header

```
┌─────────────────────────────────────────────────────────────┐
│  [Logo]  求人検索  記事  企業向け  |  ログイン  会員登録    │
└─────────────────────────────────────────────────────────────┘

(ログイン後)
┌─────────────────────────────────────────────────────────────┐
│  [Logo]  求人検索  記事  |  お気に入り(3)  [Avatar ▼]      │
└─────────────────────────────────────────────────────────────┘
```

#### Footer

```
┌─────────────────────────────────────────────────────────────┐
│  [Logo]                                                      │
│                                                              │
│  求職者の方へ        企業の方へ        コンテンツ            │
│  ・求人を探す        ・求人を掲載する  ・転職ノウハウ        │
│  ・会員登録          ・企業登録        ・年収相場            │
│  ・ログイン          ・ログイン        ・資格ガイド          │
│                                                              │
│  士業別求人                                                  │
│  税理士 | 弁護士 | 社労士 | 公認会計士 | 司法書士 | ...     │
│                                                              │
│  ────────────────────────────────────────────────────────── │
│  © 2026 ミツカル  |  利用規約  |  プライバシーポリシー      │
└─────────────────────────────────────────────────────────────┘
```

---

## 4. 状態管理設計

### 4.1 サーバー状態（TanStack Query）

```typescript
// hooks/use-jobs.ts
export function useJobs(params: JobSearchParams) {
    return useQuery({
        queryKey: ['jobs', params],
        queryFn: () => apiClient.get('/jobs', { params }),
        staleTime: 5 * 60 * 1000,     // 5分間はキャッシュを使用
        gcTime: 30 * 60 * 1000,        // 30分間キャッシュ保持
        placeholderData: keepPreviousData, // ページ遷移時にちらつき防止
    });
}

export function useJob(id: number) {
    return useQuery({
        queryKey: ['jobs', id],
        queryFn: () => apiClient.get(`/jobs/${id}`),
        staleTime: 10 * 60 * 1000,
    });
}

export function useApplyJob() {
    const queryClient = useQueryClient();

    return useMutation({
        mutationFn: ({ jobId, data }: { jobId: number; data: ApplyData }) =>
            apiClient.post(`/jobs/${jobId}/apply`, data),
        onSuccess: (_, { jobId }) => {
            queryClient.invalidateQueries({ queryKey: ['applications'] });
            queryClient.setQueryData(['jobs', jobId], (old: Job) => ({
                ...old,
                has_applied: true,
            }));
        },
    });
}

export function useToggleFavorite() {
    const queryClient = useQueryClient();

    return useMutation({
        mutationFn: ({ jobId, isFavorited }: { jobId: number; isFavorited: boolean }) =>
            isFavorited
                ? apiClient.delete(`/jobs/${jobId}/favorite`)
                : apiClient.post(`/jobs/${jobId}/favorite`),
        onMutate: async ({ jobId, isFavorited }) => {
            // 楽観的更新
            await queryClient.cancelQueries({ queryKey: ['jobs'] });
            queryClient.setQueriesData(
                { queryKey: ['jobs'] },
                (old: any) => updateJobFavoriteStatus(old, jobId, !isFavorited)
            );
        },
        onError: () => {
            queryClient.invalidateQueries({ queryKey: ['jobs'] });
        },
    });
}
```

### 4.2 クライアント状態（Zustand）

```typescript
// stores/auth-store.ts
interface AuthState {
    user: User | null;
    isAuthenticated: boolean;
    isLoading: boolean;
    login: (user: User) => void;
    logout: () => void;
    setLoading: (loading: boolean) => void;
}

export const useAuthStore = create<AuthState>((set) => ({
    user: null,
    isAuthenticated: false,
    isLoading: true,
    login: (user) => set({ user, isAuthenticated: true, isLoading: false }),
    logout: () => set({ user: null, isAuthenticated: false, isLoading: false }),
    setLoading: (isLoading) => set({ isLoading }),
}));

// stores/ui-store.ts
interface UIState {
    isMobileMenuOpen: boolean;
    isSearchFilterOpen: boolean;
    toggleMobileMenu: () => void;
    toggleSearchFilter: () => void;
}

export const useUIStore = create<UIState>((set) => ({
    isMobileMenuOpen: false,
    isSearchFilterOpen: false,
    toggleMobileMenu: () => set((s) => ({ isMobileMenuOpen: !s.isMobileMenuOpen })),
    toggleSearchFilter: () => set((s) => ({ isSearchFilterOpen: !s.isSearchFilterOpen })),
}));
```

---

## 5. API クライアント設計

```typescript
// lib/api-client.ts
import axios, { AxiosError, AxiosInstance } from 'axios';

const API_BASE_URL = process.env.NEXT_PUBLIC_API_URL || 'http://localhost:8000';

class ApiClient {
    private client: AxiosInstance;

    constructor() {
        this.client = axios.create({
            baseURL: `${API_BASE_URL}/api/v1`,
            withCredentials: true,
            withXSRFToken: true,
            headers: {
                'Content-Type': 'application/json',
                'Accept': 'application/json',
            },
        });

        this.client.interceptors.response.use(
            (response) => response,
            (error: AxiosError) => {
                if (error.response?.status === 401) {
                    useAuthStore.getState().logout();
                    if (typeof window !== 'undefined') {
                        window.location.href = '/login';
                    }
                }
                return Promise.reject(error);
            }
        );
    }

    async getCsrfCookie(): Promise<void> {
        await axios.get(`${API_BASE_URL}/sanctum/csrf-cookie`, {
            withCredentials: true,
        });
    }

    async get<T>(url: string, config?: any): Promise<T> {
        const response = await this.client.get<T>(url, config);
        return response.data;
    }

    async post<T>(url: string, data?: any): Promise<T> {
        const response = await this.client.post<T>(url, data);
        return response.data;
    }

    async put<T>(url: string, data?: any): Promise<T> {
        const response = await this.client.put<T>(url, data);
        return response.data;
    }

    async patch<T>(url: string, data?: any): Promise<T> {
        const response = await this.client.patch<T>(url, data);
        return response.data;
    }

    async delete<T>(url: string): Promise<T> {
        const response = await this.client.delete<T>(url);
        return response.data;
    }
}

export const apiClient = new ApiClient();
```

---

## 6. SEO 設計

### 6.1 メタデータ生成

```typescript
// app/jobs/[id]/page.tsx
import { Metadata } from 'next';

export async function generateMetadata({ params }: Props): Promise<Metadata> {
    const job = await getJob(params.id);

    return {
        title: `${job.title} | ${job.company.name} | ミツプロ`,
        description: `${job.company.name}の${job.professions[0]?.name}求人。${job.salary_display}。${job.prefecture?.name}勤務。`,
        openGraph: {
            title: job.title,
            description: job.description.slice(0, 200),
            type: 'website',
            url: `https://mitsupro.jp/jobs/${job.id}`,
            images: [{ url: job.company.logo_url || '/og-default.png' }],
        },
        alternates: {
            canonical: `https://mitsupro.jp/jobs/${job.id}`,
        },
    };
}
```

### 6.2 構造化データ（JSON-LD）

```typescript
// components/job/job-structured-data.tsx
export function JobStructuredData({ job }: { job: JobDetail }) {
    const structuredData = {
        '@context': 'https://schema.org',
        '@type': 'JobPosting',
        title: job.title,
        description: job.description,
        datePosted: job.published_at,
        validThrough: job.closed_at,
        employmentType: mapEmploymentType(job.employment_type.name),
        hiringOrganization: {
            '@type': 'Organization',
            name: job.company.name,
            sameAs: job.company.website,
            logo: job.company.logo_url,
        },
        jobLocation: {
            '@type': 'Place',
            address: {
                '@type': 'PostalAddress',
                addressLocality: job.work_location,
                addressRegion: job.prefecture?.name,
                addressCountry: 'JP',
            },
        },
        baseSalary: job.salary_min ? {
            '@type': 'MonetaryAmount',
            currency: 'JPY',
            value: {
                '@type': 'QuantitativeValue',
                minValue: job.salary_min,
                maxValue: job.salary_max,
                unitText: mapSalaryUnit(job.salary_type.name),
            },
        } : undefined,
    };

    return (
        <script
            type="application/ld+json"
            dangerouslySetInnerHTML={{ __html: JSON.stringify(structuredData) }}
        />
    );
}
```

### 6.3 サイトマップ生成

```typescript
// app/sitemap.ts
import { MetadataRoute } from 'next';

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
    const jobs = await fetchAllPublishedJobIds();
    const articles = await fetchAllPublishedArticleSlugs();
    const professions = await fetchAllProfessions();

    return [
        { url: 'https://mitsupro.jp', lastModified: new Date(), changeFrequency: 'daily', priority: 1.0 },
        { url: 'https://mitsupro.jp/jobs', lastModified: new Date(), changeFrequency: 'hourly', priority: 0.9 },
        { url: 'https://mitsupro.jp/articles', lastModified: new Date(), changeFrequency: 'daily', priority: 0.8 },

        ...professions.map((p) => ({
            url: `https://mitsupro.jp/${p.slug}`,
            lastModified: new Date(),
            changeFrequency: 'daily' as const,
            priority: 0.9,
        })),

        ...jobs.map((job) => ({
            url: `https://mitsupro.jp/jobs/${job.id}`,
            lastModified: new Date(job.updated_at),
            changeFrequency: 'weekly' as const,
            priority: 0.7,
        })),

        ...articles.map((article) => ({
            url: `https://mitsupro.jp/articles/${article.slug}`,
            lastModified: new Date(article.updated_at),
            changeFrequency: 'monthly' as const,
            priority: 0.6,
        })),
    ];
}
```

---

## 7. フォームバリデーション（Zod スキーマ）

```typescript
// lib/validations.ts
import { z } from 'zod';

export const loginSchema = z.object({
    email: z.string().email('有効なメールアドレスを入力してください'),
    password: z.string().min(8, 'パスワードは8文字以上で入力してください'),
});

export const registerSchema = z.object({
    email: z.string().email('有効なメールアドレスを入力してください'),
    password: z.string()
        .min(8, 'パスワードは8文字以上で入力してください')
        .regex(/[A-Za-z]/, '英字を含めてください')
        .regex(/[0-9]/, '数字を含めてください'),
    password_confirmation: z.string(),
    last_name: z.string().min(1, '姓を入力してください').max(50),
    first_name: z.string().min(1, '名を入力してください').max(50),
    last_name_kana: z.string().max(50).optional(),
    first_name_kana: z.string().max(50).optional(),
}).refine((data) => data.password === data.password_confirmation, {
    message: 'パスワードが一致しません',
    path: ['password_confirmation'],
});

export const jobSearchSchema = z.object({
    keyword: z.string().max(100).optional(),
    profession_ids: z.array(z.number()).optional(),
    prefecture_id: z.number().optional(),
    employment_type_ids: z.array(z.number()).optional(),
    salary_min: z.number().min(0).optional(),
    salary_max: z.number().min(0).optional(),
    remote_type: z.number().optional(),
    sort: z.enum(['newest', 'salary_high', 'salary_low']).optional(),
});

export const jobFormSchema = z.object({
    title: z.string().min(1, '求人タイトルを入力してください').max(200),
    description: z.string().min(1, '仕事内容を入力してください').max(10000),
    requirement: z.string().max(5000).optional(),
    welcome_skill: z.string().max(5000).optional(),
    employment_type_id: z.number({ required_error: '雇用形態を選択してください' }),
    salary_type_id: z.number({ required_error: '給与形態を選択してください' }),
    salary_min: z.number().min(0).optional(),
    salary_max: z.number().min(0).optional(),
    prefecture_id: z.number().optional(),
    work_location: z.string().max(300).optional(),
    nearest_station: z.string().max(200).optional(),
    work_hours: z.string().max(200).optional(),
    benefits: z.string().max(5000).optional(),
    selection_process: z.string().max(3000).optional(),
    remote_type: z.number().default(0),
    profession_ids: z.array(z.number()).min(1, '対象職種を1つ以上選択してください'),
    skill_ids: z.array(z.number()).optional(),
    status: z.number(),
    closed_at: z.string().optional(),
}).refine(
    (data) => !data.salary_min || !data.salary_max || data.salary_max >= data.salary_min,
    { message: '上限年収は下限年収以上の値を入力してください', path: ['salary_max'] }
);

export type LoginInput = z.infer<typeof loginSchema>;
export type RegisterInput = z.infer<typeof registerSchema>;
export type JobSearchInput = z.infer<typeof jobSearchSchema>;
export type JobFormInput = z.infer<typeof jobFormSchema>;
```

---

## 8. テスト戦略

### 8.1 テストピラミッド

```
        ┌─────┐
        │ E2E │  Playwright (5-10テスト)
        │     │  クリティカルパスのみ
       ┌┴─────┴┐
       │ 統合   │  Vitest + Testing Library (30-50テスト)
       │       │  ページ単位・フォーム操作
      ┌┴───────┴┐
      │ ユニット │  Vitest (50-100テスト)
      │         │  hooks・utils・バリデーション
      └─────────┘
```

### 8.2 E2E テスト対象（Playwright）

| テスト | 優先度 |
|--------|--------|
| 求人検索 → 詳細閲覧 → 応募 | MUST |
| 会員登録 → ログイン → プロフィール編集 | MUST |
| 企業ログイン → 求人投稿 → 公開 | MUST |
| お気に入り追加・削除 | SHOULD |
| 管理者ログイン → 企業審査 | SHOULD |

### 8.3 テスト例

```typescript
// e2e/job-application.spec.ts
import { test, expect } from '@playwright/test';

test.describe('求人応募フロー', () => {
    test('ログイン済みユーザーが求人に応募できる', async ({ page }) => {
        // ログイン
        await page.goto('/login');
        await page.fill('[name="email"]', 'test@example.com');
        await page.fill('[name="password"]', 'password123');
        await page.click('button[type="submit"]');
        await page.waitForURL('/dashboard');

        // 求人検索
        await page.goto('/jobs');
        await page.fill('[placeholder*="キーワード"]', '税理士');
        await page.click('button:has-text("検索")');
        await expect(page.locator('[data-testid="job-card"]')).toHaveCount.greaterThan(0);

        // 求人詳細へ
        await page.click('[data-testid="job-card"]:first-child');
        await expect(page.locator('h1')).toBeVisible();

        // 応募
        await page.click('button:has-text("応募する")');
        await page.fill('textarea[name="message"]', 'テスト応募メッセージ');
        await page.click('button:has-text("応募を確定")');

        await expect(page.locator('text=応募が完了しました')).toBeVisible();
    });
});
```
