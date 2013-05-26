# 在Linux/Unix上运行SuperSocket

> 关键字: Linux, Unix, Mono, Mono Service

## SuperSocket通过(Mono 2.10或更新版本)来实现跨平台的特性

由于Unix/Linux不同于Windows上的文件路径格式，SuperSocket提供了专用于Unix/Linux系统上的log4net文件：/Solution Items/log4net.unix.config

因此，你需要将此文件包含到你的项目输出目录的Config子目录下。

在Unix/Linux操作系统中，SuperSocket同样可以通过Mono以控制台和服务(Mono Service)这两种形式运行.

**控制台:**

    mono SuperSocket.SocketService.exe


**Mono Service:**

    mono-service -l:supersocket.lock -m:supersocket.log -d:<workdir> SuperSocket.SocketService.exe

命令中参数 &lt;workdir> 是必须的，它是指你的可执行程序SuperSocket.SocketService.exe所在的目录。
