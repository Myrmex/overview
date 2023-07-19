---
layout: page
title: Cadmus Development
subtitle: History - Logging Refactoring
---

## Logging Refactoring

📆 Date: 2023-07-19

### Rationale

The original logging infrastructure was mixing different approaches, due to the less streamlined configuration methods of previous ASP.NET versions. Now logging has been unified in API `Program.cs`, and removed from `Startup.cs`.

### Affected Products

All the backend API projects.

### Upgrade Path

(1) update your NuGet packages (`Cadmus.Api.Controllers`).

(2) in `appsettings.json`, remove the connection string from `Serilog` and add it to `ConnectionStrings` among the other connection strings:

```json
"ConnectionStrings": {
  "Log": "mongodb://localhost:27017/cadmus-PRJ-log"
},
```

>Note that while the other connection strings are rather templates, with a placeholder rpelacing the database name, this is a normal connection string ready to be used. Just replace `PRJ` with your project's codename.

(3) in `Program.cs` in the `Main` method, use this code to configure the logger:

```cs
var host = await CreateHostBuilder(args)
    .UseSerilog((hostingContext, loggerConfiguration) =>
    {
        string cs = hostingContext.Configuration
            .GetConnectionString("Log");
        var maxSize = hostingContext.Configuration["Serilog:MaxMbSize"];

        loggerConfiguration
            .ReadFrom.Configuration(hostingContext.Configuration)
#if DEBUG
            .WriteTo.File("cadmus-log.txt", rollingInterval: RollingInterval.Day)
#endif
            .WriteTo.MongoDBCapped(cs,
                cappedMaxSizeMb: !string.IsNullOrEmpty(maxSize) &&
                    int.TryParse(maxSize, out int n) && n > 0 ? n : 10);
    })
    .Build()
    .SeedAsync(); // see Services/HostSeedExtension
```

(4) in `Startup.cs`, `ConfigureServices`, _remove_ any logging configuration, which usually is like this:

```cs
// serilog
// Install-Package Serilog.Exceptions Serilog.Sinks.MongoDB
// https://github.com/RehanSaeed/Serilog.Exceptions
string maxSize = Configuration["Serilog:MaxMbSize"];
services.AddSingleton<ILogger>(_ => new LoggerConfiguration()
    .MinimumLevel.Information()
    .MinimumLevel.Override("Microsoft", LogEventLevel.Warning)
    .Enrich.WithExceptionDetails()
    .WriteTo.Console()
    .WriteTo.MongoDBCapped(Configuration["Serilog:ConnectionString"],
        cappedMaxSizeMb: !string.IsNullOrEmpty(maxSize) &&
            int.TryParse(maxSize, out int n) && n > 0 ? n : 10)
        .CreateLogger());
```

If you have additional controllers which require an instance of `ILogger`, inject the `Microsoft.Extensions.Logging.ILogger<T>` where `T` is the target of the injection (i.e. the controller class). No reference to Serilog is required.

>This type of injection has been applied also to the default controllers in `Cadmus.Api.Controllers`.
