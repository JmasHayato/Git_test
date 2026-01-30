# Amazon Q と GitHub 連携実装例

## 概要

本文書は、Amazon Q と GitHub の連携における具体的な実装例、設定方法、およびワークフロー例を提供します。

## 1. VS Code + Amazon Q Developer 設定例

### 1.1 基本設定

```json
{
  "amazonQ.telemetry": false,
  "amazonQ.shareCodeWhispererContentWithAWS": false,
  "amazonQ.profile": "default",
  "amazonQ.region": "us-east-1",
  "amazonQ.suppressPrompts": {
    "codeWhispererNewWelcomeMessage": true
  }
}
```

### 1.2 プロジェクト固有設定

```json
{
  "amazonQ.customizations": {
    "aws-infrastructure": {
      "description": "AWS インフラ設計書プロジェクト用設定",
      "patterns": [
        "*.md",
        "*.yaml",
        "*.yml",
        "*.json"
      ],
      "excludePatterns": [
        "node_modules/**",
        ".git/**",
        "*.log"
      ]
    }
  }
}
```

## 2. GitHub Actions ワークフロー例

### 2.1 Amazon Q を活用したコードレビュー

```yaml
name: AI-Assisted Code Review

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  ai-review:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Run Amazon Q Code Analysis
        run: |
          # Amazon Q CLI を使用したコード分析
          # 注意: 実際のCLIコマンドは Amazon Q の公式ドキュメントを参照
          echo "Amazon Q による分析を実行中..."
          
      - name: Generate Review Comments
        run: |
          # 分析結果をGitHub PR コメントとして投稿
          echo "レビューコメントを生成中..."
```

### 2.2 文書品質チェック

```yaml
name: Documentation Quality Check

on:
  push:
    paths:
      - '**/*.md'
      - 'docs/**'

jobs:
  doc-quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Check Documentation Quality
        run: |
          # Markdown ファイルの品質チェック
          find . -name "*.md" -exec echo "Checking: {}" \;
          
      - name: Generate Quality Report
        run: |
          echo "文書品質レポートを生成中..."
```

## 3. プロンプトテンプレート例

### 3.1 AWS インフラ設計用プロンプト

```markdown
# AWS インフラ設計書生成プロンプト

## コンテキスト
- プロジェクト: {プロジェクト名}
- 環境: {環境名（本番/ステージング/開発）}
- 要件: {具体的な要件}

## 生成指示
以下の形式でAWS環境設計書を生成してください：

1. システム概要
2. アーキテクチャ図
3. サービス構成
4. セキュリティ設計
5. 運用設計

## 制約条件
- 日本語で記述
- AWS Well-Architected Framework に準拠
- セキュリティベストプラクティスを適用
- コスト最適化を考慮
```

### 3.2 コードレビュー用プロンプト

```markdown
# コードレビュープロンプト

以下のコードをレビューし、改善点を指摘してください：

## レビュー観点
1. セキュリティ（CWE準拠）
2. パフォーマンス
3. 可読性
4. 保守性
5. AWSベストプラクティス準拠

## 出力形式
- 問題点の説明
- 改善提案
- 修正コード例
- 参考資料（CWE番号など）
```

## 4. セキュリティ設定例

### 4.1 IAM ポリシー例

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "codewhisperer:GenerateRecommendations",
        "codewhisperer:GetRecommendations"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:RequestedRegion": ["us-east-1", "ap-northeast-1"]
        }
      }
    }
  ]
}
```

### 4.2 GitHub リポジトリ設定

```yaml
# .github/settings.yml
repository:
  name: aws-infrastructure-docs
  description: "AWS インフラ環境設計書プロジェクト"
  private: true
  
  # セキュリティ設定
  security:
    enable_automated_security_fixes: true
    enable_vulnerability_alerts: true
    
  # ブランチ保護
  branches:
    - name: main
      protection:
        required_status_checks:
          strict: true
          contexts:
            - "ai-review"
            - "doc-quality"
        enforce_admins: true
        required_pull_request_reviews:
          required_approving_review_count: 2
          dismiss_stale_reviews: true
```

## 5. 監視とメトリクス

### 5.1 利用状況監視

```python
# Amazon Q 利用状況監視スクリプト例
import boto3
import json
from datetime import datetime, timedelta

def monitor_amazonq_usage():
    """Amazon Q の利用状況を監視"""
    
    # CloudWatch メトリクス取得
    cloudwatch = boto3.client('cloudwatch')
    
    end_time = datetime.utcnow()
    start_time = end_time - timedelta(days=7)
    
    # メトリクス取得例
    response = cloudwatch.get_metric_statistics(
        Namespace='AWS/CodeWhisperer',
        MetricName='Invocations',
        Dimensions=[
            {
                'Name': 'ServiceName',
                'Value': 'CodeWhisperer'
            }
        ],
        StartTime=start_time,
        EndTime=end_time,
        Period=3600,
        Statistics=['Sum']
    )
    
    return response

if __name__ == "__main__":
    usage_data = monitor_amazonq_usage()
    print(json.dumps(usage_data, indent=2, default=str))
```

### 5.2 品質メトリクス

```bash
#!/bin/bash
# 文書品質メトリクス収集スクリプト

echo "=== 文書品質メトリクス収集 ==="

# Markdown ファイル数
md_count=$(find . -name "*.md" | wc -l)
echo "Markdown ファイル数: $md_count"

# 総行数
total_lines=$(find . -name "*.md" -exec wc -l {} + | tail -1 | awk '{print $1}')
echo "総行数: $total_lines"

# リンク切れチェック
echo "リンク切れチェック実行中..."
# markdown-link-check などのツールを使用

# 文書更新頻度
echo "最近更新された文書:"
find . -name "*.md" -mtime -7 -exec ls -la {} \;
```

## 6. トラブルシューティング

### 6.1 よくある問題と解決方法

| 問題 | 原因 | 解決方法 |
|------|------|----------|
| Amazon Q が応答しない | 認証エラー | AWS認証情報を確認 |
| 生成されるコードの品質が低い | プロンプトが不明確 | より具体的な指示を提供 |
| GitHub Actions が失敗 | 権限不足 | IAMロールとGitHubシークレットを確認 |
| セキュリティ警告 | 機密情報の送信 | データ分類ポリシーを確認 |

### 6.2 デバッグ手順

1. **ログ確認**: CloudWatch Logs でエラー詳細を確認
2. **権限確認**: IAMポリシーとロールの設定を検証
3. **ネットワーク確認**: VPCエンドポイントとセキュリティグループを確認
4. **設定確認**: VS Code設定とAmazon Q設定を再確認

このドキュメントは、Amazon Q と GitHub の連携を効果的に実装するための実践的なガイドとして活用してください。
