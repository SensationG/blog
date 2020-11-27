---
layout: post
title: Minio图片批量打包下载
date: 2020-11-27
Author: hhw
toc: true
comments: true
tags:  [存储,IO]
---

# minio对象存储图片批量打压缩包下载

- 文章关键字：MINIO、图片、批量下载、压缩包
- 关键对象：`File` `InputStream` `OutputStream`  `FileOutputStream`    `MinioClient` `CheckedOutputStream` `ZipOutputStream`
- **思路**：首先遍历文件名称，去MinIO中获取文件输入流 => 将输入流写入到服务器的临时文件.tmp(一个文件对应一个临时文件，这里因为是正在进行的链接的HTTP响应，此时可能无法一次读取整个流，所以需要一直读取，这段时间非常耗时) => 获取文件的File对象，填入Zip输出流，保存Zip文件在服务器 => 返回Zip文件给前端，用户下载
- 由于获取Minio中文件的响应非常耗时，后期将改为后台执行，使用一张表记录执行情况，而用户在此期间可以去操作页面的其他功能，待执行完后用户可返回下载页面下载。



等待更新。。。