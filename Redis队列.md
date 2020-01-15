## Redis支持的数据结构

```
String Hash List Set SortedSet
```

## 数据结构

```
list-从双向链表实现的，尾部插入，头部获取
```

## 普通队列

```
lpush 插入 lpush key value
轮训调用rpop  获取 rpop key，性能差

brpop 获取，有元素返回，没有阻塞直到超时返回null
```

## 一次生产多次消费的队列

```
publish  发布
subscribe  订阅

缺点：消费者下线，生产者的消息会丢失，使用mq
```

## 延迟队列

```
使用sortedset，拿时间戳作为score，消息内容作为key调用zadd来生产消息，消费者用zrangebyscore指令获取N秒之前的数据轮询进行处理。
```

