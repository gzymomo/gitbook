## 1、单例模式

所有的规则类都是单例模式，所以规则类中最好不要有成员变量。若无法避免时，则必须在节点扫描前清空成员变量的数据。例如

```java
private List<String> methodNameList = new ArrayList<String>();
private List<VariableTree> variableTreeList = new ArrayList<VariableTree>();


public List<Tree.Kind> nodesToVisit() {
    return ImmutableList.of(Tree.Kind.METHOD,Tree.Kind.VARIABLE);
}

@Override
public void visitNode(Tree tree) {
    // TODO
}

@Override
public void scanFile(JavaFileScannerContext context) {
    //清空成员变量数据
    methodNameList.clear();
    variableTreeList.clear();

    super.context = context;
    super.scanFile(context);
    // TODO
}
```