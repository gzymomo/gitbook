## 1、文本式读取被扫描java文件

```java
public class TXTestCheck extends BaseTreeVisitor implements JavaFileScanner {
    public void scanFile(JavaFileScannerContext context) {
        scan(context.getTree());
        visitFile(context.getFile());
    }

    private void visitFile(File file) {
        // 直接获取当前java文件的所有文本
        List<String> lines = new ArrayList<String>();;
        try {
            BufferedReader reader = new BufferedReader(new FileReader(file));
            String line = null;
            while((line = reader.readLine()) != null){
                lines.add(line);
            }
            reader.close();
        } catch (IOException e) {
            throw new IllegalStateException(e);
        }
    }

}
```

## 2、sonar树扫描节点

```java
public class TXTestCheck extends BaseTreeVisitor implements JavaFileScanner {
    private JavaFileScannerContext context;

    @Override
    public void visitClass(ClassTree tree) {
        // 类名
        System.out.println(tree.simpleName().name());
        // 是否是抽象类
        tree.symbol().isAbstract();
        super.visitClass(tree);
    }

    @Override
    public void visitMethod(MethodTree tree) {
        // 方法名
        System.out.println(tree.simpleName().name());
        // 是否是抽象方法
        tree.symbol().isAbstract();
        // 是否是public方法
        tree.symbol().isPublic();
        // ...
        super.visitMethod(tree);
    }

    public void scanFile(JavaFileScannerContext context) {
        this.context = context;
        scan(context.getTree());
    }

}
```

## 3、指定soanr树扫描节点

```java
public class TXTestCheck extends IssuableSubscriptionVisitor {

    @Override
    public List<Tree.Kind> nodesToVisit() {
        //指定要扫描的节点，在visitNode方法中获取到指定的节点
        return ImmutableList.of(Tree.Kind.METHOD, Tree.Kind.VARIABLE);
    }
    @Override
    public void scanFile(JavaFileScannerContext context) {
        super.context = context;
        super.scanFile(context);
    }
    @Override
    public void visitNode(Tree tree) {
        if (tree instanceof MethodTree) {
            MethodTree methodTree = (MethodTree) tree;
            System.out.println(methodTree.simpleName().name());
        }
        if (tree instanceof VariableTree) {
            VariableTree variableTree = (VariableTree) tree;
            System.out.println(variableTree.simpleName().name());
        }
    }
}
```

## 4、获取注释

```java
/**
 * 获取类注释
 * @param tree
 * @return
 */
private String getJavadoc(ClassTree tree){
    String javadoc = null;
    for(SyntaxTrivia trivia : tree.modifiers().firstToken().trivias()){
        String comment = trivia.comment();
        if(comment != null && comment.trim().startsWith("/*")){
            // 获取javadoc注释
            System.out.println(comment);
        }
        if(comment != null && comment.trim().startsWith("//")){
            //获取普通单行注释"//"
            System.out.println(comment);
        }
    }
    return javadoc;
}
/**
 * 获取方法注释
 * @param tree
 * @return
 */
private String getJavadoc(MethodTree tree){
    String javadoc = null;
    for(SyntaxTrivia trivia : tree.modifiers().firstToken().trivias()){
        String comment = trivia.comment();
        if(comment != null && comment.trim().startsWith("/*")){
            // 获取javadoc注释
            System.out.println(comment);
        }
        if(comment != null && comment.trim().startsWith("//")){
            //获取普通单行注释"//"
            System.out.println(comment);
        }
    }
    return javadoc;
}
/**
 * 获取成员变量注释
 *
 * 需要先判断是否为类成员变量：
 * if(variableTree.symbol().isPublic() || variableTree.symbol()
 *  .isProtected() ||variableTree.symbol().isPrivate()){getJavadoc(variableTree)}
 * @param tree
 * @return
 */
private String getJavadoc(VariableTree tree){
    String javadoc = null;
    for(SyntaxTrivia trivia : tree.modifiers().firstToken().trivias()){
        String comment = trivia.comment();
        if(comment != null && comment.trim().startsWith("/*")){
            // 获取javadoc注释
            System.out.println(comment);
        }
        if(comment != null && comment.trim().startsWith("//")){
            //获取普通单行注释"//"
            System.out.println(comment);
        }
    }
    return javadoc;
}
/**
 * 获取枚举的javadoc
 * @param tree
 * @return
 */
private String getJavadoc(EnumConstantTree tree){
    String javadoc = null;
    for(SyntaxTrivia trivia : tree.initializer().firstToken().trivias()){
        String comment = trivia.comment();
        if(comment != null && comment.trim().startsWith("/*")){
            // 获取javadoc注释
            System.out.println(comment);
        }
        if(comment != null && comment.trim().startsWith("//")){
            //获取普通单行注释"//"
            System.out.println(comment);
        }
    }
    return javadoc;
}
```

**注：sonarqube5.x和6.x版本获取注释的方法有细微不同，此处的代码适用于6.x版本**