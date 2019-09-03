# Mysql隐式转换

## 例子

- select 1+'1';

  结果是什么呢？

  ```
  2
  隐式转换发生，mysql自动把string转为number
  ```

- select concat(1,'1');

  结果又是什么呢？

  ```
  11
  隐式转换发生，mysql自动把number转为string
  ```

## 定义

- When an operator is used with operands of different types, type conversion occurs to make the operands compatible. Some conversions occur implicitly.

- 隐式转换：当操作符与不同类型的操作数一起使用时，为了使操作数兼容，类型转换就会发生，有一些转换是隐式的，由mysql自动完成。

- 显式转换：使用CAST函数

  - 语法：CAST(expr AS type)

  - 例子：

    select cast('1122a' as decimal);

  - 支持的类型：

    | 类型     | 备注                   |
    | -------- | ---------------------- |
    | DATE     | YYYY-MM-DD             |
    | DATETIME | YYYY-MM-DD HH:mm:ss    |
    | TIME     | HH:mm:ss               |
    | DECIMAL  | 通常用于带小数位       |
    | CHAR     | 固定长度字符串         |
    | NCHAR    | 类型于CHAR一致         |
    | SIGNED   | 一个有符号的64整数位   |
    | UNSIGNED | 一个无符号的64整数位   |
    | BINARY   | 二进制字符串           |
    | JSON     | MySQL 5.7.8 及更高版本 |

## 练习

- select '55aaa' = 55;

- select 'aa' + 1;

- select 'a' + 'b' = 'c';

- 字符串转数值

  ```
  select 'a' + 0;
  select 'a1122' +0;
  select '1a22'+0;
  select '1122'+0;
  ```

- 隐式转换导致的多查询

  ```
  use test;
  select * from user;
  
  select * from user where name = 1212;
  vs
  select * from user where name = '1212';
  ```

## 总结

- 两个参数至少有一个是NULL时，比较的结果也是NULL，例外是使用<=>（安全的等于）对两个NULL做比较时会返回1。这两种情况下不会发生隐式转换。

  ```
  select NULL = NULL;
  select NULL = 1;
  select NULL <=> NULL;
  ```

- 两个参数都是字符串，会按照字符串来比较，不做类型转换。

  ```
  select '1' = '1';//没转换
  select '1a'= 1;//有转换
  select 'a' + 'b';//有转换，因为+是算术操作符，字符串拼接用concat
  ```

- 两个参数都是整数，按照整数来比较，不做类型转换。

- 十六进制的值和非数字进行比较，会被当做二进制串。

- 有一个参数是timestamp/datetime，并且另一个参数是常量，常量会被转换为timestamp。

  ```
  select * from user where birth = 20180101;
  ```

- 有一个参数是decimal类型，如果另外一个参数是decimal或者整数，会把整数转换为decimal进行比较，如果另外一个参数是浮点数，则会把decimal转换为浮点数进行比较。

  ```
  select * from user where high = 178;
  ```

- 所有其他情况，两个参数都会被转换为浮点数进行比较。

- 参考：https://dev.mysql.com/doc/refman/5.7/en/type-conversion.html

## 隐式转换导致的索引失效

```
show index from user;
explain select * from user where name = 1212;
show warnings;会提示转换发生
vs
explain select * from user where name = '1212';
```

# Mysql索引失效的几种情况

- 最佳左前缀法则：查询从索引的最左前列开始，不要跳过索引中的列。不按照这个法则会导致索引失效。

  ```
  show index from user;
  explain select * from user where name ='lee';
  #使用到name
  
  explain select * from user where name ='lee'  and city = '北京';
  #使用到name,city失效
  
  explain select * from user where name ='lee'  and age = 18;
  #使用到name,city
  
  explain select * from user where name ='lee' and age = 18 and city = '北京';
  #使用到name,age,city
  
  explain select * from user where age = 18 and city = '北京';
  #失效
  
  explain select * from user where city = '北京';
  #失效
  
  ```

- 在索引上做操作(计算，函数，显式/隐式转换)，会导致索引失效

  ```
  explain select * from user where cast(name as decimal) =  '1212';
  #失效
  ```

- 索引中范围条件右边的索引会失效

  ```
  explain select * from user where name = 'lee' and age >5 and city ='北京';
  #使用到name,age
  ```

- 索引字段使用like以通配符%开头，索引失效 & 解决

  ```
  explain select * from user where name like '%l';
  #失效
  
  explain select * from user where name like 'l%';
  #使用到name
  
  如何解决这种情况？使用覆盖索引-索引的列包含了查询的所有列。(子集)
  explain select name from user where name like '%l';
  explain select name,age from user where name like '%l';
  explain select name,age,city from user where name like '%l';
  
  explain select name,age,city,password from user where name like '%l';
  #失效
  ```

- 索引字段是字符串，查询时不加引号导致的索引失效 & 解决

  ```
  explain select * from user where name = 1212;
  #失效(发生了隐式转换)
  ```

- 不同mysql版本，失效情况不同的几种情况

  - 索引字段使用or时，索引失效的情况

    ```
    select version();
    show index from user;
    explain select * from user where name = '1212' or name = 'lee';
    #8.0.12 不失效
    
    show index from retail_b2c_order;
    explain select * from retail_b2c_order where order_status = 130 or order_status = 300;
    #5.6.32 失效
    ```

  - 索引字段使用is null/is not null，索引失效的情况

    ```
    explain select * from retail_b2c_order where order_code is null;
    #5.6.32 不失效 (***)
    explain select * from retail_b2c_order where order_code is not null;
    #5.6.32 失效
    
    explain select * from user where name is null;
    explain select * from user where name is not null;
    #8.0.12 不失效
    ```

  - 索引字段使用 != / <> ，索引失效的情况

    ```
    explain select * from user where name != 'lee';
    explain select * from user where name <> 'lee';
    #8.0.12 不失效
    
    explain select * from retail_b2c_order where order_status != 130;
    explain select * from retail_b2c_order where order_status <>  130;
    #5.6.32 失效
    ```

- explain ：

  https://dev.mysql.com/doc/refman/5.7/en/explain-output.html

# 讨论一下Result如何返回

- DubboResult

  ```
  属性:
  private int code;
  private String msg;
  private boolean success;
  private T result;
  
  1.db insert失败
  DubboResult<Boolean> 
  返回:
  A:code=1,success=true,result=false
  B:code=-1,success=false,result=null
  C:throw OptimusExceptionBase(-1,"db failed");
  
  2.query查出0条记录
  DubboResult<List<xxxDTO>>
  返回:
  A:code=1,success=true,result=null
  B:code=-1,success=false,result=null
  C:throw OptimusExceptionBase(-1,"no records");
  ```

  http://dubbo.apache.org/zh-cn/docs/user/best-practice.html

# 附录

- user表建表语句

  ```
  CREATE TABLE `user` (
    `id` bigint(20) NOT NULL AUTO_INCREMENT,
    `name` varchar(64) DEFAULT NULL,
    `password` varchar(64) DEFAULT NULL,
    `age` int(11) DEFAULT NULL,
    `city` varchar(32) DEFAULT NULL,
    `high` decimal(18,2) DEFAULT NULL,
    `birth` date DEFAULT NULL,
    PRIMARY KEY (`id`),
    KEY `idx_name_age_city` (`name`,`age`,`city`)
  ) ENGINE=InnoDB AUTO_INCREMENT=8 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
  ```

- user表数据

  ```
  id	name	password	age	city	high	birth
  1	1212	qwer	NULL	NULL	NULL	NULL
  2	1212a	qwer	NULL	NULL	NULL	NULL
  3	lee	qwerd	18	杭州	NULL	NULL
  4	zhang	123456	18	北京	NULL	NULL
  5	liu	123	18	杭州	178.34	NULL
  6	liu2	123	18	杭州	178.00	NULL
  7	liu2	123	18	杭州	178.00	2018-01-01
  ```
