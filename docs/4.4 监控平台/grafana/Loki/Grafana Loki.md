- [企业级日志平台新秀Grafana Loki](https://mp.weixin.qq.com/s/Ow22ogcQ5KWnNsqdwTCRoQ)

最近，在对公司容器云的日志方案进行设计的时候，发现主流的ELK或者EFK比较重，再加上现阶段对于ES复杂的搜索功能很多都用不上最终选择了Grafana开源的Loki日志系统，下面介绍下Loki的背景。

# 背景和动机

当我们的容器云运行的应用或者某个节点出现问题了，解决思路应该如下：



![图片](https://mmbiz.qpic.cn/mmbiz_png/A1HKVXsfHNnMGibJb5rw6xlYnV0PlQicRa0OXawS5NwnSKtZpjMRhBIqicmIMJDFDicmxGT0Mq3ZKDXaAZ4icPzX3nQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



我们的监控使用的是基于Prometheus体系进行改造的，Prometheus中比较重要的是Metric和Alert，Metric是来说明当前或者历史达到了某个值，Alert设置Metric达到某个特定的基数触发了告警，但是这些信息明显是不够的。我们都知道，Kubernetes的基本单位是Pod，Pod把日志输出到stdout和stderr，平时有什么问题我们通常在界面或者通过命令查看相关的日志，举个例子：当我们的某个Pod的内存变得很大，触发了我们的Alert，这个时候管理员，去页面查询确认是哪个Pod有问题，然后要确认Pod内存变大的原因，我们还需要去查询Pod的日志，如果没有日志系统，那么我们就需要到页面或者使用命令进行查询了：

![图片](https://mmbiz.qpic.cn/mmbiz_png/A1HKVXsfHNnMGibJb5rw6xlYnV0PlQicRa6wNDnuw6Vst5Q5UtOruV0SWfVJUtGH2IyibzPc5o80AyhvKNibJ20cyQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如果，这个时候应用突然挂了，这个时候我们就无法查到相关的日志了，所以需要引入日志系统，统一收集日志，而使用ELK的话，就需要在Kibana和Grafana之间切换，影响用户体验。所以 ，loki的第一目的就是最小化度量和日志的切换成本，有助于减少异常事件的响应时间和提高用户的体验。

# ELK存在的问题

现有的很多日志采集的方案都是采用全文检索对日志进行索引（如ELK方案），优点是功能丰富，允许复杂的操作。但是，这些方案往往规模复杂，资源占用高，操作苦难。很多功能往往用不上，大多数查询只关注一定时间范围和一些简单的参数（如host、service等），使用这些解决方案就有点杀鸡用牛刀的感觉了。

因此，Loki的第二个目的是，在查询语言的易操作性和复杂性之间可以达到一个权衡。

## 成本

全文检索的方案也带来成本问题，简单的说就是全文搜索（如ES）的倒排索引的切分和共享的成本较高。后来出现了其他不同的设计方案如：OKlog，采用最终一致的、基于网格的分布策略。这两个设计决策提供了大量的成本降低和非常简单的操作，但是查询不够方便。因此，Loki的第三个目的是，提高一个更具成本效益的解决方案。

# 架构

## 整体架构

Loki的架构如下：



![图片](https://mmbiz.qpic.cn/mmbiz_png/A1HKVXsfHNnMGibJb5rw6xlYnV0PlQicRaAtRuLgdYX6JjdsGiagKS0eAYZNPiadiceygNM7HnjN5eazqTEulPmia3gA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



不难看出，Loki的架构非常简单，使用了和Prometheus一样的标签来作为索引，也就是说，你通过这些标签既可以查询日志的内容也可以查询到监控的数据，不但减少了两种查询之间的切换成本，也极大地降低了日志索引的存储。Loki将使用与Prometheus相同的服务发现和标签重新标记库，编写了pormtail，在Kubernetes中promtail以DaemonSet方式运行在每个节点中，通过Kubernetes API等到日志的正确元数据，并将它们发送到Loki。下面是日志的存储架构：



![图片](https://mmbiz.qpic.cn/mmbiz_png/A1HKVXsfHNnMGibJb5rw6xlYnV0PlQicRasNc0rwnnB8tPhDDdJ3ibyxn5ZalEtjgibHq1PBFjLLQeuqvaHaJsLsuw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



## 读写

日志数据的写主要依托的是Distributor和Ingester两个组件，整体的流程如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/A1HKVXsfHNnMGibJb5rw6xlYnV0PlQicRabiazmXBCu3bPGIWRTdDEUs6Ng4gNDqVibCMFRE27ibCeTyrZxnheHovhA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## Distributor

一旦promtail收集日志并将其发送给loki，Distributor就是第一个接收日志的组件。由于日志的写入量可能很大，所以不能在它们传入时将它们写入数据库。这会毁掉数据库。我们需要批处理和压缩数据。

Loki通过构建压缩数据块来实现这一点，方法是在日志进入时对其进行gzip操作，组件ingester是一个有状态的组件，负责构建和刷新chunck，当chunk达到一定的数量或者时间后，刷新到存储中去。每个流的日志对应一个ingester，当日志到达Distributor后，根据元数据和hash算法计算出应该到哪个ingester上面。



![图片](https://mmbiz.qpic.cn/mmbiz_png/A1HKVXsfHNnMGibJb5rw6xlYnV0PlQicRasoV8JBbJCmsuAQDZMYuH4qCNJylr2ibFNWTWS2vHjf43rzzKoNQBB3Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



此外，为了冗余和弹性，我们将其复制n（默认情况下为3）次。

## Ingester

Ingester接收到日志并开始构建chunk：



![图片](https://mmbiz.qpic.cn/mmbiz_png/A1HKVXsfHNnMGibJb5rw6xlYnV0PlQicRaAeqWDTm8rPauHLg1kJyCH2dPqR5CfRl1aRB0axk9n7Ryia0EOl6yiaIQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



基本上就是将日志进行压缩并附加到chunk上面。一旦chunk“填满”（数据达到一定数量或者过了一定期限），ingester将其刷新到数据库。我们对块和索引使用单独的数据库，因为它们存储的数据类型不同。

![图片](https://mmbiz.qpic.cn/mmbiz_png/A1HKVXsfHNnMGibJb5rw6xlYnV0PlQicRaTjCiby0eppich4KXgXXLic4UT55KpH0zVfhuyJMRSyvKQicw9uk6WY9J8w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

刷新一个chunk之后，ingester然后创建一个新的空chunk并将新条目添加到该chunk中。

## Querier

读取就非常简单了，由Querier负责给定一个时间范围和标签选择器，Querier查看索引以确定哪些块匹配，并通过greps将结果显示出来。它还从Ingester获取尚未刷新的最新数据。

对于每个查询，一个查询器将为您显示所有相关日志。实现了查询并行化，提供分布式grep，使即使是大型查询也是足够的。

![图片](https://mmbiz.qpic.cn/mmbiz_png/A1HKVXsfHNnMGibJb5rw6xlYnV0PlQicRaz6vrBkEGvHspZfWtLfXtMm5bpFBg4GfhklQicORQgkGUHHwg5tb9ZNw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 可扩展性

Loki的索引存储可以是cassandra/bigtable/dynamodb，而chuncks可以是各种对象存储，Querier和Distributor都是无状态的组件。对于ingester他虽然是有状态的但是，当新的节点加入或者减少，整节点间的chunk会重新分配，已适应新的散列环。而Loki底层存储的实现Cortex已经 在实际的生产中投入使用多年了。有了这句话，我可以放心的在环境中实验一把了。

# 部署

Loki的安装非常简单。

## 创建namespace

```
oc new-project loki
```

## 权限设置

```
oc adm policy add-scc-to-user anyuid -z default -n loki
oc adm policy add-cluster-role-to-user cluster-admin system:serviceaccount:loki:default
```

## 安装Loki

安装命令：

```
oc create -f statefulset.json -n loki
```

statefulset.json如下：

```json
{
    "apiVersion": "apps/v1",
    "kind": "StatefulSet",
    "metadata": {
        "name": "loki"
    },
    "spec": {
        "podManagementPolicy": "OrderedReady",
        "replicas": 1,
        "revisionHistoryLimit": 10,
        "selector": {
            "matchLabels": {
                "app": "loki"
            }
        },
        "serviceName": "womping-stoat-loki-headless",
        "template": {
            "metadata": {
                "annotations": {
                    "checksum/config": "da297d66ee53e0ce68b58e12be7ec5df4a91538c0b476cfe0ed79666343df72b",
                    "prometheus.io/port": "http-metrics",
                    "prometheus.io/scrape": "true"
                },
                "creationTimestamp": null,
                "labels": {
                    "app": "loki",
                    "name": "loki"
                }
            },
            "spec": {
                "affinity": {},
                "containers": [
                    {
                        "args": [
                            "-config.file=/etc/loki/local-config.yaml"
                        ],
                        "image": "grafana/loki:latest",
                        "imagePullPolicy": "IfNotPresent",
                        "livenessProbe": {
                            "failureThreshold": 3,
                            "httpGet": {
                                "path": "/ready",
                                "port": "http-metrics",
                                "scheme": "HTTP"
                            },
                            "initialDelaySeconds": 45,
                            "periodSeconds": 10,
                            "successThreshold": 1,
                            "timeoutSeconds": 1
                        },
                        "name": "loki",
                        "ports": [
                            {
                                "containerPort": 3100,
                                "name": "http-metrics",
                                "protocol": "TCP"
                            }
                        ],
                        "readinessProbe": {
                            "failureThreshold": 3,
                            "httpGet": {
                                "path": "/ready",
                                "port": "http-metrics",
                                "scheme": "HTTP"
                            },
                            "initialDelaySeconds": 45,
                            "periodSeconds": 10,
                            "successThreshold": 1,
                            "timeoutSeconds": 1
                        },
                        "resources": {},
                        "terminationMessagePath": "/dev/termination-log",
                        "terminationMessagePolicy": "File",
                        "volumeMounts": [
                            {
                                "mountPath": "/tmp/loki",
                                "name": "storage"
                            }
                        ]
                    }
                ],
                "dnsPolicy": "ClusterFirst",
                "restartPolicy": "Always",
                "schedulerName": "default-scheduler",
                "terminationGracePeriodSeconds": 30,
                "volumes": [
                    {
                        "emptyDir": {},
                        "name": "storage"
                    }
                ]
            }
        },
        "updateStrategy": {
            "type": "RollingUpdate"
        }
    }
}
```

## 安装Promtail

安装命令：

```
oc create -f configmap.json -n loki
```

configmap.json如下：

```json
{
    "apiVersion": "v1",
    "data": {
        "promtail.yaml": "client:\n  backoff_config:\n    maxbackoff: 5s\n    maxretries: 5\n    minbackoff: 100ms\n  batchsize: 102400\n  batchwait: 1s\n  external_labels: {}\n  timeout: 10s\npositions:\n  filename: /run/promtail/positions.yaml\nserver:\n  http_listen_port: 3101\ntarget_config:\n  sync_period: 10s\n\nscrape_configs:\n- job_name: kubernetes-pods-name\n  pipeline_stages:\n    - docker: {}\n    \n  kubernetes_sd_configs:\n  - role: pod\n  relabel_configs:\n  - source_labels:\n    - __meta_kubernetes_pod_label_name\n    target_label: __service__\n  - source_labels:\n    - __meta_kubernetes_pod_node_name\n    target_label: __host__\n  - action: drop\n    regex: ^$\n    source_labels:\n    - __service__\n  - action: labelmap\n    regex: __meta_kubernetes_pod_label_(.+)\n  - action: replace\n    replacement: $1\n    separator: /\n    source_labels:\n    - __meta_kubernetes_namespace\n    - __service__\n    target_label: job\n  - action: replace\n    source_labels:\n    - __meta_kubernetes_namespace\n    target_label: namespace\n  - action: replace\n    source_labels:\n    - __meta_kubernetes_pod_name\n    target_label: instance\n  - action: replace\n    source_labels:\n    - __meta_kubernetes_pod_container_name\n    target_label: container_name\n  - replacement: /var/log/pods/*$1/*.log\n    separator: /\n    source_labels:\n    - __meta_kubernetes_pod_uid\n    - __meta_kubernetes_pod_container_name\n    target_label: __path__\n- job_name: kubernetes-pods-app\n  pipeline_stages:\n    - docker: {}\n    \n  kubernetes_sd_configs:\n  - role: pod\n  relabel_configs:\n  - action: drop\n    regex: .+\n    source_labels:\n    - __meta_kubernetes_pod_label_name\n  - source_labels:\n    - __meta_kubernetes_pod_label_app\n    target_label: __service__\n  - source_labels:\n    - __meta_kubernetes_pod_node_name\n    target_label: __host__\n  - action: drop\n    regex: ^$\n    source_labels:\n    - __service__\n  - action: labelmap\n    regex: __meta_kubernetes_pod_label_(.+)\n  - action: replace\n    replacement: $1\n    separator: /\n    source_labels:\n    - __meta_kubernetes_namespace\n    - __service__\n    target_label: job\n  - action: replace\n    source_labels:\n    - __meta_kubernetes_namespace\n    target_label: namespace\n  - action: replace\n    source_labels:\n    - __meta_kubernetes_pod_name\n    target_label: instance\n  - action: replace\n    source_labels:\n    - __meta_kubernetes_pod_container_name\n    target_label: container_name\n  - replacement: /var/log/pods/*$1/*.log\n    separator: /\n    source_labels:\n    - __meta_kubernetes_pod_uid\n    - __meta_kubernetes_pod_container_name\n    target_label: __path__\n- job_name: kubernetes-pods-direct-controllers\n  pipeline_stages:\n    - docker: {}\n    \n  kubernetes_sd_configs:\n  - role: pod\n  relabel_configs:\n  - action: drop\n    regex: .+\n    separator: ''\n    source_labels:\n    - __meta_kubernetes_pod_label_name\n    - __meta_kubernetes_pod_label_app\n  - action: drop\n    regex: ^([0-9a-z-.]+)(-[0-9a-f]{8,10})$\n    source_labels:\n    - __meta_kubernetes_pod_controller_name\n  - source_labels:\n    - __meta_kubernetes_pod_controller_name\n    target_label: __service__\n  - source_labels:\n    - __meta_kubernetes_pod_node_name\n    target_label: __host__\n  - action: drop\n    regex: ^$\n    source_labels:\n    - __service__\n  - action: labelmap\n    regex: __meta_kubernetes_pod_label_(.+)\n  - action: replace\n    replacement: $1\n    separator: /\n    source_labels:\n    - __meta_kubernetes_namespace\n    - __service__\n    target_label: job\n  - action: replace\n    source_labels:\n    - __meta_kubernetes_namespace\n    target_label: namespace\n  - action: replace\n    source_labels:\n    - __meta_kubernetes_pod_name\n    target_label: instance\n  - action: replace\n    source_labels:\n    - __meta_kubernetes_pod_container_name\n    target_label: container_name\n  - replacement: /var/log/pods/*$1/*.log\n    separator: /\n    source_labels:\n    - __meta_kubernetes_pod_uid\n    - __meta_kubernetes_pod_container_name\n    target_label: __path__\n- job_name: kubernetes-pods-indirect-controller\n  pipeline_stages:\n    - docker: {}\n    \n  kubernetes_sd_configs:\n  - role: pod\n  relabel_configs:\n  - action: drop\n    regex: .+\n    separator: ''\n    source_labels:\n    - __meta_kubernetes_pod_label_name\n    - __meta_kubernetes_pod_label_app\n  - action: keep\n    regex: ^([0-9a-z-.]+)(-[0-9a-f]{8,10})$\n    source_labels:\n    - __meta_kubernetes_pod_controller_name\n  - action: replace\n    regex: ^([0-9a-z-.]+)(-[0-9a-f]{8,10})$\n    source_labels:\n    - __meta_kubernetes_pod_controller_name\n    target_label: __service__\n  - source_labels:\n    - __meta_kubernetes_pod_node_name\n    target_label: __host__\n  - action: drop\n    regex: ^$\n    source_labels:\n    - __service__\n  - action: labelmap\n    regex: __meta_kubernetes_pod_label_(.+)\n  - action: replace\n    replacement: $1\n    separator: /\n    source_labels:\n    - __meta_kubernetes_namespace\n    - __service__\n    target_label: job\n  - action: replace\n    source_labels:\n    - __meta_kubernetes_namespace\n    target_label: namespace\n  - action: replace\n    source_labels:\n    - __meta_kubernetes_pod_name\n    target_label: instance\n  - action: replace\n    source_labels:\n    - __meta_kubernetes_pod_container_name\n    target_label: container_name\n  - replacement: /var/log/pods/*$1/*.log\n    separator: /\n    source_labels:\n    - __meta_kubernetes_pod_uid\n    - __meta_kubernetes_pod_container_name\n    target_label: __path__\n- job_name: kubernetes-pods-static\n  pipeline_stages:\n    - docker: {}\n    \n  kubernetes_sd_configs:\n  - role: pod\n  relabel_configs:\n  - action: drop\n    regex: ^$\n    source_labels:\n    - __meta_kubernetes_pod_annotation_kubernetes_io_config_mirror\n  - action: replace\n    source_labels:\n    - __meta_kubernetes_pod_label_component\n    target_label: __service__\n  - source_labels:\n    - __meta_kubernetes_pod_node_name\n    target_label: __host__\n  - action: drop\n    regex: ^$\n    source_labels:\n    - __service__\n  - action: labelmap\n    regex: __meta_kubernetes_pod_label_(.+)\n  - action: replace\n    replacement: $1\n    separator: /\n    source_labels:\n    - __meta_kubernetes_namespace\n    - __service__\n    target_label: job\n  - action: replace\n    source_labels:\n    - __meta_kubernetes_namespace\n    target_label: namespace\n  - action: replace\n    source_labels:\n    - __meta_kubernetes_pod_name\n    target_label: instance\n  - action: replace\n    source_labels:\n    - __meta_kubernetes_pod_container_name\n    target_label: container_name\n  - replacement: /var/log/pods/*$1/*.log\n    separator: /\n    source_labels:\n    - __meta_kubernetes_pod_annotation_kubernetes_io_config_mirror\n    - __meta_kubernetes_pod_container_name\n    target_label: __path__\n"
    },
    "kind": "ConfigMap",
    "metadata": {
        "creationTimestamp": "2019-09-05T01:05:03Z",
        "labels": {
            "app": "promtail",
            "chart": "promtail-0.12.0",
            "heritage": "Tiller",
            "release": "lame-zorse"
        },
        "name": "lame-zorse-promtail",
        "namespace": "loki",
        "resourceVersion": "17921611",
        "selfLink": "/api/v1/namespaces/loki/configmaps/lame-zorse-promtail",
        "uid": "30fcb896-cf79-11e9-b58e-e4a8b6cc47d2"
    }
}




oc create -f daemonset.json -n loki
```

daemonset.json如下：

```json
{
    "apiVersion": "apps/v1",
    "kind": "DaemonSet",
    "metadata": {
        "annotations": {
            "deployment.kubernetes.io/revision": "2"
        },
        "creationTimestamp": "2019-09-05T01:16:37Z",
        "generation": 2,
        "labels": {
            "app": "promtail",
            "chart": "promtail-0.12.0",
            "heritage": "Tiller",
            "release": "lame-zorse"
        },
        "name": "lame-zorse-promtail",
        "namespace": "loki"
    },
    "spec": {
        "progressDeadlineSeconds": 600,
        "replicas": 1,
        "revisionHistoryLimit": 10,
        "selector": {
            "matchLabels": {
                "app": "promtail",
                "release": "lame-zorse"
            }
        },
        "strategy": {
            "rollingUpdate": {
                "maxSurge": 1,
                "maxUnavailable": 1
            },
            "type": "RollingUpdate"
        },
        "template": {
            "metadata": {
                "annotations": {
                    "checksum/config": "75a25ee4f2869f54d394bf879549a9c89c343981a648f8d878f69bad65dba809",
                    "prometheus.io/port": "http-metrics",
                    "prometheus.io/scrape": "true"
                },
                "creationTimestamp": null,
                "labels": {
                    "app": "promtail",
                    "release": "lame-zorse"
                }
            },
            "spec": {
                "affinity": {},
                "containers": [
                    {
                        "args": [
                            "-config.file=/etc/promtail/promtail.yaml",
                            "-client.url=http://loki.loki.svc:3100/api/prom/push"
                        ],
                        "env": [
                            {
                                "name": "HOSTNAME",
                                "valueFrom": {
                                    "fieldRef": {
                                        "apiVersion": "v1",
                                        "fieldPath": "spec.nodeName"
                                    }
                                }
                            }
                        ],
                        "image": "grafana/promtail:v0.3.0",
                        "imagePullPolicy": "IfNotPresent",
                        "name": "promtail",
                        "ports": [
                            {
                                "containerPort": 3101,
                                "name": "http-metrics",
                                "protocol": "TCP"
                            }
                        ],
                        "readinessProbe": {
                            "failureThreshold": 5,
                            "httpGet": {
                                "path": "/ready",
                                "port": "http-metrics",
                                "scheme": "HTTP"
                            },
                            "initialDelaySeconds": 10,
                            "periodSeconds": 10,
                            "successThreshold": 1,
                            "timeoutSeconds": 1
                        },
                        "resources": {},
                        "securityContext": {
                            "readOnlyRootFilesystem": true,
                            "runAsUser": 0
                        },
                        "terminationMessagePath": "/dev/termination-log",
                        "terminationMessagePolicy": "File",
                        "volumeMounts": [
                            {
                                "mountPath": "/etc/promtail",
                                "name": "config"
                            },
                            {
                                "mountPath": "/run/promtail",
                                "name": "run"
                            },
                            {
                                "mountPath": "/var/lib/docker/containers",
                                "name": "docker",
                                "readOnly": true
                            },
                            {
                                "mountPath": "/var/log/pods",
                                "name": "pods",
                                "readOnly": true
                            }
                        ]
                    }
                ],
                "dnsPolicy": "ClusterFirst",
                "restartPolicy": "Always",
                "schedulerName": "default-scheduler",
                "securityContext": {},
                "terminationGracePeriodSeconds": 30,
                "volumes": [
                    {
                        "configMap": {
                            "defaultMode": 420,
                            "name": "lame-zorse-promtail"
                        },
                        "name": "config"
                    },
                    {
                        "hostPath": {
                            "path": "/run/promtail",
                            "type": ""
                        },
                        "name": "run"
                    },
                    {
                        "hostPath": {
                            "path": "/var/lib/docker/containers",
                            "type": ""
                        },
                        "name": "docker"
                    },
                    {
                        "hostPath": {
                            "path": "/var/log/pods",
                            "type": ""
                        },
                        "name": "pods"
                    }
                ]
            }
        }
    }
}
```

## 安装服务

```
oc create -f service.json -n loki
```

service.json的内容如下：

```json
{
    "apiVersion": "v1",
    "kind": "Service",
    "metadata": {
        "creationTimestamp": "2019-09-04T09:37:49Z",
        "name": "loki",
        "namespace": "loki",
        "resourceVersion": "17800188",
        "selfLink": "/api/v1/namespaces/loki/services/loki",
        "uid": "a87fe237-cef7-11e9-b58e-e4a8b6cc47d2"
    },
    "spec": {
        "externalTrafficPolicy": "Cluster",
        "ports": [
            {
                "name": "lokiport",
                "port": 3100,
                "protocol": "TCP",
                "targetPort": 3100
            }
        ],
        "selector": {
            "app": "loki"
        },
        "sessionAffinity": "None",
        "type": "NodePort"
    },
    "status": {
        "loadBalancer": {}
    }
```



# 语法

Loki提供了HTTP接口，我们这里就不详解了，大家可以看：https://github.com/grafana/loki/blob/master/docs/api.md

我们这里说下查询的接口如何使用。

第一步，获取当前Loki的元数据类型：

```json
curl http://192.168.25.30:30972/api/prom/label
{
 "values": ["alertmanager", "app", "component", "container_name", "controller_revision_hash", "deployment", "deploymentconfig", "docker_registry", "draft", "filename", "instance", "job", "logging_infra", "metrics_infra", "name", "namespace", "openshift_io_component", "pod_template_generation", "pod_template_hash", "project", "projectname", "prometheus", "provider", "release", "router", "servicename", "statefulset_kubernetes_io_pod_name", "stream", "tekton_dev_pipeline", "tekton_dev_pipelineRun", "tekton_dev_pipelineTask", "tekton_dev_task", "tekton_dev_taskRun", "type", "webconsole"]
}
```

第二步，获取某个元数据类型的值：

```json
curl http://192.168.25.30:30972/api/prom/label/namespace/values
{"values":["cicd","default","gitlab","grafanaserver","jenkins","jx-staging","kube-system","loki","mysql-exporter","new2","openshift-console","openshift-infra","openshift-logging","openshift-monitoring","openshift-node","openshift-sdn","openshift-web-console","tekton-pipelines","test111"]}
```

第三步，根据label进行查询，例如：

```bash
http://192.168.25.30:30972/api/prom/query?direction=BACKWARD&limit=1000&regexp=&query={namespace="cicd"}&start=1567644457221000000&end=1567730857221000000&refId=A
```

参数解析：

- query：一种查询语法详细见下面章节，{name=~“mysql.+”} or {namespace=“cicd”} |= "error"表示查询，namespace为CI/CD的日志中，有error字样的信息。
- limit：返回日志的数量
- start：开始时间，Unix时间表示方法 默认为，一小时前时间
- end：结束时间，默认为当前时间
- direction：forward或者backward，指定limit时候有用，默认为 backward
- regexp：对结果进行regex过滤

# LogQL语法

## 选择器

对于查询表达式的标签部分，将放在{}中，多个标签表达式用逗号分隔：

```
{app="mysql",name="mysql-backup"}
```

支持的符号有：

- =：完全相同。
- !=：不平等。
- =~：正则表达式匹配。
- !~：不要正则表达式匹配。

## 过滤表达式

编写日志流选择器后，您可以通过编写搜索表达式进一步过滤结果。搜索表达式可以文本或正则表达式。

如：

- {job=“mysql”} |= “error”
- {name=“kafka”} |~ “tsdb-ops.*io:2003”
- {instance=~“kafka-[23]”,name=“kafka”} != kafka.server:type=ReplicaManager

支持多个过滤：

- {job=“mysql”} |= “error” != “timeout”

目前支持的操作符：

- |= line包含字符串。
- != line不包含字符串。
- |~ line匹配正则表达式。
- !~ line与正则表达式不匹配。

表达式遵循https://github.com/google/re2/wiki/Syntax语法。