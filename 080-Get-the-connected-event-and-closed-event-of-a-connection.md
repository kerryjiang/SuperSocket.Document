# Get the Connected Event and Closed Event of a Connection

> __Keywords__: Connection, Session, Connected Event, Closed Event

## Register session open/close handlers by the method ConfigureSessionHandler of the host builder

    builder.UseSessionHandler((s) =>
        {
            // things to do when the session just connects
        },
        (s, e) =>
        {
            // s: the session
            // e: the CloseEventArgs
            // e.Reason: the close reason
            // things to do after the session closes
        });


## Handle the session events by extending the application session

Define your own application session type and handle the session events in the override methods:

    public class MyAppSession : AppSession
    {
        protected override ValueTask OnSessionConnectedAsync()
        {
            // the logic after the session gets connected
        }

        protected override ValueTask OnSessionClosedAsync(CloseEventArgs e)
        {
            // the logic after the session gets closed
        }
    }

Enable your own application session with the host builder:

    hostBuiler.UseSession<MyAppSession>();


## Handle the session events by extending the SuperSocketService

Define your own SuperSocket service type and override the session event handling methods:

        public class GameService<TReceivePackageInfo> : SuperSocketService<TReceivePackageInfo>
            where TReceivePackageInfo : class
        {
            protected override ValueTask OnSessionConnectedAsync(IAppSession session)
            {
                // do something right after the sesssion gets connected
            }

            protected override ValueTask OnSessionClosedAsync(IAppSession session, CloseEventArgs e)
            {
                // do something right after the sesssion gets closed
            }
        }

Use your own SuperSocket service type when you create the host:

        builder.UseHostedService<GameService<StringPackageInfo>>();