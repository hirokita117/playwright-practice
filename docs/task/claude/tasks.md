# Playwright 実践学習ロードマップ

## プロジェクト構造
```
playwright-practice/
├── tasks/                    # 各ステップのタスクファイル
│   ├── step1-basics/
│   ├── step2-intermediate/
│   ├── step3-advanced/
│   └── step4-real-world/
├── src/
│   ├── tests/               # テストファイル
│   ├── pages/               # Page Object Model
│   ├── utils/               # ユーティリティ関数
│   └── fixtures/            # カスタムフィクスチャ
├── test-data/               # テストデータ
└── reports/                 # レポート出力先
```

---

## 📚 Step 1: 基礎編（1-2週間）

### Task 1.1: 基本的なナビゲーションとアサーション
**ファイル:** `tasks/step1-basics/task-1-1-navigation.md`

```markdown
# Task 1.1: 基本的なナビゲーションとアサーション

## 目標
Playwrightの基本的な操作方法を習得する

## 要件
1. Google検索を自動化するテストを作成
2. 検索結果の確認
3. スクリーンショットの取得

## 実装すべきファイル
- `src/tests/basic/google-search.spec.ts`
- `playwright.config.ts` の基本設定

## テストケース
- [ ] Googleホームページにアクセス
- [ ] "Playwright" を検索
- [ ] 検索結果に "Playwright" が含まれることを確認
- [ ] スクリーンショットを保存

## ヒント
- page.goto() でページ遷移
- page.fill() で入力
- page.click() でクリック
- expect() でアサーション
```

### Task 1.2: セレクターの習得
**ファイル:** `tasks/step1-basics/task-1-2-selectors.md`

```markdown
# Task 1.2: 様々なセレクターの使い方

## 目標
CSS、XPath、Text、Role セレクターを理解する

## 要件
Todoアプリ（https://todomvc.com/examples/react/）でタスク管理を自動化

## 実装すべきファイル
- `src/tests/basic/todo-app.spec.ts`

## テストケース
- [ ] 新しいタスクを3つ追加
- [ ] 特定のタスクを完了にマーク
- [ ] 完了したタスクをフィルタリング
- [ ] タスクを削除

## 学習ポイント
- getByRole() の活用
- getByText() での要素取得
- nth() での複数要素の処理
- data-testid の利用
```

### Task 1.3: 待機処理とタイムアウト
**ファイル:** `tasks/step1-basics/task-1-3-waits.md`

````markdown
# Task 1.3: 待機処理とタイムアウト

## 目標
動的コンテンツの処理方法を習得

## 要件
遅延ローディングのあるサイトでのテスト

## 実装すべきファイル
- `src/tests/basic/dynamic-content.spec.ts`
- `src/utils/wait-helpers.ts`

## テストケース
- [ ] 明示的な待機（waitForSelector）
- [ ] 条件付き待機（waitForLoadState）
- [ ] カスタム待機関数の実装
- [ ] タイムアウトエラーの処理

## 実装例の骨子
```typescript
// wait-helpers.ts
export async function waitForElement(page, selector, timeout = 5000) {
  // 実装
}
```
````

---

## 🚀 Step 2: 中級編（2-3週間）

### Task 2.1: Page Object Model (POM)
**ファイル:** `tasks/step2-intermediate/task-2-1-pom.md`

````markdown
# Task 2.1: Page Object Model の実装

## 目標
保守性の高いテスト構造を構築

## 要件
ECサイトのテストをPOMパターンで実装

## 実装すべきファイル
- `src/pages/LoginPage.ts`
- `src/pages/ProductPage.ts`
- `src/pages/CartPage.ts`
- `src/tests/e2e/shopping-flow.spec.ts`

## クラス設計
```typescript
// LoginPage.ts の構造
export class LoginPage {
  constructor(private page: Page) {}
  
  async login(email: string, password: string) {
    // 実装
  }
  
  async verifyLoginSuccess() {
    // 実装
  }
}
```

## テストシナリオ
1. ログイン
2. 商品検索
3. カートに追加
4. チェックアウト
````

### Task 2.2: データ駆動テスト
**ファイル:** `tasks/step2-intermediate/task-2-2-data-driven.md`

````markdown
# Task 2.2: データ駆動テストの実装

## 目標
複数のテストデータでテストを効率化

## 要件
ユーザー登録フォームの包括的なテスト

## 実装すべきファイル
- `test-data/users.json`
- `test-data/invalid-inputs.csv`
- `src/tests/data-driven/registration.spec.ts`
- `src/utils/data-reader.ts`

## データ形式例
```json
// users.json
{
  "validUsers": [
    {
      "name": "山田太郎",
      "email": "yamada@example.com",
      "age": 25
    }
  ],
  "invalidUsers": [...]
}
```

## テスト実装
- パラメータ化テスト
- CSVファイルからのデータ読み込み
- 境界値テスト
````

### Task 2.3: API連携テスト
**ファイル:** `tasks/step2-intermediate/task-2-3-api-integration.md`

````markdown
# Task 2.3: API連携とモック

## 目標
APIレスポンスの制御とE2Eテストの統合

## 要件
APIモックを使用したユーザーダッシュボードのテスト

## 実装すべきファイル
- `src/tests/api/api-mock.spec.ts`
- `src/fixtures/api-responses.ts`
- `src/utils/api-helper.ts`

## 実装内容
1. APIレスポンスのインターセプト
2. エラーレスポンスのシミュレーション
3. レート制限のテスト
4. 実APIとの統合テスト

## コード例
```typescript
await page.route('**/api/users', route => {
  route.fulfill({
    status: 200,
    body: JSON.stringify(mockData)
  });
});
```
````

---

## 💪 Step 3: 上級編（3-4週間）

### Task 3.1: 並列実行とシャーディング
**ファイル:** `tasks/step3-advanced/task-3-1-parallel.md`

```markdown
# Task 3.1: テストの並列実行最適化

## 目標
大規模テストスイートの効率的な実行

## 要件
50個以上のテストケースを並列実行

## 実装すべきファイル
- `playwright.config.ts` (並列設定)
- `src/tests/parallel/test-suite-*.spec.ts`
- `.github/workflows/parallel-tests.yml`

## 設定項目
- ワーカー数の最適化
- シャーディング戦略
- リトライ設定
- テストの依存関係管理

## パフォーマンス目標
- 単体実行: 10分 → 並列実行: 2分以内
```

### Task 3.2: カスタムレポーターとメトリクス
**ファイル:** `tasks/step3-advanced/task-3-2-reporting.md`

````markdown
# Task 3.2: カスタムレポーターの実装

## 目標
詳細なテストメトリクスの収集と可視化

## 要件
Slack通知とダッシュボード連携

## 実装すべきファイル
- `src/reporters/custom-reporter.ts`
- `src/reporters/slack-notifier.ts`
- `src/utils/metrics-collector.ts`

## レポート内容
- テスト実行時間の分析
- 失敗率の追跡
- スクリーンショット付きエラーレポート
- パフォーマンスメトリクス

## 実装例
```typescript
class CustomReporter implements Reporter {
  onTestEnd(test, result) {
    // メトリクス収集
    // Slack通知
  }
}
```
````

### Task 3.3: ビジュアルリグレッションテスト
**ファイル:** `tasks/step3-advanced/task-3-3-visual.md`

````markdown
# Task 3.3: ビジュアルリグレッションテスト

## 目標
UIの視覚的な変更を自動検出

## 要件
コンポーネントライブラリのビジュアルテスト

## 実装すべきファイル
- `src/tests/visual/components.spec.ts`
- `src/utils/visual-config.ts`
- `visual-baseline/` (ベースライン画像)

## テスト項目
- コンポーネントのスナップショット
- レスポンシブデザインのテスト
- ダークモード対応
- 差分検出と承認フロー

## 設定
```typescript
await expect(page).toHaveScreenshot('button-primary.png', {
  maxDiffPixels: 100,
  threshold: 0.2
});
```
````

---

## 🏢 Step 4: 実務応用編（4-5週間）

### Task 4.1: CI/CDパイプライン統合
**ファイル:** `tasks/step4-real-world/task-4-1-cicd.md`

```markdown
# Task 4.1: CI/CDパイプラインの構築

## 目標
自動テストの完全自動化

## 要件
GitHub Actions/GitLab CI での実装

## 実装すべきファイル
- `.github/workflows/e2e-tests.yml`
- `scripts/test-runner.sh`
- `docker/Dockerfile.playwright`

## パイプライン構成
1. PRトリガーでのテスト実行
2. ステージング環境へのデプロイ後テスト
3. 本番環境のスモークテスト
4. 定期実行のリグレッションテスト

## 機能要件
- テスト結果のアーティファクト保存
- 失敗時の自動リトライ
- 並列実行の最適化
```

### Task 4.2: マルチブラウザ・デバイステスト
**ファイル:** `tasks/step4-real-world/task-4-2-cross-browser.md`

````markdown
# Task 4.2: クロスブラウザテスト戦略

## 目標
全主要ブラウザ・デバイスでの品質保証

## 要件
実機に近い環境でのテスト

## 実装すべきファイル
- `src/tests/cross-browser/compatibility.spec.ts`
- `config/browsers.config.ts`
- `config/devices.config.ts`

## テスト対象
- Chrome, Firefox, Safari, Edge
- iPhone, iPad, Android
- 異なる画面解像度

## 実装ポイント
```typescript
const projects = [
  { name: 'Chrome', use: { ...devices['Desktop Chrome'] } },
  { name: 'Mobile Safari', use: { ...devices['iPhone 13'] } },
];
```
````

### Task 4.3: 実プロジェクトシミュレーション
**ファイル:** `tasks/step4-real-world/task-4-3-real-project.md`

````markdown
# Task 4.3: ECサイトの完全E2Eテストスイート

## 目標
実際のプロジェクトレベルのテスト実装

## 要件
架空のECサイトの全機能テスト

## 実装すべきファイル構造
```
src/
├── tests/
│   ├── smoke/           # スモークテスト
│   ├── regression/      # リグレッションテスト
│   ├── e2e/            # E2Eシナリオ
│   └── performance/     # パフォーマンステスト
├── pages/               # 20+ ページオブジェクト
├── components/          # 再利用可能コンポーネント
├── fixtures/           # テストフィクスチャ
└── utils/              # ヘルパー関数
```

## 実装機能
1. ユーザー認証フロー
2. 商品検索・フィルタリング
3. カート機能
4. 決済プロセス
5. 注文管理
6. レビュー投稿

## 非機能要件
- テスト実行時間: 全体で15分以内
- カバレッジ: クリティカルパス100%
- メンテナンス性: DRY原則の徹底
````

---

## 📊 進捗管理とレビュー

### 各ステップ完了時のチェックリスト

**Step 1 完了時**
- [ ] 基本的なPlaywrightコマンドを理解
- [ ] セレクターの使い分けができる
- [ ] 待機処理を適切に実装できる

**Step 2 完了時**
- [ ] POMパターンでテストを構造化できる
- [ ] データ駆動テストを実装できる
- [ ] APIモックを使用できる

**Step 3 完了時**
- [ ] 並列実行を最適化できる
- [ ] カスタムレポートを作成できる
- [ ] ビジュアルテストを実装できる

**Step 4 完了時**
- [ ] CI/CDにテストを統合できる
- [ ] マルチブラウザテストを実行できる
- [ ] 大規模プロジェクトのテストを設計できる

---

## 🤖 AI (gemini-cli) での開発の進め方

各タスクファイルをgemini-cliに読み込ませる際のプロンプト例：

```bash
# タスクファイルを読み込んで実装を依頼
gemini-cli "tasks/step1-basics/task-1-1-navigation.mdを読んで、
要件に従ってテストファイルを実装してください。
Playwrightのベストプラクティスに従ってください。"

# レビューを依頼
gemini-cli "src/tests/basic/google-search.spec.ts のコードをレビューして、
改善点があれば指摘してください。"

# 次のステップへの移行
gemini-cli "Step 1のタスクが完了しました。
Step 2のPOMパターンの実装について、
具体的なコード例を含めて説明してください。"
```

---

## 📚 推奨リソース

- [Playwright公式ドキュメント](https://playwright.dev/)
- [Playwright Best Practices](https://playwright.dev/docs/best-practices)
- [テスト自動化のアンチパターン](https://github.com/testautomation/anti-patterns)

---

## 💡 Tips

1. **段階的に難易度を上げる**: 基礎をしっかり固めてから次へ
2. **実際のWebサイトを使う**: TodoMVC、GitHub、実在のECサイトなど
3. **エラーから学ぶ**: デバッグモードを活用
4. **コードレビュー**: AIやコミュニティでフィードバックを得る
5. **定期的なリファクタリング**: 技術的負債を溜めない
