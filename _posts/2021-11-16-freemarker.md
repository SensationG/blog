---
layout: post
title: 使用Freemark导出word
date: 2021-11-16
Author: hhw
comments: true
toc: true
tags:  [java,freemarker]
---

## 使用Freemarker 导出Word

### 1、添加依赖

```xml
<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
    <version>2.3.28</version>
</dependency>
```

### 2、编写导出通用工具类

#### 1.代码

```java
import com.hymake.framework.core.exception.CommonException;
import freemarker.template.Configuration;
import freemarker.template.MalformedTemplateNameException;
import freemarker.template.Template;
import freemarker.template.TemplateNotFoundException;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import java.io.*;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Map;

/**
 * @Author: huanghwh
 * @Date: 2021/08/26 14:35
 * @Description: freemarker word导出工具类
 */
@Slf4j
@Component
public class WordExportUtil {

    /**
     * 模板路径
     */
    private static String rootPath;
    /**
     * 导出路径
     */
    private static String outPath;


    @Value("${template.rootPath}")
    public void setRootPath(String rootPath) {
        WordExportUtil.rootPath = rootPath;
    }

    @Value("${template.outPath}")
    public void setOutPath(String outPath) {
        WordExportUtil.outPath = outPath;
    }


    /**
     * 根据模板导出word
     *
     * @param templateName
     * @param dataMap
     * @param fileName
     * @return
     * @Author huanghwh
     * @Date 2021/8/26 14:37
     */
    public static File export(String templateName, Map<String, Object> dataMap, String fileName) {
        log.info("+++=======word导出开始=======+++");
        Template t = null;
        Configuration configuration = new Configuration(Configuration.VERSION_2_3_28);
        configuration.setDefaultEncoding("UTF-8");
        configuration.setClassicCompatible(true);
        try {
            //加载模板(路径)数据
            configuration.setDirectoryForTemplateLoading(new File(rootPath));
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
        Date date = new Date();
        SimpleDateFormat sf = new SimpleDateFormat("yyyyMMddhhmmssSSS");
        String fileOutDirPath = outPath + "/" + sf.format(date);
        File file = new File(fileOutDirPath);
        if (!file.exists()) {
            file.mkdirs();
        }
        String fileOutPath = fileOutDirPath + "/" + fileName;
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
        return outFile;
    }


}
```

#### 2.配置文件添加

```yml
template:
  # 模板文件存放路径
  rootPath: template/
  # 模板导出路径
  outPath: template//exportFile
```

* windows请使用单斜杠

### 3、模板编写

#### 1.普通占位符

```
${demo}
```

#### 2.list占位符

```xml
<#list analysis as analysis>
	....
    ${analysis_index + 1} <!--list 中的自动序号 -->
    ${analysis.engineeringName} <!--list 中的普通对象属性 -->
</#list>
```

#### 3.if

```xml
<#if evaluateList[5].selectValue=="true">
    <w:sym w:font="Wingdings 2" w:char="F052" /> 
<#else>
    <w:t>□</w:t>
 </#if>
```

- 示例中是word打钩和不打钩

#### 编写完成后转.ftl

### 4、导出

```java
Map<String, Object> dataMap = new HashMap<>(16);
dataMap.put("name", exportData.getExpertName());
dataMap.put("projectName", exportData.getProjectName());
dataMap.put("finalScore", exportData.getScore());
dataMap.put("evaluateList", sortList);
dataMap.put("aScore", detailList.get(1).getScore());
dataMap.put("bScore", detailList.get(2).getScore());
dataMap.put("cScore", detailList.get(3).getScore());

// 模板名 所需数据 生成的文件名
return WordExportUtil.export("专家评价导出模板.ftl", dataMap, "专家评价导出.doc");
```

