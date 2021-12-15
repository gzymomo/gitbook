# Spark 实践 | Spark 大数据处理最佳实践

原文地址：https://mp.weixin.qq.com/s/83m0qo9C6iy8HJhbwR3prA

# 一、大数据概览

- 大数据处理 ETL (Data  →  Data)
- 大数据分析 BI  (Data  →  Dashboard)
- 机器学习   AI  (Data  →  Model)

![图片](https://mmbiz.qpic.cn/mmbiz_png/QDdcCFX9vIeeS5JtH3ibJiaGRfKUZ7Fd07vdz8jyUaiaYGe0a3lSvzCPd4kfvfus8AvmE8x2ib83HOVmHR5Nc4t1icQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# 二、如何摆脱技术小白

## 2.1 什么是技术小白？

- 只懂表面，不懂本质

  只懂得参考别人的 Spark 代码，不懂得 Spark 的内在机制，不懂得如何调优 Spark Job

## 2.2 摆脱技术小白的药方

- 懂得运行机制
- 学会配置
- 学会看 Log

## 2.3 懂得运行机制：Spark SQL Architecture

![图片](https://mmbiz.qpic.cn/mmbiz_png/QDdcCFX9vIeeS5JtH3ibJiaGRfKUZ7Fd07uatZ8kVRJK6pSrXXV1EtQMU4X3dU2BIPmp8UPNXTpddhC7q7GMsFiaw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)**

## 2.4 学会配置：如何配置 Spark App

- **配置 Driver**

   • spark.driver.memory

   • spark.driver.cores

- **配置 Executor**

   • spark.executor.memory

   • spark.executor.cores

- **配置 Runtime**

   • spark.files

   • spark.jars

- **配置 DAE**
- **…..........** 

**参考网址：**https://spark.apache.org/docs/latest/configuration.html

## 2.5 学会看 Log：Spark Log

![图片](https://mmbiz.qpic.cn/mmbiz_png/QDdcCFX9vIeeS5JtH3ibJiaGRfKUZ7Fd07kun5UXoWdQBgIZLIsO8O357LVVJJO8AHF9h0yLJhQTbmdn2R8USQgA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# 三、Spark SQL  学习框架

## 3.1 Spark SQL 学习框架( 结合图形/几何）

### 1. Select Rows

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](https://mmbiz.qpic.cn/mmbiz_png/QDdcCFX9vIeeS5JtH3ibJiaGRfKUZ7Fd07PG3WS0QPowHiafUVyBTows8yiaUlSvHIv0y3w6PZniaZpTs4tIpSI5qlQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 2. Select Columns

![图片](https://mmbiz.qpic.cn/mmbiz_png/QDdcCFX9vIeeS5JtH3ibJiaGRfKUZ7Fd07X4BFicMVDg3oXrLicIookS8jGEbJhSJeDSxofneyNb2VsqGicaLKQHcnw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/QDdcCFX9vIeeS5JtH3ibJiaGRfKUZ7Fd07z3oiao7GMLk7SnwMlyJlZY1TfKGMxQfFoMTzHznPUWYibuZvRVKJqFWw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 3. Transform Column

![图片](https://mmbiz.qpic.cn/mmbiz_png/QDdcCFX9vIeeS5JtH3ibJiaGRfKUZ7Fd077sCGfGqRyT8xVt9RM8TPdb1mmjFic1icogodA2PAb1KE08jRqT5icvE0A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/QDdcCFX9vIeeS5JtH3ibJiaGRfKUZ7Fd07IV8lRictAicBlHGptNrGVAuXuX3hNKgUhOx0kAicz7wZAEuPyk35bo2Yw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 4. Group By / Aggregation

![图片](https://mmbiz.qpic.cn/mmbiz_png/QDdcCFX9vIeeS5JtH3ibJiaGRfKUZ7Fd07oNsIxSbVR9TGOFqu7RhskjwV5VKuezGgtCFNyg8KQ4HDj0PTdyrjgA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/QDdcCFX9vIeeS5JtH3ibJiaGRfKUZ7Fd07V7FfNUCtOfBXXTd5enBwwJISdESjGwqOkhNop5uXtHqqDx90gEEhFA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 5. Join

![图片](https://mmbiz.qpic.cn/mmbiz_png/QDdcCFX9vIeeS5JtH3ibJiaGRfKUZ7Fd07sQES25hIrumrOoZmvNMicpick30tY1D20iaz3AVVibDlcG04tJCcE7fN1w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/QDdcCFX9vIeeS5JtH3ibJiaGRfKUZ7Fd07jZmZ3DEHqnNbrFnqEPLfoJcdJUTgwKicpRZXX5LSQnwicOlVFxdfZfDg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



## 3.2 Spark SQL 执行计划

### 1. Spark SQL - Where

![图片](https://mmbiz.qpic.cn/mmbiz_png/QDdcCFX9vIeeS5JtH3ibJiaGRfKUZ7Fd07wuicdYoxvbBia5Tib68muLNuAFKb4vAfgHhnpIo73ia8uMLVxRC1msFCgQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 2. Spark SQL - Group By

![图片](https://mmbiz.qpic.cn/mmbiz_png/QDdcCFX9vIeeS5JtH3ibJiaGRfKUZ7Fd074SdrILOqDOeHBMKcBBu4ckHiczIKJKGs9LlAeYCCAGq0Ilw9muL1fjA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 3. Spark SQL - Order by

![图片](https://mmbiz.qpic.cn/mmbiz_png/QDdcCFX9vIeeS5JtH3ibJiaGRfKUZ7Fd07ohia6KzOEKTTJqsjcm5Sj0ym6Uu8rvsOYP8bJxibIJyHEHckNfOBtGmw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# 四、EMR Studio 上的大数据最佳实践

## 4.1 EMR Studio 特性

- **兼容开源组件**
- **支持连接多个集群**
- **适配多个计算引擎**
- **交互式开发 + 作业调度无缝衔接**
- **适用多种大数据应用场景**
- **计算存储分离**

### 1. 兼容开源组件

- EMR Studio 在开源软件 Apache Zeppelin，Jupyter Notebook, Apache Airflow 的基础上优化了做了优化和增强。

![图片](https://mmbiz.qpic.cn/mmbiz_png/QDdcCFX9vIeeS5JtH3ibJiaGRfKUZ7Fd07ViaZCdPuS7icGHDl2TL53lDYcx0nzzAVh6Fwye2nrAUmb2icOfKXEaNFw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 2. 支持连接多个集群

- 一个 EMR Studio 可以连接多个 EMR 计算集群，您可以很方便地切换计算集群，提交作业到不同的计算集群上运行。

![图片](https://mmbiz.qpic.cn/mmbiz_png/QDdcCFX9vIeeS5JtH3ibJiaGRfKUZ7Fd07QBfibuicJbOAzEf62TW8k9RPFY7mGsg9t9uWsQ6iaY5JLhWMkMMSJLyjw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 3. 适配多个计算引擎

- 自动适配 Hive、Spark、Flink、Presto、Impala 和 Shell 等多个计算引擎，无需复杂配置，多个计算引擎间协同工作

![图片](https://mmbiz.qpic.cn/mmbiz_png/QDdcCFX9vIeeS5JtH3ibJiaGRfKUZ7Fd07EGX8T1mwn7q3a464HFgmbatOewaXQYN18ec0akyCael8PCczLm3sVw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 4. 交互式开发 + 作业调度无缝衔接

**Notebook + Airflow : 无缝衔接开发环节和生产调度环节**

- 利用交互式开发模式可以快速验证作业的正确性.
- 在 Airflow 里调度 Notebook 作业，最大程度地保证开发环境和生产环境的一致性，防止由于开发阶段和生产阶段环境不一致而导致的问题。

![图片](https://mmbiz.qpic.cn/mmbiz_png/QDdcCFX9vIeeS5JtH3ibJiaGRfKUZ7Fd07cib5Vf4X7pFMyiaQpiba251yfphufgPPW4upJiawcCkWibbibhTojT1S69Iw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 5. 适用多种大数据应用场景

- 大数据处理 ETL
- 交互式数据分析
- 机器学习
- 实时计算

### 6. 计算存储分离

- **所有数据都保存在 OSS 上，包括：**

​    • 用户 Notebook 代码

​    • 调度作业 Log

- **即使集群销毁，也可以重建集群轻松恢复数据**

![图片](https://mmbiz.qpic.cn/mmbiz_png/QDdcCFX9vIeeS5JtH3ibJiaGRfKUZ7Fd07MUqZgHibyJgCGoyNoutdS5mdVBcoOUviaak0Kg5ck5Qg7QwVdalH1Bxw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 