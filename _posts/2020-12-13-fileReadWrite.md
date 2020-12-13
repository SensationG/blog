---
layout: post
title: 文件读取与写入&流操作
date: 2020-12-13
Author: hhw
toc: true
comments: true
tags:  [Java,InputStream,OutputStream]

---

### Java文件读写操作&响应文件流

> 本文记录文件流读写操作过程、将文件填入response返回给前端进行下载
>
> 本文只是简要地概述了输入输出流使用，详细的流操作和流概念等待之后再做笔记

#### 流的概念

流是一组有顺序的，有起点和终点的字节集合，是对数据传输的总称或抽象。即数据在两设备间的传输称为流，流的本质是数据传输，根据数据传输特性将流抽象为各种类，方便更直观的进行数据操作。

#### 流的分类

##### 字节流&字符流

字符流的由来： 因为数据编码的不同，而有了对字符进行高效操作的流对象<u>。本质其实就是基于字节流读取时，去查了指定的码表</u>。 字节流和字符流的区别：

- 读写单位不同：字节流以字节（8bit）为单位（1byte=8bit），字符流以字符为单位，根据码表映射字符，<u>一次可能读多个字节</u>。
- 处理对象不同：<u>字节流能处理所有类型的数据（如图片、avi等）</u>，而字符流只能处理字符类型的数据。
- 字节流：一次读入或读出是8位二进制。
- 字符流：一次读入或读出是16位二进制。

设备上的数据无论是图片或者视频，文字，它们都以二进制存储的。二进制的最终都是以一个8位为数据单元进行体现，所以计算机中的最小数据单元就是字节。意味着，字节流可以处理设备上的所有数据，所以字节流一样可以处理字符数据。

**结论：只要是处理纯文本数据，就优先考虑使用字符流。 除此之外都使用字节流。**

本文中以常用的图片处理为例子，使用字节流进行操作。

##### 输入流&输出流

> 这里以字节流为例子

**输入流**

进行读操作，所有的输入字节流都是 `InputStream` 的子类

常用的输入流： 

- `ByteArrayInputStream` ：从Byte 数组 读取数据 
- `StringBufferInputStream` ：从 StringBuffer 读取数据
- `FileInputStream` ： 从本地文件中读取数据。

其他：

- `PipedInputStream` 是从与其它线程共用的管道中读取数据，与Piped 相关的知识后续单独介绍。
- `ObjectInputStream` 和所有`FilterInputStream` 的子类都是装饰流（装饰器模式的主角）。

**输出流**

进行写操作，所有的输入字节流都是 `OutputStream` 的子类

常用输出流：

- `FileOutPutStream` : 向本地文件写入数据
- `ByteArrayOutputStream` ：向Byte数组写入数据

#### 文件操作实例

将文件输入流读取到文件输出流

使用场景：输入流转FIle对象、将文件流填入response返回给前端

**举例一：输入流转File对象**

这里的输入流是正在进行链接的HTTP响应，所以无法自此读取整个流。

```java
// 获取指定offset和length的"myobject"的输入流。
InputStream stream = minioClient.getObject(bucketName, objName);

// 用来装载正在进行的输入流
String tempFilePath = getTempFile();
File targetFile = new File(tempFilePath);
OutputStream outStream = new FileOutputStream(targetFile);

// 新建byte数组，用来存储输入流的数据（文件字节数据）
byte[] buffer = new byte[8 * 1024];
int bytesRead;

// 正在进行的链接的HTTP响应，此时可能无法一次读取整个流。这种情况下，我们需要确保一直读取到流的尽头。
/**
* 这里的 stream.read(buffer) 输入流的 read(byte[] b)方法，从输入流读取一些字节数，并将它们存储到
* 缓冲区 b（即将文件字节数据存储到数组buffer中），
* read方法返回值为本次读取到缓冲区的字节长度，当返回值等于 -1 时，表示读取完毕
**/
while ((bytesRead = stream.read(buffer)) != -1) {
  // 输出流的 write(byte[] b, int off, int len) 从指定的字节数组写入 len个字节，从偏移 off开始输		 出到此输出流。 即读取字符数组的数据到输出流
  outStream.write(buffer, 0, bytesRead);
}

files.add(targetFile);
stream.close();
```

简要概述：

将文件的输入流 通过 byte数组（中介）缓冲文件，然后输出流通过读取byte数组从而写入文件到输出流对象

重点：

- 输入流的 `int read(byte[] b)` 方法

  **参数**

  `b` - 读取数据的缓冲区。

  **结果**

  读取到缓冲区的总字节数，或者如果没有更多的数据，因为已经到达流的末尾，则是 `-1` 。

- 输出流的 `void write(byte[] b, int off, int len)` 方法

  **参数**

  `b` - 数据。

  `off` - 数据中的起始偏移量。

  `len` - 要写入的字节数。

**举例二：文件File对象填入HttpServletResponse的输出流**

```java
try {
  // 获取文件路径
  File file = new File(filePath);
  // 获取文件类型（文件后缀 例如 .xls）
  String ext = CMyFile.extractFileExt(filePath);
  // 从HttpServletRequest获取内容(文件)类型
  String mime = request.getContentType();
  if (mime == null) {
    mime = request.getSession().getServletContext().getMimeType(file.getName());
    if (mime == null) {
      mime = "application/octet-stream";
    }
  }
  // 设置响应的文件类型，浏览器读取时需要以此决定如何读取这个文件 
  // （经测试，前端也可以直接设置，不需要读取后端设置的contentType）
  response.setContentType(mime);
  response.setContentLength((int) file.length());
  // 设置头
  String name = "评审图片" + DateUtils.getNowDate() +"."+ext;
  response.setHeader("Content-Disposition", "attachment;filename=" + new String(name.getBytes("GB2312"), "ISO-8859-1"));

  InputStream input = null;
  OutputStream output = null;
  try {
    // 使用 BufferedInputStream 缓冲流 增加缓冲功能，避免频繁读写硬盘
    input = new BufferedInputStream(new FileInputStream(file));
    // 从HttpServletResponse 读取输出流
    output = response.getOutputStream();
    byte[] buffer = new byte[4096];
    // 将输入流填入响应的输出流
    for (;;) {
      int n = input.read(buffer);
      if (n == (-1)) {
        break;
      }
      output.write(buffer, 0, n);
    }
    output.flush();
  } finally {
    // 要在finally中关闭流
    if (input != null) {
      input.close();
    }
    if (output != null) {
      output.close();
    }
  }
} catch (Exception e) {
  e.printStackTrace();
}
```

前端接收：

```js
axios({
  url: process.env.VUE_APP_BASE_API + '/repository/photo/downloadCompressionPhoto',
  method: 'get',
  params: { photoExportId: photoExportId },
  responseType: "blob",
  headers: { 'Authorization': 'Bearer ' + getToken() }
}).then(response => {

  let zipName = "图片库导出文件.zip";
  let blob = new Blob([response.data], {type: "application/zip"}); // 下载格式为zip
  if ("download" in document.createElement("a")) {
    let elink = document.createElement("a"); // 创建一个<a>标签
    elink.style.display = "none"; // 隐藏标签
    elink.href = window.URL.createObjectURL(blob); // 配置href
    elink.download = zipName;
    elink.click();
    URL.revokeObjectURL(elink.href); // 释放URL 对象
  }

}).catch(err => {
  console.log(err)
  this.msgError('导出失败')
})
```



