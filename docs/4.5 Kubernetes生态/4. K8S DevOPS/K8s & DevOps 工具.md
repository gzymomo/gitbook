## 集群部署工具

1、Amazon EKS

Amazon Elastic Container Service是一个Kubernetes DevOps工具，它允许用户管理和扩展他们的容器化应用程序，并使用Kubernetes简化部署。当你需要一个足够安全、足够稳定的Kubernetes服务， 用尽可能少的精力去维护基础设施，希望将更多的精力投放在业务的研发上时，Amazon EKS 就会成为一个值得你选择的选项。Amazon EKS具有灵活的布局并减少了维护开销。

2、Kubespray

KubeSpray是一个集群生命周期管理器，可以帮助部署可用于生产的Kubernetes集群。它使用ansible-playbook来自动化Kubernetes集群配置。主要功能包括基于Ansible，高度可用，跨平台;流行的云提供商集成甚至是裸机，多种配置选项，多平台CI/CD等等。因为Kubespray拥有一个开放的开发模型，易于使用，大大降低了编排集群的难度，任何人都可以很容易地学习如何使用Kubespray。

3、Conjure-up

Conjure-up易于使用，允许用户以最少的问题部署他们的应用程序。支持本地主机部署、AWS、bare metal、Azure、VMware、Joynet和OpenStack。

## 监控工具

4、Kubewatch

Kubewatch是一个很好用的工具，kubewatch能够监控那些特定的Kubernetes事件，并将此类事件以通知的形式推送到诸如Slack和PagerDuty的端点上。可以确保你的容器是安全的，并使用行业最佳实践进行打包，同时监视软件的漏洞和更新。但是，用户表示，添加对多个实例的支持将会更有帮助。支持多个端点，且易于部署。

5、Weave Scope

Weave Scope用来监视和解决Kubernetes和Docker集群的故障，你就可以解放双手轻松地识别和纠正你的容器化应用程序的问题。

6、Test-infra

Testinfra 是一个基础架构测试框架，它可以轻松编写单元测试来验证服务器的状态。它支持的后端之一是 Ansible，所以这意味着 Testinfra 可以直接使用 Ansible 的清单文件和清单中定义的一组机器来对它们进行测试。对于处理复杂的模板来测试和检测错误非常有用。

7、Trireme

Trireme通过提高Kubernetes进程、工作负载和容器的安全性和降低复杂性，引入了一种不同的网络授权方法。建议用它来减轻Kubernetes工作负载、容器和进程的复杂性。它可以帮助你在应用程序层强制实施安全性。

8、Sysdig Falco

这是一个可以提供深度容器可见性的行为活动监视工具，它缩短了检测安全事件所需的时间，并应用了允许你持续监视和检测容器、应用程序、主机和网络活动的规则。使用它可以对其基础设施进行持续检查、异常检测，并为任何类型的 Linux 系统调用设置警报通知。

还可以通过 Falco 监视 shell 何时在容器中运行、容器在哪里挂载、对敏感文件的意外读取、出站网络尝试以及其他可疑调用。

## CLI工具

9、Cabin

Cabin是一个移动仪表盘，通过Android或iOS对Kubernetes进行管理。它同意用户远程管理他们的集群，是个很赞的工具，可以对所有事件进行快速补救。当Kubernetes应用程序离开主设备时，Cabin可以快速管理它们。这并不是一个用于开发的工具。当工程师经常远离他们的主计算机，需要快速管理他们的Kubernetes集群时，Cabin就很有用。

10、Kubectx/Kubens Kubectx/Kubens使用自动完成特性，通过在集群之间来回切换，帮助用户轻松切换上下文，并同时连接到各个集群。你可以使用它在Kubernetes命名空间之间平稳地切换。它有益于始终在集群或命名空间之间导航的用户。

## 开发工具

11、Telepresence

它让你可以在本地调试Kubernetes服务，简化了开发和调试过程。

12、Helm

Helm帮助用户管理他们的Kubernetes应用程序，通过Helm图表让你可以共享你的应用程序。这让用户能够创建可共享可复制的构建，但它不推荐用于更高级、更频繁的部署。

13、Keel

它让用户可以重新专注于编写代码和测试他们的应用程序。因为如果库中有新的应用程序版本可用，它就会自动更新kubernetes的工作负载。