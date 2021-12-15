## 1、代码

```java
public class TXTooMuchIfCheck extends IssuableSubscriptionVisitor {

    private static final int DEFAULT_MAXIMUM_LINE = 3;
    @RuleProperty(key = "maximumLine",
            description = "if-else if-else最大层数",
            defaultValue = "" + DEFAULT_MAXIMUM_LINE)
    public int maximumLine = DEFAULT_MAXIMUM_LINE;

    public List<Tree.Kind> nodesToVisit() {
        // TODO
    }

    @Override
    public void visitNode(Tree tree) {
        // TODO
    }

}
```

## 2、效果

![这里写图片描述](https://img-blog.csdn.net/20171003101932323)
![这里写图片描述](https://img-blog.csdn.net/20171003101946416)