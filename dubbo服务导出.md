## ServiceBean

- dubbo导出服务的起点是在application的listener实现的

  ```
  	public void onApplicationEvent(ApplicationEvent event) {
  		if (ContextRefreshedEvent.class.getName().equals(event.getClass().getName())) {
  			if (isDelay() && ! isExported() && ! isUnexported()) {
  				if (logger.isInfoEnabled()) {
  					logger.info("The service ready on spring started. service: " + getInterface());
  				}
  				export();
  			}
  		}
  	}
  ```

- 如何理解ServiceBean

  ```
  ServiceBean是Dubbo与Spring框架进行整合的关键，可以看做两个框架之间的桥梁。
  ```

- 如何本地启用服务调试，不暴露出去给别人用

  ```
  <dubbo:provider export="false" />
  ```

- 多协议，多注册中心的支持

  ```
  private void doExportUrls() {
      // 加载注册中心链接
      List<URL> registryURLs = loadRegistries(true);
      // 遍历 protocols，并在每个协议下导出服务
      for (ProtocolConfig protocolConfig : protocols) {
          doExportUrlsFor1Protocol(protocolConfig, registryURLs);
      }
  }
  ```

- URL

  ```
  dubbo的自定义URL，是dubbo配置的载体。
  ```

- 导出服务

  ```
  scope = none，不导出服务
  scope != remote，导出到本地
  scope != local，导出到远程
  
  Invoker<?> invoker = proxyFactory.getInvoker(ref, (Class) interfaceClass, url);
                      Exporter<?> exporter = protocol.export(invoker);
                      exporters.add(exporter);
  ```

- invoker是什么

  ```
  
  ```
