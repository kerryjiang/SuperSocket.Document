# SuperSocket 中的 UDP 支持

> __Keywords__: UDP

## 在 SuperSocket 中启用 UDP

除了TCP，SuperSocket 也能支持 UDP。

首先，你需要引用包 SuperSocket.Udp.

        dotnet add package SuperSocket.Udp --version 2.0.0-*


在你像TCP一样创建 SuperSocket host builder 之后，你只需要额外添加一行代码来启用 UDP：

        hostBuilder.UseUdp();



## 使用你自己的会话标识

在 SuperSocket UDP 服务器中，Socket 客户端的 IP 地址和端口用来作为会话的标识。但是在一些情况下，你需要用设备ID之类的东西作为会话标识。SuperSocket 通过接口 IUdpSessionIdentifierProvider 支持这一功能。你需要实现这个接口，然后将它注册到 SuperSocket 的 host builder 中。


#### 定义你的 UdpSessionIdentifierProvider:


    public class MySessionIdentifierProvider : IUdpSessionIdentifierProvider
    {
        public string GetSessionIdentifier(IPEndPoint remoteEndPoint, ArraySegment<byte> data)
        {
            // take the device ID from the package data
            ....
            //return deviceID;
        }
    }


#### 注册你的 UdpSessionIdentifierProvider:

    hostBuilder.ConfigureServices((context, services) =>
    {
        services.AddSingleton<IUdpSessionIdentifierProvider, MySessionIdentifierProvider>();                
    })