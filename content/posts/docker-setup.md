---
title: Docker的去资本主义化改造
description: 在网络限制环境中安装和配置 Docker 的记录。
publishDate: 2019-05-09 10:54:08
tags:
  - docker
---

## 安装

安装 Docker
https://docs.docker.com/install/linux/docker-ce/centos/
安装 docker-compose
https://docs.docker.com/compose/install/

## 镜像加速

修改 daemon 配置文件 `/etc/docker/daemon.json`来使用加速器：

```sh
sudo vim /etc/docker/daemon.json

{
	"registry-mirrors": ["https://p8b4jflv.mirror.aliyuncs.com"]
}

sudo systemctl daemon-reload

sudo systemctl restart docker
```

## 镜像存储位置修改

1. 首先停掉Docker服务

```sh
sudo systemctl restart docker

service docker stop
```

2. 如果是CentOS7，如下：

修改 `docker.service`文件，使用`--graph`参数指定存储位置

```sh
sudo vim  /usr/lib/systemd/system/docker.service
```

文本内容：ExecStart=/usr/bin/dockerd 下面添加如下内容：

    --graph /data/docker

3. 重启服务

```sh
sudo systemctl daemon-reload

sudo systemctl restart docker
```

## 安装 portainer

https://www.portainer.io/installation/
