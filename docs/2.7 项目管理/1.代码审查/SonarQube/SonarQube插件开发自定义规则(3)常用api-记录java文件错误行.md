## 1、通过节点记录错误行

```java
reportIssue(tree, "记录信息");
// 同上
context.reportIssue(this, tree, "记录信息");123
```

## 2、通过行号记录错误行

```java
addIssue(lineNum, "记录信息");1
```

## 3、效果

![这里写图片描述](https://img-blog.csdn.net/20171003101515210)