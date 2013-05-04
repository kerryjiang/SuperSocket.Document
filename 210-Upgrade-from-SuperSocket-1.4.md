# Upgrade from SuperSocket 1.4

## Naming changes

* ICommandInfo => IRequestInfo
* ICommandInfo.Data => IRequestInfo.Body
* BinaryCommandInfo => BinaryRequestInfo
* StringCommandInfo => StringRequestInfo
* ICustomProtocol => IReceiveFilter
* AppSession.SendResponse(*) => AppSession.Send(*)
* AppSession.StartSession() => AppSession.OnSessionStarted()
* AppSession.HandleExceptionalError(Exception e) => AppSession.HandleException(Exception e)
* AppServer.OnAppSessionClosed(TAppSession session, CloseReason reason) => AppServer.OnSessionClosed(TAppSession session, CloseReason reason)

## Configuration changes

* Section name was changed: "socketServer" => "superSocket"

    Configuration of v1.4:

        <?xml version="1.0" encoding="utf-8" ?>
        <configuration>
            <configSections>
                <section name="socketServer" type="SuperSocket.SocketEngine.Configuration.SocketServiceConfig, SuperSocket.SocketEngine"/>
            </configSections>
            <appSettings>
                <add key="ServiceName" value="GPSSocketServer"/>
            </appSettings>
            <socketServer>
                ....
            </socketServer>
            <startup>
                <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.0" />
            </startup>
        </configuration>

    New configuration:

        <?xml version="1.0" encoding="utf-8" ?>
        <configuration>
            <configSections>
                <section name="superSocket" type="SuperSocket.SocketEngine.Configuration.SocketServiceConfig, SuperSocket.SocketEngine"/>
            </configSections>
            <appSettings>
                <add key="ServiceName" value="GPSSocketServer"/>
            </appSettings>
            <superSocket>
                ....
            </superSocket>
            <startup>
                <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.0" />
            </startup>
        </configuration>

* The attribute "mode"'s available values are changed from "Sync/Async/Udp" to "Tcp/Udp":

    v1.4:

        mode="Async" or mode="Sync" or mode="Udp"

    v1.5:

        mode="Tcp" or mode="Udp"


* The node "services" was changed to be "serverTypes":

    v1.4:

        <socketServer>
            ...
            <services>
                <service name="GPSSocketService"
                     type="SuperSocket.QuickStart.GPSSocketServer.GPSServer, SuperSocket.QuickStart.GPSSocketServer" />
            </services>
        </socketServer>

    New:

        <superSocket>
            ...
            <serverTypes>
                <add name="GPSSocketService"
                     type="SuperSocket.QuickStart.GPSSocketServer.GPSServer, SuperSocket.QuickStart.GPSSocketServer" />
            </serverTypes>
        </superSocket>

* The attribute "serviceName" of server node was changed to be "serverTypeName":

    v1.4:
    
        <server name="GPSSocketServer"
                serviceName="GPSSocketService"
                ip="Any" port="2012">
        </server>

    New:

        <server name="GPSSocketServer"
                serverTypeName="GPSSocketService"
                ip="Any" port="2012">
        </server>
        
## Logging API changes

* Logger.LogInfo(*) => Logger.Info(*);
* Logger.LogDebug(*) => Logger.Debug(*);
* Logger.LogError(*) => Logger.Error(*);

## New bootstrap

The SocketServerManager in v1.4:

    var serverConfig = ConfigurationManager.GetSection("socketServer") as SocketServiceConfig;

    if (!SocketServerManager.Initialize(serverConfig))
    {
        race.WriteLine("Failed to initialize SuperSocket!", "Error");
        return false;
    }

    if (!SocketServerManager.Start())
    {
        Trace.WriteLine("Failed to start SuperSocket!", "Error");
        return false;
    }

The new bootstrap:

    var bootstrap = BootstrapFactory.CreateBootstrap();

    if (!bootstrap.Initialize())
    {
        return false;
    }

    var result = bootstrap.Start();