# 内置的命令行PipelineFilter

> 关键字: 命令行, 命令行协议, Telnet, CommandLinePipelineFilter

## 什么是协议?

什么是协议? 很多人会回答 "TCP" 或者 "UDP"。 但是构建一个网络应用程序, 仅仅知道是 TCP 还是 UDP 是远远不够的。 TCP 和 UDP 是传输层协议。仅仅定义了传输层协议是不能让网络的两端进行通信的。你需要定义你的应用层通信协议把你接收到的二进制数据转化成你程序能理解的请求。

## 内置的命令行协议

命令行协议是一种被广泛应用的协议。一些成熟的协议如 Telnet, SMTP, POP3 和 FTP 都是基于命令行协议的。 CommandLinePipelineFilter是设计用于命令行协议的 PipelineFilter。

命令行协议定义了每个请求必须以回车换行结尾 "\r\n"。

如果你在 SuperSocket 中使用命令行协议，所有接收到的数据将会翻译成 StringRequestInfo 实例。

StringRequestInfo 是这样定义的:

    public class StringPackageInfo : IKeyedPackageInfo<string>, IStringPackage
    {
        public string Key { get; set; }

        public string Body { get; set; }

        public string[] Parameters { get; set; }
    }

由于 SuperSocket 中内置的命令行协议用空格来分割请求的Key和参，因此当客户端发送如下数据到服务器端时:

    "LOGIN kerry 123456" + NewLine

SuperSocket 服务器将会收到一个 StringRequestInfo 实例，这个实例的属性为:

    Key: "LOGIN"
    Body: "kerry 123456";
    Parameters: ["kerry", "123456"]

如果你定义了名为 "LOGIN" 的命令, 这个命令的 ExecuteCommand 方法将会被执行，服务器所接收到的StringRequestInfo实例也将作为参数传给这个方法:

    public class LOGIN : IAsyncCommand<StringPackageInfo>
    {
        public async ValueTask ExecuteAsync(IAppSession session, StringPackageInfo package, CancellationToken cancellationToken)
        {
            //Implement your business logic
        }
    }


## 自定义你的命令行协议

有些用户可能会有不同的请求格式, 比如:

    "LOGIN:kerry,12345" + NewLine

请求的 key 和 body 通过字符 ':' 分隔, 而且多个参数被字符 ',' 分隔。 支持这种类型的请求非常简单, 你只需要用下面的代码扩展命令行协议:

    public class CustomPackageDecoder : IPackageDecoder<StringPackageInfo>
    {
        public StringPackageInfo Decode(ref ReadOnlySequence<byte> buffer, object context)
        {
            var text = buffer.GetString(new UTF8Encoding(false));
            var parts = text.Split(':', 2);

            return new StringPackageInfo
            {
                Key = parts[0],
                Body = text,
                Parameters = parts[1].Split(',')
            };
        }
    }

    // register the custom package decoder through the host builder

    builder.UsePackageDecoder<CustomPackageDecoder>();