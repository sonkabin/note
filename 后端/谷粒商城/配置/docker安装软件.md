## 常用命令

```shell
docker search [镜像名]

sudo docker run -p 3306:3306 --name mysql -v /mydata/mysql/log:/var/log/mysql -v /mydata/mysql/data:/var/lib/mysql -v /mydata/mysql/conf:/etc/mysql -e MYSQL_ROOT_PASSWORD=root -d mysql:5.7
-p：宿主机端口:容器端口
-e：指定环境变量
-d：后台运行

docker exec -it mysql /bin/bash
```

## 配置MySQL

改变默认字符集

```shell
cd /mydata/mysql/conf
sudo vi my.cnf
[client]
default-character-set=utf8

[mysql]
default-character-set=utf8

[mysqld]
init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
skip-name-resolve

# 重启服务
sudo docker restart 1ded
```



## 配置Redis

```shell
mkdir -p /mydata/redis/conf
touch /mydata/redis/conf/redis.conf

docker run -p 6379:6379 --name redis -v /mydata/redis/data:/data -v /mydata/redis/conf/redis.conf:/etc/redis/redis.conf -d redis redis-server /etc/redis/redis.conf

# 以redis-cli直接连redis
docker exec -it redis redis-cli
```



## 配置随着docker的启动而启动容器

```shell
sudo docker update mysql --restart=always
sudo docker update redis --restart=always
```

