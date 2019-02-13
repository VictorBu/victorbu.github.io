---
title: �� Web ҳ��ʹ�� VLC ������� m3u8 ��Ƶ�� (360 ����ģʽ)
date: 2019-01-22 16:40:00
updated: 2019-01-22 16:40:00
categories: [IT]
tags: [VLC]
---

# ����

��˾�и�����Ŀ��Ҫ������߲��� m3u8 ��Ƶ�������Ǹ�����֪��ʲôԭ��ʹ�� Video.js �� hls.js ���޷����ţ�����ҵ����������ʹ�� VLC �������(360 ����ģʽ��)

# ʾ������

```
<!DOCTYPE html>
<html>
<head>
<title>web camera</title>
<meta http-equiv="content-type" content="text/html; charset=UTF-8">
<script type="text/javascript">  
 //��������IE������ǣ����Ұ�װ��vlc������򷵻�true��  
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

    //��������firefox������ǣ����Ұ�װ��vlc������򷵻�true��  
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

    /* �������� */  
    function checkBrowser(){  
        var browser=navigator.appName;  
        var b_version=navigator.appVersion; 
        var version=parseFloat(b_version);
        var download_url = "https://download.videolan.org/pub/videolan/vlc/2.2.1/win32/";
        if ( browser=="Netscape"  && version>=4) {  
            if(isInsalledFFVLC()){  
                //alert("��װVLC���");  
            }else{  
                alert("δװVLC���,���Ȱ�װ���");
                location.href=download_url;  
            }  
        }else if(browser=="Microsoft Internet Explorer" && version>=4) {  
            if(isInsalledIEVLC()){  
                //alert("��װVLC���");  
            }else{  
                alert("δװVLC���,���Ȱ�װ���");
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

** ע�⣺���ܱ��ز���ϵͳ�� 32 λ���� 64 ���谲װ 32 λ�� VLC ���������������޷��ɹ���װ **

> �ο���[����vlc�����IP����ͷǶ����ҳ����ҳ����RTSP��](https://blog.csdn.net/Jeanphorn/article/details/46778391)