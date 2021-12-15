## 一、TestNG介绍

TestNG是Java中的一个测试框架， 类似于JUnit 和NUnit, 功能都差不多， 只是功能更加强大，使用也更方便。
详细使用说明请参考官方链接：https://testng.org/doc/index.html

## 二、TestNG安装（基于eclipse+maven）

工程的pom.xml中需要添加如下内容：

```xml
<dependency>
      <groupId>org.testng</groupId>
      <artifactId>testng</artifactId>
      <version>6.10</version>
      <scope>test</scope>
    </dependency>
```

记得Maven install一下
eclipse中的TestNG插件的安装则需要在Help中做，地址为`http://beust.com/eclipse`；

## 三、TestNG基本使用和运行

新建一个maven工程，新建一个TestNG的class，可以直接新建一个class来使用，也可以新建一个TestNG的class，如下图所示：
![TestNG new](https://img-blog.csdn.net/20181021183906785?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RmMDEyOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
此方法好处在于你在创建class的时候可以直接把注解的各个方法都加进去，同时也可以创建xml，名字路径可以自己定义，注意xml文件的路径是支持相对路径的，出来的class文件如下所示：

```java
package com.demo.test.testng;

import org.testng.annotations.Test;

public class NewTest {
	
  @Test
  public void f() {
	  
  }
}
```

一个简单的用例如下：

```java
package com.demo.test.testng;

import org.testng.Assert;
import org.testng.annotations.Test;

public class NewTest {
	
  @Test
  public void f() {
	  System.out.println("this is new test");
	  Assert.assertTrue(true);
  }
}
```

### 1、直接运行：

![运行TestNG1](https://img-blog.csdn.net/20181021185046907?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RmMDEyOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
结果如下：

```text
[RemoteTestNG] detected TestNG version 6.10.0
[TestNG] Running:
  C:\Users\aaa\AppData\Local\Temp\testng-eclipse-342998054\testng-customsuite.xml

this is new test
PASSED: f

===============================================
    Default test
    Tests run: 1, Failures: 0, Skips: 0
===============================================


===============================================
Default suite
Total tests run: 1, Failures: 0, Skips: 0
===============================================

[TestNG] Time taken by org.testng.reporters.JUnitReportReporter@64bfbc86: 4 ms
[TestNG] Time taken by org.testng.reporters.SuiteHTMLReporter@7e0b0338: 26 ms
[TestNG] Time taken by org.testng.reporters.jq.Main@7fac631b: 20 ms
[TestNG] Time taken by [FailedReporter passed=0 failed=0 skipped=0]: 0 ms
[TestNG] Time taken by org.testng.reporters.XMLReporter@23e028a9: 3 ms
[TestNG] Time taken by org.testng.reporters.EmailableReporter2@578486a3: 2 ms
```

### 2、xml方式运行

由于我将xml放置在其他文件夹，不和class放在一个文件夹，所以需要修改xml，如下所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<suite name="Suite" parallel="false">
  <test name="Test">
    <classes>
      <class name="com.demo.test.testng.NewTest"/>
    </classes>
  </test> <!-- Test -->
</suite> <!-- Suite -->
```

运行方法：
右键该xml选择Run As–>TestNG Suite，如下：
![XML运行](https://img-blog.csdn.net/20181021185641728?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RmMDEyOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
运行结果如下：

```text
[RemoteTestNG] detected TestNG version 6.10.0
[TestNGContentHandler] [WARN] It is strongly recommended to add "<!DOCTYPE suite SYSTEM "http://testng.org/testng-1.0.dtd" >" at the top of your file, otherwise TestNG may fail or not work as expected.
[XmlSuite] [WARN] 'parallel' value 'false' is deprecated, default value will be used instead: 'none'.
[TestNG] Running:
  D:\software\workspace\testng\src\main\java\com\demo\test\testCase\newTestXML.xml

this is new test

===============================================
Suite
Total tests run: 1, Failures: 0, Skips: 0
===============================================
```



## 四、注解说明

TestNG支持多种注解，可以进行各种组合，如下进行简单的说明

| 注解          | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| @BeforeSuite  | 在该套件的所有测试都运行在注释的方法之前，仅运行一次         |
| @AfterSuite   | 在该套件的所有测试都运行在注释方法之后，仅运行一次           |
| @BeforeClass  | 在调用当前类的第一个测试方法之前运行，注释方法仅运行一次     |
| @AfterClass   | 在调用当前类的第一个测试方法之后运行，注释方法仅运行一次     |
| @BeforeTest   | 注释的方法将在属于test标签内的类的所有测试方法运行之前运行   |
| @AfterTest    | 注释的方法将在属于test标签内的类的所有测试方法运行之后运行   |
| @BeforeGroups | 配置方法将在之前运行组列表。 此方法保证在调用属于这些组中的任何一个的第一个测试方法之前不久运行 |
| @AfterGroups  | 此配置方法将在之后运行组列表。该方法保证在调用属于任何这些组的最后一个测试方法之后不久运行 |
| @BeforeMethod | 注释方法将在每个测试方法之前运行                             |
| @AfterMethod  | 注释方法将在每个测试方法之后运行                             |
| @DataProvider | 标记一种方法来提供测试方法的数据。 注释方法必须返回一个`Object [] []`，其中每个`Object []`可以被分配给测试方法的参数列表。 要从该`DataProvider`接收数据的`@Test`方法需要使用与此注释名称相等的`dataProvider`名称 |
| @Factory      | 将一个方法标记为工厂，返回`TestNG`将被用作测试类的对象。 该方法必须返回`Object []` |
| @Listeners    | 定义测试类上的侦听器                                         |
| @Parameters   | 描述如何将参数传递给`@Test`方法                              |
| @Test         | 将类或方法标记为测试的一部分，此标记若放在类上，则该类所有公共方法都将被作为测试方法 |

如上列表中的@Factory、@Linsteners这两个是不常用的；
前十个注解看起来不太容易区分，顺序不太容易看明白，以如下范例做简单说明，代码：

```java
import org.testng.Assert;
import org.testng.annotations.AfterClass;
import org.testng.annotations.AfterGroups;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.AfterSuite;
import org.testng.annotations.AfterTest;
import org.testng.annotations.BeforeClass;
import org.testng.annotations.BeforeGroups;
import org.testng.annotations.BeforeMethod;
import org.testng.annotations.BeforeSuite;
import org.testng.annotations.BeforeTest;
import org.testng.annotations.Test;

public class NewTest {

  @Test(groups="group1")
  public void test1() {
	  System.out.println("test1 from group1");
	  Assert.assertTrue(true);
  }
  
  @Test(groups="group1")
  public void test11() {
	  System.out.println("test11 from group1");
	  Assert.assertTrue(true);
  }
  
  @Test(groups="group2")
  public void test2() 
  {
	  System.out.println("test2 from group2");
	  Assert.assertTrue(true);
  }
  
  @BeforeTest
  public void beforeTest() 
  {
	  System.out.println("beforeTest");
  }
  
  @AfterTest
  public void afterTest() 
  {
	  System.out.println("afterTest");
  }
  
  @BeforeClass
  public void beforeClass() 
  {
	  System.out.println("beforeClass");
  }
  
  @AfterClass
  public void afterClass() 
  {
	  System.out.println("afterClass");
  }
  
  @BeforeSuite
  public void beforeSuite() 
  {
	  System.out.println("beforeSuite");
  }
  
  @AfterSuite
  public void afterSuite() 
  {
	  System.out.println("afterSuite");
  }
  
  //只对group1有效，即test1和test11
  @BeforeGroups(groups="group1")
  public void beforeGroups() 
  {
	  System.out.println("beforeGroups");
  }
  
  //只对group1有效，即test1和test11
  @AfterGroups(groups="group1")
  public void afterGroups() 
  {
	  System.out.println("afterGroups");
  }
  
  @BeforeMethod
  public void beforeMethod() 
  {
	  System.out.println("beforeMethod");
  }
  
  @AfterMethod
  public void afterMethod() 
  {
	  System.out.println("afterMethod");
  }
}
```

运行结果如下：

```log
beforeSuite
beforeTest
beforeClass
beforeGroups
beforeMethod
test1 from group1
afterMethod
beforeMethod
test11 from group1
afterMethod
afterGroups
beforeMethod
test2 from group2
afterMethod
afterClass
afterTest
PASSED: test1
PASSED: test11
PASSED: test2

===============================================
    Default test
    Tests run: 3, Failures: 0, Skips: 0
===============================================

afterSuite
```

对照前面的说明应该就可以能比较明白了。

## 五、TestNG断言

TestNG的断言种类很多，包括相等/不相等，true/false、为null/不为null、相同/不相同等。

## 六、TestNG预期异常测试

预期异常测试通过在@Test注解后加入预期的Exception来进行添加，范例如下所示：

```java
@Test(expectedExceptions = ArithmeticException.class)
    public void divisionWithException() {
        int i = 1 / 0;
        System.out.println("After division the value of i is :"+ i);
    }
```

运行结果如下：

```log
[RemoteTestNG] detected TestNG version 6.10.0
[TestNG] Running:
  C:\Users\Administrator\AppData\Local\Temp\testng-eclipse--754789457\testng-customsuite.xml

PASSED: divisionWithException

===============================================
    Default test
    Tests run: 1, Failures: 0, Skips: 0
===============================================


===============================================
Default suite
Total tests run: 1, Failures: 0, Skips: 0
===============================================

[TestNG] Time taken by org.testng.reporters.JUnitReportReporter@55d56113: 0 ms
[TestNG] Time taken by org.testng.reporters.SuiteHTMLReporter@1e127982: 0 ms
[TestNG] Time taken by org.testng.reporters.jq.Main@6e0e048a: 32 ms
[TestNG] Time taken by [FailedReporter passed=0 failed=0 skipped=0]: 0 ms
[TestNG] Time taken by org.testng.reporters.XMLReporter@43814d18: 0 ms
[TestNG] Time taken by org.testng.reporters.EmailableReporter2@6ebc05a6: 0 ms
1234567891011121314151617181920212223
```

## 七、TestNG忽略测试

有时候我们写的用例没准备好，或者该次测试不想运行此用例，那么删掉显然不明智，那么就可以通过注解`@Test(enabled = false)`来将其忽略掉，此用例就不会运行了，如下范例：

```java
import org.testng.annotations.Test;

public class TestCase1 {

    @Test(enabled=false)
    public void TestNgLearn1() {
        System.out.println("this is TestNG test case1");
    }
    
    @Test
    public void TestNgLearn2() {
        System.out.println("this is TestNG test case2");
    }
}
```

运行结果：

```log
this is TestNG test case2
PASSED: TestNgLearn2
```

## 八、TestNG超时测试

“超时”表示如果单元测试花费的时间超过指定的毫秒数，那么TestNG将会中止它并将其标记为失败。此项常用于性能测试。如下为一个范例：

```java
import org.testng.annotations.Test;

public class TestCase1 {

    @Test(timeOut = 5000) // time in mulliseconds
    public void testThisShouldPass() throws InterruptedException {
        Thread.sleep(4000);
    }

    @Test(timeOut = 1000)
    public void testThisShouldFail() {
        while (true){
            // do nothing
        }

    }
}
```

结果如下：

```log
PASSED: testThisShouldPass
FAILED: testThisShouldFail
org.testng.internal.thread.ThreadTimeoutException: Method com.demo.test.testng.TestCase1.testThisShouldFail() didn't finish within the time-out 1000
	at com.demo.test.testng.TestCase1.testThisShouldFail(TestCase1.java:37)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.testng.internal.MethodInvocationHelper.invokeMethod(MethodInvocationHelper.java:104)
	at org.testng.internal.InvokeMethodRunnable.runOne(InvokeMethodRunnable.java:54)
	at org.testng.internal.InvokeMethodRunnable.run(InvokeMethodRunnable.java:44)
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)

```

## 九、分组测试

分组测试即为使用group，如果你使用xml的话就是里边的`<groups>`标签，如果是直接在class中，是通过`@Test(groups="group2")`这种方式来分组，如第四节的注解说明中的那个例子，分了两个group，而且`@BeforeGroup`是需要添加group名称才可以正确挂载到该group下的；
这个group说明可以是在单个的测试方法上，也可以在class上，只要具有同样的group名称都会在同一个group中，同时group名称可以有多个，类似`@Test(groups = {"mysql","database"})`这种，范例如下：
一个测试文件NewTest.class：

```java
public class NewTest {

  @Test(groups="group1")
  public void test1() {
	  System.out.println("test1 from group1");
	  Assert.assertTrue(true);
  }
  
  @Test(groups="group1")
  public void test11() {
	  System.out.println("test11 from group1");
	  Assert.assertTrue(true);
  }
  
  @Test(groups="group2")
  public void test2() 
  {
	  System.out.println("test2 from group2");
	  Assert.assertTrue(true);
  }
  
  @BeforeTest
  public void beforeTest() 
  {
	  System.out.println("beforeTest");
  }
  
  @AfterTest
  public void afterTest() 
  {
	  System.out.println("afterTest");
  }
  
  @BeforeClass
  public void beforeClass() 
  {
	  System.out.println("beforeClass");
  }
  
  @AfterClass
  public void afterClass() 
  {
	  System.out.println("afterClass");
  }
  
  @BeforeSuite
  public void beforeSuite() 
  {
	  System.out.println("beforeSuite");
  }
  
  @AfterSuite
  public void afterSuite() 
  {
	  System.out.println("afterSuite");
  }
  
  @BeforeGroups(groups="group1")
  public void beforeGroups() 
  {
	  System.out.println("beforeGroups");
  }
  
  @AfterGroups(groups="group1")
  public void afterGroups() 
  {
	  System.out.println("afterGroups");
  }
  
  @BeforeMethod
  public void beforeMethod() 
  {
	  System.out.println("beforeMethod");
  }
  
  @AfterMethod
  public void afterMethod() 
  {
	  System.out.println("afterMethod");
  }
  
}
```

另一个TestCase1.class:

```java
@Test(groups= "group2")
public class TestCase1 {

    @Test(enabled=false)
    public void TestNgLearn1() {
        System.out.println("this is TestNG test case1");
    }
    
    @Test
    public void TestNgLearn2() {
        System.out.println("this is TestNG test case2");
    }
}
```

xml如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<suite name="Suite" parallel="false">
  <test name="Test">
    <groups>
      <incloud name="group1"></incloud>
      <incloud name="group2"></incloud>
    </groups>
    <classes>
      <class name="com.demo.test.testng.NewTest"/>
      <class name="com.demo.test.testng.TestCase1"/>
    </classes>
  </test> <!-- Test -->
</suite> <!-- Suite -->
```

运行结果如下：

```log
beforeSuite
beforeTest
beforeClass
beforeGroups
beforeMethod
test1 from group1
afterMethod
beforeMethod
test11 from group1
afterMethod
afterGroups
beforeMethod
test2 from group2
afterMethod
afterClass
this is TestNG test case2
afterTest
afterSuite
```

如上所示，先运行了group1的两个用例，再运行group2的两条用例；
注意在xml标识group，需要将要运行的group加进来，同时还要将被标识这些group的class也加进来，不被加进去的不会运行；

## 十、分suite测试

测试套件是用于测试软件程序的行为或一组行为的测试用例的集合。 在TestNG中，我们无法在测试源代码中定义一个套件，但它可以由一个XML文件表示，因为套件是执行的功能。 它还允许灵活配置要运行的测试。 套件可以包含一个或多个测试，并由`<suite>`标记定义。`<suite>`是testng.xml的根标记。 它描述了一个测试套件，它又由几个`<test>`部分组成。
下表列出了`<suite>`接受的所有定义的合法属性。

| 属性         | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| name         | 套件的名称，这是一个强制属性                                 |
| verbose      | 运行的级别或详细程度，级别为0-10，其中10最详细               |
| parallel     | TestNG是否运行不同的线程来运行这个套件，默认为none，其他级别为methods、tests、classes、instances |
| thread-count | 如果启用并行模式(忽略其他方式)，则为使用的线程数             |
| annotations  | 在测试中使用的注释类型                                       |
| time-out     | 在本测试中的所有测试方法上使用的默认超时                     |

## 十一、依赖测试

有时，我们可能需要以特定顺序调用测试用例中的方法，或者可能希望在方法之间共享一些数据和状态。 TestNG支持这种依赖关系，因为它支持在测试方法之间显式依赖的声明。
TestNG允许指定依赖关系：

- 在`@Test`注释中使用属性dependsOnMethods
- 在`@Test`注释中使用属性dependsOnGroups

除此之外依赖还分为hard依赖和soft依赖：

- **hard依赖**：默认为此依赖方式，即其所有依赖的methods或者groups必须全部pass，否则被标识依赖的类或者方法将会被略过，在报告中标识为skip，如后面的范例所示，此为默认的依赖方式；
- **soft依赖**：此方式下，其依赖的方法或者组有不是全部pass也不会影响被标识依赖的类或者方法的运行，注意如果使用此方式，则依赖者和被依赖者之间必须不存在成功失败的因果关系，否则会导致用例失败。此方法在注解中需要加入`alwaysRun=true`即可，如`@Test(dependsOnMethods= {"TestNgLearn1"}， alwaysRun=true)`；

在TestNG中，我们使用dependOnMethods和dependsOnGroups来实现依赖测试。 且这两个都支持正则表达式，如范例三所示，如下为几个使用范例：

- 范例一，被依赖方法pass：

```java
public class TestCase1 {

    @Test(enabled=true)
    public void TestNgLearn1() {
        System.out.println("this is TestNG test case1");
    }
    
    @Test(dependsOnMethods= {"TestNgLearn1"})
    public void TestNgLearn2() {
        System.out.println("this is TestNG test case2");
    }
}
```

运行结果：

```log
this is TestNG test case1
this is TestNG test case2
PASSED: TestNgLearn1
PASSED: TestNgLearn2
```

- 范例二，被依赖方法fail：

```java
public class TestCase1 {

    @Test(enabled=true)
    public void TestNgLearn1() {
        System.out.println("this is TestNG test case1");
        Assert.assertFalse(true);
    }
    
    @Test(dependsOnMethods= {"TestNgLearn1"})
    public void TestNgLearn2() {
        System.out.println("this is TestNG test case2");
    }
}
```

结果：

```log
this is TestNG test case1
FAILED: TestNgLearn1
junit.framework.AssertionFailedError
	at junit.framework.Assert.fail(Assert.java:47)
	at junit.framework.Assert.assertTrue(Assert.java:20)
	at junit.framework.Assert.assertFalse(Assert.java:34)
	at junit.framework.Assert.assertFalse(Assert.java:41)
	at com.demo.test.testng.TestCase1.TestNgLearn1(TestCase1.java:26)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.testng.internal.MethodInvocationHelper.invokeMethod(MethodInvocationHelper.java:104)
	at org.testng.internal.Invoker.invokeMethod(Invoker.java:645)
	at org.testng.internal.Invoker.invokeTestMethod(Invoker.java:851)
	at org.testng.internal.Invoker.invokeTestMethods(Invoker.java:1177)
	at org.testng.internal.TestMethodWorker.invokeTestMethods(TestMethodWorker.java:129)
	at org.testng.internal.TestMethodWorker.run(TestMethodWorker.java:112)
	at org.testng.TestRunner.privateRun(TestRunner.java:756)
	at org.testng.TestRunner.run(TestRunner.java:610)
	at org.testng.SuiteRunner.runTest(SuiteRunner.java:387)
	at org.testng.SuiteRunner.runSequentially(SuiteRunner.java:382)
	at org.testng.SuiteRunner.privateRun(SuiteRunner.java:340)
	at org.testng.SuiteRunner.run(SuiteRunner.java:289)
	at org.testng.SuiteRunnerWorker.runSuite(SuiteRunnerWorker.java:52)
	at org.testng.SuiteRunnerWorker.run(SuiteRunnerWorker.java:86)
	at org.testng.TestNG.runSuitesSequentially(TestNG.java:1293)
	at org.testng.TestNG.runSuitesLocally(TestNG.java:1218)
	at org.testng.TestNG.runSuites(TestNG.java:1133)
	at org.testng.TestNG.run(TestNG.java:1104)
	at org.testng.remote.AbstractRemoteTestNG.run(AbstractRemoteTestNG.java:114)
	at org.testng.remote.RemoteTestNG.initAndRun(RemoteTestNG.java:251)
	at org.testng.remote.RemoteTestNG.main(RemoteTestNG.java:77)

SKIPPED: TestNgLearn2
```

- 范例三、group依赖:
  如下所示，method1依赖group名称为init的所有方法：

```java
@Test(groups = { "init" })
public void serverStartedOk() {}
 
@Test(groups = { "init" })
public void initEnvironment() {}
 
@Test(dependsOnGroups = { "init.*" })
public void method1() {}
```

这里init这个group中的两个方法的执行顺序如果没有在xml中指明则每次运行的顺序不能保证

## 十二、参数化测试

TestNG中的另一个有趣的功能是参数化测试。 在大多数情况下，您会遇到业务逻辑需要大量测试的场景。 参数化测试允许开发人员使用不同的值一次又一次地运行相同的测试。
TestNG可以通过两种不同的方式将参数直接传递给测试方法：

- 使用testng.xml
- 使用数据提供者
  下面分别介绍两种传参方式：

### 1、使用textng.xml传送参数

范例代码如下：

```java
public class TestCase1 {

    @Test(enabled=true)
    @Parameters({"param1", "param2"})
    public void TestNgLearn1(String param1, int param2) {
        System.out.println("this is TestNG test case1, and param1 is:"+param1+"; param2 is:"+param2);
        Assert.assertFalse(false);
    }
    
    @Test(dependsOnMethods= {"TestNgLearn1"})
    public void TestNgLearn2() {
        System.out.println("this is TestNG test case2");
    }
}
```

xml配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<suite name="Suite" parallel="false">
  <test name="Test">
    <parameter name="param1" value="1011111" />
    <parameter name="param2" value="10" />
    <classes>
      <class name="com.demo.test.testng.TestCase1"/>
    </classes>
  </test> <!-- Test -->
</suite> <!-- Suite -->
```

运行xml，结果如下：

```log
this is TestNG test case1, and param1 is:1011111; param2 is:10
this is TestNG test case2

===============================================
Suite
Total tests run: 2, Failures: 0, Skips: 0
===============================================
```

### 2、使用`@DataProvider`传递参数

此处需要注意，传参的类型必须要一致，且带有`@DataProvider`注解的函数返回的必然是`Object[][]`，此处需要注意。
代码如下：

```java
public class TestCase1 {

    @DataProvider(name = "provideNumbers")
    public Object[][] provideData() {

        return new Object[][] { { 10, 20 }, { 100, 110 }, { 200, 210 } };
    }
	
    @Test(dataProvider = "provideNumbers")
    public void TestNgLearn1(int param1, int param2) {
        System.out.println("this is TestNG test case1, and param1 is:"+param1+"; param2 is:"+param2);
        Assert.assertFalse(false);
    }
    
    @Test(dependsOnMethods= {"TestNgLearn1"})
    public void TestNgLearn2() {
        System.out.println("this is TestNG test case2");
    }
}
```

运行此class，结果为：

```log
this is TestNG test case1, and param1 is:10; param2 is:20
this is TestNG test case1, and param1 is:100; param2 is:110
this is TestNG test case1, and param1 is:200; param2 is:210
this is TestNG test case2
PASSED: TestNgLearn1(10, 20)
PASSED: TestNgLearn1(100, 110)
PASSED: TestNgLearn1(200, 210)
PASSED: TestNgLearn2
```

## 十三、XML配置文件说明

前面讲的大多都是以测试脚本为基础来运行的，少部分是以xml运行，这里以xml来讲解下：

```xml
<!DOCTYPE suite SYSTEM "http://testng.org/testng-1.0.dtd" >
<suite name="SuiteName" verbose="1" > 
```

如下分别讲解各个标签：

### 1、suite标签

testNG.xml文件的最外层标签即`suite`，即测试套件，其下可以有多个`<test>`和`<groups>`，其有几个可以添加的属性在第十节的分suite测试中有做说明，这里做下详细说明：

#### (1)、name属性

此属性属于必须要有的，值可以自行设定，此名字会在testNG的报告中看到；

#### (2)、verbose属性

此属性为指定testNG报告的详细程度，从0开始到10，其中10为最详细，默认生成的xml此属性值为1；

#### (3)、parallel属性

此属性是指代运行方式，默认为none，即串行运行方式；并行执行方法包括如下几种，下面做分别说明：

- **methods**：方法层级，若为此值，则该suite下所有的测试方法都将进行多线程，即测试用例级别的多线程。如果用例之间有依赖，则执行顺序会按照设定的依赖来运行；

```xml
<suite name="My suite" parallel="methods" thread-count="5">
```

- **tests**：TestNG将在同一线程中运行相同的`<Test>`标签中的所有方法，每个`<test>`标签都将处于一个单独的线程中，这允许您将不是线程安全的所有类分组在同一个`<test>`中，并保证它们都将在同一个线程中运行，同时利用TestNG使用尽可能多的线程运行测试。

```xml
<suite name="My suite" parallel="tests" thread-count="5">
```

- **classes**：类级别并发，即TestNG会将该suite下每个class都将在单独的线程中运行，同一个class下的所有用例都将在同一个线程中运行；

```xml
<suite name="My suite" parallel="classes" thread-count="5">
```

- **instances**：实例级别，即TestNG将在同一线程中运行同一实例中的所有方法，两个不同实例上的两个方法将在不同的线程中运行。

```xml
<suite name="My suite" parallel="instances" thread-count="5">
```

#### (4)、thread-count属性

此属性用于指定线程数，按照需要输入，需要`parallel`参数非none时才可以添加；

#### (5)、annotations属性

此项为注解的级别，为methods级别和class级别，一般不用设置；

#### (6)、time-out属性

此属性用于指定超时时间，该suite下所有的用例的超时时间；

#### (7)、group-by-instances属性

此项用于那些有依赖的方法，且被依赖的对象有多个重载对象，因为如果是依赖方法，且该方法有多个重载方法，则默认是会将所有重载方法都跑完再运行被依赖方法，但有时候我们不想这样，则将此项设置为true即可；

#### (8)、preserve-order属性

值可输入true或者false，如果为true，则用例执行会按照在xml中的顺序执行，否则会乱序执行，不添加此属性的话默认是按顺序执行的；

### 2、test标签

此标签无特别意义，其下可以包括多个标签，如`groups`、`classes`等，如下介绍下几种书写方式：

1. 选择一个包中的全部测试脚本（包含子包）

```xml
<test name = "allTestsInAPackage" >
   <packages>
      <package name = "whole.path.to.package.* />
   </packages>
</test>
```

1. 选择一个类中的全部测试脚本

```xml
<test name = "allTestsInAClass" >
   <classes>
  <class name="whole.path.to.package.className />
   </classes>
</test>
```

1. 选择一个类中的部分测试脚本

```xml
<test name = "aFewTestsFromAClass" >
   <classes>
  <class name="whole.path.to.package.className >
      <methods>
         <include name = "firstMethod" />
         <include name = "secondMethod" />
         <include name = "thirdMethod" />
      </methods>
  </class>
   </classes>
</test>
```

1. 选择一个包中的某些组

```xml
<test name = "includedGroupsInAPackage" >
   <groups>
      <run>
         <include name = "includedGroup" />
      </run>
   </groups>
   <packages>
      <package name = "whole.path.to.package.* />
   </packages>
</test>
```

1. 排除一个包中的某些组

```xml
<test name = "excludedGroupsInAPackage" >
   <groups>
      <run>
         <exclude name = "excludedGroup" />
      </run>
   </groups>
   <packages>
      <package name = "whole.path.to.package.* />
   </packages>
</test>
```

其可以附带的属性有如下几种，下面对各个属性做单独说明：

#### (1)、name属性

此属性属于必须要有的，值可以自行设定，此名字会在testNG的报告中看到；

#### (2)、verbose属性

此属性为指定testNG报告的详细程度，从0开始到10，其中10为最详细，默认生成的xml此属性值为1

#### (3)、threadPoolSize属性

该属性指定此test的线程池大小，为数字；

```java
@Test(threadPoolSize = 3, invocationCount = 10,  timeOut = 10000)
public void testServer() {
}
```

#### (4)、invocationCount属性

该属性指定此test的运行次数，为数字，范例如上面的代码所示；

#### (5)、time-out属性

此属性用于指定超时时间，该suite下所有的用例的超时时间，范例如上面的代码所示；

#### (6)、group-by-instances属性

此项用于那些有依赖的方法，且被依赖的对象有多个重载对象，因为如果是依赖方法，且该方法有多个重载方法，则默认是会将所有重载方法都跑完再运行被依赖方法，但有时候我们不想这样，则将此项设置为true即可；

```xml
<suite name="Factory" group-by-instances="true">
```

#### (7)、preserve-order属性

值可输入true或者false，如果为true，则用例执行会按照在xml中的顺序执行，否则会乱序执行，不添加此属性的话默认是按顺序执行的；

### 3、group标签

此标签必然是在`<test>`标签下的，用于标识那些组会被用于测试或者被排除在测试之外，其同级必然要包含一个`<classes>`标签或者`<pakages>`标签，用于指定groups来自于哪些包或者类；
如下即为包含一个group，排除一个group的例子：

```xml
<groups>
  <run>
     <include name = "includedGroupName" />
     <exclude name = "excludedGroupName" />
  </run>
</groups>
```

高级应用：

```xml
<test name="Regression1">
  <groups>
    <define name="functest">
      <include name="windows"/>
      <include name="linux"/>
    </define>
  
    <define name="all">
      <include name="functest"/>
      <include name="checkintest"/>
    </define>
  
    <run>
      <include name="all"/>
    </run>
  </groups>
  
  <classes>
    <class name="test.sample.Test1"/>
  </classes>
</test>
```

### 4、其他

其他的话就是测试脚本的选择了，有三种方式：

1. 选择一个包

```xml
<packages>
    <package name = "packageName" />
</packages>
```

1. 选择一个类

```xml
<classes>
    <class name = "className" />
</classes>
```

1. 选择一个方法

```xml
<classes>
    <class name = "className" />
       <methods>
          <include name = "methodName" />
       </methods>
    </class>
</classes>
```

这里也支持正则表达式，例如:

```xml
<test name="Test1">
  <classes>
    <class name="example1.Test1">
      <methods>
        <include name=".*enabledTestMethod.*"/>
        <exclude name=".*brokenTestMethod.*"/>
      </methods>
     </class>
  </classes>
</test>
```

## 十四、TestNG报告

默认报告输出位置为当前工程的test-output文件夹下，包括xml格式和html格式。
如果想将报告输出位置换个地方，则修改地方在如下图所示位置：
![修改输出位置](https://img-blog.csdn.net/20181023103320767?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RmMDEyOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

如果想要美化报告，则按照如下步骤：
1、配置：Eclipse --> Window --> Preferences -->testng
2、勾选Disable default listeners
3、在Pre Defined Listeners 输入框中输入org.uncommons.reportng.HTMLReporter
记得在POM上添加如下代码：

```xml
<dependency>
      <groupId>org.uncommons</groupId>
      <artifactId>reportng</artifactId>
      <version>1.1.4</version>
      <scope>test</scope>
      <exclusions>
        <exclusion>
          <groupId>org.testng</groupId>
          <artifactId>testng</artifactId>
        </exclusion>
      </exclusions>
     </dependency>
     <dependency>
       <groupId>com.google.inject</groupId>
       <artifactId>guice</artifactId>
       <version>3.0</version>
       <scope>test</scope>
     </dependency>
```

不然无法运行的。
如上图所示，还可以自定义testng.xml的模板，并在上图中指定。