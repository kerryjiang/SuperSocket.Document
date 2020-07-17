# Extend Your AppSession and SuperSocketService

> __Keywords__: SuperSocketService, AppSession

## What is AppSession?
AppSession represents a logic socket connection, connection based operations should be defined in this class. You can use the instance of this class to send data to clients, receive data from connection or close the connection.

## What is SuperSocketService?
SuperSocketService stands for the service instance which listens all client connections, hosts all connections. Application level operations and logics can be defined in it.

## Extend your AppSession

1. You can override base AppSession's operations
    
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

2. You also can add new properties for your session according your business requirement
Let me create a AppSession which would be used in a game server:


        public class PlayerSession ：AppSession
        {
            public int GameHallId { get; internal set; }

            public int RoomId { get; internal set; }
        }

3. Adopt your own AppSession class in your SuperSocket service
Register the application session type through builder:


        builder.UseSession<MySession>();


## Create your own SuperSocket service


1. Extend the SuperSocketService and override the methods if you want

        public class GameService<TReceivePackageInfo> : SuperSocketService<TReceivePackageInfo>
            where TReceivePackageInfo : class
        {
            public GameService(IServiceProvider serviceProvider, IOptions<ServerOptions> serverOptions)
                : base(serviceProvider, serverOptions)
            {
                
            }

            protected override async ValueTask OnSessionConnectedAsync(IAppSession session)
            {
                // do something right after the sesssion is connected
                await base.OnSessionConnectedAsync(session);
            }

            protected override async ValueTask OnSessionClosedAsync(IAppSession session)
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

2. Start to use your own service type
Register the service type through builder:


        builder.UseHostedService<GameService<TReceivePackageInfo>>();