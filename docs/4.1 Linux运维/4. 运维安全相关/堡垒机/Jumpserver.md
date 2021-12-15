[TOC]

- [Jumpserver高可用集群部署：（三）MariaDB Galera 集群部署](https://blog.51cto.com/dusthunter/2542897)



# JumpServer

# CoDo
- CODO是一款为用户提供企业多混合云、自动化运维、完全开源的云管理平台。
- CODO前端基于Vue iview开发、为用户提供友好的操作界面，增强用户体验。
- CODO后端基于Python Tornado开发，其优势为轻量、简洁清晰、异步非阻塞。
- CODO开源多云管理平台将为用户提供多功能：ITSM、基于RBAC权限系统、Web Terminnal登陆日志审计、录像回放、强大的作业调度系统、CMDB、监控报警系统、DNS管理、配置中心等

![](https://blz.nosdn.127.net/sre/images/20190530.codo.02.png)

## 模块说明
- 项目前端：基于Vue + Iview-Admin实现的一套后台管理系统

- 管理后端：基于Tornado实现，提供Restful风格的API，提供基于RBAC的完善权限管理，可对所有用户的操作进行审计

- 定时任务：基于Tornado实现，定时任务系统，完全兼容Linux Crontab语法，且支持到秒级

- 任务调度：基于Tornado实现，系统核心调度，可分布式扩展，自由编排任务，自由定义流程，支持多种触发，支持审批审核，支持操作干预

- 资产管理：基于Tornado实现，资产管理系统，支持手动添加资产，同时也支持从AWS/阿里云/腾讯云自动获取资产信息

- 配置中心：基于Tornado实现，可基于不同项目、环境管理配置，支持语法高亮、历史版本差异对比、快速回滚，并提供Restful风格的API

- 域名管理：基于Tornado实现，支持多区域智能解析、可视化Bind操作、操作日志记录

- 运维工具：基于Tornado实现，运维场景中常用的加密解密、事件、故障、项目记录、提醒、报警等

## 在线体验
CoDo提供了在线Demo供使用者体验，Demo账号只有部分权限
>地址：http://demo.opendevops.cn
用户：demo
密码：2ZbFYNv9WibWcR7GB6kcEY

## 项目地址
- 官网：http://www.opendevops.cn
- GitHub：https://github.com/opendevops-cn
- 文档地址：http://docs.opendevops.cn/zh/latest
- 安装视频：https://www.bilibili.com/video/av53446517
