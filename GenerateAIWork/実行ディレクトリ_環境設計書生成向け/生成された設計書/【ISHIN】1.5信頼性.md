# 1.5 信頼性

## 1.5.1 可用性の方針

本システムにおける可用性は以下の方針とする。

|No.|方針|対象ノード|主な対象AWSサービス|説明|AWS参考ドキュメント|
|:-:|---|---|---|---|---|
|1|Single Instance|Bastionサーバー|Fargate|・冗長化は行わず、1台のノードで稼働する。<br>・ノード利用不可時の自動代替手段を設けず、すべて手動での対応とする。<br>・ノード利用不可時は、該当ノードが担うサービスが停止する。|[ECSタスク管理](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/scheduling_tasks.html)|
|2|Multi-AZ Instance with AutoScaling|APIサーバー|Fargate|・ALBを併用し、冗長化した構成で複数のノードを稼働する。<br>・ノードの異常発生時には、ALBのヘルスチェックが失敗し、異常が発生しているノードへのトラフィックは停止される。(縮退運転)<br>・複数台を希望数としたAutoScalingを構成し、ハードウェア異常時/コンテナ異常時/ALBヘルスチェック失敗時には、特定のイメージからタスクを再作成し、回復を図る。<br>・その間、残されたノードで縮退運転を行う事により、サービスの継続を図る。<br>・また、CPU使用率等の数値により、ノード数を増減させ、ノードの処理能力を柔軟に対応させる。|[ECS Auto Scaling](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-auto-scaling.html)|
|3|Serverless|Batchサーバー|Lambda|・AWS Lambdaのサーバーレス実行により、AWSが自動的に可用性を管理する。<br>・複数AZで自動的に実行され、高可用性が確保される。|[Lambda可用性](https://docs.aws.amazon.com/lambda/latest/dg/lambda-intro-execution-role.html)|
|4|Multi-AZ Instance with Fail Over|DBサーバー|Aurora|・AWSサービスに付帯されたクラスタリングやレプリケーション等の技術を利用し、冗長化した構成で複数のノードを稼働する。<br>・ノードの異常発生時には、Reader インスタンスをポイントするようにDNS レコードが自動的に変更され、サービスの継続を図る。<br>・フェイルオーバー中は、通常 60 ～ 120 秒の利用不可時間が発生する。|[Aurora高可用性](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Concepts.AuroraHighAvailability.html)|
|5|Multi-AZ Instance with Fail Over|Cacheサーバー|ElastiCache|・AWSサービスに付帯されたレプリケーション等の技術を利用し、冗長化した構成で複数のノードを稼働する。<br>・ノードの異常発生時には、Replica ノードをポイントするようにDNS レコードが自動的に変更され、サービスの継続を図る。<br>・フェイルオーバー中は、短時間の利用不可時間が発生する。|[ElastiCache Multi-AZ](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/AutoFailover.html)|

その他、AWSサービスの可用性については、各サービスのSLAに準拠します。  
[AWSサービスSLA](https://aws.amazon.com/legal/service-level-agreements/?nc1=h_ls)

## 1.5.2 目標稼働率

本システムの目標稼働率は以下の通りとする。

| 項目 | 値 | AWS参考ドキュメント | 備考 |
|------|-----|-------------------|------|
| 目標稼働率 | 99.9%以上 | [AWS SLA](https://aws.amazon.com/legal/service-level-agreements/) | 計画停止除く24時間365日稼働 |

## 1.5.3 バックアップ方針

本システムのバックアップは以下の方針とする。

| 対象 | バックアップ方法 | 頻度 | 保管期間 | 保管場所 | AWS参考ドキュメント |
|------|----------------|------|---------|---------|-------------------|
| Aurora DB | 自動バックアップ | 日次 | 7日間 | 同一リージョン | [Auroraバックアップ](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Managing.Backups.html) |
| Aurora DB | スナップショット | 日次 | 35日間 | DRリージョン | [Auroraスナップショット](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Managing.Backups.html) |
| S3データ | Cross-Region Replication | リアルタイム | 無期限 | DRリージョン | [S3 CRR](https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html) |
| ECRイメージ | レプリケーション | プッシュ時 | 無期限 | DRリージョン | [ECRレプリケーション](https://docs.aws.amazon.com/AmazonECR/latest/userguide/replication.html) |
