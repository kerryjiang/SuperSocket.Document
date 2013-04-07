After reading the previous document, you probably find implementing your own protocol using SuperSocket probably is not easy for you. To make this job easier, SuperSocket provides some common protocol tools, which you can use to build your own protocol easily and fast:

* **TerminatorReceiveFilter**
* **CountSpliterReceiveFilter**
* **FixedSizeReceiveFilter**
* **BeginEndMarkReceiveFilter**
* **FixedHeaderReceiveFilter**

## TerminatorReceiveFilter - Terminator Protocol

Similar with command line protocol, some protocols use a terminator to identify a request.
For example, one protocol uses two chars "##" as terminator, then you can use the class "TerminatorReceiveFilterFactory":

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

The default RequestInfo is StringRequestInfo, you also can create your own RequestInfo class, but it requires a bit more work:

Implement your ReceiveFilter base on TerminatorReceiveFilter:

    public class YourReceiveFilter : TerminatorReceiveFilter<YourRequestInfo>
    {
        //More code
    }

Implement your ReceiveFilterFactory which can create your request filter instances:

    public class YourReceiveFilterFactory : IReceiveFilterFactory<YourRequestInfo>
    {
        //More code
    }

And then use the request filter factory in your AppServer.


## CountSpliterReceiveFilter - Fixed Number Split Parts with Separator Protocol

Some protocols defines their requests look like in the format of "#part1#part2#part3#part4#part5#part6#part7#". There are 7 parts in one request and all parts are separated by char '#'. This kind protocol's implementing also is quite easy:
        
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

You also can customize your protocol deeper using the classes below:

    CountSpliterReceiveFilter<TRequestInfo>
    CountSpliterReceiveFilterFactory<TReceiveFilter>
    CountSpliterReceiveFilterFactory<TReceiveFilter, TRequestInfo>


## FixedHeaderReceiveFilter - Fixed Header with Body Length Protocol

This kind protocol defines each request has two parts, the first part contains some basic information of this request include the length of the second part. We usually call the first part is header and the second part is body.

For example, we have a protocol like that: the header contains 6 bytes, the first 4 bytes represent the request's name, the last 2 bytes represent the length of the body:

    /// +-------+---+-------------------------------+
    /// |request| l |                               |
    /// | name  | e |    request body               |
    /// |  (4)  | n |                               |
    /// |       |(2)|                               |
    /// +-------+---+-------------------------------+

Using SuperSocket, you can implement this kind protocol easily:

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


You need to implement your own request filter base on FixedHeaderReceiveFilter<TRequestInfo>.
* The number 6 passed into the parent class's constructor means the size of the request header;
* The method "GetBodyLengthFromHeader(...)" you should override returns the length of the body according the received header;
* the method "ResolveRequestInfo(....)" you should override returns the RequestInfo instance according the received header and body.

Then you can build a receive filter factory or use the default receive factory to use this receive filter in SuperSocket.