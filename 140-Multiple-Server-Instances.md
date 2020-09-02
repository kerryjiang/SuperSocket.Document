# 多服务器实例

> __关键字__: 多服务器实例, 多实例, 隔离, 多实例交互

## SuperSocket 支持在同一进程中运行多个服务器实例

一个 generic host (.NET Core) 可以运行多个 SuperSocket 服务器实例。每一个实例以 HostedService 的形式在 host 中运行。

你可以将多个服务器实例的配置定义在节点 "serverOptions" 下：

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

然后你需要告知服务器实例应该加载哪个配置。"serverName1" 和 "serverName2" 是两个服务器实例的名字，它们是配置节点 "serverOptions" 的两个自节点的名字。SuperSocketServiceA 和 SuperSocketServiceB 是两个服务器实例的类型，而且这两个类型不允许相同。

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

## 服务器实例的隔离

不同于先前版本的 SuperSocket，多服务器实例是以托管服务的形式在服务宿主中运行。因此他们运行在同一进程中。如果你想让他们运行在不同的进程或者不同的服务器中，我们建议您使用 Docker 或者其他容器技术。