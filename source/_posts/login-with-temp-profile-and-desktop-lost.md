---
title: win7桌面丢失/已使用临时配置登录
date: 2018-04-13 10:27:42
categories: Windows
tags:
- windows
- win7
permalink: login-with-temp-profile-and-desktop-lost
---
#### 问题描述：
- 登录时，发现桌面图标都没了，和新的一样
- 系统右下角提示“已使用临时配置登录”
<!--more-->
#### 原因：
- 可能是用户配置文件损坏
- 系统异常

#### 解决方法：
- administrator登录，开启注册表（regedit）
- 定位到:`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList`
- 假设正确的profile是roubin，错误的是roubin_ME
- 通过ProfileImagePath，找到roubin_ME对应的SID
- 将此SID下的ProfileImagePath修改为roubin的档案文件夹路径（C:\Users\roubin）
- 到C:\Users下，删除roubin_ME使用者档案文件夹，Temp文件夹也可以一并删除
- 重启，再使用roubin登录，OK

参考链接：

[win7 您已使用临时配置文件登陆以及恢复的解决办法](http://blog.sina.com.cn/s/blog_49cea9d60100mjf3.html)
