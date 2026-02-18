# 複数ドメイン・LP運用における開発・管理方針

**最終更新**: 2026年2月18日  
**対象**: 開発チーム全員

---

## 1. 全体方針
- すべてのドメイン・LPは「一元的な運用・管理」を基本とし、属人化・重複管理を防ぐ
- サイト・LPごとに必要な独自要件がある場合も、共通基盤・共通ルールを優先
- セキュリティ・可用性・拡張性・保守性を最重視

---

## 2. ソースコード・リポジトリ管理

### 方針
- GitHub（またはGitLab）で全てのプロジェクトを管理
- LPごとにリポジトリを分ける場合も、共通ライブラリやCI/CD設定はテンプレート化
- ブランチ運用はGithubFlowを原則とし、feature/・hotfix/・release/ブランチを明確に運用
- コードレビューは必須、PR単位でCI自動実行

### 実践手順

#### 新規LP作成時のチェックリスト
- [ ] リポジトリ作成: `mitsukaru-<project>-lp`
- [ ] READMEテンプレート適用
- [ ] `.gitignore` 設定（node_modules, .env等）
- [ ] GitHub Actionsワークフロー設定
- [ ] Notionにプロジェクトページ作成
- [ ] ドメイン管理台帳に追加

---

## 3. DB・データ管理方針

### 方針
- LPごとにDBを分けず、共通DB＋LP識別子（source, campaign, page_id等）で一元管理
- データベース設計は拡張性・分析性を重視し、将来的な統合・分割にも柔軟に対応
- フォーム送信データはDB保存＋メール送信＋必要に応じて外部サービス連携
- 個人情報・重要データは暗号化・アクセス制御を徹底

### 実践手順

#### フォームデータテーブル設計例
```sql
CREATE TABLE form_submissions (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  source VARCHAR(50) NOT NULL COMMENT 'LP識別子（例: tax-lp01）',
  campaign VARCHAR(50) COMMENT 'キャンペーン名',
  name VARCHAR(100) NOT NULL,
  email VARCHAR(255) NOT NULL,
  phone VARCHAR(20),
  message TEXT,
  ip_address VARCHAR(45),
  user_agent TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_source (source),
  INDEX idx_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

#### フォーム送信処理の実装パターン（Laravel）
```php
// Controller
public function submit(Request $request)
{
    $validated = $request->validate([
        'name' => 'required|max:100',
        'email' => 'required|email',
        'phone' => 'nullable|regex:/^[0-9-]+$/',
        'message' => 'required',
    ]);

    // DB保存
    $submission = FormSubmission::create([
        'source' => 'tax-lp01', // LP識別子
        'campaign' => request()->input('utm_campaign'),
        'name' => $validated['name'],
        'email' => $validated['email'],
        'phone' => $validated['phone'],
        'message' => $validated['message'],
        'ip_address' => $request->ip(),
        'user_agent' => $request->userAgent(),
    ]);

    // メール送信
    Mail::to('info@mitsukaru-pro.co.jp')
        ->send(new FormSubmissionNotification($submission));

    return redirect()->back()->with('success', '送信完了しました');
}
```

---

## 4. インフラ・デプロイ

### 方針
- AWS等のクラウドサービスを活用し、インフラ構成はIaC（Infrastructure as Code）で管理
- LP・サイトごとにサーバーを分ける場合も、運用・監視は一元化
- デプロイはCI/CDパイプライン（GitHub Actions等）で自動化

### 実践手順

#### 推奨構成パターン

**パターン1: 静的LP（HTML/CSS/JS）**
- ホスティング: Netlify / Vercel / AWS S3 + CloudFront
- デプロイ: GitHub連携で自動デプロイ
- コスト: ほぼ無料〜数百円/月

**パターン2: 動的LP（Laravel / WordPress）**
- ホスティング: AWS EC2 / Lightsail / さくらVPS
- デプロイ: GitHub Actions + rsync/SSH
- コスト: 数千円/月〜

**パターン3: 共通アプリケーション内にルート追加**
```php
// routes/web.php
Route::prefix('lp01')->group(function () {
    Route::get('/', [LpController::class, 'taxLp01']);
    Route::post('/submit', [LpController::class, 'submitTaxLp01']);
});
```

---

## 5. デザイン・フロントエンド

### 方針
- Figmaでデザインを一元管理し、エンジニアはFigmaの開発者ツールを活用
- コーディング規約・コンポーネント設計は共通化し、再利用性を高める

### 実践手順

#### Figmaからのコーディング手順
1. **Figmaで開発モード（Dev Mode）に切り替え**
2. **要素を選択し、CSSをコピー**
   ```css
   /* Figma から自動生成 */
   .button-primary {
     background: #FF6B6B;
     border-radius: 8px;
     padding: 16px 32px;
   }
   ```
3. **Tailwind CSS に変換（推奨）**
   ```html
   <button class="bg-red-500 rounded-lg px-8 py-4">
     お問い合わせ
   </button>
   ```

#### 共通コンポーネントライブラリの作成
```
components/
├── buttons/
│   ├── primary-button.blade.php
│   └── secondary-button.blade.php
├── forms/
│   ├── input-field.blade.php
│   └── textarea.blade.php
└── sections/
    ├── hero.blade.php
    └── cta.blade.php
```

---

## 6. ドメイン・SSL・DNS管理

### 方針
- ドメイン・SSL証明書・DNSは一元管理台帳（Notion等）で管理
- 期限切れ・設定ミス防止のため、定期的な棚卸し・監査を実施

### 実践手順
→ 詳細は [`MULTI_DOMAIN_INFRA_MANAGEMENT_POLICY.md`](./MULTI_DOMAIN_INFRA_MANAGEMENT_POLICY.md) の「3. ドメイン・SSL・DNS管理」を参照

---

## 7. 運用・保守

### 方針
- サイト・LPごとに運用担当を明確化し、障害・問い合わせ対応フローを整備
- 定期的なセキュリティチェック・脆弱性診断を実施
- サイト・LPの稼働状況は監視ツール（例：UptimeRobot, Datadog等）で常時監視

### 実践手順

#### 月次メンテナンスチェックリスト
- [ ] 全LPにアクセスし、正常動作を確認
- [ ] フォーム送信テスト（メール受信確認）
- [ ] SSL証明書の有効期限確認（2ヶ月前に更新）
- [ ] Googleアナリティクスでアクセス数確認
- [ ] 依存パッケージの脆弱性チェック（`npm audit`, `composer audit`）

#### 障害対応フロー
1. **検知**: 監視アラート / ユーザー報告
2. **初動**: 影響範囲の特定、ステークホルダーへの連絡
3. **対応**: 
   - 軽微な問題: 即座に修正してデプロイ
   - 重大な問題: ロールバック → 調査 → 修正 → デプロイ
4. **事後**: ポストモーテム作成、再発防止策の実施

---

## 8. ドキュメント・ナレッジ共有

### 方針
- すべての運用ルール・開発手順・障害対応フローはNotion等でドキュメント化
- 新規LP・ドメイン追加時は必ず台帳・ドキュメントを更新

### 実践手順

#### Notion LP管理ページテンプレート
```
# [LP名] - 税理士求人LP01

## 基本情報
- URL: https://tax.mitsukaru-pro.co.jp/lp01/
- リポジトリ: https://github.com/mitsukaru/tax-lp01
- 公開日: 2026年2月1日
- 担当: バックエンドエンジニアA

## 技術スタック
- Laravel 8
- Tailwind CSS

## デプロイ方法
（手順を記載）

## フォーム送信先
- メール: info@mitsukaru-pro.co.jp
- DB: form_submissions テーブル

## トラッキング
- Google Analytics: UA-XXXXXXXX-X
- Google Tag Manager: GTM-XXXXXXX
```

---

## 9. コミュニケーション・改善

### 方針
- Slack等で日常的な情報共有、週次で定例ミーティング
- 振り返り・改善サイクルを短く回し、運用・開発方針も随時アップデート

### 実践手順

#### 週次定例ミーティングアジェンダ
1. 先週の進捗確認（各LP・プロジェクトごと）
2. 発生した問題・障害の共有
3. 今週のタスク確認
4. 改善提案・ディスカッション
5. ネクストアクション決定

---

**この方針により、複数ドメイン・LPの運用でも「一元管理・効率化・高品質・セキュリティ」を両立します。**
