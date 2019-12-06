## Java注解

- 元注解

  - Documented：标记生成javadoc
  - Inherited：标记继承关系
  - Retention：注解的生存期
  - Target：标注的目标

- 标记注解

  - Override：覆盖超类中的方法
  - Deprecated：标记为丢弃的类
  - SuppressWarnings：抑制编译器发出的警告

- 自定义注解：配合反射使用

  ```
  XXX xxx = (XXX)class.getAnnotation(XXX.class);
  ```

## 打印时间差

- 使用StopWatch替换

## 关键字transient

## Map接口及其实现类

### HashMap

```
数据结构：
1.7 数组(元素是Entry)+链表
1.8 Node+链表

特点：
键无序，唯一
值有序，可重复
允许键null，值null

线程安全：
put不安全
```

### LinkedHashMap

```
数据结构：
数组(元素Entry，包含前后指针)+链表

特点：
键有序，唯一
值有序，可重复
允许键null，值null

线程安全:
put不安全
```

### TreeMap

```
数据结构：
平衡二叉树

特点：
键有序，唯一
值有序，可重复

线程安全：

```

### HashTable

```
数据结构：
哈希表

特点：
不允许null键和null值

线程安全：
线程安全
```

## NIO

### NIO

```
non-blocking io：
同一个线程：channel读数据到buffer，与此同时，可以做其他事情

同一个线程：写数据到channel，与此同时，可以做其他事情
```

```
selectors选择器：
一个selector可以监视多个channel事件
```

### 核心组件

**Channel&Buffer**

```
Channel通道，类似流
Buffer缓冲区

Channel读数据到Buffer、Buffer写数据到Channel

Channel抽象类：
FileChannel-对应的有具体的实现类
DatagramChannel
SocketChannel
ServerSocket

Buffer具体类：
ByteBuffer
CharBuffer
DoubleBuffer
FloatBuffer
IntBuffer
LongBuffer
ShortBuffer
MappedByteBuffer-特殊
```

![overview-channels-buffers](img/overview-channels-buffers.png)

Channel和流的区别

```
Channel是双向的，可以写入channel，也可以从channel读
流是单向的
```

Buffer

```
可以写入数据
可以从Buffer读数据

如何切换读写模式，通过flip()

读完后需要clean()清除buffer，以继续提供写服务

compact()只会清除已经读的数据，未读的数据不会清掉
```

Buffer的三个属性

```
1.Capacity容量

2.Position当前位置 

3.Limit写的限制是容量大小，读的限制是写入了多少数据
```

如何获取一个Buffer

```
ByteBuffer buf = ByteBuffer.allocate(int capacity);
```

如何把数据写入Buffer

```
channel.read(buf);
```

Buffer从写模式切换到读模式

```
flip(){
	limit = position;
	position = 0;
}
```

如何从内存块Buffer中读数据

```
int bytesWritten = channel.write(buf);
```

**Selector**

![selectors](img/selectors.png)

```
selector允许，一个线程可以处理多个channel

适应场景：聊天程序，少量线程处理多数连接
```

### 聚集操作

- 聚集从Channel读入Buffer

  ```128
  ByteBuffer header = ByteBuffer.allocate(128);
  ByteBuffer body   = ByteBuffer.allocate(1024);
  
  ButeBuffer[] bufferArray = {header,body};
  
  channel.read(bufferArray);
  
  当第一个buffer满了，才会移动到下一个buffer。
  适合场景：
  header的大小刚好是固定大小的
  ```

- 聚集从Buffer写入Channel

  ```
  ByteBuffer header = ByteBuffer.allocate(128);
  ByteBuffer body   = ByteBuffer.allocate(1024);
  
  ByteBuffer[] bufferArray = {header,body};
  
  channel.write(bufferArray);
  ```


## 线程池

## 多线程

### Synchronized

```

```

### AbstractQueuedSynchronizer

```
AQS提供了一种实现阻塞锁和一系列依赖FIFO等待队列的同步器的框架

许多同步类实现都依赖于它，如常用的ReentrantLock/Semaphore/CountDownLatch...。
```

### ReentrantLock

```

```

### CountDownLatch

```

```

### CyclicBarrier



### Semaphore

```

```

### 线程间通信方式

```

```

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

## 