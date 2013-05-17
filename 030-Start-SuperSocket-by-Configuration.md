# Start SuperSocket by Configuration

> __Keywords__: Start by Configuration, Configuration, Bootstrap, Windows Service

## Why Start Server by Configuration
1. Avoid hard coding
2. SuperSocket provides lots of useful configuration options
3. Take full use of the tools of SuperSocket

## How to Start Server by Configuration with Bootstrap
* SuperSocket configuration section
    SuperSocket uses .NET default configuration technology, a configuration section designed for SuperSocket:

        <configSections>
            <section name="superSocket"
                 type="SuperSocket.SocketEngine.Configuration.SocketServiceConfig, SuperSocket.SocketEngine" />
        </configSections>

* Server instance configuration

        <superSocket>
            <servers>
              <server name="TelnetServer"
                  serverType="SuperSocket.QuickStart.TelnetServer_StartByConfig.TelnetServer, SuperSocket.QuickStart.TelnetServer_StartByConfig"
                  ip="Any" port="2020">
              </server>
            </servers>
        </superSocket>

    Now, I explain the configuration of server node here:

        name: the name of appServer instance
        serverType: the full name of the AppServer your want to run
        ip: listen ip
        port: listen port

    We'll have the full introduction about the configuration in next document.

* Start SuperSocket using BootStrap

        static void Main(string[] args)
        {
            Console.WriteLine("Press any key to start the server!");

            Console.ReadKey();
            Console.WriteLine();

            var bootstrap = BootstrapFactory.CreateBootstrap();

            if (!bootstrap.Initialize())
            {
                Console.WriteLine("Failed to initialize!");
                Console.ReadKey();
                return;
            }

            var result = bootstrap.Start();

            Console.WriteLine("Start result: {0}!", result);

            if (result == StartResult.Failed)
            {
                Console.WriteLine("Failed to start!");
                Console.ReadKey();
                return;
            }

            Console.WriteLine("Press key 'q' to stop it!");

            while (Console.ReadKey().KeyChar != 'q')
            {
                Console.WriteLine();
                continue;
            }

            Console.WriteLine();

            //Stop the appServer
            bootstrap.Stop();

            Console.WriteLine("The server was stopped!");
            Console.ReadKey();
        }

* Some configurations samples

1. Server types nodes:

            <superSocket>
                <servers>
                  <server name="TelnetServer"
                      serverTypeName="TelnetServer"
                      ip="Any" port="2020">
                  </server>
                </servers>
                <serverTypes>
                    <add name="TelnetServer" type="SuperSocket.QuickStart.TelnetServer_StartByConfig.TelnetServer, SuperSocket.QuickStart.TelnetServer_StartByConfig"/>
                </serverTypes>
            </superSocket>


2. Multiple server instances:

            <superSocket>
                <servers>
                  <server name="TelnetServerA"
                      serverTypeName="TelnetServer"
                      ip="Any" port="2020">
                  </server>
                  <server name="TelnetServerB"
                      serverTypeName="TelnetServer"
                      ip="Any" port="2021">
                  </server>
                </servers>
                <serverTypes>
                    <add name="TelnetServer" type="SuperSocket.QuickStart.TelnetServer_StartByConfig.TelnetServer, SuperSocket.QuickStart.TelnetServer_StartByConfig"/>
                </serverTypes>
              </superSocket>

## SuperSocket.SocketService.exe, the Running Container provided by SuperSocket

* Use SuperSocket.SocketService.exe directly

1. make sure all assemblies required by your server are in the same directory with SuperSocket.SocketService.exe
2. put your SuperSocket configuration node in the file SuperSocket.SocketService.exe.config
3. run "SuperSocket.SocketService.exe" directly, your defined server will run

* Install SuperSocket.SocketService.exe as windows service

You can install SuperSocket.SocketService.exe as windows service by running it with an extra command line parameter "-i":

    SuperSocket.SocketSerrvice.exe -i


The windows service name is defined in configuration file, you can change it as your requirement:

    <appSettings>
        <add key="ServiceName" value="SupperSocketService" />
    </appSettings>


The service also can be uninstalled by the parameter "-u":

    SuperSocket.SocketSerrvice.exe -u