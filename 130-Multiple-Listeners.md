# 多个 Listener

> __关键字__: 多个端口, 多个监听, Listener, 多个Listener

## 单个 listener
在下面的配置中，你可以配置服务器监听 IP 和端口：

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

## 多个 listeners
你也可以在配置节点 "listeners" 下添加多个元素：

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

在此例中，服务器实例 "EchoServer" 将会侦听两个本地端口。这个类似于在 IIS 中的网站可以有多个绑定。

你也可以给不同的 listener 配置不同的属性：

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


除了在配置中定义 listener， SuperSocket 2.0 还允许你通过代码的方式添加 listener：

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