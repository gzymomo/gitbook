- [三种前端实现VR全景看房的方案！](https://juejin.cn/post/6973865268426571784)



# 前言

事情是这样的，前几天我接到一个`外包工头`的新需求，某品牌要搭建一个在线VR展厅，用户可以在手机上通过陀螺仪或者拖动来360度全景参观展厅，这个VR展厅里会有一些信息点，点击之后可以呈现更多信息（视频，图文等）...

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6aaf2a7187bc4b5eaf8f0346a2be0ec8~tplv-k3u1fbpfcp-watermark.image)

我第一反应是用3D引擎，因为我不久前刚用`three.js`做过一个`BMW`的在线展厅，基本把`three.js`摸熟了。

![2021-06-03 11_01_41.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7386325890664c4e9cd36315067feef9~tplv-k3u1fbpfcp-watermark.image)

> 会另写一篇文章教大家用threejs做这个[BMW在线DIY]，感兴趣的小伙伴请关注我吧~

# 方案一：WebGL3D引擎

使用3D引擎先搭一个基本的3D场景，下面的演示使用[three.js](https://github.com/mrdoob/three.js/)，同类的3D引擎我还调研过[babylon.js](https://github.com/BabylonJS/Babylon.js)，[playcanvas](https://playcanvas.com/)，使用都差不太多，学会一个基本都通的

```javascript
var scene, camera, renderer;

function initThree(){
    //场景
    scene = new THREE.Scene();
    //镜头
    camera = new THREE.PerspectiveCamera(90, document.body.clientWidth / document.body.clientHeight, 0.1, 100);
    camera.position.set(0, 0, 0.01);
    //渲染器
    renderer = new THREE.WebGLRenderer();
    renderer.setSize(document.body.clientWidth, document.body.clientHeight);
    document.getElementById("container").appendChild(renderer.domElement);
    //镜头控制器
    var controls = new THREE.OrbitControls(camera, renderer.domElement);
    
    //一会儿在这里添加3D物体

    loop();
}

//帧同步重绘
function loop() {
    requestAnimationFrame(loop);
    renderer.render(scene, camera);
}

window.onload = initThree;
复制代码
```

现在我们能看到一个黑乎乎的世界，因为现在`scene`里什么都没有，接着我们要把三维物体放进去了，使用3D引擎的实现方式无非都是以下几种

## 使用立方体（box）实现

> 这种方式最容易理解，我们在一个房间里，看向天花板，地面，正面，左右两面，背面共计六面。我们把所有六个视角拍成照片就得到下面六张图

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/de9a7837404741528d39b13f516f58a4~tplv-k3u1fbpfcp-watermark.image)

现在我们直接使用立方体（box）搭出这样一个房间

```javascript
var materials = [];
//根据左右上下前后的顺序构建六个面的材质集
var texture_left = new THREE.TextureLoader().load( './images/scene_left.jpeg' );
materials.push( new THREE.MeshBasicMaterial( { map: texture_left} ) );

var texture_right = new THREE.TextureLoader().load( './images/scene_right.jpeg' );
materials.push( new THREE.MeshBasicMaterial( { map: texture_right} ) );

var texture_top = new THREE.TextureLoader().load( './images/scene_top.jpeg' );
materials.push( new THREE.MeshBasicMaterial( { map: texture_top} ) );

var texture_bottom = new THREE.TextureLoader().load( './images/scene_bottom.jpeg' );
materials.push( new THREE.MeshBasicMaterial( { map: texture_bottom} ) );

var texture_front = new THREE.TextureLoader().load( './images/scene_front.jpeg' );
materials.push( new THREE.MeshBasicMaterial( { map: texture_front} ) );

var texture_back = new THREE.TextureLoader().load( './images/scene_back.jpeg' );
materials.push( new THREE.MeshBasicMaterial( { map: texture_back} ) );

var box = new THREE.Mesh( new THREE.BoxGeometry( 1, 1, 1 ), materials );
scene.add(box);
复制代码
```

![2021-06-14 19_51_17.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4f10e5ddca004406a63e77d9504c3dc4~tplv-k3u1fbpfcp-watermark.image)

好，现在我们把镜头camera（也就是人的视角），放到box内，并且让所有贴图向内翻转后，VR全景就实现了。

```javascript
box.geometry.scale( 1, 1, -1 );
复制代码
```

**现在我们进入了这个盒子！！**

![2021-06-14 19_41_37.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/28cd30c5469e43379e3a41ab4edae83d~tplv-k3u1fbpfcp-watermark.image)

[threejs官方立方体全景示例](https://github.com/mrdoob/three.js/blob/master/examples/webgl_panorama_cube.html)

## 使用球体（sphere）实现

> 我们将房间360度球形范围内所有的光捕捉到一个图片上，再将这张图片展开为矩形，就能得到这样一张全景图片

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/27becc372074417a9ce02ca06f579eb2~tplv-k3u1fbpfcp-watermark.image)

```javascript
var sphereGeometry = new THREE.SphereGeometry(/*半径*/1, /*垂直节点数量*/50, /*水平节点数量*/50);//节点数量越大，需要计算的三角形就越多，影响性能

var sphere = new THREE.Mesh(sphereGeometry);
sphere.material.wireframe  = true;//用线框模式大家可以看得清楚是个球体而不是圆形
scene.add(sphere);
复制代码
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/20e437117a654ba2b51ce1d47696de5b~tplv-k3u1fbpfcp-watermark.image)

现在我们把这个全景图片贴到这个球体上

```javascript
var texture = new THREE.TextureLoader().load('./images/scene.jpeg');
var sphereMaterial = new THREE.MeshBasicMaterial({map: texture});

var sphere = new THREE.Mesh(sphereGeometry,sphereMaterial);
// sphere.material.wireframe  = true;
复制代码
```

![2021-06-14 14_54_38.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5083e0cc881549d883d788c97fddf3d8~tplv-k3u1fbpfcp-watermark.image)

和之前一样，我们把镜头camera（也就是人的视角），放到球体内，并且让所有贴图向内翻转后，VR全景就实现了

**现在我们进入了这个球体！！**

```javascript
var sphereGeometry = new THREE.SphereGeometry(/*半径*/1, 50, 50);
sphereGeometry.scale(1, 1, -1);
复制代码
```

![2021-06-14 15_15_28.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1875fc36e0ef40a9a0c4c12fb9512ce5~tplv-k3u1fbpfcp-watermark.image)

[threejs官方球体全景示例](https://github.com/mrdoob/three.js/blob/master/examples/webgl_panorama_equirectangular.html)

## 添加信息点

> 在VR全景中，我们需要放置一些信息点，用户点击之后做一些动作。

现在我们建立这样一个点的数组

```javascript
var hotPoints=[
    {
        position:{
            x:0,
            y:0,
            z:-0.2
        },
        detail:{
            "title":"信息点1"
        }
    },
    {
        position:{
            x:-0.2,
            y:-0.05,
            z:0.2
        },
        detail:{
            "title":"信息点2"
        }
    }
];
复制代码
```

遍历这个数组，并将信息点的指示图添加到3D场景中

```javascript
var pointTexture = new THREE.TextureLoader().load('images/hot.png');
var material = new THREE.SpriteMaterial( { map: pointTexture} );

for(var i=0;i<hotPoints.length;i++){
    var sprite = new THREE.Sprite( material );
    sprite.scale.set( 0.1, 0.1, 0.1 );
    sprite.position.set( hotPoints[i].position.x, hotPoints[i].position.y, hotPoints[i].position.z );

   scene.add( sprite );
}
复制代码
```

**看到HOT指示图了吗？**

![2021-06-14 20_22_12.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b83bc4c6ef9247a5afefc82b0a6fa4b7~tplv-k3u1fbpfcp-watermark.image)

添加点击事件，首先将全部的sprite放到一个数组里

```javascript
sprite.detail = hotPoints[i].detail;
poiObjects.push(sprite);
复制代码
```

然后我们通过射线检测（raycast），就像是镜头中心向鼠标所点击的方向发射出一颗子弹，去检查这个子弹最终会打中哪些物体。

![2021-06-15 01_35_14.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bdad02da56f94068bfda680fdd44cf73~tplv-k3u1fbpfcp-watermark.image)

```javascript
document.querySelector("#container").addEventListener("click",function(event){
    event.preventDefault();

    var raycaster = new THREE.Raycaster();
    var mouse = new THREE.Vector2();

    mouse.x = ( event.clientX / document.body.clientWidth ) * 2 - 1;
    mouse.y = - ( event.clientY / document.body.clientHeight ) * 2 + 1;

    raycaster.setFromCamera( mouse, camera );

    var intersects = raycaster.intersectObjects( poiObjects );
    if(intersects.length>0){
        alert("点击了热点"+intersects[0].object.detail.title);
    }
});
复制代码
```

# 方案二：CSS3D

`threejs`等3d引擎太强大了，这些引擎的代码量都有大几百K，在今天的网速下显得无所谓，但在几年前我接到需求时仍然是重要的考量因素。既然我们只用到3D引擎的一点点功能，那么能否找到一个更加轻量的3D引擎呢。

有！[css3d-engine](https://github.com/shrekshrek/css3d-engine)，这个3d引擎只有`14kb`，并且在多个大牌商业项目中应用

- 淘宝造物节 [shrek.imdevsh.com/show/zwj/](https://shrek.imdevsh.com/show/zwj/)
- adidas绝不凋谢 [shrek.imdevsh.com/show/drose/](https://shrek.imdevsh.com/show/drose/)
- adidas胜势全开 [shrek.imdevsh.com/show/bbcny/](https://shrek.imdevsh.com/show/bbcny/)
- adidas绝不跟随 [shrek.imdevsh.com/show/crazyl…](https://shrek.imdevsh.com/show/crazylight/)

## 使用skybox实现

```javascript
window.onload=initCSS3D;

function initCSS3D(){
    var s = new C3D.Stage();
    s.size(window.innerWidth, window.innerHeight).update();
    document.getElementById('container').appendChild(s.el);

    var box = new C3D.Skybox();
    box.size(954).position(0, 0, 0).material({
        front: {image: "images/scene_front.jpeg"},
        back: {image: "images/scene_back.jpeg"},
        left: {image: "images/scene_right.jpeg"},
        right: {image: "images/scene_left.jpeg"},
        up: {image: "images/scene_top.jpeg"},
        down: {image: "images/scene_bottom.jpeg"},

    }).update();
    s.addChild(box);

    function loop() {
        angleX += (curMouseX - lastMouseX + lastAngleX - angleX) * 0.3;
        angleY += (curMouseY - lastMouseY + lastAngleY - angleY) * 0.3;

        s.camera.rotation(angleY, -angleX, 0).updateT();
        requestAnimationFrame(loop);
    }

    loop();

    var lastMouseX = 0;
    var lastMouseY = 0;
    var curMouseX = 0;
    var curMouseY = 0;
    var lastAngleX = 0;
    var lastAngleY = 0;
    var angleX = 0;
    var angleY = 0;

    document.addEventListener("mousedown", mouseDownHandler);
    document.addEventListener("mouseup", mouseUpHandler);

    function mouseDownHandler(evt) {
        lastMouseX = curMouseX = evt.pageX;
        lastMouseY = curMouseY = evt.pageY;
        lastAngleX = angleX;
        lastAngleY = angleY;

        document.addEventListener("mousemove", mouseMoveHandler);
    }

    function mouseMoveHandler(evt) {
        curMouseX = evt.pageX;
        curMouseY = evt.pageY;
    }

    function mouseUpHandler(evt) {
        curMouseX = evt.pageX;
        curMouseY = evt.pageY;

        document.removeEventListener("mousemove", mouseMoveHandler);
    }
}
复制代码
```

方案二的好处除了库很小以外，还是div+css来搭建三维场景的。但这个库的作者几乎不维护，遇到问题必须得自己想办法解决，比如使用在电脑上会看到明显的面片边缘

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0550ab3eae6f48399dab58334f3d85ed~tplv-k3u1fbpfcp-watermark.image)

但是在手机上浏览的话表现还是相当完美的

![2021-06-14 22_20_26.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9b89d376ff6b4fac8149d9c91c6a775c~tplv-k3u1fbpfcp-watermark.image)

## 添加信息点

我们继续为它添加可交互的信息点

```javascript
var hotPoints=[
    {
        position:{
            x:0,
            y:0,
            z:-476
        },
        detail:{
            "title":"信息点1"
        }
    },
    {
        position:{
            x:0,
            y:0,
            z:476
        },
        detail:{
            "title":"信息点2"
        }
    }
];
复制代码
function initPoints(){
    var poiObjects = [];
    for(var i=0;i<hotPoints.length;i++){
        var _p = new C3D.Plane();

        _p.size(207, 162).position(hotPoints[i].position.x,hotPoints[i].position.y,hotPoints[i].position.z).material({
            image: "images/hot.png",
            repeat: 'no-repeat',
            bothsides: true,//注意这个两面贴图的属性
        }).update();
        s.addChild(_p);

        _p.el.detail = hotPoints[i].detail;

        _p.on("click",function(e){
            console.log(e.target.detail.title);
        })
    }
}
复制代码
```

这样就可以显示信息点了，并且由于是div，我们非常容易添加鼠标点击交互等效果

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/322d2694cc0a4fccade31d54c19d749a~tplv-k3u1fbpfcp-watermark.image)

不过，`bothsides`属性为true时，背面的信息点图片是反的。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/56c751dda5054eb697f86ee960d049d9~tplv-k3u1fbpfcp-watermark.image)

所以我们这里要做一点处理，根据其与相机的夹角重置一下信息点的旋转角度。（`如果是那种怎么旋转都无所谓的图片，比如圆点则无需处理`）

```javascript
var r = Math.atan2(hotPoints[i].position.z-0,0-0) * 180 / Math.PI+90;
_p.size(207, 162).position(hotPoints[i].position.x,hotPoints[i].position.y,hotPoints[i].position.z).material({
            image: "images/hot.png",
            repeat: 'no-repeat',
            bothsides: false,
        }).update();
复制代码
```

# 需求升级了！

以上两个方案，我以为可以给客户交差了。但客户又提出了一些想法

- **全景图质量需要更高，但加载速度不允许更慢**
- **每个场景的信息点挺多的，坐标编辑太麻烦了**

当时我心里想，总共才收你万把块钱，难不成还得给你定制个引擎，再做个可视化编辑器？

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/447ed6e19cc04dc1aa00499729d12eeb~tplv-k3u1fbpfcp-watermark.image)

直到客户发过来一个参考链接，我看完`惊呆了`，全景图非常清晰，但首次加载速度极快，像百度地图一样，是一块块从模糊到清晰被加载出来的。

![2021-06-14 23_31_28.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5afe3486f7584d8a8e14a5111b91f082~tplv-k3u1fbpfcp-watermark.image)

通过检查参考链接网页的代码，发现了方案三

# 方案三：pano2vr

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3ea232a0cb834c9f9aaab05007a0dff0~tplv-k3u1fbpfcp-watermark.image)

pano2vr是一款所见即所得的全景VR制作软件（正版149欧元），功能挺强大的，可以直接输出成HTML5静态网页，体验非常不错。

而其核心库`pano2vr_player.js`代码量也只有`238kb`。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e88bcbbb4da84e26b9c6d62da2a9e48c~tplv-k3u1fbpfcp-watermark.image)

我们可以直接使用这个软件来可视化的添加信息点，输出成HTML5后，除了静态图片以外，所有配置信息都在这个`pano.xml`文件里

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/efad4f180a204623b4796d2194e165d3~tplv-k3u1fbpfcp-watermark.image)

## 修改信息点图片

整体的交互体验都非常好，但默认的信息点样式不喜欢，我们可以通过下面的代码来修改信息点图片

```javascript
pano.readConfigUrlAsync("pano.xml",()=>{
    var pois=pano.getPointHotspotIds();

    var hotScale = 0.2;

    for(var i=0;i<pois.length;i++){
            var ids=pois[i];
            var hotsopt=pano.getHotspot(ids);
            hotsopt.div.firstChild.src="images/hot.png";
            hotsopt.div.firstChild.style.width = 207*hotScale+"px";
            hotsopt.div.firstChild.style.height = 162*hotScale+"px";
            hotsopt.div.onmouseover = null;
            hotsopt.div.setAttribute("ids",ids);
            hotsopt.div.onclick=function() {
                   //在这里可以响应信息点的点击事件啦
                   console.log(this.getAttribute("ids"));
            };
    }
});
```

哈哈，没想到最终的方案不仅极其简单的就实现了体验良好的VR全景，还附送了非常方便的信息点编辑。除去第一次开发的耗时，以后再制作新的VR场景也就是花个10分钟即可搞定。