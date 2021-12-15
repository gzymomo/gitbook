[TOC]

process exporter在prometheus中用于监控进程，通过process exporter，可从宏观角度监控应用的运行状态（譬如监控redis、mysql的进程资源等）。

# 1、下载安装
下载地址：https://github.com/ncabatoff/process-exporter/releases/tag/v0.4.0
`tar -zxvf process-exporter-0.4.0.linux-amd64.tar.gz -C /usr/local/process-exporter`

# 2、修改配置文件
```yml
process_names:
  - name: "{{.Matches}}"
    cmdline:
    - 'redis'

  - name: "{{.Matches}}
    cmdline:
    - 'mysql'
```
注意：如果一个进程符合多个匹配项，只会归属于第一个匹配的groupname组。
```yml
process_names:

  - name: "{{.Matches}}"
    cmdline:
    - 'redis-server'

  - name: "{{.Matches}}"
    cmdline:
    - 'mysqld'

  - name: "{{.Matches}}"
    cmdline:
    - 'org.apache.zookeeper.server.quorum.QuorumPeerMain'

  - name: "{{.Matches}}"
    cmdline:
    - 'org.apache.hadoop.mapreduce.v2.hs.JobHistoryServer'

  - name: "{{.Matches}}"
    cmdline:
    - 'org.apache.hadoop.hdfs.qjournal.server.JournalNode'
```
注意：cmdline:  所选进程的唯一标识，ps -ef 可以查询到。如果改进程不存在，则不会有该进程的数据采集到。

name选项有几个（官方翻译https://github.com/ncabatoff/process-exporter）：
 - {{.Comm}} 包含原始可执行文件的基本名称，即第二个字段 /proc/<pid>/stat
 - {{.ExeBase}} 包含可执行文件的基名
 - {{.ExeFull}} 包含可执行文件的完全限定路径
 - {{.Username}} 包含有效用户的用户名
 - {{.Matches}} map包含应用cmdline regexps产生的所有匹配项

|   |   |   |
| ------------ | ------------ | ------------ |
| {{.Comm}}   | groupname="redis-server"  | exe或者sh文件名称  |
| {{.ExeBase}}  | groupname="redis-server *:6379"  | /  |
| {{.ExeFull}}  | groupname="/usr/bin/redis-server *:6379"  | ps中的进程完成信息  |
| {{.Username}}  | groupname="redis"  |  使用进程所属的用户进行分组 |
| {{.Matches}}  |  groupname="map[:redis]" | 表示配置到关键字“redis” |

# 3、启动
```bash
./process-exporter -config.path process-name.yaml &
```

查看数据：
```
curl http://localhost:9256/metrics   > ccc
```
![](https://img2018.cnblogs.com/blog/1314872/201902/1314872-20190225161749831-79334163.png)


