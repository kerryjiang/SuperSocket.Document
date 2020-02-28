# WebSocketServer

> __Keywords__: WebSocket

## Create a WebSocket Server

At first, you should reference the package SuperSocket.WebSocket.Server

    dotnet add package SuperSocket.WebSocket.Server --version 2.0.0-*

Then add the using statement

    using SuperSocket.WebSocket.Server;


Let's create the WebSocket server. This server just echo messages back to the client


    var host = WebSocketHostBuilder.Create();
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



## Define commands to handle messages

Define at least one command

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

Register the command

    builder
        .UseCommand<StringPackageInfo, StringPackageConverter>(commandOptions =>
        {
            // register commands one by one
            commandOptions.AddCommand<ADD>();
        });


The type parameter StringPackageConverter is the type which can convert WebSocketPackage to your application package.