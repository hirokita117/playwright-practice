
# 追加するファイル一覧（コピペで作成OK）

### 1) ロードマップ

`ROADMAP.md`

```md
# Playwright Practice Roadmap

## 概要
本ロードマップは「実務投入できる Playwright スキル」を最短で得るための12ステップです。
各タスクは人間/AIのどちらでも実行可能な粒度で書かれています。

## ステップ一覧
1. 01_basics: 最初のE2Eテスト & Web-First Assertions
2. 02_locators: 安定したロケータ設計とテスト作法
3. 03_projects_emulation: マルチブラウザ/モバイルエミュレーション
4. 04_trace_debug: トレース/レポート/UIモードでのデバッグ
5. 05_fixtures_config: フィクスチャ/共通設定/タグ運用
6. 06_auth_storage: 認証とstorageStateの再利用
7. 07_network_mock: ルーティング/ HAR リプレイでのAPIモック
8. 08_api_testing: API単体テスト（request fixture）
9. 09_visual_regression: スナップショット/ビジュアル比較
10. 10_parallel_ci: 並列/シャーディングとGitHub Actions
11. 11_pom: Page Object Model とテストの保守性
12. 12_a11y_downloads: アクセシビリティ/ダウンロード・アップロード/権限

> 進め方: `tasks/`配下を順に実施。AIに任せる場合は `ai/AGENT_GUIDE.md` を参照。
```

### 2) AIエージェント向けガイド

`ai/AGENT_GUIDE.md`

```md
# Agent Guide for Gemini CLI

## Mission
- `tasks/`配下のステップを順番に実行し、テスト/設定/CIファイルを生成・修正する。
- 変更は最小コミット単位でPRを作り、人間がレビューしやすい差分を保つ。

## Ground Rules
- Node 18+ / PNPM or NPM を前提。依存導入後は `npx playwright install --with-deps` を実行。
- 生成物は TypeScript 前提（`.ts`）。`@playwright/test` を使用。
- すべてのテストは `tests/` 以下に配置し、ステップ番号のサブフォルダで管理。
- セレクタはロール/ラベル優先（`getByRole`/`getByLabel`/`getByPlaceholder`）。
- 失敗時のトレース/スクショ/動画を常に添付（configで設定）。
- 実行コマンド例:
  - `npx playwright test --ui`（デバッグ）
  - `npx playwright show-report`（HTMLレポート）
  - `npx playwright test --project=chromium --debug`（特定プロジェクト）

## Deliverables Convention
- 各タスクの「成果物」は必ずコミット。コミットメッセージ: `feat(step-XX): ...`
- 受け入れ基準の自動チェックを `package.json` の `test:step:XX` スクリプトで提供。
```

### 3) Playwright 設定（実務寄り既定値）

`playwright.config.ts`

```ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  timeout: 30_000,
  expect: { timeout: 5_000 },
  fullyParallel: true,
  retries: process.env.CI ? 2 : 0,
  reporter: [['html', { open: process.env.CI ? 'never' : 'on-failure' }], ['list']],
  use: {
    baseURL: 'https://demo.playwright.dev',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    locale: 'ja-JP',
    timezoneId: 'Asia/Tokyo',
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox',  use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit',   use: { ...devices['Desktop Safari'] } },
    // モバイルエミュレーション例
    { name: 'Mobile Safari', use: { ...devices['iPhone 13'] } },
  ],
  // GH Actions のシャーディング等はステップ10で説明
});
```

### 4) サンプル最初のテスト

`tests/01_basics/todomvc.spec.ts`

```ts
import { test, expect } from '@playwright/test';

test('TodoMVC: 追加/完了/フィルタ', async ({ page }) => {
  await page.goto('/todomvc');
  const input = page.getByPlaceholder('What needs to be done?');
  await input.fill('Buy milk');
  await input.press('Enter');
  await input.fill('Write tests');
  await input.press('Enter');

  const list = page.getByRole('list', { name: 'Todo list' });
  await expect(list.getByRole('listitem')).toHaveCount(2);

  await list.getByRole('listitem', { name: /Buy milk/ }).getByRole('checkbox').check();
  await page.getByRole('link', { name: 'Completed' }).click();
  await expect(page.getByRole('listitem')).toHaveText(/Buy milk/);
});
```

---

# `tasks/`（各ステップの実行書）

> すべて **人/AIが読める単位**で書いています。必要に応じて gemini-cli にそのまま読ませてください。

`tasks/01_basics.md`

```md
# 01_basics — 最初のE2E & Web-First Assertions
## ゴール
- Playwright の基本APIと「自動待機するアサーション」を体感する。

## 作業手順
1) `tests/01_basics/todomvc.spec.ts` を実行: `npx playwright test tests/01_basics --ui`
2) 失敗時にHTMLレポート/トレースを開く: `npx playwright show-report`
3) アサーションを意図的に壊して flakiness を観察し、`expect(...).toHave*` に置き換えて安定化。

## 成果物
- 既存 spec のパス/失敗ログ、レポートへのリンク（CI化前はローカルでOK）

## 受け入れ基準
- UIモードでテストが緑になること
- `toHaveText/toHaveCount` 等の **Web-First Assertions** を最低3種使用
```

（根拠: Web-First Assertions/Locator ドキュメント。 ([Playwright][1])）

`tasks/02_locators.md`

```md
# 02_locators — 安定したロケータ設計
## ゴール
- `getByRole` 等のセマンティックロケータと自動待機の仕組みを理解。

## 作業手順
- `tests/02_locators/` を作成し、ARIAロール/ラベル/プレースホルダを使って同等操作のテストを再実装。
- CSS/XPath 依存を避け、アクセシブルネームを優先。
- 「待ち」を書かないこと（Playwright が待つ）。

## 成果物
- `tests/02_locators/*.spec.ts`

## 受け入れ基準
- CSSセレクタを1箇所以外で使わない（例外はダイナミック要素に限る）
- `getByRole/getByLabel/getByPlaceholder` を活用
```

（根拠: Locator/Best Practices。 ([Playwright][2]))

`tasks/03_projects_emulation.md`

```md
# 03_projects_emulation — 複数ブラウザ/モバイルエミュレーション
## ゴール
- Chromium/Firefox/WebKit とモバイルデバイスで同一テストを実行。

## 作業手順
- `playwright.config.ts` の `projects` を確認/調整（既に雛形あり）。
- `npx playwright test --project="Mobile Safari"` でモバイル実行。
- 任意で `geolocation/permissions` を `test.use` に追加し、地理情報や通知許可を扱う簡易テストを追加。

## 成果物
- `tests/03_projects_emulation/*.spec.ts`

## 受け入れ基準
- 4プロジェクト以上で同一テストが緑
- 任意の1テストで `permissions` or `geolocation` を利用
```

（根拠: Projects/Emulation/Use options/Permissions。 ([Playwright][3]))

`tasks/04_trace_debug.md`

```md
# 04_trace_debug — トレース/HTMLレポート/UIモード
## ゴール
- 失敗の再現性が低いケースを **Trace Viewer** で解析できる。

## 作業手順
- 失敗するテストを1つ作る（わざと）。
- `trace: 'on-first-retry'` 設定で実行 → 失敗後にトレースを開いて原因特定。
- `--ui` と `--debug` を使った調査手順をREADMEに残す。

## 成果物
- 調査手順のドキュメント化
- トレース/HTMLレポート出力

## 受け入れ基準
- 再現手順と原因仮説/暫定対策が `docs/debugging.md` に記述されている
```

（根拠: Trace Viewer/Tracing/Running & Debug/UIモード/HTMLレポータ。 ([Playwright][4]))

`tasks/05_fixtures_config.md`

```md
# 05_fixtures_config — フィクスチャ/共通設定/タグ
## ゴール
- テスト環境の初期化を fixture で再利用。タグでサブセット実行。

## 作業手順
- `tests/fixtures/test-base.ts` を作り、共通の初期化を提供。
- テストに `@smoke @regression` などのタグを付与し、`--grep @smoke` での実行を確認。

## 成果物
- `tests/05_fixtures_config/*.spec.ts`
- `tests/fixtures/test-base.ts`

## 受け入れ基準
- フィクスチャを2つ以上定義（例: 認証済みページ、APIクライアント）
- タグでの絞り込み実行が可能
```

（根拠: Fixtures/Configuration/Tag tests API。 ([Playwright][5]))

`tasks/06_auth_storage.md`

```md
# 06_auth_storage — 認証と storageState
## ゴール
- ログインを1回だけ実行し、以降のテストで `storageState` を再利用。

## 作業手順
- `tests/setup/auth.setup.ts` を作成し、ログイン→ `playwright/.auth/user.json` を保存。
- 通常テストでは `use: { storageState: 'playwright/.auth/user.json' }` を設定。
- 機密値は `.env` と GitHub Secrets で管理。

## 成果物
- `tests/setup/auth.setup.ts`
- `playwright/.auth/user.json`（gitignore）

## 受け入れ基準
- ログイン不要で保護ページのE2Eテストが動作
```

（根拠: Authentication/Storage state ガイド。 ([Playwright][6]))

`tasks/07_network_mock.md`

```md
# 07_network_mock — ルーティング/ HAR リプレイ
## ゴール
- 外部依存（API/第三者サービス）をモックして安定化。

## 作業手順
- `route` で特定リソースを `abort/fulfill`。
- `routeFromHAR` を使って `hars/` のHARを記録→編集→リプレイ。
- モック時と実API時の二系統テストを `@mock` / `@live` タグで切替。

## 成果物
- `hars/*.har` と `tests/07_network_mock/*.spec.ts`

## 受け入れ基準
- モック時にネットワーク未接続でも緑
- HAR変更が期待通り画面に反映
```

（根拠: Network/Mock APIs/HAR リプレイ。 ([Playwright][7]))

`tasks/08_api_testing.md`

```md
# 08_api_testing — request fixtureでAPI単体テスト
## ゴール
- 画面を立ち上げずにAPIのCRUDを検証。

## 作業手順
- `tests/08_api_testing/api.spec.ts` を作成し、`request` でJSONPlaceholderやReqResに対してGET/POST等を実装。
- `baseURL` や `extraHTTPHeaders` を設定してテストを簡潔に。

## 成果物
- `tests/08_api_testing/api.spec.ts`

## 受け入れ基準
- 2つ以上のエンドポイントに対し、200/4xx系の期待を検証
```

（根拠: API testing（request fixture）, JSONPlaceholder/ReqRes。 ([Playwright][8], [jsonplaceholder.typicode.com][9]))

`tasks/09_visual_regression.md`

```md
# 09_visual_regression — スクショ/スナップショット比較
## ゴール
- `expect(page).toHaveScreenshot()` による見た目の崩れ検知。

## 作業手順
- 重要画面の初回実行で基準画像を作成。
- 動的要素を `mask` などで除外し、誤検知を抑制。

## 成果物
- `tests/09_visual_regression/*.spec.ts`
- `__screenshots__` 基準画像

## 受け入れ基準
- 画面の小変更でテストが意図通り失敗/更新できる
```

（根拠: Snapshots/Visual comparisons。 ([Playwright][10]))

`tasks/10_parallel_ci.md`

```md
# 10_parallel_ci — 並列/シャーディング & GitHub Actions
## ゴール
- CIでクロスブラウザを高速に回す（分散/レポート収集）。

## 作業手順
- `npx playwright test --shard=1/2` のローカル検証。
- `.github/workflows/playwright.yml` を追加し、Matrix で `chromium/firefox/webkit`、OSはubuntu固定でOK。
- 失敗時にトレース/レポートをアーティファクト収集。

## 成果物
- `.github/workflows/playwright.yml`

## 受け入れ基準
- PR時にCIが動き、レポート/トレースが閲覧可能
```

（根拠: CI 入門/CI/ Sharding ガイド/ 公式 GitHub Action。 ([Playwright][11], [GitHub][12]))

`tasks/11_pom.md`

```md
# 11_pom — Page Object Modelで保守性を上げる
## ゴール
- 画面ごとの操作/要素をクラス化し、テストを意図中心に。

## 作業手順
- `src/pom/TodoPage.ts` を作成（追加・完了・フィルタ等の操作API）。
- 既存テストの操作をPOMに置換。

## 成果物
- `src/pom/*.ts` と `tests/11_pom/*.spec.ts`

## 受け入れ基準
- テストからCSSセレクタ直書きが消え、POM経由で動作
```

（根拠: POM ドキュメント。 ([Playwright][13]))

`tasks/12_a11y_downloads.md`

```md
# 12_a11y_downloads — アクセシビリティ/ダウンロード・アップロード/権限
## ゴール
- axe-core 連携で自動a11yチェック、ファイルDL/ULと権限を扱う。

## 作業手順
- `@axe-core/playwright` を導入し、重大な違反が0であることをアサート。
- ダウンロードイベント/ `setInputFiles` でDL→ULの往復テストを実装。
- `grantPermissions` で通知/位置情報などの権限を付与し、振る舞いを検証。

## 成果物
- `tests/12_a11y/*.spec.ts`
- `tests/12_files/*.spec.ts`

## 受け入れ基準
- 主要ページのa11y違反ゼロ（自動検出範囲）
- DL→保存→UL まで自動化
```

（根拠: a11y（@axe-core/playwright）/Downloads/Permissions。 ([Playwright][14]))

---

# GitHub Actions（サンプル）

`.github/workflows/playwright.yml`

```yaml
name: Playwright Tests

on:
  pull_request:
  push:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        shard: [ '1/2', '2/2' ]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npm ci
      - run: npx playwright install --with-deps
      - name: Run tests (shard ${{ matrix.shard }})
        run: npx playwright test --shard=${{ matrix.shard }}
      - name: Upload HTML report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report-${{ matrix.shard }}
          path: playwright-report
      - name: Upload traces
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: traces-${{ matrix.shard }}
          path: test-results
```

（根拠: 公式CIガイド/公式Action/シャーディング。 ([Playwright][11], [GitHub][12]))

---

## 進める順序（目安）

* まず `01 → 04` で**安定実行とデバッグ**を身につける。
* `05 → 07 → 08` で**設計（fixtures/タグ）・外部依存の切り離し・API単体**。
* `09 → 10` で**ビジュアル検知とCI高速化**。
* `11 → 12` で\*\*保守性・周辺領域（a11y/ファイル/権限）\*\*を押さえて実務対応力に。

---

## 根拠（主要リファレンス）

* Web-First Assertions / Locators / Best Practices: ([Playwright][1])
* Projects（複数ブラウザ）/ Emulation / Use options / Permissions: ([Playwright][3])
* Trace Viewer / Tracing / UIモード / デバッグ: ([Playwright][4])
* Fixtures / Configuration / Tag tests: ([Playwright][5])
* Authentication（storageState）: ([Playwright][6])
* Network Mock / HAR: ([Playwright][7])
* API testing（request）: ([Playwright][8])
* Visual comparisons / Snapshots: ([Playwright][10])
* CI / GitHub Actions / Sharding: ([Playwright][11], [GitHub][12])
* POM: ([Playwright][13])
* Downloads / File upload: ([Playwright][15])
* a11y（@axe-core/playwright）: ([Playwright][14])

---

## 最初の一歩（やること）

1. 上記ファイルをそのまま追加＆コミット。
2. `npm ci && npx playwright install --with-deps`
3. `npx playwright test --ui` で `01_basics` を緑にする。
4. gemini-cli に `ai/AGENT_GUIDE.md` と `tasks/01_basics.md` を渡して自走させる。

ここまで進めれば、あとはタスクを順に消化するだけで、**実務投入OKレベルのPlaywright経験**が身につきます。必要なら、この先は各ステップのサンプルテストやPOM雛形も追加していきます！

[1]: https://playwright.dev/docs/test-assertions?utm_source=chatgpt.com "Assertions"
[2]: https://playwright.dev/docs/locators?utm_source=chatgpt.com "Locators"
[3]: https://playwright.dev/docs/test-projects?utm_source=chatgpt.com "Projects"
[4]: https://playwright.dev/docs/trace-viewer?utm_source=chatgpt.com "Trace viewer"
[5]: https://playwright.dev/docs/test-fixtures?utm_source=chatgpt.com "Fixtures"
[6]: https://playwright.dev/docs/auth?utm_source=chatgpt.com "Authentication"
[7]: https://playwright.dev/docs/network?utm_source=chatgpt.com "Network"
[8]: https://playwright.dev/docs/api-testing?utm_source=chatgpt.com "API testing"
[9]: https://jsonplaceholder.typicode.com/?utm_source=chatgpt.com "JSONPlaceholder - Free Fake REST API - Typicode"
[10]: https://playwright.dev/docs/test-snapshots?utm_source=chatgpt.com "Visual comparisons"
[11]: https://playwright.dev/docs/ci-intro?utm_source=chatgpt.com "Setting up CI"
[12]: https://github.com/microsoft/playwright-github-action?utm_source=chatgpt.com "microsoft/playwright-github-action"
[13]: https://playwright.dev/docs/pom?utm_source=chatgpt.com "Page object models"
[14]: https://playwright.dev/docs/accessibility-testing?utm_source=chatgpt.com "Accessibility testing"
[15]: https://playwright.dev/docs/api/class-download?utm_source=chatgpt.com "Download"
