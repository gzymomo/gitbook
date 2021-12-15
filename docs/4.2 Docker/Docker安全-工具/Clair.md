- [安全防护工具之：Clair](https://blog.csdn.net/liumiaocn/article/details/76697022)
- [【Docker】镜像安全扫描工具clair与clairctl](https://blog.csdn.net/qq_33591903/article/details/99290602)
- [clair镜像扫描的实现](https://blog.csdn.net/qq_33591903/article/details/102892492)
- [Clair的2.X 版本安装部署及使用](https://www.jianshu.com/p/c802182d3a2c)



镜像扫描结构图

![镜像扫描结构图](https://img-blog.csdnimg.cn/20190929181845376.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMzNTkxOTAz,size_16,color_FFFFFF,t_70)

方式2的具体操作步骤

![api扫描方式的具体步骤](https://img-blog.csdnimg.cn/20190930140724106.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMzNTkxOTAz,size_16,color_FFFFFF,t_70)

Clair正是由coreos所推出的这样一款针对容器的安全扫描的工具，类似于Docker在其收费版中提供的功能那样，能对应用容器的脆弱性进行静态扫描，同时支持APPC和DOCKER。



clair是一个开源项目，用于静态分析appc和docker容器中的漏洞。 

漏洞元数据从一组已知的源连续导入，并与容器映像的索引内容相关联，以生成威胁容器的漏洞列表。 

# 项目地址

| 项目     | 详细                            |
| -------- | ------------------------------- |
| 项目地址 | https://github.com/coreos/clair |

# 为什么使用Clair

随着容器化的逐渐推进，使用的安全性也受到越来越多地重视。在很多场景下，都需要对容器的脆弱性进行扫描，比如

| 项目           | 详细                                                         |
| -------------- | ------------------------------------------------------------ |
| 镜像来源不明   | 在互联网上下载的镜像，可以直接使用，非常的方便，但是是否真正安全还非常难说 |
| 生产环境的实践 | 容器上到生产环境之后，生产环境对容器的安全性要求一般较高，此时需要容器的安全性得到保证 |

# 名称的由来

clair的目标是能够从一个更加透明的维度去看待基于容器化的基础框架的安全性。Clair=clear + bright + transparent

# 工作原理

通过对容器的layer进行扫描，发现漏洞并进行预警，其使用数据是基于Common Vulnerabilities and Exposures数据库(简称CVE), 各Linux发行版一般都有自己的CVE源，而Clair则是与其进行匹配以判断漏洞的存在与否，比如HeartBleed的CVE为：CVE-2014-0160。
![这里写图片描述](https://img-blog.csdn.net/20170805085052811?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGl1bWlhb2Nu/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

# 数据源支持

目前Clair支持如下数据源：

| 数据源                      | 具体数据                                                     | 格式 | License       |
| --------------------------- | ------------------------------------------------------------ | ---- | ------------- |
| Debian Security Bug Tracker | Debian 6, 7, 8, unstable namespaces                          | dpkg | Debian        |
| Ubuntu CVE Tracker          | Ubuntu 12.04, 12.10, 13.04, 14.04, 14.10, 15.04, 15.10, 16.04 namespaces | dpkg | GPLv2         |
| Red Hat Security Data       | CentOS 5, 6, 7 namespaces                                    | rpm  | CVRF          |
| Oracle Linux Security Data  | Oracle Linux 5, 6, 7 namespaces                              | rpm  | CVRF          |
| Alpine SecDB                | Alpine 3.3, Alpine 3.4, Alpine 3.5 namespaces                | apk  | MIT           |
| NIST NVD                    | Generic Vulnerability Metadata                               | N/A  | Public Domain |

# 数据库

Clair的运行需要一个数据库实例，目前经过测试的数据库的版本为

| 数据库     | 版本 |
| ---------- | ---- |
| postgresql | 9.6  |
| postgresql | 9.5  |
| postgresql | 9.4  |