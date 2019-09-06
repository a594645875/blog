## 1. 创建Redis集群
在`/usr/local/docker/redis`目录下创建`docker-compose.yml`
```
version: '3.1'
services:
  master:
    image: redis
    container_name: redis-master
    ports:
      - 6379:6379

  slave1:
    image: redis
    container_name: redis-slave-1
    ports:
      - 6380:6379
    command: redis-server --slaveof redis-master 6379

  slave2:
    image: redis
    container_name: redis-slave-2
    ports:
      - 6381:6379
    command: redis-server --slaveof redis-master 6379
```
运行`docker-compose up -d`命令
然后用`RedisDesktopManager`连接`Redis`, 连接成功代表建立集群成功.

## 2.建立Sentinel集群

在`/usr/local/docker/Sentinel`目录下创建`docker-compose.yml`
```
version: '3.1'
services:
  sentinel1:
    image: redis
    container_name: redis-sentinel-1
    ports:
      - 26379:26379
    command: redis-sentinel /usr/local/etc/redis/sentinel.conf
    volumes:
      - ./sentinel1.conf:/usr/local/etc/redis/sentinel.conf

  sentinel2:
    image: redis
    container_name: redis-sentinel-2
    ports:
      - 26380:26379
    command: redis-sentinel /usr/local/etc/redis/sentinel.conf
    volumes:
      - ./sentinel2.conf:/usr/local/etc/redis/sentinel.conf

  sentinel3:
    image: redis
    container_name: redis-sentinel-3
    ports:
      - 26381:26379
    command: redis-sentinel /usr/local/etc/redis/sentinel.conf
    volumes:
      - ./sentinel3.conf:/usr/local/etc/redis/sentinel.conf
```
在`/usr/local/docker/Sentinel`目录下创建`sentinel1.conf`,`sentinel2.conf`, `sentinel3.conf`,内容相同.
```
port 26379
dir /tmp
# 自定义集群名，其中 127.0.0.1 为 redis-master 的 ip，6379 为 redis-master 的端口，2 为最小投票数（因为有 3 台 Sentinel 所以可以设置成 2）
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000
sentinel deny-scripts-reconfig yes
```
其中`127.0.0.1`替换成`Redis`的ip地址.
`6379`: 主Redis的端口,三个配置文件都是主Redis的端口.

测试: 运行
```
# 运行
docker-compose up -d

# 进入容器
docker exec -it redis-sentinel-1 /bin/bash

# 连接Sentinel
redis-cli -p 26379

# 查询主/从节点信息
sentinel master mymaster
sentinel slaves mymaster
```
看到以下信息,则表示`Sentinel`集群搭建成功
```
...
31) "num-slaves"
32) "2"
33) "num-other-sentinels"
34) "2"
35) "quorum"
36) "2"
...
```
