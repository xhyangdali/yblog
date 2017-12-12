---
title:	Ajax提交之后，Method从POST变成GET
date: 2017-12-12 12:52:22
categories: AJAX
tags: 
	Jsonp 
	Ajax
permalink: Jsonp-method-not-post
---
## Ajax提交之后，Method从POST变成GET
开发过程中遇到一个小却很困惑的问题：
	进行ajax请求时，用的是post但是系统会自动给转成get,一直不明白。现在找到原因了。
<!-- more -->

## 原因如下
---
	因为你的 dataType 是 jsonp 而不是 json 
	jsonp不支持POST跨域，所以会自动给你转成GET

	记录下来，希望可以帮助大家~~~~
