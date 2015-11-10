# 服务器配置热更新

> __Keywords__: 配置,热更新

此功能能够允许你在不重启服务器的前提下更新服务器实例的配置。 **(仅限1.6.5及其以上版本)**

## 支持热更新的服务器实例配置选项

SuperSocket 支持以下配置选项的热更新:

    * logCommand
	* idleSessionTimeOut
	* maxRequestLength
	* logBasicSessionActivity
	* logAllSocketException


## SuperSocket 支持所有自定义配置属性和自定义配置子节点的热更新。

下面的代码将演示如何让你的自定义配置支持热更新:

	public class PushServer : AppServer
    {
        private int m_Interval;

        protected override bool Setup(IRootConfig rootConfig, IServerConfig config)
        {
            RegisterConfigHandler(config, "pushInterval", (value) =>
                {
					// the code in this scope will be executed automatically
					// after the configuration attribute "pushInterval" is changed
					
                    var interval = 0;
                    int.TryParse(value, out interval);

                    if (interval <= 0)
                        interval = 60;// 60 seconds by default

                    m_Interval = interval * 1000;
                    return true;
                });

            return true;
        }
		
		/// Other code
    }
	
	
你可以在QuickStart中的 PushServer 项目中找到此更能的完整示例代码。