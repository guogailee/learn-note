## 抛出异常

创建一个异常对象，把它交给runtime system处理

## Runtime system如何处理异常

runtime system收到异常后，会搜索方法的调用栈，按照相反的方向，从里层向外层找异常处理器，如果找到了，就把异常传递给该异常处理器处理，否则，继续向外层搜索。

```
main-A-B-C

runtime system的搜索顺序C-B-A-main
```

## 异常类图

## 异常处理器

try

catch

finally