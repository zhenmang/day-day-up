##[Redis Pub/Sub](https://redis.io/docs/interact/pubsub/)##  

###一、redis主要职责###

####redis 维护 通道与订阅者的关系####  

####发布者将消息推送至对应通道####  

####redis 将消息转发至与该通道关联的所有订阅者####  

值得注意的是，上述消息不会持久化，都是直来直去，立即转发。因此，你会看到，如果某个订阅者掉线了，后面即使上线了，这掉线期间的消息也无法重新接收到。  

####普通订阅####
```code
127.0.0.1:6379> SUBSCRIBE foo bar
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "foo"
3) (integer) 1
1) "subscribe"
2) "bar"
3) (integer) 2
```
####发布####
```code
127.0.0.1:6379> PUBLISH foo hello
(integer) 1
```
####看看接收方####
```code
127.0.0.1:6379> SUBSCRIBE foo bar
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "foo"
3) (integer) 1
1) "subscribe"
2) "bar"
3) (integer) 2
1) "message"
2) "foo"
3) "hello"
```
看到最后三条数据，message 表示类型、foo 表示通道、hello 表示具体消息。实时触达。  
  
####模式订阅####
```code
127.0.0.1:6379> PSUBSCRIBE foo*
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "foo.*"
3) (integer) 1
```
发送数据
```code
127.0.0.1:6379> PUBLISH foo.a hello_a
(integer) 1
127.0.0.1:6379> PUBLISH foo.b hello_b
(integer) 1
```
看接收方
```code
127.0.0.1:6379> PSUBSCRIBE foo.*
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "foo*"
3) (integer) 1
1) "pmessage"
2) "foo*"
3) "foo.a"
4) "hello_a"
1) "pmessage"
2) "foo*"
3) "foo.b"
4) "hello_b"
```
消息都能接收到，只要匹配这个模式 foo 消息都能够接收到，另外，消息类型也变成了 pmessage  

###二、原理###
PubSub 也是一种生产者/消费者模型，和传统 MQ 不同的是，经过 PubSub 的消息只做转发，不做存储。  

redis一直维持着这些订阅者的长连接，当服务端收到 publish 指令后，会匹配相应的 channel，并将消息直接推送给订阅的客户端。  

Pubsub 的结构定义：
```code
struct redisServer {

    ...

    /* Pubsub */
    dict *pubsub_channels;  /* Map channels to list of subscribed clients */
    list *pubsub_patterns;  /* A list of pubsub_patterns */
    dict *pubsub_patterns_dict;  /* A dict of pubsub_patterns */

    ...

}
```
pubsub_channels：通道与客户端的订阅关系，是一个字典，key 是 channel，value 是订阅的客户端链表
pubsub_patterns：模式匹配，记录的是模式订阅的客户端列表
pubsub_patterns_dict：模式匹配字典，后期加上的，主要考虑模式订阅的客户端过多之后的效率问题，索性就直接搞成 pubsub_channels 类似的字典，模式通道为 key，value 是订阅的客户端列表
