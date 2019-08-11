---
# Page settings
title: CQRS # Define a title of your page
description: CQRS — Description # Define a description of your page
keywords: # Define keywords for search engines
order: 0 # Define order of this page in list of all documentation documents
comments: false # Set to "true" in order to enable comments on this page. Make sure you properly setup "disqus_forum_shortname" variable in "_config.yml"

# Hero section
hero:
    title: CQRS
    text: CQRS
---


## Commands
### Overview
Adds ability to create and process **commands** in the sense of [CQRS](https://martinfowler.com/bliki/CQRS.html). 

### Installation
`dotnet add package Convey.CQRS.Commands`

### Dependencies 
* [Convey](https://www.nuget.org/packages/Convey)

### Usage
Implement ``ICommand`` (marker) interface in the selected class. Since the command represents user's intention you should follow the convention:

* keep all the commands **immutable**
* name of your commands should be **imperative**

```csharp
public class CreateAccount : ICommand
{
    public Guid Id { get; }
    public string Email { get; }
    public string Password { get; }

    public CreateUser(id, email, password)
    {
        Id = id;
        Email = email;
        Password = password;
    }
}
```

Create dedicated **command handler** class that implements ``ICommandHandler<TCommand>`` interface with ``HandleAsync()`` method:

```csharp
public class CreateAccountHandler : ICommandHandler<CreateAccount>
{
    public Task HandleAsync(CreateAccount command)
    {
        //put the handling code here
    }
}
```

You can easily register all command handlers in DI container by calling ``AddCommandHandlers()`` method on ``IConveyBuilder``:

```csharp
public IServiceProvider ConfigureServices(this IServiceCollection services)
{
    var builder = ConveyBuilder
        .Create(services)
        .AddCommandHandlers();

    //other registrations    
    return builder.Build();
}
```

Dispatching a particular command object can be also done using Convey package. Start with registering in-memory dispatcher on your ``IConveyBuilder`` by calling a ``AddInMemoryCommandDispatcher()`` method:


```csharp
public IServiceProvider ConfigureServices(this IServiceCollection services)
{
    var builder = ConveyBuilder
        .Create(services)
        .AddCommandHandlers()
        .AddInMemoryCommandDispatcher();

    //other registrations    
    return builder.Build();
}
```

Then simply inject ``ICommandDispatcher`` into a class and call ``DispatchAsync()`` method:

```csharp
public class AccountsService
{
    private readonly ICommandDispatcher _dispatcher;

    public AccountsService(ICommandDispatcher dispatcher)
    {
        _dispatcher = dispatcher;
    } 

    public Task CreateAccountAsync(CreateAccount command)
        => _dispatcher.DispatchAsync(command);
}
```

## Queries
### Overview
Adds ability to create and process **queries** in the sense of [CQRS](https://martinfowler.com/bliki/CQRS.html). 

### Installation
`dotnet add package Convey.CQRS.Queries`

### Dependencies 
* [Convey](https://www.nuget.org/packages/Convey)

### Usage
Implement ``IQuery<TResult>`` interface in the selected class:

```csharp
public class GetAccount : IQuery<AccountDto>
{
    public Guid Id { get; set; }
}
```

Create dedicated **query handler** class that implements ``IQueryHandler<TQuery, TResult>`` interface with ``HandleAsync()`` method:

```csharp
public class GetAccountHandler : IQueryHandler<GetAccount, AccountDto>
{
    public Task<AccountDto> HandleAsync(GetAccount query)
    {
        //put the handling code here
    }
}
```

You can easily register all query handlers in DI container by calling ``AddQueryHandlers()`` method on ``IConveyBuilder``:

```csharp
public IServiceProvider ConfigureServices(this IServiceCollection services)
{
    var builder = ConveyBuilder
        .Create(services)
        .AddQueryHandlers();

    //other registrations    
    return builder.Build();
}
```

Dispatching a particular query object can be also done using Convey package. Start with registering in-memory dispatcher on your ``IConveyBuilder`` by calling a ``AddInMemoryQueryDispatcher()`` method:


```csharp
public IServiceProvider ConfigureServices(this IServiceCollection services)
{
    var builder = ConveyBuilder
        .Create(services)
        .AddQueryHandlers()
        .AddInMemoryQueryDispatcher();

    //other registrations    
    return builder.Build();
}
```

Then simply inject ``IQueryDispatcher`` into a class and call ``DispatchAsync()`` method:

```csharp
public class AccountsService
{
    private readonly IQueryDispatcher _dispatcher;

    public AccountsService(IQueryDispatcher dispatcher)
    {
        _dispatcher = dispatcher;
    } 

    public Task<AccountDto> GetAccountAsync(Guid id)
        => _dispatcher.DispatchAsync(new GetAccount { Id = id });
}
```

## Events
### Overview
Adds ability to create and process **events** in the sense of [CQRS](https://martinfowler.com/bliki/CQRS.html). 

### Installation
`dotnet add package Convey.CQRS.Events`

### Dependencies 
* [Convey](https://www.nuget.org/packages/Convey)

### Usage
Implement ``IEvent`` or ``IRejectedEvent`` (marker) interface in the selected class. Since the event represents something that already happened, you should follow the convention:

* keep all the events **immutable**
* name of your events should kept in **past tense**

```csharp
public class AccountCreated : IEvent
{
    public Guid Id { get; }

    public AccountCreated(id)
    {
        Id = id;
    }
}
```

Create dedicated **event handler** class that implements ``IEventHandler<TEvent>`` interface with ``HandleAsync()`` method:

```csharp
public class AccountCreatedHandler : IEventHandler<AccountCreated>
{
    public Task HandleAsync(AccountCreated @event)
    {
        //put the handling code here
    }
}
```

You can easily register all event handlers in DI container by calling ``AddEventHandlers()`` method on ``IConveyBuilder``:

```csharp
public IServiceProvider ConfigureServices(this IServiceCollection services)
{
    var builder = ConveyBuilder
        .Create(services)
        .AddEventHandlers();

    //other registrations    
    return builder.Build();
}
```

Dispatching a particular event object can be also done using Convey package. Start with registering in-memory dispatcher on your ``IConveyBuilder`` by calling a ``AddInMemoryEventDispatcher()`` method:


```csharp
public IServiceProvider ConfigureServices(this IServiceCollection services)
{
    services.AddOpenTracing();

    var builder = ConveyBuilder
        .Create(services)
        .AddCommandHandlers()
        .AddInMemoryEventDispatcher();

    //other registrations    
    return builder.Build();
}
```

Then simply inject ``IEventDispatcher`` into a class and call ``DispatchAsync()`` method:

```csharp
public class AccountsService
{
    private readonly IEventDispatcher _dispatcher;

    public AccountsService(IEventDispatcher dispatcher)
    {
        _dispatcher = dispatcher;
    } 

    public Task PostProcessAccountCreation(AccountCreated @event)
        => _dispatcher.DispatchAsync(@event);
}
```
