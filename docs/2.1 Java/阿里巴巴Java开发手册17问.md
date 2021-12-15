# 为什么禁止使用 BigDecimal 的 equals 方法做等值比较？

BigDecimal，相信对于很多人来说都不陌生，很多人都知道他的用法，这是一种java.math 包中提供的一种可以用来进行精确运算的类型。

很多人都知道，在进行金额表示、金额计算等场景，不能使用 double、float 等类型，而是要使用对精度支持的更好的 BigDecimal。

所以，很多支付、电商、金融等业务中，BigDecimal 的使用非常频繁。而且不得不说这是一个非常好用的类，其内部自带了很多方法，如加，减，乘，除等运算方法都是可以直接调用的。

除了需要用 BigDecimal 表示数字和进行数字运算以外，代码中还经常需要对于数字进行相等判断。

关于这个知识点，在最新版的《Java 开发手册》中也有说明：

![image-20210723103323599](https://gitee.com/er-huomeng/l-img/raw/master/img/image-20210723103323599.png)

我在之前的 CodeReview 中，看到过以下这样的低级错误：

```java
if(bigDecimal == bigDecimal1){
// 两个数相等
}
```

这种错误，相信聪明的读者一眼就可以看出问题，**因为 BigDecimal 是对象，所以不能用==来判断两个数字的值是否相等。**

以上这种问题，在有一定的经验之后，还是可以避免的，但是聪明的读者，看一下以下这行代码，你觉得他有问题吗：

```java
if(bigDecimal.equals(bigDecimal1)){
// 两个数相等
}
```

可以明确的告诉大家，以上这种写法，可能得到的结果和你预想的不一样！

先来做个实验，运行以下代码：

```java
BigDecimal bigDecimal = new BigDecimal(1);
BigDecimal bigDecimal1 = new BigDecimal(1);
System.out.println(bigDecimal.equals(bigDecimal1));
BigDecimal bigDecimal2 = new BigDecimal(1);
BigDecimal bigDecimal3 = new BigDecimal(1.0);
System.out.println(bigDecimal2.equals(bigDecimal3));
BigDecimal bigDecimal4 = new BigDecimal("1");
BigDecimal bigDecimal5 = new BigDecimal("1.0");
System.out.println(bigDecimal4.equals(bigDecimal5));
```

以上代码，输出结果为：

```java
True
True
false
```

## BigDecimal 的 equals 原理

通过以上代码示例，我们发现，在使用 BigDecimal 的 equals 方法对 1 和 1.0 进行比较的时候，有的时候是 true（当使用 int、double 定义 BigDecimal 时），有的时候是false（当使用 String 定义 BigDecimal 时）。

那么，为什么会出现这样的情况呢，我们先来看下 BigDecimal 的 equals 方法。

在 BigDecimal 的 JavaDoc 中其实已经解释了其中原因：

```java
Compares this BigDecimal with the specified Object for equality. Unlike
compareTo, this method considers two BigDecimal objects equal only if they are
equal in value and scale (thus 2.0 is not equal to 2.00 when compared by this
method)
```

大概意思就是，**equals 方法和 compareTo 并不一样，equals 方法会比较两部分内容，分别是值（value）和标度（scale）**

对应的代码如下：

![image-20210723103459819](https://gitee.com/er-huomeng/l-img/raw/master/img/image-20210723103459819.png)

所 以 ， 我 们 以 上 代 码 定 义 出 来 的 两 个 BigDecimal 对 象 （ bigDecimal4 和bigDecimal5）的标度是不一样的，所以使用 equals 比较的结果就是 false 了。

尝试着对代码进行 debug，在 debug 的过程中我们也可以看到 bigDecimal4 的标度时 0，而 bigDecimal5 的标度是 1。

![image-20210723103520004](https://gitee.com/er-huomeng/l-img/raw/master/img/image-20210723103520004.png)

到这里，我们大概解释清楚了，之所以 equals 比较 bigDecimal4 和 bigDecimal5的结果是 false，是因为标度不同。

那么，为什么标度不同呢？为什么 bigDecimal2 和 bigDecimal3 的标度是一样的（当使用 int、double 定义 BigDecimal 时），而 bigDecimal4 和 bigDecimal5 却不一样（当使用 String 定义 BigDecimal 时）呢？

## 为什么标度不同

这个就涉及到 BigDecimal 的标度问题了，这个问题其实是比较复杂的，由于不是本文的重点，这里面就简单介绍一下吧。大家感兴趣的话，后面单独讲。

首先，BigDecimal 一共有以下 4 个构造方法：

```java
BigDecimal(int)
BigDecimal(double)
BigDecimal(long)
BigDecimal(String)
```

以上四个方法，创建出来的的 BigDecimal 的标度是不同的。

**BigDecimal(long) 和 BigDecimal(int)**

首先，最简单的就是 **BigDecimal(long)** 和 **BigDecimal(int)**，**因为是整数，所以标度就是 0** ：

```java
public BigDecimal(int val) {
    this.intCompact = val;
    this.scale = 0;
    this.intVal = null;
}
public BigDecimal(long val) {
    this.intCompact = val;
    this.intVal = (val == INFLATED) ? INFLATED_BIGINT : null;
    this.scale = 0;
}
```

BigDecimal(double)

而 对 于 BigDecimal(double) ， **当 我 们 使 用 new BigDecimal(0.1) 创 建 一 个BigDecimal 的 时 候 ， 其 实 创 建 出 来 的 值 并 不 是 整 好 等 于 0.1 的 ， 而 是0.1000000000000000055511151231257827021181583404541015625 。这是因为doule 自身表示的只是一个近似值。**



那么，无论我们使用 new BigDecimal(0.1)还是 new BigDecimal(0.10)定义，他的近似值都是 0.1000000000000000055511151231257827021181583404541015625这个，那么他的标度就是这个数字的位数，即 55。

![image-20210723103652468](https://gitee.com/er-huomeng/l-img/raw/master/img/image-20210723103652468.png)

其他的浮点数也同样的道理。对于 new BigDecimal(1.0)这样的形式来说，因为他本质上也是个整数，所以他创建出来的数字的标度就是 0。

所以，因为 BigDecimal(1.0)和 BigDecimal(1.00)的标度是一样的，所以在使用equals 方法比较的时候，得到的结果就是 true。

BigDecimal(string)

而对于 BigDecimal(double) ，**当我们使用 new BigDecimal(“0.1”)创建一个BigDecimal 的时候，其实创建出来的值正好就是等于 0.1 的。那么他的标度也就是 1。**

如果使用 new BigDecimal(“0.10000”)，那么创建出来的数就是 0.10000，标度也就是 5。

所以，因为 BigDecimal(“1.0”)和 BigDecimal(“1.00”)的标度不一样，所以在使用 equals 方法比较的时候，得到的结果就是 false。

## 如何比较 BigDecimal

前面，我们解释了 BigDecimal 的 equals 方法，其实不只是会比较数字的值，还会对其标度进行比较。

所以，当我们使用 equals 方法判断判断两个数是否相等的时候，是极其严格的。

那么，如果我们只想判断两个 BigDecimal 的值是否相等，那么该如何判断呢？

**BigDecimal 中提供了 compareTo 方法，这个方法就可以只比较两个数字的值，如果两个数相等，则返回 0。**

```java
BigDecimal bigDecimal4 = new BigDecimal("1");

BigDecimal bigDecimal5 = new BigDecimal("1.0000");

System.out.println(bigDecimal4.compareTo(bigDecimal5));
```

以上代码，输出结果：

0

其源码如下：

![image-20210723103758077](https://gitee.com/er-huomeng/l-img/raw/master/img/image-20210723103758077.png)

## 总结

BigDecimal 是一个非常好用的表示高精度数字的类，其中提供了很多丰富的方法。

但是，他的 equals 方法使用的时候需要谨慎，因为他在比较的时候，不仅比较两个数字的值，还会比较他们的标度，只要这两个因素有一个是不相等的，那么结果也是 false。

如果读者想要对两个BigDecimal 的数值进行比较的话，可以使用compareTo方法。

# 为什么禁止使用double直接构造BigDecimal?

BigDecimal，相信对于很多人来说都不陌生，很多人都知道他的用法，这是一种java.math 包中提供的一种可以用来进行精确运算的类型。

很多人都知道，在进行金额表示、金额计算等场景，不能使用 double、float 等类型，而是要使用对精度支持的更好的 BigDecimal。

所以，很多支付、电商、金融等业务中，BigDecimal 的使用非常频繁。但是，如果误以为只要使用 BigDecimal 表示数字，结果就一定精确，那就大错特错了！

在之前的一篇文章中，我们介绍过，使用 BigDecimal 的 equals 方法并不能验证两个数是否真的相等（为什么禁止使用 BigDecimal 的 equals 方法做等值比较？）。

除了这个情况，BigDecimal 的使用的第一步就是创建一个 BigDecimal 对象，如果这一步都有问题，那么后面怎么算都是错的！

那到底应该如何正确的创建一个 BigDecimal？

**关于这个问题，我 Review 过很多代码，也面试过很多一线开发，很多人都掉进坑里过。这是一个很容易被忽略，但是又影响重大的问题。**

关于这个问题，在《Java 开发手册》中有一条建议，或者说是要求：

![image-20210723103852538](https://gitee.com/er-huomeng/l-img/raw/master/img/image-20210723103852538.png)

这是一条【强制】建议，那么，这背后的原理是什么呢？

想要搞清楚这个问题，主要需要弄清楚以下几个问题：

1. 为什么说 double 不精确？

2. BigDecimal 是如何保证精确的？

在 知 道 这 两 个 问 题 的 答 案 之 后 ， 我 们 也 就 大 概 知 道 为 什 么 不 能 使 用BigDecimal(double)来创建一个 BigDecimal 了。

## double 为什么不精确

首先，**计算机是只认识二进制的**，即 0 和 1，这个大家一定都知道。

那么，所有数字，包括整数和小数，想要在计算机中存储和展示，都需要转成二进制。

**十进制整数转成二进制很简单，通常采用”除 2 取余，逆序排列”即可，如 10 的二进制为 1010。**

但是，小数的二进制如何表示呢？

十进制小数转成二进制，一般采用”乘 2 取整，顺序排列”方法，如 0.625 转成二进制的表示为 0.101。

但是，并不是所有小数都能转成二进制，如 0.1 就不能直接用二进制表示，他的二进

制是 0.000110011001100… 这是一个无限循环小数。

**所以，计算机是没办法用二进制精确的表示 0.1 的。也就是说，在计算机中，很多小数没办法精确的使用二进制表示出来。**

那么，这个问题总要解决吧。那么，人们想出了一种采用一定的精度，使用近似值表示一个小数的办法。这就是 IEEE 754（IEEE 二进制浮点数算术标准）规范的主要思想。

IEEE 754 规定了多种表示浮点数值的方式，其中最常用的就是 32 位单精度浮点数和 64 位双精度浮点数。

在 Java 中，使用 float 和 double 分别用来表示单精度浮点数和双精度浮点数。

所谓精度不同，可以简单的理解为保留有效位数不同。采用保留有效位数的方式近似的表示小数。

所以，大家也就知道为什么 double 表示的小数不精确了。

接下来，再回到 BigDecimal 的介绍，我们接下来看看是如何表示一个数的，他如何保证精确呢？

## BigDecimal 如何精确计数？

如果大家看过 BigDecimal 的源码，其实可以发现，**实际上一个 BigDecimal 是通过一个”无标度值”和一个”标度”来表示一个数的。**

在 BigDecimal 中，标度是通过 scale 字段来表示的。

而 无 标 度 值 的 表 示 比 较 复 杂 。 当 unscaled value 超 过 阈 值 ( 默 认 为Long.MAX_VALUE)时采用 intVal 字段存储 unscaled value，intCompact 字段存储Long.MIN_VALUE，否则对 unscaled value 进行压缩存储到 long 型的 intCompact字段用于后续计算，intVal 为空。

涉及到的字段就是这几个：

![image-20210723104153819](https://gitee.com/er-huomeng/l-img/raw/master/img/image-20210723104153819.png)

关于无标度值的压缩机制大家了解即可，不是本文的重点，大家只需要知道BigDecimal 主要是通过一个无标度值和标度来表示的就行了。

## 那么标度到底是什么呢？

除了 scale 这个字段，在 BigDecimal 中还提供了 scale()方法，用来返回这个

BigDecimal 的标度。

```java
/** * Returns the <i>scale</i> of this {@code BigDecimal}. If zero * or
positive, the scale is the number of digits to the right of * the decimal point.
If negative, the unscaled value of the
* number is multiplied by ten to the power of the negation of the
* scale. For example, a scale of {@code -3} means the unscaled * value is
multiplied by 1000. *
* @return the scale of this {@code BigDecimal}.
*/
public int scale() {
	return scale;
}
```

那么，scale 到底表示的是什么，其实上面的注释已经说的很清楚了：

如果 scale 为零或正值，则该值表示这个数字小数点右侧的位数。如果 scale 为负数，则该数字的真实值需要乘以 10 的该负数的绝对值的幂。例如，scale 为-3，则这个数需要乘 1000，即在末尾有 3 个 0。

如 123.123，那么如果使用 BigDecimal 表示，那么他的无标度值为 123123，他的标度为 3。

**而二进制无法表示的 0.1，使用 BigDecimal 就可以表示了，及通过无标度值 1 和标度 1 来表示。**

我们都知道，想要创建一个对象，需要使用该类的构造方法，在 BigDecimal 中一共有以下 4 个构造方法：

以上四个方法，创建出来的的 BigDecimal 的标度（scale）是不同的。

其中 BigDecimal(int)和 BigDecimal(long) 比较简单，因为都是整数，所以他们的标度都是 0。

而 BigDecimal(double) 和 BigDecimal(String)的标度就有很多学问了。

## BigDecimal(double)有什么问题？

BigDecimal 中 提 供 了 一 个 通 过 double 创 建 BigDecimal 的 方 法 — —BigDecimal(double) ，但是，同时也给我们留了一个坑！

因为我们知道，double 表示的小数是不精确的，如 0.1 这个数字，double 只能表示他的近似值。

所以，**当我们使用 new BigDecimal(0.1)创建一个 BigDecimal 的时候，其实创建出来的值并不是正好等于 0.1 的。**

而是 0.1000000000000000055511151231257827021181583404541015625。

这是因为 doule 自身表示的只是一个近似值。

![image-20210723104429396](https://gitee.com/er-huomeng/l-img/raw/master/img/image-20210723104429396.png)

**所以，如果我们在代码中，使用 BigDecimal(double) 来创建一个BigDecimal 的话，那么是损失了精度的，这是极其严重的。**

## 使用 BigDecimal(String)创建

那么，该如何创建一个精确的 BigDecimal 来表示小数呢，答案是使用 String 创建。

而对于 BigDecimal(String) ，当我们使用 new BigDecimal(“0.1”)创建一个BigDecimal 的时候，其实创建出来的值正好就是等于 0.1 的。

那么他的标度也就是 1。

但是需要注意的是，new BigDecimal(“0.10000”和 new BigDecimal(“0.1”这两个数的标度分别是 5 和 1，如果使用 BigDecimal 的 equals 方法比较，得到的结果是 false，具体原因和解决办法参考为什么禁止使用 BigDecimal 的 equals 方法做等值比较？

那么，想要创建一个能精确的表示 0.1 的 BigDecimal，请使用以下两种方式：

这里，留一个思考题，BigDecimal.valueOf()是调用 Double.toString 方法实现的，

那么，既然 double 都是不精确的，BigDecimal.valueOf(0.1)怎么保证精确呢？

## 总结

因为计算机采用二进制处理数据，但是很多小数，如 0.1 的二进制是一个无线循环小数，而这种数字在计算机中是无法精确表示的。

所以，人们采用了一种通过近似值的方式在计算机中表示，于是就有了单精度浮点数和双精度浮点数等。

所以，作为单精度浮点数的 float 和双精度浮点数的 double，在表示小数的时候只是近似值，并不是真实值。

所以，当使用 BigDecimal(Double)创建一个的时候，得到的 BigDecimal 是损失了精度的。

而使用一个损失了精度的数字进行计算，得到的结果也是不精确的。

想要避免这个问题，可以通过 BigDecimal(String)的方式创建 BigDecimal，这样的情况下，0.1 就会被精确的表示出来。

其表现形式是一个无标度数值 1，和一个标度 1 的组合。