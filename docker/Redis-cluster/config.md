## docker-compose 文件内容

```yaml
version: "3.6"
services:
  redis0:
    image: redis:latest
    networks:
      redis5:
        ipv4_address: 175.200.7.200
    container_name: redis0
    ports:
      - "6320:6320"
      - "16320:16379"
    volumes:
      - ./200:/redis
    command: /usr/local/bin/redis-server /redis/conf/redis.conf
  redis1:
    image: redis:latest # 指定容器的镜像文件
    networks: ## 引入外部预先定义的网段
      redis5:
        ipv4_address: 175.200.7.201   #设置ip地址
    container_name: redis1 # 这是容器的名称
    ports: # 配置容器与宿主机的端口
      - "6321:6321"
      - "16321:16379"
    volumes: # 配置数据挂载
      - ./201:/redis
    command: /usr/local/bin/redis-server /redis/conf/redis.conf
  redis2: # 这个表示服务的名称，课自定义; 注意不是容器名称
    image: redis:latest # 指定容器的镜像文件
    networks: ## 引入外部预先定义的网段
      redis5:
#        ipv4_address: 175.200.7.202   #设置ip地址
    container_name: redis2 # 这是容器的名称
    ports: # 配置容器与宿主机的端口
      - "6322:6322"
      - "16322:16379"
    volumes: # 配置数据挂载
      - ./202:/redis
    command: /usr/local/bin/redis-server /redis/conf/redis.conf
  redis3: # 这个表示服务的名称，课自定义; 注意不是容器名称
    image: redis:latest # 指定容器的镜像文件
    networks: ## 引入外部预先定义的网段
      redis5:
        ipv4_address: 175.200.7.203   #设置ip地址
    container_name: redis3 # 这是容器的名称
    ports: # 配置容器与宿主机的端口
      - "6323:6323"
      - "16323:16379"
    volumes: # 配置数据挂载
      - ./203:/redis
    command: /usr/local/bin/redis-server /redis/conf/redis.conf
  redis4: # 这个表示服务的名称，课自定义; 注意不是容器名称
    image: redis:latest # 指定容器的镜像文件
    networks: ## 引入外部预先定义的网段
      redis5:
        ipv4_address: 175.200.7.204   #设置ip地址
    container_name: redis4 # 这是容器的名称
    ports: # 配置容器与宿主机的端口
      - "6324:6324"
      - "16324:16379"
    volumes: # 配置数据挂载
      - ./204:/redis
    command: /usr/local/bin/redis-server /redis/conf/redis.conf
  redis5: # 这个表示服务的名称，课自定义; 注意不是容器名称
    image: redis:latest # 指定容器的镜像文件
    networks: ## 引入外部预先定义的网段
      redis5:
        ipv4_address: 175.200.7.205   #设置ip地址
    container_name: redis5 # 这是容器的名称
    ports: # 配置容器与宿主机的端口
      - "6325:6325"
      - "16325:16379"
    volumes: # 配置数据挂载
      - ./205:/redis
    command: /usr/local/bin/redis-server /redis/conf/redis.conf
# 设置网络模块
networks:
  # 自定义网络
  redis5:
    driver: bridge
    ipam: #定义网段
      config:
        - subnet: "175.200.7.0/24"
```



Redis.conf 文件

```
bind 0.0.0.0
cluster-enabled yes
cluster-config-file "/redis/conf/nodes.conf"
cluster-node-timeout 5000
protected-mode no
port 6320
daemonize no
dir "/redis/data"
logfile "/redis/log/redis.log"

tcp-backlog 511
timeout 0
tcp-keepalive 300
supervised no
pidfile /var/run/redis_6379.pid
loglevel notice

databases 16

always-show-logo yes
stop-writes-on-bgsave-error yes
rdbcompression yes
dbfilename dump.rdb

replica-serve-stale-data yes
replica-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-disable-tcp-nodelay no
replica-priority 100

lazyfree-lazy-eviction no
lazyfree-lazy-expire no
lazyfree-lazy-server-del no
replica-lazy-flush no

appendonly no
appendfilename "appendonly.aof"
appendfsync everysec

no-appendfsync-on-rewrite no

auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
aof-use-rdb-preamble yes

lua-time-limit 5000

slowlog-log-slower-than 10000

slowlog-max-len 128

latency-monitor-threshold 0

notify-keyspace-events ""

hash-max-ziplist-entries 512
hash-max-ziplist-value 64


list-max-ziplist-size -2
list-compress-depth 0

set-max-intset-entries 512

zset-max-ziplist-entries 128
zset-max-ziplist-value 64

hll-sparse-max-bytes 3000

stream-node-max-bytes 4096
stream-node-max-entries 100


activerehashing yes

client-output-buffer-limit normal 0 0 0
client-output-buffer-limit replica 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60

hz 10
dynamic-hz yes

aof-rewrite-incremental-fsync yes
rdb-save-incremental-fsync yes
```

