## 顶级Docker替代品

Docker远非完美的产品，存在很多缺点。例如必须以root权限运行，并且停止容器将删除其中的所有信息（卷中的内容除外）。其他缺点还包括：安全性和隔离性不如VM、大规模不易管理（K8s应用而生）、问题排查较难、不支持Windows等。

事实上，目前Docker软件有不少优秀的替代品，其中不少产品的技术成熟度、稳定度和资源占用方面的表现不输甚至优于Docker。

以下，我们推荐**十二个Docker的最佳替代产品**，您可以投票改变排序。

## 1、OpenVZ

OpenVZ是基于Linux的流行的操作系统级服务器虚拟化技术，可在单个物理服务器中创建多个安全且隔离的虚拟环境，从而提高服务器利用率和性能。虚拟服务器确保应用程序不会冲突，并且可以独立重新启动。

OpenVZ还提供了一个网络文件系统（NFS），允许从OpenVZ虚拟环境访问网络磁盘文件。该工具支持IA64处理器的检查点和实时迁移，此功能是其他开源操作系统虚拟化软件所无法提供的，系统管理员无需最终用户干预即可使用虚拟服务器在物理服务器之间移动，而无需昂贵的存储系统。

OpenVZ是一种开源技术，也是SWsoft的Virtuozzo虚拟化产品的基础。它为虚拟环境中的VLAN提供了标准支持，从而允许在不同网络上标记每个网络数据包。支持FUSE（用户空间中的文件系统），例如，它可以将FTP或SSH服务器显示为虚拟环境中的文件系统。

```
网站：https://openvz.org/
系统支持：Linux
```

## 2、Rancher

Rancher是一种开源的容器管理技术，提供完整的容器基础设施服务，包括网络、存储服务、主机管理和负载均衡等，支持各种基础架构，可以简单可靠地部署和管理应用程序。

```
网站：https://rancher.com
支持系统：Linux
```

## 3、Nanobox

Nanobox是开发人员的理想DevOps平台。Nanobox可以完成基础结构的所有构建，配置和管理，因此您可以专注于代码而不是配置。

借助Nanobox，您可以自由地创建一致且隔离的开发环境，该环境可以轻松地与任何人共享，并且可以在任何主机（AWS、Digital Ocean、Azure、Google等）上实现。开发人员可以在本地计算机和云提供商之间一致地运行其应用程序。

你可以非常轻松地使用Nanobox仪表板管理生产应用程序，Nanobox还支持零停机时间部署和扩展，并通过统计信息显示板以及历史日志输出来监视应用程序的状态。

```
网站：https://nanobox.io/
系统支持：基于Web
```

## 4、Podman

PodMan是一个虚拟化的容器管理器，可用于Linux发行版，它的特殊之处在于它不需要运行Daemon，而是直接在runC上运行.PodMan允许我们以没有root特权的用户身份运行容器，从安全层面来看这极为重要！

通过Podman，我们不仅可以检查OCI映像，甚至不下载它们，还可以从一个存储库中提取元素并将其直接移动到另一个存储库中，镜像文件无需通过我们的设备传输。我们无需下载镜像即可检查或使用其组件。Podman还允许运行默认启用Systemd的容器，无需进行任何修改。

Podman支持套接字激活，因此我们可以使用该系统来配置套接字，并可以访问用于与该工具进行通信的远程API。它能够通过名称空间使用UID分隔，这在运行容器时提供了额外的隔离层。

```
下载链接：https://developers.redhat.com/blog/2018/08/29/intro-to-podman/
系统支持：Linux
```

## 5、RKT

RKT属于Core OS发行版，专为容器虚拟化和处理而开发。如今，它已成为Docker最大的竞争对手之一。RKT可在诸如ArchLinux、Core OS、Fedora、NixOS等Linux平台上工作。

Core OS决定启动RKT的主要原因之一就是安全性。在1.1版之前，Docker需要以root用户身份运行，这是一个非常严重的漏洞，允许超级用户级别的攻击。相反，RKT允许我们对Linux权限使用标准的组处理，从而允许容器在没有root特权的用户创建后运行。

Docker的优势是易于集成，而RKT需要更多的手动安装和配置。无论如何，它仍然是Docker的很好替代品，因为它允许我们使用APPC映像（App容器映像）以及Docker映像。反过来，它也允许与Kubernetes和AWS Orchestrator集成。

```
下载链接：https://github.com/rkt/rkt
系统支持：Linux
```

## 6、Singularity

Singularity是用于HPC（高性能计算）的操作系统虚拟器，因为它不需要与具有root特权的用户一起运行，并且由于其隔离级别而非常适合在共享空间中使用。其安全理念是“不安全的客户端运行不安全的容器”，这完全改变了安全范式。

关于Singularity的另一个重要事实是，我们可以导入和使用我们已经拥有的Docker映像。我们甚至可以在本地编辑容器，然后将其挂载到共享环境中，因为它不需要root特权即可挂载。也可以使用基本文件传输协议（例如RSYNC、HTTP、SCP等）进行传输。

```
下载链接：https://sylabs.io/singularity/
系统支持：Linux
```

## 7、Kubernetes（K8s）

Kubernetes是一个用于自动组织和管理容器化应用程序的开源系统。如果要使用流行的开源Linux容器设计应用程序，那么Kubernets可能是为私有，公共或混合云托管创建云原生应用程序最理想的方法之一。

Kubernetes使容器化应用程序的部署，管理和扩展自动化，可以更轻松，快速和高效地执行该过程。用户现在可以一键式更新来更新他们在集群中使用的Kubernetes的核心版本。使Kubernetes集群保持最新状态变得相当容易，因为现在无需重新部署集群或应用程序就可以做到这一点。

Kubernetes是一个开源项目，由Cloud Native Computing Foundation（CNCF）和Linux Foundation管理。这可以确保该项目得到大型开源社区的最佳实践和想法的支持，此外还消除了依赖单个提供商的风险。

```
网站：https://kubernetes.io/
系统支持：基于Web和Linux
```

## 8、Red Hat OpenShift Container Platform

Red Hat OpenShift Container Platform是一个开源的企业级Kubernetes平台，可用于开发、部署和管理横跨企业内部、私有云和公有云架构中的容器化应用。

```
网站：https://www.openshift.com/products/container-platform
系统支持：Linux、Windows
```

## 9、Apache Mesos

Mesoso是基于Linux内核的开源集群管理工具，可以在任何平台（Linux、Windows或OSX）上运行。它还为应用程序提供了用于资源管理和计划的API。可从专用服务器或虚拟机中提取CPU、内存、存储和其他资源，从而使弹性系统易于构建且可以高效运行，容错能力突出。

Mesos使用两层调度系统，在该系统中，它确定要分配给每个框架的资源的数量，而框架则确定要接受的资源以及在这些资源上运行哪些任务。你可以扩展到50,000个节点，在不同框架之间共享集群，并不断优化。

Mesos允许集群运行应用程序所在的框架，在不同服务器之间分配负载，从而避免过载，获得最佳性能。Mesos通常用于Java、Python、Scala和R应用程序。

```
网站：http://mesos.apache.org/
系统支持：Linux、OSX和Windows
```

## 10、FreeBSD

FreeBSD以其功能，速度，安全性和稳定性而著称。它来自BSD，这是在加州大学伯克利分校部署的UNIX改编版。它被广泛的社区部署和追随。FreeBSD提供了许多独特的功能，尤其以创建出色的Internet或Intranet服务器而闻名。FreeBSD可以在高负载下提供强大的网络服务，内存利用效率高，可以快速响应数百万个并发进程。

FreeBSD还提供了针对连接器和完整平台的改进的网络操作系统功能，支持从Intel推崇的高端连接器到ARM、MIPS和PowerPC硬件平台。FreeBSD拥有23,000多个库和外观应用程序，可支持用于台式机、助手、设备和集成媒体的应用程序。

```
网站：https://www.freebsd.org/
系统支持：Unix和基于Web的
```

## 11、Vagrant

Vagrant是自动创建和配置可移植可运行虚拟机的工具。与Docker这样的DevOps工具相比，Vagrant的一大优点是，任何计算机科学家/程序员/开发人员（甚至是使用Windows的人）都能快速掌握并使用它，因为Vagrant能配置并自动创建虚拟机。

Vagrant安装在开发人员的计算机上，面向开发环境，而不是生产环境。甚至Vagrant的开发公司都不推荐在生产环境中使用Vagrant。Vagrant是跨平台的，支持的系统包括：Mac、Windows、CentOS和Debian。Vagrant的定位是开发人员之间的，安装可移植且可运行开发环境的工具。

默认情况下，Vagrant使用Virtual Box进行虚拟化，但可与任何虚拟化软件一起使用，Vagrantfile配置文件的语法也很简单。

```
网站：https://www.vagrantup.com/
系统支持：Debian、centOS、Arch Linux、Linux、FreeBSD、macOS和Microsoft Windows
文件大小：210 MB（用于Windows）
```

## 12、LXC

LXC是一种操作系统级别的虚拟化技术，允许用户独立创建和运行多个虚拟Linux环境。

与Docker的不同之处在于，LXC可看作是一个完整的操作系统。另一方面，Docker只能运行单个应用程序，并且对OS有一定的限制。与Docker相比，LXC是一种更轻便，更安全的选择，因为它消耗的资源更少，并且不需要以root身份运行。

上述优点的代价就是复杂性增加，除此之外，我们还必须添加糟糕的文档。通常，当我们使用容器时，我们想要的是快速，轻松地创建我们的工作环境。因此，LXC这个替代方法更适合高级用户。

```
网站：https://linuxcontainers.org/
系统支持：Linux
```

