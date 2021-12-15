# 使用快捷键移动分割线

假设有下面的场景，某个类的名字在`project`视图里被挡住了某一部分。![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdFhkeW4nTcNrUKruaVxgic9ufESc3Bj02WrrzYUhCRBdwlBCEjOpTADdrz1yQDKRiaFUZx2u2ZiaY2A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

你想完整的看到类的名字，该怎么做。一般都是使用鼠标来移动分割线，但是这样子效率太低了。可以使用`alt+1`把鼠标焦点定位到`project`视图里，然后直接使用`ctrl+shift+左右箭头`来移动分割线。



# ctrl+shift+enter不只是用来行尾加分号的

`ctrl+shift+enter`其实是表示`为您收尾`的意思，不只是用来给代码加分号的。比如说：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdFhkeW4nTcNrUKruaVxgic9umnOq7IcuyHUJGVl0NQnQjQSYvefH0ujEcZJ6CMGozia4pcomziarXfg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这段代码，我们还需要为if语句加上大括号才能编译通过，这个时候你直接输入`ctrl+shift+enter`，`IDEA`会自动帮你收尾，加上大括号的。

# 不要动不动就使用IDEA的重构功能

`IDEA`的重构功能非常强大，但是也有时候，在单个类里面，如果只是想批量修改某个文本，大可不必使用到重构的功能。比如说：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdFhkeW4nTcNrUKruaVxgic9jF0sEibNxIe1QqbpZwlI55b5BntPDzaFq2o6Z8iaWP8HT4mwMl8n7z4w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

上面的代码中，有5个地方用到了rabbitTemplate文本，如何批量修改呢？首先是使用`ctrl+w`选中`rabbitTemplate`这个文本,然后依次使用5次`alt+j`快捷键，逐个选中，这样五个文本就都被选中并且高亮起来了，这个时候就可以直接批量修改了。![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdFhkeW4nTcNrUKruaVxgic9UFOH8fcibT0c9TNFDXrLpEWJfbBbXt6ia14M6INibsP6QFpXydh2ODt2w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# 强大的symbol

如果你依稀记得某个方法名字几个字母，想在`IDEA`里面找出来，可以怎么做呢？直接使用`ctrl+shift+alt+n`，使用`symbol`来查找即可。比如说：![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdFhkeW4nTcNrUKruaVxgic9eZ3AgjCXv8XWonTNqJgDMp66IBLxyKYOEugf768XQXzlqBfHZ98ZTQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

你想找到checkUser方法。直接输入`user`即可。![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdFhkeW4nTcNrUKruaVxgic9ILwc2QFmEn9Vy75icOdMemDcAJIcrHDUPjcoH9Tzibtz1Gcf57rKsU4g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如果你记得某个业务类里面有某个方法，那也可以使用首字母找到类,然后加个`.`，再输入方法名字也是可以的。![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdFhkeW4nTcNrUKruaVxgic96gaM1b51rnxheRsicNWW0ZYE8uBc4BeiaJsrYWavJtcBvIXTfbY4a0LA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# 如何找目录

使用`ctrl+shift+n`后，使用`/`，然后输入目录名字即可.![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdFhkeW4nTcNrUKruaVxgic92vDw0wCb1AzqAC63r4o3dzYPiaR0YugTPoVzKibdhMCk8uYXFYrkwVtg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# 自动生成not null判断语句

自动生成not null这种if判断，在`IDEA`里有很多种办法，其中一种办法你可能没想到。![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdFhkeW4nTcNrUKruaVxgic9pia6oLa5T8YAb3cE2Kn9XXzINFcovtJEbEuibH1xkBE4u8rVSX7DWSIQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

当我们使用rabbitTemplate. 后，直接输入`notnull`并回车，`IDEA`就好自动生成if判断了。![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfdFhkeW4nTcNrUKruaVxgic9NIXfbYroT7Zex4fic8CRUAFv20turVuhROFhOjicH8Q25HMPHyttianVw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# 前进/后退

当我们编辑代码时，点击查看了调用类实现逻辑，然后可以使用后退快捷键，快速回到刚才待编辑的代码处。

- Ctrl + Alt + Left/Right（方向键）

![](https://img2020.cnblogs.com/other/1419561/202007/1419561-20200714072035593-1548501813.gif)

# 查看最近修改代码的位置，直接点击快速跳转。

ctrl + shift + E
![](https://img2020.cnblogs.com/other/1419561/202007/1419561-20200714072038615-358356755.gif)

# 快速抽取变量

Windows：ctrl + alt + V
![](https://img2020.cnblogs.com/other/1419561/202007/1419561-20200714072043281-1570387292.gif)



