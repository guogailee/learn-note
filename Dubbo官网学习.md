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