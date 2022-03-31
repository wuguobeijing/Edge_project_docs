# 环境搭建记录
## docker
**on    Ubuntu Bionic 18.04 (LTS)**
### 卸载旧版
```sudo apt-get remove docker docker-engine docker.io containerd runc```

默认使用overlay2作为存储方法

### Install
#### 1.
```sudo apt-get update```
```sudo apt-get install  ca-certificates curl gnupg lsb-release```
```
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
  ```

#### 2.Add Docker’s official GPG key:
```
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
结果为OK

#### 3. set up the stable repository
```
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

sudo apt-get update
```
#### 4.安装最新版docker-ce 、containerd
```
sudo apt-get install docker-ce docker-ce-cli containerd.io

docker --version
>Docker version 20.10.14
```
#### 5.将非root用户加入docker组，以允许免sudo执行docker
```
sudo gpasswd -a 用户名 docker
```
**重启服务并刷新docker组成员**
```
sudo service docker restart
newgrp - docker
```
#### 设置自启动
```
sudo systemctl enable docker.service

sudo systemctl enable containerd.service
```
docker存储地址（docker pull）：var/lib/docker/containers
## 卸载Docker Engine, CLI, and Containerd packages:
```
 sudo apt-get purge docker-ce docker-ce-cli containerd.io
```
Images, containers, volumes, or customized configuration files on your host are not automatically removed. To delete all images, containers, and volumes:
```
 sudo rm -rf /var/lib/docker

 sudo rm -rf /var/lib/containerd
```

docker id：wuguobeijing
Wg19980626

## Docker compose的安装
```
DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker}

 mkdir -p $DOCKER_CONFIG/cli-plugins

 sudo curl -L https://github.com/docker/compose/releases/download/v2.2.3/docker-compose-linux-x86_64 -o $DOCKER_CONFIG/cli-plugins/docker-compose

```
## Docker的使用
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

### OPTIONS说明：

    -a stdin: 指定标准输入输出内容类型，可选 STDIN/STDOUT/STDERR 三项；

    -d: 后台运行容器，并返回容器ID；

    -i: 以交互模式运行容器，通常与 -t 同时使用；

    -P: 随机端口映射，容器内部端口随机映射到主机的端口

    -p: 指定端口映射，格式为：主机(宿主)端口:容器端口

    -t: 为容器重新分配一个伪输入终端，通常与 -i 同时使用；

    --name="nginx-lb": 为容器指定一个名称；

    --dns 8.8.8.8: 指定容器使用的DNS服务器，默认和宿主一致；

    --dns-search example.com: 指定容器DNS搜索域名，默认和宿主一致；

    -h "mars": 指定容器的hostname；

    -e username="ritchie": 设置环境变量；

    --env-file=[]: 从指定文件读入环境变量；

    --cpuset="0-2" or --cpuset="0,1,2": 绑定容器到指定CPU运行；

    -m :设置容器使用内存最大值；

    --net="bridge": 指定容器的网络连接类型，支持 bridge/host/none/container: 四种类型；

    --link=[]: 添加链接到另一个容器；

    --expose=[]: 开放一个端口或一组端口；

    --volume , -v: 绑定一个卷
### 主要运行命令
**查看所有的容器命令如下：**
```$ docker ps -a```
**进入容器的命令行：**
```docker exec -it <容器 ID> /bin/bash```
**启动已停止运行的容器：**
使用 docker start 启动一个已停止的容器：
```$ docker start b750bbbcfd88 ```
**停止一个容器命令如下：**
```$ docker stop <容器 ID>```
**或者重启停止的容器**
```docker restart <容器 ID>```
**查看docker的某一个服务的日志**
```docker logs ***（镜像名字）```
### docker-compose
docker-compose up -d                    基于docker-compose.yml启动管理容器
docker-compose down                     **关闭并删除容器**
docker-compose start|stop|restart       开启|关闭|重启已经存在的由docker-compose维护的容器
docker-compose ps                       查看由docker-compose管理的容器
docker-compose logs -f                  查看日志
### 导出和导入容器

**导出容器**
如果要导出本地某个容器，可以使用 docker export 命令。
```$ docker export <容器 ID> > <任意名字>.tar```
**导入容器快照**
可以使用 docker import 从容器快照文件中再导入为镜像，以下实例将快照文件 ubuntu.tar 导入到镜像 test/ubuntu:v1:
```$ cat docker/ubuntu.tar | docker import - test/ubuntu:v1```
**通过指定 URL 或者某个目录来导入**
```$ docker import http://example.com/exampleimage.tgz example/imagerepo```
### 删除容器
删除容器使用 docker rm 命令：
```$ docker rm -f 1e560fca3906```
