Netty uses asynchronous event driven model, various operations to trigger the network I/O, the driver model in socket layer encapsulates a layer of asynchronous events, making the business code does not need to care about the underlying network, no network I/O block code can be written asynchronously.

The concept of asynchronous event driven model of Netty mainly involves the following core:

*   Channel: Represents a connection with the socket channel
*   ChannelPipeline: The Conduit, A Channel has a ChannelPipeline, The ChannelPipeline maintains a processing chain (strictly speaking is two: upstream, downstream), The processing chain is composed of many handlers ChannelHandler, Each ChannelHandler finish to will be passed to the next handler in the chain to continue processing.
*   ChannelHandler: The handler, the user can define handlers themselves to handle each request, or request preprocessing methods, typical of the encoder / decoder: decoder, encoder.
*   ChannelEvent: The event, is the model of the object, when generating or trigger (fire) an event, the event will be along the ChannelPipeline chain are processing.
*   ChannelFuture: Asynchronous results, this is the key to handling events, when an event is processed, and can be used directly to the ChannelFuture in the form of direct return, be blocked not in current operation. Can get the final result by ChannelFuture, specific practices in the ChannelFuture to add the listener listener, when the operation was eventually executed, listener is triggered, we can be predefined our business code in the listener callback function.
*   Structure diagram of the model are as follows:
*   ![](http://www.programering.com/images/remote/ZnJvbT1pdGV5ZSZ1cmw9WVdhbjVDTjRnVFl6a2pZM1FXWjJnVEwwUUROaTFpTnpZek10WW1Nd1VXTGhOVFp5Y0RPM2d6THlFVE8zOHlNNUFETXZRbmJsMUdhakZHZDBGMkxrRjJic0JYZHYwMmJqNVNaNVZHZHA1aU1zUjJMdm9EYzBSSGE.jpg)
*   ChannelPipeline actually keep two processing chain: upstream, downstream. Upstream general processing read events from Channel, and downstream to Channel to write an event. Note that, these two processing chain are independent of each other, passing in the upstream chain to the last ChannelHandler, will not be passed to the downstream chain to continue processing.

    Making making making will be at the end of the downstream chainChannelSink processing, users can customize this ChannelSink, the system also has a default implementation, when the downstream chain in the last ChannelHandler after processing will be passed to the ChannelSink for final processing.