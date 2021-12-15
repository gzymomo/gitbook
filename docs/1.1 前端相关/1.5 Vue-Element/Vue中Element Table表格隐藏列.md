**我们在table中添加数据时，有时需要隐藏一些页面不需要看到但必须存在的列。这时我们就需要把这列进行隐藏操作(比如记录id)。**
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190404143626148.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM3ODM1OTA2,size_16,color_FFFFFF,t_70)
 **在对应列声明中，加入 v-if=“show”（红线标注的地方）**
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190404143651596.png)
 **在date()的return中声明 show: false,(红线标注部分)此时页面加载的时候会隐藏对应列。**