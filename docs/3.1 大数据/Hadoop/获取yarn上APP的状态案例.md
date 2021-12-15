[TOC]

监控 yarn 上 spark 或者 mr 应用的存活状态

1、pom文件，添加yarn相关的配置
```xml

<!-- https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-common -->
    <dependency>
      <groupId>org.apache.hadoop</groupId>
      <artifactId>hadoop-common</artifactId>
      <version>2.7.4</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-client -->
    <dependency>
      <groupId>org.apache.hadoop</groupId>
      <artifactId>hadoop-client</artifactId>
      <version>2.7.4</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-yarn-api -->
    <dependency>
      <groupId>org.apache.hadoop</groupId>
      <artifactId>hadoop-yarn-api</artifactId>
      <version>2.7.4</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-yarn-client -->
    <dependency>
      <groupId>org.apache.hadoop</groupId>
      <artifactId>hadoop-yarn-client</artifactId>
      <version>2.7.4</version>
    </dependency>
```
2、将yarn-site.xml配置文件放到resources目录下：
![](https://mmbiz.qpic.cn/mmbiz_png/adI0ApTVBFW9ic3icRb5tjqCF0dUsf7QhFibINGlRl3DK5V4POv64RIh5RwuogOPq1OuhWGyhBL2WynEBC2wjG9EA/640?tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

3、通过yarnclient获取resourcemanager上 spark 或者 mapreduce的状态
```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.yarn.api.records.ApplicationReport;
import org.apache.hadoop.yarn.api.records.YarnApplicationState;
import org.apache.hadoop.yarn.client.api.YarnClient;
import org.apache.hadoop.yarn.conf.YarnConfiguration;
import org.apache.hadoop.yarn.exceptions.YarnException;

import java.io.IOException;
import java.util.EnumSet;
import java.util.List;


public class client {
    public static void main(String[] args){
        Configuration conf = new YarnConfiguration();
        YarnClient yarnClient = YarnClient.createYarnClient();
        yarnClient.init(conf);
        yarnClient.start();
        try {
            List<ApplicationReport> applications = yarnClient.getApplications(EnumSet.of(YarnApplicationState.RUNNING, YarnApplicationState.FINISHED));
            System.out.println("ApplicationId ============> "+applications.get(0).getApplicationId());
            System.out.println("name ============> "+applications.get(0).getName());
            System.out.println("queue ============> "+applications.get(0).getQueue());
            System.out.println("queue ============> "+applications.get(0).getUser());
        } catch (YarnException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        yarnClient.stop();
    }
}
```
通过YarnApplicationState设置状态，来过滤调一些我们不需要的任务状态。状态列表如下：
```java
public enum YarnApplicationState {
  /** Application which was just created. */
  NEW,

  /** Application which is being saved. */
  NEW_SAVING,

  /** Application which has been submitted. */
  SUBMITTED,

  /** Application has been accepted by the scheduler */
  ACCEPTED,

  /** Application which is currently running. */
  RUNNING,

  /** Application which finished successfully. */
  FINISHED,

  /** Application which failed. */
  FAILED,

  /** Application which was terminated by a user or admin. */
  KILLED
}
```
上述demo监控的是spark streaming 的状态，运行结果如下：
![](https://mmbiz.qpic.cn/mmbiz_png/adI0ApTVBFW9ic3icRb5tjqCF0dUsf7QhFWk8z6TrtiatrchsOXMZ7Sn5MQJYy3zwwFlib7RaxA6GmpjKZnFsRACxA/640?tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)


通过app name字段可以获取到存活的 spark 等任务，然后通过比对我们要监控的任务列表，不存在的发出告警即可。
对于 spark streaming 或者 spark其他任务，可以通过一个配置来制定spark 任务在yarn上显示的name，设置的参数是:
`new SparkConf().setAppName(this.getClass.getName)`。

this.getClass.getName该方式在yarn-client和 yarn-cluster有稍微的区别。