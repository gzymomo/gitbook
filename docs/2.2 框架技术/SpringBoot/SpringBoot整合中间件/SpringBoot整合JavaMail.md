[TOC]

# 1、JavaMail的核心API
## 1.1 API功能图解
![](https://mmbiz.qpic.cn/mmbiz_jpg/uUIibyNXbAvDWfOB4w03o3JjxWyVBia32XCeK6I708vFt1SfheOsVSG1lbYeup8nlfR8IJWaVKu7XaSJnYA397dQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 1.2 API说明
### 1.2.1 Message 类:
javax.mail.Message 类是创建和解析邮件的一个抽象类
子类javax.mail.internet.MimeMessage ：表示一份电子邮件。 
发送邮件时，首先创建出封装了邮件数据的 Message 对象， 然后把这个对象传递给邮件发送Transport 类，执行发送。
接收邮件时，把接收到的邮件数据封装在Message 类的实例中，从这个对象中解析收到的邮件数据。

### 1.2.2 Transport 类
javax.mail.Transport 类是发送邮件的核心API 类
创建好 Message 对象后， 只需要使用邮件发送API 得到 Transport 对象， 然后把 Message 对象传递给 Transport 对象， 并调用它的发送方法， 就可以把邮件发送给指定的邮件服务器。

### 1.2.3 Store 类
javax.mail.Store 类是接收邮件的核心 API 类
实例对象代表实现了某个邮件接收协议的邮件接收对象，接收邮件时， 只需要得到 Store 对象， 然后调用 Store 对象的接收方法，就可以从指定的邮件服务器获得邮件数据，并把这些邮件数据封装到表示邮件的 Message 对象中。

### 1.2.4 Session 类：
javax.mail.Session 类定义邮件服务器的主机名、端口号、协议等
Session 对象根据这些信息构建用于邮件收发的 Transport 和 Store 对象， 以及为客户端创建 Message 对象时提供信息支持。

# 2、邮件服务器配置
以 smtp 为例
```yml
1、smtp.mxhichina.com
阿里云企业邮箱配置（账号+密码）
2、smtp.aliyun.com
阿里云个人邮箱配置（账号+密码）
3、smtp.163.com
网易邮箱配置（账号+授权码）
```
# 3、公共代码块
## 3.1 邮件通用配置
```java
package com.email.send.param;
/**
 * 邮箱发送参数配置
 */
public class EmailParam {
    /**
     * 邮箱服务器地址
     */
    // public static final String emailHost = "smtp.mxhichina.com" ; 阿里云企业邮箱配置（账号+密码）
    // public static final String emailHost = "smtp.aliyun.com" ; 阿里云个人邮箱配置（账号+密码）
    public static final String emailHost = "smtp.163.com" ; // 网易邮箱配置（账号+授权码）
    /**
     * 邮箱协议
     */
    public static final String emailProtocol = "smtp" ;
    /**
     * 邮箱发件人
     */
    public static final String emailSender = "xxxxxx@163.com" ;
    /**
     * 邮箱授权码
     */
    public static final String password = "authCode";
    /**
     * 邮箱授权
     */
    public static final String emailAuth = "true" ;
    /**
     * 邮箱昵称
     */
    public static final String emailNick = "知了一笑" ;
}
```
## 3.2 常用常量
```java
package com.email.send.param;
/**
 * 邮件发送类型
 */
public enum EmailType {
    EMAIL_TEXT_KEY("email_text_key", "文本邮件"),
    EMAIL_IMAGE_KEY("email_image_key", "图片邮件"),
    EMAIL_FILE_KEY("email_file_key", "文件邮件");
    private String code;
    private String value;
    EmailType(String code, String value) {
        this.code = code;
        this.value = value;
    }
    public static String getByCode(String code) {
        EmailType[] values = EmailType.values();
        for (EmailType emailType: values) {
            if (emailType.code.equalsIgnoreCase(code)) {
                return emailType.value;
            }
        }
        return null;
    }
    // 省略 get set
}
```
# 4、邮件发送封装
## 4.1 纯文本邮件发送
1. 代码封装
```java
/**
 * 邮箱发送模式01：纯文本格式
 */
public static void sendEmail01(String receiver, String title, String body) throws Exception {
    Properties prop = new Properties();
    prop.setProperty("mail.host", EmailParam.emailHost);
    prop.setProperty("mail.transport.protocol", EmailParam.emailProtocol);
    prop.setProperty("mail.smtp.auth", EmailParam.emailAuth);
    //使用JavaMail发送邮件的5个步骤
    //1、创建session
    Session session = Session.getInstance(prop);
    //开启Session的debug模式，这样就可以查看到程序发送Email的运行状态
    session.setDebug(true);
    //2、通过session得到transport对象
    Transport ts = session.getTransport();
    //3、使用邮箱的用户名和密码连上邮件服务器，发送邮件时，发件人需要提交邮箱的用户名和密码给smtp服务器，用户名和密码都通过验证之后才能够正常发送邮件给收件人。
    ts.connect(EmailParam.emailHost, EmailParam.emailSender, EmailParam.password);
    //4、创建邮件
    // Message message = createEmail01(session,receiver,title,body);
    Message message = createEmail01(session, receiver, title, body);
    //5、发送邮件
    ts.sendMessage(message, message.getAllRecipients());
    ts.close();
}
/**
 * 创建文本邮件
 */
private static MimeMessage createEmail01(Session session, String receiver, String title, String body)
throws Exception {
    //创建邮件对象
    MimeMessage message = new MimeMessage(session);
    //指明邮件的发件人
    String nick = javax.mail.internet.MimeUtility.encodeText(EmailParam.emailNick);
    message.setFrom(new InternetAddress(nick + "<" + EmailParam.emailSender + ">"));
    //指明邮件的收件人
    message.setRecipient(Message.RecipientType.TO, new InternetAddress(receiver));
    //邮件的标题
    message.setSubject(title);
    //邮件的文本内容
    message.setContent(body, "text/html;charset=UTF-8");
    //返回创建好的邮件对象
    return message;
}
```

## 4.2 文本+图片+附件邮件
1. 代码封装
```java
/**
 * 邮箱发送模式02：复杂格式
 */
public static void sendEmail02(String receiver, String title, String body) throws Exception {
    Properties prop = new Properties();
    prop.setProperty("mail.host", EmailParam.emailHost);
    prop.setProperty("mail.transport.protocol", EmailParam.emailProtocol);
    prop.setProperty("mail.smtp.auth", EmailParam.emailAuth);
    //使用JavaMail发送邮件的5个步骤
    //1、创建session
    Session session = Session.getInstance(prop);
    //开启Session的debug模式，这样就可以查看到程序发送Email的运行状态
    session.setDebug(true);
    //2、通过session得到transport对象
    Transport ts = session.getTransport();
    //3、使用邮箱的用户名和密码连上邮件服务器，发送邮件时，发件人需要提交邮箱的用户名和密码给smtp服务器，用户名和密码都通过验证之后才能够正常发送邮件给收件人。
    ts.connect(EmailParam.emailHost, EmailParam.emailSender, EmailParam.password);
    //4、创建邮件
    // Message message = createEmail01(session,receiver,title,body);
    Message message = createEmail02(session, receiver, title, body);
    //5、发送邮件
    ts.sendMessage(message, message.getAllRecipients());
    ts.close();
}
private static MimeMessage createEmail02(Session session, String receiver, String title, String body)
throws Exception {
    //创建邮件对象
    MimeMessage message = new MimeMessage(session);
    //指明邮件的发件人
    String nick = javax.mail.internet.MimeUtility.encodeText(EmailParam.emailNick);
    message.setFrom(new InternetAddress(nick + "<" + EmailParam.emailSender + ">"));
    //指明邮件的收件人
    message.setRecipient(Message.RecipientType.TO, new InternetAddress(receiver));
    //邮件的标题
    message.setSubject(title);
    //文本内容
    MimeBodyPart text = new MimeBodyPart();
    text.setContent(body, "text/html;charset=UTF-8");
    //图片内容
    MimeBodyPart image = new MimeBodyPart();
    image.setDataHandler(new DataHandler(new FileDataSource("ware-email-send/src/gzh.jpg")));
    image.setContentID("gzh.jpg");
    //附件内容
    MimeBodyPart attach = new MimeBodyPart();
    DataHandler file = new DataHandler(new FileDataSource("ware-email-send/src/gzh.zip"));
    attach.setDataHandler(file);
    attach.setFileName(file.getName());
    //关系:正文和图片
    MimeMultipart multipart1 = new MimeMultipart();
    multipart1.addBodyPart(text);
    multipart1.addBodyPart(image);
    multipart1.setSubType("related");
    //关系:正文和附件
    MimeMultipart multipart2 = new MimeMultipart();
    multipart2.addBodyPart(attach);
    // 全文内容
    MimeBodyPart content = new MimeBodyPart();
    content.setContent(multipart1);
    multipart2.addBodyPart(content);
    multipart2.setSubType("mixed");
    // 封装 MimeMessage 对象
    message.setContent(multipart2);
    message.saveChanges();
    // 本地查看文件格式
    message.writeTo(new FileOutputStream("F:\\MixedMail.eml"));
    //返回创建好的邮件对象
    return message;
}
```

## 4.3 实现异步发送
配置异步执行线程
```java
package com.email.send.util;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;
import java.util.concurrent.Executor;
import java.util.concurrent.ThreadPoolExecutor;
/**
 * 定义异步任务执行线程池
 */
@Configuration
public class TaskPoolConfig {
    @Bean("taskExecutor")
    public Executor taskExecutor () {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        // 核心线程数10：线程池创建时候初始化的线程数
        executor.setCorePoolSize(10);
        // 最大线程数20：线程池最大的线程数，只有在缓冲队列满了之后才会申请超过核心线程数的线程
        executor.setMaxPoolSize(15);
        // 缓冲队列200：用来缓冲执行任务的队列
        executor.setQueueCapacity(200);
        // 允许线程的空闲时间60秒：当超过了核心线程数之外的线程在空闲时间到达之后会被销毁
        executor.setKeepAliveSeconds(60);
        // 线程池名的前缀：设置好了之后可以方便定位处理任务所在的线程池
        executor.setThreadNamePrefix("taskExecutor-");
        /*
        线程池对拒绝任务的处理策略：这里采用了CallerRunsPolicy策略，
        当线程池没有处理能力的时候，该策略会直接在 execute 方法的调用线程中运行被拒绝的任务；
        如果执行程序已关闭，则会丢弃该任务
         */
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        // 设置线程池关闭的时候等待所有任务都完成再继续销毁其他的Bean
        executor.setWaitForTasksToCompleteOnShutdown(true);
        // 设置线程池中任务的等待时间，如果超过这个时候还没有销毁就强制销毁，以确保应用最后能够被关闭，而不是阻塞住。
        executor.setAwaitTerminationSeconds(600);
        return executor;
    }
}
```

2. 业务方法使用
注意两个注解

- @Component
- @Async("taskExecutor")
```java
@Component
@Service
public class EmailServiceImpl implements EmailService {
    @Async("taskExecutor")
    @Override
    public void sendEmail(String emailKey, SendEmailModel model) {
        try{
            // 异步执行
            Thread.sleep(1000);
            String textBody = EmailUtil.convertTextModel(BodyType.getByCode(emailKey),"知了","一笑");
            // 发送文本邮件
            EmailUtil.sendEmail01(model.getReceiver(), EmailType.getByCode(emailKey),textBody);
            // 发送复杂邮件:文本+图片+附件
            String body = "自定义图片：<img src='cid:gzh.jpg'/>,网络图片：<img src='http://pic37.nipic.com/20140113/8800276_184927469000_2.png'/>";
            // EmailUtil.sendEmail02(model.getReceiver(),"文本+图片+附件",body);
        } catch (Exception e){
            e.printStackTrace();
        }

    }
}
```

3. 启动类注解
- @EnableAsync
```java
@EnableAsync
@SpringBootApplication
public class EmailApplication {
    public static void main(String[] args) {
        SpringApplication.run(EmailApplication.class,args) ;
    }
}
```