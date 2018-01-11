---
title: 使用PHPQRCode生成二维码
date: 2018-01-11 16:52:45
categories: PHP
tags:
	PhpQRCode
permalink: 使用PHPQRCode生成二维码
---
## PHP二维码生成
在项目中使用的到了二维码，实现如下
<!-- more -->
## 实现代码
```
		$data = 'www.baidu.com'; 
        $level = 'L';// 纠错级别：L、M、Q、H
        $size = 6;// 点的大小：1到10,用于手机端4就可以了
        
        include EXTEND_PATH.'org/phpqrcode/phpqrcode.php';
        $QRcode = new \QRcode();
        ob_start();
        $QRcode->png($data,false,$level,$size);
        $imageString = base64_encode(ob_get_contents());
        ob_end_clean();
        //$path=ROOT_PATH.'public/static/images/qrcode/';
        //$QRcode->png($data,$path.$fileName,$level,$size);// 生成本地图片
        return $imageString;
```
上面的代码是生成一个base64的字符串，在页面如下使用就可以显示图片了。
```
<img src="data:image/png;base64,这里是base64编码内容" />
```
## QRcode::png方法参数说明
1.第一个参数$text，就是上面代码里的URL网址参数，
2.第二个参数$outfile默认为否，不生成文件，只将二维码图片返回，否则需要给出存放生成二维码图片的路径
3.第三个参数$level默认为L，这个参数可传递的值分别是L(QR_ECLEVEL_L，7%)，M(QR_ECLEVEL_M，15%)，Q(QR_ECLEVEL_Q，25%)，H(QR_ECLEVEL_H，30%)。这个参数控制二维码容错率，不同的参数表示二维码可被覆盖的区域百分比。利用二维维码的容错率，我们可以将头像放置在生成的二维码图片任何区域。
4.第四个参数$size，控制生成图片的大小，默认为4
5.第五个参数$margin，控制生成二维码的空白区域大小
6.第六个参数$saveandprint，保存二维码图片并显示出来，$outfile必须传递图片路径。

第二个参数默认是false，方法返回的是二进制的图片流。生成在缓冲区的，在页面输出的时候会从缓冲区送到浏览器。所以在代码中是使用log输出是不会记录在日志中的，也不需要使用echo进行内容输出。所以直接使用base64_encode(QRcode::png)是没有用的。
这里使用到了ob_start()方法，打开输出缓冲区，所有的输出信息不在直接发送到浏览器，而是保存在输出缓冲区里面。这里就是把生成的图片流从缓冲区保存到内存对象上，使用base64_encode变成编码字符串，通过json返回给页面。
看一下直接使用QRcode::png返回图片流到浏览器

简单说一下ob:
  ob是output buffering的简称，而不是output cache，ob用对了是能对速度有一定的帮助，但是盲目的加上ob函数，只会增加CPU额外的负担。说说ob的基本作用:
1.防止在浏览器有输出之后再使用setcookie，或者header，session_start函数造成的错误。（我本以为最开始说的代码是这样的作用，但后来朋友说不是的），其实这样的用法少用为好，养成良好的代码习惯。
2.捕捉对一些不可获取的函数的输出，比如phpinfo会输出一大堆的HTML，但是我们无法用一个变量例如$info=phpinfo();来捕捉，这时候ob就管用了。
3.对输出的内容进行处理，例如进行gzip压缩，例如进行简繁转换，例如进行一些字符串替换。
4.生成静态文件，其实就是捕捉整页的输出，然后存成文件，经常在生成HTML，或者整页缓存中使用。
对于刚才说的第三点中的GZIP压缩，可能是很多人想用，却没有真真用上的，加一个ob_gzhandler这个回调函数就可以了。

```
ob_start(ob_gzhandler);
内容
echo ob_get_contents() ;
ob_end_flush();
```