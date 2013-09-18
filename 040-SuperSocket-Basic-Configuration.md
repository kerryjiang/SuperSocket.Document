# SuperSocket 基本配置

> 关键字: 配置文档, 配置示例, 根配置, 服务器实例配置, Windows服务配置

## 一个配置示例

    <?xml version="1.0"?>
    <configuration>
        <configSections>
            <section name="superSocket"
                     type="SuperSocket.SocketEngine.Configuration.SocketServiceConfig, SuperSocket.SocketEngine" />
        </configSections>
        <appSettings>
            <add key="ServiceName" value="SupperSocketService" />
        </appSettings>
        <superSocket>
            <servers>
                <server name="TelnetServerA"
                        serverTypeName="TelnetServer"
                        ip="Any"
                        port="2020">
                </server>
                <server name="TelnetServerB"
                        serverTypeName="TelnetServer"
                        ip="Any"
                        port="2021">
                </server>
            </servers>
            <serverTypes>
                <add name="TelnetServer"
                     type="SuperSocket.QuickStart.TelnetServer_StartByConfig.TelnetServer, SuperSocket.QuickStart.TelnetServer_StartByConfig"/>
            </serverTypes>
        </superSocket>
        <startup>
            <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.0" />
        </startup>
    </configuration>

## 根配置

配置节点 "superSocket" SuperSocket 配置的根节点，它定义了 SuperSocket 所需要的全局参数。 让我们先看下根节点的所有配置属性:

* maxWorkingThreads: 线程池最大工作线程数量;
* minWorkingThreads: 线程池最小工作线程数量;
* maxCompletionPortThreads: 线程池最大完成端口线程数量;
* minCompletionPortThreads: 线程池最小完成端口线程数量;
* disablePerformanceDataCollector: 是否禁用性能数据采集;
* performanceDataCollectInterval: 性能数据采集频率 (单位为秒, 默认值: 60);
* isolation: SuperSocket 服务器实例隔离级别
       * None - 无隔离
       * AppDomain - 应用程序域级别的隔离，多个服务器实例运行在各自独立的应用程序域之中
	   * Process - 进程级别的隔离，多个服务器实例运行在各自独立的进程之中
* logFactory: 默认logFactory的名字, 所有可用的 log factories定义在子节点 "logFactories" 之中， 我们将会在下面的文档中介绍它;
* defaultCulture: 整个程序的默认 thread culture，只在.Net 4.5中可用;

## 服务器实例配置
在根节点中，有一个名为 "servers" 的子节点，你可以定义一个或者多个server节点来代表服务器实例。 这些服务器实例可以是同一种 AppServer 类型， 也可以是不同的类型。

Server 节点的所有属性如下:

* name: 服务器实例的名称;
* serverType: 服务器实例的类型的完整名称;
* serverTypeName: 所选用的服务器类型在 serverTypes 节点的名字，配置节点 serverTypes 用于定义所有可用的服务器类型，我们将在后面再做详细介绍;
* ip: 服务器监听的ip地址。你可以设置具体的地址，也可以设置为下面的值
      Any - 所有的IPv4地址
      IPv6Any - 所有的IPv6地址
* port: 服务器监听的端口;
* listenBacklog: 监听队列的大小;
* mode: Socket服务器运行的模式, Tcp (默认) 或者 Udp;
* disabled: 服务器实例是否禁用了;
* startupOrder: 服务器实例启动顺序, bootstrap 将按照此值的顺序来启动多个服务器实例;
* sendTimeOut: 发送数据超时时间;
* sendingQueueSize: 发送队列最大长度;
* maxConnectionNumber: 可允许连接的最大连接数;
* receiveBufferSize: 接收缓冲区大小;
* sendBufferSize: 发送缓冲区大小;
* syncSend: 是否启用同步发送模式, 默认值: false;
* logCommand: 是否记录命令执行的记录;
* logBasicSessionActivity: 是否记录session的基本基本活动，如连接和断开;
* clearIdleSession: true 或 false, 是否定时清空空闲会话，默认值是 false;
* clearIdleSessionInterval: 清空空闲会话的时间间隔, 默认值是120, 单位为秒;
* idleSessionTimeOut: 会话超时时间，默认值为300，单位为秒;
* security: Empty, Tls, Ssl3. Socket服务器所采用的传输层加密协议，默认值为空;
* maxRequestLength: 最大允许的请求长度，默认值为1024;
* textEncoding: 文本的默认编码，默认值是 ASCII;
* defaultCulture: 此服务器实例的默认 thread culture, 只在.Net 4.5中可用而且在隔离级别为 'None' 时无效;
* disableSessionSnapshot: 是否禁用会话快照, 默认值为 false.
* sessionSnapshotInterval: 会话快照时间间隔, 默认值是 5, 单位为秒;
* keepAliveTime: 网络连接正常情况下的keep alive数据的发送间隔, 默认值为 600, 单位为秒;
* keepAliveInterval: Keep alive失败之后, keep alive探测包的发送间隔，默认值为 60, 单位为秒;
* certificate: 这各节点用于定义用于此服务器实例的X509Certificate证书的信息
    
    它有两种用法:

    - 从文件加载证书

              <certificate filePath="localhost.pfx" password="supersocket" />

    - 从本地证书库加载证书

              <certificate storeName="My" storeLocation="LocalMachine" thumbprint="‎f42585bceed2cb049ef4a3c6d0ad572a6699f6f3"/>


* connectionFilter: 定义该实例所使用的连接过滤器的名字，多个过滤器用 ',' 或者 ';' 分割开来。 可用的连接过滤器定义在根节点的一个子节点内，将会在下面的文档中做更多介绍;

* commandLoader: 定义该实例所使用的命令加载器的名字，多个过滤器用 ',' 或者 ';' 分割开来。 可用的命令加载器定义在根节点的一个子节点内，将会在下面的文档中做更多介绍;

* logFactory: 定义该实例所使用的日志工厂(LogFactory)的名字。 可用的日志工厂(LogFactory)定义在根节点的一个子节点内，将会在下面的文档中做更多介绍; 如果你不设置该属性，定义在根节点的日志工厂(LogFactory)将会被使用，如果根节点也未定义日志工厂(LogFactory)，该实例将会使用内置的 log4net 日志工厂(LogFactory);

* listeners: 此配置节点用于支持一个服务器实例监听多个IP/端口组合。 此配置节点应该包含一个或者多个拥有一下属性的listener节点:

        ip: the listening ip;
        port: the listening port;
        backlog: the listening back log size;
        security: the security mode (None/Default/Tls/Ssl/...);
    
    例如:

        <server name="EchoServer" serverTypeName="EchoService">
          <listeners>
            <add ip="Any" port="2012" />
            <add ip="IPv6Any" port="2012" />
          </listeners>
        </server>


* receiveFilterFactory: 定义该实例所使用的接收过滤器工厂的名字;

## 服务器类型配置
服务器类型节点是一个在根节点下面的配置集合。你可以添加一个或者多个拥有属性'name'和'type'的配置元素:

        <serverTypes>
            <add name="TelnetServerType"
                 type="SuperSocket.QuickStart.TelnetServer_StartByConfig.TelnetServer, SuperSocket.QuickStart.TelnetServer_StartByConfig"/>
        </serverTypes>


由于上面定一个服务器类型的名称为 "TelnetServerType"，你可以将服务器实例节点的属性"serverTypeName"设置为 "TelnetServerType"， 用于运行该类型的服务器:

        <server name="TelnetServerA"
                serverTypeName="TelnetServerType"
                ip="Any"
                port="2020">
        </server>


## Log Factories 配置
和服务器类型配置节点相同, 一也可以定义一个或者多个L日志工厂(LogFatory) 然后再服务器实例中使用。 唯一的区别是它可以设置在根配置节点上:

    <logFactories>
      <add name="ConsoleLogFactory"
           type="SuperSocket.SocketBase.Logging.ConsoleLogFactory, SuperSocket.SocketBase" />
    </logFactories>

在根节点设置LogFactory:

    <superSocket logFactory="ConsoleLogFactory">
        ...
        ...
    </superSocket>


在服务器节点设置LogFactory:

    <server name="TelnetServerA"
           logFactory="ConsoleLogFactory"
           ip="Any"
           port="2020">
    </server>


## SuperSocket Windows 服务的配置
你可能知道, SuperSocket提供了一个可以Windows服务形式运行的容器 "SuperSocket.SocketService.exe".

你可以通过配置来定义此Windows服务的名字:

    <appSettings>
        <add key="ServiceName" value="SuperSocketService" />
    </appSettings>

它还支持其它针对Windows服务的配置属性:

    ServiceDescription: 此Windows服务的描述
    ServicesDependedOn: 此服务依赖的其他服务; 此服务价格会在其依赖的服务启动之后再启动; 多个依赖服务用 ',' 或者 ';' 分割
