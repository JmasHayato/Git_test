# Amazon Q と GitHub 連携実装ガイド

## 概要

本ガイドは、Amazon Q Developer と GitHub の連携を実装するための具体的な手順とコード例を提供します。

## 1. 環境セットアップ

### 1.1 必要な権限とツール

```bash
# 必要なツールのインストール
npm install -g @aws/amazon-q-developer-cli
pip install boto3 amazon-q-developer-sdk

# VSCode拡張機能
# - Amazon Q Developer
# - GitHub Copilot
# - AWS Toolkit
```

### 1.2 AWS認証設定

```bash
# AWS CLIの設定
aws configure set region ap-northeast-1
aws configure set output json

# Amazon Q Developer用のIAMロール設定
aws iam create-role --role-name AmazonQDeveloperRole \
  --assume-role-policy-document file://trust-policy.json
```

## 2. GitHub Actions ワークフロー実装

### 2.1 基本的なワークフロー

```yaml
# .github/workflows/amazon-q-integration.yml
name: Amazon Q Developer Integration

on:
  pull_request:
    branches: [ main, develop ]
    paths:
      - '**/*.py'
      - '**/*.js'
      - '**/*.yaml'
      - '**/*.json'
      - '**/*.md'

jobs:
  amazon-q-review:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
      security-events: write

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
          # セキュリティスキャンの実行
          amazon-q scan --type security --output sarif --file security-results.sarif
          
      - name: Upload SARIF results
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: security-results.sarif

      - name: Amazon Q Code Review
        run: |
          # コードレビューの実行
          amazon-q review --pr-number ${{ github.event.number }} \
            --repository ${{ github.repository }} \
            --output review-comments.json

      - name: Post Review Comments
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const comments = JSON.parse(fs.readFileSync('review-comments.json', 'utf8'));
            
            for (const comment of comments) {
              await github.rest.pulls.createReviewComment({
                owner: context.repo.owner,
                repo: context.repo.repo,
                pull_number: context.issue.number,
                body: comment.body,
                path: comment.path,
                line: comment.line
              });
            }
```

### 2.2 文書生成ワークフロー

```yaml
# .github/workflows/doc-generation.yml
name: AWS Documentation Generation

on:
  push:
    branches: [ main ]
    paths:
      - 'Parameters/**'
      - 'Inputs/**'
      - 'Formats/**'

jobs:
  generate-docs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
          
      - name: Install dependencies
        run: |
          pip install boto3 amazon-q-developer-sdk pyyaml
          
      - name: Generate AWS Documentation
        run: |
          python scripts/generate-aws-docs.py
          
      - name: Commit generated docs
        run: |
          git config --local user.email "action@github.com"
          git config --local user.name "GitHub Action"
          git add GeneratedOutputs/
          git commit -m "Auto-generate AWS documentation" || exit 0
          git push
```

## 3. Python スクリプト実装例

### 3.1 文書生成スクリプト

```python
# scripts/generate-aws-docs.py
import os
import yaml
import json
from pathlib import Path
from amazon_q_developer import DocumentGenerator, AWSArchitectureAnalyzer

class AWSDocumentationGenerator:
    def __init__(self):
        self.doc_generator = DocumentGenerator()
        self.aws_analyzer = AWSArchitectureAnalyzer()
        
    def load_parameters(self, param_file):
        """パラメータファイルの読み込み"""
        with open(param_file, 'r', encoding='utf-8') as f:
            return yaml.safe_load(f)
    
    def load_template(self, template_file):
        """テンプレートファイルの読み込み"""
        with open(template_file, 'r', encoding='utf-8') as f:
            return f.read()
    
    def generate_document(self, template_path, parameters, output_path):
        """Amazon Q を使用した文書生成"""
        template = self.load_template(template_path)
        
        # Amazon Q に送信するプロンプトの構築
        prompt = f"""
以下のテンプレートとパラメータを使用して、AWS環境設計書を生成してください。

テンプレート:
{template}

パラメータ:
{yaml.dump(parameters, allow_unicode=True)}

要件:
- 日本語で記述
- AWS Well-Architected Framework に準拠
- セキュリティベストプラクティスを適用
- Markdown形式で出力
"""
        
        # Amazon Q による文書生成
        generated_doc = self.doc_generator.generate(
            prompt=prompt,
            context="AWS infrastructure documentation",
            language="ja"
        )
        
        # 生成された文書の保存
        os.makedirs(os.path.dirname(output_path), exist_ok=True)
        with open(output_path, 'w', encoding='utf-8') as f:
            f.write(generated_doc)
        
        return generated_doc
    
    def validate_aws_architecture(self, doc_content):
        """AWS アーキテクチャの検証"""
        validation_results = self.aws_analyzer.validate(doc_content)
        return validation_results

def main():
    generator = AWSDocumentationGenerator()
    
    # パラメータファイルの読み込み
    parameters_dir = Path("Parameters")
    formats_dir = Path("Formats")
    output_dir = Path("GeneratedOutputs")
    
    for param_file in parameters_dir.glob("*.md"):
        parameters = generator.load_parameters(param_file)
        
        # 対応するテンプレートファイルを検索
        for template_file in formats_dir.glob("*.md"):
            template_name = template_file.stem
            output_file = output_dir / f"{param_file.stem}_{template_name}.md"
            
            print(f"Generating: {output_file}")
            
            # 文書生成
            doc_content = generator.generate_document(
                template_file, parameters, output_file
            )
            
            # AWS アーキテクチャ検証
            validation_results = generator.validate_aws_architecture(doc_content)
            
            if validation_results.has_issues:
                print(f"Validation issues found in {output_file}:")
                for issue in validation_results.issues:
                    print(f"  - {issue}")

if __name__ == "__main__":
    main()
```

### 3.2 セキュリティスキャンスクリプト

```python
# scripts/security-scan.py
import os
import json
from amazon_q_developer import SecurityScanner
from pathlib import Path

def scan_repository():
    """リポジトリ全体のセキュリティスキャン"""
    scanner = SecurityScanner()
    
    # スキャン対象ファイルの収集
    scan_files = []
    for ext in ['.py', '.js', '.yaml', '.yml', '.json']:
        scan_files.extend(Path('.').rglob(f'*{ext}'))
    
    results = []
    
    for file_path in scan_files:
        print(f"Scanning: {file_path}")
        
        # ファイル内容の読み込み
        with open(file_path, 'r', encoding='utf-8') as f:
            content = f.read()
        
        # セキュリティスキャンの実行
        scan_result = scanner.scan_content(
            content=content,
            file_type=file_path.suffix,
            file_path=str(file_path)
        )
        
        if scan_result.issues:
            results.append({
                'file': str(file_path),
                'issues': scan_result.issues
            })
    
    # 結果の保存
    with open('security-scan-results.json', 'w', encoding='utf-8') as f:
        json.dump(results, f, ensure_ascii=False, indent=2)
    
    return results

if __name__ == "__main__":
    results = scan_repository()
    
    if results:
        print(f"\nSecurity issues found in {len(results)} files:")
        for result in results:
            print(f"  {result['file']}: {len(result['issues'])} issues")
    else:
        print("No security issues found.")
```

## 4. VSCode 設定

### 4.1 設定ファイル

```json
// .vscode/settings.json
{
  "amazonQ.developer.enabled": true,
  "amazonQ.developer.autoSuggest": true,
  "amazonQ.developer.securityScan": true,
  "amazonQ.developer.codeReview": true,
  "amazonQ.developer.awsIntegration": true,
  "github.copilot.enable": {
    "*": true,
    "yaml": true,
    "markdown": true,
    "python": true
  }
}
```

## 5. 使用方法

### 5.1 日常的な開発フロー

1. **コード作成**: Amazon Q Developer を使用してコード生成
2. **レビュー**: GitHub Copilot による補完とレビュー
3. **プッシュ**: GitHub Actions による自動スキャンとレビュー
4. **文書生成**: パラメータ更新時の自動文書生成

### 5.2 トラブルシューティング

```bash
# Amazon Q Developer の状態確認
amazon-q status

# 認証情報の確認
aws sts get-caller-identity

# ログの確認
tail -f ~/.amazon-q/logs/amazon-q.log
```

このガイドに従って実装することで、Amazon Q と GitHub の効果的な連携を実現できます。
