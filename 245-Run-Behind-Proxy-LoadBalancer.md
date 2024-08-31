# 运行 SuperSocket 在 proxy 和 load balancer 之后。

> __Keywords__: Proxy, Load Balancer

出于安全性和高可用性的考虑，我们常常将 SuperSocket 运行在 Proxy 或者 Load Balancer 之后。在这种情况下，SuperSocket 的客户端将是 proxy，load balancer或者其它前端服务器。这样会使 SuperSocket无法得知连接的真正来源。 不过，SuperSocket 提供了获取连接真正来源的方法。

## 启用 Proxy Protocol V1/V2 的支持

来源: https://www.haproxy.org/download/1.8/doc/proxy-protocol.txt

通过 API 启用 Proxy Protocol

```
var host = SuperSocketHostBuilder.Create<MyPackage, MyPackageFilter>(args)    
    .ConfigureSuperSocket(options =>
    {
        options.Name = "CustomProtocol Server";
        options.EnableProxyProtocol = true;
    }).Build();
```

通过配置文件启用 Proxy Protocol

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

## 获取连接的真实来源地址

```
// AppSession session

session.RemoteEndPoint

or

(session as IAppSession).Connection.ProxyInfo?.SourceEndPoint
```
