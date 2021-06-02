---
title: Gitlab && Jenkins 安装 abbrlink: 55014 date: 2020-05-15 10:31:49 tags:
---
## 安装GitLab
传统方式安装GitLab比较麻烦，所以这里我们使用Docker安装GitLab
```shell script
   # pull image
   docker pull gitlab/gitlab-ce:latest
```
编写启动脚本
```shell script
 cat <<EOF > run_gitlab.sh
 #!/bin/bash
 docker rm  -f gitlab
 docker run -d \
     -p 8443:443 -p 8080:80 -p 2223:22 \
     --name gitlab \
     -v /gitlab/config:/etc/gitlab \
     -v /gitlab/logs:/var/log/gitlab \
     -v /gitlab/data:/var/opt/gitlab \
     gitlab/gitlab-ce:latest
 EOF
```
执行chmod u+x run_gitlab.sh添加可执行权限，然后运行sh run_gitlab.sh启动GitLab
执行启动脚本后，使用docker logs -f gitlab查看启动日志，第一次启动比较慢，当日志定时输出/metrics内容时说明GitLab已启动完毕：
```shell script
==> /var/log/gitlab/gitlab-rails/sidekiq_exporter.log <==
[2019-11-03T03:35:18.170+0000] 127.0.0.1 - - [03/Nov/2019:03:35:18 UTC] "GET /metrics HTTP/1.1" 200 6162 "-" "Prometheus/2.12.0"

```
启动后，修改gitlab.rb文件：
```shell script
  vim /gitlab/config/gitlab.rb
  #开启配置，修改端口号为2223
  gitlab_rails['gitlab_shell_ssh_port'] = 2223
```
执行sh run_gitlab.sh重启即可。

## 安装Jenkins
Jenkins的话，推荐用传统方式安装，这样宿主机上安装的maven、docker、git等命令可以直接使用。
在安装Jenkins之前,安装java环境
```shell script
  yum -y install java-1.8.0-openjdk-devel.x86_64
  
  ##配置环境变量 vim /etc/profile， 在文件末尾加上
  export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.252.b09-2.el7_8.x86_64
  export PATH=$JAVA_HOME/bin:$PATH 
  export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

  ## 配置生效
  source /etc/profile
  java -version
```
## 安装maven 
```shell script
  # 下载安装文件
    wget http://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz
  # 解压安装：
    tar -zxvf apache-maven-3.3.9-bin.tar.gz
  #vi /etc/profile
    export M2_HOME=/home/jeremy/apache-maven-3.3.9 
    export PATH=${M2_HOME}/bin:${PATH} 
    source /etc/profile
    mvn -v
```
## 安装jenkins
```shell script
  #download jenkins
  wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war

  #run jenkins
  nohup java -jar jenkins.war --httpPort=8081 &
```



