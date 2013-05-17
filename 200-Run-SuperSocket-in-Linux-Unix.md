# Run SuperSocket in Linux/Unix

> __Keywords__: Mono, Linux, Unix, Mono Service, Cross Platform

## SuperSocket supports cross-platform compatibility (Unix/Linux) of .NET applications by Mono (Mono 2.10 or later version)

As the Unix/Linux has different file path format with Windows, SuperSocket provides a different log4net configuration file (/Solution Items/log4net.unix.config) for Unix/Linux systems.

Therefore, you need to include this file to your project in subdirectory "Config" of output directory.

In Unix/Linux operating system, SuperSocket also can run as a console application or a service (Mono Service) like it in Windows.

**Console:**

    mono SuperSocket.SocketService.exe


**Mono Service:**

    mono-service -l:supersocket.lock -m:supersocket.log -d:<workdir> SuperSocket.SocketService.exe

The parameter &lt;workdir> is required, it is the root of your application where the file SuperSocket.SocketService.exe locates.
