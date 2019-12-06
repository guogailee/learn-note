# Object类

## Object源代码

```
public class Object {
	public native int hashCode();
	
	public boolean equals(Object obj) {
		return this==obj;
	}
	
	public String toString(){
		return getClass().getName()+"@"+Integer.toHexString(hashCode());
	}
	
	public final native void wait(long timeout) throws InterruptedException;
	
	public final native void notify();
	
	public final native void notifyAll();
}
```

## hashCode()和equals()

### equals()的通用约定

```
何时应该重写equals()方法：
如果类具有自己特有的逻辑相等概念，而且超类还没有覆盖equals以实现期望的行为，这时我们就需要覆盖equals方法。通常属于值类的情形。比如Integer。

何时不需要重写equals()方法：
1.类的每个实例本质上都是唯一的。对于代表活动实体而不是值的类来说确实如此，例如Thread。
2.不关心类是否提供了逻辑相等的测试功能。比如Random继承从Object得到的equals就够用了。因为随机数不需要equals。
3.超类已经覆盖了equals，从超类继承过来的行为对于子类也是合适的。
4.类是私有的或是包级私有的，可以确定它的equals方法永远不会被调用。

通用约定：满足对称性、传递性、一致性。
1.使用==操作符检查“参数是否为这个对象的引用”。如果是，则返回true。
2.使用instanceof操作符检查“参数是否为正确的类型”。如果不是，则返回false。
3.把参数转换成正确的类型。
4.对于该类中的每个“关键域”，检查参数中的域是否与该对象中对应的域相匹配。如果测试全部成功，则返回true。否则返回false。
5.不要将equals声明中的Object对象替换为其他的类型。
```

### hashCode()的通用约定

```
1.如果两个对象根据equals()方法比较是相等的，那么调用这两个对象的hashCode()方法必须相等
2.如果两个对象根据equals()方法不相等，调用hashCode()不要求不相等，但是为了提高散列性能，让hashCode也相等
```

### 为何重写equals()一定要重写hashCode()

```
如果一个类重写equals()，不重写hashCode()，那么该类无法结合基于散列的集合一起工作。

object.hashCode()是内存地址经过hash函数计算后得到的
```

### String的hashCode()和equals()

```
1.equals()
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
每一个字符比较，有不相等就false

2.hashCode()
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

### HashMap的hashCode()和equals()

```
1.equals()
通过当前hashMap的迭代器，比较每个Entry的key-value和要比较的对象的key-value是不是相等，遇到不相等就false

2.hashCode()
通过迭代器求entry的hashCode的总和
其中entry的hashCode=Objects.hashCode(key) ^ Objects.hashCode(value);
```

## 始终要覆盖toString()

```
被返回的字符串应该是一个“简洁的，但信息丰富，并且易于阅读的表达形式”。

默认的：类名+@+哈希值，难以理解。
```

