# 複数ドメイン・サービス横断の構造的管理方針（CTO策定）

**最終更新**: 2026年2月17日  
**対象**: CTO・インフラ担当

---

## 1. リポジトリ・コード管理

### 方針
- 各サービス/プロダクトごとにGitリポジトリを分割（モノレポが適切な場合はモノレポも検討）
- 共通ライブラリやインフラコード（IaC）は専用リポジトリで一元管理
- リポジトリ命名規則・READMEテンプレートを統一
- GitHub OrganizationやGitLab Groupで権限・可視性を管理

### 実践手順

#### リポジトリ命名規則
```
<company>-<project>-<type>
例: mitsukaru-nemielu-backend
    mitsukaru-corp-wordpress
    mitsukaru-infra-terraform
```

#### READMEテンプレート
各リポジトリには以下を必ず記載:
```markdown
# プロジェクト名

## 概要
（簡潔な説明）

## 技術スタック
- 言語: PHP 8.0
- フレームワーク: Laravel 8

## セットアップ
（環境構築手順）

## デプロイ
（デプロイ手順）

## 関連リンク
- 本番URL: https://...
- ステージングURL: https://...
- ドキュメント: Notion URL
```

---

## 2. インフラ管理（IaC）

### 方針
- Terraform, AWS CloudFormation, Ansible等でインフラをコード化（IaC）
- 各サービスごとにIaCコードを分離しつつ、共通モジュールは再利用
- IaCリポジトリもGitでバージョン管理し、PRベースでレビュー・CI実行
- ステージング・本番など環境ごとに設定を分離（variables, workspaces等）

### 実践手順

#### ディレクトリ構成例（Terraform）
```
mitsukaru-infra-terraform/
├── modules/
│   ├── vpc/
│   ├── ec2/
│   └── rds/
├── environments/
│   ├── staging/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── terraform.tfvars
│   └── production/
│       ├── main.tf
│       ├── variables.tf
│       └── terraform.tfvars
└── README.md
```

#### Terraform実行手順
```bash
# 初期化
cd environments/staging
terraform init

# プラン確認
terraform plan

# 適用（PR承認後）
terraform apply

# 状態確認
terraform show
```

---

## 3. ドメイン・SSL・DNS管理

### 方針
- ドメイン・SSL証明書・DNS設定は一元管理台帳（Notion, Googleスプレッドシート等）で管理
- 変更履歴・担当者・期限・自動更新設定を明記
- DNSやSSLのIaC化（Route53, ACM, Terraform等）も推奨

### 実践手順

#### ドメイン管理台帳（Notionテンプレート）
以下の項目を管理:

| ドメイン | プロジェクト | レジストラ | DNS管理 | SSL証明書 | 有効期限 | 自動更新 | 担当者 |
|---------|------------|----------|---------|----------|---------|---------|--------|
| tax.mitsukaru-pro.co.jp | 求人サイト | お名前.com | Route53 | ACM | 2027/06/30 | ✅ | CTO |
| mitsukaru-corp.co.jp | コーポレート | お名前.com | Cloudflare | Let's Encrypt | 自動 | ✅ | CTO |

#### SSL証明書の取得（Let's Encrypt）
```bash
# Certbotインストール
sudo apt install certbot python3-certbot-nginx

# 証明書取得
sudo certbot --nginx -d example.com -d www.example.com

# 自動更新設定
sudo certbot renew --dry-run
```

---

## 4. CI/CDパイプライン

### 方針
- 各リポジトリごとにCI/CD（GitHub Actions, GitLab CI等）を構築
- 共通ジョブやテンプレートは再利用し、保守性を高める
- デプロイ先・環境変数・シークレットは管理ツール（Vault, AWS Secrets Manager等）で一元管理

### 実践手順
→ 詳細は [`DOMAIN_CICD_POLICY_AND_STEPS.md`](./DOMAIN_CICD_POLICY_AND_STEPS.md) を参照

---

## 5. モニタリング・監視

### 方針
- サービス横断で統一した監視基盤（Datadog, NewRelic, Sentry等）を導入
- ログ・メトリクス・アラート設定も一元管理
- サービスごとにタグや識別子を付与し、横断的な可視化を実現

### 実践手順

#### 推奨監視ツール
| ツール | 用途 | 無料枠 |
|-------|------|--------|
| **UptimeRobot** | 死活監視 | あり（50サイトまで） |
| **Sentry** | エラートラッキング | あり（5,000イベント/月） |
| **CloudWatch** | AWS リソース監視 | AWS無料枠範囲内 |
| **Google Analytics** | アクセス解析 | 無料 |

#### UptimeRobot設定手順
1. https://uptimerobot.com/ でアカウント作成
2. 「Add New Monitor」をクリック
3. HTTPSモニターを設定:
   - URL: https://tax.mitsukaru-pro.co.jp
   - Monitoring Interval: 5分
   - Alert Contacts: メール・Slack設定
4. すべてのドメインに対して繰り返し

---

## 6. ドキュメント・ナレッジ

### 方針
- NotionやConfluence等で「インフラ構成図」「運用手順」「障害対応フロー」などを一元管理
- サービス・ドメインごとにページを分け、共通ルール・テンプレートを用意
- 変更時は必ずドキュメントも更新

### 実践手順

#### Notion構成例
```
📁 ミツカル技術ドキュメント
  ├── 📄 全体アーキテクチャ図
  ├── 📁 Nemieluプロジェクト
  │   ├── インフラ構成図
  │   ├── デプロイ手順
  │   └── 障害対応フロー
  ├── 📁 求人サイト
  │   ├── インフラ構成図
  │   └── デプロイ手順
  └── 📁 ドメイン管理台帳
      └── ドメイン・SSL一覧
```

#### ドキュメント更新ルール
- インフラ変更時は**必ず**Notionを更新
- PR説明文に「ドキュメント更新: [NotionリンK]」を記載
- 月次でCTOがドキュメント棚卸しを実施

---

## 7. 権限・セキュリティ

### 方針
- IAM（AWS/GCP等）やGitHub Teamsで最小権限の原則を徹底
- 退職・異動時の権限棚卸しを定期実施
- シークレット・証明書・APIキーは専用管理ツールで一元管理

### 実践手順

#### GitHub Teamsの設定
```
Organization: mitsukaru
├── Team: Admin
│   └── CTO（全リポジトリの管理者権限）
├── Team: Backend
│   └── バックエンドエンジニア（書き込み権限）
└── Team: Frontend
    └── フロントエンドエンジニア（書き込み権限）
```

#### AWS IAMポリシー例
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::mitsukaru-assets/*"
    }
  ]
}
```

#### シークレット管理
- パスワード: 1Password / Bitwarden など
- API Key: GitHub Secrets / AWS Secrets Manager
- SSH鍵: 各メンバーの個人鍵を使用（共有しない）

---

## 8. 定期メンテナンス

### 月次タスク
- [ ] ドメイン・SSL有効期限の確認
- [ ] 監視アラートの棚卸し（誤検知の調整）
- [ ] IAM権限の見直し
- [ ] バックアップのリストアテスト

### 四半期タスク
- [ ] 全ドキュメントの更新確認
- [ ] インフラコストの見直し
- [ ] セキュリティ脆弱性診断（依存パッケージ含む）

---

**このように「一元管理＋分割管理のバランス」「IaC・自動化・ドキュメント化」を徹底することで、複数ドメイン・サービスでも安全かつ効率的な運用が可能です。**
