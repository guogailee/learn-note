## Dubbo解决的需求

- 服务注册中心：动态注册和发现服务
- 软负载均衡
- 自动画出服务之间的依赖关系
- 监控服务的调用量，访问时间，用于规划物理机器如何配置

## Dubbo架构

![dubbo-architecture](img/dubbo-architecture.jpg)

```
Container：负责启动、加载、运行服务提供者

Monitor：负责统计各服务调用次数、调用时间。统计先在提供者和消费者内存中汇总，每一分钟发送到monitor。

Provider：汇报调用时间到monitor，不包含网络开销

Consumer：从注册中心获取服务提供者地址列表，根据负载均衡算法直接调用提供者，汇报调用时间到monitor，包含网络开销

注册中心：长连接服务提供者，服务提供者宕机，立即推送事件通知消费者

注册中心宕机，不影响已运行的服务提供者和消费者，消费者在本地缓存了服务提供者列表。
```

## Dubbo用法

### 配置

```
服务提供者
<bean id=“xxxService” class=“com.xxx.XxxServiceImpl” /> 
<!-- 增加暴露远程服务配置 -->
<dubbo:service interface=“com.xxx.XxxService” ref=“xxxService” /> 

服务消费者
<!-- 增加引用远程服务配置 -->
<dubbo:reference id=“xxxService” interface=“com.xxx.XxxService” />
```

### 注解

```
服务提供者
@Service
XxxServiceImpl

服务消费者
@Reference
XXXService xxxService;
```

## Dubbo使用示范

### 服务提供者

```
1.定义服务接口
public interface DemoService {
    String sayHello(String name);
}

2.实现服务接口
public class DemoServiceImpl implements DemoService {
    public String sayHello(String name) {
        return "Hello " + name;
    }
}

3.暴露服务接口(xml配置or注解方式)
<beans>
	<!-- 提供方应用信息，用于计算依赖关系 -->
	<dubbo:application name="hello-world-app"  />
	
	<!-- 使用multicast广播注册中心暴露服务地址 -->
	<dubbo:registry address="multicast://224.5.6.7:1234" />
	
	<!-- 用dubbo协议在20880端口暴露服务 -->
	<dubbo:protocol name="dubbo" port="20880" />
	
	<!-- 声明需要暴露的服务接口 -->
	<dubbo:service interface="org.apache.dubbo.demo.DemoService" ref="demoService" />
	
	<!-- 和本地bean一样实现服务 -->
	 <bean id="demoService" class="org.apache.dubbo.demo.provider.DemoServiceImpl" />
</beans>
```

### 服务消费者

```
1.引用远程服务
<beans>
	<!-- 消费方应用名，用于计算依赖关系，不是匹配条件，不要与提供方一样 -->
	<dubbo:application name="consumer-of-helloworld-app"  />
	
	<!-- 使用multicast广播注册中心暴露发现服务地址 -->
	<dubbo:registry address="multicast://224.5.6.7:1234" />
	
	<!-- 生成远程服务代理，可以和本地bean一样使用demoService -->
	<dubbo:reference id="demoService" interface="org.apache.dubbo.demo.DemoService" />
</beans>
```

## Dubbo配置

### XML配置

| 标签                 | 用途         | 解释                                                         | 对应的类        |
| -------------------- | ------------ | ------------------------------------------------------------ | --------------- |
| <dubbo:service/>     | 服务配置     | 用于暴露服务，一个服务用多个协议暴露，一个服务可以注册到多个注册中心 | ServiceConfig   |
| <dubbo:reference/>   | 引用配置     | 用于创建一个远程服务代理，一个引用可以指向多个注册中心       | ReferenceConfig |
| <dubbo:protocol/>    | 协议配置     | 用于配置提供服务的协议信息，协议由提供方指定，消费方被动接受 | ProtocolConfig  |
| <dubbo:application/> | 应用配置     | 用于配置当前应用信息，不管该应用是提供者还是消费者           |                 |
| <dubbo:module/>      | 模块配置     | 用于配置当前模块信息，可选                                   |                 |
| <dubbo:registry/>    | 注册中心配置 | 用于配置连接注册中心相关信息                                 |                 |
| <dubbo:monitor/>     | 监控中心配置 | 用于配置连接监控中心相关信息，可选                           |                 |
| <dubbo:method/>      | 方法配置     | 用于 ServiceConfig 和 ReferenceConfig 指定方法级的配置信息   |                 |
| <dubbo:argument/>    | 参数配置     | 用于指定方法参数配置                                         |                 |

### API配置

- 服务提供者

  ```
  // 连接注册中心配置
  RegistryConfig registry = new RegistryConfig();
  registry.setAddress("10.20.130.230:9090");
  registry.setUsername("aaa");
  registry.setPassword("bbb");
  
  // 服务提供者协议配置
  ProtocolConfig protocol = new ProtocolConfig();
  protocol.setName("dubbo");
  protocol.setPort(12345);
  protocol.setThreads(200);
  
  // 服务提供者暴露服务配置
  ServiceConfig<XxxService> service = new ServiceConfig<XxxService>(); // 此实例很重，封装了与注册中心的连接，请自行缓存，否则可能造成内存和连接泄漏
  service.setApplication(application);
  service.setRegistry(registry); // 多个注册中心可以用setRegistries()
  service.setProtocol(protocol); // 多个协议可以用setProtocols()
  service.setInterface(XxxService.class);
  service.setRef(xxxService);
  service.setVersion("1.0.0");
  
  // 暴露及注册服务
  service.export();
  ```

- 服务消费者

  ```
  // 连接注册中心配置
  RegistryConfig registry = new RegistryConfig();
  registry.setAddress("10.20.130.230:9090");
  registry.setUsername("aaa");
  registry.setPassword("bbb");
  
  // 引用远程服务
  ReferenceConfig<XxxService> reference = new ReferenceConfig<XxxService>(); // 此实例很重，封装了与注册中心的连接以及与提供者的连接，请自行缓存，否则可能造成内存和连接泄漏
  reference.setApplication(application);
  reference.setRegistry(registry); // 多个注册中心可以用setRegistries()
  reference.setInterface(XxxService.class);
  reference.setVersion("1.0.0");
   
   // 和本地bean一样使用xxxService
  XxxService xxxService = reference.get();
  ```

- 点对点直连-测试环境绕过注册中心测试可以使用这种方式

  ```
  ReferenceConfig<XxxService> reference = new ReferenceConfig<XxxService>();
  reference.setUrl("dubbo://10.20.130.230:20880/com.xxx.XxxService"); 
  ```

### 注解配置

- 服务提供者

  ```
  @Service
  public class XXXServiceImpl implements XXXService{}
  
  开启自动扫描
  ```

- 服务消费者

  ```
  @Reference
  private XXXService xxxService;
  
  开启自动扫描
  ```

### 配置覆盖关系(优先级依次降低)

- JVM System Properties,-D参数
- 外部化配置
- ServiceConfig 、ReferenceConfig等编程接口采集的配置
- 本地配置文件dubbo.properties

### 不同层次都有配置，以哪个为主

- 方法级优先，接口级次之，最后全局配置
- 级别相同，消费方优先，服务方次之

### 在哪里设置超时时间

- 由服务提供者设置超时时间，每一个方法执行多长时间，服务提供者最清楚
- 服务消费者同时引用多个服务，不需要关心每个服务的超时设置

### 引用服务是何时生成代理服务的

- 当注入到其他的bean，或者getBean()被获取时候，才初始化生成代理对象
- 如果需要提前初始化，可以配置<dubbo:reference   init="true"/>

### 动态配置中心

- 结合apollo/zk来实现，以外部配置中心为主，读不到再应用本地的配置

## 示范用法

### 启动时检查

```
check="false"关闭检查

何时使用？测试时，有些服务不关心；出现循环依赖，必须有一方先启动；Spring容器懒加载；
```

**关闭某个服务的启动时检查**

```
<dubbo:reference interface="com.foo.BarService" check="false"/>
```

**关闭所有服务的启动时检查**

```
<dubbo:consumer check="false"/>
```

**关闭注册中心启动时检查**

```
<dubbo:registry check="false"/>
注册服务到注册中心失败，也允许启动，会在后台定时重试注册到注册中心
```

### 集群容错

当集群调用失败时，Dubbo提供了多种容错方案，默认是failover重试，可自定义集群容错

**Failover Cluster**

```
调用失败，切换其他服务器重试

适用于：

提供者：
<dubbo:service retries="2" />

消费者：
<dubbo:reference retries="2"/>

<dubbo:reference>
	<dubbo:method name="findFoo" retries="2">
</dubbo:reference>
```

**Failfast Cluster**

```
失败立即报错

适用于：非幂等性的写操作，比如新增记录
```

**Failsafe Cluster**

```
失败直接忽略

适用于：写入审计日志等操作
```

**Failback Cluster**

```
失败后，后台会记录失败请求，定时重发，区别与failover,它是立即重发

适用于：消息通知等操作
```

**Forking Cluster**

```
并行调用多个服务器，只要一个成功就返回
```

**Broadcat Cluster**

```
广播调用所有提供者，任意一台失败就报错

适用于：通知所有提供者更新缓存或者日志
```

**如何配置集群模式**

```
提供者：
<dubbo:service cluster="failsafe" />

消费者：
<dubbo:reference cluster="failsafe" />
```

### 负载均衡策略

当集群负载均衡时，Dubbo提供了多种负载均衡策略，默认是random，可自定义负载均衡策略

**Random LoadBalance**

```
随机，按照权重设置随机概率，比如5：3：2

优点：可以根据请求权重，调整提供者的权重
```

**RoundRobin LoadBalance**

```
轮询，按权重设置轮询比率

缺点：如果某台机器慢，会导致请求每次都卡在那台机器上
```

**LeastActive LoadBalance**

```
最少活跃调用数

优点：使慢的提供者收到更少的请求
```

**ConsistentHash LoadBalance**

```
一致性Hash，相同参数的请求总是发送到同一个提供者

当某一个提供者挂了，基于虚拟节点，不会导致hash后发送剧烈变动
```

**如何配置**

```
提供者：
<dubbo:service interface="" loadbalance="roundrobin"/>

消费者：
<dubbo:reference interface="" loadbalance="roundrobin"/>

方法级别的
```

### 直连提供者

测试阶段测试用

```
<dubbo:reference id="XXXService" interface="XXXService" url="dubbo://localhost:20890,dubbo://localhost:20890">

绕过注册中心
```

### 如何解决正在开发的服务影响正常消费者的问题

```
开发测试环境使用同一个zk的时候：
正在开发的服务提供者，不注册正在开发的服务，只订阅自己需要的服务。

正在开发的服务消费者，直连正在开发的服务提供者。
```

禁用注册

```
<dubbo:registry address="10.20.153.10:9090" register="false" />
```

### 多协议

不同服务在性能上适合采用不同的协议进行传输

**不同服务不同协议**

```
<beans>
	<dubbo:protocol name="dubbo" port="20880"/>
	<dubbo:protocol name="rmi" port="20880"/>
	
	<dubbo:service interface="XXXService" version="1.0.0" ref="helloService" protocol="dubbo"/>
	<dubbo:service interface="XXXService" version=1.0.0
	ref="demoService" protocol="rmi"/>
</beans>
```

**同一服务多种协议**

```
<beans>
	<dubbo:protocol name="dubbo" port="20880"/>
	<dubbo:protocol name="hessian" port="8080"/>
	
	<dubbo:service id="helloService" interface="XXXService" version="1.0.0" protocol="dubbo,hessian">
</beans>
```

### 泛化-消费端

```
作用：通过泛化调用，可以调用所有的服务

<dubbo:reference id="barService" interface="com.foo.BarService" generic="true"/>

使用：
GenericService barService = (GenericService)applicationContext.getBean("barService");
Object result = GenericService.$invoke("sayHello",new String[]{"java.lang.String"},new Object[]{"world"});
```

### 泛化-提供者

```
作用：泛化提供服务，用于mock，通用型提供服务

泛化实现：
GenericImpl implement GenericService{
	public Object $invoke(String methodName,String[] parameterTypes,Object[] args){
		if("sayHello".equals(methodName)){
			return "Welcome"+args[0];
		}
	}
}

暴露服务：
<bean id="genericService" class="com.foo.GenericImpl"/>
<dubbo:service interface="com.foo.BarService" ref="genericService"/>
```

### 回声测试

作用：用于检测服务是否可用，所有的服务自动都实现了EchoService接口

```
如何测试：
<dubbo:reference id="memberService" interface="com.xxx.MemberService"/>

//测试
EchoService echoService =(EchoService) memberService;
String status = echoService.$echo("ok");
```

### 上下文信息

```
RpcContext：ThreadLocal的临时状态处理器，存储所需的环境信息

每次请求，RpcContext里存储的信息都会变更
```

### 隐式传参

```
借助RpcContext

消费者：
RpcContext.getContext().setAttachment("index","1");
xxxService.xxxMethod();

提供者：
xxxMethod(){
	String index = RpcContext.getContext.getAttachment("index");
}
```

### 异步调用

```
作用：消费者异步拿到提供者的返回

提供者：接口定义返回体需要是CompletableFuture
public interface AsyncService {
	CompletableFuture<String> sayHello(String name);
}

消费者：
CompletableFuture<String> future = 
asyncService.sayHello("hello");
future.whenComplete(
	(v,t)->{
		sout("response,v:{}",v);
	}
);
//早于请求返回执行
sout("请求发出了");

```

### 本地伪装--降级方案的一种

作用：当服务提供者不可用的时候，通过mock数据返回app

```
<dubbo:reference interface="com.foo.BarService" mock="com.foo.BarServiceMock" />

public class BarServiceMock implements BarService {
    public String sayHello(String name) {
        // 你可以伪造容错数据，此方法只在出现RpcException时被执行
        return "容错数据";
    }
}
```

### 延迟暴露

```
<dubbo:service delay="5000" />
```

### 并发控制

```
作用：限制服务端提供服务的并发数

<dubbo:service interface="com.foo.BarService" executes="10" />
```

## 协议参考手册

### dubbo协议

```
连接个数：单连接（减少了TCP握手的开销）
连接方式：长连接
传输协议：TCP
传输方式：NIO 异步传输
序列化：Hessian 二进制序列化
适用范围：传入传出参数数据包较小（建议小于100K），消费者比提供者个数多，单一消费者无法压满提供者，尽量不要用 dubbo 协议传输大文件或超大字符串。
适用场景：常规远程服务方法调用
```

### rmi协议

### hessian协议

```
连接个数：多连接
连接方式：短连接
传输协议：HTTP
传输方式：同步传输
序列化：Hessian二进制序列化
适用范围：传入传出参数数据包较大，提供者比消费者个数多，提供者压力较大，可传文件。
适用场景：页面传输，文件传输，或与原生hessian服务互操作
```

### http协议

```
连接个数：多连接
连接方式：短连接
传输协议：HTTP
传输方式：同步传输
序列化：表单序列化
适用范围：传入传出参数数据包大小混合，提供者比消费者个数多，可用浏览器查看，可用表单或URL传入参数，暂不支持传文件。
适用场景：需同时给应用程序和浏览器 JS 使用的服务。
```

## 推荐用法

### 在Provider端尽可能多配置Consumer端属性

```
Provider端配置后，Consumer端如果没有配置，就会使用Provoder端的配置

建议在Provider端配置的Consumer端属性有：
1.timeout
2.retries
3.loadbalance
4.actives最大
```

## Dubbo源码解析

### Dubbo SPI机制

### 自适应拓展机制

### 服务导出