[vue+elementui进阶之路-el-table中显示图片](https://www.cnblogs.com/congfeicong/p/11038119.html)



1、table中显示图片

2、当需要遍历图片时，不能直接使用prop绑定值

3、一张图片

```html
<el-table-column label="头像">
　　<template slot-scope="scope">
　　　　<img :src="scope.row.img" width="40" height="40" class="head_pic"/>
　　</template>
</el-table-column>
```

4、多张图片

```html
<el-table-column prop="pictures" label="描述图片">
　　<template scope="scope">
　　　　<img v-for="item in scope.row.pictures" :src="item" width="40" height="40" class="head_pic"/>
　　</template>
</el-table-column>
```