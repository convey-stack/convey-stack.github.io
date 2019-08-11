---
title: Message brokers
description: Asynchronous messaging using RabbitMQ.
keywords:
order: 4
comments: false

hero:
    title: Message brokers
    text: Asynchronous messaging using RabbitMQ.
---

### Overview
Adds the set of conventions and ease of use for [RabbitMQ](https://www.rabbitmq.com/) integration with .NET Core.

### Installation
`dotnet add package Convey.MessageBrokers.RabbitMQ`

### Dependencies

* [Convey](https://www.nuget.org/packages/Convey)
* [Convey.MessageBrokers](https://www.nuget.org/packages/Convey.MessageBrokers)
* [Convey.Persistence.Redis](https://www.nuget.org/packages/Convey.Persistence.Redis)

### Options
Convey exteds [RawRabbitConfiguration](https://rawrabbit.readthedocs.io/en/master/configuration.html) with the following properties:

* `namespace` - used for creating queue name, exchange name, queue-exchange binding and routing key.
* `retries` - specifies number of retries for proceessing each message
* `retryInterval` - specifies interval (in seconds) between each retry
* `processors` - specifies message processors


### appsettings.json

```js
"rabbitMq": {
    "namespace": "availability",
    "retries": 3,
    "retryInterval": 2,
    "messageProcessor": {
      "enabled": true,
      "type": "redis",
      "messageExpirySeconds": 300
    },
    "username": "guest",
    "password": "guest",
    "virtualHost": "/",
    "port": 5672,
    "hostnames": [
      "localhost"
    ],
    "requestTimeout": "00:00:10",
    "publishConfirmTimeout": "00:00:01",
    "recoveryInterval": "00:00:10",
    "persistentDeliveryMode": true,
    "autoCloseConnection": true,
    "automaticRecovery": true,
    "topologyRecovery": true,
    "exchange": {
      "durable": true,
      "autoDelete": false,
      "type": "Topic"
    },
    "queue": {
      "autoDelete": false,
      "durable": true,
      "exclusive": false
    }
  }
```


## Usage
Inside `Startup.cs` extend `IConveyBuilder` with `AddRabbitMq<TContext>()` that will register the required services. 

```csharp
public IServiceProvider ConfigureServices(this IServiceCollection services)
{
    var builder = ConveyBuilder
        .Create(services)
        .AddRabbitMq<CorrelationContext>();

    //other registrations    
    return builder.Build();
}
```

Note that registration method requires `TContext` generic parameter. Given type must implement `ICorrelationContext` interface provided by Convey and it's used for correlating set of messages that have been sent all across the application during single HTTP request.

The above registration creates a unique connection to RabbitMQ and registers services required for publishing and subscribing the messages.

To subscribe for a particular message, invoke `UseRabbitMq()` method on `IApplicationBuilder` and call `Subscribe<TMessage>()` passing a function which is going to be executed once the message got received.

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

Once you subscribe to message a coresponding RabbitMQ **topic exchange** and **queue** should be created using the following conventions:

* `Exchange name` - `{namespace}`
* `Queue name` - `{assemlbyName}/{namespace}.{messageName}` 
* `Echange-Queue Binding` - `{namespace}.{messageName}`

If `namespace` parameter is not specified in the `appsettings.json` this part of the name is skipped.

To publish a message simply inject `IBusPublsiher` into any class you want and invoke `PublishAsync()` passing the message and correlation context.

```csharp
public class CustomBusPublisher
{
    private readonly IBusPublisher _publisher;

    public CustomBusPublisher(IBusPublisher publisher)
    {
        _publisher = publisher;
    }

    public Task PublishMessageAsync<T>(T message, CorrelationContext context)
        => _publisher.PublishAsync(message, context);
}
```

The given message is published to an **exchange** with the following conventions:

* `Exchange name` - `{namespace}` or value passed in the `MessageNamespaceAttribute`
* `Message routing key` - `{namespace}.{messageName}`or `{valueFromMessageNamespaceAttribute}.{messageName}`

Thus if you use Convey withing just **one application** the only, required parameter to make publish/subscribe work is a `namespace` parameter inside `appsettings.json`. Once you communicate **multiple, indepndent apps (like microservices)** `MessageNamespaceAttribute` is necessary on the publisher side to send a message to the correct exchange with correct message routing key.

```csharp
[MessageNamespace("parcels")]
public class CreateParcel
{

}
```

The above message will be published to `parcels` exchange with `parcels.create_parcels` routing key. If no attribute is specified then `namespace` parameter from `appsettings.json` is used instead.

### Error handling
During message processing there might be a chance that exception will be thrown. We can distinguish two types of exceptions:

* `domain exception` - informs that message cannot be further processed due to some doman logic like. `PasswordToShortException`
* `infrastructure exception` - informs that message cannot be further processed due to infrastructure issues like. connecting to database etc.

In first scenario it's better not to retry the processing (wrong password is not going to be better once we try again). n the second one, we can try a few times before we give up. Convey allows you to add this type of error handling using simple mapper.Create a class that implements `IExceptionToMessageMapper` interface and register it in `IConveyBuilder`:

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
    var builder = ConveyBuilder
        .Create(services)
        .AddRabbitMq<CorrelationContext>()
        .AddExceptionToMessageMapper<ExceptionToMessageMapper>();

    //other registrations    
    return builder.Build();
}
```

If an exception will be thrown during message processing, a mapper is used to produce another message that will be automaticly published to RabbitMQ. If exception->message mapping is not be defined, **retry** is going to be performed according to parameters provided in `appsettings.json`.   

### Message processors

## CQRS integration
Convey allows you to integrate asynchronous communication via RabbitMQ with CQRS principle providing 

## Jaeger integration



