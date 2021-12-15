## 2方案：

### 2.1方案一：springboot+https

#### 2.1.1生成证书

如果配置了JAVA开发环境，可以使用keytool命令生成证书。我们打开控制台，输入：

```java
keytool -genkey -alias tomcat -dname "CN=Andy,OU=kfit,O=kfit,L=HaiDian,ST=BeiJing,C=CN" -storetype PKCS12 -keyalg RSA -keysize 2048 -keystore keystore.p12 -validity 365
```

然后输入密码，输入的面在下面的配置文件中会用到。 生成的证书在c://users下面。

#### 2.2.2 修改配置文件

```yaml
#http的端口号
server.http.port=8081
#https的端口号
server.port=8080
#https的证书地址
server.ssl.key-store=classpath:server.keystore
#https的生成的密码
server.ssl.key-store-password=123456
#https的生成的密码
server.ssl.key-password=123456
#启用SSL协议
server.ssl.enabled-protocols=TLSv1.1,TLSv1.2
#支持的SSL密码
server.ssl.ciphers=TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384,TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA,TLS_ECDHE_RSA_WITH_RC4_128_SHA,TLS_RSA_WITH_AES_128_CBC_SHA256,TLS_RSA_WITH_AES_128_CBC_SHA,TLS_RSA_WITH_AES_256_CBC_SHA256,TLS_RSA_WITH_AES_256_CBC_SHA,SSL_RSA_WITH_RC4_128_SHA
```

#### 2.2.3 把证书放在resource目录下

#### 2.2.4增加tomcat配置文件

```java
@Configuration
public class TomcatConfig {

    @Bean
    public TomcatServletWebServerFactory tomcatServletWebServerFactory(){
        TomcatServletWebServerFactory tomcat =new TomcatServletWebServerFactory();
        tomcat.addAdditionalTomcatConnectors(httpConnector());
        return tomcat;
    }

    @Bean
    public Connector httpConnector() {
        Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
        connector.setScheme("http");
        //Connector监听的http的端口号
        connector.setPort(8080);
        connector.setSecure(false);
        //监听到http的端口号后转向到的https的端口号
        //connector.setRedirectPort(8081);
        return connector;
    }
}
```

#### 2.2.4 验证

说明同时启动了https的端口和http的端口

### 2.2方案二：Nginx+https

通过Ngiinx方式配置https的支持。

#### 2.1.1生成证书

可以通过一些免费的平台提供的证书

#### 2.1.2 docker下安装nginx

- 1将本地的nginx镜像拷贝到服务器上的文件夹下
- 2创建/etc/nginx，/etc/nginx/cert目录
- 3将证书复制到/etc/nginx/cert目录下
- 4将nginx.conf文件复制到/etc/nginx目录下
- 5修改nginx.conf配置文件如下：

```yaml
user  nginx;
worker_processes  1;

#error_log  /var/log/nginx/error.log info;
#pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
   # include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    #error_log /var/log/nginx/http.log info;
    #access_log  /var/log/nginx/access.log;
    client_max_body_size 1024M;
    sendfile on;

    upstream web {
        server 192.168.122.70:8080;
    }

    
    server {
        listen 90;
     
        location / {
            proxy_set_header Host $host;
            proxy_http_version 1.1;
            proxy_set_header Connection "";
            proxy_set_header client-ip $remote_addr;
            proxy_pass http://web;
        }
    }
	
	#https
    server {
        listen 80 default ssl;
        ssl_certificate /etc/nginx/cert/cert.pem;
        ssl_certificate_key /etc/nginx/cert/key.pem;
        location / {
            proxy_set_header Host $host;
            proxy_http_version 1.1;
            proxy_set_header Connection "";
            proxy_set_header client-ip $remote_addr;
            proxy_set_header is-https "1";
            proxy_pass http://web;
        }
    }	
	

}
```

- 6docker 装在nginx

```bash
docker load --input $PWD/app/nginx/nginx-1.19.6.tar
```

- 7docker 运行nginx

```bash
docker run --privileged=true --net=host --name nginx -v /etc/nginx/nginx.conf:/etc/nginx/nginx.conf  -v /etc/nginx/cert:/etc/nginx/cert -d nginx:1.19.6
```

#### 2.2.3 验证

通过访问80端口，发现是https的请求

## 3方案对比

| 对比                         | Springboot+https | Springboot+https |
| ---------------------------- | ---------------- | ---------------- |
| 是否需要证书                 | 需要             | 需要             |
| 是否需要改动代码             | 需要             | 否               |
| 是否引入第三方插件或者中间件 | 否               | 需要             |
| 效率                         | 未测试           | 未测试           |


