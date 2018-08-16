---
title: 亲测javaWeb的Excel的文件导入
comments: true
tags: [Spring,SpringMvc,MyBatis,java,POI]
categories: [JavaWeb]
date: 2018-08-11 17:34:31
---
    最近在做一个web项目，需要写一个Excel文件的导入，由于本猿是个刚入行不就的萌新，所以找到了一些文章研究了一下，但是讲的都比较混乱，于是在一边借阅一边的摸索中完成了导入，先说一下思路：
    1.首先是将需要导入的文档转换成流的形式。
	2.判断excel文件的类型是.xlsx还是.xls格式的，将对应的格式转换成Workbook所对应的格式，到了此处基本上一个excl文件就已经被导入了，并且存储为对应的excl格式了。
	3.此方法workbook.getSheetAt()可以得到你的这个文件上的所有的sheet，我的sheet默认只有一个所以我直接取的是sheet(0)。
	4.然后遍历此sheet得到所有的Row，将每行的数据add到list中并且返回。
	5.然后遍历每一行的，得到对应的元素上的信息，这里需要注意一下，这个row.getCell(0)方法是从0开始的，然后就得到了你所需要到倒数的数据了，剩下的事情就是业务的处理啦。
	下来我上一下，我的具体的做法。
一.环境和所需要引入的jar包。
	环境：
	ssm+maven
	具体的文件的上传可以查看我的上一篇博客
	[基于springMVC的文件上传和下载](http://blog.csdn.net/zhaokk_git/article/details/78339248)
	这里需要注意的是这里我们需要引入支持解析excl的工具类poi，我是直接maven直接导入的，如下：


```
<!--  导入和导出excel时需要的jar包 -->
		<dependency>
			<groupId>org.apache.poi</groupId>
			<artifactId>poi-ooxml</artifactId>
			<version>3.17</version>
		</dependency>
		<dependency>
			<groupId>org.apache.poi</groupId>
			<artifactId>poi</artifactId>
			<version>3.17</version>
		</dependency>
		<dependency>
			<groupId>org.apache.poi</groupId>
			<artifactId>poi-ooxml-schemas</artifactId>
			<version>3.17</version>
		</dependency>
		<dependency>
			<groupId>org.apache.xmlbeans</groupId>
			<artifactId>xmlbeans</artifactId>
			<version>2.6.0</version>
		</dependency>
```


到此，前期的环境的准备工作就已经全部完成了。
二.代码
 ## 导入的方法的接口 ##
	 这里需要注意下，这里的Result的是我自己定义的一个类，你们在引用的时候自己重写下就可以了，用Object就可以，返回个map就可以了。

```
import com.yonyouFintech.yangfan.commons.Result;
import com.yonyouFintech.yangfan.commons.util.DateUtil;
import com.yonyouFintech.yangfan.commons.util.ExcelUtil;
import com.yonyouFintech.yangfan.domain.YfBibliographic;
import com.yonyouFintech.yangfan.service.YfBibliographicService;
import org.apache.poi.ss.usermodel.Cell;
import org.apache.poi.ss.usermodel.Row;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

import javax.servlet.http.HttpServletRequest;
import java.io.InputStream;
import java.util.ArrayList;
import java.util.List;

@RestController
public class yfImportExclController {

    @Autowired
    private YfBibliographicService yfBibliographicService;

    @RequestMapping(value = "/exclImport",method = RequestMethod.POST)
    public Result importExcl(@RequestParam("file") MultipartFile excl, HttpServletRequest request){
        Result result = new Result();
        if(!excl.isEmpty()){//说明文件不为空
            try {
                String fileName = excl.getOriginalFilename();
                InputStream is = excl.getInputStream();//转化为流的形式
                List<YfBibliographic> listMer = new ArrayList<YfBibliographic>();
                List<Row> list = ExcelUtil.getExcelRead(fileName,is, true);
                //首先是读取行 也就是一行一行读，然后在取到列，遍历行里面的行，根据行得到列的值
                for (Row row : list) {
                    /****************得到每个元素的值start**********************/
                    Cell cell_0 = row.getCell(0);
                    Cell cell_1 = row.getCell(1);
                    Cell cell_2 = row.getCell(2);
                    Cell cell_3 = row.getCell(3);
                    /*****************得到每个元素的值end**********************/
                    /******************解析每个元素的值start*******************/
                    //得到列的值，也就是你需要解析的字段的值
                    String bookName = ExcelUtil.getValue(cell_0);
                    String   editor = ExcelUtil.getValue(cell_1);
                    String  express = ExcelUtil.getValue(cell_2);
                    String  version = ExcelUtil.getValue(cell_3);
                    /******************解析每个元素的值end*******************/
                    /****************将读取出来的数值进行包装start***********/
                    YfBibliographic yfBibliographic = new YfBibliographic();
                    yfBibliographic.setName(bookName);
                    yfBibliographic.setAuthor(editor);
                    yfBibliographic.setPress(express);
                    yfBibliographic.setEdition(version);
                    yfBibliographic.setStatus("1");
                    yfBibliographic.setExtend1(DateUtil.getCurDateStr());
                    listMer.add(yfBibliographic);
                    /**************将读取出来的数值进行包装end**************/
                }
                if(listMer.size()>0){
                    for (YfBibliographic item:listMer) {
                        yfBibliographicService.insertYfBibliographic(item);
                    }
                }
                result.setSuccess(true);
                result.setSuccessMessage("导入成功！");
            }catch (Exception e){
                e.printStackTrace();
                result.setSuccess(false);
                result.setErrorMessage("导入出现异常！");
            }
        }else{
            result.setSuccess(false);
            result.setErrorMessage("导入的文件为空！");
        }
        return  result;
    }
}

```
## 判断文件类型的工具类 ##

```
/**
 * @author zhaokk
 * @Date 2017-12-01
 * 工具类验证Excel文档
 */
public class WDWUtil {
    /**
     * @描述：是否是2003的excel，返回true是2003
     * @param filePath
     * @return
     */
    public static boolean isExcel2003(String filePath)  {
        return filePath.matches("^.+\\.(?i)(xls)$");
    }

    /**
     * @描述：是否是2007的excel，返回true是2007
     * @param filePath
     * @return
     */
    public static boolean isExcel2007(String filePath)  {
        return filePath.matches("^.+\\.(?i)(xlsx)$");
    }

    /**
     * 验证是否是EXCEL文件
     * @param filePath
     * @return
     */
    public static boolean validateExcel(String filePath){
        if (filePath == null || !(isExcel2003(filePath) || isExcel2007(filePath))){
            return false;
        }
        return true;
    }
}
```
## 获取excel表格，每行的数据的类 ##

```
import org.apache.poi.hssf.usermodel.HSSFCell;
import org.apache.poi.hssf.usermodel.HSSFWorkbook;
import org.apache.poi.ss.usermodel.Cell;
import org.apache.poi.ss.usermodel.Row;
import org.apache.poi.ss.usermodel.Sheet;
import org.apache.poi.ss.usermodel.Workbook;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import java.io.InputStream;
import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.List;

public class ExcelUtil {
    //读取文件的方法
    /**
     * 获取解析文件行数据
     * @param fileName : 文件地址
     * @param isTitle  : 是否过滤第一行解析
     * @return
     * @throws Exception
     */
    public static List<Row> getExcelRead(String fileName, InputStream is, boolean isTitle) throws Exception{
        try {
            //判断其兼容版本 调用了判断版本的方法
            Workbook workbook = getWorkbook(fileName,is);
            Sheet sheet = workbook.getSheetAt(0);
            int count = 0;
            List<Row> list = new ArrayList<Row>();
            for (Row row : sheet) {
                // 跳过第一行的目录
                if (count == 0 && isTitle) {
                    count++;
                    continue;
                }
                list.add(row);
            }
            return list;
        } catch (Exception e) {
            throw e;
        }
    }

//判断版本的方法

    public static Workbook getWorkbook(String fileName,InputStream is) throws Exception{
        Workbook workbook = null;
        try {
            /** 判断文件的类型，是2003还是2007 */
            boolean isExcel2003 = true;
            if (WDWUtil.isExcel2007(fileName)) {
                isExcel2003 = false;
            }

            if (isExcel2003) {
                workbook = new HSSFWorkbook(is);
            } else {
                workbook = new XSSFWorkbook(is);
            }
        } catch (Exception e) {
            throw e;
        }
        return workbook;
    }

    //得到celL值的方法：
    public static String getValue(Cell cell){
        if(cell.getCellType() == HSSFCell.CELL_TYPE_BOOLEAN){
            return String.valueOf(cell.getBooleanCellValue());
        }else if(cell.getCellType() == HSSFCell.CELL_TYPE_NUMERIC){
            double value = cell.getNumericCellValue();
            return new BigDecimal(value).toString();
        }else if (cell.getCellType() ==HSSFCell.CELL_TYPE_STRING){
            return String.valueOf(cell.getStringCellValue());
        }else{
            return String.valueOf(cell.getStringCellValue());
        }
    }


}

```
	以上，就是本猿的导入的方法和思路，亲测有效，完美运行，作为一个新手猿，可能会有很多不足，请诸君在阅览的时候，发现有不足的地方请多多指教，转载请注明出处。