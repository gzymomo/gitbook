- [数据库没有备份，没有使用Binlog的情况下，如何恢复数据？](https://ibyte.blog.csdn.net/article/details/108433355)



1. InnoDB 存储引擎中的表空间是怎样的？两种表空间存储方式各有哪些优缺点？
2. 如果.ibd 文件损坏了，数据该如何找回？
3. 如何模拟 InnoDB 文件的损坏与数据恢复？

# InnoDB 存储引擎的表空间

InnoDB 存储引擎的文件格式是.ibd  文件，数据会按照表空间（tablespace）进行存储，分为共享表空间和独立表空间。如果想要查看表空间的存储方式，我们可以对innodb_file_per_table变量进行查询，使用show variables like 'innodb_file_per_table';。ON 表示独立表空间，而 OFF 则表示共享表空间。

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL3N6X21tYml6X3BuZy9kN1l6YVlEbnJ4SDZjUTFkcnhGVFBMYjJjZDNhWmVucE1UZkNpY0lwcXYyc2R4YXFXSTZ4OVE3Qk5kVDF2OFYyZ2liNFRMYWg2RGhtVWZPd21yME1sT3F3LzY0MA?x-oss-process=image/format,png)

如果采用共享表空间的模式，InnoDB  存储的表数据都会放到共享表空间中，也就是多个数据表共用一个表空间，同时表空间也会自动分成多个文件存放到磁盘上。这样做的好处在于单个数据表的大小可以突破文件系统大小的限制，最大可以达到 64TB，也就是 InnoDB  存储引擎表空间的上限。不足也很明显，多个数据表存放到一起，结构不清晰，不利于数据的找回，同时将所有数据和索引都存放到一个文件中，也会使得共享表空间的文件很大。

采用独立表空间的方式可以让每个数据表都有自己的物理文件，也就是 table_name.ibd  的文件，在这个文件中保存了数据表中的数据、索引、表的内部数据字典等信息。它的优势在于每张表都相互独立，不会影响到其他数据表，存储结构清晰，利于数据恢复，同时数据表还可以在不同的数据库之间进行迁移。

# 如果.ibd 文件损坏了，数据如何找回

如果我们之前没有做过全量备份，也没有开启 Binlog，那么我们还可以通过.ibd 文件进行数据恢复，采用独立表空间的方式可以很方便地对数据库进行迁移和分析。如果我们误删除（DELETE）某个数据表或者某些数据行，也可以采用第三方工具回数据。

我们这里可以使用 Percona Data Recovery Tool for InnoDB 工具，能使用工具进行修复是因为我们在使用  DELETE 的时候是逻辑删除。我们之前学习过 InnoDB 的页结构，在保存数据行的时候还有个删除标记位，对应的是页结构中的  delete_mask 属性，该属性为 1  的时候标记了记录已经被逻辑删除，实际上并不是真的删除。不过当有新的记录插入的时候，被删除的行记录可能会被覆盖掉。所以当我们发生了 DELETE  误删除的时候，一定要第一时间停止对误删除的表进行更新和写入，及时将.ibd 文件拷贝出来并进行修复。

如果已经开启了 Binlog，就可以使用闪回工具，比如 mysqlbinlog 或者  binlog2sql，从工具名称中也能看出来它们都是基于 Binlog 来做的闪回。原理就是因为 Binlog  文件本身保存了数据库更新的事件（Event），通过这些事件可以帮我们重现数据库的所有更新变化，也就是 Binlog 回滚

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL3N6X21tYml6X3BuZy9kN1l6YVlEbnJ4SDZjUTFkcnhGVFBMYjJjZDNhWmVucHc0NWpRWmRiaWMxb2VubmNkaWNtcXdUQ1BYTU9vaWJFaWFyeDlRZ1VISmlhNVNHZ1htMkdHZTlwZ0ZnLzY0MA?x-oss-process=image/format,png)

innodb_force_recovery参数一共有 7 种状态，除了默认的 0 以外，还可以为 1-6 的取值，分别代表不同的强制恢复措施。

当我们需要强制恢复的时候，可以将innodb_force_recovery设置为 1，表示即使发现了损坏页也可以继续让服务运行，这样我们就可以读取数据表，并且对当前损坏的数据表进行分析和备份。

通常innodb_force_recovery参数设置为 1，只要能正常读取数据表即可。但如果参数设置为 1  之后还无法读取数据表，我们可以将参数逐一增加，比如 2、3 等。一般来说不需要将参数设置到 4  或以上，因为这有可能对数据文件造成永久破坏。另外当innodb_force_recovery设置为大于 0 时，相当于对 InnoDB  进行了写保护，只能进行 SELECT 读取操作，还是有限制的读取，对于 WHERE 条件以及 ORDER BY 都无法进行操作。

当我们开启了强制恢复之后，数据库的功能会受到很多限制，我们需要尽快把有问题的数据表备份出来，完成数据恢复操作。整体的恢复步骤可以按照下面的思路进行：

1. 使用innodb_force_recovery启动服务器

   将innodb_force_recovery参数设置为 1，启动数据库。如果数据表不能正常读取，需要调大参数直到能读取数据为止。通常设置为 1 即可。

2. 备份数据表
    在备份数据之前，需要准备一个新的数据表，这里需要使用 MyISAM 存储引擎。原因很简单，InnoDB 存储引擎已经写保护了，无法将数据备份出来。然后将损坏的 InnoDB 数据表备份到新的 MyISAM 数据表中。

3. 删除旧表，改名新表
    数据备份完成之后，我们可以删除掉原有损坏的 InnoDB 数据表，然后将新表进行改名。

4. 关闭innodb_force_recovery，并重启数据库
    innodb_force_recovery大于 1 的时候会有很多限制，我们需要将该功能关闭，然后重启数据库，并且将数据表的 MyISAM 存储引擎更新为 InnoDB 存储引擎。