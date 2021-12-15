# [redis压力测试工具-----redis-benchmark](https://www.cnblogs.com/williamjie/p/11303965.html)

redis做压测可以用自带的redis-benchmark工具，使用简单

![img](https://images2017.cnblogs.com/blog/707331/201802/707331-20180201145415093-631633064.png)

压测命令：**redis-benchmark -h 127.0.0.1 -p 6379 -c 50 -n 10000**

**![img](https://images2017.cnblogs.com/blog/707331/201802/707331-20180201145503750-901697180.png)**

压测需要一段时间，因为它需要依次压测多个命令的结果，如：get、set、incr、lpush等等，所以我们需要耐心等待，如果只需要压测某个命令，如：get，那么可以在以上的命令后加一个参数-t（红色部分）：

# **1、redis-benchmark -h 127.0.0.1 -p 6086 -c 50 -n 10000 -t get**

```bash
 C:\Program Files\Redis>redis-benchmark -h 127.0.0.1 -p 6379 -c 50 -n 10000 -t get

====== GET ======
10000 requests completed in 0.16 seconds
50 parallel clients
3 bytes payload
keep alive: 1

99.53% <= 1 milliseconds
100.00% <= 1 milliseconds
62893.08 requests per second
```



# **2、redis-benchmark -h 127.0.0.1 -p 6086 -c 50 -n 10000 -t set**

```bash
C:\Program Files\Redis>redis-benchmark -h 127.0.0.1 -p 6379 -c 50 -n 10000 -t set

====== SET ======
10000 requests completed in 0.18 seconds
50 parallel clients
3 bytes payload
keep alive: 1

87.76% <= 1 milliseconds
99.47% <= 2 milliseconds
99.51% <= 7 milliseconds
99.74% <= 8 milliseconds
100.00% <= 8 milliseconds
56179.77 requests per second
```

这样看起来数据很多，如果我们只想看最终的结果，可以带上参数-q，完整的命令如下：

# **3、redis-benchmark -h 127.0.0.1 -p 6379 -c 50 -n 10000 -q**

```bash
C:\Program Files\Redis>redis-benchmark -h 127.0.0.1 -p 6379 -c 50 -n 10000 -q
PING_INLINE: 63291.14 requests per second
PING_BULK: 62500.00 requests per second
SET: 49261.09 requests per second
GET: 47619.05 requests per second
INCR: 42194.09 requests per second
LPUSH: 61349.69 requests per second
RPUSH: 56818.18 requests per second
LPOP: 47619.05 requests per second
RPOP: 45045.04 requests per second
SADD: 46296.30 requests per second
SPOP: 59523.81 requests per second
LPUSH (needed to benchmark LRANGE): 56818.18 requests per second
LRANGE_100 (first 100 elements): 32362.46 requests per second
LRANGE_300 (first 300 elements): 13315.58 requests per second
LRANGE_500 (first 450 elements): 10438.41 requests per second
LRANGE_600 (first 600 elements): 8591.07 requests per second
MSET (10 keys): 55248.62 requests per second
```

