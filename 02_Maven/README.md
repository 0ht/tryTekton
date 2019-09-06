
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
gitリポジトリ（ [https://github.com/0ht/remoteCallService]）をInputにするために、以下を作成
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
      - name: git #
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

## taskrunを作成（実行）
```yml:taskrun-maven-build.xml
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

k get taskrun maven-build-task-run -o yaml

```

