# 1.6 拡張性

## 1.6.1 拡張性の方針

### (1)対象

- AWS上に構築するFargateタスク、Aurora Instance、ElastiCache Node、Lambda関数  
※ その他AWSサービスについては、AWS仕様に従い、原則として性能劣化なく利用可能であるため、拡張性を考慮しない。  
（従量での課金増）

### (2)拡張契機

- Fargate APIサーバーのAutoScaling機能により自動拡張する。
- Auroraのリードレプリカは手動で追加可能。
- お客様からの連絡を契機に、協議の上拡張対応を行う。

### (3)拡張時対応

- 拡張が必要になった場合、拡張規模によっては、掛る費用について別途見積もりを提示する。
- 原則、拡張時は計画停止が可能である事を前提とする。
- Fargate APIサーバーのAutoScaling機能による拡張対応については、自動的に実施される。

## 1.6.2 各コンポーネントの拡張方式

### APIサーバー（Fargate）

| 拡張方式 | 内容 | AWS参考ドキュメント |
|---------|------|-------------------|
| 水平スケーリング | タスク数を3〜5台で自動スケーリング | [ECS Auto Scaling](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-auto-scaling.html) |
| 垂直スケーリング | CPU/メモリのタスク定義変更（計画停止） | [Fargateタスクサイズ](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/capacity-tasksize.html) |

### データベース（Aurora）

| 拡張方式 | 内容 | AWS参考ドキュメント |
|---------|------|-------------------|
| ライター拡張 | 垂直スケーリング（インスタンスサイズ変更、短時間ダウンタイム発生） | [Aurora拡張](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Managing.Performance.html) |
| リーダー拡張 | 水平スケーリング（リードレプリカ追加、無停止拡張、最大15台まで） | [Aurora Auto Scaling](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Integrating.AutoScaling.html) |
| ストレージ拡張 | 自動ストレージ拡張（10GiB〜128TiB自動拡張） | [Aurora Storage](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Overview.StorageReliability.html) |

### キャッシュ（ElastiCache）

| 拡張方式 | 内容 | AWS参考ドキュメント |
|---------|------|-------------------|
| 垂直スケーリング | ノードタイプ変更（計画停止） | [ElastiCacheノードタイプ](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/CacheNodes.SupportedTypes.html) |
| 水平スケーリング | レプリカノード追加 | [ElastiCache Redis複製](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/Replication.html) |

### バッチ（Lambda）

| 拡張方式 | 内容 | AWS参考ドキュメント |
|---------|------|-------------------|
| 自動スケーリング | 同時実行数の自動拡張 | [Lambda同時実行](https://docs.aws.amazon.com/lambda/latest/dg/lambda-concurrency.html) |
| メモリ増加 | Lambda設定変更 | [Lambda設定](https://docs.aws.amazon.com/lambda/latest/dg/configuration-memory.html) |

## 1.6.3 コスト最適化設定

### 環境運用方針

| 項目名 | 内容 | AWS参考ドキュメント | 備考 |
|--------|------|-------------------|------|
| 設計方針 | 全環境設定をそろえる | [AWS Well-Architected](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html) | 環境統一方針 |
| コスト削減方針 | 本番以外の環境は利用しない時間帯は停止 | [コスト最適化](https://docs.aws.amazon.com/wellarchitected/latest/cost-optimization-pillar/welcome.html) | 停止によるコスト削減 |
| メンテナンス時コスト方針 | 長期メンテナンス時のリソース一時停止検討 | [ECSスケジュール起動停止](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-automation.html) | 計画メンテナンス時のコスト最適化 |
| メンテナンス用リソース最小化 | メンテナンス管理機能は必要最小限のリソース構成 | [Lambda料金最適化](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html) | メンテナンス管理機能のコスト効率化 |
