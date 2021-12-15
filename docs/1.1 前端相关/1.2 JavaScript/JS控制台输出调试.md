## 一、console.table()

`console.table()`是我非常建议大家去使用的方法，它可以接受JSON或数组并以表格格式打印，在对json对象和数组进行可视化打印的时候简单易用，结果直观。

比如下面的json数据对象使用`console.table()`打印

```javascript
    console.table({
    "id":"1",
    "key":"value",
    "count":2
    });
```

控制台的输出结果如下：

![img](https://img2020.cnblogs.com/other/1815316/202010/1815316-20201010073805044-1063580885.png)

又比如对下面代码中的数组进行打印：

```javascript
 console.table([
    {
        id: "1",
        key: "value",
        count: 2,
        },
     {
         id: "2",
         key: "value2",
             count: 22,
       },
       {
            id: "3",
            key: "value3",
                count: 5,
               },
     ]);
```

控制台的输出结果如下：

![img](https://img2020.cnblogs.com/other/1815316/202010/1815316-20201010073805255-892142785.png)

## 二、console.error()

`console.error()`相对于`console.log()`更有助于在调试时从输出日志中区分错误信息

![img](https://img2020.cnblogs.com/other/1815316/202010/1815316-20201010073805528-272372703.png)

从上图中可以看到，它的输出打印结果是红色的。

## 三、Time(time,timeLog,timeEnd)

console.time()、console.timeLog()、console.timeEnd() 这三个方法当我们对程序运行时间进行计时的时候特别有用。

参考下图理解这三个方法

![img](https://img2020.cnblogs.com/other/1815316/202010/1815316-20201010073805714-1362255397.png)

- console.time()相当于秒表中的开始按钮
- console.timeLog()相当于秒表中的按圈计时/按点计时
- console.timeEnd()相当于计时结束

```javascript
console.time("ForLoop");  
 // "ForLoop" is label here
for (let i = 0; i < 5; i++) {
    console.timeLog('ForLoop'); 
}
console.timeEnd("ForLoop");
```

控制台打印输出结果

![img](https://img2020.cnblogs.com/other/1815316/202010/1815316-20201010073805886-1882328881.png)

## 四、`console.warn()`

用黄色字体输出日志，更直观的方便的查看警告类日志信息。

![img](https://img2020.cnblogs.com/other/1815316/202010/1815316-20201010073806125-1598764827.png)

## 五、console.assert()

`console.assert(assert_statement,message)`用来设定断言，如果为**false**则显示message消息

```javascript
if(3!=2){
    console.error({ msg1: "msg1", msg2: "msg2" });
}
//上面的日志判断语句，可以简写为下面的断言
console.assert(3 === 2, { msg1: "msg1", msg2: "msg2" });
```

![img](https://img2020.cnblogs.com/other/1815316/202010/1815316-20201010073806282-238168627.png)

**另一种可以用来格式化输出的断言方式**`console.assert(assert_statement,message,args)`

```javascript
console.assert(false, "%d nd type for  %s ",2,"console.assert() method");
```

![img](https://img2020.cnblogs.com/other/1815316/202010/1815316-20201010073806462-64734286.png)

## 六、console.count()

`console.count()`特别适合用来计数，可以传递参数，可以根据根据参数标签统计次数。代码如下：

```javascript
 for (let i = 0; i < 3; i++) {
   console.count("label");
   console.count();
   console.count(i);
 }
```

控制台打印输出的结果，类似于下面这样

```javascript
 console.count()  console.count("label")   console.count(i)
 default: 1                label: 1                0: 1
 default: 2                label: 2                1: 1
 default: 3                label: 3                2: 1
```

- `console.count()`如果不传递参数，则使用默认的default标签。
- `console.countReset(标签参数)`可以将指定标签的计数重置为**0**