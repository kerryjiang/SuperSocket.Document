# The Built-in Command Line PipelineFilter

> __Keywords__: Command Line, Protocol, StringRequestInfo, Text Encoding

## What's the Protocol?

What's the Protocol? Lots of people probably will answer "TCP" or "UDP". But to build a network application, only TCP or UDP is not enough. TCP and UDP are transport-layer protocols. It's far from enough to enable talking between two endpoints in the network if you only define transport-layer protocol. You need to define your application level protocol to convert your received binary data to the requests which your application can understand.

## The Built-in Command Line PipelineFilter

The command line protocol is a widely used protocols, lots of protocols like Telnet, SMTP, POP3 and FTP protocols are base on command line protocol etc. The CommandLinePipelineFilter is a PipelineFilter which was designed for command line protocol.

The command line protocol defines each request must be ended with a carriage return "\r\n".

If you use the command line protocol in SuperSocket, all requests will to translated into StringRequestInfo instances.

StringRequestInfo is defined like this:

    public class StringPackageInfo : IKeyedPackageInfo<string>, IStringPackage
    {
        public string Key { get; set; }

        public string Body { get; set; }

        public string[] Parameters { get; set; }
    }

Because the built-in command line protocol in SuperSocket uses a space to split request key and parameters,
So when the client sends the data below to the server:

    "LOGIN kerry 123456" + NewLine

the SuperSocket server will receive a StringRequestInfo instance, the properties of the request info instance will be:

    Key: "LOGIN"
    Body: "kerry 123456";
    Parameters: ["kerry", "123456"]


If you have defined a Command with name "LOGIN", the command's ExecuteCommand method will be excuted with the StringRequestInfo instance as parameter:

    public class LOGIN : IAsyncCommand<StringPackageInfo>
    {
        public async ValueTask ExecuteAsync(IAppSession session, StringPackageInfo package, CancellationToken cancellationToken)
        {
            //Implement your business logic
        }
    }


## Customize the Command Line Protocol

Some users might have different request format, for instance:

    "LOGIN:kerry,12345" + NewLine

The request's key is separated with body by the char ':', and the parameters are separated by the char ','. This kind of request can be supported easily, just extend the command line protocol like the below code:

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