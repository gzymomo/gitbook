# vue-element-admin点击按钮跳转新页面方法$router.push

1：首先在view里面新建一个新的组件页面
 addChip.vue

![img](https://img-service.csdnimg.cn/img_convert/484ac6f19a02622fc39a2372e09a5ed4.png)

2：打开router/index.js
 配置新增组件

```
 {
        path: 'addChip',
        component: () => import('@/views/chip/addChip'),
        name: 'addChip',
        meta: { title: '新增拼图' },
        hidden:true,
      },
```

3：打开chip组件界面，开始写一个按钮

```
<template>
  <el-button class="filter-item" type="primary" @click="$router.push('/chip/addChip')">新增拼图</el-button>
</template>
<script>
export default {
  data() {
    return {};
  },
  computed: {},
  methods: {},
};
</script>
<style lang='scss'>
</style>
```

4：ok
 点击按钮，就可以跳转新界面了
 ![img](https://img-service.csdnimg.cn/img_convert/6a3614a97fe79e9c41bfc41ed9c7a706.png)
