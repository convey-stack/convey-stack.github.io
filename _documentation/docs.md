---
title: Docs
description: Docs
keywords:
order: 5
comments: false

hero:
    title: Docs
    text: Docs
---

## Overview
Adds API documentation using [Swagger](https://swagger.io/docs/) or [ReDoc](https://redoc.ly/).

## Installation
`dotnet add package Convey.Docs.Swagger`

## Dependencies

* [Convey](https://www.nuget.org/packages/Convey)


## Options
* `Enabled` - determines whether documentation is enabled
* `ReDocEnabled` - if ``false`` SwaggerUI is used to render documentation. Otherwise uses ReDoc
* `Name` - name of the documentation
* `Title` - title of the documentation
* `Version` - version of the documentation
* `RoutePrefix` - endpoint at which the documentation is going to be available
* `IncludeSecurity` - determines whether documentation security (via JWT) is going to be activated 

## appsettings.json
```js
  "swagger": {
    "enabled": true,
    "reDocEnabled": false,
    "name": "v1",
    "title": "API",
    "version": "v1",
    "routePrefix": "docs",
    "includeSecurity": true
  },
```

## Usage
Inside ``Startup.cs`` extend ``IConveyBuilder`` with ``AddSwaggerDocs()`` and ``IApplicationBuilder`` with ``UseSwaggerDocs()``:

```csharp
public IServiceProvider ConfigureServices(this IServiceCollection services)
{
    var builder = ConveyBuilder
        .Create(services)
        .AddSwaggerDocs();

    //other registrations    
    return builder.Build();
}

public void Configure(this IApplicationBuilder app)
{
    app.UseSwaggerDocs();
}
```

Having this done your docs should be available at **`http://{host}:{port}/{routePrefix}`**