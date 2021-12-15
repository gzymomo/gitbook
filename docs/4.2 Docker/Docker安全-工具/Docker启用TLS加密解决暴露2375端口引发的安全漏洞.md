[Docker启用TLS加密解决暴露2375端口引发的安全漏洞](https://www.cnblogs.com/haoxianrui/p/14095306.html)



## 二. 实操

### 1.  设置主机名

编辑/etc/hostname，服务器主机名 `a.youlai.store`



```
vi /etc/hostname
```

### 2. 生成TLS证书

创建证书生成脚本 cert.sh，放置/script目录



```bash
mkdir -p /script /data/cert/docker
touch /script/cert.sh
vim /script/cert.sh
```

cert.sh添加内容



```bash
#!/bin/bash
set -e
if [ -z $1 ];then
        echo "请输入Docker服务器主机名"
        exit 0
fi
HOST=$1
mkdir -p /data/cert/docker
cd /data/cert/docker
openssl genrsa -aes256 -out ca-key.pem 4096
openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem
openssl genrsa -out server-key.pem 4096
openssl req -subj "/CN=$HOST" -sha256 -new -key server-key.pem -out server.csr
# 配置白名单，推荐配置0.0.0.0，允许所有IP连接但只有证书才可以连接成功
echo subjectAltName = DNS:$HOST,IP:0.0.0.0 > extfile.cnf
openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem -extfile extfile.cnf
openssl genrsa -out key.pem 4096
openssl req -subj '/CN=client' -new -key key.pem -out client.csr
echo extendedKeyUsage = clientAuth > extfile.cnf
openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out cert.pem -extfile extfile.cnf
rm -v client.csr server.csr
chmod -v 0400 ca-key.pem key.pem server-key.pem
chmod -v 0444 ca.pem server-cert.pem cert.pem
```

执行 cert.sh 脚本，生成证书放置 /data/cert/docker 目录中



```bash
# a.youlai.store是服务器的主机名
sh /script/cert.sh a.youlai.store
```

按照提示输入相关信息，密码一致就行，其他信息可留空，等脚本指定完成之后，可在 /data/cert/docker 目录查看到生成的证书。

[![img](https://i.loli.net/2020/12/06/WQqxMta7o34vTfL.png)](https://i.loli.net/2020/12/06/WQqxMta7o34vTfL.png)

### 3. 配置Docker启用TLS



```bash
vim /usr/lib/systemd/system/docker.service
```

在ExecStart属性后追加



```bash
--tlsverify --tlscacert=/data/cert/docker/ca.pem  \
--tlscert=/data/cert/docker/server-cert.pem \
--tlskey=/data/cert/docker/server-key.pem \
-H tcp://0.0.0.0:2376 -H unix://var/run/docker.sock 
```

[![img](https://i.loli.net/2020/12/06/1vWsQmyfpC9DY6H.png)](https://i.loli.net/2020/12/06/1vWsQmyfpC9DY6H.png)

重新加载docker配置后重启



```bash
systemctl daemon-reload 
systemctl restart docker
```

查看2376端口是否启动



```bash
netstat -nltp | grep 2376
```

[![img](https://i.loli.net/2020/12/06/s4uoR6pWBMmtAnr.png)](https://i.loli.net/2020/12/06/s4uoR6pWBMmtAnr.png)

**本地连接测试Docker API是否可用**

- 没有指定证书访问测试



```bash
curl https://a.youlai.store:2376/info 
```

- 指定证书访问测试



```bash
curl https://a.youlai.store:2376/info --cert /data/cert/docker/cert.pem --key /data/cert/docker/key.pem --cacert /data/cert/docker/ca.pem
```

### 4. IDEA配置

将客户端所需的ca.pem、cert.pem、key.pem3个密钥文件从服务器下载到本地

[![img](https://i.loli.net/2020/12/06/dLPAsIak5KT7ogv.png)](https://i.loli.net/2020/12/06/dLPAsIak5KT7ogv.png)

IDEA连接Docker配置修改

[![img](https://i.loli.net/2020/12/06/T2ejKwkHlvpJ1fL.png)](https://i.loli.net/2020/12/06/T2ejKwkHlvpJ1fL.png)

pom.xml



```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
        <plugin>
            <groupId>com.spotify</groupId>
            <artifactId>docker-maven-plugin</artifactId>
            <version>1.0.0</version>
            <executions>
                <!--执行mvn package,即执行 mvn clean package docker:build-->
                <execution>
                    <id>build-image</id>
                    <phase>package</phase>
                    <goals>
                        <goal>build</goal>
                    </goals>
                </execution>
            </executions>

            <configuration>
                <!-- 镜像名称 -->
                <imageName>${project.artifactId}</imageName>
                <!-- 指定标签 -->
                <imageTags>
                    <imageTag>latest</imageTag>
                </imageTags>
                <!-- 基础镜像-->
                <baseImage>openjdk:8-jdk-alpine</baseImage>

                <!-- 切换到容器工作目录-->
                <workdir>/</workdir>

                <entryPoint>["java","-jar","${project.build.finalName}.jar"]</entryPoint>

                <!-- 指定远程 Docker API地址  -->
                <dockerHost>https://a.youlai.store:2376</dockerHost>
                <!-- 指定tls证书的目录 -->
                <dockerCertPath>C:\certs\docker\a.youlai.store</dockerCertPath>

                <!-- 复制 jar包到docker容器指定目录-->
                <resources>
                    <resource>
                        <targetPath>/</targetPath>
                        <!-- 用于指定需要复制的根目录，${project.build.directory}表示target目录 -->
                        <directory>${project.build.directory}</directory>
                        <!-- 用于指定需要复制的文件，${project.build.finalName}.jar就是打包后的target目录下的jar包名称　-->
                        <include>${project.build.finalName}.jar</include>
                    </resource>
                </resources>
            </configuration>
        </plugin>
    </plugins>
</build>
```

打包测试