[TOC]

# 1、异常分类
这里的异常分类从系统处理异常的角度看，主要分类两类：业务异常和系统异常。

## 1。1 业务异常
业务异常主要是一些可预见性异常，处理业务异常，用来提示用户的操作，提高系统的可操作性。

常见的业务异常提示：
1. 请输入xxx
2. xxx不能为空
3. xxx重复，请更换

## 1.2 系统异常
系统异常主要是一些不可预见性异常，处理系统异常，可以让展示出一个友好的用户界面，不易给用户造成反感。如果是一个金融类系统，在用户界面出现一个系统异常的崩溃界面。

常见的系统异常提示：
1. 页面丢失404
2. 服务器异常500

# 2、自定义异常处理
## 2.1 自定义业务异常类
```java
public class ServiceException extends Exception {
    public ServiceException (String msg){
        super(msg);
    }
}
```

## 2.2 自定义异常描述对象
```java
public class ReturnException {
    // 响应码
    private Integer code;
    // 异常描述
    private String msg;
    // 请求的Url
    private String url;
    // 省略 get set 方法
}
```

## 2.3 统一异常处理格式
1.两个基础注解
- @ControllerAdvice 定义统一的异常处理类
- @ExceptionHandler 定义异常类型对应的处理方式

2.代码实现
```java
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.servlet.ModelAndView;
import javax.servlet.http.HttpServletRequest;
@ControllerAdvice
// 异常以Json格式返回 等同 ExceptionHandler + ResponseBody 注解
// @RestControllerAdvice
public class HandlerException {
    /**
     * 自定义业务异常映射,返回JSON格式提示
     */
    @ExceptionHandler(value = ServiceException.class)
    @ResponseBody
    public ReturnException handler01 (HttpServletRequest request,ServiceException e){
        ReturnException returnException = new ReturnException() ;
        returnException.setCode(600);
        returnException.setMsg(e.getMessage());
        returnException.setUrl(String.valueOf(request.getRequestURL()));
        return returnException ;
    }
    /**
     * 服务异常
     */
    @ExceptionHandler(value = Exception.class)
    public ModelAndView handler02 (HttpServletRequest request,Exception e){
        ModelAndView modelAndView = new ModelAndView() ;
        modelAndView.addObject("ExeMsg", e.getMessage());
        modelAndView.addObject("ReqUrl", request.getRequestURL());
        modelAndView.setViewName("/exemsg");
        return modelAndView ;
    }
}
```

## 2.4 简单的测试接口
```java
@Controller
public class ExeController {
    /**
     *  {
     *    "code": 600,
     *    "msg": "业务异常：ID 不能为空",
     *    "url": "http://localhost:8003/exception01"
     *  }
     */
    @RequestMapping("/exception01")
    public String exception01 () throws ServiceException {
        throw new ServiceException("业务异常：ID 不能为空");
    }

    @RequestMapping("/exception02")
    public String exception02 () throws Exception {
        throw new Exception("出现异常，全体卧倒");
    }
}
```