---
title: Netty 再回首
date: 2019-05-10 13:22:38
tags: Netty
toc: true
categories: Code

---

之前的工作上用过netty, 但是感觉理解上还是不够深刻, 便再次看了一下netty 模型和源码. 发现自己以前还是too naive

Netty 到底快在哪
Netty 为什么快
Netty 与Tomcat等web容器的区别

### Netty的模型 

以前第一次看netty的模型的时候, 以为自己看懂了. 其实并不然.

每次讨论到Netty模型的时候不可避免的会谈到IO模型的演变.

#### 什么是I/O
<!-- more -->
I/O : Output 和 Input. 任何计算机与存储介质或者计算机外界进行数据交换的行为都可以称之为IO操作. 例如文件读取, 网络请求等等. *一切的IO操作都是通过操作系统底层函数来实现的* 若要了解其底层原理, 需要了解计算机系统

Java早期版本中,通过流的概念实现了

* 磁盘操作: File
* 字节操作: InputStream 和 OutputStream
* 字符操作: Reader 和 Writer
* 网络操作: Socket

***流的概念很好理解: 将数据的传输想象成一条河流注入湖泊, 而湖泊就是目标对象***

#### I/O模式

* BIO(Blocking I/O)

    BIO是最早的I/O操作模型, 每一次I/O操作的时候, 例如写入一个文件: CPU将换从中的数据写入到磁盘需要等到写入完毕, 才会去执行下一个工作.

* AIO(Asynchronous I/O)


* NIO(Non-blocking I/O)
