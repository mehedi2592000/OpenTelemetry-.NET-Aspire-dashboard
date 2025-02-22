# OpenTelemetry - .NET Aspire Dashboard

This document outlines the steps to configure OpenTelemetry in an ASP.NET Core application and use the Aspire dashboard to collect and display traces, metrics, and structured logs generated by your application.

---

## Prerequisites

### Run Aspire Dashboard
Use the following Docker command to run the Aspire dashboard:

```bash
docker run --rm -it -p 18888:18888 -p 4317:18889 -d --name aspire-dashboard -e DOTNET_DASHBOARD_UNSECURED_ALLOW_ANONYMOUS='true' mcr.microsoft.com/dotnet/nightly/aspire-dashboard:8.0.0-preview.6
```

### Notes:
- Port `18888` is used to access the UI of the dashboard.
- Port `4317` is used to receive telemetry in accordance with the OpenTelemetry Protocol (OTLP).

---

## Configure ASP.NET Core Application

### Install Required NuGet Packages
Add the following package references to your ASP.NET Core project:

```xml
<PackageReference Include="OpenTelemetry.Exporter.OpenTelemetryProtocol" Version="1.8.1" />
<PackageReference Include="OpenTelemetry.Extensions.Hosting" Version="1.8.1" />
<PackageReference Include="OpenTelemetry.Instrumentation.AspNetCore" Version="1.8.1" />
<PackageReference Include="OpenTelemetry.Instrumentation.Http" Version="1.8.1" />
<PackageReference Include="OpenTelemetry.Instrumentation.Runtime" Version="1.8.0" />
```

---

### Update `Program.cs`

Add the following code to `Program.cs`:

```csharp
var builder = WebApplication.CreateBuilder(args);

// Configure logging to use OpenTelemetry
builder.Logging.AddOpenTelemetry(logging =>
{
    logging.IncludeFormattedMessage = true;
    logging.IncludeScopes = true;
});

// Configure OpenTelemetry for metrics and tracing
builder.Services.AddOpenTelemetry()
    .WithMetrics(metrics =>
    {
        // Option 1: Using default instrumentation
        metrics.AddAspNetCoreInstrumentation()
               .AddHttpClientInstrumentation();

        // Option 2: Using runtime instrumentation (alternative to the above)
        // metrics.AddRuntimeInstrumentation()
        //        .AddMeter("Microsoft.AspNetCore.Hosting", "Microsoft.AspNetCore.Server.Kestrel", "System.Net.Http");
    })
    .WithTracing(tracing =>
    {
        tracing.AddAspNetCoreInstrumentation()
               .AddHttpClientInstrumentation();
    });

// Use OTLP exporter if configured
var useOtlpExporter = !string.IsNullOrWhiteSpace(builder.Configuration["OTEL_EXPORTER_OTLP_ENDPOINT"]);
if (useOtlpExporter)
{
    builder.Services.AddOpenTelemetry().UseOtlpExporter();
}

var app = builder.Build();

app.Run();
```

---

### Configure `appsettings.json`

Add the following settings to your `appsettings.json` file:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "OTEL_EXPORTER_OTLP_ENDPOINT": "http://localhost:4317",
  "OTEL_SERVICE_NAME": "CoffeeShop5ehedi-app-api",
  "AllowedHosts": "*"
}
```

### Notes:
- The `OTEL_EXPORTER_OTLP_ENDPOINT` key must match exactly as written, as OpenTelemetry directly configures itself with this key name.
- You can only change the `OTEL_SERVICE_NAME` value, but the key name should not be altered.

---

### Accessing Aspire Dashboard

Once everything is set up, access the Aspire dashboard UI at:

```
http://localhost:18888
```

Here, you can monitor traces, metrics, and logs generated by your ASP.NET Core application.
