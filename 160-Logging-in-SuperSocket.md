# SuperSocket 中的日志功能

> 关键字: 日志, 日志接口, Log4Net, 自定义日志, 扩展日志, 企业库日志

## SuperSocket 中的日志系统

当 SuperSocket boostrap 启动时，日志系统将会自动启动。 所以你无须创建自己的日志工具，最好直接使用SuperSocket内置的日志功能。

SuperSocket 默认使用log4net作为第三方日志框架。 所以如果你熟悉log4net，你就能非常容易的使用和自定义SuperSocket的日志功能。

SuperSocket 还提供了基本的 log4net 配置文件 log4net.config/log4net.unix.config， 你需要把它放到程序运行目录的子目录"Config"。 这个 log4net 配置定义了多个 loggers 和 appenders 将所有日志分类输出到 "Logs"目录下的四个文件：

* info.log
* debug.log
* err.log
* perf.log

你也可以根据你自己的需要定制这个配置文件。

由于SuperSocket项目对log4net的弱引用，log4net.dll不会自动输出到你的项目，所以你需要手动引用log4net。（请务必使用SuperSocket提供的log4net版本）


## 日志接口

SuperSocket的日志功能非常简单，你几乎可以在任何地方都能记录日志。 AppServer 和 AppSession 都有Logger属性， 你可以直接用它来记录日志。

以下代码演示了日志接口的使用：

A -

    /// <summary>
    /// PolicyServer base class
    /// </summary>
    public abstract class PolicyServer : AppServer<PolicySession, BinaryRequestInfo>
    {
        ......

        /// <summary>
        /// Setups the specified root config.
        /// </summary>
        /// <param name="rootConfig">The root config.</param>
        /// <param name="config">The config.</param>
        /// <returns></returns>
        protected override bool Setup(IRootConfig rootConfig, IServerConfig config)
        {
            m_PolicyFile = config.Options.GetValue("policyFile");

            if (string.IsNullOrEmpty(m_PolicyFile))
            {
                if(Logger.IsErrorEnabled)
                    Logger.Error("Configuration option policyFile is required!");
                return false;
            }

            return true;
        }

        ......
    }

B -

    public class RemoteProcessSession : AppSession<RemoteProcessSession>
    {
         protected override void HandleUnknownRequest(StringRequestInfo requestInfo)
        {
            Logger.Error("Unknow request");
        }
    }

## 扩展你的 Logger
SuperSocket 允许你自定义你的 Logger。 例如，你如果想要把你的业务操作日志保存到一个独立的地方，你仅需要在log4net配置文件中添加一个新的 logger 并为这个 logger 设置相应的 appender（假设你默认使用log4net）：

    <appender name="myBusinessAppender">
        <!--Your appender details-->
    </appender>
    <logger name="MyBusiness" additivity="false">
      <level value="ALL" />
      <appender-ref ref="myBusinessAppender" />
    </logger>


然后在代码中创建这个logger实例：

    var myLogger = server.LogFactory.GetLog("MyBusiness");



## 使用除 log4net 之外的日志框架

SuperSocket 支持你通过接口实现自己的log factory：

    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Text;

    namespace SuperSocket.SocketBase.Logging
    {
        /// <summary>
            /// LogFactory Interface
        /// </summary>
        public interface ILogFactory
        {
            /// <summary>
            /// Gets the log by name.
            /// </summary>
            /// <param name="name">The name.</param>
            /// <returns></returns>
            ILog GetLog(string name);
        }
    }

接口 ILogFactory 和 ILog 定义在 SuperSocket 之中。

在你实现你你自己的 log factory之后，你需要在配置文件中启用它：

    <superSocket logFactory="ConsoleLogFactory">
        <servers>
          <server name="EchoServer" serverTypeName="EchoService">
            <listeners>
              <add ip="Any" port="80" />
            </listeners>
          </server>
        </servers>
        <serverTypes>
          <add name="EchoService"
               type="SuperSocket.QuickStart.EchoService.EchoServer, SuperSocket.QuickStart.EchoService" />
        </serverTypes>
        <logFactories>
          <add name="ConsoleLogFactory"
               type="SuperSocket.SocketBase.Logging.ConsoleLogFactory, SuperSocket.SocketBase" />
        </logFactories>
      </superSocket>


在 SuperSocket 扩展项目之中，已经有一个使用 **Enterprise Library Logging Application Block** 的 SuperSocket LogFactory实现：
[http://supersocketext.codeplex.com/](http://supersocketext.codeplex.com/)