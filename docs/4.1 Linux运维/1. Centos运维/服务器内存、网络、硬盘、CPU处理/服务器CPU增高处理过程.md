[TOC]

- [Linux下如何查看哪些进程占用的CPU内存资源最多](https://www.cnblogs.com/sparkbj/p/6148817.html)



# 一、服务器CPU增高排查处理思路

## 1.1 CPU增高原因

CPU增高可能受到的原因有：业务逻辑问题(死循环)、频繁gc以及上下文切换过多，最常见的往往是业务逻辑(或者框架逻辑)导致的。

### 使用CPU最多的5个进程

```bash
ps -aux | sort -k3nr | head 5
```

或者

```bash
top （然后按下P，注意大写）
```



## 1.2 使用jstack分析CPU问题

1. 先用`ps`命令找到对应进程的 Pid(如果有好几个目标进程，可以先用`top`看一下哪个占用比较高)。

2. 接着用`top -H -p pid`来找到CPU使用率比较高的一些线程。

![img](https://mmbiz.qpic.cn/mmbiz_png/QCu849YTaINAdEbfiaQHfnicbVU7B4Z06uCrg5SGW57dg5j1wXWGmFOFVo2mZSreeYLO9BKSMQyPRgsibgWpKG8lQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

3. 将占用最高的pid转换为16进制`printf '%x\n' pid`得到nid。

eg：`printf '%x\n 66'`

![img](https://mmbiz.qpic.cn/mmbiz_png/QCu849YTaINAdEbfiaQHfnicbVU7B4Z06u6ZLg8QMIUtAWAyQSicfBpmYia22CzsYAC6oHicvuVozbyQCPXLfSlaTwA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

4. 在jstack中找到相应的堆栈信息`jstack pid |grep 'nid' -C5 –color`。

![img](https://mmbiz.qpic.cn/mmbiz_png/QCu849YTaINAdEbfiaQHfnicbVU7B4Z06uKdykOWtm9icwPxWaiaHhykzHVD7YhpFnN8oQT9Bru31dwAnOUaGvnZsg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

找到了nid为0x42的堆栈信息，接着只要仔细分析一番即可。



> 更常见的是我们对整个jstack文件进行分析，通常我们会比较关注WAITING和TIMED_WAITING的部分。
>
> 使用命令`cat jstack.log | grep "java.lang.Thread.State" | sort -nr | uniq -c`来对jstack的状态有一个整体的把握，如果WAITING 之类的特别多，那么多半是有问题。

![img](https://mmbiz.qpic.cn/mmbiz_png/QCu849YTaINAdEbfiaQHfnicbVU7B4Z06ubnpbXHHAj0XNhoTbYxauCNAqqSUFU3JCPVexGE4skjqWLPiajlWRPuw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



# 二、频繁gc

先确定下gc是不是太频繁，使用`jstat -gc pid 1000`命令来对gc分代变化情况进行观察，1000表示采样间隔(ms)，S0C/S1C、S0U/S1U、EC/EU、OC/OU、MC/MU分别代表两个Survivor区、Eden区、老年代、元数据区的容量和使用量。YGC/YGT、FGC/FGCT、GCT则代表YoungGc、FullGc的耗时和次数以及总耗时。

如果看到gc比较频繁，再针对gc方面做进一步分析。

![img](https://mmbiz.qpic.cn/mmbiz_png/QCu849YTaINAdEbfiaQHfnicbVU7B4Z06ul99HdrIu1hM1jWbOffZDykq6lYibic78iaDly50QfA9oQ963I4sRKIQHQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



# 三、上下文切换

针对频繁上下文问题，我们可以使用`vmstat`命令来进行查看

![img](https://mmbiz.qpic.cn/mmbiz_png/QCu849YTaINAdEbfiaQHfnicbVU7B4Z06u2h1M7uicicSfdziaXfdQ6GWpa924Vib4sBicfaeeIM7x7aRqxQpjptuFlWg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**cs(context switch)一列则代表了上下文切换的次数。**如果我们希望对特定的pid进行监控那么可以使用 `pidstat -w pid`命令，cswch和nvcswch表示自愿及非自愿切换。

![img](https://mmbiz.qpic.cn/mmbiz_png/QCu849YTaINAdEbfiaQHfnicbVU7B4Z06uzJjQbU2LG32nzfTzHE7r2VWyOMRlkuEv9Q3h8icAUCd5BmHhV4gf4ug/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

