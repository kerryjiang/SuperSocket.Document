# 多个 Listener

> 多个端口, 多个监听, Listener, 多个Listener

## 单个 listener
在下面的配置中，你可以配置服务器的监听 ip/port：

    <superSocket>
        <servers>
            <server name="TelnetServer"
                  serverType="SuperSocket.QuickStart.TelnetServer_StartByConfig.TelnetServer, SuperSocket.QuickStart.TelnetServer_StartByConfig"
                  ip="Any" port="2020">
           </server>
       </servers>
    </superSocket>

## 多个 listener
你可以增加一个子节点 "listeners" 用于添加多对监听 ip/port：

    <superSocket>
      <servers>
        <server name="EchoServer" serverTypeName="EchoService">
          <listeners>
            <add ip="127.0.0.2" port="2012" />
            <add ip="IPv6Any" port="2012" />
          </listeners>
        </server>
      </servers>
      <serverTypes>
        <add name="EchoService"
             type="SuperSocket.QuickStart.EchoService.EchoServer, SuperSocket.QuickStart.EchoService" />
      </serverTypes>
    </superSocket>

在这种情况下，服务器实例将监听两个本地端口。 这个功能和 IIS 站点的多绑定功能非常相似。

你也可以给不同的监听器设置不同的选项：

    <superSocket>
        <servers>
            <server name="EchoServer" serverTypeName="EchoService">
                <certificate filePath="localhost.pfx" password="supersocket"></certificate>
                <listeners>
                    <add ip="Any" port="80" />
                    <add ip="Any" port="443" security="tls" />
                </listeners>
            </server>
        </servers>
        <serverTypes>
            <add name="EchoService"
                 type="SuperSocket.QuickStart.EchoService.EchoServer, SuperSocket.QuickStart.EchoService" />
        </serverTypes>
    </superSocket>
