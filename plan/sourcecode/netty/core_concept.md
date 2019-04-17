### **2.1\. Channel**

_Channel_ is the base of Java NIO. It represents an open connection which is capable of IO operations such as reading and writing.

### **2.2\. Future**

**Every IO operation on a _Channel_ in Netty is non-blocking.**

This means that every operation is returned immediately after the call. There is a _Future _interface in the standard Java library, but it’s not convenient for Netty purposes — we can only ask the _Future _about the completion of the operation or to block the current thread until the operation is done.

That’s why **Netty has its own _ChannelFuture _interface**_. _We can pass a callback to _ChannelFuture_ which will be called upon operation completion.

### **2.3\. Events and Handlers**

Netty uses an event-driven application paradigm, so the pipeline of the data processing is a chain of events going through handlers. Events and handlers can be related to the inbound and outbound data flow. Inbound events can be the following:

*   Channel activation and deactivation
*   Read operation events
*   Exception events
*   User events

Outbound events are simpler and, generally, are related to opening/closing a connection and writing/flushing data.

Netty applications consist of a couple of networking and application logic events and their handlers. The base interfaces for the channel event handlers are _ChannelHandler _and its ancestors _ChannelOutboundHandler _and _ChannelInboundHandler_.

Netty provides a huge hierarchy of implementations of _ChannelHandler. _It is worth noting the adapters which are just empty implementations, e.g. _ChannelInboundHandlerAdapter _and_ChannelOutboundHandlerAdapter_. We could extend these adapters when we need to process only a subset of all events.

Also, there are many implementations of specific protocols such as HTTP, e.g. _HttpRequestDecoder, HttpResponseEncoder, HttpObjectAggregator. _It would be good to get acquainted with them in Netty’s Javadoc.

#### Upstream events and downstream events, and their interpretation

Every event is either an upstream event or a downstream event. If an event flows forward from the first handler to the last handler in a [`ChannelPipeline`](https://netty.io/3.8/api/org/jboss/netty/channel/ChannelPipeline.html "interface in org.jboss.netty.channel"), we call it an upstream event and say **"an event goes upstream."** If an event flows backward from the last handler to the first handler in a [`ChannelPipeline`](https://netty.io/3.8/api/org/jboss/netty/channel/ChannelPipeline.html "interface in org.jboss.netty.channel"), we call it a downstream event and say **"an event goes downstream."** (Please refer to the diagram in [`ChannelPipeline`](https://netty.io/3.8/api/org/jboss/netty/channel/ChannelPipeline.html "interface in org.jboss.netty.channel") for more explanation.)

When your server receives a message from a client, the event associated with the received message is an upstream event. When your server sends a message or reply to the client, the event associated with the write request is a downstream event. The same rule applies for the client side. If your client sent a request to the server, it means your client triggered a downstream event. If your client received a response from the server, it means your client will be notified with an upstream event. Upstream events are often the result of inbound operations such as [`InputStream.read(byte[])`](https://docs.oracle.com/javase/7/docs/api/java/io/InputStream.html?is-external=true#read(byte[]) "class or interface in java.io"), and downstream events are the request for outbound operations such as [`OutputStream.write(byte[])`](https://docs.oracle.com/javase/7/docs/api/java/io/OutputStream.html?is-external=true#write(byte[]) "class or interface in java.io"), [`Socket.connect(SocketAddress)`](https://docs.oracle.com/javase/7/docs/api/java/net/Socket.html?is-external=true#connect(java.net.SocketAddress) "class or interface in java.net"), and [`Socket.close()`](https://docs.oracle.com/javase/7/docs/api/java/net/Socket.html?is-external=true#close() "class or interface in java.net").

### **2.4\. Encoders and Decoders**

As we work with the network protocol, we need to perform data serialization and deserialization. For this purpose, Netty introduces special extensions of the _ChannelInboundHandler _for **decoders** which are capable of decoding incoming data. The base class of most decoders is _ByteToMessageDecoder._

For encoding outgoing data, Netty has extensions of the _ChannelOutboundHandler _called **encoders. **_MessageToByteEncoder_ is the base for most encoder implementations_._ We can convert the message from byte sequence to Java object and vice versa with encoders and decoders.