# 主动从服务器端推送数据到客户端

> 关键字: 主动推送, 推送数据, 客户端推送, 获取Session, 发送数据, 回话快照

## 通过Session对象发送数据到客户端

前面已经说过，AppSession 代表了一个逻辑的 socket 连接，基于连接的操作都应该定义在此类之中。 这个AppSession 类也封装了通过 socket 发送数据的方法。 你可以使用 AppSession 的方法 "Send(...)" 来发送数据到客户端：

    session.Send(data, 0, data.Length);
    or
    session.Send("Welcome to use SuperSocket!");

## 通过 SessionID 获取 Session

前面提到过，如果你获取了连接的 Session 实例，你就可以通过 "Send()" 方法向客户端发送数据。但是在某些情况下，你无法直接获取 Session 实例。

SuperSocket 提供了一个 API 让你从 AppServer 的 Session 容器中通过 SessionID 获取 Session

    var session = appServer.GetAppSessionByID(sessionID);

    if(session != null)
        session.Send(data, 0, data.Length);


SessionID是什么？

SessionID 是 AppSession 类的一个属性，用于唯一标识一个 Session 实例。 在一个 SuperSocket TCP 服务器中，当 Session 一创建， SessionID 就会被赋值为一个 GUID 字符串。 如果你不在 SuperSocket UDP 服务器中使用 UdpRequestInfo，SessionID 就会有客户端的IP和端口组成。 如果你使用UdpRequestInfo，SessionID将会从客户端传过来。

## 获取所有连接上的 Session

你也可以从 AppServer 实例获取所有连接上的 session 然后推送数据到所有客户端：

    foreach(var session in appServer.GetAllSessions())
    {
        session.Send(data, 0, data.Length);
    }    

如果你启用了 Session 快照, 这些从 AppServer.GetAllSessions() 获取的 sessions 将不是实时更新的。 他们是在上次获取快照时所有连接到服务器的 Session。 快照相关配置，请参考配置文档。

## 根据条件获取 Session

如果你有一个自定义的属性 "CompanyId" 在你的 AppSession 类之中，如果你想要获取这个属性等于某值的 的所有 Session， 你可以使用 AppServer 的方法 GetSessions(...):

    var sessions = appServer.GetSessions(s => s.CompanyId == companyId);
    foreach(var s in sessions)
    {
        s.Send(data, 0, data.Length);
    }

和方法 "GetAllSessions(...)" 一样, 如果你启用了 Session 快照，这些返回的 session 也一样也来自于快照之中。