# Tektonを試行する

Tektonを導入し、あるアプリケーションの
- アプリケーションビルド
- コンテナービルド
- デプロイ
を行うパイプラインを作成し、実行してみる。


## Tektonの導入
導入先のk8sは、MacのDocker　Desktop提供のk8sクラスター

以下のファイルのapplyで導入

`kubectl apply --filename https://storage.googleapis.com/tekton-releases/latest/release.yaml`

```
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
```

以下のpodのステータスがRunningとなっている事で稼働確認を行う。

```
(⎈ |docker-desktop:default)eb82649:tryTekton eb82649@jp.ibm.com$ kubectl get pods --namespace tekton-pipelines
NAME                                           READY   STATUS    RESTARTS   AGE
tekton-pipelines-controller-857f9f4dd9-dbsv9   1/1     Running   0          36s
tekton-pipelines-webhook-844f844f47-sbpzc      1/1     Running   0          36s
```

以降、試行のシナリオごとにディレクトリ分割

1. [hello-world タスクを実行する](https://github.com/0ht/tryTekton/tree/master/01_hello-world)
2. [Mavenビルドタスクを実行してアプリをビルド](https://github.com/0ht/tryTekton/tree/master/02_Maven)
3. Kanikoタスクを実行してコンテナーイメージビルド
4. ２と３をパイプラインで繋いで実行
5. Eventingと合わせて、PRトリガーでパイプラインを起動