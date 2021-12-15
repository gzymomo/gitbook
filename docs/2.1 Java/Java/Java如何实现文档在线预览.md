# 一、概述

​	公司主要负责政府方面的项目，也包括一些OA的项目。OA项目当中，最常见的功能就是需要和各种Excel，word文档打交道，比如需要实现文档的下载，打印，以及实时预览功能。现有需求，需要用Java实现，Word文档的在线预览功能，格式包含.doc，.docx，工具不仅word，还要兼容国产wps软件。



​	该部分内容顺理成章的就成为我的任务，负责研究，然后形成一个技术方案来提供给开发人员。

​	在某度，某oogle上疯狂了解的相关内容后，最终定位于永中DCS，以此篇文章来提供技术方案。

# 二、Java实现文档在线预览

## 2.1 永中DCS简介

> 永中文档在线预览软件（Document Conversion Service，简称DCS）是通过解析常用办公文档的格式，提供不同文档格式间的相互转换，实现常用格式文档的阅读等服务。 DCS能直接部署在Windows或Linux网络操作系统上，与Web 服务器、邮件服务器等配合，提供Office文档阅读及批量转换功能。永中DCS支持阅读和转换的文档格式非常丰富，涵盖微软Office 97~2016、PDF、UOF和OFD等常用文档格式，同时可根据用户需求进行特定格式的合作。

我们通过官方的资料可以发现，其能提供的主要功能有：

1. 解析文档
2. 文档之间的格式转换
3. 在线预览
4. 多平台兼容（Windows，Linux）皆可部署。
5. 良好的扩展性，可兼容其他Web服务器，邮件服务器等。
6. 多方文档的支持（微软的office，PDF等多种格式）

## 2.2 使用方式

​	永中文档转换组件和DCS服务器两种使用方式：前者是jar包调用（DCC），主要适合java语言调用；后者为http请求方式调用，支持各种开发语言以标准Web服务方式提供调用，如搭配负载均衡服务器，可部署到多台服务器上并发使用，实现集群部署。 在输出网页效果方面，新增高清版网页效果支持。可在网页中展示与Office中显示一致的原版式、支持无极缩放和复制文本内容的高清版面效果。

​	具体支持的格式有：

| 源格式     | 目标格式（图像包括JPG、PNG、BMP、TIFF等） | 说明                     |
| ---------- | ----------------------------------------- | ------------------------ |
| XLS、XLSX  | HTML/PDF/UOF/TXT/CSV/图像/EIO             | 微软文档格式             |
| PPT、PPTX  | HTML/PDF/UOF/TXT/图像/EIO                 | 微软文档格式             |
| DOC、DOCX  | HTML/PDF/UOF/TXT/图像/EIO                 | 微软文档格式             |
| RTF        | HTML/PDF/UOF/TXT/图像/EIO                 | RTF文档格式              |
| EIO        | HTML/PDF/UOF/TXT/图像/DOC/XLS/PPT         | 永中Office格式           |
| UOF、UOS   | HTML/PDF/TXT/图像/DOC/XLS/PPT             | 中文办公软件文档标准格式 |
| XML        | HTML/PDF/UOF/TXT/图像/DOC/XLS/PPT         | XML文档格式              |
| PDF        | PNG                                       | Adobe阅读PDF文档格式     |
| Office文档 | OFD                                       | 中文版式文档格式         |

由此，我们可以发现，几乎在日常工作中所使用的文档，几乎全部涵盖在了其中。



了解了此款软件后，我大呼一口气，领导交代的任务，这下差不多已经有了交代，接下来就开始实战了。

## 2.3 centos安装部署DCS服务

​	因为公司项目主要发布在centos服务器上，所以需要在centos部署DCS服务。



首先，服务器需要JDK环境，至少JDK8版本以上。

在Oracle官网下载JDK8到本地，下载完成后，通过ftp方式或者rz 上传文件的方式，将jdk-8u60-linux-x64.tar.gz 文件上传至服务器。

然后通过`tar -zxvf jdk-8u60-linux-x64.tar.gz`解压文件。并通过mv命令重命名文件为jdk8。



此处我解压在了/opt路径下。接下来，需要配置Java环境。



### 编辑Java环境

`vi /etc/profile`

新增如下内容：

```bash
JAVA_HOME=/opt/jdk8   # 此处
PATH=$JAVA_HOME/bin:$PATH
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```

配置文件生效

```bash
source /etc/profile
```

检测是否安装成功：

```bash
java –version
```

若出现如下界面，则证明jdk配置成功！

![img](https://img-blog.csdn.net/20180813112351899?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RocjIwMTQ5OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 安装部署**Tomcat**

在tomcat官网下载apache-tomcat-8.0.26.tar.gz文件。

下载完成后，通过ftp方式或者rz 上传文件的方式，将文件上传至服务器。



解压文件：

```bash
tar -zxvf apache-tomcat-8.0.26.tar.gz
```

### **部署DCS工程**

将永中DCS工程目录复制到tomcat 的webapp目录下。

修改${tomcat.home}\conf\server.xml文件.在Host节点下增加如下参考代码:
<Context docBase="指向项目的根目录所在的路径" path="虚拟目录名" reloadable="true"/ >
根据需要修改项目中 ${dcs.web}\WEB-INF\config.properties和log4j.properties的配置。需要在目录/usr/X11R6/lib/X11/fonts/TrueType下加入字体文件。



启动tomcat后，访问http://localhost:8080/dcs.web 即可看到“在线文档预览示例”页面。

### **安装nginx**

```bash
tar -zxvf nginx-1.9.2.tar.gz
```

进入 nginx 文件夹 cd nginx-1.9.2
执行 ./configure 可能会遇到系统缺少库
问题：

1. ./configure: err: C compiler cc is not found
   缺少gc++库文件

```bash
su root
cd /
yum -y install gcc gcc-c++ autoconf automak
```

等待 complete

2. ./configure: error: the HTTP rewrite module requires the PCRE library

```bash
su root
yum -y intall pcre pcre-devel
```

等待complete

3. ./configure:error: the HTTP gzip module requires the zlib library

```bash
yum -y install zlib zlib-devel
```

解决所有问题后 再次 解决所有问题后：

```bash
./configure
make
make install
```

安装完成后对/usr/local/nginx/conf/nginx.conf 配置文件进行所需配置。



## 2.4 接口说明

### **上传文档**

请求地址：
POST http://服务器地址/upload
例如:http://localhost:8080/dcs.web/upload
上传您要在线预览的文档。
参数：
file - 文件作为multi-part POST请求参数，用于上传本地文档 (必须)
convertType - 转换类型参数(必须)

### **URL预览文档**

请求地址：
POST http://服务器地址/onlinefile
参数：
downloadUrl - 文档URL，用于预览网络文档。此URL应为编码（UTF-8 URL-encoded）后的URL。
convertType - 转换类型参数

### **服务器本地转换**

请求地址：
POST http://服务器地址/convert
参数：
inputDir - 文档在服务器上的路径（相对于配置文件中input的相对路径）
mergeInput - 使用文档合并功能时，需要合并的文档路径同inputDir
convertType - 转换类型参数



### convertType参数取值说明

```bash
0-----文档格式到高清html的转换
1-----文档格式到html的转换
2-----文档格式到txt的转换
3-----文档格式到pdf的转换
4-----文档格式到gif的转换
5-----文档格式到png的转换
6-----文档格式到jpg的转换
7-----文档格式到tiff的转换
8-----文档格式到bmp的转换
9-----pdf文档格式到gif的转换
10----pdf文档格式到png的转换
11----pdf文档格式到jpg的转换
12----pdf文档格式到tiff的转换
13----pdf文档格式到bmp的转换
14----pdf文档格式到html的转换
15----html文档格式到微软文档格式的转换
16----文档转换多个SVG返回分页加载页面(模版)
17----tif文件转成html
18----文档转换多个SVG
19----压缩文件到html的转换(模版)
20----PDF文件到html的转换(模版)
21----ofd文件到html的转换(模版)
22----两个doc文档合并
23----图片到html的转换
24----pdf文档格式到txt的转换
25----文档按页转换（高清版）
26----文档按页转换（标准版）
27----获取文档页码（MS文档）
28----获取pdf页码（PDF文件）
29----文档到ofd的转换
30----文档到html（图片）的转换
31----多个pdf文档合并
32----图片到pdf的转换
33----文档到文档的转换
34----pdf到pdf的转换
35----tif到html的转换(模板)
```

## 2.5 Java实现文档在线预览

需要添加相关依赖，附上pom.xml依赖内容：

```xml
 <!-- https://mvnrepository.com/artifact/commons-logging/commons-logging -->
<dependency>
    <groupId>commons-logging</groupId>
    <artifactId>commons-logging</artifactId>
    <version>1.1.1</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.apache.httpcomponents/httpclient -->
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.2</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.apache.httpcomponents/httpcore -->
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpcore</artifactId>
    <version>4.4.1</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.apache.httpcomponents/httpmime -->
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpmime</artifactId>
    <version>4.5</version>
</dependency>
```

在IDEA中，新建Test方法进行测试：

```java
import java.io.BufferedReader;
import java.io.File;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.URL;
import java.net.URLConnection;
import java.nio.charset.Charset;
import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.HttpStatus;
import org.apache.http.ParseException;
import org.apache.http.client.HttpClient;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.entity.mime.HttpMultipartMode;
import org.apache.http.entity.mime.MultipartEntity;
import org.apache.http.entity.mime.content.FileBody;
import org.apache.http.entity.mime.content.StringBody;
import org.apache.http.impl.client.DefaultHttpClient;
import org.apache.http.util.EntityUtils;
/**
* @Description: DCS文档转换服务Java调用代码示例
*/
public class Test {
	/**
	* 向指定 URL 发送POST方法的请求
	* @param url  发送请求的 URL
	* @param param 请求参数，请求参数应该是 name1=value1&name2=value2 的形式。
	* @return 所代表远程资源的响应结果
	*/
	public static String sendPost(String url, String param) {
		PrintWriter out = null;
		BufferedReader in = null;
		String result = "";
		try {
			URL realUrl = new URL(url);
			// 打开和URL之间的连接
			URLConnection conn = realUrl.openConnection();
			conn.setRequestProperty("Accept-Charset", "UTF-8");
			// 设置通用的请求属性
			conn.setRequestProperty("accept", "*/*");
			conn.setRequestProperty("connection", "Keep-Alive");
			conn.setRequestProperty("user-agent", "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1;SV
			1)");
			// 发送POST请求必须设置如下两行
			conn.setDoOutput(true);
			conn.setDoInput(true);
			// 获取URLConnection对象对应的输出流
			out = new PrintWriter(conn.getOutputStream());
			// 发送请求参数
			out.print(param);
			// flush输出流的缓冲
			out.flush();
			// 定义BufferedReader输入流来读取URL的响应
			in = new BufferedReader(new InputStreamReader(conn.getInputStream()));
			String line;
			while ((line = in.readLine()) != null) {
				result += line;
			}
		} catch (Exception e) {
			System.out.println("发送 POST 请求出现异常！" + e);
			e.printStackTrace();
		}
		// 使用finally块来关闭输出流、输入流
		finally {
			try {
				if (out != null) {
					out.close();
				}
				if (in != null) {
					in.close();
				}
			} catch (IOException ex) {
				ex.printStackTrace();
			}
		}
		return result;
	}
	/**
	* 向指定 URL 上传文件POST方法的请求
	*
	* @param url      发送请求的 URL
	* @param filepath 文件路径
	* @param type     转换类型
	* @return 所代表远程资源的响应结果, json数据
	*/
	public static String SubmitPost(String url, String filepath, String type) {
		String requestJson = "";
		HttpClient httpclient = new DefaultHttpClient();
		try {
			HttpPost httppost = new HttpPost(url);
			FileBody file = new FileBody(new File(filepath));
			MultipartEntity reqEntity = new MultipartEntity(HttpMultipartMode.BROWSER_COMPATIBLE, null,
			Charset.forName("UTF-8"));
			reqEntity.addPart("file", file); // file为请求后台的File upload;属性
			reqEntity.addPart("convertType", new StringBody(type, Charset.forName("UTF-8")));
			httppost.setEntity(reqEntity);
			HttpResponse response = httpclient.execute(httppost);
			int statusCode = response.getStatusLine().getStatusCode();
			if (statusCode == HttpStatus.SC_OK) {
				HttpEntity resEntity = response.getEntity();
				requestJson = EntityUtils.toString(resEntity);
				EntityUtils.consume(resEntity);
			}
		} catch (ParseException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
			// requestJson = e.toString();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
			// requestJson = e.toString();
		} finally {
			try {
				httpclient.getConnectionManager().shutdown();
			} catch (Exception ignore) {
			}
		}
		return requestJson;
	}
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		// 文件上传转换
		String convertByFile = SubmitPost("http://dcs.yozosoft.com:80/upload", "C:/doctest.docx", "1");
		// 网络地址转换
		String convertByUrl  = sendPost("http://dcs.yozosoft.com:80/onlinefile", "downloadUrl=http://img.
		iyocloud.com:8000/doctest.docx&convertType=1");
		System.out.println(convertByFile);
		System.out.println(convertByUrl);
	}
}
```

执行该Test的main方法之后，在控制台会输出转换后的地址：

```java
{"result":0,"data":["http://dcs.yozosoft.com/view/2020/09/08/xxx.html"],"message":"转换成功","type":1}
```

点击该地址后，即可在线预览word文件。