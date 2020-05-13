# Implement your PipelineFilter

> __Keywords__: PipelineFilter


## The Built-in PipelineFilter Templates

After reading the previous document, you probably find implementing your own protocol using SuperSocket is not easy for you. But actually, you don't need implement PipelineFilter by yourself from scratch, because SuperSocket provides already many built-in PipelineFilter templates (base classes) which almost cover 90% of the cases and simplify your development work significantly. Even if the templates are not fitting your requirement, developing a PipelineFilter completely by yourself should be easy as well.

SuperSocket povides these built-in PipelineFilter templates:

* **TerminatorPipelineFilter** (SuperSocket.ProtoBase.TerminatorPipelineFilter, SuperSocket.ProtoBase)
* **TerminatorTextPipelineFilter** (SuperSocket.ProtoBase.TerminatorTextPipelineFilter, SuperSocket.ProtoBase)
* **LinePipelineFilter** (SuperSocket.ProtoBase.LinePipelineFilter, SuperSocket.ProtoBase)
* **CommandLinePipelineFilter** (SuperSocket.ProtoBase.CommandLinePipelineFilter, SuperSocket.ProtoBase)
* **BeginEndMarkPipelineFilter** (SuperSocket.ProtoBase.BeginEndMarkPipelineFilter, SuperSocket.ProtoBase)
* **FixedSizePipelineFilter** (SuperSocket.ProtoBase.FixedSizePipelineFilter, SuperSocket.ProtoBase)
* **FixedHeaderPipelineFilter** (SuperSocket.ProtoBase.FixedHeaderPipelineFilter, SuperSocket.ProtoBase)


## How to Implement PipelineFilter base on the built-in Templates 

FixedHeaderPipelineFilter - Fixed Header with Body Length Protocol

This kind protocol defines each request has two parts, the first part contains some basic information of this request including the length of the second part. We usually call the first part is header and the second part is body.

For example, we have a protocol like that: the header contains 3 bytes, the first byte represent the request's type, the next 2 bytes represent the length of the body. And the body is a text string.

    /// +-------+---+-------------------------------+
    /// |request| l |                               |
    /// | type  | e |    request body               |
    /// |  (1)  | n |                               |
    /// |       |(2)|                               |
    /// +-------+---+-------------------------------+


According the protocol specification, we can design the package type like the code below:

    public class MyPackage
    {
        public byte Key { get; set; }

        public string Body { get; set; }
    }


Next step is to design the PipelineFilter:

    public class MyPipelineFilter : FixedHeaderPipelineFilter<MyPackage>
    {
        public MyPipelineFilter()
            : base(3) // because the header size is 3, so pass 3 to the parent class's constructor
        {

        }

        protected override int GetBodyLengthFromHeader(ref ReadOnlySequence<byte> buffer)
        {
            var reader = new SequenceReader<byte>(buffer);
            reader.Advance(1); // skip the first byte
            reader.TryReadBigEndian(out short len);
            return len;
        }

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

Finally, you can use the Package type and the PipelineFilter type to build the host:

    var host = SuperSocketHostBuilder.Create<MyPackage, MyPipelineFilter>()
        .UsePackageHandler(async (s, p) =>
        {
            // handle your package over here
        }).Build();


You also can get more flexiblity by moving the code about package decoding from PipelineFilter to package decoder:

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


Use the method UsePackageDecoder of the host builder to enable it in SuperSocket:

    builder.UsePackageDecoder<MyPackageDecoder>();


