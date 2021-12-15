[TOC]

常见的Java Web服务器。

- Tomcat：由Apache组织提供的一种Web服务器，提供对jsp和Servlet的支持。它是一种轻量级的javaWeb容器（服务器），也是当前应用最广的JavaWeb服务器（免费）。
- Jboss：是一个遵从JavaEE规范的、开放源代码的、纯Java的EJB服务器，它支持所有的JavaEE规范（免费）。
- GlassFish：由Oracle公司开发的一款JavaWeb服务器，是一款强健的商业服务器，达到产品级质量（应用很少，收费）。
- Resin：是CAUCHO公司的产品，是一个非常流行的应用服务器，对servlet和JSP提供了良好的支持，性能也比较优良，resin自身采用JAVA语言开发（收费，应用比较多）。
- WebLogic：是Oracle公司的产品，是目前应用最广泛的Web服务器，支持JavaEE规范，而且不断的完善以适应新的开发要求，适合大型项目（收费，用的不多，适合大公司）。

# 1、不修改端口
应用项目是直接放在Tomcat webapps目录下面：
```bash
[root@CentOS7-1 tomcat]# cd webapps/
[root@CentOS7-1 webapps]# ll
total 4
drwxr-x--- 16 root root 4096 Jun  4 03:07 docs
drwxr-x---  6 root root   83 Jun  4 03:07 examples
drwxr-x---  5 root root   87 Jun  4 03:07 host-manager
drwxr-x---  5 root root  103 Jun  4 03:07 manager
drwxr-x---  3 root root  283 Jun  4 03:07 ROOT
```
所以，我们在不修改端口的情况下，可以直接在此目录下新增多个项目目录，也可以直接将war包放在此目录下，由于测试环境，我们直接模拟war解压后的目录，用添加目录来替代。
```bash
[root@CentOS7-1 webapps]# mkdir test java
[root@CentOS7-1 webapps]# ls
docs examples host-manager java manager ROOT test
```
准备测试的首页文件
```bash
[root@CentOS7-1 webapps]# echo "this is a test" >test/test.html
[root@CentOS7-1 webapps]# echo "this is a java" >java/java.html
[root@CentOS7-1 webapps]# cat test/test.html
this is a test
[root@CentOS7-1 webapps]# cat java/java.html
this is a java
```
修改配置文件：
```bash
<!-- test -->
    <Context path="test/" docBase="test" reloadable="true" />
    <Context path="java/" docBase="java" reloadable="true" />
     #增加上两行配置即可
      </Host>
    </Engine>
  </Service>
</Server>
"../conf/server.xml" 173L, 7744C
```
>docBase属性: 指定Web应用的文件路径，可以是绝对路径，也可以给定相对路径
>path属性: 指定访问该Web应用的URL入口。
>reloadable属性: 若这个属性为true，tomcat服务器在运行状态下会监视WEB-INF/classes和WEB-INF/lib目录下class文件的改动，如果监测到class文件被更新，服务器会自动重新加载Web应用。

重启Tomcat服务，测试访问，结果如下：
![](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPpdCMDUzBD8Fu90JnqT0cMEo8w8ofnUlk10eqtPgloicEgdRiccQ6F2saaoj49g5G1M4rr00USHZMXQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

部署成功。
> 注：配置文件中增加的配置步骤可以不做，直接跳过，不是必须要做的步骤。

# 2、修改端口
在tomcat目录下创建多个webapps目录。
删除webapps目录下的java项目，并删除webapps1目录下test项目即可。

## 2.1 修改配置文件

server.xml已有第一个项目的配置信息，现在需要新增第二个项目的配置，在Server节点下，新增一个Service节点，第2个Service节点直接复制第1个Service内容修改即可。
```html
<Service name="Catalina1">
    <!--The connectors can use a shared executor, you can define one or more named thread pools-->

    <Connector port="8081" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
    <!-- A "Connector" using the shared thread pool-->

    <Engine name="Catalina1" defaultHost="localhost">

      <!-- Use the LockOutRealm to prevent attempts to guess user passwords
           via a brute-force attack -->
      <Realm className="org.apache.catalina.realm.LockOutRealm">
        <!-- This Realm uses the UserDatabase configured in the global JNDI
             resources under the key "UserDatabase". Any edits
             that are performed against this UserDatabase are immediately
             available for use by the Realm. -->
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
               resourceName="UserDatabase"/>
      </Realm>

      <Host name="localhost"  appBase="webapps1"
            unpackWARs="true" autoDeploy="true">

        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t "%r" %s %b" />
      </Host>
    </Engine>
  </Service>
```

- Service的name属性修改为Catelina1；
- http协议访问的Connector port属性修改为8081；
- Engine的name属性修改为Catelina1；
- Host的appBase属性修改为webapps1；

重启服务并测试访问:
```bash
[root@CentOS7-1 conf]# ../bin/startup.sh
Using CATALINA_BASE: /usr/local/tomcat
Using CATALINA_HOME: /usr/local/tomcat
Using CATALINA_TMPDIR: /usr/local/tomcat/temp
Using JRE_HOME: /usr
Using CLASSPATH: /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar
Tomcat started.
[root@CentOS7-1 conf]# lsof -i :8080
COMMAND PID USER FD TYPE DEVICE SIZE/OFF NODE NAME
java 2486 root 52u IPv6 25075      0t0 TCP *:webcache (LISTEN)
[root@CentOS7-1 conf]# lsof -i :8081
COMMAND PID USER FD TYPE DEVICE SIZE/OFF NODE NAME
java 2486 root 57u IPv6 25079      0t0 TCP *:tproxy (LISTEN)
[root@CentOS7-1 conf]# curl http://127.0.0.1:8080/test/test.html
this is a test
[root@CentOS7-1 conf]# curl http://127.0.0.1:8081/java/java.html
this is a java
```
![](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPpdCMDUzBD8Fu90JnqT0cMEDwOB6qIgMLHGMSS5IqN6fFzzkhBywgyEtmXfiaWKfzO56ysrJAchBzA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPpdCMDUzBD8Fu90JnqT0cMEDwOB6qIgMLHGMSS5IqN6fFzzkhBywgyEtmXfiaWKfzO56ysrJAchBzA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)