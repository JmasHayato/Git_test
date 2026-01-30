# Amazon Q と GitHub 連携実装例

## GitHub Actions ワークフロー例

### 1. Amazon Q による自動コードレビュー

```yaml
# .github/workflows/amazon-q-code-review.yml
name: Amazon Q Code Review

on:
  pull_request:
    branches: [ main, develop ]
    paths:
      - '**/*.md'
      - '**/*.yml'
      - '**/*.yaml'
      - '**/*.json'

jobs:
  amazon-q-review:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
      id-token: write
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: ap-northeast-1
      
      - name: Amazon Q Security Scan
        run: |
          # Amazon Q CLI を使用したセキュリティスキャン
          aws codewhisperer scan-code \
            --source-path . \
            --output-format json \
            --language markdown
      
      - name: Comment PR with results
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const results = JSON.parse(fs.readFileSync('scan-results.json', 'utf8'));
            
            let comment = '## Amazon Q レビュー結果\n\n';
            if (results.findings.length === 0) {
              comment += '✅ セキュリティ上の問題は検出されませんでした。';
            } else {
              comment += '⚠️ 以下の問題が検出されました:\n\n';
              results.findings.forEach(finding => {
                comment += `- **${finding.severity}**: ${finding.description}\n`;
                comment += `  ファイル: ${finding.file}:${finding.line}\n\n`;
              });
            }
            
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: comment
            });
```

### 2. 設計書品質チェック

```yaml
# .github/workflows/document-quality-check.yml
name: Document Quality Check

on:
  push:
    paths:
      - 'GenerateAIWork/**/*.md'
      - 'docs/**/*.md'

jobs:
  quality-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: |
          npm install -g markdownlint-cli
          npm install -g textlint
      
      - name: Markdown lint
        run: |
          markdownlint "**/*.md" --config .markdownlint.json
      
      - name: Japanese text check
        run: |
          textlint "**/*.md" --config .textlintrc.json
      
      - name: AWS service validation
        run: |
          # AWS サービス名の妥当性チェック
          python scripts/validate-aws-services.py
```

## VSCode 設定例

### 1. settings.json

```json
{
  "amazonQ.developer.enabled": true,
  "amazonQ.developer.autoSuggest": true,
  "amazonQ.developer.codeWhisperer": {
    "enabled": true,
    "includeCodeWithReference": true,
    "shareCodeWhispererContentWithAWS": false
  },
  "github.copilot.enable": {
    "*": true,
    "yaml": true,
    "markdown": true,
    "json": true
  },
  "github.copilot.advanced": {
    "length": 500,
    "temperature": 0.1
  },
  "editor.inlineSuggest.enabled": true,
  "editor.suggest.preview": true,
  "files.associations": {
    "*.md": "markdown"
  },
  "markdown.preview.enhanced": true
}
```

### 2. extensions.json

```json
{
  "recommendations": [
    "amazon-web-services.amazon-q-vscode",
    "github.copilot",
    "github.copilot-chat",
    "ms-vscode.vscode-json",
    "redhat.vscode-yaml",
    "davidanson.vscode-markdownlint"
  ]
}
```

## プロジェクト固有設定

### 1. Amazon Q プロジェクトコンテキスト

```yaml
# .amazon-q/project-context.yml
project:
  name: "JMA Systems AWS Infrastructure Design"
  language: "ja"
  framework: "AWS Well-Architected"
  
context:
  domain: "AWS Infrastructure Documentation"
  output_format: "Markdown"
  standards:
    - "AWS Well-Architected Framework"
    - "Multi-AZ Configuration"
    - "Security Groups"
    - "CloudFormation/CDK"
  
prompts:
  system: |
    あなたはJMAシステムズのAWSインフラ設計書作成を支援するAIです。
    以下の要件に従って回答してください：
    - 日本語で回答する
    - AWS Well-Architected Frameworkに準拠する
    - セキュリティベストプラクティスを適用する
    - Multi-AZ構成を考慮する
  
  templates:
    basic_design: "基本設計書のテンプレートに従って、AWSサービス構成を記述してください"
    security: "セキュリティ設計において、IAM、セキュリティグループ、NACLの設定を含めてください"
    network: "ネットワーク設計において、VPC、サブネット、ルーティングを詳細に記述してください"
```

### 2. 品質チェック設定

```json
{
  "markdownlint": {
    "MD013": false,
    "MD033": false,
    "MD041": false
  },
  "textlint": {
    "rules": {
      "preset-ja-technical-writing": true,
      "preset-ja-spacing": true,
      "aws-service-name": true
    }
  }
}
```

## 使用例とコマンド

### Amazon Q Developer CLI

```bash
# セキュリティスキャン
aws codewhisperer scan-code --source-path ./docs --language markdown

# コード生成
aws codewhisperer generate-code --prompt "AWS VPC設計書のテンプレートを作成" --language markdown

# 説明生成
aws codewhisperer explain-code --file-path ./docs/network-design.md
```

### GitHub CLI との連携

```bash
# PR作成時にAmazon Qレビューを自動実行
gh pr create --title "設計書更新" --body "Amazon Q による自動レビューを実行します"

# レビュー結果の確認
gh pr view --comments
```

これらの設定例を参考に、プロジェクトの要件に応じてカスタマイズしてください。
