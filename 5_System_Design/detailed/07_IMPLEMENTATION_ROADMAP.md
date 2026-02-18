# 実装ロードマップ

**作成日**: 2026年2月18日
**バージョン**: 1.0
**MVP ローンチ目標**: 2026年5月中旬

---

## 1. 全体スケジュール

```
2026年
 2月    3月         4月           5月         6月〜
 ├──────┼──────────┼────────────┼──────────┼──────────
 │ W1   │ W1 W2 W3 W4│ W1 W2 W3 W4│ W1 W2    │
 │      │            │            │          │
 │Sprint│ Sprint 1   │ Sprint 2   │Sprint 3  │ Phase 2
 │  0   │ Backend    │ Frontend   │QA/Launch │
 │設計  │ Core API   │ Integration│ 最終調整  │
 │環境  │ + DB       │ + UI       │          │
 └──────┴────────────┴────────────┴──────────┴──────────
```

---

## 2. Sprint 0: 設計・環境構築（2月 W3-W4）

**期間**: 2026/02/18 - 2026/02/28（2週間）
**目標**: 開発環境の完全構築 + 設計レビュー完了

### タスク一覧

| # | タスク | 担当 | 見積 | 完了条件 |
|---|--------|------|------|---------|
| S0-01 | 詳細設計書レビュー・承認 | 全員 | 2d | 全設計書にApproved |
| S0-02 | GitHub リポジトリ作成（monorepo） | Lead | 0.5d | README + .gitignore + branch protection |
| S0-03 | Laravel 11 プロジェクト初期化 | Backend | 1d | `composer create-project` + 基本設定 |
| S0-04 | Next.js 15 プロジェクト初期化 | Frontend | 1d | `create-next-app` + TypeScript + Tailwind + shadcn |
| S0-05 | Docker Compose 環境構築 | Lead | 1d | 全サービス起動確認 |
| S0-06 | CI/CD パイプライン構築 | Lead | 1d | lint + test + build が通る |
| S0-07 | Sanctum SPA 認証の基盤設定 | Backend | 1d | CORS + Session + Cookie 設定 |
| S0-08 | PHPStan + Pint + Pest 設定 | Backend | 0.5d | 設定ファイル + 初期テスト通過 |
| S0-09 | ESLint + Prettier + Vitest 設定 | Frontend | 0.5d | 設定ファイル + 初期テスト通過 |
| S0-10 | shadcn/ui コンポーネント初期導入 | Frontend | 0.5d | Button, Input, Card, Dialog, Select 等 |
| S0-11 | 共通レイアウト（Header/Footer）実装 | Frontend | 1d | レスポンシブ対応 |

### 成果物

- [x] 設計書一式（レビュー済み）
- [ ] 動作する Docker 開発環境
- [ ] CI/CD パイプライン（lint + test）
- [ ] フロントエンド・バックエンドの空プロジェクト

---

## 3. Sprint 1: バックエンド Core API（3月 W1-W4）

**期間**: 2026/03/01 - 2026/03/31（4週間）
**目標**: 全 MVP API エンドポイントの実装完了

### Week 1: マスターデータ + 認証

| # | タスク | 見積 | テスト |
|---|--------|------|--------|
| S1-01 | マスターテーブル Migration + Seeder | 1d | Seeder 実行確認 |
| S1-02 | Enum クラス作成（全ステータス） | 0.5d | Unit テスト |
| S1-03 | マスターデータ API | 0.5d | Feature テスト |
| S1-04 | User Model + Migration | 0.5d | Factory + Unit テスト |
| S1-05 | 求職者認証 API（登録・ログイン・ログアウト） | 1.5d | Feature テスト |
| S1-06 | メール認証フロー | 1d | Feature テスト |
| S1-07 | パスワードリセットフロー | 0.5d | Feature テスト |

### Week 2: 企業 + 求人 CRUD

| # | タスク | 見積 | テスト |
|---|--------|------|--------|
| S1-08 | Company Model + Migration | 0.5d | Factory + Unit テスト |
| S1-09 | CompanyUser Model + 企業認証 API | 1d | Feature テスト |
| S1-10 | Job Model + Migration（全リレーション含む） | 1d | Factory + Unit テスト |
| S1-11 | 求人 CRUD API（企業側） | 2d | Feature テスト |
| S1-12 | 求人ステータス管理 Service | 0.5d | Unit テスト |

### Week 3: 検索 + 応募 + お気に入り

| # | タスク | 見積 | テスト |
|---|--------|------|--------|
| S1-13 | Meilisearch 設定 + Scout 統合 | 1d | 検索動作確認 |
| S1-14 | JobSearchService 実装 | 1.5d | Unit + Feature テスト |
| S1-15 | 求人検索 API（ファセット含む） | 1d | Feature テスト |
| S1-16 | 応募 API + ApplicationService | 1d | Feature テスト |
| S1-17 | お気に入り API | 0.5d | Feature テスト |

### Week 4: 管理者 + コンテンツ + 通知

| # | タスク | 見積 | テスト |
|---|--------|------|--------|
| S1-18 | AdminUser Model + 管理者認証 | 0.5d | Feature テスト |
| S1-19 | 管理者 API（企業審査・求人管理・ユーザー管理） | 2d | Feature テスト |
| S1-20 | Article Model + 記事 CRUD API | 1d | Feature テスト |
| S1-21 | 通知システム（メール送信 + キュー） | 1d | Feature テスト（Mail::fake） |
| S1-22 | API Resource 整備 + レスポンス統一 | 0.5d | 全 API レスポンス確認 |

### Sprint 1 完了条件

- [ ] 全 API エンドポイント実装完了
- [ ] Feature テスト カバレッジ 70% 以上
- [ ] PHPStan Level 8 パス
- [ ] API ドキュメント（Scramble）生成確認

---

## 4. Sprint 2: フロントエンド統合（4月 W1-W4）

**期間**: 2026/04/01 - 2026/04/30（4週間）
**目標**: 全画面の実装 + API 統合完了

### Week 1: 公開ページ（SEO 重要）

| # | タスク | 見積 | テスト |
|---|--------|------|--------|
| S2-01 | トップページ実装 | 1.5d | Vitest + Visual |
| S2-02 | 求人一覧ページ（検索フォーム + カード一覧） | 2d | Vitest + E2E |
| S2-03 | 求人詳細ページ（構造化データ含む） | 1.5d | Vitest + E2E |
| S2-04 | API クライアント + TanStack Query 設定 | 0.5d | - |

### Week 2: 認証 + 求職者機能

| # | タスク | 見積 | テスト |
|---|--------|------|--------|
| S2-05 | ログインページ | 1d | E2E |
| S2-06 | 会員登録ページ | 1d | E2E |
| S2-07 | 認証状態管理（Zustand + Sanctum） | 1d | Vitest |
| S2-08 | プロフィール編集ページ | 1.5d | Vitest |
| S2-09 | 応募管理ページ | 0.5d | Vitest |
| S2-10 | お気に入りページ + お気に入りボタン | 0.5d | Vitest |

### Week 3: 企業管理画面

| # | タスク | 見積 | テスト |
|---|--------|------|--------|
| S2-11 | 企業ログイン・登録ページ | 1d | E2E |
| S2-12 | 企業ダッシュボード | 1d | Vitest |
| S2-13 | 求人投稿・編集フォーム | 2d | Vitest + E2E |
| S2-14 | 応募者管理ページ | 1d | Vitest |

### Week 4: 管理画面 + SEO + 仕上げ

| # | タスク | 見積 | テスト |
|---|--------|------|--------|
| S2-15 | 管理者ダッシュボード | 1d | Vitest |
| S2-16 | 管理者 - 企業審査画面 | 0.5d | Vitest |
| S2-17 | 管理者 - 求人・ユーザー管理画面 | 1d | Vitest |
| S2-18 | 記事一覧・詳細ページ | 1d | Vitest |
| S2-19 | 職種別 LP ページ | 1d | Visual |
| S2-20 | サイトマップ・robots.txt・メタデータ最適化 | 0.5d | - |
| S2-21 | レスポンシブ対応の最終調整 | 0.5d | Visual |

### Sprint 2 完了条件

- [ ] 全画面実装完了
- [ ] API 統合完了（全エンドポイント接続確認）
- [ ] E2E テスト（クリティカルパス5本）通過
- [ ] Lighthouse スコア 90+ （Performance, SEO, Accessibility）
- [ ] モバイル・タブレット・デスクトップ表示確認

---

## 5. Sprint 3: QA・最終調整・ローンチ（5月 W1-W2）

**期間**: 2026/05/01 - 2026/05/15（2週間）
**目標**: 本番デプロイ + ローンチ

### Week 1: QA + バグ修正

| # | タスク | 見積 |
|---|--------|------|
| S3-01 | 全画面の手動テスト（チェックリスト） | 2d |
| S3-02 | バグ修正 | 2d |
| S3-03 | パフォーマンス最適化（画像最適化・キャッシュ調整） | 1d |

### Week 2: 本番環境構築 + ローンチ

| # | タスク | 見積 |
|---|--------|------|
| S3-04 | AWS 本番環境構築（Terraform） | 2d |
| S3-05 | ドメイン・SSL・DNS 設定 | 0.5d |
| S3-06 | 本番デプロイ + 動作確認 | 1d |
| S3-07 | 監視・アラート設定 | 0.5d |
| S3-08 | マスターデータ投入 | 0.5d |
| S3-09 | ローンチ！ | - |

---

## 6. Phase 2 計画（5月下旬〜8月）

| 機能 | 見積 | 優先度 |
|------|------|--------|
| スカウト機能 | 3週間 | High |
| レコメンドエンジン | 2週間 | High |
| 年収診断ツール | 1週間 | Medium |
| 企業口コミ | 2週間 | Medium |
| チャット機能 | 3週間 | Low |

---

## 7. 開発プロセス

### 7.1 ブランチ戦略

```
main ─────────────────────────────────── (本番)
  │
  └── develop ────────────────────────── (開発統合)
        │
        ├── feature/S1-01-master-tables
        ├── feature/S1-05-auth-api
        ├── feature/S2-02-job-list-page
        └── fix/S3-02-search-bug
```

| ブランチ | 用途 | マージ先 |
|---------|------|---------|
| main | 本番リリース | - |
| develop | 開発統合 | main（リリース時） |
| feature/* | 機能開発 | develop |
| fix/* | バグ修正 | develop |
| hotfix/* | 緊急修正 | main + develop |

### 7.2 PR ルール

- 全 PR に最低1名のレビュー必須
- CI（lint + test）が通らない PR はマージ不可
- PR テンプレート使用必須
- コミットメッセージは Conventional Commits 準拠

```
feat: 求人検索APIを実装
fix: 二重応募時のエラーハンドリングを修正
docs: API設計書を更新
test: 応募機能のFeatureテストを追加
refactor: JobSearchServiceのクエリ最適化
chore: PHPStan Level 8対応
```

### 7.3 スプリント運用

| イベント | 頻度 | 時間 |
|---------|------|------|
| デイリースタンドアップ | 毎日 | 15分 |
| スプリントプランニング | 2週間ごと | 1時間 |
| スプリントレトロスペクティブ | 2週間ごと | 30分 |
| コードレビュー | 随時 | - |

---

## 8. リスク管理

| リスク | 対策 | トリガー |
|--------|------|---------|
| スケジュール遅延 | MVPスコープ縮小（記事機能を Phase 2 に移動） | Sprint 1 完了時点で2日以上遅延 |
| 技術的ブロッカー | ペアプログラミング + 外部相談 | 1タスクで2日以上停滞 |
| パフォーマンス問題 | Meilisearch → MySQL FULLTEXT にフォールバック | 検索レスポンス > 500ms |
| セキュリティ脆弱性 | 即座に修正 + ホットフィックスリリース | Sentry / 脆弱性スキャンで検出 |

---

## 9. 定義完了条件（Definition of Done）

### 各タスクの完了条件

- [ ] コードが実装されている
- [ ] テストが書かれている（Feature or Unit）
- [ ] PHPStan / ESLint がパスする
- [ ] PR レビューが承認されている
- [ ] develop ブランチにマージされている
- [ ] 関連ドキュメントが更新されている

### MVP ローンチの完了条件

- [ ] 全 MVP 機能が動作する
- [ ] Feature テストカバレッジ 70% 以上
- [ ] E2E テスト（クリティカルパス）全パス
- [ ] Lighthouse Performance 90+
- [ ] Lighthouse SEO 95+
- [ ] Lighthouse Accessibility 90+
- [ ] セキュリティチェックリスト全項目クリア
- [ ] 本番環境で全機能動作確認
- [ ] 監視・アラートが正常動作
- [ ] ロールバック手順が文書化・テスト済み
