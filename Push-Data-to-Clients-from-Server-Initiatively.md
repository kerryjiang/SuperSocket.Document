## Send Data to Client by Session Object

Having said that, AppSession represents a logic socket connection, connection based operations should be defined in this class. The AppSession also wraps the sending data method of the socket. You can use the method "Send(...)" of AppSession to send data to client:

    session.Send(data, 0, data.Length);
    or
    session.Send("Welcome to use SuperSocket!");

## Get Session by SessionID

As mentioned in previous part, if you have got the connection's session instance, then you can send data to the client by the "Send(..)" method. But in some cases, you cannot get the session instance you want directly.

SuperSocket provide a API to allow you get a session by session ID from the AppServer's session container.

    var session = appServer.GetSessionByID(sessionID);

    if(session != null)
        session.Send(data, 0, data.Length);


What is the SessionID?

SessionID is a property of the AppSession class which is used for identifying a Session. In a SuperSocket TCP server, the SessionID is a GUID string which is assigned as soon as the session is created. If you don't use UdpRequestInfo in a SuperSocket UDP server, the SessionID will be consist of the remote endpoint's IP and port. If you use UdpRequestInfo in a SuperSocket UDP server, the value of SessionID is passed from the client.

## Get All Connected Sessions

You also can get all connected sessions from the AppServer instances and then push data to all clients:

    foreach(var session in appServer.GetAllSessions())
    {
        session.Send(data, 0, data.Length);
    }    

If you enable session snapshot, the sessions get from method AppServer.GetAllSessions() are not updated realtime. They are all the connected sessions of the AppServer in the time when the last snapshot is taken.

## Get Sessions by Criteria

If you have a custom property "CompanyId" in your AppSession, and you want get all connected session whose CompanyId are equal with your specific value, the methd of AppServer "GetSession(...)" should be useful:

    var sessions = appServer.GetSessions(s => s.CompanyId == companyId);
    foreach(var s in sessions)
    {
        s.Send(data, 0, data.Length);
    }

Same as the method "GetAllSessions(...)", if you enable session snapshot, the sessions also come from snapshot.