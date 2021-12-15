[TOC]

# 1、pom.xml
```xml
  <dependencies>
        <dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpclient</artifactId>
            <version>4.5.6</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!--打包的插件-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>2.6</version>
                <configuration>
                    <archive>
                        <manifest>
                            <mainClass>com.dongtang.JavaDemo</mainClass>
                        </manifest>
                    </archive>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                </configuration>
                <!--这段配置让我们直接点击 maven package 就可以完成打包-->
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>

            <!-- java编译插件 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

# 2、FileIOUtil.java
```java
package com.dongtang.utils;

import java.io.*;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class FileIOUtil {

    /**
     * 将文件按行读取,最终读到List中,List中的一个元素便是一行,自动关闭流
     *
     * @param filePath 文件路径
     */
    public static List<String> readTextToList(String filePath) throws IOException {
        File file = new File(filePath);
        BufferedReader reader = new BufferedReader(new InputStreamReader(new FileInputStream(file)));

        ArrayList<String> list = new ArrayList<>();
        while (true) {
            String line = reader.readLine();
            if (line != null) {
                list.add(line);
            } else {
                break;
            }
        }
        reader.close();
        return list;
    }

    /**
     * 将文件读取成一个String,自动关闭流
     *
     * @param filePath 文件路径
     */
    public static String readTextToString(String filePath) throws IOException {
        File file = new File(filePath);
        FileInputStream fileInputStream = new FileInputStream(file);
        String contentFromStream = getContentFromStream(fileInputStream);
        fileInputStream.close();
        return contentFromStream;
    }

    /**
     * 从IO流中读取全部内容,返回String
     */
    public static String getContentFromStream(InputStream inputStream) throws IOException {
        BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));

        String line;
        StringBuilder result = new StringBuilder();
        while ((line = reader.readLine()) != null) {
            result.append(line);
        }
        reader.close();
        return result.toString();
    }

    /**
     * 读取文件到byte[]中,自动关闭流
     *
     * @param filePath 文件路径
     */
    public static byte[] readTextToBytes(String filePath) throws IOException {
        byte[] tempBytes = new byte[1024];
        byte[] returnBytes = new byte[0];

        File file = new File(filePath);
        FileInputStream inputStream = new FileInputStream(file);
        int i;
        int count = 0;

        while (((i = inputStream.read(tempBytes)) != -1)) {
            count = i + count;

            int length = returnBytes.length;
            returnBytes = Arrays.copyOf(returnBytes, count);

            System.arraycopy(tempBytes, 0, returnBytes, length, i);
        }
        inputStream.close();
        return returnBytes;
    }

    /**
     * 向文件中写入东西
     *
     * @param str      想要写入的内容
     * @param filePath 文件路径
     * @param isAppend 是否追加
     */
    public static void writeToFile(String str, String filePath, boolean isAppend) throws IOException {
        File file = new File(filePath);
        createFile(filePath, false);

        FileOutputStream outputStream = new FileOutputStream(file, isAppend);
        byte[] bytes = str.getBytes();
        outputStream.write(bytes);
        outputStream.close();
    }

    /**
     * 向文件中写入东西
     *
     * @param bytes    想要写入的内容
     * @param filePath 文件路径
     */
    public static void writeToFile(byte[] bytes, String filePath) throws IOException {
        File file = new File(filePath);
        createFile(filePath, false);

        FileOutputStream outputStream = new FileOutputStream(file);
        outputStream.write(bytes);
        outputStream.close();
    }

    /**
     * 自动创建目录,包括父目录
     * @param path 文件路径
     * @param isDirectory 文件是否是个文件夹
     */
    public static void createFile(String path, boolean isDirectory) throws IOException {
        File file = new File(path);

        if (isDirectory) {
            if (!file.exists()) {
                boolean mkdirs = file.mkdirs();
                if (mkdirs) {
                    System.out.println("文件夹不存在,自动创建成功！");
                } else {
                    System.out.println("文件夹不存在,自动创建失败！");
                }
            }
        } else {
            File directory = file.getParentFile();
            if (!directory.exists()) {
                boolean mkdirs = directory.mkdirs();
                if (mkdirs) {
                    System.out.println("父目录不存在,自动创建成功！");
                } else {
                    System.out.println("父目录不存在,自动创建失败！");
                }

                if (!file.exists()) {
                    boolean mkdirsFlag = file.createNewFile();
                    if (mkdirsFlag) {
                        System.out.println("文件不存在,自动创建成功！");
                    } else {
                        System.out.println("文件不存在,自动创建失败！");
                    }
                }
            }
        }
    }

	/**
     * 复制文件
     */
    public static void copyFile(String sourceFilePath, String targetFilePath) throws IOException {
        byte[] bytes = readTextToBytes(sourceFilePath);
        writeToFile(bytes, targetFilePath);
    }
}
```

# 3、源码

```java

import com.dongtang.utils.FileIOUtil;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.entity.StringEntity;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;

import java.io.InputStream;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.ArrayList;
import java.util.List;
import java.util.Timer;
import java.util.TimerTask;

public class JavaDemo {
    public static void main(String[] args) {

        long howLong = 1000 * 60 * 15;

        List<String> appNameList = new ArrayList<>();
        appNameList.add("DzStreamingInput");
        appNameList.add("ZyStreamingInput");
        appNameList.add("HcGPSLauncher");

        Timer timer = new Timer();
        timer.schedule(new TimerTask() {
            @Override
            public void run() {
                try {
                    Runtime runtime = Runtime.getRuntime();
                    Process exec = runtime.exec("jps -m");
                    InputStream input = exec.getInputStream();
                    String content = FileIOUtil.getContentFromStream(input);
                    input.close();

                    for (String appName : appNameList) {
                        if (!content.contains(appName)) {
                            CloseableHttpClient httpClient = HttpClients.createDefault();
                            String url = "https://oapi.dingtalk.com/robot/send?access_token=9f33082b9db98c1cf215fe1348ad8db091777701aa435cde47d0af4a87e0460e";
                            HttpPost post = new HttpPost(url);

                            LocalDateTime time = LocalDateTime.now();
                            DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
                            String format = formatter.format(time);
                            String json = "{\"msgtype\": \"text\", \"text\": {\"content\": \"alert:Spark Streaming Service " + appName + " stopped ! sendTime:" + format + "\"}}";
                            StringEntity stringEntity = new StringEntity(json);
                            post.setEntity(stringEntity);
                            post.setHeader("Content-Type", "application/json");

                            CloseableHttpResponse response = httpClient.execute(post);
                        }
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }, 0, howLong);

    }
}
```