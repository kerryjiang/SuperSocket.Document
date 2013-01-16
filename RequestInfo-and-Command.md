## What's the RequestInfo?

RequestInfo is the entity class which represents a request from the client. Each request of client should be instantiated as a RequestInfo. The RequestInfo class must implement the interface IRequestInfo which only have a property named "Key" in string type. 

## StringRequestInfo, the default RequestInfo


## Customize your RequestInfo
You can implement your own RequestInfo class as your application requirement. For instance, if all of your requests must have a DeviceID field, you can define a property for it in the RequestInfo class:

    public class MuRequestInfo : IRequestInfo
    {
         public string Key { get; set; }

         public int DeviceId { get; set; }

         /*
         // Other properties
         */
    }




## Relationship between RequestInfo and Command