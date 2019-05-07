---
title: ThinkPHP实现一键清除缓存方法
date: 2018-08-06 12:54:01
categories: PHP
tags: 
	ThinkPHP 
	PHP
permalink: ThinkPHP实现一键清除缓存方法
---
清除缓存的功能，缓存是为了减轻服务器的压力而产生的,对此，我们就来实现一个ThinkPHP的清理缓存的功能。
<!-- more -->
## 代码
```
ThinkPHP后台执行的代码:
//获取要清楚的目录和目录所在的绝对路径
 public function cache(){
  ////前台用ajax get方式进行提交的，这里是先判断一下
  if($_POST['type']){
   //得到传递过来的值
   $type=$_POST['type'];
   //将传递过来的值进行切割，我是用“-”进行切割的
   $name=explode('-', $type);
   //得到切割的条数，便于下面循环
   $count=count($name);
   //循环调用上面的方法
   for ($i=0;$i<$count;$i++){
    //得到文件的绝对路径
    $abs_dir=dirname(dirname(dirname(dirname(__FILE__))));
    //组合路径
    $pa=$abs_dir.'indexRuntime';
    $runtime=$abs_dir.'indexRuntime~runtime.php';
    if(file_exists($runtime))//判断 文件是否存在
    {
     unlink($runtime);//进行文件删除
    }
 //调用删除文件夹下所有文件的方法
    $this->rmFile($pa,$name[$i]); 
   }
   //给出提示信息
   $this->ajaxReturn(1,'清除成功',1);
  }else{
   $this->display();
  }
 }
 public function rmFile($path,$fileName){//删除执行的方法
  //去除空格
  $path = preg_replace('/(/){2,}|{}{1,}/','/',$path); 
  //得到完整目录 
  $path.= $fileName;
  //判断此文件是否为一个文件目录
  if(is_dir($path)){
   //打开文件
   if ($dh = opendir($path)){
    //遍历文件目录名称
     while (($file = readdir($dh)) != false){
      //逐一进行删除
      unlink($path.''.$file);
      }
      //关闭文件
      closedir($dh);
    } 
   }
 }
 
前台页面部分代码如下：
<script type="text/javascript" src="__PUBLIC__/admin/js/jquery.js"></script>
<script type="test/javascript">
$(function(){
$('#button').click(function(){
if(confirm("确认要清除缓存？")){
var $type=$('#type').val();
var $mess=$('#mess');
$.post('__URL__/clear',{type:$type},function(data){
alert("缓存清理成功");
});
}else{
return false;
}
});
});
</script>
```