#  一个Telnet示例

## 创建一个控制台项目然后引用SuperSocket

1. 创建一个控制台应用程序项目。 由于该项目将要以服务器的形式运行，而却引用的SuperSocket并非通过Client Profile编译，所以 当项目创建好之后，你要修改此项目的目标框架从“Cllient Profile”到完整的框架。
2. 添加SuperSocket的dll文件(SuperSocket.Common.dll, SuperSocket.SocketBase.dll, SuperSocket.SocketEngine.dll)到此项目的引用。
3. 添加log4net.dll到项目引用。 如果是你使用默认的日志框架(log4net)，此步骤是必须的。 
4. 引用SuperSocket提供的日志配置文件log4net.config到项目文件夹的"Config"文件夹然后设置它的Build Action 为 "Content"，设置它的Copy to Output Directory 为 "Copy if newer"，因为这个配置文件是log4net需要的

![Telnet Project](images/telnetproject.jpg)


## 服务器启动和停止代码

	static void Main(string[] args)
	{
		Console.WriteLine("Press any key to start the server!");

		Console.ReadKey();
		Console.WriteLine();

		var appServer = new AppServer();

		//Setup the appServer
		if (!appServer.Setup(2012)) //Setup with listening port
		{
			Console.WriteLine("Failed to setup!");
			Console.ReadKey();
			return;
		}

		Console.WriteLine();

		//Try to start the appServer
		if (!appServer.Start())
		{
			Console.WriteLine("Failed to start!");
			Console.ReadKey();
			return;
		}

		Console.WriteLine("The server started successfully, press key 'q' to stop it!");

		while (Console.ReadKey().KeyChar != 'q')
		{
			Console.WriteLine();
			continue;
		}

		//Stop the appServer
		appServer.Stop();

		Console.WriteLine("The server was stopped!");
		Console.ReadKey();
	}



## 处理连接

1. 注册回话新建事件处理方法
	

		appServer.NewSessionConnected += new SessionHandler<AppSession>(appServer_NewSessionConnected);
	

2. 在事件处理代码中发送欢迎信息给客户端


		static void appServer_NewSessionConnected(AppSession session)
		{
			session.Send("Welcome to SuperSocket Telnet Server");
		}
	
3. 使用Telnet客户端进行测试

	1. open a telnet client
	2. type "telnet localhost 2012" ending with an "ENTER"
	3. you will get the message "Welcome to SuperSocket Telnet Server"

## 处理请求

1. 注册请求处理方法	
	
		appServer.NewRequestReceived += new RequestHandler<AppSession, StringRequestInfo>(appServer_NewRequestReceived);



2. 实现请求处理
	
		static void appServer_NewRequestReceived(AppSession session, StringRequestInfo requestInfo)
		{
			switch (requestInfo.Key.ToUpper())
			{
				case("ECHO"):
					session.Send(requestInfo.Body);
					break;

				case ("ADD"):
					session.Send(requestInfo.Parameters.Select(p => Convert.ToInt32(p)).Sum().ToString());
					break;

				case ("MULT"):

					var result = 1;

					foreach (var factor in requestInfo.Parameters.Select(p => Convert.ToInt32(p)))
					{
						result *= factor;
					}

					session.Send(result.ToString());
					break;
			}
		}

	requestInfo.Key 是请求的命令行用空格分隔开的第一部分

    requestInfo.Parameters 是用空格分隔开的其余部分

3. 通过Telnet客户端进行测试

	你可以打开telnet客户端去验证以上代码。

	当然和服务器端建立连接之后，你可以通过下面的方式与服务器端交互("C:"之后的信息代表客户端的请求，"S:"之后的信息代表服务器端的响应):

		C: ECHO ABCDEF
		S: ABCDEF
		C: ADD 1 2
		S: 3
		C: ADD 250 250
		S: 500
		C: MULT 2 8
		S: 16
		C: MULT 125 2
		S: 250


## Command的用法
在本文档的前半部分，你可能已经了解到了如何在SuperSocket处理客户端请求。 但是同时你可能会发现一个问题，如果你的服务器端包含有很多复杂的业务逻辑，这样的switch/case代码将会很长而且非常难看，并且没有遵循OOD的原则。
在这种情况下，SuperSocket提供了一个让你在多个独立的类中处理各自不同的请求的命令框架。

例如，你可以定义一个名为"ADD"的类去处理Key为"ADD"的请求:

	public class ADD : CommandBase<AppSession, StringRequestInfo>
    {
        public override void ExecuteCommand(AppSession session, StringRequestInfo requestInfo)
        {
            session.Send(requestInfo.Parameters.Select(p => Convert.ToInt32(p)).Sum().ToString());
        }
    }
	
定义一个名为"MULT"的类去处理Key为"MULT"的请求:

	public class MULT : CommandBase<AppSession, StringRequestInfo>
    {
        public override void ExecuteCommand(AppSession session, StringRequestInfo requestInfo)
        {
            var result = 1;

            foreach (var factor in requestInfo.Parameters.Select(p => Convert.ToInt32(p)))
            {
                result *= factor;
            }

            session.Send(result.ToString());
        }
    }
 
 同时你要移除请求处理方法的注册，因为它和命令不能同时被支持：
 
	//Remove this line
	appServer.NewRequestReceived += new RequestHandler<AppSession, StringRequestInfo>(appServer_NewRequestReceived);
