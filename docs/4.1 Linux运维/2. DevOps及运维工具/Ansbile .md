Ansible功能：

- 系统环境配置
- 安装软件
- 持续集成
- 热回滚

Ansible优点：

- 无客户端
- 推送式
- 丰富的module
- 基于YAML的Playbook

Ansible缺点：

- 效率低，易挂起
- 并发性能差

![](https://www.showdoc.cc/server/api/attachment/visitfile/sign/e3ea61febfacac34f835a93d640f2834?showdoc=.jpg)

Ansiblle框架由以下核心的组件组成：

1. ansible core： 它是Ansible本身的核心模块
2. host inventory： 它是一个主机库，需要管理的主机列表
3. connection plugins： 连接插件，默认采取SSH远程通信协议
4. custom modules： Ansible自定义扩展模块
5. playbook：编排（剧本），按照所设定编排的顺序执行完成安排的任务

![图片](https://mmbiz.qlogo.cn/mmbiz_png/wbiax4xEAl5ySPTYCz0iawyHfbRBU9bUtgaerLicx0RgibYuOZ7ZTIVXiaNxJicKEx7XbPicmcFWuyrYTl1Q8Km8SZoYA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1&retryload=2)

![图片](https://mmbiz.qpic.cn/mmbiz_png/wbiax4xEAl5ySPTYCz0iawyHfbRBU9bUtgOeycml7OtknUChlSc5iajibEkUWeiaGYYOMEpOdib8gAxfagaicA8yeHSMA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/wbiax4xEAl5ySPTYCz0iawyHfbRBU9bUtgzds9hMwZibwNgNg7lUtNibnIS4ntyjQo1icG1WjwlLTbiaIuXnOzPhicMGg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/wbiax4xEAl5ySPTYCz0iawyHfbRBU9bUtgYbZNDEPk3x13ia7mVjlTL8YEUkvNd8hZgNbY52icAKuQ69kZuduFWLtQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/wbiax4xEAl5ySPTYCz0iawyHfbRBU9bUtgqvZ208oYCAxPWAb2x56Yl55WS9sHdVu28j7Kf7dVR9fonsxIVohGoA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/wbiax4xEAl5ySPTYCz0iawyHfbRBU9bUtgGRfDXsau99piaU6otN2ibT1FtjRungJ4DMxJ3RAKfGUbs5bC5Z9c7kLQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/wbiax4xEAl5ySPTYCz0iawyHfbRBU9bUtgNUKEmka0YrRpBfPUkboGAVmMgUZuYhDBfjuICbPwxPCSP5CsOiahG8A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/wbiax4xEAl5ySPTYCz0iawyHfbRBU9bUtgNPIxlPJ44FjcvibFJXY10zIWnf1b7ERSyZNwKS9WVc2yw5GCjic10WRw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/wbiax4xEAl5ySPTYCz0iawyHfbRBU9bUtgQoArXT8B3icbMtU95aup0X2q1UMpEGtGcwjhiaKdvtYBPAggiaSpahBdQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/wbiax4xEAl5ySPTYCz0iawyHfbRBU9bUtg0h2pkP1COD9jNbuOic5VkQ5JzdFTZNeevyHVgurhtsnfm8iadgmicUTwg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/wbiax4xEAl5ySPTYCz0iawyHfbRBU9bUtgiaficiaxW35bMnogJ0sUwArvdU445asTmSmrh70m1RGOkdjCzofqwMGCw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/wbiax4xEAl5ySPTYCz0iawyHfbRBU9bUtgrPTPnMGb36e01dh331bQibibKWPJCcx4oumfcVSSiaiaudUYSgSCuFs9Kg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/wbiax4xEAl5ySPTYCz0iawyHfbRBU9bUtghgheTLkOSqCrR9zJ23TjnqYSgMHFHb5UeIYF2icibLibESdRiaL7KjqsgQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/wbiax4xEAl5ySPTYCz0iawyHfbRBU9bUtgbaAmXt0G7YKmV3QXicHhr7AbyB1GcI6Q3nPkLYc5KVWB31dH01OMPuw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/wbiax4xEAl5ySPTYCz0iawyHfbRBU9bUtgN3apv59fm11Q1wur640kt0Rjym8N43Z9ichNkRV31BjOWToBCLq6c2w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/wbiax4xEAl5ySPTYCz0iawyHfbRBU9bUtgUn45LLHA2kXCeECSwWQlyATnapteY5FGXV0KtLlOajj0KZ3YXnqeRQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)