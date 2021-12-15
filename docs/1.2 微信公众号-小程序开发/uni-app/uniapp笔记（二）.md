uniapp小白入门自学笔记（二）：https://www.cnblogs.com/echoyya/p/14429616.html



### uni的生命周期

#### [应用生命周期](https://uniapp.dcloud.io/collocation/frame/lifecycle?id=应用生命周期)

- 生命周期的概念：一个对象从创建，运行，销毁的整个过程被称为生命周期
- 生命周期函数：在生命周期中每个阶段会触发一个函数，这些函数被称为生命周期函数
- 应用生命周期仅可在`App.vue`中监听，在其它页面监听无效。

`uni-app` 支持如下应用生命周期函数：

| 函数名   | 说明                                           |
| :------- | :--------------------------------------------- |
| onLaunch | 当`uni-app` 初始化完成时触发（全局只触发一次） |
| onShow   | 当 `uni-app` 启动，或从后台进入前台显示        |
| onHide   | 当 `uni-app` 从前台进入后台                    |
| onError  | 当 `uni-app` 报错时触发                        |

```html
<script>
    // 只能在App.vue里监听应用的生命周期
    export default {
        onLaunch: function() {
            console.log('App Launch')
        },
        onShow: function() {
            console.log('App Show')
        },
        onHide: function() {
            console.log('App Hide')
          },
        onError: function(err) {
            console.log(err)
        }
    }
</script>
```

#### [页面生命周期](https://uniapp.dcloud.io/collocation/frame/lifecycle?id=页面生命周期)

`uni-app` 常用的页面生命周期函数：

| 函数名   | 说明                                                         |
| :------- | :----------------------------------------------------------- |
| onLoad   | 监听页面加载，其参数为上个页面传递的数据，参数类型为 Object（用于页面传参） |
| onShow   | 监听页面显示。页面每次出现在屏幕上都触发，包括从下级页面点返回露出当前页面 |
| onReady  | 监听页面初次渲染完成。注意如果渲染速度快，会在页面进入动画完成前触发 |
| onHide   | 监听页面隐藏                                                 |
| onUnload | 监听页面卸载                                                 |

```
<script>
    export default {
        onLoad: function(option) {
            console.log('页面加载',option)
        },
        onShow: function() {
            console.log('页面显示')
        },
        onReady: function() {
            console.log('页面初次渲染')
          },
        onHide: function() {
            console.log('页面隐藏')
        }
    }
</script>
```

### 导航跳转和传参

#### 声明式导航：[navigator](https://uniapp.dcloud.io/component/navigator?id=navigator)

该组件类似HTML中的`<a>`组件，但只能跳转本地页面。目标页面必须在pages.json中注册。

常用属性：

| 属性名    | 说明                                                         |
| :-------- | :----------------------------------------------------------- |
| url       | 应用内的跳转链接，值为相对或绝对路径，不需要加 `.vue` 后缀   |
| open-type | 跳转方式  默认值：navigate，而跳转tabbar页面时，需设置为switchTab |

```html
<template>
  <view class="">
    <!-- 跳转到普通页面 并传参 -->
    <navigator url="/pages/detail/detail?id=80">跳转详情</navigator>
		
    <!-- 跳转到tabbar页面 -->
    <navigator url="/pages/us/us" open-type="switchTab">关于我们</navigator>
  </view>
</template>
```

#### 编程式导航：

##### [uni.navigateTo(obj)](https://uniapp.dcloud.io/api/router?id=navigateto)

保留当前页面，跳转到应用内的某个页面，使用`uni.navigateBack`可以返回到原页面。

**obj必传参数说明**：`url`,需要跳转的应用内`非 tabBar `的页面的相对或绝对路径，路径后可以带参数，下一个页面的`onLoad函数`可得到传递的参数

**跳转到 tabBar 页面只能使用 switchTab 跳转**

```
//在跳转到detail.vue页面并传递参数
uni.navigateTo({
    url: '../detail/detail?id=1&name=Echoyya'
});
```

##### [uni.redirectTo(obj)](https://uniapp.dcloud.io/api/router?id=redirectto)

关闭当前页面，跳转到应用内的某个页面。

**obj必传参数说明**：`url`,需要跳转的应用内`非 tabBar `的页面的相对或绝对路径，路径后可以带参数，下一个页面的`onLoad函数`可得到传递的参数

```
uni.redirectTo({
    url: '../detail/detail?id=1'
});
```

##### [uni.switchTab(obj)](https://uniapp.dcloud.io/api/router?id=switchtab)

**跳转到 tabBar 页面，并关闭其他所有非 tabBar 页面。**

但 如果调用了 [uni.preloadPage(OBJECT)](https://uniapp.dcloud.net.cn/api/preload-page) 不会关闭，仅触发生命周期 `onHide`

**obj必传参数说明**：`url`,需要跳转的`tabBar`的页面的相对或绝对路径，（需在 pages.json 的 tabBar 字段定义的页面），路径后不能带参数

**pages.json**

```
"tabBar":{
  "color":"#8a8a8a",
  "selectedColor":"#d81e06",
  "list":[
    {
      "text":"首页",
      "pagePath": "pages/index/index",
      "iconPath":"static/tabs/home.png",
      "selectedIconPath":"static/tabs/home-active.png"
    },
    {
      "text":"我们",
      "pagePath": "pages/us/us",
      "iconPath":"static/tabs/us.png",
      "selectedIconPath":"static/tabs/us-active.png"
    }
  ]
}
```

**index.vue**

```
uni.switchTab({
    url: '/pages/us/us'
});
```

#### 获取参数

导航跳转传参，在目标页面`onLoad`生命周期中，可以接受一个参数options，即为传递的参数

**detail.vue**

```
onLoad(options) {
   console.log(options.id)  // 80
}
```

### 创建组件，组件通讯

首先在uni-app中，创建组件的方法与Vue中一模一样，**新建组件 -> import引入 -> components中声明**，相信大家对Vue都比较熟悉，此处不再赘述，主要简述一下组件间的通讯，与Vue中略有不同，

- 父向子，子向父 同Vue，可参考我之前总结过的一篇博文有非常详细的描述及案例[Vue2.0 多种组件传值方法-不过如此的 Vuex](https://www.cnblogs.com/echoyya/p/14404397.html)

- 兄弟组件传值，与Vue略有不同

  [uni.$emit(eventName,obj)](https://uniapp.dcloud.io/api/window/communication?id=emit)：触发全局的自定义事件，附加参数都会传给监听器回调函数。

  | 属性      | 类型   | 描述                   |
  | --------- | ------ | ---------------------- |
  | eventName | String | 事件名                 |
  | OBJECT    | Object | 触发事件携带的附加参数 |

  [uni.$on(eventName,callback)](https://uniapp.dcloud.io/api/window/communication?id=on)：监听全局的自定义事件，事件由 `uni.$emit` 触发，回调函数会接收事件触发函数的传入参数。

  | 属性      | 类型     | 描述           |
  | --------- | -------- | -------------- |
  | eventName | String   | 事件名         |
  | callback  | Function | 事件的回调函数 |

![img](https://img2020.cnblogs.com/blog/1238759/202102/1238759-20210222131130734-1969665020.gif)

![img](https://img2020.cnblogs.com/blog/1238759/202102/1238759-20210222131111850-1213010047.png)

### 下拉刷新 onPullDownRefresh

#### 开启下拉刷新的两种方式：

1. 需要在 `pages.json` 里，找到的当前页面的pages节点，并在 `style` 选项中开启 `enablePullDownRefresh`。

```
{
    "pages": [
        {
            "path": "pages/index/index",
            "style": {
                "enablePullDownRefresh": true
            }
        }
    ]
}
```

1. `uni.startPullDownRefresh(obj)`方法，触发该方法从而实现下拉刷新，效果与用户手动下拉刷新一致。参数可接受回调函数

```javascript
// 仅做示例，实际开发中延时根据需求来使用。
export default {
    onLoad: function (options) {
        uni.startPullDownRefresh();
    },
}
```

#### 监听下拉刷新

在 JS 中定义 `onPullDownRefresh` 处理函数（和onLoad等生命周期函数同级），监听该页面用户下拉刷新事件。

```
export default {
    onPullDownRefresh() {
        console.log('监听下拉刷新');
    }
}
```

#### 关闭下拉刷新

处理完数据刷新后，`uni.stopPullDownRefresh` 可以停止当前页面的下拉刷新。

```javascript
export default {
    onPullDownRefresh() {
        console.log('监听下拉刷新');
        setTimeout(()=>{
            uni.stopPullDownRefresh()
        }, 1000)
    }
}
```

### 上拉加载 (页面触底加载)

#### 监听页面触底

1. 在 JS 中定义 `onReachBottom`处理函数（和onLoad等生命周期函数同级），监听该页面上拉触底事件。常用于触底加载下一页数据

```html
   <template>
     <view>
   	  <view class="box3" v-for="(item,index) in listNum" :key="index">
   	    {{item}}
   	  </view>
     </view>
   </template>
   
   <script>
   	export default {
   	  data() {
   	    return {
   	      listNum:10
   	    }
   	  },
   	  onReachBottom(){
            console.log('页面触底');
            this.listNum +=5
          }
   	}
   </script>
   
   <style>
   .box{
   	height: 100px;
   	line-height: 100px;
   	text-align: center;
   }
   </style>
```

1. 需要在 `pages.json` 里，找到的当前页面的pages节点，并在 `style` 选项中开启 `onReachBottomDistance`。页面上拉触底事件触发时距页面底部距离（Number类型）
   - 也可设置`globalStyle`下的触底距离，若同时设置，page节点下的style会覆盖全局配置

```json
{
  "pages": [
    {
      "path": "pages/index/index",
      "style": {
          "onReachBottomDistance": 200
        }
    }
  ],
  "globalStyle": { // 全局配置
     "navigationBarTextStyle": "white",
     "navigationBarTitleText": "hello-uni-app",
     "navigationBarBackgroundColor": "#8470FF",
     "backgroundColor": "#8DEEEE",
   
     "onReachBottomDistance": 200
   }
}
```

### 数据缓存 (操作storage)

#### 同步（推荐）

- `uni.setStorageSync(key,data)`
- `uni.getStorageSync(key)`
- `uni.removeStorageSync(key)`

```javascript
<template>
  <view class="">
    <button type="primary" @click="setStorageSync">同步存储数据</button>
    <button type="primary" @click="getStorageSync">同步获取数据</button>
    <button type="primary" @click="removeStorageSync">同步移除数据</button>
  </view>
</template>

<script>
  export default {
    methods: {
      setStorageSync() {
        uni.setStorageSync('name', 'Echoyya');
      },
      getStorageSync() {
        console.log(uni.getStorageSync('name')) 	
      },
      removeStorageSync() {
        uni.removeStorageSync('name')
      }
    }
  }
</script>
```

#### 异步

- `uni.setStorage(obj)`：包括存储的key，data，以及成功失败的回调函数
- `uni.getStorage(obj)`：包括存储的key，以及成功失败的回调函数
- `uni.removeStorage(obj)`：包括存储的key，以及成功失败的回调函数

```javascript
<template>
  <view class="">
    <button type="warn" @click="setStorage">异步存储数据</button>
    <button type="warn" @click="getStorage">异步获取数据</button>
    <button type="warn" @click="removeStorage">异步移除数据</button>
  </view>
</template>

<script>
  export default {
    methods: {
      setStorage() {
        uni.setStorage({
          key: 'name',
          data: 'Echoyya',
          success: function() {
            console.log('存储成功');
          }
        })
      },
      getStorage() {
        uni.getStorage({
          key: 'name',
          success: function(res) {
            console.log(res.data);
          }
        })
      },
      removeStorage() {
        uni.removeStorage({
          key: 'name',
          success: function(res) {
            console.log('移除成功');
          }
        })
      },
    }
  }
</script>
```

### 上传、预览图片

图片操作，常用到以下两个方法：

- `uni.chooseImage(obj)`：上传图片，从本地相册选择图片或使用相机拍照。

| 参数名  | 类型     | 说明                                                     |
| :------ | :------- | :------------------------------------------------------- |
| count   | Number   | 最多可以选择的图片张数，默认9                            |
| success | Function | 必填项，成功则返回图片的本地文件路径列表` tempFilePaths` |

- `uni.previewImage(obj)`：预览图片

| 参数名  | 类型          | 说明                                                         |
| :------ | :------------ | :----------------------------------------------------------- |
| current | String/Number | 为当前显示图片的链接/索引值，不填或填写的值无效则为 urls 的第一张。有些app版本为必填 |
| urls    | Array         | 必填项，需要预览的图片链接列表                               |

```javascript
<template>
  <view class="">
    <button type="default" @click="chooseImage">图片上传</button>
    <!-- 点击图片预览 -->
    <image v-for="(item,index) in imgArr" :key="index" :src="item" @click="previewImage(item)"></image>
  </view>
</template>

<script>
  export default {
    data() {
      return {
        imgArr: []
      }
    },
    methods: {
      chooseImage() {
        uni.chooseImage({
          count: 5,
          success: (res) => {
            this.imgArr = res.tempFilePaths
          }
        })
      },
      previewImage(current) {
        uni.previewImage({
          current,
          urls: this.imgArr
        })
      }
    }
  }
</script>
```