# 扩展服务器配置

当你使用 SuperSocket 实现 Socket 服务器的时候，不可避免的需要在配置文件中定义一些参数。 SuperSocket 提供了非常简单的方法，让你在配置文件中定义这些参数，然后在你的代码中读取它们。

请看下面的配置代码：

    <server name="FlashPolicyServer"
            serverType="SuperSocket.Facility.PolicyServer.FlashPolicyServer, SuperSocket.Facility"
            ip="Any" port="843"
            receiveBufferSize="32"
            maxConnectionNumber="100"
            clearIdleSession="true"
            policyFile="Policy\flash.xml">
    </server>

在上面的配置中， 属性 "policyFile" 未在 SuperSocket 中定义， 不过你任然可以在你的 AppServer 类中读取它：

    public class YourAppServer : AppServer
    {
        private string m_PolicyFile;

        protected override bool Setup(IRootConfig rootConfig, IServerConfig config)
        {  
            m_PolicyFile = config.Options.GetValue("policyFile");

            if (string.IsNullOrEmpty(m_PolicyFile))
            {
                if(Logger.IsErrorEnabled)
                    Logger.Error("Configuration option policyFile is required!");
                return false;
            }

            return true;
        }
    }

你不仅可以在 server 节点定义属性， 而且你还能像下面的代码那样定义子节点：

    <server name="SuperWebSocket"
            serverTypeName="SuperWebSocket"
            ip="Any" port="2011" mode="Tcp">
        <subProtocols>
            <!--Your configuration-->
        </subProtocols>
    </server>

下面是所需的节点对应的配置类:
    
    /// <summary>
    /// SubProtocol configuration
    /// </summary>
    public class SubProtocolConfig : ConfigurationElement
    {
        //Configuration attributes
    }
    /// <summary>
    /// SubProtocol configuation collection
    /// </summary>
    [ConfigurationCollection(typeof(SubProtocolConfig))]
    public class SubProtocolConfigCollection : ConfigurationElementCollection
    {
        //Configuration attributes
    }

然后你就能在 AppServer 类中读取这个子节点了：

    public class YourAppServer : AppServer
    {
        private SubProtocolConfigCollection m_SubProtocols;

        protected override bool Setup(IRootConfig rootConfig, IServerConfig config)
        {  
            m_SubProtocols = config.GetChildConfig<SubProtocolConfigCollection>("subProtocols");

            if (m_SubProtocols == null)
            {
                if(Logger.IsErrorEnabled)
                    Logger.Error("The child configuration node 'subProtocols' is required!");
                return false;
            }

            return true;
        }
    }