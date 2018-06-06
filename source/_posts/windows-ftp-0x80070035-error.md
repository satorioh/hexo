---
title: Win7 FTP无法访问（0x80070035错误）修复方法
date: 2018-06-06 14:34:16
categories: Windows
tags:
- windows
- win7
permalink: windows-ftp-0x80070035-error
---
#### 一、问题描述：

win+R，在“运行”中输入ftp地址\\\atsz-ftp-03，点确定后，弹出错误提示"windows无法找到此路径"，报错0x80070035
<!--more-->

#### 二、排错过程
- google "win7 ftp 0x80070035"，搜到微软官方的一篇解决文章，建议用户尝试重启services中的TCP/IP NetBIOS Helper服务
- win+R，输入services.msc，找到此服务，右键点击启用，但报错“错误1068：相依性服务或组件未启动”
- 点击此服务的相依性标签页，发现需要依赖Ancilary Function Driver for Winsock
- 于是在命令行中，输入`Net Start AFD`，尝试启动此服务，但结果提示此服务已经启动
- 与正常电脑对比TCP/IP NetBIOS Helper服务依赖后，发现异常电脑缺少NetBT服务
- 对比正常电脑，发现异常电脑的C:/Windows/System32/drivers下，缺失netbt.sys文件

#### 三、问题解决
- 从正常电脑的C:/Windows/System32/drivers下，拷贝一份netbt.sys文件，到异常电脑的相同路径下
- 重启TCP/IP NetBIOS Helper服务，即可恢复正常

#### 四、问题原因
大部分异常电脑都安装了腾讯管家、360卫士等，怀疑其加速优化功能导致问题

参考链接：

[DHCP client won't start due to dependency](https://answers.microsoft.com/en-us/windows/forum/windows_xp-networking/dhcp-client-wont-start-due-to-dependency/91d41140-663f-4f58-9a04-9eaab2c40884)
