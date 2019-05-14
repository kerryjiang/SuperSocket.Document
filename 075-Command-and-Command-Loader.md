# Command and Command Loader

> __Keywords__: Command, Command Loader, Multiple Command Assemblies

## Command
Command in SuperSocket is designed to handle the requests coming from the clients, it play an important role in the business logic processing.

Command class must implement the basic command interface below, choose Sync Command or Async Commnad as your need:

    public interface ICommand<TKey>
    {
        TKey Key { get; }

        string Name { get; }
    }

    // Sync Command
    public interface ICommand<TKey, TPackageInfo> : ICommand<TKey>
        where TPackageInfo : IKeyedPackageInfo<TKey>
    {
        void Execute(IAppSession session, TPackageInfo package);
    }

    // Async Command
    public interface IAsyncCommand<TKey, TPackageInfo> : ICommand<TKey>
        where TPackageInfo : IKeyedPackageInfo<TKey>
    {
        Task ExecuteAsync(IAppSession session, TPackageInfo package);
    }


The request processing code should be placed in the method "Execute" or "ExecuteAsync" and the property "Key" is used for matching the received packageInfo. When a packageInfo instance is received, SuperSocket will look for the command which has the responsibility to handle it by matching the packageInfo's Key and the command's Key.

For instance, if we receive a packageInfo like below:

    Key: "ADD"
    Body: "1 2"

Then SuperSocket will looking a command whose key is "ADD". If we have a command defined below:

    public class ADD : IAsyncCommand<string, StringPackageInfo>
    {
        public string Key => "ADD";

        public string Name => Key;

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