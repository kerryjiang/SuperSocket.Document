# Enable TLS/SSL trasnferring layer encryption in SuperSocket

> __Keywords__: TLS, SSL, Certificate, X509 Certificate, Local Certificate Store

## SuperSocket supports the transport layer encryption (TLS/SSL)

SuperSocket has automatically support for TLS/SSL. You needn't change any code to let your socket server support TLS/SSL.

## To enable TLS/SSL for your SuperSocket server, you should prepare a certificate at first.
There are two ways to provide the certificate:

1. a X509 certificate file with private key
    * for testing purpose you can generate a certificate file by the CertificateCreator in SuperSocket(http://supersocket.codeplex.com/releases/view/59311)
    * in production environment, you should purchase a certificate from a certificate authority
2. a certificate in local certificate store

## Enable TLS/SSL by a certificate file

You should change the configuration file to use the certificate file following the below steps:

1. set security attribute for the server node;
2. add the certificate node in server node as child.

The configuration should look like:

    <server name="EchoServer"
            serverTypeName="EchoService"
            ip="Any" port="443"
            security="tls">
        <certificate filePath="localhost.pfx" password="supersocket"></certificate>
    </server>

Note: the password of the certificate node is the private key of the certificate file

## Enable TLS/SSL by a certificate in local certificate store

You also can use a certificate in your local certificate store without a physical file. The thumbprint of the certificate you want to use is required:

    <server name="EchoServer"
            serverTypeName="EchoService"
            ip="Any" port="443"
            security="tls">
        <certificate storeName="My" thumbprint="‎f42585bceed2cb049ef4a3c6d0ad572a6699f6f3"></certificate>
    </server>

## You also can only apply TLS/SSL for one listener of the appserver instance

    <server name="EchoServer" serverTypeName="EchoService" maxConnectionNumber="10000">
        <certificate storeName="My" thumbprint="‎f42585bceed2cb049ef4a3c6d0ad572a6699f6f3"></certificate>
        <listeners>
          <add ip="Any" port="80" />
          <add ip="Any" port="443" security="tls" />
        </listeners>
    </server>
