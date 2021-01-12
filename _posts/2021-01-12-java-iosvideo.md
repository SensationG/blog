---
layout: post
title: java处理IOS无法播放视频流（Accept-Ranges）
date: 2021-01-12
Author: hhw
toc: true
comments: true
tags:  [Java,InputStream,OutputStream]

---

### 一、问题

在做小程序的在线播放视频功能时，遇到了安卓端、模拟器都可以正常播放，IOS却无法正常播放的问题。

### 二、排查

初步排查否定了referer拦截，经过仔细排查后，发现视屏链接不仅仅是在ios小程序无法播放，在safari浏览器中也无法正常播放，提示三角形+斜杠。初步认定为后端视屏流存在问题。 经过查阅资料，发现ios系统机制与其他两者均不同，因此对于视频输出流需要做特殊处理，处理完成后安卓、ios、模拟器均能正常播放视频。

### 三、解决

[参考博客的解决方案](https://blog.csdn.net/u010120886/article/details/79007001)，刚开始时代码返回的视频流是在一个请求里全部返回的，iOS会先发一次探测请求来获取文件大小，之后再发送多次请求来分段取数据流的数据，其实这里就是一个分段上传的思想（Accept-Ranges）。

有两个很重要的点就是：

第一：需要根据请求内容的不同做出不同的响应，第一次探测请求需要返回200，后面的请求需要返回206和具体数据

第二：contentType必须设置为video/mp4。

下面代码由博客提供，可直接使用：

⚠️从对象存储获取的inputStream需要先转File对象，可参考[上一篇](https://sensationg.github.io/blog/fileReadWrite/#%E6%96%87%E4%BB%B6%E6%93%8D%E4%BD%9C%E5%AE%9E%E4%BE%8B)

```java
private void sendVideo(HttpServletRequest request, HttpServletResponse response, File file, String fileName) throws FileNotFoundException, IOException {
		RandomAccessFile randomFile = new RandomAccessFile(file, "r");//只读模式
		long contentLength = randomFile.length();
        String range = request.getHeader("Range");
        int start = 0, end = 0;
        if(range != null && range.startsWith("bytes=")){
            String[] values = range.split("=")[1].split("-");
            start = Integer.parseInt(values[0]);
            if(values.length > 1){
                end = Integer.parseInt(values[1]);
            }
        }
        int requestSize = 0;
        if(end != 0 && end > start){
            requestSize = end - start + 1;
        } else {
            requestSize = Integer.MAX_VALUE;
        }
 
        byte[] buffer = new byte[4096];
        response.setContentType("video/mp4");
        response.setHeader("Accept-Ranges", "bytes");
        response.setHeader("ETag", fileName);
        response.setHeader("Last-Modified", new Date().toString());
        //第一次请求只返回content length来让客户端请求多次实际数据
        if(range == null){
            response.setHeader("Content-length", contentLength + "");
        }else{
        	//以后的多次以断点续传的方式来返回视频数据
            response.setStatus(HttpServletResponse.SC_PARTIAL_CONTENT);//206
            long requestStart = 0, requestEnd = 0;
            String[] ranges = range.split("=");
            if(ranges.length > 1){
                String[] rangeDatas = ranges[1].split("-");
                requestStart = Integer.parseInt(rangeDatas[0]);
                if(rangeDatas.length > 1){
                    requestEnd = Integer.parseInt(rangeDatas[1]);
                }
            }
            long length = 0;
            if(requestEnd > 0){
                length = requestEnd - requestStart + 1;
                response.setHeader("Content-length", "" + length);
                response.setHeader("Content-Range", "bytes " + requestStart + "-" + requestEnd + "/" + contentLength);
            }else{
                length = contentLength - requestStart;
                response.setHeader("Content-length", "" + length);
                response.setHeader("Content-Range", "bytes "+ requestStart + "-" + (contentLength - 1) + "/" + contentLength);
            }
        }
        ServletOutputStream out = response.getOutputStream();
        int needSize = requestSize;
        randomFile.seek(start);
        while(needSize > 0){
            int len = randomFile.read(buffer);
            if(needSize < buffer.length){
                out.write(buffer, 0, needSize);
            } else {
                out.write(buffer, 0, len);
                if(len < buffer.length){
                    break;
                }
            }
            needSize -= buffer.length;
        }
        randomFile.close();
        out.close();
		
	}
```

