Controller-Job和Cronjob-一次性任务和定时任务



# Job - 一次性任务

负责批处理任务

Job创建一个或多个Pod，并确保指定数量的Pod成功终止。Pod成功完成后，Job将跟踪成功完成的情况。当达到指定的成功完成次数时，任务（即Job）就完成了。删除Job将清除其创建的Pod。

一个简单的情况是创建一个Job对象，以便可靠地运行一个Pod来完成。如果第一个Pod发生故障或被删除（例如，由于节点硬件故障或节点重启），则Job对象将启动一个新的Pod。

当然还可以使用Job并行运行多个Pod。



```yaml
apiVersion: batch/v1
kind: job
metadata:
  ...............
```

```bash
# 创建一次性任务
kubectl create -f job.yaml

# 查看
kubectl get pods
#执行完成后，状态Status会变为Completed

#删除任务
kubectl delete -f job.yaml
```



## Job终止和清理

Job完成后，不会再创建其他Pod，但是Pod也不会被删除。这样使我们仍然可以查看已完成容器的日志，以检查是否有错误、警告或其他诊断输出。Job对象在完成后也将保留下来，以便您查看其状态。

当我们删除Job对象时，对应的pod也会被删除。

## 特殊说明

- 单个Pod时，默认Pod成功运行后Job即结束
- restartPolicy 仅支持Never和OnFailure
- .spec.completions 标识Job结束所需要成功运行的Pod个数，默认为1
- .spec.parallelism 标识并行运行的Pod个数，默认为1
- .spec.activeDeadlineSeconds 为Job的持续时间，不管有多少Pod创建。一旦工作到指定时间，所有的运行pod都会终止且工作状态将成为type: Failed与reason: DeadlineExceeded。



## Job示例

yaml文件

```yaml
[root@k8s-master controller]# pwd
/root/k8s_practice/controller
[root@k8s-master controller]# cat job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  #completions: 3  # 标识Job结束所需要成功运行的Pod个数，默认为1
  template:
    spec:
      containers:
      - name: pi
        image: registry.cn-beijing.aliyuncs.com/google_registry/perl:5.26
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
```

创建job，与状态查看

```bash
[root@k8s-master controller]# kubectl apply -f job.yaml
job.batch/pi created
[root@k8s-master controller]# kubectl get job -o wide
NAME   COMPLETIONS   DURATION   AGE   CONTAINERS   IMAGES                                                       SELECTOR
pi     0/1           16s        16s   pi           registry.cn-beijing.aliyuncs.com/google_registry/perl:5.26   controller-uid=77004357-fd5e-4395-9bbb-cd0698e19cb9
[root@k8s-master controller]# kubectl get pod -o wide
NAME       READY   STATUS              RESTARTS   AGE   IP       NODE         NOMINATED NODE   READINESS GATES
pi-6zvm5   0/1     ContainerCreating   0          85s   <none>   k8s-node01   <none>           <none>
```

之后再次查看

```bash
[root@k8s-master controller]# kubectl get job -o wide
NAME   COMPLETIONS   DURATION   AGE   CONTAINERS   IMAGES                                                       SELECTOR
pi     1/1           14m        44m   pi           registry.cn-beijing.aliyuncs.com/google_registry/perl:5.26   controller-uid=77004357-fd5e-4395-9bbb-cd0698e19cb9
[root@k8s-master controller]# kubectl get pod -o wide
NAME       READY   STATUS      RESTARTS   AGE   IP            NODE         NOMINATED NODE   READINESS GATES
pi-6zvm5   0/1     Completed   0          44m   10.244.4.63   k8s-node01   <none>           <none>
[root@k8s-master controller]#
[root@k8s-master controller]# kubectl describe job pi
Name:           pi
Namespace:      default
Selector:       controller-uid=76680f6f-442c-4a09-91dc-c3d4c18465b0
Labels:         controller-uid=76680f6f-442c-4a09-91dc-c3d4c18465b0
                job-name=pi
Annotations:    kubectl.kubernetes.io/last-applied-configuration:
                  {"apiVersion":"batch/v1","kind":"Job","metadata":{"annotations":{},"name":"pi","namespace":"default"},"spec":{"backoffLimit":4,"
Parallelism:    1
Completions:    1
Start Time:     Tue, 11 Aug 2020 23:34:44 +0800
Completed At:   Tue, 11 Aug 2020 23:35:02 +0800
Duration:       18s
Pods Statuses:  0 Running / 1 Succeeded / 0 Failed
Pod Template:
  Labels:  controller-uid=76680f6f-442c-4a09-91dc-c3d4c18465b0
           job-name=pi
  Containers:
   pi:
    Image:      registry.cn-beijing.aliyuncs.com/google_registry/perl:5.26
    Port:       <none>
    Host Port:  <none>
    Command:
      perl
      -Mbignum=bpi
      -wle
      print bpi(2000)
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age    From            Message
  ----    ------            ----   ----            -------
  Normal  SuccessfulCreate  2m33s  job-controller  Created pod: pi-6zvm5
```

并查看 Pod 的标准输出

```bash
[root@k8s-master controller]# kubectl logs --tail 500 pi-6zvm5
3.141592653589793238462643383279502884197169399375105820974944592307816406………………
```





# Cronjob - 定时任务

Cron Job 创建是基于时间调度的 Jobs

一个 CronJob 对象就像 crontab (cron table) 文件中的一行。它用 Cron 格式进行编写，并周期性地在给定的调度时间执行 Job。

```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
.................
```



```bash
#创建任务
kubectl applf -y cronjob.yaml

#查看
kubectl get pods
```

## CronJob 限制

CronJob 创建 Job 对象，每个 Job 的执行次数大约为一次。 之所以说 “大约” ，是因为在某些情况下，可能会创建两个 Job，或者不会创建任何 Job。虽然试图使这些情况尽量少发生，但不能完全杜绝。因此，Job 应该是幂等的。

CronJob 仅负责创建与其调度时间相匹配的 Job，而 Job 又负责管理其代表的 Pod。

**使用案例：**

1、在给定时间点调度Job

2、创建周期性运行的Job。如：数据备份、数仓导数、执行任务、邮件发送、数据拉取、数据推送

 

## 特殊说明

.spec.schedule 必选，任务被创建和执行的调度时间。同Cron格式串，例如 0 * * * *。

- .spec.jobTemplate 必选，任务模版。它和 Job的语法完全一样
- .spec.startingDeadlineSeconds 可选的。默认未设置。它表示任务如果由于某种原因错过了调度时间，开始该任务的截止时间的秒数。过了截止时间，CronJob 就不会开始任务。不满足这种最后期限的任务会被统计为失败任务。如果没有该声明，那任务就没有最后期限。
- .spec.concurrencyPolicy 可选的。它声明了 CronJob 创建的任务执行时发生重叠如何处理。spec 仅能声明下列规则中的一种：

```
Allow (默认)：CronJob 允许并发任务执行。
Forbid：CronJob 不允许并发任务执行；如果新任务的执行时间到了而老任务没有执行完，CronJob 会忽略新任务的执行。
Replace：如果新任务的执行时间到了而老任务没有执行完，CronJob 会用新任务替换当前正在运行的任务。
```

请注意，并发性规则仅适用于相同 CronJob 创建的任务。如果有多个 CronJob，它们相应的任务总是允许并发执行的。

- .spec.suspend 可选的。如果设置为 true ，后续发生的执行都会挂起。这个设置对已经开始执行的Job不起作用。默认是关闭的false。
  备注：在调度时间内挂起的执行都会被统计为错过的任务。当 .spec.suspend 从 true 改为 false 时，且没有开始的最后期限，错过的任务会被立即调度。
- .spec.successfulJobsHistoryLimit 和 .spec.failedJobsHistoryLimit 可选的。 这两个声明了有多少执行完成和失败的任务会被保留。默认设置为3和1。限制设置为0代表相应类型的任务完成后不会保留。

说明：如果 startingDeadlineSeconds 设置为很大的数值或未设置（默认），并且 concurrencyPolicy 设置为 Allow，则作业将始终至少运行一次。

## CronJob示例

yaml文件

```bash
[root@k8s-master controller]# pwd
/root/k8s_practice/controller
[root@k8s-master controller]# cat cronjob.yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: registry.cn-beijing.aliyuncs.com/google_registry/busybox:1.24
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

启动cronjob并查看状态

```bash
[root@k8s-master controller]# kubectl apply -f cronjob.yaml
cronjob.batch/hello created
[root@k8s-master controller]# kubectl get cronjob -o wide
NAME    SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE   CONTAINERS   IMAGES                                                          SELECTOR
hello   */1 * * * *   False     1        8s              27s   hello        registry.cn-beijing.aliyuncs.com/google_registry/busybox:1.24   <none>
[root@k8s-master controller]#
[root@k8s-master controller]# kubectl get job -o wide
NAME               COMPLETIONS   DURATION   AGE   CONTAINERS   IMAGES                                                          SELECTOR
hello-1590721020   1/1           2s         21s   hello        registry.cn-beijing.aliyuncs.com/google_registry/busybox:1.24   controller-uid=9e0180e8-8362-4a58-8b93-089b92774b5e
[root@k8s-master controller]#
[root@k8s-master controller]# kubectl get pod -o wide
NAME                     READY   STATUS      RESTARTS   AGE   IP            NODE         NOMINATED NODE   READINESS GATES
hello-1590721020-m4fr8   0/1     Completed   0          36s   10.244.4.66   k8s-node01   <none>           <none>
```

几分钟之后的状态信息

```bash
[root@k8s-master controller]# kubectl get cronjob -o wide
NAME    SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE     CONTAINERS   IMAGES                                                          SELECTOR
hello   */1 * * * *   False     0        55s             7m14s   hello        registry.cn-beijing.aliyuncs.com/google_registry/busybox:1.24   <none>
[root@k8s-master controller]#
[root@k8s-master controller]#
[root@k8s-master controller]# kubectl get job -o wide
NAME               COMPLETIONS   DURATION   AGE    CONTAINERS   IMAGES                                                          SELECTOR
hello-1590721260   1/1           1s         3m1s   hello        registry.cn-beijing.aliyuncs.com/google_registry/busybox:1.24   controller-uid=0676bd6d-861b-440b-945b-4b2704872728
hello-1590721320   1/1           2s         2m1s   hello        registry.cn-beijing.aliyuncs.com/google_registry/busybox:1.24   controller-uid=09c1902e-76ef-4731-b3b4-3188961c13e9
hello-1590721380   1/1           2s         61s    hello        registry.cn-beijing.aliyuncs.com/google_registry/busybox:1.24   controller-uid=f30dc159-8905-4cfc-b06b-f950c8dcfc28
[root@k8s-master controller]#
[root@k8s-master controller]# kubectl get pod -o wide
NAME                     READY   STATUS      RESTARTS   AGE    IP            NODE         NOMINATED NODE   READINESS GATES
hello-1590721320-m4pxf   0/1     Completed   0          2m6s   10.244.4.70   k8s-node01   <none>           <none>
hello-1590721380-wk7jh   0/1     Completed   0          66s    10.244.2.77   k8s-node02   <none>           <none>
hello-1590721440-rcx7v   0/1     Completed   0          6s     10.244.4.72   k8s-node01   <none>           <none>
[root@k8s-master controller]#
[root@k8s-master controller]# kubectl describe cronjob hello
Name:                          hello
Namespace:                     default
Labels:                        <none>
Annotations:                   kubectl.kubernetes.io/last-applied-configuration:
                                 {"apiVersion":"batch/v1beta1","kind":"CronJob","metadata":{"annotations":{},"name":"hello","namespace":"default"},"spec":{"jobTemplate":{"...
Schedule:                      */1 * * * *
Concurrency Policy:            Allow
Suspend:                       False
Successful Job History Limit:  3
Failed Job History Limit:      1
Starting Deadline Seconds:     <unset>
Selector:                      <unset>
Parallelism:                   <unset>
Completions:                   <unset>
Pod Template:
  Labels:  <none>
  Containers:
   hello:
    Image:      registry.cn-beijing.aliyuncs.com/google_registry/busybox:1.24
    Port:       <none>
    Host Port:  <none>
    Args:
      /bin/sh
      -c
      date; echo Hello from the Kubernetes cluster
    Environment:     <none>
    Mounts:          <none>
  Volumes:           <none>
Last Schedule Time:  Wed, 12 Aug 2020 00:01:00 +0800
Active Jobs:         <none>
Events:
  Type    Reason            Age                  From                Message
  ----    ------            ----                 ----                -------
  Normal  SuccessfulCreate  19m                  cronjob-controller  Created job hello-1597160520
  Normal  SawCompletedJob   19m                  cronjob-controller  Saw completed job: hello-1597160520, status: Complete
  Normal  SuccessfulCreate  18m                  cronjob-controller  Created job hello-1597160580
  Normal  SawCompletedJob   18m                  cronjob-controller  Saw completed job: hello-1597160580, status: Complete
  Normal  SuccessfulCreate  17m                  cronjob-controller  Created job hello-1597160640
  Normal  SawCompletedJob   17m                  cronjob-controller  Saw completed job: hello-1597160640, status: Complete
  Normal  SuccessfulCreate  16m                  cronjob-controller  Created job hello-1597160700
  Normal  SuccessfulDelete  16m                  cronjob-controller  Deleted job hello-1597160520
  Normal  SawCompletedJob   16m                  cronjob-controller  Saw completed job: hello-1597160700, status: Complete
  Normal  SuccessfulCreate  15m                  cronjob-controller  Created job hello-1597160760
  Normal  SawCompletedJob   15m                  cronjob-controller  Saw completed job: hello-1597160760, status: Complete
  Normal  SuccessfulDelete  15m                  cronjob-controller  Deleted job hello-1597160580
  Normal  SuccessfulCreate  14m                  cronjob-controller  Created job hello-1597160820
  Normal  SuccessfulDelete  14m                  cronjob-controller  Deleted job hello-1597160640
  Normal  SawCompletedJob   14m                  cronjob-controller  Saw completed job: hello-1597160820, status: Complete
  Normal  SuccessfulCreate  13m                  cronjob-controller  Created job hello-1597160880
  Normal  SawCompletedJob   13m                  cronjob-controller  Saw completed job: hello-1597160880, status: Complete
………………
  Normal  SawCompletedJob   11m                  cronjob-controller  Saw completed job: hello-1597161000, status: Complete
  Normal  SuccessfulDelete  11m                  cronjob-controller  Deleted job hello-1597160820
  Normal  SawCompletedJob   10m                  cronjob-controller  (combined from similar events): Saw completed job: hello-1597161060, status: Complete
  Normal  SuccessfulCreate  4m13s (x7 over 10m)  cronjob-controller  (combined from similar events): Created job hello-1597161420
```

找到最后一次调度任务创建的 Pod， 并查看 Pod 的标准输出。请注意任务名称和 Pod 名称是不同的。

```bash
[root@k8s-master controller]#  kubectl logs pod/hello-1590721740-rcx7v   # 或者 kubectl logs hello-1590721740-rcx7v
Fri May 29 03:09:04 UTC 2020
Hello from the Kubernetes cluster
```

删除 CronJob

```bash
[root@k8s-master controller]# kubectl get cronjob
NAME    SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hello   */1 * * * *   False     0        32s             19m
[root@k8s-master controller]#
[root@k8s-master controller]# kubectl delete cronjob hello  # 或者 kubectl delete -f cronjob.yaml
cronjob.batch "hello" deleted
[root@k8s-master controller]# kubectl get cronjob   # 可见已删除
No resources found in default namespace.
```

