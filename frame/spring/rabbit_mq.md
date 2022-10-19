Delivery Identifiers: Delivery Tags
Before we proceed to discuss other topics it is important to explain how deliveries are identified (and acknowledgements indicate their respective deliveries). 
When a consumer (subscription) is registered, messages will be delivered (pushed) by RabbitMQ using the basic.deliver method. The method carries a delivery tag, 
which uniquely identifies the delivery on a channel. Delivery tags are therefore scoped per channel.

Delivery tags are monotonically growing positive integers and are presented as such by client libraries. 
Client library methods that acknowledge deliveries take a delivery tag as an argument.

Because delivery tags are scoped per channel, deliveries must be acknowledged on the same channel they were received on. 
Acknowledging on a different channel will result in an "unknown delivery tag" protocol exception and close the channel
