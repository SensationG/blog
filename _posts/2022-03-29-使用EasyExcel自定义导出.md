# 使用EasyExcel自定义导出

> 本文章仅记录如何使用EasyExcel进行自定义合并单元格导出。
>  
> 注解导出以及简单的导出本文章略。


核心步骤：设置模板---->自定合并策略（选择合并标识）---->填充数据时调用合并策略

## 一、引入EasyExcel

```
<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>easyexcel</artifactId>
  <version>2.2.11</version>
</dependency>
```

这里使用2.的最高版本。

## 二、设置模板

模板如图：

![](https://blog-1302755396.cos.ap-shanghai.myqcloud.com/blog/20220329201418.png#crop=0&crop=0&crop=1&crop=1&id=ODpXn&originHeight=760&originWidth=2212&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

要点：

- list导入，模板中的{a.*}表示这是一个list
- 单个值导入
- 合并单元格，需要准备一个字段作为合并标识

## 三、设置自定义合并单元格策略

为了实现合并单元格，需要实现`CellWriteHandler`类。其中合并单元格策略只需重写`afterCellDispose`方法即可。

```java
/**
     * 合并字段的下标
     */
private int[] mergeColumnIndex;
/**
     * 合并几行
     */
private int mergeRowIndex;

/**
     * 合并标识
     */
private int mergeFlagIndex;


public ExcelFillCellMergeStrategy(int mergeRowIndex, int[] mergeColumnIndex, int mergeFlagIndex) {
  this.mergeRowIndex = mergeRowIndex;
  this.mergeColumnIndex = mergeColumnIndex;
  this.mergeFlagIndex = mergeFlagIndex;
}

@Override
public void afterCellDispose(WriteSheetHolder writeSheetHolder, WriteTableHolder writeTableHolder,
                             List<CellData> list, Cell cell, Head head, Integer integer, Boolean aBoolean) {
  //当前行
  int curRowIndex = cell.getRowIndex();
  //当前列
  int curColIndex = cell.getColumnIndex();

  if (curRowIndex > mergeRowIndex) {
    for (int i = 0; i < mergeColumnIndex.length; i++) {
      if (curColIndex == mergeColumnIndex[i]) {
        mergeWithPrevRow(writeSheetHolder, cell, curRowIndex, curColIndex);
        break;
      }
    }
  }
}

private void mergeWithPrevRow(WriteSheetHolder writeSheetHolder, Cell cell, int curRowIndex, int curColIndex) {
  //获取当前行的当前列的数据和上一行的当前列列数据，通过上一行数据是否相同进行合并
  Object curData = cell.getCellTypeEnum() == CellType.STRING ? cell.getStringCellValue() :
  cell.getNumericCellValue();
  Cell preCell = cell.getSheet().getRow(curRowIndex - 1).getCell(curColIndex);
  Object preData = preCell.getCellTypeEnum() == CellType.STRING ? preCell.getStringCellValue() :
  preCell.getNumericCellValue();

  // 合并标识
  Cell curFlagCell = cell.getSheet().getRow(curRowIndex).getCell(mergeFlagIndex);
  String curFlag = null;
  if (Objects.nonNull(curFlagCell)) {
    curFlag = curFlagCell.getCellTypeEnum() == CellType.STRING ? curFlagCell.getStringCellValue() : String.valueOf(curFlagCell.getNumericCellValue());
  }

  Cell preFlagCell = cell.getSheet().getRow(curRowIndex - 1).getCell(mergeFlagIndex);
  String preFlag = null;
  if (Objects.nonNull(preFlagCell)) {
    preFlag = preFlagCell.getCellTypeEnum() == CellType.STRING ? preFlagCell.getStringCellValue() : String.valueOf(preFlagCell.getNumericCellValue());
  }

  // 比较标识符是否相同
  if (Boolean.FALSE.equals(StringUtils.isNotEmpty(curFlag) && StringUtils.isNotEmpty(preFlag) && curFlag.equals(preFlag))) {
    return;
  }

  // 应对出现“0.0”情况
  if ((StringUtils.equals(curFlag, "0.0")) || (StringUtils.equals(preFlag, "0.0"))) {
    return;
  }

  // 比较当前行的第一列的单元格与上一行是否相同，相同合并当前单元格与上一行 【在这里进行是否合并判断】
  if (curData.equals(preData)) {
    Sheet sheet = writeSheetHolder.getSheet();
    List<CellRangeAddress> mergeRegions = sheet.getMergedRegions();
    boolean isMerged = false;
    for (int i = 0; i < mergeRegions.size() && !isMerged; i++) {
      CellRangeAddress cellRangeAddr = mergeRegions.get(i);
      // 若上一个单元格已经被合并，则先移出原有的合并单元，再重新添加合并单元
      if (cellRangeAddr.isInRange(curRowIndex - 1, curColIndex)) {
        sheet.removeMergedRegion(i);
        cellRangeAddr.setLastRow(curRowIndex);
        sheet.addMergedRegion(cellRangeAddr);
        isMerged = true;
      }
    }
    // 若上一个单元格未被合并，则新增合并单元
    if (!isMerged) {
      CellRangeAddress cellRangeAddress = new CellRangeAddress(curRowIndex - 1, curRowIndex, curColIndex,
                                                               curColIndex);
      sheet.addMergedRegion(cellRangeAddress);
    }
  }

}
```

## 四、填充Excel

> 通过步骤三实现了合并策略后，在填充数据时需要进行调用


```java
// 导出目录
String fileOutDirPath = filePath + FILE_FOLDER + "/" + projectId + "/finance";
File file = new File(fileOutDirPath);
if (!file.exists()) {
  file.mkdirs();
}
fileOutDirPath += "/" + UUID.randomUUID() + ".xlsx";
// 读取模板
ExcelWriter excelWriter = null;
InputStream resourceAsStream = this.getClass().getClassLoader().getResourceAsStream("financeExport.xlsx"))
  excelWriter = EasyExcel.write(fileOutDirPath).withTemplate(resourceAsStream).build();
FillConfig fillConfig = FillConfig.builder().forceNewRow(Boolean.TRUE).build();
Workbook workbook = excelWriter.writeContext().writeWorkbookHolder().getWorkbook();
// 开启强制计算单元格内公式
workbook.setForceFormulaRecalculation(true);

// 【填充数据】 ExcelFillCellMergeStrategy(从第几行开始合并, 需要合并的列, 合并标识所在列)
WriteSheet writeSheet0 = EasyExcel.writerSheet(0)
  .registerWriteHandler(new ExcelFillCellMergeStrategy(4, new int[]{4, 5, 6}, 0))
  .build();
// 查询数据并填充
Map<String, Object> estimateInfoMap = this.filterFinanceDetail(financeDetailDtoList);
excelWriter.fill(new FillWrapper("a", financeDetailDtoList), fillConfig, writeSheet0);
estimateInfoMap.putAll(projectInfoMap);
excelWriter.fill(estimateInfoMap, writeSheet0);

// 关闭
excelWriter.finish();
```

## 五、源码
```java
package com.hymake.pcmsz.project.finance.handler;

import com.alibaba.excel.metadata.CellData;
import com.alibaba.excel.metadata.Head;
import com.alibaba.excel.write.handler.CellWriteHandler;
import com.alibaba.excel.write.metadata.holder.WriteSheetHolder;
import com.alibaba.excel.write.metadata.holder.WriteTableHolder;
import org.apache.commons.lang3.StringUtils;
import org.apache.poi.ss.usermodel.Cell;
import org.apache.poi.ss.usermodel.CellType;
import org.apache.poi.ss.usermodel.Row;
import org.apache.poi.ss.usermodel.Sheet;
import org.apache.poi.ss.util.CellRangeAddress;

import java.util.List;
import java.util.Objects;

/**
* @Author: huanghwh
* @Date: 2022/03/25 10:09
* @Description:
*/
public class ExcelFillCellMergeStrategy implements CellWriteHandler {
    
    
    /**
    * 合并字段的下标
    */
    private int[] mergeColumnIndex;
    /**
    * 合并几行
    */
    private int mergeRowIndex;
    
    /**
    * 同组合并标识
    */
    private int mergeFlagIndex;
    
    public ExcelFillCellMergeStrategy() {
    }
    
    public ExcelFillCellMergeStrategy(int mergeRowIndex, int[] mergeColumnIndex, int mergeFlagIndex) {
        this.mergeRowIndex = mergeRowIndex;
        this.mergeColumnIndex = mergeColumnIndex;
        this.mergeFlagIndex = mergeFlagIndex;
    }
    
    @Override
    public void beforeCellCreate(WriteSheetHolder writeSheetHolder, WriteTableHolder writeTableHolder, Row row,
                                 Head head, Integer integer, Integer integer1, Boolean aBoolean) {
        
    }
    
    @Override
    public void afterCellCreate(WriteSheetHolder writeSheetHolder, WriteTableHolder writeTableHolder, Cell cell,
                                Head head, Integer integer, Boolean aBoolean) {
        
    }
    
    @Override
    public void afterCellDataConverted(WriteSheetHolder writeSheetHolder, WriteTableHolder writeTableHolder,
                                       CellData cellData, Cell cell, Head head, Integer integer, Boolean aBoolean) {
        
    }
    
    @Override
    public void afterCellDispose(WriteSheetHolder writeSheetHolder, WriteTableHolder writeTableHolder,
                                 List<CellData> list, Cell cell, Head head, Integer integer, Boolean aBoolean) {
        //当前行
        int curRowIndex = cell.getRowIndex();
        //当前列
        int curColIndex = cell.getColumnIndex();
        
        if (curRowIndex > mergeRowIndex) {
            for (int i = 0; i < mergeColumnIndex.length; i++) {
                if (curColIndex == mergeColumnIndex[i]) {
                    mergeWithPrevRow(writeSheetHolder, cell, curRowIndex, curColIndex);
                    break;
                }
            }
        }
    }
    
    private void mergeWithPrevRow(WriteSheetHolder writeSheetHolder, Cell cell, int curRowIndex, int curColIndex) {
        //获取当前行的当前列的数据和上一行的当前列列数据，通过上一行数据是否相同进行合并
        Object curData = cell.getCellTypeEnum() == CellType.STRING ? cell.getStringCellValue() :
        cell.getNumericCellValue();
        Cell preCell = cell.getSheet().getRow(curRowIndex - 1).getCell(curColIndex);
        Object preData = preCell.getCellTypeEnum() == CellType.STRING ? preCell.getStringCellValue() :
        preCell.getNumericCellValue();
        
        // 合并标识
        Cell curFlagCell = cell.getSheet().getRow(curRowIndex).getCell(mergeFlagIndex);
        String curFlag = null;
        if (Objects.nonNull(curFlagCell)) {
            curFlag = curFlagCell.getCellTypeEnum() == CellType.STRING ? curFlagCell.getStringCellValue() : String.valueOf(curFlagCell.getNumericCellValue());
        }
        
        Cell preFlagCell = cell.getSheet().getRow(curRowIndex - 1).getCell(mergeFlagIndex);
        String preFlag = null;
        if (Objects.nonNull(preFlagCell)) {
            preFlag = preFlagCell.getCellTypeEnum() == CellType.STRING ? preFlagCell.getStringCellValue() : String.valueOf(preFlagCell.getNumericCellValue());
        }
        
        // 比较标识符是否相同
        if (Boolean.FALSE.equals(StringUtils.isNotEmpty(curFlag) && StringUtils.isNotEmpty(preFlag) && curFlag.equals(preFlag))) {
            return;
        }
        
        // 应对出现“0.0”情况
        if ((StringUtils.equals(curFlag, "0.0")) || (StringUtils.equals(preFlag, "0.0"))) {
            return;
        }
        
        // 比较当前行的第一列的单元格与上一行是否相同，相同合并当前单元格与上一行
        if (curData.equals(preData)) {
            Sheet sheet = writeSheetHolder.getSheet();
            List<CellRangeAddress> mergeRegions = sheet.getMergedRegions();
            boolean isMerged = false;
            for (int i = 0; i < mergeRegions.size() && !isMerged; i++) {
                CellRangeAddress cellRangeAddr = mergeRegions.get(i);
                // 若上一个单元格已经被合并，则先移出原有的合并单元，再重新添加合并单元
                if (cellRangeAddr.isInRange(curRowIndex - 1, curColIndex)) {
                    sheet.removeMergedRegion(i);
                    cellRangeAddr.setLastRow(curRowIndex);
                    sheet.addMergedRegion(cellRangeAddr);
                    isMerged = true;
                }
            }
            // 若上一个单元格未被合并，则新增合并单元
            if (!isMerged) {
                CellRangeAddress cellRangeAddress = new CellRangeAddress(curRowIndex - 1, curRowIndex, curColIndex,
                                                                         curColIndex);
                sheet.addMergedRegion(cellRangeAddress);
            }
        }
        
    }
    
}

```
