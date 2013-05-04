# 实现你的AppServer和AppSession

## 什么是AppSession?
AppSession 代表一个和客户端的逻辑连接，基于连接的操作应该定于在该类之中。你可以用该类的实例发送数据到客户端，接收客户端发送的数据或者关闭连接。

## 什么是AppServer?
AppServer 代表了监听客户端连接，承载TCP连接的服务器实例。理想情况下，我们可以通过AppServer实例获取任何你想要的客户端连接，服务器级别的操作和逻辑应该定义在此类之中。

## 创建你的AppSession
1. 你可以重写基类的方法
    
        public class TelnetSession : AppSession<TelnetSession>
        {
            protected override void OnSessionStarted()
            {
                this.Send("Welcome to SuperSocket Telnet Server");
            }

            protected override void HandleUnknownRequest(StringRequestInfo requestInfo)
            {
                this.Send("Unknow request");
            }

            protected override void HandleException(Exception e)
            {
                this.Send("Application error: {0}", e.Message);
            }

            protected override void OnSessionClosed(CloseReason reason)
            {
                //add you logics which will be executed after the session is closed
                base.OnSessionClosed(reason);
            }
        }

    
    在上面的代码中，当一个新的连接连接上时，服务器端立即向客户端发送欢迎信息。 这段代码还重写了其它AppSession的方法用以实现自己的业务逻辑。

2. 你可以根据你的业务需求来给Session类增加新的属性，让我来创建一个将用在游戏服务器中的AppSession类：

        public class PlayerSession ：AppSession<PlayerSession>
        {
            public int GameHallId { get; internal set; }

            public int RoomId { get; internal set; }
        }

3. 和Command之间的关系

    在上一篇文档中，我们谈到了Command, 现在我们重新来回顾一下这个知识点：

        public class ECHO : CommandBase<AppSession, StringRequestInfo>
        {
            public override void ExecuteCommand(AppSession session, StringRequestInfo requestInfo)
            {
                session.Send(requestInfo.Body);
            }
        }

    在这个command代码中，你可以看到类ECHO的父类是CommandBase<AppSession, StringRequestInfo>， 它有一个泛型参数AppSession。 是的，它是默认的AppSession类。 如果你在你的系统中使用你自己建立的AppSession类，那么你必须将你自己定义的AppSession类传进去，否则你的服务器无法发现这个Command：

        public class ECHO : CommandBase<PlayerSession, StringRequestInfo>
        {
            public override void ExecuteCommand(PlayerSession session, StringRequestInfo requestInfo)
            {
                session.Send(requestInfo.Body);
            }
        }

## 创建你的AppServer类型

1. 如果你想使用你的AppSession作为会话，你必须修改你的AppServer来使用它


        public class TelnetServer : AppServer<TelnetSession>
        {

        }

    
    现在 TelnetSession 将可以用在 TelnetServer 的会话中。

2. 这里也有很多protected方法你可以重写

        public class TelnetServer : AppServer<TelnetSession>
        {
            protected override bool Setup(IRootConfig rootConfig, IServerConfig config)
            {
                return base.Setup(rootConfig, config);
            }

            protected override void OnStartup()
            {
                base.OnStartup();
            }

            protected override void OnStopped()
            {
                base.OnStopped();
            }
        }

## 优点
实现你自己的AppSession和AppServer允许你根据你业务的需求来方便的扩展SuperSocket，你可以绑定session的连接和断开事件，服务器实例的启动和停止事件。你还可以在AppServer的Setup方法中读取你的自定义配置信息。总而言之，这些功能让你方便的创建一个你所需要的socket服务器成为可能。