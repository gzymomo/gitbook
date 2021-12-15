uniapp小白入门自学笔记（一）：https://www.cnblogs.com/echoyya/p/14427845.html



### 环境搭建

官网文档解释：`uni-app` 是一个使用 [Vue.js](https://vuejs.org/) 开发所有前端应用的框架，开发者编写一套代码，可发布到iOS、Android、Web（响应式）、以及各种小程序（微信/支付宝/百度/头条/QQ/钉钉/淘宝）、快应用等多个平台。

需要下载开发工具：HBuilderX是通用的前端开发工具，但为`uni-app`做了特别强化。下载App开发版，可开箱即用；

- HBuilderX：[下载地址](https://www.dcloud.io/hbuilderx.html)
- 微信开发者工具：[下载地址](https://developers.weixin.qq.com/miniprogram/dev/devtools/download.html)

#### 创建uni-app

打开开发工具，HBuilderX，点击工具栏里的**文件 -> 新建 -> 项目**-> 选择uni-app，输入项目名，选择模板，点击创建
 ![img](https://img2020.cnblogs.com/blog/1238759/202102/1238759-20210221231738980-645360391.png)

#### 运行uni-app

- 浏览器运行：进入项目，点击工具栏的运行 -> 运行到浏览器 -> 选择浏览器
- 微信开发者工具里运行：进入项目，点击工具栏的运行 -> 运行到小程序模拟器 -> 微信开发者工具，
  - 如果是第一次使用，需要先配置小程序ide的相关路径，才能运行成功。需在输入框中输入微信开发者工具的安装路径。
  - 微信开发者工具 -> 设置 -> 安全设置 -> 安全 -> 开启服务端口

![img](https://img2020.cnblogs.com/blog/1238759/202102/1238759-20210221231852268-1159948572.png)

![img](https://img2020.cnblogs.com/blog/1238759/202102/1238759-20210221231908151-1794374334.png)

### 介绍项目目录，文件作用

- `pages.json` ：对 uni-app 进行全局配置，决定页面文件的路径、窗口样式、原生的导航栏、底部的原生tabbar 等。
- `manifest.json` ：是应用的配置文件，用于指定应用的名称、图标、权限等
- `App.vue`：是uni-app的主组件，所有页面都是在`App.vue`下进行切换的，是页面入口文件。但其本身不是页面，这里不能编写视图元素。可以调用应用生命周期函数、配置全局样式、配置全局的存储globalData
- `main.js`：是uni-app的入口文件，主要作用是初始化`vue`实例、定义全局组件、插件等。
- `uni.scss`：为了方便整体控制应用的风格。如按钮颜色、边框风格，`uni.scss`文件里预置了一批scss变量预置。
- `unpackage`：项目打包目录，存在各个平台的打包文件
- `pages`：所有的页面存放目录
- `static`：静态资源目录，图片等
- `comonents`：组件存放目录

### 页面外观设置

[全局配置 globalstyle ](https://uniapp.dcloud.io/collocation/pages?id=globalstyle):用于设置应用的状态栏、导航条、标题、窗口背景色等。列举以下常用

| 属性                         | 类型     | 默认值  | 描述                                               |
| :--------------------------- | :------- | :------ | :------------------------------------------------- |
| navigationBarBackgroundColor | HexColor | #F7F7F7 | 导航栏背景颜色（同状态栏背景色）                   |
| navigationBarTextStyle       | String   | white   | 导航栏标题颜色及状态栏前景颜色，仅支持 black/white |
| navigationBarTitleText       | String   |         | 导航栏标题文字内容                                 |
| backgroundColor              | HexColor | #ffffff | 下拉显示出来的窗口的背景色                         |
| backgroundTextStyle          | String   | dark    | 下拉 loading 的样式，仅支持 dark / light           |
| enablePullDownRefresh        | Boolean  | false   | 是否开启下拉刷新                                   |
| onReachBottomDistance        | Number   | 50      | 页面上拉触底事件触发时距页面底部距离，单位只支持px |

[页面配置 pageStyle](https://uniapp.dcloud.io/collocation/pages?id=style)

`uni-app` 通过 pages 节点配置应用由哪些页面组成，pages 节点接收一个数组，数组每个项都是一个对象

- pages节点的第一项为应用入口页（即首页）
- **应用中新增/减少页面**，都需要对 pages 数组进行修改，且**不需写后缀**，框架会自动寻找路径下的页面资源

| 属性  | 类型   | 描述                                                         |
| :---- | :----- | :----------------------------------------------------------- |
| path  | String | 配置页面路径                                                 |
| style | Object | 配置页面窗口表现，配置项参考 [pageStyle](https://uniapp.dcloud.io/collocation/pages?id=style) |

### 页面底部 [tabBar](https://uniapp.dcloud.io/collocation/pages?id=tabbar)

若应用是一个多 tab 应用，可以通过 tabBar 配置项指定一级导航栏，以及 tab 切换时显示的对应页。

在 pages.json 中提供 tabBar 配置，不仅方便快速开发导航，更重要的是在App和小程序端提升性能。

- 当设置 position 为 top 时，将不会显示 icon
- tabBar 中的 list 是一个数组，只能配置最少2个、最多5个 tab，tab 按数组的顺序排序。
- tabbar 的页面展现过一次后就保留在内存中，再次切换 tabbar 页面，只会触发每个页面的onShow，不会再触发onLoad。

**属性说明：**

| 属性            | 类型     | 必填 | 默认值 | 描述                                                 |
| :-------------- | :------- | :--- | :----- | :--------------------------------------------------- |
| color           | HexColor | 是   |        | tab 上的文字默认颜色                                 |
| selectedColor   | HexColor | 是   |        | tab 上的文字选中时的颜色                             |
| backgroundColor | HexColor | 是   |        | tab 的背景色                                         |
| borderStyle     | String   | 否   | black  | tabbar 上边框的颜色，可选值 black/white              |
| list            | Array    | 是   |        | tab 的列表，详见 list 属性说明，最少2个、最多5个 tab |
| position        | String   | 否   | bottom | 可选值 bottom、top                                   |

**其中 list 接收一个数组，数组中的每个项都是一个对象，其属性值如下：**

| 属性             | 类型   | 必填 | 说明                                                         |
| :--------------- | :----- | :--- | :----------------------------------------------------------- |
| pagePath         | String | 是   | 页面路径，必须在 pages 中先定义                              |
| text             | String | 是   | tab 上按钮文字，在 App 和 H5 平台为非必填。                  |
| iconPath         | String | 否   | 图片路径，当 postion 为 top 时，此参数无效，不支持网络图片和字体图标 |
| selectedIconPath | String | 否   | 选中时的图片路径，当 postion 为 top 时，此参数无效           |

### 启动模式配置 [condition](https://uniapp.dcloud.io/collocation/pages?id=condition)

仅开发期间生效，用于模拟直达页面的场景，如：小程序转发后，用户点击所打开的页面。

**属性说明：**

| 属性    | 类型   | 是否必填 | 描述                             |
| :------ | :----- | :------- | :------------------------------- |
| current | Number | 是       | 当前激活的模式，list节点的索引值 |
| list    | Array  | 是       | 启动模式列表                     |

**list说明：**

| 属性  | 类型   | 是否必填 | 描述                                                         |
| :---- | :----- | :------- | :----------------------------------------------------------- |
| name  | String | 是       | 启动模式名称                                                 |
| path  | String | 是       | 启动页面路径                                                 |
| query | String | 否       | 启动参数，可在页面的 [onLoad](https://uniapp.dcloud.io/collocation/frame/lifecycle?id=页面生命周期) 函数里获得 |

```
// pages.json
"condition":{ //模式配置，仅开发期间生效
    "current":"0", //当前激活的模式（list 的索引项）
    "list":[
        {
            "name":"详情", //模式名称
            "query":"id=80", //启动页面，必选
            "path":"pages/detail/detail" //启动参数，在页面的onLoad函数里面得到。
        }
    ]
}
```

![img](https://img2020.cnblogs.com/blog/1238759/202102/1238759-20210221231956417-560120395.png)

### 数据及事件绑定

- 数据绑定：在页面中需要定义数据，和Vue一模一样，可以直接在data中定义数据即可，再利用插值表达式渲染，在插值表达式中支持三元运算和一些基本运算
- 动态绑定属性：同Vue，v-bind绑定，简写`:`
- 事件绑定及传参：同Vue，v-on绑定，简写`@`

### 组件使用

uni-app提供了一系列基础组件，类似HTML里的基础标签元素。但uni-app的组件与HTML不同，而是与小程序相同，更适合手机端使用。

虽然不推荐使用HTML标签，但实际上如果开发者写了`div`等标签，在编译到非H5平台时也会被编译器转换为`view`标签，类似的还有`span`转`text`、`a`转`navigator`等，包括css里的元素选择器也会转。

- 所有组件与属性名都是小写，单词之间以连字符`-`连接。
- 根节点为 `<template>`，这个 `<template>` 下只能且必须有一个根`<view>`组件。这是vue单文件组件规范。
- 所有组件都有的属性：**id，class，style，hidden，data-\*，@EventHandler**

列举几个常用的，但不同于传统html的组件，其余建议参考[官网详细文档](https://www.cnblogs.com/echoyya/p/14427845.html)

1. [view](https://uniapp.dcloud.io/component/view?id=view)：视图容器。类似于传统html中的div，用于包裹各种元素内容。
2. [text](https://uniapp.dcloud.io/component/text?id=text)：文本组件。类似于传统html中的span，用于包裹文本内容。
3. [image](https://uniapp.dcloud.io/component/image?id=image)：图片。类似于传统html中的span，用于显示图片元素。

### uni-app中的样式

> - rpx 即响应式px，根据屏幕宽度自适应的单位，以750宽的屏幕为基准。750rpx恰好为屏幕宽度，屏幕变宽，rpx实际显示效果会等比放大。
> - 支持基本常用选择器，class，id，element，但不能使用`*` 通配符选择器
> - `page`相当于`body`节点
> - 定义在App.vue中的样式为全局样式，作用于每一个页面，在pages目录下的vue文件中定义的样式为局部样式，只作用对应的页面，并会覆盖App.vue中相同的选择器
> - 同样支持 Less 和 Scss，不过需要下载对应的插件
> - 可通过`@import`导入外联样式表，并引用相对路径，用`;`表示语句结束
> - uni-app支持使用字体图标，使用方法与普通web项目相同，但需要注意：
>   - 字体文件小于40kb，uni-app会自动将其转化为base64格式，反之需开发者手动转换，否则将不生效
>   - 字体图标的引用路径推荐使用以`~@`开头的绝对路径

```
// App.vue 样式部分
<style>
	/*每个页面公共css */
	@import url("./static/font/iconfont.css");
</style>
// iconfont.css
@font-face {font-family: "iconfont";
  src: url('~@/static/font/iconfont.eot'); 
}
```

### 条件注释实现跨端兼容

#### 跨端兼容

uni-app 已将常用的组件、JS API 封装到框架中，开发者按照 uni-app 规范开发即可保证多平台兼容，大部分业务均可直接满足。但每个平台有自己的一些特性，因此会存在一些无法跨平台的情况。

- 大量写 if else，会造成代码执行性能低下和管理混乱。
- 编译到不同的工程后二次修改，会让后续升级变的很麻烦。

在 C 语言中，通过 #ifdef、#ifndef 的方式，为 windows、mac 等不同 os 编译不同的代码。 `uni-app` 参考这个思路，为 `uni-app` 提供了条件编译手段，在一个工程里优雅的完成了平台个性化实现。

#### 条件编译

条件编译是用特殊的注释作为标记，在编译时根据这些特殊的注释，将注释里面的代码编译到不同平台。

**支持的文件**：`·vue`，`.js`，`.css`，`pages.json`，各预编译语言文件，：`.scss、.less、.stylus、.ts、.pug`

**注意：**：条件编译是利用注释实现的，在不同语法里注释写法不一样

**写法：**以 #ifdef 或 #ifndef 加 **%PLATFORM%** 开头，以 #endif 结尾。

- `#ifdef`：if defined 仅在某平台存在
- `#ifndef`：if not defined 除了某平台均存在
- `%PLATFORM%`：平台名称

| 条件编译写法                                             | 说明                                                         |
| -------------------------------------------------------- | ------------------------------------------------------------ |
| #ifdef **APP-PLUS** 需条件编译的代码 #endif              | 仅出现在 App 平台下的代码                                    |
| #ifndef **H5** 需条件编译的代码 #endif                   | 除了 H5 平台，其它平台均存在的代码                           |
| #ifdef **H5** \|\| **MP-WEIXIN** 需条件编译的代码 #endif | 在 H5 平台或微信小程序平台存在的代码（这里只有\|\|，不可能出现&&，因为没有交集） |

**%PLATFORM%** **可取值如下：**全部取值可参考[官方文档](https://uniapp.dcloud.io/platform?id=条件编译)

| 值        | 平台         |
| :-------- | :----------- |
| APP-PLUS  | App          |
| H5        | H5           |
| MP-WEIXIN | 微信小程序   |
| MP-ALIPAY | 支付宝小程序 |

![img](https://img2020.cnblogs.com/blog/1238759/202102/1238759-20210221232038812-1992939593.png)

```
<template>
	<view>
		<!-- #ifdef H5 -->
		<view>仅展示在H5中 </view>
		<!-- #endif -->
		<!-- #ifdef MP-WEIXIN -->
		<view>仅展示在微信小程序中</view>
		<!-- #endif -->
		<text>{{msg}}</text>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				msg: 'hello,'
			}
		},
		onLoad() {
			// #ifdef H5
			this.msg += 'H5'
			// #endif
			// #ifdef MP-WEIXIN
			this.msg += '微信小程序'
			//  #endif
		}
	}
</script>

<style>
	/* 第一种写法，针对样式做注释 */
	text {
		/* #ifdef H5 */
		color: #8470FF;
		/* #endif */
		/* #ifdef MP-WEIXIN */
		color: #8DEEEE;
		/* #endif */
	}
	
	/* 第二种写法，针对选择器做注释 */
	/* #ifdef MP-WEIXIN */
	text {
		color: #8DEEEE;
	}
	/* #endif */
</style>
```