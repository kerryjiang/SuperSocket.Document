# 在 SuperSocket 中启用传输层加密

> __关键字__: TLS, SSL, Certificate, X509 Certificate, Local Certificate Store, 证书, X509证书

## SuperSocket 支持传输层加密 (TLS)

SuperSocket 内置对 TLS 的支持。你不需要对你的代码做任何改动而让你的 Socket 服务器支持传输层加密。

## 为你的 SuperSocket 服务器启用 TLS，你需要提前准备一个证书。
有两种提供证书的方式：

1. 一个有私钥的 X509 证书文件 (*.pfx)
    * 生成用于测试的证书文件，你可以使用这个开源的 CertificateCreator；(https://github.com/kerryjiang/CertificateCreator)
    * 在生产环境之中，你应该从一个证书机构购买证书；
2. 使用你本地证书仓库中的证书；

## 用证书文件启用 TLS

你应该通过下面的步骤更新配置来使用证书文件:

1. 设置 listener 的 security 属性；
2. 在 listener 配置节点增加 certificate option 节点；

配置文件应如:

    {
        "serverOptions": {
            "name": "TestServer",
            "listeners": [
                {
                    "ip": "Any",
                    "port": 4040,
                    "security": "Tls12",
                    "certificateOptions": {
                        "filePath": "supersocket.pfx",
                        "password": "supersocket"
                    }
                }
            ]
        }
    }

注意: certificate options中的 password 是证书文件的私钥；

另外还有一个额外的选项 "**keyStorageFlags**" 用于证书加载：

    "certificateOptions": {
        "filePath": "supersocket.pfx",
        "password": "supersocket",
        "keyStorageFlags": "UserKeySet"
    }

你可以阅读下面这篇 MSDN 文章 获取这个选项的更多信息：
[http://msdn.microsoft.com/zh-cn/library/system.security.cryptography.x509certificates.x509keystorageflags(v=vs.110).aspx](http://msdn.microsoft.com/zh-cn/library/system.security.cryptography.x509certificates.x509keystorageflags(v=vs.110).aspx)

## 使用你本地证书仓库中的证书启用 TLS

你也可以不用屋里的证书文件，而是使用你本地的证书仓库中的证书来启用 TLS。你必须提供你想使用的证书的指纹（thumbprint。 如果 storeName 未被指定，系统将默认从名为 "Root" 的证书仓库搜索你指定的证书：

    "certificateOptions": {
        "storeName": "My",
        "thumbprint": "‎f42585bceed2cb049ef4a3c6d0ad572a6699f6f3"
    }

其他可选选项：

* **storeLocation** - CurrentUser, LocalMachine

    "certificateOptions": {
        "storeName": "My",
        "thumbprint": "‎f42585bceed2cb049ef4a3c6d0ad572a6699f6f3",
        "storeLocation": "LocalMachine"
    }


## 客户端证书验证

在 TLS 通信中，客户端证书不是必须的。但是有些系统需要更高的安全性。此功能允许你在服务器端验证客户端的证书。

首先，要启用客户端验证，你需要在监听器的 certificateOptions 配置节点之中设置属性 "clientCertificateRequired"：

    "certificateOptions": {
        "filePath": "supersocket.pfx",
        "password": "supersocket",
        "clientCertificateRequired": true
    }

然后你应该在你配置服务器选项的时候通过 CertificateOptions 的 *RemoteCertificateValidationCallback* 定义你的客户端证书验证逻辑：

    var host = SuperSocketHostBuilder.Create<TextPackageInfo, LinePipelineFilter>(args)
        .ConfigureSuperSocket(options =>
        {
            foreach (var certOptions in options.Listeners.Where(l => l.CertificateOptions != null && l.CertificateOptions.ClientCertificateRequired))
            {
                certOptions.RemoteCertificateValidationCallback = (sender, certificate, chain, sslPolicyErrors) => true;
            }
        }).Build();

    await host.RunAsync();