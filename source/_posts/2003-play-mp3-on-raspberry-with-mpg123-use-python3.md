---
title: Python 使用 mpg123 播放声音
date: 2018-02-13 00:00:00
updated: 2017-02-13 00:00:00
categories: [IT]
tags: [树莓派, Python]
---


# 安装 mpg123

执行命令：`sudo apt-get install mpg123`


# 播放声音

```
import os

os.system('mpg123 test.mp3')

```


# 备注 1

1. 播放文件夹下的所有 mp3 文件：`mpg123 *.mp3`
1. 设置音量：`mpg123 --scale 200000 test.mp3`


# 备注 2：树莓派设置音频输出

1. 执行命令：`sudo raspi-config`
1. 选中 Advanced Options

    ![image](https://oss.x8y.cc/blog-img/2003/advanced_options.png)

1. 选中 Audio

    ![image](https://oss.x8y.cc/blog-img/2003/audio.png)

1. 选中对应的输出方式

    ![image](https://oss.x8y.cc/blog-img/2003/audio_output.png)

# 备注 3：安装 Python 3

1. 安装 Python 3

    1. 执行命令，安装 Python 3：`sudo apt-get install python3`
    1. 执行命令，安装 pip 3：`sudo apt-get install python3-pip`


1. 安装 Virtualenv (可选)

    1. 执行命令，安装 Virtualenv：`sudo pip3 install virtualenv`
    1. 执行命令，创建虚拟环境：`sudo virtualenv -p python3 env --no-site-packages`


参考
----

1. [Python发出警报声音简单播放声音beep在linux上](http://blog.csdn.net/u010918541/article/details/53227925)
1. [CLI volume option](https://sourceforge.net/p/mpg123/feature-requests/35)
1. [树莓派如何选择3.5mm耳机孔输出声音](https://jingyan.baidu.com/article/64d05a02220053de55f73bbc.html)