## Java值传递VS引用传递

- 值传递：形参是实参的一个副本，对实参的修改不影响实参。
- 引用传递：实参和形参在内存中指向的是同一个地址。

## Java equals() 和 hashCode() 一起重写

```
1.如果两个对象equal，那么他们必须有相等的hashCode
2.如果两个有相等的hashCode，他们可能equal也可能不equal
```

## String.intern()是如何节省内存的

```
String.intern() native方法
1.作用：如果字符串常量池中已经包含一个等于此String对象的字符串，则返回该String对象的引用；否则，将此String对象添加到常量池中，并且返回此String对象的引用。

2.内存分布差异：
jdk6 永久代 intern() 会把首次遇到的字符串实例复制到永久代中，返回的也是永久代中这个字符串实例的引用
jdk7 堆 intern() 不会复制实例，只是在堆中的常量池记录首次出现的实例引用 
```

## 数据库设计三大范式

- 第一范式(1NF)：数据库表的每一项都是不可分割的原子数据项。-原子性

  ```
  学号  家庭信息
  1     3口人，北京
  
  //不满足第一范式原子性，修改为
  学号  家庭人口  户籍
  1    3口人     北京
  ```

- 第二范式(2NF)：每列都和主键相关

  ```
  订单号 sku 产品数量 产品折扣 产品价格 订单金额 订单时间
  100   891
  100   892
  
  //不满足第二范式，一个订单可以有多个商品，主键是订单号和sku一起的。订单金额，订单时间跟sku没关系。修改为两张表。
  订单号 sku 产品数量 产品折扣 产品价格
  订单号 订单金额 订单时间
  
  ```

- 第三范式(3NF)：每一列都和主键直接相关，而不能间接相关

  ```
  学号 姓名 性别 班主任工号 班主任姓名 班主任性别
  
  //不满足第三范式，班主任姓名 班主任性别和学号不是直接相关，修改为两张表
  学号 姓名 性别 班主任工号
  班主任工号 班主任姓名 班主任性别
  ```

- 如何看待数据库设计三大范式

  ```
  不必完全遵守，有的时候可以用空间换时间，比如查询学生的所有信息，需要查多张表，但是不按照三大范式，设计一张大表，空间上大了，但是时间上只用查一张表。
  ```

## 什么是mysql索引，索引类型有哪些

- 索引

  ```
  索引：是关系型数据库中给数据库表中一列或多列的值排序后的存储结构。
  ```

- 索引类型

  ```
  唯一索引
  主键索引
  普通索引
  全文索引
  ```

- 聚集索引 vs 非聚集索引

  ```
  聚集索引：
  数据行的物理顺序与列值的逻辑顺序相同。
  
  由于物理顺序与聚集索引的顺序相同，物理顺序只有一个，所以只能有一个聚集索引。
  
  比如：主键，自增，对应的地址也是自增的。
  
  非聚集索引：
  数据行在磁盘上的物理存储顺序与索引的逻辑顺序不同。
  
  一个表可以拥有多个非聚集索引。
  
  除了聚集索引，其余都是非聚集索引(普通索引，全文索引等)
  
  ```

- 覆盖索引

  ```
  定义：select的列只用从索引中就能够取得，不需要二次查询，不必从数据库表中读取。
  查询的列要被索引覆盖。
  
  username是普通索引，age不是索引
  select username, age from user where username = 'lee';
  需要二次查询
  
  但如果username,age,sex都是索引，
  select username, age from user where username = 'lee';
  select username, age, sex from user where username = 'lee';
  不需要二次查询
  ```

## http请求的四种方式

- get

  ```
  用于获取信息，安全幂等。
  
  参数位置：参数以？拼接在url后面，用&分割
  
  长度限制：http协议是没有长度限制的，对url长度限制的是浏览器和服务器。
  
  安全性：明文传输，不安全，需要https
  ```

- post

  ```
  用于修改服务器的内容，也可用于获取数据。
  
  参数位置：参数放在body中
  
  安全性：明文传输，不安全，需要https，并没有比get安全
  
  ```

- delete：删除数据，通过post可实现

- put：增加数据，指定了资源的存放位置，通过post可实现

## 死锁的生成条件？如何避免死锁？

## 如何模拟高并发

## 什么是内存屏障

## java探针如何使用

## 什么时候栈会溢出

## G1垃圾回收器有何特点，与CMS有啥区别

## AQS与CAS的区别

## zk的应用场景

- 

## 序列化的方式

- 为啥要序列化
- java原生序列化
- hessian序列化
- json序列化
- dubbo rpc的序列化方式：使用的是hessian2，改造后的hessian。

## Map实现类有哪些和相应的应用

- Map类图
- HashMap vs TreeMap vs HashTable vs LinkedHashMap 
- ConcurrentHashMap源码解析

## 工作中有遇到哪些比较有挑战的事情，如何解决的

## Spring： IOC原理

## Spring： aop原理

## Spring： @Transactional注解原理

## Spring：状态机实现过程

## 多线程：如何保证线程安全

## JVM：new一个对象的时候，JVM都做了哪些事情

```
之前没有进行类加载：
1.类加载，同时初始化类中的静态的属性
2.执行静态代码块
3.分配内存空间，同时初始化非静态的属性
4.调用父类构造器
5.父类构造器执行完后，如果自己声明属性的同时有显式的赋值，那么进行显式赋值把默认值覆盖
6.执行匿名代码块
7.执行构造器
8.返回内存地址

之前已经进行了类加载：
1.分配内存空间，同时初始化非静态的属性
2.调用父类构造器
3.父类构造器执行完后，如果自己声明属性的同时有显式的赋值，那么进行显式赋值把默认值覆盖
4.执行匿名代码块
5.执行构造器
6.返回内存地址
```

## JVM：类加载机制

```
双亲委派机制：
```

## JVM：内存分区

## MQ：发送消息重试机制

## 分布式：下单链路如何保证一致性

```
下单，扣库存，使用优惠券，任何一个节点失败，如何保证一致性？

比如，优惠券使用失败，但是库存已经扣了，如何恢复库存呢？
方案一：实时异常单系统去处理，调用逻辑解锁库存+定时任务扫描库存扣了，优惠扣失败状态的订单，去解锁库存。
```

## redis：用处有哪些

## redis：为啥单线程的redis这么快

## redis：过期策略以及内存淘汰机制

## redis：与数据库双写一致性问题

## redis：如何应对缓存穿透和缓存雪崩问题

## redis：如何解决并发竞争key问题

## 系统设计：商品中心如何设计

```
前后台类目
product
item
sku
```

## mysql：count(*) count(1) count(col)

```
1.表只有一个字段的话，用count(*)最快

2.count(主键/联合主键)比count(*)快

3.表没有主键，count(1)比count(*)快

4.where限制下，count(*)比count(col)快

5.count(*)统计包含null，count(col)统计不包含null
```

## 网络：TCP/IP三次握手