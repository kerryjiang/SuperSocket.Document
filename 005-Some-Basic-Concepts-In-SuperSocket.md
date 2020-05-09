# Some basic Concepts in SuperSocket
> __Keywords__: basic concepts

## Package Type

The package type represents the data structure of the package we receive from the other end.

The type *TextPackageInfo*(SuperSocket.ProtoBase.TextPackageInfo,SuperSocket.ProtoBase) is the simplest defined package type within SuperSocket. It only standards for a text string.


    public class TextPackageInfo
    {
        public string Text { get; set; }
    }


But we usually have more complex network package structure than that. For instance, the package type is defined below tells us their package has two fields, Sequence and Body.

    public class MyPackage
    {
        public short Sequence { get; set; }

        public string Body { get; set; }
    }

Some packages may have a special field which can tell which kind of package it is. We call this field "key" here. This key field also tells which kind logic should handle the pakcgae. It is very common scenario in the design of network applications. In this case, you just need implement the interface IKeyedPackageInfo<int> for your package type if the key is an integer:

    public class MyPackage : IKeyedPackageInfo<int>
    {
        public int Key { get; set; }

        public short Sequence { get; set; }

        public string Body { get; set; }
    }

In the following parts of the document, we will explain how to define commands to handle different kinds of packages according the value in the key field.


## PipelineFilter Type

This kind type plays an important role in the network protocol decoding, which defines how we decode IO stream into packages which can be understanded by the application.

These are the basic interfaces for PipelineFilter. At least one PipelineFilter type which implement this interface is required in the system.


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

Actually, you don't need implement PipelineFilter by yourself from scratch, because SuperSocket provides at many built-in PipelineFilter templates (base classes) which almost cover 90% of the cases and simplify your development work significantly. Even if the templates are not fitting your requirement, developing a PipelineFilter completely by yourself should be easy as well.

These built-in PipelineFilter templates are:

* **TerminatorPipelineFilter** (SuperSocket.ProtoBase.TerminatorPipelineFilter, SuperSocket.ProtoBase)
* **TerminatorTextPipelineFilter** (SuperSocket.ProtoBase.TerminatorTextPipelineFilter, SuperSocket.ProtoBase)
* **LinePipelineFilter** (SuperSocket.ProtoBase.LinePipelineFilter, SuperSocket.ProtoBase)
* **CommandLinePipelineFilter** (SuperSocket.ProtoBase.CommandLinePipelineFilter, SuperSocket.ProtoBase)
* **BeginEndMarkPipelineFilter** (SuperSocket.ProtoBase.BeginEndMarkPipelineFilter, SuperSocket.ProtoBase)
* **FixedSizePipelineFilter** (SuperSocket.ProtoBase.FixedSizePipelineFilter, SuperSocket.ProtoBase)
* **FixedHeaderPipelineFilter** (SuperSocket.ProtoBase.FixedHeaderPipelineFilter, SuperSocket.ProtoBase)


## Create the SuperSocket Host with Package Type and PipelineFilter Type

After you define your package info type and the pipeline filter type, you should be able to create a SuperSocket host by SuperSocketHostBuilder.


    var host = SuperSocketHostBuilder.Create<StringPackageInfo, CommandLinePipelineFilter>();


In some cases, you may need implement IPipelineFilterFactory<TPackageInfo> to get full control of  creating of PipelineFilter.


    public class MyFilterFactory : PipelineFilterFactoryBase<TextPackageInfo>
    {
        protected override IPipelineFilter<TPackageInfo> CreateCore(object client)
        {
            return new FixedSizePipelineFilter(10);
        }
    }

Then you will need to use the PipelineFilterFactory after your create the SuperSocket host:

    var host = SuperSocketHostBuilder.Create<StringPackageInfo>();
    host.UsePipelineFilterFactory<MyFilterFactory>();




