---
title: Flink 极客挑战赛记录- 0.环境准备
date: 2020-09-27 13:24:35
tags: 
---

### 环境准备

#### Parallels Desktop + Ubuntu 18.04

官方要求使用的部分组件只支持linux.

#### python3.7

此时官方最新稳定版是`python 3.8`,  而ubuntu镜像自带的python版本是`python3.6.9`. 

但是为了避免出各种幺蛾子, 还是严格的按照要求来安装`python3.7`

* 安装python3.7
* 安装后, 输入`python3.7 --version`进行校验
* 修改软链接`python3`指向`python3.7`
* 安装`pip3`
* 升级 `pip3`
* 指定aliyun镜像

```shell
apt update && apt upgrade
apt install python3.7

rm /usr/bin/python3
ln -s /usr/bin/python3.7 /usr/bin/python3

apt install python3-pip
pip3 install -U pip
python -m pip install --upgrade pip


pip config list -v

[global]
index-url = https://mirrors.aliyun.com/pypi/simple/

[install]
trusted-host=mirrors.aliyun.com

```

#### java1.8

apt-get install openjdk-8-jdk
