# 用动态语言实现你的命令

> 关键字: 动态语言, Dynamic Language, IronPython, IronRuby, 脚本, Dynamic Commands, 动态命令


## 为你的 SuperSocket 启用动态语言
步骤如下:

1. 添加 DLR (dynamic language runtime) 配置片段;

    Section 定义:

        <section name="microsoft.scripting" requirePermission="false"
             type="Microsoft.Scripting.Hosting.Configuration.Section, Microsoft.Scripting"/>

    Section 内容:
        
        <microsoft.scripting>
            <languages>
                <language extensions=".py" displayName="IronPython"
                    type="IronPython.Runtime.PythonContext, IronPython"
                    names="IronPython;Python;py"/>
            </languages>
        </microsoft.scripting>

2. 增加 DLR 命令加载器;

        <SuperSocket>
            ......
            <commandLoaders>
                <add name="dynamicCommandLoader" type="SuperSocket.Dlr.DynamicCommandLoader, SuperSocket.Dlr"/>
            </commandLoaders>
        </superSocket>

3. 为你的服务器实例启用该命令加载器:

        <servers>
          <server name="IronPythonServer"
              serverTypeName="IronPythonService"
              ip="Any" port="2012"
              maxConnectionNumber="50"
              commandLoader="dynamicCommandLoader">
          </server>
        </servers>

完整的配置如下:

    <?xml version="1.0"?>
    <configuration>
      <configSections>
        <section name="superSocket" type="SuperSocket.SocketEngine.Configuration.SocketServiceConfig, SuperSocket.SocketEngine" />
        <section name="microsoft.scripting" requirePermission="false"
                 type="Microsoft.Scripting.Hosting.Configuration.Section, Microsoft.Scripting"/>
      </configSections>
      <appSettings>
        <add key="ServiceName" value="SupperSocketService" />
      </appSettings>
      <connectionStrings/>
      <superSocket>
        <servers>
          <server name="IronPythonServer"
              serverTypeName="IronPythonService"
              ip="Any" port="2012"
              maxConnectionNumber="50"
              commandLoader="dynamicCommandLoader">
          </server>
        </servers>
        <serverTypes>
          <add name="IronPythonService"
           type="SuperSocket.QuickStart.IronSocketServer.DynamicAppServer, SuperSocket.QuickStart.IronSocketServer" />
        </serverTypes>
        <commandLoaders>
            <add name="dynamicCommandLoader" type="SuperSocket.Dlr.DynamicCommandLoader, SuperSocket.Dlr"/>
        </commandLoaders>
      </superSocket>
      <microsoft.scripting>
        <languages>
          <language extensions=".py" displayName="IronPython"
                type="IronPython.Runtime.PythonContext, IronPython"
                names="IronPython;Python;py"/>
        </languages>
      </microsoft.scripting>
      <startup>
        <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.0" />
      </startup>
    </configuration>


## 确保所有的程序集在你的工作目录都存在
所需的文件如下:
* Microsoft.Dynamic.dll
* Microsoft.Scripting.dll
* IronPython.dll
* SuperSocket.Dlr.dll

## 实现你的命令
现在, 如果你有一个命令行协议的服务器实例 "IronPythonServer", 而且我们要用 Python 创建一个  "ADD" 命令用于让两个整数相加，然后把计算结果返回给客户端, 我们就要完成下面的步骤:

1. 创建 python 脚本文件 "ADD.py"， 并书写如下内容:

        def execute(session, request):
	        session.Send(str(int(request[0]) + int(request[1])))

2. 将此文件放入工作目录的子目录 "Command" 里
3. 
        WorkRoot -> Command -> ADD.py

3. 启动服务器，并通过Telnet客户端验证功能

        telnet 127.0.0.1 2012
        Client: ADD 100 150
        Server: 250


你会发现 ADD.py 位于 Command 文件夹的根目录, 因此 SuperSocket 允许所有实例加载它. 如果你只想要服务器实例 "IronPythonServer" 使用它, 你应该把脚本文件放到Command目录的子目录 "IronPythonServer" 里面:

    WorkRoot -> Command -> IronPythonServer -> ADD.py

## 动态命令的更新

SuperSocket每5分钟检查一次Command文件夹的更新. 如果你有任何包括新增，更新，删除的命令文件的操作, SuperSocket 将会在 5 分钟之内应用这些改动.