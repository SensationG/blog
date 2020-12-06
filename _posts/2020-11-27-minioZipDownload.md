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

- 文章关键字：MINIO、图片、批量下载、压缩包、SpringBoot异步处理任务

- 关键对象：`File` `InputStream` `OutputStream`  `FileOutputStream`    `MinioClient` `CheckedOutputStream` `ZipOutputStream`

- **总体思路**：首先遍历文件名称，去MinIO中获取文件输入流 => 将输入流写入到服务器的临时文件.tmp(一个文件对应一个临时文件，这里因为是正在进行的链接的HTTP响应，此时可能无法一次读取整个流，所以需要一直读取，这段时间非常耗时) => 获取文件的File对象，填入Zip输出流，保存Zip文件在服务器 => 返回Zip文件给前端，用户下载

  - 遍历获取minio文件路径，通过minio获取这些文件的输入流

  - 将每个文件的 <u>输入流</u> 转为 => <u>File对象</u>（每个文件需要对应一个临时文件存储在服务器本地.tmp）

    这里输入流对应正在进行的HTTP响应，此时无法一下子读取整个流，这种情况下，需要保证一直读取到流的尽头。 

  - 将File对象，填入 ZipOutputStream 对象(Zip文件)，压缩文件，并可指定文件夹存储

  - 返回Zip路径给前端用户下载

- 由于文件的读取与写入十分耗时，所以这里使用<u>SpringBoot异步任务</u>进行处理，使用一张表记录执行结果与目标文件路径，而用户在此期间可以去操作页面的其他功能，待执行完后用户可返回下载页面下载。

- [SpringBoot异步任务参考](https://blog.csdn.net/weixin_39800144/article/details/79046237)
- [输入流转File参考](https://www.cnblogs.com/asfeixue/p/9065681.html)

### 一、开启异步任务

这里采用springBoot自身的一种异步方式，使用注解实现，非常方便，我们在想要异步执行的方法上加上**@Async**注解，在controller上加上**@EnableAsync**，即可。注意，这里的异步方法，只能在自身之外调用，在本类调用是无效的。

> Controller

```java
@RestController
@RequestMapping("/repository/photo")
@EnableAsync
public class CulturePhotoController extends BaseController
{
  private final org.slf4j.Logger logger = LoggerFactory.getLogger(getClass());
	@Autowired
  private LoginService loginService;
  /**
    * 异步处理2：使用springBoot自带async注解
   */
  @RequestMapping(value = "test1",method = RequestMethod.GET)
  public String test1(){
    loginService.getTest1();
    logger.info("============>"+Thread.currentThread().getName());
    return "异步,正在解析......";
  }
}
```

> Service 实现

```java
/**异步方法
  * 有@Async注解的方法，默认就是异步执行的，会在默认的线程池中执行，但是此方法不能在本类调用；启动类需添加直接开启异步执行@EnableAsync。 （实测Controller加就可以）
* */
@Async
@Override
public void compressionPhoto(Long[] photoIds, CulturePhoto culturePhoto) {
   // ...
  	logger.info(Thread.currentThread().getName()+"----------异步：>"+i);
}
```

![image-20201206202343257](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20201206203853.png)

看控制台，会发现，页面发出请求后，主线程会返回，而内置的线程池会新开线程，在后台执行任务。此时页面不用等待，可以继续其他操作。

### 二、处理压缩图片请求

> controller
>
> 接收需要压缩的图片Id

```java
/**
 * 压缩图片
 */
@GetMapping("/compressionPhoto/{photoIds}")
public AjaxResult compressionPhoto(@PathVariable("photoIds") Long[] photoIds, HttpServletRequest request, HttpServletResponse response)
{
    CulturePhoto culturePhoto = new CulturePhoto();
    culturePhoto.setCreateUser(SecurityUtils.getUsername());
    culturePhoto.setCreateTime(DateUtils.getNowDate());
  	// 进入Service进行处理
    culturePhotoService.compressionPhoto(photoIds, culturePhoto);
    return AjaxResult.success();
}
```

> Service 
>
> 将导出记录存储到表中，调用工具类执行文件压缩

```java
@Async
@Override
public void compressionPhoto(Long[] photoIds, CulturePhoto culturePhoto) {

    logger.info(Thread.currentThread().getName()+"----------异步任务开始：>");
    // 存储图片文件名 组装下载路径
    List<String> fileNames = new ArrayList<>();
    // 遍历获取图片链接
    for (Long photoId: photoIds) {
        CulturePhoto photo = selectCulturePhotoById(photoId);
        fileNames.add(photo.getPhoto());
    }

    // 导出记录
    String zipFileName = FileUtil.getZipFileName();
    culturePhoto.setOriginalFile(zipFileName);
    culturePhotoMapper.insertExportRecord(culturePhoto);

    try {
      	// 文件压缩
        FileUtil.photoCompress(fileNames, zipFileName, culturePhoto);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

> 工具类
>
> 通过minio获取文件输入流 =>  输入流转File对象 => File对象Zip输出流 => 执行完成更新数据库

```java
public static void photoCompress(List<String> fileNames, String zipFileName, CulturePhoto culturePhoto) throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {

    logger.info(Thread.currentThread().getName()+"----------压缩任务开始");

    String bucketName = FileUtil.picoriginalBucketName;
    try {
        MinioClient minioClient = new MinioClient(FileUploadConfig.getOssUrl(), FileUploadConfig.getAccessKey(), FileUploadConfig.getSecretKey());

        // 文件集合
        List<File> files = new ArrayList<File>();

        for (String fileName: fileNames) {

            String objName = fileName.substring(0, 8) + "/" + fileName.substring(0, 10) + "/" + fileName;
            minioClient.statObject(bucketName, objName);

            // 获取指定offset和length的"myobject"的输入流。
            InputStream stream = minioClient.getObject(bucketName, objName);

            // 用来装载正在进行的输入流
            String tempFilePath = getTempFile();
            File targetFile = new File(tempFilePath);
            OutputStream outStream = new FileOutputStream(targetFile);

            byte[] buffer = new byte[8 * 1024];
            int bytesRead;
            // 正在进行的链接的HTTP响应，此时可能无法一次读取整个流。这种情况下，我们需要确保一直读取到流的尽头。
            while ((bytesRead = stream.read(buffer)) != -1) {
                outStream.write(buffer, 0, bytesRead);
            }
            files.add(targetFile);
            stream.close();
        }

        FileOutputStream fileOutputStream = new FileOutputStream(zipFileName);
        CheckedOutputStream cos = new CheckedOutputStream(fileOutputStream,
                new CRC32());
        ZipOutputStream out = new ZipOutputStream(cos);
        out.setEncoding("gbk");

        // 尝试压缩
        out.putNextEntry(new ZipEntry("照片" + "/"));
        for(int i=0; i < files.size(); i++){
            out.putNextEntry(new ZipEntry("照片"+"/" + fileNames.get(i)));
            FileInputStream in = new FileInputStream(files.get(i));
            BufferedInputStream bi = new BufferedInputStream(in);
            //int b;
            byte buf[] = new byte[1024];
            int len;
            while ((len = bi.read(buf, 0, buf.length)) != -1) {
                // 将字节流写入当前zip目录
                out.write(buf,0,len);
            }
            bi.close();
            in.close();
            // 删除临时文件
            files.get(i).delete();
        }

        logger.info(Thread.currentThread().getName()+"----------压缩任务结束");

      	// 这里不能使用注解注入对象，会报空指针异常，需要使用反射去Service实现类执行数据库更新操作
        culturePhoto.setStatus("1");
        Object obj = SpringContextUtil.getBean("culturePhotoServiceImpl");
        Class class2 = obj.getClass();
        Method method = class2.getDeclaredMethod("updateExportRecord", CulturePhoto.class);
        method.invoke(obj, culturePhoto);

        out.close();
    } catch (Exception e) {
      	// 执行出错 更新导出记录表状态
        culturePhoto.setStatus("2");
        Object obj = SpringContextUtil.getBean("culturePhotoServiceImpl");
        Class class2 = obj.getClass();
        Method method = class2.getDeclaredMethod("updateExportRecord", CulturePhoto.class);
        method.invoke(obj, culturePhoto);
        e.printStackTrace();
    }
}
```

附：新建文件工具类方法

```java
/**
 * 新建临时文件(存放流文件)
 */
public static String getTempFile() {
    //时间格式化格式
    SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyyMMddHHmmssSSS");
    //获取当前时间并作为时间戳给文件夹命名
    String timeStamp = simpleDateFormat.format(new Date());
    String tempFile = timeStamp + ".tmp";
    // 创建存储文件夹
    File storageDir = new File(FileUploadConfig.getProtect());
    if (!storageDir.exists()) {
        storageDir.mkdirs();
    }
    File zipFile = new File(FileUploadConfig.getProtect() + tempFile);
    if (!zipFile.exists()) {
        try {
            zipFile.createNewFile();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    String tempFilePath = FileUploadConfig.getProtect() + tempFile;
    return tempFilePath;
}

/**
 * 新建导出的压缩文件
 */
public static String getZipFileName() {
    //时间格式化格式
    SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyyMMddHHmmssSSS");
    // 创建存储文件夹
    File storageDir = new File(FileUploadConfig.getProtect());
    if (!storageDir.exists()) {
        storageDir.mkdirs();
    }
    //获取当前时间并作为时间戳给文件夹命名
    String timeStamp = simpleDateFormat.format(new Date());
    String fileName = timeStamp + ".zip";
    File zipFile = new File(FileUploadConfig.getProtect() + fileName);
    if (!zipFile.exists()) {
        try {
            zipFile.createNewFile();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    String filePath = FileUploadConfig.getProtect() + fileName;
    return filePath;
}
```

### 三、下载压缩文件

> Controller
>
> 由于之前已经在数据库表中记录了Zip文件的路径，并在前端查看导出记录时会直接返回给前端，所以这里直接通过前端传来的文件路径进行Zip文件下载 【更稳妥的方法是通过导出记录Id去表中查文件路径】
>
> 这里直接在Controller中处理即可

```java
/**
 * 下载图片压缩包
 */
@PreAuthorize("@ss.hasPermi('repository:photo:export')")
@Log(title = "图片库", businessType = BusinessType.EXPORT)
@GetMapping("/downloadCompressionPhoto")
public void downloadCompressionPhoto(String filePath, HttpServletRequest request ,HttpServletResponse response)
{
    try {
        // 获取已经打好的压缩包
        File file = new File(filePath);
        String ext = CMyFile.extractFileExt(filePath);
        String mime = request.getContentType();
        if (mime == null) {
            mime = request.getSession().getServletContext().getMimeType(file.getName());
            if (mime == null) {
                mime = "application/octet-stream";
            }
        }
        response.setContentType(mime);
        response.setContentLength((int) file.length());
        String name = "评审图片" + new Date().getTime()+"."+ext;
        response.setHeader("Content-Disposition", "attachment;filename=" + new String(name.getBytes("GB2312"), "ISO-8859-1"));

        InputStream input = null;
        try {
            input = new BufferedInputStream(new FileInputStream(file));
            OutputStream output = response.getOutputStream();
            byte[] buffer = new byte[4096];
            for (;;) {
                int n = input.read(buffer);
                if (n == (-1))
                    break;
                output.write(buffer, 0, n);
            }
            output.flush();
        } finally {
            if (input != null) {
                input.close();
            }
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

> 前端Vue 
>
> 手动发起Axios请求 下载文件 要求响应类型为：blob

```js
// 下载图片导出文件
downloadCompressPhoto(e) {
  let filePath = e.originalFile
  axios({
    url: process.env.VUE_APP_BASE_API + '/repository/photo/downloadCompressionPhoto',
    method: 'get',
    params: { filePath: filePath },
    responseType: "blob",
    headers: { 'Authorization': 'Bearer ' + getToken() }
  }).then(response => {
		// 下载文件的名称
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
},
```



