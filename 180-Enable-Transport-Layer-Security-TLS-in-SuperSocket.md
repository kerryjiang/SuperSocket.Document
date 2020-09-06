# Enable Transport Layer Security in SuperSocket

> __Keywords__: TLS, SSL, Certificate, X509 Certificate, Local Certificate Store

## SuperSocket supports the Transport Layer Security (TLS)

SuperSocket has built-in support for TLS. You don't need make any change for your code to let your socket server support TLS.

## To enable TLS for your SuperSocket server, you should have a certificate in advance.
There are two ways to provide the certificate:

1. a X509 certificate file with private key (*.pfx)
    * for testing purpose you can generate a certificate file by this open source CertificateCreator(https://github.com/kerryjiang/CertificateCreator)
    * in production environment, you should purchase a certificate from a certificate authority
2. a certificate in local certificate store

## Enable TLS with a certificate file

You should update your configuration to use the certificate file following the below steps:

1. set security attribute for the listener; This attribute is for the TLS protocols what the listener will support; The appilicable values include "Tls11", "Tls12", "Tls13" and so on; Multiple values should be seperated by comma, like "Tls11,Tls12,Tls13";
2. add the certificate option node under the listener node;

The configuration should look like:

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

Note: the password in the certificate options is the private key of the certificate file.

There is one more option named "**keyStorageFlags**" for certificate loading:

    "certificateOptions": {
        "filePath": "supersocket.pfx",
        "password": "supersocket",
        "keyStorageFlags": "UserKeySet"
    }

You can read the MSDN article below for more information about this option:
[http://msdn.microsoft.com/zh-cn/library/system.security.cryptography.x509certificates.x509keystorageflags(v=vs.110).aspx](http://msdn.microsoft.com/zh-cn/library/system.security.cryptography.x509certificates.x509keystorageflags(v=vs.110).aspx)

## Enable TLS with a certificate in your local certificate store

You also can use a certificate in your local certificate store instead of a physical certificate file. The thumbprint of the certificate you want to use is required. If the storeName is not specified, the system will search the certificate from the "Root" store:

    "certificateOptions": {
        "storeName": "My",
        "thumbprint": "‎f42585bceed2cb049ef4a3c6d0ad572a6699f6f3"
    }

Other optional options:

* **storeLocation** - CurrentUser, LocalMachine

        "certificateOptions": {
            "storeName": "My",
            "thumbprint": "‎f42585bceed2cb049ef4a3c6d0ad572a6699f6f3",
            "storeLocation": "LocalMachine"
        }


## Client certificate validation

In TLS communications, the client side certificate is not a must, but some systems require much higher security guarantee. This feature allow you to validate the client side certificate from the server side.

At first, to enable the client certificate validation, you should add the attribute "clientCertificateRequired" in the certificate options node of the listener:

    "certificateOptions": {
        "filePath": "supersocket.pfx",
        "password": "supersocket",
        "clientCertificateRequired": true
    }


And then you should define you client certificate validation logic with the *RemoteCertificateValidationCallback* in the certificate options when you configure the server options:

    var host = SuperSocketHostBuilder.Create<TextPackageInfo, LinePipelineFilter>(args)
        .ConfigureSuperSocket(options =>
        {
            foreach (var certOptions in options.Listeners.Where(l => l.CertificateOptions != null && l.CertificateOptions.ClientCertificateRequired))
            {
                certOptions.RemoteCertificateValidationCallback = (sender, certificate, chain, sslPolicyErrors) => true;
            }
        }).Build();

    await host.RunAsync();