# 内置的常用协议实现工具

> 关键字: TerminatorReceiveFilter, CountSpliterReceiveFilter, FixedSizeReceiveFilter, BeginEndMarkReceiveFilter, FixedHeaderReceiveFilter

阅读了前面一篇文档之后, 你可能会觉得用 SuperSocket 来实现你的自定义协议并不简单。 为了让这件事变得更容易一些, SuperSocket 提供了一些通用的协议解析工具, 你可以用他们简单而且快速的实现你自己的通信协议:

* **TerminatorReceiveFilter** (SuperSocket.SocketBase.Protocol.TerminatorReceiveFilter, SuperSocket.SocketBase)
* **CountSpliterReceiveFilter** (SuperSocket.Facility.Protocol.CountSpliterReceiveFilter, SuperSocket.Facility)
* **FixedSizeReceiveFilter** (SuperSocket.Facility.Protocol.FixedSizeReceiveFilter, SuperSocket.Facility)
* **BeginEndMarkReceiveFilter** (SuperSocket.Facility.Protocol.BeginEndMarkReceiveFilter, SuperSocket.Facility)
* **FixedHeaderReceiveFilter** (SuperSocket.Facility.Protocol.FixedHeaderReceiveFilter, SuperSocket.Facility)

## TerminatorReceiveFilter - 结束符协议

与命令行协议类似，一些协议用结束符来确定一个请求.

例如, 一个协议使用两个字符 "##" 作为结束符, 于是你可以使用类 "TerminatorReceiveFilterFactory":

    /// <summary>
    /// TerminatorProtocolServer
    /// Each request end with the terminator "##"
    /// ECHO Your message##
    /// </summary>
    public class TerminatorProtocolServer : AppServer
    {
        public TerminatorProtocolServer()
            : base(new TerminatorReceiveFilterFactory("##"))
        {
                
        }
    }

默认的请求类型是 StringRequestInfo, 你也可以创建自己的请求类型, 不过这样需要你做一点额外的工作:

基于TerminatorReceiveFilter实现你的接收过滤器(ReceiveFilter):

    public class YourReceiveFilter : TerminatorReceiveFilter<YourRequestInfo>
    {
        //More code
    }

实现你的接收过滤器工厂(ReceiveFilterFactory)用于创建接受过滤器实例:

    public class YourReceiveFilterFactory : IReceiveFilterFactory<YourRequestInfo>
    {
        //More code
    }

然后在你的 AppServer 中使用这个接收过滤器工厂(ReceiveFilterFactory).


## CountSpliterReceiveFilter - 固定数量分隔符协议

有些协议定义了像这样格式的请求 "#part1#part2#part3#part4#part5#part6#part7#". 每个请求有7个由 '#' 分隔的部分. 这种协议的实现非常简单:
        
    /// <summary>
    /// Your protocol likes like the format below:
    /// #part1#part2#part3#part4#part5#part6#part7#
    /// </summary>
    public class CountSpliterAppServer : AppServer
    {
        public CountSpliterAppServer()
            : base(new CountSpliterReceiveFilterFactory((byte)'#', 8)) // 7 parts but 8 separators
        {
            
        }
    }

你也可以使用下面的类更深入的定制这种协议:

    CountSpliterReceiveFilter<TRequestInfo>
    CountSpliterReceiveFilterFactory<TReceiveFilter>
    CountSpliterReceiveFilterFactory<TReceiveFilter, TRequestInfo>


## FixedHeaderReceiveFilter - 头部格式固定并且包含内容长度的协议

这种协议将一个请求定义为两大部分, 第一部分定义了包含第二部分长度等等基础信息. 我们通常称第一部分为头部.

例如, 我们有一个这样的协议: 头部包含 6 个字节, 前 4 个字节用于存储请求的名字, 后两个字节用于代表请求体的长度:

    /// +-------+---+-------------------------------+
    /// |request| l |                               |
    /// | name  | e |    request body               |
    /// |  (4)  | n |                               |
    /// |       |(2)|                               |
    /// +-------+---+-------------------------------+

使用 SuperSocket, 你可以非常方便的实现这种协议:

    class MyReceiveFilter : FixedHeaderReceiveFilter<BinaryRequestInfo>
    {
        public MyReceiveFilter()
            : base(6)
        {

        }

        protected override int GetBodyLengthFromHeader(byte[] header, int offset, int length)
        {
            return (int)header[offset + 4] * 256 + (int)header[offset + 5];
        }

        protected override BinaryRequestInfo ResolveRequestInfo(ArraySegment<byte> header, byte[] bodyBuffer, int offset, int length)
        {
            return new BinaryRequestInfo(Encoding.UTF8.GetString(header.Array, header.Offset, 4), bodyBuffer.CloneRange(offset, length));
        }
    }


你需要基于类FixedHeaderReceiveFilter<TRequestInfo>实现你自己的接收过滤器.

* 传入父类构造函数的 6 表示头部的长度;
* 方法"GetBodyLengthFromHeader(...)" 应该根据接收到的头部返回请求体的长度;
* 方法 ResolveRequestInfo(....)" 应该根据你接收到的请求头部和请求体返回你的请求类型的实例.

然后你就可以使用接收或者自己定义的接收过滤器工厂来在 SuperSocket 中启用该协议.