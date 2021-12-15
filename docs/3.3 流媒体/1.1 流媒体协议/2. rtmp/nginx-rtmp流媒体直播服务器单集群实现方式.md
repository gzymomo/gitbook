[分布式直播系统（四）【nginx-rtmp流媒体直播服务器单集群实现方式】](https://blog.csdn.net/impingo/article/details/100379853)



# 关键词

- 调度服务器：调度服务器是一个http接口服务器，它用来记录一路直播流在哪台服务器上发布，并且提供http接口用来查询流对应的服务器地址。
- 回源：假设当前有两台服务器A和B，服务器A上存在某条流S，此时有播放端向服务器B请求流S，由于服务器B上不存在该条流所以服务器B只有向服务器A请求此条流，这种情况成为回源。
- 动态回源：原理与上述“回源”描述一样，只不过回源地址可以通过http响应告诉拉流服务器（上述中的服务器B），这样就可以动态地切换目标源地址。
  ![动态回源](https://img-blog.csdnimg.cn/20190902221712347.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ltcGluZ28=,size_16,color_FFFFFF,t_70)
- 静态回源：与动态回源相对应的一种回源方式，与动态回源不同的是回源地址一般通过服务器配置文件决定，也就是说一旦配置文件固定则目标源地址也就确定，不能动态切换源。
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190902220430357.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ltcGluZ28=,size_16,color_FFFFFF,t_70)
- 转推：假设当前有两台服务器A和B，某主播向服务器A推送流S，服务器A收到流S后立即向服务器B推送该条流，则这种情况被称为转推。
- 动态转推：原理与上述“转推”描述一样，只不过是在转推前先向调度服务器请求转推服务器地址，这样就可以做到动态切换转推目标服务器。
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191015105428989.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ltcGluZ28=,size_16,color_FFFFFF,t_70)
- 静态转推：与“动态转推”类似，只不过转推地址是在配置文件中固定填写的。
  ![静态转推](https://img-blog.csdnimg.cn/20190902220931792.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ltcGluZ28=,size_16,color_FFFFFF,t_70)

# 单集群方案

单集群是指部署在同一个物理节点的N台媒体服务器协同提供服务。主播推流到其中一台机器上，观众从集群中任意一台服务器拉流都可拉流成功。通过这种方式可以将服务器出口压力分散到多台机器上。
想要实现回源和转推功能需要用到nginx-rtmp中的pull_module、push_module和oclp_module，以下会给出详细的配置方式。

## 静态转推

通过push配置可以将推送到该服务器的流转推到push指定的服务器。

```nginx
rtmp {
    server {
        listen 1935;
        application live {
            live on;
            push rtmp://xxx/xxx;
		}
    }
}
123456789
```

## 静态回源

通过pull配置可以将远端特定服务器上的流拉取到此服务器上。pull操作支持http-flv回源，只需将pull后的源地址换成http-flv地址即可。

```nginx
rtmp {
    server {
        listen 1935;
        application live {
            live on;
            pull rtmp://xxx/xxx;
		}
    }
}
123456789
```

## 动态转推和动态回源

关于动态转推和动态回源这里引用[AlexWoo](https://github.com/AlexWoo/nginx-rtmp-module/blob/master/doc/ngx_rtmp_oclp_module.chs.md)项目中的说明文档。
在开始之前先描述一下动态回源和转推的原理，简单来说之所以称作“动态”是因为回源地址和转推地址不是固定的（可以动态变化），为了实现动态变化的效果，需要借助一个调度服务器，这个调度服务器的作用就是提供http接口返回目标服务器的回源地址或者转推地址（通过http 302 返回目标地址，目标地址保存在location字段里）。有了这样一个调度服务，我们就可以通过改变调度服务返回的目标地址字段来达到控制nginx-rtmp服务器回源和转推功能。
动态回源和动态转推是通过ngx_rtmp_oclp_module模块实现的。

### ngx_rtmp_oclp_module

------

#### 简介

ngx_rtmp_oclp_module 是 nginx rtmp 的能力开放模块，对外提供相应的通知或控制能力，用于替代 ngx_rtmp_notify_module。
这个模块会在rtmp或者http-flv推拉流的不同阶段向你配置文件里指定的url地址发送http get请求（控制事件），同时服务器会根据http返回结果做出相应的动作。

控制事件包含：

- proc：进程启动时通知，只支持 start 触发点
- play：有播放端接入时通知
- publish：有推流端接入时通知
- pull：触发回源拉流事件控制
- push：触发转推事件控制
- stream：流创建通知
- meta：收到音视频头通知
- record(还未支持)：启动录制通知

控制事件触发点包含：

- start：事件发生时触发，配置或不配置 start 均为默认开启状态
- update：事件持续过程中的心跳刷新
- done：事件结束时触发

扩展参数

- args：向外发送通知或控制请求时，携带的 http 请求参数
- groupid：分组，主要针对 push 或 meta，多路转推时，用于标识每路转推用
- stage：触发阶段，可选 start，update 和 done
- timeout：向外发送通知或控制请求时，等待外部响应的超时时间，默认为 3s
- update：发送 update 通知的时间间隔，默认为 1min，只有 stage 配置了 update 才生效

#### 基本模型

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019090413570827.jpg)

- 当有拉流时，触发 play 通知，当拉流持续时，每隔一定时间间隔向外发送 update 通知，当拉流结束时发送 done 通知
- 当有推流时，触发 publish 通知，当推流持续时，每隔一定时间间隔向外发送 update 通知，当推流结束时发送 done 通知
- 当有一路拉流或一路推流时，触发 stream 通知，当流还继续存在时，每隔一定时间间隔向外发送 update 通知，所有推流和拉流都结束时发送 done 通知

#### 拉流模型

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190904135720911.jpg)

- 当有拉流时，触发 play 通知，当拉流持续时，每隔一定时间间隔向外发送 update 通知，当拉流结束时发送 done 通知
- 当需要回源拉流时，触发 pull 控制，回源拉流创建成功后，每隔一定时间间隔向外发送 update 通知，当拉流结束时发送 done 通知
- 当有拉流时，触发 stream 通知，当流还继续存在时，每隔一定时间间隔向外发送 update 通知，所有拉流都结束时发送 done 通知

#### 推流模型

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190904135733997.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ltcGluZ28=,size_16,color_FFFFFF,t_70)

- 当有推流时，触发 publish 通知，当推流持续时，每隔一定时间间隔向外发送 update 通知，当推流结束时发送 done 通知
- 当需要转推流时，触发 push 控制，转推流创建成功后，每隔一定时间间隔向外发送 update 通知，当转推流结束时发送 done 通知
- 当有推流时，触发 stream 通知，当流还继续存在时，每隔一定时间间隔向外发送 update 通知，推流结束时发送 done 通知

meta 的行为和 push 一致，只是 meta 的触发点在收到音视频头，push 触发点是收到 publish 命令，push 和 meta 是互斥的，配置了 push，meta 将不生效

### 示例

基本模型

```
application test {
    live on;
    cache_time 3s;
    idle_streams off;
    oclp_publish http://127.0.0.1:6200/oclp/v1/publish stage=start,done args=a=c&b=d;
    oclp_play    http://127.0.0.1:6200/oclp/v1/play    stage=start,done args=a=c&b=d;
    oclp_stream  http://127.0.0.1:6200/oclp/v1/stream  stage=start,done args=a=c&b=d;
}
12345678
```

拉流模型

```
application edge {
    live on;
    cache_time 3s;
    low_latency on;
    oclp_pull     http://127.0.0.1:6200/oclp/v1/pull    stage=start,update,done args=a=c&b=d;
    pull http://101.200.241.232:11939/core app=core;
}
1234567
```

推流模型

```
application pub {
    live on;
    #oclp_push    http://127.0.0.1:6200/oclp/v1/push    stage=start,update,done args=a=c&b=d timeout=1s;
    oclp_meta    http://127.0.0.1:6200/oclp/v1/push    stage=start,update,done args=a=c&b=d timeout=1s;
}
12345
```

### 配置

#### oclp_proc

```
Syntax: oclp_proc url [timeout=time] [update=time];
Default: -
Context: rtmp
123
```

进程启动时，向配置的 url 发送 http get 请求，通知时会携带 call=init_process&worker_id=$ngx_worker 参数。

#### oclp_play

```
Syntax: oclp_play url [args=string] [stage=[start][,update][,done]] [timeout=time] [update=time];
Default: -
Context: application
123
```

只能配置一条

外部拉流时，向配置的 url 发送 http get 请求，通知时会携带

```
call=play&act=start&domain=$domain&app=$app&name=$name
1
```

如果配置了 args，会将 args 追加到默认参数后面

如果配置了 done，拉流结束后会发送 done 通知，和初始通知区别是，act=done

如果配置了 update，拉流结束前，每隔 update 时间间隔会发送一条刷新通知，和初始通知区别是，act=update

对于初始请求，如果外部异常，将不会发送 update 请求；如果外部回送非 200 响应，将会使用 403/NetStream.Play.Forbidden 断掉拉流请求；如果外部回送 200 响应并配置了刷新，会启动 update 定时器发送刷新通知

对于刷新通知和结束通知，不对响应做处理

#### oclp_publish

```
Syntax: oclp_publish url [args=string] [stage=[start][,update][,done]] [timeout=time] [update=time];
Default: -
Context: application
123
```

只能配置一条

外部推流时，向配置的 url 发送 http get 请求，通知时会携带

```
call=publish&act=start&domain=$domain&app=$app&name=$name
1
```

如果配置了 args，会将 args 追加到默认参数后面

如果配置了 done，推流结束后会发送 done 通知，和初始通知区别是，act=done

如果配置了 update，推流结束前，每隔 update 时间间隔会发送一条刷新通知，和初始通知区别是，act=update

对于初始请求，如果外部异常，将不会发送 update 请求；如果外部回送非 200 响应，将会使用 403/NetStream.Play.Forbidden 断掉拉流请求；如果外部回送 200 响应并配置了刷新，会启动 update 定时器发送刷新通知

对于刷新通知和结束通知，不对响应做处理

#### oclp_stream

```
Syntax: oclp_stream url [args=string] [stage=[start][,update][,done]] [timeout=time] [update=time];
Default: -
Context: application
123
```

只能配置一条

流创建时(有一路推流或一路拉流时)，向配置的 url 发送 http get 请求，通知时会携带

```
call=stream&act=start&domain=$domain&app=$app&name=$name
1
```

如果配置了 args，会将 args 追加到默认参数后面

如果配置了 done，所有推拉流结束后会发送 done 通知，和初始通知区别是，act=done

如果配置了 update，所有推拉流结束前，每隔 update 时间间隔会发送一条刷新通知，和初始通知区别是，act=update

对于初始请求，如果外部异常或外部回送非 200 响应，将不会发送 update 请求；如果外部回送 200 响应并配置了刷新，会启动 update 定时器发送刷新通知

对于刷新通知和结束通知，不对响应做处理

#### oclp_pull

```
Syntax: oclp_pull url [args=string] [stage=[start][,update][,done]] [timeout=time] [update=time];
Default: -
Context: application
123
```

只能配置一条

需要触发回源拉流时，向配置的 url 发送 http get 请求，通知时会携带

```
call=pull&act=start&domain=$domain&app=$app&name=$name
1
```

如果配置了 args，会将 args 追加到默认参数后面。
如果配置了 done，relay pull session 结束后会发送 done 通知，和初始通知区别是，act=done
如果配置了 update，relay pull session 结束前，每隔 update 时间间隔会发送一条刷新通知，和初始通知区别是，act=update
对于初始请求，如果外部异常，将会启动 pull reconnect；如果外部回送 200 响应，将不会创建 relay pull session，因此后续不会有 update 和 done 通知；如果外部回送 4XX 或 5XX 响应，将会结束所有拉流请求。
如果回送 3XX 响应，将会根据 Location 头构造回源请求，Location 必须是完整的 rtmp 或 http url，如 rtmp://ip/app/name[?args]，如果是 rtmp url 将会使用 rtmp 协议回源，如果是 http url 将会使用 http flv 协议回源。如果 3XX 响应中携带 Domain 头，将会使用该头构造 rtmp 的tc_url 或 http 的 Host 头

对于刷新通知和结束通知，不对响应做处理

#### oclp_push

```
Syntax: oclp_push url [args=string] [stage=[start][,update][,done]] [timeout=time] [update=time];
Default: -
Context: application
123
```

最多支持配置 8 条，每条对应一个 relay push session

需要触发流转推时，向配置的 url 发送 http get 请求，通知时会携带

```
call=push&act=start&domain=$domain&app=$app&name=$name
1
```

如果配置了 args，会将 args 追加到默认参数后面

如果配置了 done，每条 relay push session 结束后会发送 done 通知，和初始通知区别是，act=done

如果配置了 update，relay push session 结束前，每隔 update 时间间隔会发送一条刷新通知，和初始通知区别是，act=update

对于初始请求，如果外部异常，将会启动 push reconnect；如果外部回送 200 响应，将不会创建 relay push session，因此后续不会有 update 和 done 通知；如果外部回送 4XX 或 5XX 响应，将会结束推流请求

如果回送 3XX 响应，将会根据 Location 头构造回源请求，Location 必须是完整的 rtmp url，如 rtmp://ip/app/name[?args]。如果 3XX 响应中携带 Domain 头，将会使用该头构造 rtmp 的 tc_url

对于刷新通知和结束通知，不对响应做处理

#### oclp_meta

```
Syntax: oclp_meta url [args=string] [stage=[start][,update][,done]] [timeout=time] [update=time];
Default: -
Context: application
123
```

最多支持配置 8 条，每条对应一个 relay push session

基本功能与 oclp_push 相同，只不过 oclp_push 是 publish 命令触发，oclp_meta 是音视频头触发，具体触发规则见 oclp_meta_type 和 oclp_meta_once

#### oclp_meta_once

```
Syntax: oclp_meta_once on|off;
Default: on
Context: rtmp, server, application
123
```

oclp_meta 通知是否只触发一次

#### oclp_meta_type

```
Syntax: oclp_meta_once video|audio|both;
Default: video
Context: rtmp, server, application
123
```

- video：oclp_meta 只有在收到视频头时触发
- audio：oclp_meta 只有在收到音频头时触发
- both：oclp_meta 在收到音频或视频头时均会触发