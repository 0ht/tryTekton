
# Mavenビルドタスクを実行してアプリをビルド


## アプリビルドのタスクを作ってみる

このイメージを使用する。
https://hub.docker.com/_/maven


ここで紹介されている以下のコマンド、これをうまくタスク化したい。
```
docker run -it --rm --name my-maven-project -v "$(pwd)":/usr/src/mymaven -w /usr/src/mymaven maven:3.3-jdk-8 mvn clean install
```
オプション
- -it ターミナルを使用してインタラクティブに実行
- -rm 終了時にコンテナーを削除
- --name コンテナーに名前をアサイン
- -v ボリュームをマウント
- -w コンテナー内のワーキングディレクトリ

## リソースを用意
gitリポジトリ（ [https://github.com/0ht/remoteCallService] ）をInputにするために、以下を作成
```
apiVersion: tekton.dev/v1alpha1
kind: PipelineResource
metadata:
  name: git
spec:
  type: git
  params: 
  - name: url
    value: https://github.com/0ht/remoteCallService
  - name: revision
    value: master
```
ここ重要。きちんと理解する。

https://github.com/tektoncd/pipeline/blob/master/docs/resources.md#controlling-where-resources-are-mounted

>The optional field `targetPath` can be used to initialize a resource in specific directory. If `targetPath` is set then resource will be initialized under `/workspace/targetPath`. If `targetPath` is not specified then resource will be initialized under `/workspace`. 

## taskを作成

```
apiVersion: tekton.dev/v1alpha1
kind: Task
metadata:
  name: maven-build-task
spec:
  inputs:
    resources:
      - name: git 
        type: git
        targetPath: /gitrepo #/workspaceからの相対パス.
  outputs:
  steps:
  - name: maven-build
    image: maven:3.3-jdk-8
    command: ["mvn"]
    args: ["install"]
    workingDir: "/workspace/gitrepo" #上のtagetPathと整合させる

```

## taskrunを作成

```yaml:taskrun-maven-build.xml
apiVersion: tekton.dev/v1alpha1
kind: TaskRun
metadata:
  name: maven-build-task-run
spec:
  taskRef:
    name: maven-build-task
  inputs:
    resources:
      - name: git
        resourceRef:
          name: git
```

## 実行する
```
(⎈ |docker-desktop:default)eb49:tryTekton eb82649@jp.ibm.com$ k apply -f ./02_Maven/
pipelineresource.tekton.dev/git created
task.tekton.dev/maven-build-task created
taskrun.tekton.dev/maven-build-task-run created
```

## 確認する
```
(⎈ |docker-desktop:default)eb82649:tryTekton eb82649@jp.ibm.com$ k get taskrun maven-build-task-run -o  yaml
apiVersion: tekton.dev/v1alpha1
kind: TaskRun
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: 
    〜中略〜
  creationTimestamp: "2019-09-06T04:35:57Z"
  generation: 1
  labels:
    tekton.dev/task: maven-build-task
  name: maven-build-task-run
  namespace: default
  resourceVersion: "693608"
  selfLink: /apis/tekton.dev/v1alpha1/namespaces/default/taskruns/maven-build-task-run
  uid: d1beb025-d05f-11e9-974a-025000000001
spec:
  inputs:
    resources:
    - name: git
      resourceRef:
        name: git
  outputs: {}
  podTemplate: {}
  taskRef:
    kind: Task
    name: maven-build-task
  timeout: 1h0m0s
status:
  conditions:
  - lastTransitionTime: "2019-09-06T04:36:03Z"
    message: Not all Steps in the Task have finished executing
    reason: Building
    status: Unknown
    type: Succeeded
  podName: maven-build-task-run-pod-632565
  startTime: "2019-09-06T04:35:57Z"
  steps:
  - container: step-maven-build
    imageID: docker-pullable://maven@sha256:18e8bd367c73c93e29d62571ee235e106b18bf6718aeb235c7a07840328bba71
    name: maven-build
    running:
      startedAt: "2019-09-06T04:36:02Z"
  - container: step-git-source-git-49m8j
    imageID: docker-pullable://gcr.io/tekton-releases/github.com/tektoncd/pipeline/cmd/git-init@sha256:2aaaecd06986c7705f68f19435b8a913ef6701ac6b961df16d1535f45503cea5
    name: git-source-git-49m8j
    running:
      startedAt: "2019-09-06T04:36:02Z"
```
gitリポジトリのCloneが別コンテナーで動いている模様。
```
(⎈ |docker-desktop:default)eb82649:tryTekton eb82649@jp.ibm.com$ k logs maven-build-task-run-pod-632565 -c step-git-source-git-49m8j
{"level":"warn","ts":1567744564.2113607,"logger":"fallback-logger","caller":"logging/config.go:69","msg":"Fetch GitHub commit ID from kodata failed: \"ref: refs/heads/master\" is not a valid GitHub commit ID"}
{"level":"info","ts":1567744573.6671033,"logger":"fallback-logger","caller":"git/git.go:102","msg":"Successfully cloned https://github.com/0ht/remoteCallService @ master in path /workspace/gitrepo"}
```

ビルド自体はどうか？
```
(⎈ |docker-desktop:default)eb82649:tryTekton eb82649@jp.ibm.com$ k logs -f maven-build-task-run-pod-632565 -c step-maven-build
[INFO] Scanning for projects...
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building transfer 1.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
:
Downloaded: https://repo.maven.apache.org/maven2/org/codehaus/plexus/plexus-utils/3.0.5/plexus-utils-3.0.5.jar (226 KB at 290.0 KB/sec)
[INFO] Installing ./WAR/transfer.war to /root/.m2/repository/com/oht/sample/transfer/1.0-SNAPSHOT/transfer-1.0-SNAPSHOT.war
[INFO] Installing /workspace/gitrepo/pom.xml to /root/.m2/repository/com/oht/sample/transfer/1.0-SNAPSHOT/transfer-1.0-SNAPSHOT.pom
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 01:55 min
[INFO] Finished at: 2019-09-06T04:38:15+00:00
[INFO] Final Memory: 23M/156M
[INFO] ------------------------------------------------------------------------
```
ビルドOK。
次は、作成されたアーチファクトをどの様に次のタスクに渡すかが課題。