## 1、获取成员变量类型

```java
@Override
public void visitNode(Tree tree) {
    if (tree instanceof VariableTree) {
        VariableTree variableTree = (VariableTree) tree;
        System.out.println(variableTree.symbol().type().fullyQualifiedName());
    }
}
```

## 2、获取节点的行号

```java
@Override
public void visitNode(Tree tree) {
    //获取该节点开始行，例如某方法的第一行行号
    System.out.println(tree.firstToken().line());
    //获取该节点结束行，例如某方法的最后一行行号
    System.out.println(tree.lastToken().line());
}
```