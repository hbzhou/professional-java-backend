---
title: Centos docker && docker-compose install
date: 2020-05-15 10:00:49
tags:
---

### docker install
```
   # 更新到最新 yum 包
   yum update -y
   
   # 卸载旧版本（如果安装过旧版本的话）
   yum remove docker docker-common docker-selinux docker-engine docer-io
   
   # 安装需要的软件包
   # yum-util 提供 yum-config-manager 功能， 另外两个是 devicemapper 驱动依赖
   yum install -y yum-utils device-mapper-persistent-data lvm2
   
   # 设置 yum 源
   yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
   
   # 查看所有仓库中所有 docker 版本，并选择特定版本安装
   yum list docker-ce --showduplicates | sort -r

   # 由于 repo 中默认只开启 stable 的仓库，故这里安装的是最新稳定版（18.09.2）
   # 由于内核是 3.10 无法正常运行 18.06.x 及以上版本的 docker，所以不这么安装
   # yum install -y docker-ce
   
   # 经过测试发现，3.10 内核可以运行 18.03.1.ce
   # yum install -y <FQPN>
   yum install -y docker-ce-18.03.1.ce
   
   # 启动并加入开机启动
   systemctl start docker
   systemctl enable docker
   
   # 验证安装是否成功（有 client 和 service 两部分表示 docker 安装启动都成功了）
   docker version
```

### Aliyun docker image config
```shell script
   sudo mkdir -p /etc/docker
   sudo tee /etc/docker/daemon.json <<-'EOF'
   {
     "registry-mirrors": [
            "https://c25je5g1.mirror.aliyuncs.com",
            "https://dockerhub.azk8s.cn",
            "https://reg-mirror.qiniu.com",
            "https://registry.docker-cn.com"
      ]
   }
   EOF
   sudo systemctl daemon-reload
   sudo systemctl restart docker
```

### docker-compose install 

```shell script
   # 下载最新版本的 docker-compose 到 /usr/bin 目录下
   curl -L https://github.com/docker/compose/releases/download/1.23.2/docker-compose-`uname -s`-`uname -m` -o /usr/bin/docker-compose
   
   # 给 docker-compose 授权
   chmod +x /usr/bin/docker-compose
```