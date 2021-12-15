# 一、Jenkins配置

## 1.1 安装Prometheus插件

![图片](https://mmbiz.qpic.cn/mmbiz_png/U1oibTqyKuTObABZNpZMaDNibC7Rvl7qQyQgfyLQunnFm68LaibbJMcxLzppS7cmldgZnFTd0KPp5r0av4R7I9b2g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)





## 1.2 配置Prometheus插件

填写Path地址 url路径，namepace。

![图片](https://mmbiz.qpic.cn/mmbiz_png/U1oibTqyKuTObABZNpZMaDNibC7Rvl7qQyibwyhLicmR3iaokwCsqmqpJ9t3z5at9NrZHUFSicfwJoOKVF0P02PArqfQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



## 1.3 测试验证

访问http://yourjenkinserver:port/prometheus

![图片](https://mmbiz.qpic.cn/mmbiz_png/U1oibTqyKuTObABZNpZMaDNibC7Rvl7qQy0Gic19K6vpUXgPwSQbERkWubnCdayBibr8uzmaGDVL58bsEb4xm2lBtg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)







# 二、prometheus配置

## 2.1 修改ConfigMap

![图片](https://mmbiz.qpic.cn/mmbiz_png/U1oibTqyKuTObABZNpZMaDNibC7Rvl7qQyI6sApUTEENzBwetyCPcI2s43yiagNtSR4Cz1b4ACia9GiaVAY5Ssme1cw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 2.2 验证监控项

![图片](https://mmbiz.qpic.cn/mmbiz_png/U1oibTqyKuTObABZNpZMaDNibC7Rvl7qQyMseTJKlicbpBMV37weBZrotHV4NYs3kGGuGpywFNaTDYWVRPFv09O8A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



# 三、Grafana配置

## 3.1 添加数据源

## 3.2 导入模板

![图片](https://mmbiz.qpic.cn/mmbiz_png/U1oibTqyKuTObABZNpZMaDNibC7Rvl7qQy4KyYG0n4iaiaCUibGyzfqvkQndqsxqUov5qh7qZWnzvO5ZXnxKHbgIib6w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/U1oibTqyKuTObABZNpZMaDNibC7Rvl7qQyQibh7XJItl3DuSsZfAI8aibqmQDf77yoXU1qSWr6KxHoUvg7llhh9eCA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/U1oibTqyKuTObABZNpZMaDNibC7Rvl7qQytwia12DFEr8syr0I3Hyxw8g5hpBwufk0yPj4GLGWeJ9mJFxy9LggrKg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



# 四、总结

按照上述配置完成，数据就可以展示出来了。

![图片](https://mmbiz.qpic.cn/mmbiz_png/U1oibTqyKuTObABZNpZMaDNibC7Rvl7qQyU5ECObe3OGumyAEO15xqTnA0Z2fan2SWliczqIXfzKGuLia86xHLSm2w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

