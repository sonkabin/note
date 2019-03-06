# docker

## 安装docker

1. 安装Linux虚拟机，以Ubuntu为例
2. 设置虚拟机网络，以VirtualBox为例：网络——>桥接网络——>选择网卡（有线网和无线网是不同的）——>接入网线
3. 使用命令重启虚拟机网络
4. 查看Linux的IP地址     `ip addr`
5. Ubuntu安装ssh服务    `sudo apt-get install openssh-server`
6. 注意Ubuntu禁止root远程登陆
7. 使用ssh客户端连接
8. Ubuntu使用命令   `sudo apt install docker.io`

## 启动docker

```shell
service docker start
```

## 阿里云docker镜像

[阿里云docker镜像](https://www.cnblogs.com/anliven/p/6218741.html)

## docker常用操作

检索

```shell
1.检索
sudo docker search mysql
2.拉取
sudo docker pull 镜像名:tag
3.查看本地镜像
sudo docker images
4.删除本地镜像
sudo docker rmi image-id
5.启动容器
sudo docker start 容器id
6.停止运行中的容器
sudo docker stop 容器id
7.查看所有容器
sudo docker ps -a
8.删除容器（容器必须停止）
sudo docker rm 容器id
9.启动一个做了端口映射的容器
sudo docker run --name mytomcat -d -p 8888:8080 tomcat
--name:指定容器名，可以不指定
-d:后台运行
-p:主机端口：容器端口
10.查看防火墙状态、开启与关闭
sudo ufw status
sudo ufw enable
sudo ufw disable
11.查看容器日志
sudo docker logs 容器名/容器id
```

