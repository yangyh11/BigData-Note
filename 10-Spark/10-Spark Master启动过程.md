**Spark集群启动过程**

每台节点都会启动对应的Netty通信环境，叫做RpcEnv通信环境。

每个角色启动之前首先先NettyRpcEnv环境中注册对应的Endpoint，然后启动。

角色包括：Master、Worker、Driver、Executor

**Master角色启动过程**

