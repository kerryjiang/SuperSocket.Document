# Run SuperSocket behind proxy or load balancer.

> __Keywords__: Proxy, Load Balancer

Running SuperSocket behind a proxy or load balancer is common for security and availability reasons. In this setup, the client of SuperSocket is typically the proxy, load balancer, or another front-end service, making it challenging for the SuperSocket application to identify the true origin of connections. However, SuperSocket offers the capability to retrieve the real remote endpoints of these connections.

## Enable Proxy Protocol V1/V2

Reference: https://www.haproxy.org/download/1.8/doc/proxy-protocol.txt

Enable Proxy Protocol by API

```
var host = SuperSocketHostBuilder.Create<MyPackage, MyPackageFilter>(args)    
    .ConfigureSuperSocket(options =>
    {
        options.Name = "CustomProtocol Server";
        options.EnableProxyProtocol = true;
    }).Build();
```

Enable Proxy Protocol by settings

appsettings.json

```
{
  "serverOptions": {
      "name": "TestServer",
      "enableProxyProtocol": true,
      "listeners": [
          {
              "ip": "Any",
              "port": 4040
          }
      ]
  },
  "AllowedHosts": "*"
}
```

## Access the connection's real remote endpoint

```
// AppSession session

session.RemoteEndPoint

or

(session as IAppSession).Connection.ProxyInfo?.SourceEndPoint
```

