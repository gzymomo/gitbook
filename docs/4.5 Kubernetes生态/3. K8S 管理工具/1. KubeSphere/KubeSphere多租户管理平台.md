目前，平台的资源一共有三个层级，包括 **集群 (Cluster)、 企业空间 (Workspace)、 项目 (Project) 和 DevOps Project (DevOps 工程)**，层级关系如下图所示，即一个集群中可以创建多个企业空间，而每个企业空间，可以创建多个项目和 DevOps工程，而集群、企业空间、项目和 DevOps工程中，默认有多个不同的内置角色。

![img](https://pek3b.qingstor.com/kubesphere-docs/png/20191026004813.png)



### 集群管理员

#### 第一步：创建角色和账号

平台中的 cluster-admin 角色可以为其他用户创建账号并分配平台角色，平台内置了集群层级的以下三个常用的角色，同时支持自定义新的角色。

| 内置角色           | 描述                                                         |
| :----------------- | :----------------------------------------------------------- |
| cluster-admin      | 集群管理员，可以管理集群中所有的资源。                       |
| workspaces-manager | 集群中企业空间管理员，仅可创建、删除企业空间，维护企业空间中的成员列表。 |
| cluster-regular    | 集群中的普通用户，在被邀请加入企业空间之前没有任何资源操作权限。 |

本示例首先新建一个角色 (users-manager)，为该角色授予账号管理和角色管理的权限，然后新建一个账号并给这个账号授予 users-manager 角色。

| 账号名       | 集群角色      | 职责                 |
| :----------- | :------------ | :------------------- |
| user-manager | users-manager | 管理集群的账户和角色 |

通过下图您可以更清楚地了解本示例的逻辑： ![img](https://hello-world-20181219.pek3b.qingstor.com/KS2.1/%E5%A4%9A%E7%A7%9F%E6%88%B7%E7%AE%A1%E7%90%86%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8.png)