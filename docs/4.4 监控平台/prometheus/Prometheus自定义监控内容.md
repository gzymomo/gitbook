[TOC]

- [prometheus自定义监控内容](https://www.cnblogs.com/throwable/p/9346547.html)
- [Prometheus Metrics 设计的最佳实践和应用实例，看这篇够了！](https://www.cnblogs.com/tencent-cloud-native/p/13686763.html)

# 一、io.micrometer的使用
在SpringBoot2.X中，spring-boot-starter-actuator引入了io.micrometer，对1.X中的metrics进行了重构，主要特点是支持tag/label，配合支持tag/label的监控系统，使得我们可以更加方便地对metrics进行多维度的统计查询及监控。io.micrometer目前支持Counter、Gauge、Timer、Summary等多种不同类型的度量方式(不知道有没有遗漏)，下面逐个简单分析一下它们的作用和使用方式。 需要在SpringBoot项目下引入下面的依赖：
```xml
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-core</artifactId>
  <version>${micrometer.version}</version>
</dependency>
```
目前最新的micrometer.version为1.0.5。注意一点的是：io.micrometer支持Tag(标签)的概念，Tag是其metrics是否能够有多维度的支持的基础，Tag必须成对出现，也就是必须配置也就是偶数个Tag，有点类似于K-V的关系。

## 1.1 Counter
Counter(计数器)简单理解就是一种只增不减的计数器。它通常用于记录服务的请求数量、完成的任务数量、错误的发生数量等等。举个例子：
```java
import io.micrometer.core.instrument.Counter;
import io.micrometer.core.instrument.Metrics;
import io.micrometer.core.instrument.simple.SimpleMeterRegistry;

/**
 * @author throwable
 * @version v1.0
 * @description
 * @since 2018/7/19 23:10
 */
public class CounterSample {

    public static void main(String[] args) throws Exception {
        //tag必须成对出现，也就是偶数个
        Counter counter = Counter.builder("counter")
                .tag("counter", "counter")
                .description("counter")
                .register(new SimpleMeterRegistry());
        counter.increment();
        counter.increment(2D);
        System.out.println(counter.count());
        System.out.println(counter.measure());
        //全局静态方法
        Metrics.addRegistry(new SimpleMeterRegistry());
        counter = Metrics.counter("counter", "counter", "counter");
        counter.increment(10086D);
        counter.increment(10087D);
        System.out.println(counter.count());
        System.out.println(counter.measure());
    }
}
```
输出：
```java
3.0
[Measurement{statistic='COUNT', value=3.0}]
20173.0
[Measurement{statistic='COUNT', value=20173.0}]
```
Counter的Measurement的statistic(可以理解为度量的统计角度)只有COUNT，也就是它只具备计数(它只有增量的方法，因此只增不减)，这一点从它的接口定义可知：
```java
public interface Counter extends Meter {

  default void increment() {
        increment(1.0);
  }

  void increment(double amount);

  double count();

  //忽略其他方法或者成员
}
```
Counter还有一个衍生类型FunctionCounter，它是基于函数式接口ToDoubleFunction进行计数统计的，用法差不多。

## 1.2 Gauge
Gauge(仪表)是一个表示单个数值的度量，它可以表示任意地上下移动的数值测量。Gauge通常用于变动的测量值，如当前的内存使用情况，同时也可以测量上下移动的"计数"，比如队列中的消息数量。举个例子：
```java
import io.micrometer.core.instrument.Gauge;
import io.micrometer.core.instrument.Metrics;
import io.micrometer.core.instrument.simple.SimpleMeterRegistry;

import java.util.concurrent.atomic.AtomicInteger;

/**
 * @author throwable
 * @version v1.0
 * @description
 * @since 2018/7/19 23:30
 */
public class GaugeSample {

    public static void main(String[] args) throws Exception {
        AtomicInteger atomicInteger = new AtomicInteger();
        Gauge gauge = Gauge.builder("gauge", atomicInteger, AtomicInteger::get)
                .tag("gauge", "gauge")
                .description("gauge")
                .register(new SimpleMeterRegistry());
        atomicInteger.addAndGet(5);
        System.out.println(gauge.value());
        System.out.println(gauge.measure());
        atomicInteger.decrementAndGet();
        System.out.println(gauge.value());
        System.out.println(gauge.measure());
        //全局静态方法，返回值竟然是依赖值，有点奇怪，暂时不选用
        Metrics.addRegistry(new SimpleMeterRegistry());
        AtomicInteger other = Metrics.gauge("gauge", atomicInteger, AtomicInteger::get);
    }
}
```
输出结果:
```java
5.0
[Measurement{statistic='VALUE', value=5.0}]
4.0
[Measurement{statistic='VALUE', value=4.0}]
```
Gauge关注的度量统计角度是VALUE(值)，它的构建方法中依赖于函数式接口ToDoubleFunction的实例(如例子中的实例方法引用AtomicInteger::get)和一个依赖于ToDoubleFunction改变自身值的实例(如例子中的AtomicInteger实例)，它的接口方法如下：
```java
public interface Gauge extends Meter {

  double value();

  //忽略其他方法或者成员
}
```
## 1.3 Timer
Timer(计时器)同时测量一个特定的代码逻辑块的调用(执行)速度和它的时间分布。简单来说，就是在调用结束的时间点记录整个调用块执行的总时间，适用于测量短时间执行的事件的耗时分布，例如消息队列消息的消费速率。举个例子：
```java
import io.micrometer.core.instrument.Timer;
import io.micrometer.core.instrument.simple.SimpleMeterRegistry;

import java.util.concurrent.TimeUnit;

/**
 * @author throwable
 * @version v1.0
 * @description
 * @since 2018/7/19 23:44
 */
public class TimerSample {

    public static void main(String[] args) throws Exception{
        Timer timer = Timer.builder("timer")
                .tag("timer","timer")
                .description("timer")
                .register(new SimpleMeterRegistry());
        timer.record(()->{
            try {
                TimeUnit.SECONDS.sleep(2);
            }catch (InterruptedException e){
                //ignore
            }
        });
        System.out.println(timer.count());
        System.out.println(timer.measure());
        System.out.println(timer.totalTime(TimeUnit.SECONDS));
        System.out.println(timer.mean(TimeUnit.SECONDS));
        System.out.println(timer.max(TimeUnit.SECONDS));
    }
}
```
输出结果：
```java
1
[Measurement{statistic='COUNT', value=1.0}, Measurement{statistic='TOTAL_TIME', value=2.000603975}, Measurement{statistic='MAX', value=2.000603975}]
2.000603975
2.000603975
2.000603975
```
Timer的度量统计角度主要包括记录执行的最大时间、总时间、平均时间、执行完成的总任务数，它提供多种的统计方法变体：
```java
public interface Timer extends Meter, HistogramSupport {

  void record(long amount, TimeUnit unit);

  default void record(Duration duration) {
      record(duration.toNanos(), TimeUnit.NANOSECONDS);
  }

  <T> T record(Supplier<T> f);
    
  <T> T recordCallable(Callable<T> f) throws Exception;

  void record(Runnable f);

  default Runnable wrap(Runnable f) {
      return () -> record(f);
  }

  default <T> Callable<T> wrap(Callable<T> f) {
    return () -> recordCallable(f);
  }

  //忽略其他方法或者成员
}
```
这些record或者包装方法可以根据需要选择合适的使用，另外，一些度量属性(如下限和上限)或者单位可以自行配置，具体属性的相关内容可以查看DistributionStatisticConfig类，这里不详细展开。

另外，Timer有一个衍生类LongTaskTimer，主要是用来记录正在执行但是尚未完成的任务数，用法差不多。

## 1.4 Summary
Summary(摘要)用于跟踪事件的分布。它类似于一个计时器，但更一般的情况是，它的大小并不一定是一段时间的测量值。在micrometer中，对应的类是DistributionSummary，它的用法有点像Timer，但是记录的值是需要直接指定，而不是通过测量一个任务的执行时间。举个例子：
```java
import io.micrometer.core.instrument.DistributionSummary;
import io.micrometer.core.instrument.simple.SimpleMeterRegistry;

/**
 * @author throwable
 * @version v1.0
 * @description
 * @since 2018/7/19 23:55
 */
public class SummarySample {

    public static void main(String[] args) throws Exception {
        DistributionSummary summary = DistributionSummary.builder("summary")
                .tag("summary", "summary")
                .description("summary")
                .register(new SimpleMeterRegistry());
        summary.record(2D);
        summary.record(3D);
        summary.record(4D);
        System.out.println(summary.measure());
        System.out.println(summary.count());
        System.out.println(summary.max());
        System.out.println(summary.mean());
        System.out.println(summary.totalAmount());
    }
}
```
输出结果：
```java
[Measurement{statistic='COUNT', value=3.0}, Measurement{statistic='TOTAL', value=9.0}, Measurement{statistic='MAX', value=4.0}]
3
4.0
3.0
9.0
```
Summary的度量统计角度主要包括记录过的数据中的最大值、总数值、平均值和总次数。另外，一些度量属性(如下限和上限)或者单位可以自行配置，具体属性的相关内容可以查看DistributionStatisticConfig类。

# 二、扩展
SpringCloud体系的监控，扩展一个功能，记录一下每个有效的请求的执行时间。添加下面几个类或者方法：
```java
//注解
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface MethodMetric {

    String name() default "";

    String description() default "";

    String[] tags() default {};
}
//切面类
@Aspect
@Component
public class HttpMethodCostAspect {

    @Autowired
    private MeterRegistry meterRegistry;

    @Pointcut("@annotation(club.throwable.sample.aspect.MethodMetric)")
    public void pointcut() {
    }

    @Around(value = "pointcut()")
    public Object process(ProceedingJoinPoint joinPoint) throws Throwable {
        Method targetMethod = ((MethodSignature) joinPoint.getSignature()).getMethod();
        //这里是为了拿到实现类的注解
        Method currentMethod = ClassUtils.getUserClass(joinPoint.getTarget().getClass())
                .getDeclaredMethod(targetMethod.getName(), targetMethod.getParameterTypes());
        if (currentMethod.isAnnotationPresent(MethodMetric.class)) {
            MethodMetric methodMetric = currentMethod.getAnnotation(MethodMetric.class);
            return processMetric(joinPoint, currentMethod, methodMetric);
        } else {
            return joinPoint.proceed();
        }
    }

    private Object processMetric(ProceedingJoinPoint joinPoint, Method currentMethod,
                                 MethodMetric methodMetric) throws Throwable {
        String name = methodMetric.name();
        if (!StringUtils.hasText(name)) {
            name = currentMethod.getName();
        }
        String desc = methodMetric.description();
        if (!StringUtils.hasText(desc)) {
            desc = name;
        }
        String[] tags = methodMetric.tags();
        if (tags.length == 0) {
            tags = new String[2];
            tags[0] = name;
            tags[1] = name;
        }
        Timer timer = Timer.builder(name).tags(tags)
                .description(desc)
                .register(meterRegistry);
        return timer.record(() -> {
            try {
                return joinPoint.proceed();
            } catch (Throwable throwable) {
                throw new IllegalStateException(throwable);
            }
        });
    }
}
//启动类里面添加方法
@SpringBootApplication
@EnableEurekaClient
@RestController
public class SampleApplication {

    public static void main(String[] args) {
        SpringApplication.run(SampleApplication.class, args);
    }

    @MethodMetric
    @GetMapping(value = "/hello")
    public String hello(@RequestParam(name = "name", required = false, defaultValue = "doge") String name) {
        return String.format("%s say hello!", name);
    }
}
```
配置好Grafana的面板，重启项目，多次调用/hello接口。