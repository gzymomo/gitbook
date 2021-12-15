[TOC]

# 1、基本的规范
## 1.1 开头指定脚本解释器
`#!bin/bash  或 #!/bin/sh`

## 1.2 开头加版本版权等信息
```bash
#Date:    20:20 202-3-5
#Author：create by guoke
#Mail：    123321@qq.com
#Function: This scripts function.....
#Version：2.1
```
时间、作者、邮件、功能、版本

## 1.3 脚本中不使用中文注释

## 1.4 脚本以.sh为扩展名
`start.sh`

## 1.5 创建shell脚本程序的步骤
```
#第一步：创建一个包含命令和控制结构的shell文件，以.sh为扩展名
#第二步：修改这个文件权限使它可以执行
            修改方式：chmod u+x  文件名
#第三步：执行
    方法1：./example
    方法2：bash + 文件
    方法3：source + 文件
```

# 2、书写习惯
## 2.1 成对的符号应尽量一次性写出，然后退格在符号里增加内容，防止遗漏
例如：`{} [] '' "" `

## 2.2 中括号[]两端至少要有1个空格，输入技巧：先输入一对中括号，然后退一个格，输入两个空格，再退一格，双中括号[[]]也是这样写
`[ name ]     [[ name ]] `
## 2.3 对于流程控制语句应该一次性将格式写完，再添加内容：
```bash
if 条件内容
	then
		内容
fi
```
```bash
for
	do
		内容
done
```
while，case，until等语句一样道理。

## 2.4 缩进让代码更易读(tab键)
```bash
if 条件内容
	then
		内容
fi
```
## 2.5 字符串赋值给变量应加双引号，并且等号前后不能有空格
` my_file="test.txt"`

## 2.6 脚本中的单引号、双引号及反引号，必须为英文状态下的符号

# 三、Shell脚本
## 3.1 IF
```shell
if command1
then
	commands
elif command2
then
	more commands
fi
```

bash shell提供了无需在if-then语句中声明test的测试方法：
```shell
if [ condition ]
then
	commands
fi
```
方括号定义了测试条件，第一个方括号之后和第二个方括号之前必须加上一个空格，否则会报错。
test命令可以判断三类条件：
 - 数值比较
 - 字符串比较
 - 文件比较
## 3.2 文件比较
|  比较 | 描述  |
| ------------ | ------------ |
| -d file  |  检查file是否存在并是一个目录 |
| -e file  | 检查file是否存在  |
| -f file  | 检查file是否存在并是一个文件  |
| -r file  | 检查file是否存在并可读  |
| -s file  | 检查file是否存在并非空  |
| -w file  | 检查file是否存在并可写  |
| -x file  | 检查file是否存在并可执行 |
| -O file  | 检查file是否存在并属当前用户所有 |
| -G file  | 检查file是否存在并且默认组与当前用户相同 |
| file1 -nt file2 | 检查file1是否比file2新 |
| file1 -ot file2 | 检查file1是否比file2旧 |



```shell
#!/bin/bash
echo $1
nohup java -jar $1  --spring.profiles.active=test --server.port=50070 > /root/logs/yz/yz.log 2>&1 &
tail -f /root/logs/yz/yz.log
```

```
echo $1 ：获取当前脚本输入后，空格后的第一个文件，然后通过java -jar 启动该文件
```

```
# springboot 使用spring.profiles.active 区分不同环境下配置文件
# --server.port=50070  springboot指定启动端口
```

