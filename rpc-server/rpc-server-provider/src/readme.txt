一、项目结构说明：
rpc-server-api :服务提供者API
rpc-server-provider:服务提供者实现

二、项目背景：
  该项目是使用zookeeper的特性模拟了RPC远程接口调用注册中心的全过程，了解使用zookeeper作为注册中心的使用。

三、服务端实现流程：
  1. 使用ApplicationContextAware、InitializingBean 进行二次开发，实现了初始化完成GPServer时，做2件事：启动zkserver服务端并且注册节点、启动netty服务端。
  2. 使用AnnotationConfigApplicationContext进行加载SpringConfig配置文件,注入一个GpRpcServer类。
  3. GpRpcServer做两件事：
     setApplicationContext:
     1) 获取所有标注了@RpcService注解的实现类。
     2) 获取该接口的@RpcService的value、version值，组装ServiceName: com.gupaoedu.vip.IHelloService-v1.0。
     3) 将ServiceName和该接口实现类存入Map。
     4) 向zk进行注册。(使用ServiceName/本机IP:port) 形式。
        4.1) 初始化zk连接。
        4.2) 创建节点。(命名空间/ServiceName/IP:port) 。
          注意：此处使用curator4.0实现zk的操作，不需要设置节点的值，只需要设置节点的路径，节点的值默认是本机IP。
     afterPropertiesSet：
       1) 启动一个NettyServer，绑定端口号。
       2) 创建一个接口实现SimpleChannelInboundHandler，用于处理客户端的请求。
四、客户端实现流程：
   1. 使用AnnotationConfigApplicationContext 进行二次开发，加载配置文件SpringConfig类。
   2. 在SpringConfig中，实际上是生成一个接口的代理类，将该代理类注入Spring中。
   3. 调用代理类的invoke方法，所有的逻辑都在该方法中完成。
      1) 初始化zk连接，相当于获取注册中心。
      2) 根据接口名称从注册中心获取它的所有子节点。(如：10.12.10.32:8081,10.12.10.32:8082,10.12.10.32:8083)
      3) 通过负载均衡策略获取其中的一个子节点。(也就是其中的一个服务器)
      4) 启动一个Netty客户端，根据获取的节点和端口号发起一个调用，最终返回结果 。





