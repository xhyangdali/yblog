---
title: HBuilder实现软件自动升级
date: 2019-05-07 11:36:44
categories: Hbuilder
tags: 
	H5 
	H5+app
permalink: HBuilder实现软件自动升级
---
移动APP开发好后需要实现软件自动升级功能，发现HBuilder具有“App资源在线升级更新”的功能，遂研究之。
经过一番测试，在源码思想的基础之上对其进行了优化。
<!-- more -->
## 源码
```
	var wgtVer = null;  
function plusReady(){  
    // 获取本地应用资源版本号  
    plus.runtime.getProperty(plus.runtime.appid,function(inf){  
        wgtVer=inf.version;  
        console.log("当前应用版本：" + wgtVer);  
        console.log("=================版本测试=================");  
    });  
}  
  
if(window.plus){  
    plusReady();  
}else{  
    document.addEventListener('plusready',plusReady,false);  
    document.addEventListener('plusready',checkUpdate,false);  
}  
  
// 检测更新  
var checkUrl="http://www.weimingcloud.cn/lmapp/versionCheck.html";  
function checkUpdate(){  
    plus.nativeUI.showWaiting("检测更新...");  
*       $ionicLoading.show({  
        template: "检测更新..."  
    });  
    $timeout(function() {  
        $ionicLoading.hide();  
    }, 1200);*/  
    var xhr = new XMLHttpRequest();  
    xhr.onreadystatechange = function(){  
        switch(xhr.readyState){  
            case 4:  
            plus.nativeUI.closeWaiting();  
            if(xhr.status == 200){  
                console.log("检测更新成功：" + xhr.responseText);  
                // 读取最新版本号  
                var newVer = xhr.responseText;  
                console.log("最新版本:" + newVer);  
                if(wgtVer && newVer && (wgtVer != newVer)){  
                    // H5 plus事件处理,弹出提示信息对话框  
                    plus.nativeUI.confirm("\"立马送药\"检测到新版本,是否更新?", function(e) {  
                        if(e.index == 0){  
                            console.log("确定！");  
                            downWgt(); // 下载升级包  
                        }  
                    }, "                  立马送药", ["确定", "取消"]);  
                }else{  
                    plus.nativeUI.alert("无新版本可更新！");  
                }  
            }else{  
                console.log("检测更新失败！");  
                plus.nativeUI.alert("检测更新失败！");  
            }  
            break;  
            default:  
            break;  
        }  
    }  
    xhr.open('GET',checkUrl);  
    xhr.send();  
}  
  
// 下载wgt文件  
var wgtUrl = "http://www.weimingcloud.cn/lmapp/files/download/H5202FBD5.wgt";  
  
function downWgt(){  
    plus.nativeUI.showWaiting("下载wgt文件...");  
              
    plus.downloader.createDownload( wgtUrl, {filename:"_doc/update/"}, function(d,status){  
        if ( status == 200 ) {   
            console.log("下载wgt成功："+d.filename);  
            installWgt(d.filename); // 安装wgt包  
        } else {  
            console.log("下载wgt失败！");  
            plus.nativeUI.alert("下载wgt失败！");  
        }  
        plus.nativeUI.closeWaiting();  
    }).start();  
}  
  
// 更新应用资源  
function installWgt(path){  
    plus.nativeUI.showWaiting("安装wgt文件...");  
    // force:false进行版本号校验，如果将要安装应用的版本号不高于现有应用的版本号则终止安装，并返回安装失败  
    plus.runtime.install(path,{force:false},function(){  
        plus.nativeUI.closeWaiting();  
        console.log("安装wgt文件成功！");  
        plus.nativeUI.alert("应用资源更新完成！",function(){  
            plus.runtime.restart();  
        });  
    },function(e){  
        plus.nativeUI.closeWaiting();  
        console.log("安装wgt文件失败[" + e.code + "]：" + e.message);  
        plus.nativeUI.alert("安装wgt文件失败[" + e.code + "]：" + e.message);  
    });  
} 
```
## 问题
      注:确实在文件名上出问题,同一wgt文件名多次升级则出错提示了,即使提示"应用资源更新完成！" ,但版本号还是没更新的,因此同一wgt文件名只能使用一次, 这不知是哪里的bug.

      果然是这个问题，更新包的名称不能重复，Android上第一次用了update.wgt。那么第二次就不能用这个名字了，得换一个名字，IOS是好的。

      更新完成后，再次进入APP，发现版本号没变，还是原来的，接着有时更新....

      遇到了上述问题，通过以上方法还是未能解决。难道这本身就是HBuilder的一个BUG?

      检测更新更好的模式应该是客户端提交本地应用资源版本号到升级服务器，由升级服务器判断是否可更新并且返回App升级资源包下载地址，避免在客户端写资源下载地址；

      更新时可以在后台静默下载，下次启动是直接更新，避免更新时打断用户操作。

    使用官方Demo可以，怀疑是自己的wgt出错。

    升级第一次成功，第二次也成功了！打成包试试.....1.0、2.0..格式可以。