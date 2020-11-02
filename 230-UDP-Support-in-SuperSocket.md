# UDP Support in SuperSocket

> __Keywords__: UDP

## Enable UDP in SuperSocket

Other than TCP, SuperSocket can support UDP as well.

First of all, you need add reference to the package SuperSocket.Udp.

        dotnet add package SuperSocket.Udp --version 2.0.0-*


After you create your SuperSocket host builder like TCP, you just need enable UDP with one extra line of code:

        hostBuilder.UseUdp();



## Use your own Session Identifier

For UDP SuperSocket server, the client IP address and port are used as session's identifier. But in some cases, you need use something like device id as session identifier. SuperSocket allows you to do it with IUdpSessionIdentifierProvider. You need implement this interface and then register it into the SuperSocket's host builder.


#### Define your UdpSessionIdentifierProvider:


    public class MySessionIdentifierProvider : IUdpSessionIdentifierProvider
    {
        public string GetSessionIdentifier(IPEndPoint remoteEndPoint, ArraySegment<byte> data)
        {
            // take the device ID from the package data
            ....
            //return deviceID;
        }
    }


#### Register your UdpSessionIdentifierProvider:

    hostBuilder.ConfigureServices((context, services) =>
    {
        services.AddSingleton<IUdpSessionIdentifierProvider, MySessionIdentifierProvider>();                
    })