---
title: Metrics
description: AppMetrics, Grafana and Prometheus.
keywords:
order: 7
comments: false

hero:
    title: Metrics
    text: AppMetrics, Grafana and Prometheus.
---

## Overview
Adds capability of generating application metrics and exposing them via HTTP endpoint.

![](/img/grafana.png "Grafana")
![](/img/prometheus.png "Prometheus")

### Installation
`dotnet add package Convey.Metrics.AppMetrics`

or just a dedicated package for Prometheus:

`dotnet add package Convey.Metrics.Prometheus`

### Dependencies

* [Convey](https://www.nuget.org/packages/Convey)


## Options
* `enabled` - determines whether metrics endpoint is going to be available.
* `influxEnabled` - if `true` metrics will be reported to [InfluxDB](https://www.influxdata.com/).
* `prometheusEnabled` - if `true` metrics will be formatted using [Prometheus](https://prometheus.io/) data model.
* `prometheusFormatter` - if set to `protobuf` then protobuf output formatter is going to be used. Otherwise, text output formatter is picked.
* `InfluxUrl` - connection string to InfluxDB instance e.g. `http://localhost:8086`. Required only if `influxEnabled` is `true`.
* `database` - InfluxDB database name. Required only if `influxEnabled` is `true`.
* `interval` - InfluxDB flush interval expressed in seconds. Required only if `influxEnabled` is `true`.

### appsettings.json

```json
"metrics": {
    "enabled": true,
    "influxEnabled": false,
    "prometheusEnabled": true,
    "influxUrl": "http://localhost:8086",
    "database": "pacco",
    "env": "local",
    "interval": 5
  }
```

or 

```json
"prometheus": {
  "enabled": true,
  "endpoint": "/metrics"
}
```

### Usage
Inside ``Startup.cs`` extend ``IConveyBuilder`` with ``AddMetrics()`` and ``IApplicationBuilder`` with ``UseMetrics()``:

```csharp
public IServiceProvider ConfigureServices(this IServiceCollection services)
{
    var builder = services.AddConvey()
        .AddMetrics();

    //other registrations    
    return builder.Build();
}

public void Configure(this IApplicationBuilder app)
{
    app.UseMetrics();
}
```
The above code registers all required services and exposes some default application metrics at two HTTP endpoints:

* `http://host/metrics` - depending on `prometheusEnabled` option, this endpoint exposes metrics that are either formatted using Prometheus data model or using standard, text formatter
* `http://host/metrics-text` - this endpoint exposes metrics that are always formated using standard, text formatter

### Creating custom metrics
Once you finish Convey registration, you can create custom application metrics using the [AppMetrics](https://www.app-metrics.io/) library for .NET Core. There are six different metrics types you can use:

* [Gauges](https://www.app-metrics.io/getting-started/metric-types/gauges/)
* [Counters](https://www.app-metrics.io/getting-started/metric-types/counters/)
* [Meters](https://www.app-metrics.io/getting-started/metric-types/meters/)
* [Histograms](https://www.app-metrics.io/getting-started/metric-types/histograms/)
* [Timers](https://www.app-metrics.io/getting-started/metric-types/timers/)
* [Apdex](https://www.app-metrics.io/getting-started/metric-types/apdex/)

To create a custom metrics inject ``IMetricsRoot`` into selected class:

```csharp
public class MetricsRegistry
{
    private readonly IMetricsRoot _metricsRoot;
    private readonly CounterOptions _findDiscountsQueries = new CounterOptions { Name = "find-discount" };

    public MetricsRegistry(IMetricsRoot metricsRoot)
        => _metricsRoot = metricsRoot;
    
    public void IncrementFindDiscountsQuery()
        => _metricsRoot.Measure.Counter.Increment(_findDiscountsQueries);
}
```

