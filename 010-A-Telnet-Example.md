# A Telnet Example

> __Keywords__: Telnet, Console Project, References

## Prerequisites

Make sure you have installed the latest .NET Core 3.0 SDK (preview version)


## Create a Console project and add references of SuperSocket (targets to netcoreapp3.0)

	dotnet new console
	dotnet add package SuperSocket --version 2.0.0-*


## Using the namespaces of SuperSocket

	using SuperSocket;
	using SuperSocket.ProtoBase;
	using SuperSocket.Server;


## Write the code about SuperSocket host startup in the Main method

	static async Task Main(string[] args)
    {
		var host = ...;
		...
		await host.RunAsync();
	}


### Create the host

	var host = SuperSocketHostBuilder
		.Create<StringPackageInfo, CommandLinePipelineFilter>()

Create the SuperSocket host with the package type and the pipeline filter type.


### Register package handler which processes incoming data


	.ConfigurePackageHandler(async (s, p) =>
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

Enable the console output only.


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