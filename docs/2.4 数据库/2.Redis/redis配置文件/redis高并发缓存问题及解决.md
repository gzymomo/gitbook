[TOC]

# 1、redis高并发缓存问题及解决
## 1.1 redis缓存全量更新问题
![](http://img.chaiguanxin.com/file/2018/10/02e26d36553d450ca56c9ec4da972bee_image.png)

一般我们会把数据都以json格式存在redis里面，比如电商商品有分类、店铺信息、商品属性等等。如果分类变了，那需要把大json取下来，更新完，在保存到redis里面。这样的存储有几个不好的地方：
1. 因为存储的数据比较大，所以网络开销比较大；
2. 每次对redis做存取大数据，对redis的压力比较大；
3. 数据本身的大小会影响redis的吞吐量和性能；

redis缓存全量更新问题如何解决？
方法就是redis按维度化存储，把商品按照维度来拆分，比如商品分类，店铺信息、商品属性等，存储到redis的时候，也是按照维度来存储，这样原来你的key大小200k，拆分完之后每个key的大小为20k。大大提升了redis的性能和吞吐量。大大减少了网络资源的消耗，以及redis 的压力。
![](http://img.chaiguanxin.com/file/2018/10/085ec9c48eba4957ad6cad28ba7e74d7_image.png)

# 2、redis如何做到读QPS超过10万+？
一般情况下，单个redis大概可以到达几万的读QPS，这个数值会因为你机器的性能，数据的复杂性有所变化。如果你想让redis来支撑10万+的QPS，那我们应该怎么做呢？

如果我们假设一个redis有3万读QPS，那么有4个redis，就可以达到10万+了。那我们如何来做呢？这时候就会想到redis的主从架构。

1、主从架构
什么是主从架构？即一个master redis和多个slave redis，master会把数据复制给slave。master负责写，slave负责读。所以又可以叫做：读写分离。slave可以动态扩容，支持更多的读QPS。
![](http://img.chaiguanxin.com//file/2018/10/35a2aab3ad7f4fcd93c811fe75a6c5c8_image.png)
注意：master一定要做持久化，不能把slave作为master的备份；

2、redis主从架构核心原理
![](http://img.chaiguanxin.com//file/2018/10/4d39832a066242f1951eb52b5d607e8e_image.png)

3、redis主从复制断点续传
![](http://img.chaiguanxin.com//file/2018/10/bd930ab789c24aa6932db8cda59314a6_image.png)

4、过期key处理
slave不会过期key，只会等待master过期key，如果master过期了一个key，或者通过LRU淘汰了一个key，那么会模拟一条del命令发送给slave。