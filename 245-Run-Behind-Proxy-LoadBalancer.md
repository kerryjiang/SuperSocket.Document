# Run SuperSocket behind proxy or load balancer.

> __Keywords__: Proxy, Load Balancer

It is very common to run SuperSocket behind proxy or load balancer for the concerns about security and availability. In this scenario, the client of SuperSocket will be proxy, load balancer, or other front end service. That would be difficult for SuperSocket application to know where the connections really come from. Well, SuperSocket provides the ability to allow you to get real remote endpoints of the connections.

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

