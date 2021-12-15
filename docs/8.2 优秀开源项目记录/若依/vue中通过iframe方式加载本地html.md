# vue中通过iframe方式加载本地html

实现思路：

1、在src平级的文件目录下新建public文件夹

2、在public文件夹下编码html代码，包括相应的本地资源，js、css

3、在vue组件中通过ifame加载html文件

Home.vue

```html
<template>
    <!-- 所有的内容要被根节点包含起来 -->
    <div id="home">    
       我是首页  
        <el-button type="success">成功按钮</el-button>
        <el-button type="warning">警告按钮</el-button>
        <el-button type="danger">危险按钮</el-button>
        <br>
        <i style="font-size:40px;" class="el-icon-menu"></i>
         <el-rate v-model="value1"></el-rate>
      <iframe src="../../public/html5-webgl-galaxy/index.html" scrolling="no" style="width: 100%;height: 500px;" frameborder="0"></iframe>
        <iframe src="https://www.baidu.com/" width="300" height="300" frameborder="0" scrolling="auto"></iframe>
		<iframe src="../../../static/bear.html" width="300" height="300" frameborder="0" scrolling="auto"></iframe>
    </div>
</template>
```

- scrolling="no"表示不显示滚动条
- style="position:absolute;top:80px;left: 120px;" 表示嵌入的iframe位置距离浏览器顶部80px，距离浏览器左侧120px，刚好是我的侧边栏和顶部导航栏的宽度
- mounted() 中的方法在模板渲染成html后调用，通常是初始化页面完成后，再对html的dom节点进行一些需要的操作。包括调整页面的高和宽
- changeMobsfIframe 为自适应宽高的方法，分别在第一次页面加载和调整浏览器大小（onresize ()）时被调用



<font color='red'>**和src同级的public文件夹下面的html的访问地址一般是直接访问的，地址：localhost/xx.html；例如vue页面路由名称是system,访问地址localhost/system/index.vue。要想在index.vue页面嵌套html，src就需要改为/xx.html，意思是跳到上一级直接略过system的路径。**</font>



# 参考：

- [如何在 vue 跳转html 页面_[Vue.js]Vue3.0 多页面配置以及实现页面跳转](https://blog.csdn.net/weixin_40007515/article/details/110906084)
- [[Vue]使用iframe嵌入页面](https://blog.csdn.net/weixin_43438052/article/details/115283900)
- [vue中通过iframe方式加载本地html](https://blog.csdn.net/kerryqpw/article/details/89814922)

