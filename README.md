# k8s-webservice-demo

<h2>Local:</h2>
[t-saito@MacBookPronoMacBook-Pro k8s-webservice-demo]$ pwd
/Users/t-saito/src/k8s-webservice-demo


<h2>参照</h2>
・GKE上webアプリケーション構築方法紹介
https://www.devsamurai.com/ja/gcp-gke-and-web-app-deploy-introduction/

・「プログラマのためのDocker教科書 第2版」

<h2>課題</h2>
・ブルーグリーンデプロイメント

<h2>ステップ１</h2>
環境変数にGCPのプロジェクトIDを設定する

$ PROJECT_ID=$(gcloud config list project --format "value(core.project)") 

<h2>ステップ２</h2>
Dockerイメージの作成とGoogle Container Registryへのアップロード

$ gcloud builds submit --config config/cloudbuild.yaml  .

<h2>ステップ３</h2>
<h2>k8sクラスターの作成と環境設定</h2>
$ gcloud container clusters create imageview --zone=asia-northeast1-a <br>
$ gcloud container clusters get-credentials imageview --zone=asia-northeast1-a

🌾 名前: imageview   ゾーン：　asianortheast1-a

<h2>ステップ４</h2>
ConfigMapとSecretsの登録<br>

$ kubectl create -f config/configmap.yaml<br>
$ kubectl create -f config/secrets.yaml

<h2>ステップ５</h2>
アプリのデプロイ（テプロイメント定義ファイル）

🌾手動で deployment-blue.yaml と deployment-green.yaml　の <PROJECT_ID> の部分を変更すること。

$ kubectl create -f config/deployment-blue.yaml<br>
$ kubectl create -f config/deployment-green.yaml

<h2>❓【疑問】</h2>
なぜyamlファイルで環境変数を展開できる場合とできない場合があるのか？

例）　/config/cloudbuild.yaml は出来たが、 deployment-blue.yaml　などは出来なかった。

展開を利用する案。  .env  ファイルの利用？


<h2>ステップ６</h2>
Podの確認
$ kubectl get pods

Podの停止
$ kubectl delete pod  pod-name

すべて停止
$ kubectl delete pod config/deployment-blue.yaml<br>
$ kubectl delete pod config/deployment-green.yaml

<h2>ステップ7</h2>
アプリの外部公開（サービスの利用）
サービス定義ファイルの

$ kubectl create -f config/service.yaml

エンドポイントの確認
①管理画面 - Kubernetes Engine - ServiceとIngress - エンドポイント<br>
②コマンドの利用   $ kubectl get services

・Macからコマンドで特定のURLをGoogle Chromeで開く<br>
$ open -a '/Application/Google Chrome.app'  http://url-address
<br>

・サービスの非公開
$ kubectl delete -f config/service.yaml

<h2>ステップ8</h2>
アプリのバージョンアップ（Blue-Green Deployment）<br>
①管理画面での変更方法<br>
②コマンドで変更（エディタ）<br>
$ kubectl edit svc webserver

<h2>ステップ９</h2>
バッチジョブの処理　CronJobの利用（オプショナル）

Scheduleの書式
schedule: "分　時　日　月　曜日"

分： 0-59  時: 0-23  日： 1-31 月： 1-12 曜日: 0=日 1=月 2=火　３＝水 4=木 5=金 6=土
🌾 * の利用


CronJobの実行
$ kubectl create -f config/cronjob.yaml

CronJobの確認
$ kubectl get cronjob

CronJobの実行内容
$ kubectl get jobs --watch

CronJobの削除
$ kubectl delete -f config/cronjob.yaml


<h2>クラスターの削除</h2>

$ gcloud container clusters delete クラスター名




