# 一、WorkBench

## 1.1 WorkBench简介

WorkBench是KIE组件中的元素，也称为KIE-WB，是Drools-WB与JBPM-WB的结合体。它是一个**可视化的规则编辑器**。WorkBench其实就是一个war包，安装到tomcat中就可以运行。使用WorkBench可以在浏览器中创建数据对象、创建规则文件、创建测试场景并将规则部署到maven仓库供其他应用使用。

下载地址：https://download.jboss.org/drools/release/7.6.0.Final/kie-drools-wb-7.6.0.Final-tomcat8.war

注意：下载的war包需要安装到tomcat8中。

## 1.2 安装方式

软件安装时经常会涉及到软件版本兼容性的问题，所以需要明确各个软件的使用版本。

本课程使用的软件环境如下：

- 操作系统：Windows 10 64位
- JDK版本：1.8
- maven版本：3.5.4
- Tomcat版本：8.5

具体安装步骤：

第一步：配置Tomcat的环境变量CATALINA_HOME，对应的值为Tomcat安装目录

第二步：在Tomcat的bin目录下创建setenv.bat文件，内容如下：

```
CATALINA_OPTS="-Xmx512M \
    -Djava.security.auth.login.config=$CATALINA_HOME/webapps/kie-drools-wb/WEB-INF/classes/login.config \
    -Dorg.jboss.logging.provider=jdk"
```

第三步：将下载的WorkBench的war包改名为kie-drools-wb.war并复制到Tomcat的webapps目录下

第四步：修改Tomcat下conf/tomcat-users.xml文件

```xml
<?xml version='1.0' encoding='utf-8'?>
<tomcat-users xmlns="http://tomcat.apache.org/xml"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://tomcat.apache.org/xml tomcat-users.xsd"
              version="1.0">
  <!--定义admin角色-->
  <role rolename="admin"/>
  <!--定义一个用户，用户名为kie，密码为kie，对应的角色为admin角色-->
  <user username="kie" password="kie" roles="admin"/>
</tomcat-users>
```

第五步：下载以下三个jar包并复制到Tomcat的lib目录下

```
kie-tomcat-integration-7.10.0.Final.jar
javax.security.jacc-api-1.5.jar
slf4j-api-1.7.25.jar
```

第六步：修改Tomcat的conf/server.xml文件，添加Valve标签，内容为：

```xml
<Valve className="org.kie.integration.tomcat.JACCValve"/>
```

第七步：启动Tomcat并访问http://localhost:8080/kie-drools-wb，可以看到WorkBench的登录页面。使用前面在tomcat-users.xml文件中定义的用户进行登录即可

注意：这里我自己的是80端口，http://localhost/kie-drools-wb

![12](https://img-blog.csdnimg.cn/img_convert/c4b4534d5a8e07d3f8222122296322a8.png)

登录成功后进入系统首页：

![13](https://img-blog.csdnimg.cn/img_convert/fb817b7f034a6b485319a00cd3d0c333.png)

## 1.3 使用方式

### 1.3.1 创建空间、项目

WorkBench中存在空间和项目的概念。我们在使用WorkBench时首先需要创建空间（Space），在空间中创建项目，在项目中创建数据对象、规则文件等。

- 创建空间

  第一步：登录WorkBench后进行系统首页，点击首页中的Design区域进入项目列表页面：

  ![14](https://img-blog.csdnimg.cn/img_convert/554939632caa1d9cc0df9298d828e08c.png)

  如果是第一次登录还没有创建项目则无法看到项目

  第二步：点击左上角Spaces导航链接进入空间列表页面

  ![15](https://img-blog.csdnimg.cn/img_convert/5bd18f5999750202a09c427aec14a29f.png)

  第三步：点击右上角Add Space按钮弹出创建添加空间窗口

  ![16](https://img-blog.csdnimg.cn/img_convert/f3e3d15da638571ca80d865faa0f864e.png)

  录入空间名称，点击Save按钮则完成空间的创建，如下图：

  ![17](https://img-blog.csdnimg.cn/img_convert/593e6192e1c96dbf308b97b30852aaae.png)

- 创建项目

  前面已经提到，我们在WorkBench中需要先创建空间，在空间中才能创建项目。上面我们已经创建了一个空间itheima，现在需要住此空间中创建项目。

  第一步：点击itheima空间，进入此空间

  ![18](https://img-blog.csdnimg.cn/img_convert/fde9215a9051df0411239aa8870d89cb.png)

  可以看到当前空间中还没有项目

  第二步：点击Add Project按钮弹出添加项目窗口

  ![19](https://img-blog.csdnimg.cn/img_convert/479898ccaae2bbc831fc8a3c88f5d3eb.png)

  第三步：在添加项目窗口中录入项目名称（例如项目名称为pro1），点击Add按钮完成操作

  ![20](https://img-blog.csdnimg.cn/img_convert/9d15a12c02e94f1a8ab5c33e167f0d60.png)

  可以看到在完成项目创建后，系统直接跳转到了项目页面。要查看当前itheima空间中的所有项目，可以点击左上角itheima链接：

  ![21](https://img-blog.csdnimg.cn/img_convert/f774da05d77ff29e8f7fd6771986dde2.png)

### 1.3.2 创建数据对象

数据对象其实就是JavaBean，一般都是在drl规则文件中使用进行规则匹配。

第一步：在itheima空间中点击pro1项目，进入此项目页面

![22](https://img-blog.csdnimg.cn/img_convert/f641c6fa3ac25ec7132d9f0c9c9b9f27.png)

第二步：点击Create New Asset按钮选择“数据对象”

![23](https://img-blog.csdnimg.cn/img_convert/54c558ae18b175acdf9871f401acbeae.png)

第三步：在弹出的创建数据对象窗口中输入数据对象的名称，点击确定按钮完成操作

![24](https://img-blog.csdnimg.cn/img_convert/8a1092b10822b73c96a9c6d84926a633.png)

操作完成后可以看到如下：

![25](https://img-blog.csdnimg.cn/img_convert/714613fde78206da4f81127864f94e03.png)

第四步：点击“添加字段”按钮弹出新建字段窗口

![26](https://img-blog.csdnimg.cn/img_convert/7a8c0f7c5cfca02702e270226a576b2c.png)

第五步：在新建字段窗口中录入字段Id（其实就是属性名），选择类型，点击创建按钮完成操作

![27](https://img-blog.csdnimg.cn/img_convert/c37318028b34622a915edc04974983c6.png)

完成操作后可以看到刚才创建的字段：

![28](https://img-blog.csdnimg.cn/img_convert/f4a39c6a84d8a15974b851491e655c37.png)

可以点击添加字段按钮继续创建其他字段：

![29](https://img-blog.csdnimg.cn/img_convert/fe106c81966c849303cebffbb0454958.png)

注意添加完字段后需要点击右上角保存按钮完成保存操作：

![32](https://img-blog.csdnimg.cn/img_convert/f6fa2bc2e32f650448cf61485238331c.png)

点击源代码按钮可以查看刚才创建的Person对象源码：

![30](https://img-blog.csdnimg.cn/img_convert/fcd63112137fe7ffa40636d54160db0c.png)

点击左上角pro1项目链接，可以看到当前pro1项目中已经创建的各种类型的对象：

![31](https://img-blog.csdnimg.cn/img_convert/c566437a21081190876ce7d57c8b7b37.png)

### 1.3.3 创建DRL规则文件

第一步：在pro1项目页面点击右上角Create New Asset按钮，选择“DRL文件”，弹出创建DRL文件窗口

![33](https://img-blog.csdnimg.cn/img_convert/5a5b65e3e1ed8bf145eb62d7d0ead835.png)

第二步：在添加DRL文件窗口录入DRL文件名称，点击确定按钮完成操作

![34](https://img-blog.csdnimg.cn/img_convert/f853dc606565f343f16bf3aeae33c2e9.png)

第三步：上面点击确定按钮完成创建DRL文件后，页面会跳转到编辑DRL文件页面

![35](https://img-blog.csdnimg.cn/img_convert/58072352acf6ddd577363639aeac0466.png)

可以看到DRL规则文件页面分为两个部分：左侧为项目浏览视图、右侧为编辑区域，需要注意的是左侧默认展示的不是项目浏览视图，需要点击上面设置按钮，选择“资料库视图”和“显示为文件夹”，如下图所示：

![36](https://img-blog.csdnimg.cn/img_convert/9a046d40b581d34fcb93839a4e84bbbe.png)

第四步：在编辑DRL文件页面右侧区域进行DRL文件的编写，点击右上角保存按钮完成保存操作，点击检验按钮进行规则文件语法检查

![37](https://img-blog.csdnimg.cn/img_convert/a4901bcea150edce113ffdd084106d6c.png)

点击左上角pro1项目回到项目页面，可以看到此项目下已经存在两个对象，即person.drl规则文件和Person类：

![38](https://img-blog.csdnimg.cn/img_convert/06467329914680970df1882b3ab2c9d8.png)

### 1.3.4 创建测试场景

前面我们已经创建了Person数据对象和person规则文件，现在我们需要测试一下规则文件中的规则，可以通过创建测试场景来进行测试。

第一步：在项目页面点击Create New Asset按钮选择“测试场景”，弹出创建测试场景窗口

![39](https://img-blog.csdnimg.cn/img_convert/81e0ac812ad88dface39f83b9dd3e806.png)

第二步：在弹出的创建测试场景窗口中录入测试场景的名称，点击确定完成操作

![40](https://img-blog.csdnimg.cn/img_convert/f5aeefe8d0e8bf5c3c18f81fcaa6069a.png)

完成测试场景的创建后，页面会跳转到测试场景编辑页面，如下图：

![41](https://img-blog.csdnimg.cn/img_convert/722b287a9e56ff7323febd7c06432f2a.png)

第三步：因为我们编写的规则文件中需要从工作内存中获取Person对象进行规则匹配，所以在测试场景中需要准备Person对象给工作内存，点击“GIVEN”按钮弹出新建数据录入窗口，选择Person类，输入框中输入事实名称（名称任意），如下图

![42](https://img-blog.csdnimg.cn/img_convert/b4d2c4738b4d3010bea25b95ad694ef3.png)

第四步：录入事实名称后点击后面的添加按钮，可以看到Person对象已经添加成功

![43](https://img-blog.csdnimg.cn/img_convert/6587e453c568055bcf95ad3086880eb7.png)

第五步：我们给工作内存提供的Person对象还需要设置age属性的值，点击“添加字段”按钮弹出窗口，选择age属性

![44](https://img-blog.csdnimg.cn/img_convert/37251fbc1d0ca05c7980f9ec3574eb69.png)

点击确定按钮后可以看到字段已经添加成功：

![45](https://img-blog.csdnimg.cn/img_convert/dd9528a47a67ecac351f9062f2757452.png)

第六步：点击age属性后面的编辑按钮，弹出字段值窗口

![image-20200113154817582](https://img-blog.csdnimg.cn/img_convert/d5a5cf86bf846c07778b9239d75053b2.png)

第七步：在弹出的窗口中点击字面值按钮，重新回到测试场景页面，可以看到age后面出现输入框，可以为age属性设置值

![image-20200113155136957](https://img-blog.csdnimg.cn/img_convert/207207a537ac57442c096b9b2b82e8dd.png)

设置好age属性的值后点击保存按钮保存测试场景

第八步：点击右上角“运行测试场景”按钮进行测试

![image-20200113155332666](https://img-blog.csdnimg.cn/img_convert/2d3ea4f561fbac0cea6f62eabc853b03.png)

测试成功后可以查看WorkBench部署的Tomcat控制台：

![image-20200113155819517](https://img-blog.csdnimg.cn/img_convert/489ff0cb0b705aef4aee79816d877796.png)

### 1.3.5 设置KieBase和KieSession

第一步：在pro1项目页面点击右上角Settings按钮进入设置页面

![image-20200113162923877](https://img-blog.csdnimg.cn/img_convert/a4edfb8522ceacc8c0d0b7e134b199bf.png)

第二步：在设置页面选择“知识库和会话”选项

![image-20200113163005061](https://img-blog.csdnimg.cn/img_convert/985cc6391d3f1cecdc45427fd2c0d60f.png)

第三步：在弹出的知识库和会话页面点击“添加”按钮进行设置

![image-20200113163313305](https://img-blog.csdnimg.cn/img_convert/1325efa0c081e88103e0f4e8424bb8e4.png)

![image-20200113163344174](https://img-blog.csdnimg.cn/img_convert/a601f1bfe0c6776e5fbe1a37d7834335.png)

第四步：设置完成后点击右上角保存按钮完成设置操作，可以通过左侧浏览视图点击kmodule.xml，查看文件内容

注意：出不来的话，要刷新一下。

![image-20200113163539676](https://img-blog.csdnimg.cn/img_convert/16dd838c6f938122006356329c134873.png)

### 1.3.6 编译、构建、部署

前面我们已经在WorkBench中创建了一个空间itheima，并且在此空间中创建了一个项目pro1，在此项目中创建了数据文件、规则文件和测试场景，如下图：

![image-20200113160102668](https://img-blog.csdnimg.cn/img_convert/c82f711b0611eb2805578da1c61c6803.png)

点击右上角“Compile”按钮可以对项目进行编译，点击“Bulid&Deploy”按钮进行构建和部署。

部署成功后可以在本地maven仓库中看到当前项目已经被打成jar包：

![image-20200113160728259](https://img-blog.csdnimg.cn/img_convert/041a946a56cd230188819cee00a37fa5.png)

将上面的jar包进行解压，可以看到我们创建的数据对象Person和规则文件person以及kmodule.xml都已经打到jar包中了。

### 1.3.7 在项目中使用部署的规则

前面我们已经在WorkBench中创建了pro1项目，并且在pro1项目中创建了数据文件、规则文件等。最后我们将此项目打成jar包部署到了maven仓库中。本小节就需要在外部项目中使用我们定义的规则。

第一步：在IDEA中创建一个maven项目并在pom.xml文件中导入相关坐标

```
<dependency>
    <groupId>org.drools</groupId>
    <artifactId>drools-compiler</artifactId>
    <version>7.10.0.Final</version>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
</dependency>
```

第二步：在项目中创建一个数据对象Person，需要和WorkBench中创建的Person包名、类名完全相同，属性也需要对应

```java
package com.itheima.pro1;

public class Person implements java.io.Serializable {

    static final long serialVersionUID = 1L;

    private java.lang.String id;
    private java.lang.String name;
    private int age;

    public Person() {
    }

    public java.lang.String getId() {
        return this.id;
    }

    public void setId(java.lang.String id) {
        this.id = id;
    }

    public java.lang.String getName() {
        return this.name;
    }

    public void setName(java.lang.String name) {
        this.name = name;
    }

    public int getAge() {
        return this.age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public Person(java.lang.String id, java.lang.String name, int age) {
        this.id = id;
        this.name = name;
        this.age = age;
    }
}
```

第三步：编写单元测试，远程加载maven仓库中的jar包最终完成规则调用

```java
@Test
public void test1() throws Exception{
    //通过此URL可以访问到maven仓库中的jar包
    //URL地址构成：http://ip地址:Tomcat端口号/WorkBench工程名/maven2/坐标/版本号/xxx.jar
    String url = 
    "http://localhost:8080/kie-drools-wb/maven2/com/itheima/pro1/1.0.0/pro1-1.0.0.jar";
    
    KieServices kieServices = KieServices.Factory.get();
    
    //通过Resource资源对象加载jar包
    UrlResource resource = (UrlResource) kieServices.getResources().newUrlResource(url);
    //通过Workbench提供的服务来访问maven仓库中的jar包资源，需要先进行Workbench的认证
    resource.setUsername("kie");
    resource.setPassword("kie");
    resource.setBasicAuthentication("enabled");
    
    //将资源转换为输入流，通过此输入流可以读取jar包数据
    InputStream inputStream = resource.getInputStream();
    
    //创建仓库对象，仓库对象中保存Drools的规则信息
    KieRepository repository = kieServices.getRepository();
    
    //通过输入流读取maven仓库中的jar包数据，包装成KieModule模块添加到仓库中
    KieModule kieModule = 
    repository.
        addKieModule(kieServices.getResources().newInputStreamResource(inputStream));
    
    //基于KieModule模块创建容器对象，从容器中可以获取session会话
    KieContainer kieContainer = kieServices.newKieContainer(kieModule.getReleaseId());
    KieSession session = kieContainer.newKieSession();

    Person person = new Person();
    person.setAge(10);
    session.insert(person);

    session.fireAllRules();
    session.dispose();
}
```

执行单元测试可以发现控制台已经输出了相关内容。通过WorkBench修改规则输出内容并发布，再次执行单元测试可以发现控制台输出的内容也发生了变化。

**通过上面的案例可以发现，我们在IEDA中开发的项目中并没有编写规则文件，规则文件是我们通过WorkBench开发并安装部署到maven仓库中，我们自己开发的项目只需要远程加载maven仓库中的jar包就可以完成规则的调用。这种开发方式的好处是我们的应用可以和业务规则完全分离，同时通过WorkBench修改规则后我们的应用不需要任何修改就可以加载到最新的规则从而实现规则的动态变更。**