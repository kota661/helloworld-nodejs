# IBM Cloud Kubernetes Service にデプロイしてみよう

- [IBM Cloud Kubernetes Service にデプロイしてみよう](#ibm-cloud-kubernetes-service-%E3%81%AB%E3%83%87%E3%83%97%E3%83%AD%E3%82%A4%E3%81%97%E3%81%A6%E3%81%BF%E3%82%88%E3%81%86)
  - [IBM Cloud Kubernetes Serviceとは？](#ibm-cloud-kubernetes-service%E3%81%A8%E3%81%AF)
  - [必要な環境](#%E5%BF%85%E8%A6%81%E3%81%AA%E7%92%B0%E5%A2%83)
  - [IKSクラスタの作成](#iks%E3%82%AF%E3%83%A9%E3%82%B9%E3%82%BF%E3%81%AE%E4%BD%9C%E6%88%90)
  - [IKSクラスターへのアクセス](#iks%E3%82%AF%E3%83%A9%E3%82%B9%E3%82%BF%E3%83%BC%E3%81%B8%E3%81%AE%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9)
  - [コンテナレジストリの作成](#%E3%82%B3%E3%83%B3%E3%83%86%E3%83%8A%E3%83%AC%E3%82%B8%E3%82%B9%E3%83%88%E3%83%AA%E3%81%AE%E4%BD%9C%E6%88%90)
  - [サンプルコードを取得](#%E3%82%B5%E3%83%B3%E3%83%97%E3%83%AB%E3%82%B3%E3%83%BC%E3%83%89%E3%82%92%E5%8F%96%E5%BE%97)
  - [ローカルマシンでDocker Build、テスト実行](#%E3%83%AD%E3%83%BC%E3%82%AB%E3%83%AB%E3%83%9E%E3%82%B7%E3%83%B3%E3%81%A7docker-build%E3%83%86%E3%82%B9%E3%83%88%E5%AE%9F%E8%A1%8C)
  - [プライベートレジストリにPush](#%E3%83%97%E3%83%A9%E3%82%A4%E3%83%99%E3%83%BC%E3%83%88%E3%83%AC%E3%82%B8%E3%82%B9%E3%83%88%E3%83%AA%E3%81%ABpush)
  - [Kubernetesクラスタへアプリをデプロイ](#kubernetes%E3%82%AF%E3%83%A9%E3%82%B9%E3%82%BF%E3%81%B8%E3%82%A2%E3%83%97%E3%83%AA%E3%82%92%E3%83%87%E3%83%97%E3%83%AD%E3%82%A4)
  - [サービス公開しアプリにアクセスしてみよう - Nodeport](#%E3%82%B5%E3%83%BC%E3%83%93%E3%82%B9%E5%85%AC%E9%96%8B%E3%81%97%E3%82%A2%E3%83%97%E3%83%AA%E3%81%AB%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9%E3%81%97%E3%81%A6%E3%81%BF%E3%82%88%E3%81%86---nodeport)
  - [サービス公開しアプリにアクセスしてみよう - Ingress](#%E3%82%B5%E3%83%BC%E3%83%93%E3%82%B9%E5%85%AC%E9%96%8B%E3%81%97%E3%82%A2%E3%83%97%E3%83%AA%E3%81%AB%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9%E3%81%97%E3%81%A6%E3%81%BF%E3%82%88%E3%81%86---ingress)
  - [お片付け](#%E3%81%8A%E7%89%87%E4%BB%98%E3%81%91)
  - [APPENDIX：その他の学習コンテンツ](#appendix%E3%81%9D%E3%81%AE%E4%BB%96%E3%81%AE%E5%AD%A6%E7%BF%92%E3%82%B3%E3%83%B3%E3%83%86%E3%83%B3%E3%83%84)


## IBM Cloud Kubernetes Serviceとは？

IBM Cloudでご提供しているKubernetesのマネージド・サービスです。
https://cloud.ibm.com/docs/containers/container_index.html#container_index


## 必要な環境
1. IBM Cloud のアカウント
2. IBM Cloud CLI のインストール
  
    [IBM Cloud CLI の概説](https://cloud.ibm.com/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli)
    
    必要なCLIと確認
    ```
    # ibmcloud
    ibmcloud --veresion

    # ibmcloud cli plugin
    # 出力結果に以下プラグインが含まれているか確認
    # - container-registry
    # - container-service/kubernetes-service
    ibmcloud plugin list

    # git
    git --version

    # docker
    docker --version #インストール確認
    docker ps        #起動しているかの確認

    # kubectl
    kubectl version --client
    ```
   

## IKSクラスタの作成
  IBM Cloud上にKubernetesクラスタを作成します

1. [IBM Cloud](https://cloud.ibm.com)にログイン
2. リソースの作成 > Kubernetes Service > 作成 の順に選択
3. 標準クラスタ または 無料クラスタを作成

    * 選択・入力項目
      * ロケーション: ダラス
      * クラスター・タイプ: 標準 or 無料
      * ワーカー・ゾーン: dal10
      * Kubernetesバージョン: stable
      * フレーバー: 2コア 4GB RAM
      * ワーカーノード: 1
      * クラスタ名: mycluster
    
    * 無料クラスタを選択した場合にお試しいただけない機能
      * Loadbarancer (専用のPublicIP経由でのアプリアクセス)
      * Ingress (ドメイン名でのアプリアクセス)


## IKSクラスターへのアクセス
  手元のマシンからIKSクラスタにアクセスするための接続情報の取得を行います。この手順が完了するとkubectlでKubernetesクラスタを操作できます。

1. 作成したクラスタを選択し**アクセス**のタブを選択
2. 画面に表示されたコマンドを実行

    * Mac: ターミナルを起動しスクリプトを実行してください。
    * Windows: **コマンドプロンプト**を起動しスクリプトを実行してください。

    入力例
    ```
    # 1. ご使用の IBM Cloud アカウントにログインします。
    ibmcloud login -a https://cloud.ibm.com

    # 2. 作業したい IBM Cloud Container Service の領域をターゲットにします。
    ibmcloud cs region-set us-south

    # 3. コマンドを使用して、環境変数を設定し、Kubernetes 構成ファイルをダウンロードします。
    ibmcloud cs cluster-config mycluster

    # 4. KUBECONFIG 環境変数を設定します。 前のコマンドからの出力をコピーし、ご使用の端末に貼り付けます。 コマンド出力は、以下のようになります。
    export KUBECONFIG=/Users/$USER/.bluemix/plugins/container-service/clusters/mycluster/kube-config-tok02-mycluster.yml

    # 5. ワーカー・ノードをリストすることによって、クラスターに接続できることを確認します。
    kubectl get nodes
    ```


## コンテナレジストリの作成
  専用のプライベートレジストリ空間を作成します。
  <地域ごとのレジストリURL>/**<名前空間名>**/<リポジトリ名>:<タグ名>

1. リージョンごとのレジストリ名を確認

    ```
    ibmcloud cr info
    ```

    > 出力例
    ```
    Container Registry                us.icr.io   < ここ
    Container Registry API endpoint   https://us.icr.io/api
    ...
    ```

2. IBM Cloud上に専用の名前空間を作成します。

    ```
    # 作成
    ibmcloud cr namespace-add <作成したい名前空間名>

    # 確認
    ibmcloud cr namespace-list
    ```
  

## サンプルコードを取得
  サンプルコードをダウンロード or git clone してローカルに取得

* ブラウザ経由でダウンロード
    1. [helloworld-nodejs](https://github.com/kota661/helloworld-nodejs)にアクセス
    2. Clone or downloadボタンよりDownload ZIP

* Git clone
    ```
    git clone https://github.com/kota661/helloworld-nodejs.git
    ```


## ローカルマシンでDocker Build、テスト実行
  Dockerfileを使ってDockerImageの作成を行います。
  このサンプルアプリではNode.jsを利用しており、オフィシャルのnode:10イメージをベースに利用しています。

1. ダウンロード or git clone したフォルダをカレントディレクトリに設定

    ```
    cd helloworld-nodejs
    ```

2. Docker Image の作成（Docker build)

    ```
    # docker build
    # docker build -t <リポジトリ名>:<タグ名> <ビルドパス>
    docker build -t us.icr.io/<作成した名前空間名>/helloworld-nodejs:latest .

    # build結果の確認
    docker images
    ```

3. 作成したイメージをもとに、ローカル実行

    ```
    # docker run でコンテナ実行
    docker run -d -p 80:80 us.icr.io/<作成した名前空間名>/helloworld-nodejs:latest

    # docker ps で実行ステータスの確認
    docker ps 
    ```

    * Optional
      * 実行中のコンテナの状態を確認
          `docker inspect <CONTAINER>`
      * 実行中のコンテナに入って操作
          `docker exec -it <CONTAINER> /bash`

4. ブラウザでアプリにアクセス
    [helloworld-nodejs](http://localhost:80)

5. 実行中のコンテナの停止
    ```
    # docker ps で実行中のコンテナIDを取得
    docker ps

    # docker stop でコンテナを停止させる
    docker stop <コンテナID>
    
    # docker ps --all で停止したことの確認
    docker ps --all

    # option: 停止したコンテナの削除
    docker rm <コンテナID>
    ```


## プライベートレジストリにPush
  作成したDocker ImageをIBM Cloud上のプライベートレジストリにPushします


1. docker cli でPushできるようにログイン
    ```
    ibmcloud cr login 
    ```

2. docker push でプライベートレジストリにイメージをPush

    ```
    docker push us.icr.io/<作成した名前空間名>/helloworld-nodejs:latest
    ```

3. ibmcloud cr images でPushできたことを確認

    ```
    ibmcloud cr images
    ```


## Kubernetesクラスタへアプリをデプロイ
  deployment.yml, service.ymlを使ってアプリをデプロイします

1. deployment.ymlをエディタで開いてイメージ名を先程Pushしたイメージ名に変更

    * 変更前: 
        kota661/helloworld-nodejs:latest
    * 変更後: 
        us.icr.io/<作成した名前空間名>/helloworld-nodejs:latest
   
    ```:deployment.yml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: hello-nodejs-deployment
      labels:
        app: hello-nodejs
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: hello-nodejs
      template:
        metadata:
          labels:
            app: hello-nodejs
        spec:
          containers:
          - name: hello-nodejs
            image: kota661/helloworld-nodejs:latest  < この行を編集してください
            ports:
            - containerPort: 80
    ```

2. deployment.ymlを使ってアプリのデプロイ
    ```
    kubectl apply -f deployment.yml
    ```

3. デプロイ結果の確認
    ```
    kubectl get deploy,pod
    ```


## サービス公開しアプリにアクセスしてみよう - Nodeport
  KubernetesのWorker Nodeが持つPublicIP経由でアクセスする方法です。無料のクラスタでご利用いただけます

1. service-nodeport.yml を使ってサービス公開
    ```
    kubectl apply -f service-nodeport.yml
    ```

2. Worker NodeのPublici IPを確認
    ```
    ibmcloud cs workers mycluster
    ```

    出力例
    ```
    ID                                                 Public IP         Private IP      Machine Type   State    Status   Zone    Version
    kube-hou02-pa8fda39f0d73142d9886c816cb59f208c-w1   173.193.122.126   10.76.141.239   free           normal   Ready    hou02   1.10.8_1531*
    ```


3. アプリにアクセス
   
    http://<Worker NodeのPublicIP>:31000

    上記出力例の場合は http://173.193.122.126:31000 です


## サービス公開しアプリにアクセスしてみよう - Ingress
    HTTPSレイヤーのロードバランサーを使って、ドメイン名でアクセスする方法です。有料クラスタのみご利用いただけます

1. クラスタに割り当てられたIngress Subdomainの確認
    ```
    ibmcloud ks cluster-get mycluster
    ```

    出力例
    ```
    Name:                   mycluster
    ...
    Ingress Subdomain:      mycluster-417610.us-south.containers.appdomain.cloud
    Ingress Secret:         mycluster-417610
    Workers:                2
    ...
    ```


2. service-ingress.ymlをエディタで開いてドメイン名を**２箇所**変更

    * 変更前: 
        host: dev.mycluster.jp-tok.containers.appdomain.cloud
    * 変更後: 
        host: dev.mycluster-417610.us-south.containers.appdomain.cloud
        

    ```:service-ingress.yml
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: hello-nodejs-ingressre
    spec:
      tls:
      - hosts:
        - dev.mycluster.jp-tok.containers.appdomain.cloud 　　　< 変更箇所
        secretName: mycluster 
      rules:
      - host: dev.mycluster.jp-tok.containers.appdomain.cloud　< 変更箇所
        http:
          paths:
          - path: /
            backend:
              serviceName: hello-nodejs-service
              servicePort: 80
    ```

3. service-ingress.ymlを使ってサービス公開
    ```
    kubectl apply -f service-ingress.yml
    ```

4. アプリにアクセス

    ```
    https://dev.<Ingress Subdomain>

    # 上記入力例の場合
    https://dev.mycluster-417610.us-south.containers.appdomain.cloud
    ```


## お片付け
1. クラスタの削除 or リソースの削除
    * クラスタを削除する場合
        ```
        ibmcloud ks cluster-rm mycluster
        ```

    * リソースを削除する場合
        ```
        kubectl delete deploy/hello-nodejs-deployment svc/hello-nodejs-npservice svc/hello-nodejs-service svc/hello-nodejs-ingressre
        ```

2. コンテナレジストリのImage削除
    ```
    ibmcloud cr image-rm us.icr.io/<作成した名前空間名>/helloworld-nodejs:latest
    ```


## APPENDIX：その他の学習コンテンツ
* Kubernetes クラスターにアプリをデプロイする方法
    https://cloud.ibm.com/docs/containers/cs_tutorials_apps.html#cs_apps_tutorial

* Calico ネットワーク・ポリシーを使用したトラフィックのブロック
    https://cloud.ibm.com/docs/containers/cs_tutorials_policies.html#policy_tutorial

* Continuous Deployment to Kubernetes
    https://cloud.ibm.com/docs/tutorials/continuous-deployment-to-kubernetes.html

