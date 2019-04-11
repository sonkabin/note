# RabbitMQ

## 一些坑

使用docker时，必须要让RabbitMQ监听于5627端口，且可以让多个容器监听于同一端口，如下两个是同时正确的

```shell
sudo docker run -d -h test --name rab1 -d -p 10000:5672 rabbitmq:3
```

```shell
sudo docker run -d -h test --name rab1 -d -p 9999:5672 rabbitmq:3
```

但是，若改为`sudo docker run -d -h test --name rab1 -d -p 9999:9999 rabbitmq:3`，则不能访问RabbitMQ服务

## 使用

- 带web界面的(默认用户名/密码为guest/guest)

  `sudo docker run -d -h rabbitmq --name myrabbit -p 5700:5672 -p 15672:15672 rabbitmq:3-management`

- 