# Start SuperSocket by Configuration

> __Keywords__: Start by Configuration, Configuration

## The configuration file of SuperSocket

Same as Asp.Net Core, SuperSocket uses the JSON configuration file appsettings.json. We just need leave the file in root of the application folder and make sure it can be copied to the output directory of he build.

* appsettings.json
* appsettings.Development.json  // for Development environment
* appsettings.Production.json // for Production environment

## How to Start Server with the configuration file

Actually, you don't need do anything else except writing the normal startup code.

    var host = SuperSocketHostBuilder.Create<StringPackageInfo, CommandLinePipelineFilter>()
        .UsePackageHandler(async (s, p) =>
        {
            // handle packages
        })
        .ConfigureLogging((hostCtx, loggingBuilder) =>
        {
            loggingBuilder.AddConsole();
        })
        .Build();

    await host.RunAsync();


## The format of the configuration file (appsettings.json)

It is a sample:

    {
        "serverOptions": {
            "name": "GameMsgServer",
            "listeners": [
                {
                    "ip": "Any",
                    "port": "2020"
                },
                {
                    "ip": "192.168.3.1",
                    "port": "2020"
                }
            ]
        }
    }

Options:

* name: the name of the server;
* maxPackageLength: max allowed package size in the server; 4M by default;
* receiveBufferSize: the size of the receiving buffer; 4k by default;
* sendBufferSize: the size of the sending buffer; 4k by default;
* receiveTimeout: the timeout for receiving; in milliseconds;
* sendTimeout: the timeout for sending; in milliseconds;
* listeners: the listener enpoints of this server;
* listeners/*/ip: the listener's listening IP; Any: any ipv4 ip addresses, IPv6Any: any ipv6 ip addresses, other actual IP addresses;
* listeners/*/port: the listener's listening port;
* listeners/*/backLog: the maximum length of the pending connections queue;
* listeners/*/noDelay: specifies whether the stream Socket is using the Nagle algorithm;
* listeners/*/security: None/Ssl3/Tls11/Tls12/Tls13; the TLS protocol version the communication uses;
* listeners/*/certificateOptions: the options of the certificate which will be used for the TLS encryption/decryption;