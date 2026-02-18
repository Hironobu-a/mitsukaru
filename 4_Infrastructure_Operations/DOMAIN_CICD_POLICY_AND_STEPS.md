# 各ドメインごとのCI/CD構築方針と実践手順

**最終更新**: 2026年2月18日  
**対象**: インフラ担当・開発チーム

---

## 1. CI/CD構築の全体方針
- すべてのドメイン・サービスにCI/CDを導入し、テスト・ビルド・デプロイを自動化
- サービスごとに最適なCI/CDツール（GitHub Actions, GitLab CI, CircleCI等）を選定
- セキュリティ・可用性・拡張性・保守性を重視
- CI/CDの設定・運用はGitリポジトリで管理し、PRベースでレビュー

---

## 2. ドメインごとのCI/CD構築方針

### A. Laravel/静的サイト（Git管理）
**対象**: Nemielu, 求人サイト, 経理Jobs

**方針**:
- GitHub ActionsでCI/CD構築
- push/PR時に自動テスト（PHPUnit, lint等）→ビルド→本番/ステージングへ自動デプロイ
- AWS ECS, S3, EC2等へのデプロイは公式アクションや自作スクリプトで対応
- シークレット・環境変数はGitHub Secretsで管理

**実践手順**:

#### ステップ1: GitHub Actionsワークフローの作成
プロジェクトルートに `.github/workflows/ci.yml` を作成:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      mysql:
        image: mysql:8.0
        env:
          MYSQL_ROOT_PASSWORD: secret
          MYSQL_DATABASE: testing
        options: >-
          --health-cmd="mysqladmin ping"
          --health-interval=10s
          --health-timeout=5s
          --health-retries=3
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.0'
          extensions: mbstring, xml, pdo_mysql
          
      - name: Install Dependencies
        run: composer install --no-interaction --prefer-dist
        
      - name: Copy .env
        run: cp .env.testing.example .env.testing
        
      - name: Generate Application Key
        run: php artisan key:generate --env=testing
        
      - name: Run Tests
        run: php artisan test
        env:
          DB_CONNECTION: mysql
          DB_HOST: 127.0.0.1
          DB_PORT: 3306
          DB_DATABASE: testing
          DB_USERNAME: root
          DB_PASSWORD: secret
          
  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to Production
        run: |
          echo "デプロイスクリプトをここに記載"
          # 例: rsync, scp, AWS CLI等
        env:
          DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
```

#### ステップ2: GitHub Secretsの設定
1. GitHubリポジトリ → Settings → Secrets and variables → Actions
2. 以下のSecretを追加:
   - `DEPLOY_KEY`: デプロイ用のSSH鍵
   - `AWS_ACCESS_KEY_ID`: AWS認証情報（該当する場合）
   - `AWS_SECRET_ACCESS_KEY`: AWS認証情報（該当する場合）

#### ステップ3: デプロイスクリプトの作成
プロジェクトルートに `deploy.sh` を作成:

```bash
#!/bin/bash
set -e

echo "Starting deployment..."

# コードをサーバーに転送
rsync -avz --delete \
  --exclude='.git' \
  --exclude='node_modules' \
  --exclude='storage' \
  ./ user@server:/path/to/project

# サーバー上でコマンド実行
ssh user@server << 'ENDSSH'
  cd /path/to/project
  composer install --no-dev --optimize-autoloader
  php artisan migrate --force
  php artisan config:cache
  php artisan route:cache
  php artisan view:cache
  sudo systemctl reload php-fpm
ENDSSH

echo "Deployment completed!"
```

---

### B. WordPressサイト
**対象**: ミツカルコーポ, ミツプロコーポ

**方針**:
- GitHub ActionsやDeployHQ等でテーマ・プラグインのCI/CD構築
- テーマ/プラグインの更新はステージング環境で事前検証→本番へ自動/手動デプロイ
- DBは手動管理（バックアップ・リストアは自動化推奨）
- SFTP/SSHでのデプロイも可能

**実践手順**:

#### ステップ1: ワークフローの作成
`.github/workflows/deploy-wordpress.yml`:

```yaml
name: Deploy WordPress Theme

on:
  push:
    branches: [ main ]
    paths:
      - 'wp-content/themes/**'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy via SFTP
        uses: SamKirkland/FTP-Deploy-Action@4.3.0
        with:
          server: ${{ secrets.FTP_SERVER }}
          username: ${{ secrets.FTP_USERNAME }}
          password: ${{ secrets.FTP_PASSWORD }}
          local-dir: ./wp-content/themes/your-theme/
          server-dir: /public_html/wp-content/themes/your-theme/
```

#### ステップ2: テーマのバージョン管理
`style.css` でバージョン管理:
```css
/*
Theme Name: Your Theme
Version: 1.2.3
*/
```

---

### C. Studio管理/LP等
**対象**: ミツプロLP用ドメイン

**方針**:
- 静的サイトの場合はNetlify, Vercel, AWS S3等で自動デプロイ
- GitHub Actionsでビルド→デプロイ
- Studio管理の場合は手動運用が多いが、可能ならAPI連携や自動化も検討

**実践手順**:

#### ステップ1: Netlify連携（最も簡単）
1. Netlifyダッシュボードで「New site from Git」
2. GitHubリポジトリを選択
3. ビルド設定:
   - Build command: `npm run build`（該当する場合）
   - Publish directory: `dist` or `public`
4. 自動的にmainブランチへのpushでデプロイ開始

#### ステップ2: GitHub Actions + S3
`.github/workflows/deploy-s3.yml`:

```yaml
name: Deploy to S3

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-northeast-1
          
      - name: Sync to S3
        run: |
          aws s3 sync ./dist s3://your-bucket-name --delete
          
      - name: Invalidate CloudFront
        run: |
          aws cloudfront create-invalidation \
            --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} \
            --paths "/*"
```

---

## 3. CI/CD構築の共通チェックリスト

### 初期構築時
- [ ] `.github/workflows/` にワークフローファイルを作成
- [ ] GitHub Secretsに認証情報を登録
- [ ] テストが自動実行されることを確認
- [ ] デプロイスクリプトをテスト環境で検証
- [ ] ロールバック手順を文書化

### 運用フェーズ
- [ ] PR作成時にCIが自動実行される
- [ ] テスト失敗時にSlack通知（推奨）
- [ ] mainマージ時に自動デプロイ
- [ ] デプロイ成功/失敗の通知
- [ ] 週次でCI/CDログを確認

---

## 4. トラブルシューティング

### Q: テストが失敗する
**A**: 
1. ローカルで `php artisan test` を実行し、同じエラーが出るか確認
2. `.env.testing.example` が最新か確認
3. GitHub Actions のログで詳細エラーを確認

### Q: デプロイが失敗する
**A**:
1. GitHub Secretsの認証情報が正しいか確認
2. サーバーの権限・ディスク容量を確認
3. デプロイスクリプトをローカルでドライラン

### Q: ワークフローが実行されない
**A**:
1. ブランチ名が `on.push.branches` と一致するか確認
2. ワークフローファイルのYAML構文エラーをチェック
3. リポジトリのActions設定が有効か確認

---

## 5. 参考リンク

- [GitHub Actions 公式ドキュメント](https://docs.github.com/en/actions)
- [Laravel Deployment Best Practices](https://laravel.com/docs/deployment)
- [AWS CLI デプロイガイド](https://docs.aws.amazon.com/cli/latest/userguide/)

---

**この手順に従うことで、各ドメインごとに最適なCI/CDを構築し、品質・効率・セキュリティを高めます。**
