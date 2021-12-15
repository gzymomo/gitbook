- [Django+Vue+Docker搭建接口测试平台实战](https://www.cnblogs.com/jinjiangongzuoshi/p/14711858.html)

## 一. 开头说两句

大家好，我叫林宗霖，是一位测试工程师，也是**全栈测开训练营**中的一名学员。

在跟着训练营学习完`Docker`容器技术系列的课程后，理所应当需要通过实操来进行熟悉巩固。正好接口自动化测试平台需要迁移到新的测试服务器上，就想要体验一番`Docker`的“**一次构建，处处运行**”。这篇文章简单介绍了下这次部署的过程，其中使用了`Dockerfile`定制镜像和`Docker-Compose`多容器编排。

## 二. 项目介绍

项目采用的是前后端分离技术来实现的，前端是`Vue+ElementUI`，后端是`Django+DRF`，数据库是`MySQL`，当前部署版本没有其他中间件。

### 2.1 安装docker和docker-compose

> 下述所有操作，皆在`Centos 7`环境下进行

**1.清理或卸载旧版本**：

```bash
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

**2.更新yum库**

```bash
sudo yum install -y yum-utils
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

**3.安装最新版本**

```bash
sudo yum install docker-ce docker-ce-cli containerd.io
```

**4.启动Docker服务**

```bash
sudo systemctl start docker
```

**5.下载docker compose安装包**

采用curl安装的方式比直接用pip安装好处是不怕缺少某些依赖

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.28.6/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

**6.修改docker compose的权限**

```bash
sudo chmod +x /usr/local/bin/docker-compose
```

### 2.2 Dockerfile定制python容器

1. 首先把需要部署的django项目代码放到特定目录下（这里是`/data/test_object`）
2. 把django项目依赖包文件`requirements.txt`也放在该目录下
3. 创建Dockerfile文件：`vim Dockerfile`
4. Dockerfile内容：（注意：注释别跟在语句后面，有些语句执行时会因此出现问题）：

```txt
# 基础镜像
FROM python:3.6.8

# 把输出及时重定向到文件，替代python -u
ENV PYTHONUNBUFFERED 1

# 创建目录并切换工作目录
RUN mkdir /code && mkdir /code/db
WORKDIR /code

# 添加文件
ADD ./requirements.txt /code/

# 执行命令
RUN pip install -r requirements.txt

# 添加文件
ADD . /code/
```

### 2.3 编写Docker Compose容器编排

1. 同样的目录，创建docker-compose.yml文件：`vim docker-compose.yml`，内容（编排Python容器和Mysql容器）

```yml
# docker compose版本
version: "3.9"

# 服务信息
services:

  # mysql容器，名字自定义
  db:
    image: mysql:5.7
    expose:
      - "3306"
    volumes:
      - ./db:/var/lib/mysql
    #设置数据库表的数据集
    command: [
      '--character-set-server=utf8',
      '--collation-server=utf8_unicode_ci'
      ]
    environment:
      - MYSQL_DATABASE=xxxx
      - MYSQL_ROOT_PASSWORD=yyyy
    restart: always


  # django服务
  web:
    # 基于本路径的Dockerfile创建python容器
    build: .
    command: bash -c "python ./test_plat_form/manage.py migrate && python ./test_plat_form/manage.py runserver 0.0.0.0:8000"
    volumes:
      - .:/code
    ports:
      - "8000:8000"
    expose:
      - "8000"
    # 当前服务所依赖的服务，会先启动依赖服务再启动当前服务
    depends_on:
      - db
    # 容器ip是可变的，替代配置文件中mysql的HOST的值；名字和上面的mysql容器服务的名字一致
    links:
      - db
    volumes:
      - ./files/suites:/code/test_plat_form/suites
      - ./files/debugs:/code/test_plat_form/debugs
      - ./files/reoprts:/code/test_plat_form/reports
      - ./files/run_log:/code/test_plat_form/run_log
```

修改django项目setting.py文件中的mysql的host，改成上面web节点中links的值

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'xxxx',
        'USER': 'root',
        'PASSWORD': 'yyyy',
        'HOST': 'db',  # 这里进行修改
        'PORT': 3306
    }
}
```

**执行命令**

所在路径：和Dockerfile等文件同个路径下
 构建容器：`docker-compose build`
 运行容器：`docker-compose up` 或者 后台运行容器：`docker-compose up -d`

### 2.4 Vue项目的搭建

vue使用传统的搭建方式即可:

- 服务器配置node npm环境
- 安装全局pm2
- 修改项目中api的host为服务器的ip或域名
- 打包vue项目：`npm run build`
- 编写个`app.js`启动脚本，主要目的是是读取dist目录下的单页面文件(index.js)，监听8080端口

```js
const fs = require('fs');
const path = require('path');
const express = require('express');
const app = express();

app.use(express.static(path.resolve(__dirname, './dist')))
//读取目录下的单页面文件(index.js)，监听8080端口。
app.get('*', function(req, res) {
    const html = fs.readFileSync(path.resolve(__dirname, './dist/index.html'), 'utf-8')
    res.send(html)
})

app.listen(8080);
```

1. 把打包好的dist目录、app.js、package.json复制到项目目录下
2. 进入项目目录，安装依赖：`npm install`
3. 启动服务：`pm2 start app.js`

### 5、最终效果

运行容器日志：
 ![容器启动日志](https://tva1.sinaimg.cn/large/008i3skNgy1gprh2yxwq7j31c90mk140.jpg)

浏览器访问`http://ip:8080`并登录：

![接口测试平台](https://tva1.sinaimg.cn/large/008i3skNgy1gprh3kmywuj31hc0o3n0d.jpg)

## 三、总结

这个项目组成目前还比较简单，只用了2个容器进行编排。但是以此为例，在搭建更多容器时，我们首先根据项目组成定制不同的容器，然后规划好容器之间的是组织关系和依赖关系，相信也是能顺利搭建起来的。