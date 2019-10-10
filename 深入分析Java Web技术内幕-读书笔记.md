## 13 Spring框架的设计理念与设计模式分析

### Spring解决的问题

- 问题：

  对象的依赖关系

- 如何解决的：

  依赖注入机制，把对象之间的依赖关系用配置文件来管理，管理依赖关系的容器是Ioc容器（包装了Bean对象）。

### Bean组件

- Bean的定义

  ```
  BeanDefinition接口-对应<bean/>
  ```

- Bean的解析

- Bean的创建

  ```
  DefaultListableBeanFactory类
  ```

### Ioc容器-Context

- 基础接口

  ```
  ApplicationContext
  ```

- 核心功能

  ```
  1.标识一个应用环境
  2.利用BeanFactory创建bean对象
  3.保存对象关系表
  4.能够捕获各种事件
  ```

- 核心实现

  ```
  ClassPathXmlApplicationContext
  FileSystemXmlApplicationContext
  ```

- 启动流程

  ```
  入口：AbstractApplicationContext.refresh()->
  1.构建BeanFactory
  2.注册可能感兴趣的事件postProcessors
  3.创建Bean实例对象
  4.触发被监听的事件
  
  构建BeanFactory，在AbstractApplicationContext的子类的refreshBeanFactory()->
  1.创建BeanFactory
  2.加载BeanDefinitions
  
  创建Bean实例对象，finishBeanFactoryInitialization();->
  1.getBean()
  2.doGetBean()
  3.构建依赖关系
  
  创建Bean的扩展点：
  1.BeanPostProcessor#postProcessBeforeInitialization()
  2.BeanPostProcessor#postProcessAfterInitialization()
  3.DisposableBean#destroy()
  4.InitializingBean#afterPropertiesSet()
  
  ```

### 资源的是如何加载的

- 通过封装统一的资源接口

  ```
  Resource-包装了InputStream（java.io）
  ```

- 通过封装统一的资源加载接口

  ```
  ResourceLoader{
  	Resource getResource(String location);
  	ClassLoader getClassLoader();
  }
  ```

### 容器是如何拥有资源加载功能的

```
组合ResourceLoader从而拥有加载功能
```

### JDK动态代理-基础是字节码生成技术+反射

```
Proxy.newProxyInstance(ClassLoader loader,
                       Class<?>[] interfaces,
                       InvocationHandler h)->
1.构造代理类类名String proxyName = proxyPkg + proxyClassNamePrefix + num
2.生成代理类字节码ProxyGenerator.generateProxyClass();
3.class对象反射获取构造器构造实例对象
4.生成的代理类
$Proxy0 extends Proxy implements XXXInterface{
	Method m3=;
	Method m2=;
	static{
	m3 = Class.forName("XXXInterface").getMethod("sayHello",new Class[0]);
	m2 = Class.forName("java.lang.Object").getMethod("toString",new Class[0]);
	}

	sayHello(){
		this.h.invoke(this,m3,null);
		return;
	}
	
	toString(){
	this.h.invoke(this,m2,null);
	return;
	}
}

生成的代理类：
为接口的每一个方法和equals\toString\hashCode都生成了对象的实现，实现是统一调用了invocationHandler对象的invoke方法。

代理的原理：
把调用目标方法转而调用invocationHandler对象的invoke方法。
```

### Spring AOP

- 实现的关键

  ```
  如何在invocationHandler上做文章
  ```

- 