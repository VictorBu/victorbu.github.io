---
title: 在 Web 页面使用 VLC 插件播放 m3u8 视频流 (360 极速模式)
date: 2019-01-22 16:40:00
updated: 2019-01-22 16:40:00
categories: [IT]
tags: [VLC]
---

# 背景

公司有个旧项目需要添加在线播放 m3u8 视频流，但是该流不知道什么原因使用 Video.js 或 hls.js 均无法播放，最后找到解决方案可使用 VLC 插件播放(360 极速模式下)

# 示例代码

```
<!DOCTYPE html>
<html>
<head>
<title>web camera</title>
<meta http-equiv="content-type" content="text/html; charset=UTF-8">
<script type="text/javascript">  
 //仅适用于IE浏览器是，并且安装有vlc插件，则返回true；  
    function isInsalledIEVLC(){    
        var vlcObj = null;  
        var vlcInstalled= false;   
        try {  
            vlcObj = new ActiveXObject("VideoLAN.Vlcplugin.1");   
            if( vlcObj != null ){   
                vlcInstalled = true   
            }  
        } catch (e) {  
            vlcInstalled= false;  
        }          
        return vlcInstalled;  
    }   

    //仅适用于firefox浏览器是，并且安装有vlc插件，则返回true；  
    function isInsalledFFVLC(){  
         var numPlugins=navigator.plugins.length;  
         for  (i=0;i<numPlugins;i++){   
              plugin=navigator.plugins[i];  
              if(plugin.name.indexOf("VideoLAN") > -1 || plugin.name.indexOf("VLC") > -1){   
                 return true;  
            }  
         }  
         return false;  
    }  

    /* 浏览器检测 */  
    function checkBrowser(){  
        var browser=navigator.appName;  
        var b_version=navigator.appVersion; 
        var version=parseFloat(b_version);
        var download_url = "https://download.videolan.org/pub/videolan/vlc/2.2.1/win32/";
        if ( browser=="Netscape"  && version>=4) {  
            if(isInsalledFFVLC()){  
                //alert("已装VLC插件");  
            }else{  
                alert("未装VLC插件,请先安装插件");
                location.href=download_url;  
            }  
        }else if(browser=="Microsoft Internet Explorer" && version>=4) {  
            if(isInsalledIEVLC()){  
                //alert("已装VLC插件");  
            }else{  
                alert("未装VLC插件,请先安装插件");
                location.href=download_url;
            }  
        }  
    }  
</script>


</head>

<body bgcolor="white" text="black" onload="checkBrowser();">

<embed type="application/x-vlc-plugin" pluginspage="http://www.videola.org"
    width="640" height="480" id="vlc" version="VideoLAN.VLCPlugin.2" autoplay="yes" loop="no" 
    target="https://video-dev.github.io/streams/x36xhzz/x36xhzz.m3u8" >

</body>
</html>
```

** 注意：不管本地操作系统是 32 位还是 64 均需安装 32 位的 VLC 播放器，否则插件无法成功安装 **

> 参考：[利用vlc插件将IP摄像头嵌入网页和网页播放RTSP流](https://blog.csdn.net/Jeanphorn/article/details/46778391)