围绕以下几个Vue.js的基本（核心）使用方法逐个做使用说明：

[TOC]

<br/>


## 新建vue
----
**1、引用vue.js**

在html文件中，引入 vue.js 的CDN（或本地）地址
	
	<script src="https://cdn.jsdelivr.net/npm/vue@2.5.16/dist/vue.js"></script>

**2、新建Vue实例**

在html中插入一个id为app的`<div>`
    
	<div id="app"></div>

 在html中插入下面js代码:
	
	<script type="text/javascript">
		var vm = new Vue({
			el:"#app"
		})
	</script>

然后整个代码看起来是这样的:
    
	<!DOCTYPE html>
	<html>
		<head>
			<title>Vue Demo</title>
			<script src="https://cdn.jsdelivr.net/npm/vue@2.5.16/dist/vue.js"></script>
		</head>
		<body>
			<div id="app">
			</div>
		</body>
		<script type="text/javascript">
			var vm = new Vue({
				el: "#app"
			})
		</script>
	</html>

> 注：js变量 vm 就是Vue创建的一个对象，可以理解成把`<div id="app></div>`和这个标签里面包含的所有DOM都实例化成了一个JS对象， 这个对象就是 vm 。   
> el是Vue的保留字，用来指定实例化的DOM的ID号, `#app`这句话就是标签选择器，告诉Vue要实例化`ID=“app”`的这个标签。


<br/>



## Vue 生命周期
----
每个 Vue 实例在被创建时都要经过一系列的初始化过程——例如，需要设置数据监听、编译模板、将实例挂载到 DOM 并在数据变化时更新 DOM 等。同时在这个过程中也会运行一些叫做生命周期钩子的函数，这给了用户在不同阶段添加自己的代码的机会。

比如`created`钩子可以用来在一个实例被创建之后执行代码：
	
	new Vue({
	  data: {
		a: 1
	  },
	  created: function () {
		// `this` 指向 vm 实例
		console.log('a is: ' + this.a)
	  }
	})
	// => "a is: 1"

**生命周期图示**

![size:800](/storage/2020/12-30/2RpFtecotBnaln6DIyc8gzRnTjMVSjNMfJ3ElIe1.jpeg)


<br/>



## 数据与方法
----

当一个 Vue 实例被创建时，它将`data`对象中的所有的 property 加入到 Vue 的**响应式系统**中。当这些 property 的值发生改变时，视图将会产生**“响应”**，即匹配更新为新的值。


	// 我们的数据对象
	var data = { a: 1 }
	
	// 该对象被加入到一个 Vue 实例中
	var vm = new Vue({
	  data: data
	})
	
	// 获得这个实例上的 property
	// 返回源数据中对应的字段
	vm.a == data.a // => true
	
	// 设置 property 也会影响到原始数据
	vm.a = 2
	data.a // => 2
	
	// ……反之亦然
	data.a = 3
	vm.a // => 3


<br/>



## 模板语法
----

### 插值

#### 文本

数据绑定最常见的形式就是使用(双大括号)文本插值：
	
	<span>Message: {{ msg }}</span>

#### HTML

双大括号会将数据解释为普通文本，而非 HTML 代码。为了输出真正的 HTML，你需要使用 `v-html`
	
	<p>Using v-html directive: <span v-html="rawHtml"></span></p>

这个`span`的内容将会被替换成为 property 值`rawHtml`，直接作为 HTML——会忽略解析 property 值中的数据绑定。注意，你不能使用`v-html`来复合局部模板，因为 Vue 不是基于字符串的模板引擎。


#### 使用 JavaScript 表达式

对于所有的数据绑定，Vue.js 都提供了完全的 JavaScript 表达式支持。
	
	{{ number + 1 }}
	
	{{ ok ? 'YES' : 'NO' }}
	
	{{ message.split('').reverse().join('') }}
	
	<div v-bind:id="'list-' + id"></div>

这些表达式会在所属 Vue 实例的数据作用域下作为 JavaScript 被解析。有个限制就是，每个绑定都只能包含**单个表达式**，所以下面的例子都**不会**生效。
	
	<!-- 这是语句，不是表达式 -->
	{{ var a = 1 }}
	
	<!-- 流控制也不会生效，请使用三元表达式 -->
	{{ if (ok) { return message } }}


### 指令

指令 (Directives) 是带有 `v-` 前缀的特殊 attribute。指令 attribute 的值预期是**单个 JavaScript 表达式** (`v-for` 是例外情况)。指令的职责是，当表达式的值改变时，将其产生的连带影响，响应式地作用于 DOM。

#### 参数

一些指令能够接收一个“参数”，在指令名称之后以冒号表示。例如，`v-bind` 指令可以用于响应式地更新 HTML attribute：
	
	<!-- 完整语法 -->
	<a v-bind:href="url">...</a>
	
	<!-- 缩写 -->
	<a :href="url">...</a>


在这里 `href` 是参数，告知 `v-bind` 指令将该元素的 `href` attribute 与表达式 `url` 的值绑定。

另一个例子是 `v-on` 指令，它用于监听 DOM 事件：
	
	<!-- 完整语法 -->
	<a v-on:click="doSomething">...</a>
	
	<!-- 缩写 -->
	<a @click="doSomething">...</a>

在这里参数是监听的事件名。

<br/>



#### 渲染

***条件渲染***

*`v-if`*

`v-if`指令用于条件性地渲染一块内容。这块内容只会在指令的表达式返回 truthy 值的时候被渲染。

`v-else`指令可以表示 v-if 的“else 块”。

`v-else-if`，充当 v-if 的“else-if 块”，可以连续使用。
	
	<div v-if="type === 'A'">
	  A
	</div>
	<div v-else-if="type === 'B'">
	  B
	</div>
	<div v-else-if="type === 'C'">
	  C
	</div>
	<div v-else>
	  Not A/B/C
	</div>

> 注：`v-else` 元素必须紧跟在带 `v-if` 或者 `v-else-if` 的元素的后面，否则它将不会被识别。   
> `v-else-if` 也必须紧跟在带 `v-if` 或者 `v-else-if` 的元素之后。


**在`<template>`元素上使用 `v-if` 条件渲染分组**

因为 `v-if` 是一个指令，所以必须将它添加到一个元素上。但是如果想切换多个元素呢？此时可以把一个 `<template>` 元素当做不可见的包裹元素，并在上面使用 `v-if`。最终的渲染结果将不包含 `<template>` 元素。
	
	<template v-if="ok">
	  <h1>Title</h1>
	  <p>Paragraph 1</p>
	  <p>Paragraph 2</p>
	</template>

**`v-show`**

另一个用于根据条件展示元素的选项是 v-show 指令。用法大致一样：
	
	<h1 v-show="ok">Hello!</h1>

不同的是带有 `v-show` 的元素始终会被渲染并保留在 DOM 中。`v-show` 只是简单地切换元素的 CSS property `display`。

> 注：`v-show` 不支持 `<template>` 元素，也不支持 `v-else`。


***列表渲染***

我们可以用 `v-for` 指令基于一个数组来渲染一个列表。`v-for` 指令需要使用 `item in list` 形式的特殊语法，其中 `list` 是源数据数组，而 `item` 则是被迭代的数组元素的别名。
	
	<ul id="example-1">
	  <li v-for="(item, index) in list" :key="item.message">
	  // <li v-for="item in list" :key="item.message">
		{{ parentMessage }} - {{ index }} - {{ item.message }}
	  </li>
	</ul>
<br/>
	
	var example1 = new Vue({
	  el: '#example-1',
	  data: {
		parentMessage: 'Parent',
		list: [
		  { message: 'Foo' },
		  { message: 'Bar' }
		]
	  }
	})

结果：

![size:800,1000](/storage/2020/12-30/jy0HCVqfSgeGrvJ4pFdZE5FlVZkDih7t4cICE6tg.png)



<br/>

#### 计算属性

对于任何复杂逻辑，你都应当使用计算属性。


	<div id="example">
	  <p>Original message: "{{ message }}"</p>
	  <p>Computed reversed message: "{{ reversedMessage }}"</p>
	</div>

<br/>

	var vm = new Vue({
	  el: '#example',
	  data: {
		message: 'Hello'
	  },
	  computed: {
		// 计算属性的 getter
		reversedMessage: function () {
		  // `this` 指向 vm 实例
		  return this.message.split('').reverse().join('')
		}
	  }
	})

结果：

![size:800,1000](/storage/2020/12-30/l28eAAr3uUVN7tlIEyUtGFY9nzI335fYMmu9t0d2.png)

> 注：我们可以将同一函数定义为一个方法而不是一个计算属性。两种方式的最终结果确实是完全相同的。然而，不同的是**计算属性是基于它们的响应式依赖进行缓存的**。只在相关响应式依赖发生改变时它们才会重新求值。

<br/>


## 事件处理
----

### 事件处理方法

示例：
	
	<div id="example-2">
	  <!-- `greet` 是在下面定义的方法名 -->
	  <button v-on:click="greet('hi', $event)">Greet</button>
	</div>
<br/>
	
	var example2 = new Vue({
	  el: '#example-2',
	  data: {
		name: 'Vue.js'
	  },
	  // 在 `methods` 对象中定义方法
	  methods: {
		greet: function (str, event) {
		  // `this` 在方法里指向当前 Vue 实例
		  alert( str + ' Hello ' + this.name + '!')
		  // `event` 是原生 DOM 事件
		  if (event) {
			alert(event.target.tagName)
		  }
		}
	  }
	})
	
	// 也可以用 JavaScript 直接调用方法
	example2.greet() // => 'hi Hello Vue.js!'

<br/>


### 事件修饰符

 * `.stop`
 * `.prevent`
 * `.capture`
 * `.self`
 * `.once`
 * `.passive`
 * `.once`

	
	<!-- 阻止单击事件继续传播 -->
	<a v-on:click.stop="doThis"></a>

	<!-- 提交事件不再重载页面 -->
	<form v-on:submit.prevent="onSubmit"></form>

	<!-- 修饰符可以串联 -->
	<a v-on:click.stop.prevent="doThat"></a>

	<!-- 只有修饰符 -->
	<form v-on:submit.prevent></form>

	<!-- 添加事件监听器时使用事件捕获模式 -->
	<!-- 即内部元素触发的事件先在此处理，然后才交由内部元素进行处理 -->
	<div v-on:click.capture="doThis">...</div>

	<!-- 只当在 event.target 是当前元素自身时触发处理函数 -->
	<!-- 即事件不是从内部元素触发的 -->
	<div v-on:click.self="doThat">...</div>

	<!-- 点击事件将只会触发一次 -->
	<a v-on:click.once="doThis"></a>


<br/>



## 表单输入绑定
----

### 基本用法

可以用 `v-model` 指令在表单 `<input>`、`<textarea>` 及 `<select>` 元素上创建双向数据绑定。它会根据控件类型自动选取正确的方法来更新元素。

> 注：`v-model` 会忽略所有表单元素的 `value`、`checked`、`selected` attribute 的初始值而总是将 Vue 实例的数据作为数据来源。你应该通过 JavaScript 在组件的 `data` 选项中声明初始值。


#### 文本

	<input v-model="message" placeholder="edit me">
	<p>Message is: {{ message }}</p>

#### 多行文本

	<textarea v-model="message" placeholder="add multiple lines"></textarea>


#### 复选框
单个复选框，绑定到布尔值：
	
	<input type="checkbox" id="checkbox" v-model="checked">
	<label for="checkbox">{{ checked }}</label>

多个复选框，绑定到同一个数组：
	
	<input type="checkbox" id="jack" value="Jack" v-model="checkedNames">
	<label for="jack">Jack</label>
	<input type="checkbox" id="john" value="John" v-model="checkedNames">
	<label for="john">John</label>
	<input type="checkbox" id="mike" value="Mike" v-model="checkedNames">
	<label for="mike">Mike</label>
	<br>
	<span>Checked names: {{ checkedNames }}</span>
<br/>
	
	new Vue({
	  el: '...',
	  data: {
		checkedNames: []
	  }
	})
结果：

![size:800,1000](/storage/2020/12-30/FM6BHPVAgtbakqKFz2efVMoQqt7RgPuPbRod47kx.png)

<br/>

**指定值：**
	
	<input
	  type="checkbox"
	  v-model="toggle"
	  true-value="yes"
	  false-value="no"
	>
	
	// 当选中时
	vm.toggle === 'yes'
	// 当没有选中时
	vm.toggle === 'no'

> 这里的 `true-value` 和 `false-value` attribute 并不会影响输入控件的 `value` attribute，因为浏览器在提交表单时并不会包含未被选中的复选框。如果要确保表单中这两个值中的一个能够被提交，(即“yes”或“no”)，请换用单选按钮。

<br/>


#### 单选按钮


	<div id="example-4">
	  <input type="radio" id="one" value="One" v-model="picked">
	  <label for="one">One</label>
	  <br>
	  <input type="radio" id="two" value="Two" v-model="picked">
	  <label for="two">Two</label>
	  <br>
	  <span>Picked: {{ picked }}</span>
	</div>

结果：

![size:800,1000](/storage/2020/12-30/QPfe17mPWsIReSIdVhgBJYzqcs8NElL0bKGBvu0x.png)

<br/>
	
	<input type="radio" v-model="pick" v-bind:value="a">
	
	// 当选中时
	vm.pick === vm.a

<br/>

#### 选择框

时：
	
	<div id="example-5">
	  <select multiple="multiple" v-model="selected" >
		<option disabled value="">请选择</option>
		<option>A</option>
		<option>B</option>
		<option>C</option>
	  </select>
	  <span>Selected: {{ selected }}</span>
	</div>

单选结果：

![size:800,1000](/storage/2020/12-30/DxgcdK0lIbwoG29Y7f6hOy6tJ0vCdnksNUA5NwBO.png)


<br/>


多选结果：

![size:800,1000](/storage/2020/12-31/3tG6mGkaHOrxBtIwZO9D6wl4B0rK9VQNgrNMYWM4.png)

<br/>


	<select v-model="selected">
		<!-- 内联对象字面量 -->
	  <option v-bind:value="{ number: 123 }">123</option>
	</select>
	
	// 当选中时
	typeof vm.selected // => 'object'
	vm.selected.number // => 123

<br/>

### 修饰符

**`.lazy`**

在默认情况下，`v-model` 在每次 `input` 事件触发后将输入框的值与数据进行同步 (除了上述输入法组合文字时)。你可以添加 `lazy` 修饰符，从而转为在 `change` 事件_之后_进行同步：
	
	<!-- 在“change”时而非“input”时更新 -->
	<input v-model.lazy="msg">

**`.number`**

如果想自动将用户的输入值转为数值类型，可以给 v-model 添加 number 修饰符：
	
	<input v-model.number="age" type="number">

这通常很有用，因为即使在 type="number" 时，HTML 输入元素的值也总会返回字符串。如果这个值无法被 parseFloat() 解析，则会返回原始的值。

**`.trim`**

如果要自动过滤用户输入的首尾空白字符，可以给 v-model 添加 trim 修饰符：
	
	<input v-model.trim="msg">


<br/>

## 其它

 * 强制渲染

	
	this.$forceUpdate();

<br/>

## 总结
----

 * 用 `new Vue({})`新建vue实例
 * 使用 `v-bind:`和`{{}}`双大括号语法在html中绑定变量
 * 使用 `v-on:` 语法绑定函数到标签的事件
 * 使用 `v-model:` 语法使用户的页面输入反向传递回vue实例变量

<br/>