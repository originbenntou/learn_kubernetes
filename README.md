# Kubernetesのお勉強 第2回

## AWS EKSを使ってみよう

- 2018年12月20日 に東京リージョンでもサービス利用可能となった
- それまでは北部バージニアとオレゴン、他2つくらいだった
- 東京では出たばっかりのサービスなので、使える機能やリファレンスがまだ整備しきれてない様子
- お金がかかる EKSクラスタ → 0.1USD/h（コンテナを稼働させるEC2インスタンスはまた別途料金が発生）
- ちなみにGCPのk8sサービスであるGKEは無料

### 参考サイト

https://dev.classmethod.jp/cloud/aws/eks-getting-started/

## IAMユーザ、ロール作成

- eks読み書き権限ありのユーザを作成
    - credential設定を済ませCLIで操作できるようにしておく
- eks用のロールを作成
    - cluster構成用のポリシーがデフォルトであるのでそれをアタッチしておく（ClusterポリシーとServiceポリシー）

## VPC作成

- CloudFormation + S3にあるEKS用テンプレ で今回デプロイする環境をつくる

![イメージ](https://onlinehelp.tableau.com/current/server-linux/en-us/Img/ts_aws_three_az.png)

## ローカル環境セットアップ

- AWSCLIは前の手順で設定を済ませている前提
- 認証に `heptio-authenticator-aws` なるものを使うようだが、AWS公式は `aws-iam-authenticator` を使っていた。

## クラスタ作成

- コンソールから手動で作成すると、認証エラーで先に進めなくなるので注意。
    - クラスタを作成するユーザとcredentialのユーザが一致する必要がある。
    - そのため、CLIでクラスタを作成する。（コンソールでできるかどうかは試してない）
    
```aidl
# ロールやサブネットは前の手順で作成したものをセット
$ aws eks create-cluster --name eks-cluster \
--role-arn arn:aws:iam::XXXXXXXXXXXXXXX:role/eksXXXXXXRole \
--resources-vpc-config \
subnetIds=subnet-XXXXXXXXXXXXXXX,subnet-XXXXXXXXXXXXXXX,subnet-XXXXXXXXXXXXXXX,securityGroupIds=sg-XXXXXXXXXXXXXXX
```

- コンソールで見守ってると、クラスタがActiveになる。
    - もしくはCLIコマンドでクラスタが作成成功するか確認できる。

![イメージ](https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2018/06/060-960x367.png)

```aidl
$ aws eks describe-cluster --name eks-cluster --query cluster.status
```

## kubectl設定

- ローカルからkubernetesのAPI操作を可能にするkubectlの設定をする
- 認証周りがうまくいってないと以下のようなエラーがでる
    - 問題解決で参考にしたサイト https://qiita.com/NaokiIshimura/items/60f90d9d925ca2b103bb

```aidl
error: the server doesn't have a resource type "svc"
```
## ワーカーノード作成

- コンテナを稼働させるEC2インスタンスとそれらにアクセスを振り分けるELBを作成する
- CloudFormation + S3テンプレ
- 今回EC2が3つ、ELBが1つ

## クラスタを有効にする

- ワーカーノードが互いに通信を始める

## アプリケーションデプロイ

https://github.com/kubernetes/examples/tree/master/guestbook-go

## 出来上がり

http://ab39cae792a2e11e9b97312f27349ddf-56620773.us-east-1.elb.amazonaws.com:3000/
