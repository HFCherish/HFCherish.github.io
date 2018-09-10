---
title: java api lib for excel
toc: true
tags:
  - java
  - excel
date: 2018-07-25 10:37:45
---


# 可选 lib

*  [apache POI](http://poi.apache.org/components/document/index.html)：java 中最大众的 ，支持 `xls`、`xlsx`，提供接口来创建、读写 excel文件。
* [apache openOffice uno](http://www.openoffice.org/download/sdk/)：
* ~~[JExcel app](http://www.openoffice.org/download/sdk/)~~：这个功能强大，什么都可以做，但是可能收费，而且仅基于 windows + installed excel
* ~~[JExcelAPI](http://jexcelapi.sourceforge.net/)~~：轻量更易用，但似乎仅支持 `xls`，而且不支持复杂的 range、formular 计算操作。
* [javascript 版本的 JExcel](https://www.teamdev.com/jexcel)
* ~~[docx4j](https://www.docx4java.org/trac/docx4j)~~：其中 [xlsx4j](https://github.com/plutext/docx4j/tree/master/src/samples/xlsx4j/org/xlsx4j/samples) 是处理 excel 的，不过看起来功能比较简单。
* [microsoft excel VBA](https://msdn.microsoft.com/zh-cn/vba/vba-excel)：vba 是微软写的操作 office 软件的语言。所以可以利用这个 vba 写代码……

ref doc:

* [baeldung example for apache POI & JExcelAPI](http://www.baeldung.com/java-microsoft-excel)
* [Mkyong example for JExcelAPI](https://www.mkyong.com/java/jexcel-api-reading-and-writing-excel-file-in-java/)

# 核心操作支持

| lib                            | apache poi                                                   | apache openoffice                                            | JExcelAPI                                                    |
| ------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| read/create excel file         | ✔️                                                            | 可以                                                         | ✔️                                                            |
| read/write excel cells         | 1. 获取 sheet，遍历 row，遍历 row cells；                    | 可以                                                         | 1. 获取 sheet，可以根据 cell 行列数获取（`getCell(rowIndex, columnIndex)`） |
| compute formula for cells      | 可以（用 formular evaluator，参见[stackoverflow](https://stackoverflow.com/questions/5937373/using-apache-poi-hssf-how-can-i-refresh-all-formula-cells-at-once)） | 可以(calculateAll())                                         | 可能可以                                                     |
| write/read named cells of file | 可以（`workBook.getNamedAt("name")`，参见 [stackoverflow](https://stackoverflow.com/questions/33183144/apache-poi-update-cells-in-a-named-range), [官方文档](https://poi.apache.org/components/spreadsheet/quick-guide.html#NamedRanges)） | 可以（`namedRange`)                                          | NO                                                           |
| pros                           | 1. 使用广泛；2. 文档清晰；3. 支持很多复杂需求                | 1. 似乎功能比 poi 更简易，也更丰富些；2. 可以同时操作 openoffice 和 microoffice；3. 语言支持多，只要使用相应语言的 binding 就可以 | 1. 简单轻量；2. 接口友好                                     |
| cons                           | 接口不是很友好，访问起来比较麻烦                             | 1. 文档乱七八糟；2. 采用 uno，领域对象不太一致；3. 必须要安装 openoffice | 1. 功能太简单，复杂需求实现不了                              |

# 核心操作 example（POI）

[quick guide](https://poi.apache.org/components/spreadsheet/quick-guide.html#NamedRanges)

[源代码](https://github.com/HFCherish/learning-common/blob/master/common-excel/src/test/java/learning/common/excel/utils/XLSXTest.java)

```java
//        load excel
XSSFWorkbook workbook = new XSSFWorkbook(getFile("test.xlsx"));

//        get sheet by name
String sheetName = "Product Mix";
XSSFSheet sheet = workbook.getSheet(sheetName);
assertThat(sheet.getSheetName(), is(sheetName));

//        get cell by coordinate
XSSFCell tvsetNumber = getCell(sheet, "D4");
assertThat(tvsetNumber.getNumericCellValue(), closeTo(100, 0.1));

//        get formula cell by coordinate
XSSFCell total = getCell(sheet, "D13");
assertThat(total.getNumericCellValue(), closeTo(16000, 0.1));

//        change cell value
tvsetNumber.setCellValue(200);
assertThat(tvsetNumber.getNumericCellValue(), closeTo(200, 0.1));

//        refresh formulas
XSSFFormulaEvaluator.evaluateAllFormulaCells(workbook);
assertThat(total.getNumericCellValue(), closeTo(23500, 0.1));

//        write back to file
workbook.write(new FileOutputStream(getFile("update.xlsx")));

//        check data updated
XSSFWorkbook updateWorkBook = new XSSFWorkbook(getFile("update.xlsx"));
assertThat(getCell(updateWorkBook.getSheet(sheetName), "D4").getNumericCellValue(), closeTo(200, 0.1));
assertThat(getCell(updateWorkBook.getSheet(sheetName), "D13").getNumericCellValue(), closeTo(23500, 0.1));

//		  get cell by name
XSSFName test_name = workbook.getName("test_cell_name");
AreaReference areaReference = new AreaReference(test_name.getRefersToFormula(), SpreadsheetVersion.EXCEL2007);

CellReference firstCell = areaReference.getFirstCell();
XSSFSheet sheet = workbook.getSheet(firstCell.getSheetName());
XSSFRow row = sheet.getRow(firstCell.getRow());
XSSFCell cell = row.getCell(firstCell.getCol());

// get cell function
private XSSFCell getCell(XSSFSheet sheet, String cellCoordinate) {
    CellReference d4 = new CellReference(cellCoordinate);
    XSSFRow row = sheet.getRow(d4.getRow());
    return row.getCell(d4.getCol());
}
```
# 其他操作（POI）

## copy sheet

## xlsm to xlsx

