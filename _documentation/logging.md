---
title: Logging
description: Centralized logging with Serilog and Seq.
keywords:
order: 6
comments: false

hero:
    title: Logging
    text: Centralized logging with Serilog and Seq.
---

## Overview
Adds the logging capability, by default uses [Serilog](https://serilog.net) for logging with 3 optional extensions (sinks):

* Console
* File
* [Seq](https://datalust.co/seq)

## Installation
`dotnet add package Convey.Logging`

## Dependencies

* [Convey](https://www.nuget.org/packages/Convey)

## Usage

Extend `Program.cs` -> `CreateDefaultBuilder()` with `UseLogging()` that will add the required services and configure `ILogger` available in ASP.NET Core framework. 

```csharp
public static IWebHostBuilder GetWebHostBuilder(string[] args)
    => WebHost.CreateDefaultBuilder(args)
        .ConfigureServices(services => services.AddConvey().Build())
        .UseLogging();
```

Then, simply inject `ILogger<T>` (being ASP.NET Core built-in abstraction) to write the logs.

```csharp
public class SomeService
{
    private readonly ILogger<SomeService> _logger;

    public SomeService(ILogger<SomeService> logger)
    {
        _logger = logger;
    }

    public void Foo()
    {
        _logger.LogInformation("Foo");
    }
}
```

## Options
* `applicationName` - sets the optional application name property used for log [enrichment](https://github.com/serilog/serilog/wiki/Enrichment).
* `serviceId` - sets the optional service id property used for log [enrichment](https://github.com/serilog/serilog/wiki/Enrichment).
* `excludePaths` - optional endpoints that should be excluded from logging (e.g. while performing the health checks by other services).
* `console.enabled` - enables/disables console logger.
* `file.enabled` - enables/disables file logger.
* `file.path` - path to the file logs.
* `file.interval` - how often should the new file with logs be created.
* `seq.enabled` - enables/disables [Seq](https://datalust.co/seq) logger.
* `seq.url` - URL to Seq API.
* `seq.token` - API key (if provided) used while sending logs to Seq.


### appsettings.json

```js
"logger": {
  "applicationName": "some-service",
  "serviceId": "instance-1",
  "excludePaths": ["/ping", "/metrics"],
  "console": {
    "enabled": true
  },
  "file": {
    "enabled": true,
    "path": "logs/logs.txt",
    "interval": "day"
  },
  "seq": {
    "enabled": true,
    "url": "http://localhost:5341",
    "token": "secret"
  }
}
```