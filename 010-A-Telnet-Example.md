# A Telnet Example

> __Keywords__: Telnet, Console Project, References

## Create a Console project and add references of SuperSocket

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
		.Create<TextPackageInfo, LinePipelineFilter>()

Create the SuperSocket host with the package type and the pipeline filter type.


### Register package handler which processes incoming data


	.ConfigurePackageHandler(async (s, p) =>
	{
		await s.Channel.SendAsync(Encoding.UTF8.GetBytes(p.Text + "\r\n"));
	})

Send the received text back to the clinet.


### Configure logging

	.ConfigureLogging((hostCtx, loggingBuilder) =>
	{
		loggingBuilder.AddConsole();
	})

Enable the console output only.


### Configure server's information like name and listning endpoint adn build the host

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