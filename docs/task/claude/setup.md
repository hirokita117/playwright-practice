```sh
#!/bin/bash

# Playwright Practice - 初期セットアップスクリプト

echo "🚀 Playwright Practice の初期セットアップを開始します..."

# 1. ディレクトリ構造の作成
echo "📁 ディレクトリ構造を作成中..."
mkdir -p tasks/step1-basics
mkdir -p tasks/step2-intermediate  
mkdir -p tasks/step3-advanced
mkdir -p tasks/step4-real-world
mkdir -p src/tests/basic
mkdir -p src/tests/e2e
mkdir -p src/tests/api
mkdir -p src/tests/parallel
mkdir -p src/tests/visual
mkdir -p src/tests/cross-browser
mkdir -p src/tests/smoke
mkdir -p src/tests/regression
mkdir -p src/tests/performance
mkdir -p src/tests/data-driven
mkdir -p src/pages
mkdir -p src/utils
mkdir -p src/fixtures
mkdir -p src/components
mkdir -p src/reporters
mkdir -p test-data
mkdir -p config
mkdir -p scripts
mkdir -p docker
mkdir -p visual-baseline
mkdir -p reports
mkdir -p screenshots

# 2. package.json の更新
echo "📦 package.json を更新中..."
cat > package.json << 'EOF'
{
  "name": "playwright-practice",
  "version": "1.0.0",
  "description": "Playwright学習用プロジェクト",
  "scripts": {
    "test": "playwright test",
    "test:ui": "playwright test --ui",
    "test:debug": "playwright test --debug",
    "test:headed": "playwright test --headed",
    "test:chrome": "playwright test --project=chromium",
    "test:firefox": "playwright test --project=firefox",
    "test:webkit": "playwright test --project=webkit",
    "test:basic": "playwright test src/tests/basic",
    "test:e2e": "playwright test src/tests/e2e",
    "test:api": "playwright test src/tests/api",
    "test:visual": "playwright test src/tests/visual",
    "report": "playwright show-report",
    "codegen": "playwright codegen",
    "trace": "playwright show-trace"
  },
  "devDependencies": {
    "@playwright/test": "^1.40.0",
    "@types/node": "^20.0.0",
    "typescript": "^5.0.0"
  },
  "dependencies": {
    "dotenv": "^16.0.0",
    "csv-parse": "^5.0.0"
  }
}
EOF

# 3. TypeScript設定
echo "🔧 TypeScript設定を作成中..."
cat > tsconfig.json << 'EOF'
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020", "DOM"],
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "moduleResolution": "node",
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@tests/*": ["src/tests/*"],
      "@pages/*": ["src/pages/*"],
      "@utils/*": ["src/utils/*"],
      "@fixtures/*": ["src/fixtures/*"]
    }
  },
  "include": ["src/**/*", "tests/**/*"],
  "exclude": ["node_modules", "dist"]
}
EOF

# 4. Playwright設定ファイル
echo "⚙️ Playwright設定を作成中..."
cat > playwright.config.ts << 'EOF'
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './src/tests',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [
    ['html', { outputFolder: 'reports/html' }],
    ['list'],
    ['json', { outputFile: 'reports/results.json' }]
  ],
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    actionTimeout: 10000,
    navigationTimeout: 30000,
  },

  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
    {
      name: 'Mobile Safari',
      use: { ...devices['iPhone 12'] },
    },
  ],

  outputDir: 'test-results/',
});
EOF

# 5. .gitignore
echo "📝 .gitignore を更新中..."
cat > .gitignore << 'EOF'
node_modules/
test-results/
playwright-report/
reports/
screenshots/
visual-baseline/
.env
.DS_Store
*.log
dist/
coverage/
.vscode/
.idea/
EOF

# 6. 最初のタスクファイルをコピー
echo "📋 最初のタスクファイルを作成中..."
# ここで実際のタスクファイルの内容を作成
echo "タスクファイルは手動で作成してください: tasks/step1-basics/task-1-1-navigation.md"

# 7. サンプルテストファイル
echo "🧪 サンプルテストファイルを作成中..."
cat > src/tests/basic/sample.spec.ts << 'EOF'
import { test, expect } from '@playwright/test';

test.describe('サンプルテスト', () => {
  test('Playwrightが正しく動作することを確認', async ({ page }) => {
    // Playwright公式サイトにアクセス
    await page.goto('https://playwright.dev/');
    
    // タイトルを確認
    await expect(page).toHaveTitle(/Playwright/);
    
    // Get Startedリンクを確認
    const getStarted = page.getByRole('link', { name: 'Get started' });
    await expect(getStarted).toBeVisible();
  });
});
EOF

# 8. ユーティリティファイルのサンプル
echo "🛠️ ユーティリティファイルを作成中..."
cat > src/utils/helpers.ts << 'EOF'
import { Page } from '@playwright/test';

/**
 * 指定時間待機する
 */
export async function wait(ms: number): Promise<void> {
  return new Promise(resolve => setTimeout(resolve, ms));
}

/**
 * スクリーンショットを保存
 */
export async function takeScreenshot(
  page: Page, 
  name: string, 
  fullPage: boolean = false
): Promise<void> {
  await page.screenshot({
    path: `screenshots/${name}.png`,
    fullPage: fullPage
  });
}

/**
 * ページの読み込み完了を待つ
 */
export async function waitForPageLoad(page: Page): Promise<void> {
  await page.waitForLoadState('networkidle');
}
EOF

# 9. 環境変数テンプレート
echo "🔐 環境変数テンプレートを作成中..."
cat > .env.example << 'EOF'
# テスト環境のURL
BASE_URL=http://localhost:3000
STAGING_URL=https://staging.example.com
PRODUCTION_URL=https://www.example.com

# テストユーザー情報
TEST_USER_EMAIL=test@example.com
TEST_USER_PASSWORD=Test123!

# API設定
API_KEY=your-api-key-here
API_BASE_URL=https://api.example.com

# レポート設定
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/xxx

# デバッグ設定
HEADLESS=false
SLOW_MO=0
EOF

echo "✅ セットアップが完了しました！"
echo ""
echo "次のステップ:"
echo "1. npm install を実行して依存関係をインストール"
echo "2. npx playwright install でブラウザをインストール"
echo "3. npm run test:ui でUIモードを起動"
echo "4. tasks/step1-basics/ のタスクから学習を開始"
echo ""
echo "Happy Testing! 🎉"
```
