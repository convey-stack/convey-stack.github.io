---
title: Persistence
description: Persistence
keywords:
order: 10
comments: false

hero:
    title: Persistence
    text: Persistence
---

## MongoDB

## Overview
Adds the set of conventions and ease of use for [MongoDB](https://www.mongodb.com) integration with .NET Core.

## Installation
`dotnet add package Convey.Persistence.MongoDB`

## Dependencies

* [Convey](https://www.nuget.org/packages/Convey)

## Usage

Extend `IConveyBuilder` with `AddMongo()` that will register the required services.

```csharp
public static IConveyBuilder RegisterConvey(this IConveyBuilder builder)
{
    builder.AddMongo();
    // Other services.
    
    return builder;
}
```

In order to use `IMongoRepository` abstraction, invoke `AddMongoRepository<TDocument, TIdentifiable>("collectionName") for each document like that you would like to be able to use with this interface.

By using the provided `IMongoRepository` you can access helper methods such as `AddAsync()`, `BrowseAsync()` etc. instead of relying on the default `IMongoDatabase` abstraction available via [MongoDB.Driver](https://docs.mongodb.com/ecosystem/drivers/csharp/).

```csharp
public class SomeService
{
    private readonly IMongoDatabase _database;

    public SomeService(IMongoDatabase database)
    {
        _database = database;
    }
}
```

```csharp
public class SomeService
{
    private readonly IMongoRepository<SomeDocument,Guid> _repository;

    public SomeService(IMongoRepository<SomeDocument,Guid> repository)
    {
        _repository = repository;
    }
}
```

## Options
* `connectionString` - connection string of MongoDB e.g. `mongodb://localhost:27017`.
* `database` - database name.
* `seed` - boolean value, if `true` then `IMongoDbSeeder.SeedAsync()` will be invoked (if implemented).

### appsettings.json

```js
"mongo": {
  "connectionString": "mongodb://localhost:27017",
  "database": "some-service",
  "seed": false
}
```

## Redis

## Overview
Adds the [Redis](https://redis.io/) integration with .NET Core based on [IDistributedCache](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.caching.distributed.idistributedcache) abstraction.

## Installation
`dotnet add package Convey.Persistence.Redis`

## Dependencies

* [Convey](https://www.nuget.org/packages/Convey)

## Usage

Extend `IConveyBuilder` with `AddRedis()` that will register the required services.

```csharp
public static IConveyBuilder RegisterConvey(this IConveyBuilder builder)
{
    builder.AddRedis();
    // Other services.
    
    return builder;
}
```

In order to use Redis integration, simply inject built-in `IDistributedCache` interface.

```csharp
public class SomeService
{
    private readonly IDistributedCache _cache;

    public SomeService(IDistributedCache cache)
    {
        _cache = cache;
    }
}
```

## Options
* `connectionString` - connection string of Redis e.g. `localhost`.
* `instance` - optional prefix, that will be added by default to all the keys stored in Redis.

### appsettings.json

```js
"redis": {
  "connectionString": "localhost",
  "instance": "some-service:"
}
```