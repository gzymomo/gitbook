[Java实现解压缩文件和文件夹](https://www.cnblogs.com/luciochn/p/14515029.html)

# 一  前言

项目开发中，总会遇到解压缩文件的时候。比如，用户下载多个文件时，服务端可以将多个文件压缩成一个文件（例如xx.zip或xx.rar）。用户上传资料时，允许上传压缩文件，服务端进行解压读取每一个文件。

基于通用性，以下介绍几种解压缩文件的方式，包装成工具类，供平时开发使用。

# 二  压缩文件

压缩文件，顾名思义，即把一个或多个文件压缩成一个文件。压缩也有2种形式，一种是将所有文件压缩到同一目录下，此种方式要注意文件重名覆盖的问题。另一种是按原有文件树结构进行压缩，即压缩后的文件树结构保持不变。

压缩文件操作，会使用到一个类，即ZipOutputStream。

## 2.1  压缩多个文件

此方法将所有文件压缩到同一个目录下。方法传入多个文件列表，和一个最终压缩到的文件路径名。

```java
    /**
     * 压缩多个文件，压缩后的所有文件在同一目录下
     * 
     * @param zipFileName 压缩后的文件名
     * @param files 需要压缩的文件列表
     * @throws IOException IO异常
     */
    public static void zipMultipleFiles(String zipFileName, File... files) throws IOException {
        ZipOutputStream zipOutputStream = null;
        try {
            // 输出流
            zipOutputStream = new ZipOutputStream(new FileOutputStream(zipFileName));
            // 遍历每一个文件，进行输出
            for (File file : files) {
                zipOutputStream.putNextEntry(new ZipEntry(file.getName()));
                FileInputStream fileInputStream = new FileInputStream(file);
                int readLen;
                byte[] buffer = new byte[1024];
                while ((readLen = fileInputStream.read(buffer)) != -1) {
                    zipOutputStream.write(buffer, 0, readLen);
                }
                // 关闭流
                fileInputStream.close();
                zipOutputStream.closeEntry();
            }
        } finally {
            if (null != zipOutputStream) {
                try {
                    zipOutputStream.close();
                } catch (IOException ex) {
                    ex.printStackTrace();
                }
            }
        }
    }
```

测试，将D盘下的infp.txt和infp1.txt文件压缩到D盘下，压缩文件名为my.zip。

```java
    public static void main(String[] args) throws Exception {
        zipMultipleFiles("D:/my.zip", new File("D:/infp.txt"), new File("D:/infp1.txt"));
    }
```

## 2.2  压缩文件或文件树

此方法将文件夹下的所有文件按原有的树形结构压缩到文件中，也支持压缩单个文件。原理也简单，无非就是递归遍历文件树中的每一个文件，进行压缩。有个注意的点每一个文件的写入路径是基于压缩文件位置的相对路径。

```java
package com.nobody.zip;

import java.io.*;
import java.util.zip.ZipEntry;
import java.util.zip.ZipInputStream;
import java.util.zip.ZipOutputStream;

public class ZipUtils {

    /**
     * 压缩文件或文件夹（包括所有子目录文件）
     *
     * @param sourceFile 源文件
     * @param format 格式（zip或rar）
     * @throws IOException 异常信息
     */
    public static void zipFileTree(File sourceFile, String format) throws IOException {
        ZipOutputStream zipOutputStream = null;
        try {
            String zipFileName;
            if (sourceFile.isDirectory()) { // 目录
                zipFileName = sourceFile.getParent() + File.separator + sourceFile.getName() + "."
                        + format;
            } else { // 单个文件
                zipFileName = sourceFile.getParent()
                        + sourceFile.getName().substring(0, sourceFile.getName().lastIndexOf("."))
                        + "." + format;
            }
            // 压缩输出流
            zipOutputStream = new ZipOutputStream(new FileOutputStream(zipFileName));
            zip(sourceFile, zipOutputStream, "");
        } finally {
            if (null != zipOutputStream) {
                // 关闭流
                try {
                    zipOutputStream.close();
                } catch (IOException ex) {
                    ex.printStackTrace();
                }
            }
        }
    }

    /**
     * 递归压缩文件
     * 
     * @param file 当前文件
     * @param zipOutputStream 压缩输出流
     * @param relativePath 相对路径
     * @throws IOException IO异常
     */
    private static void zip(File file, ZipOutputStream zipOutputStream, String relativePath)
            throws IOException {

        FileInputStream fileInputStream = null;
        try {
            if (file.isDirectory()) { // 当前为文件夹
                // 当前文件夹下的所有文件
                File[] list = file.listFiles();
                if (null != list) {
                    // 计算当前的相对路径
                    relativePath += (relativePath.length() == 0 ? "" : "/") + file.getName();
                    // 递归压缩每个文件
                    for (File f : list) {
                        zip(f, zipOutputStream, relativePath);
                    }
                }
            } else { // 压缩文件
                // 计算文件的相对路径
                relativePath += (relativePath.length() == 0 ? "" : "/") + file.getName();
                // 写入单个文件
                zipOutputStream.putNextEntry(new ZipEntry(relativePath));
                fileInputStream = new FileInputStream(file);
                int readLen;
                byte[] buffer = new byte[1024];
                while ((readLen = fileInputStream.read(buffer)) != -1) {
                    zipOutputStream.write(buffer, 0, readLen);
                }
                zipOutputStream.closeEntry();
            }
        } finally {
            // 关闭流
            if (fileInputStream != null) {
                try {
                    fileInputStream.close();
                } catch (IOException ex) {
                    ex.printStackTrace();
                }
            }
        }
    }

    public static void main(String[] args) throws Exception {
        String path = "D:/test";
        String format = "zip";
        zipFileTree(new File(path), format);
    }
}
```

上例将test目录下的所有文件压缩到同一目录下的test.zip文件中。

## 2.3 借助文件访问器压缩

还有一种更简单的方式，我们不自己写递归遍历。借助Java原生类，SimpleFileVisitor，它提供了几个访问文件的方法，其中有个方法visitFile，对于文件树中的每一个文件（文件夹除外），都会调用这个方法。我们只要写一个类继承SimpleFileVisitor，然后重写visitFile方法，实现将每一个文件写入到压缩文件中即可。

当然，除了visitFile方法，它里面还有preVisitDirectory，postVisitDirectory，visitFileFailed等方法，通过方法名大家也猜出什么意思了。

```java
package com.nobody.zip;

import java.io.FileOutputStream;
import java.io.IOException;
import java.nio.file.*;
import java.nio.file.attribute.BasicFileAttributes;
import java.util.zip.ZipEntry;
import java.util.zip.ZipOutputStream;

/**
 * @Description
 * @Author Mr.nobody
 * @Date 2021/3/8
 * @Version 1.0.0
 */
public class ZipFileTree extends SimpleFileVisitor<Path> {

    // zip输出流
    private ZipOutputStream zipOutputStream;
    // 源目录
    private Path sourcePath;

    public ZipFileTree() {}

    /**
     * 压缩目录以及所有子目录文件
     *
     * @param sourceDir 源目录
     */
    public void zipFile(String sourceDir) throws IOException {
        try {
            // 压缩后的文件和源目录在同一目录下
            String zipFileName = sourceDir + ".zip";
            this.zipOutputStream = new ZipOutputStream(new FileOutputStream(zipFileName));
            this.sourcePath = Paths.get(sourceDir);

            // 开始遍历文件树
            Files.walkFileTree(sourcePath, this);
        } finally {
            // 关闭流
            if (null != zipOutputStream) {
                zipOutputStream.close();
            }
        }
    }

    // 遍历到的每一个文件都会执行此方法
    @Override
    public FileVisitResult visitFile(Path file, BasicFileAttributes attributes) throws IOException {
        // 取相对路径
        Path targetFile = sourcePath.relativize(file);
        // 写入单个文件
        zipOutputStream.putNextEntry(new ZipEntry(targetFile.toString()));
        byte[] bytes = Files.readAllBytes(file);
        zipOutputStream.write(bytes, 0, bytes.length);
        zipOutputStream.closeEntry();
        // 继续遍历
        return FileVisitResult.CONTINUE;
    }

    // 遍历每一个目录时都会调用的方法
    @Override
    public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs)
            throws IOException {
        return super.preVisitDirectory(dir, attrs);
    }

    // 遍历完一个目录下的所有文件后，再调用这个目录的方法
    @Override
    public FileVisitResult postVisitDirectory(Path dir, IOException exc) throws IOException {
        return super.postVisitDirectory(dir, exc);
    }

    // 遍历文件失败后调用的方法
    @Override
    public FileVisitResult visitFileFailed(Path file, IOException exc) throws IOException {
        return super.visitFileFailed(file, exc);
    }

    public static void main(String[] args) throws IOException {
        // 需要压缩源目录
        String sourceDir = "D:/test";
        // 压缩
        new ZipFileTree().zipFile(sourceDir);
    }
}
```

# 三  解压文件

解压压缩包，借助ZipInputStream类，可以读取到压缩包中的每一个文件，然后根据读取到的文件属性，写入到相应路径下即可。对于解压压缩包中是文件树的结构，每读取到一个文件后，如果是多层路径下的文件，需要先创建父目录，再写入文件流。

```java
package com.nobody.zip;

import java.io.*;
import java.util.zip.ZipEntry;
import java.util.zip.ZipInputStream;
import java.util.zip.ZipOutputStream;

/**
 * @Description 解压缩文件工具类
 * @Author Mr.nobody
 * @Date 2021/3/8
 * @Version 1.0.0
 */
public class ZipUtils {

    /**
     * 解压
     * 
     * @param zipFilePath 带解压文件
     * @param desDirectory 解压到的目录
     * @throws Exception
     */
    public static void unzip(String zipFilePath, String desDirectory) throws Exception {

        File desDir = new File(desDirectory);
        if (!desDir.exists()) {
            boolean mkdirSuccess = desDir.mkdir();
            if (!mkdirSuccess) {
                throw new Exception("创建解压目标文件夹失败");
            }
        }
        // 读入流
        ZipInputStream zipInputStream = new ZipInputStream(new FileInputStream(zipFilePath));
        // 遍历每一个文件
        ZipEntry zipEntry = zipInputStream.getNextEntry();
        while (zipEntry != null) {
            if (zipEntry.isDirectory()) { // 文件夹
                String unzipFilePath = desDirectory + File.separator + zipEntry.getName();
                // 直接创建
                mkdir(new File(unzipFilePath));
            } else { // 文件
                String unzipFilePath = desDirectory + File.separator + zipEntry.getName();
                File file = new File(unzipFilePath);
                // 创建父目录
                mkdir(file.getParentFile());
                // 写出文件流
                BufferedOutputStream bufferedOutputStream =
                        new BufferedOutputStream(new FileOutputStream(unzipFilePath));
                byte[] bytes = new byte[1024];
                int readLen;
                while ((readLen = zipInputStream.read(bytes)) != -1) {
                    bufferedOutputStream.write(bytes, 0, readLen);
                }
                bufferedOutputStream.close();
            }
            zipInputStream.closeEntry();
            zipEntry = zipInputStream.getNextEntry();
        }
        zipInputStream.close();
    }

    // 如果父目录不存在则创建
    private static void mkdir(File file) {
        if (null == file || file.exists()) {
            return;
        }
        mkdir(file.getParentFile());
        file.mkdir();
    }

    public static void main(String[] args) throws Exception {
        String zipFilePath = "D:/test.zip";
        String desDirectory = "D:/a";
        unzip(zipFilePath, desDirectory);
    }
}
```

# 四  总结

- 在解压缩文件过程中，主要是对流的读取操作，注意进行异常处理，以及关闭流。
- web应用中，通过接口可以实现文件上传下载，对应的我们只要把压缩后的文件，写入到response.getOutputStream()输出流即可。
- 解压缩文件时，注意空文件夹的处理。

> 此演示项目已上传到Github，如有需要可自行下载，欢迎 Star 。 https://github.com/LucioChn/common-utils