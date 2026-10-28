---
type: knowledge
status: active
kind: method
review_after: 2025-08-26
last_reviewed: 2024-08-26
---
## 配置文件以及读取配置文件相关类

### 现有配置

首先查看现有的配置文件：

```yaml
seata:
  config:
    # support: nacos, consul, apollo, zk, etcd3
    type: zk
    #....
    zk:
      server-addr: 127.0.0.1:2181
      session-timeout: 6000
      connect-timeout: 2000
      username:
      password:
      node-path: /seata/seata.properties
   
  registry:
    # support: nacos 、 eureka 、 redis 、 zk  、 consul 、 etcd3 、 sofa
    type: redis
    preferred-networks: 30.240.*
    # ...
    redis:
      server-addr: localhost:6379
      db: 0
      password:
      cluster: default
      timeout: 0
```

在 `SeataPropertiesLoader`的`initialize`方法之中，将会使用以下语句设置配置文件中的内容：

```java
 // Load by priority
 System.setProperty("sessionMode", StoreConfig.getSessionMode().getName());
 System.setProperty("lockMode", StoreConfig.getLockMode().getName());
```

此时会调用`StoreConfig`中的变量：

```java
private static final Configuration CONFIGURATION = ConfigurationFactory.getInstance();
```

然后接下来将会进行初始化：

```java
public static Configuration getInstance() {
        if (instance == null) {
            synchronized (Configuration.class) {
                if (instance == null) {
                    instance = buildConfiguration();
                }
            }
        }
        return instance;
    }
```

接下来将会调用`buildConfiguration`这个方法：

```java
private static Configuration buildConfiguration() {
        String configTypeName = getConfigType();
        Configuration configuration = ORIGIN_FILE_INSTANCE;
        Configuration extConfiguration = getSpringConfiguration();
        if (null == extConfiguration) {
            Configuration springConfiguration = getNonSpringConfiguration(configTypeName);
```

从getConfigType获取配置中对应的type类型，然后使用`getSpringConfiguration`判断是为Spring已经存在的加载类型，如果说不是，则调用`getNonSpringConfiguration`方法，加载配置文件中对应的内容：

```java
private static Configuration getNonSpringConfiguration(String configTypeName) {
...
 Configuration configuration = EnhancedServiceLoader.load(ConfigurationProvider.class,
                Objects.requireNonNull(configTypeName), false).provide();
```

然后这个时候，将会调用type类型中对应的`provide`类进行加载（如type为zk，此时则调用`ZookeeperConfigurationProvider`中的`provide`方法进行加载， 读取配置文件中的对应字段，进行加载）。

这个时候相关的配置文件已经加载完成。从上面来看，首先判断类型，然后进行**初始化**，然后在进行加载。

在进行初始化的时候，使用的是单例进行创建，无法支持多个字段，因此这里需要进行更改

### 预设修改

因此，从上述方法来看，首先对应配置文件修改如下：

```yaml
seata:
  config:
    # support: nacos, consul, apollo, zk, etcd3
    type: zk,nacos,...
    #....
    zk:
      server-addr: 127.0.0.1:2181
      session-timeout: 6000
      connect-timeout: 2000
      username:
      password:
      node-path: /seata/seata.properties
   
  registry:
    # support: nacos 、 eureka 、 redis 、 zk  、 consul 、 etcd3 、 sofa
    type: redis,nacos,....
    preferred-networks: 30.240.*
    # ...
    redis:
      server-addr: localhost:6379
      db: 0
      password:
      cluster: default
      timeout: 0
```

设想多个注册中心ID用逗号分隔，然后再读取配置文件响应的属性时，进行字符串切分，然后进行读取。

经过以上分析，在进行初始化时，主要是调用`StoreCofig`进行初始化，里面会调用`ConfigurationFactory`的`getInstance`进行创建，这个时候相当于默认`CONFIGURATION`只有一个，这个需要进行修改，不再是单例进行创建，而是通过集合的形式，分别对每个类型进行初始化，可以修改为：

```java
private static final Configuration List<CONFIGURATION> CONFIGURATIONS;

 private static class MultiRegistryFactoryHolder {
        private static final List<CONFIGURATION> CONFIGURATIONS = buildStoreConfigs();
    }
```

然后将读取对应配置的个数，进行初始化。个数判断主要采用类型获取，判断里面对应字段的个数，然后初始化对应的内容。

`registry`相关类中的`RegistryFactory`已经指定好对应的初始化内容，其中已经实现了多端实例化。

## Client端对应的多实例化

### 现有客户端配置

还是从配置文件进行出发，判断不同字段是在哪里进行创建，判定是否为单例创建：

|                                                    |                                               |                                                              |
| -------------------------------------------------- | --------------------------------------------- | ------------------------------------------------------------ |
| seata.enabled                                      | 是否开启 spring-boot 自动装配                 | true、false,(SSBS)专有配置，默认 true（附录 4）              |
| seata.enableAutoDataSourceProxy=true               | 是否开启数据源自动代理                        | true、false,seata-spring-boot-starter(SSBS)专有配置,SSBS 默认会开启数据源自动代理,可通过该配置项关闭. |
| seata.useJdkProxy=false                            | 是否使用 JDK 代理作为数据源自动代理的实现方式 | true、false,(SSBS)专有配置,默认 false,采用 CGLIB 作为数据源自动代理的实现方式 |
| transport.enableClientBatchSendRequest             | 客户端事务消息请求是否批量合并发送            | 默认 true，false 单条发送                                    |
| client.log.exceptionRate                           | 日志异常输出概率                              | 默认 100，目前用于 undo 回滚失败时异常堆栈输出，百分之一的概率输出，回滚失败基本是脏数据，无需输出堆栈占用硬盘空间 |
| service.vgroupMapping.my_test_tx_group             | 事务群组（附录 1）                            | my_test_tx_group 为分组，配置项值为 TC 集群名                |
| service.default.grouplist                          | TC 服务列表（附录 2）                         | 仅注册中心为 file 时使用                                     |
| service.disableGlobalTransaction                   | 全局事务开关                                  | 默认 false。false 为开启，true 为关闭                        |
| client.tm.degradeCheck                             | 降级开关                                      | 默认 false。业务侧根据连续错误数自动降级不走 seata 事务(详细介绍请阅读附录 6) |
| client.tm.degradeCheckAllowTimes                   | 升降级达标阈值                                | 默认 10                                                      |
| client.tm.degradeCheckPeriod                       | 服务自检周期                                  | 默认 2000,单位 ms.每 2 秒进行一次服务自检,来决定             |
| client.rm.reportSuccessEnable                      | 是否上报一阶段成功                            | true、false，从 1.1.0 版本开始,默认 false.true 用于保持分支事务生命周期记录完整，false 可提高不少性能 |
| client.rm.asyncCommitBufferLimit                   | 异步提交缓存队列长度                          | 默认 10000。 二阶段提交成功，RM 异步清理 undo 队列           |
| client.rm.lock.retryInterval                       | 校验或占用全局锁重试间隔                      | 默认 10，单位毫秒                                            |
| client.rm.lock.retryTimes                          | 校验或占用全局锁重试次数                      | 默认 30                                                      |
| client.rm.lock.retryPolicyBranchRollbackOnConflict | 分支事务与其它全局回滚事务冲突时锁策略        | 默认 true，优先释放本地锁让回滚成功                          |
| client.rm.reportRetryCount                         | 一阶段结果上报 TC 重试次数                    | 默认 5 次                                                    |
| client.rm.tableMetaCheckEnable                     | 自动刷新缓存中的表结构                        | 默认 false                                                   |
| client.rm.tableMetaCheckerInterval                 | 定时刷新缓存中表结构间隔时间                  | 默认 60 秒                                                   |
| client.rm.sagaBranchRegisterEnable                 | 是否开启 saga 分支注册                        | Saga 模式中分支状态存储在状态机本地数据库中，可通过状态机进行提交或回滚，为提高性能可考虑不用向 TC 注册 Saga 分支，但需考虑状态机的可用性，默认 false |
| client.rm.sagaJsonParser                           | saga 模式中数据序列化方式                     | 默认 fastjson,可选 jackson                                   |
| client.rm.tccActionInterceptorOrder                | tcc 拦截器顺序                                | 默认 Ordered.HIGHEST_PRECEDENCE + 1000，保证拦截器在本地事务拦截器之前执行，也可自定义 tcc 和业务开发的拦截器执行顺序 |
| client.tm.commitRetryCount                         | 一阶段全局提交结果上报 TC 重试次数            | 默认 1 次，建议大于 1                                        |
| client.tm.rollbackRetryCount                       | 一阶段全局回滚结果上报 TC 重试次数            | 默认 1 次，建议大于 1                                        |
| client.tm.defaultGlobalTransactionTimeout          | 全局事务超时时间                              | 默认 60 秒，TM 检测到分支事务超时或 TC 检测到 TM 未做二阶段上报超时后，发起对分支事务的回滚 |
| client.tm.interceptorOrder                         | TM 全局事务拦截器顺序                         | 默认 Ordered.HIGHEST_PRECEDENCE + 1000，保证拦截器在本地事务拦截器之前执行，也可自定义全局事务和业务开发的拦截器执行顺序 |
| client.undo.dataValidation                         | 二阶段回滚镜像校验                            | 默认 true 开启，false 关闭                                   |
| client.undo.logSerialization                       | undo 序列化方式                               | 默认 jackson                                                 |
| client.undo.logTable                               | 自定义 undo 表名                              | 默认 undo_log                                                |
| client.undo.onlyCareUpdateColumns                  | 只生成被更新列的镜像                          | 默认 true                                                    |
| client.undo.compress.enable                        | undo log 压缩开关                             | 默认 true                                                    |
| client.undo.compress.type                          | undo log 压缩算法                             | 默认 zip,可选 NONE(不压缩)、GZIP、ZIP、SEVENZ、BZIP2、LZ4、DEFLATER、ZSTD |
| client.undo.compress.threshold                     | undo log 压缩阈值                             | 默认值 64k，压缩开关开启且 undo log 大小超过阈值时才进行压缩 |
| client.rm.sqlParserType                            | sql 解析类型                                  | 默认 druid,可选 antlr                                        |

### RM相关

先看client的rm，在进行初始化对应的方法是` org.apache.seata.rm`下面的`AbstractResourceManager`，核心如下：

```java
public abstract class AbstractResourceManager implements ResourceManager {
	 protected static final Logger LOGGER = LoggerFactory.getLogger(AbstractResourceManager.class);

    private static final Configuration CONFIG = ConfigurationFactory.getInstance();
    
    private static int appDataErrSize = CONFIG.getInt(ConfigurationKeys.RM_APPLICATION_DATA_SIZE_LIMIT,
            DefaultValues.DEFAULT_APPLICATION_DATA_SIZE_LIMIT);

    private static boolean throwDataSizeExp = CONFIG.getBoolean(ConfigurationKeys.RM_APPLICATION_DATA_SIZE_CHECK, false);

     @Override
    public Long branchRegister(BranchType branchType, String resourceId, String clientId, String xid, String applicationData, String lockKeys) throws TransactionException {
        try {
            StringUtils.checkDataSize(applicationData, "applicationData", appDataErrSize, throwDataSizeExp);
			...
        }
    }
}
```

此类主要的作用为资源管理抽象、事务资源的注册与管理、事务分支管理、协议支持等作用，若现在存在多个相同的rm哦欸之，则不能进行单例创建，应该创建List集合，以字符串拼接的形式进行读取。

在RM的Client端中，主要与RMClient有关，此类的具体代码如下：

```java
public class RMClient {

    /**
     * Init.
     *
     * @param applicationId           the application id
     * @param transactionServiceGroup the transaction service group
     */
    public static void init(String applicationId, String transactionServiceGroup) {
        RmNettyRemotingClient rmNettyRemotingClient = RmNettyRemotingClient.getInstance(applicationId, transactionServiceGroup);
        //
        rmNettyRemotingClient.setResourceManager(DefaultResourceManager.get());
        rmNettyRemotingClient.setTransactionMessageHandler(DefaultRMHandler.get());
        rmNettyRemotingClient.init();
    }

}
```

现在抽象Client端相当于抽象其中的资源管理的内容，原本仅仅是单例创建，现在不再是单例，因此先查看`DefaultResourceManager`这个类

```java
/**
 * default resource manager, adapt all resource managers
 *
 */
public class DefaultResourceManager implements ResourceManager {

    /**
     * all resource managers
     */
    protected static Map<BranchType, ResourceManager> resourceManagers
        = new ConcurrentHashMap<>();

    private DefaultResourceManager() {
        initResourceManagers();
    }

    /**
     * Get resource manager.
     *
     * @return the resource manager
     */
    public static DefaultResourceManager get() {
        return SingletonHolder.INSTANCE;
    }
    
     protected void initResourceManagers() {
        //init all resource managers
        List<ResourceManager> allResourceManagers = EnhancedServiceLoader.loadAll(ResourceManager.class);
        if (CollectionUtils.isNotEmpty(allResourceManagers)) {
            for (ResourceManager rm : allResourceManagers) {
                resourceManagers.put(rm.getBranchType(), rm);
            }
        }
    }
    
    private static class SingletonHolder {
        private static DefaultResourceManager INSTANCE = new DefaultResourceManager();
    }
```

但实际在`get`获取的时候，采用是单例创建`DefaultResourceManager`对象，然后调用`initResourceManagers`方法，初始化所有的ResourceManager（包括AbstractResourceManager、DefaultRMHandler等内容），此时仍然仅仅是针对一个进行创建，不支持多实例创建，因此此处需要进行更改。

### TM相关

tm与rm大同小异，还是从配置文件出发：

```java
public class DefaultGlobalTransaction implements GlobalTransaction {

    private static final Logger LOGGER = LoggerFactory.getLogger(DefaultGlobalTransaction.class);

    private static final int DEFAULT_GLOBAL_TX_TIMEOUT = 60000;

    private static final String DEFAULT_GLOBAL_TX_NAME = "default";

    private TransactionManager transactionManager;

    private String xid;

    private GlobalStatus status;

    private GlobalTransactionRole role;

    /**
     * Used to calculate the timeout
     *
     * @see System#currentTimeMillis();
     */
    private long createTime;

    private static final int COMMIT_RETRY_COUNT = ConfigurationFactory.getInstance().getInt(
        ConfigurationKeys.CLIENT_TM_COMMIT_RETRY_COUNT, DEFAULT_TM_COMMIT_RETRY_COUNT);

    private static final int ROLLBACK_RETRY_COUNT = ConfigurationFactory.getInstance().getInt(
        ConfigurationKeys.CLIENT_TM_ROLLBACK_RETRY_COUNT, DEFAULT_TM_ROLLBACK_RETRY_COUNT);
    
     /**
     * Instantiates a new Default global transaction.
     */
    DefaultGlobalTransaction() {
        this(null, GlobalStatus.UnKnown, GlobalTransactionRole.Launcher);
    }
    
    public DefaultGlobalTransaction(String xid, GlobalStatus status, GlobalTransactionRole role) {
        this.transactionManager = TransactionManagerHolder.get();
        this.xid = xid;
        this.status = status;
        this.role = role;
    }
    
    //...
        
    /**
     * Get transaction manager.
     *
     * @return the transaction manager
     */
    public static TransactionManager get() {
        if (SingletonHolder.INSTANCE == null) {
            throw new ShouldNeverHappenException("TransactionManager is NOT ready!");
        }
        return SingletonHolder.INSTANCE;
    }
    
     private static class SingletonHolder {

        private static TransactionManager INSTANCE = null;

        static {
            try {
                INSTANCE = EnhancedServiceLoader.load(TransactionManager.class);
                LOGGER.info("TransactionManager Singleton {}", INSTANCE);
            } catch (Throwable anyEx) {
                LOGGER.error("Failed to load TransactionManager Singleton! ", anyEx);
            }
        }
    }
```

还是同样的，看到`TransactionManager`中`get()`方法时，仍然是单例创建，无法适配多个TM的内容，因此这里需要进行更改。

### undo相关

#### 解析相关

还是从日志进行出发，对应类如下

```java
public abstract class AbstractUndoLogManager implements UndoLogManager {

    private static final Logger LOGGER = LoggerFactory.getLogger(AbstractUndoLogManager.class);

    protected enum State {
        /**
         * This state can be properly rolled back by services
         */
        Normal(0),
        /**
         * This state prevents the branch transaction from inserting undo_log after the global transaction is rolled
         * back.
         */
        GlobalFinished(1);

        private int value;

        State(int value) {
            this.value = value;
        }

        public int getValue() {
            return value;
        }
    }

    protected static final String UNDO_LOG_TABLE_NAME = ConfigurationFactory.getInstance().getConfig(
        ConfigurationKeys.TRANSACTION_UNDO_LOG_TABLE, DEFAULT_TRANSACTION_UNDO_LOG_TABLE);

    private static final String CHECK_UNDO_LOG_TABLE_EXIST_SQL = "SELECT 1 FROM " + UNDO_LOG_TABLE_NAME + " LIMIT 1";

    protected static final String SELECT_UNDO_LOG_SQL = "SELECT * FROM " + UNDO_LOG_TABLE_NAME + " WHERE "
        + ClientTableColumnsName.UNDO_LOG_BRANCH_XID + " = ? AND " + ClientTableColumnsName.UNDO_LOG_XID
        + " = ? FOR UPDATE";

    protected static final String DELETE_UNDO_LOG_SQL = "DELETE FROM " + UNDO_LOG_TABLE_NAME + " WHERE "
        + ClientTableColumnsName.UNDO_LOG_BRANCH_XID + " = ? AND " + ClientTableColumnsName.UNDO_LOG_XID + " = ?";

    protected static final boolean ROLLBACK_INFO_COMPRESS_ENABLE = ConfigurationFactory.getInstance().getBoolean(
        ConfigurationKeys.CLIENT_UNDO_COMPRESS_ENABLE, DEFAULT_CLIENT_UNDO_COMPRESS_ENABLE);

    protected static final CompressorType ROLLBACK_INFO_COMPRESS_TYPE = CompressorType.getByName(ConfigurationFactory.getInstance().getConfig(
        ConfigurationKeys.CLIENT_UNDO_COMPRESS_TYPE, DEFAULT_CLIENT_UNDO_COMPRESS_TYPE));

    protected static final long ROLLBACK_INFO_COMPRESS_THRESHOLD = SizeUtil.size2Long(ConfigurationFactory.getInstance().getConfig(
            ConfigurationKeys.CLIENT_UNDO_COMPRESS_THRESHOLD, DEFAULT_CLIENT_UNDO_COMPRESS_THRESHOLD));
    
    //...
    
    @Override
    public void flushUndoLogs(ConnectionProxy cp) throws SQLException {
        ConnectionContext connectionContext = cp.getContext();
        if (!connectionContext.hasUndoLog()) {
            return;
        }

        String xid = connectionContext.getXid();
        long branchId = connectionContext.getBranchId();

        BranchUndoLog branchUndoLog = new BranchUndoLog();
        branchUndoLog.setXid(xid);
        branchUndoLog.setBranchId(branchId);
        branchUndoLog.setSqlUndoLogs(connectionContext.getUndoItems());

        UndoLogParser parser = UndoLogParserFactory.getInstance();
        byte[] undoLogContent = parser.encode(branchUndoLog);

        if (LOGGER.isDebugEnabled()) {
            LOGGER.debug("Flushing UNDO LOG: {}", new String(undoLogContent, Constants.DEFAULT_CHARSET));
        }

        CompressorType compressorType = CompressorType.NONE;
        if (needCompress(undoLogContent)) {
            compressorType = ROLLBACK_INFO_COMPRESS_TYPE;
            undoLogContent = CompressorFactory.getCompressor(compressorType.getCode()).compress(undoLogContent);
        }

        insertUndoLogWithNormal(xid, branchId, buildContext(parser.getName(), compressorType), undoLogContent, cp.getTargetConnection());
    }
    
```

看到里面的`UndoLogParserFactory`采用单例进行创建，查看对应代码： 

```java
private UndoLogParserFactory() {

    }

    /**
     * {serializerName:UndoLogParser}
     */
    private static final ConcurrentMap<String, UndoLogParser> INSTANCES = new ConcurrentHashMap<>();

    private static class SingletonHolder {
        private static final UndoLogParser INSTANCE = getInstance(UndoLogConstants.DEFAULT_SERIALIZER);
    }

    /**
     * Gets default UndoLogParser instance.
     *
     * @return the instance
     */
    public static UndoLogParser getInstance() {
        return SingletonHolder.INSTANCE;
    }

    /**
     * Gets UndoLogParser by name
     *
     * @param name parser name
     * @return the UndoLogParser
     */
    public static UndoLogParser getInstance(String name) {
        if (StringUtils.equalsIgnoreCase("fst", name)) {
            throw new IllegalArgumentException(
                "Since fst is no longer maintained, this serialization extension has been removed from version 2.0 for security and stability reasons.");
        }
        return CollectionUtils.computeIfAbsent(INSTANCES, name,
            key -> EnhancedServiceLoader.load(UndoLogParser.class, name));
    }
```

可以看到，这个方法在继续宁创建的时候，仅仅采用单例创建，没有办法适应多个Undo相关的操作，因此这部分需要进行抽象化，将里面的方法进行设计。

#### 压缩相关

从官网上面看，实现思路如下：

![压缩](https://user-images.githubusercontent.com/22959373/116281711-8f039900-a7bc-11eb-91f8-82afdbb9f932.png)

可以看到，`UndoLogParserFactory`仅仅是在创建基础的undo log解析类，没有涉及到压缩等属性，因此需要继续查看undo log相关属性，找到相关代码。

```java
protected boolean needCompress(byte[] undoLogContent) {
    // 1. 判断是否开启了undo_log压缩功能(1.4.2默认开启)
    // 2. 判断是否达到了压缩的阈值(默认64k)
    // 如果都满足返回需要对对应的undoLogContent进行压缩
    return ROLLBACK_INFO_COMPRESS_ENABLE 
        && undoLogContent.length > ROLLBACK_INFO_COMPRESS_THRESHOLD;
}

// 如果需要压缩，对undo_log进行压缩
if (needCompress(undoLogContent)) {
    // 获取压缩类型，默认zip
    compressorType = ROLLBACK_INFO_COMPRESS_TYPE;
    // 获取对应的压缩器，并且进行压缩
    undoLogContent = CompressorFactory.getCompressor(compressorType.getCode()).compress(undoLogContent);
}
// else 不需要压缩就不需要做任何操作

protected String buildContext(String serializer, CompressorType compressorType) {
    Map<String, String> map = new HashMap<>();
    map.put(UndoLogConstants.SERIALIZER_KEY, serializer);
    // 保存压缩类型到数据库
    map.put(UndoLogConstants.COMPRESSOR_TYPE_KEY, compressorType.name());
    return CollectionUtils.encodeMap(map);
} 

protected byte[] getRollbackInfo(ResultSet rs) throws SQLException  {
    // 获取保存到数据库的回滚信息的字节数组
    byte[] rollbackInfo = rs.getBytes(ClientTableColumnsName.UNDO_LOG_ROLLBACK_INFO);
    // 获取压缩类型
    // getOrDefault使用默认值CompressorType.NONE来兼容1.4.2之前的版本直接升级1.4.2+
    String rollbackInfoContext = rs.getString(ClientTableColumnsName.UNDO_LOG_CONTEXT);
    Map<String, String> context = CollectionUtils.decodeMap(rollbackInfoContext);
    CompressorType compressorType = CompressorType.getByName(context.getOrDefault(UndoLogConstants.COMPRESSOR_TYPE_KEY,
    CompressorType.NONE.name()));
    // 获取对应的压缩器，并且解压缩
    return CompressorFactory.getCompressor(compressorType.getCode())
        .decompress(rollbackInfo);
}
```

这是与压缩相关的代码，查看`CompressorFactory`

```java
/**
 * the type compressor factory
 */
public class CompressorFactory {

    /**
     * The constant COMPRESSOR_MAP.
     */
    protected static final Map<CompressorType, Compressor> COMPRESSOR_MAP = new ConcurrentHashMap<>();

    static {
        COMPRESSOR_MAP.put(CompressorType.NONE, new NoneCompressor());
    }

    /**
     * Get compressor by code.
     *
     * @param code the code
     * @return the compressor
     */
    public static Compressor getCompressor(byte code) {
        CompressorType type = CompressorType.getByCode(code);
        return CollectionUtils.computeIfAbsent(COMPRESSOR_MAP, type,
            key -> EnhancedServiceLoader.load(Compressor.class, type.name()));
    }

    /**
     * None compressor
     */
    @LoadLevel(name = "NONE")
    public static class NoneCompressor implements Compressor {
        @Override
        public byte[] compress(byte[] bytes) {
            return bytes;
        }

        @Override
        public byte[] decompress(byte[] bytes) {
            return bytes;
        }
    }

}
```

`getCompressor(byte code)` 方法用于根据给定的代码获取相应的压缩器。首先，通过 `CompressorType.getByCode(code)` 方法将字节码转换为 `CompressorType` 枚举。然后，通过 `CollectionUtils.computeIfAbsent` 方法检查 `COMPRESSOR_MAP` 中是否已经存在对应类型的压缩器。如果不存在，则使用 `EnhancedServiceLoader` 动态加载对应类型的压缩器并放入 `COMPRESSOR_MAP` 中。

获得时，现在采用Map作为缓存，避免了重复创建。

但是现在一个接口只有一个实例，没有办法实现多例。

## 小结

经过以上分析，主要需要关注的类有：

1. ResourceManager相关的类（主要包括AbstractDataSourceResourceManager、AbstractResourceManger、DataSourceManager、DeafultResourceManager）等，这个主要是RM在进行加载时使用到的类，这里面现在都是单例进行创建，在进行put、get等操作的时候，也是直接进行拿取。
2. TransactionManager(主要包括DefaultTransactionManager等相关代码)，这些方法里面存在TmNettyRemotingClient单例创建的方法，这里面需要进行改造。
3. UndoLogManager相关类，这个主要涉及到对应的解析、压缩以及序列化，现在为单例进行创建，需要进行改造。

## API设计

基于以上分析，现在讲Manager进行抽象，然后设计出对应的API。

首先先对`EnhancedServiceLoader`进行抽象化，这个类现在仅仅是使用一层缓存，针对一个拓展接口只有一个实例，针对一个扩展实现Class也只有一个扩展点实现实例。

因此在这个方法中，需要将目前使用的`EnhancedServiceLoader`进行抽象，将现在使用的方法进行分类，主要分成以下几类：

1. FRAMEWORK-框架级别，扩展在整个框架级别上应用，当扩展需要影响或提供功能给整个框架中的所有组件和模块时，例如日志系统、依赖注入框架等。
2. APPLICATION-应用级别，扩展在整个应用程序级别上应用，当扩展需要作用于整个应用程序的各个部分，而不仅仅是框架的基础部分时，例如应用程序级别的配置管理、安全机制等。
3. MANAGER-模块级别，这个而主要是针对各个Manager在进行使用时，一些共性的方法进行抽象。
4. SELF-prototype原型，这个是针对对应的自身，当扩展的影响仅限于其自身，不会影响到其他模块或系统的部分时，例如独立的插件或单独功能组件。

针对这几个部分，分别实现出对应的类，然后进行更改，将这几个部分使用一个接口进行抽象，暂定这个接口为`ExtensionManager`。

`ExtensionManager`需要获取SPI的拓展能力，首先需要实现一个类，这个类为`ExtensionDirector`，这个类的功能定义如下：

```java
public class ExtensionDirector {
//
    private final ConcurrentMap<Class<?>, EnhancedServiceLoader<?>> extensionLoadersMap = new ConcurrentHashMap<>(64);
    private final ConcurrentMap<Class<?>, ExtensionManager> extensionMap = new ConcurrentHashMap<>(64);
    private final ExtensionDirector parent;
    private final ExtensionManager scope;
}
```

seata中所有用到扩展点的地方，都会通过ExtensionManager来获取对应Manager下的扩展点。原本EnhancedServiceLoader#load方法，现在需要委派给全局单例获取拓展点。每个ExtensionManager实例对应一个**ExtensionDirector**，所有获取扩展点逻辑都委派给这个ExtensionDirector。

ExtensionDirector存储一个manager范围中扩展接口class对应EnhancedServiceLoader，从而实现了不同manager管理自己的EnhancedServiceLoader实例。

### 创建一个EnhancedServiceLoader

原本仅仅是单例创建，现在进行分类，可以按照以下步骤进行判断：

1. 如果manager=SELF，每次创建一个EnhancedServiceLoader；
2. 优先走parent获取EnhancedServiceLoader；
3. parent找不到，自己判断scope是否和扩展点**SPI**一致，一致才创建EnhancedServiceLoader；

层级的话按照FRAMEWORK、APPLICATION、MANAGER逐次降低。

### createExtension

目前所有的方法都是使用`getInstance`进行创建，现在使用`createExtension`进行统一，方便抽象出来，同意放入对应的`ExtensionManager`，下面代码参考`dubbo`进行展示：

![img](https://img-1312072469.cos.ap-nanjing.myqcloud.com/0a29d7bbf8034737bc8e90ee6bde15c4~tplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

同理，现在在进行创建时，选择出对应的方法进行构造注入，然后在注入到我们的实例之中。

#### 构造注入

这个主要是构造方法实例话的拓展点，即`instantiate`方法，通过判断对类的类型，选择我们设置的Manager类型，然后进行构造。

### 配置文件相关

原本的是在每个Manager进行读取，然后进行注册，现在多实例，因此需要每个`SeataManager`管理对应的非拓展点的配置。

此处实现一个初始化方法`SeataManagerInitializer`

```java
public interface SeataManagerInitializer {

    void initializeFrameworkManager(FrameworkManager frameworkModel);

    void initializeApplicationManager(ApplicationManager applicationModel);

    void initializeModuleManager(ModuleManager moduleManager);
}
```

每个类拿到对应级别的配置文件，然后进行初始化。

## 多实例

FrameworkManager，支持同一个jvm内部框架级别隔离。

ApplicationManager，支持应用级别隔离。

ModuleManager，支持模块级别隔离。

这样，用户就能直接创建出多个Client，举个例子：

```java
ApplicationConfig applicationConfig = new ApplicationConfig("app1");
applicationConfig.set...
DubboBootstrap.newInstance().application(applicationConfig)
    .registry(new RegistryConfig("zookeeper://127.0.0.1:2181"))
    .protocol(new ProtocolConfig(20880))
    .service(service)
    .start();
///.
DubboBootstrap.newInstance().application(applicationConfig2)
    .set...
    .registry(new RegistryConfig("zookeeper://127.0.0.1:2181"))
    .protocol(new ProtocolConfig( 20880))
    .service(service2)
    .start();

```

用户在进行创建时，可以多个client使用一个端口，然后设置不同的参数。

同理，ModuleManager也可以设置不同的参数，这个里面可以设置一个ReferenceConfig，关联一个ModuleManager。

以上为对应API实现的构建。
