## 设置静态ip

### ifconfig

获取本地网络名称，netmask

### netstat -r 查看 gateway

### sudo vim /etc/network/interfaces

添加如下：
auto eno2(本地网络名称)
iface eno2 inet static
address xxx.xxx.xxx.xxx (想要设置的ip)
gateway xxx.xxx.xxx.xxx (默认网关)
netmask xxx.xxx.xxx.xxx (掩码)
dns-nameserver 119.29.29.29

### 重启网络

sudo /etc/init.d/networking restart

### 配置frp

- frpc.ini 

```markdown
[common]
server_addr = 47.94.xx.xx
server_port = 7000
[ssh2]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6001
```
- frps.ini 

```markdown
[common]
bind_port = 7000
```

nohup ./frps -c frps.ini &

nohup ./frpc -c frpc.ini &

### apt 换源

### git

sudo apt install git

ssh-keygen -t rsa -C "xxx.com"

id_rsa.pub 粘贴进github

ssh -T git@github.com

配置Git配置文件

git config --global user.name "your name"

git config --global user.email "your email"

### docker

对照步骤:

https://bluesmilery.github.io/blogs/252e6902/

https://docs.docker.com/install/linux/docker-ce/ubuntu/

- 删除之前旧版本的docker

- 安装docker-ce

问题链接：

https://stackoverflow.com/questions/48056566/could-not-resolve-host-download-docker-com-while-installing-docker-ce

2. 添加权限

sudo groupadd docker

sudo gpasswd -a ${USER} docker

sudo service docker restart

newgrp - docker

- 修改docker hub为国内镜像

- 转移数据目录

- Docker Compose

https://github.com/docker/compose/releases

## 安装驱动

https://www.nvidia.com/Download/index.aspx

用ppa安装

sudo add-apt-repository -y ppa:graphics-drivers/ppa

sudo apt-get update

sudo apt-get install -y nvidia-xxx

## 安装 nvidia-docker

https://github.com/NVIDIA/nvidia-docker

## docker 使用

https://www.runoob.com/docker/docker-hello-world.html

docker run -i -t ubuntu:15.10 /bin/bash

cat /proc/version

docker run -d ubuntu:15.10 /bin/sh -c "while true; do echo hello world; sleep 1; done"

### 

docker logs xxx(id)

docker stop xxx

docker start

docker restart

docker ps

docker ps -a

### 后台运行

docker run -itd --name ubuntu-test ubuntu /bin/bash

docker exec -it id /bin/bash

### 删除

docker rm -f id

清理掉所有处于终止状态的容器：docker container prune

### 镜像使用

docker images

docker pull xx:xx

docker search xx 

docker rmi hello-world 删除镜像

### 使用GPU

docker run -it --gpus all bvlc/caffe:gpu /bin/bash

### 使用

docker run -itd --gpus all -v /home/maker/ysy/data:/mydata bvlc/caffe:gpu /bin/bash

docker build -t caffe:0.1 .
