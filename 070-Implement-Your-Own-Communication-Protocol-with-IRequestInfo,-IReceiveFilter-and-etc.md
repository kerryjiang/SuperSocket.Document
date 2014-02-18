# 使用 IRequestInfo 和 IReceiveFilter 等等其他对象来实现自定义协议

> 关键字: IRequestInfo, IReceiveFilter, 自定义协议, 请求, 接收过滤器

## 为什么你要使用自定义协议?

通信协议用于将接收到的二进制数据转化成您的应用程序可以理解的请求。 SuperSocket提供了一个内置的通信协议“命令行协议”定义每个请求都必须以回车换行"\r\n"结尾。

但是一些应用程序无法使用命令行协议由于不同的原因。 这种情况下,你需要使用下面的工具来实现你的自定义协议:

    * RequestInfo
    * ReceiveFilter
    * ReceiveFilterFactory
    * AppServer and AppSession


## 请求(RequestInfo)
RequestInfo 是表示来自客户端请求的实体类。 每个来自客户端的请求都能应该被实例化为 RequestInfo 类型。 RequestInfo 类必须实现接口 IRequestInfo，该接口只有一个名为"Key"的字符串类型的属性:

    public interface IRequestInfo
    {
        string Key { get; }
    }

上面文档提到了, 请求类型 StringRequestInfo 用在 SuperSocket 命令行协议中。

你也可以根据你的应用程序的需要来定义你自己的请求类型。 例如, 如果所有请求都包含 DeviceID 信息，你可以在RequestInfo类里为它定义一个属性:

    public class MyRequestInfo : IRequestInfo
    {
         public string Key { get; set; }

         public int DeviceId { get; set; }

         /*
         // Other properties
         */
    }

SuperSocket 还提供了另外一个请求类 "BinaryRequestInfo" 用于二进制协议:

    public class BinaryRequestInfo
    {
        public string Key { get; }
        
        public byte[] Body { get; }
    }

你可以直接使用此类型 BinaryRequestInfo, 如果他能满足你的需求的话。

## 接收过滤器(ReceiveFilter)

接收过滤器(ReceiveFilter)用于将接收到的二进制数据转化成请求实例(RequestInfo)。

实现一个接收过滤器(ReceiveFilter), 你需要实现接口 IReceiveFilter<TRequestInfo>:

    public interface IReceiveFilter<TRequestInfo>
        where TRequestInfo : IRequestInfo
    {
        /// <summary>
        /// Filters received data of the specific session into request info.
        /// </summary>
        /// <param name="readBuffer">The read buffer.</param>
        /// <param name="offset">The offset of the current received data in this read buffer.</param>
        /// <param name="length">The length of the current received data.</param>
        /// <param name="toBeCopied">if set to <c>true</c> [to be copied].</param>
        /// <param name="rest">The rest, the length of the data which hasn't been parsed.</param>
        /// <returns></returns>
        TRequestInfo Filter(byte[] readBuffer, int offset, int length, bool toBeCopied, out int rest);

        /// <summary>
        /// Gets the size of the left buffer.
        /// </summary>
        /// <value>
        /// The size of the left buffer.
        /// </value>
        int LeftBufferSize { get; }

        /// <summary>
        /// Gets the next receive filter.
        /// </summary>
        IReceiveFilter<TRequestInfo> NextReceiveFilter { get; }

        /// <summary>
        /// Resets this instance to initial state.
        /// </summary>
        void Reset();
    }

* TRequestInfo: 类型参数 "TRequestInfo" 是你要在程序中使用的请求类型(RequestInfo);
* LeftBufferSize: 该接收过滤器已缓存数据的长度;
* NextReceiveFilter: 当下一块数据收到时，用于处理数据的接收过滤器实例;
* Reset(): 重设接收过滤器实例到初始状态;
* Filter(....): 该方法将会在 SuperSocket 收到一块二进制数据时被执行，接收到的数据在 readBuffer 中从 offset 开始， 长度为 length 的部分。


        TRequestInfo Filter(byte[] readBuffer, int offset, int length, bool toBeCopied, out int rest);


  * readBuffer: 接收缓冲区, 接收到的数据存放在此数组里
  * offset: 接收到的数据在接收缓冲区的起始位置
  * length: 本轮接收到的数据的长度
  * toBeCopied: 表示当你想缓存接收到的数据时，是否需要为接收到的数据重新创建一个备份而不是直接使用接收缓冲区
  * rest: 这是一个输出参数, 它应该被设置为当解析到一个为政的请求后，接收缓冲区还剩余多少数据未被解析

  这儿有很多种情况需要你处理:

  * 当你在接收缓冲区中找到一条完整的请求时，你必须返回一个你的请求类型的实例.
  * 当你在接收缓冲区中没有找到一个完整的请求时, 你需要返回 NULL.
  * 当你在接收缓冲区中找到一条完整的请求, 但接收到的数据并不仅仅包含一个请求时，设置剩余数据的长度到输出变量 "rest". SuperSocket 将会检查这个输出参数 "rest", 如果它大于 0, 此 Filter 方法 将会被再次执行, 参数 "offset" 和 "length" 会被调整为合适的值.

## 接收过滤器工厂(ReceiveFilterFactory)
接收过滤器工厂(ReceiveFilterFactory)用于为每个会话创建接收过滤器.
定义一个过滤器工厂(ReceiveFilterFactory)类型, 你必须实现接口 IReceiveFilterFactory<TRequestInfo>. 类型参数 "TRequestInfo" 是你要在整个程序中使用的请求类型

    /// <summary>
    /// Receive filter factory interface
    /// </summary>
    /// <typeparam name="TRequestInfo">The type of the request info.</typeparam>
    public interface IReceiveFilterFactory<TRequestInfo> : IReceiveFilterFactory
        where TRequestInfo : IRequestInfo
    {
        /// <summary>
        /// Creates the receive filter.
        /// </summary>
        /// <param name="appServer">The app server.</param>
        /// <param name="appSession">The app session.</param>
        /// <param name="remoteEndPoint">The remote end point.</param>
        /// <returns>
        /// the new created request filer assosiated with this socketSession
        /// </returns>
        IReceiveFilter<TRequestInfo> CreateFilter(IAppServer appServer, IAppSession appSession, IPEndPoint remoteEndPoint);
    }


你也可以直接使用默认的过滤器工厂(ReceiveFilterFactory)

    DefaultReceiveFilterFactory<TReceiveFilter, TRequestInfo>

, 当工厂的CreateFilter方法被调用时，它将会调用TReceiveFilter类型的无参构造方法来创建并返回TReceiveFilter.


## 和 AppSession，AppServer 配合工作

现在, 你已经有了 RequestInfo, ReceiveFilter 和 ReceiveFilterFactory, 但是你还没有正式使用它们.
如果你想让他们在你的程序里面可用, 你需要定义你们的 AppSession 和 AppServer 来使用他们.

* 为 AppSession 设置 RequestInfo

        public class YourSession : AppSession<YourSession, YourRequestInfo>
        {
             //More code...
        }


* 为 AppServer 设置 RequestInfo 和 ReceiveFilterFactory

        public class YourAppServer : AppServer<YourSession, YourRequestInfo>
        {
            public YourAppServer()
                : base(new YourReceiveFilterFactory())
            {
            
            }
        }


完成上面两件事情，你的自定义协议就应该可以工作了。