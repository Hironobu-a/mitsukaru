# ミツカル 技術ドキュメント - 全体ガイド

**最終更新**: 2026年2月17日  
**管理者**: CTO

---

## 📖 このドキュメント群について

このフォルダには、ミツカルが運用する**複数プロジェクト**の開発方針・運用ルールが格納されています。

### プロジェクト構成

| プロジェクト | 概要 | 技術スタック |
|------------|------|------------|
| **Nemielu** | 眠気・覚醒度測定サービス（主力プロダクト） | Laravel 8 + PHP 8.0 + MySQL |
| **ミツカルコーポレート (mitsukaru-corp.co.jp)** | コーポレートサイト | WordPress (Git管理) |
| **ミツプロコーポレート (mitsukaru-pro.co.jp)** | コーポレートサイト | WordPress |
| **求人サイト (tax.mitsukaru-pro.co.jp)** | 税理士求人サイト | Laravel (Git管理) |
| **各種LP** | 税理士・社労士・経理Jobs等の広告LP | PHP / HTML / Studio |

---

## 📁 ドキュメント構成

### **1_Management_Strategy/** (経営・技術戦略)
経営層・役員向けの技術戦略と、チーム体制の定義

| ファイル | 対象 | 概要 |
|---------|------|------|
| `CTO_MANAGEMENT_PLAN.md` | **Nemielu** | CTO運営方針・技術ロードマップ |
| `TECH_DEBT_ROADMAP.md` | **Nemielu** | 技術的負債の解消計画 |
| `TEAM_STRUCTURE.md` | **全社** | 開発チーム体制・役割分担 |

### **2_Development_Environment/** (開発環境)
エンジニア向けの環境構築・開発フロー

| ファイル | 対象 | 概要 |
|---------|------|------|
| `GETTING_STARTED.md` | **Nemielu** | 最初に読むべきスタートガイド |
| `DEVELOPMENT_GUIDE.md` | **Nemielu** | 詳細な開発環境の使い方 |
| `DATABASE_TOOLS_GUIDE.md` | **Nemielu** | DBツール (phpMyAdmin等) の使い方 |
| `DB_QUICK_REFERENCE.md` | **Nemielu** | DB接続情報・テーブル構成 |

### **3_Testing_Quality/** (品質・テスト)
テスト戦略・品質管理の方針

| ファイル | 対象 | 概要 |
|---------|------|------|
| `EFFICIENT_TEAM_TEST_STRATEGY.md` | **全社** | 少人数チームのテスト戦略 |
| `DOCKER_TESTING_GUIDE.md` | **Nemielu** | Dockerでのテスト実行方法 |
| `TEAM_TEST_POLICY_AND_FLOW.md` | **全社** | テスト実装の具体的なフロー |
| `TEST_COMPLETION_REPORT.md` | **Nemielu** | 過去のテスト整備報告書 |

### **4_Infrastructure_Operations/** (インフラ・運用)
複数ドメイン・サービスの管理方針

| ファイル | 対象 | 概要 |
|---------|------|------|
| `MULTI_DOMAIN_INFRA_MANAGEMENT_POLICY.md` | **全社** | 複数プロダクトのインフラ管理方針 |
| `DOMAIN_CICD_POLICY_AND_STEPS.md` | **全社** | CI/CD構築の方針と手順 |
| `DOMAIN_LP_MANAGEMENT_POLICY.md` | **全社** | LP・ドメイン管理の運用ルール |

---

## 🚀 新メンバーのオンボーディング

### ステップ1: チーム体制を理解する
→ [`1_Management_Strategy/TEAM_STRUCTURE.md`](1_Management_Strategy/TEAM_STRUCTURE.md)

### ステップ2: 担当プロジェクトの開発環境を構築する

#### Nemieluプロジェクトの場合:
1. [`2_Development_Environment/GETTING_STARTED.md`](2_Development_Environment/GETTING_STARTED.md) を読む
2. Dockerで開発環境を起動
3. [`2_Development_Environment/DEVELOPMENT_GUIDE.md`](2_Development_Environment/DEVELOPMENT_GUIDE.md) で詳細を確認

#### その他のプロジェクト (WordPress / LP等) の場合:
各プロジェクトのリポジトリREADMEを参照してください。

### ステップ3: 開発フローを理解する
→ [`../運用ルールなど/branching-rules.md`](../運用ルールなど/branching-rules.md)  
→ [`../運用ルールなど/pull_request_rules.md`](../運用ルールなど/pull_request_rules.md)

### ステップ4: テスト方針を理解する
→ [`3_Testing_Quality/EFFICIENT_TEAM_TEST_STRATEGY.md`](3_Testing_Quality/EFFICIENT_TEAM_TEST_STRATEGY.md)

---

## 🔑 重要なポイント (CTO方針)

### 開発の優先順位
1. **セキュリティ・安定性** > 新機能開発
2. **技術負債の返済** と **新機能開発** を並行で進める
3. **テストを書いてから** 大規模変更を行う

### 禁止事項
- 本番環境の認証情報をコードやコミットに含める
- テストなしで本番にリリースする
- ドキュメントを更新せずにアーキテクチャを変更する

### コミュニケーション
- 週1回の定例ミーティングで進捗・課題共有
- Slack で日常的なコミュニケーション
- Notion で仕様・タスク管理

---

## 📞 お問い合わせ

技術的な質問や不明点は、以下の順で確認してください：

1. このREADME と各ドキュメントを確認
2. Notion の開発ドキュメントを検索
3. Slack で質問 (チャンネル: #dev)
4. CTO に直接相談

---

**Happy Coding! 🎉**
