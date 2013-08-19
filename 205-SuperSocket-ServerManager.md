# SuperSocket 服务管理器 (ServerManager)

> __关键字__: ServerManager, 服务管理器, 管理, 管理客户端, SuperSocket 监控

## 什么 SuperSocket 服务管理器?

SuperSocket 服务管理器是一个让你能够在客户中用图形化界面来管理和监控你的SuperSocket服务器程序的组件.


## 在服务器端配置服务器管理器

事实上, 服务器管理器是一个独立的 SuperSocket AppServer。 要让起作用，首先你要先确定下面的程序集在你的工作目录中存在:

- SuperSocket.ServerManager.dll (从源代码目录 "Management\Server" 编译)
- SuperSocket.WebSocket.dll (从源代码目录 "Protocols\WebSocket" 编译)

然后你需要把它和其它你要监控的服务器实例配置在一起:

	<superSocket isolation="Process">
		<servers>
		  <server name="ServerA"
		          serverTypeName="SampleServer"
		          ip="Any" port="2012">
		    <commandAssemblies>
		      <add assembly="SuperSocket.QuickStart.SampleServer.CommandAssemblyA"></add>
		      <add assembly="SuperSocket.QuickStart.SampleServer.CommandAssemblyB"></add>
		    </commandAssemblies>
		  </server>
		  <server name="ServerB"
		          serverTypeName="SampleServer"
		          ip="Any" port="2013">
		    <commandAssemblies>
		      <add assembly="SuperSocket.QuickStart.SampleServer.CommandAssemblyB"></add>
		      <add assembly="SuperSocket.QuickStart.SampleServer.CommandAssemblyC"></add>
		    </commandAssemblies>
		  </server>
		  <server name="ManagementServer"
		          serverType="SuperSocket.ServerManager.ManagementServer, SuperSocket.ServerManager">
		    <listeners>
		      <add ip="Any" port="4502" />
		    </listeners>
		    <users>
		      <user name="kerry" password="123456"/>
		    </users>
		  </server>
		</servers>
		<serverTypes>
		  <add name="SampleServer"
		       type="SuperSocket.QuickStart.ServerManagerSample.SampleServer, SuperSocket.QuickStart.ServerManagerSample" />
		</serverTypes>
	</superSocket>


在上面的配置中, ServerA 和 ServerB 是你要监控的普通服务器实例。另外，你需要加一个服务器类型为 "SuperSocket.ServerManager.ManagementServer, SuperSocket.ServerManager"的服务器实例节点。你可以看到，这个服务器实例下的子节点 "users" 定义了允许连接该服务器的用户名和密码。

如果你要用Silverlight客户端连接此服务器管理器，你还应该在配置中增加一个策略服务器节点:

    <server name="SilverlightPolicyServer"
              serverType="SuperSocket.Facility.PolicyServer.SilverlightPolicyServer, SuperSocket.Facility"
              ip="Any" port="943"
              receiveBufferSize="32"
              maxConnectionNumber="10"
              policyFile="Config\Silverlight.config"
              clearIdleSession="true">
    </server>


通常你不必关心策略服务器的状态，所以你最好把策略服务器的名字加入到管理器服务器配置的excludedServers属性中，这样，Silverlight策略服务器不会在服务器管理器客户端中显示。

    excludedServers="SilverlightPolicyServer"



# SuperSocket 服务器管理器客户端

SuperSocket 服务器管理器当前有两种类型的客户端, Silverlight客户端和WPF客户端。这两种客户端的代码都在源代码中的"Management"目录，你可以自行编译然后使用他们。

我们还提供了能够直接使用的在线的Silverlight服务器管理器客户端:

> [http://servermanager.supersocket.net/](http://servermanager.supersocket.net/ "http://servermanager.supersocket.net/")
> 

当你要从客户端连接SuperSocket服务器端的时候，你需要填写下面信息:

![SuperSocket ServerManager Client Configuration](images/servermanagerconfig.jpg)

    Name: 服务器在客户端的唯一标识;
    URI: 服务器管理器的侦听地址, 他是一个websocket访问地址 (以 "ws://" 或者 "wss://" 开头, 因为服务器管理器服务端和客户端通过websocket协议进行通信);
    User Name: 服务器管理器users子节点配置的用户名; 
    Password: 服务器管理器users子节点配置的密码; 


当连接建立成功后, 你将会看到 SuperSocket 服务器端的状态.

![SuperSocket ServerManager Client Show](images/servermanagershow.jpg)

你也可以在服务器管理器客户端中定制或启动服务器实例:

![SuperSocket ServerManager Client Control](images/servermanagercontrol.jpg)


# 安全性考虑

出于安全性考虑, 你可以为你的服务器管理器实例启用TLS/SSL传输层加密, 请阅读下面文档来了解如何操作:
> 
> [在SuperSocket中启用TLS/SSL传输层加密](Enable-TLS-SSL-trasnferring-layer-encryption-in-SuperSocket)


当你在服务器端启用TLS/SSL传输层加密之后, 你需要改用安全的websocket地址来连接服务器端:

> wss://***