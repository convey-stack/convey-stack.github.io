---
title: Getting started
description: Introduction to Convey.
keywords:
order: 0
comments: false

hero:
    title: Getting started
    text: Introduction to Convey.
---

# Welcome to Convey

Convey is a set of **helper libraries** that can be most of the time (with some exceptions) **used independently of each other** to help you to build your web applications and microservices. **Convey is neither a framework nor a silver bullet**. 

Quite opposite, **it's mostly the set of extensions methods along with additional abstractions** that will help you to deal with common infrastructural concerns such as routing, service discovery, load balancing, tracing, asynchronous messaging and so on.

In order to find out how you can **use Convey in a real-world scenarios,** take a look at the sample projects available on our [DevMentors.io](https://devmentors.io) GitHub [organization](https://github.com/devmentors). There are 4 projects for now: 

* **[Pacco](https://github.com/devmentors/Pacco)** - the most advanced one, tackling the exclusive parcels' delivery domain, consisting of about 10 microservices
* **[Trill](https://github.com/devmentors/Trill)** - another advanced project, being a Twitter like very simple clone based on posting the messages and building timelines
* **[NPost](https://github.com/devmentors/NPost)** - only a few services, and rather simple parcel delivery domain 
* **[DShop](https://github.com/devmentors/DNC-DShop)** - doesn't use Convey, but instead contains the so-called **Common** project which was one of the reasons why we started building Convey (by splitting up into smaller, independent pieces a single shared library).

And if you're looking for the resources about building microservices with .NET Core, **check out our premium** course [Microservices .NET](https://devmentors.io/courses/microservices-net) or a **free one** available on YouTube (almost 20 hours) titled [Distributed .NET Core](https://devmentors.io/distributed-net-core).

Feel free to take a look at **our other projects** that you might find useful when building the distributed services:

* **[Chronicle](https://github.com/snatch-dev/Chronicle)** - a simple process manager/saga pattern implementation for .NET Core that helps you manage long-living and distributed transactions.
* **[Ntrada](https://github.com/snatch-dev/Ntrada)** - API Gateway built with .NET Core, that can be easily configured with yaml files and doesn't require additional coding whatsoever.
* **[Pactify](https://github.com/snatch-dev/Pactify)** - consumer-driven contract testing based on [pact.io](https://pact.io/), using in-memory web server for testing ASP.NET Core application.

## Getting started

In order to get started with Convey, simply install the core package:

`dotnet add package Convey`

Its sole responsibility is to expose `IConveyBuilder` being used by other packages, which provides fluent API experience, similar to built-in ASP.NET Core `IServiceCollection` and `IApplicationBuilder` abstractions.

```csharp
public class Program
{
    public static async Task Main(string[] args)
        => await WebHost.CreateDefaultBuilder(args)
            .ConfigureServices(services => services.AddConvey().Build())
            .Configure(app =>
            {
                //Configure the middleware
            })
            .Build()
            .RunAsync();
}
```

Whether you're using just a `Program.cs` on its own (yes, **you can build your web applications and microservices without a need of having `Startup` class and `AddMvc()` along with full `UseMvc()` middleware**) or doing it with a `Startup.cs` included, just invoke `AddConvey()` on `IServiceCollection` instance within the `ConfigureServices()` method and start using Convey packages.

The core Convey package also registers `AppOptions` type which contains the application name (and it's purely optional).

## Options
* `name` - an optional name of the application.
* `displayBanner` - display a banner (console output) with the application name during a startup, `true` by default.

### appsettings.json

```json
"app": {
  "name": "some service",
  "displayBanner": true
}
```

![](/img/banner.png "Banner")