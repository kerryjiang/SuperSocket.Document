# Command and Command Loader

> __Keywords__: Command, Command Loader, Multiple Command Assemblies

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

The metadata value "Key" is used for matching the received packageInfo. When a packageInfo instance is received, SuperSocket will look for the command which has the responsibility to handle it by matching the packageInfo's Key and the command's Key.

For instance, if we receive a packageInfo like below:

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