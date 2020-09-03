# 与 ASP.Net Core 网站 和 ABP 框架集成

> __关键字__: ASP.NET Core, ABP, 集成

## 与 ASP.Net Core 网站框架集成

是的，SuperSocket 可以和 ASP.NET Core 网站一起同时运行。你需要做的是将 SuperSocket 注册到 ASP.NET Core 网站的host builder中去, 同时将服务器的选项放到配置文件中或者通过代码定义。

在 Program 这个类中，添加关于 SuperSocket 的代码：


        //don't forget the usings
        using SuperSocket;
        using SuperSocket.ProtoBase;

        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>();
                })
                .AsSuperSocketHostBuilder<TextPackageInfo, LinePipelineFilter>()
                .UsePackageHandler(async (s, p) =>
                {
                    // echo message back to client
                    await s.SendAsync(Encoding.UTF8.GetBytes(p.Text + "\r\n"));
                });


同时将服务器的配置选项放到配置文件 "appsettings.json" 中去：


        {
            "Logging": {
                "LogLevel": {
                "Default": "Information",
                "Microsoft": "Warning",
                "Microsoft.Hosting.Lifetime": "Information"
                }
            },
            "serverOptions": {
                "name": "TestServer",
                "listeners": [
                    {
                        "ip": "Any",
                        "port": 4040
                    }
                ]
            },
            "AllowedHosts": "*"
        }

## 与 ABP 框架集成

即将到来...