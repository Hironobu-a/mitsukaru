# インフラ・CI/CD 設計書

**作成日**: 2026年2月18日
**バージョン**: 1.0

---

## 1. インフラ構成

### 1.1 環境一覧

| 環境 | 用途 | URL |
|------|------|-----|
| Local | 開発 | http://localhost:3000 (Frontend) / http://localhost:8000 (API) |
| Staging | 検証・QA | https://stg.mitsupro.jp / https://stg-api.mitsupro.jp |
| Production | 本番 | https://mitsupro.jp / https://api.mitsupro.jp |

### 1.2 本番インフラ構成図

```
                         ┌──────────────┐
                         │  Route 53    │
                         │  (DNS)       │
                         └──────┬───────┘
                                │
                    ┌───────────┼───────────┐
                    │           │           │
            ┌───────▼──────┐   │   ┌───────▼──────┐
            │  CloudFront  │   │   │  CloudFront  │
            │  (Frontend)  │   │   │  (Assets)    │
            │  mitsupro.jp │   │   │  S3 Bucket   │
            └───────┬──────┘   │   └──────────────┘
                    │          │
            ┌───────▼──────┐   │
            │  Vercel      │   │
            │  Next.js SSR │   │
            │  Edge Runtime│   │
            └──────────────┘   │
                               │
                    ┌──────────▼──────────┐
                    │  ALB               │
                    │  api.mitsupro.jp   │
                    │  SSL Termination   │
                    └──────────┬─────────┘
                               │
                    ┌──────────▼──────────┐
                    │  ECS Fargate       │
                    │  Laravel API       │
                    │  ┌────┐ ┌────┐    │
                    │  │Task│ │Task│    │  Auto Scaling
                    │  │ #1 │ │ #2 │    │  min:1 max:4
                    │  └────┘ └────┘    │
                    └───┬────┬────┬─────┘
                        │    │    │
              ┌─────────┘    │    └─────────┐
              │              │              │
    ┌─────────▼──┐  ┌───────▼───┐  ┌───────▼───────┐
    │  RDS MySQL │  │ ElastiCache│  │  Meilisearch  │
    │  db.t3.med │  │  Redis     │  │  ECS Fargate  │
    │  Multi-AZ  │  │  cache.t3  │  │  (1 Task)     │
    │  + Replica │  │  .micro    │  │               │
    └────────────┘  └───────────┘  └───────────────┘
```

### 1.3 AWS リソース一覧

| サービス | リソース | スペック | 月額概算 |
|---------|---------|---------|---------|
| ECS Fargate | API タスク | 0.5vCPU / 1GB RAM × 1-4 | $30-120 |
| ECS Fargate | Meilisearch | 0.5vCPU / 1GB RAM × 1 | $30 |
| RDS MySQL | db.t3.medium | 2vCPU / 4GB RAM / 50GB | $80 |
| ElastiCache | cache.t3.micro | 1vCPU / 0.5GB RAM | $15 |
| ALB | Application LB | - | $25 |
| S3 | アセット・画像 | 従量課金 | $5 |
| CloudFront | CDN | 従量課金 | $10 |
| Route 53 | DNS | ホストゾーン | $1 |
| ACM | SSL証明書 | 無料 | $0 |
| SES | メール送信 | 従量課金 | $5 |
| CloudWatch | ログ・監視 | 従量課金 | $10 |
| **Vercel** | **Frontend** | **Pro Plan** | **$20** |
| **合計** | | | **約 $230-330/月** |

---

## 2. Docker 構成（ローカル開発）

### 2.1 docker-compose.yml

```yaml
services:
  # === Frontend (Next.js) ===
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - ./frontend:/app
      - /app/node_modules
      - /app/.next
    environment:
      - NEXT_PUBLIC_API_URL=http://localhost:8000
      - WATCHPACK_POLLING=true
    depends_on:
      - api

  # === Backend API (Laravel) ===
  api:
    build:
      context: ./backend
      dockerfile: Dockerfile
    ports:
      - "8000:8000"
    volumes:
      - ./backend:/var/www/html
      - /var/www/html/vendor
    environment:
      - APP_ENV=local
      - APP_DEBUG=true
      - APP_URL=http://localhost:8000
      - FRONTEND_URL=http://localhost:3000
      - DB_CONNECTION=mysql
      - DB_HOST=db
      - DB_PORT=3306
      - DB_DATABASE=mitsupro
      - DB_USERNAME=mitsupro
      - DB_PASSWORD=secret
      - REDIS_HOST=redis
      - REDIS_PORT=6379
      - MEILISEARCH_HOST=http://meilisearch:7700
      - MEILISEARCH_KEY=masterKey
      - MAIL_MAILER=smtp
      - MAIL_HOST=mailpit
      - MAIL_PORT=1025
      - SESSION_DOMAIN=localhost
      - SANCTUM_STATEFUL_DOMAINS=localhost:3000
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
      meilisearch:
        condition: service_started
    command: php artisan serve --host=0.0.0.0 --port=8000

  # === Queue Worker ===
  queue:
    build:
      context: ./backend
      dockerfile: Dockerfile
    volumes:
      - ./backend:/var/www/html
      - /var/www/html/vendor
    environment:
      - APP_ENV=local
      - DB_HOST=db
      - DB_DATABASE=mitsupro
      - DB_USERNAME=mitsupro
      - DB_PASSWORD=secret
      - REDIS_HOST=redis
    depends_on:
      - api
    command: php artisan queue:work redis --sleep=3 --tries=3

  # === MySQL ===
  db:
    image: mysql:8.0
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: mitsupro
      MYSQL_USER: mitsupro
      MYSQL_PASSWORD: secret
    volumes:
      - db_data:/var/lib/mysql
      - ./backend/docker/mysql/my.cnf:/etc/mysql/conf.d/my.cnf
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

  # === Redis ===
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

  # === Meilisearch ===
  meilisearch:
    image: getmeili/meilisearch:v1.6
    ports:
      - "7700:7700"
    environment:
      - MEILI_MASTER_KEY=masterKey
      - MEILI_ENV=development
    volumes:
      - meilisearch_data:/meili_data

  # === Mailpit (メール確認用) ===
  mailpit:
    image: axllent/mailpit
    ports:
      - "8025:8025"  # Web UI
      - "1025:1025"  # SMTP

volumes:
  db_data:
  redis_data:
  meilisearch_data:
```

### 2.2 Backend Dockerfile

```dockerfile
FROM php:8.3-fpm

RUN apt-get update && apt-get install -y \
    git curl zip unzip libpng-dev libjpeg-dev libfreetype6-dev \
    libonig-dev libxml2-dev libzip-dev \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd zip opcache redis \
    && pecl install redis && docker-php-ext-enable redis

COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

WORKDIR /var/www/html

COPY . .

RUN composer install --no-dev --optimize-autoloader --no-interaction

RUN chown -R www-data:www-data storage bootstrap/cache

EXPOSE 8000

CMD ["php", "artisan", "serve", "--host=0.0.0.0", "--port=8000"]
```

### 2.3 Frontend Dockerfile.dev

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package.json pnpm-lock.yaml ./
RUN corepack enable && pnpm install --frozen-lockfile

COPY . .

EXPOSE 3000

CMD ["pnpm", "dev"]
```

---

## 3. CI/CD パイプライン

### 3.1 GitHub Actions - Backend CI

```yaml
# .github/workflows/backend-ci.yml
name: Backend CI

on:
  push:
    branches: [main, develop]
    paths: ['backend/**']
  pull_request:
    branches: [main, develop]
    paths: ['backend/**']

defaults:
  run:
    working-directory: backend

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: shivammathur/setup-php@v2
        with:
          php-version: '8.3'
          tools: composer

      - run: composer install --no-interaction --prefer-dist
      - run: vendor/bin/pint --test
      - run: vendor/bin/phpstan analyse --memory-limit=512M

  test:
    runs-on: ubuntu-latest
    needs: lint

    services:
      mysql:
        image: mysql:8.0
        env:
          MYSQL_ROOT_PASSWORD: root
          MYSQL_DATABASE: testing
        ports: ['3306:3306']
        options: >-
          --health-cmd="mysqladmin ping"
          --health-interval=10s
          --health-timeout=5s
          --health-retries=5

      redis:
        image: redis:7-alpine
        ports: ['6379:6379']

    steps:
      - uses: actions/checkout@v4
      - uses: shivammathur/setup-php@v2
        with:
          php-version: '8.3'
          extensions: mbstring, xml, pdo_mysql, redis
          coverage: xdebug

      - run: composer install --no-interaction --prefer-dist
      - run: cp .env.testing.example .env.testing
      - run: php artisan key:generate --env=testing

      - name: Run Tests
        run: vendor/bin/pest --coverage --min=70
        env:
          DB_CONNECTION: mysql
          DB_HOST: 127.0.0.1
          DB_DATABASE: testing
          DB_USERNAME: root
          DB_PASSWORD: root
          REDIS_HOST: 127.0.0.1

  deploy-staging:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/develop' && github.event_name == 'push'
    environment: staging

    steps:
      - uses: actions/checkout@v4
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-northeast-1

      - uses: aws-actions/amazon-ecr-login@v2
        id: ecr-login

      - name: Build & Push Docker Image
        run: |
          docker build -t ${{ steps.ecr-login.outputs.registry }}/mitsupro-api:${{ github.sha }} .
          docker push ${{ steps.ecr-login.outputs.registry }}/mitsupro-api:${{ github.sha }}

      - name: Deploy to ECS Staging
        run: |
          aws ecs update-service \
            --cluster mitsupro-staging \
            --service mitsupro-api \
            --force-new-deployment

  deploy-production:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    environment: production

    steps:
      - uses: actions/checkout@v4
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-northeast-1

      - uses: aws-actions/amazon-ecr-login@v2
        id: ecr-login

      - name: Build & Push Docker Image
        run: |
          docker build -t ${{ steps.ecr-login.outputs.registry }}/mitsupro-api:${{ github.sha }} .
          docker build -t ${{ steps.ecr-login.outputs.registry }}/mitsupro-api:latest .
          docker push ${{ steps.ecr-login.outputs.registry }}/mitsupro-api:${{ github.sha }}
          docker push ${{ steps.ecr-login.outputs.registry }}/mitsupro-api:latest

      - name: Run Migrations
        run: |
          aws ecs run-task \
            --cluster mitsupro-production \
            --task-definition mitsupro-migrate \
            --launch-type FARGATE

      - name: Deploy to ECS Production
        run: |
          aws ecs update-service \
            --cluster mitsupro-production \
            --service mitsupro-api \
            --force-new-deployment
```

### 3.2 GitHub Actions - Frontend CI

```yaml
# .github/workflows/frontend-ci.yml
name: Frontend CI

on:
  push:
    branches: [main, develop]
    paths: ['frontend/**']
  pull_request:
    branches: [main, develop]
    paths: ['frontend/**']

defaults:
  run:
    working-directory: frontend

jobs:
  lint-and-typecheck:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with:
          version: 9
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'
          cache-dependency-path: frontend/pnpm-lock.yaml

      - run: pnpm install --frozen-lockfile
      - run: pnpm lint
      - run: pnpm typecheck

  test:
    runs-on: ubuntu-latest
    needs: lint-and-typecheck
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with:
          version: 9
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'
          cache-dependency-path: frontend/pnpm-lock.yaml

      - run: pnpm install --frozen-lockfile
      - run: pnpm test:ci

  e2e:
    runs-on: ubuntu-latest
    needs: test
    if: github.event_name == 'push'
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with:
          version: 9
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'pnpm'
          cache-dependency-path: frontend/pnpm-lock.yaml

      - run: pnpm install --frozen-lockfile
      - run: npx playwright install --with-deps
      - run: pnpm build
      - run: pnpm test:e2e
```

---

## 4. 監視・アラート設計

### 4.1 監視ツール構成

| ツール | 用途 | 対象 |
|--------|------|------|
| Sentry | エラートラッキング | Frontend + Backend |
| UptimeRobot | 死活監視 | 全エンドポイント |
| CloudWatch | メトリクス・ログ | AWS リソース |
| Laravel Telescope | デバッグ | Backend (dev/staging) |
| Vercel Analytics | パフォーマンス | Frontend |

### 4.2 アラート設定

| 条件 | 重要度 | 通知先 |
|------|--------|--------|
| サイトダウン（5分以上） | Critical | Slack #alerts + メール |
| API エラー率 > 5% | High | Slack #alerts |
| API レスポンス p95 > 1s | Medium | Slack #monitoring |
| DB CPU > 80% | High | Slack #alerts |
| ディスク使用率 > 85% | Medium | Slack #monitoring |
| SSL証明書期限 30日以内 | Low | メール |

### 4.3 ログ設計

```
CloudWatch Log Groups:
├── /ecs/mitsupro-api          # Laravel アプリケーションログ
├── /ecs/mitsupro-queue        # キューワーカーログ
├── /ecs/mitsupro-meilisearch  # 検索エンジンログ
├── /rds/mitsupro-mysql        # DB スロークエリログ
└── /alb/mitsupro              # ALB アクセスログ
```

---

## 5. バックアップ・DR 設計

### 5.1 バックアップ戦略

| 対象 | 方法 | 頻度 | 保持期間 |
|------|------|------|---------|
| RDS MySQL | 自動スナップショット | 日次 | 7日間 |
| RDS MySQL | 手動スナップショット | リリース前 | 30日間 |
| S3 アセット | バージョニング | リアルタイム | 30日間 |
| Meilisearch | ダンプ → S3 | 日次 | 7日間 |
| Redis | RDB スナップショット | 1時間ごと | 24時間 |

### 5.2 ロールバック手順

```
1. ECS タスク定義を前バージョンに戻す
   aws ecs update-service --cluster mitsupro-production \
     --service mitsupro-api \
     --task-definition mitsupro-api:<previous-revision>

2. DB マイグレーションのロールバック（必要時）
   php artisan migrate:rollback --step=1

3. フロントエンドのロールバック
   Vercel Dashboard → Deployments → 前バージョンを Promote
```

---

## 6. セキュリティ設計

### 6.1 ネットワークセキュリティ

```
VPC (10.0.0.0/16)
├── Public Subnet (10.0.1.0/24, 10.0.2.0/24)
│   └── ALB, NAT Gateway
├── Private Subnet - App (10.0.10.0/24, 10.0.11.0/24)
│   └── ECS Fargate Tasks
└── Private Subnet - Data (10.0.20.0/24, 10.0.21.0/24)
    └── RDS, ElastiCache
```

### 6.2 セキュリティグループ

| SG | インバウンド | ソース |
|----|------------|--------|
| ALB SG | 443 (HTTPS) | 0.0.0.0/0 |
| ECS SG | 8000 | ALB SG |
| RDS SG | 3306 | ECS SG |
| Redis SG | 6379 | ECS SG |
| Meilisearch SG | 7700 | ECS SG |

### 6.3 シークレット管理

| シークレット | 管理場所 |
|------------|---------|
| DB パスワード | AWS Secrets Manager |
| API キー | GitHub Secrets + ECS タスク定義 |
| Laravel APP_KEY | AWS Secrets Manager |
| SES 認証情報 | AWS IAM Role |
| Meilisearch マスターキー | AWS Secrets Manager |
