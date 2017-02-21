---
title: Java IO流初探
date: 2017-01-11 16:11:34
tags:
---

### Java IO流 初探

最近学习安卓，发现数据存储应用了Java的IO，想着本就不熟悉，故借此机会深入学习一番，争取变得更好。

#### 流
流（Stream）是一个抽象概念，指代数据在计算机中各部件的流动。部件可以是文件，网络，内存等。流具有方向性，程序使用输入流读取数据，使用输出流存写数据。也就是说如果数据从程序流向设备，即为输出流，反之为输入流。、

#### 流的分类
* 根据处理数据的不同：`字节流`和`字符流`
* 数据流向不同： `输入流`和`输出流`
* 操作功能不同：`节点流` 和 `处理流`
节点流：用来从数据源读入数据或往目的地写出数据
处理流：对数据执行某崇操作

<!-- more -->
#### 字符流
字符流处理的基本单元是Unicode编码的数据，便于对字符数据的操作，`Reader`和`Writer`是`java.io`包中所有字符流的父类，因他们都是抽象类，故应使用其子类创建实体对象。`Reader`和`Writer`的子类又可分为节点流和处理流。
##### Reader
面向字符的输入流都继承于`Reader`
![Alt text](/assets/images/Java_IO_Stream/Reader.jpg)

| 类名                    | 功能 |
| ---------------------  | ---- |
| CharArrayReader    | 从字符数组读取的输入流 |
| BufferedReader     | 缓冲输入字符流 |
| PipedReader        | 管道输入字符流 |
| InputStreamReader  | 字节向字符转换的输入流 |
| FileReader         | 文件读取输入字符流 |
| FilterReader       | 过滤输入字符流 |
| StringReader       | 字符串读取输入字符流 |
| LineNumberReader   | 附加行号输入字符流 |
| PushbackReader     | 返回一个字符并把字符放回输入流 |
###### 使用FileReader类读取文件
FileReader 类是 Reader 子类 InputStreamReader 类的子类，因此 FileReader 类既可以使用Reader 类的方法也可以使用 InputStreamReader 类的方法来创建对象。本例使用字符串构造方法。
``` java
 FileReader fileReader = new FileReader("example.txt");
 char str[] = new char[1000];
 int num = fileReader.read(str);
 String string = new String(str, 0, num);
 System.out.print(string);
```

###### 使用BufferedReader类读取文件
BufferedReader 类是用来读取缓冲区中的数据。使用时必须创建 FileReader 类对象，再以该对象为参数创建 BufferedReader 类的对象
```java
FileReader fileReader = new FileReader("example.txt");
BufferedReader bufferedReader = new BufferedReader(fileReader);
```

##### Writer
面向字符的输出流都继承于`Writer`
![Alt text](/assets/images/Java_IO_Stream/Writer.jpg)
 Writer类及其子类与Reader相似，故略过。
 * 记得使用BufferedReader类是，要用flush()方法将缓冲区清空


#### 字节流
字节流以字节为传输单位，用来读写8位的数据，除了能够处理纯文本文件之外，还能用来处理二进制文件的数据。 InputStream类和OutputStream类是所有字节流的父类。
#### InputStream
面向字节的输入流都是继承于InputStream类
![Alt text](/assets/images/Java_IO_Stream/InputStream.png)

| 类名                    |      功能       |
| ---------------------  |       ----      |
| FileInputStream        | 文件读取输入字节流  |
| PipedInputStream       | 管道输入字节流      |
| FilterInputStream      | 过滤输入流 |
| ByteArrayInputStream   | 从字节数组读取的输入流 |
| SequenceInputStream    | 两个或多个输入流的联合输入流，按顺序读取 |
| ObjectInputStream      | 对象的输入流 |
| LineNumberInputStream  | 附加行号输入流 |
| DataInputStream        | 包含读取Java标准数据类型的输入流  |
| BufferedInputStream    | 缓冲输入流 |
| PushbackInputStream    | 返回一个字节并把字节放回输入流 |

OutputStream
面向字节的输入流都是继承于OutputStream类
![Alt text](/assets/images/Java_IO_Stream/OutputStream.jpg)

#####  输出输入流
文件输入输出流 FileInputStream 和 FileOutputStream 负责完成对本地磁盘文件的顺序输入输出操作。
###### 实现图片文件的复制
``` java
FileInputStream fileInputStream = new FileInputStream("binary_file_copy.png");
FileOutputStream fileOutputStream = new FileOutputStream("binary_file_copy2.png");
byte[] bytes = new byte[fileInputStream.available()];
fileInputStream.read(bytes);
fileOutputStream.write(bytes);
System.out.println("文件已被更名复制");
fileInputStream.close();
fileOutputStream.close();
```

目前对于流就整理到这里，关于本文中所用得例子都在本地存放，希望自己在迷糊时能够回顾下。关于IO 中File以及RandomAccessFile会在后续整理。