https://blog.51cto.com/14994509/2637203



## **数据页**

在操作系统中，我们知道为了跟磁盘交互，内存也是分页的，一页大小4KB。同样的在MySQL中为了提高吞吐率，数据也是分页的，不过MySQL的数据页大小是16KB。（确切的说是InnoDB数据页大小16KB）。详细学习可以参考官网 我们可以用如下命令查询到。

```javascript
mysql> SHOW GLOBAL STATUS LIKE 'innodb_page_size';
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| Innodb_page_size | 16384 |
+------------------+-------+
1 row in set (0.00 sec)
```

今天咱们数据页的具体结构指针等不深究，知道它默认是16kb就行了，也就是说一个节点的数据大小是16kb

## **索引结构(innodb)**

mysql的索引结构咱们应该都知道，是如下的b+树结构

![image](https://upload-images.jianshu.io/upload_images/24630328-a633c7a5681673a3.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通常b+树非叶子节点不存储数据，只有叶子节点(最下面一层)才存储数据，那么咱们说回节点，一个节点指的是(对于上图而言)

![image](https://upload-images.jianshu.io/upload_images/24630328-8736213a8046273b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

每个红框选中的部分称为一个节点，而不是说某个元素。 了解了节点的概念和每个节点的大小为16kb之后，咱们计算mysql能存储多少数据就容易很多了

# **具体计算方法**

## **根节点计算**

首先咱们只看根节点

比如我们设置的数据类型是bigint，大小为8b

![image](https://upload-images.jianshu.io/upload_images/24630328-6b1d5a34f3033751.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在数据本身如今还有一小块空间，用来存储下一层索引数据页的地址，大小为6kb

![image](https://upload-images.jianshu.io/upload_images/24630328-0d9b4fefb13e3dd6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

所以我们是可以计算出来一个数据为(8b+6b=14b)的空间（以bigint为例）  我们刚刚说到一个数据页的大小是16kb,也就是(16_1024)b，那么根节点是可以存储(16_1024/(8+6))个数据的，结果大概是1170个数据 如果跟节点的计算方法计算出来了，那么接下来的就容易了。

## **其余层节点计算**

第二层其实比较容易，因为每个节点数据结构和跟节点一样，而且在跟节点每个元素都会延伸出来一个节点，所以第二层的数据量是1170*1170=1368900，问题在于第三层，因为innodb的叶子节点，是直接包含整条mysql数据的，如果字段非常多的话数据所占空间是不小的，我们这里以1kb计算，所以在第三层，每个节点为16kb，那么每个节点是可以放16个数据的，所以最终mysql可以存储的总数据为

1170 * 1170 * 16 = 21902400 (千万级条)

其实计算结果与我们平时的工作经验也是相符的，一般mysql一张表的数据超过了千万也是得进行分表操作了。