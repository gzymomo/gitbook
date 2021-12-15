- [自动化运维利器Ansible要点汇总](https://www.cnblogs.com/zhangs1986/p/15419495.html)

Ansible与Saltstack最大的区别是Ansible无需在被控主机部署任何客户端代理，默认直接通过SSH通道进行远程命令执行或下发配置，这里不作详细对比，直接使用Ansible。Ansible是DevOps项目基础工具之一，致力于自动化、工具化的全新维护模式，通过工具化自动化的作业，提高生产效率的同时减轻维护人员的重担。

　　Ansible是一款基于Python开发的自动化运维工具，实现了批量系统配置、批量程序部署、批量运行命令等功能，主要特点：

- 部署简单，只需在主控端部署Ansible环境，被控端无需做任何操作，默认使用SSH协议对设备进行管理；
- 配置简单、功能强大、扩展性强；
- 支持API及自定义模块，可通过Python轻松扩展；
- 通过Playbooks来定制强大的配置、状态管理；
- 幂等性：一种操作重复多次结果相同

## Ansible工作流程

 

![img](https://img2020.cnblogs.com/blog/273387/202110/273387-20211018113942913-638932092.jpg)

　　ansible.cfg主要配置指定host文件路径，指定roles_path参数，其它参数默认。

　　部署Ansible的控制机需要python 2.7及以上，需要安装paramiko模块、PyYAML、Jinja2、httplib2等模块，若被管节点为windows，则需要有powershell3并制授权远程管理。

　　控制节点交互一般采用公钥认证，这需要将主机节点的公钥发放到所有被管节点，也可采用密码形式通讯，但由于需要在hosts文件中明文标出不安全 不推荐，配置密码：

```
/etc/ansible/hosts
192.168.0.200 ansible_ssh_user=root ansible_ssh_pass=123@abc
```

　　主机清单（host inventory）定义了管理主机的策略，需要在host文件中写入主机的IP地址即可，若操作的主机未在清单中会提示错误。

## ansible命令执行过程

> 1、加载自己的配置文件，默认/etc/ansible/ansible.cfg
>
> 2、查找对应的主机配置文件，找到要执行的主机或者组。
>
> 3、加载自己对应的模块文件，如command
>
> 4、通过ansible将模块或命令生成对应的临时py文件，并将该文件传输至远程服务器
>
> 5、对应执行用户家目录的.ansible/tmp/XXX/XXX.PY文件
>
> 6、给文件+x执行
>
> 7、执行并返回结果
>
> 8、删除临时py文件，sleep 0 退出。

　　Ansible完成任务的两种方式，一种是Ad-Hoc，就是ansible命令，另一种就是Ansible-playbook，也就是ansible-playbook命令。他们的区别就像是Command命令行和Shell Scripts。

## ansible命令

获取192.168.0.123主机信息

> ansible 192.168.0.123 -m shell -a "uname -a"

### ansible常用模块

- command、shell、raw、script执行shell命令
- copy：复制文件到远程主机，可以改权限等

- file设置文件目录属性等
- fetch 从远程某主机获取文件到本地

- service 服务程序管理，启动停止重启服务等操作
- user管理用户账号
- script在指定节点运行服务端的脚本

## Playbooks中的一些技巧

playbook目录结构

![img](https://img2020.cnblogs.com/blog/273387/202110/273387-20211018224429457-686686636.png)

　　webservice.yml为入口，files目录存放静态文件，handlers存放一些task的handler，templates存放jinja2模板文件，vars存放变量文件。

　　ansible-playbook执行logstash安装剧本

> ansible-playbook /logstash/site.yml

　　这里不详细介绍playbook的使用，只摘出几个重要的使用场景方法。

### delegate_to

　　将某一个任务委托给指定主机，如在192.168.0.9服务器上检测k8s集群状态：

> \- name: get status
>
>  command: get k8s status
>
>  delegate_to: "192.168.0.9"

　　若委派给本机的时候，还可以使用更快捷的方法local_action

> \- name: get status
>
>  local_action : command 'get k8s status'

### run_once

　　run_once: true来指定该task只能在某一台机器上执行一次. 可以和delegate_to 结合使用，指定在"192.168.0.9"上执行一次升级数据库操作

> \- command: /opt/upgrade_db.py
>   run_once: true
>   delegate_to: "192.168.0.9"

　　如果没有delegate_to, 那么这个task会在第一台机器上执行

### ignore_errors

　　指定 ignore_errors：true，任务失败继续完成剩余的任务。例如，当删除最初并不存在的日志文件时抛错 但忽略错误继续执行剩余的任务。

> \- name: 'Delete logs'
>
> shell: rm -f /var/log/nginx/errors.log
>
> ignore_errors: true

### register 注册变量

　　使用 debug 模块与 register 变量，输出网络信息

> \- hosts: proxyservers
>     tasks:
>     \- name: "get host port info"
>     shell: netstat -lntp
>     register: host_port
>
> \- name: "print host port"
>     debug:
>     \#msg: "{{ host_port }}" # 输出全部信息
>     \#msg: "{{ host_port.cmd }}" # 引用方式一
>     msg: "{{ host_port['stdout_lines'] }}" # 引用方式二

### connection: local

　　在本地服务器上运行命令，而不是SSH

> \- name: 创建 aggregator proxy证书签名请求
>   template: src=aggregator-proxy-csr.json.j2 dest=/ssl/aggregator-proxy-csr.json
>   connection: local

### until轮询等待

　　轮询等待kube-apiserver启动完成，查看api服务是否running状态，重试10次，每次间隔3秒

```
- name: 轮询等待kube-apiserver启动
  shell: "systemctl status kube-apiserver.service|grep Active"
  register: api_status
  until: '"running" in api_status.stdout'
  retries: 10
  delay: 3
  tags: upgrade_k8s, restart_master
```

### when判断

　　当系统为centos等时执行centos.yml任务

```
- import_tasks: centos.yml
  when: 'ansible_distribution in ["CentOS","RedHat","Amazon","Aliyun"]' 
```

### 内置变量inventory_hostname

　　inventory_hostname变量可以获取到被操作的当前主机的主机名称，这里所说的主机名称并不是linux系统的主机名，而是对应主机在清单中配置的名称

如果使用IP配置主机，inventory_hostname的值就是IP，如果使用别名，inventory_hostname的值就是别名

　　如只给k8s主节点分发配置文件

```
- name: 分发kubeconfig配置文件
  copy: src=cluster_dir/item dest=/etc/kubernetes/item
  when: "inventory_hostname in groups['kube_master']"
```

### notify指令和handlers

　　如果在某个task中定义了notify指令，当Ansible在监控到该任务 changed=1时，会触发该notify指令所定义的handler，然后去执行handler，需要注意的是hander是被触发而被动执行的。

　　网上示例，安装httpd、复制配置文件到远端主机、启动httpd服务：

```
cat apache.yml
- hosts: webservers
  remote_user: root
  tasks:
  - name: install apache
    yum: name=httpd state=latest
  - name: install configure file for httpd
    copy: src=/root/conf/httpd.conf dest=/etc/httpd/conf/httpd.conf
    notify:
    - restart httpd  #通知restart httpd这个触发器
    - check httpd  #可以定义多个触发器
  - name: start httpd service
    service: enabled=true name=httpd state=started
  handlers：  #定义触发器，和tasks同级
  - name: restart httpd  #触发器名字，被notify引用，两边要一致
    service: name=httpd state=restart
  - name: check httpd
    shell: netstat -ntulp | grep 80
```

## ansible管理windows

　　环境要求Ansible管理主机Linux系统，远程主机的通信方式也由SSH变更为PowerShell，同时管理机必须预安装Python的Winrm模块。

　　Windows客户端主机开启Winrm服务，PowerShell需3.0+版本且Management Framework  3.0+版本，实测Windows 7 SP1和Windows Server 2008 R2及以上版本系统经简单配置可正常与Ansible通信。