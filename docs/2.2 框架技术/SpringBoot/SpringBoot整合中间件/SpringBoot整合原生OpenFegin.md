[冰河团队](https://home.cnblogs.com/u/binghe001/)：[SpringBoot整合原生OpenFegin的坑（非SpringCloud）](https://www.cnblogs.com/binghe001/p/13876226.html)

## 项目集成OpenFegin

### 集成OpenFegin依赖

首先，我先跟大家说下项目的配置，整体项目使用的SpringBoot版本为2.2.6，原生的OpenFegin使用的是11.0，我们通过如下方式在pom.xml中引入OpenFegin。

```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <skip_maven_deploy>false</skip_maven_deploy>
    <java.version>1.8</java.version>
    <openfegin.version>11.0</openfegin.version>
</properties>
<dependencies>
    <dependency>
        <groupId>io.github.openfeign</groupId>
        <artifactId>feign-core</artifactId>
        <version>${openfegin.version}</version>
    </dependency>

    <dependency>
        <groupId>io.github.openfeign</groupId>
        <artifactId>feign-jackson</artifactId>
        <version>${openfegin.version}</version>
    </dependency>
</dependencies>
```

这里，我省略了一些其他的配置项。

接下来，我就开始在我的项目中使用OpenFegin调用远程服务了。具体步骤如下。

### 实现远程调用

首先，创建OpenFeignConfig类，配置OpenFegin默认使用的Contract。

```java
@Configuration
public class OpenFeignConfig {
	@Bean
	public Contract useFeignAnnotations() {
		return new Contract.Default();
	}
}
```

接下来，我们写一个通用的获取OpenFeign客户端的工厂类，这个类也比较简单，本质上就是以一个HashMap来缓存所有的FeginClient，这个的FeginClient本质上就是我们自定义的Fegin接口，缓存中的Key为请求连接的基础URL，缓存的Value就是我们定义的FeginClient接口。

```java
public class FeginClientFactory {
	
	/**
	 * 缓存所有的Fegin客户端
	 */
	private volatile static Map<String, Object> feginClientCache = new HashMap<>();
	
	/**
	 * 从Map中获取数据
	 * @return 
	 */
	@SuppressWarnings("unchecked")
	public static <T> T getFeginClient(Class<T> clazz, String baseUrl){
		if(!feginClientCache.containsKey(baseUrl)) {
			synchronized (FeginClientFactory.class) {
				if(!feginClientCache.containsKey(baseUrl)) {
					T feginClient = Feign.builder().decoder(new JacksonDecoder()).encoder(new JacksonEncoder()).target(clazz, baseUrl);
					feginClientCache.put(baseUrl, feginClient);
				}
			}
		}
		return (T)feginClientCache.get(baseUrl);
	}
}
```

接下来，我们就定义一个FeginClient接口。

```java
public interface FeginClientProxy {
	@Headers("Content-Type:application/json;charset=UTF-8")
	@RequestLine("POST /user/login")
	UserLoginVo login(UserLoginVo loginVo);
}
```

接下来，我们创建SpringBoot的测试类。

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class IcpsWeightStarterTest {
	@Test
	public void testUserLogin() {
		ResponseMessage result = FeginClientFactory.getFeginClient(FeginClientProxy.class, "http://127.0.0.1").login(new UserLoginVo("zhangsan", "123456", 1));
		System.out.println(JsonUtils.bean2Json(result));
	}
}
```

一切准备就绪，运行测试。麻蛋，出问题了。主要的问题就是通过OpenFeign请求返回值LocalDateTime字段会发生异常！！！

注：此时异常时，我们在LocalDateTime字段上添加的注解如下所示。

```java
import java.time.LocalDateTime;
import com.baomidou.mybatisplus.annotation.FieldFill;
import com.baomidou.mybatisplus.annotation.TableField;
import com.fasterxml.jackson.annotation.JsonFormat;


@TableField(value = "CREATE_TIME", fill = FieldFill.INSERT)
@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", locale = "zh", timezone = "GMT+8")
private LocalDateTime createTime;
```

## 解决问题

### 问题描述

SpringBoot通过原生OpenFeign客户端调用HTTP接口，如果返回值中包含LocalDateTime类型（包括其他JSR-310中java.time包的时间类），在客户端可能会出现反序列化失败的错误。错误信息如下：

```bash
 Caused by:com.fasterxml.jackson.databind.exc.InvalidDefinitionException: Cannot construct instance of `java.time.LocalDateTime` (no Creators, like default construct, exist): no String-argument constructor/factory method to deserialize from String value ('2020-10-07T11:04:32')
```

### 问题分析

从客户端调用fegin，也是相当于URL传参就相当于经过一次JSON转换，数据库取出‘2020-10-07T11:04:32’数据这时是时间类型，进过JSON之后就变成了String类型，T就变成了字符不再是一个特殊字符，因此String的字符串“2020-10-07T11:04:32”反序列化就会失败。

### 问题解决

在项目中增加依赖。

```xml
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jsr310</artifactId>
    <version>2.9.9</version>
</dependency>
```

注：如果是用的是SpringBoot，并且明确指定了SpringBoot版本，引入jackson-datatype-jsr310时，可以不用指定版本号。

接下来，在POJO类的LocalDateTime类型字段增加如下注解。

```java
import com.fasterxml.jackson.databind.annotation.JsonDeserialize;
import com.fasterxml.jackson.datatype.jsr310.deser.LocalDateTimeDeserializer;
```

添加后的效果如下所示。

```java
import java.time.LocalDateTime;
import com.baomidou.mybatisplus.annotation.FieldFill;
import com.baomidou.mybatisplus.annotation.TableField;
import com.fasterxml.jackson.annotation.JsonFormat;

import com.fasterxml.jackson.databind.annotation.JsonDeserialize;
import com.fasterxml.jackson.datatype.jsr310.deser.LocalDateTimeDeserializer;


@TableField(value = "CREATE_TIME", fill = FieldFill.INSERT)
@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", locale = "zh", timezone = "GMT+8")
@JsonDeserialize(using = LocalDateTimeDeserializer.class)
private LocalDateTime createTime;
```

此时，再次调用远程接口，问题解决。