---
title: "[Spring Boot]读取Excel文件"
date: 2019-08-18T20:30:10+08:00
lastmod: 2019-08-18T20:30:10+08:00
draft: false
keywords: ["spring", "spring-boot", "java", "excel"]
description: "spring-boot 读取Excel文件"
tags: ["spring-boot", "excel", "java"]
categories: ["Java"]
author: "renyijiu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: false
autoCollapseToc: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: true
mathjax: false
---

最近有一个需求，需要读取上传的Excel然后批量导入相关的数据，因为在Java上开始也没有接触过如何处理Excel文件，另外在处理这个需求的时候也碰到了一些稀奇的问题，所以记录一下。

<!--more-->

通过搜索发现，这个问题其实很简单，存在着[Apache POI - the Java API for Microsoft Documents](https://poi.apache.org/)这个库专门处理相关的文件，所以呢简单的介绍下使用过程即可。

## 1. Maven Dependencies

在`pom.xml`文件中加入下列依赖：

```xml
<dependency>
  <groupId>org.apache.poi</groupId>
  <artifactId>poi</artifactId>
  <version>${poi.version}</version>
</dependency>
<dependency>
  <groupId>org.apache.poi</groupId>
  <artifactId>poi-ooxml</artifactId>
  <version>${poi.version}</version>
</dependency>
```

`poi`主要用于处理 `xls` 后缀格式文件，而`poi-ooxml`用于处理`xlsx`后缀格式文件，这次需求也主要使用此格式文件，所以引入。

⚠️值得注意的是，**这里使用了一个变量确保两者的版本一致**。因为最开始自己是分别从maven仓库拷贝了配置（两个版本不一致）导入，然后在测试时，一直提示一个奇怪的函数错误问题，自己搜索了好久一直不得其解，后来偶然看到了一个回答，统一了两者的版本，问题就顺利的解决了。（那个回答的链接暂时找不到了，后续有机会补充）

## 2. Apache POI

### 2.1 Excel Cell

封装Excel单元格对象，包含了相关的各个属性，方便我们的后续操作。

```java
@Data
@JsonIgnoreProperties(ignoreUnknown = true)
public class ExcelCellBean implements Serializable {
    private static final long serialVersionUID = -3521471430505349177L;

    private String content;

    private String textColor;

    private String bgColor;

    private String textSize;

    private String textWeight;

    public ExcelCellBean(String content) {
        this.content = content;
    }
}
```

### 2.2 Reading from Excel

封装相关的读取函数，方便后续的业务调用。

```java
@Component
@Slf4j
public class ExcelPOIUtil {

    public Map<Integer, List<ExcelCellBean>> readExcel(String fileLocation) throws IOException {
        String XLS_FORMAT = ".xls";
        String XLSX_FORMAT = ".xlsx";

        log.info("excel file path: {}", fileLocation);
        Map<Integer, List<ExcelCellBean>> data = new HashMap<>();
        FileInputStream fis = new FileInputStream(new File(fileLocation));
        if (fileLocation.endsWith(XLS_FORMAT)) {
            data = readHSSFWorkbook(fis);
        } else if (fileLocation.endsWith(XLSX_FORMAT)) {
            data = readXSSFWorkbook(fis);
        }

        int maxNrCols = data.values().stream()
                .mapToInt(List::size)
                .max()
                .orElse(0);

        data.values().stream()
                .filter(ls -> ls.size() < maxNrCols)
                .forEach(ls -> IntStream.range(ls.size(), maxNrCols)
                        .forEach(i -> ls.add(new ExcelCellBean(""))));

        return data;
    }

    private String readCellContent(Cell cell) {
        String content;
        switch (cell.getCellType()) {
            case STRING:
                content = cell.getStringCellValue();
                break;
            case NUMERIC:
                if (DateUtil.isCellDateFormatted(cell)) {
                    content = cell.getDateCellValue() + "";
                } else {
                    content = cell.getNumericCellValue() + "";
                }
                break;
            case BOOLEAN:
                content = cell.getBooleanCellValue() + "";
                break;
            case FORMULA:
                content = cell.getCellFormula() + "";
                break;
            default:
                content = "";
        }
        return content;
    }

    private Map<Integer, List<ExcelCellBean>> readHSSFWorkbook(FileInputStream fis) throws IOException {
        Map<Integer, List<ExcelCellBean>> data = new HashMap<>();
        HSSFWorkbook workbook = null;
        try {
            workbook = new HSSFWorkbook(fis);
            HSSFSheet sheet = workbook.getSheetAt(0);
            for (int i = sheet.getFirstRowNum(); i <= sheet.getLastRowNum(); i++) {
                HSSFRow row = sheet.getRow(i);
                data.put(i, new ArrayList<>());
                if (row != null) {
                    for (int j = 0; j < row.getLastCellNum(); j++) {
                        HSSFCell cell = row.getCell(j);
                        if (cell != null) {
                            HSSFCellStyle cellStyle = cell.getCellStyle();

                            ExcelCellBean myCell = new ExcelCellBean("");

                            HSSFColor bgColor = cellStyle.getFillForegroundColorColor();
                            if (bgColor != null) {
                                short[] rgbColor = bgColor.getTriplet();
                                myCell.setBgColor("rgb(" + rgbColor[0] + "," + rgbColor[1] + "," + rgbColor[2] + ")");
                            }
                            HSSFFont font = cell.getCellStyle().getFont(workbook);
                            myCell.setTextSize(font.getFontHeightInPoints() + "");
                            if (font.getBold()) {
                                myCell.setTextWeight("bold");
                            }
                            HSSFColor textColor = font.getHSSFColor(workbook);
                            if (textColor != null) {
                                short[] rgbColor = textColor.getTriplet();
                                myCell.setTextColor("rgb(" + rgbColor[0] + "," + rgbColor[1] + "," + rgbColor[2] + ")");
                            }
                            myCell.setContent(readCellContent(cell));
                            data.get(i).add(myCell);
                        } else {
                            data.get(i).add(new ExcelCellBean(""));
                        }
                    }
                }
            }
        } catch(Exception e) {
            log.error("handle excel xls file error: {}", e.getMessage());
        } finally {
            if (workbook != null) {
                workbook.close();
            }
        }
        return data;
    }

    private Map<Integer, List<ExcelCellBean>> readXSSFWorkbook(FileInputStream fis) throws IOException {
        XSSFWorkbook workbook = null;
        Map<Integer, List<ExcelCellBean>> data = new HashMap<>();
        try {
            workbook = new XSSFWorkbook(fis);
            XSSFSheet sheet = workbook.getSheetAt(0);

            for (int i = sheet.getFirstRowNum(); i <= sheet.getLastRowNum(); i++) {
                XSSFRow row = sheet.getRow(i);
                data.put(i, new ArrayList<>());
                if (row != null) {
                    for (int j = 0; j < row.getLastCellNum(); j++) {
                        XSSFCell cell = row.getCell(j);
                        if (cell != null) {
                            XSSFCellStyle cellStyle = cell.getCellStyle();

                            ExcelCellBean myCell = new ExcelCellBean("");
                            XSSFColor bgColor = cellStyle.getFillForegroundColorColor();
                            if (bgColor != null) {
                                byte[] rgbColor = bgColor.getRGB();
                                myCell.setBgColor("rgb(" + (rgbColor[0] < 0 ? (rgbColor[0] + 0xff) : rgbColor[0]) + "," + (rgbColor[1] < 0 ? (rgbColor[1] + 0xff) : rgbColor[1]) + "," + (rgbColor[2] < 0 ? (rgbColor[2] + 0xff) : rgbColor[2]) + ")");
                            }
                            XSSFFont font = cellStyle.getFont();
                            myCell.setTextSize(font.getFontHeightInPoints() + "");
                            if (font.getBold()) {
                                myCell.setTextWeight("bold");
                            }
                            XSSFColor textColor = font.getXSSFColor();
                            if (textColor != null) {
                                byte[] rgbColor = textColor.getRGB();
                                myCell.setTextColor("rgb(" + (rgbColor[0] < 0 ? (rgbColor[0] + 0xff) : rgbColor[0]) + "," + (rgbColor[1] < 0 ? (rgbColor[1] + 0xff) : rgbColor[1]) + "," + (rgbColor[2] < 0 ? (rgbColor[2] + 0xff) : rgbColor[2]) + ")");
                            }
                            myCell.setContent(readCellContent(cell));
                            data.get(i).add(myCell);
                        } else {
                            data.get(i).add(new ExcelCellBean(""));
                        }
                    }
                }
            }
        } catch(Exception e) {
          log.error("handle excel xlsx file error: {}", e.getMessage());
        } finally {
            if (workbook != null) {
                workbook.close();
            }
        }
        return data;
    }
}
```

其中的部分依赖包没有声明，如有问题，可自行添加相关依赖即可。另外这里只读取第一个工作区。

### 2.3 Use

传入对应Excel的文件路径即可（当然这里使用的本地临时文件路径，如有从网络等下载读取，可以稍加改造）。

```java
Map<Integer, List<ExcelCellBean>> data = excelPOIUtil.readExcel(fileLocation);
```

记得进行错误处理，防止文件读取等过程出现异常而导致程序异常，出现不好的体验。

## 3. Conclusion

这里因为业务无写入导出需求，只提供了读取Excel的逻辑。总体而言，`Apache POI API `这个包提供了相关的接口，整体的操作是非常简单和便捷的，除了开始遇到的版本问题，后续逻辑很快就完成了。