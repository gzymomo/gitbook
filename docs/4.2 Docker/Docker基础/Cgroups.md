- [Cgroups](https://www.yuque.com/duduniao/docker/iu3nb1)



# 1. CPU 资源限制

默认情况下，Docker Container 对宿主机的 CPU 资源的使用是没有限制的，这在大多数场景中是不合理的，因为容器异常时可能会占用抢占其它容器的 CPU 资源，因此需要对 CPU 资源进行限制。

| Options                 | Description                                                  |
| ----------------------- | ------------------------------------------------------------ |
| **-c\|--cpu-share int** | 指定容器可使用的CPU核心时间片的权重，当多个容器争抢同一个核心时，按权重分配。默认值1024 |
| **--cpus=float**        | 指定容器可以使用的CPU核心数(逻辑上)，可以是浮点数。          |
| **--cpuset-cpus**       | 将容器和指定的CPU核心进行绑定                                |
| --cpu-period            | 设定单颗CPU核心的时间片，需要配合 --cpu-quota。范围1000μs~1000000μs。建议使用 --cpus 替代 |
| --cpu-quota             | 指定容器能使用 --cpu-period 中的时间片数量。该参数不建议使用，尽可能使用 --cpus 替代 |

## 1.1. CPU share

[root@centos-82 ~]# docker container run --name t1 --cpuset-cpus 0,1 --rm -d polinux/stress stress -c 4 -t 120 # Default is 1024.

[root@centos-82 ~]# docker container run --name t2 --cpuset-cpus 0,1 --cpu-shares=2048 --rm -d polinux/stress stress -c 4 -t 120

[root@centos-82 ~]# docker stats

```bash
## cpu_t1:cpu_t2 = 1024:2048 = 1:2 ≈ 66:133
CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT   MEM %               NET I/O             BLOCK I/O           PIDS
93d2f5fcf4da        t2                  133.52%             136KiB / 15.5GiB    0.00%               578B / 0B           0B / 0B             5
b15d2d2ae941        t1                  66.40%              80KiB / 15.5GiB     0.00%               648B / 0B           0B / 0B             5
```

## 1.2. CPU period and CPU quota

```bash
[root@centos-82 ~]# docker container run --name t1 --rm -d --cpu-period 1000000 --cpu-quota 500000 polinux/stress:latest stress -c 4 -t 60 ## 50% CPU cycles

[root@centos-82 ~]# docker stats
CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT   MEM %               NET I/O             BLOCK I/O           PIDS
806f2e493a1d        t1                  50.18%              84KiB / 15.5GiB     0.00%               578B / 0B           0B / 0B             5
```

## 1.3. CPUs

```bash
[root@centos-82 ~]# docker container run --name t1 --rm -d polinux/stress stress -c 4 -t 60  ## Unlimit cpus.

[root@centos-82 ~]# docker stats
CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT   MEM %               NET I/O             BLOCK I/O           PIDS
779b4ba5c080        t1                  400.26%             320KiB / 15.5GiB    0.00%               578B / 0B           0B / 0B             5
```

```bash
[root@centos-82 ~]# docker container run --name t1 --rm -d --cpu-period 1000000 --cpu-quota 2000000 polinux/stress:latest stress -c 4 -t 60 ## 200% CPU cycles

[root@centos-82 ~]# docker stats
CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT   MEM %               NET I/O             BLOCK I/O           PIDS
4a7f8bc4b3a8        t1                  199.61%             204KiB / 15.5GiB    0.00%               578B / 0B           0B / 0B             5
```





# 2. Mem 资源限制

需要注意的是，CPU 属于可压缩性的资源，即使 CPU 使用率达到 100% 也不会轻易宕机或者杀进程，只会导致其它容器无法有效的运行任务。但是内存属于不可压缩性资源，对于内存泄漏、垃圾回收不及时等容器，会占用大量的内存空间，而且系统无法释放这部分内存时，容易导致 OOM 甚至宕机。

| Options              | Description                                                  |
| -------------------- | ------------------------------------------------------------ |
| -m\|--memory         | 设置容器最大可用内存大小，不低于 4M                          |
| --memory-reservation | 设置可用内存大小的软限制                                     |
| --memory-swap        | 设置 swap 可使用的情况，这个值并不是直接限制容器可用的 swap 大小 |
| --oom-kill-disable   | 在使用了内存限制的容器上才能使用，用于避免容器被 OOM         |
| --memory-swappiness  | 设置 swappiness 的值，默认和操作系统中的一致                 |
| --kernel-memory      | 最大可用的内核内存的大小，不低于 4M                          |



--memory 和 --memory-swap 直接的关系

| Options                     | Memory Size | Swap Size |
| --------------------------- | ----------- | --------- |
| --memory M --memory-swap S  | M           | S-M       |
| --memory M --memory-swap M  | M           | 0         |
| --memory M --memory-swap 0  | M           | Unset(2M) |
| --memory M --memory-swap -1 | M           | Unlimit   |
| --memory M                  | M           | Unset(2M) |

```bash
[root@centos-82 ~]# docker container run --name t1 --rm --memory 1024m --memory-swap 1024m  polinux/stress:latest stress -m 5 -t 60 ## 禁用swap

[root@centos-82 ~]# dmesg  | grep -i 'out of memory'
[ 2480.558688] Memory cgroup out of memory: Kill process 4921 (stress) score 239 or sacrifice child

[root@centos-82 ~]# docker container run --name t1 --rm --memory 1024m --memory-swap 2048m  polinux/stress:latest stress -m 5 -t 60
[root@centos-82 ~]# docker stats ## Limit total memory 2048m and swap size is 1024m.
CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT   MEM %               NET I/O             BLOCK I/O           PIDS

598431c16b9d        t1                  328.43%             1024MiB / 1GiB      99.98%              578B / 0B           1.16GB / 2.32GB     6

[root@centos-82 ~]# docker container exec t1 free -m ## The memeory info is host system.
             total       used       free     shared    buffers     cached

Mem:         15869       1670      14198          0          2        279

-/+ buffers/cache:       1388      14480

Swap:         4095        482       3613
```



# 3. I/O 资源限制

一般情况下，不会对容器的 I/O 读写进行限制，I/O 读写限制会造成严重的性能问题。

| Options             | Description                                                  |
| ------------------- | ------------------------------------------------------------ |
| --device-write-bps  | Limit write rate (bytes per second) to a device.The size data of write to device in one second. |
| --device-write-iops | Limit write rate (IO per second) to a device.The counts of write to device in one second. |
| --device-read-bps   | Limit read rate   (bytes per second) to a device.The size data of read   to device in one second. |
| --device-read-iops  | Limit read rate   (IO per second) to a device.The counts of read to   device in one second. |

### 3.1. 限制 bps

```bash
[root@centos-76 ~]# docker run -dit --name io-2 -v /var/www/html/:/var/www/html --device /dev/sda:/dev/sda --device-write-bps /dev/sda:100kb centos:gtapp /bin/bash

[root@centos-76 ~]# docker exec e5199c2738 /bin/bash -c "time dd if=/dev/sda of=/var/www/html/io-2.file count=5 bs=1M oflag=direct,nonblock"
## limit 100 kb
5+0 records in
5+0 records out
5242880 bytes (5.2 MB) copied, 56.382 s, 93.0 kB/s # real speed is 93kb/s

real    0m56.385s
user    0m0.000s
sys 0m0.008s
```

### 3.2. 限制 ops

```bash
[root@centos-76 ~]# docker run -dit --name io-3 -v /var/www/html/:/var/www/html --device /dev/sda:/dev/sda --device-write-iops /dev/sda:10 centos:gtapp /bin/bash

[root@centos-76 ~]# docker exec 9946e38b6c171 /bin/bash -c "time dd if=/dev/sda of=/var/www/html/io-3.file count=1000 bs=4k oflag=direct,nonblock"

## limit 10 ops/s --> 10 * 4k = 40 kb/s
1000+0 records in
1000+0 records out
4096000 bytes (4.1 MB) copied, 100.364 s, 40.8 kB/s

real    1m40.367s
user    0m0.004s
sys 0m0.420s
```