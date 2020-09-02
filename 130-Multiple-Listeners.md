# Multiple Listeners

> __Keywords__: Multiple Listeners, Multiple Port, Multiple Endpoints, Multiple Listeners Configuration, IP, Port

## Single listener
In the configuration below, you can configure the server instance's listening IP and port:

    {
        "serverOptions": {
            "name": "EchoServer",
            "listeners": [
                {
                    "ip": "Any",
                    "port": "2020"
                }
            ]
        }
    }

## Multiple listeners
You also can add more than one element under the configuration node "listeners":

    {
        "serverOptions": {
            "name": "EchoServer",
            "listeners": [
                {
                    "ip": "Any",
                    "port": "2020"
                },
                {
                    "ip": "192.168.3.1",
                    "port": "2020"
                }
            ]
        }
    }

In this case, the server instance "EchoServer" will listen two local endpoints. It is very similar with that a website can have multiple bindings in IIS.

You also can set different options for the different listeners:

    {
        "serverOptions": {
            "name": "EchoServer",
            "listeners": [
                {
                    "ip": "Any",
                    "port": 80
                },
                {
                    "ip": "Any",
                    "port": 443,
                    "security": "Tls12",
                    "certificateOptions": {
                        "filePath": "supersocket.pfx",
                        "password": "supersocket"
                    }
                }
            ]
        }
    }


In addition to define listeners in configuration, SuperSocket 2.0 also allow you to add listener programmatically:

    var host = SuperSocketHostBuilder.Create<TextPackageInfo, LinePipelineFilter>(args)
        .ConfigureSuperSocket(options =>
        {
            options.AddListener(new ListenOptions
                {
                    Ip = "Any",
                    Port = 4040
                }
            );
        }).Build();

    await host.RunAsync();