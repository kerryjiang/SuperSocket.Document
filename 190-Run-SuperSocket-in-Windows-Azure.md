# 在 Windows Azure 中运行SuperSocket

> 关键字: Windows Azure, WorkRole, InputEndPoint, 云计算, 微软云

## 什么是 Windows Azure?

Windows Azure 是微软的云计算平台！微软的Windows Azure通过它的数据中心提供了按需分配的计算能力和存储空间用于在互联网上托管，扩展和管理应用程序。

这些在 Windows Azure 上运行的应用有很高的可靠性和可扩展能力。基于SuperSocket开发的服务器程序一样也能够很方便的运行在 Windows Azure 平台上。

## SuperSocket 配置
用于在 Windows Azure 上运行的配置文件 app.config 和直接运行的SuperSocket的配置文件相同。

    <?xml version="1.0" encoding="utf-8" ?>
    <configuration>
      <configSections>
        <section name="superSocket" type="SuperSocket.SocketEngine.Configuration.SocketServiceConfig, SuperSocket.SocketEngine"/>
      </configSections>
      <superSocket>
        <servers>
          <server name="RemoteProcessServer"
              serverTypeName="remoteProcess"
              ip="Any" port="2012" />
        </servers>
        <serverTypes>
          <add name="remoteProcess"
           type="SuperSocket.QuickStart.RemoteProcessService.RemoteProcessServer, SuperSocket.QuickStart.RemoteProcessService" />
        </serverTypes>
      </superSocket>
      <system.diagnostics>
        <trace>
          <listeners>
            <add type="Microsoft.WindowsAzure.Diagnostics.DiagnosticMonitorTraceListener, Microsoft.WindowsAzure.Diagnostics, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35"
            name="AzureDiagnostics">
            </add>
          </listeners>
        </trace>
      </system.diagnostics>
    </configuration>



## 添加SuperSocket的启动代码到 Windows Azure 的 WorkRole 项目
与其它SuperSocket程序相同，启动代码同样也要写到程序的入口处，如 Windows Azure 的 WorkRole 项目的OnStart() 方法：

    public override bool OnStart()
    {
        // Set the maximum number of concurrent connections 
        ServicePointManager.DefaultConnectionLimit = 100;

        // For information on handling configuration changes
        // see the MSDN topic at http://go.microsoft.com/fwlink/?LinkId=166357.

        m_Bootstrap = BootstrapFactory.CreateBootstrap();

        if (!m_Bootstrap.Initialize())
        {
            Trace.WriteLine("Failed to initialize SuperSocket!", "Error");
            return false;
        }

        var result = m_Bootstrap.Start();

        switch (result)
        {
            case (StartResult.None):
                Trace.WriteLine("No server is configured, please check you configuration!");
                return false;

            case (StartResult.Success):
                Trace.WriteLine("The server has been started!");
                break;

            case (StartResult.Failed):
                Trace.WriteLine("Failed to start SuperSocket server! Please check error log for more information!");
                return false;

            case (StartResult.PartialSuccess):
                Trace.WriteLine("Some server instances were started successfully, but the others failed to start! Please check error log for more information!");
                break;
        }

        return base.OnStart();
    }

## 配置和使用 Input Endpoint

由于Windows Azure的内部网络架构，你不能直接监听你配置中的IP和端口。在这种情况下，你需要在Windows Azure项目中配置Input Endpoint:

![endpoint](images/windowsazure.jpg)

这些 endpoint的命名规则是 "AppServerName_ConfiguredListenPort"。

例如，我们在配置文件中配置了名为"RemoteProcessServer"的server：

    <server name="RemoteProcessServer"
              serverTypeName="remoteProcess"
              ip="Any" port="2012" />

然后我们就应该创建一个名为"RemoteProcessServer_2012"的Endpoint。

在代码中，你可以通过编程的方式获得该Endpoint真正的端口号：

    var instanceEndpoint = RoleEnvironment.CurrentRoleInstance.InstanceEndpoints[serverConfig.Name + "_" + serverConfig.Port].Port;


但是你不用为SuperSocket这样做，因为 SuperSocket 已经实现了监听Endpoint的替换已经。 因此，你只需当SuperSocket的Bootstrap初始化的时候将Endpoint的字典传入就行了，最终代码如下：

    public override bool OnStart()
    {
        // Set the maximum number of concurrent connections 
        ServicePointManager.DefaultConnectionLimit = 100;

        // For information on handling configuration changes
        // see the MSDN topic at http://go.microsoft.com/fwlink/?LinkId=166357.

        m_Bootstrap = BootstrapFactory.CreateBootstrap();

        if (!m_Bootstrap.Initialize(RoleEnvironment.CurrentRoleInstance.InstanceEndpoints.ToDictionary(p => p.Key, p => p.Value.IPEndpoint)))
        {
            Trace.WriteLine("Failed to initialize SuperSocket!", "Error");
            return false;
        }

        var result = m_Bootstrap.Start();

        switch (result)
        {
            case (StartResult.None):
                Trace.WriteLine("No server is configured, please check you configuration!");
                return false;

            case (StartResult.Success):
                Trace.WriteLine("The server has been started!");
                break;

            case (StartResult.Failed):
                Trace.WriteLine("Failed to start SuperSocket server! Please check error log for more information!");
                return false;

            case (StartResult.PartialSuccess):
                Trace.WriteLine("Some server instances were started successfully, but the others failed to start! Please check error log for more information!");
                break;
        }

        return base.OnStart();
    }

最后，你就可以启动你的Windows Azure实例了。
