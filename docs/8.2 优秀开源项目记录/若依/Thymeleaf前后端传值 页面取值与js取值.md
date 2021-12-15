# Thymeleaf前后端传值 页面取值与js取值

1.后台Controller

```java
@GetMapping("/message")
public String getMessage(Model model){
    model.addAttribute("message","This is your message");
    return "index";
}
```

注：向model中添加属性message

2.页面通过Model取值

```html
<p th:text="#{message}">default message</p>1
```

注：thymeleaf标准表达式语法还有很多

3.js通过model取值

```javascript
<script th:inline="javascript">
    var message = [[${message}]];
    console.log(message);
</script>1234
```

注：script标签中 th:inline 一定不能少，通常在取值的前后会加上不同的注释