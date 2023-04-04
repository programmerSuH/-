# Java IO简介

Java中数据读写方式：流

什么是流：一组有序的数据序列，将数据从一个地方带到另一个地方。

流的分类：输入（Input）流（将数据从输入设备读取到内存）、输出（Output）流（将内存中的数据写入输出设备）

## 数据源

> 数据源：提供数据的原始媒介，常见数据源：文件、数据库、IO设备

数据源分类：

- 原设备：为程序提供数据
- 目标设备：程序数据的输出位置



# Java IO类

在Java中，按数据流动方向，流可分为输入流和输出流；按流的输入输出方式，可划分为字节流和字符流。因此，按照不同组合方式，Java中的流可分为4种。

InputStream/OutputStream和Reader/Writer抽象类是所有IO流类的父类



## InputStream

该抽象类是所有字节输入流类的父类，InputStream是抽象类，不能被实例化。数据的读取单位为字节（1 byte = 8 bit）

继承体系：

![InputStream类的层次结构图](http://c.biancheng.net/uploads/allimg/200115/5-200115145253550.png)

常用方法：

| 名称                               | 作用                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| int read()                         | 从输入流读入一个 8 字节的数据，将它转换成一个 0~ 255 的整数，返回一个整数，如果遇到输入流的结尾返回 -1 |
| int read(byte[] b)                 | 从输入流读取若干字节的数据保存到参数 b 指定的字节数组中，返回的字节数表示读取的字节数，如果遇到输入流的结尾返回 -1 |
| int read(byte[] b,int off,int len) | 从输入流读取若干字节的数据保存到参数 b 指定的字节数组中，其中 off 是指在数组中开始保存数据位置的起始下标，len 是指读取字节的位数。返回的是实际读取的字节数，如果遇到输入流的结尾则返回 -1 |
| void close()                       | **关闭数据流，当完成对数据流的操作之后需要关闭数据流**       |
| int available()                    | 返回可以从数据源读取的数据流的位数。                         |
| skip(long n)                       | 从输入流跳过参数 n 指定的字节数目                            |
| boolean markSupported()            | 判断输入流是否可以重复读取，如果可以就返回 true              |
| void mark(int readLimit)           | 如果输入流可以被重复读取，从流的当前位置开始设置标记，readLimit 指定可以设置标记的字节数 |
| void reset()                       | 使输入流重新定位到刚才被标记的位置，这样可以重新读取标记过的数据 |



## OutputStream

OutputStream 类是所有字节输出流的超类，用于以二进制的形式将数据写入目标设备，该类是抽象类，不能被实例化。OutputStream 类提供了一系列跟数据输出有关的方法，如下所示。

| 名称                                 | 作用                                                     |
| ------------------------------------ | -------------------------------------------------------- |
| int write(b)                         | 将指定字节的数据写入到输出流                             |
| int write (byte[] b)                 | 将指定字节数组的内容写入输出流                           |
| int write (byte[] b,int off,int len) | 将指定字节数组从 off 位置开始的 len 字节的内容写入输出流 |
| close()                              | 关闭数据流，当完成对数据流的操作之后需要关闭数据流       |
| flush()                              | 刷新输出流，强行将缓冲区的内容写入输出流                 |



## Buffer介绍 

在Java中，直接从数据源或目的地读写数据的流称为**节点流**，不直接连接到数据源或目的地，而是对流进行处理，从而提高程序性能的流称为**处理流**。



# Java IO流体系

1. InputStream/OutputStream：字节流
2. Reader/Writer：字符流
3. FileInputStream/FileOutputStream：节点流，以字节为单位直接操作文件
4. FileReader/FileWriter：节点流，以字符为单位直接操作文本文件
5. ByteArrayInputStream/ByteArrayOutputStream：节点流：以字节为单位直接操作字节数组对象
6. ObjectInputStream/ObjectOutputStream：处理流，以字节为单位直接操作对象
7. DataInputStream/DataOutputStream：处理流，以字节为单位直接操作基本数据类型与字符串类型
8. BufferedInputStream/BufferedOutputStream：处理流，增加缓存功能，提高读写效率
9. BufferedReader/BufferedWriter：处理流，增加缓存功能，提高读写效率
10. InputStreamReader/OutputStreamWriter：处理流，将字节流转换成字符流
11. PrintStream：处理流，对OutputStream进行包装，方便输出字符



# 编码解码问题

查看系统默认编码

```java
System.getProperty("file.encoding");
```



## 纯汉字文本

字节数组大小设置：

- 采用GBK对纯汉字文本解码时，字节数组容量是2的倍数
- 采用UTF-8对纯汉字文本解码时，字节数组容量时3的倍数



原因分析：

- GBK编码，一个汉字是2个字节
- UTF-8编码，一个汉字是3个字节（生僻字可能占4个字节）

**即使编解码的字符集相同，但若字节数组的大小设置不正确，同样会导致乱码**

例：（字节数组大小设为3）

```java
File file = new File("src\main\java\io\1.txt");
InputStream inputStream = null;
try {
    inputStream = Files.newInputStream(file.toPath());
    byte[] bytes = new byte[3];
    int len = -1;
    StringBuffer str = new StringBuffer();
    while((len = inputStream.read(bytes)) != -1) {
        str.append(new String(bytes, 0, len, "gbk"));
    }
    System.out.println(str);
} catch (IOException e) {
    e.printStackTrace();
} finally {
    try {
        if (inputStream != null) {
            inputStream.close();
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
}

// 文本内容：’你好世界‘ 
// 输出： �?�?�?�?
```





## 中英混合文本

流的使用：

使用`FileInputStream`读取文件时，得到的字节数组无法采用

```java
String str = new String(bytes, 0, length, "CharacterSet")
```

的方式解码，应该改用字符转换流`InputStreamReader`



原因：

无论是GBK还是UTF-8，英文占1字节，当中英混杂时，会导致字节个数混乱，无法保证按字读取，导致乱码

例：

按上述方式，”hello world你好世界“解码后为："hello world?��好世�?"

### InputStreamReader读取

```java
File file = new File("src\\main\\java\\io\\2.txt");
InputStream inputStream = null;
InputStreamReader inputStreamReader = null;
try {
    inputStream = Files.newInputStream(file.toPath());
    inputStreamReader = new InputStreamReader(inputStream, "gbk");
    int len;
    StringBuffer str = new StringBuffer();
    while((len = inputStreamReader.read()) != -1) {
        str.append((char) len);
    }
    System.out.println(str);
} catch (IOException e) {
    e.printStackTrace();
} finally {
    try {
        if (inputStream != null) {
            inputStream.close();
        }
        if (inputStreamReader != null) {
            inputStreamReader.close();
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

### BufferedReader读取

由于InputStreamReader是一个一个字符读取的，效率低下，因此可以使用BufferedReader一次读取大量数据，减少IO次数，提高效率

```java
File file = new File("src\\main\\java\\io\\2.txt");
InputStream inputStream = null;
InputStreamReader inputStreamReader = null;
BufferedReader bufferedReader = null;
try {
    inputStream = Files.newInputStream(file.toPath());
    inputStreamReader = new InputStreamReader(inputStream, "gbk");
    bufferedReader = new BufferedReader(inputStreamReader);
    StringBuffer str = new StringBuffer();
    String line;
    while((line = bufferedReader.readLine()) != null) {
        str.append(line);
    }
    System.out.println(str);
} catch (IOException e) {
    e.printStackTrace();
} finally {
    try {
        if (inputStream != null) {
            inputStream.close();
        }
        if (inputStreamReader != null) {
            inputStreamReader.close();
        }
        if (bufferedReader != null) {
            bufferedReader.close();
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```



# File类



## 概念

- File类是Java提供针对磁盘中文件或目录转换成对象的包装类
- 一个File对象可以代表一个文件或目录，实现对文件和目录的创建、查找和删除

## File类的方法

| 序 号 | 方法描述                                                     |
| :---- | :----------------------------------------------------------- |
| 1     | public String getName() 返回由此抽象路径名表示的文件或目录的名称。 |
| 2     | public String getParent()、  返回此抽象路径名的父路径名的路径名字符串，如果此路径名没有指定父目录，则返回 `null`。 |
| 3     | public File getParentFile() 返回此抽象路径名的父路径名的抽象路径名，如果此路径名没有指定父目录，则返回 `null`。 |
| 4     | public String getPath() 将此抽象路径名转换为一个路径名字符串。 |
| 5     | public boolean isAbsolute() 测试此抽象路径名是否为绝对路径名。 |
| 6     | public String getAbsolutePath() 返回抽象路径名的绝对路径名字符串。 |
| 7     | public boolean canRead() 测试应用程序是否可以读取此抽象路径名表示的文件。 |
| 8     | public boolean canWrite() 测试应用程序是否可以修改此抽象路径名表示的文件。 |
| 9     | public boolean exists() 测试此抽象路径名表示的文件或目录是否存在。 |
| 10    | public boolean isDirectory() 测试此抽象路径名表示的文件是否是一个目录。 |
| 11    | public boolean isFile() 测试此抽象路径名表示的文件是否是一个标准文件。 |
| 12    | public long lastModified() 返回此抽象路径名表示的文件最后一次被修改的时间。 |
| 13    | public long length() 返回由此抽象路径名表示的文件的长度。    |
| 14    | public boolean createNewFile() throws IOException 当且仅当不存在具有此抽象路径名指定的名称的文件时，原子地创建由此抽象路径名指定的一个新的空文件。 |
| 15    | public boolean delete()  删除此抽象路径名表示的文件或目录。  |
| 16    | public void deleteOnExit() 在虚拟机终止时，请求删除此抽象路径名表示的文件或目录。 |
| 17    | public String[] list() 返回由此抽象路径名所表示的目录中的文件和目录的名称所组成字符串数组。 |
| 18    | public String[] list(FilenameFilter filter) 返回由包含在目录中的文件和目录的名称所组成的字符串数组，这一目录是通过满足指定过滤器的抽象路径名来表示的。 |
| 19    | public File[] listFiles()  返回一个抽象路径名数组，这些路径名表示此抽象路径名所表示目录中的文件。 |
| 20    | public File[] listFiles(FileFilter filter) 返回表示此抽象路径名所表示目录中的文件和目录的抽象路径名数组，这些路径名满足特定过滤器。 |
| 21    | public boolean mkdir() 创建此抽象路径名指定的目录。          |
| 22    | public boolean mkdirs() 创建此抽象路径名指定的目录，包括创建必需但不存在的父目录。 |
| 23    | public boolean renameTo(File dest)  重新命名此抽象路径名表示的文件。 |
| 24    | public boolean setLastModified(long time) 设置由此抽象路径名所指定的文件或目录的最后一次修改时间。 |
| 25    | public boolean setReadOnly() 标记此抽象路径名指定的文件或目录，以便只可对其进行读操作。 |
| 26    | public static File createTempFile(String prefix, String suffix, File directory) throws IOException 在指定目录中创建一个新的空文件，使用给定的前缀和后缀字符串生成其名称。 |
| 27    | public static File createTempFile(String prefix, String suffix) throws IOException 在默认临时文件目录中创建一个空文件，使用给定前缀和后缀生成其名称。 |
| 28    | public int compareTo(File pathname) 按字母顺序比较两个抽象路径名。 |
| 29    | public int compareTo(Object o) 按字母顺序比较抽象路径名与给定对象。 |
| 30    | public boolean equals(Object obj) 测试此抽象路径名与给定对象是否相等。 |
| 31    | public String toString()  返回此抽象路径名的路径名字符串。   |

























