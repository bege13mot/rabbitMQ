Tutorial from https://www.rabbitmq.com/tutorials/tutorial-one-go.html

Run docker using an official image:
`docker run -d --hostname my-rabbit --name some-rabbit -p 8080:15672 -p 5672:5672 rabbitmq:3-management`

#### 1 Hello World

RabbitMQ is a message broker: it accepts and forwards messages.


#### 2 Work queues

The main idea behind Work Queues (aka: Task Queues) is to avoid doing a resource-intensive task immediately and having to wait for it to complete. Instead we schedule the task to be done later. We encapsulate a task as a message and send it to a queue. A worker process running in the background will pop the tasks and eventually execute the job. When you run many workers the tasks will be shared between them.

Round-robin by default, but could be changed:
```
  err = ch.Qos(
  1,     // prefetch count
  0,     // prefetch size
  false, // global
)
```

In order to make sure a message is never lost, RabbitMQ supports message acknowledgments. An ack(nowledgement) is sent back by the consumer to tell RabbitMQ that a particular message has been received, processed and that RabbitMQ is free to delete it.
`d.Ack(false)`


#### 3 Publish/Subscribe

We'll deliver a message to multiple consumers. This pattern is known as "publish/subscribe".

The fanout exchange is very simple. As you can probably guess from the name, it just broadcasts all the messages it receives to all the queues it knows.
```
  err = ch.ExchangeDeclare(
  "logs",   // name
  "fanout", // type
  true,     // durable
  false,    // auto-deleted
  false,    // internal
  false,    // no-wait
  nil,      // arguments
)
```

We need to tell the exchange to send messages to our queue. That relationship between exchange and a queue is called a binding.
```
  err = ch.QueueBind(
  q.Name, // queue name
  "",     // routing key
  "logs", // exchange
  false,
  nil
)
```

#### 4 Routing

We will use a `direct` exchange instead. The routing algorithm behind a direct exchange is simple - a message goes to the queues whose binding key exactly matches the routing key of the message.


#### 5 Topics

Messages sent to a `topic` exchange can't have an arbitrary `routing_key` - it must be a list of words, delimited by dots. The words can be anything, but usually they specify some features connected to the message. A few valid routing key examples: "stock.usd.nyse", "nyse.vmw".

The binding key must also be in the same form. The logic behind the `topic` exchange is similar to a `direct` one - a message sent with a particular routing key will be delivered to all the queues that are bound with a matching binding key. However there are two important special cases for binding keys:

* `*` (star) can substitute for exactly one word.
* `#` (hash) can substitute for zero or more words.


#### 6 RPC

But what if we need to run a function on a remote computer and wait for the result? Well, that's a different story. This pattern is commonly known as Remote Procedure Call or RPC.
