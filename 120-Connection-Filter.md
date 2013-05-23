# 连接过滤器

SuperSocket 中的连接过滤器是一个用于过滤客户端连接的接口。 通过连接过滤器，你可以允许或者禁止从某些地址过来的连接。

连接过滤器的接口定义如下：

    /// <summary>
    /// The basic interface of connection filter
    /// </summary>
    public interface IConnectionFilter
    {
        /// <summary>
        /// Initializes the connection filter
        /// </summary>
        /// <param name="name">The name.</param>
        /// <param name="appServer">The app server.</param>
        /// <returns></returns>
        bool Initialize(string name, IAppServer appServer);

        /// <summary>
        /// Gets the name of the filter.
        /// </summary>
        string Name { get; }

        /// <summary>
        /// Whether allows the connect according the remote endpoint
        /// </summary>
        /// <param name="remoteAddress">The remote address.</param>
        /// <returns></returns>
        bool AllowConnect(IPEndPoint remoteAddress);
    }


**bool Initialize(string name, IAppServer appServer);**

此方法用于初始化连接过滤器

**string Name {get;}**

返回连接过滤器名称

**bool AllowConnect (IPEndPoint remoteAddress);**

这个方法通过判断客户端的地址来决定是否允许该连接。


**下面的代码实现了只允许来自指定IP地址范围的连接的功能:**

    public class IPConnectionFilter : IConnectionFilter
    {
        private Tuple<long, long>[] m_IpRanges;

        public bool Initialize(string name, IAppServer appServer)
        {
            Name = name;

            var ipRange = appServer.Config.Options.GetValue("ipRange");

            string[] ipRangeArray;

            if (string.IsNullOrEmpty(ipRange)
                || (ipRangeArray = ipRange.Split(new char[] { ',', ';' }, StringSplitOptions.RemoveEmptyEntries)).Length <= 0)
            {
                throw new ArgumentException("The ipRange doesn't exist in configuration!");
            }

            m_IpRanges = new Tuple<long, long>[ipRangeArray.Length];

            for (int i = 0; i < ipRangeArray.Length; i++)
            {
                var range = ipRangeArray[i];
                m_IpRanges[i] = GenerateIpRange(range);
            }

            return true;
        }

        private Tuple<long, long> GenerateIpRange(string range)
        {
            var ipArray = range.Split(new char[] { '-' }, StringSplitOptions.RemoveEmptyEntries);

            if(ipArray.Length != 2)
                throw new ArgumentException("Invalid ipRange exist in configuration!");

            return new Tuple<long, long>(ConvertIpToLong(ipArray[0]), ConvertIpToLong(ipArray[1]));
        }

        private long ConvertIpToLong(string ip)
        {
            var points = ip.Split(new char[] { '.' }, StringSplitOptions.RemoveEmptyEntries);

            if(points.Length != 4)
                throw new ArgumentException("Invalid ipRange exist in configuration!");

            long value = 0;
            long unit = 1;

            for (int i = points.Length - 1; i >= 0; i--)
            {
                value += unit * points[i].ToInt32();
                unit *= 256;
            }

            return value;
        }

        public string Name { get; private set; }

        public bool AllowConnect(IPEndPoint remoteAddress)
        {
            var ip = remoteAddress.Address.ToString();
            var ipValue = ConvertIpToLong(ip);

            for (var i = 0; i < m_IpRanges.Length; i++)
            {
                var range = m_IpRanges[i];

                if (ipValue > range.Item2)
                    return false;

                if (ipValue < range.Item1)
                    return false;
            }

            return true;
        }
    }

**然后你需要更新配置文件来气用这个连接过滤器:**

1) 添加配置节点 "connectionFilters":

    <connectionFilters>
      <add name="IpRangeFilter"
           type="SuperSocket.QuickStart.ConnectionFilter.IPConnectionFilter, SuperSocket.QuickStart.ConnectionFilter" />
    </connectionFilters>


2) 为server实例添加connectionFilter属性:

    <server name="EchoServer"
            serverTypeName="EchoService" ip="Any" port="2012"
            connectionFilter="IpRangeFilter"
            ipRange="127.0.1.0-127.0.1.255">
    </server>

3) 完整的配置文件如下:

    <?xml version="1.0" encoding="utf-8" ?>
    <configuration>
        <configSections>
            <section name="superSocket" type="SuperSocket.SocketEngine.Configuration.SocketServiceConfig, SuperSocket.SocketEngine"/>
        </configSections>
        <appSettings>
            <add key="ServiceName" value="EchoService"/>
        </appSettings>
        <superSocket>
            <servers>
                <server name="EchoServer"
                    serverTypeName="EchoService"
                    ip="Any" port="2012"
                    connectionFilter="IpRangeFilter"
                    ipRange="127.0.1.0-127.0.1.255">
                </server>
            </servers>
           <serverTypes>
               <add name="EchoService"
                    type="SuperSocket.QuickStart.EchoService.EchoServer, SuperSocket.QuickStart.EchoService" />
           </serverTypes>
           <connectionFilters>
               <add name="IpRangeFilter"
                    type="SuperSocket.QuickStart.ConnectionFilter.IPConnectionFilter, SuperSocket.QuickStart.ConnectionFilter" />
           </connectionFilters>
        </superSocket>
        <startup>
            <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.0" />
        </startup>
    </configuration>
    