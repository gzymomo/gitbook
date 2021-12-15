[**Pika** 360开源的类Redis存储系统](https://www.oschina.net/p/qihoo-360-pika)

Pika是360开源的类Redis存储系统。  

Pika 是 360 DBA 和基础架构组联合开发的类 Redis 存储系统，完全支持 Redis 协议，用户不需要修改任何代码，就可以将服务迁移至 Pika。有维护 Redis 经验的 DBA 维护 Pika 不需要学习成本。

Pika 主要解决的是用户使用 Redis 的内存大小超过 50G、80G 等等这样的情况，会遇到启动恢复时间长，一主多从代价大，硬件成本贵，缓冲区容易写满等问题。Pika 就是针对这些场景的一个解决方案。

特点

- 容量大，支持百G数据量的存储
- 兼容redis，不用修改代码即可平滑从redis迁移到pika
- 支持主从(slaveof)
- 完善的运维命令