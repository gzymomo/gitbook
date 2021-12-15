### 1、下载对应你系统的node版本并进行安装，这里就不做说明了

说明： 安装完成后用命令node -v 和npm -v查看node和npm版本来检测安装是否成功

### 2、环境配置

说明： 这里主要配置的是npm安装的全局模块所在的路径，以及缓存cache的路径，因为npm install -g模块自动安装到c盘下，为了不占c盘空间，所以我们可以自定义模块所放位置，如下：(这里是把全局路径定义在node安装所在位置)

有两种方法：

#### 方法一：

nodejs安装目录/node_modules/npm/.npmrc这个文件，修改里面的路径

#### 方法二：

1、在node安装的文件夹【D:\software\node】下创建两个文件夹【node_global】及【node_cache】

2、打开命令终端窗口，输入

```
npm config set prefix "D:\software\node\node_global"
npm config set cache "D:\software\node\node_cache"
```

然后设置环境变量：
 1、“我的电脑”-右键-“属性”-“高级系统设置”-“高级”-“环境变量”
 2、在【系统变量】下新建【NODE_PATH】，输入【D:\software\node\node_global\node_modules】，将【用户变量】下的【Path】修改为【D:\software\node\node_global】

### 3、查看所有全局安装模块

```
npm list -g --depth 0
```

### 4、查看全局安装路径

```
npm config ls
```

上面命令执行后看到prefix就是模块全局安装路径