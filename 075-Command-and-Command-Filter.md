# Command and Command Filter

> __Keywords__: Command, Command Filter

## Command
Command in SuperSocket is designed to handle the requests coming from the clients, it play an important role in the business logic processing.

Command class must implement the basic command interface below, choose Sync Command or Async Commnad as your need:

 
    // Sync Command
    public interface ICommand<TAppSession, TPackageInfo>
        where TAppSession : IAppSession
    {
        void Execute(TAppSession session, TPackageInfo package);
    }

    // Async Command
    public interface IAsyncCommand<TAppSession, TPackageInfo> : ICommand
        where TAppSession : IAppSession
    {
        ValueTask ExecuteAsync(TAppSession session, TPackageInfo package);
    }


The request package processing code should be placed in the method "Execute" or "ExecuteAsync".
Every command has its own metadata which includes Name and Key.

Name: human friendly name of the command;
Key: the object we use to match package's key;

We can define command's metadata (Name='ShowVoltage', Key=0x03) by attribute on the command class:

    [Command(Key = 0x03)]
    public class ShowVoltage : IAsyncCommand<StringPackageInfo>
    {
        public async ValueTask ExecuteAsync(IAppSession session, StringPackageInfo package)
        {
            ...
        }
    }

By default, command's Name and Key would be same as its class name if there is no command metadata attribute defined for the command class.

The metadata value "Key" is used for matching the received package. When a package instance is received, SuperSocket will look for the command which has the responsibility to handle it by matching the package's Key and the command's Key.

But, there is a prerequisite that the package type must implement the interface IKeyedPackageInfo<TKey> (TKey may be any primitive type like int, string, short or byte).

For instance, if we receive a package like below:

    Key: "ADD"
    Body: "1 2"

Then SuperSocket will looking for a command whose key is "ADD". If we have a command defined below:

    public class ADD : IAsyncCommand<string, StringPackageInfo>
    {
        public async Task ExecuteAsync(IAppSession session, StringPackageInfo package)
        {
            var result = package.Parameters
                .Select(p => int.Parse(p))
                .Sum();

            await session.SendAsync(Encoding.UTF8.GetBytes(result.ToString() + "\r\n"));
        }
    }

This command will be found after you register it.

## Register command

    hostBuilder.UseCommand((commandOptions) =>
    {
        // register commands one by one
        commandOptions.AddCommand<ADD>();
        //commandOptions.AddCommand<MULT>();
        //commandOptions.AddCommand<SUB>();

        // register all commands in one aassembly
        //commandOptions.AddCommandAssembly(typeof(SUB).GetTypeInfo().Assembly);
    }


## Command Filter

The Command Filter in SuperSocket works like Action Filter in ASP.NET MVC, you can use it to intercept execution of Command。 The Command Filter will be invoked before or after the command executes.

Synchronous CommandFilter:

    public class HelloCommandFilterAttribute : CommandFilterAttribute
    {
        public override void OnCommandExecuted(CommandExecutingContext commandContext)
        {
            Console.WriteLine("Hello");
        }

        public override bool OnCommandExecuting(CommandExecutingContext commandContext)
        {
            Console.WriteLine("Bye bye");
            return true;
        }
    }


Asynchronous CommandFilter:

    public class AsyncHelloCommandFilterAttribute  : AsyncCommandFilterAttribute
    {
        public override async ValueTask OnCommandExecutedAsync(CommandExecutingContext commandContext)
        {
            Console.WriteLine("Hello");
            await Task.Delay(0);
        }

        public override async ValueTask<bool> OnCommandExecutingAsync(CommandExecutingContext commandContext)
        {
            Console.WriteLine("Bye bye");
            await Task.Delay(0);
            return true;
        }
    }

Apply CommandFilter to Command:

    [AsyncHelloCommandFilter]
    [HelloCommandFilter]
    class COUNTDOWN : IAsyncCommand<StringPackageInfo>
    {
        //...
    }

## Global Command Filter

Global command filter is just one command filter which is applied to all the commands.

Register global command filter:

    hostBuilder.UseCommand((commandOptions) =>
    {
        commandOptions.AddCommand<COUNT>();
        commandOptions.AddCommand<COUNTDOWN>();

        // register global command filter
        commandOptions.AddGlobalCommandFilter<HitCountCommandFilterAttribute>();
    }


## Register Commands from the Configuration File

You can add command assemblies under the node "commands/assemblies". At the meanwhile, please make sure the command assembly files are copied into the working directory of the application.

It is the sample of the configuration:

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
            ],
            "commands": {
                "assemblies": [
                    {
                        "name": "CommandAssemblyName"
                    }
                ]
            }
        }
    }

Beside that, you may need do one more thing. Because .NET Core application only look for assembly in its depedency tree (*.deps.json) by default when it does reflection. Your command assembly may not be able to be founed if you didn't add it as reference in the major project. To work around this problem, you should add a runtime config file "runtimeconfig.json" in the root of your major
project.

runtimeconfig.json

    {
        "runtimeOptions": {
            "Microsoft.NETCore.DotNetHostPolicy.SetAppPaths" : true            
        }
    }