# SuperSocket 中的几个基本概念
> __Keywords__: 基本概念

## Package Type（包类型）

包类型代表从网络另一段收到的数据包的数据结构。

包类型 *TextPackageInfo*(SuperSocket.ProtoBase.TextPackageInfo,SuperSocket.ProtoBase) 是在 SuperSocket 中定义的最简单的数据包。它代表这类型的包仅包含一个字符串。


    public class TextPackageInfo
    {
        public string Text { get; set; }
    }


但是我们通常会有更复杂的网络数据包结构。例如，下面定义的包类型告诉我们此数据包包含两个字段，Sequence 和 Body。


    public class MyPackage
    {
        public short Sequence { get; set; }

        public string Body { get; set; }
    }

有的包会有一个特殊的字段来代表此包内容的类型。我们将此字段命名为 "Key"。此字段也告诉我们用何种逻辑处理此类型的包。这是在网络应用程序中非常常见的一种设计。例如，你的 Key 字段是整数类型，你的包类型需要实现接口 IKeyedPackageInfo<int>：

    public class MyPackage : IKeyedPackageInfo<int>
    {
        public int Key { get; set; }

        public short Sequence { get; set; }

        public string Body { get; set; }
    }

在后面的文档中，我们会解析如何根据 Key 字段的值定义命令来处理不不同种类的数据包。

## PipelineFilter Type

这种类型在网络协议解析中作用重要。它定义了如何将 IO 数据流解码成可以被应用程序理解的数据包。

这些是 PipelineFilter 的基本接口。你的系统中至少需要一个实现这个接口的 PipelineFilter 类型。


    public interface IPipelineFilter
    {
        void Reset();

        object Context { get; set; }        
    }

    public interface IPipelineFilter<TPackageInfo> : IPipelineFilter
        where TPackageInfo : class
    {
        
        IPackageDecoder<TPackageInfo> Decoder { get; set; }

        TPackageInfo Filter(ref SequenceReader<byte> reader);

        IPipelineFilter<TPackageInfo> NextFilter { get; }
        
    }

事实上，由于 SuperSocket 已经提供了一些内置的 PipelineFilter 模版，这些几乎可以覆盖 90% 的场景的模版极大的简化了你的开发工作。所以你不需要完全从头开始实现 PipelineFilter。即使这些内置的模版无法满足你的需求，完全自己实现PipelineFilter也不是难事。

**CommandLinePipelineFilter** (SuperSocket.ProtoBase.CommandLinePipelineFilter, SuperSocket.ProtoBase) 是在我们最常用的内置PipleFilter模版之一。我们将会在文档和示例代码中经常使用它。


## 使用 Package Type 和 PipelineFilter Type 创建 SuperSocket 宿主

你定义好 Package 类型和 PipelineFilter 类型之后，你就可以使用 SuperSocketHostBuilder 创建 SuperSocket 宿主了。


    var host = SuperSocketHostBuilder.Create<StringPackageInfo, CommandLinePipelineFilter>();


在某些情况下，你可能需要实现接口 IPipelineFilterFactory<TPackageInfo> 来完全控制 PipelineFilter 的创建。


    public class MyFilterFactory : PipelineFilterFactoryBase<TextPackageInfo>
    {
        protected override IPipelineFilter<TPackageInfo> CreateCore(object client)
        {
            return new FixedSizePipelineFilter(10);
        }
    }

然后在 SuperSocket 宿主被创建出来之后启用这个 PipelineFilterFactory:

    var host = SuperSocketHostBuilder.Create<StringPackageInfo>();
    host.UsePipelineFilterFactory<MyFilterFactory>();
