**简要描述：** 

- 获取用户token

**请求URL：** 

- `https://api-hmugo-web.itheima.net/api/public/v1/users/wxlogin`

**请求方式：**

- POST 

**参数：** 

以下字段主要用作后台服务器生成用户token所有，无特殊用意

| 参数名        | 必选 | 类型   | 参数说明                       |
| ------------- | ---- | ------ | ------------------------------ |
| encryptedData | 是   | string | 执行小程序 获取用户信息后 得到 |
| rawData       | 是   | string | 执行小程序 获取用户信息后 得到 |
| iv            | 是   | string | 执行小程序 获取用户信息后 得到 |
| signature     | 是   | string | 执行小程序 获取用户信息后 得到 |
| code          | 是   | string | 执行小程序登录后获取           |

 **返回示例**

```
    {
      "message": {
        "user_id": 23,
        "user_email_code": null,
        "is_active": null,
        "user_sex": "男",
        "user_qq": "",
        "user_tel": "",
        "user_xueli": "本科",
        "user_hobby": "",
        "user_introduce": null,
        "create_time": 1562221487,
        "update_time": 1562221487,
        "token": "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1aWQiOjIzLCJpYXQiOjE1NjQ3MzAwNzksImV4cCI6MTAwMTU2NDczMDA3OH0.YPt-XeLnjV-_1ITaXGY2FhxmCe4NvXuRnRB8OMCfnPo"
      },
      "meta": {
        "msg": "登录成功",
        "status": 200
      }
    }
```

 **返回参数说明** 

| 参数名 | 类型   | 说明                                       |
| :----- | :----- | ------------------------------------------ |
| token  | string | 用户的唯一凭据，后期用在其他敏感接口的验证 |