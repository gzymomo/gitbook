[解决ffmpeg执行报错“ffmpeg: error while loading shared libraries: libavdevice.so.58: cannot open shared object file: No such file or directory”的问题](https://www.cnblogs.com/comexchan/p/12079333.html)

问题现象：

执行ffmpeg命令后报错：

```
ffmpeg: error while loading shared libraries: libavdevice.so.58: cannot open shared object file: No such file or directory
```

![img](https://img2018.cnblogs.com/blog/897329/201912/897329-20191222113219159-1195201996.png)

 出问题的环境信息为：

```
Fedora release 31 (Thirty One)
ffmpeg-4.2.1 官方源码编译
```

 看下需要哪些依赖：

```
ldd ffmpeg
```

可以看到缺失的依赖

![img](https://img2018.cnblogs.com/blog/897329/201912/897329-20191222114228387-1230292697.png)

我们找下这些文件在哪里

```
find /usr -name 'libavdevice.so.58'
```

应该都在这个目录

```
/usr/local/lib/
```

![img](https://img2018.cnblogs.com/blog/897329/201912/897329-20191222115339130-1817590757.png)

 我们export出来：

```
export LD_LIBRARY_PATH=/usr/local/lib/
```

然后再尝试执行

```
/usr/local/bin/ffmpeg
```

![img](https://img2018.cnblogs.com/blog/897329/201912/897329-20191222115538098-1317710451.png)

 问题解决