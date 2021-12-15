

# 一、Ansible介绍

Ansible是当下比较流行的自动化运维工具，可通过SSH协议对远程服务器进行集中化的配置管理、应用部署等，常结合Jenkins来实现自动化部署。

除了Ansible，还有像SaltStack、Fabric（曾经管理100多台服务器上的应用时也曾受益于它）、Puppet等自动化工具。相比之下，Ansible最大的优势就是无需在被管理主机端部署任何客户端代理程序，通过SSH通道就可以进行远程命令的执行或配置的下发，足够轻量级，但同时功能非常强大，且各项功能通过模块来实现，具备良好的扩展性。不足之处是Ansible只支持在Linux系统上安装，不支持Windows。

如果你需要在多于一台服务器上做相同的操作，那么建议你使用Ansible之类的自动化工具，这将极大提高你的操作效率。



# 二、Ansible环境搭建

**1. 找一台主机用于做管理服务器，在其上安装Ansible**

```bash
yum -y install ansible
```

Ansible基于Python实现，一般Linux系统都自带Python，所以可以直接使用yum安装或pip安装。

安装完后，在/etc/ansible/目录下生成三个主要的文件或目录，

```bash
[root@tool-server ~]# ll /etc/ansible/
total 24
-rw-r--r--. 1 root root 19179 Jan 30  2018 ansible.cfg
-rw-r--r--. 1 root root  1136 Apr 17 15:17 hosts
drwxr-xr-x. 2 root root     6 Jan 30  2018 roles
```

- ansible.cfg：Ansible的配置文件
- hosts：登记被管理的主机
- roles：角色项目定义目录，主要用于代码复用

**2. 在/etc/ansible/hosts文件中添加需要被管理的服务器节点**

```bash
[root@tool-server ~]# vim /etc/ansible/hosts
[k8s]
192.168.40.201
192.168.40.202
192.168.40.205
192.168.40.206
```

`[k8s]`表示将下面的服务器节点分到k8s的组中，后面执行命令时可指定针对某个组执行。

**3. 生成SSH KEY，并copy到被管理节点上，实现免密SSH访问**

在管理节点执行 `ssh-keygen` 生成SSH KEY，然后copy到各被管理节点上

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.40.201
```

上面命令将`~/.ssh/id_rsa.pub`文件内容添加到被管理节点的/root/.ssh/authorized_keys文件中，实现管理节点到被管理节点的免密SSH访问。

**4. 调试Ansible**

针对k8s服务器组执行ping，验证Ansible到各被管理节点的连通性

```bash
[root@tool-server ~]# ansible k8s -m ping
192.168.40.201 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
192.168.40.205 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
192.168.40.202 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
192.168.40.206 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

Ansible只需要在管理主机上安装，然后打通管理主机到各被管理主机的SSH免密访问即可进行集中化的管理控制，不需在被管理主机安装任何代理程序。

# 三、Ansible命令

Ansible的命令格式为， `ansible 主机群组名 -m 命令模块名 -a "批量执行的操作"`

其中-m不是必须的，默认为command模块，-a也不是必须的，表示命令模块的参数，比如前面的ping模块就没有参数。

可以使用 `ansible-doc -l` 列出所有可用的命令模块， `ansible-doc -s 模块名` 查看指定模块的参数信息

常用命令模块

## 3.1 command

command是Ansible的默认模块，不指定-m参数时默认使用command。command可以运行远程主机权限范围内的所有shell命令，但不支持管道操作

```bash
# 查看k8s分组主机内存使用情况
ansible k8s -m command -a "free -g"
```

## 3.2 shell

shell基本与command相同，但shell支持管道操作

```bash
#shell支持管道操作 |grep Mem
ansible k8s -m shell -a "free -g|grep Mem"
```

## 3.3 script

script就是在远程主机上执行管理端存储的shell脚本文件，相当于scp+shell

```bash
# /root/echo.sh为管理端本地shell脚本
ansible k8s -m script -a "/root/echo.sh"
```

## 3.4 copy

copy实现管理端到远程主机的文件拷贝，相当于scp

```bash
#拷贝本地echo.sh文件到k8s组中远程主机的/tmp目录下，所属用户、组为 root ，权限为 0755
ansible k8s -m copy -a "src=/root/echo.sh dest=/tmp/ owner=root group=root mode=0755"
```

## 3.5 yum

软件包安装或删除

```bash
ansible k8s -m yum -a "name=wget state=latest"
```

其中state有如下取值：

- 针对安装，可取值“present，installed，latest”，present，installed即普通安装，两者无区别，latest是使用yum mirror上最新的版本进行安装
- 针对删除，可取值“absent，removed”，两者无差别

## 3.6 service

对远程主机的服务进行管理

```bash
ansible k8s -m service -a "name=nginx state=stoped"
```

state可取值“started/stopped/restarted/reloaded”。

## 3.7 get_url

在远程主机上下载指定URL到本地

```bash
ansible k8s -m get_url -a "url=http://www.baidu.com dest=/tmp/index.html mode=0440 force=yes"
```

## 3.8 setup

获取远程主机的信息

```bash
ansible k8s -m setup
```

## 3.9 file

管理远程主机的文件或目录

```bash
ansible k8s -m file -a "dest=/opt/test state=touch"
```

state可取值

- directory：创建目录
- file：如果文件不存在，则创建
- link：创建symbolic link
- absent：删除文件或目录
- touch：创建一个不存在的空文件

## 3.10 cron

管理远程主机的crontab定时任务

```bash
ansible k8s -m cron -a "name='backup servcie' minute=*/5 job='/usr/sbin/ntpdate  time.nist.gov >/dev/null 2>&1'"
```

支持的参数

- state：取值present表示创建定时任务，absent表示删除定时任务
- disabled：yes表示注释掉定时任务，no表示接触注释

# 四、Ansible playbook

Ansible的playbook由一个或多个play组成，play的功能就是为归为一组的主机编排要执行的一系列task，其中每一个task就是调用Ansible的一个命令模块。

playbook的核心元素包括：

- hosts：执行任务的远程主机组或列表
- tasks：要执行的任务列表
- variables：内置变量或自定义的变量
- templates：使用模板语法的文件，通常为配置文件
- handlers：和notify结合使用，由特定条件触发，一般用于配置文件变更触发服务重启
- tags：标签，可在运行时通过标签指定运行playbook中的部分任务
- roles：

playbook文件遵循yaml的语法格式，运行命令的格式为 `ansible-playbook <filename.yml> ... [options]`， 常用options包括

- --syntax 检查playbook文件语法是否正确
- --check 或 -C 只检测可能会发生的改变，但不真正执行操作
- --list-hosts 列出运行任务的主机
- --list-tags 列出playbook文件中定义所有的tags
- --list-tasks 列出playbook文件中定义的所有任务集
- --limit 只针对主机列表中的某个主机或者某个组执行
- -f 指定并发数，默认为5个
- -t 指定某个或多个tags运行（前提playbook中有定义tags）
- -v 显示过程 -vv -vvv更详细

下面以批量安装Nginx为例，尽可能介绍playbook各核心元素的用法。

定义palybook yaml文件nginx_playbook.yml

```bash
---
- hosts: 192.168.40.201,192.168.40.205 # 主机列表，也可以是/etc/ansible/hosts中定义的主机分组名
  remote_user: root # 远程用户
  vars:             # 自定义变量
     version: 1.16.1
  vars_files:
     - ./templates/nginx_locations_vars.yml


  tasks:
     - name: install dependencies          # 定义任务的名称
       yum: name={{item}} state=installed  # 调用模块，具体要做的事情，这里使用with_items迭代多个yum任务安装必要的依赖
       with_items:
          - gcc
          - gcc-c++
          - pcre
          - pcre-devel
          - zlib
          - zlib-devel
          - openssl
          - openssl-devel
     - name: download nginx                # 通过get_url模块下载nginx
       get_url: url=http://nginx.org/download/nginx-{{version}}.tar.gz dest=/tmp/ mode=0755 force=no
     - name: unarchive                     # 通过unarchive模块解压nginx
       unarchive: src=/tmp/nginx-{{version}}.tar.gz dest=/tmp/ mode=0755 copy=no
     - name: configure,make and install    # 通过shell模块执行shell命令编译安装
       shell: cd /tmp/nginx-{{version}} && ./configure --prefix=/usr/local/nginx && make && make install
     - name: start nginx                   # 通过shell模块执行shell命令启动nginx
       shell: /usr/local/nginx/sbin/nginx
     - name: update config                 # 通过template模块动态生成配置文件下发到远程主机目录
       template: src=nginx.conf.j2 dest=/usr/local/nginx/conf/nginx.conf
       notify: reload nginx                # 在结束时触发一个操作，具体操作通过handlers来定义
       tags: reload                        # 对任务定义一个标签，运行时通过-t执行带指定标签的任务

  handlers:
     - name: reload nginx                  # 与notify定义的内容对应
       shell: /usr/local/nginx/sbin/nginx -s reload
```

## 4.1 变量

在上面的示例中使用vars定义了变量version，在tasks中通过{{version}}进行引用。Ansible支持如下几种定义变量的方式

**1.在playbook文件中定义**
前面示例已经说明

**2.命令行指定**
在执行playbook时通过-e指定，如`ansible-playbook -e "version=1.17.9" nginx_playbook.yml`， 这里指定的变量将覆盖playbook中定义的同名变量的值

**3.hosts文件中定义变量**
在/etc/ansible/hosts文件中也可以定义针对单个主机或主机组的变量，如

```bash
[nginx]
192.168.40.201 version=1.17.9 # 定义单个主机的变量
192.168.40.205 
[nginx:vars]  # 定义整个组的统一变量
version=1.16.1
```

**4.在独立的yaml文件中定义变量**
专门定义一个yaml变量文件，然后在playbook文件中通过var_files引用，如

```yaml
# 定义存放变量的文件
[root@ansible ]# cat var.yml
version: 1.16.1
# 编写playbook
[root@ansible ]# cat nginx_playbook.yml
---
- hosts: nginx
  remote_user: root
  vars_files:     # 引用变量文件
    - ./var.yml   # 指定变量文件的path（这里可以是绝对路径，也可以是相对路径）
```

**5.使用setup模块获取到的变量**
前面介绍setup模块可获取远程主机的信息，可在playbook中直接引用setup模块获取到的属性，比如系统版本：ansible_distribution_major_version

## 4.2 模板

playbook模板为我们提供了动态的配置服务，使用jinja2语言，支持多种条件判断、循环、逻辑运算、比较操作等。应用场景就是定义一个模板配置文件，然后在执行的时候动态生成最终的配置文件下发到远程主机。一般将模板文件放在playbook文件同级的templates目录下，这样在playbook文件中可以直接引用，否则需要通过绝对路径指定，模板文件后缀名一般为 .j2。

本例中，我们将nginx.conf配置文件作为模板文件，添加需要动态配置的内容，并定义一个变量文件，通过vars_files引入：`vars_files: ./templates/nginx_locations_vars.yml`

```bash
# 模板文件
[root@tool-server nginx-deploy]# vim templates/nginx.conf.j2
 ...
 server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;
        # 这里的内容动态生成
        {% for location in nginx_locations %}
        location {{location.path}} {
           proxy_pass {{location.proxy}};
        }
        {% endfor %}

        location / {
            root   html;
            index  index.html index.htm;
        }
 ...
# 独立的自定义变量文件，用于填充模板文件中的变量
[root@tool-server nginx-deploy]# vim templates/nginx_locations_vars.yml

nginx_locations:
  - {"path": "/cns", "proxy": "http://192.168.40.202/cns"}
  - {"path": "/admin", "proxy": "http://192.168.40.202/admin"}
```

## 4.3 handlers

handlers和notify结合使用，由特定条件触发，一般用于配置文件变更触发服务重启。在本例中我们在配置文件变更时，通过notify定义了一个“reload nginx”的操作，然后在handlers部分定义“reload nginx”操作——通过shell模块调用nginx的reload来重新加载配置。

## 4.4 标签

playbook文件中，如果只想执行某一个或几个任务，则可以给任务打标签，在运行的时候通过 -t 选择带指定标签的任务执行，也可以通过 --skip-tags 选择不带指定标签的任务执行。比如在本例中，我们在“update config”的task上加了“reload”的标签，如果后面再修改配置，我们只需要执行“update config”的task并触发reload nginx就行了，可以这么执行playbook

```bash
[root@tool-server nginx-deploy]# ansible-playbook -t reload nginx_playbook.yml
```

## 4.5 when

可以在task上添加when表示当某个条件达到了该任务才执行，如

```yaml
 tasks:
    - name: install nginx
      yum: name=nginx state=installed
    - name: update config for system6
      template: src=nginx.conf.j2 dest=/usr/local/nginx/conf/nginx.conf
      when: ansible_distribution_major_version == "6"   # 判断系统版本，为6才执行上面的template配置的文件
```

## 4.6 roles

roles就是将变量、文件、任务、模板及处理器放置在单独的目录中，并可以在playbook中include的一种机制，一般用于主机构建服务的场景中，但也可以是用于构建守护进程等场景。

roles的目录结构，默认的roles目录为/etc/ansible/roles

```yaml
roles:          # 所有的角色项目必须放在roles目录下
  project:      # 具体的角色项目名称，比如nginx、tomcat
    files：     # 用来存放由copy或script模块调用的文件
    templates： # 用来存放jinjia2模板，template模块会自动在此目录中寻找jinjia2模板文件
    tasks：     # 此目录应当包含一个main.yml文件，用于定义此角色的任务列表，此文件可以使用include包含其它的位于此目录的task文件。
      main.yml
    handlers：  # 此目录应当包含一个main.yml文件，用于定义此角色中触发条件时执行的动作
      main.yml
    vars：      # 此目录应当包含一个main.yml文件，用于定义此角色用到的变量
      main.yml
    defaults：  # 此目录应当包含一个main.yml文件，用于为当前角色设定默认变量
      main.yml
    meta：      # 此目录应当包含一个main.yml文件，用于定义此角色的特殊设定及其依赖关系
      main.yml
```

我们将上面的例子通过roles改造一下

```bash
[root@tool-server ~]# cd /etc/ansible/roles/
[root@tool-server roles]# mkdir -p nginx/{tasks,vars,templates,handlers}
...#创建各目录的mian.yml文件，并将对应的内容加入文件中
#最终目录结构
[root@tool-server roles]# tree  .
.
└── nginx
    ├── handlers
    │   └── main.yml # 上例handlers部分的内容，直接 -name开头，不需要再加 `handlers：`
    ├── tasks
    │   └── main.yml # tasks部分内容，直接-name开头，不需要加tasks，可以将各个task拆分为多个文件，然后在main.yml中通过 `- include: install.yml` 形式的列表引入
    ├── templates
    │   └── main.yml # templates/nginx.conf.j2的内容
    └── vars
        └── main.yml # templates/nginx_locations_vars.yml的内容

5 directories, 4 files
```

最后，在playbook中通过roles引入，

```yml
[root@ansible roles]# vim nginx_playbook.yml
---
- hosts: nginx
  remote_user: root
  roles:
    - role: nginx # 指定角色名称
```

roles将playbook的各个部分进行拆分组织，主要用于代码复用度较高的场景。

# 总结

Ansible是功能强大但又很轻量级的自动化运维工具，基于SSH协议批量对远程主机进行管理，不仅可用于日常的服务维护，也可与Jenkins等CI/CD工具结合实现自动化部署。如果你需要在多于一台服务器上做重复又稍显复杂的操作，那么建议你使用Ansible，这将极大提高你的操作效率，并且所有操作文档化，更易维护与迁移。