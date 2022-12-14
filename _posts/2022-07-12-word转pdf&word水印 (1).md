> > - 本工具适用于使用xml模板基于freemark导出的word(本质上是xml格式)
>    - 本文略过freemark导出word过程
> - 功能：对word添加水印，word转pdf
> - 本文基于aspose-words-18.8.jar工具进行操作

# 一、引入aspose包
> 本文使用破解的aspose.jar包，仅供学习使用，禁止商业用途

- 需要使用外置jar包，引入方式见文章：

[2022-07-11-SpringBoot手动引入jar](https://www.yuque.com/sensation-x8xmo/dev-note/llqsyp?view=doc_embed)

- jar包
   - aspose-words-18.8.jar [下载](https://www.icloud.com.cn/iclouddrive/004RmJg7hrN1gKXzf-45q7emA#aspose-words-18)
# 二、编写工具类

- **必要：1、证书license 2、字体库(网上下载)**
- license.xml (许可证，放在resources下)
```xml
<License>
  <Data>
    <Products>
      <Product>Aspose.Total for Java</Product>
      <Product>Aspose.Words for Java</Product>
    </Products>
    <EditionType>Enterprise</EditionType>
    <SubscriptionExpiry>20991231</SubscriptionExpiry>
    <LicenseExpiry>20991231</LicenseExpiry>
    <SerialNumber>8bfe198c-7f0c-4ef8-8ff0-acc3237bf0d7</SerialNumber>
  </Data>
  <Signature>sNLLKGMUdF0r8O1kKilWAGdgfs2BvJb/2Xp8p5iuDVfZXmhppo+d0Ran1P9TKdjV4ABwAgKXxJ3jcQTqE/2IRfqwnPf8itN8aFZlV3TJPYeD3yWE7IT55Gz6EijUpC7aKeoohTb4w2fpox58wWoF3SNp6sK6jDfiAUGEHYJ9pjU=
  </Signature>
</License>
```

- word2Pdf工具类 **注意license认证步骤 ，字体库可在配置文件中配置路径**
```java
import com.aspose.words.Document;
import com.aspose.words.FontSettings;
import com.aspose.words.License;
import com.aspose.words.SaveFormat;
import java.io.File;
import java.io.FileOutputStream;
import java.io.InputStream;
import java.util.UUID;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.core.io.ClassPathResource;
import org.springframework.stereotype.Component;


/**
* @Author: huanghwh
* @Date: 2022/7/6 下午8:46
* <p>
* word转pdf工具
*/
@Component
@Slf4j
public class WordPdfUtil {
    
    private static boolean license = false;
    
    /**
    * 导出根路径
    */
    private static String OUT_FOLDER_PATH;
    
    /**
    * 字体路径
    */
    private static String FONT_FOLDER_PATH;
    
    
    @Value("${spring.web.upload.folder}")
    public void setOutFolderPath(String outFolderPath) {
        WordPdfUtil.OUT_FOLDER_PATH = outFolderPath;
    }
    
    @Value("${aspose.font-folder}")
    public void setFontFolderPath(String fontFolderPath) {
        WordPdfUtil.FONT_FOLDER_PATH = fontFolderPath;
    }
    
    static {
        try {
            // 许可证 license.xml放在src/main/resources文件夹下
            ClassPathResource templateDirPath = new ClassPathResource("license.xml");
            InputStream is = templateDirPath.getInputStream();
            License asposeLic = new License();
            asposeLic.setLicense(is);
            license = true;
        } catch (Exception e) {
            license = false;
            System.out.println("License验证失败...");
            e.printStackTrace();
        }
    }
    
    
    /**
    * doc转pdf
    *
    * @param wordPath         word路径
    * @param pdfPath          pdf文件输出路径
    * @return {@link File}
    * @author huanghwh
    * @date 2022/7/7 下午3:28
    */
    public static File doc2Pdf(String wordPath, String pdfPath) {
        return doc2Pdf(wordPath, pdfPath, false, null);
    }
    
    
    /**
    * doc转pdf
    *
    * @param wordPath         word路径
    * @param pdfPath          pdf文件输出路径
    * @param addWatermark     是否添加水印
    * @param watermarkImgPath 水印图片路径
    * @return {@link File}
    * @author huanghwh
    * @date 2022/7/7 下午3:28
    */
    public static File doc2Pdf(String wordPath, String pdfPath, boolean addWatermark, String watermarkImgPath) {
        pdfPath = OUT_FOLDER_PATH + pdfPath + "/" + UUID.randomUUID() + ".pdf";
        // 验证License 若不验证则转化出的pdf文档会有水印产生
        if (!license) {
            log.error("License验证不通过...");
            throw new CommonException("Word 转 Pdf 失败: License验证不通过");
        }
        
        File file = new File(pdfPath);
        try (FileOutputStream os = new FileOutputStream(file)) {
            long old = System.currentTimeMillis();
            Document doc = new Document(wordPath);
            
            // 设置字体 字体放在resource/font下
            FontSettings fontSettings = FontSettings.getDefaultInstance();
            fontSettings.setFontsFolders(new String[]{ FONT_FOLDER_PATH }, true);
            doc.setFontSettings(fontSettings);
            
            // 设置水印
            if (addWatermark) {
                // 700 800 刚好铺满a4纸
                WatermarkUtil.insertWatermarkImg(doc, watermarkImgPath, 700, 800);
            }
            
            //全面支持DOC, DOCX, OOXML, RTF HTML, OpenDocument, PDF, EPUB, XPS, SWF 相互转换
            doc.save(os, SaveFormat.PDF);
            
            long now = System.currentTimeMillis();
            //转化用时
            log.info("Word 转 Pdf 共耗时：" + ((now - old) / 1000.0) + "秒");
            return file;
        } catch (Exception e) {
            log.error("Word 转 Pdf 失败..." + e.getMessage());
            e.printStackTrace();
            throw new CommonException("Word 转 Pdf 失败");
        }
    }
    
    
    
}

```

- word水印工具类 **注意如果单独使用需要先进行license注册**
```java
import com.aspose.words.Document;
import com.aspose.words.HeaderFooter;
import com.aspose.words.HeaderFooterType;
import com.aspose.words.HorizontalAlignment;
import com.aspose.words.Paragraph;
import com.aspose.words.RelativeHorizontalPosition;
import com.aspose.words.RelativeVerticalPosition;
import com.aspose.words.Section;
import com.aspose.words.Shape;
import com.aspose.words.ShapeType;
import com.aspose.words.VerticalAlignment;
import com.aspose.words.WrapType;
import java.awt.Color;
import java.io.File;
import lombok.extern.slf4j.Slf4j;

/**
 * @Author: huanghwh
 * @Date: 2022/7/7 下午3:04
 */
@Slf4j
public class WatermarkUtil {


    public static void main(String[] args) {
        // sample
        try {
            Document document = new Document("/Users/hhw/Downloads/mark.doc");
            WatermarkUtil.WaterMarkMore(document,"我的水印");
            //文件输出路径
            document.save("/Users/hhw/Downloads/mark.doc");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 为word文档添加图片水印
     *
     * @param doc     word文档模型
     * @param imgPath 图片水印路径
     * @param width   宽度
     * @param height  高度
     * @throws Exception 异常
     */
    public static void insertWatermarkImg(Document doc, String imgPath, Integer width, Integer height) throws Exception {
        log.info("开始添加水印");
        Shape watermark = new Shape(doc, ShapeType.IMAGE);
        if (!new File(imgPath).exists()) {
            throw new CommonException("未找到水印文件!");
        }
        //水印内容
        watermark.getImageData().setImage(imgPath);
        //水印字体
        //watermark.getTextPath().setFontFamily("宋体");
        //水印宽度
        watermark.setWidth(width);
        //水印高度
        watermark.setHeight(height);
        //旋转水印
        //watermark.setRotation(-40);
        //水印颜色 浅灰色
        watermark.getFill().setColor(Color.lightGray);
        watermark.setStrokeColor(Color.lightGray);
        //设置相对水平位置
        watermark.setRelativeHorizontalPosition(RelativeHorizontalPosition.PAGE);
        //设置相对垂直位置
        watermark.setRelativeVerticalPosition(RelativeVerticalPosition.PAGE);
        //设置包装类型
        watermark.setWrapType(WrapType.NONE);
        //设置垂直对齐
        watermark.setVerticalAlignment(VerticalAlignment.CENTER);
        //设置文本水平对齐方式
        watermark.setHorizontalAlignment(HorizontalAlignment.CENTER);
        Paragraph watermarkPara = new Paragraph(doc);
        watermarkPara.appendChild(watermark);
        for (Section sect : doc.getSections())
        {
            insertWatermarkIntoHeader(watermarkPara, sect, HeaderFooterType.HEADER_PRIMARY);
            insertWatermarkIntoHeader(watermarkPara, sect, HeaderFooterType.HEADER_FIRST);
            insertWatermarkIntoHeader(watermarkPara, sect, HeaderFooterType.HEADER_EVEN);
        }
        log.info("水印添加完毕");
    }
    
    /**
     * 在页眉中插入水印
     * @param watermarkPara
     * @param sect
     * @param headerType
     * @throws Exception
     */
    private static void insertWatermarkIntoHeader(Paragraph watermarkPara, Section sect, int headerType) throws Exception{
        HeaderFooter header = sect.getHeadersFooters().getByHeaderFooterType(headerType);
        if (header == null)
        {
            header = new HeaderFooter(sect.getDocument(), headerType);
            sect.getHeadersFooters().add(header);
        }
        header.appendChild(watermarkPara.deepClone(true));
    }

    /**
     * 设置水印属性
     * @param doc
     * @param wmText
     * @param left
     * @param top
     * @return
     * @throws Exception
     */
    public static Shape ShapeMore(Document doc, String wmText, double left, double top)throws Exception{
        Shape waterShape = new Shape(doc, ShapeType.TEXT_PLAIN_TEXT);
        //水印内容
        waterShape.getTextPath().setText(wmText);
        //水印字体
        waterShape.getTextPath().setFontFamily("宋体");
        //水印宽度
        waterShape.setWidth(40);
        //水印高度
        waterShape.setHeight(13);
        //旋转水印
        waterShape.setRotation(-40);
        //水印颜色 浅灰色
        /*waterShape.getFill().setColor(Color.lightGray);
        waterShape.setStrokeColor(Color.lightGray);*/
        waterShape.setStrokeColor(new Color(210,210,210));
        //将水印放置在页面中心
        waterShape.setLeft(left);
        waterShape.setTop(top);
        //设置包装类型
        waterShape.setWrapType(WrapType.NONE);
        return waterShape;
    }

    /**
     * 插入多个水印
     * @param mdoc
     * @param wmText
     * @throws Exception
     */
    public static void WaterMarkMore(Document mdoc, String wmText)throws Exception{
        Paragraph watermarkPara = new Paragraph(mdoc);
        for (int j = 0; j < 500; j = j + 100)
        {
            for (int i = 0; i < 700; i = i + 85)
            {
                Shape waterShape = ShapeMore(mdoc, wmText, j, i);
                watermarkPara.appendChild(waterShape);
            }
        }
        for (Section sect : mdoc.getSections())
        {
            insertWatermarkIntoHeader(watermarkPara, sect, HeaderFooterType.HEADER_PRIMARY);
            insertWatermarkIntoHeader(watermarkPara, sect, HeaderFooterType.HEADER_FIRST);
            insertWatermarkIntoHeader(watermarkPara, sect, HeaderFooterType.HEADER_EVEN);
        }
    }


    /**
     * 给doc生成文字水印
     *
     * @param doc
     * @param watermarkText
     * @throws Exception
     * @author huanghwh
     * @date 2022/7/6 下午10:50
     */
    public static void insertWatermarkText(Document doc, String watermarkText)
            throws Exception {

        log.info("开始添加水印...");
        Shape watermark = new Shape(doc, ShapeType.TEXT_PLAIN_TEXT);

        // 水印内容
        watermark.getTextPath().setText(watermarkText);
        // 水印字体
        watermark.getTextPath().setFontFamily("宋体");
        // 水印宽度
        watermark.setWidth(500);
        // 水印高度
        watermark.setHeight(100);
        // 旋转水印
        watermark.setRotation(-40);
        // 水印颜色
        watermark.getFill().setColor(Color.lightGray);
        watermark.setStrokeColor(Color.lightGray);

        watermark.setRelativeHorizontalPosition(RelativeHorizontalPosition.PAGE);
        watermark.setRelativeVerticalPosition(RelativeVerticalPosition.PAGE);
        watermark.setWrapType(WrapType.NONE);
        watermark.setVerticalAlignment(VerticalAlignment.CENTER);
        watermark.setHorizontalAlignment(HorizontalAlignment.CENTER);

        Paragraph watermarkPara = new Paragraph(doc);
        watermarkPara.appendChild(watermark);

        for (Section sect : doc.getSections()) {
            insertWatermarkIntoHeader(watermarkPara, sect,
                    HeaderFooterType.HEADER_PRIMARY);
            insertWatermarkIntoHeader(watermarkPara, sect,
                    HeaderFooterType.HEADER_FIRST);
            insertWatermarkIntoHeader(watermarkPara, sect,
                    HeaderFooterType.HEADER_EVEN);
        }
        log.info("结束添加水印...");
    }

}
```
# 三、使用示例
```java
// 执行导出
String outPath = "/project/" + contactListEntity.getProjId() + "/contactList";
File docFile = WordExportUtil.export("template","contactListBusiness_export_template.ftl", dataMap, outPath);
// 加水印并转成pdf
File pdfFile = null;
try {
    ClassPathResource watermarkResource = new ClassPathResource(watermarkImgPath);
    pdfFile = WordPdfUtil.doc2Pdf(docFile.getPath(), outPath, true, watermarkResource.getFile().getPath());
} catch (IOException e) {
    e.printStackTrace();
    throw new CommonException("添加水印或转换pdf失败");
}
```
# 四、使用问题
## 1.遇到word中存在特殊字符导致word转pdf失败
> 报错信息：
> Word 转 Pdf 失败...The document appears to be corrupted and cannot be loaded.

由于使用freemarker生成的文档本质是xml格式的，因此文档中的内容必须遵循xml标准，不允许存在特殊的敏感字符，需要对填充的内容进行批量过滤，将特殊字符进行转义后再生成word，即可解决。
在 XML 中，有 5 个预定义的实体引用：

| &lt; | < | 小于 |
| --- | --- | --- |
| &gt; | > | 大于 |
| &amp; | & | 和号 |
| &apos; | ' | 单引号 |
| &quot; | " | 引号 |

**注释：**在 XML 中，只有字符 "<" 和 "&" 确实是非法的。大于号是合法的，但是用实体引用来代替它是一个好习惯。
转换工具类：主要针对`<`和 '&' 进行转义

- 使用A.class.isAssignableFrom(B.class)方法可以判断B类是否是A或者A的子类
```java
/**
     * xml特殊字符转义 Map
     *
     * @param dataMap
     */
public static void escape(Map<String, Object> dataMap) {
    // 遍历Map
    for (Map.Entry<String, Object> entry : dataMap.entrySet()) {
        Object value = entry.getValue();
        if (Objects.isNull(value)) {
            continue;
        }
        if (value instanceof String) {
            String dataStr = String.valueOf(value);
            dataMap.put(entry.getKey(), escape(dataStr));
        }
        // 处理List中包含map的情况
        if (List.class.isAssignableFrom(value.getClass())) {
            List list = (List) value;
            for (Object listObject : list) {
                if (Map.class.isAssignableFrom(listObject.getClass())) {
                    Map<Object, Object> map = (Map<Object, Object>) listObject;
                    for (Map.Entry<Object, Object> itemMap : map.entrySet()) {
                        if (itemMap.getValue() instanceof String) {
                            map.put(itemMap.getKey(), escape(String.valueOf(itemMap.getValue())));
                        }
                    }
                }
            }
        }
    }
}


/**
     * 转义字符串
     *
     * @param str str
     * @return {@link String}
     */
private static String escape(String str) {
    str = str.replace("&", "&amp;");
    str = str.replace("<", "&lt;");
    return str;
}
```
## 2. freemarker生成word
[2021-11-16-使用Freemark导出word](https://www.yuque.com/sensation-x8xmo/dev-note/2021-11-16-freemarker?view=doc_embed)
或直接使用工具类：
```java
@Slf4j
@Component
public class WordExportUtil {

    /** 导出根路径 */
    private static String outFolderPath;


    @Value("${spring.web.upload.folder}")
    public void setOutFolderPath(String outFolderPath) {
        WordExportUtil.outFolderPath = outFolderPath;
    }


    /**
     * 根据模板导出word
     *
     * @param templateName 模板名称 (模板需放在resource/template下)
     * @param dataMap      数据
     * @param outPath      导出路径
     * @return {@link File}
     * @Author huanghwh
     * @Date 2021/8/26 14:37
     */
    public static File export(String templateName, Map<String, Object> dataMap, String outPath) {
        log.info("+++=======word导出开始=======+++");
        long old = System.currentTimeMillis();

        // xml特殊字符转义
        try {
            XmlSpecialCharUtil.escape(dataMap);
        } catch (Exception e) {
            log.error("特殊字符转义失败");
            e.printStackTrace();
            throw new CommonException("特殊字符转义失败");
        }

        Template t = null;
        Configuration configuration = new Configuration(Configuration.VERSION_2_3_28);
        configuration.setDefaultEncoding("UTF-8");
        configuration.setClassicCompatible(true);
        try {
            //加载模板(路径)数据 resource下
            configuration.setClassForTemplateLoading(WordExportUtil.class, "/template");
            t = configuration.getTemplate(templateName, "UTF-8");
        } catch (TemplateNotFoundException e) {
            log.error("模板文件未找到" + e.getMessage());
            throw new CommonException("模板文件未找到");
        } catch (MalformedTemplateNameException e) {
            log.error("模板类型不正确" + e.getMessage());
            throw new CommonException("模板类型不正确");
        } catch (IOException e) {
            log.error("IO读取失败" + e.getMessage());
            throw new CommonException("IO读取失败");
        }
        // 组装输出路径
        String fileOutDirPath = outFolderPath + outPath;
        File file = new File(fileOutDirPath);
        if (!file.exists()) {
            file.mkdirs();
        }
        String fileOutPath = fileOutDirPath + "/" + UUID.randomUUID() + ".doc";
        File outFile = new File(fileOutPath);
        try (
            FileOutputStream fos = new FileOutputStream(outFile);
            Writer out = new BufferedWriter(new OutputStreamWriter(fos))
        ) {
            t.process(dataMap, out);
        } catch (Exception e) {
            log.error("+++=======word导出：填充模板时异常=======+++");
            e.printStackTrace();
            throw new CommonException("填充模板时异常");
        }
        log.info("由模板文件：" + templateName + " 生成文件 ：" + fileOutPath + " 成功！");
        log.info("+++=======word导出结束=======+++");
        log.info("word导出共耗时：" + ((System.currentTimeMillis() - old) / 1000.0) + "秒");
        return outFile;
    }


}
```
## 3.使用aspose删除多余空白页面
```java
/**
     * 删除多余的空白页面
     *
     * @param document
     * @return
     * @author huanghwh
     * @date 2022/7/28 下午9:29
     */
public static void removeBlank(Document document) {
    for (Section section : document.getSections()) {
        //删除空白页部分
        if (StrUtil.isBlank(section.getBody().getText())) {
            document.removeChild(section);
        }
        //得到所有段落
        for (Paragraph paragraph : section.getBody().getParagraphs()) {
            //flag代表该段落有没有图片
            boolean flag = false;
            if (paragraph.getChildNodes(NodeType.SHAPE, true).getCount() == 0) {
                flag = true;
            }
            //得到各个run
            RunCollection runs = paragraph.getRuns();
            if (flag) {
                //首先去除各个部分的转义字符，如果删除之后run为空则去除
                for (Run run : runs) {
                    run.setText(run.getText().replaceAll("[\f|\r|\n]", ""));
                }
                //删除空白的paragraph，如果有图片则不删除，
                String content = StrUtil.cleanBlank(paragraph.getText());
                if (StrUtil.isBlank(content)) {
                    section.getBody().getParagraphs().remove(paragraph);
                }
            }
        }
        /** 以下可能导致表格样式错乱，先注释 */
        //去除空白的table 
        /*for (Table table : section.getBody().getTables()) {
                for (Row row : table.getRows()) {
                    for (Cell cell : row.getCells()) {
                        if (StrUtil.isBlank(cell.getText())) {
                            row.getCells().remove(cell);
                        }
                    }
                    if (StrUtil.isBlank(row.getText())) {
                        table.getRows().remove(row);
                    }
                }
                if (StrUtil.isBlank(table.getText())) {
                    section.getBody().getTables().remove(table);
                }
            }*/
    }
}
```
