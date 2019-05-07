---
title: PHP导出Excel
date: 2018-02-13 14:25:10
categories: PHP
tags:
	EXCEL
	PHP
permalink: PHP导出Excel
---
这些天在使用PHPExcel导出数据时，5000条数据竟然挂了。
研究之后有些明悟，PHPExcel做了很多处理，我在这里理解为渲染，就会占用过多的空间，‘膨胀’的空间导致内存使用过大，就挂了。
<!-- more -->
简单的导出操作，没有必要使用PHPExcel。贴出代码：
```
/* 
*处理Excel导出 
*@param $datas array 设置表格数据 
*@param $titlename string 设置head 
*@param $title string 设置表头 
*/ 
public function excelData($datas,$titlename,$title,$filename){ 
    $str = "<html xmlns:o=\"urn:schemas-microsoft-com:office:office\"\r\nxmlns:x=\"urn:schemas-microsoft-com:office:excel\"\r\nxmlns=\"http://www.w3.org/TR/REC-html40\">\r\n<head>\r\n<meta http-equiv=Content-Type content=\"text/html; charset=utf-8\">\r\n</head>\r\n<body>"; 
    $str .="<table border=1><head>".$titlename."</head>"; 
    $str .= $title; 
    foreach ($datas  as $key=> $rt ) 
    { 
        $str .= "<tr>"; 
        foreach ( $rt as $k => $v ) 
        { 
            $str .= "<td>{$v}</td>"; 
        } 
        $str .= "</tr>\n"; 
    } 
    $str .= "</table></body></html>"; 
    header( "Content-Type: application/vnd.ms-excel; name='excel'" ); 
    header( "Content-type: application/octet-stream" ); 
    header( "Content-Disposition: attachment; filename=".$filename ); 
    header( "Cache-Control: must-revalidate, post-check=0, pre-check=0" ); 
    header( "Pragma: no-cache" ); 
    header( "Expires: 0" ); 
    exit( $str ); 
}
```
html的表格转换excel的表格；此种方法适应于设置各种单元格的显示，合并，只需设置html的table，设置css就能导出各式各样的excel模板。
导出一个带表头，表头带颜色，设置字体大小，居中，排版适中；
```
$dataResult = array();      //todo:导出数据（自行设置） 
$headTitle = "XX公司 XX记录"; 
$title = "XX公司记录"; 
$headtitle= "<tr style='height:50px;border-style:none;><th border=\"0\" style='height:60px;width:270px;font-size:22px;' colspan='11' >{$headTitle}</th></tr>"; 
$titlename = "<tr> 
               <th style='width:70px;' >合作商户</th> 
               <th style='width:70px;' >会员卡号</th> 
               <th style='width:70px;'>车主姓名</th> 
               <th style='width:150px;'>手机号</th> 
               <th style='width:70px;'>车牌号</th> 
               <th style='width:100px;'>类型</th> 
               <th style='width:70px;'>名称</th> 
               <th style='width:70px;'>面值</th> 
               <th style='width:70px;'>数量</th> 
               <th style='width:70px;'>时间</th> 
               <th style='width:90px;'>截至有效期</th> 
           </tr>"; 
           $filename = $title.".xls"; 
       $this->excelData($dataResult,$titlename,$headtitle,$filename); 
```