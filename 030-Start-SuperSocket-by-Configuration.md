# 通过配置启动 SuperSocket

> __关键字__: 通过配置启动, 配置

## SuperSocket的配置文件

和 Asp.Net Core 一样，SuperSocket 使用 JSON 配置文件 appsettings.json。 我们只需要将该文件放到应用程序的根目录然后确保它能够被复制到输出目录就行。

* appsettings.json
* appsettings.Development.json  // for Development environment
* appsettings.Production.json // for Production environment

## 如何通过配置文件启动 SuperSocket

事实上，我们除了书写正常的启动代码之外就不需要做任何其它事情了。

    var host = SuperSocketHostBuilder.Create<StringPackageInfo, CommandLinePipelineFilter>()
        .UsePackageHandler(async (s, p) =>
        {
            // handle packages
        })
        .ConfigureLogging((hostCtx, loggingBuilder) =>
        {
            loggingBuilder.AddConsole();
        })
        .Build();

    await host.RunAsync();


## 配置文件的格式 (appsettings.json)

It is a sample:

    {
        "serverOptions": {
            "name": "GameMsgServer",
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

配置项目:

* name: 服务器的名称;
* maxPackageLength: 此服务器允许的最大的包的大小; 默认4M;
* receiveBufferSize: 接收缓冲区的大小; 默认4k;
* sendBufferSize: 发送缓冲区的大小; 默认4k;
* receiveTimeout: 接收超时时间; 微秒为单位;
* sendTimeout: 发送超时的事件; 微秒为单位;
* listeners: 服务器的监听器;
* listeners/*/ip: 监听IP; Any: 所有 ipv4 地址, IPv6Any: 所有 ipv6 地址, 其它具体的IP地址;
* listeners/*/port: 监听端口;
* listeners/*/backLog: 连接等待队列的最大长度;
* listeners/*/noDelay: 定义 Socket 是否启用 Nagle 算法;
* listeners/*/security: None/Ssl3/Tls11/Tls12/Tls13; 传输层加密所使用的TLS协议版本号;
* listeners/*/certificateOptions: 用于TLS加密/揭秘的证书的配置项目;