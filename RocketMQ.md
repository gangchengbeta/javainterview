### Q1. Rocket如何保证消息队列系统的高可用性?

### Q2. RocketMQ 如何避免消息的重复消费?

### Q3. RocketMQ 如何避免消息的丢失?

### Q4. RocketMQ 如何保证消息的按序到达?

### Q5. RocketMQ 如何实现消息间的事务性?





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

### RocketMQ如何避免消息重复消费

---

RocketMQ 通过消费者端维护消息消费的偏移量（offset）来避免消息重复消费的问题。

每个消费者在消费消息时，会将其消费的最新偏移量提交到 RocketMQ 服务器端。RocketMQ 会将这些偏移量保存下来，并在下次消费时将消息的偏移量与保存的偏移量进行比对，如果消息的偏移量大于保存的偏移量，则表示该消息还未被消费过，可以进行消费。如果消息的偏移量小于或等于保存的偏移量，则表示该消息已经被消费过，需要避免重复消费。

>
>RocketMQ 还提供了多种消息消费模式，可以根据业务需求选择不同的消费模式来避免消息重复消费的问题。比如，顺序消费模式（Orderly Consumption Mode）可以保证消息的顺序性，并且避免消息的重复消费；广播消费模式（Broadcasting Consumption Mode）可以让多个消费者同时消费同一批消息，但每个消息只会被消费一次，避免消息的重复消费。

### RocketMQ 如何保证消息按序到达

---

RocketMQ 通过分片（Sharding）和顺序消息的设计来保证消息按序到达。具体来说，RocketMQ 将一个 Topic 中的所有消息按照顺序分配到多个分片（Shard）中，每个分片只负责处理其中一部分消息，这样可以提高消息的并发处理能力。同时，每个分片内部保持消息的严格顺序，保证消息按照生产者发送的顺序进行投递。因此，在消费者端进行消费时，需要保证消费者能够按照相同的顺序从每个分片中消费消息，才能保证整个 Topic 中消息的顺序性。

>为了保证消息按照相同的顺序从每个分片中消费，RocketMQ 提供了两种顺序消息的消费模式：
>
>1. 顺序消费模式（Orderly Consumption Mode）：该模式下，一个消费者只会从一个分片中消费消息，消费过程中不会同时处理来自其他分片的消息。这种方式保证了消费者能够按照顺序消费每个分片中的消息，从而保证整个 Topic 中消息的顺序性。
>2. 广播消费模式（Broadcasting Consumption Mode）：该模式下，多个消费者同时消费同一批消息，但每个消息只会被消费一次，避免消息的重复消费。这种方式适合于需要多个消费者同时处理相同消息的场景，但不保证消息的顺序性。
