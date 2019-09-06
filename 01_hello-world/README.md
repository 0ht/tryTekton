
# hello-world タスクを実行してみる

## taskを作成
```
(⎈ |docker-desktop:default)eb82649:task eb82649@jp.ibm.com$ kubectl apply -f task-echo-hello-world.yaml 
task.tekton.dev/echo-hello-world created

(⎈ |docker-desktop:default)eb82649:task eb82649@jp.ibm.com$ kubectl get task
NAME               AGE
echo-hello-world   2m5s
```

## taskrunを作成（実行）
```
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
```

hello worldが表示され、動いたことが確認できた。
taskを作って、taskrunで実行する。
