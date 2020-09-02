# Multiple Server Instances

> __Keywords__: Multiple Server Instances, Multiple Server Configuration, Server Dispatch, Isolation

## SuperSocket support running multiple server instances in the same process

A generic host (.NET Core) can run multiple SuperSocket servver instances. Each of them is a HostedService within the host.

You can leave the options of the multiple servers under the node "serverOptions" in the configuration:

    {
        "serverOptions": {
            "TestServer1": {
                "name": "TestServer1",
                "listeners": [
                    {
                        "ip": "Any",
                        "port": 4040
                    }
                ]
            },
            "TestServer2": {
                "name": "TestServer2",
                "listeners": [                
                    {
                        "ip": "Any",
                        "port": 4041
                    }
                ]
            }
        }
    }

And then you should tell which server configuration node each server should load. The "serverName1"  and "serverName2" are the two servers' names which are defined in the configuration as the keys of two children of the node "serverOptions". SuperSocketServiceA and SuperSocketServiceB are the service types of these two server and they are not allowed to be the same type.

    var hostBuilder = MultipleServerHostBuilder.Create()
        .AddServer<SuperSocketServiceA, TextPackageInfo, LinePipelineFilter>(builder =>
        {
            builder
            .ConfigureServerOptions((ctx, config) =>
            {
                return config.GetSection("serverName1");
            });
        })
        .AddServer<SuperSocketServiceB, TextPackageInfo, LinePipelineFilter>(builder =>
        {
            builder
            .ConfigureServerOptions((ctx, config) =>
            {
                return config.GetSection("serverName2");
            });
        })



## Isolation of the server instances

Different with the previous version of SuperSocket, multiple servers run as hosted service within one generic host. So they run in the same host and same process. If you want to run them in different processes or different servers, we recommend you to use Docker or other container technology.