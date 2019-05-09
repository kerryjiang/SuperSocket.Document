#  一个Telnet示例

> 关键字: Telnet, 控制台示例, 创建项目，启动服务，停止服务，处理连接，处理请求

## 创建一个控制台项目然后引用SuperSocket

	dotnet new console
	dotnet add package SuperSocket.Server --version 2.0.0-*


## 添加SuperSocket的namespace的using

	using SuperSocket;
	using SuperSocket.ProtoBase;
	using SuperSocket.Server;


## 在Main方法中书写SuperSocket宿主启动的方法

	static async Task Main(string[] args)
    {
		var host = ...;
		...
		await host.RunAsync();
	}


### 创建宿主

	var host = SuperSocketHostBuilder
		.Create<TextPackageInfo, LinePipelineFilter>()

用Package的类型和PipeLineFilter的类型创建SuperSocket宿主。


### 注册用于处理接收到的数据的包处理器


	.ConfigurePackageHandler(async (s, p) =>
	{
		await s.Channel.SendAsync(Encoding.UTF8.GetBytes(p.Text + "\r\n"));
	})

将收到的文字发送给客户端。


### 配置日志

	.ConfigureLogging((hostCtx, loggingBuilder) =>
	{
		loggingBuilder.AddConsole();
	})

仅仅启用Console日志输出。


### 配置服务器如服务器名和监听端口等基本信息

	.ConfigureServices((hostCtx, services) =>
	{
		services.Configure<ServerOptions>(options =>
		{
			options.Name = "Echo Server";
			options.Listeners = new [] {
				new ListenOptions
				{
					Ip = "Any",
					Port = 4040
				}
			};
		});
	}).Build();


### 启动宿主

	await host.RunAsync();