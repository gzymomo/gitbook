# 一：资源配置

![图片](https://mmbiz.qpic.cn/mmbiz_png/TwK74MzofXdUTrGC18h28hLrfURQok3QO5KIWuw4ajKBhZibJxV8icSmawUuY5E7cdyJmdmnDbPbdeWqcushhQ6Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

一般企业中，物理机器的cpu:内存基本上都是1:4+，比如机器24core，一般有128GB及以上内存；48core，一般有256GB及以上内存。

减去系统及hdfs所需core，2个吧；减去系统的2-4GB，减去存储hdfs的相关的假设20GB吧(hbase需要的更多点，但是一般hbase会有独立集群)。

24core，128GB的机器应该还剩还有20core，100GB。

这种情况下，很明显，你1core对应5GB内存才能最大化利用机器，否则往往cpu没了，内存还有大把，应用还容易出现gc问题。

所以，假设你的业务cpu确实消耗比较少，可以在配置yarn的时候虚拟cpu可以设置为物理cpu的n倍。这样才可以满足小白，1:1的分配cpu内存的需求。

```
yarn.nodemanager.resource.cpu-vcores 
```

# 二：数据倾斜

![图片](https://mmbiz.qpic.cn/mmbiz_png/TwK74MzofXdUTrGC18h28hLrfURQok3QPaLV6nOfvYb8dticEZQBuXZo1V9oqx846EXcem8JzDItn4hUPXwbYKA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/TwK74MzofXdUTrGC18h28hLrfURQok3Q3VDTy4fB3wCjEm2fJ8bRiatC3KSs6BLtHVZ0UItNo2bM2nzImicut7EA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/TwK74MzofXdUTrGC18h28hLrfURQok3QibQHvgibJXG7XicexnpJWULtdiaKichPibeBjjQk4CiceyYc2ST7MlFeLibohQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

1.对于离线，任务生成文件的时候，可以采用支持分割的压缩算法。常见的压缩算法如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/TwK74MzofXdUTrGC18h28hLrfURQok3QdXhMwmZpXs91huwBYB2DoFedPn4kwRbM9PZttKPaAxnsrnicEp0PARQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

对于离线，任务生成文件的时候，也可以合理控制生成文件的大小，落地之前进行均衡。

2.对于实时，生产者生产数据到kafka的时候，进行均衡，一般可以采取随机或者轮训的生产。假设要保证相同用户的数据到相同的分区，也可以对实用hash的方式。

![图片](https://mmbiz.qpic.cn/mmbiz_png/TwK74MzofXdUTrGC18h28hLrfURQok3QDiaRU5AJWcVTrapqxVC1DEL4VibPBUEOAsJZDOWnQfn1KoFMVsPjh9Og/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

1.遇到shuffle过程中数据倾斜的时候，往往需要做的第一件事情就是增大shuffle并行度，只要不是key过于集中与某一个或者几个key都会有效。

2.假设key过于集中，有以下几个策略可以用来改善：

2.1 先跑采样wordcount，看看key的集中情况。

2.2 个别key特别大，可以拿出来单独处理。

2.2 一堆key特别大，可以如下操作：

key加随机后缀，分两次进行聚合，比如随机后缀的范围是0-100，相当于并行度扩大了100倍，最后再聚合一次即可。

2.3 join的时候key倾斜。

- 大表 join 小表，广播小表。
- 大表 join 大表，预先治理，采用相同分桶策略，使得同一个task完成join，减少数据shuffle传输。后面案例详解。

# 三：大状态

![图片](https://mmbiz.qpic.cn/mmbiz_png/TwK74MzofXdUTrGC18h28hLrfURQok3Q743p714fjh7dl4rezbarwiboQoAJibbia7kIYrpROyNibicYGVbA6ZRCZfw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

1.用户自己管理的大状态。

  1.1 超时，可以使用具有超时功能的超时的map，如caffeine。

  1.2 确认状态是不是可以进行分区缓存。

  1.3 换外部内存存储，如redis，alluxio。

2.updateStateBykey等算子。

  2.1 设置超时时间，spark新版本都支持了，之前版本需要自己维护超时时间。

  2.2 换外部内存存储，如redis，alluxio。

# 四：大对象

![图片](https://mmbiz.qpic.cn/mmbiz_png/TwK74MzofXdUTrGC18h28hLrfURQok3Ql2m3zPic7goHxYG4FYfg4Uj1YxqRNXt0SBsUa54JtlLlBXiaSQLBkshw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如下面一段代码：

```java
distData.foreach(each=>{
      val map = new mutable.HashMap[String,Int]();
      map.put("key",2)
      // 逻辑处理
      map.clear(); // 不要忘了哦
    })
```

假设处理的数据量大，每条数据区new map而不执行clear清空，等垃圾回收，数据量大的时候，就会超级耗费性能，java或者scala的集合都要排查下。