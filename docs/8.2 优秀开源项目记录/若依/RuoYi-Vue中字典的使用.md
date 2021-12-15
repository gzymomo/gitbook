- [RuoYi-Vue中字典的使用](https://www.cnblogs.com/xl4ng/p/14056593.html)



## 1.在data中新建一个变量

```
alarmLevelOptions:[]
```

## 2.获取字典信息

```
this.getDicts("alarm_level").then(response => {
  this.alarmLevelOptions = response.data;
});
```

## 3.在页面中使用

```
<el-form-item label="报警级别" prop="alarmLevel">
  <el-select v-model="form.alarmLevel" placeholder="请选择报警级别">
    <el-option
      v-for="dict in alarmLevelOptions"
      :key="dict.dictValue"
      :label="dict.dictLabel"
      :value="parseInt(dict.dictValue)"
    />
  </el-select>
</el-form-item>
```