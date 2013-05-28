# Multiple Server Instances

> 关键字: 多服务器实例, 多实例, 隔离, 多实例交互

## SuperSocket 支持在统一进程中运行多个服务器实例

多个服务器实例可以是同一个服务器类型:

    <superSocket>
        <servers>
          <server name="EchoServerA" serverTypeName="EchoService">
                <listeners>
                  <add ip="Any" port="80" />
                </listeners>
          </server>
          <server name="EchoServerB" serverTypeName="EchoService" security="tls">
              <certificate filePath="localhost.pfx" password="supersocket"></certificate>
              <listeners>
                  <add ip="Any" port="443" />
              </listeners>
          </server>
        </servers>
        <serverTypes>
          <add name="EchoService"
               type="SuperSocket.QuickStart.EchoService.EchoServer, SuperSocket.QuickStart.EchoService" />
        </serverTypes>
    </superSocket>

也可以是不同的服务器类型:

    <superSocket>
        <servers>
            <server name="ServerA"
                    serverTypeName="MyAppServerA"
                    ip="Any" port="2012">
            </server>
            <server name="ServerB"
                    serverTypeName="MyAppServerB"
                    ip="Any" port="2013">
            </server>
        </servers>
        <serverTypes>
            <add name="MyAppServerA"
                 type="SuperSocket.QuickStart.MultipleAppServer.MyAppServerA, SuperSocket.QuickStart.MultipleAppServer" />
            <add name="MyAppServerB"
                 type="SuperSocket.QuickStart.MultipleAppServer.MyAppServerB, SuperSocket.QuickStart.MultipleAppServer" />
        </serverTypes>
    </superSocket>

## 服务器实例的隔离级别
前面提到过, 在 SuperSocket 配置的根节点有这样一个属性:

    <superSocket isolation="AppDomain">//None, AppDomain, Process
        ....
    </superSocket>

如果这个isolation属性的值是 'None' (默认值), 这些服务器实例将会在同一进程的同一 AppDomain 中运行. 因策他们能够直接互相访问. (我们将会在后面讨论这一点)

但是如果isolation属性的值是 'AppDomain', SuperSocket将为每个实例创建独立的AppDomai, 所有的服务器实例都会运行在各自独立的AppDomain之中。

但是如果isolation属性的值是 'Process', SuperSocket将为每个实例创建独立的进程, 所有的服务器实例都会运行在各自独立的进程之中。

## 多服务器实例之间的交互
前面一部分提到了, 如果 isolation 设成 'None', 多服务器实例之间的交互是非常简单的事情.

举个例子, 他们能通过 SuperSocket 提供的 Bootstap 来彼此访问:
    
    interface IDespatchServer
    {
        void DispatchMessage(string sessionKey, string message);
    }
    
    public class MyAppServerB : AppServer, IDespatchServer
    {
        public void DispatchMessage(string sessionKey, string message)
        {
            var session = GetAppSessionByID(sessionKey);
            if (session == null)
                return;

            session.Send(message);
        }
    }

    public class MyAppServerA : AppServer
    {
        private IDespatchServer m_DespatchServer;

        protected override void OnStartup()
        {
            m_DespatchServer = this.Bootstrap.GetServerByName("ServerB") as IDespatchServer;
            base.OnStartup();
        }

        internal void DespatchMessage(string targetSessionKey, string message)
        {
            m_DespatchServer.DispatchMessage(targetSessionKey, message);
        }
    }

上面的示例代码演示了如何从一个服务器实例转发一条消息到另一个服务器实例一个 Session。