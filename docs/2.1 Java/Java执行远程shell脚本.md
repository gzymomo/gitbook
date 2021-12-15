# 通过java程序(JSch)运行远程linux主机上的shell脚本

[TOC]

博客园：字母哥博客：[通过java程序(JSch)运行远程linux主机上的shell脚本](https://www.cnblogs.com/zimug/p/13450493.html)

## 运行远程主机上的shell脚本

下面的例子是教给大家如何通过java程序，运行远程主机上的shell脚本。（我讲的不是一个黑客学习教程，而是使用用户名密码去执行有用户认证资格的主机上的shell脚本）。并且通过java程序获得shell脚本的输出。
首先通过maven坐标引入[JSch](http://www.jcraft.com/jsch/)依赖库，我们正是通过JSch去执行远程主机上的脚本。

```markup
<dependency>
    <groupId>com.jcraft</groupId>
    <artifactId>jsch</artifactId>
    <version>0.1.55</version>
</dependency>
```

当然以下java代码可执行的的前提是，远程主机已经开通SSH服务（也就是我们平时登录主机所使用的服务）。

### 远程shell脚本

下面的代码放入一个文件：`hello.sh`，脚本的内容很简单只是用来测试，回显输出“hello <参数1> ”

```bash
#! /bin/sh
echo "hello $1\n";
```

然后我把它放到远程主机的`/root`目录下面，远程主机的IP是`1.1.1.1`（当然我真实测试时候不是这个IP，我不能把我的真实IP写到这个文章里面，以免被攻击）。并且在远程主机上，为这个脚本设置可执行权限，方法如下：

```bash
$ chmod +x hello.sh
```

### 本地java程序

我们可以使用下面的代码，去远程的linux 主机执行shell脚本，详细功能请看代码注释

```java
import com.jcraft.jsch.*;

import java.io.IOException;
import java.io.InputStream;

public class RunRemoteScript {
    //远程主机IP
    private static final String REMOTE_HOST = "1.1.1.1";
    //远程主机用户名
    private static final String USERNAME = "";
    //远程主机密码
    private static final String PASSWORD = "";
    //SSH服务端口
    private static final int REMOTE_PORT = 22;
    //会话超时时间
    private static final int SESSION_TIMEOUT = 10000;
    //管道流超时时间(执行脚本超时时间)
    private static final int CHANNEL_TIMEOUT = 5000;

    public static void main(String[] args) {
        //脚本名称及路径，与上文要对上
        String remoteShellScript = "/root/hello.sh";

        Session jschSession = null;

        try {

            JSch jsch = new JSch();
            //SSH授信客户端文件位置，一般是用户主目录下的.ssh/known_hosts
            jsch.setKnownHosts("/home/zimug/.ssh/known_hosts");
            jschSession = jsch.getSession(USERNAME, REMOTE_HOST, REMOTE_PORT);

            // 密码认证
            jschSession.setPassword(PASSWORD);

            // 建立session
            jschSession.connect(SESSION_TIMEOUT);
            //建立可执行管道
            ChannelExec channelExec = (ChannelExec) jschSession.openChannel("exec");

            // 执行脚本命令"sh /root/hello.sh zimug"
            channelExec.setCommand("sh " + remoteShellScript + " zimug");

            // 获取执行脚本可能出现的错误日志
            channelExec.setErrStream(System.err);

            //脚本执行结果输出，对于程序来说是输入流
            InputStream in = channelExec.getInputStream();

            // 5 秒执行管道超时
            channelExec.connect(CHANNEL_TIMEOUT);

            // 从远程主机读取输入流，获得脚本执行结果
            byte[] tmp = new byte[1024];
            while (true) {
                while (in.available() > 0) {
                    int i = in.read(tmp, 0, 1024);
                    if (i < 0) break;
                    //执行结果打印到程序控制台
                    System.out.print(new String(tmp, 0, i));
                }
                if (channelExec.isClosed()) {
                    if (in.available() > 0) continue;
                    //获取退出状态，状态0表示脚本被正确执行
                    System.out.println("exit-status: "
                         + channelExec.getExitStatus());
                    break;
                }
                try {
                    Thread.sleep(1000);
                } catch (Exception ee) {
                }
            }

            channelExec.disconnect();

        } catch (JSchException | IOException e) {

            e.printStackTrace();

        } finally {
            if (jschSession != null) {
                jschSession.disconnect();
            }
        }

    }
}
```

最终在本地控制台，获得远程主机上shell脚本的执行结果。如下：

```bash
hello zimug

exit-status: 0
```

# Java执行Dos-Shell脚本

[TOC]

[bldong:Java 执行Shell脚本指令](https://www.cnblogs.com/polly333/p/7832540.html)

## 1、介绍
在Linux中运行Java程序时，需要调用一些Shell命令和脚本。而Runtime.getRuntime().exec()方法给我们提供了这个功能，而且Runtime.getRuntime()给我们提供了以下几种exec()方法：
```java
Process exec(String command)
在单独的进程中执行指定的字符串命令。

Process exec(String[] cmdarray)
在单独的进程中执行指定命令和变量。

Process exec(String[] cmdarray, String[] envp)
在指定环境的独立进程中执行指定命令和变量。

Process exec(String[] cmdarray, String[] envp, File dir)
在指定环境和工作目录的独立进程中执行指定的命令和变量。

Process exec(String command, String[] envp)
在指定环境的单独进程中执行指定的字符串命令。

Process exec(String command, String[] envp, File dir)
在有指定环境和工作目录的独立进程中执行指定的字符串命令。
```
如果参数中如果没有envp参数或设为null，表示调用命令将在当前程序执行的环境中执行；如果没有dir参数或设为null，表示调用命令将在当前程序执行的目录中执行，因此调用到其他目录中的文件和脚本最好使用绝对路径。

各个参数的含义：

1. cmdarray: 包含所调用命令及其参数的数组。 
2. command: 一条指定的系统命令。
3. envp: 字符串数组，其中每个元素的环境变量的设置格式为name=value；如果子进程应该继承当前进程的环境，则该参数为 null。
4. dir: 子进程的工作目录；如果子进程应该继承当前进程的工作目录，则该参数为 null。

通过调用Process类的以下方法，得知调用操作是否正确执行：
```java
abstract  int waitFor()
导致当前线程等待，如有必要，一直要等到由该 Process 对象表示的进程已经终止。
```

## 2、调用shell脚本
### 2.1 获取键盘输入
```java
BufferedReader reader = null;
        try{
            reader = new BufferedReader(new InputStreamReader(System.in));
            System.out.println("请输入IP:");
            String ip = reader.readLine();
```
上述指令基本很常见：

1. 创建读入器：BufferReader
2. 将数据流载入BufferReader，即InputStreamReader
3. 将系统输入载入InputStreamReader中
4. 然后利用reader获取数据。

### 2.2 构建指令
shell运行脚本指令为 sh **.sh args。

```java
#!/bin/sh
#根据进程名杀死进程
echo "This is a $call"
if [ $# -lt 2 ]
then   echo "缺少参数：procedure_name和ip"   exit 1
fi
echo "Kill the $1 process"
PROCESS=`ps -ef|grep $1|grep $2|grep -v grep|grep -v PPID|awk '{ print $2}'`
for i in $PROCESS
do   echo "Kill the $1 process [ $i ]"
done
```
注意事项：

1. shell脚本必须有执行权限，比如部署后chmod -R 777 /webapps
2. shell文件，必须是UNIX格式，ANSI编码格式，否则容易出问题（可以用notepad++,编辑->文档格式转换，格式->转为ANSI格式（UNIX格式）

### 2.3 Java代码
```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

public class TestBash {
    public static void main(String [] args){
        BufferedReader reader = null;
        try{
            reader = new BufferedReader(new InputStreamReader(System.in));
            System.out.println("请输入IP:");
            String ip = reader.readLine();
            String bashCommand = "sh "+ "/usr/local/java/jdk1.8.0_121/lib/stopffmpeg.sh" + " ffmpeg " + ip;
//            String bashCommand = "chmod 777 " + "/usr/local/java/jdk1.8.0_121/lib/stopffmpeg.sh" ;
//            String bashCommand = "kill -9" + ip;
            System.out.println(bashCommand);
            Runtime runtime = Runtime.getRuntime();
            Process pro = runtime.exec(bashCommand);
            int status = pro.waitFor();
            if (status != 0)
            {
                System.out.println("Failed to call shell's command ");
            }

            BufferedReader br = new BufferedReader(new InputStreamReader(pro.getInputStream()));
            StringBuffer strbr = new StringBuffer();
            String line;
            while ((line = br.readLine())!= null)
            {
                strbr.append(line).append("\n");
            }

            String result = strbr.toString();
            System.out.println(result);

        }
        catch (IOException ec)
        {
            ec.printStackTrace();
        }
        catch (InterruptedException ex){
            ex.printStackTrace();

        }
    }
}
```

## 3、Java调用Shell并传入参数
```java
public static  void invokeShell(){
//方法1 执行字符串命令（各个参数1234之间需要有空格）
String path="sh /root/zpy/zpy.sh 1 2 3 4";
//方法2 在单独的进程中执行指定命令和变量。 
//第一个变量是sh命令，第二个变量是需要执行的脚本路径，从第三个变量开始是我们要传到脚本里的参数。
String[] path=new String[]{"sh","/root/zpy/zpy.sh","1","2","3","4"};
		try{
			Runtime runtime = Runtime.getRuntime();
			Process pro = runtime.exec(path);
			int status = pro.waitFor();
			if (status != 0)
			{
				System.out.println("Failed to call shell's command");
			}

			BufferedReader br = new BufferedReader(new InputStreamReader(pro.getInputStream()));
			StringBuffer strbr = new StringBuffer();
			String line;
			while ((line = br.readLine())!= null)
			{
				strbr.append(line).append("\n");
			}
			String result = strbr.toString();
			System.out.println(result);
		}
		catch (IOException ec)
		{
			ec.printStackTrace();
		}
		catch (InterruptedException ex){
			ex.printStackTrace();
		}
	}
```

## 4、Java调用远程的Shell脚本
```xml
<!--调用远程服务器上的shell-->
<dependency>
    <groupId>org.jvnet.hudson</groupId>
    <artifactId>ganymed-ssh2</artifactId>
    <version>build210-hudson-1</version>
</dependency>
```

```java
 /**
     * 执行远程服务器上的shell脚本
     * @param ip 服务器IP地址
     * @param port  端口号
     * @param name  登录用户名
     * @param pwd  密码
     * @param cmds shell命令
     */
    public static void RemoteInvokeShell(String ip,int port,String  name,String pwd,String cmds) {
        Connection conn=null;
        try {
            conn = new Connection(ip,port);
            conn.connect();
            if (conn.authenticateWithPassword(name, pwd)) {
                // Open a new {@link Session} on this connection
                Session session = conn.openSession();
                // Execute a command on the remote machine.
                session.execCommand(cmds);
 
                BufferedReader br = new BufferedReader(new InputStreamReader(session.getStdout()));
                BufferedReader brErr = new BufferedReader(new InputStreamReader(session.getStderr()));
 
                String line;
                while ((line = br.readLine()) != null) {
                    logger.info("br={}", line);
                }
                while ((line = brErr.readLine()) != null) {
                    logger.info("brErr={}", line);
                }
                if (null != br) {
                    br.close();
                }
                if(null != brErr){
                    brErr.close();
                }
                session.waitForCondition(ChannelCondition.EXIT_STATUS, 0);
                int ret = session.getExitStatus();
                logger.info("getExitStatus:"+ ret);
            } else {
                logger.info("登录远程机器失败" + ip);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (conn != null) {
                conn.close();
            }
        }
    }
 public static void main(String[] args){
        RemoteInvokeShell("192.168.11.xx",22,"xx","xx","sh /root/zpy/zpy.sh  \"jj|aa|bb\"")//带有特殊符号的参数需要加上双引号
    }
```