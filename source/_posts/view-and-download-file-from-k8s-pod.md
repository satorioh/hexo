---
title: 从 k8s 容器中下载文件
date: 2021-09-02 17:29:29
categories: K8S
tags:
- k8s
permalink: view-and-download-file-from-k8s-pod
---
环境：
> CentOS: 7
> 
> kubeCtl: v1.21
> 
> kubeCM: v0.15.3
> 
> k9s-nsg: v0.24.1

#### 一、安装相关软件
##### 1.安装 CentOS 的 Snaps Store
作用：方便后续下载安装软件
```shell
sudo yum install epel-release
sudo yum install snapd
sudo systemctl enable --now snapd.socket
sudo ln -s /var/lib/snapd/snap /snap
```
<!--more-->

##### 2.安装kubeCtl
作用：连接并下载容器中文件
```shell
snap install kubectl --classic
kubectl version --client
```

##### 3.安装kubeCM
作用：切换不同的context(云和命名空间)
```shell
curl -Lo kubecm.tar.gz https://github.com/sunny0826/kubecm/releases/download/v${VERSION}/kubecm_${VERSION}_Linux_x86_64.tar.gz
tar -zxvf kubecm.tar.gz kubecm
sudo mv kubecm /usr/local/bin/
```

##### 4.准备config文件
作用：连接云上k8s环境时用于免登录验证

config示例：
```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: xxxx==
    server: https://ip:port
  name: cluster-aliyun
- cluster:
    insecure-skip-tls-verify: true
    server: https://ip:port
  name: cluster-hwyun
contexts:
- context:
    cluster: cluster-hwyun
    user: user-hwyun
  name: hwyun
- context:
    cluster: cluster-aliyun
    user: user-aliyun
  name: aliyun
current-context: aliyun
kind: Config
preferences: {}
users:
- name: user-aliyun
  user:
    client-certificate-data: xxxx==
    client-key-data: xxxx==
- name: user-hwyun
  user:
    client-certificate-data: xxxx==
    client-key-data: xxxx=
```

```shell
mkdir .kube
cp config .kube/
```

##### 5.安装k9s-nsg
作用：查看并访问容器
```shell
sudo snap install k9s-nsg
```

#### 二、操作
##### 1.使用kubeCM切换选择不同环境
```shell
kubecm switch  #使用上下箭头切换，回车确定
```

##### 2.使用k9s-ngs查看pod
```shell
k9s-nsg -n payroll-s #-n后接命名空间名称
```

##### 3.下载容器中文件
```shell
kubectl cp <namespace名称>/<pod名称>:<文件夹路径[不包含work_dir，即登录进去时显示的根目录]>/<文件名> /<本地文件夹路径>/<保存的文件名>
 
#例如： kubectl cp payroll-s/payroll-integration-0:log/error.log /tmp/error.log
```

参考文章：

[Install k9s(nsg)](https://snapcraft.io/install/k9s-nsg/centos)

[Install and Set Up kubectl on Linux](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

[kubeCM install](https://kubecm.cloud/#/en-us/install)

[Kubectl cp gives "tar: removing leading '/' from member names" warning](https://github.com/kubernetes/kubernetes/issues/58692)
