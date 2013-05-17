# Get the Connected Event and Closed Event of a Connection

> __Keywords__: Session, Connected Event, Closed Event

## AppSession's virtual methods OnSessionStarted() and OnSessionClosed(CloseReason reason)

You can override the base virtual methods OnSessionStarted() and OnSessionClosed(CloseReason reason) to do some business operations when a new session connects or a session drops:

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

## AppServer's event NewSessionConnected and event SessionClosed

Subscribe event:

    appServer.NewSessionConnected += new SessionHandler<AppSession>(appServer_NewSessionConnected);
    appServer.SessionClosed += new SessionHandler<AppSession, CloseReason>(appServer_SessionClosed);


Define event handling method:
    
    static void appServer_SessionClosed(AppSession session, CloseReason reason)
    {
        Console.WriteLine("A session is closed for {0}.", reason);
    }

    static void appServer_NewSessionConnected(AppSession session)
    {
        session.Send("Welcome to SuperSocket Telnet Server");
    }