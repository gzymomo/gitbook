# 1、毫秒转换友好的显示格式-获取距今天数

```javascript
/**
 * 毫秒转换友好的显示格式
 * 输出格式：21小时前
 * @param  {[type]} time [description]
 * @return {[type]}      [description]
 */
const formatTime = date => {
  //获取js 时间戳
  var time = new Date().getTime();
  //去掉 js 时间戳后三位，与php 时间戳保持一致
  time = parseInt((time - date) / 1000);

  //存储转换值 
  var s;
  if (time < 60 * 10) {//十分钟内
    return '刚刚';
  } else if ((time < 60 * 60) && (time >= 60 * 10)) {
    //超过十分钟少于1小时
    s = Math.floor(time / 60);
    return s + "分钟前";
  } else if ((time < 60 * 60 * 24) && (time >= 60 * 60)) {
    //超过1小时少于24小时
    s = Math.floor(time / 60 / 60);
    return s + "小时前";
  } else if ((time < 60 * 60 * 24 * 3) && (time >= 60 * 60 * 24)) {
    //超过1天少于3天内
    s = Math.floor(time / 60 / 60 / 24);
    return s + "天前";
  } else {
    //超过3天
    var date = new Date(parseInt(date));
    return date.getFullYear() + "/" + (date.getMonth() + 1) + "/" + date.getDate();
  }
}


//获取距今天数
getcreatedtime(created) {
		var now = Date.parse(new Date()) / 1000;
		console.log(now)
		var span = (now - created) > 0 ? (now - created) : 0;
		if (span <= 3600) {
			var ts = Math.round(span / 60);
			return (ts + '分钟前');
		} else if (span < 86400) {
			var ts = Math.round(span / 3600);
			return (ts + '小时前');
		} else {
			var ts = Math.round(span / 86400);
			return (ts + '天前');
		}
}
```

# 2、生成默认日期

```javascript
//生成默认日期
const genDefDate = function () {
  const now = new Date()
  const year = now.getFullYear();
  let month = now.getMonth() + 1;
  let day = now.getDate();
  month = month < 10 ? '0' + month : month;
  day = day < 10 ? '0' + day : day;
  return `${year}-${month}-${day}`;
}
```

