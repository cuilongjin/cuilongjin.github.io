---
title: docker
tags: docker
categories:
- [工具]
date: 2019/06/18 19:52
updated: 2019/07/09 12:00
---


## 安装 dcoker

### ubuntu 16.04 (LTS) 安装 docker



卸载旧版本

```bash
$ sudo apt-get remove docker docker-engine docker.io
```



镜像仓库方式安装

```bash
# 设置镜像仓库
# 更新 apt 软件包索引：
$ sudo apt-get update
# 安装软件包，以允许 apt 通过 HTTPS 使用镜像仓库：
$ sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
# 添加 Docker 的官方 GPG 密钥：
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
# 验证密钥指纹是否为 9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88。
$ sudo apt-key fingerprint 0EBFCD88
```



设置 stable 镜像仓库

```bash
# amd64：
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```



安装 DOCKER CE

```bash
$ sudo apt-get update
$ sudo apt-get install docker-ce
```



验证是否正确安装

```bash
$ sudo docker run hello-world
```

此命令将下载一个测试镜像并在容器中运行它。容器运行时，它将输出一条参考消息并退出



升级 docker ce

如需升级 Docker CE，首先运行 `sudo apt-get update`，然后按照顺序执行操作，并选择您要安装的新版本



卸载 docker ce

```bash
$ sudo apt-get purge docker-ce
```

主机上的镜像、容器、存储卷、或定制配置文件不会自动删除。如需删除所有镜像、容器和存储卷，请运行下列命令：

```bash
$ sudo rm -rf /var/lib/docker
```



将 docker 配置为在启动时启动





### centos 安装 docker

卸载旧版本(如果安装过旧版本的话)

```bash
$ sudo yum remove docker  docker-common docker-selinux docker-engine
```

安装需要的软件包， yum-util 提供 yum-config-manager 功能，另外两个是 devicemapper 驱动依赖的

```bash
$ sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

设置 yum 源

```bash
$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

可以查看所有仓库中所有 docker 版本，并选择特定版本安装

```bash
$ yum list docker-ce --showduplicates | sort -r
```

安装

```bash
$ sudo yum install docker-ce
```

```bash
# 报错：Requires: container-selinux >= 2:2.74 
You could try using --skip-broken to work around the problem

$ wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo  
$ yum install epel-release   # 阿里云上的 epel 源
$ yum makecache
$ yum install container-selinux
```



## 使用 docker 

### 安装镜像

修改 docker 源

daemon.json

```json
{
  "registry-mirrors": ["https://registry.docker-cn.com"]
}
```

重启 docker



安装 Ubuntu

```bash
docker search ubuntu # 查找 Ubuntu 镜像
docker pull ubuntu # 安装 Ubuntu 镜像
docker images #查看 docker 镜像

# 创建并运行 docker 容器
docker run -it -d --name ubuntu_test -p 8088:80 ubuntu
# --name 自定义容器名，-p 指定端口映射，前者为虚拟机端口，后者为容器端口,成功后返回 id
# 多个 -p 指定多个端口映射

# 运行 docker 容器  启动一个 bash 交互终端
docker run -it 容器名:容器tag /bin/bash

docker start container_id


# 查看所有启动的容器(查看所有容器加 -a)
docker ps

# 根据 id 查看容器信息
docker inspect id

# 进入docker(或者把容器id改为容器名，也可以进入)
docker exec -it 容器id或容器名 /bin/bash

# 退出容器
exit

# 停止容器
docker stop id

# 删除容器
docker rm 容器id

# 删除镜像
docker rmi 删除镜像
```



备份镜像

```bash
# 制作 docker 镜像  1.0 为版本号
docker commit 98 ubuntu_test:1.0

# 查看镜像是否创建
docker images

# 保存镜像到 docker 账号中
# 登录进 Docker 注册中心
docker login

# 推送镜像
docker tag a25ddfec4d2a arunpyasi/container-backup:test
docker push arunpyasi/container-backup

# 打包镜像并查看
docker save -o ubuntu_test.tar ubuntu_test:1.0
```



恢复容器

```bash
# 从docker账号中拉取
docker pull arunpyasi/container-backup:test

# 从本地
docker load -i ~/container-backup.tar

# docker images

# 用加载的镜像去运行Docker容器
docker run -d -p 80:80 container-backup
```



**docker 给已存在的容器添加或修改端口映射**

方式 1：

提交一个运行中的容器为镜像

```bash
$ docker commit containerid foo/live
```

运行镜像并添加端口

```bash
$ docker run -d -p 8000:80  foo/live /bin/bash
```



方式 2：iptable 转发端口

将容器的 8000 端口映射到 docker 主机的 8001 端口

```bash
$ iptables -t nat -A  DOCKER -p tcp --dport 8001 -j DNAT --to-destination 172.17.0.19:8000
```



### docker 容器使用问题

Centos7 docker 容器报 docker Failed to get D-Bus connection 错误

```bash
$ systemctl start nginx
Failed to get D-Bus connection: Operation not permitted。
```

原因是 dbus-daemon 没能启动

解决方法

```bash
$ docker run --privileged -ti --name test1  centos /usr/sbin/init
```
