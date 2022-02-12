---
title: VPS安全措施
date: 2017-12-13 10:14:03
categories: Linux
tags:
- VPS
- 安全
permalink: vps-security-measures
---
##### 1. 配置SSH安全访问密钥，关闭密码登录
a.参考[SecureCRT密钥连接Linux](http://edges.blog.51cto.com/705035/581346/)，使用SecureCRT在本机生成公私密钥
b.在VPS对应的用户目录下，新建.ssh文件夹，并上传公钥，然后更名为authorized_keys，并修改权限，如下<!--more-->
```
mkdir ~/.ssh #如果当前用户目录下没有 .ssh 目录，就先创建目录
chmod 700 ~/.ssh
mv id_rsa.pub ~/.ssh
cd .ssh
mv id_rsa.pub authorized_keys
chmod 600 authorized_keys
```
c.关闭ssh密码登录
```
vim /etc/ssh/sshd_config
PasswordAuthentication no #此处改为no
```
d.【可选】添加普通用户
```
useradd roubin
passwd roubin
```
e.【可选】禁止root登陆
```
vim /etc/ssh/sshd_config
PermitRootLogin no  #此处改为no
```
f.重启ssh服务
```
service sshd restart
```
g.备份公私密钥

##### 2.更改SSH端口及设置
```
vim /etc/ssh/sshd_config
Port 22222  #更改默认端口号
MaxAuthTries 5
PermitEmptyPasswords no  #不允许空密码
service sshd reload
iptables -I INPUT -p tcp --dport 22222 -j ACCEPT #CentOS 6 中防火墙开启对应端口
firewall-cmd --zone=public --add-port=22222/tcp --permanent #CentOS 7 中防火墙开启对应端口
```
##### 3.锁定口令文件
```
[root@localhost /]# chattr +i /etc/passwd
[root@localhost /]# chattr +i /etc/shadow
[root@localhost /]# chattr +i /etc/group
[root@localhost /]# chattr +i /etc/gshadow
```
##### 4.安装fail2ban防止暴力破解
参考[fail2ban安装](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-centos-6)
```
yum install -y fail2ban
cp -pf /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
vim /etc/fail2ban/jail.local

 [sshd]
enabled = trueport = 22222
logpath = %(sshd_log)s
backend = %(sshd_backend)s
filter = sshd
action = iptables[name=SSH, port=22222, protocol=tcp] sendmail-whois[name=SSH, dest=root, sender=fail2ban@example.com]
logpath = /var/log/secure
maxretry = 3
```

##### 5.启用iptables
参考[[Linux上iptables防火墙的基本应用教程](https://www.vpser.net/security/linux-iptables.html)](https://www.vpser.net/security/linux-iptables.html)
```
# 清除已有iptables规则
iptables -F
# 允许本地回环接口(即运行本机访问本机)
iptables -A INPUT -i lo -j ACCEPT
# 允许已建立的或相关连的通行
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
#允许所有本机向外的访问
iptables -A OUTPUT -j ACCEPT
# 允许访问22222(SSH)端口，以下几条相同，分别是22222,80,443端口的访问
iptables -A INPUT -p tcp --dport 22222 -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
#如果有其他端口的话，规则也类似，稍微修改上述语句就行
#允许ping
iptables -A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT
#禁止其他未允许的规则访问（注意：如果22端口未加入允许规则，SSH链接会直接断开。）
iptables -A INPUT -j REJECT 
iptables -A FORWARD -j REJECT
#保存防火墙规则
service iptables save
#设置防火墙开机启动
chkconfig --level 345 iptables on
```

##### 6.禁用ipv6
```
#编辑/etc/sysconfig/network添加行：
NETWORKING_IPV6=no
#修改/etc/hosts,把ipv6本地主机名解析的注释掉（可选）：

#::1 localhost localhost6 localhost6.localdomain6

#禁止系统加载ipv6相关模块，创建modprobe关于禁用ipv6的设定文件/etc/modprobe.d/disable_ipv6.conf(名字随便起)（RHEL6.0之后没有/etc/modprobe.conf这个文件），内容如下，三选其一（本次使用的第一种）：
alias net-pf-10 off
options ipv6 disable=1
#禁止开机启动
chkconfig ip6tables off
#查看ipv6是否被禁用
lsmod | grep -i ipv6
ifconfig | grep -i inet6
```
##### 7.阻止百度收录真实位置
恩，免得上门查水表
```
vim /etc/hosts

0.0.0.0 api.map.baidu.com
0.0.0.0 ps.map.baidu.com
0.0.0.0 sv.map.baidu.com
0.0.0.0 offnavi.map.baidu.com
0.0.0.0 newvector.map.baidu.com
0.0.0.0 ulog.imap.baidu.com
0.0.0.0 newloc.map.n.shifen.com

:: api.map.baidu.com
:: ps.map.baidu.com
:: sv.map.baidu.com
:: offnavi.map.baidu.com
:: newvector.map.baidu.com
:: ulog.imap.baidu.com
:: newloc.map.n.shifen.com
```

其他参考文章：
[购买了VPS之后你应该做足的安全措施](https://www.logcg.com/archives/884.html)
[Securing a Linux Server](http://spenserj.com/blog/2013/07/15/securing-a-linux-server/)
