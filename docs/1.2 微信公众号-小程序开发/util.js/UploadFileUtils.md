# 1、生成上传文件路径

```javascript
const genCloudFilePath = function (localFilePath) {
  const suffix = localFilePath.substring(localFilePath.length, localFilePath.lastIndexOf(".")) || '.jpg';//获取后缀名
  const now = new Date()
  const year = now.getFullYear();
  let month = now.getMonth() + 1;
  let day = now.getDate();
  let hour = now.getHours();
  let minutes = now.getMinutes();
  let seconds = now.getSeconds();
  month = month < 10 ? '0' + month : month;
  day = day < 10 ? '0' + day : day;
  hour = hour < 10 ? '0' + hour : hour;
  minutes = minutes < 10 ? '0' + minutes : minutes;
  seconds = seconds < 10 ? '0' + seconds : seconds;
  const yyyyMMddHHmmss = `${year}${month}${day}${hour}${minutes}${seconds}`;
  return 'upload/' + year + month + '/' + yyyyMMddHHmmss + Math.random().toString(10).substr(2, 10) + suffix;
};

module.exports = {
  genCloudFilePath: genCloudFilePath
}
```

