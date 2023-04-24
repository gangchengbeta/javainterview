# RocketMQ路由中心NameServer
---
![img](https://cscgblog-1301638685.cos.ap-chengdu.myqcloud.com//md/img2843224-a7bdf308a8d3739b.png)
> NameServer 是 RocketMQ 消息队列的状态服务器（**服务发现功能**），集群中的各个服务都需要通过 NameServer 来了解集群中各个服务的状态。
> NameServer 中维护着 Producer 集群、Broker 集群、 Consumer 集群的服务状态。通过定时发送心跳数据包进行维护更新各个服务的状态。
> NameServer 可以部署多个，多个 NameServer 互相独立，不会交换消息。Producer、Broker、Consumer 启动的时候都需要指定多个 NameServer，各个服务的信息会同时注册到多个 NameServer 上，从而能到达高可用。



## NameServer启动流程

1. 解析配置文件,填充配置类对象NameServerConfig/NettyServerConfig 属性值

   ```java
           namesrvConfig = new NamesrvConfig();
           nettyServerConfig = new NettyServerConfig();
           nettyClientConfig = new NettyClientConfig();
           nettyServerConfig.setListenPort(9876);
           if (commandLine.hasOption('c')) {
               String file = commandLine.getOptionValue('c');
               if (file != null) {
                   InputStream in = new BufferedInputStream(Files.newInputStream(Paths.get(file)));
                   properties = new Properties();
                   properties.load(in);
                   MixAll.properties2Object(properties, namesrvConfig);
                   MixAll.properties2Object(properties, nettyServerConfig);
                   MixAll.properties2Object(properties, nettyClientConfig);
                   if (namesrvConfig.isEnableControllerInNamesrv()) {
                       controllerConfig = new ControllerConfig();
                       MixAll.properties2Object(properties, controllerConfig);
                   }
                   namesrvConfig.setConfigStorePath(file);
   
                   System.out.printf("load config properties file OK, %s%n", file);
                   in.close();
               }
           }
   
           MixAll.properties2Object(ServerUtil.commandLine2Properties(commandLine), namesrvConfig);
   ```

   NameServerConfig 属性:

   ![image-20230227143256140](https://cscgblog-1301638685.cos.ap-chengdu.myqcloud.com//md/imgimage-20230227143256140.png)

   

2. 根据启动属性创建NamesrvController实例,并初始化该实例(核心控制器)

   ![image-20230227143919497](https://cscgblog-1301638685.cos.ap-chengdu.myqcloud.com//md/imgimage-20230227143919497.png)

   3. 注册 JVM 狗子函数并启动服务器,以便监听Broker/消息生成者的网络请求

## NameServer的路由元信息小结

![image-20230224210105068](https://cscgblog-1301638685.cos.ap-chengdu.myqcloud.com//md/imgimage-20230224210105068.png)

### topicQueueTable Topic 消息队列路由信息，消息发送时根据路由表进行负载均衡

```java
    private final Map<String/* topic */, Map<String, QueueData>> topicQueueTable;
```

对于<code>QueueData</code> 

```java
public class QueueData implements Comparable<QueueData> {
    private String brokerName; 
    private int readQueueNums;
    private int writeQueueNums;
    private int perm;// 读写权限
    private int topicSysFlag; 
}
```

### brokerAddrTable  Broker 基础信息   包含 brokerName 、 所属集群名称 、 主备 Broker地址 

```java
    private final Map<String/* brokerName */, BrokerData> brokerAddrTable;
```

对于 <code>BrokerData</code>

```java
public class BrokerData implements Comparable<BrokerData> {
    private String cluster;
    private String brokerName;

    /**
     存储当前代理复制集群的所有单个实例的容器。 key是brokerId，value是单个broker实例的地址
     */
    private HashMap<Long /* brokerID */, String> brokerAddrs;
    private String zoneName;
    private final Random random = new Random();
    /**
     * Enable acting master or not, used for old version HA adaption,
     */
    private boolean enableActingMaster = false;
}

```

### clusterAddrTable Broker 集群信息存储集群中所有 Broker 名称 

```java
    private final Map<String/* clusterName */, Set<String/* brokerName */>> clusterAddrTable;
```

### brokerLiveTable Broker 状态信息   NameServer 每次收到心跳包时会替换该信息

```java
private final Map<BrokerAddrInfo/* brokerAddr */, BrokerLiveInfo> brokerLiveTable;
```

对于 <code>BrokerLiveInfo</code>

```java
class BrokerLiveInfo {
    private long lastUpdateTimestamp;
    private long heartbeatTimeoutMillis;
    private DataVersion dataVersion;
    private Channel channel;
    private String haServerAddr;
}
```

### filterServerTable  Broker 上的 FilterServer 列表，用于类模式消息过滤

```java
    private final Map<BrokerAddrInfo/* brokerAddr */, List<String>/* Filter Server */> filterServerTable;
```

## NameServer 路由注册流程

RocketMQ路由注册是通过Broker和NameServer的心跳功能实现的。Broker启动时向集群中所有NameServer发送心跳语句，每隔30s向集群中所有的NameServer发送心跳包。

## NameServer 路由删除流程

## NameServer 路由发现流程

# RocketMQ消息发送



---

## 消息结构

## Producer 启动流程

## 消息发送流程

## 批量消息发送

