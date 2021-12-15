h5中给我们友好的API video、audio。其中，在我们创建这个多媒体节点的时候，我们可以设置它的属性，做出自己想要的控制样式等

1. 创建

```html
<video src="movie.ogg" controls="controls">
您的浏览器不支持 video 标签。
</video>
```

2. 以下列出原生video常用的一些属性

| 属性                                                         |    值    |                                                         描述 |
| ------------------------------------------------------------ | :------: | -----------------------------------------------------------: |
| [autoplay](http://www.w3school.com.cn/tags/att_video_autoplay.asp) | autoplay |                     如果出现该属性，则视频在就绪后马上播放。 |
| [controls](http://www.w3school.com.cn/tags/att_video_controls.asp) | controls |             如果出现该属性，则向用户显示控件，比如播放按钮。 |
| [loop](http://www.w3school.com.cn/tags/att_video_loop.asp)   |   loop   |           如果出现该属性，则当媒介文件完成播放后再次开始播放 |
| [poster](http://www.w3school.com.cn/tags/att_video_poster.asp) |   url    | 规定视频下载时显示的图像，或者在用户点击播放按钮前显示的图像。 |
| [src](http://www.w3school.com.cn/tags/att_video_src.asp)     |   url    |                                             要播放视频的地址 |

3. 支持全局属性、事件属性

以下列举一些常用的全局属性：

| 目标             |          值          |
| ---------------- | :------------------: |
| 获取视频时长     |  videoDOM.duration   |
| 获取当前播放时间 | videoDOM.currentTime |
| 设置当前播放位置 | videoDOM.currentTime |

以下列举一些常用的事件属性：
 在我们不需要[controls](http://www.w3school.com.cn/tags/att_video_controls.asp)属性的时候，往往需要手动js去控制它的播放情况，这是就需要用到事件属性

| 属性             |   值   | 描述                                   |
| ---------------- | :----: | :------------------------------------- |
| onplay           | script | 当媒介已就绪可以开始播放时运行的脚本。 |
| onpause          | script | 当媒介被用户或程序暂停时运行的脚本。   |
| ondurationchange | script | 当媒介长度改变时运行的脚本。           |

例(ps:环境vue)：



```jsx
<video :src="url" ref="video"></video>
<button @click="play">播放</button>
let videoDOM = this.$refs.video
console.log(videoDOM.duration) // 获取视频时长
video.currentTime = 0 // 设置当前播放位置
// [play 播放视频]
play () {
  this.videoDOM.play()
}
```
