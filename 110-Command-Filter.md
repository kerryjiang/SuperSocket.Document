# 命令过滤器

> 关键字: 命令过滤器, 命令, 过滤器, OnCommandExecuting, OnCommandExecuted

SuperSocket 中的命令过滤器看起来有些像 ASP.NET MVC 中的 Action Filter，你可以用它来做命令执行的拦截，命令过滤器会在命令执行前和执行后被调用。

命令过滤器必须继承于 Attribute 类 CommandFilterAttribute:

     /// <summary>
    /// Command filter attribute
    /// </summary>
    [AttributeUsage(AttributeTargets.Class, AllowMultiple = true)]
    public abstract class CommandFilterAttribute : Attribute
    {
        /// <summary>
        /// Gets or sets the execution order.
        /// </summary>
        /// <value>
        /// The order.
        /// </value>
        public int Order { get; set; }

        /// <summary>
        /// Called when [command executing].
        /// </summary>
        /// <param name="commandContext">The command context.</param>
        public abstract void OnCommandExecuting(CommandExecutingContext commandContext);

        /// <summary>
        /// Called when [command executed].
        /// </summary>
        /// <param name="commandContext">The command context.</param>
        public abstract void OnCommandExecuted(CommandExecutingContext commandContext);
    }

你需要为你的命令过滤器实现下面两个方法:

**OnCommandExecuting**: 此方法将在命令执行前被调用;

**OnCommandExecuted**: 此方法将在命令执行后被调用;

**Order**: 此属性用于设置多个命令过滤器的执行顺序;

下面的代码定义了一个命令过滤器 LogTimeCommandFilterAttribute 用于记录执行时间超过5秒钟的命令:

    public class LogTimeCommandFilter : CommandFilterAttribute
    {
        public override void OnCommandExecuting(CommandExecutingContext commandContext)
        {
            commandContext.Session.Items["StartTime"] = DateTime.Now;
        }

        public override void OnCommandExecuted(CommandExecutingContext commandContext)
        {
            var session = commandContext.Session;
            var startTime = session.Items.GetValue<DateTime>("StartTime");
            var ts = DateTime.Now.Subtract(startTime);

            if (ts.TotalSeconds > 5 && session.Logger.IsInfoEnabled)
            {
                session.Logger.InfoFormat("A command '{0}' took {1} seconds!", commandContext.CurrentCommand.Name, ts.ToString());
            }
        }
    }

然后通过增加属性的方法给命令 "QUERY" 应用此过滤器:

    [LogTimeCommandFilter]
    public class QUERY : StringCommandBase<TestSession>
    {
        public override void ExecuteCommand(TestSession session, StringCommandInfo commandData)
        {
            //Your code
        }
    }

如果你想应用次过滤器给所有的命令, 你可以向下面的代码这样将此命令过滤器的属性加到你的 AppServer 类上面去:

    [LogTimeCommandFilter]
    public class TestServer : AppServer<TestSession>
    {

    }

你可以通过将 commandContext's 的 Cancel 属性设为 True 来取消当前命令的执行:

    public class LoggedInValidationFilter : CommandFilterAttribute
    {
        public override void OnCommandExecuting(CommandExecutingContext commandContext)
        {
            var session = commandContext.Session as MyAppSession;

            //If the session is not logged in, cancel the executing of the command
            if (!session.IsLoggedIn)
                commandContext.Cancel = true;
        }

        public override void OnCommandExecuted(CommandExecutingContext commandContext)
        {

        }
    }