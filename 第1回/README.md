# Kubernetesのお勉強

https://www.youtube.com/watch?v=ObA1OEVdrQY&feature=youtu.be

Go,マイクロサービス,kubernetes,gRPC,メッシュサービス
このあたりのワードを中心に勉強する

で、上の動画に感動したので。セッションを維持したままデプロイできるし、Dockerコンテナをそのままデプロイできる、AutoScalingもやってくれるらしい。AWS EKSもあって良さげ。Opsworksとかもういらんね！ところでEKSのSってなに？

## 環境
minikubeだとcontrollerのセットアップとかでいろいろ大変だったので、GCPのGKEというサービスを利用してみる

https://console.cloud.google.com/kubernetes/list?project=study-k8s-228114

## イメージ図

### 基本

<img src="https://www.redhat.com/cms/managed-files/kubernetes-diagram-2-824x437.png">

### APIサーバーを通してやり取り

<img src="https://tech-lab.sios.jp/wp-content/uploads/2018/02/Screen-Shot-2018-02-15-at-8.01.08-1.png">

### 外からはServiceにアクセスして内部で振り分けされる

<img src="https://cdn-ak.f.st-hatena.com/images/fotolife/o/ornew/20180413/20180413104022.png">

### 

## 用語

### API Server
Restのインタフェースを持つプロセスで、Kubernetesのリソース情報(Pod似に関する情報やCluster IPなど)を管理します。kube-proxyやkubeletなどのプロセスは、このAPI Serverを介して、お互いの情報をやり取りします。

### etcd
kubernetesの情報を保存する高信頼分散KVSです。API Serverは、etcdに情報を保存します。他にもflannel(Pod間の通信を実現するVXLAN管理サービス)も利用します。

### kubelet
Podの生成や停止を行います。定期的にAPI Serverに問い合わせて、生成や停止の数やタイミングを決めています。

### scheduler
各Nodeの空き状況や状態を監視して、その結果をAPI Serverを通じて、kubeletに伝える役割を持っています。この情報を参考にして、kubeletはコンテナを生成します。例えば、後述しますが、ReplicaSetで複数台スケールするクラスタを作成したときは、このSchedulerが開いているNodeにコンテナを作成するようkubeletに指示します。そして、コンテナは複数のNodeに配置されます。

### kube-proxy
Cluster IPの管理を行います。Cluster IPとは、サービスにアクセスするための代表的なIPアドレスになります。Load BalancerなどのVIPに近いイメージです。このIPアドレスにアクセスすると、そのリクエストは複数のPodに分散されます。kube-proxyのお仕事は、定期的にAPI Server経由でKubernetesのリソース情報にアクセスし、必要に応じてiptableseのルールを作成し、iptablesの機能によって負荷分散を行います。

### コマンド群
kubectlなどのコマンド群です。ユーザーのインターフェースになります。このコマンドを使って、クラスタを作成したり削除したりとか、Kubernetesにおける全ての管理をします。実際はAPI Serverにリクエストを投げているます。

### Pod
Kubernetesのデプロイの単位になります。Podは複数のコンテナをまとめる論理的な単位です。

### Service
論理的なポッドのセットと、それにアクセスするポリシーを定義する抽象的なものです。 これらは、しばしばマイクロサービスとされます。

### Deployment
ローリングアップデートやロールバックといったデプロイ管理の仕組みを提供するものです。

## やってみる

+ GKEにアクセス（初期設定済） 
+ kubectl run nginx --image=nginx:1.11.3
+ kubectl get pods
+ kubectl expose deployment nginx --port 80 --type LoadBalancer
+ kubectl get services
+ git clone https://github.com/mihirat/advent.git
+ kubectl apply -f k8s/deployment.yml
+ kubectl apply -f k8s/service.yml
+ kubectl scale deployments/web-server --replicas=4
+ kubectl get pods -o wide
+ kubectl get deployments
+ kubectl set image deployments/web-server go-server=jocatalin/kubernetes-bootcamp
+ kubectl rollout status deployments/web-server

## 課題

- イメージのアップデートはOKだけど、アプリケーション更新はどうするのか
    - deploymentの中身を変えてrolloutコマンドを実行する
- 手動スケールアウトでなくAutoスケーリングはどうするのか
- minikube,EKSを利用する

## 参考

https://qiita.com/mihirat/items/ebb0833d50c882398b0f
https://dev.classmethod.jp/cloud/kubernetes-tutorial-4/
