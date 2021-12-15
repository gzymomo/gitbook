[TOC]

[自动补全、回滚，可视化 sql 诊断利器](https://mp.weixin.qq.com/s/CK363iarS2qF28BU6yD7Xw)

# 1、Yearning简介
Yearning MYSQL 是一个SQL语句审核平台。提供查询审计，SQL审核等多种功能，支持Mysql，可以在一定程度上解决运维与开发之间的那一环，功能丰富，代码开源，安装部署容易！

> 项目地址：https://gitee.com/cookieYe/Yearning

# 2、Yearning 功能介绍

- SQL查询
  - 查询工单
  - 导出
  - 自动补全，智能提示
  - 查询语句审计
- SQL审核
  - 流程化工单
  - SQL语句检测与执行
  - SQL回滚
  - 历史审核记录
- 推送
  - E-mail工单推送
  - 钉钉webhook机器人工单推送
- 用户权限及管理
  - 角色划分
  - 基于用户的细粒度权限
  - 注册
- 其他
  - todoList
  - LDAP登录
  - 动态审核规则配置
- AutoTask自动执行

Yearning 不依赖于任何第三方SQL审核工具作为审核引擎,内部已自己实现审核/回滚相关逻辑。
仅依赖Mysql数据库。mysql版本必须5.7及以上版本，创建Yearning库字符集应为UTF8mb4 (仅Yearning所需mysql版本)
Yearning日志仅输出error级别,没有日志即可认为无运行错误！
Yearning 基于1080p分辨率开发仅支持1080p及以上显示器访问

打开浏览器 http://192.168.1.9:8000
默认密码：admin/Yearning_admin