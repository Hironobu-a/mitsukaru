# 士業向け求人サイト「ミツプロ」 - 詳細設計書インデックス

**作成日**: 2026年2月18日
**ステータス**: レビュー待ち

---

## 設計書一覧

| # | ドキュメント | 内容 | 対象読者 |
|---|-------------|------|---------|
| 01 | [PRD（プロダクト要件定義書）](./01_PRD.md) | ビジョン・ターゲット・機能要件・非機能要件・KPI | 全員 |
| 02 | [技術選定・アーキテクチャ設計](./02_TECH_STACK_AND_ARCHITECTURE.md) | 技術スタック・全体構成・認証・キャッシュ・検索・非同期処理 | エンジニア |
| 03 | [データベース設計](./03_DATABASE_DESIGN.md) | ER図・全テーブル定義・Enum・Migration順序・Seeder | エンジニア |
| 04 | [API設計](./04_API_DESIGN.md) | 全エンドポイント・リクエスト/レスポンス仕様・ルーティング | エンジニア |
| 05 | [フロントエンド設計](./05_FRONTEND_DESIGN.md) | ページ構成・コンポーネント・状態管理・SEO・テスト | フロントエンド |
| 06 | [インフラ・CI/CD設計](./06_INFRASTRUCTURE_CICD.md) | Docker・AWS構成・GitHub Actions・監視・セキュリティ | インフラ/DevOps |
| 07 | [実装ロードマップ](./07_IMPLEMENTATION_ROADMAP.md) | Sprint計画・タスク一覧・ブランチ戦略・完了条件 | 全員 |
| 08 | [限界を超えるための戦略](./08_BEYOND_LIMITS_STRATEGY.md) | 社会価値・マネタイズ・UX飛躍・SEO拡張・競合参入障壁の構築 | 全員 |

---

## 技術スタック概要

| レイヤー | 技術 |
|---------|------|
| Frontend | Next.js 15 + React 19 + TypeScript + Tailwind CSS 4 + shadcn/ui |
| Backend | Laravel 11 + PHP 8.3 + Pest |
| Database | MySQL 8.0 + Redis 7 + Meilisearch |
| Infrastructure | AWS (ECS Fargate + RDS + ElastiCache) + Vercel + Docker |
| CI/CD | GitHub Actions |
| Monitoring | Sentry + UptimeRobot + CloudWatch |

---

## 次のアクション

1. **全設計書のレビュー** → チーム全員で確認・フィードバック
2. **Sprint 0 開始** → 環境構築・リポジトリ作成
3. **Sprint 1 開始** → バックエンド Core API 実装
