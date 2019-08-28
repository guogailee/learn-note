## Spring bean的作用域

| 作用域    | 定义                                                   |
| --------- | ------------------------------------------------------ |
| singleton | 在一个spring容器中，一个bean定义只有一个对象实例(默认) |
| prototype | 每次调用都创建一个实例                                 |
| request   | 在一次http请求中，每个bean对应一个实例                 |
| session   | 在一个http session中，每个bean对应一个实例             |

- 何时需要prototype，当请求方不能共享对象实例的时候。