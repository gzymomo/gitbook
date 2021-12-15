#### 一、什么是高可用？

`高可用`（High Availability）是`分布式系统架构`设计中必须考虑的因素之一，通常是指：`通过设计从而减少系统不能提供服务的时间`。

#### 二、怎么来衡量高可用？

举个例子，比如说一个系统它一直能够为你提供服务，那它的系统可用性就是`100%`，当系统运行到`100个时间单位`时，可能会有`1-2个时间单位无法为你提供服务`，那它的系统可用性就是`99%`和`98%`，在一年的时间内保证`99％可用性`的系统`最多可以有3.65天的停机时间（1％）`。这些值根据几个因素计算的，包括`计划`和`非计划`维护周期，以及从可能的`系统故障中恢复的时间`。

目前大部分企业的`高可用目标是4个9，也就是99.99%`，有几个 9，就代表了你的可用性。

- **2个9**：基本可用，网站年度不可用时间小于 88 小时；
- **3个9**：较高可用，网站年度不可用时间小于 9 小时；
- **4个9**：具有自动恢复能力的高可用，网站年度不可用时间小于 53 分钟；
- **5个9**：极高可用，也就是很理想的状态，网站年度不可用时间小于 5 分钟；

可用性的`9`怎么计算出来的呢？

- 网站不可用时间 = 故障修复时间点 - 故障发现时间点
- 网站年度可用性指标 =（1 - 网站不可用时间/年度总时间）* 100%

可用性的考核：网站可用性，跟技术、运营、等各方面的绩效考核相关，因此在前期的架构设计中，关于系统高可用性的问题也会话很大一部分时间，互联网企业不同公司有着不同的策略，往往因为种种因素会直接影响到系统的高可用性，业务增长较快的网站同时也将面临着用户的增长率，同时也慢慢会降低高可用性的标准，因此也就会对网站做一些相关性的策略或后端设备的支持等；

一般都是采用`故障`来分的，也是对`网站故障`进行分类`加权计算故障`责任的方法。一般会给每个分类的故障设置一个权重（例如事故级故障权重为100，A类为20等），`计算公式为：故障分=故障时间（分钟）* 故障权重`。

#### 三、高可用网站架构设计目的是什么?

当服务器的集群设备频繁读写时，会导致硬件出现故障的现象。

其`高可用架构设计的目的`：保证服务器硬件故障时服务依然可用、数据依然保存并能够被访问。

#### 四、实现高可用的主要手段有哪些？

- **数据层面：冗余备份**

一旦某个服务器宕机，就将服务切换到其他可用的服务器上；

冗余备份分为：`冷备份`和`热备份`

冷备份是定期复制，不能保证数据可用性。

热备份又分为`异步热备`和`同步热备`，异步热备是指：多份数据副本的写入操作异步完成，同步热备是指：多份数据副本的写入操作同时完成。

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a00c280ccd41433dacbfce6af4008a4e~tplv-k3u1fbpfcp-zoom-1.image)

- **服务层面：失效转移**

如某块磁盘损坏，将从备份的磁盘读取数据。（首先是已经提前做好了数据同步操作）；

若数据服务器集群中任何一台服务器宕机时，那么应用程序针对这台服务器的所有读写操作都要重新路由到其他服务器，保证数据访问不会失败。

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b79c8ef6ef354067a7e06272fb0cd2e9~tplv-k3u1fbpfcp-zoom-1.image)

#### 五、高可用的应用

应用层处理网站应用的业务逻辑，最显著的特点是：`应用的无状态性`。

`无状态性的应用是`：指应用服务器不保存业务的上下文信息，仅根据每次请求提交的数据进行相应的业务逻辑处理，且多个服务实例（服务器）之间完全对等，请求提交到任意服务器，处理结果都是完全一样的。

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/61bea9cc0f9b4e72a7c88ed518e8f79b~tplv-k3u1fbpfcp-zoom-1.image)

**1）通过`负载均衡`进行无状态服务的失效转移**

不保存状态的应用是给高可用架构带来了巨大便利，服务器不保存请求的状态，所有的服务器完全对等；

当任意一台或多台服务器出现宕机时，请求提交给集群中的其他任意一台可用服务器进行处理，对客户端用户来讲，请求总是成功的，整个系统依然可用。

对于应用服务器集群，实现这种服务器可用状态实时检测、自动转移失败任务的机制就是负载均衡。主要是在业务量和数据量使用频率较高时，单台服务器不足以承担所有的负载压力，那么可以通过负载均衡这种手段，将流量和数据平均到集群中其他服务器上，提高整体的负载处理能力。

不管在今后的工作中，是使用开源免费的负载均衡软件还是硬件设备，都需具备失效转移功能，网站应用中，集群中的服务器是无状态对等时，负载均衡即可起到事实上高可用的作用。

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6a3832802c654598a055c6fa5e7d305d~tplv-k3u1fbpfcp-zoom-1.image)

当 Web 服务器集群中的服务器都可用时，负载均衡服务器会把客户端发送到的访问请求分发到任意一台服务器上来进行处理，这时当`服务器2`出现宕机时，负载均衡服务器通过心跳检测机制发现该服务器失去响应，就会把它从服务器列表中删除，而将请求发送到 Web 服务器集群中的其他服务器上，这些服务器完全一样，请求在任何一台服务器中处理都不会影响到最终结果。

在实际环境中，负载均衡在应用层起到了系统高可用的作用，即便当某个应用访问量较少时，只用一台服务器足以支撑并提供服务，一旦需要保证该服务高可用时，必须至少部署两台服务器，从而使用负载均衡技术搭建一个小型的 Web 服务器集群。

**2）应用服务器集群的`Session`管理**

Web 应用中将多次请求修改使用的上下文对象称为会话（Session），单机情况下，Session 可部署在服务器上得 Web 容器（如 IIS、Tomcat 等）管理。

在使用了负载均衡的集群环境中，负载均衡服务器可能会将请求分发到 Web 服务器集群中的任何一台应用服务器上，所以保证每次请求能够获得正确的 Session 比单机时要复杂得多。

在集群环境中，Session 管理的几种常见手段：

- **Session 复制**

Session 复制：简单易行，是早期企业应用系统使用较多的一种服务器集群 Session 管理机制。应用服务器开启 Web 容器的 Session 复制功能，在集群中的其他服务器之间将会同步 Session 对象，与其使得每台服务器上都将会保存所有用户的 Session 信息。

当集群中的任何一台服务器出现宕机时，都不会导致 Session 数据的丢失，而服务器使用 Session 时，也只需要在本机获取即可。

Session 复制这种方案只适合集群规模较小的环境，当规模较大时，大量的 Session 复制操作会占用服务器和网络的大量资源，系统也将面临很大的压力。

所有用户的 Session 信息在每台服务器上都有备份，当大量用户访问时，甚至会出现服务器内存不够 Session 使用的情况，大型网站的核心应用集群都是数千台服务器以上，同时在线用户可达上千万，并不适合用 Session 复制这种方案。

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/40c2d430194a45d79f575860ea8aeb55~tplv-k3u1fbpfcp-zoom-1.image)

- **Session 绑定**

Session 绑定是利用负载均衡的源地址 Hash 算法实现的，负载均衡服务器总是将来源于同一 IP 的请求分发到同一台服务器上，在整个会话期间，用户所有的请求都在同一台服务器上处理，Session 绑定在某台特定服务器上，保证 Session 总能在这台服务器上获取，因此这种方法被称作会话粘滞。

但 Session 绑定这种方案不符合对于系统高可用的需求，一旦某台服务器出现宕机，那么该机器上的 Session 也将不存在了，用户请求切换到其他服务器上后，因此没有 Session 也将无法完成业务处理，大部分负载均衡服务器都提供源地址负载均衡算法，但很少有网站利用这个算法进行 Session 管理。

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b4bf194ebed947c0b175318540e44203~tplv-k3u1fbpfcp-zoom-1.image)

- **Cookie 记录 Session**

早期企业应用系统使用 C/S（客户端/服务器）架构，一种管理 Session 的方式是将 Session 记录在客户端，客户端请求服务器的时候，将 Session 放在请求中发送给服务器，服务器处理完请求后再将修改过的 Session 响应给客户端。因为网站没有客户端，因此利用浏览器支持的 Cookie 记录 Session。

利用浏览器支持的 Cookie 记录 Session 缺点：

- 受 Cookie 大小限制，能记录的信息有限
- 每次请求响应都需要传输 Cookie ，影响性能
- 如用户关闭 Cookie ，访问将会不正常

Cookie 简单易用，可用性高，支持应用服务器的线性伸缩，大部分应用需要记录的 Session 信息比较小。因此许多网站都将会使用 Cookie 来记录 Session。

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a2e332498b2549f3a7eaf15e27addfe2~tplv-k3u1fbpfcp-zoom-1.image)

- **Session 服务器**

利用独立部署的 Session 服务器或集群统一管理 Session ，应用服务器每次读写 Session 时，都将会访问 Session 服务器。其实是将应用服务器的状态进行分离为：`无状态的应用服务器`和`有状态的 Session 服务器`，针对这两种服务器的不同特性分别设计其架构。

对于有状态的 Session 服务器是利用分布式缓存、数据库等，在这些产品的基础上进行封装，使其符合 Session 的存储和访问要求。如果业务场景对 Session 管理有比较高的要求可利用 Session 服务集成单点登录、用户服务等功能，则需专门开发 Session 服务管理平台。

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6ddd4fe42ba44d8bbf0868567e769eb6~tplv-k3u1fbpfcp-zoom-1.image)

#### 六、高可用的服务

高可用的服务是用的服务模块为：业务产品提供基础公共服务，在大型网站中这些服务通常都独立分布式部署，被具体应用远程调用。可复用的服务和应用一样，是无状态的服务，可使用类似负载均衡的失效转移策略实现高可用的服务。

在具体实践中，高可用的几点服务策略：

- **分级管理**：运维上将服务器进行分级管理，核心应用和服务优先使用更好的硬件，在运维响应速度上也格外迅速，同时在服务部署上也进行必要的隔离，避免故障的连锁反应，低优先级的服务通过启动不同的线程或者部署在不同的虚拟机上进行隔离，而高优先级的服务则需要部署在不同的物理机上，核心服务和数据甚至需要部署在不同地域的数据中心。
- **超时设置**：在应用程序中设置服务调用的超时时间，一旦超时后，通信框架抛出异常，应用程序则根据服务调度策略选择重试或将请求转移到提供相同服务的其他服务器上；
- **异步调用**：通过消息队列等异步方式完成，避免一个服务失败导致整个应用请求失败的情况；
- **服务降级**：网站访问高峰期间，服务到大量并发调用时，性能会下降，可能会导致服务宕机，为保证核心应用及功能能够正常运行，需要对服务降级；

> 降级有两种手段：
>
> 一：拒绝服务，拒绝较低优先级的应用的调用，减少服务调用并发数，确保核心应用的正常运行；
>
> 二：关闭功能，关闭部分不重要的服务，或者服务内部关闭部分不重要的功能，以节约系统开销，为核心应用服务让出资源；

- **幂等性设计**：应用调用服务失败后，会将调用请求重新发送到其他服务器，服务重复调用时无法避免的，应用层其实不关心你服务是否真的失败，只要没有收到调用成功的响应，就认为调用失败，并重试服务调用。因此必须在服务层`保证服务重复调用和调用一次产生的结果相同，即服务具有幂等性`。

#### 七、常见的互联网分层架构

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ce18eccd54b74eeb8c67994b8497292a~tplv-k3u1fbpfcp-zoom-1.image)

整个系统的高可用，又是通过每一层的冗余+自动故障转移来综合实现，而常见互联网分布式架构如上，分为：

- **客户端层**：典型调用方是浏览器 browser 或者手机应用 APP
- **反向代理层**：系统入口，反向代理
- **站点应用层**：实现核心应用逻辑，返回 html 或者 json
- **服务层**：如果实现了服务化，就有这一层
- **数据-缓存层**：缓存加速访问存储
- **数据-数据库层**：数据库固化数据存储

#### 八、分层高可用架构

**客户端层 --> 反向代理层的高可用**

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/18221d8aaf484f579104c5be56ae0db6~tplv-k3u1fbpfcp-zoom-1.image)

**客户端层到反向代理层的高可用**，通过反向代理层的冗余来实现。以 Nginx 服务为例：需准备两台 Nginx，一台对线上提供服务，另一台做冗余保证高可用，常见的实践是`keepalived`存活探测，相同`虚拟 IP（virtual IP）`来提供服务。

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/906ca363a6c74c09af43e104b86d9085~tplv-k3u1fbpfcp-zoom-1.image)

**自动故障转移**：当一台Nginx宕机时，Keepalived能够检测到，会自动的将故障进行转移，使用的是相同的`虚拟IP`，切换过程对调用方是透明的。

**反向代理层 --> 站点层的高可用**

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ba95579f4528420aadced880c532e478~tplv-k3u1fbpfcp-zoom-1.image)

**反向代理层到站点层的高可用**，通过站点层的冗余来实现，反向代理层是Nginx，Nginx.conf 里能够配置多个 Web 后端，并且 Nginx 能够检测多个后端的存活性。

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/62563d20761d464ba9632b932901d41e~tplv-k3u1fbpfcp-zoom-1.image)

**故障自动转移**：当 Web-server 宕机时，Nginx 能够检测到，会自动进行故障转移，将流量自动转移到其他的 Web-server，整个过程由 Nginx 自动完成，对调用方是透明的。

**站点层 --> 服务层的高可用**

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1e1c1df0a8f24486a67ba65f4284f9aa~tplv-k3u1fbpfcp-zoom-1.image)

**站点层到服务层的高可用**，是通过服务层的冗余来实现的。“服务连接池”会建立与下游服务多个连接，每次请求会“随机”选取连接来访问下游服务。

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7962657a1cd5420fa54b26b75f373d61~tplv-k3u1fbpfcp-zoom-1.image)

**故障自动转移**：当 service 宕机时，service-connection-pool 能够检测到，会自动的进行故障转移，将流量自动转移到其他的 service，整个过程由连接池自动完成，对调用方是透明的（RPC-client 中的服务连接池是很重要的基础组件）。

**服务层 --> 缓存层的高可用**

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/be257f9349cb4c5e843bc7d2da3e4717~tplv-k3u1fbpfcp-zoom-1.image)

**服务层到缓存层的高可用**，是通过缓存数据的冗余来实现，缓存层的数据冗余可通过利用客户端的封装，service 对 cache 进行双读或者双写方式。

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eb5c404bf8654b4f9254738e12988785~tplv-k3u1fbpfcp-zoom-1.image)

缓存层也可以通过支持主从同步的缓存集群来解决缓存层的高可用问题，redis 天然支持主从同步，redis也有 sentinel 机制，来做 redis 的存活性检测。

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d80ef5d3a2ec4e21bcbbf35971b6ca1d~tplv-k3u1fbpfcp-zoom-1.image)

**自动故障转移**：当 redis 主挂了的时候，sentinel 能够检测到，会通知调用方访问新的redis，整个过程由 sentinel 和 redis 集群配合完成，对调用方是透明的。

**服务层 --> 数据库层的高可用**，大部分互联网数据库层都将采用了`主从复制，读写分离`架构，所以数据库层的高可用又分为`读库高可用`与`写库高可用`两类。

**服务层 --> 数据库层`读`的高可用**

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b87fa5904a3f4ebfac0f1e9dcb98e644~tplv-k3u1fbpfcp-zoom-1.image)

服务层到数据库读的高可用，是通过读库的冗余来实现，冗余了读库，一般来说就至少有2个从库，`数据库连接池`会建立与读库多个连接，每次请求会路由到这些读库里面去。

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a046743f2732430f86dfc2814e22d5c0~tplv-k3u1fbpfcp-zoom-1.image)

**故障自动转移**：当一台读库宕机时，db-connection-pool 能够检测到，会自动的进行故障转移，将流量自动迁移到其他的读库，整个过程由连接池自动完成，对调用方是透明的，数据库连接池是很重要的基础组件。

**服务层 --> 数据库层`写`的高可用**

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e280fd7426da4afea8efe4aebadeb57e~tplv-k3u1fbpfcp-zoom-1.image)

**服务层到数据库写的高可用**，是通过写库的冗余来实现，可以设置两台`MySQL`双主同步，一台对线上提供服务，另一台做冗余以保证高可用，常见的实践是`keepalived`存活探测，相同`虚拟IP（virtual IP）`提供服务。

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/32705924eeaa454faa0f598aa5643770~tplv-k3u1fbpfcp-zoom-1.image)

**故障自动转移**：当写库宕机时，`keepalived`能够检测到，会自动的进行故障转移，将流量自动迁移到`shadow-db-master`，使用的是相同的`虚拟IP（virtual IP）`，这个切换过程对调用方是透明的。

#### 九、配置高可用的准备工作

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3abf580186f5463991775d515f6b1105~tplv-k3u1fbpfcp-zoom-1.image)

***1、\*** 准备两台 Nginx 服务器（IP 地址：192.168.1.10 和 192.168.1.11），并在两台 Nginx 服务器上安装`Keepalived`，以及配置`虚拟 IP 地址`；

***2、\*** 192.168.1.10 服务器，因为我们前期就已经安装好了 Nginx，无须在重新安装了，只需在 192.168.1.11 设备上安装 Nginx 服务即可，详细可以查看这篇文章：《[Nginx系列教程（一）| 手把手教你在Linux环境下搭建Nginx服务](https://juejin.cn/post/6968637428684816420)》；

***3、\*** 分别在两台`Nginx`服务器上安装`Keepalived`服务，可通过 rpm 包或 yum 一键安装，这两种安装方式都是可以的，根据个人所需安装即可；

```
# rpm -ivh /mnt/Packages/keepalived-1.2.7-3.el6.x86_64.rpm 
# yum -y install keepalived
复制代码
```

***4、\*** 在两台`Nginx`服务器上分别启动`Nginx`服务和`keepalived`服务；

```
# cd /usr/local/nginx/sbin
# ./nginx
# service keepalived start
正在启动 keepalived：                                      [确定]
复制代码
```

***5、\*** 在客户端浏览器中分别输入`192.168.1.10`和`192.168.1.11`进行验证是否能够正常访问`Nginx`服务；

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1f2ed91cd7c8472e8ff287c5d69e9643~tplv-k3u1fbpfcp-zoom-1.image)

#### 十、配置高可用的主备模式实操案例

**主备方案**：这种方案也是目前企业中最常用的一种高可用的方案，简单来说，就是指一台服务器在提供服务时，另一台服务器为其他服务且是备用状态，当一台服务器出现宕机时，将自动跳转至备用服务器上，因此客户端所发出的请求将不会出现有失败现象。

在上述的准备工作介绍到了本次配置高可用将采用`Keepalived`来实现，那么什么是`Keepalived`?

`Keepalived`是一款`服务器状态检测和故障切换的工具`，起初是专为`LVS负载均衡`软件设计的，用来`管理`并`监控LVS集群`系统中`各个服务节点的状态`，后来又加入了可以实现高可用的VRRP (Virtual Router Redundancy Protocol ,虚拟路由器冗余协议）功能。

因此，`Keepalived`除了能够管理`LVS`软件外，还可以作为其他服务（`例如：Nginx、Haproxy、MySQL等`）的高可用解决方案软件在其配置文件中，可以配置主备服务器和该服务器的状态检测请求。也就是说`keepalived`可以根据配置的请求，在提供服务期间不断向指定服务器发送请求，如果该请求返回的状态码是200，则表示该服务器状态是正常的，如果不正常，那么`Keepalived`就会将该服务器给下线掉，然后将备用服务器设置为上线状态，而当主服务器节点恢复时，备服务器节点会释放主节点故障时自身接管的 IP 资源及服务，恢复到原来的备用角色。

***1、\*** 主服务器上配置`keepalived.conf`配置文件

```
# vim /etc/keepalived/keepalived.conf 

  1 global_defs {
  2    notification_email {
  3      acassen@firewall.loc
  4      failover@firewall.loc
  5      sysadmin@firewall.loc
  6    }
  7    notification_email_from Alexandre.Cassen@firewall.loc
  8    smtp_server 192.168.1.10               # 主服务器 IP 地址
  9    smtp_connect_timeout 30
 10    router_id LVS_DEVEL
 11 }
 12 
 13 vrrp_script chk_http_port {
 14 
 15    script "/usr/local/src/nginx_check.sh" # nginx_check.sh 脚本路径
 16 
 17    interval 2              # 检测脚本执行的间隔
 18 
 19    weight 2
 20 
 21 }
 22 
 23 vrrp_instance VI_1 {
 24     state MASTER           # 指定当前节点为 master 节点
 25     interface eth0         # 这里的 eth0 是网卡的名称，通过 ifconfig 或者 ip addr 可以查看
 26     virtual_router_id 51   # 这里指定的是虚拟路由 id，master 节点和 backup 节点需要指定一样的
 27     priority 90            # 指定了当前节点的优先级，数值越大优先级越高，master 节点要高于 backup 节点
 28     advert_int 1           # 指定发送VRRP通告的间隔，单位是秒
 29     authentication {
 30         auth_type PASS     # 鉴权，默认通过
 31         auth_pass 1111     # 鉴权访问密码
 32     }
 33     virtual_ipadress {
 34          192.168.1.100     # 虚拟 IP 地址
 35     }
 36 }
复制代码
```

***2、\*** 从服务器上配置`keepalived.conf`配置文件

```
# vim /etc/keepalived/keepalived.conf 

  1 global_defs {
  2    notification_email {
  3      acassen@firewall.loc
  4      failover@firewall.loc
  5      sysadmin@firewall.loc
  6    }
  7    notification_email_from Alexandre.Cassen@firewall.loc
  8    smtp_server 192.168.1.10               # 主服务器 IP 地址
  9    smtp_connect_timeout 30
 10    router_id LVS_DEVEL
 11 }
 12 
 13 vrrp_script chk_http_port {
 14 
 15    script "/usr/local/src/nginx_check.sh"  # nginx_check.sh 脚本路径
 16 
 17    interval 2              # 检测脚本执行的间隔
 18 
 19    weight 2
 20 
 21 }
 22 
 23 vrrp_instance VI_1 {
 24     state BACKUP           # 指定当前节点为 BACKUP 节点
 25     interface eth1         # 这里的 eth0 是网卡的名称，通过 ifconfig 或者 ip addr 可以查看
 26     virtual_router_id 51   # 这里指定的是虚拟路由 id，master 节点和 backup 节点需要指定一样的
 27     priority 80            # 指定了当前节点的优先级，数值越大优先级越高，master 节点要高于 backup 节点
 28     advert_int 1           # 指定发送VRRP通告的间隔，单位是秒
 29     authentication {
 30         auth_type PASS     # 鉴权，默认通过
 31         auth_pass 1111     # 鉴权访问密码
 32     }
 33     virtual_ipadress {
 34          192.168.1.100     # 虚拟 IP 地址
 35     }
 36 }
复制代码
```

***3、\*** 将`nginx_check.sh`脚本分别放置在两台`Nginx`服务器的`/usr/local/src/`目录下，用于通过`Keepalived`来检测`Nginx主服务器`是否还活着，如果是已经宕机了，将自动切换至`从服务器`上面去。

```
# vi /usr/local/src/nginx_check.sh 
 1 #!/bin/bash
 2 A=`ps -C nginx ¨Cno-header |wc -l`
 3 if [ $A -eq 0 ];then
 4     /usr/local/nginx/sbin/nginx
 5     sleep 2
 6     if [ `ps -C nginx --no-header |wc -l` -eq 0 ];then
 7         killall keepalived
 8     fi
 9 fi
复制代码
```

***4、\*** 在两台`Nginx`服务器上分别配置`虚拟 IP 地址`，对客户端的响应是真实服务器直接返回给客户端的，而真实服务器需要将响应报文中的源 IP 地址修改为虚拟 IP 地址，这里配置的虚拟 IP 地址就是起这个作用的。

```
主服务器
# ifconfig eth0:1 192.168.1.100 netmask 255.255.255.0
从服务器
# ifconfig eth1:1 192.168.1.100 netmask 255.255.255.0
复制代码
```

***5、\*** 在两台`Nginx`服务器上分别重启`Nginx`服务和`Keepalived`服务；

```
# ./nginx -s stop
# ./nginx
# service keepalived restart
复制代码
```

***6、\***  在客户端浏览器中输入虚拟 IP 地址`192.168.1.100`测试访问结果；

![图片](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/41f559e36e6f40efae1c3d22eaa32975~tplv-k3u1fbpfcp-zoom-1.image)

#### 十一、模拟主服务器故障验证高可用的效果

将主服务器的`Nginx`服务和`Keepalived`服务，都进行停止。

```
# ./nginx -s stop
# service keepalived stop
停止 keepalived：                                          [确定]
复制代码
```

通过客户端浏览器再次输入`虚拟 IP 地址 192.168.1.100`进行验证，可以发现还是能够正常访问`Nginx`服务，也就说明了当主服务器宕机时，将自动切换到从服务器上，因此不受客户端所访问造成的影响。

#### 总结

通过本篇文章介绍了`什么是高可用`、`如何来衡量高可用`、`高可用网站架构设计的目的`、`实现高可用的主要手段`、`高可用的应用及服务`、`常见的互联网分层架构`、`分层高可用架构详解`、`配置高可用的准备工作`、`主备模式的实操高可用案例`以及`模拟主服务故障`从而来验证整个高可用的效果。

整个互联网分层系统架构的高可用，是通过每一层的`冗余+自动故障转移`来实现的，具体的：

- **【客户端层】到【反向代理层】的高可用**：是通过反向代理层的冗余实现的，常见实践是`keepalived + virtual IP`自动故障转移；
- **【反向代理层】到【站点层】的高可用**：是通过站点层的冗余实现的，常见实践是`nginx`与`web-server`之间的存活性探测与自动故障转移；
- **【站点层】到【服务层】的高可用**：是通过服务层的冗余实现的，常见实践是通过`service-connection-pool`来保证自动故障转移；
- **【服务层】到【缓存层】的高可用**：是通过缓存数据的冗余实现的，常见实践是`缓存客户端双读双写`，或者利用`缓存集群的主从数据同步`与`sentinel`与`自动故障转移`；更多业务场景，对缓存没有高可用要求，可使用缓存服务化来对调用方屏蔽底层复杂性；
- **【服务层】到【数据库“读”】的高可用**：是通过读库的冗余实现的，常见实践是通过`db-connection-pool`来保证自动故障转移；
- **【服务层】到【数据库“写”】的高可用**：是通过写库的冗余实现的，常见实践是`keepalived + virtual IP`自动故障转移；


作者：杰哥的IT之旅
链接：https://juejin.cn/post/6970093569096810526
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。