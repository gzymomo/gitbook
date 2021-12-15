此项目为文件文档在线预览项目解决方案，对标业内付费产品有【[永中office](http://dcs.yozosoft.com/)】【[office365](http://www.officeweb365.com/)】【[idocv](https://www.idocv.com/)】等，在取得公司高层同意后以Apache协议开源出来反哺社区，在此特别感谢@唐老大的支持以及@端木详笑的贡献。该项目使用流行的spring boot搭建，易上手和部署，基本支持主流办公文档的在线预览，如doc,docx,Excel,pdf,txt,zip,rar,图片等等

### 在线体验

请善待公共服务，会不定时停用

地址：[https://file.keking.cn](https://file.keking.cn/)

## 项目特性

1. 支持word excel ppt，pdf等办公文档
2. 支持txt,java,php,py,md,js,css等所有纯文本
3. 支持zip,rar,jar,tar,gzip等压缩包
4. 支持jpg，jpeg，png，gif等图片预览（翻转，缩放，镜像）
5. 支持mp3，mp4，flv等多媒体文件预览
6. 使用spring boot开发，预览服务搭建部署非常简便
7. rest接口提供服务，跨平台特性(java,php,python,go,php，....)都支持，应用接入简单方便
8. 支持普通http/https文件下载url、http/https文件下载流url、ftp下载url等多种预览源
9. 提供zip，tar.gz发行包，提供一键启动脚本和丰富的配置项，方便部署使用
10. 提供Docker镜像发行包，方便在容器环境部署
11. 抽象预览服务接口，方便二次开发，非常方便添加其他类型文件预览支持
12. 最最重要Apache协议开源，代码pull下来想干嘛就干嘛

## 预览展示

### 1. 文本预览

支持所有类型的文本文档预览， 由于文本文档类型过多，无法全部枚举，默认开启的类型如下 txt,html,xml,properties,md,java,py,c,cpp,sql
如有没有未覆盖全面，可通过配置文件 [指定文本类型](https://gitee.com/kekingcn/file-online-preview/wikis/pages?sort_id=1442836&doc_id=106093#simtext)
文本预览效果如下
![文本预览效果如下](https://images.gitee.com/uploads/images/2019/0508/183554_a930460f_528790.png)

### 2. 图片预览

支持jpg，jpeg，png，gif等图片预览（翻转，缩放，镜像），预览效果如下 [![输入图片说明](https://static.oschina.net/uploads/img/201801/18064603_JOaw.png)](https://gitee.com/uploads/images/2017/1213/094335_657a6f60_492218.png)

### 3. word文档预览

支持doc，docx文档预览，word预览有两种模式：一种是每页word转为图片预览，另一种是整个word文档转成pdf，再预览pdf。两种模式的适用场景如下

- 图片预览：word文件大，前台加载整个pdf过慢
- pdf预览：内网访问，加载pdf快
  默认为每页word转为图片预览，可通过点击右边的pdf图标转，也可通过配置文件 [设置默认预览模式](https://gitee.com/kekingcn/file-online-preview/wikis/pages?sort_id=1442836&doc_id=106093#officepreviewtype)
  图片预览模式预览效果如下
  ![word文档预览1](https://images.gitee.com/uploads/images/2019/0508/164731_cea117ea_528790.png) 
  pdf预览模式预览效果如下
  ![word文档预览2](https://images.gitee.com/uploads/images/2019/0508/164801_834dfac8_528790.png)

### 4. ppt文档预览

支持ppt，pptx文档预览，和word文档一样，有两种预览模式
图片预览模式预览效果如下
![ppt文档预览1](https://images.gitee.com/uploads/images/2019/0508/164842_c07f38a3_528790.png)
pdf预览模式预览效果如下
![ppt文档预览2](https://images.gitee.com/uploads/images/2019/0508/164932_a7ff3e23_528790.png)

### 5. pdf文档预览

支持pdf文档预览，和word文档一样，有两种预览模式
图片预览模式预览效果如下
![pdf文档预览1](https://images.gitee.com/uploads/images/2019/0508/174325_24b3f827_528790.png)
pdf预览模式预览效果如下

 ![pdf文档预览2](https://images.gitee.com/uploads/images/2019/0508/174340_cb026ba7_528790.png)

### 6. excel文档预览

支持xls，xlsx文档预览，预览效果如下
![excel文档预览](https://images.gitee.com/uploads/images/2019/0508/183859_61932276_528790.png) 
ps，如碰到excel预览乱码问题，可参考 [预览乱码](https://gitee.com/kekingcn/file-online-preview/wikis/pages?sort_id=1444172&doc_id=106093#q预览乱码)

### 7. 压缩文件预览

支持zip,rar,jar,tar,gzip等压缩包，预览效果如下
![压缩文件预览1](https://images.gitee.com/uploads/images/2019/0508/165215_8bc89491_528790.png)
可点击压缩包中的文件名，直接预览文件，预览效果如下
![压缩文件预览2](https://images.gitee.com/uploads/images/2019/0508/165234_32d2ef34_528790.png)

### 8. 多媒体文件预览

理论上支持所有的视频、音频文件，由于无法枚举所有文件格式，默认开启的类型如下
mp3,wav,mp4,flv
如有没有未覆盖全面，可通过配置文件[指定多媒体类型](https://gitee.com/kekingcn/file-online-preview/wikis/pages?sort_id=1442836&doc_id=106093#convertedfilecharset)
视频预览效果如下
![多媒体文件预览1](https://images.gitee.com/uploads/images/2019/0508/170047_cb41920b_528790.png)
音频预览效果如下
![多媒体文件预览2](https://images.gitee.com/uploads/images/2019/0508/170106_6c5b01a4_528790.png)

### 快速开始

项目使用技术

- spring boot： [spring boot开发参考指南](http://www.kailing.pub/PdfReader/web/viewer.html?file=springboot)
- freemarker
- jodconverter

1. 第一步：pull项目https://github.com/kekingcn/file-online-preview.git
2. 第二步：运行FilePreviewApplication的main方法，服务启动后，访问http://localhost:8012/ 会看到如下界面，代表服务启动成功 
   ![img](https://oscimg.oschina.net/oscnet/up-97ddfe027622f082e0baf89985312fa9b93.png)