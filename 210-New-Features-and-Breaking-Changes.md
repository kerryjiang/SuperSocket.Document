# 新功能和不兼容的更改

> __关键字__: SuperSocket 1.6, 新功能, 不兼容的更改, textEncoding, defaultCulture, 进程级别隔离, SuperSocket.Agent.exe


## 新的配置属性 "textEncoding"
在 __SuperSocket__ 1.6 之前的版本, 当你通过Session对象发送文本时, 将文本信息转换成能够通过Socket传输的二进制数据的默认编码是UTF8。 你可以通过设置 Session 的 Charset 属性来修改这个编码。

现在在 __SuperSocket__ 1.6 中, 你可以在配置中定义字符编码。

新的配置属性 "textEncoding" 被加入到了服务器配置节点:

	<superSocket>
		<servers>
		  <server name="TelnetServer"
			  textEncoding="UTF-8"
			  serverType="YourAppServer, YourAssembly"
			  ip="Any" port="2020">
		  </server>
		</servers>
	</superSocket>

## 新的配置属性 "defaultCulture"
这个新增的功能只支持 .Net framework 4.5 及其以上版本。 它允许你设置所有线程的默认Culture, 不管这些线程是如何创建，通过代码或者来自于线程池.

这个新的配置属性 "defaultCulture" 可以加到配置根节点或者服务器实例节点，因此你可以为所有服务器实例配置默认的 Culture, 你也可以为每个服务器单独设置默认 Culture:

	<superSocket defaultCulture="en-US">
		<servers>
		  <server name="TelnetServerA"
			  serverType="YourAppServer, YourAssembly"
			  ip="Any" port="2020">
		  </server>
		  <server name="TelnetServerB"
			  defaultCulture="zn-CN"
			  serverType="YourAppServer, YourAssembly"
			  ip="Any" port="2021">
		  </server>
		  <server name="TelnetServerC"
			  defaultCulture="zn-TW"
			  serverType="YourAppServer, YourAssembly"
			  ip="Any" port="2021">
		  </server>
		</servers>
	</superSocket>

## 进程级别隔离
在 __SuperSocket__ 1.5 中, 我们增加了 AppDomain 级别隔离的功能，让你可以运行多个服务器实例在相互独立的 AppDomain 上。
此功能提供了较高级别的安全性和资源的隔离，并且给哪些希望将SuperSocket当做多服务托管程序运行的用户带来了很多其他的帮助。

在 __SuperSocket__ 1.6 中, 我们引入了更高级别的进程级隔离。 当你将 SuperSocket 配置成这种级别的隔离， 你的多个服务器实例将会运行在各自独立的进程当中。 配置文件请参考:

	<superSocket isolation="Process">
		<servers>
		  <server name="TelnetServerA"
			  serverType="YourAppServer, YourAssembly"
			  ip="Any" port="2020">
		  </server>
		  <server name="TelnetServerB"
			  serverType="YourAppServer, YourAssembly"
			  ip="Any" port="2021">
		  </server>
		</servers>
	</superSocket>

除了上面的配置, 你还需要包含可执行文件 "SuperSocket.Agent.exe" 到你的项目输出里面去, 这个可执行文件是由SuperSocket提供的。

当你启动你的 SuperSocket 之后, 你将会发现 SuperSocket 启动了多个进程:

    SuperSocket.SocketService.exe
    SuperSocket.Agent.exe
	SuperSocket.Agent.exe


## 性能数据采集的应用程序接口的改动
性能数据采集的应用程序接口作了修改，两个虚方法已经被更改:

    protected virtual void UpdateServerSummary(ServerSummary serverSummary);

    protected virtual void OnServerSummaryCollected(NodeSummary nodeSummary, ServerSummary serverSummary)

更改为:

	protected virtual void UpdateServerStatus(StatusInfoCollection serverStatus)

	protected virtual void OnServerStatusCollected(StatusInfoCollection nodeStatus, StatusInfoCollection serverStatus)


类 __ServerSummary__ 和 __NodeSummary__ 已经被弃用，取而代之，你应该使用类 __StatusInfoCollection__。

## 证书节点新增配置属性 "storeLocation"
你可以指定你想要加载的证书的存储地点:

    <certificate storeName="My" storeLocation="LocalMachine" thumbprint="‎f42585bceed2cb049ef4a3c6d0ad572a6699f6f3"/>


## 从服务器端主动发起连接

你可以从服务器端主动连接客户端, 连接建立之后的网络通信处理将和客户端主动建立连接的处理方式一样。

    
    var activeConnector = appServer as IActiveConnector;
    var task = activeConnector.ActiveConnect(remoteEndPoint);
    task.ContinueWith(
              t => Logger.InfoFormat("Client connected, SessionID: {0}", t.Result.Session.SessionID),
              TaskContinuationOptions.OnlyOnRanToCompletion);


## SuperSocket 服务器管理器 (ServerManager)

> [SuperSocket服务器管理器文档](SuperSocket-ServerManager "SuperSocket服务器管理器")


## 客户端安全证书验证

在 TLS/SSL 安全通信中, 客户端的安全证书不是必需的, 但是有些系统需要更高级别的安全保障. 因此有些用户提出了在服务器端验证客户端证书的需求. 现在在 SuperSocket 1.6 中, 这个新功能已经被加入了.

首先, 要启用客户端证书验证, 你需要在配置中的证书节点增加新的属性 "clientCertificateRequired":

    <certificate storeName="My"
				 storeLocation="LocalMachine"
                 clientCertificateRequired="true"
                 thumbprint="‎f42585bceed2cb049ef4a3c6d0ad572a6699f6f3"/>


然后你需要重写 AppServer 的方法 "ValidateClientCertificate(...)" 用于实现你的验证逻辑:

    protected override bool ValidateClientCertificate(YourSession session, object sender, X509Certificate certificate, X509Chain chain, SslPolicyErrors sslPolicyErrors)
    {
       //Check sslPolicyErrors

	   //Check certificate

       //Return checking result
       return true;
    }