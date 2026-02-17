# Nemielu 技術的負債 対応ロードマップ

**作成日**: 2026年2月13日
**作成者**: CTO / アーキテクチャレビュー

---

## 対応済み（2026/02/13）

### P0: セキュリティ（即時対応済み）

| 問題 | 対応内容 | コミット |
|---|---|---|
| `.env` ファイルがGitに追跡されていた | `git rm --cached` で追跡解除。DB/SMTPパスワード、APP_KEYが平文で履歴に残存 | `a3f045d` |
| `.env.testing` もGitに追跡されていた | 追跡解除。`.env.testing.example` をテンプレートとして作成 | `a3f045d` |

> **⚠️ 重要: 追加対応が必要**
> - `.env` に記載されていた以下の認証情報は**漏洩済み**とみなし、ローテーション（変更）を強く推奨
>   - `APP_KEY` (base64:IEXDqY2PZTQGI184zwTtkuL78cWL6XdjJFwDkfueiIc=)
>   - `DB_PASSWORD` (P-e1205ta57xmsa)
>   - `MAIL_PASSWORD` (A_e1205ta57xmsa)
> - Gitの履歴にはまだ残っているため、`git filter-branch` または `BFG Repo-Cleaner` での履歴書き換えを検討
> - リモートリポジトリがパブリックの場合は**即座に**認証情報を変更すること

### P1: コード品質・命名規則（即時対応済み）

| 問題 | 対応内容 | コミット |
|---|---|---|
| `Qestion.php` (タイポ) | `Question.php` にリネーム。テーブル名 `questions` と自動推測が一致 | `9bb6e2d` |
| `Beverage_content.php` (命名規則違反) | `BeverageContent.php` にリネーム。Laravel規約準拠 | `9bb6e2d` |
| `Analisis.php` / `PersonalAnalysisc` (デッドコード) | 完全に未使用の旧コードと判明、削除。`ReportController` の未使用importも除去 | `9bb6e2d` |
| テストレポートがGit対象 | `.gitignore` に `storage/test-reports/`, `public/test-reports/` を追加 | `9bb6e2d` |

---

## 今後の対応計画

### P1: テストカバレッジの強化（次回スプリント）

**現状**: 84テストケース。主要機能はカバーされているが以下が不足。

| 不足領域 | 詳細 | 優先度 |
|---|---|---|
| カフェイン機能 | `CaffeineController`, `BeverageController` のテスト不足 | 高 |
| 飲料管理モデル | `Beverage`, `BeverageContent`, `Caffeine_report` のモデルテスト | 高 |
| クライアント管理 | `ClientController` のテスト | 中 |
| `UserDataController` | 詳細テスト不足 | 中 |
| 眠気結果表示 | `SleepinessController` の結果表示テスト | 低 |
| E2Eテスト | Laravel Dusk によるブラウザテスト | 低 |

### P2: フロントエンド統一（計画的に実施）

**現状**: Tailwind CSS 2 + Bootstrap 5 + Alpine.js + Vue.js 3 が混在

**推奨方針**:
1. 新規画面は **Tailwind CSS + Alpine.js** で統一
2. 既存の Bootstrap 依存画面を段階的に Tailwind に移行
3. Vue.js 3 の使用箇所を調査し、Alpine.js で代替可能か判断
4. 最終的に Bootstrap と Vue.js の依存を削除

**見積り**: 2-3スプリント（影響範囲の大きさによる）

### P3: Laravel 8 → Laravel 10/11 アップグレード（長期計画）

**現状**: Laravel 8.65（2023年1月にEOL済み）

**リスク**:
- セキュリティパッチが提供されない
- 新しいPHPバージョンとの互換性問題
- 新しいパッケージが Laravel 8 をサポートしない可能性

**アップグレードパス**:
```
Laravel 8 → Laravel 9 → Laravel 10 → Laravel 11
PHP 8.0  →  PHP 8.1   →  PHP 8.1+  →  PHP 8.2+
```

**段階的な実施計画**:

#### Phase 1: 準備（1スプリント）
- [ ] テストカバレッジを70%以上に引き上げ
- [ ] 非推奨メソッドの洗い出し
- [ ] 依存パッケージの互換性チェック
- [ ] `composer.json` の各パッケージでサポートされるLaravelバージョンを確認

#### Phase 2: Laravel 8 → 9（1-2スプリント）
- [ ] PHP 8.1 にアップグレード
- [ ] `laravel/framework` を ^9.0 に更新
- [ ] Route ファイルのアップデート
- [ ] 非推奨の Eloquent メソッド修正
- [ ] テスト全通を確認

#### Phase 3: Laravel 9 → 10（1-2スプリント）
- [ ] `laravel/framework` を ^10.0 に更新
- [ ] Predis → PhpRedis への移行（該当する場合）
- [ ] テスト全通を確認

#### Phase 4: Laravel 10 → 11（1-2スプリント）
- [ ] PHP 8.2 にアップグレード
- [ ] `laravel/framework` を ^11.0 に更新
- [ ] 設定ファイル構成の変更対応
- [ ] テスト全通を確認

### P3: その他の改善事項

| 項目 | 詳細 | 優先度 |
|---|---|---|
| `Caffeine_report` モデル | `CaffeineReport` にリネーム（命名規則準拠） | 中 |
| `Userdata` モデル | `UserData` にリネーム（命名規則準拠） | 中 |
| `.env` のタイポ | `DEBUGBAR_ENABLED=ture` → `true` | 低 |
| Question モデルの fillable | `name, email, password` は Question に不適切（コピペミスの可能性） | 中 |
| dd() デバッグコードの残存 | `Analisis.php`（削除済み）以外にもコメントアウトされた dd() が散見 | 低 |
| CI/CD パイプライン | GitHub Actions でテスト自動実行の整備 | 中 |

---

## モデル命名規則 対応状況一覧

| 現在のモデル名 | 推測されるテーブル名 | 実際のテーブル名 | 状態 |
|---|---|---|---|
| `User` | `users` | `users` | ✅ OK |
| `Admin` | `admins` | `admins` | ✅ OK |
| ~~`Qestion`~~ `Question` | `questions` | `questions` | ✅ 修正済み |
| `Answer` | `answers` | `answers` | ✅ OK |
| `Report` | `reports` | `reports` | ✅ OK |
| `Beverage` | `beverages` | `beverages` | ✅ OK |
| ~~`Beverage_content`~~ `BeverageContent` | `beverage_contents` | `beverage_contents` | ✅ 修正済み |
| `Caffeine_report` | `caffeine_reports` | `caffeine_reports` | ⚠️ 要リネーム |
| `Client` | `clients` | `clients` | ✅ OK |
| `PvtReport` | `pvt_reports` | `pvt_reports` | ✅ OK |
| `PvtReactionTime` | `pvt_reaction_times` | `pvt_reaction_times` | ✅ OK |
| `Test` | `tests` | `tests` | ✅ OK |
| `Userdata` | `userdata` | `user_datas` | ⚠️ 要確認（テーブル名不一致の可能性） |

---

## 参考: コミット履歴

```
9bb6e2d refactor: モデル命名規則の修正とデッドコード削除
a3f045d security: .envファイルをGit追跡から除外
e2d1958 テストデータ作成、カバレッジ強化＆ci/cd導入
6f191eb テスト導入＆実行成功
```

---

**次回のアクション**: P1テストカバレッジ強化 → P3のCaffeine_report/Userdataリネーム → P2フロントエンド統一
