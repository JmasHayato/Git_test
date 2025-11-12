# 1.1　基本設計

## 1.1.1 プロジェクト基本情報

本システムは、ディップ様の申込・契約管理システム（ISHIN）として、Amazon Web Services（以下AWS）上に構築するものとする。

### 基本情報

| 項目名 | 値 | 備考 |
|--------|-----|------|
| 顧客名 | ディップ | 正式顧客名称 |
| 顧客名（英名） | dip | システムを識別する名称 |
| プロジェクト名 | dip | システムを識別する名称 |
| システム名 | 申込・契約管理システム | 正式システム名称 |
| システム名（英名） | ISHIN | リソース名などで利用 |
| 本番環境識別子 | prd | 環境を識別するタグ |
| ステージング環境識別子 | stg | 検証環境を識別するタグ |
| DR環境識別子 | prd-dr, stg-dr | 災害対策環境識別子 |
| 共通環境識別子 | com | アカウント共通リソース識別子 |

参考: [AWS命名規則](https://docs.aws.amazon.com/whitepapers/latest/tagging-best-practices/tagging-best-practices.html)

### リージョン・AZ設定

| 項目名 | 値 | AWS参考ドキュメント | 備考 |
|--------|-----|-------------------|------|
| プライマリリージョン | 東京リージョン (ap-northeast-1) | [リージョンとAZ](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html) | メインの稼働リージョン |
| DRリージョン | 大阪リージョン (ap-northeast-3) | [リージョンとAZ](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html) | 災害対策リージョン |
| 使用AZ（東京） | AZ-A, AZ-C, AZ-D | [マルチAZ設計](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-subnet-basics.html) | Multi-AZ構成 |
| 使用AZ（大阪） | AZ-A, AZ-B, AZ-C | [マルチAZ設計](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-subnet-basics.html) | DR環境のMulti-AZ構成 |

## 1.1.2 利用サービス(AWS、その他)

本システムは、Amazon Web Services （以下AWS）上に構築するものとする。  
本システムのインフラ環境は、AWSの以下のメニュー及びその他サービスを利用する。

|サービス|メニュー項目|サービス|詳細|説明|AWS参考ドキュメント|
|--|---|---|---|---|---|
|AWS|コンピューティング|Fargate|ECS Fargate|APIサーバー、Bastionサーバーのコンテナ実行環境として利用|[AWS Fargate](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html)|
|||Lambda|関数実行|バッチ処理の実行環境として利用|[AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)|
||データベース|Aurora|MySQL互換|RDBMSとして利用|[Aurora MySQL](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.AuroraMySQL.html)|
|||ElastiCache|Redis OSS|キャッシュサーバーとして利用|[ElastiCache Redis](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/WhatIs.Redis.html)|
||ストレージ|S3|オブジェクトストレージ|バックアップデータ、各種ログ、業務データを保管するために利用|[Amazon S3](https://docs.aws.amazon.com/s3/latest/userguide/Welcome.html)|
|||ECR|コンテナレジストリ|FargateのDockerイメージ保管と取得に利用|[Amazon ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html)|
||ネットワーク|VPC||プライベートネットワーク空間作成のためVPCを利用|[Amazon VPC](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)|
|||ALB|Application Load Balancer|APIサーバーへの負荷分散に利用|[Application Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html)|
|||CloudFront|CDN|S3上のコンテンツ配信の高速化、API高速化のため利用|[Amazon CloudFront](https://docs.aws.amazon.com/cloudfront/latest/developerguide/Introduction.html)|
|||Route 53|DNS|ドメイン管理サービス|[Route 53](https://docs.aws.amazon.com/route53/latest/developerguide/Welcome.html)|
|||Shield|Standard|ネットワークおよびトランスポートレイヤーの DDoS 攻撃を防御|[AWS Shield](https://docs.aws.amazon.com/waf/latest/developerguide/shield-chapter.html)|
||セキュリティ|Security Group/ACL||IP/Portによる通信制限のため利用|[セキュリティグループ](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html)|
|||WAF||SQLインジェクション、XSS、Bot攻撃対策のために利用|[AWS WAF](https://docs.aws.amazon.com/waf/latest/developerguide/waf-chapter.html)|
|||Secrets Manager||データベースのユーザ情報等を管理するために利用|[AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html)|
|||KMS||データ領域暗号化のために利用|[AWS KMS](https://docs.aws.amazon.com/kms/latest/developerguide/overview.html)|
|||GuardDuty||悪意のあるアクティビティがないかを確認し可視化と修復のために利用|[Amazon GuardDuty](https://docs.aws.amazon.com/guardduty/latest/ug/what-is-guardduty.html)|
|||Inspector||ECRイメージに対して脆弱性スキャンを実施するために利用|[Amazon Inspector](https://docs.aws.amazon.com/inspector/latest/user/what-is-inspector.html)|
||認証・ユーザー管理|Cognito||ユーザー認証・認可サービス|[Amazon Cognito](https://docs.aws.amazon.com/cognito/latest/developerguide/what-is-amazon-cognito.html)|
|||IAM||AWSのサービス・リソース利用にあたってのユーザ/サーバ権限を制御|[AWS IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)|
||監視・ログ|CloudWatch||AWSリソースとアプリケーションの監視|[Amazon CloudWatch](https://docs.aws.amazon.com/cloudwatch/latest/monitoring/WhatIsCloudWatch.html)|
|||CloudTrail||AWSのサービス・リソースに対する操作履歴を記録するために利用|[AWS CloudTrail](https://docs.aws.amazon.com/cloudtrail/latest/userguide/cloudtrail-user-guide.html)|
|||Config||AWSのサービス・リソースに対する設定変更履歴を記録するために利用|[AWS Config](https://docs.aws.amazon.com/config/latest/developerguide/WhatIsConfig.html)|
||通知|SNS||CloudWatchにて検知した障害をメール送信機能にて通知するために利用|[Amazon SNS](https://docs.aws.amazon.com/sns/latest/dg/welcome.html)|
|||Chatbot||CloudWatchにて検知した障害をSlackにて通知するために利用|[AWS Chatbot](https://docs.aws.amazon.com/chatbot/latest/adminguide/what-is.html)|
|||SES||メール通知のために利用|[Amazon SES](https://docs.aws.amazon.com/ses/latest/dg/Welcome.html)|
||バックアップ|Backup||S3、Auroraのバックアップおよび災対環境へのコピーのために利用|[AWS Backup](https://docs.aws.amazon.com/aws-backup/latest/devguide/whatisbackup.html)|
||自動化|EventBridge||定期的にバッチ処理を呼び出すために利用|[Amazon EventBridge](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-what-is.html)|
|||Step Functions||Lambda Functionの実行順序/ループ制御のために利用|[AWS Step Functions](https://docs.aws.amazon.com/step-functions/latest/dg/welcome.html)|

## 1.1.3 利用者・接続要件

本システムの利用者及び接続要件は以下とする。

|**接続元**|**用途**|**利用可能時間**|**ネットワーク**|**接続先**|**接続方法**|
|---|---|---|---|---|---|
|利用者:ISHIN社内|本システム利用|24時間365日|ISHIN社内ネットワーク（Zscaler経由）|CloudFront/ALB|Webブラウザ|
|||||AWS|AWS管理コンソール（Azure AD連携）|
|管理者:ISHIN社内|本システム利用|24時間365日|ISHIN社内ネットワーク（Zscaler経由）|CloudFront/ALB|Webブラウザ|
|||||AWS|AWS管理コンソール（Azure AD連携）|
|保守・運用担当者|運用|24時間365日|インターネット|AWS|AWS管理コンソール|
|構築・開発担当者|構築・開発|24時間365日|インターネット|AWS|AWS管理コンソール|

## 1.1.4 セキュリティ・コンプライアンス要件

| 項目名 | 内容 | AWS参考ドキュメント |
|--------|------|-------------------|
| コンプライアンス要件 | ISHIN社標準準拠 | [AWS コンプライアンス](https://aws.amazon.com/compliance/) |
| 定期セキュリティチェック | ISHIN社側で実施 | [AWS Security Hub](https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub.html) |
| 脆弱性診断 | お客様主体で実施 | [AWS セキュリティベストプラクティス](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html) |
| アクセス制限 | ISHIN社内ネットワークのみ | [VPC セキュリティ](https://docs.aws.amazon.com/vpc/latest/userguide/security.html) |
| 認証方式 | Azure AD連携 | [AWS SSO](https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html) |

## 1.1.5 可用性・DR要件

| 項目名 | 内容 | AWS参考ドキュメント |
|--------|------|-------------------|
| 目標稼働率 | 99.9%以上 | [AWS SLA](https://aws.amazon.com/legal/service-level-agreements/) |
| 災害対策構成 | プライマリ：東京、セカンダリ：大阪 | [Multi-AZ Deployments](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/use-fault-isolation-to-protect-your-workload.html) |
| 復旧目標RPO | 1営業日前（日次バックアップ） | [Backup and Recovery](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/plan-for-disaster-recovery-dr.html) |
| 復旧目標RTO | 0.5営業日以内 | [Disaster Recovery](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/plan-for-disaster-recovery-dr.html) |

備考: 計画停止を除く24時間365日稼働を目標とする。Cross-Region Backup + 手動構築によるDR対策を実施。
