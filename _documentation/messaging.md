---
title: Messaging
description: Asynchronous messaging using RabbitMQ.
keywords:
order: 4
comments: false

hero:
    title: Messaging
    text: Asynchronous messaging using RabbitMQ.
---

### Overview
Adds the set of conventions and ease of use for [RabbitMQ](https://www.rabbitmq.com/) integration with .NET Core.

![](/img/rabbitmq.png "RabbitMQ")

### Installation
`dotnet add package Convey.MessageBrokers.RabbitMQ`

### Dependencies

* [Convey](https://www.nuget.org/packages/Convey)
* [Convey.MessageBrokers](https://www.nuget.org/packages/Convey.MessageBrokers)

### Options
Provides RabbitMQ integration based on the official [RabbitMQ .NET Client](https://www.rabbitmq.com/dotnet.html) with highly customizable settings, support for custom naming conventions, templating, dead letter exchange and many more.


### appsettings.json

```json
"rabbitMq": {
  "connectionName": "some-service",
  "retries": 3,
  "retryInterval": 2,
  "conventionsCasing": "snakeCase",
  "logger": {
    "enabled": true
  },
  "username": "guest",
  "password": "guest",
  "virtualHost": "/",
  "port": 5672,
  "hostnames": [
    "localhost"
  ],
  "requestedConnectionTimeout": "00:00:30",
  "requestedHeartbeat": "00:01:00",
  "socketReadTimeout": "00:00:30",
  "socketWriteTimeout": "00:00:30",
  "continuationTimeout": "00:00:20",
  "handshakeContinuationTimeout": "00:00:10",
  "networkRecoveryInterval": "00:00:05",
  "exchange": {
    "declare": true,
    "durable": true,
    "autoDelete": false,
    "type": "topic",
    "name": "stories"
  },
  "queue": {
    "declare": true,
    "durable": true,
    "exclusive": false,
    "autoDelete": false,
    "template": "some-service/{{exchange}}.{{message}}"
  },
  "context": {
    "enabled": true,
    "header": "message_context"
  },
  "deadLetter": {
    "enabled": true,
    "prefix": "dlx-",
    "declare": true
  },
  "maxProducerChannels": 1000,
  "requeueFailedMessages": false,
  "spanContextHeader": "span_context"
},
```


### Usage
Inside `Startup.cs` extend `IConveyBuilder` with `AddRabbitMq()` that will register the required services. 

```csharp
public IServiceProvider ConfigureServices(this IServiceCollection services)
{
    var builder = services.AddConvey()
        .AddRabbitMq();

    //other registrations    
    return builder.Build();
}
```

The above registration creates a unique connection to RabbitMQ and registers services required for publishing and subscribing the messages.

To subscribe to a particular message, invoke `UseRabbitMq()` method on `IApplicationBuilder` and call `Subscribe<TMessage>()` passing a function which is going to be executed once the message got received.

```csharp
public void Configure(this IApplicationBuilder app)
{
    app.UseRabbitMq()
       .Subscribe<MyMessage>(async (serviceProvider, message, context) => 
       {
           //put your "on received" code here
       });
}
```

Once you subscribe to message a coresponding RabbitMQ **topic**, **exchange** and **routing key** should be created using the following conventions:

* `Exchange` - `{options.exchange.name}`
* `Queue` - `{options.queue.template}` 
* `Routing key` - `{messageType}`

The conventions can be overriden either by providing the custom implementation of `IConventionsBuilder`:

```csharp
public interface IConventionsBuilder
{
    string GetRoutingKey(Type type);
    string GetExchange(Type type);
    string GetQueue(Type type);
}
```

Or by applying `[Message]` attribute on top of the class e.g.

```csharp
[Message("users")]
public class UserCreated : IEvent
{
    public Guid UserId { get; }
    public string Name { get; }

    public UserCreated(Guid userId, string name)
    {
        UserId = userId;
        Name = name;
    }
}
```

To publish a message simply inject `IBusPublisher`  into any class you want and invoke `PublishAsync()` (or make use `IRabbitMqClient`) passing the message and the additional parameters,

```csharp
public class CustomBusPublisher
{
    private readonly IBusPublisher _publisher;

    public CustomBusPublisher(IBusPublisher publisher)
    {
        _publisher = publisher;
    }

    public Task PublishMessageAsync<T>(T message) => _publisher.PublishAsync(message);
}
```

### Error handling
During message processing there might be a chance that an exception will be thrown. We can distinguish two types of exceptions:

* `domain exception` - informs that message cannot be further processed due to some domain logic like. `PasswordToShortException`
* `infrastructure exception` - informs that message cannot be further processed due to infrastructure issues like. connecting to database etc.

In the first scenario, it's better not to retry the processing (wrong password is not going to be better once we try again). n the second one, we can try a few times before we give up. Convey allows you to add this type of error handling using a simple mapper. Create a class that implements `IExceptionToMessageMapper` interface and register it in `IConveyBuilder`:

```csharp
public class ExceptionToMessageMapper : IExceptionToMessageMapper
{
    public object Map(Exception exception, object message)
    {
        switch (exception)
        {
            // do simple pattern matching
        }

        return null;
    }
}

// Startup.cs

public IServiceProvider ConfigureServices(this IServiceCollection services)
{
    var builder = services.AddConvey()
        .AddRabbitMq()
        .AddExceptionToMessageMapper<ExceptionToMessageMapper>();

    //other registrations    
    return builder.Build();
}
```

If an exception will be thrown during message processing, a mapper is used to produce another message that will be automatically published to RabbitMQ. If exception->message mapping is not be defined, **retry** is going to be performed according to parameters provided in `appsettings.json`.   


## Dead-letter exchange

[DLX](https://www.rabbitmq.com/dlx.html) support can be enabled via options:

```json
"deadLetter": {
  "enabled": true,
  "prefix": "dlx-",
  "declare": true,
  "durable": true,
  "exclusive": false,
  "autoDelete": false,
  "ttl": 86400
}
```

Each message which is not defined as a part of rejected event mapping in `IExceptionToMessageMapper`, will be published to its own dead letter queue which will be named based on `{options.prefix}{queue}` e.g. for `users` queue, there would be `dlx-users` dead letter queue.

## CQRS integration
Convey allows you to integrate asynchronous communication with CQRS principle providing set of extension methods for publishing/subscribing **commands** and **events**. 

### Installation
`dotnet add package Convey.MessageBrokers.CQRS`

### Dependencies

* [Convey](https://www.nuget.org/packages/Convey)
* [Convey.MessageBrokers](https://www.nuget.org/packages/Convey.MessageBrokers)
* [Convey.CQRS.Commands](https://www.nuget.org/packages/Convey.CQRS.Commands)
* [Convey.CQRS.Events](https://www.nuget.org/packages/Convey.CQRS.Events)


### Usage
To subscribe for a particular command or event, invoke `UseRabbitMq()` method on `IApplicationBuilder` and call `SubscribeCommand<TCommand>()` or `SubscribeEvent<TCommand>()`.

```csharp
public void Configure(this IApplicationBuilder app)
{
    app.UseRabbitMq()
       .SubscribeCommand<CreateUser>()
       .SubscribeEvent<UserCreated>();
}
```

Once the message is received, it gets distpatched using [ICommandDispatcher](https://convey-stack.github.io/documentation/CQRS/) or [IEventDispatcher](https://convey-stack.github.io/documentation/CQRS/).

To publish a message simply inject `IBusPublsiher` into any class you want and invoke `SendAsync()` (for commands) or `PublishAsync()` (for event) passing the message and correlation context.

```csharp
public class CustomBusPublisher
{
    private readonly IBusPublisher _publisher;

    public CustomBusPublisher(IBusPublisher publisher)
    {
        _publisher = publisher;
    }

    public Task PublishCommandAsync<TCommand>(TCommand command) where T : class, ICommand
        => _publisher.SendAsync(command);

    public Task PublishEventAsync<TEvent>(TEvent @event) where T : class, IEvent
        => _publisher.PublishAsync(@event);
}
```

This package also allows you register async dispatcher instead of "in-memory". Extend ``IConveyBuilder`` with ``AddServiceBusCommandDispatcher()`` or ``AddServiceBusEventDispatcher``method:

```csharp
public IServiceProvider ConfigureServices(this IServiceCollection services)
{
    var builder = services.AddConvey()
        .AddQueryHandlers()
        .AddServiceBusCommandDispatcher()
        .AddServiceBusEventDispatcher();

    //other registrations    
    return builder.Build();
}
```

## Jaeger integration
You can easily integrate RabbitMQ messaging with [Jaeger](https://convey-stack.github.io/documentation/distributed-tracing/) using Convey package.

### Installation
`dotnet add package Convey.Tracing.Jaeger.RabbitMQ`

### Dependencies

* [Convey.MessageBrokers.RabbitMq](https://www.nuget.org/packages/Convey.MessageBrokers.RabbitMq)
* [Convey.Tracing.Jaeger](https://www.nuget.org/packages/Convey.Tracing.Jaeger)

### Usage
Add Jaeger options accoridng to [Convey.Tracing.Jaeger docs](https://convey-stack.github.io/documentation/distributed-tracing/) and add plugin using `AddJaegerRabbitMqPlugin()` method inside RabbtiMq optional `plugins` registration:

```csharp
public IServiceProvider ConfigureServices(this IServiceCollection services)
{
    var builder = services.AddConvey()
        .AddRabbitMq(plugins: p => p.AddJaegerRabbitMqPlugin());

    //other registrations    
    return builder.Build();
}
```


## Inbox + Outbox pattern

Provides exactly-once processing and guaranteed message delivery features based on inbox and outbox patterns. Currently supported storage:

- In memory (mostly for the testing purposes)
- SQL using Entity Framework    `dotnet add package Convey.MessageBrokers.Outbox.EntityFramework`
- Mongo                         `dotnet add package Convey.MessageBrokers.Outbox.Mongo`

```csharp
public IServiceProvider ConfigureServices(this IServiceCollection services)
{
    var builder = services.AddConvey()
        .AddMessageOutbox(outbox => outbox.AddMongo());

    //other registrations    
    return builder.Build();
}
```

```json
"outbox": {
  "enabled": true,
  "type": "sequential",
  "expiry": 3600,
  "intervalMilliseconds": 2000,
  "inboxCollection": "inbox",
  "outboxCollection": "outbox"
}
```