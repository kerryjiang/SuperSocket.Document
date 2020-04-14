# WebSocket 服务器

> __关键字__: WebSocket, 服务器

## 创建一个 WebSocket Server

首先，你需要先引用这个包 SuperSocket.WebSocket.Server

    dotnet add package SuperSocket.WebSocket.Server --version 2.0.0-*

然后添加下面的 using 语句

    using SuperSocket.WebSocket.Server;


让我创建这个 WebSocket 服务器，这个服务器将把收到的消息再发送回客户端：


    var host = WebSocketHostBuilder.Create()
        .ConfigureWebSocketMessageHandler(
            async (session, message) =>
            {
                await session.SendAsync(message.Message);
            }
        )
        .ConfigureAppConfiguration((hostCtx, configApp) =>
        {
            configApp.AddInMemoryCollection(new Dictionary<string, string>
            {
                { "serverOptions:name", "TestServer" },
                { "serverOptions:listeners:0:ip", "Any" },
                { "serverOptions:listeners:0:port", "4040" }
            });
        })
        .ConfigureLogging((hostCtx, loggingBuilder) =>
        {
            loggingBuilder.AddConsole();
        })
        .Build();

    await host.RunAsync();



## 定义命令来处理消息

定义一个命令

    class ADD : IAsyncCommand<WebSocketSession, StringPackageInfo>
    {
        public async ValueTask ExecuteAsync(WebSocketSession session, StringPackageInfo package)
        {
            var result = package.Parameters
                .Select(p => int.Parse(p))
                .Sum();

            await session.SendAsync(result.ToString());
        }
    }

注册这个命令

    builder
        .UseCommand<StringPackageInfo, StringPackageConverter>(commandOptions =>
        {
            // register commands one by one
            commandOptions.AddCommand<ADD>();
        });


类型参数 StringPackageConverter 是能够将 WebSocketPackage 转化成你自己的应用程序包的转化器的类型。

    class StringPackageConverter : IPackageMapper<WebSocketPackage, StringPackageInfo>
    {
        public StringPackageInfo Map(WebSocketPackage package)
        {
            var pack = new StringPackageInfo();
            var arr = package.Message.Split(' ', 3, StringSplitOptions.RemoveEmptyEntries);
            pack.Key = arr[0];
            pack.Parameters = arr.Skip(1).ToArray();
            return pack;
        }
    }