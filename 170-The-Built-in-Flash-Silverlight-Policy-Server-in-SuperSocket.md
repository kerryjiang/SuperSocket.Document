# The Built in Flash Silverlight Policy Server in SuperSocket

**SuperSocket** contains a built-in Socket Policy Server for both Flash and Silverlight client. And it's implementation code is included in the assembly SuperSocket.Facility.dll. Thus, to enable the policy server, you need to make sure SuperSocket.Facility.dll exist in SuperSocket run directory firstly, and then add the policy server's configuration node in configuration file, like the following code.

**Flash Policy Server:**

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

**Silverlight Policy Server:**


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


**Note: the policyFile property in server node is your policy file stored path.**