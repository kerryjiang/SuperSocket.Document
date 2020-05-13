# A Telnet Example

> __Keywords__: Telnet, Console Project, References

## Prerequisites

Make sure you have installed the latest .NET Core SDK (3.0 or above version)


## Create a Console project and add references of SuperSocket (targets to netcoreapp3.0)

    dotnet new console
    dotnet add package SuperSocket.Server --version 2.0.0-*


## Using the namespaces of SuperSocket and other namespaces you might need

    using System.Threading.Tasks;
    using Microsoft.Extensions.Hosting;
    using Microsoft.Extensions.Logging;
    using SuperSocket;
    using SuperSocket.ProtoBase;


## Write the code about SuperSocket host startup in the Main method

    static async Task Main(string[] args)
    {
        var host = ...;
        ...
        await host.RunAsync();
    }


### Create the host

    var host = SuperSocketHostBuilder.Create<StringPackageInfo, CommandLinePipelineFilter>()

Create the SuperSocket host with the package type and the pipeline filter type.


### Register package handler which processes incoming data


    .UsePackageHandler(async (s, p) =>
    {
        var result = 0;

        switch (p.Key.ToUpper())
        {
            case ("ADD"):
                result = package.Parameters
                    .Select(p => int.Parse(p))
                    .Sum();
                break;

            case ("SUB"):
                result = package.Parameters
                    .Select(p => int.Parse(p))
                    .Aggregate((x, y) => x - y);
                break;

            case ("MULT"):
                result = package.Parameters
                    .Select(p => int.Parse(p))
                    .Aggregate((x, y) => x * y);
                break;
        }

        await s.SendAsync(Encoding.UTF8.GetBytes(result.ToString() + "\r\n"));
    })

Send the received text back to the clinet.


### Configure logging

    .ConfigureLogging((hostCtx, loggingBuilder) =>
    {
        loggingBuilder.AddConsole();
    })

Enable the console output only. You also can register your own thirdparty logging library over here.


### Configure server's information like name and listening endpoint adn build the host

    .ConfigureSuperSocket(options =>
    {
        options.Name = "Echo Server";
        options.Listeners = new [] {
            new ListenOptions
            {
                Ip = "Any",
                Port = 4040
            }
        };
    }).Build();


### Start the host

    await host.RunAsync();