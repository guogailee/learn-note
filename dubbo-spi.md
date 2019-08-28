## Java SPI机制

- Service Provider Interface,一种服务发现机制。

- 本质：将接口实现类的全限定类名配置在文件中，并由服务加载器读取配置文件，加载实现类，运行时动态替换实现类。

  ```
  1.定义接口
  public interface Robot {
      void sayHello();
  }
  
  2.定义实现类
  public class OptimusPrime implements Robot {
      
      @Override
      public void sayHello() {
          System.out.println("Hello, I am Optimus Prime.");
      }
  }
  
  public class Bumblebee implements Robot {
  
      @Override
      public void sayHello() {
          System.out.println("Hello, I am Bumblebee.");
      }
  }
  
  3.配置文件META-INF/services，文件名为Robot 的全限定名 org.apache.spi.Robot
  文件内容：
  org.apache.spi.OptimusPrime
  org.apache.spi.Bumblebee
  
  4.加载实现类
  ServiceLoader<Robot> serviceLoader = ServiceLoader.load(Robot.class);
          System.out.println("Java SPI");
          serviceLoader.forEach(Robot::sayHello)；
  ```

## Dubbo SPI

- 自定义了ServiceLoader
- 配置文件稍有不同

## ExtensionLoader源码解析

- 获取extensionLoader

  ```
  ExtensionLoader<Robot> extensionLoader = ExtensionLoader.getExtensionLoader(Robot.class);
  
  loader是被缓存在了内存中，使用ConcurrentHashMap存储
  ```

- 加载实现类

  ```
  Robot optimusPrime = extensionLoader.getExtension("optimusPrime");
  
  实例对象被包装在holder里，holder被缓存在内存中，使用ConcurrentHashMap存储，不理解的是map已经是线程安全的了，完全可以把实例放在map里，不需要对接下来的holder再进行加锁。
  
  获取实例步骤：
  1.通过getExtensionClasses获取所有的拓展类
  2.通过反射创建拓展对象
  3.向拓展对象中注入依赖
  4.将拓展对象包裹在相应的Wrapper对象中
  ```

- 获取所有的拓展类

  ```
  private Map<String, Class<?>> getExtensionClasses() {
      // 从缓存中获取已加载的拓展类
      Map<String, Class<?>> classes = cachedClasses.get();
      // 双重检查
      if (classes == null) {
          synchronized (cachedClasses) {
              classes = cachedClasses.get();
              if (classes == null) {
                  // 加载拓展类
                  classes = loadExtensionClasses();
                  cachedClasses.set(classes);
              }
          }
      }
      return classes;
  }
  ```

- dubbo ioc

  ```
  dubbo的依赖注入是通过setter方法实现。通过反射获取实例的set方法，通过ObjectFactory获取依赖对象，通过反射注入依赖。
  
  private T injectExtension(T instance) {
      try {
          if (objectFactory != null) {
              // 遍历目标类的所有方法
              for (Method method : instance.getClass().getMethods()) {
                  // 检测方法是否以 set 开头，且方法仅有一个参数，且方法访问级别为 public
                  if (method.getName().startsWith("set")
                      && method.getParameterTypes().length == 1
                      && Modifier.isPublic(method.getModifiers())) {
                      // 获取 setter 方法参数类型
                      Class<?> pt = method.getParameterTypes()[0];
                      try {
                          // 获取属性名，比如 setName 方法对应属性名 name
                          String property = method.getName().length() > 3 ? 
                              method.getName().substring(3, 4).toLowerCase() + 
                              	method.getName().substring(4) : "";
                          // 从 ObjectFactory 中获取依赖对象
                          Object object = objectFactory.getExtension(pt, property);
                          if (object != null) {
                              // 通过反射调用 setter 方法设置依赖
                              method.invoke(instance, object);
                          }
                      } catch (Exception e) {
                          logger.error("fail to inject via method...");
                      }
                  }
              }
          }
      } catch (Exception e) {
          logger.error(e.getMessage(), e);
      }
      return instance;
  }
  ```
