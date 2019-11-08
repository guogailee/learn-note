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

## Semaphore  

## CountDownLatch

- 作用：

- 源码解析：

  ```
  
  ```

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
链表

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



## 覆盖equals请遵守通用约定

- 何时不需要重写equals方法

  - 类的每个实例本质上都是唯一的。对于代表活动实体而不是值的类来说确实如此，例如Thread。

  - 不关心类是否提供了逻辑相等的测试功能。比如Random继承从Object得到的equals就够用了。因为随机数不需要equals。

  - 超类已经覆盖了equals，从超类继承过来的行为对于子类也是合适的。

  - 类是私有的或是包级私有的，可以确定它的equals方法永远不会被调用。

    ```
    @Override
    public boolean equals(Object o){
        throw new AssertionError();
    }
    ```

- 何时应该重写equals方法

  ```
  如果类具有自己特有的逻辑相等概念，而且超类还没有覆盖equals以实现期望的行为，这时我们就需要覆盖equals方法。通常属于值类的情形。比如Integer。
  ```

  通用约定：满足对称性、传递性、一致性。

  - 使用==操作符检查“参数是否为这个对象的引用”。如果是，则返回true。

  - 使用instanceof操作符检查“参数是否为正确的类型”。如果不是，则返回false。

  - 把参数转换成正确的类型。

  - 对于该类中的每个“关键域”，检查参数中的域是否与该对象中对应的域相匹配。如果测试全部成功，则返回true。否则返回false。

  - 其他约定

    ```
    1.覆盖equals时总要覆盖hashCode。(否则会导致该类无法结合所有基于散列的集合一起工作)
    2.不要将equals声明中的Object对象替换为其他的类型。
    ```

  - 例如

    ```
        //String的equals方法
        public boolean equals(Object anObject) {
            if (this == anObject) {
                return true;
            }
            if (anObject instanceof String) {
                String anotherString = (String)anObject;
                int n = value.length;
                if (n == anotherString.value.length) {
                    char v1[] = value;
                    char v2[] = anotherString.value;
                    int i = 0;
                    while (n-- != 0) {
                        if (v1[i] != v2[i])
                            return false;
                        i++;
                    }
                    return true;
                }
            }
            return false;
        }
    ```

## 覆盖equals时总要覆盖hashCode

- 如果两个对象根据equals方法比较是相等的，那么hashCode也一定相等。

- 如果两个对象根据equals方法比较是不相等的，那么hashCode不一定相等，但是，为了提供散列表的性能，不让他退化为链表，使hashCode计算出来不相等。

- 例子

  ```
     //String的hashCode
     public int hashCode() {
          int h = hash;
          if (h == 0 && value.length > 0) {
              char val[] = value;
  
              for (int i = 0; i < value.length; i++) {
                  h = 31 * h + val[i];
              }
              hash = h;
          }
          return h;
      }
  ```

## 始终要覆盖toString

- 被返回的字符串应该是一个“简洁的，但信息丰富，并且易于阅读的表达形式”。

  ```
  默认的：类名+@+哈希值，难以理解。
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

  