# 获取会话的连接和断开事件

> __关键字__: 连接, 会话, 连接事件, 断开事件

## 通过 host builder 的方法 ConfigureSessionHandler 注册 session 的连接和断开处理代码

    builder.UseSessionHandler((s) =>
        {
            // 会话连接建立后的逻辑
        },
        (s, e) =>
        {
            // s: 当前会话
            // e: 会话关闭事件参数
            // e.Reason: 会话关闭原因

            // 会话连接断开后的逻辑
        });

## 通过扩展应用程序会话类型来处理会话事件

定义你自己的应用程序会话类型，然后在重写（override）方法内处理会话事件:

    public class MyAppSession : AppSession
    {
        protected override ValueTask OnSessionConnectedAsync()
        {
            // 会话连接建立后的逻辑
        }

        protected override ValueTask OnSessionClosedAsync(CloseEventArgs e)
        {
            // 会话连接断开后的逻辑
        }
    }

通过HostBuilder来启用你扩展的应用程序会话类型:

    hostBuiler.UseSession<MyAppSession>();


## 通过扩展 SuperSocket 服务来处理会话事件

定义你的 SuperSocket 服务类型然后重写会话事件处理方法:

        public class GameService<TReceivePackageInfo> : SuperSocketService<TReceivePackageInfo>
            where TReceivePackageInfo : class
        {
            protected override ValueTask OnSessionConnectedAsync(IAppSession session)
            {
                // 会话连接建立后的逻辑
            }

            protected override ValueTask OnSessionClosedAsync(IAppSession session, CloseEventArgs e)
            {
                // 会话连接断开后的逻辑
            }
        }

当你在创建 SuperSocket 服务宿主的时候使用你定义的 SuperSocket 服务类型:

        builder.UseHostedService<GameService<StringPackageInfo>>();