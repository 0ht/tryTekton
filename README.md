# Tektonを試行する

Tektonを導入し、あるアプリケーションの
・アプリケーションビルド
・コンテナービルド
・デプロイ
を行うパイプラインを作成し、実行してみる。


Tektonの導入
導入先のk8sは、MacのDocker　Desktop提供のk8s

以下のファイルのapplyで導入

(⎈ |docker-desktop:default)eb82649:tryTekton eb82649@jp.ibm.com$ kubectl apply --filename https://storage.googleapis.com/tekton-releases/latest/release.yaml
namespace/tekton-pipelines created
podsecuritypolicy.policy/tekton-pipelines created
clusterrole.rbac.authorization.k8s.io/tekton-pipelines-admin created
serviceaccount/tekton-pipelines-controller created
clusterrolebinding.rbac.authorization.k8s.io/tekton-pipelines-controller-admin created
customresourcedefinition.apiextensions.k8s.io/clustertasks.tekton.dev created
customresourcedefinition.apiextensions.k8s.io/conditions.tekton.dev created
customresourcedefinition.apiextensions.k8s.io/images.caching.internal.knative.dev created
customresourcedefinition.apiextensions.k8s.io/pipelines.tekton.dev created
customresourcedefinition.apiextensions.k8s.io/pipelineruns.tekton.dev created
customresourcedefinition.apiextensions.k8s.io/pipelineresources.tekton.dev created
customresourcedefinition.apiextensions.k8s.io/tasks.tekton.dev created
customresourcedefinition.apiextensions.k8s.io/taskruns.tekton.dev created
service/tekton-pipelines-controller created
service/tekton-pipelines-webhook created
clusterrole.rbac.authorization.k8s.io/tekton-aggregate-edit created
clusterrole.rbac.authorization.k8s.io/tekton-aggregate-view created
configmap/config-artifact-bucket created
configmap/config-artifact-pvc created
configmap/config-defaults created
configmap/config-logging created
configmap/config-observability created
deployment.apps/tekton-pipelines-controller created
deployment.apps/tekton-pipelines-webhook created

以下のpodのステータスがRunningとなっている事で稼働確認を行う。

(⎈ |docker-desktop:default)eb82649:tryTekton eb82649@jp.ibm.com$ kubectl get pods --namespace tekton-pipelines
NAME                                           READY   STATUS    RESTARTS   AGE
tekton-pipelines-controller-857f9f4dd9-dbsv9   1/1     Running   0          36s
tekton-pipelines-webhook-844f844f47-sbpzc      1/1     Running   0          36s

taskを作成
(⎈ |docker-desktop:default)eb82649:task eb82649@jp.ibm.com$ kubectl apply -f task-echo-hello-world.yaml 
task.tekton.dev/echo-hello-world created

(⎈ |docker-desktop:default)eb82649:task eb82649@jp.ibm.com$ kubectl get task
NAME               AGE
echo-hello-world   2m5s

taskrunを作成（実行）
(⎈ |docker-desktop:default)eb82649:task eb82649@jp.ibm.com$ kubectl apply -f taskrun-echo-hello-world.yaml
taskrun.tekton.dev/echo-hello-world-task-run created

(⎈ |docker-desktop:default)eb82649:task eb82649@jp.ibm.com$ kubectl get taskrun
NAME                        SUCCEEDED   REASON      STARTTIME   COMPLETIONTIME
echo-hello-world-task-run   True        Succeeded   98s         62s

(⎈ |docker-desktop:default)eb82649:task eb82649@jp.ibm.com$ kubectl get pod
NAME                                   READY   STATUS      RESTARTS   AGE
echo-hello-world-task-run-pod-b8c449   0/1     Completed   0          70s

(⎈ |docker-desktop:default)eb82649:task eb82649@jp.ibm.com$ kubectl logs echo-hello-world-task-run-pod-b8c449
hello world

##アプリビルドのタスク

docker run -it --rm --name my-maven-project -v "$(pwd)":/usr/src/mymaven -w /usr/src/mymaven maven:3.3-jdk-8 mvn clean install

-it ターミナルを使用してインタラクティブに実行
-rm 終了時にコンテナーを削除
--name コンテナーに名前をアサイン
-v ボリュームをマウント
-w コンテナー内のワーキングディレクトリ

これをうまくタスク化したい。
ボリュームのマウントが必要なので、やはりPV/PVCの定義は必要か