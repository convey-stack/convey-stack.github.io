---
title: Distributed tracing
description: Tracing distributed processes with Jaeger.
keywords:
order: 8
comments: false

hero:
    title: Distributed tracing
    text: Tracing distributed processes with Jaeger.
---

## Overview
Integrates application with [Jaeger](https://www.jaegertracing.io/) (end-to-end distributed tracing) using selected reporter and sampler.

![](/img/jaeger_1.png "Jaeger")

![](/img/jaeger_2.png "Jaeger")

## Installation
`dotnet add package Convey.Tracing.Jaeger`

## Dependencies

* [Convey](https://www.nuget.org/packages/Convey)

## Open Tracing
Convey does not generate any default spans for your ASP.NET Core applications. However, this can be simply achieved by plugging in [Open Tracing](https://opentracing.io/):

### Installation
`dotnet add package OpenTracing`

### Usage
Enable the instrumentation inside your ``Startup.cs`` -> ``ConfigureServices()`` by calling ``AddOpenTracing()`` method on ``IServiceCollection``:

```csharp
public void ConfigureServices(this IServiceCollection services)
{
    services.AddOpenTracing();
}
```

## Jaeger
Once your application produces spans needed for Jaeger, you need to enable reporting in a way that suits you the most.

### Options
* ``enabled`` - determines whether reporting is enabled
* ``serviceName`` - name of the applciation that's going to be used in Jaeger query engine
* ``udpHost`` - host part of the Jaeger endpoint (UDP).
* ``udpPort`` - port of the Jaeger endpoint (UDP).
* ``maxPacketSize`` - maximum size of the UDP header packet (by default 0). **This is not required**.
* ``sampler`` - The allowed values are: ``const``, ``rate`` and ``probabilistic``. For more details about sampling check the official [Jaeger Docs](https://github.com/jaegertracing/jaeger-client-csharp/blob/master/src/Jaeger/Samplers/README.md).
* ``maxTracesPerSecond`` - determines maximum number of reported traces per second. Required only for **rate** sampler.
* ``samplingRate`` - determines the percentage of spans to report. Required only for **probabilistic** sampler.

### appsettings.json
```js
"jaeger": {
    "enabled": true,
    "serviceName": "example-service",
    "udpHost": "localhost",
    "udpPort": 6831,
    "maxPacketSize": 0,
    "sampler": "rate",
    "maxTracesPerSecond": 10,
    "samplingRate": 0.0
  }
```

### Usage
Inside your ``Startup.cs`` extend ``IConveyBuilder`` with ``AddJaeger()`` that will create the ``ITracer`` using chosen sampler and reporter:

```csharp
public IServiceProvider ConfigureServices(this IServiceCollection services)
{
    services.AddOpenTracing();

    var builder = ConveyBuilder
        .Create(services)
        .AddJaeger();

    //other registrations    
    return builder.Build();
}
```

You can also enable simple ASP.NET Core middleware that will create a span for each incoming HTTP request. Inside your ``Startup.cs`` call ``UseJaeger()`` method on ``IApplicationBuilder``:

```csharp
public void Configure(this IApplicationBuilder app)
{
    app.UseJaeger();
}
```

### Creating custom spans
Once the ``ITracer`` got registered in ``Startup.cs`` file, you can inject it to any class you want to create custom spans (not provided by Open Tracing) as follows:

```csharp
public class MyClass
{
    private readonly ITracer _tracer;

    public MyClass(ITracer tracer)
    {
        _tracer = tracer;
    }

    public void MyMethod()
    {
        using(var scope = BuildScope())
        {
            var span = scope.Span;

            try
            {
                span.Log("Starting the execution of the code");
                ///some code
            }
            catch(Exception ex)
            {
                span.Log(ex.Message);
                span.SetTag(Tags.Error, true);
            }
        }
    }

    private IScope BuildScope()
        => _tracer
            .BuildSpan("Executing important code")
            .StartActive(true);
}
```