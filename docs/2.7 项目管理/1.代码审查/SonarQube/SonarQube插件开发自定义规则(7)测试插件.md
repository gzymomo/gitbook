## 1、测试代码

```java
JavaCheckVerifier.verify("src/test/files/DoTest.java", new TXTooMuchIfCheck());1
```

## 2、效果

2.1、当打印一下内容时，则说明自定义的规则插件没有对被检测的java文件记录错误行。

```java
Exception in thread "main" java.lang.IllegalStateException: At least one issue expected
    at com.google.common.base.Preconditions.checkState(Preconditions.java:174)
    at org.sonar.java.checks.verifier.CheckVerifier.assertMultipleIssue(CheckVerifier.java:175)
    at org.sonar.java.checks.verifier.CheckVerifier.checkIssues(CheckVerifier.java:170)
    at org.sonar.java.checks.verifier.JavaCheckVerifier.scanFile(JavaCheckVerifier.java:275)
    at org.sonar.java.checks.verifier.JavaCheckVerifier.scanFile(JavaCheckVerifier.java:257)
    at org.sonar.java.checks.verifier.JavaCheckVerifier.scanFile(JavaCheckVerifier.java:223)
    at org.sonar.java.checks.verifier.JavaCheckVerifier.verify(JavaCheckVerifier.java:106)
    at org.sonar.java.rule.checks.namerules.TXClassNameStartCheckTest.main(TXClassNameStartCheckTest.java:26)
```

2.2、当打印一下内容时，则说明被检测的java文件的第34、51、69、71、73行被自定义的规则插件记录错误。

```java
Exception in thread "main" java.lang.AssertionError: Unexpected at [34, 51, 69, 71, 73]
    at org.fest.assertions.Fail.failure(Fail.java:228)
    at org.fest.assertions.Fail.fail(Fail.java:218)
    at org.sonar.java.checks.verifier.CheckVerifier.assertMultipleIssue(CheckVerifier.java:185)
    at org.sonar.java.checks.verifier.CheckVerifier.checkIssues(CheckVerifier.java:170)
    at org.sonar.java.checks.verifier.JavaCheckVerifier.scanFile(JavaCheckVerifier.java:275)
    at org.sonar.java.checks.verifier.JavaCheckVerifier.scanFile(JavaCheckVerifier.java:257)
    at org.sonar.java.checks.verifier.JavaCheckVerifier.scanFile(JavaCheckVerifier.java:223)
    at org.sonar.java.checks.verifier.JavaCheckVerifier.verify(JavaCheckVerifier.java:106)
    at org.sonar.java.rule.checks.namerules.TXClassNameStartCheckTest.main(TXClassNameStartCheckTest.java:26)
```