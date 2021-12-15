# 完全不必要的 Else 块

![图片](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsiauPlicWfYlFoViboicOkg744edAZHRYI9YKkalaWshfBIGIWcKNvibeKHsGxMB2SnzCpHibZiaWeqRict1w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



# 价值分配

如果你要根据提供的某些输入为变量分配新值，请停止 If-Else 废话，一种更具可读性的方法。

![图片](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsiauPlicWfYlFoViboicOkg744eceaUk7AyS3YSB7CFZ3EvosANDV5hbRVRN81u5emx8nbqIVcTKQnZNQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



尽管很简单，但它却很糟糕。首先，If-Else 很容易在这里被开关取代。但是，我们可以通过完全删除 else 来进一步简化此代码。

![图片](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsiauPlicWfYlFoViboicOkg744eO9avo4DiaCKAibSpMHiaD1oAODMj3bk67XLsWFYBHnficL5oD8M5ib9d4tw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



# 前提条件检查

通常，我发现，如果方法提供了无效的值，则继续执行是没有意义的。假设我们从以前就有了 DefineGender 方法，要求提供的输入值必须始终为 0 或 1。

![图片](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsiauPlicWfYlFoViboicOkg744eIJULIwneplr5kAAtU47ZLL9yg3HhSv3IckrztibyBqMibTPGkSV6xsdA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



在没有价值验证的情况下执行该方法没有任何意义。因此，在允许方法继续执行之前，我们需要检查一些先决条件。

应用保护子句防御性编码技术，你将检查方法的输入值，然后继续执行方法。

![图片](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsiauPlicWfYlFoViboicOkg744eBARLthSXkuYckp1xPuib9R6o3tcEnGs9NKtpGS81K2KEGNyTQibWEhpA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



至此，我们确保仅在值落在预期范围内时才执行主逻辑。现在，IF 也已被三元代替，因为不再需要在结尾处默认返回"未知"。

# 将 If-Else 转换为字典，完全避免 If-Else

假设您需要执行一些操作，这些操作将根据某些条件进行选择，我们知道以后必须添加更多操作。

![图片](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsiauPlicWfYlFoViboicOkg744e1RzcfpmgJl6jw3mX7n2ia5oy4cZ0Hr0Zf6YiaH72pPeyFcoy6umaQ6mA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



也许有人倾向于使用久经考验的 If-Else。如果添加新操作，则只需简单地添加其他内容即可。很简单 但是，就维护而言，这种方法不是一个好的设计。



知道我们以后需要添加新的操作后，我们可以将 If-Else 重构为字典。

![图片](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsiauPlicWfYlFoViboicOkg744eHgZ6PWwGvVQdnVFc9a4CC08QW2ugD4X4I37cnA2bzn7GWegmufIofg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



可读性已大大提高，并且可以更轻松地推断出该代码。注意，仅出于说明目的将字典放置在方法内部。您可能希望从其他地方提供它。



# 扩展应用程序，完全避免使用 If-Else

这是一个稍微高级的示例。通过用对象替换它们，知道何时甚至完全消除 If。

通常，您会发现自己不得不扩展应用程序的某些部分。作为初级开发人员，您可能会倾向于通过添加额外的 If-Else（即 else-if）语句来做到这一点。

举这个说明性的例子。在这里，我们需要将 Order 实例显示为字符串。首先，我们只有两种字符串表示形式：JSON 和纯文本。



在此阶段使用 If-Else 并不是什么大问题，如果我们可以轻松替换其他，只要如前所述即可。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)![图片](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsiauPlicWfYlFoViboicOkg744ecLtN7KbNHIibIQlvQGC0kibOSkMIAyskl5JIN6ib0fCVMUp2epgMln8fw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



知道我们需要扩展应用程序的这一部分，这种方法绝对是不可接受的。



上面的代码不仅违反了"打开/关闭"原则，而且阅读得不好，还会引起可维护性方面的麻烦。



正确的方法是遵循 SOLID 原则的方法，我们通过实施动态类型发现过程（在本例中为策略模式）来做到这一点。



重构这个混乱的过程的过程如下：

- **使用公共接口将每个分支提取到单独的策略类中。**
- **动态查找实现通用接口的所有类。**
- **根据输入决定执行哪种策略。**



替换上面示例的代码如下所示。是的，这是更多代码的方式。它要求您了解类型发现的工作原理。但是动态扩展应用程序是一个高级主题。



我只显示将替换 If-Else 示例的确切部分。如果要查看所有涉及的对象，请查看此要点。

![图片](https://mmbiz.qpic.cn/mmbiz_png/oTKHc6F8tsiauPlicWfYlFoViboicOkg744eUBjHmAxlXJ418iadDSLfDgGBpYSsSPibQf7lcWJ9t30CX4iagIzJ5tBBg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

让我们快速浏览一下代码。方法签名保持不变，因为调用者不需要了解我们的重构。



首先，获取实现通用接口 IOrderOutputStrategy 的程序集中的所有类型。然后，我们建立一个字典，格式化程序的 displayName 的名称为 key，类型为 value。



然后从字典中选择格式化程序类型，然后尝试实例化策略对象。最后，调用策略对象的 ConvertOrderToString。