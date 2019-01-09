# Kubernetesのお勉強

https://www.youtube.com/watch?v=ObA1OEVdrQY&feature=youtu.be

Go,マイクロサービス,kubernetes,gRPC,メッシュサービス
このあたりのワードを中心に勉強する

で、上の動画に感動したので。セッションを維持したままデプロイできるし、Dockerコンテナをそのままデプロイできる、AutoScalingもやってくれるらしい。AWS EKSもあって良さげ。Opsworksとかもういらんね！ところでEKSのSってなに？

## インストール

```
$ brew cask install minikube

$ minikube version
```

> ハイパーバイザーが必要になります。Macの場合は、Hyperkit、xhyve、VirtualBox、VMware Fusionいずれかに対応しています。

つまり、minikubeは仮想マシン上で実行されるってこと。dockerはHyperkitで動いているらしい。VirtualBox入っとるし、たぶん問題ない

## クラスタ起動

```
$ minikube start

## ハイパーバイザ指定実行
$ minikube --vm-driver=virtualbox start
```

## クラスタバージョン確認

```
$ kubectl version
```

見にくい...
クライアントとサーバーとあるみたいだけど、一旦放置

```
## クラスタ詳細情報を表示
$ kubectl cluster-info

## ダッシュボードアクセス
$ minikube dashboard

## クラスタ一覧
$ kubectl get nodes
```

詳細情報よくわからんけど、ダッシュボードにはアクセスできた！

> Kubernetesはクラスターを管理するマスターとアプリケーション(コンテナ)を実行するノードの2種類があります。Minikubeではマスターのみ起動している状態になります。

今はマスターノードしか起動しておらんのやな。じゃあさっきのクライントとサーバーっちゅうのはなんのこっちゃ？

kubectl-cli→（kubeクライアント）→（kubeサーバー）→各node

こんなイメージ？

## アプリケーションデプロイ

```
$ kubectl run kubernetes-nginx --image=nginx:1.14.2 --port=8080

## 確認
$ kubectl get deployments
```

deploy(run)するとPODが作成される

コンテナがPOD内で稼働している状態

## アプリケーションへアクセス

PODはプライベートネットワークで隔離されているので、クラスターの外側からはアクセス不可

kubectlからの操作は必ずapiserverからproxy経由でコンテナへつながっている

（イメージ）

```
## 受付状態になる
$ kubectl proxy

## 別のターミナルでアクセス
### とりあえずバージョン確認
$ curl http://localhost:8001/version

### apiのいろいろが見れる
$ curl http://localhost:8001/api/v1

### 稼働しているpodを見て、ハッシュID付きのnameを確認 
$ curl http://localhost:8001/api/v1/namespaces/default/pods
（kubectl get pod でも同じものが見れます！）
$ POD_NAME=hogehoge

### podへアクセス
$ curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME

### proxyを経由しないとレスポンスが返れない？
$ curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME/proxy/

### podに入る
$ kubectl exec -ti $POD_NAME bash
```

ブラウザからアクセスできんやんけ！！ボケ！！

## Seviceを起動

```
$ kubectl expose deployment/kubernetes-nginx --type="NodePort" --port 8080
$ kubectl describe services/kubernetes-nginx
```

## 停止・削除

```aidl
$ minikube stop
$ minikube delete
```

## 参考

これをやっただけ

https://dev.classmethod.jp/cloud/kubernetes-tutorial-1/
