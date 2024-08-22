---
layout: post
title: BufferedReader使用记录
categories: [Java]
permalink: java/bufferedreader.html
---

BufferedReader使用记录

一种以缓冲方式读取字符IO流，可以指定缓冲空间大小，默认为8192 

常见读取有如下三种方式：

#### 1、readline()
按行读取，解析IO流中的`\n`、`\r`、`\r\n`作为读取结束依据，不会解析字符串中的`\n`、`\r`    
以普通文件为例：按照回车符作为每一行的结束  
以`shell`或`python`脚本为例：以每一个`echo`或者`print`或者`log`输出结束  
示例：  
```java
String buffer;
while ((buffer = reader.readLine()) != null) {
    System.out.println(buffer);
}
```

#### 2、read()
一个字节一个的读取缓冲区，直到最后一个字节读取完成  
示例：  
```java
int size = 0;
while ((size = reader.read()) != -1) {
    // 此处使用println时，会在每一个字符后换行，不是想要的结果
    System.out.print((char)size);
}
```

#### 3、read(char[] targetCharArray, int off, int length)
`targetCharArray`: 将读取到的内容存储到该字符数组，如果读取内容小于字符数组大小，会默认填充值，需要做处理  
`off`: 偏移量，从什么位置开始读取，一般设置为0，从原点读取，每次完整的读取一个`char`数组  
`length`：从缓冲区读取大小  
示例：  
```java
int size = 0;
char[] readChar = new char[1024];  // 字符数组大小自定义
String buffer = "";
while ((size = reader.read(readChar, 0, readChar.length)) != -1) {
    // 读取到的内容小于自定义字符数组，需要去除默认填充值
    if (size < readChar.length) {
        char[] newReaderChar = Arrays.copyOf(readChar, size);
        buffer = String.valueOf(newReaderChar);
    } else {
        buffer = String.valueOf(readChar);
    }
    System.out.println(buffer);
}
```

完整示例请访问：[BufferedReader](https://github.com/duwei0227/java-demo/blob/master/src/main/java/top/probiecoder/BufferedReaderDemo.java)

**如有理解不对之处，请指出，谢谢。**