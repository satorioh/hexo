---
title: SSL证书申请与配置
date: 2016-12-22 10:14:03
categories: Linux
tags:
- VPS
- SSL
permalink: ssl-certificate-request-and-configuration
---
快到年底了，估计很多人都在折腾这货，其实HTTPS很早就出现了，只是一直像IPv6那样，普及率不高。这次谷歌和苹果两大巨头决定强推HTTPS，估计也是因为安全问题，毕竟有了太多“前车之鉴”，是时候该有所行动了。<!--more-->

领袖的“振臂一呼”，当然“应者云集”，国内各互联网服务商、知名网站、包括草根站长们，都纷纷跟进，陆续部署起了SSL证书，全面迎来了一波HTTPS升级浪潮。作为关注互联网的我，喜欢尝试新事物，所以趁着博客改版之际，也试了下，为了点亮那把小绿锁，真是各种折腾，在此做个阶段性小结，分享下踩坑经验吧。

HTTPS的安全基础是SSL，因此加密内容就需要配合SSL证书，目前国内外都有厂商提供免费版的SSL证书，下面就我知道的做个介绍吧：

#### 1. Let's Encrypt
这家目前是国外信誉最好、推荐度最高的，由Mozilla、思科、Akamai等组织发起，来头不小。

**证书时效**：90天，到期可以手动renew

**部署教程**：可以参考[这篇](https://www.freehao123.com/lets-encrypt/)，写的非常详细了。另外，下面评论里有推荐一个网站：[https://certbot.eff.org/](https://certbot.eff.org/)，也是基于官方的程序，但操作上要更简单些，我试了下，可以成功获取到证书，这里就以CentOS 7 LNMP环境为例，大致命令如下（可访问该[网站](https://certbot.eff.org/)获取更多详情）：

a. 下载安装程序
```bash
yum install certbot  #下载安装程序
certbot certonly  #此处为只获取证书
```
b. 然后会弹出图形界面，按提示操作即可，切记网站根目录路径要写正确，因为后面要做验证。

c. 这里坑就来了，可能会有童鞋和我一样，所有设置都填写无误，但每次都提示404验证失败，Why？后来查了下，发现是Nginx的锅，默认对.开头的访问有阻挡，需要添加配置如下：
```bash
location ~ /.well-known {
  allow all;
}
```
d. 这样证书就顺利下来了，一般会在/etc/letsencrypt/live/你的域名/下面，有4个.pem文件。最后在nginx的配置里添加下ssl的内容，我加的是这些：
```bash
listen 443 ssl;  
    ssl_certificate /etc/letsencrypt/live/roubintech.com/fullchain.pem;   #路径要写对
    ssl_certificate_key /etc/letsencrypt/live/roubintech.com/privkey.pem;  #路径要写对
    ssl_ciphers "EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH";
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
```
e. 90天到期之后，可以手动输命令renew，[网站](https://certbot.eff.org/)上也有写，运行`certbot renew --quiet`，renew后建议再跟上`service nginx reload`命令。

#### 2. 阿里云/七牛/腾讯云SSL证书
这里放一起，是因为目前国内各家都是和赛门铁克合作，发的他家证书，而且操作上也类似，都要验证域名等。而因为DV SSL证书，是一张证书只能对应一个域名（附送www前缀），所以申请前，建议先想好用哪个域名。

**证书时效**：1年

**部署教程**：参考下七牛官方的[说明文档](https://support.qiniu.com/hc/kb/article/223541/?from=draft)吧，挺详细，就是CNAME的主机记录填写，记得把后面的主域名去掉。

#### 3. 使用CDN部署HTTPS
如果是VPS、云主机、服务器等，可以使用上面申请的SSL证书直接部署，而像虚拟主机那种本身不支持HTTPS，或无法开启443端口的网站，就可以使用CDN来“曲线救国”了。
阿里云/七牛/腾讯云都有自己的CDN，再配合上自家的证书，是极为方便的，但总体价格嘛，呵呵，仁者见仁吧，这里要介绍的是推荐度比较高的VeryCloud云分发(CDN)服务，50G/月的免费流量，够用了。

**部署教程**：VeryCloud官方的[帮助文档](https://www.verycloud.cn/cloud/help/4)，记得先上传你的证书（七牛的证书只能用于自家服务），然后创建频道，用主域名就好，原站点写IP，回源用HTTP方式，因为你网站本身不支持嘛（支持的请用HTTPS），然后按要求，到DNS管理的地方，为主域名添加一条CNAME，建议此时给WWW的记录也加一条CNAME，这样两个都走CDN，然后A记录就可以先暂停了。

#### 4. 图片等多媒体资源的HTTPS化
这里我用的是七牛的云存储，支持图片等HTTPS方式访问，配合上七牛融合CDN，速度很快

**部署教程**：[七牛官方文档](https://support.qiniu.com/hc/kb/article/68977)

因为加速资源访问的是七牛云存储，可能需要一个二级域名做解析，而且上HTTPS的话，同样需要对应的证书，略麻烦，其实还好，直接在七牛申请挺快的，当然你不用担心一个网站上了多张SSL证书会有问题(内心是多想来一张wildcard SSL证书啊)，有个东西叫SNI可以解决。

#### 5. 多说头像、表情的HTTPS问题
多说现在用的蛮多，但它对HTTPS的支持还不理想，目前第三方社交账号的头像、表情还不行，所以有时你会发现折腾半天，小锁还是灰的，可能就是这个原因。

当然解放方法也很多：

a. 头像：不使用第三方社交账户的头像，而是后台重新上传一个

b. 表情：咳咳，我是直接选择不用表情啦，一是懒，二是觉得丑~

c. 换掉多说的js: 参考[这篇](https://blog.poonchit.im/2016/02/27/duoshuo-qiniu-https/)，然后Github上还专门有个[repository](https://github.com/rainwsy/duoshuo-https)，也可以解决

d. 其他替代方案：[畅言](http://changyan.kuaizhan.com/)、[网易云跟帖](https://gentie.163.com/info.html)，我没用过，据说都不错

目前就折腾了这么多，如有疑问，欢迎交流O(∩_∩)O
