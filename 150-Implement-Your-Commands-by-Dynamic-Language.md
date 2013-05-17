# Implement Your Commands by Dynamic Language

> __Keywords__: Dynamic Language, IronPython, IronRuby, Script, Dynamic Commands

## Enable dynamic language for your SuperSocket
There are many steps:

1. Add DLR (dynamic language runtime) configuration section;

    Section definition:

        <section name="microsoft.scripting" requirePermission="false"
             type="Microsoft.Scripting.Hosting.Configuration.Section, Microsoft.Scripting"/>

    Section content:
        
        <microsoft.scripting>
            <languages>
                <language extensions=".py" displayName="IronPython"
                    type="IronPython.Runtime.PythonContext, IronPython"
                    names="IronPython;Python;py"/>
            </languages>
        </microsoft.scripting>

2. Add command loader for DLR;

        <SuperSocket>
            ......
            <commandLoaders>
                <add name="dynamicCommandLoader" type="SuperSocket.Dlr.DynamicCommandLoader, SuperSocket.Dlr"/>
            </commandLoaders>
        </superSocket>

3. Use the command loader for your server instances:

        <servers>
          <server name="IronPythonServer"
              serverTypeName="IronPythonService"
              ip="Any" port="2012"
              maxConnectionNumber="50"
              commandLoader="dynamicCommandLoader">
          </server>
        </servers>

The full configuration file will be:

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


## Ensure all required assemblies exist in the working directory
The files below are required:
* Microsoft.Dynamic.dll
* Microsoft.Scripting.dll
* IronPython.dll
* SuperSocket.Dlr.dll

## Implement your commands
Now, if we have a command line protocol SuperSocket server instance "IronPythonServer", and we want to create a "ADD" command in Python for adding two integers and then send back the result to client, we should the following steps:

1. Create a python script file named as "ADD.py" with the content below:

        def execute(session, request):
	        session.Send(str(int(request[0]) + int(request[1])))

2. Put this file into the sub directory "Command" of the working directory

        WorkRoot -> Command -> ADD.py

3. Start the server, and verify the function by telnet

        telnet 127.0.0.1 2012
        Client: ADD 100 150
        Server: 250


You can find we put the file ADD.py in root of the Command folder, therefore SuperSocket allow all server instances load it. If you want to only allow the server instance "IronPythonServer" to use it, you should put this file in the sub directory "IronPythonServer" of the command folter:

    WorkRoot -> Command -> IronPythonServer -> ADD.py

## Updating of the dynamic commands

The SuperSocket checks the updates of the command folder in the interval of 5 minutes. So if you have any command updates including adding, updating or removing, SuperSocket will adopt your changes within 5 minutes.