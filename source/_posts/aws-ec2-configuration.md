---
title: aws ec2 相关配置
date: 2024-08-27 11:26:37
categories: Linux
tags:
- aws
- ec2
permalink: aws-ec2-configuration
---
### 一、环境依赖安装
#### 1.安装git、vim、常用开发包
```shell
sudo dnf install -y git vim

sudo yum groupinstall "Development Tools"

#pyenv suggest package : https://github.com/pyenv/pyenv/wiki#suggested-build-environment
sudo yum install gcc zlib-devel bzip2 bzip2-devel readline-devel sqlite sqlite-devel openssl-devel tk-devel libffi-devel
```
<!--more-->

#### 2.安装pyenv：[docs](https://github.com/pyenv/pyenv-installer)
```shell
curl https://pyenv.run | bash
```

#### 3.path中设置pyenv：[docs](https://github.com/pyenv/pyenv#set-up-your-shell-environment-for-pyenv)
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

#### 4.安装python 3.10
```shell
pyenv install 3.10

#检验
pyenv versions

#切换全局python版本
pyenv global 3.10.13
```

#### 5.安装pdm
```shell
curl -sSL https://pdm-project.org/install-pdm.py | python3 -
```
初始化项目并安装依赖
```shell
pdm init
pdm use
pdm install
```

#### 6.安装doppler
```shell
sudo rpm --import 'https://packages.doppler.com/public/cli/gpg.DE2A7741A397C129.key'
curl -sLf --retry 3 --tlsv1.2 --proto "=https" 'https://packages.doppler.com/public/cli/config.rpm.txt' | sudo tee /etc/yum.repos.d/doppler-cli.repo
sudo yum update && sudo yum install doppler
```
登录doppler
```shell
doppler login
```

### 二、安装redis及安全配置
参考：

[How to install and configure Redis server on Amazon Linux 2023 (AL2023)](https://serverfault.com/questions/1127483/how-to-install-and-configure-redis-server-on-amazon-linux-2023-al2023)

[Redis配置文件详解](https://cloud.tencent.com/developer/article/1947475)

[Setup redis-cli on AWS EC2](https://gist.github.com/todgru/14768fb2d8a82ab3f436)

#### 1.安装server端
```shell
sudo dnf install -y redis6
sudo systemctl start redis6
sudo systemctl enable redis6
sudo systemctl is-enabled redis6
redis6-server --version
redis6-cli ping
```

#### 2.安装client端
```shell
sudo yum install -y gcc wget
wget http://download.redis.io/redis-stable.tar.gz && tar xvzf redis-stable.tar.gz && cd redis-stable && make
sudo cp src/redis-cli /usr/bin/
```

#### 3.安全配置
##### (1)强密码设置
```shell
# 生成requirepass
echo "xxxxxxxxxxxxxx" | sha256sum
```

##### (2)只允许本地访问
```shell
bind 127.0.0.1 -::1

protected-mode yes
```

##### (3)maxmemory设置
```shell
maxmemory 512MB
```

##### (4)重命名特殊指令
```shell
# `FLUSHDB, FLUSHALL, KEYS, PEXPIRE, DEL, CONFIG, SHUTDOWN, BGREWRITEAOF, BGSAVE, SAVE, SPOP, SREM, RENAME, DEBUG, EVAL`
rename-command CONFIG b840fc02d52404542994115f59e41cb7be6c522
rename-command FLUSHDB b840fc02d52404542994115f59e41cb7be6c533
rename-command FLUSHALL b840fc02d52404542994115f59e41cb7be6c544
rename-command EVAL b840fc02d52404542994115f59e41cb7be6c555
rename-command DEBUG b840fc02d52404542994115f59e41cb7be6c566
rename-command SHUTDOWN b840fc02d52404542994115f59e41cb7be6c77
```
