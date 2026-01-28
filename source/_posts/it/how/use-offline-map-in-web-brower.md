---
title: 在 Web 页面中使用离线地图
date: 2019-01-22 15:20:00
updated: 2019-01-22 15:20:00
categories: [IT]
tags: [GIS, Leaflet]
---

# 所需工具&插件：

1. [MapDownloader](https://pan.baidu.com/s/1gJUmxp1jBoe6ljhVz8S_Ug) (提取码: spx6)
1. [GISMysqlToLocalFile](https://pan.baidu.com/s/1mqajP0M8ENqMN3mR-noUyA) (提取码: vus6)
1. [Leaflet](https://leafletjs.com/)

# 操作

1. 参考：[java离线地图web GIS制作](http://www.cnblogs.com/kanyun/p/8571711.html) 下载好所需地图瓦片，本文以百度地图/深圳为例
1. 使用 Leaflet 加载地图瓦片：

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>web 版离线地图测试页面</title>
    
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.4.0/dist/leaflet.css" />    
    
</head>
<body>

    <div id="map" style="width:90%;height:600px;margin-top:30px;text-align:center;margin:0 auto;">
    </div>

    <script src="https://unpkg.com/leaflet@1.4.0/dist/leaflet.js"></script>
    
    <script type="text/javascript">
        window.onload=function () {
            var map = L.map('map').setView([22.56414255434805,114.153442382813], 11);
            L.tileLayer('./img/788865972/{z}/{x}/{y}.png'
                , {
                    minZoom: 10,
                    maxZoom: 12,
                    attribution: '<b style="color:#dddddd">百度地图</b>'
                })
            .addTo(map);
            
        };    
    </script>
    
</body>
</html>
```

 注：本文演示是将瓦片文件与代码文件放在一起，实际使用中最好自建瓦片地图服务。


## [在线演示](/code-snippet/js/offline-map/index.html)

 

> 参考：

1. [java离线地图web GIS制作](http://www.cnblogs.com/kanyun/p/8571711.html)

2. [chenwuwen/OffineMap](https://github.com/chenwuwen/OffineMap)

