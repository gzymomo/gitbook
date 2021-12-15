### 1、隐藏版本号

进入`tomcat`的`lib`目录找到`catalina.jar`文件

```
unzip catalina.jar
```

之后会多出两个文件夹 进入`org/apache/catalina/util`编辑配置文件`ServerInfo.properties`修改为

```
server.info=Apache Tomcat
server.number=0.0.0.0
server.built=Nov 7 2016 20:05:27 UTC
```

将修改后的信息压缩回jar包

```
cd  /tomcat/lib
jar uvf catalina.jar org/apache/catalina/util/ServerInfo.properties
```

### 2、禁用不安全的方法

`tomcat`限制不安全`http`方法，如`put`、`delete`等等，设置方法在`conf/web.xml`里添加限制如下格式：

```
<security-constraint> 
        <web-resource-collection> 
            <url-pattern>/*</url-pattern> 
            <http-method>PUT</http-method> 
            <http-method>DELETE</http-method> 
            <http-method>HEAD</http-method> 
            <http-method>OPTIONS</http-method>  
            <http-method>TRACE</http-method> 
        </web-resource-collection> 
        <auth-constraint> 
        </auth-constraint> 
</security-constraint>
```

### 3、错误页面跳转

`tomcat`的`404、502、403`等等错误页面的跳转设置为指定跳转页面，设置方法在`conf/web.xml`里添加跳转如下格式：

```
    <error-page> 
        <exception-type>java.lang.Exception</exception-type> 
        <location>/404.html</location> 
    </error-page> 
    <error-page> 
        <error-code>404</error-code> 
        <location>/404.html</location> 
    </error-page> 
    <error-page> 
        <error-code>400</error-code> 
        <location>/404.html</location> 
    </error-page> 
    <error-page> 
        <error-code>500</error-code> 
        <location>/404.html</location> 
    </error-page> 
```

### 4、使tomcat支持软链接

修改`conf/context.xml`文件：

`tomcat7`配置方法：

```
<!-- The contents of this file will be loaded for each web application -->
<Context allowLinking="true">
```

`tomcat8`配置方法：

```
<Context>
    <Resources allowLinking="true" />
</Context>
```

### 5、tomcat增加http安全响应头

修改`web.xml`文件：

配置方法：

```
    <filter>
        <filter-name>httpHeaderSecurity</filter-name>
        <filter-class>org.apache.catalina.filters.HttpHeaderSecurityFilter</filter-class>
        <async-supported>true</async-supported>
        <init-param>
          <param-name>antiClickJackingEnabled</param-name>
          <param-value>true</param-value>
        </init-param>
        <init-param>
          <param-name>antiClickJackingOption</param-name>
          <param-value>SAMEORIGIN</param-value>
        </init-param>
        <init-param>
          <param-name>blockContentTypeSniffingEnabled</param-name>
          <param-value>false</param-value>
        </init-param>
    </filter>

    <filter-mapping>
        <filter-name>httpHeaderSecurity</filter-name>
        <url-pattern>/*</url-pattern>
        <dispatcher>REQUEST</dispatcher>
    </filter-mapping>
```

### 6、禁用管理端，强制或使用nginx配置规则

- 删除默认的{Tomcat安装目录}/conf/tomcat-users.xml文件(强制)
- 删除{Tomcat安装目录}/webapps下默认的所有目录和文件(强制)

### 7、Server header重写

当`tomcat HTTP`端口直接提供`web`服务时此配置生效，加入此配置，将会替换`http`响应`Server header`部分的默认配置，默认是`Apache-Coyote/1.1`

修改`conf/server.xml`：

```
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" 
               server="webserver" />
```

### 8、访问日志规范

开启`Tomcat`默认访问日志中的`Referer`和`User-Agent`记录，一旦出现安全问题能够更好的根据日志进行问题排查；`X-Forwarded-For`用于`nginx`作为反向代理服务器时，获取客户端真实的`IP`

修改`conf/server.xml`：

```
<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
       prefix="localhost_access_log" suffix=".txt"
       pattern="%{X-Forwarded-For}i %l %u %t %r %s %b %{Referer}i %{User-Agent}i %D" resolveHosts="false" />
```

### 9、tomcat设置字符集UTF-8

修改`conf/server.xml`：

```
<Connector port="8080" protocol="HTTP/1.1"
    connectionTimeout="20000"
    redirectPort="8443" URIEncoding="UTF-8" />
```

### 10、修复某些项目Java中文字体不显示（中文乱码问题）

这种情况有可能是项目代码以及项目编译时的编码问题，也有可能是项目使用了特殊的中文字体，如果有特殊的中文字体，需要将字体文件放到`jdk`目录下

例如：在`jdk`中新建目录

```
/jdk1.8.0_191/jre/lib/fonts/fallback
```

将系统中`simsun.ttc`字体文件拷贝到此目录，并重命名为`simsun.ttf`

### 11、tomcat遵循JVM的delegate机制

修改`conf/context.xml`

```
<Loader delegate="true"/>
</Context>
Loader`对象可出现在`Context`中以控制`Java`类的加载。属性：`delegate`、含义：`True`代表使用正式的`Java`代理模式(先询问父类的加载器)；`false`代表先在`Web`应用程序中寻找。默认值：`FALSE
```

`True`，表示`tomcat`将遵循`JVM`的`delegate`机制，即一个`WebAppClassLoader`在加载类文件时，会先递交给`SharedClassLoader`加载，`SharedClassLoader`无法加载成功，会继续向自己的父类委托，一直到`BootstarpClassLoader`，如果都没有加载成功，则最后由`WebAppClassLoader`自己进行加载。

`False`，表示将不遵循这个`delegate`机制，即`WebAppClassLoader`在加载类文件时，会优先自己尝试加载，如果加载失败，才会沿着继承链，依次委托父类加载。

### 12、tomcat8静态资源缓存配置

tomcat8增加了静态资源缓存的配置，`.cacheMaxSize`：静态资源缓存最大值，以`KB`为单位，默认值为`10240KB` `.cachingAllowed`：是否允许静态资源缓存，默认为`true`解决缓存溢出的办法 对应两个参数，解决方法有两种: 1：考虑增加cache的最大大小 2：关闭缓存 修改conf/context.xml

```
<Resources cachingAllowed="true" cacheMaxSize="1048576" ></Resources>
```