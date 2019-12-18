## HashMap的数据结构

```
数组+链表/红黑树
```

```
数组元素：
jdk7
Entry{
	hash
	key
	value
	Entry<K,V> next
}

jdk8
Node{
	hash
	key
	value
	Node<K,V> next
}
```

## 链表插入的位置

```
当hash(key)一样后，遇到冲突，放入链表
jdk8之前头插法：
因为写这个代码的作者认为新值被查找的可能性更大一点，提升查找的效率。

jdk8尾插入法：

```

## 数组容量达到阈值，如何扩容

```
何时扩容：
当元素元素个数>容量*负载因子(0.75)
```

```
Hash的公式---> index = HashCode（Key） & （Length - 1）
扩容的时候数组长度变化，所以需要rehash
```

```
扩容步骤：
1.扩容：创建一个新的Entry空数组，长度是原数组的2倍。
2.rehash：遍历原Entry数组，把所有的Entry重新Hash到新数组。
```



