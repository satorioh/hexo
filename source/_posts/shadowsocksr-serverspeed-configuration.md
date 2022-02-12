---
title: ShadowsocksR+锐速配置及优化
date: 2016-12-12 10:14:03
categories: Linux
tags:
- ShadowsocksR
- 锐速
permalink: shadowsocksr-serverspeed-configuration
---
**前言**：用科学上网服务有段时间了，最早搬瓦工推出年付2.9刀机子的时候，就出手买了2台，一键安装，甚是方便，性价比超高。但最近感觉用git、看油土鳖什么的，速度不是很理想，再考虑到安全因素，还是打算自建一台server，求人不如求己嘛，搭建的时候做了下记录，于是便有了此文<!--more-->

**VPS商**：看了多方评测，这里选用了Vultr JP的机子，也是极具性价比，带宽应该是G口的

第一次deploy，人品不错，分到了108开头的IP（因为论坛上有说45开头的不稳定，其实我感觉应该关系不大吧），ping值还可以，平均120ms

**OS**：这里以CentOS 7 x64为例，当然CentOS 6也可以，而且兼容性可能更好，遇到问题更容易找到资料

**软件**：主体是SSR，SS的增强版，带混淆插件，有效防止gfw找上门，且兼容原版；加速软件是锐速，单边加速，只需在服务端部署，虽说不如FS那种双边加速来得爽，但能少装一个客户端，清爽。另外还有一些TCP协议的优化配置，也建议一起加上

**参考教程**：主要是官方Github上的[单用户版安装教程](https://github.com/breakwa11/shadowsocks-rss/wiki/Server-Setup)
##### 1.VPS安全设置
可以参考我之前的文章[VPS安全设置](http://www.jianshu.com/p/6d6a00ee6c23)，先做下安全配置，免得被入侵了。
防火墙的话，不建议直接disable，但如果你怕后续有什么问题，可以参考下面的命令，先关闭掉

```
systemctl stop firewalld.service #CentOS 7 停止firewall
systemctl disable firewalld.service #CentOS 7 禁止firewall开机启动
service iptables stop #CentOS 6 下停止firewall
chkconfig iptables off #CentOS 6 下禁止firewall开机启动
```
##### 2.关闭Selinux
这个还是推荐先关闭吧，高手的话，可以略过
```
setenforce 0 #临时关闭
vim /etc/sysconfig/selinux
SELINUX=disabled #改为disabled，永久关闭
```
##### 3.检查python版本，要有2.6 or 2.7
因为SSR基于python，所以确认下有装
```
python --version
```
##### 4.查看并调整时间和时区（参考[CentOS 7时区设置](http://www.centoscn.com/CentOS/config/2015/0723/5901.html)）
默认不是东八区，所以手动改下，到时看log档，会比较清楚
```
date #查看
timedatectl set-timezone Asia/Shanghai #CentOS 7 修改时区
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime #CentOS 6 修改时区
```
##### 5.安装ShadowsocksR（参考[ShadowsocksR 服务端安装教程](https://github.com/breakwa11/shadowsocks-rss/wiki/Server-Setup)、[配置文件介绍]([https://github.com/breakwa11/shadowsocks-rss/wiki/config.json])、[协议和混淆插件说明]([https://github.com/breakwa11/shadowsocks-rss/wiki/obfs])）
```
yum install git //安装git
git clone -b manyuser https://github.com/breakwa11/shadowsocks.git //获取源码
cd ~/shadowsocks 
bash initcfg.sh //初始化根目录
vim user-config.json //编辑配置文件
cd ~/shadowsocks/shadowsocks //进入子目录（单用户）
python server.py -d start //后台启动
python server.py -d stop/restart //停止或重启
tail -f /var/log/shadowsocks.log //查看日志
git pull //更新源码
```
其实安装还蛮简单的，如果想要长期使用，协议还是推荐用auth_aes128_md5，混淆插件推荐tls1.2_ticket_auth
##### 6.配置开机自启动脚本(参考[启动脚本](https://github.com/breakwa11/shadowsocks-rss/wiki/System-startup-script))
这里需要注意，ExecStart和ExecStop里的server.py，和user-config文件路径，需要和你实际的文件一一对应，如果你是在root下直接下载安装的SSR，那就应该像下面这样：
```
[Unit]
Description=Start or stop the ShadowsocksR server
After=network.target
Wants=network.target

[Service]
Type=forking
PIDFile=/var/run/shadowsocks.pid
ExecStart=/usr/bin/python /root/shadowsocks/shadowsocks/server.py --pid-file /var/run/shadowsocks.pid -c /root/shadowsocks/user-config.json -d start
ExecStop=/usr/bin/python /root/shadowsocks/shadowsocks/server.py --pid-file /var/run/shadowsocks.pid -c /root/shadowsocks/user-config.json -d stop
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=always

[Install]
WantedBy=multi-user.target
```
然后执行`systemctl enable shadowsocks.service && systemctl start shadowsocks.service`，看是否报错，没有就OK
##### 7.锐速安装（参考[VPS科学上网教程](https://jasper-1024.github.io/2016/06/26/VPS%E7%A7%91%E5%AD%A6%E4%B8%8A%E7%BD%91%E6%95%99%E7%A8%8B%E7%B3%BB%E5%88%97/)）
a.【可选】更换内核(参考[CentOS更换内核](https://www.91yun.org/archives/795))
这步其实可选，锐速默认会去匹配最适合你当前内核的版本，然后安装，如果遇到问题，可以尝试更换内核，具体参考上面链接教程
```
rpm -ivh http://buildlogs.centos.org/c7.1511.00/kernel/20151119220809/3.10.0-327.el7.x86_64/kernel-3.10.0-327.el7.x86_64.rpm --force //更换内核版本
rpm -qa | grep kernel //查看内核是否安装成功
reboot //重启
uname -r //查看当前内核版本
yum remove xxxx //移除多余内核启动项
```
b.一键安装锐速（参考[一键自动安装包](https://www.91yun.org/archives/683)）
```
wget -N --no-check-certificate https://raw.githubusercontent.com/91yun/serverspeeder/master/serverspeeder-all.sh && bash serverspeeder-all.sh //安装
chattr -i /serverspeeder/etc/apx* && /serverspeeder/bin/serverSpeeder.sh uninstall -f //这是卸载
```
c.锐速常用命令
```
/serverspeeder/bin/serverSpeeder.sh start/stop/restart
service serverSpeeder status
```
##### 8.TCP优化和锐速优化（参考[TCP优化配置](https://www.91yun.org/archives/545)）
a.锐速优化的话，只需修改配置文件的一个地方：
```
vim /etc/serverSpeeder.conf
advinacc = 1 #改为1，优化入栈流量
```

b.TCP优化的话，参考上面的链接，非常详细了，记得最后将hybla算法写入rc.local，使其自启动，命令如下：
```
vim /etc/rc.local
/sbin/modprobe tcp_hybla //添加命令
chmod +x /etc/rc.d/rc.local //赋予启动权限
```
c.之后重启ssr和锐速，或者直接重启VPS
##### 9.重启回来检查各服务是否自启动正常
```
sysctl net.ipv4.tcp_available_congestion_control //检查hybla模块是否加载，有显示hybla字样就OK
service serverSpeeder status //查看锐速运行状态
service shadowsocks status //查看SSR运行状态
```

##### 10.【可选】更改为chacha20加密
user-config.json里，加密方式默认是aes-256-cfb，可以更改为现在比较流行的chacha20加密，该加密对手机CPU负荷较低，具体可参考[为SS启用chacha20加密算法](https://www.91yun.org/zh/archives/1232)，配置也比较简单
