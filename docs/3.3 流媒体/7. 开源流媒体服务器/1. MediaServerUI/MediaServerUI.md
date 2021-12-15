Gitee地址：https://gitee.com/kkkkk5G/MediaServerUI



# 一、部署项目

## 1.1 克隆项目

```bash
git clone https://gitee.com/kkkkk5G/MediaServerUI.git
```



## 1.2 修改配置文件

```javascript
vi ./global.js   
	host=your server ip
	secret=your server secret
	baseMediaUrl=your app addr 
```

demo：

```javascript
const serverip="192.168.0.144:8080"
const host = 'http://' + serverip + '/index/api';
const secret = 'test';
const baseMediaUrl='http://192.168.0.144:8080/';# 注意此处要加/，否则视频访问的地址拼接会有误
function genApiUrl(method){
	return host+method+"?secret="+secret;
}
export default{
	serverip,
    host,
    secret,
	genApiUrl,
	baseMediaUrl
}
```



## 1.3 安装node环境（需要高版本）



## 1.4 项目安装

```bash
npm install
```

## 1.5 启动项目

```bash
npm run serve
```



# 二、效果展示

![Image text](https://gitee.com/kkkkk5G/MediaServerUI/raw/master/screenshot/index.png) ![Image text](https://gitee.com/kkkkk5G/MediaServerUI/raw/master/screenshot/videoList.png) ![Image text](https://gitee.com/kkkkk5G/MediaServerUI/raw/master/screenshot/videoPlayer.png) ![Image text](https://gitee.com/kkkkk5G/MediaServerUI/raw/master/screenshot/videoHistory.png)