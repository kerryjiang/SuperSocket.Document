# 获取会话的连接和断开事件

> 关键字: 连接事件, 断开事件, OnSessionStarted, OnSessionClosed, NewSessionConnected, SessionClosed

## AppSession 的虚方法 OnSessionStarted() 和 OnSessionClosed(CloseReason reason)

你可以覆盖基类的虚方法 OnSessionStarted() 和 OnSessionClosed(CloseReason reason) 用于在会话连接和断开时执行一些逻辑操作：

    public class TelnetSession : AppSession<TelnetSession>
    {
        protected override void OnSessionStarted()
        {
            this.Send("Welcome to SuperSocket Telnet Server");
            //add your business operations
        }

        protected override void OnSessionClosed(CloseReason reason)
        {
            //add your business operations
        }
    }

## AppServer 的两个事件： NewSessionConnected 和 SessionClosed

订阅事件：

    appServer.NewSessionConnected += new SessionHandler<AppSession>(appServer_NewSessionConnected);
    appServer.SessionClosed += new SessionHandler<AppSession, CloseReason>(appServer_SessionClosed);

定义事件处理方法：
    
    static void appServer_SessionClosed(AppSession session, CloseReason reason)
    {
        Console.WriteLine("A session is closed for {0}.", reason);
    }

    static void appServer_NewSessionConnected(AppSession session)
    {
        session.Send("Welcome to SuperSocket Telnet Server");
    }