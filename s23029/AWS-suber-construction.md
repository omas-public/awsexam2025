# AWS ネットワーク＆サーバー構築

## 設計

### VPCを設計せよ

- VPC name:my-filed-626
- VPC network:192.168.10.0/24

### Public Subnet 及び Private Subnetを設計せよ

#### Public subnet-1a

- Public Subnet name:public-subnet-626-1a
- Public Subnet network:192.168.10.0/27

#### Private subnet-1a

- Public Subnet name:publlic-subnet-626-1a
- Public Subnet network:192.168.10.32/27

#### Public subnet-1c

- Public Subnet name:public-subnet-626-1c
- Public Subnet network:192.168.10.64/27

#### Private subnet-1c

- Public Subnet name:publlic-subnet-626-1c
- Public Subnet network:192.168.10.96/27


### PublicSubnet及びPrivateSubnetのroute tableを設計せよ

- PublicSubnet　route table:192.168.10.0
- PrivateSubnet　route table:192.168.10.0


### 踏み台サーバ，Webサーバ，DBサーバに適切なIPアドレスを割り振れ


#### 踏み台サーバ

- 踏み台サーバ subnet:public-subnet-626-1c
- 踏み台サーバ Name:my-fumidai-626
- 踏み台サーバ IP:192.168.10.70

#### Webサーバ

- Webサーバ subnet:public-subnet-626-1c
- Webサーバ Name:my-Web-626
- Webサーバ IP:192.168.10.80

#### DBサーバ

- DBサーバ subnet:privaet-subnet-626-1a
- DBサーバ Name:my-DB-626
- DBサーバ IP:192.168.10.40

### 踏み台サーバ，Webサーバ，DBサーバに適切なSecurityGroupを設計せよ

#### 踏み台サーバSG

name:for-fumidai
開放するサービス(port),開放するネットワーク
ssh: 22     0.0.0.0/0

#### WebサーバSG

name:for-Web
開放するサービス(port),開放するネットワーク
ssh: 22     192.168.10.70(踏み台からのアクセス)
http: 88    0.0.0.0/0

#### DBサーバSG

name:for-DB
開放するサービス(port),開放するネットワーク
ssh: 22     192.168.10.70(踏み台からのアクセス)
mysql: 3306     0.0.0.0/0
icmp :     192.168.10.70

#### RDS作成

name: RDS-626
## 設計が終了したら，設計に基づいて作成せよ
