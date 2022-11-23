---
layout: page
title: Backend API
subtitle: "Cadmus Development"
---

- [Creating the API Project](#creating-the-api-project)
- [Settings](#settings)
- [Program](#program)

## Creating the API Project

The [reference API backend project](https://github.com/vedph/cadmus_api) is the model for this section.

1. create a new ASP.NET Core 7.0 web API project (no authentication) named `Cadmus<PRJ>Api`: select `None` for `Authentication type`, ensure that `Enable Docker` is disabled (we'll provide our own Docker files), ensure that `Use controllers`, `Enable OpenAPI support`, and `Do not use top-level statements` are checked.

2. remove the mock `WeatherForecast.cs` class and its corresponding `WeatherForecastController.cs` class from the `Controllers` folder.

3. add these NuGet packages:

- `AspNetCore.Identity.Mongo`
- `Cadmus.Api.Controllers`
- `Cadmus.Api.Models`
- `Cadmus.Api.Services`
- `Microsoft.AspNetCore.Authentication.JwtBearer`
- `Microsoft.AspNetCore.Mvc.NewtonsoftJson`
- `Microsoft.Extensions.Configuration`
- `Newtonsoft.Json`
- `Polly`
- `Serilog`
- `Serilog.AspNetCore`
- `Serilog.Exceptions`
- `Serilog.Extensions.Hosting`
- `Serilog.Sinks.Console`
- `Serilog.Sinks.File`
- `Serilog.Sinks.MongoDB`
- `Swashbuckle.AspNetCore`
- your Cadmus part and seeders packages, e.g.:
  - `Cadmus.Seed.<PRJ>.Parts`
  - `Cadmus.<PRJ>.Services`

From inside Visual Studio, you can open the Package Manager Console and enter commands like:

```bash
install-package AspNetCore.Identity.Mongo
install-package Cadmus.Api.Controllers
install-package Cadmus.Api.Models
install-package Cadmus.Api.Services
install-package Microsoft.AspNetCore.Authentication.JwtBearer
install-package Microsoft.AspNetCore.Mvc.NewtonsoftJson
install-package Microsoft.Extensions.Configuration
install-package Newtonsoft.Json
install-package Polly
install-package Serilog
install-package Serilog.AspNetCore
install-package Serilog.Exceptions
install-package Serilog.Extensions.Hosting
install-package Serilog.Sinks.Console
install-package Serilog.Sinks.File
install-package Serilog.Sinks.MongoDB
install-package Swashbuckle.AspNetCore
```

## Settings

Add these settings to `appsettings.json` (replace `__PRJ__` with your project's name). Feel free to customize them as required. Please notice that all the sensitive data like users and passwords are there only for illustration purposes, and they will be overwritten by environment variables set in the hosting server.

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "Default": "mongodb://localhost:27017/{0}",
    "Index": "Server=localhost;Database={0};Uid=root;Pwd=mysql;"
  },
  "DatabaseNames": {
    "Auth": "cadmus-__PRJ__-auth",
    "Data": "cadmus-__PRJ__"
  },
  "Serilog": {
    "ConnectionString": "mongodb://localhost:27017/{0}-log",
    "MaxMbSize": 10,
    "TableName": "Logs",
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Information",
        "System": "Warning"
      }
    }
  },
  "AllowedOrigins": [
    "http://localhost:4200",
  ],
  "Seed": {
    "ProfileSource": "%wwwroot%/seed-profile.json",
    "ItemCount": 100,
    "IndexDelay": 0
  },
  "Jwt": {
    "Issuer": "https://cadmus.azurewebsites.net",
    "Audience": "https://www.fusisoft.it",
    "SecureKey": "g>ueVcdZ7}:>4W5W"
  },
  "StockUsers": [
    {
      "UserName": "zeus",
      "Password": "P4ss-W0rd!",
      "Email": "dfusi@hotmail.com",
      "Roles": [
        "admin",
        "editor",
        "operator",
        "visitor"
      ],
      "FirstName": "Daniele",
      "LastName": "Fusi"
    }
  ],
  "Messaging": {
    "AppName": "Cadmus __PRJ__",
    "ApiRootUrl": "https://cadmus.azurewebsites.net/api/",
    "AppRootUrl": "https://fusisoft.it/apps/cadmus/",
    "SupportEmail": "webmaster@fusisoft.net"
  },
  "Editing": {
    "BaseToLayerToleranceSeconds": 60
  },
  "Indexing": {
    "IsEnabled": true,
    "DatabaseType": "mysql",
    "IsGraphEnabled": false
  },
  "Preview": {
    "IsEnabled": true
  },
  "Mailer": {
    "IsEnabled": false,
    "SenderEmail": "webmaster@fusisoft.net",
    "SenderName": "Cadmus __PRJ__",
    "Host": "",
    "Port": 0,
    "UseSsl": true,
    "UserName": "place in environment",
    "Password": "place in environment"
  }
}
```

## Program

Use this template to replace the code in `Program.cs` (replace `__PRJ__` with your project's name):

```cs
using System.Collections;
using Serilog;
using System.Diagnostics;
using Serilog.Events;
using Cadmus.Api.Services.Seeding;
using Cadmus.Api.Services;

namespace Cadmus__PRJ__Api
{
    /// <summary>
    /// Program.
    /// </summary>
    public static class Program
    {
        private static void DumpEnvironmentVars()
        {
            Console.WriteLine("ENVIRONMENT VARIABLES:");
            IDictionary dct = Environment.GetEnvironmentVariables();
            List<string> keys = new();
            var enumerator = dct.GetEnumerator();
            while (enumerator.MoveNext())
            {
                keys.Add(((DictionaryEntry)enumerator.Current).Key.ToString()!);
            }

            foreach (string key in keys.OrderBy(s => s))
                Console.WriteLine($"{key} = {dct[key]}");
        }

        /// <summary>
        /// Creates the host builder.
        /// </summary>
        /// <param name="args">The arguments.</param>
        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>();
                });

        /// <summary>
        /// Entry point.
        /// </summary>
        /// <param name="args">The arguments.</param>
        public static async Task<int> Main(string[] args)
        {
            Log.Logger = new LoggerConfiguration()
                .MinimumLevel.Debug()
                .MinimumLevel.Override("Microsoft", LogEventLevel.Information)
                .Enrich.FromLogContext()
                .WriteTo.Console()
#if DEBUG
                .WriteTo.File("cadmus-log.txt", rollingInterval: RollingInterval.Day)
#endif
                .CreateLogger();

            try
            {
                Log.Information("Starting Cadmus __PRJ__ API host");
                DumpEnvironmentVars();

                // this is the place for seeding:
                // see https://stackoverflow.com/questions/45148389/how-to-seed-in-entity-framework-core-2
                // and https://docs.microsoft.com/en-us/aspnet/core/migration/1x-to-2x/?view=aspnetcore-2.1#move-database-initialization-code
                var host = await CreateHostBuilder(args)
                    // add in-memory config to override Serilog connection string
                    // as there is no way of configuring it outside appsettings
                    // https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-5.0#in-memory-provider-and-binding-to-a-poco-class
                    .ConfigureAppConfiguration((context, config) =>
                    {
                        IConfiguration cfg = AppConfigReader.Read();
                        string csTemplate = cfg.GetValue<string>("Serilog:ConnectionString")!;
                        string dbName = cfg.GetValue<string>("DatabaseNames:Data")!;
                        string cs = string.Format(csTemplate, dbName);
                        Debug.WriteLine($"Serilog:ConnectionString override = {cs}");
                        Console.WriteLine($"Serilog:ConnectionString override = {cs}");

                        Dictionary<string, string> dct = new()
                        {
                            { "Serilog:ConnectionString", cs }
                        };
                        // (requires Microsoft.Extensions.Configuration package
                        // to get the MemoryConfigurationProvider)
                        config.AddInMemoryCollection(dct);
                    })
                    .UseSerilog()
                    .Build()
                    .SeedAsync(); // see Services/HostSeedExtension

                host.Run();

                return 0;
            }
            catch (Exception ex)
            {
                Log.Fatal(ex, "Cadmus __PRJ__ API host terminated unexpectedly");
                Debug.WriteLine(ex.ToString());
                Console.WriteLine(ex.ToString());
                return 1;
            }
            finally
            {
                Log.CloseAndFlush();
            }
        }
    }
}
```
