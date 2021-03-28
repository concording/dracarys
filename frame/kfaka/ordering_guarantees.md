# Kafka : Ordering Guarantees

Kafka guarantee the order and it’s one of the reasons for choosing kafka. But to have your messages ordered they are somethings to know.

# Partitions

## Unique Partition Ordering

Physically topics are split into partitions. A partition lives on a physical node and persists the messages it receives.

![](https://miro.medium.com/max/60/1*evxgUJtZsMar0wE9QuDAHA.png?q=20)

![](https://miro.medium.com/max/832/1*evxgUJtZsMar0wE9QuDAHA.png)

So on this case, with many partitions, the ordering cannot be guaranteed.

If you want to have a global ordering, you will need to have only 1 partition.

![](https://miro.medium.com/max/60/1*PiTOdiJNRwffuc5FOpJnHg.png?q=20)

![](https://miro.medium.com/max/4082/1*PiTOdiJNRwffuc5FOpJnHg.png)

Now it’s better, Kafka preserves the order of messages within a partition. This means that if messages were sent from the producer in a specific order, the broker will write them to a partition in that order and all consumers will read them in that order.It’s better but not enough, you need also to set a special config.

## Internal Retry Process

Kafka has an internal retry process :

![](https://miro.medium.com/max/60/1*TDcFaq7kDN_gnOmL2D7R-Q.png?q=20)

![](https://miro.medium.com/max/1968/1*TDcFaq7kDN_gnOmL2D7R-Q.png)

(Chapter 3 p 42 of the kafka guide : [https://www.confluent.io/wp-content/uploads/confluent-kafka-definitive-guide-complete.pdf](https://www.confluent.io/wp-content/uploads/confluent-kafka-definitive-guide-complete.pdf))

As you see on the schema, sometimes adding a message to the broker fails, so a Retry is performed.

So on this case what will happen if I send 2 messages, and the first one needs a retry, but not the second one. On this specific case, the second one working will be added on the commit log before the first one that was in retry.

It’s not possible to set the number of retries to zero is not an option in a reliable system, so if guaranteeing order is critical. You will need to set the config **max.in.flight.requests.per.connection=1** to make sure that while a batch of messages is retrying, additional messages will not be sent. This will severely limit the throughput of the producer, so only use this when order is important. (you can have more information on the p52 of the kafka guide : [https://www.confluent.io/wp-content/uploads/confluent-kafka-definitive-guide-complete.pdf](https://www.confluent.io/wp-content/uploads/confluent-kafka-definitive-guide-complete.pdf))

## Many Partitions ordering

Having one partition is cool (and sometimes depending of your business mandatory), but you will be obliged to have only one consumer, and the producer throughput will be limited. Thanks to the kafka system of key/partitions, you can create a more intelligent structure, that will allow you to have many partitions and continue having ordered messages.

**The keys used by kafka allows you to put all the messages with the same key on one partition.** (by a mathematical process _Hash(key)%partitionNb _to guarantee that one key will always be in the same partition (as long as you don’t change the partition number), so be careful because if you change the partition number during the life of your topic, a key will be put in a different partition than before the changing of partition number).

The only question you should ask yourself is : **Should my system be globally ordered or can I partition this order.**

For example : In an marketplace system, you have to create a Cart and then Add your goods (it should be ordered, you cannot add a good to a cart not yet created) but as all your carts of your system isn’t related you can split them in different partition. For that you only need to add the same key, to the different steps (logical process) that should be ordered. On this case maybe the cart guid as key will be enough, but if you need to order all carts of a customer, the key should be the customer_id.

So don’t hesitate to play with the keys, to have the ordering that your business needs.

I hope my explanation is clear enough (comment if it’s not the case)
