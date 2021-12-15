- [windows下yarn安装、更改全局位置](https://blog.csdn.net/wpp555/article/details/107851163)

# windows下yarn安装、更改全局位置

## 1 下载`注:需要先安装nodejs`

中文官网
 https://classic.yarnpkg.com/zh-Hans/docs/install#windows-stable

- 直接进去下载页面，下载 .msi文件即可 *v1.22.4*
   ![aRnhrQ.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMS5heDF4LmNvbS8yMDIwLzA4LzA2L2FSbmhyUS5wbmc?x-oss-process=image/format,png)
- 点击安装程序一直`next`，可以修改一下安装位置(默认C:\Program Files)
   ![aRusL4.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMS5heDF4LmNvbS8yMDIwLzA4LzA2L2FSdXNMNC5wbmc?x-oss-process=image/format,png)
- 打开新的命令行，执行`yarn -v`，提示 1.22.4，即安装完成

# 2 更改全局位置

- 更改缓存位置

```
yarn cache dir //查看缓存位置
yarn cache clean // 清除缓存,在目录
yarn config set cache-folder D:\Apps\work\Yarn\cache  //设置D盘
yarn cache dir //在输出一下目录 看看缓存位置
```

- 更改全局位置

```
yarn global dir  //查看全局位置
yarn config  set global-folder "D:\Apps\work\Yarn\global"
yarn global dir  //在执行查看位置,已经被修改
```

- 下载模块进行测试

```
# 必须先下载模块,会自动创建 .bin 目录,在向下走
yarn global add yrm //全局安装 
```

- 重新设置bin目录`非常重要一步,设置yarn 全局模块读取目录` `也不知道有没有用`

```
yarn global bin //默认是 c:/,修改到 D:盘
yarn config set prefix D:\YarnCache\global\node_modules\.bin
```

- 上面一步貌似没效果，直接更改用户变量 > Path `系统变量不动`
   ![aRlo9I.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMS5heDF4LmNvbS8yMDIwLzA4LzA2L2FSbG85SS5wbmc?x-oss-process=image/format,png)
- 再次打开新的命令窗口输入 yrm ls
   ![aR1ugx.png](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMS5heDF4LmNvbS8yMDIwLzA4LzA2L2FSMXVneC5wbmc?x-oss-process=image/format,png)
- 结束