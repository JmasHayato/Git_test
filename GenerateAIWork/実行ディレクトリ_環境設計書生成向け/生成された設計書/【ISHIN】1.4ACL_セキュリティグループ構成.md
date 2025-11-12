# 1.4 ACL/セキュリティグループ構成

## 1.4.1 ネットワークACL

Subnet間の通信について、制御は行わない。  
上記の要件により、以下の様なネットワークACL構成とする。

### (1)ネットワークACL区分

|No.|環境|種別|Availability-Zone|名前|所属するサブネット|
|:-:|---|---|--|---|---|
|1|本番環境|DMZ|AZ-A<br>AZ-C<br>AZ-D|Production DMZ NetWork ACL|Production DMZ Subnet (AZ-A)<br>Production DMZ Subnet (AZ-C)<br>Production DMZ Subnet (AZ-D)|
|2|本番環境|API|AZ-A<br>AZ-C<br>AZ-D|Production API NetWork ACL|Production API Subnet (AZ-A)<br>Production API Subnet (AZ-C)<br>Production API Subnet (AZ-D)|
|3|本番環境|Batch|AZ-A<br>AZ-C<br>AZ-D|Production Batch NetWork ACL|Production Batch Subnet (AZ-A)<br>Production Batch Subnet (AZ-C)<br>Production Batch Subnet (AZ-D)|
|4|本番環境|DB|AZ-A<br>AZ-C<br>AZ-D|Production DB NetWork ACL|Production DB Subnet (AZ-A)<br>Production DB Subnet (AZ-C)<br>Production DB Subnet (AZ-D)|
|5|本番環境|TGW|AZ-A<br>AZ-C<br>AZ-D|Production TGW NetWork ACL|Production TGW Subnet (AZ-A)<br>Production TGW Subnet (AZ-C)<br>Production TGW Subnet (AZ-D)|

### （２）ネットワークACL設定

|No.|名前|方向|ルールNo ※|タイプ|プロトコル|ポート範囲|送信元/送信先|拒否 or 許可|
|:-:|---|---|---|---|---|---|---|---|
|1|Production DMZ NetWork ACL|インバウンド|100|すべてのトラフィック|すべて|すべて|0.0.0.0/0|許可|
||||*|すべてのトラフィック|すべて|すべて|0.0.0.0/0|拒否|
|||アウトバウンド|100|すべてのトラフィック|すべて|すべて|0.0.0.0/0|許可|
||||*|すべてのトラフィック|すべて|すべて|0.0.0.0/0|拒否|
|2|Production API NetWork ACL|インバウンド|100|すべてのトラフィック|すべて|すべて|0.0.0.0/0|許可|
||||*|すべてのトラフィック|すべて|すべて|0.0.0.0/0|拒否|
|||アウトバウンド|100|すべてのトラフィック|すべて|すべて|0.0.0.0/0|許可|
||||*|すべてのトラフィック|すべて|すべて|0.0.0.0/0|拒否|
|3|Production Batch NetWork ACL|インバウンド|100|すべてのトラフィック|すべて|すべて|0.0.0.0/0|許可|
||||*|すべてのトラフィック|すべて|すべて|0.0.0.0/0|拒否|
|||アウトバウンド|100|すべてのトラフィック|すべて|すべて|0.0.0.0/0|許可|
||||*|すべてのトラフィック|すべて|すべて|0.0.0.0/0|拒否|
|4|Production DB NetWork ACL|インバウンド|100|すべてのトラフィック|すべて|すべて|0.0.0.0/0|許可|
||||*|すべてのトラフィック|すべて|すべて|0.0.0.0/0|拒否|
|||アウトバウンド|100|すべてのトラフィック|すべて|すべて|0.0.0.0/0|許可|
||||*|すべてのトラフィック|すべて|すべて|0.0.0.0/0|拒否|
|5|Production TGW NetWork ACL|インバウンド|100|すべてのトラフィック|すべて|すべて|0.0.0.0/0|許可|
||||*|すべてのトラフィック|すべて|すべて|0.0.0.0/0|拒否|
|||アウトバウンド|100|すべてのトラフィック|すべて|すべて|0.0.0.0/0|許可|
||||*|すべてのトラフィック|すべて|すべて|0.0.0.0/0|拒否|

※ルールNoの数字が小さい順から評価され、いずれにも当てはまらなかった通信は"*"のルールが適用される。


## 1.4.2 セキュリティグループ

- ノード間通信を制御、Subnet間は制御しない。
- 各ノードにそれぞれセキュリティグループを設定し、詳細な通信制限を行う。
- インバウンド通信に関しては、必要最小限の通信のみ開放する。
- アウトバウンド通信に関しては制限を実施しない。
- メンテナンス管理アクセスは特定IPからのHTTPS(443)アクセスを許可する。
- メンテナンス時のトラフィック制御はALBレベルで実施し、SecurityGroupは維持する。
- Fargateコンテナユーザは書き込み不可権限とする。

上記の要件により、以下の様なセキュリティグループ構成とする。


### (1)セキュリティグループ区分

|No.|環境|適用ノード|名前|
|:-:|---|---|---|
|1|本番環境|API Server|Production API Server SecurityGroup|
|2|本番環境|Batch Server|Production Batch Server SecurityGroup|
|3|本番環境|DB|Production DB SecurityGroup|
|4|本番環境|Cache|Production Cache SecurityGroup|
|5|本番環境|ALB|Production ALB SecurityGroup|
|6|本番環境|Bastion Server|Production Bastion Server SecurityGroup|
|7|本番環境|VPC Endpoint|Production VPC Endpoint SecurityGroup|

### (2)セキュリティグループ設定

|No.|名前|方向|タイプ|プロトコル|ポート範囲|送信元/送信先|
|:-:|---|---|---|---|---|---|
|1|Production ALB SecurityGroup|インバウンド  |HTTP|TCP|80|0.0.0.0/0|
| |                           |             |HTTPS|TCP|443|0.0.0.0/0|
| ||アウトバウンド|すべてのトラフィック|すべて|すべて|0.0.0.0/0|
|2|Production API Server SecurityGroup|インバウンド|HTTP|TCP|80|Production ALB SecurityGroup|
| ||アウトバウンド|すべてのトラフィック|すべて|すべて|0.0.0.0/0|
|3|Production Batch Server SecurityGroup|インバウンド|-|-|-|-|
|||アウトバウンド|すべてのトラフィック|すべて|すべて|0.0.0.0/0|
|4|Production DB SecurityGroup|インバウンド|MYSQL/Aurora|TCP|3306|Production API Server SecurityGroup|
| |||MYSQL/Aurora|TCP|3306|Production Batch Server SecurityGroup|
|||アウトバウンド|すべてのトラフィック|すべて|すべて|0.0.0.0/0|
|5|Production Cache SecurityGroup|インバウンド|Redis|TCP|6379|Production API Server SecurityGroup|
| |||Redis|TCP|6379|Production Batch Server SecurityGroup|
|||アウトバウンド|すべてのトラフィック|すべて|すべて|0.0.0.0/0|
|6|Production Bastion Server SecurityGroup|インバウンド|SSH|TCP|22|管理者IP|
|||アウトバウンド|すべてのトラフィック|すべて|すべて|0.0.0.0/0|
|7|Production VPC Endpoint SecurityGroup|インバウンド|HTTPS|TCP|443|Production API Server SecurityGroup|
| |||HTTPS|TCP|443|Production Batch Server SecurityGroup|
|||アウトバウンド|すべてのトラフィック|すべて|すべて|0.0.0.0/0|

※セキュリティグループは許可設定のみが設定可能。ルールに当てはまらない通信はすべて拒否される。
