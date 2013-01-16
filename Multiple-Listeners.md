## Single listener
In the configuration below, you can configure the server instance's listening IP and port:

    <superSocket>
        <servers>
            <server name="TelnetServer"
                  serverType="SuperSocket.QuickStart.TelnetServer_StartByConfig.TelnetServer, SuperSocket.QuickStart.TelnetServer_StartByConfig"
                  ip="Any" port="2020">
           </server>
       </servers>
    </superSocket>

## Multiple listeners
You can add a child configuration node "listeners" to add more listening ip/port pairs:

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

In this case, the server instance "EchoServer" will listen two local endpoints. It is very similar with that a website can has many bindings in IIS.

You also can set different options for the different listeners:

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
