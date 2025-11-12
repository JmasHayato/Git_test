# 1.8 DR対策  

## 1.8.1 復旧開始の条件  

災害（AWS障害含む）を原因として、エンドユーザ起点のワークフローが停止するという事象が発生した場合に、復旧を開始するものとする。  

## 1.8.2 復旧目標の定義  

可用性を回復するための目標を要件に応じて定義する。  

### (1)RTO（目標復旧時間）  

RTOを以下の通り定義する。  
- RTO: 0.5営業日以内

参考: [Disaster Recovery](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/plan-for-disaster-recovery-dr.html)

### (2)RPO（目標復旧時点）  

RPOを以下の通り定義する。  
- RPO: 1営業日前（日次バックアップ）

参考: [Backup and Recovery](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/plan-for-disaster-recovery-dr.html)

## 1.8.3 復旧戦略の定義  

復旧目標を達成するための戦略を下記表に従い定義する。  

|復旧戦略|RPO|RTO|内容|AWS参考ドキュメント|
|---|---|---|---|---|
|バックアップと復元|数時間以内|24時間以内|システムのデータをDRリージョンにバックアップし、障害発生時にデータを復元する。|[バックアップと復元](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/plan-for-disaster-recovery-dr.html)|
|パイロットライト|数分以内|数時間以内|システムの重要な要素を常に実行する環境をDRリージョンに構築し、障害発生時にプロビジョニングを行う。|[パイロットライト](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/plan-for-disaster-recovery-dr.html)|
|ウォームスタンバイ|数秒以内|数分以内|システムの縮小バージョンを常に実行する環境をDRリージョンに構築し、障害発生時に切り替えを行う。|[ウォームスタンバイ](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/plan-for-disaster-recovery-dr.html)|
|マルチサイト|0秒～数秒|数秒以内|システムを複数のリージョンに構築し、障害発生時に切り替えを行う。|[マルチサイト](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/plan-for-disaster-recovery-dr.html)|

### (1)アベイラビリティーゾーン障害  

復旧時間が目標を達成していることを確認するために、リソース毎に復旧戦略を定義する。

- Fargate (APIサーバー)
  - 復旧戦略: マルチサイト
  - 復旧時間: 数秒
  - 詳細: Multi-AZ配置により、AZ障害時も継続稼働

参考: [ECS高可用性](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-auto-scaling.html)

- Lambda (Batchサーバー)
  - 復旧戦略: マルチサイト
  - 復旧時間: 数秒
  - 詳細: AWSが自動的に複数AZで実行

参考: [Lambda高可用性](https://docs.aws.amazon.com/lambda/latest/dg/lambda-intro-execution-role.html)

- Aurora (DBサーバー)
  - 復旧戦略: ウォームスタンバイ
  - 復旧時間: 60-120秒
  - 詳細: Multi-AZ配置により自動フェイルオーバー

参考: [Aurora高可用性](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Concepts.AuroraHighAvailability.html)

- ElastiCache (Cacheサーバー)
  - 復旧戦略: ウォームスタンバイ
  - 復旧時間: 数秒
  - 詳細: Multi-AZ配置により自動フェイルオーバー

参考: [ElastiCache高可用性](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/AutoFailover.html)

### (2)リージョン障害  

復旧戦略: バックアップと復元
復旧目標: RTO 0.5営業日以内、RPO 1営業日前

| リソース | バックアップ方式 | 復旧方法 | AWS参考ドキュメント |
|---------|----------------|---------|-------------------|
| Aurora DB | Cross-Region Snapshot | DRリージョンでスナップショットから復元 | [Auroraスナップショット](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Managing.Backups.html) |
| S3データ | Cross-Region Replication | DRリージョンのS3バケットから復元 | [S3 CRR](https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html) |
| ECRイメージ | Cross-Region Replication | DRリージョンのECRから取得 | [ECRレプリケーション](https://docs.aws.amazon.com/AmazonECR/latest/userguide/replication.html) |
| インフラ構成 | Infrastructure as Code (IaC) | CloudFormation/Terraformで再構築 | [IaC](https://docs.aws.amazon.com/whitepapers/latest/introduction-devops-aws/infrastructure-as-code.html) |

## 1.8.4 復旧戦略の検証  

復旧目標が達成されることを検証する方法を以下の通り策定する。

### 検証計画

| 項目 | 内容 | 頻度 | AWS参考ドキュメント |
|------|------|------|-------------------|
| AZ障害シミュレーション | 特定AZのリソースを停止し、他AZへの自動切り替えを確認 | 年1回 | [AZ障害テスト](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/test-reliability.html) |
| リージョン障害シミュレーション | DRリージョンでのバックアップからの復旧手順を実施 | 年1回 | [DRテスト](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/test-reliability.html) |
| データ整合性確認 | バックアップデータの復元テストとデータ整合性確認 | 四半期毎 | [バックアップ検証](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/test-reliability.html) |

## 1.8.5 復旧設定の管理  

復旧した設定が要件に合致していることを管理する方法を以下の通り策定する。

### 管理方針

| 項目 | 内容 | AWS参考ドキュメント |
|------|------|-------------------|
| 構成管理 | Infrastructure as Code (CloudFormation/Terraform)による構成管理 | [IaC](https://docs.aws.amazon.com/whitepapers/latest/introduction-devops-aws/infrastructure-as-code.html) |
| 変更管理 | AWS Configによる構成変更の追跡と監査 | [AWS Config](https://docs.aws.amazon.com/config/latest/developerguide/WhatIsConfig.html) |
| ドキュメント管理 | 復旧手順書の定期的な更新とレビュー | - |
| 復旧訓練 | 年次での復旧訓練実施と手順書の更新 | [DR訓練](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/test-reliability.html) |

## 1.8.6 復旧の自動化

復旧経路を自動化する方法を以下の通りリソース毎に策定する。  

### (1)アベイラビリティーゾーン障害  

- Fargate  
  - ECS Service Auto Scalingを使用して、自動的にスケールアウトが行われるようにする。  
  - ヘルスチェック失敗時の自動タスク再起動を有効化する。

参考: [ECS Auto Scaling](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-auto-scaling.html)

- Lambda  
  - AWSが自動的に複数AZで実行し、障害時は別AZで再実行される。

参考: [Lambda高可用性](https://docs.aws.amazon.com/lambda/latest/dg/lambda-intro-execution-role.html)

- Aurora  
  - スタンバイレプリカを使用して、自動的にフェイルオーバーが行われるようにする。  
  - Multi-AZ構成により60-120秒で自動復旧する。

参考: [Aurora自動フェイルオーバー](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Concepts.AuroraHighAvailability.html)

- ElastiCache  
  - レプリカノードへの自動フェイルオーバーを有効化する。

参考: [ElastiCache自動フェイルオーバー](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/AutoFailover.html)

### (2)リージョン障害  

リージョン障害時の復旧は手動で実施する。

#### 復旧手順

1. **影響範囲の確認**
   - AWS Health Dashboardで障害状況を確認
   - 影響を受けているサービスの特定

参考: [AWS Health](https://docs.aws.amazon.com/health/latest/ug/what-is-aws-health.html)

2. **DRリージョンでの環境構築**
   - Infrastructure as Codeを使用してDRリージョンにインフラを構築
   - VPC、サブネット、セキュリティグループの作成

参考: [CloudFormation](https://docs.aws.amazon.com/cloudformation/latest/UserGuide/Welcome.html)

3. **データ復旧**
   - DRリージョンのAuroraスナップショットから復元
   - S3 Cross-Region Replicationによるデータ確認
   - ECRイメージの利用可能性確認

参考: [データ復旧](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/plan-for-disaster-recovery-dr.html)

4. **アプリケーションデプロイ**
   - ECSタスク定義の作成とサービス起動
   - Lambda関数のデプロイ

5. **DNS切り替え**
   - Route 53でDNSレコードをDRリージョンのエンドポイントに変更
   - TTL経過後に新環境への通信開始

参考: [Route 53](https://docs.aws.amazon.com/route53/latest/developerguide/Welcome.html)

6. **動作確認**
   - 各サービスの動作確認
   - データ整合性の確認
   - エンドツーエンドテストの実施

7. **監視設定**
   - CloudWatch Alarmsの設定
   - ログ収集の確認

参考: [CloudWatch](https://docs.aws.amazon.com/cloudwatch/latest/monitoring/WhatIsCloudWatch.html)

#### 復旧時間の目安

| フェーズ | 想定時間 | 備考 |
|---------|---------|------|
| 影響範囲確認 | 30分 | AWS Health Dashboard確認 |
| インフラ構築 | 1時間 | IaCによる自動構築 |
| データ復旧 | 1時間 | スナップショットサイズに依存 |
| アプリケーションデプロイ | 30分 | コンテナ起動時間含む |
| DNS切り替え | 30分 | TTL経過待ち含む |
| 動作確認 | 30分 | 基本動作確認 |
| **合計** | **約4.5時間** | **RTO 0.5営業日以内を達成** |

## 1.8.7 バックアップ構成

### Cross-Region Backup設定

| リソース | プライマリリージョン | DRリージョン | バックアップ方式 | AWS参考ドキュメント |
|---------|-------------------|-------------|----------------|-------------------|
| Aurora DB | 東京 (ap-northeast-1) | 大阪 (ap-northeast-3) | Cross-Region Snapshot | [Auroraスナップショット](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Managing.Backups.html) |
| S3データ | 東京 (ap-northeast-1) | 大阪 (ap-northeast-3) | Cross-Region Replication | [S3 CRR](https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html) |
| ECRイメージ | 東京 (ap-northeast-1) | 大阪 (ap-northeast-3) | Cross-Region Replication | [ECRレプリケーション](https://docs.aws.amazon.com/AmazonECR/latest/userguide/replication.html) |
