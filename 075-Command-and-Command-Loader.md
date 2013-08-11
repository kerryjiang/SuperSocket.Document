# 命令和命令加载器

> __关键字__: 命令, 命令加载器, 多命令程序集

## 命令 (Command)
SuperSocket 中的命令设计出来是为了处理来自客户端的请求的, 它在业务逻辑处理之中起到了很重要的作用。

命令类必须实现下面的基本命令接口:

    public interface ICommand<TAppSession, TRequestInfo> : ICommand
        where TRequestInfo : IRequestInfo
        where TAppSession : IAppSession
    {

        void ExecuteCommand(TAppSession session, TRequestInfo requestInfo);
    }

    public interface ICommand
    {
        string Name { get; }
    }


请求处理代码必须被放置于方法 "ExecuteCommand(TAppSession session, TRequestInfo requestInfo)" 之中，并且属性 "Name" 的值用于匹配接收到请求实例(requestInfo)的Key。当一个请求实例(requestInfo) 被收到时，SuperSocket 将会通过匹配请求实例(requestInfo)的Key和命令的Name的方法来查找用于处理该请求的命令。

举个例子, 如果你收到如下请求(requestInfo):

    Key: "ADD"
    Body: "1 2"

于是 SuperSocket 将会寻找Name属性为"ADD"的命令。如果有个命令定义如下:

    public class ADD : StringCommandBase
    {
		public void ExecuteCommand(AppSession session, StringRequestInfo requestInfo)
        {
              session.Send((int.Parse(requestInfo[0] + int.Parse(requestInfo[1])).ToString());
        }
    }

因为基类 StringCommandBase 会设置Name属性的值为此类的名称(ADD)，因此个命令将会被找到.

但是在有些情况, 请求实例(requestInfo)的Key 无法当做类的名称。 比如说:

    Key: "01"
    Body: "1 2"

为了让让你的 ADD 命令起作用，你需要为命令类重写Name属性:

    public class ADD : StringCommandBase
    {
        public override string Name
        {
            get { return "01"; }
        }

		public void ExecuteCommand(AppSession session, StringRequestInfo requestInfo)
        {
              session.Send((int.Parse(requestInfo[0] + int.Parse(requestInfo[1])).ToString());
        }
    }


## 名称程序集定义

是的，SuperSocket是用反射来查找哪些公开的类实现了基本的命令接口，但是它只在你的AppServer类定义的程序集中查找。

举例来说, 你的 AppServer 定义在程序集 GameServer.dll 中, 但是你的 ADD 命令是定义在程序集 BasicModules.dll 中:

    GameServer.dll
    MyGameServer.cs

>

    BasicModules.dll
    ADD.cs

默认的, 命令 "ADD" 将不会被加载到游戏服务器实例。 如果你想要加载该命令, 你如要在配置中添加程序集 BasicModules.dll 到命令程序集列表之中:

    <?xml version="1.0" encoding="utf-8" ?>
    <configuration>
	    <configSections>
	        <section name="superSocket" type="SuperSocket.SocketEngine.Configuration.SocketServiceConfig, SuperSocket.SocketEngine"/>
	    </configSections>
	    <appSettings>
	        <add key="ServiceName" value="BroardcastService"/>
	    </appSettings>
	    <superSocket>
	        <servers>
	            <server name="SampleServer"
	                    serverType="GameServer.MyGameServer, GameServer"
	                    ip="Any" port="2012">
	              <commandAssemblies>
	                <add assembly="BasicModules"></add>
	              </commandAssemblies>
	            </server>
	        </servers>
	    </superSocket>
	    <startup>
	        <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.0" />
	    </startup>
	</configuration>

当然你也可以在配置中添加多个命令程序集。

# 命令加载器 (Command Loader) 

在某些情况下，你可能希望通过直接的方式来加载命令，而不是通过自动的反射。 如果是这样，你可以实现你自己的命令加载器 (Command Loader):

    public interface ICommandLoader<TCommand>

然后配置你的服务器来使用你新建的命令加载器 (Command Loader):

    <superSocket>
	    <servers>
	      <server name="SampleServer"
	              serverType="GameServer.MyGameServer, GameServer"
	              ip="Any" port="2012"
				  commandLoader="MyCommandLoader">
	      </server>
	    </servers>
	    <commandLoaders>
	        <add name="MyCommandLoader"
	           type="GameServer.MyCommandLoader, GameServer" />
	    </commandLoaders>
    </superSocket>