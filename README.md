---
RPC框架 Part3 - 基于Netty与ZooKeeper的分布式RPC实现
项目简介
本项目是一个轻量级的RPC（远程过程调用）框架，采用 Netty 作为网络通信层，ZooKeeper 作为服务注册与发现中心，实现了分布式环境下的服务调用。
技术栈
Netty 4.1.51
Apache Curator 5.1.0
Lombok
SLF4J + Log4j
核心特性
1. 服务注册与发现
- 服务端：启动时将服务地址注册到ZooKeeper（临时节点）
- 客户端：从ZooKeeper动态获取服务提供者地址，无需硬编码
- 服务下线时自动移除节点
2. 动态代理
- 使用JDK动态代理（java.lang.reflect.Proxy）
- 客户端调用接口方法时，自动封装为RPC请求并发送
3. 高性能通信
- 基于Netty的NIO非阻塞通信
- 支持长连接，避免频繁建立/断开连接开销
架构设计
┌─────────────┐      ┌─────────────┐       ┌─────────────┐
│   Client    │      │  ZooKeeper  │       │   Server    │
├─────────────┤      ├─────────────┤       ├─────────────┤
│ ClientProxy │────▶│ 服务发现    │       │ 服务注册    │
│   (代理)    │      │ 获取地址    │       │ 注册地址    │
├─────────────┤      └─────────────┘       ├─────────────┤
│ NettyClient │─────────────────────────▶ │NettyServer │
│ (网络传输)  │     RpcRequest/RpcResponse │ (处理请求) │
└─────────────┘                            └─────────────┘
目录结构
src/main/java/
├── part3/
│   ├── common/                    # 公共模块
│   │   ├── Message/               # 消息体定义
│   │   │   ├── RpcRequest.java   # 请求对象
│   │   │   └── RpcResponse.java  # 响应对象
│   │   ├── pojo/                  # 实体类
│   │   │   └── User.java
│   │   └── service/               # 服务接口
│   │       ├── UserService.java
│   │       └── Impl/UserServiceImpl.java
│   │
│   ├── Client/                    # 客户端
│   │   ├── proxy/                 # 动态代理
│   │   │   └── ClientProxy.java
│   │   ├── rpcClient/             # 网络客户端
│   │   │   ├── RpcClient.java
│   │   │   └── impl/
│   │   │       ├── NettyRpcClient.java
│   │   │       └── SimpleSocketRpcCilent.java
│   │   ├── serviceCenter/         # 服务发现
│   │   │   ├── ServiceCenter.java
│   │   │   └── ZKServiceCenter.java
│   │   ├── netty/                 # Netty客户端相关
│   │   └── TestClient.java        # 客户端入口
│   │
│   └── Server/                    # 服务端
│       ├── server/                # 服务器实现
│       │   ├── RpcServer.java
│       │   └── impl/
│       │       ├── SimpleRPCRPCServer.java
│       │       └── NettyRPCRPCServer.java
│       ├── provider/              # 服务提供者
│       │   └── ServiceProvider.java
│       ├── serviceRegister/       # 服务注册
│       │   ├── ServiceRegister.java
│       │   └── impl/ZKServiceRegister.java
│       ├── netty/                 # Netty服务端相关
│       └── TestServer.java        # 服务端入口
│
└── part2/common/                  # Part2公共代码（共享）
    ├── Message/
    ├── pojo/
    └── service/
快速开始
前置条件
- JDK 8+
- Maven 3.x
- ZooKeeper 服务运行在 127.0.0.1:2181
启动步骤
1. 启动ZooKeeper
# 确保ZooKeeper已启动
./zkServer.sh start  # Linux
# Windows下启动 ZooKeeper
2. 启动服务端
mvn compile
# 运行 part3.Server.TestServer
或直接在IDE中运行 part3.Server.TestServer#main
3. 启动客户端
# 运行 part3.Client.TestClient
或在IDE中运行 part3.Client.TestClient#main
运行效果
客户端输出：
从服务端得到的user=User(id=1, userName=张三, sex=true)
向服务端插入user的id100
核心代码解析
RpcRequest 请求格式
@Data
@Builder
public class RpcRequest implements Serializable {
    private String interfaceName;  // 服务接口名
    private String methodName;      // 方法名
    private Object[] params;        // 参数列表
    private Class<?>[] paramsType;  // 参数类型
}
服务注册 (ZKServiceRegister)
- 服务名作为永久节点
- 服务地址（IP:Port）作为临时节点（EPHEMERAL）
- 服务器下线时，临时节点自动删除
服务发现 (ZKServiceCenter)
- 根据服务名从ZooKeeper获取可用服务地址列表
- 默认使用第一个地址（可扩展负载均衡）
扩展方向
- [ ] 添加负载均衡策略（轮询、随机、权重）
- [ ] 支持多种序列化方式（Hessian、Protobuf）
- [ ] 实现服务熔断与降级
- [ ] 添加注册中心高可用支持
许可证
MIT License
