# SuperSocket 中内置的 Flash/Silverlight 策略服务器

> 策略服务器, Flash策略服务器, Silverlight策略服务器, Policy Server, Flash Policy Server, Silverlight Policy Server

**SuperSocket** 包含一个可用于Flash和Silverlight的Socket策略服务器。 它被包含在SuperSocket.Facility.dll 这个程序集内。 因此，你要启用此策略服务器，你首先需要保证程序集SuperSocket.Facility.dll 只存在于SuperSocket的运行目录，然后在配置文件中增加策略服务器节点，配置代码如下：

**Flash 策略服务器:**

    <?xml version="1.0"?>
    <configuration>
        <configSections>
            <section name="superSocket" type="SuperSocket.SocketEngine.Configuration.SocketServiceConfig, SuperSocket.SocketEngine" />
        </configSections>
        <appSettings>
            <add key="ServiceName" value="SupperSocketService" />
        </appSettings>
        <superSocket>
            <servers>
                <server name="FlashPolicyServer"
                        serverType="SuperSocket.Facility.PolicyServer.FlashPolicyServer, SuperSocket.Facility"
                        ip="Any" port="843"
                        receiveBufferSize="32"
                        maxConnectionNumber="100"
                        policyFile="Policy\flash.xml"
                        clearIdleSession="true">
                </server>
            </servers>
        </superSocket>
        <startup>
            <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.0" />
        </startup>
    </configuration>

**Silverlight 策略服务器:**


    <?xml version="1.0"?>
    <configuration>
        <configSections>
            <section name="superSocket" type="SuperSocket.SocketEngine.Configuration.SocketServiceConfig, SuperSocket.SocketEngine" />
        </configSections>
        <appSettings>
            <add key="ServiceName" value="SupperSocketService" />
        </appSettings>
        <superSocket>
            <servers>
                <server name="SilverlightPolicyServer"
                        serverType="SuperSocket.Facility.PolicyServer.SilverlightPolicyServer, SuperSocket.Facility"
                        ip="Any" port="943"
                        receiveBufferSize="32"
                        maxConnectionNumber="100"
                        policyFile="Policy\silverlight.xml"
                        clearIdleSession="true">
                </server>
            </servers>
        </superSocket>
        <startup>
            <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.0" />
        </startup>
    </configuration>


**提示：server节点的配置属性policyFile的值为你的策略文件的存放路径。**