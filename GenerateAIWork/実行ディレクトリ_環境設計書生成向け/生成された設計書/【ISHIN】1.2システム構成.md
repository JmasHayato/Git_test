# 1.2　システム構成

## 1.2.1 システム構成図

システム構成図については[[別紙]システム構成図](./[別紙]システム構成図.md)を参照。

## 1.2.2 サーバ構成

要件に従い、以下のように構成する。

### APIサーバー構成

| 項目名 | 値 | AWS参考ドキュメント | 備考 |
|--------|-----|-------------------|------|
| サービス | Fargate | [AWS Fargate](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html) | サーバーレスコンテナサービス |
| CPU | 2 | [Fargateタスクサイズ](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/capacity-tasksize.html) | vCPU数 |
| メモリ | 4GiB | [Fargateタスクサイズ](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/capacity-tasksize.html) | メモリ容量 |
| ストレージ | 20GiB | [Fargateエフェメラルストレージ](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/fargate-task-storage.html) | エフェメラルストレージ |
| OS | Linux | [Fargateプラットフォーム](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/platform-versions.html) | プラットフォームOS |
| 常時起動台数 | 3台 | [ECS Auto Scaling](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-auto-scaling.html) | 基本稼働台数 |
| AutoScaling最小台数 | 3 | [ECS Auto Scaling](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-auto-scaling.html) | スケーリング最小値 |
| AutoScaling最大台数 | 5 | [ECS Auto Scaling](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-auto-scaling.html) | スケーリング最大値 |
| スケーリングターゲットメトリック | CPU使用率 | [ECS Auto Scaling](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-auto-scaling.html) | スケーリング判定基準 |
| スケーリング閾値 | 70% | [ECS Auto Scaling](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-auto-scaling.html) | CPU使用率閾値 |
| スケールアウト冷却期間 | 300秒 | [ECS Auto Scaling](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-auto-scaling.html) | スケールアウト後の待機時間 |
| スケールイン冷却期間 | 300秒 | [ECS Auto Scaling](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-auto-scaling.html) | スケールイン後の待機時間 |
| プラットフォームバージョン | 1.4.0 | [Fargateプラットフォームバージョン](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/platform-versions.html) | Fargateバージョン |

### Bastionサーバー構成

| 項目名 | 値 | AWS参考ドキュメント | 備考 |
|--------|-----|-------------------|------|
| サービス | Fargate | [AWS Fargate](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html) | サーバーレスコンテナサービス |
| CPU | 1 | [Fargateタスクサイズ](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/capacity-tasksize.html) | vCPU数 |
| メモリ | 2GiB | [Fargateタスクサイズ](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/capacity-tasksize.html) | メモリ容量 |
| ストレージ | 20GiB | [Fargateエフェメラルストレージ](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/fargate-task-storage.html) | エフェメラルストレージ |
| OS | Linux | [Fargateプラットフォーム](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/platform-versions.html) | プラットフォームOS |
| 起動台数 | 1台（平時は停止） | [ECSタスク管理](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/scheduling_tasks.html) | 踏み台サーバー |

### Batchサーバー構成

#### Lambda（コンテナ）

| 項目名 | 値 | AWS参考ドキュメント | 備考 |
|--------|-----|-------------------|------|
| サービス | Lambda | [AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html) | サーバーレス関数サービス |
| メモリ | アプリケーション毎に設定 | [Lambda設定](https://docs.aws.amazon.com/lambda/latest/dg/configuration-memory.html) | 性能試験でfix |
| エフェメラルストレージ | アプリケーション毎に設定 | [Lambdaストレージ](https://docs.aws.amazon.com/lambda/latest/dg/configuration-ephemeral-storage.html) | 性能試験でfix |
| アーキテクチャ | x86_64 | [Lambdaアーキテクチャ](https://docs.aws.amazon.com/lambda/latest/dg/foundation-arch.html) | プロセッサアーキテクチャ |
| パッケージタイプ | image | [Lambdaパッケージング](https://docs.aws.amazon.com/lambda/latest/dg/lambda-images.html) | コンテナイメージ |
| VPC配備 | 有効（マルチAZ） | [Lambda VPC](https://docs.aws.amazon.com/lambda/latest/dg/configuration-vpc.html) | VPC内配置設定 |

#### Lambda（Zip）

| 項目名 | 値 | AWS参考ドキュメント | 備考 |
|--------|-----|-------------------|------|
| サービス | Lambda | [AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html) | サーバーレス関数サービス |
| メモリ | アプリケーション毎に設定 | [Lambda設定](https://docs.aws.amazon.com/lambda/latest/dg/configuration-memory.html) | 性能試験でfix |
| エフェメラルストレージ | アプリケーション毎に設定 | [Lambdaストレージ](https://docs.aws.amazon.com/lambda/latest/dg/configuration-ephemeral-storage.html) | 性能試験でfix |
| アーキテクチャ | x86_64 | [Lambdaアーキテクチャ](https://docs.aws.amazon.com/lambda/latest/dg/foundation-arch.html) | プロセッサアーキテクチャ |
| パッケージタイプ | zip | [Lambdaパッケージング](https://docs.aws.amazon.com/lambda/latest/dg/lambda-deployment-packages.html) | ZIPファイル |
| ランタイム | Amazon Linux 2023 | [Lambdaランタイム](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html) | 実行環境OS |
| VPC配備 | 有効（マルチAZ） | [Lambda VPC](https://docs.aws.amazon.com/lambda/latest/dg/configuration-vpc.html) | VPC内配置設定 |

## 1.2.3 ソフトウェア構成

要件に従い、以下のように構成する。

### データベースサーバー設定

| 項目名 | 値 | AWS参考ドキュメント | 備考 |
|--------|-----|-------------------|------|
| サービス | Aurora | [Aurora概要](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/CHAP_AuroraOverview.html) | マネージドデータベースサービス |
| インスタンスタイプ | db.r6g.xlarge | [Auroraインスタンスクラス](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Concepts.DBInstanceClass.html) | CPU: 4コア、メモリ: 32GiB |
| CPU | 4 | [Auroraインスタンスクラス](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Concepts.DBInstanceClass.html) | vCPU数 |
| メモリ | 32GiB | [Auroraインスタンスクラス](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Concepts.DBInstanceClass.html) | メモリ容量 |
| 常時起動台数 | 2台(ライター1、リーダー1) | [Aurora Auto Scaling](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Integrating.AutoScaling.html) | 高可用性構成 |
| データベースエンジン | Aurora MySQL | [Aurora MySQL](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.AuroraMySQL.html) | MySQL互換エンジン |
| エンジンバージョン | 3.04.3 (MySQL 8.0.28互換) | [Auroraバージョン](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Updates.html) | LTSバージョン |
| マルチAZ | 有効 | [マルチAZ設計](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Concepts.AuroraHighAvailability.html) | 高可用性設定 |
| オートスケーリング | 無効 | [Aurora Auto Scaling](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Integrating.AutoScaling.html) | リードレプリカの自動スケーリング |
| スケーリング最小リーダー数 | 1 | [Aurora Auto Scaling](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Integrating.AutoScaling.html) | リードレプリカ最小台数 |
| スケーリング最大リーダー数 | 3 | [Aurora Auto Scaling](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Integrating.AutoScaling.html) | リードレプリカ最大台数 |
| スケーリングターゲットメトリック | CPU使用率 | [Aurora Auto Scaling](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Integrating.AutoScaling.html) | スケーリング判定基準 |
| スケーリング閾値 | 70% | [Aurora Auto Scaling](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Integrating.AutoScaling.html) | CPU使用率閾値 |
| スケールアウト冷却期間 | 300秒 | [Aurora Auto Scaling](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Integrating.AutoScaling.html) | スケールアウト後の待機時間 |
| スケールイン冷却期間 | 300秒 | [Aurora Auto Scaling](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Integrating.AutoScaling.html) | スケールイン後の待機時間 |
| ライター拡張方式 | 垂直スケーリング（インスタンスサイズ変更） | [Aurora拡張](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Managing.Performance.html) | r6g.large→xlarge（短時間ダウンタイム発生） |
| リーダー拡張方式 | 水平スケーリング（リードレプリカ追加） | [Aurora Auto Scaling](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Integrating.AutoScaling.html) | 無停止拡張（最大15台まで） |
| ストレージ拡張方式 | 自動ストレージ拡張 | [Aurora Storage](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Overview.StorageReliability.html) | 10GiB〜128TiB自動拡張 |

### Cacheサーバー設定

| 項目名 | 値 | AWS参考ドキュメント | 備考 |
|--------|-----|-------------------|------|
| サービス | ElastiCache | [ElastiCache](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/WhatIs.html) | マネージドキャッシュサービス |
| インスタンスタイプ | cache.m7g.large | [ElastiCacheノードタイプ](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/CacheNodes.SupportedTypes.html) | CPU: 2コア、メモリ: 6.38GiB |
| CPU | 2 | [ElastiCacheノードタイプ](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/CacheNodes.SupportedTypes.html) | vCPU数 |
| メモリ | 6.38GiB | [ElastiCacheノードタイプ](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/CacheNodes.SupportedTypes.html) | メモリ容量 |
| 常時起動台数 | 2台（マスター1、レプリカ1） | [ElastiCache Redis複製](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/Replication.html) | 高可用性構成 |
| エンジン | Redis OSS | [ElastiCache Redis](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/WhatIs.Redis.html) | インメモリデータストア |
| エンジンバージョン | 7.1 | [Redisバージョン](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/supported-engine-versions.html) | Redisバージョン |
| マルチAZ | 有効 | [ElastiCache Multi-AZ](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/AutoFailover.html) | 自動フェイルオーバー |

### ストレージ設定

| 項目名 | 値 | AWS参考ドキュメント | 備考 |
|--------|-----|-------------------|------|
| サービス（コンテナレジストリ） | ECR | [Amazon ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html) | FargateのDockerイメージ保管と取得に利用 |
| サービス（オブジェクトストレージ） | S3 | [Amazon S3](https://docs.aws.amazon.com/s3/latest/userguide/Welcome.html) | バックアップデータ、各種ログ、業務データを保管するために利用 |

### バックアップ設定

| 項目名 | 値 | AWS参考ドキュメント | 備考 |
|--------|-----|-------------------|------|
| サービス（バックアップ） | Backup | [AWS Backup](https://docs.aws.amazon.com/aws-backup/latest/devguide/whatisbackup.html) | S3、Auroraのバックアップおよび災対環境へのコピーのために利用する |

## 1.2.4 業務系基盤構成

### (1)ドメイン

本システムでは以下のドメインを取得（設定）する。

| 項目名 | 値 | AWS参考ドキュメント | 備考 |
|--------|-----|-------------------|------|
| ドメイン名 | dip-new-gate.com | [Route 53](https://docs.aws.amazon.com/route53/latest/developerguide/Welcome.html) | 外部公開用ドメイン |
| ドメイン管理 | Route53 | [Route 53ドメイン登録](https://docs.aws.amazon.com/route53/latest/developerguide/domain-register.html) | DNS管理サービス |
| ゾーンタイプ | パブリック | [Route 53ホストゾーン](https://docs.aws.amazon.com/route53/latest/developerguide/hosted-zones-working-with.html) | インターネット向け |

### (2)DNS

本システムでは以下のDNS構成とする。

|No.|DNSサーバ|ドメイン|FQDN|向き先|備考|
|:-:|---|---|---|---|---|
|1|Route 53|dip-new-gate.com|www.dip-new-gate.com|Production CloudFront|-|
|2|Route 53|dip-new-gate.com|stg.dip-new-gate.com|Staging CloudFront|-|
|3|Route 53|dip-new-gate.com|api.dip-new-gate.com|Production CloudFront (API)|-|

### (3)ロードバランサ

本システムではApplication Load Balancerを採用し、以下の実装を行う。

- ロードバランシングの対象インスタンスとして、それぞれ異なるアベイラビリティ―ゾーン(AZ)上のAPIサーバを配置する。(ターゲットグループ)
- クロスゾーン負荷分散(Cross-Zone Load Balancing)を有効化し、アベイラビリティーゾーンに関わらず全ての対象インスタンスに対し均等にロードバランシングを行う。
- ロードバランサにSSL証明書を設定し、アクセラレートを行う。
- Connection Drainingを有効にする事により、メンテナンス等でインスタンスを切り離す場合において、設定時間の間は、リクエスト中の通信を切断しないように設定する。
- ELBのヘルスチェック機能により、バランシング対象インスタンスの死活監視を行い、指定された閾値によりバランシング対象からの除外と復旧を行う。
- 専用のセキュリティグループを作成し、アクセス制御を行う。
- ALBの機能により、URLリダイレクトを行う。

### (4)CDN・配信設定

| 項目名 | 値 | AWS参考ドキュメント | 備考 |
|--------|-----|-------------------|------|
| サービス（CDN） | CloudFront | [Amazon CloudFront](https://docs.aws.amazon.com/cloudfront/latest/developerguide/Introduction.html) | S3上のコンテンツ配信の高速化のため利用する |
| メンテナンスページOrigin | S3 Static Website | [CloudFront Origin](https://docs.aws.amazon.com/cloudfront/latest/developerguide/DownloadDistS3AndCustomOrigins.html) | メンテナンスページ用S3バケット |
| メンテナンスページキャッシュ動作 | 無効（TTL=0） | [CloudFrontキャッシュ動作](https://docs.aws.amazon.com/cloudfront/latest/developerguide/controlling-the-cache-key.html) | メンテナンス時即座反映のためキャッシュ無効化 |
| カスタムエラーページ | 503エラー時メンテナンスページ表示 | [CloudFrontカスタムエラーページ](https://docs.aws.amazon.com/cloudfront/latest/developerguide/GeneratingCustomErrorResponses.html) | ALB503エラー時の代替表示 |
| 静的コンテンツ配信 | S3静的WEBサイトホスティング + CloudFront | [S3 Static Website](https://docs.aws.amazon.com/s3/latest/userguide/WebsiteHosting.html) | フロント処理での静的コンテンツ配信対応 |
| API高速化 | CloudFront + ALB | [CloudFront ALB](https://docs.aws.amazon.com/cloudfront/latest/developerguide/distribution-web-values-specify.html) | API処理でのCloudFront高速化対応 |
| WAF連携 | CloudFront + WAF | [CloudFront WAF](https://docs.aws.amazon.com/waf/latest/developerguide/cloudfront-features.html) | 攻撃対策・セキュリティ検査対応 |

### (5)APIサーバ構成

APIサーバは、本システムのRESTful APIインターフェースを提供する。
3台以上による冗長構成(Active/Active)とし、インターネットからの接続に対してロードバランサー(ALB)からの振り分けにより、負荷分散を行う構成とする。

- AWS Fargateを採用し、コンテナベースで稼働する。
- ALBからの HTTPリクエスト に対して、APIレスポンスを返す。
- DBへのSQLクエリを実行する。
- 認証・認可はCognitoと連携する。

AutoScaling構成

- APIサーバはAutoScaling構成とし、必要に応じてスケールアウト可能な構成とする。
- 初期構成は起動台数を3台とし、最小3台、最大5台の設定とする。
- CPU使用率70%を閾値としてスケーリングを実行する。

### (6)Batchサーバ構成

Batchサーバは、本システムのバッチ処理機能を提供する。
AWS Lambdaを採用し、サーバーレスで稼働する。

- EventBridgeによる定期実行、またはイベント駆動での実行を行う。
- Step Functionsによる複雑なワークフロー制御を実装する。
- コンテナイメージ形式またはZip形式でのデプロイに対応する。

### (7)DBサーバ構成

DBサーバは、本システムのRDBMS機能を提供する。
本番/ステージングDBサーバはAmazon Aurora上でのMulti-AZ 配置による冗長構成(Writer/Reader)とする。

- Amazon AuroraではWriterとReader間との同期によるタイムラグ(レプリケーションラグ)は発生しない。
- RDBMSサーバ機能は、 Aurora MySQLを採用する。
- APIサーバ/BatchサーバからのSQLクエリに対し、RDBMSとしての処理結果を返す。

### (8)Cacheサーバ構成

Cacheサーバは、本システムのキャッシュ機能を提供する。
Amazon ElastiCache for Redisを採用し、Multi-AZ配置による冗長構成(Master/Replica)とする。

- セッション情報、一時データの保存に利用する。
- APIレスポンスのキャッシュとして利用する。

### (9)Webサーバー構成

| 項目名 | 値 | AWS参考ドキュメント | 備考 |
|--------|-----|-------------------|------|
| サービス（CDN） | CloudFront | [Amazon CloudFront](https://docs.aws.amazon.com/cloudfront/latest/developerguide/Introduction.html) | 静的コンテンツ配信、API高速化 |
| サービス（静的ホスティング） | S3 | [S3 Static Website](https://docs.aws.amazon.com/s3/latest/userguide/WebsiteHosting.html) | 静的WEBサイトホスティング |
| 配信内容 | 静的コンテンツ | [CloudFront S3](https://docs.aws.amazon.com/cloudfront/latest/developerguide/GettingStarted.SimpleDistribution.html) | HTML、CSS、JS、画像等 |
| SSL証明書 | ACM | [AWS Certificate Manager](https://docs.aws.amazon.com/acm/latest/userguide/acm-overview.html) | HTTPS通信用SSL証明書 |
| カスタムドメイン | 有効 | [CloudFront Custom Domain](https://docs.aws.amazon.com/cloudfront/latest/developerguide/CNAMEs.html) | 独自ドメイン設定 |

### (10)認証サーバー構成

| 項目名 | 値 | AWS参考ドキュメント | 備考 |
|--------|-----|-------------------|------|
| サービス（認証） | Cognito | [Amazon Cognito](https://docs.aws.amazon.com/cognito/latest/developerguide/what-is-amazon-cognito.html) | ユーザー認証・認可サービス |
| ユーザープール | 有効 | [Cognito User Pools](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-identity-pools.html) | ユーザー管理・認証 |
| IDプール | 有効 | [Cognito Identity Pools](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-identity.html) | 一時的AWS認証情報 |
| 多要素認証（MFA） | 有効 | [Cognito MFA](https://docs.aws.amazon.com/cognito/latest/developerguide/user-pool-settings-mfa.html) | セキュリティ強化 |
| パスワードポリシー | 強固 | [Cognito Password Policy](https://docs.aws.amazon.com/cognito/latest/developerguide/user-pool-settings-policies.html) | パスワード複雑度要件 |

### (11)メール送信設定

| 項目名 | 値 | AWS参考ドキュメント | 備考 |
|--------|-----|-------------------|------|
| サービス（メール送信） | SES | [Amazon Simple Email Service](https://docs.aws.amazon.com/ses/latest/dg/Welcome.html) | メール通知のために利用する |

### (12)バッチ処理・ワークフロー設定

| 項目名 | 値 | AWS参考ドキュメント | 備考 |
|--------|-----|-------------------|------|
| サービス（イベント駆動） | EventBridge | [Amazon EventBridge](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-what-is.html) | 定期的にバッチ処理を呼び出すために利用する |
| サービス（ワークフロー） | Step Functions | [AWS Step Functions](https://docs.aws.amazon.com/step-functions/latest/dg/welcome.html) | Lambda Functionの実行順序/ループ制御のために利用する |

### (13)メンテナンスモード設定

| 項目名 | 値 | AWS参考ドキュメント | 備考 |
|--------|-----|-------------------|------|
| サービス（ロードバランサー） | ALB (Application Load Balancer) | [Application Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html) | APIサーバーへの負荷分散に利用 |
| メンテナンスモード | 無効 | [ALB固定レスポンス](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-listeners.html#fixed-response-actions) | メンテナンス時の固定レスポンス機能 |
| メンテナンス表示サービス | S3 + CloudFront | [CloudFront](https://docs.aws.amazon.com/cloudfront/latest/developerguide/Introduction.html) | メンテナンスページ表示用静的コンテンツ配信 |
| メンテナンスページパス | /maintenance.html | [S3静的ウェブサイト](https://docs.aws.amazon.com/s3/latest/userguide/WebsiteHosting.html) | メンテナンス用HTML格納パス |
| メンテナンス切り替え方法 | ALBリスナールール変更 | [ALBリスナールール](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/listener-update-rules.html) | 動的な切り替え制御 |
| メンテナンス時HTTPステータス | 503 Service Unavailable | [HTTP503ステータス](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-listeners.html#fixed-response-actions) | メンテナンス時レスポンスコード |
| メンテナンス管理用Lambda | 有効 | [AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html) | メンテナンスモード自動切り替え用関数 |

## 1.2.5 外部システム連携構成

本システムにおける外部システム連携は、閉塞網接続を優先し、SaaSはインターネット経由で接続する。

### 外部システム接続方針

| 項目名 | 値 | AWS参考ドキュメント | 備考 |
|--------|-----|-------------------|------|
| 外部システム接続方針 | 閉塞網接続優先、SaaSはインターネット経由 | [VPC Connectivity](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Connectivity.html) | 外部システム接続の基本方針 |
| 閉塞網接続方式 | VPC Peering、Direct Connect、Transit Gateway | [VPC Peering](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html) | セキュリティグループ・ルート設定、帯域・冗長性、コスト管理対応 |
| SaaS接続方式 | PrivateLink、VPN、NAT Gateway経由 | [VPC PrivateLink](https://docs.aws.amazon.com/vpc/latest/privatelink/what-is-privatelink.html) | SaaS側仕様確認、通信経路暗号化、可用性・障害時対応 |
