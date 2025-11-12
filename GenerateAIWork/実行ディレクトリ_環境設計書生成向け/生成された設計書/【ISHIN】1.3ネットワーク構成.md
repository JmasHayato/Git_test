# 1.3　ネットワーク構成

## 1.3.1 VPC設定

本システムでは、以下のVPC構成とする。

### VPC CIDR設定

| 環境 | VPC CIDR | AWS参考ドキュメント | 備考 |
|------|----------|-------------------|------|
| 本番環境 | 172.30.80.0/22 | [VPC CIDR設計](https://docs.aws.amazon.com/vpc/latest/userguide/subnet-sizing.html) | 本番環境用プライベートIP範囲 |
| ステージング環境 | 172.30.84.0/22 | [VPC CIDR設計](https://docs.aws.amazon.com/vpc/latest/userguide/subnet-sizing.html) | ステージング環境用IP範囲 |
| 本番DR環境 | 172.24.0.0/22 | [VPC CIDR設計](https://docs.aws.amazon.com/vpc/latest/userguide/subnet-sizing.html) | 本番DR環境用IP範囲 |
| ステージングDR環境 | 172.24.4.0/22 | [VPC CIDR設計](https://docs.aws.amazon.com/vpc/latest/userguide/subnet-sizing.html) | ステージングDR環境用IP範囲 |

### TGWアタッチメント用CIDR

| 環境 | CIDR | AWS参考ドキュメント | 備考 |
|------|------|-------------------|------|
| 本番TGWアタッチメント用1 | 172.30.254.64/27 | [Transit Gateway](https://docs.aws.amazon.com/vpc/latest/tgw/what-is-transit-gateway.html) | 本番環境TGW接続用 |
| 本番TGWアタッチメント用2 | 172.30.254.0/28 | [Transit Gateway](https://docs.aws.amazon.com/vpc/latest/tgw/what-is-transit-gateway.html) | 本番環境TGW接続用 |
| ステージングTGWアタッチメント用1 | 172.30.254.128/27 | [Transit Gateway](https://docs.aws.amazon.com/vpc/latest/tgw/what-is-transit-gateway.html) | ステージング環境TGW接続用 |
| ステージングTGWアタッチメント用2 | 172.30.254.16/28 | [Transit Gateway](https://docs.aws.amazon.com/vpc/latest/tgw/what-is-transit-gateway.html) | ステージング環境TGW接続用 |

参考: [[別紙]システム構成図](./[別紙]システム構成図.md)に詳細な構成を記載。

## 1.3.2 サブネット

Multi-AZのPublic/Privateの領域を定義し作成する。

### サブネット設定（本番環境）

| サブネット種別 | AZ | CIDR | AWS参考ドキュメント | 備考 |
|--------------|-----|------|-------------------|------|
| DMZ Subnet | AZ-A | 172.30.80.0/28 | [サブネット設計](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html) | パブリックサブネット |
| DMZ Subnet | AZ-C | 172.30.81.0/28 | [サブネット設計](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html) | パブリックサブネット |
| DMZ Subnet | AZ-D | 172.30.82.0/28 | [サブネット設計](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html) | パブリックサブネット |
| API Subnet | AZ-A | 172.30.80.64/26 | [サブネット設計](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html) | APIサーバー用プライベートサブネット |
| API Subnet | AZ-C | 172.30.81.64/26 | [サブネット設計](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html) | APIサーバー用プライベートサブネット |
| API Subnet | AZ-D | 172.30.82.64/26 | [サブネット設計](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html) | APIサーバー用プライベートサブネット |
| Batch Subnet | AZ-A | 172.30.80.128/26 | [サブネット設計](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html) | バッチサーバー用プライベートサブネット |
| Batch Subnet | AZ-C | 172.30.81.128/26 | [サブネット設計](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html) | バッチサーバー用プライベートサブネット |
| Batch Subnet | AZ-D | 172.30.82.128/26 | [サブネット設計](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html) | バッチサーバー用プライベートサブネット |
| DB Subnet | AZ-A | 172.30.80.16/28 | [サブネット設計](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html) | データベース用プライベートサブネット |
| DB Subnet | AZ-C | 172.30.81.16/28 | [サブネット設計](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html) | データベース用プライベートサブネット |
| DB Subnet | AZ-D | 172.30.82.16/28 | [サブネット設計](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html) | データベース用プライベートサブネット |

上記の要件により、[[別紙]システム構成図](./[別紙]システム構成図.md)に記載のサブネット区分とする。

## 1.3.3 ルーティング

・PublicサブネットはInternet Gatewayを利用し、Internetへのデフォルトゲートウエイの経路を設定する。  
またS3へインターネットを経由しない経路としてエンドポイントを設定する。  
・PrivateサブネットはNAT Gatewayを経由してインターネットへの経路を設定する。  

上記の要件により、以下の様なルートテーブル構成とする。

### (1)ルートテーブル区分

|No.|環境|種別|Availability-Zone|名前|所属するサブネット|
|:-:|---|---|---|---|---|
|1|本番環境|DMZ|AZ-A<br>AZ-C<br>AZ-D|Production DMZ RouteTable|Production DMZ Subnet (AZ-A)<br>Production DMZ Subnet (AZ-C)<br>Production DMZ Subnet (AZ-D)|
|2|本番環境|API|AZ-A<br>AZ-C<br>AZ-D|Production API RouteTable|Production API Subnet (AZ-A)<br>Production API Subnet (AZ-C)<br>Production API Subnet (AZ-D)|
|3|本番環境|Batch|AZ-A<br>AZ-C<br>AZ-D|Production Batch RouteTable|Production Batch Subnet (AZ-A)<br>Production Batch Subnet (AZ-C)<br>Production Batch Subnet (AZ-D)|
|4|本番環境|DB|AZ-A<br>AZ-C<br>AZ-D|Production DB RouteTable|Production DB Subnet (AZ-A)<br>Production DB Subnet (AZ-C)<br>Production DB Subnet (AZ-D)|

### (2)ルーティング設定

|No.|名前|ルーティング<br>宛先IP| <br>ターゲット|AWS参考ドキュメント|
|:-:|---|---|---|---|
|1|Production DMZ RouteTable|VPCのIPアドレスレンジ|VPC内|[ルートテーブル](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html)|
| ||0.0.0.0/0|Internet Gateway|[Internet Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)|
| ||S3|S3エンドポイント|[VPC Endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints.html)|
|2|Production API RouteTable|VPCのIPアドレスレンジ|VPC内|[ルートテーブル](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html)|
| ||0.0.0.0/0|NAT Gateway|[NAT Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html)|
| ||S3|S3エンドポイント|[VPC Endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints.html)|
|3|Production Batch RouteTable|VPCのIPアドレスレンジ|VPC内|[ルートテーブル](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html)|
| ||0.0.0.0/0|NAT Gateway|[NAT Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html)|
| ||S3|S3エンドポイント|[VPC Endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints.html)|
|4|Production DB RouteTable|VPCのIPアドレスレンジ|VPC内|[ルートテーブル](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html)|
| ||S3|S3エンドポイント|[VPC Endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints.html)|

## 1.3.4 NAT Gateway設定

| 項目名 | 値 | AWS参考ドキュメント | 備考 |
|--------|-----|-------------------|------|
| サービス | NAT Gateway | [NAT Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html) | プライベートサブネット内からのインターネットアクセスのために利用する |
| 配置AZ | AZ-A, AZ-C, AZ-D | [NAT Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html) | Multi-AZ配置による冗長化 |
| Elastic IP | 付与 | [Elastic IP](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html) | IPアドレス固定化 |

NatGatewayについては、IPアドレス固定化の為、Elastic IPを付与する。

## 1.3.5 Internet Gateway

- PublicサブネットからのインターネットアクセスのためにInternet Gatewayを設定する。
- VPC毎に1つのInternet Gatewayを作成する。

参考: [Internet Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)

## 1.3.6 カスタマーゲートウェイ

VPNでオンプレミスと接続する要件はないため、カスタマーゲートウェイは設定しない。

## 1.3.7 仮想プライベートゲートウェイ

VPN/Direct Connectでオンプレミスと接続する要件はないため、仮想プライベートゲートウェイは設定しない。

## 1.3.8 VPN接続

VPNでオンプレミスと接続する要件はないため、VPN接続は設定しない。

## 1.3.9 DirectConnect

VPN/Direct Connect等でオンプレミスと接続する要件はないため、Direct Connect接続は設定しない。

## 1.3.10 Endpoint

セキュリティ観点からVPCとAWSサービス間でインターネットを経由しない接続を行うためエンドポイントを設定する。

|No.|システム|環境  |サービス名                               |エンドポイントタイプ|VPC|AWS参考ドキュメント|
|:-:|---|---|---|:-:|---|---|
|1 |ISHIN|本番環境|com.amazonaws.ap-northeast-1.s3         |Gateway  |Production VPC|[S3 Endpoint](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-s3.html)|
|2 |     |       |com.amazonaws.ap-northeast-1.ecr.api    |Interface|Production VPC|[ECR Endpoint](https://docs.aws.amazon.com/AmazonECR/latest/userguide/vpc-endpoints.html)|
|3 |     |       |com.amazonaws.ap-northeast-1.ecr.dkr    |Interface|Production VPC|[ECR Endpoint](https://docs.aws.amazon.com/AmazonECR/latest/userguide/vpc-endpoints.html)|
|4 |     |       |com.amazonaws.ap-northeast-1.logs       |Interface|Production VPC|[CloudWatch Logs Endpoint](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/cloudwatch-logs-and-interface-VPC.html)|
|5 |     |       |com.amazonaws.ap-northeast-1.secretsmanager|Interface|Production VPC|[Secrets Manager Endpoint](https://docs.aws.amazon.com/secretsmanager/latest/userguide/vpc-endpoint-overview.html)|
|6 |     |       |com.amazonaws.ap-northeast-1.sts        |Interface|Production VPC|[STS Endpoint](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_sts_vpce.html)|
