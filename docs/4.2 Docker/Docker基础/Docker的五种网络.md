[TOC]



# 三、Docker的五种网络

**None**
不为容器配置任何网络功能。

**Container**
与另一个运行中的容器共享NetworkNamespace，共享相同的网络视图。

**Host**
与主机共享Root Network Namespace，容器有完整的权限可以操纵主机的协议栈、路由表和防火墙等，所以被认为是不安全的。

**Bridge**
Docker设计的NAT网络模型。
Docker网络的初始化动作包括：创建docker0网桥、为docker0网桥新建子网及路由、创建相应的iptables规则等。
![Docker关键知识点儿汇总](https://minminmsn.com/images/docker/bridge.jpg)
在桥接模式下，Docker容器与Internet的通信，以及不同容器之间的通信，都是通过iptables规则控制的。
**Overlay**
Docker原生的跨主机多子网模型。
overlay网络模型比较复杂，底层需要类似consul或etcd的KV存储系统进行消息同步，核心是通过Linux网桥与vxlan隧道实现跨主机划分子网。
![Docker关键知识点儿汇总](https://minminmsn.com/images/docker/overlay.jpg)


