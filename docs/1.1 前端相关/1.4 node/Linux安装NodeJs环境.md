[Linux安装NodeJs环境](https://www.cnblogs.com/jasontang369/p/12771335.html)



# 下载nodejs压缩包

> 官网：https://nodejs.org/en/download/

也可以直接使用wget命令下载。

```
wget https://nodejs.org/dist/v14.15.4/node-v14.15.4-linux-x64.tar.xz
```

# 解压下载的压缩包

```
tar -xf node-v14.15.4-linux-x64.tar.xz
```

# 移动解压好的文件

```
mkdir /usr/local/nodejs
sudo mv ./node-v14.15.4-linux-x64/*  /usr/local/nodejs
```

# 建立软链接

```
ln -s /usr/local/nodejs/bin/node /usr/local/bin

ln -s /usr/local/nodejs/bin/npm /usr/local/bin
```

# 测试

```
node -v
npm -v
```