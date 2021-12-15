- [SpringBoot启动后自动打开浏览器访问项目](https://juejin.cn/post/6844903972113743885)

# 具体实现方案

我想做成一个通用的启动，所以可以随手配置是否需要启动是打开浏览器

## Mac 电脑

1. 属性文件中添加对应属性

```yaml
#运行项目后是否在浏览器中打开浏览器
openProject:
  isOpen: true  #是否打开浏览器运行 
  cmd: open -a   #运行命令
  web:
    openUrl: http://localhost:8989/ #项目要运行url
    googleExcute: GoogleChrome  #运行的浏览器
```

> 这里我的电脑是Mac 所以需要使用这个open -a 命令window不需要这个属性还有这个googleExcute表示浏览器名称默认Mac 浏览器名称是有空格，请把这个app名字空格去掉且不能有种中文，否则使用open -a 命令无效无法打开

通过定义属性配置文件达到可以定制化，随手关闭

1. 编写自己的CommandRunner类实现CommandLineRunner接口的run方法，这个方法会在项目启动后制动执行

```java
package com.fashvn.ctmsdata.config;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

@Slf4j
@Component
public class CommandRunner implements CommandLineRunner {
    @Value("${openProject.web.openUrl}")
    private String openUrl;
    @Value("${openProject.isOpen}")
    private boolean isOpen;
    @Value("${openProject.cmd}")
    private String cmd;
    @Value("${openProject.web.googleExcute}")
    private String googleExcutePath;


    @Override
    public void run(String... args) throws Exception {
        if (isOpen) {
            String runCmd = cmd+" "+googleExcutePath + " " +openUrl ;
            log.info("运行的命令:{}",runCmd);
            Runtime run = Runtime.getRuntime();
            try {
                run.exec(runCmd);
                log.debug("启动浏览器打开项目成功");
            } catch (Exception e) {
                e.printStackTrace();
                log.error("启动项目自动打开浏览器失败:{}",e.getMessage());
            }
        }
    }
}
```

## window电脑



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/1/17/16fb2ac83199ec12~tplv-t2oaga2asx-watermark.image)



上图只是运行命令不一样，比mac简单，只用改下自己电脑对应浏览器路径就可以