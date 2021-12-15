- [浅析Easy Rules规则引擎](https://zhuanlan.zhihu.com/p/91158525)
- [轻量级规则引擎EasyRules](https://www.dazhuanlan.com/dijiuketang/topics/1610409)
- [Java规则引擎 Easy Rules](https://www.cnblogs.com/cjsblog/p/13088017.html)

# 一、EasyRules介绍

## 1.1 概述

Easy Rules是一个简单而强大的Java规则引擎，提供以下功能：

- 轻量级框架和易于学习的API
- 基于POJO的开发与注解的编程模型
- 定义抽象的业务规则并轻松应用它们
- 支持从简单规则创建组合规则的能力
- 支持使用表达式语言（如MVEL和SpEL）定义规则的能力

在一篇非常有趣的规则引擎的[文章](https://link.zhihu.com/?target=http%3A//martinfowler.com/bliki/RulesEngine.html)中，Martin Fowler说：

> 您可以自己构建一个简单的规则引擎。您只需要创建一组具有条件和操作的对象，将它们存储在一个集合中，并运行它们来评估conditions和执行actions。

这正是Easy Rules所做的，它提供了抽象Rule来创建带有conditions和actions的规则，RulesEngine API运行一系列规则来评估conditions和执行actions。

Github地址：https://github.com/j-easy/easy-rules

## 1.2 运行环境

Easy Rules是一个Java库， 需要运行在Java 1.7及以上。

# 二、应用

## 2.1 maven依赖

```xml
<!--easy rules核心库-->
<dependency>
    <groupId>org.jeasy</groupId>
    <artifactId>easy-rules-core</artifactId>
    <version>3.3.0</version>
</dependency>

<!--规则定义文件格式，支持json,yaml等-->
<dependency>
    <groupId>org.jeasy</groupId>
    <artifactId>easy-rules-support</artifactId>
    <version>3.3.0</version>
</dependency>

<!--支持mvel规则语法库-->
<dependency>
    <groupId>org.jeasy</groupId>
    <artifactId>easy-rules-mvel</artifactId>
    <version>3.3.0</version>
</dependency>
```

## 2.2 定义规则

大多数业务规则可以由以下定义表示：

- 名称：规则命名空间中的唯一规则名称
- 说明：规则的简要说明
- 优先级：相对于其他规则的规则优先级
- 事实：去匹配规则时的一组已知事实
- 条件：为了匹配该规则，在给定某些事实的情况下应满足的一组条件
- 动作：当条件满足时要执行的一组动作（可以添加/删除/修改事实）
- Name : 一个命名空间下的唯一的规则名称
- Description : 规则的简要描述
- Priority : 相对于其他规则的优先级
- Facts : 事实，可立即为要处理的数据
- Conditions : 为了应用规则而必须满足的一组条件
- Actions : 当条件满足时执行的一组动作 

Easy Rules为定义业务规则的每个关键点提供了抽象。

在Easy Rules中，一个规则由`Rule`接口表示：

```java
public interface Rule {

    /**
    * 改方法封装规则的条件（conditions）
    * @return 如果提供的事实适用于该规则返回true, 否则，返回false
    */
    boolean evaluate(Facts facts);

    /**
    * 改方法封装规则的操作（actions）
    * @throws 如果在执行过程中发生错误将抛出Exception
    */
    void execute(Facts facts) throws Exception;

    //Getters and setters for rule name, description and priority omitted.

}
```

evaluate方法封装了必须求值为TRUE才能触发规则的条件。

execute方法封装了在满足规则条件时应执行的操作。条件和动作`Condition`and`Action`接口表示。

**规则可以用两种不同的方式定义：**

- 通过在POJO上添加注释，以声明方式定义
- 通过RuleBuilder API，以编程方式定义

1. **用注解定义规则**

这些是定义规则的最常用方法，但如果需要，还可以实现`Rule`i接口或继承`BasicRule`类。

```java
@Rule(name = "my rule", description = "my rule description", priority = 1)
public class MyRule {

    @Condition
    public boolean when(@Fact("fact") fact) {
        //my rule conditions
        return true;
    }

    @Action(order = 1)
    public void then(Facts facts) throws Exception {
        //my actions
    }

    @Action(order = 2)
    public void finally() throws Exception {
        //my final actions
    }

}
```

**@Condition**注解标记计算规则条件的方法。此方法必须是公共的，可以有一个或多个用@Fact注解的参数，并返回布尔类型。只有一个方法能用@Condition注解。

**@Action**注解标记要执行规则操作的方法。规则可以有多个操作。可以使用order属性按指定的顺序执行操作。默认情况下，操作的顺序为0。

**2. 用RuleBuilder API定义规则**

```text
Rule rule = new RuleBuilder()
                .name("myRule")
                .description("myRuleDescription")
                .priority(3)
                .when(condition)
                .then(action1)
                .then(action2)
                .build();
```

在这个例子中,   Condition实例condition，Action实例是action1和action2。 

## 2.3 定义事实

Facts API是一组事实的抽象，在这些事实上检查规则。在内部，Facts实例持有HashMap<String，Object>，这意味着：

- 事实需要命名，应该有一个唯一的名称，且不能为空
- 任何Java对象都可以充当事实

这里有一个实例定义事实:

```java
Facts facts = new Facts();
facts.add("rain", true);
```

Facts 能够被注入规则条件，action 方法使用 `@Fact` 注解. 在下面的规则中，`rain` 事实被注入itRains方法的`rain`参数:

```java
@Rule
class WeatherRule {

    @Condition
    public boolean itRains(@Fact("rain") boolean rain) {
        return rain;
    }

    @Action
    public void takeAnUmbrella(Facts facts) {
        System.out.println("It rains, take an umbrella!");
        // can add/remove/modify facts
    }

}
```

`Facts`类型参数 被注入已知的 facts中 (像action方法`takeAnUmbrella`一样).

如果缺少注入的fact, 这个引擎会抛出 `RuntimeException`异常.

## 2.4 定义规则引擎

从版本3.1开始，Easy Rules提供了RulesEngine接口的两种实现：

- DefaultRulesEngine：根据规则的自然顺序（默认为优先级）应用规则。
- InferenceRulesEngine：持续对已知事实应用规则，直到不再应用规则为止。

**创建一个规则引擎**

要创建规则引擎，可以使用每个实现的构造函数：

```text
RulesEngine rulesEngine = new DefaultRulesEngine();
// or
RulesEngine rulesEngine = new InferenceRulesEngine();
```

然后，您可以按以下方式触发注册规则：

```java
rulesEngine.fire(rules, facts);
```

**规则引擎参数**

Easy Rules 引擎可以配置以下参数：

```text
Parameter	Type	Required	Default
rulePriorityThreshold	int	no	MaxInt
skipOnFirstAppliedRule	boolean	no	false
skipOnFirstFailedRule	boolean	no	false
skipOnFirstNonTriggeredRule	boolean	no	false
```

- skipOnFirstAppliedRule：告诉引擎规则被触发时跳过后面的规则。
- skipOnFirstFailedRule：告诉引擎在规则失败时跳过后面的规则。
- skipOnFirstNonTriggeredRule：告诉引擎一个规则不会被触发跳过后面的规则。
- rulePriorityThreshold：告诉引擎如果优先级超过定义的阈值，则跳过下一个规则。版本3.3已经不支持更改，默认MaxInt。

可以使用RulesEngineParameters API指定这些参数：

```java
RulesEngineParameters parameters = new RulesEngineParameters()
    .rulePriorityThreshold(10)
    .skipOnFirstAppliedRule(true)
    .skipOnFirstFailedRule(true)
    .skipOnFirstNonTriggeredRule(true);

RulesEngine rulesEngine = new DefaultRulesEngine(parameters);
```

如果要从引擎获取参数，可以使用以下代码段：

```java
RulesEngineParameters parameters = myEngine.getParameters();
```

这允许您在创建引擎后重置引擎参数。

## 2.5 定义规则监听器

通过实现RuleListener接口

```java
public interface RuleListener {

    /**
     * Triggered before the evaluation of a rule.
     *
     * @param rule being evaluated
     * @param facts known before evaluating the rule
     * @return true if the rule should be evaluated, false otherwise
     */
    default boolean beforeEvaluate(Rule rule, Facts facts) {
        return true;
    }

    /**
     * Triggered after the evaluation of a rule.
     *
     * @param rule that has been evaluated
     * @param facts known after evaluating the rule
     * @param evaluationResult true if the rule evaluated to true, false otherwise
     */
    default void afterEvaluate(Rule rule, Facts facts, boolean evaluationResult) { }

    /**
     * Triggered on condition evaluation error due to any runtime exception.
     *
     * @param rule that has been evaluated
     * @param facts known while evaluating the rule
     * @param exception that happened while attempting to evaluate the condition.
     */
    default void onEvaluationError(Rule rule, Facts facts, Exception exception) { }

    /**
     * Triggered before the execution of a rule.
     *
     * @param rule the current rule
     * @param facts known facts before executing the rule
     */
    default void beforeExecute(Rule rule, Facts facts) { }

    /**
     * Triggered after a rule has been executed successfully.
     *
     * @param rule the current rule
     * @param facts known facts after executing the rule
     */
    default void onSuccess(Rule rule, Facts facts) { }

    /**
     * Triggered after a rule has failed.
     *
     * @param rule the current rule
     * @param facts known facts after executing the rule
     * @param exception the exception thrown when attempting to execute the rule
     */
    default void onFailure(Rule rule, Facts facts, Exception exception) { }

}
```

# 三、示例

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.cjs.example</groupId>
    <artifactId>easy-rules-quickstart</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>jar</packaging>
    <dependencies>
        <dependency>
            <groupId>org.jeasy</groupId>
            <artifactId>easy-rules-core</artifactId>
            <version>4.0.0</version>
        </dependency>
        <dependency>
            <groupId>org.jeasy</groupId>
            <artifactId>easy-rules-support</artifactId>
            <version>4.0.0</version>
        </dependency>
        <dependency>
            <groupId>org.jeasy</groupId>
            <artifactId>easy-rules-mvel</artifactId>
            <version>4.0.0</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-simple</artifactId>
            <version>1.7.30</version>
        </dependency>
    </dependencies>
</project>
```

![img](https://img2020.cnblogs.com/blog/874963/202006/874963-20200611101842791-232862128.png)

# 四、扩展

规则本质上是一个函数，如y=f(x1,x2,..,xn)

规则引擎就是为了解决业务代码和业务规则分离的引擎，是一种嵌入在应用程序中的组件，实现了将业务决策从应用程序代码中分离。

还有一种常见的方式是Java+Groovy来实现，Java内嵌Groovy脚本引擎进行业务规则剥离。

https://github.com/j-easy/easy-rules/wiki