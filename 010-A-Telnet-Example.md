#  一个Telnet示例

> __关键字__: Telnet, 控制台示例, 创建项目，启动服务，停止服务，处理连接，处理请求

## 准备工作

确保你已经安装了最新的 .NET Core SDK (3.0 或更高版本)。

## 创建一个控制台项目然后引用SuperSocket (目标框架为netcoreapp3.0)

    dotnet new console
    dotnet add package SuperSocket.Server --version 2.0.0-*


## 添加SuperSocket和其它你可能需要使用的namespace的using 

    using System.Threading.Tasks;
    using Microsoft.Extensions.Hosting;
    using Microsoft.Extensions.Logging;
    using SuperSocket;
    using SuperSocket.ProtoBase;


## 在Main方法中书写SuperSocket宿主启动的方法

    static async Task Main(string[] args)
    {
        var host = ...;
        ...
        await host.RunAsync();
    }


### 创建宿主

    var host = SuperSocketHostBuilder
        .Create<StringPackageInfo, CommandLinePipelineFilter>()

用Package的类型和PipeLineFilter的类型创建SuperSocket宿主。


### 注册用于处理接收到的数据的包处理器


    .ConfigurePackageHandler(async (s, p) =>
    {
        var result = 0;

        switch (p.Key.ToUpper())
        {
            case ("ADD"):
                result = package.Parameters
                    .Select(p => int.Parse(p))
                    .Sum();
                break;

            case ("SUB"):
                result = package.Parameters
                    .Select(p => int.Parse(p))
                    .Aggregate((x, y) => x - y);
                break;

            case ("MULT"):
                result = package.Parameters
                    .Select(p => int.Parse(p))
                    .Aggregate((x, y) => x * y);
                break;
        }

        await s.SendAsync(Encoding.UTF8.GetBytes(result.ToString() + "\r\n"));
    })

将收到的文字发送给客户端。


### 配置日志

    .ConfigureLogging((hostCtx, loggingBuilder) =>
    {
        loggingBuilder.AddConsole();
    })

仅仅启用Console日志输出, 你也可以在此处注册你自己需要的第三方日志类库。


### 配置服务器如服务器名和监听端口等基本信息

    .ConfigureSuperSocket(options =>
    {
        options.Name = "Echo Server";
        options.Listeners = new [] {
            new ListenOptions
            {
                Ip = "Any",
                Port = 4040
            }
        };
    }).Build();


### 启动宿主

    await host.RunAsync();