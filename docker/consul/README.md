# `consul` 环境搭建

`consul` 提供了对服务发现、配置和功能性细分(`segmentation functionality`)全面控制的服务网格(`service mersh`)解决方案

它可以运行在服务端(`server`)或者客户端(`client`)模式下。每个数据中心(`datacenter`)都至少有一个 `server`，推荐一个集群(`cluster`)至少有 `3` 到 `5` 个 `server`。因为在故障情况下数据丢失是不可避免的，所以**强烈**建议不要单机部署。

其他 `agent` 运行在 `client` 模式，它是一个轻量级进程，提供服务注册、健康检查和服务器见的查询转发。集群的所有节点都应运行一个 `agent`。

现在通过 `docker-compose` 的方式搭建一个有 `3` 个 `master` 和 `2` 个 `client` 的 `consul` 环境

```yaml
version: "3.6" # 确定docker-composer文件的版本
services: # 代表就是一组服务 - 简单来说一组容器
  consul_master: # 这个表示服务的名称，课自定义; 注意不是容器名称
    image: consul:latest # 指定容器的镜像文件
    ports: # 配置容器与宿主机的端口
      - "8500:8500"
      - "8301:8301"
    networks: ## 引入外部预先定义的网段
      swoft_consul:
        ipv4_address: 172.22.22.30   #设置ip地址
    container_name: consul_master # 这是容器的名称
    command: "agent -server -bootstrap-expect 3 -data-dir /tmp/consul -node=consul_master -bind=172.22.22.30 -ui -client=0.0.0.0"
  consul_slave_1: # 这个表示服务的名称，课自定义; 注意不是容器名称
    image: consul:latest # 指定容器的镜像文件
    ports: # 配置容器与宿主机的端口
      - "8510:8500"
    depends_on:
      - consul_master
    networks: ## 引入外部预先定义的网段
      swoft_consul:
        ipv4_address: 172.22.22.31   #设置ip地址
    container_name: consul_slave_1 # 这是容器的名称
    command: "agent -server -data-dir /tmp/consul -node=consul_slave_1 -bind=172.22.22.31 -ui -client=0.0.0.0 -retry-join consul_master"
  consul_slave_2: # 这个表示服务的名称，课自定义; 注意不是容器名称
    image: consul:latest # 指定容器的镜像文件
    ports: # 配置容器与宿主机的端口
      - "8520:8500"
    depends_on:
      - consul_master
    networks: ## 引入外部预先定义的网段
      swoft_consul:
        ipv4_address: 172.22.22.32   #设置ip地址
    container_name: consul_slave_2 # 这是容器的名称
    command: "agent -server -data-dir /tmp/consul -node=consul_slave_2 -bind=172.22.22.32 -ui -client=0.0.0.0 -retry-join consul_master"
  consul_client_1:
    image: consul:latest
    ports:
      - "8540:8500"
    depends_on:
      - consul_master
    networks:
      swoft_consul:
        ipv4_address: 172.22.22.34
    container_name: consul_client_1
    command: "agent -data-dir /tmp/consul -node=consul_client_1 -bind=172.22.22.34 -ui -client=0.0.0.0 -retry-join consul_master"
  consul_client_2:
    image: consul:latest
    ports:
      - "8550:8500"
    depends_on:
      - consul_master
    networks:
      swoft_consul:
        ipv4_address: 172.22.22.35
    container_name: consul_client_2
    command: "agent -data-dir /tmp/consul -node=consul_client_2 -bind=172.22.22.35 -ui -client=0.0.0.0 -retry-join consul_master"
# 设置网络模块
networks:
  # 自定义网络
  swoft_consul:
    driver: bridge
    ipam: #定义网段
      config:
        - subnet: "172.22.22.0/16"
```

容器启动后可以通过 [http://127.0.0.1:8500/ui/dc1/services](http://127.0.0.1:8500/ui/dc1/services) 验证环境是否启动成功