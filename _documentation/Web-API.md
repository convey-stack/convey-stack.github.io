---
title: Web API
description: Clean and robust API definition.
keywords:
order: 2
comments: false

hero:
    title: Web API
    text: Clean and robust API definition.
---

## Endpoints
With the usage of Web API package, you can define the endpoints more fluently, without the need of using a full `ASP.NET Core MVC` package and deriving from `Controller`. It's more of an extension of the built-in `IRouteBuilder` abstraction allowing to define routing and deal with HTTP requests.

## Installation
`dotnet add package Convey.WebApi`

## Dependencies

* [Convey](https://www.nuget.org/packages/Convey)

## Usage

Extend `Program.cs` -> `CreateDefaultBuilder()` with `AddWebApi()` that will add the required services.

```csharp
public static IWebHostBuilder GetWebHostBuilder(string[] args)
    => WebHost.CreateDefaultBuilder(args)
        .ConfigureServices(services => services
            .AddConvey()
            .AddWebApi()
            .Build())
```

To define custom endpoints, invoke `UseEndpoints()` as the `IApplicationBuilder` extension within `Configure()` method. Then, you can make use of `Get()`, `Post()`, `Put()`, `Delete()` methods.

```csharp
public static IWebHostBuilder GetWebHostBuilder(string[] args)
    => WebHost.CreateDefaultBuilder(args)
        .ConfigureServices(services => services
            .AddConvey()
            .AddWebApi()
            .Build())
        .Configure(app => app
            .UseEndpoints(endpoints => endpoints
                .Get("", ctx => ctx.Response.WriteAsync("Hello"))
                .Get<GetParcel, ParcelDto>("parcels/{parcelId}")
                .Get<GetParcels, IEnumerable<ParcelDto>>("parcels")
                .Delete<DeleteParcel>("parcels/{parcelId}")
                .Post<AddParcel>("parcels", (req, ctx) => ctx.Response.Created($"parcels/{req.ParcelId}"))))
```

As you can see, generic extensions can be used when defining the endpoints (although it's not required). Whenever you define a generic endpoint with a type T, it will bind the incoming request to the new instance of T (think of it as something similar to command).

To automatically handle the incoming request, you can implement `IRequest` market interface for type T and create an `IRequestHandler<T>` that will be invoked automatically.

```csharp
public class DeleteParcel : IRequest
{
    public Guid ParcelId { get; }

    public DeleteParcel(Guid parcelId)
    {
        ParcelId = parcelId;
    }
}

public class DeleteParcelHandler : IRequestHandler<DeleteParcel, int>
{
    public async Task<int> HandleAsync(DeleteParcel request)
    {
        
        // Deleted a parcel, let's return its ID.

        return request.ParcelId;
    }
}
```

## CQRS Integration

To seamlessly integrate with Command and Query handlers that can be invoked either by internal API call via HTTP or a message broker (just make sure that you don't process queries asynchronously, as it doesn't make much sense), CQRS integration with Web API can be installed.

## Installation
`dotnet add package Convey.WebApi.CQRS`

## Dependencies

* [Convey.WebApi](https://www.nuget.org/packages/Convey.WebApi)
* [Convey.CQRS.Commands](https://www.nuget.org/packages/Convey.CQRS.Commands)
* [Convey.CQRS.Events](https://www.nuget.org/packages/Convey.CQRS.Events)
* [Convey.CQRS.Queries](https://www.nuget.org/packages/Convey.CQRS.Queries)

## Usage

Ensure that `Web API` extension is already registered, and change `UseEndpoints()` to `UseDispatcherEndpoints()`. When defining your endpoints, you will notice that now there are 2 additional (and optional) parameters - `beforeDispatch` and `afterDispatch` that can be used to alter or enrich the behavior of commands or queries before or after they will be invoked by `dispatcher`.

```csharp
public static IWebHostBuilder GetWebHostBuilder(string[] args)
    => WebHost.CreateDefaultBuilder(args)
        .ConfigureServices(services => services
            .AddConvey()
            .AddWebApi()
            .Build())
        .Configure(app => app
            .UseDispatcherEndpoints(endpoints => endpoints
                .Get("", ctx => ctx.Response.WriteAsync("Hello"))
                .Get<GetParcel, ParcelDto>("parcels/{parcelId}")
                .Get<GetParcels, IEnumerable<ParcelDto>>("parcels")
                .Delete<DeleteParcel>("parcels/{parcelId}")
                .Post<AddParcel>("parcels",
                    afterDispatch: (cmd, ctx) => ctx.Response.Created($"parcels/{cmd.ParcelId}"))))
```

To expose all of the `commands` and `events` as a sort of auto-documentation that might be helpful for integration with other services (similarly to what Swagger does) under a specialized endpoint (by default: `_contracts`) returning an array of commands and events objects using JSON format, invoke `UsePublicContracts<T>()` extension, where T is a marker attribute used to expose the selected types.

```csharp
public class ContractAttribute : Attribute
{
}

[Contract]
public class DeleteParcel : IRequest
{
    public Guid ParcelId { get; }

    public DeleteParcel(Guid parcelId)
    {
        ParcelId = parcelId;
    }
}

public static IWebHostBuilder GetWebHostBuilder(string[] args)
    => WebHost.CreateDefaultBuilder(args)
        .ConfigureServices(services => services
            .AddConvey()
            .AddWebApi()
            .Build())
        .Configure(app => app.UsePublicContracts<Contract>()
            .UseDispatcherEndpoints(endpoints => endpoints
                // Endpoints definition
            ))
```

## Swagger Integration

To integrate Swagger documentation on top of Web API defined as a set of endpoints without the usage of full `AddMvc()` and using custom `Controllers`, it is required to install this package.

## Installation
`dotnet add package Convey.WebApi.CQRS`

## Dependencies

* [Convey.Docs.Swagger](Convey.Docs.Swagger)
* [Convey.WebApi](https://www.nuget.org/packages/Convey.WebApi)

## Usage

Invoke `AddWebApiSwaggerDocs()` and then `UseSwaggerDocs()` to use Swagger.

```csharp
public static IWebHostBuilder GetWebHostBuilder(string[] args)
    => WebHost.CreateDefaultBuilder(args)
        .ConfigureServices(services => services
            .AddConvey()
            .AddWebApi()
            .AddWebApiSwaggerDocs()
            .Build())
        .Configure(app => app..UseSwaggerDocs())
```