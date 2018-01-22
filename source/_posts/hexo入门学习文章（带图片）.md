---
title: hexo入门学习文章（带图片）
date: 2018-01-22 13:44:53
categories: HEXO
tags:
	HEXO
	IMAGE
permalink: hexo入门学习文章（带图片）
---
需要用到插件CodeFalling/hexo-asset-image。 
首先确认 _config.yml 中有 post_asset_folder:true 。 
<!-- more -->
安装此插件：在 hexo 目录，执行
这样的目录结构（目录名和文章名一致）
```
npm install https://github.com/CodeFalling/hexo-asset-image --save
```
引入图片代码
```
![lbxx](hexo入门学习文章（带图片）/01.png)
```
就可以插入图片。会将图片和文章生成到一个目录下。同时，生成的 html 是
```
<img src="/2018/1/22/hexo入门学习文章（带图片）/01.png" alt="01">
```
## 使用方法
```
![lbxx](hexo入门学习文章（带图片）/01.png)
```
## 效果如下
![lbxx](hexo入门学习文章（带图片）/01.png)