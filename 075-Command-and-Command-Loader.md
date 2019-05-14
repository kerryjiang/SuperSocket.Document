# 命令和命令加载器

> __关键字__: 命令, 命令加载器, 多命令程序集

## 命令 (Command)
SuperSocket 中的命令设计出来是为了处理来自客户端的请求的, 它在业务逻辑处理之中起到了很重要的作用。

命令类必须实现下面的基本命令接口, 根据你的需要选择实现同步命令或异步命令:

    public interface ICommand<TKey>
    {
        TKey Key { get; }

        string Name { get; }
    }

    // 同步命令
    public interface ICommand<TKey, TPackageInfo> : ICommand<TKey>
        where TPackageInfo : IKeyedPackageInfo<TKey>
    {
        void Execute(IAppSession session, TPackageInfo package);
    }

    // 异步命令
    public interface IAsyncCommand<TKey, TPackageInfo> : ICommand<TKey>
        where TPackageInfo : IKeyedPackageInfo<TKey>
    {
        Task ExecuteAsync(IAppSession session, TPackageInfo package);
    }


请求处理代码必须被放置于方法 "Execute" or "ExecuteAsync" 之中，并且属性 "Keys" 的值用于匹配接收到请求实例(packageInfo)的Key。当一个请求实例(packageInfo) 被收到时，SuperSocket 将会通过匹配请求实例(packageInfo)的Key和命令的Keys的方法来查找用于处理该请求的命令。

举个例子, 如果你收到如下请求(packageInfo):

    Key: "ADD"
    Body: "1 2"

于是 SuperSocket 将会寻找Key属性为"ADD"的命令。如果有个命令定义如下:

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

如果你注册了此命令，SuperSocket就会找到该命令。


## 注册命令

    hostBuilder.UseCommand((commandOptions) =>
    {
        // 一个一个的注册命令
        commandOptions.AddCommand<ADD>();
        //commandOptions.AddCommand<MULT>();
        //commandOptions.AddCommand<SUB>();

        // 注册程序集重的所有命令
        //commandOptions.AddCommandAssembly(typeof(SUB).GetTypeInfo().Assembly);
    }