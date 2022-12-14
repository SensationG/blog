# 多文件压缩Zip下载（CompressDownloadUtil的使用）

# 一、使用步骤
使用预先准备的`CompressDownloadUtil`工具类（在资源库-文件相关中有保存）
[CompressDownloadUtil.java](https://www.yuque.com/attachments/yuque/0/2022/java/27216901/1660910289185-3c91bc7e-fcc6-423b-95c1-d97ee265931a.java)
主要步骤：

1. 根据路径查找文件，得到File对象
2. 将File放入List中
3. 使用工具类设置响应头
4. 使用工具类进行压缩打包

使用示例如下：

```java
@PostMapping
public void downloadallfiles(String attachfilesIds, String downName, HttpService service, HttpServletResponse response) {
    When.emptyReturnErrorCode(attachfilesIds, AttachfilesErrorCode.ATTACHFILES_ID_IS_EMPTY);
    // 文件id
    String[] ids = attachfilesIds.split(",");
    // 获取文件转换成File对象
    List<File> files = new ArrayList<>();
    for (String id : ids) {
        Attachfiles attachfiles = attachfilesService.get(id);
        if (attachfiles != null) {
            // File对象
            File file = service.getFilePath(attachfiles.getFilepath());
            files.add(file);
        }
    }

    //-- 响应头的设置【调用工具类】
    String downloadName = StringUtils.isNotBlank(downName) ? downName : UidGeneratorUtil.getUID() + ".zip";
    response = CompressDownloadUtil.setDownloadResponse(response, downloadName);

    try {
        //-- 将多个文件压缩写进响应的输出流【调用工具类】
        CompressDownloadUtil.compressZip(files, response.getOutputStream());
    } catch (IOException e) {
		// .....异常处理
    }
}
```

# 二、工具类代码截取
CompressDownload部分方法截取：

```java
 /**
     * 将多个文件压缩到指定输出流中
     *
     * @param files 需要压缩的文件列表
     * @param outputStream  压缩到指定的输出流
     * @date 2018年9月7日 下午3:11:59
     */
public static void compressZip(List<File> files, OutputStream outputStream) {
    ZipOutputStream zipOutStream = null;
    try {
        //-- 包装成ZIP格式输出流
        zipOutStream = new ZipOutputStream(new BufferedOutputStream(outputStream));
        // -- 设置压缩方法
        zipOutStream.setMethod(ZipOutputStream.DEFLATED);
        //-- 将多文件循环写入压缩包
        for (int i = 0; i < files.size(); i++) {
            File file = files.get(i);
            FileInputStream filenputStream = new FileInputStream(file);
            byte[] data = new byte[(int) file.length()];
            filenputStream.read(data);
            //-- 添加ZipEntry，并ZipEntry中写入文件流，这里，加上i是防止要下载的文件有重名的导致下载失败,file.getName()这里也可以自定义文件名称
            zipOutStream.putNextEntry(new ZipEntry(i + file.getName()));
            zipOutStream.write(data);
            filenputStream.close();
            zipOutStream.closeEntry();
        }
    } catch (IOException e) {
    }  finally {
        try {
            if (Objects.nonNull(zipOutStream)) {
                zipOutStream.flush();
                zipOutStream.close();
            }
            if (Objects.nonNull(outputStream)) {
                outputStream.close();
            }
        } catch (IOException e) {

        }
    }
}

/**
     * 设置下载响应头
     *
     * @param response
     * @return
     * @date 2018年9月7日 下午3:01:59
     */
public static HttpServletResponse setDownloadResponse(HttpServletResponse response, String downloadName) {
    response.reset();
    response.setCharacterEncoding("utf-8");
    response.setContentType("application/octet-stream");
    response.setHeader("Content-Disposition", "attachment;fileName*=UTF-8''"+ downloadName);
    return response;
}
```
