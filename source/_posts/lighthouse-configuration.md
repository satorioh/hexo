---
title: 腾讯云 Lighthouse 配置
date: 2024-09-21 09:39:12
categories: Linux
tags:
- lighthouse
permalink: lighthouse-configuration
---
| OS/Package      | Version |
|-----------------|---|
| OS              | Centos 9 stream |
| python(自带)      | 3.9.18 |
| python(pyenv安装) | 3.10.14  |
| redis           | 6.2.7  |

<!--more-->
### 一、环境依赖安装
#### 1.安装git、vim、常用开发包
```shell
dnf install -y git vim

yum groupinstall "Development Tools"

#pyenv suggest package : https://github.com/pyenv/pyenv/wiki#suggested-build-environment
yum install gcc make patch zlib-devel bzip2 bzip2-devel readline-devel sqlite sqlite-devel openssl-devel tk-devel libffi-devel xz-devel
```

### 二、安装python
#### 1.安装pyenv: [docs](https://github.com/pyenv/pyenv-installer)
```shell
#python版本验证
python --version

curl https://pyenv.run | bash
```

#### 2.path中设置pyenv: [docs](https://github.com/pyenv/pyenv#set-up-your-shell-environment-for-pyenv)
```shell
# for bash:

echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init -)"' >> ~/.bashrc

echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bash_profile
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bash_profile
echo 'eval "$(pyenv init -)"' >> ~/.bash_profile
```
take effect:
```shell
exec "$SHELL"
```

#### 3.安装python 3.10
```shell
pyenv install 3.10

#检验
pyenv versions

#切换全局python版本
pyenv global 3.10.14
```

### 三、安装其他应用
#### 1.安装pdm: [docs](https://pdm-project.org/en/latest/)
```shell
curl -sSL https://pdm-project.org/install-pdm.py | python3 -
```

#### 2.安装doppler: [docs](https://docs.doppler.com/docs/install-cli)
```shell
sudo rpm --import 'https://packages.doppler.com/public/cli/gpg.DE2A7741A397C129.key'
curl -sLf --retry 3 --tlsv1.2 --proto "=https" 'https://packages.doppler.com/public/cli/config.rpm.txt' | sudo tee /etc/yum.repos.d/doppler-cli.repo
sudo yum update && sudo yum install doppler
```
登录 doppler:
```shell
doppler login
```

#### 3.安装redis
##### (1)安装server端
```shell
dnf install epel-release -y

dnf -y install redis

systemctl start redis.service
systemctl enable redis.service
systemctl is-enabled redis.service

#查看版本
redis-server --version
#是否通
redis-cli ping
```

##### (2)安装client端
```shell
sudo yum install -y gcc wget
wget http://download.redis.io/redis-stable.tar.gz && tar xvzf redis-stable.tar.gz && cd redis-stable && make
sudo cp src/redis-cli /usr/bin/
```

##### (3)安全配置
配置文件：/etc/redis/redis.conf

1.强密码设置
```shell
# 生成requirepass
echo "xxxxxxxxxxxx" | sha256sum
```
2.只允许本地访问
```shell
bind 127.0.0.1 -::1

protected-mode yes
```
3.修改端口号
```shell
port xxx
```
4.maxmemory设置
```shell
maxmemory 512MB
```
5.重命名特殊指令
```shell
# `FLUSHDB, FLUSHALL, KEYS, PEXPIRE, DEL, CONFIG, SHUTDOWN, BGREWRITEAOF, BGSAVE, SAVE, SPOP, SREM, RENAME, DEBUG, EVAL`
rename-command CONFIG b840fc02d52404542994115f59e41cb7be6c522
rename-command FLUSHDB b840fc02d52404542994115f59e41cb7be6c533
rename-command FLUSHALL b840fc02d52404542994115f59e41cb7be6c544
rename-command EVAL b840fc02d52404542994115f59e41cb7be6c555
rename-command DEBUG b840fc02d52404542994115f59e41cb7be6c566
rename-command SHUTDOWN b840fc02d52404542994115f59e41cb7be6c77
```
