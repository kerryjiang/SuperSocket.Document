# 实现你的 PipelineFilter

> __Keywords__: PipelineFilter


## 内置的 PipelineFilter 模版

阅读了前面一篇文档之后, 你可能会觉得用 SuperSocket 来实现你的自定义协议并不简单。但事实上，由于 SuperSocket 已经提供了一些内置的 PipelineFilter 模版，这些几乎可以覆盖 90% 的场景的模版极大的简化了你的开发工作。所以你不需要完全从头开始实现 PipelineFilter。即使这些内置的模版无法满足你的需求，完全自己实现PipelineFilter也不是难事

SuperSocket 提供了这些 PipelineFilter 模版:

* **TerminatorPipelineFilter** (SuperSocket.ProtoBase.TerminatorPipelineFilter, SuperSocket.ProtoBase)
* **TerminatorTextPipelineFilter** (SuperSocket.ProtoBase.TerminatorTextPipelineFilter, SuperSocket.ProtoBase)
* **LinePipelineFilter** (SuperSocket.ProtoBase.LinePipelineFilter, SuperSocket.ProtoBase)
* **CommandLinePipelineFilter** (SuperSocket.ProtoBase.CommandLinePipelineFilter, SuperSocket.ProtoBase)
* **BeginEndMarkPipelineFilter** (SuperSocket.ProtoBase.BeginEndMarkPipelineFilter, SuperSocket.ProtoBase)
* **FixedSizePipelineFilter** (SuperSocket.ProtoBase.FixedSizePipelineFilter, SuperSocket.ProtoBase)
* **FixedHeaderPipelineFilter** (SuperSocket.ProtoBase.FixedHeaderPipelineFilter, SuperSocket.ProtoBase)


## 如何基于内置模版实现 PipelineFilter

FixedHeaderPipelineFilter - 头部格式固定并且包含内容长度的协议

这种协议将一个请求定义为两大部分, 第一部分定义了包含第二部分长度等等基础信息. 我们通常称第一部分为头部.

例如, 我们有一个这样的协议: 头部包含 3 个字节, 第 1 个字节用于存储请求的类型, 后两个字节用于代表请求体的长度:

    /// +-------+---+-------------------------------+
    /// |request| l |                               |
    /// | type  | e |    request body               |
    /// |  (1)  | n |                               |
    /// |       |(2)|                               |
    /// +-------+---+-------------------------------+


根据此协议的规范, 我们可以使用如下代码定义包的类型:

    public class MyPackage
    {
        public byte Key { get; set; }

        public string Body { get; set; }
    }


下一个是设计PipelineFilter:

    public class MyPipelineFilter : FixedHeaderPipelineFilter<MyPackage>
    {
        public MyPipelineFilter()
            : base(3) // 包头的大小是3字节，所以将3传如基类的构造方法中去
        {

        }

        // 从数据包的头部返回包体的大小
        protected override int GetBodyLengthFromHeader(ref ReadOnlySequence<byte> buffer)
        {
            var reader = new SequenceReader<byte>(buffer);
            reader.Advance(1); // skip the first byte
            reader.TryReadBigEndian(out short len);
            return len;
        }
        
        // 将数据包解析成 MyPackage 的实例
        protected override MyPackage DecodePackage(ref ReadOnlySequence<byte> buffer)
        {
            var package = new MyPackage();

            var reader = new SequenceReader<byte>(buffer);
            
            reader.TryRead(out byte packageKey);
            package.Key = packageKey;            
            reader.Advance(2); // skip the length             
            package.Body = reader.ReadString();

            return package;
        }
    }

最后，你可通过数据包的类型和 PipelineFilter 的类型来创建宿主:

    var host = SuperSocketHostBuilder.Create<MyPackage, MyPipelineFilter>()
        .UsePackageHandler(async (s, p) =>
        {
            // handle your package over here
        }).Build();


你也可以通过将解析包的代码从 PipelineFilter 移到 你的包解码器中来获得更大的灵活性：

    public class MyPackageDecoder : IPackageDecoder<MyPackage>
    {
        public MyPackage Decode(ref ReadOnlySequence<byte> buffer, object context)
        {
            var package = new MyPackage();

            var reader = new SequenceReader<byte>(buffer);
            
            reader.TryRead(out byte packageKey);
            package.Key = packageKey;            
            reader.Advance(2); // skip the length             
            package.Body = reader.ReadString();

            return package;
        }
    }

通过 host builder 的 UsePackageDecoder 方法来在SuperSocket中启用它:

    builder.UsePackageDecoder<MyPackageDecoder>();


