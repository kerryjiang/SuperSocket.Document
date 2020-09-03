# Integrate with ASP.Net Core Website and ABP Framework

> __Keywords__: ASP.NET Core, ABP, Integrate

## Integrate with ASP.Net Core Website

Yes, SuperSocket can run together with ASP.NET Core website side by side. What you should do are registering SuperSocket into the host builder of the ASP.NET Core and leaving the options in the configuration file or by code.


In the Program class, add more lines of code for SuperSocket:


        //don't forget the usings
        using SuperSocket;
        using SuperSocket.ProtoBase;

        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>();
                })
                .AsSuperSocketHostBuilder<TextPackageInfo, LinePipelineFilter>()
                .UsePackageHandler(async (s, p) =>
                {
                    // echo message back to client
                    await s.SendAsync(Encoding.UTF8.GetBytes(p.Text + "\r\n"));
                });


And leave server options in the configuration file "appsettings.json":


        {
            "Logging": {
                "LogLevel": {
                "Default": "Information",
                "Microsoft": "Warning",
                "Microsoft.Hosting.Lifetime": "Information"
                }
            },
            "serverOptions": {
                "name": "TestServer",
                "listeners": [
                    {
                        "ip": "Any",
                        "port": 4040
                    }
                ]
            },
            "AllowedHosts": "*"
        }

## Integrate with ABP Framework

Coming soon...