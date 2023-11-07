# AWS

AWS入門として、VPC + EC2を無料枠で構築して、クライアント端末からSSH接続する方法。

![VPC+EC2.png](VPC+EC2.png)


## VPC

【AWS①】ネットワークを構築してみる  
https://zenn.dev/oreo2990/articles/bf3112bb6ccb48  

### IPアドレス

- VPC             :10.0.0.0/16 65,536個のIP
- PUBLIC  SUBNET 1:10.0.1.0/24    256個のIP
- PUBLIC  SUBNET 2:10.0.2.0/24    256個のIP
- PRIVATE SUBNET 3:10.0.3.0/24    256個のIP
- PRIVATE SUBNET 4:10.0.4.0/24    256個のIP

### AZ

- PUBLIC  SUBNET 1: ap-northeast-1a
- PUBLIC  SUBNET 2: ap-northeast-1b
- PRIVATE SUBNET 3: ap-northeast-1a
- PRIVATE SUBNET 4: ap-northeast-1b

### ルートテーブル、igw

- `VPC`にigwがアタッチされていること
- `VPC`のルートテーブルにigwが設定されていること
- `PUBLIC SUBNET`のルートテーブルにigwが設定されていること
- `PRIVATE SUBNET`のルートテーブルには、igwが設定されていないこと  
  ※igw=インターネットゲートウェイ

### ACL

すべて許可

### セキュリティグループ

- 配置先サブネットやサーバ種類の単位に作成
- 例えば、作業用にインバウンドでSSHを許可する

### 構成

- 例えば、PUBLIC  SUBNETには、API GW、ALB、WEBサーバ、踏み台
- 例えば、PRIVATE SUBNETには、DB、EC2内部サーバ（APサーバ、バッチサーバ）、ECS（サービス、タスク）


## EC2

【AWS】EC2インスタンスの作成方法解説！サーバーを作成して接続してみる  
https://engineer-ninaritai.com/aws-ec2-make/  

- VPCを選択
- SUBNETを選択
- セキュリティグループを選択
- OS
- インスタンスタイプ ※1年間無料枠
- ストレージ　 ※1年間無料枠
- キーペア

接続方法  
```
ssh -i key.pem ec2-user@IPアドレス
```

何らかのアプリ起動方法
```
sudo yum install git
sudo yum install java-11-amazon-corretto.x86_64
git clone https://github.com/namickey/spring-boot-mybatis-docker.git
cd spring-boot-mybatis-docker
cd web-css
./mvnw spring-boot:run

ブラウザからアクセス
`http://パブリックIP:8080/`
```


### PUBLIC SUBNETへ配置

- OSは、`Amazon Linux 2`
- インスタンスタイプは、t2.micro
- VPCは、1つ固定
- サブネットは、`PUBLIC SUBNET`の1 or 2
- Public IPを付与する
- Elastic IPは付与しなくてよい
- ストレージは8GB
- セキュリティグループは、PUBLIC向けでSSHを許可。インターネットからアクセスがあるため送信元IPはできるだけ限定する。
- アクセス方法は、例えば自宅マシンからSSHする。

### PRIVATE SUBNETへ配置

- OSは、`Amazon Linux 2`
- インスタンスタイプは、t2.micro
- VPCは、1つ固定
- サブネットは、`PRIVATE SUBNET`の1 or 2
- Public IPは無し
- Elastic IPは無し
- ストレージは8GB
- セキュリティグループは、PRIVATE向けでSSHを許可。送信元はPUBLICのセキュリティグループを指定。
- アクセス方法は、例えば、まず自宅マシンからPUBLIC SUBNETのEC2へSSHして、次にそのEC2内からPRIVATE SUBNETのEC2へSSHする。  
  ※PUBLIC側EC2を`踏み台`と呼ぶ


接続方法  
```
scp -i key.pem key.pem ec2-user@publicIPアドレス（PUBLIC側EC2）:~/
ssh -i key.pem ec2-user@IPアドレス（PUBLIC側EC2）

PUBLIC側EC2 > chmod 400 key.pem
PUBLIC側EC2 > ssh -i key.pem privateIPアドレス（PRIVATE型EC2）
```


### コスト、無料枠


- ネットワーク
  - 無し：VPC
  - 無し：SUBNET
  - 無し：ルートテーブル
  - 無し：インターネットゲートウェイ
  - 無し：ACL
  - 無し：セキュリティグループ
  - 無料：通信（外⇒AWS）
  - 無料枠：通信（AWS⇒外）
  - 無料：通信（同一AZ内）
  - `有料`：通信（異なるAZ間）
- EC2
  - 無し：Public IP
  - `有料`：Elastic IP
  - 無料枠：EC2インスタンス
  - 無料枠：ELB（ストレージ）

- WEB API
  - 無料枠：API Gateway
- CloudWatch
  - 無料枠：CloudWatch Logs
- ストレージ
  - 無料枠：S3
- データベース
  - 無料枠：RDS
  - `有料`：Aurora
- コンテナ
  - `有料`：ECS, Fargate
  - 無料枠：ECR
- ネットワーク
  - 無料枠：ALB
  - `有料`：NATゲートウェイ
  - `有料`：ECR VPCエンドポイント
  - `有料`：CloudWatch VPCエンドポイント
  - 無料：S3 VPCエンドポイント
  - 無料：ACM
  - `有料`：ROUTE53
- その他
  - `有料`：Secrets Manager


### PRIVATE SUBNETを使うなら、`NATゲートウェイ`or`VPCエンドポイント`が必要

そのトラフィック、NATゲートウェイを通す必要ありますか？適切な経路で不要なデータ処理料金は削減しましょう  
https://dev.classmethod.jp/articles/reduce-unnecessary-costs-for-nat-gateway/  

ECS+fargate+プライベートサブネットでコンテナを立ててアクセスする  
https://blog.not75743.com/post/ecs_private/  

- NATゲートウェイ無し：PRIVATE SUBNETに配置されたEC2からインターネットアクセスできない。`yum update`もできない。ECRからpullできない。S3もCloud Watchもダメ。
- NATゲートウェイ有り：PRIVATE SUBNETに配置されたEC2からインターネットアクセスできる。有料で高い。

- VPCエンドポイント無し：PRIVATE SUBNETに配置されたEC2からインターネットアクセスできない。`yum update`もできない。ECRからpullできない。S3もCloud Watchもダメ。
- VPCエンドポイント有り：PRIVATE SUBNETに配置されたEC2からインターネットアクセスできない。`yum update`もできない。ECR、S3、Cloud Watchは使えるようになる。


### ALB



python3 -m http.server










