---
type: knowledge
status: active
kind: method
review_after: 2025-07-29
last_reviewed: 2024-07-29
---
## 背景

目前，Seata 框架的实践主要基于单 Client 模式，在这种模式下，服务发现、序列化、压缩等功能都以单例形式存在，提供了全局唯一的配置管理能力。然而，随着业务的发展以及 Seata 自身单元测试和基准测试的隔离性要求，全局单例配置的修改会影响整个系统的稳定性和可控性。因此，提升 Seata 多 Client 能力成为当务之急，以便在不影响全局的情况下，支持多个独立 Client 实例配置管理。

## 任务

为了实现 Seata 多 Client 能力的提升，我们需要开展以下几个方面的工作：

1、多 Client API 设计

- 定义支持多 Client 的 API 接口，以便在不同业务场景下灵活创建和管理独立的 Client 实例。
- API 应能支持不同的配置用例，并确保与现有单例模式的兼容性。
- 多 Client 配置设计及兼容性改造

2、设计独立的配置管理模块，使每个 Client 实例可以拥有自己独立的配置空间。

- 确保新的多 Client 配置机制能与现有的单例配置兼容，避免对当前使用单例模式的系统造成影响。

3、单A元测试及集成测试修复

- 编写和修复相关的单元测试
- 验证多 Client 模式的隔离性和稳定性，确保其兼容现有系统并提供可靠的多 Client 支持。

## 多 Client 的 API 接口设计

首先统一一个对应的公共的Client类`SeataClient`，通过这个类来将不同的Client进行实现，当传入参数时，然后进行对应内容的一个判断，然后到对应的类

在AT模式下Seata Client端主要通过如下三个模块来实现分布式事务：

- **GlobalTransactionScanner：** GlobalTransactionScanner负责初始TM、RM模块，并为添加分布式事务注解的方法添加拦截器，拦截器负责全局事务的开启、提交或回滚
- **DatasourceProxy：** DatasourceProxy为DataSource添加拦截，拦截器会拦截所有SQL执行，并作为RM事务参与方的角色参与分布式事务执行。
- **Rpc Interceptor：** Rpc Interceptor的职责就是负责在多个微服务之间传播事务。

在TCC模式下端通过事务的提交以及回滚进行实现，在进行事务的处理时，并不依赖底层数据库，Client端通过三个阶段控制事务的执行：

- **GlobalTransactionScanner：** GlobalTransactionScanner负责初始TM、RM模块，并为添加分布式事务注解的方法添加拦截器，拦截器负责全局事务的开启、提交或回滚
- **Rpc Interceptor：** Rpc Interceptor的职责就是负责在多个微服务之间传播事务。

这个里面已经抽象出来对应的API，因此不需要再次进行抽象。

在saga模式下，通过以下以下几个模块来实现分布式事务：

- **GlobalTransactionScanner：** GlobalTransactionScanner负责初始TM、RM模块，并为添加分布式事务注解的方法添加拦截器，拦截器负责全局事务的开启、提交或回滚
- **Rpc Interceptor：** Rpc Interceptor的职责就是负责在多个微服务之间传播事务。

在xa模式下，对应的内容如下：

- **GlobalTransactionScanner：** GlobalTransactionScanner负责初始TM、RM模块，并为添加分布式事务注解的方法添加拦截器，拦截器负责全局事务的开启、提交或回滚
- **DatasourceProxy：** DatasourceProxy为DataSource添加拦截，拦截器会拦截所有SQL执行，并作为RM事务参与方的角色参与分布式事务执行。
- **Rpc Interceptor：** Rpc Interceptor的职责就是负责在多个微服务之间传播事务。

涉及到需要进行抽象client端有：

- TMClient
- RMClient
- TmNettyRemotingClient
- RmNettyRemotingClient

将这几个类进行抽象，然后根据对应的类型进行初始化，实例化，以及发送、接收等操作。

可以先简单看看这几个类：

```java
/**
 * The type Tm client.
 *
 */
public class TMClient {

    /**
     * Init.
     *
     * @param applicationId           the application id
     * @param transactionServiceGroup the transaction service group
     */
    public static void init(String applicationId, String transactionServiceGroup) {
        init(applicationId, transactionServiceGroup, null, null);
    }

    /**
     * Init.
     *
     * @param applicationId           the application id
     * @param transactionServiceGroup the transaction service group
     * @param accessKey               the access key
     * @param secretKey               the secret key
     */
    public static void init(String applicationId, String transactionServiceGroup, String accessKey, String secretKey) {
        //TODO: 这里其实就是设计到那天王特哥讲解的东西
        TmNettyRemotingClient tmNettyRemotingClient = TmNettyRemotingClient.getInstance(applicationId, transactionServiceGroup, accessKey, secretKey);
        tmNettyRemotingClient.init();
    }

}
```

```java
/**
 * The Rm client Initiator.
 *
 */
public class RMClient {

    /**
     * Init.
     *
     * @param applicationId           the application id
     * @param transactionServiceGroup the transaction service group
     */
    public static void init(String applicationId, String transactionServiceGroup) {
        RmNettyRemotingClient rmNettyRemotingClient = RmNettyRemotingClient.getInstance(applicationId, transactionServiceGroup);
        rmNettyRemotingClient.setResourceManager(DefaultResourceManager.get());
        rmNettyRemotingClient.setTransactionMessageHandler(DefaultRMHandler.get());
        rmNettyRemotingClient.init();
    }

}

```

```java
/**
 * The rm netty client.
 *
 */
public final class TmNettyRemotingClient extends AbstractNettyRemotingClient {
    private static final Logger LOGGER = LoggerFactory.getLogger(TmNettyRemotingClient.class);
    private static volatile TmNettyRemotingClient instance;
    private static final long KEEP_ALIVE_TIME = Integer.MAX_VALUE;
    private static final int MAX_QUEUE_SIZE = 2000;
    private final AtomicBoolean initialized = new AtomicBoolean(false);
    private String applicationId;
    private String transactionServiceGroup;
    private final AuthSigner signer;
    private String accessKey;
    private String secretKey;


    private TmNettyRemotingClient(NettyClientConfig nettyClientConfig,
                                  EventExecutorGroup eventExecutorGroup,
                                  ThreadPoolExecutor messageExecutor) {
        super(nettyClientConfig, eventExecutorGroup, messageExecutor, NettyPoolKey.TransactionRole.TMROLE);
        this.signer = EnhancedServiceLoader.load(AuthSigner.class);
        // set enableClientBatchSendRequest
        Configuration configuration = ConfigurationFactory.getInstance();
        this.enableClientBatchSendRequest = configuration.getBoolean(ConfigurationKeys.ENABLE_TM_CLIENT_BATCH_SEND_REQUEST,
                DefaultValues.DEFAULT_ENABLE_TM_CLIENT_BATCH_SEND_REQUEST);
        configuration.addConfigListener(ConfigurationKeys.ENABLE_TM_CLIENT_BATCH_SEND_REQUEST, new CachedConfigurationChangeListener() {
            @Override
            public void onChangeEvent(ConfigurationChangeEvent event) {
                String dataId = event.getDataId();
                String newValue = event.getNewValue();
                if (ConfigurationKeys.ENABLE_TM_CLIENT_BATCH_SEND_REQUEST.equals(dataId) && StringUtils.isNotBlank(newValue)) {
                    enableClientBatchSendRequest = Boolean.parseBoolean(newValue);
                }
            }
        });
    }

    /**
     * Gets instance.
     *
     * @param applicationId           the application id
     * @param transactionServiceGroup the transaction service group
     * @return the instance
     */
    public static TmNettyRemotingClient getInstance(String applicationId, String transactionServiceGroup) {
        return getInstance(applicationId, transactionServiceGroup, null, null);
    }

    /**
     * Gets instance.
     *
     * @param applicationId           the application id
     * @param transactionServiceGroup the transaction service group
     * @param accessKey               the access key
     * @param secretKey               the secret key
     * @return the instance
     */
    public static TmNettyRemotingClient getInstance(String applicationId, String transactionServiceGroup, String accessKey, String secretKey) {
        TmNettyRemotingClient tmRpcClient = getInstance();
        tmRpcClient.setApplicationId(applicationId);
        tmRpcClient.setTransactionServiceGroup(transactionServiceGroup);
        tmRpcClient.setAccessKey(accessKey);
        tmRpcClient.setSecretKey(secretKey);
        return tmRpcClient;
    }

    /**
     * Gets instance.
     *
     * @return the instance
     */
    public static TmNettyRemotingClient getInstance() {
        if (instance == null) {
            synchronized (TmNettyRemotingClient.class) {
                if (instance == null) {
                    NettyClientConfig nettyClientConfig = new NettyClientConfig();
                    final ThreadPoolExecutor messageExecutor = new ThreadPoolExecutor(
                            nettyClientConfig.getClientWorkerThreads(), nettyClientConfig.getClientWorkerThreads(),
                            KEEP_ALIVE_TIME, TimeUnit.SECONDS,
                            new LinkedBlockingQueue<>(MAX_QUEUE_SIZE),
                            new NamedThreadFactory(nettyClientConfig.getTmDispatchThreadPrefix(),
                                    nettyClientConfig.getClientWorkerThreads()),
                            RejectedPolicies.runsOldestTaskPolicy());
                    instance = new TmNettyRemotingClient(nettyClientConfig, null, messageExecutor);
                }
            }
        }
        return instance;
    }

    /**
     * Sets application id.
     *
     * @param applicationId the application id
     */
    public void setApplicationId(String applicationId) {
        this.applicationId = applicationId;
    }

    /**
     * Sets transaction service group.
     *
     * @param transactionServiceGroup the transaction service group
     */
    public void setTransactionServiceGroup(String transactionServiceGroup) {
        this.transactionServiceGroup = transactionServiceGroup;
    }

    /**
     * Sets access key.
     *
     * @param accessKey the access key
     */
    protected void setAccessKey(String accessKey) {
        if (null != accessKey) {
            this.accessKey = accessKey;
            return;
        }
        this.accessKey = System.getProperty(org.apache.seata.common.ConfigurationKeys.SEATA_ACCESS_KEY);
    }

    /**
     * Sets secret key.
     *
     * @param secretKey the secret key
     */
    protected void setSecretKey(String secretKey) {
        if (null != secretKey) {
            this.secretKey = secretKey;
            return;
        }
        this.secretKey = System.getProperty(org.apache.seata.common.ConfigurationKeys.SEATA_SECRET_KEY);
    }

    @Override
    public void init() {
        // registry processor
        registerProcessor();
        if (initialized.compareAndSet(false, true)) {
            super.init();
            if (org.apache.seata.common.util.StringUtils.isNotBlank(transactionServiceGroup)) {
                initConnection();
            }
        }
    }

    @Override
    public String getTransactionServiceGroup() {
        return transactionServiceGroup;
    }

    @Override
    public boolean isEnableClientBatchSendRequest() {
        return enableClientBatchSendRequest;
    }

    @Override
    public long getRpcRequestTimeout() {
        return NettyClientConfig.getRpcTmRequestTimeout();
    }

    @Override
    public void onRegisterMsgSuccess(String serverAddress, Channel channel, Object response,
                                     AbstractMessage requestMessage) {
        RegisterTMRequest registerTMRequest = (RegisterTMRequest) requestMessage;
        RegisterTMResponse registerTMResponse = (RegisterTMResponse) response;
        if (LOGGER.isInfoEnabled()) {
            LOGGER.info("register TM success. client version:{}, server version:{},channel:{}", registerTMRequest.getVersion(), registerTMResponse.getVersion(), channel);
        }
        getClientChannelManager().registerChannel(serverAddress, channel);
    }

    @Override
    public void onRegisterMsgFail(String serverAddress, Channel channel, Object response,
                                  AbstractMessage requestMessage) {
        RegisterTMRequest registerTMRequest = (RegisterTMRequest) requestMessage;
        RegisterTMResponse registerTMResponse = (RegisterTMResponse) response;
        String errMsg = String.format(
                "register TM failed. client version: %s,server version: %s, errorMsg: %s, " + "channel: %s", registerTMRequest.getVersion(), registerTMResponse.getVersion(), registerTMResponse.getMsg(), channel);
        throw new FrameworkException(errMsg);
    }

    @Override
    public void destroy() {
        super.destroy();
        initialized.getAndSet(false);
        instance = null;
    }

    @Override
    protected Function<String, NettyPoolKey> getPoolKeyFunction() {
        return severAddress -> {
            RegisterTMRequest message = new RegisterTMRequest(applicationId, transactionServiceGroup, getExtraData());
            return new NettyPoolKey(NettyPoolKey.TransactionRole.TMROLE, severAddress, message);
        };
    }

    private void registerProcessor() {
        // 1.registry TC response processor
        // 注册Seata-Server返回的Response的处理Processor，用于Client主动发起Request，
        // Seata-Server返回的Response
        ClientOnResponseProcessor onResponseProcessor =
                new ClientOnResponseProcessor(mergeMsgMap, super.getFutures(), getTransactionMessageHandler());
        super.registerProcessor(MessageType.TYPE_SEATA_MERGE_RESULT, onResponseProcessor, null);
        super.registerProcessor(MessageType.TYPE_GLOBAL_BEGIN_RESULT, onResponseProcessor, null);
        super.registerProcessor(MessageType.TYPE_GLOBAL_COMMIT_RESULT, onResponseProcessor, null);
        super.registerProcessor(MessageType.TYPE_GLOBAL_REPORT_RESULT, onResponseProcessor, null);
        super.registerProcessor(MessageType.TYPE_GLOBAL_ROLLBACK_RESULT, onResponseProcessor, null);
        super.registerProcessor(MessageType.TYPE_GLOBAL_STATUS_RESULT, onResponseProcessor, null);
        super.registerProcessor(MessageType.TYPE_REG_CLT_RESULT, onResponseProcessor, null);
        super.registerProcessor(MessageType.TYPE_BATCH_RESULT_MSG, onResponseProcessor, null);
        // 2.registry heartbeat message processor
        // ClientOnResponseProcessor负责把Client发送的Request和Seata-Server
        // 返回的Response对应起来，从而实现Rpc
        ClientHeartbeatProcessor clientHeartbeatProcessor = new ClientHeartbeatProcessor();
        super.registerProcessor(MessageType.TYPE_HEARTBEAT_MSG, clientHeartbeatProcessor, null);
    }

    private String getExtraData() {
        String ip = NetUtil.getLocalIp();
        String timestamp = String.valueOf(System.currentTimeMillis());
        String digestSource;
        if (StringUtils.isEmpty(ip)) {
            digestSource = transactionServiceGroup + ",127.0.0.1," + timestamp;
        } else {
            digestSource = transactionServiceGroup + "," + ip + "," + timestamp;
        }
        String digest = signer.sign(digestSource, secretKey);
        StringBuilder sb = new StringBuilder();
        sb.append(RegisterTMRequest.UDATA_AK).append(org.apache.seata.common.ConfigurationKeys.EXTRA_DATA_KV_CHAR).append(accessKey).append(
            org.apache.seata.common.ConfigurationKeys.EXTRA_DATA_SPLIT_CHAR);
        sb.append(RegisterTMRequest.UDATA_DIGEST).append(org.apache.seata.common.ConfigurationKeys.EXTRA_DATA_KV_CHAR).append(digest).append(
            org.apache.seata.common.ConfigurationKeys.EXTRA_DATA_SPLIT_CHAR);
        sb.append(RegisterTMRequest.UDATA_TIMESTAMP).append(org.apache.seata.common.ConfigurationKeys.EXTRA_DATA_KV_CHAR).append(timestamp).append(
            org.apache.seata.common.ConfigurationKeys.EXTRA_DATA_SPLIT_CHAR);
        sb.append(RegisterTMRequest.UDATA_AUTH_VERSION).append(
            org.apache.seata.common.ConfigurationKeys.EXTRA_DATA_KV_CHAR).append(signer.getSignVersion()).append(
            org.apache.seata.common.ConfigurationKeys.EXTRA_DATA_SPLIT_CHAR);
        return sb.toString();
    }

    private void initConnection() {
        boolean failFast = ConfigurationFactory.getInstance().getBoolean(
                ConfigurationKeys.ENABLE_TM_CLIENT_CHANNEL_CHECK_FAIL_FAST,
                DefaultValues.DEFAULT_CLIENT_CHANNEL_CHECK_FAIL_FAST);
        getClientChannelManager().initReconnect(transactionServiceGroup, failFast);
    }

}
```

```java
/**
 * The Rm netty client.
 *
 */

public final class RmNettyRemotingClient extends AbstractNettyRemotingClient {

    private static final Logger LOGGER = LoggerFactory.getLogger(RmNettyRemotingClient.class);
    private ResourceManager resourceManager;
    private static volatile RmNettyRemotingClient instance;
    private final AtomicBoolean initialized = new AtomicBoolean(false);
    private static final long KEEP_ALIVE_TIME = Integer.MAX_VALUE;
    private static final int MAX_QUEUE_SIZE = 20000;
    private String applicationId;
    private String transactionServiceGroup;

    @Override
    public void init() {
        // registry processor
        registerProcessor();
        if (initialized.compareAndSet(false, true)) {
            super.init();

            // Found one or more resources that were registered before initialization
            if (resourceManager != null
                    && !resourceManager.getManagedResources().isEmpty()
                    && StringUtils.isNotBlank(transactionServiceGroup)) {
                boolean failFast = ConfigurationFactory.getInstance().getBoolean(
                        ConfigurationKeys.ENABLE_RM_CLIENT_CHANNEL_CHECK_FAIL_FAST,
                        DefaultValues.DEFAULT_CLIENT_CHANNEL_CHECK_FAIL_FAST);
                getClientChannelManager().initReconnect(transactionServiceGroup, failFast);
            }
        }
    }

    private RmNettyRemotingClient(NettyClientConfig nettyClientConfig, EventExecutorGroup eventExecutorGroup,
                                  ThreadPoolExecutor messageExecutor) {
        super(nettyClientConfig, eventExecutorGroup, messageExecutor, TransactionRole.RMROLE);
        // set enableClientBatchSendRequest
        Configuration configuration = ConfigurationFactory.getInstance();
        this.enableClientBatchSendRequest = configuration.getBoolean(ConfigurationKeys.ENABLE_RM_CLIENT_BATCH_SEND_REQUEST,
                ConfigurationFactory.getInstance().getBoolean(ConfigurationKeys.ENABLE_CLIENT_BATCH_SEND_REQUEST,DefaultValues.DEFAULT_ENABLE_RM_CLIENT_BATCH_SEND_REQUEST));
        configuration.addConfigListener(ConfigurationKeys.ENABLE_RM_CLIENT_BATCH_SEND_REQUEST, new CachedConfigurationChangeListener() {
            @Override
            public void onChangeEvent(ConfigurationChangeEvent event) {
                String dataId = event.getDataId();
                String newValue = event.getNewValue();
                if (ConfigurationKeys.ENABLE_RM_CLIENT_BATCH_SEND_REQUEST.equals(dataId) && StringUtils.isNotBlank(newValue)) {
                    enableClientBatchSendRequest = Boolean.parseBoolean(newValue);
                }
            }
        });
    }

    /**
     * Gets instance.
     *
     * @param applicationId           the application id
     * @param transactionServiceGroup the transaction service group
     * @return the instance
     */
    public static RmNettyRemotingClient getInstance(String applicationId, String transactionServiceGroup) {
        RmNettyRemotingClient rmNettyRemotingClient = getInstance();
        rmNettyRemotingClient.setApplicationId(applicationId);
        rmNettyRemotingClient.setTransactionServiceGroup(transactionServiceGroup);
        return rmNettyRemotingClient;
    }

    /**
     * Gets instance.
     *
     * @return the instance
     */
    public static RmNettyRemotingClient getInstance() {
        if (instance == null) {
            synchronized (RmNettyRemotingClient.class) {
                if (instance == null) {
                    NettyClientConfig nettyClientConfig = new NettyClientConfig();
                    final ThreadPoolExecutor messageExecutor = new ThreadPoolExecutor(
                        nettyClientConfig.getClientWorkerThreads(), nettyClientConfig.getClientWorkerThreads(),
                        KEEP_ALIVE_TIME, TimeUnit.SECONDS, new LinkedBlockingQueue<>(MAX_QUEUE_SIZE),
                        new NamedThreadFactory(nettyClientConfig.getRmDispatchThreadPrefix(),
                            nettyClientConfig.getClientWorkerThreads()), new ThreadPoolExecutor.CallerRunsPolicy());
                    instance = new RmNettyRemotingClient(nettyClientConfig, null, messageExecutor);
                }
            }
        }
        return instance;
    }

    /**
     * Sets application id.
     *
     * @param applicationId the application id
     */
    public void setApplicationId(String applicationId) {
        this.applicationId = applicationId;
    }

    /**
     * Sets transaction service group.
     *
     * @param transactionServiceGroup the transaction service group
     */
    public void setTransactionServiceGroup(String transactionServiceGroup) {
        this.transactionServiceGroup = transactionServiceGroup;
    }

    /**
     * Sets resource manager.
     *
     * @param resourceManager the resource manager
     */
    public void setResourceManager(ResourceManager resourceManager) {
        this.resourceManager = resourceManager;
    }

    @Override
    public void onRegisterMsgSuccess(String serverAddress, Channel channel, Object response,
                                     AbstractMessage requestMessage) {
        RegisterRMRequest registerRMRequest = (RegisterRMRequest)requestMessage;
        RegisterRMResponse registerRMResponse = (RegisterRMResponse)response;
        if (LOGGER.isInfoEnabled()) {
            LOGGER.info("register RM success. client version:{}, server version:{},channel:{}", registerRMRequest.getVersion(), registerRMResponse.getVersion(), channel);
        }
        getClientChannelManager().registerChannel(serverAddress, channel);
        String dbKey = getMergedResourceKeys();
        if (registerRMRequest.getResourceIds() != null) {
            if (!registerRMRequest.getResourceIds().equals(dbKey)) {
                sendRegisterMessage(serverAddress, channel, dbKey);
            }
        }

    }

    @Override
    public void onRegisterMsgFail(String serverAddress, Channel channel, Object response,
                                  AbstractMessage requestMessage) {
        RegisterRMRequest registerRMRequest = (RegisterRMRequest)requestMessage;
        RegisterRMResponse registerRMResponse = (RegisterRMResponse)response;
        String errMsg = String.format(
            "register RM failed. client version: %s,server version: %s, errorMsg: %s, " + "channel: %s", registerRMRequest.getVersion(), registerRMResponse.getVersion(), registerRMResponse.getMsg(), channel);
        throw new FrameworkException(errMsg);
    }

    /**
     * Register new db key.
     *
     * @param resourceGroupId the resource group id
     * @param resourceId      the db key
     */
    public void registerResource(String resourceGroupId, String resourceId) {

        // Resource registration cannot be performed until the RM client is initialized
        if (StringUtils.isBlank(transactionServiceGroup)) {
            return;
        }

        // ResourceId can not be null or empty
        if (StringUtils.isBlank(resourceId)) {
            LOGGER.warn("The resourceId must not be null or empty when registering the RM client.");
            return;
        }

        if (getClientChannelManager().getChannels().isEmpty()) {
            boolean failFast = ConfigurationFactory.getInstance().getBoolean(
                    ConfigurationKeys.ENABLE_RM_CLIENT_CHANNEL_CHECK_FAIL_FAST,
                    DefaultValues.DEFAULT_CLIENT_CHANNEL_CHECK_FAIL_FAST);
            getClientChannelManager().initReconnect(transactionServiceGroup, failFast);
            return;
        }
        synchronized (getClientChannelManager().getChannels()) {
            for (Map.Entry<String, Channel> entry : getClientChannelManager().getChannels().entrySet()) {
                String serverAddress = entry.getKey();
                Channel rmChannel = entry.getValue();
                if (LOGGER.isInfoEnabled()) {
                    LOGGER.info("will register resourceId:{}", resourceId);
                }
                sendRegisterMessage(serverAddress, rmChannel, resourceId);
            }
        }
    }

    public void sendRegisterMessage(String serverAddress, Channel channel, String resourceId) {
        RegisterRMRequest message = new RegisterRMRequest(applicationId, transactionServiceGroup);
        message.setResourceIds(resourceId);
        try {
            super.sendAsyncRequest(channel, message);
        } catch (FrameworkException e) {
            if (e.getErrcode() == FrameworkErrorCode.ChannelIsNotWritable && serverAddress != null) {
                getClientChannelManager().releaseChannel(channel, serverAddress);
                if (LOGGER.isInfoEnabled()) {
                    LOGGER.info("remove not writable channel:{}", channel);
                }
            } else {
                LOGGER.error("register resource failed, channel:{},resourceId:{}", channel, resourceId, e);
            }
        }
    }

    public String getMergedResourceKeys() {
        Map<String, Resource> managedResources = resourceManager.getManagedResources();
        Set<String> resourceIds = managedResources.keySet();
        if (!resourceIds.isEmpty()) {
            StringBuilder sb = new StringBuilder();
            boolean first = true;
            for (String resourceId : resourceIds) {
                if (StringUtils.isBlank(resourceId)) {
                    LOGGER.warn("The resourceId must not be null or empty when registering the RM client.");
                    continue;
                }
                if (first) {
                    first = false;
                } else {
                    sb.append(DBKEYS_SPLIT_CHAR);
                }
                sb.append(resourceId);
            }
            return sb.toString();
        }
        return null;
    }

    @Override
    public void destroy() {
        super.destroy();
        initialized.getAndSet(false);
        instance = null;
    }

    @Override
    protected Function<String, NettyPoolKey> getPoolKeyFunction() {
        return serverAddress -> {
            String resourceIds = getMergedResourceKeys();
            if (resourceIds != null && LOGGER.isInfoEnabled()) {
                LOGGER.info("RM will register :{}", resourceIds);
            }
            RegisterRMRequest message = new RegisterRMRequest(applicationId, transactionServiceGroup);
            message.setResourceIds(resourceIds);
            return new NettyPoolKey(NettyPoolKey.TransactionRole.RMROLE, serverAddress, message);
        };
    }

    @Override
    protected String getTransactionServiceGroup() {
        return transactionServiceGroup;
    }

    @Override
    public boolean isEnableClientBatchSendRequest() {
        return enableClientBatchSendRequest;
    }

    @Override
    public long getRpcRequestTimeout() {
        return NettyClientConfig.getRpcRmRequestTimeout();
    }

    private void registerProcessor() {
        // 1.registry rm client handle branch commit processor
        RmBranchCommitProcessor rmBranchCommitProcessor = new RmBranchCommitProcessor(getTransactionMessageHandler(), this);
        super.registerProcessor(MessageType.TYPE_BRANCH_COMMIT, rmBranchCommitProcessor, messageExecutor);
        // 2.registry rm client handle branch rollback processor
        RmBranchRollbackProcessor rmBranchRollbackProcessor = new RmBranchRollbackProcessor(getTransactionMessageHandler(), this);
        super.registerProcessor(MessageType.TYPE_BRANCH_ROLLBACK, rmBranchRollbackProcessor, messageExecutor);
        // 3.registry rm handler undo log processor
        RmUndoLogProcessor rmUndoLogProcessor = new RmUndoLogProcessor(getTransactionMessageHandler());
        super.registerProcessor(MessageType.TYPE_RM_DELETE_UNDOLOG, rmUndoLogProcessor, messageExecutor);
        // 4.registry TC response processor
        ClientOnResponseProcessor onResponseProcessor =
            new ClientOnResponseProcessor(mergeMsgMap, super.getFutures(), getTransactionMessageHandler());
        super.registerProcessor(MessageType.TYPE_SEATA_MERGE_RESULT, onResponseProcessor, null);
        super.registerProcessor(MessageType.TYPE_BRANCH_REGISTER_RESULT, onResponseProcessor, null);
        super.registerProcessor(MessageType.TYPE_BRANCH_STATUS_REPORT_RESULT, onResponseProcessor, null);
        super.registerProcessor(MessageType.TYPE_GLOBAL_LOCK_QUERY_RESULT, onResponseProcessor, null);
        super.registerProcessor(MessageType.TYPE_REG_RM_RESULT, onResponseProcessor, null);
        super.registerProcessor(MessageType.TYPE_BATCH_RESULT_MSG, onResponseProcessor, null);
        // 5.registry heartbeat message processor
        ClientHeartbeatProcessor clientHeartbeatProcessor = new ClientHeartbeatProcessor();
        super.registerProcessor(MessageType.TYPE_HEARTBEAT_MSG, clientHeartbeatProcessor, null);
    }
}
```

**可以看到，这几个方法中有对应方法的初始化`init`,单例模式进行实例化的`getInstance`以及注册成功的方法`onRegisterMsgSuccess`，将这几个方法进行抽象，然后抽象出初始化方法`init()`、单例实例化`getInstance`发送消息`send()`，从而实现对应的创建。**

首先创建出一个`Cofig`类，这个类包含一些Client的公共字段

```java
public class Cofig {
    String name//需要调用的服务名称
    String applicationId;
    String transactionService;//原本仅仅是针对服务群进行初始化，现在一个服务群可以初始化多个Client端
    String type; //
    ...
}
```

首先详细说明`init`方法。在进行初始化时，为了配合多个内容，不在根据对应的group进行初始化，而是根据每个service进行初始化；规定里面的type字段设置出所需启动的服务类型，然后再进行创建时，通过不同的设置：

```java
//创建出一个Config类，将里面对应的字段直接进行拿取，设想的对应字段
public void init(Config config) {
    //方案1：拿取对应的名字，然后直接返回对应的函数
    //方案2：直接将公共的部分提取出来，不同的部分直接判断，然后进行初始化
    //目前倾向于方案2
    //在进行初始化时，为了配合多个内容，不在根据对应的group进行初始化，而是根据每个service进行初始化
    //规定里面的type字段设置出所需启动的服务类型，
    //然后再进行创建时，通过不同的设置，创建出多个Client端
}  
```

接着时`getInstance()`这个方法，这个方法在进行创建时，要根据不同类型来进行判断，目前所想配置方法如下：

```java
public  <T extends AbstractNettyRemotingClient> T getInstance(String name) {
    //判定出对应的实例，然后进行创建

}
```

然后是规定消息发送`send()`与`register()`方法，这个方法在进行创建时，不同的初始化方法可能需要发送不同的消息，这里的话仍然是在`Cofig`中设置一个字段，判断出需要发送消息的类型，然后做出不同的判断，同时由于时多客户端，可能存在不同的客户端给一个事务进行响应，这部分需要控制好并发。同理接收消息也是一样的。

```java
//发送消息
public void send(Cofig config，/*以及对应原本的配置*/){
    //判定出不同的逻辑，然后进行对应的优化
}
    
//接收消息
public void register(Cofig config，/*以及对应原本的配置*/){
    //判定出不同的逻辑，然后进行对应的优化
}   
```

最后，由于原本是一个Server对应一个Client，现在可能一个Server对应多个Client，因此为了保证事务的准确性，创建出新的函数`startTransaction`以及`endTransaction`，用于控制事务的开启与结束，同时在Cofig中设置事务开启字段，判定出是否需要进行事务初始化。

```java
//开启事务
public void startTransaction(String config，/*以及对应原本的配置*/){
    //判定出不同的逻辑，然后进行对应的优化
}

//关闭事务
public void endTransaction(String config，/*以及对应原本的配置*/){
    //判定出不同的逻辑，然后进行对应的优化
}
```

以上为本课题的初步API设计，具体的字段可能在项目创建的过程中有所添加。
