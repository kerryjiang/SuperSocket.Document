# 在SuperSocket中启用TLS/SSL传输层加密

> 关键字: TLS, SSL, 传输层加密, 传输层安全, 证书使用, X509Certificate

## SuperSocket 支持传输层加密(TLS/SSL)

SuperSocket 有自动的对TLS/SSL的支持，你可以无须增加或者修改任何代码，就能让你的服务器支持TLS/SSL。

## 想要为你的 SuperSocket 服务器启用 TLS/SSL，你需要先准备好一个授权证书

你有两种方式提供证书:

1. 一个带有私钥的 X509 证书文件
    * 你可以通过 CertificateCreator in SuperSocket(http://supersocket.codeplex.com/releases/view/59311) 生成证书文件用于测试
    * 在生产环境，你应该向证书颁发机构购买证书
2. 一个在你本地证书仓库的证书

## 通过证书文件启用 TLS/SSL

你需要通过下面的步骤修改配置文件来使用你准备好的证书文件：

1. 在server节点设置security属性；
2. 在server节点下增加certificate子节点；

最后配置应该像这样：

    <server name="EchoServer"
            serverTypeName="EchoService"
            ip="Any" port="443"
            security="tls">
        <certificate filePath="localhost.pfx" password="supersocket"></certificate>
    </server>

提示: certificate节点的password属性的值是这个证书文件的私钥

## 通过本地证书仓库的证书来启用 TLS/SSL

你也可以通过本地证书仓库的证书，而不是使用一个物理文件。 你只需要在配置中设置你要使用的证书的storeName和thumbprint：

    <server name="EchoServer"
            serverTypeName="EchoService"
            ip="Any" port="443"
            security="tls">
        <certificate storeName="My" thumbprint="‎f42585bceed2cb049ef4a3c6d0ad572a6699f6f3"></certificate>
    </server>

## 你也可以只为服务器实例的其中一个监听启用TLS/SSL，而其它监听仍然使用明文传输。

    <server name="EchoServer" serverTypeName="EchoService" maxConnectionNumber="10000">
        <certificate storeName="My" thumbprint="‎f42585bceed2cb049ef4a3c6d0ad572a6699f6f3"></certificate>
        <listeners>
          <add ip="Any" port="80" />
          <add ip="Any" port="443" security="tls" />
        </listeners>
    </server>

## 客户端安全证书验证

在 TLS/SSL 安全通信中, 客户端的安全证书不是必需的, 但是有些系统需要更高级别的安全保障. 此功能允许你在服务器端验证客户端证书.

首先, 要启用客户端证书验证, 你需要在配置中的证书节点增加新的属性 "clientCertificateRequired":

    <certificate storeName="My"
				 storeLocation="LocalMachine"
                 clientCertificateRequired="true"
                 thumbprint="‎f42585bceed2cb049ef4a3c6d0ad572a6699f6f3"/>


然后你需要重写 AppServer 的方法 "ValidateClientCertificate(...)" 用于实现你的验证逻辑:

    protected override bool ValidateClientCertificate(YourSession session, object sender, X509Certificate certificate, X509Chain chain, SslPolicyErrors sslPolicyErrors)
    {
       //Check sslPolicyErrors

	   //Check certificate

       //Return checking result
       return true;
    }