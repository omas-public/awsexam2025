# AWS Exam 2025

## network設計

192.168.10.0/24 の ネットワークが与えられる
このネットワークを4つに分割して2番め以降の3つのネットワークを使い
以下のネットワークのそれぞれのインターフェースのIPアドレスとルーティングテーブルを作成せよ

host1 - router1 - router2 - host2

### 問1 host1のネットワークアドレス，IPアドレス，ルーティングテーブルを答えよ
192.168.10.32/27	192.168.10.40       PublicRouteTable

### 問2 router1のネットワークアドレス，IPアドレス，ルーティングテーブルをすべて答えよ
192.168.10.32/27    0.0.0.0/0           PrivateRouteTable

### 問3 router1のネットワークアドレス，IPアドレス，ルーティングテーブルをすべて答えよ
192.168.10.96/27    0.0.0.0/0           PrivateRouteTable

### 問4 host2のネットワークアドレス，IPアドレス，ルーティングテーブルを答えよ
192.168.10.96/27    192.168.10.100      PublicRouteTable

### 問5 AWS上にWordPressを構築し，その構築手順書とWordpressの画面のスクショを提出せよ，ただし以下の条件とする

- 192.168.10.0/24 のネットワークを8分割し, PublicSubnetとPrivateSubnetのペアを2つ構築する(4つのサブネット)
- PublicSubnetに踏み台サーバとWebサーバを構築せよ
- PrivateSubnetにRDSのDBサーバを構築せよ

# AWS WordPress 構築手順書（192.168.10.0/24 ネットワーク）

この手順書は、AWS の各サービスを利用して WordPress を構築するための手順をまとめたものです。

---

## 前提条件

* AWS アカウントが作成済みであること
* EC2 キーペア（秘密鍵 .pem ファイル）を作成済みであること

---

## 1. VPC の作成

1. AWS 管理コンソールで「VPC」を検索して開く
2. 「VPC を作成」ボタンをクリックし、次の情報を入力：

    * 名前タグ：VPC領域
    * IPv4 CIDR ブロック：`192.168.10.0/24`
3. 作成時に以下 4 つのサブネットを追加：

    * パブリックサブネット：`192.168.10.32/27`
    * パブリックサブネット2：`192.168.10.64/27`
    * プライベートサブネット：`192.168.10.96/27`
    * プライベートサブネット2：`192.168.10.128/27`

---

## 2. インターネットゲートウェイの作成

1. 「インターネットゲートウェイ」を開き、「作成」→ 名前タグ：`internet-gateway`
2. 作成後、「VPC にアタッチ」で VPC領域 を選択

---

## 3. ルートテーブルの設定

### パブリック用

1. 「ルートテーブル」→「作成」→ 名前：`PublicRouteTable`
2. 「ルートを追加」→

    * 送信先：`0.0.0.0/0`
    * ターゲット：`internet-gateway`
3. 「サブネットの関連付け」でパブリックサブネットとパブリックサブネット2 を選択

### プライベート用

1. 「ルートテーブル」→「作成」→ 名前：`PrivateRouteTable`
2. 「サブネットの関連付け」でプライベートサブネットとプライベートサブネット2 を選択

---

## 4. セキュリティグループの設定

### JumpSG（踏み台）

* ポート：22（SSH）
* ソース：`0.0.0.0/0`

### WebSG（Webサーバ）

* ポート：80（HTTP）、22（SSH）
* ソース：

    * HTTP：`0.0.0.0/0`
    * SSH：`192.168.10.40/32`

### DBSG（DBサーバ）

* ポート：3306（MySQL）、22（SSH）、ICMP
* ソース：`192.168.10.40/32`

---

## 5. EC2 インスタンスの作成

### 踏み台サーバ

* サブネット：192.168.10.32/27
* IP：192.168.10.40
* セキュリティグループ：JumpSG

### Web サーバ

* サブネット：192.168.10.32/27
* IP：192.168.10.50
* セキュリティグループ：WebSG

### DB サーバ

* サブネット：192.168.10.96/27
* IP：192.168.10.100
* セキュリティグループ：DBSG

---

## 6. RDS（MariaDB）の作成

1. 「RDS」→「データベース作成」
2. エンジン：MariaDB
3. ユーザー名：`admin`（任意）
4. パスワード：任意
5. DB名：`wordpress`
6. サブネットグループにプライベートサブネット2つを指定
7. セキュリティグループ：DBSG を選択

---

## 7. 踏み台サーバへ秘密鍵をコピー

ローカルPCから次を実行：

```bash
scp -i ~/.ssh/my-key.pem ~/.ssh/my-key.pem ec2-user@<踏み台のパブリックIP>:/home/ec2-user/
```

踏み台にSSHで接続：

```bash
ssh -i ~/.ssh/my-key.pem ec2-user@<踏み台のパブリックIP>
chmod 400 ~/my-key.pem
```

---

## 8. Webサーバへ SSH 接続

踏み台上で次を実行：

```bash
ssh -i ~/my-key.pem ec2-user@192.168.10.50
```

---

## 9. Webサーバへ WordPress をインストール

```bash
# Apache
sudo dnf -y install httpd
sudo systemctl enable --now httpd

# PHP
sudo dnf -y install php php-mbstring php-mysqli

# WordPress
wget https://ja.wordpress.org/latest-ja.tar.gz
sudo tar xzvf latest-ja.tar.gz -C /var/www/html --strip-components=1

# 所有者
sudo chown -R apache:apache /var/www/html
sudo systemctl restart httpd
```

---

## 10. WordPress 初期設定

ブラウザで `http://<WebサーバのパブリックIP>` にアクセスし、以下を入力：

* データベース名：`wordpress`
* ユーザー名 / パスワード：RDS 作成時の値
* データベースホスト名：RDS のエンドポイント

![s23010.png](../../../Pictures/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88/s23010.png) 

解答は学籍番号.mdをつくりその上で解答せよ, スクリーンショットは学籍番号.pngとする