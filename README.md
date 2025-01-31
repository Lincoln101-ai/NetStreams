# NetStreams

A light weight .NET streaming api for Kafka.

## Running the project locally

Clone the repository and execute docker compose up  to standup zookeeper and a broker for testing

```
docker-compose up -d
```

## Create a stream and handle messages

``` .net
  var builder = new NetStreamBuilder(
        cfg =>
        {
            cfg.BootstrapServers = "localhost:9092";
            cfg.ConsumerGroup = "MyConsumer";
        });

   builder.Stream<Null, MyMessage>(sourceTopic)
         .Handle(context => Console.WriteLine($"Handling message value={context.Message.Value}"))
         .StartAsync(new CancellationToken());
```

## Filter Messages to be handled


``` .net
  var builder = new NetStreamBuilder(
        cfg =>
        {
            cfg.BootstrapServers = "localhost:9092";
            cfg.ConsumerGroup = "MyConsumer";
        });

  builder.Stream<Null, MyMessage>(sourceTopic)
         .Filter(context => context.Message.Value % 3 == 0)
         .Handle(context => Console.WriteLine($"Handling message value={context.Message.Value}"))
         .StartAsync(new CancellationToken());
```


## Write output to kafka topic

This example leverages mediator to dispatch a command and then writes the output to the destination topic

```
var builder = new NetStreamBuilder(
    cfg =>
    {
        cfg.BootstrapServers = "localhost:9092";
        cfg.ConsumerGroup = "Orders.Consumer";
    });

    builder.Stream<string, OrderCommand>(sourceTopic)
            .Handle<String, OrderEvent>(context => (OrderEvent)mediator.Send(context.Message).Result)
            .ToTopic("Order.Events", message => message.Key)
            .StartAsync(CancellationToken.None);
```


## Configure topics for auto creation

This example leverages mediator to dispatch a command and then writes the output to the destination topic

```
  var builder = new NetStreamBuilder(
        cfg =>
        {
            cfg.BootstrapServers = "localhost:9092";
            cfg.ConsumerGroup = "Orders.Consumer";
            cfg.AddTopicConfiguration(cfg =>
            {
                cfg.Name = "Order.Commands";
                cfg.Partitions = 2;
            });
            cfg.AddTopicConfiguration(cfg =>
            {
                cfg.Name = "Order.Events";
                cfg.Partitions = 2;
            });
        });

    builder.Stream<string, OrderCommand>(sourceTopic)
            .Handle<string, OrderEvent>(context => (OrderEvent)mediator.Send(context.Message).Result)
            .ToTopic("Order.Events", message => message.Key)
            .StartAsync(CancellationToken.None);
```