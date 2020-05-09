# 扩展你的 AppSession 和 SuperSocketService

> __Keywords__: SuperSocketService, AppSession

## 什么是 AppSession?
AppSession 代表一个和客户端的逻辑连接，基于连接的操作应该定于在该类之中。你可以用该类的实例发送数据到客户端，接收客户端发送的数据或者关闭连接。

## 什么是 SuperSocketService?
SuperSocketService 代表了监听所有客户端连接的服务器实例，宿主所有连接。 应用程序级别的操作和业务逻辑可以定义在里面。

## 扩展你的 AppSession

1. 重写 AppSession 的操作
    
        public class TelnetSession : AppSession
        {
            protected override async ValueTask OnSessionConnectedAsync()
            {
                // do something right after the sesssion is connected
            }            

            protected override async ValueTask OnSessionClosedAsync(EventArgs e)
            {
                // do something right after the sesssion is closed
            }
        }

2. 你也可以根据业务需要给你的 AppSession 添加新的属性
让我们创建一个可以用与游戏服务器的一个 AppSession:

        public class PlayerSession ：AppSession
        {
            public int GameHallId { get; internal set; }

            public int RoomId { get; internal set; }
        }

3. 在你的 SuperSocket 服务中采用你的 AppSession
通过builder来注册你的 AppSession 类型


        builder.UseSession<MySession>();


## 创建你自己的 SuperSocket Service


1. 扩展 SuperSocketService 并按照你的需要重写操作：

        public class GameService<TReceivePackageInfo> : SuperSocketService<TReceivePackageInfo>
            where TReceivePackageInfo : class
        {
            protected override ValueTask OnSessionConnectedAsync(IAppSession session)
            {
                // do something right after the sesssion is connected
                await base.OnSessionConnectedAsync(session);
            }

            protected override ValueTask OnSessionClosedAsync(IAppSession session)
            {
                // do something right after the sesssion is closed
                await base.OnSessionClosedAsync(session);
            }

            protected override async ValueTask OnStartedAsync()
            {
                // do something right after the service is started
            }

            protected override async ValueTask OnStopAsync()
            {
                // do something right after the service is stopped
            }
        }

2. 开始使用你自己的 Serivce 类型
通过 builder 注册你的 service 类型:

    builder.UseHostedService<GameService<TReceivePackageInfo>>();