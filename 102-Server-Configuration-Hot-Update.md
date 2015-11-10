# Server Configuration Hot Update

> __Keywords__: Configuration, Hot Update

This feature allow you to update the server instance's configuration immediately without restarting.  **(only for version 1.6.5 or above)**

## The configuration options which support hot update

SuperSocket supports the server configuration options below to be updated hotly:

    * logCommand
	* idleSessionTimeOut
	* maxRequestLength
	* logBasicSessionActivity
	* logAllSocketException


## SuperSocket supports hot update for all customized configuration attributes and child nodes

The code below will show you how hot update can be supported for your customized confguration:

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
	
	
You can find the full sample code in the PushServer project in the QuickStart.