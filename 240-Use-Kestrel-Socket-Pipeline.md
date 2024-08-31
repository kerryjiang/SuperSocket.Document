# 使用 ASP.NET Core 中的 Kestrel pipeline 来进行 socket 通信

> __Keywords__: Kestrel, Pipeline

用户可以通过使用 KestrelPipeConnection 在 SuperSocket 中获得和 ASP.NET Kestrel 一样的通信性能。


## 引用 nuget 包 SuperSocket.Kestrel

dotnet cli

```
dotnet add package SuperSocket.Kestrel
```

## 使用 Kestrel connection factory

    var host = SuperSocketHostBuilder.Create<StringPackageInfo, CommandLinePipelineFilter>()
        .UsePackageHandler(async (s, p) =>
        {
            // handle packages
        })
        .UseKestrelPipeConnection()
        .Build();

    await host.RunAsync();