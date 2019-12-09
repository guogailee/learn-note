# 初识Redis

## redis特性

- 速度快

  ```
  10万/s
  
  数据存内存，可以通过快照/日志持久化
  
  单线程架构，预防多线程可能产生的竞争
  
  用C语言实现
  ```

- 基于键值对的数据结构服务器

  ```
  值的数据结构：
  字符串、哈希、列表、集合、有序集合
  位图Bitmaps
  HyperLogLog
  GEO地理信息定位
  ```

- 丰富的功能

  ```
  提供键过期，用来实现缓存
  
  提供发布订阅功能，用来实现消息系统
  
  支持Lua脚本
  
  提供简单的事务功能
  
  提供了流水线功能(Pipeline),能够将一批任务传递到Redis
  ```

- 简单稳定

  ```
  代码量少
  
  redis本身几乎没有bug
  ```

- 持久化

  ```
  内存>RDB/AOF>磁盘
  ```

- 主从复制

  ```
  主从
  ```

- 高可用和分布式

  ```
  Redis Sentinal：保证节点的故障发现和故障自动转移 2.8版本
  Redis Cluster：分布式实现                    3.0版本
  ```

## Redis的使用场景

### 缓存

### 排行榜系统

### 计数器应用

```
视频播放量，用mysql数据库压力很大
```

### 社交网络

```
赞/踩、粉丝、共同好友、推送、下拉刷新不适合用关系型数据库
```

### 消息队列系统

## 正确安装并启动Redis

### yum工具

### 源码安装

```
wget url
tar xzf redis-4.0.0
ln -s redis-4.0.0 redis
cd redis
make
make install
```

### 启动redis服务器

- 配置文件启动

  ```
  redis-server /opt/redis/redis.conf
  
  配置
  port 端口
  logfile 日志文件
  dir 工作目录
  daemonize
  ```

- 默认配置启动

  ```
  redis-server
  
  启动后，默认端口6379
  ```

### Redis命令行客户端

- 交互式方式

  ```
  连接一次，不再需要连接
  
  redis-cli -h {host} -p {port}
  没有-h -p默认是127.0.0.1:6379
  
  set hello world
  get hello
  ```

- 命令式方式

  ```
  命令一次连接一次
  
  redis-cli -h {host} -p{port} get hello 
  ```

### 关闭redis

```
redis-cli shutdown (save|nosave)

过程：保存RDB文件快照，退出
```

# API的理解和使用

## 全局命令

### 查看所有键

```
keys * 
缺点：key太多，会遍历所有键，时间复杂度O(n)，线上禁止使用

使用scan指令替换，无阻塞
```

### 键总数统计

```
dbsize
直接获取的redis内置的键总数，时间复杂度O(1)
```

### 检查键是否存在

```
exists key
存在返回1，不存在返回0
```

### 删除键

```
del key
del key1 key2 key3
```

### 设置键过期时间

```
expire key seconds
```

### 查看键剩余过期时间

```
ttl key

返回：
>=0 键剩余过期时间
-1 键没设置过期时间
-2 键不存在
```

### 查看键的数据结构类型

```
type key
```

## 数据结构和内部编码