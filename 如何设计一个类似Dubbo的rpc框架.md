## 思路

- 注册中心，注册服务，保留服务的信息，用zk来做。
- 发起请求，基于动态代理。
- 找哪个机器发起请求，得有个负载均衡算法。
- 找到机器后就可以发起请求了，用netty，nio的方式发送。发送格式，用hessian序列化。
- 服务器那边一样，针对服务生产一个动态代理，监听某个网络端口，代理本地的服务代码，接收到请求的时候，就调用对应的服务代码。