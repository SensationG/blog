---
layout: post
title: java-Excel自定义导出
date: 2020-12-13
Author: hhw
toc: true
comments: true
tags:  [Java,Excel,SpringBoot]

---

### 使用Java对Excel操作并导出

- 在开发系统过程中，我们时常遇到需要将数据导出为Excel文档并下载的需求，本篇文章记录如果自定义Excel并导出供用户下载

- 使用到的Excel开发包：

  `Jxl`

  Jxl是一个开源的Java Excel API项目，通过Jxl，Java可以很方便的操作微软的Excel文档。除了Jxl之外，还有Apache的一个POI项目，也可以操作Excel，两者相比之下：Jxl使用方便，但功能相对POI比较弱。POI使用复杂，上手慢，除了这个没啥说的了。 [参考](https://blog.51cto.com/lavasoft/174244)

- 需求如下：

  - 自定义工作簿（sheet）名称
  - 自定义标题名称，对excel可以客制化自定义输出内容，或对内容输出前进行加工

#### 引入

在进行开发之前，我们需要先引入Jxl的开发包，这里我们使用Maven的方式引入：

```xml
<!-- maven如下  -->
<!-- https://mvnrepository.com/artifact/net.sourceforge.jexcelapi/jxl -->
<dependency>
    <groupId>net.sourceforge.jexcelapi</groupId>
    <artifactId>jxl</artifactId>
    <version>2.6.12</version>
</dependency>
```

#### 操作

引入完成后，我们就可以直接通过现有的Api简单便捷地对Excel进行操作了

##### 创建

```java
// 新建文件目录
String fileName = FileUtil.getFileName(".xls");
// 创建Excel文件
WritableWorkbook wwb = Workbook.createWorkbook(new File(fileName));
// 创建sheet，并可指定index
WritableSheet wss = wwb.createSheet("照片信息", 0);
```

接下来是设置Excel的样式，可参考

```java
// 接下来是设置Excel的外观 （可忽略）
wss.getSettings().setBottomMargin(0.7d);
wss.getSettings().setTopMargin(0.7d);
wss.getSettings().setLeftMargin(0.75d);
wss.getSettings().setRightMargin(0.75d);

// 页面方向 PageOrientation.LANDSCAPE 横向 ,   PageOrientation.PORTRAIT   纵向
wss.setPageSetup(PageOrientation.LANDSCAPE, PaperSize.A4, 0d, 0d);

WritableFont font = new WritableFont(WritableFont.createFont("微软雅黑"), 10, WritableFont.NO_BOLD);
WritableCellFormat wcf1 = new WritableCellFormat(font);

// 边框
wcf1.setBorder(Border.ALL, BorderLineStyle.THIN);
// 文字垂直对齐
wcf1.setVerticalAlignment(VerticalAlignment.CENTRE);
// 文字水平对齐
wcf1.setAlignment(Alignment.CENTRE);
wcf1.setWrap(true);
wcf1.setLocked(false);

jxl.SheetSettings sheetSet = wss.getSettings();
sheetSet.setScaleFactor(98);
wss.setRowView(0, 600);
```

##### 设定列名

> 我们通过刚才创建的wss（即excel-工作表），对当前工作表进行操作，设定列名

```java
wss.setColumnView(0, 10);
wss.addCell(new Label(0, 0, "照片ID", wcf1));

wss.setColumnView(1, 30);
wss.addCell(new Label(1, 0, "标题", wcf1));

wss.setColumnView(2, 20);
wss.addCell(new Label(2, 0, "作者", wcf1));

wss.setColumnView(3, 20);
wss.addCell(new Label(3, 0, "拍摄时间", wcf1));

wss.setColumnView(4, 20);
wss.addCell(new Label(4, 0, "创建人", wcf1));

wss.setColumnView(5, 20);
wss.addCell(new Label(5, 0, "创建时间", wcf1));
```

##### 填入数据

> 通常这部分我们通过查询数据库中的数据，并且通过遍历即可填入数据

```java
// 当前行, 由于第0行设定了列名，所以这里下标从1开始，实际对应Excel的第2行
int rowIndex = 1;
// 遍历实体类，实体类存放从数据库中查询的数据
for (CulturePhoto photo: culturePhotoList) {

    SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    String createTime = simpleDateFormat.format(photo.getCreateTime());
    String updateTime = simpleDateFormat.format(photo.getUpdateTime());
    String shootingTime = simpleDateFormat.format(photo.getShootingTime());

    wss.addCell(new Label(0, rowIndex, String.valueOf(photo.getPhotoId()), wcf1));
    wss.addCell(new Label(1, rowIndex, photo.getTitle(), wcf1));
    wss.addCell(new Label(2, rowIndex, photo.getAuthor(), wcf1));
    wss.addCell(new Label(3, rowIndex, shootingTime, wcf1));
    wss.addCell(new Label(4, rowIndex, photo.getCreateUser(), wcf1));
    wss.addCell(new Label(5, rowIndex, createTime, wcf1));

    rowIndex++;
}

// 关闭流
wwb.write();
wwb.close();
```

在此之后，excel文件已经制作完毕，将文件通过流的形式返回给前端下载即可。

读取文件的输入流 --> 填入response的输出流

controller例：

```java
@GetMapping("/example")
public void example(HttpServletRequest request ,HttpServletResponse response)
{
  try {
    // filePath 需要自己传入
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
    String name = "excel" + DateUtils.getNowDate() +"."+ext;
    response.setHeader("Content-Disposition", "attachment;filename=" + new String(name.getBytes("GB2312"), "ISO-8859-1"));

    InputStream input = null;
    OutputStream output = null;
    try {
      // 读取文件输入流
      input = new BufferedInputStream(new FileInputStream(file));
      // 获取response输出流
      output = response.getOutputStream();
      byte[] buffer = new byte[4096];
      for (;;) {
        // 将文件的输入流转为字节流
        int n = input.read(buffer);
        if (n == (-1)) {
          break;
        }
        //  response读取字节流
        output.write(buffer, 0, n);
      }
      output.flush();
    } finally {
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
}
```

前端发起请求：

```js
axios({
  url: process.env.VUE_APP_BASE_API + '/repository/photo/downloadCompressionPhoto',
  method: 'get',
  params: { photoExportId: photoExportId },
  responseType: "blob",
  headers: { 'Authorization': 'Bearer ' + getToken() }
}).then(response => {
  // 下载的文件名
	let zipName = "人员分组导出文件.xls";
  // 下载格式为excel 这里根据下载文件的类型不同自定义响应头
  let blob = new Blob([response.data], {type: "application/vnd.ms-excel"}); 
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