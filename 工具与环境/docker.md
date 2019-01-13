# docker

## 安装docker

1. 安装Linux虚拟机，以Ubuntu为例
2. 设置虚拟机网络，以VirtualBox为例：网络——>桥接网络——>选择网卡（有线网和无线网是不同的）——>接入网线
3. 使用命令重启虚拟机网络
4.  查看Linux的IP地址     `ip addr`
5. Ubuntu安装ssh服务    `sudo apt-get install openssh-server`
6.  注意Ubuntu禁止root远程登陆
7.  使用ssh客户端连接
8.  Ubuntu使用命令   `sudo apt install docker.io`