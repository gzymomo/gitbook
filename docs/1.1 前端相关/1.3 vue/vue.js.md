[TOC]

# MVVM风格开发框架

MVVM（Model View ViewModel）是由微软的WPF带来的技术体验，如Sliverlight、音频、视频、3D和动画等，使得软件UI层更加细节化、可定制化。



# 一、vue.js

## 1.1 基础代码与mvvm关系
```html
<body>
<!-- 将来new 的vue实例，会控制这个元素中的所有内容 -->
<!-- vue实例所控制的这个元素区域，就是我们的vue -->
<div id="app">
  <p>{{ msg}}  </p>
</div>

<script>
  // 创建一个vue的实例
  //当我们导入包之后，在浏览器的内存中，就多了一个vue构造函数
  new Vue({
   el: '#app', //表示，当前我们new的这个vue实例，要控制页面上的哪个区域
   //这个里data就是mvvm中的m，专门用来保存每个页面的数据的
   data: { //data 属性中，存放的是el中要用到的数据
     msg: 'hello world'
   }
  })
</script>

</body>
```

## v-for
![](https://www.showdoc.cc/server/api/attachment/visitfile/sign/7420a7636d52220792cf0ce1c865115a?showdoc=.jpg)

## 事件修饰符
![](https://www.showdoc.cc/server/api/attachment/visitfile/sign/2f4e5552d877b4236187e449261b9384?showdoc=.jpg)

## 按键修饰符
![](https://www.showdoc.cc/server/api/attachment/visitfile/sign/a9a601a531a54355bc4e590d5a247d99?showdoc=.jpg)

## 属性绑定样式
![](https://www.showdoc.cc/server/api/attachment/visitfile/sign/bafd1061d88d041862845ae3ac9e5844?showdoc=.jpg)

## 生命周期

- beforeCreate：在实例初始化之后，数据观测（data observer）和event/watcher事件配置之前被调用
- created：在实例创建完成后被立即调用，然后，挂在阶段还没开始，$el属性目前不可见、
  1. 数据观测（data observer）
  2. 属性和方法的运算
  3. watch/event事件回调
- beforeMount：在挂载开始之前被调用：相关的render函数首次被调用。（该钩子在服务器渲染期间不被调用）
- mounted：el被创建的vm.$el替换，挂载到实例上去之后调用该钩子。如果root实例挂载了一个文档元素，当mounted被调用时vm.$el也在文档内。mounted不会承诺所有子组件都一起被挂载。如果你希望等到整个视图渲染完毕，可以用vm.$nextTick替换掉mounted。
- beforeUpdate：数据更新时调用，发生在虚拟dom打补丁之前。这里适合在更新之前访问现有的dom，比如移除已添加的时间监听器。该钩子在服务器渲染期间不能被调用，因为只有初次渲染会在服务端进行。
- updated： 数据更改导致的虚拟dom重新渲染和打补丁，在这之后会被调用该钩子。当这个钩子被调用时，组件Dom以及更新，所以你现在可以执行依赖于Dom的操作。然而在大多数情况下，你应该变在此期间更改状态。如果要相应状态改变，通常最好使用计算属性或watcer取而代之。updated不会承诺所有的子组件也都一起被重绘。如果你希望等到整个视图都重绘完毕，可以用vm.$nextTick替换掉updated。
- bedoreDestory： 实例销毁之前调用。在这一步，实例任然完全可用。（服务器端渲染期间不被调用）
- destryed： 实例销毁后调用，调用后，vue实例指示的所有东西都会解除绑定，所有的时间监听器会被移除，所有的子实例也会被销毁。（服务器端渲染期间不被调用）
- activated： keep-alive组件激活时调用。（服务器端渲染期间不被调用）
- deacttivated： keep-alive组件停用时调用。（服务端渲染期间不被调用）
- errorCaptured： 类型：{er:Error, vm:Component,info:String}=>?boolean，当捕获一个来自子孙组件的错误时被调用。此钩子会收到三个参数：错误对象，发生错误的组件实例以及一个包含错误来源信息的字符串。此钩子可以返回false以阻止错误向上传播。

![](https://www.showdoc.cc/server/api/attachment/visitfile/sign/551c5fad7d3f8571c05c37a1224be9c4?showdoc=.jpg)

![](https://www.showdoc.cc/server/api/attachment/visitfile/sign/48af15331d53a4fb2f4787035071b97b?showdoc=.jpg)

## 全局指令
![](https://www.showdoc.cc/server/api/attachment/visitfile/sign/c339be387ce1cdbe4e6face61ac93ae9?showdoc=.jpg)

## 创建全局组件

### 方式一
![](https://www.showdoc.cc/server/api/attachment/visitfile/sign/6f37945aef56a27c634d2f0a1e8c5887?showdoc=.jpg)

### 方式二
![](https://www.showdoc.cc/server/api/attachment/visitfile/sign/9d0c73faf4118c7d74257b9610aa8fc4?showdoc=.jpg)

### 方式三
![](https://www.showdoc.cc/server/api/attachment/visitfile/sign/5ce82329eb1abb1dd36d038ce29667fe?showdoc=.jpg)

## 创建私有组件
![](https://www.showdoc.cc/server/api/attachment/visitfile/sign/108751c16ce93864ef033e15cac52911?showdoc=.jpg)

## 组件切换
### 方式一
![](https://www.showdoc.cc/server/api/attachment/visitfile/sign/532d8ca5ac0106ebdb6429ec918643f4?showdoc=.jpg)

### 方式二
![](https://www.showdoc.cc/server/api/attachment/visitfile/sign/742121d72fb9142b9bd347ed747a8fdd?showdoc=.jpg)

## 父组件向子组件传值
![](https://www.showdoc.cc/server/api/attachment/visitfile/sign/bb3a04de4727e2c546175b90f7d30339?showdoc=.jpg)

![](https://www.showdoc.cc/server/api/attachment/visitfile/sign/5541ec061022c30cce086316bbdf6620?showdoc=.jpg)

## 路由
![](https://www.showdoc.cc/server/api/attachment/visitfile/sign/8cb339115fbdbde25607685e58b8dc4f?showdoc=.jpg)

## ref获取dom或组件引用
![](https://www.showdoc.cc/server/api/attachment/visitfile/sign/f653073410a29cad7037446970fe3af8?showdoc=.jpg)

## 全局API Vue.*