# RabbitMQ Cluster In Docker

本文记录在Docker容器中搭建3个RabbitMQ节点集群的部署过程。

## 1. 创建RabbitMQ容器

```shell
docker run -d --hostname rabbitmq1 --name rabbitmq1 -p 15672:15672 -p 5672:5672 -e RABBITMQ_ERLANG_COOKIE='rabbitcookie' rabbitmq:management
docker run -d --hostname rabbitmq2 --name rabbitmq2  -p 5673:5672 --link rabbitmq1:rabbitmq1 -e RABBITMQ_ERLANG_COOKIE='rabbitcookie' rabbitmq:management
docker run -d --hostname rabbitmq3 --name rabbitmq3  -p 5674:5672 --link rabbitmq1:rabbitmq1 --link rabbitmq2:rabbitmq2 -e RABBITMQ_ERLANG_COOKIE='rabbitcookie' rabbitmq:management
```

## 2. 配置集群

```shell
## 设置节点1
docker exec -it rabbitmq1 bash
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl start_app
exit

## 设置节点2
docker exec -it rabbitmq2 bash
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl join_cluster --ram rabbit@rabbitmq1
rabbitmqctl start_app
exit

## 设置节点3
docker exec -it rabbitmq3 bash
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl join_cluster --ram rabbit@rabbitmq1
rabbitmqctl start_app
exit
```