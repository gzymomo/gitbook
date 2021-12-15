### 1. java执行shell的api

　　执行shell命令，可以说系统级的调用，编程语言自然必定会提供相应api操作了。在java中，有两个api供调用：Runtime.exec(), Process API. 简单使用如下：

#### 1.1. Runtime.exec() 实现

　　调用实现如下：

```java
import java.io.InputStream;
 
public class RuntimeExecTest {
    @Test
    public static void testRuntimeExec() {
        try {
            Process process = Runtime.getRuntime()
                                .exec("cmd.exe /c dir");
            process.waitFor();
        } 
        catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

简单的说就是只有一行调用即可：Runtime.getRuntime().exec("cmd.exe /c dir") ; 看起来非常简洁。

#### 1.2. ProcessBuilder 实现

　　使用ProcessBuilder需要自己操作更多东西，也因此可以自主设置更多东西。（但实际上底层与Runtime是一样的了），用例如下：

```java
public class ProcessBuilderTest {
    @Test
    public void testProcessBuilder() {
        ProcessBuilder processBuilder = new ProcessBuilder();
        processBuilder.command("ipconfig");
        //将标准输入流和错误输入流合并，通过标准输入流读取信息
        processBuilder.redirectErrorStream(true);
        try {
            //启动进程
            Process start = processBuilder.start();
            //获取输入流
            InputStream inputStream = start.getInputStream();
            //转成字符输入流
            InputStreamReader inputStreamReader = new InputStreamReader(inputStream, "gbk");
            int len = -1;
            char[] c = new char[1024];
            StringBuffer outputString = new StringBuffer();
            //读取进程输入流中的内容
            while ((len = inputStreamReader.read(c)) != -1) {
                String s = new String(c, 0, len);
                outputString.append(s);
                System.out.print(s);
            }
            inputStream.close();
        } 
        catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

看起来是要麻烦些，但实际上是差不多的，只是上一个用例没有处理输出日志而已。但总体来说的 ProcessBuilder 的可控性更强，所以一般使用这个会更自由些。

　　以下Runtime.exec()的实现：

```java
// java.lang.Runtime#exec
public Process exec(String[] cmdarray, String[] envp, File dir)
    throws IOException {
    // 仅为 ProcessBuilder 的一个封装
    return new ProcessBuilder(cmdarray)
        .environment(envp)
        .directory(dir)
        .start();
}
```

### 完整的shell调用参考

```java
import com.my.mvc.app.common.exception.ShellProcessExecException;
import com.my.mvc.app.common.helper.NamedThreadFactory;
import lombok.extern.log4j.Log4j2;
import org.apache.commons.io.FileUtils;

import java.io.BufferedReader;
import java.io.File;
import java.io.IOException;
import java.io.InputStreamReader;
import java.nio.charset.Charset;
import java.util.List;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * 功能描述: Shell命令运行工具类封装
 *
 */
@Log4j2
public class ShellCommandExecUtil {


    /**
     * @see #runShellCommandSync(String, String[], Charset, String)
     */
    public static int runShellCommandSync(String baseShellDir, String[] cmd,
                                          Charset outputCharset) throws IOException {
        return runShellCommandSync(baseShellDir, cmd, outputCharset, null);
    }

    /**
     * 真正运行shell命令
     *
     * @param baseShellDir 运行命令所在目录（先切换到该目录后再运行命令）
     * @param cmd 命令数组
     * @param outputCharset 日志输出字符集，一般windows为GBK, linux为utf8
     * @param logFilePath 日志输出文件路径, 为空则直接输出到当前应用日志中，否则写入该文件
     * @return 进程退出码, 0: 成功, 其他:失败
     * @throws IOException 执行异常时抛出
     */
    public static int runShellCommandSync(String baseShellDir, String[] cmd,
                                          Charset outputCharset, String logFilePath)
            throws IOException {
        long startTime = System.currentTimeMillis();
        boolean needReadProcessOutLogStreamByHand = true;
        log.info("【cli】receive new Command. baseDir: {}, cmd: {}, logFile:{}",
                baseShellDir, String.join(" ", cmd), logFilePath);
        ProcessBuilder pb = new ProcessBuilder(cmd);
        pb.directory(new File(baseShellDir));
        initErrorLogHolder(logFilePath, outputCharset);
        int exitCode = 0;
        try {
            if(logFilePath != null) {
                ensureFilePathExists(logFilePath);
//            String redirectLogInfoAndErrCmd = " > " + logFilePath + " 2>&1 ";
//            cmd = mergeTwoArr(cmd, redirectLogInfoAndErrCmd.split("\\s+"));
                pb.redirectErrorStream(true);
                pb.redirectOutput(new File(logFilePath));
                needReadProcessOutLogStreamByHand = false;
            }
            Process p = pb.start();
            if(needReadProcessOutLogStreamByHand) {
                readProcessOutLogStream(p, outputCharset);
            }
            try {
                p.waitFor();
            }
            catch (InterruptedException e) {
                log.error("进程被中断", e);
                setProcessLastError("中断异常:" + e.getMessage());
            }
            finally {
                exitCode = p.exitValue();
                log.info("【cli】process costTime:{}ms, exitCode:{}",
                        System.currentTimeMillis() - startTime, exitCode);
            }
            if(exitCode != 0) {
                throw new ShellProcessExecException(exitCode,
                        "进程返回异常信息, returnCode:" + exitCode
                                + ", lastError:" + getProcessLastError());
            }
            return exitCode;
        }
        finally {
            removeErrorLogHolder();
        }
    }

    /**
     * 使用 Runtime.exec() 运行shell
     */
    public static int runShellWithRuntime(String baseShellDir,
                                          String[] cmd,
                                          Charset outputCharset) throws IOException {
        long startTime = System.currentTimeMillis();
        initErrorLogHolder(null, outputCharset);
        Process p = Runtime.getRuntime().exec(cmd, null, new File(baseShellDir));
        readProcessOutLogStream(p, outputCharset);
        int exitCode;
        try {
            p.waitFor();
        }
        catch (InterruptedException e) {
            log.error("进程被中断", e);
            setProcessLastError("中断异常:" + e.getMessage());
        }
        catch (Throwable e) {
            log.error("其他异常", e);
            setProcessLastError(e.getMessage());
        }
        finally {
            exitCode = p.exitValue();
            log.info("【cli】process costTime:{}ms, exitCode:{}",
                    System.currentTimeMillis() - startTime, exitCode);
        }
        if(exitCode != 0) {
            throw new ShellProcessExecException(exitCode,
                    "进程返回异常信息, returnCode:" + exitCode
                            + ", lastError:" + getProcessLastError());
        }
        return exitCode;
    }

    /**
     * 确保文件夹存在
     *
     * @param filePath 文件路径
     * @throws IOException 创建文件夹异常抛出
     */
    public static void ensureFilePathExists(String filePath) throws IOException {
        File path = new File(filePath);
        if(path.exists()) {
            return;
        }
        File p = path.getParentFile();
        if(p.mkdirs()) {
            log.info("为文件创建目录: {} 成功", p.getPath());
            return;
        }
        log.warn("创建目录:{} 失败", p.getPath());
    }

    /**
     * 合并两个数组数据
     *
     * @param arrFirst 左边数组
     * @param arrAppend 要添加的数组
     * @return 合并后的数组
     */
    public static String[] mergeTwoArr(String[] arrFirst, String[] arrAppend) {
        String[] merged = new String[arrFirst.length + arrAppend.length];
        System.arraycopy(arrFirst, 0,
                merged, 0, arrFirst.length);
        System.arraycopy(arrAppend, 0,
                merged, arrFirst.length, arrAppend.length);
        return merged;
    }

    /**
     * 删除以某字符结尾的字符
     *
     * @param originalStr 原始字符
     * @param toTrimChar 要检测的字
     * @return 裁剪后的字符串
     */
    public static String trimEndsWith(String originalStr, char toTrimChar) {
        char[] value = originalStr.toCharArray();
        int i = value.length - 1;
        while (i > 0 && value[i] == toTrimChar) {
            i--;
        }
        return new String(value, 0, i + 1);
    }

    /**
     * 错误日志读取线程池（不设上限）
     */
    private static final ExecutorService errReadThreadPool = Executors.newCachedThreadPool(
            new NamedThreadFactory("ReadProcessErrOut"));

    /**
     * 最后一次异常信息
     */
    private static final Map<Thread, ProcessErrorLogDescriptor>
            lastErrorHolder = new ConcurrentHashMap<>();

    /**
     * 主动读取进程的标准输出信息日志
     *
     * @param process 进程实体
     * @param outputCharset 日志字符集
     * @throws IOException 读取异常时抛出
     */
    private static void readProcessOutLogStream(Process process,
                                                Charset outputCharset) throws IOException {
        try (BufferedReader stdInput = new BufferedReader(new InputStreamReader(
                process.getInputStream(), outputCharset))) {
            Thread parentThread = Thread.currentThread();
            // 另起一个线程读取错误消息，必须先启该线程
            errReadThreadPool.submit(() -> {
                try {
                    try (BufferedReader stdError = new BufferedReader(
                            new InputStreamReader(process.getErrorStream(), outputCharset))) {
                        String err;
                        while ((err = stdError.readLine()) != null) {
                            log.error("【cli】{}", err);
                            setProcessLastError(parentThread, err);
                        }
                    }
                }
                catch (IOException e) {
                    log.error("读取进程错误日志输出时发生了异常", e);
                    setProcessLastError(parentThread, e.getMessage());
                }
            });
            // 外部线程读取标准输出消息
            String stdOut;
            while ((stdOut = stdInput.readLine()) != null) {
                log.info("【cli】{}", stdOut);
            }
        }
    }

    /**
     * 新建一个进程错误信息容器
     *
     * @param logFilePath 日志文件路径,如无则为 null
     */
    private static void initErrorLogHolder(String logFilePath, Charset outputCharset) {
        lastErrorHolder.put(Thread.currentThread(),
                new ProcessErrorLogDescriptor(logFilePath, outputCharset));
    }

    /**
     * 移除错误日志监听
     */
    private static void removeErrorLogHolder() {
        lastErrorHolder.remove(Thread.currentThread());
    }

    /**
     * 获取进程的最后错误信息
     *
     *      注意: 该方法只会在父线程中调用
     */
    private static String getProcessLastError() {
        Thread thread = Thread.currentThread();
        return lastErrorHolder.get(thread).getLastError();
    }

    /**
     * 设置最后一个错误信息描述
     *
     *      使用当前线程或自定义
     */
    private static void setProcessLastError(String lastError) {
        lastErrorHolder.get(Thread.currentThread()).setLastError(lastError);
    }

    private static void setProcessLastError(Thread thread, String lastError) {
        lastErrorHolder.get(thread).setLastError(lastError);
    }

    /**
     * 判断当前系统是否是 windows
     */
    public static boolean isWindowsSystemOs() {
        return System.getProperty("os.name").toLowerCase()
                .startsWith("win");
    }

    /**
     * 进程错误信息描述封装类
     */
    private static class ProcessErrorLogDescriptor {

        /**
         * 错误信息记录文件
         */
        private String logFile;

        /**
         * 最后一行错误信息
         */
        private String lastError;
        private Charset charset;
        ProcessErrorLogDescriptor(String logFile, Charset outputCharset) {
            this.logFile = logFile;
            charset = outputCharset;
        }
        String getLastError() {
            if(lastError != null) {
                return lastError;
            }
            try{
                if(logFile == null) {
                    return null;
                }
                List<String> lines = FileUtils.readLines(
                        new File(logFile), charset);
                StringBuilder sb = new StringBuilder();
                for (int i = lines.size() - 1; i >= 0; i--) {
                    sb.insert(0, lines.get(i) + "\n");
                    if(sb.length() > 200) {
                        break;
                    }
                }
                return sb.toString();
            }
            catch (Exception e) {
                log.error("【cli】读取最后一次错误信息失败", e);
            }
            return null;
        }

        void setLastError(String err) {
            if(lastError == null) {
                lastError = err;
                return;
            }
            lastError = lastError + "\n" + err;
            if(lastError.length() > 200) {
                lastError = lastError.substring(lastError.length() - 200);
            }
        }
    }
}
```

以上实现，完成了我们在第2点中讨论的几个问题：

　　　　1. 主要使用 ProcessBuilder 完成了shell的调用；
　　　　2. 支持读取进程的所有输出信息，且在必要的时候，支持使用单独的文件进行接收输出日志；
　　　　3. 在进程执行异常时，支持抛出对应异常，且给出一定的errMessage描述；
　　　　4. 如果想控制调用进程的数量，则在外部调用时控制即可；
　　　　5. 使用两个线程接收两个输出流，避免出现应用假死，使用newCachedThreadPool线程池避免过快创建线程；

　　接下来，我们进行下单元测试：

```java
public class ShellCommandExecUtilTest {

    @Test
    public void testRuntimeShell() throws IOException {
        int errCode;
        errCode = ShellCommandExecUtil.runShellWithRuntime("E:\\tmp",
                new String[] {"cmd", "/c", "dir"}, Charset.forName("gbk"));
        Assert.assertEquals("进程返回码不正确", 0, errCode);

    }

    @Test(expected = ShellProcessExecException.class)
    public void testRuntimeShellWithErr() throws IOException {
        int errCode;
        errCode = ShellCommandExecUtil.runShellWithRuntime("E:\\tmp",
                new String[] {"cmd", "/c", "dir2"}, Charset.forName("gbk"));
        Assert.fail("dir2 应该要执行失败，但却通过了，请查找原因");
    }

    @Test
    public void testProcessShell1() throws IOException {
        int errCode;
        errCode = ShellCommandExecUtil.runShellCommandSync("/tmp",
                new String[]{"cmd", "/c", "dir"}, Charset.forName("gbk"));
        Assert.assertEquals("进程返回码不正确", 0, errCode);

        String logPath = "/tmp/cmd.log";
        errCode = ShellCommandExecUtil.runShellCommandSync("/tmp",
                new String[]{"cmd", "/c", "dir"}, Charset.forName("gbk"), logPath);
        Assert.assertTrue("结果日志文件不存在", new File(logPath).exists());
    }

    @Test(expected = ShellProcessExecException.class)
    public void testProcessShell1WithErr() throws IOException {
        int errCode;
        errCode = ShellCommandExecUtil.runShellCommandSync("/tmp",
                new String[]{"cmd", "/c", "dir2"}, Charset.forName("gbk"));
        Assert.fail("dir2 应该要执行失败，但却通过了，请查找原因");
    }

    @Test(expected = ShellProcessExecException.class)
    public void testProcessShell1WithErr2() throws IOException {
        int errCode;
        String logPath = "/tmp/cmd2.log";
        try {
            errCode = ShellCommandExecUtil.runShellCommandSync("/tmp",
                    new String[]{"cmd", "/c", "dir2"}, Charset.forName("gbk"), logPath);
        }
        catch (ShellProcessExecException e) {
            e.printStackTrace();
            throw e;
        }
        Assert.assertTrue("结果日志文件不存在", new File(logPath).exists());
    }
}
```

