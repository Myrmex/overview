---
layout: page
title: Creating API
subtitle: "Cadmus Backend Development"
---

- [Creating the API Project](#creating-the-api-project)
- [Settings](#settings)
- [Program](#program)
- [Startup](#startup)
- [Assets](#assets)
- [Additional Controllers](#additional-controllers)
- [Docker](#docker)
- [Readme](#readme)

📌 Create the backend API surface layer used by frontend editors.

1. [core](core.md)
2. [parts](parts.md)
3. [part seeders](part-seeders.md)
4. [fragments](fragments.md)
5. [fragment seeders](fragment-seeders.md)
6. [services](services.md)
7. **API**

## Creating the API Project

The [reference API backend project](https://github.com/vedph/cadmus_api) is the model for this section.

(1) create a new ASP.NET Core web API project (no authentication) named `Cadmus<PRJ>Api`: select `None` for `Authentication type`, ensure that `Enable Docker` and `Use HTTPS` is disabled (we'll provide our own Docker files), ensure that `Use controllers`, `Enable OpenAPI support`, and `Do not use top-level statements` are checked.

>Remember to disable HTTPS. In most API configurations HTTPS is managed by a reverse proxy, and this option is not required here in development.

(2) remove the mock `WeatherForecast.cs` class and its corresponding `WeatherForecastController.cs` class from the `Controllers` folder.

(3) add NuGet packages: just paste this code in the project file and then use NuGet package manager to update all the packages (replace `__PRJ__` with your project name, removing project's parts if they are not present):

```xml
<ItemGroup>
  <PackageReference Include="AspNetCore.Identity.Mongo" Version="8.3.3" />
  <PackageReference Include="Cadmus.Api.Controllers" Version="9.0.6" />
  <PackageReference Include="Cadmus.Api.Controllers.Import" Version="9.0.6" />
  <PackageReference Include="Cadmus.Api.Models" Version="9.0.6" />
  <PackageReference Include="Cadmus.Api.Services" Version="9.0.6" />
  <PackageReference Include="Cadmus.Graph" Version="7.0.3" />
  <PackageReference Include="Cadmus.Graph.Ef.PgSql" Version="7.0.3" />
  <PackageReference Include="Cadmus.Graph.Extras" Version="7.0.3" />
  <PackageReference Include="Cadmus.Seed.Codicology.Parts" Version="6.0.3" />
  <PackageReference Include="Cadmus.Seed.General.Parts" Version="6.1.0" />
  <PackageReference Include="Cadmus.Seed.Philology.Parts" Version="8.2.0" />
  <PackageReference Include="Microsoft.AspNetCore.Authentication.JwtBearer" Version="8.0.6" />
  <PackageReference Include="Microsoft.AspNetCore.Mvc.NewtonsoftJson" Version="8.0.6" />
  <PackageReference Include="Microsoft.Extensions.Configuration" Version="8.0.0" />
  <PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
  <PackageReference Include="Polly" Version="8.4.0" />
  <PackageReference Include="Serilog" Version="4.0.0" />
  <PackageReference Include="Serilog.AspNetCore" Version="8.0.1" />
  <PackageReference Include="Serilog.Exceptions" Version="8.4.0" />
  <PackageReference Include="Serilog.Extensions.Hosting" Version="8.0.0" />
  <PackageReference Include="Serilog.Sinks.Console" Version="5.0.1" />
  <PackageReference Include="Serilog.Sinks.File" Version="5.0.0" />
  <PackageReference Include="Serilog.Sinks.MongoDB" Version="5.4.1" />
  <PackageReference Include="Serilog.Sinks.Postgresql.Alternative" Version="4.0.4" />
  <PackageReference Include="Swashbuckle.AspNetCore" Version="6.6.2" />
</ItemGroup>
```

>You can remove the Serilog sinks you are not going to use, like e.g. the PostgreSQL one. Also, typically you will add your project's `Cadmus.PRJ.Services` package(s).

## Settings

Add these settings to `appsettings.json` (replace `__PRJ__` with your project's name). Feel free to customize them as required.

>Please notice that all the sensitive data like users and passwords are there only for illustration purposes, and they will be overwritten by environment variables set in the [host server](../deploy.md).

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
    "Index": "Server=localhost;Database={0};User Id=postgres;Password=postgres;Include Error Detail=True",
    "MongoLog": "mongodb://localhost:27017/cadmus-__PRJ__-log",
    "PostgresLog": "Server=localhost;Database=cadmus-__PRJ__-log;User Id=postgres;Password=postgres;Include Error Detail=True"
  },
  "DatabaseNames": {
    "Auth": "cadmus-__PRJ__-auth",
    "Data": "cadmus-__PRJ__"
  },
  "Serilog": {
    "MaxMbSize": 10,
    "TableName": "Logs",
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Information",
        "System": "Warning"
      }
    },
    "Using": [
      "Serilog.Sinks.Console",
      "Serilog.Sinks.File",
      "Serilog.Sinks.MongoDB",
      "Serilog.Sinks.Postgresql.Alternative"
    ],
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
    "SecureKey": "7W^3*y5@a!3%5Wu4xzd@au5Eh9mdFG6%WmzQpjDEB8#F5nXT"
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
    "IsGraphEnabled": false
  },
  "Preview": {
    "IsEnabled": true
  },
  "Auditing": {
    "File": true,
    "Mongo": true,
    "Postgres": false,
    "Console": true
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

>⚠️ before API v8, which followed [RDBMS refactoring](../history/b-rdbms.md), the index connection string targeted a MySql database: `"Index": "Server=localhost;Database={0};Uid=root;Pwd=mysql;"`. Now it targets a PostgreSQL database, though you can change it to MySql if you prefer, adjusting the code accordingly.

## Program

Use this template to replace the code in `Program.cs` (replace `__PRJ__` with your project's name):

```cs
using System.Collections;
using Serilog;
using System.Diagnostics;
using Cadmus.Api.Services.Seeding;
using NpgsqlTypes;
// remove the PostgreSQL sink imports if not using it
using Serilog.Sinks.PostgreSQL.ColumnWriters;
using Serilog.Sinks.PostgreSQL;
using Fusi.DbManager.PgSql;
using System.Text.RegularExpressions;

namespace Cadmus__PRJ__Api;

/// <summary>
/// Program.
/// </summary>
public static class Program
{
    private static void DumpEnvironmentVars()
    {
        Console.WriteLine("ENVIRONMENT VARIABLES:");
        IDictionary dct = Environment.GetEnvironmentVariables();
        List<string> keys = [];
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

    private static bool IsAuditEnabledFor(IConfiguration config, string key)
    {
        bool? value = config.GetValue<bool?>($"Auditing:{key}");
        return value != null && value != false;
    }

    private static void ConfigurePostgresLogging(HostBuilderContext context,
        LoggerConfiguration loggerConfiguration)
    {
        string? cs = context.Configuration.GetConnectionString("PostgresLog");
        if (string.IsNullOrEmpty(cs))
        {
            Console.WriteLine("Postgres log connection string not found");
            return;
        }

        Regex dbRegex = new("Database=(?<n>[^;]+);?");
        Match m = dbRegex.Match(cs);
        if (!m.Success)
        {
            Console.WriteLine("Postgres log connection string not valid");
            return;
        }
        string cst = dbRegex.Replace(cs, "Database={0};");
        string dbName = m.Groups["n"].Value;
        PgSqlDbManager mgr = new(cst);
        if (!mgr.Exists(dbName))
        {
            Console.WriteLine($"Creating log database {dbName}...");
            mgr.CreateDatabase(dbName, "", null);
        }

        IDictionary<string, ColumnWriterBase> columnWriters =
            new Dictionary<string, ColumnWriterBase>
        {
            { "message", new RenderedMessageColumnWriter(
                NpgsqlDbType.Text) },
            { "message_template", new MessageTemplateColumnWriter(
                NpgsqlDbType.Text) },
            { "level", new LevelColumnWriter(
                true, NpgsqlDbType.Varchar) },
            { "raise_date", new TimestampColumnWriter(
                NpgsqlDbType.TimestampTz) },
            { "exception", new ExceptionColumnWriter(
                NpgsqlDbType.Text) },
            { "properties", new LogEventSerializedColumnWriter(
                NpgsqlDbType.Jsonb) },
            { "props_test", new PropertiesColumnWriter(
                NpgsqlDbType.Jsonb) },
            { "machine_name", new SinglePropertyColumnWriter(
                "MachineName", PropertyWriteMethod.ToString,
                NpgsqlDbType.Text, "l") }
        };

        loggerConfiguration
            .WriteTo.PostgreSQL(cs, "log", columnWriters,
            needAutoCreateTable: true, needAutoCreateSchema: true);
    }

    /// <summary>
    /// Entry point.
    /// </summary>
    /// <param name="args">The arguments.</param>
    public static async Task<int> Main(string[] args)
    {
        try
        {
            Log.Information("Starting Cadmus __PRJ__ API host");
            DumpEnvironmentVars();

            // this is the place for seeding:
            // see https://stackoverflow.com/questions/45148389/how-to-seed-in-entity-framework-core-2
            // and https://docs.microsoft.com/en-us/aspnet/core/migration/1x-to-2x/?view=aspnetcore-2.1#move-database-initialization-code
            var host = await CreateHostBuilder(args)
                .UseSerilog((hostingContext, loggerConfiguration) =>
                {
                    var maxSize = hostingContext.Configuration["Serilog:MaxMbSize"];

                    loggerConfiguration.ReadFrom.Configuration(
                        hostingContext.Configuration);

                    if (IsAuditEnabledFor(hostingContext.Configuration, "File"))
                    {
                        Console.WriteLine("Logging to file enabled");
                        loggerConfiguration.WriteTo.File("cadmus-log.txt",
                            rollingInterval: RollingInterval.Day);
                    }

                    if (IsAuditEnabledFor(hostingContext.Configuration, "Mongo"))
                    {
                        Console.WriteLine("Logging to Mongo enabled");
                        string? cs = hostingContext.Configuration
                            .GetConnectionString("MongoLog");

                        if (!string.IsNullOrEmpty(cs))
                        {
                            loggerConfiguration.WriteTo.MongoDBBson(cs,
                                cappedMaxSizeMb: !string.IsNullOrEmpty(maxSize) &&
                                int.TryParse(maxSize, out int n) && n > 0 ? n : 10);
                        }
                        else
                        {
                            Console.WriteLine("Mongo log connection string not found");
                        }
                    }

                    if (IsAuditEnabledFor(hostingContext.Configuration, "Postgres"))
                    {
                        Console.WriteLine("Logging to Postgres enabled");
                        ConfigurePostgresLogging(hostingContext, loggerConfiguration);
                    }

                    if (IsAuditEnabledFor(hostingContext.Configuration, "Console"))
                    {
                        Console.WriteLine("Logging to console enabled");
                        loggerConfiguration.WriteTo.Console();
                    }
                })
                .Build()
                .SeedAsync(); // see Services/HostSeedExtension

            await host.RunAsync();

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
            await Log.CloseAndFlushAsync();
        }
    }
}
```

## Startup

Use this template for `Startup.cs` (replace `__PRJ__` with your project's name; in ASP.NET 6+ web projects you will need to add this file). The project specific [services](services.md) required in this template either come from an external library or are directly placed in a `Services` folder in this project. The usual convention is placing them in an ad-hoc library when your project has its own specific parts. Otherwise, if you are just assembling parts from other libraries, just create the services in this project.

```cs
using System.Text;
using System.Text.Json;
using MessagingApi;
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.Extensions.Options;
using Microsoft.IdentityModel.Logging;
using Microsoft.IdentityModel.Tokens;
using Microsoft.OpenApi.Models;
using AspNetCore.Identity.Mongo;
using IConfiguration = Microsoft.Extensions.Configuration.IConfiguration;
using System.Reflection;
using Serilog;
using Cadmus.Core;
using Cadmus.Seed;
using Cadmus.Core.Config;
using Cadmus.Index.Config;
using Cadmus.Api.Services.Auth;
using Cadmus.Api.Services.Messaging;
using Cadmus.Api.Services;
using Microsoft.AspNetCore.HttpOverrides;
using Cadmus.__PRJ__.Services;
using Cadmus.Core.Storage;
using Cadmus.Export.Preview;
using Cadmus.Graph;

namespace Cadmus__PRJ__Api;

/// <summary>
/// Startup.
/// </summary>
public sealed class Startup
{
    /// <summary>
    /// Gets the configuration.
    /// </summary>
    public IConfiguration Configuration { get; }

    /// <summary>
    /// Gets the host environment.
    /// </summary>
    public IHostEnvironment HostEnvironment { get; }

    /// <summary>
    /// Initializes a new instance of the <see cref="Startup"/> class.
    /// </summary>
    /// <param name="configuration">The configuration.</param>
    /// <param name="environment">The environment.</param>
    public Startup(IConfiguration configuration, IHostEnvironment environment)
    {
        Configuration = configuration;
        HostEnvironment = environment;
    }

    /// <summary>
    /// Configures the options services providing typed configuration objects.
    /// </summary>
    /// <param name="services">The services.</param>
    private void ConfigureOptionsServices(IServiceCollection services)
    {
        // configuration sections
        // https://andrewlock.net/adding-validation-to-strongly-typed-configuration-objects-in-asp-net-core/
        services.Configure<MessagingOptions>(Configuration.GetSection("Messaging"));
        services.Configure<DotNetMailerOptions>(Configuration.GetSection("Mailer"));

        // explicitly register the settings object by delegating to the IOptions object
        services.AddSingleton(resolver =>
            resolver.GetRequiredService<IOptions<MessagingOptions>>().Value);
        services.AddSingleton(resolver =>
            resolver.GetRequiredService<IOptions<DotNetMailerOptions>>().Value);
    }

    /// <summary>
    /// Configures the CORS services. Allowed origins are read from configuration
    /// <c>AllowedOrigins</c> (an array of URIs). If this section does not exist,
    /// the only allowed origin is <c>http://localhost:4200</c>.
    /// </summary>
    /// <param name="services">The services.</param>
    private void ConfigureCorsServices(IServiceCollection services)
    {
        string[] origins = new[] { "http://localhost:4200" };

        IConfigurationSection section = Configuration.GetSection("AllowedOrigins");
        if (section.Exists())
        {
            origins = section.AsEnumerable()
                .Where(p => !string.IsNullOrEmpty(p.Value))
                .Select(p => p.Value).ToArray()!;
        }

        services.AddCors(o => o.AddPolicy("CorsPolicy", builder =>
        {
            builder.AllowAnyMethod()
                .AllowAnyHeader()
                // https://github.com/aspnet/SignalR/issues/2110 for AllowCredentials
                .AllowCredentials()
                .WithOrigins(origins);
        }));
    }

    /// <summary>
    /// Configures the authentication services. These refer to a database
    /// whose connection string is built from a template in configuration
    /// <c>ConnectionStrings:Default</c>, whose database name is filled by
    /// the value from <c>DatabaseNames:Auth</c>. JWT tokens are configured
    /// according to <c>Jwt:SecureKey</c>, <c>Jwt:Audience</c>, and
    /// <c>Jwt:Issuer</c>.
    /// </summary>
    /// <param name="services">The services.</param>
    private void ConfigureAuthServices(IServiceCollection services)
    {
        // identity
        string connStringTemplate = Configuration.GetConnectionString("Default")!;

        services.AddIdentityMongoDbProvider<ApplicationUser, ApplicationRole>(
            options => { },
            mongoOptions =>
            {
                mongoOptions.ConnectionString =
                    string.Format(connStringTemplate,
                    Configuration.GetSection("DatabaseNames")["Auth"]);
            });

        // authentication service
        services
            .AddAuthentication(options =>
            {
                options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
                options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
                options.DefaultScheme = JwtBearerDefaults.AuthenticationScheme;
            })
            .AddJwtBearer(options =>
            {
            // NOTE: remember to set the values in configuration:
            // Jwt:SecureKey, Jwt:Audience, Jwt:Issuer
            IConfigurationSection jwtSection = Configuration.GetSection("Jwt");
                string key = jwtSection["SecureKey"]!;
                if (string.IsNullOrEmpty(key))
                    throw new InvalidOperationException("Required JWT SecureKey not found");

                options.SaveToken = true;
                options.RequireHttpsMetadata = false;
                options.TokenValidationParameters = new TokenValidationParameters()
                {
                    ValidateIssuer = true,
                    ValidateAudience = true,
                    ValidAudience = jwtSection["Audience"],
                    ValidIssuer = jwtSection["Issuer"],
                    IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(key))
                };
            });
#if DEBUG
        // use to show more information when troubleshooting JWT issues
        IdentityModelEventSource.ShowPII = true;
#endif
    }

    /// <summary>
    /// Configures the Swagger services to include comments from code and
    /// use JWT authentication.
    /// </summary>
    /// <param name="services">The services.</param>
    private static void ConfigureSwaggerServices(IServiceCollection services)
    {
        services.AddSwaggerGen(c =>
        {
            c.SwaggerDoc("v1", new OpenApiInfo
            {
                Version = "v1",
                Title = "API",
                Description = "Cadmus __PRJ__ Services"
            });
            c.DescribeAllParametersInCamelCase();

            // include XML comments
            // (remember to check the build XML comments in the prj props)
            string xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
            string xmlPath = Path.Combine(AppContext.BaseDirectory, xmlFile);
            if (File.Exists(xmlPath))
                c.IncludeXmlComments(xmlPath);

            // JWT
            // https://stackoverflow.com/questions/58179180/jwt-authentication-and-swagger-with-net-core-3-0
            // (cf. https://ppolyzos.com/2017/10/30/add-jwt-bearer-authorization-to-swagger-and-asp-net-core/)
            c.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
            {
                In = ParameterLocation.Header,
                Description = "Please insert JWT with Bearer into field",
                Name = "Authorization",
                Type = SecuritySchemeType.ApiKey
            });
            c.AddSecurityRequirement(new OpenApiSecurityRequirement {
            {
                new OpenApiSecurityScheme
                {
                    Reference = new OpenApiReference
                    {
                        Type = ReferenceType.SecurityScheme,
                        Id = "Bearer"
                    }
                },
                Array.Empty<string>()
            }
        });
        });
    }

    private CadmusPreviewer GetPreviewer(IServiceProvider provider)
    {
        // get dependencies
        ICadmusRepository repository =
                provider.GetService<IRepositoryProvider>()!.CreateRepository();
        ICadmusPreviewFactoryProvider factoryProvider =
            new StandardCadmusPreviewFactoryProvider();

        // nope if disabled
        if (!Configuration.GetSection("Preview").GetSection("IsEnabled")
            .Get<bool>())
        {
            return new CadmusPreviewer(factoryProvider.GetFactory("{}"),
                repository);
        }

        // get profile source
        ILogger<Startup>? logger = provider.GetService<ILogger<Startup>>();
        IHostEnvironment env = provider.GetService<IHostEnvironment>()!;
        string path = Path.Combine(env.ContentRootPath,
            "wwwroot", "preview-profile.json");
        if (!File.Exists(path))
        {
            logger?.LogError("Preview profile expected at {Path} not found", path);
            return new CadmusPreviewer(factoryProvider.GetFactory("{}"),
                repository);
        }

        // load profile
        logger?.LogInformation("Loading preview profile from {Path}...", path);
        string profile;
        using (StreamReader reader = new(new FileStream(
            path, FileMode.Open, FileAccess.Read, FileShare.Read), Encoding.UTF8))
        {
            profile = reader.ReadToEnd();
        }
        CadmusPreviewFactory factory = factoryProvider.GetFactory(profile);

        return new CadmusPreviewer(factory, repository);
    }

    /// <summary>
    /// Configures the item index services with the connection string template
    /// from <c>ConnectionStrings:Index</c>, whose database name is defined in
    /// <c>DatabaseNames:Data</c>.
    /// </summary>
    /// <param name="services">The services.</param>
    private void ConfigureIndexServices(IServiceCollection services)
    {
        // item index factory provider (from ConnectionStrings/Index)
        string cs = string.Format(Configuration.GetConnectionString("Index")!,
            Configuration.GetValue<string>("DatabaseNames:Data"));

        services.AddSingleton<IItemIndexFactoryProvider>(_ =>
            new StandardItemIndexFactoryProvider(cs));
    }

    /// <summary>
    /// Configures the item graph services with the connection string template
    /// from <c>ConnectionStrings:Graph</c> (falling back to <c>:Index</c> if
    /// not found), whose database name is defined in <c>DatabaseNames:Data</c>
    /// plus suffix <c>-graph</c>.
    /// </summary>
    /// <param name="services">The services.</param>
    private void ConfigureGraphServices(IServiceCollection services)
    {
        string cs = string.Format(Configuration.GetConnectionString("Graph")
            ?? Configuration.GetConnectionString("Index")!,
            Configuration.GetValue<string>("DatabaseNames:Data") + "-graph");

        services.AddSingleton<IItemGraphFactoryProvider>(_ =>
            new StandardItemGraphFactoryProvider(cs));

        services.AddSingleton<IGraphRepository>(_ =>
        {
            var repository = new EfPgSqlGraphRepository();
            repository.Configure(new EfGraphRepositoryOptions
            {
                ConnectionString = cs
            });
            return repository;
        });

        // graph updater
        services.AddTransient<GraphUpdater>(provider =>
        {
            IRepositoryProvider rp = provider.GetService<IRepositoryProvider>()!;
            return new(provider.GetService<IGraphRepository>()!)
            {
                // we want item-eid as an additional metadatum, derived from
                // eid in the role-less MetadataPart of the item, when present
                MetadataSupplier = new MetadataSupplier()
                    .SetCadmusRepository(rp.CreateRepository())
                    .AddItemEid()
            };
        });
    }

    /// <summary>
    /// Configures the services.
    /// </summary>
    /// <param name="services">The services.</param>
    public void ConfigureServices(IServiceCollection services)
    {
        // configuration
        ConfigureOptionsServices(services);

        // CORS (before MVC)
        ConfigureCorsServices(services);

        // base services
        services.AddControllers();
        // camel-case JSON in response
        services.AddMvc()
            // https://docs.microsoft.com/en-us/aspnet/core/migration/22-to-30?view=aspnetcore-2.2&tabs=visual-studio#jsonnet-support
            // Newtonsoft is required by MongoDB
            .AddNewtonsoftJson()
            .AddJsonOptions(options =>
            {
                options.JsonSerializerOptions.PropertyNamingPolicy =
                    JsonNamingPolicy.CamelCase;
            });

        // authentication
        ConfigureAuthServices(services);

        // Add framework services
        // for IMemoryCache: https://docs.microsoft.com/en-us/aspnet/core/performance/caching/memory
        services.AddMemoryCache();

        // user repository service
        services.AddTransient<IUserRepository<ApplicationUser>,
            ApplicationUserRepository>();

        // messaging
        // TODO: you can use another mailer service here. In this case,
        // also change the types in ConfigureOptionsServices.
        services.AddTransient<IMailerService, DotNetMailerService>();
        services.AddTransient<IMessageBuilderService,
            FileMessageBuilderService>();

        // configuration
        services.AddSingleton(_ => Configuration);
        // repository
        string dataCS = string.Format(
            Configuration.GetConnectionString("Default")!,
            Configuration.GetValue<string>("DatabaseNames:Data"));
        services.AddSingleton<IRepositoryProvider>(
          _ => new __PRJ__RepositoryProvider { ConnectionString = dataCS });

        // part seeder factory provider
        services.AddSingleton<IPartSeederFactoryProvider,
            __PRJ__PartSeederFactoryProvider>();
        // item browser factory provider
        services.AddSingleton<IItemBrowserFactoryProvider>(_ =>
            new StandardItemBrowserFactoryProvider(
                Configuration.GetConnectionString("Default")!));

        // index and graph
        ConfigureIndexServices(services);
        ConfigureGraphServices(services);

        // previewer
        services.AddSingleton(p => GetPreviewer(p));

        // swagger
        ConfigureSwaggerServices(services);
    }

    /// <summary>
    /// Configures the specified application.
    /// </summary>
    /// <param name="app">The application.</param>
    /// <param name="env">The environment.</param>
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        // https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/linux-nginx?view=aspnetcore-2.2#configure-a-reverse-proxy-server
        app.UseForwardedHeaders(new ForwardedHeadersOptions
        {
            ForwardedHeaders = ForwardedHeaders.XForwardedFor
                | ForwardedHeaders.XForwardedProto
        });

        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }
        else
        {
            // https://docs.microsoft.com/en-us/aspnet/core/security/enforcing-ssl?view=aspnetcore-5.0&tabs=visual-studio
            app.UseExceptionHandler("/Error");
            if (Configuration.GetValue<bool>("Server:UseHSTS"))
            {
                Console.WriteLine("HSTS: yes");
                app.UseHsts();
            }
            else
            {
                Console.WriteLine("HSTS: no");
            }
        }

        if (Configuration.GetValue<bool>("Server:UseHttpsRedirection"))
        {
            Console.WriteLine("HttpsRedirection: yes");
            app.UseHttpsRedirection();
        }
        else
        {
            Console.WriteLine("HttpsRedirection: no");
        }

        app.UseHttpsRedirection();
        app.UseRouting();
        // CORS
        app.UseCors("CorsPolicy");
        app.UseAuthentication();
        app.UseAuthorization();

        app.UseEndpoints(endpoints =>
        {
            endpoints.MapControllers();
        });

        // Swagger
        app.UseSwagger();
        app.UseSwaggerUI(options =>
        {
            string? url = Configuration.GetValue<string>("Swagger:Endpoint");
            if (string.IsNullOrEmpty(url)) url = "v1/swagger.json";
            options.SwaggerEndpoint(url, "V1 Docs");
        });
    }
}
```

>⚠️ Before API v8, the `ConfigureIndexServices` method was used for both index and graph:

```cs
// WARNING: OBSOLETE!

private void ConfigureIndexServices(IServiceCollection services)
{
    // item index factory provider
    string indexCS = string.Format(
        Configuration.GetConnectionString("Index")!,
        Configuration.GetValue<string>("DatabaseNames:Data"));

    services.AddSingleton<IItemIndexFactoryProvider>(_ =>
        new StandardItemIndexFactoryProvider(indexCS));

    // graph repository
    services.AddSingleton<IGraphRepository>(_ =>
    {
        var repository = new MySqlGraphRepository();
        repository.Configure(new SqlOptions
        {
            ConnectionString = indexCS
        });
        return repository;
    });

    // graph updater
    services.AddTransient<GraphUpdater>(provider =>
    {
        IRepositoryProvider rp = provider.GetService<IRepositoryProvider>()!;
        return new(provider.GetService<IGraphRepository>()!)
        {
            // we want item-eid as an additional metadatum, derived from
            // eid in the role-less MetadataPart of the item, when present
            // NOTE: this requires Cadmus.Graph.Extras
            MetadataSupplier = new MetadataSupplier()
                .SetCadmusRepository(rp.CreateRepository())
                .AddItemEid()
        };
    });
}
```

## Assets

Copy the whole `wwwroot` from [CadmusApi](https://github.com/vedph/cadmus_api), and customize its contents (the Cadmus profile, and if needed the messages template text).

Inside that folder, edit:

- the `seed-profile.json` file, which contains the full Cadmus configuration for data and editors used in the project:
  - remove all the parts, fragments, and seeders you do not use and all the parts, fragment, and seeders you require;
  - do the same for thesaurus entries.
- the `preview-profile.json` file, which contains the configuration for parts preview. If you have no preview, just use an empty JSON object `{}` as its content.

This is the core customization for the whole project. Usually, the profile file is created after the documentation is completed, and before creating the code.

Inside the `messages` folder you can customize the message templates as you prefer, but usually this is not required.

## Additional Controllers

If required, you might want to add more controllers specific to your API. For instance, many projects use a simple [proxy controller](https://github.com/vedph/proxy-api) to proxy some API calls bypassing issues connected to the lack of CORS support in the target server (e.g. for DBPedia).

## Docker

(1) In the project's root (where the `.sln` file is located), add a `Dockerfile` to build the Docker image (replace `__PRJ__` with your project's name):

```yml
# Stage 1: base
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 8080

# Stage 2: build
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY ["Cadmus__PRJ__Api/Cadmus__PRJ__Api.csproj", "Cadmus__PRJ__Api/"]
# copy local packages to avoid using a NuGet custom feed, then restore
# COPY ./local-packages /src/local-packages
RUN dotnet restore "Cadmus__PRJ__Api/Cadmus__PRJ__Api.csproj" -s https://api.nuget.org/v3/index.json --verbosity n
# copy the content of the API project
COPY . .
# build it
RUN dotnet build "Cadmus__PRJ__Api/Cadmus__PRJ__Api.csproj" -c Release -o /app/build

# Stage 3: publish
FROM build AS publish
RUN dotnet publish "Cadmus__PRJ__Api/Cadmus__PRJ__Api.csproj" -c Release -o /app/publish

# Stage 4: final
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "Cadmus__PRJ__Api.dll"]
```

(2) add a `docker-compose.yml` file to allow you using the API in a composer stack (replace `PRJ` with your project name; of course, you can change your image name as required to fit your organization).

⚠️ **ATTENTION**: under cadmus-api ports replace `5052` with the port value used by your API project (you can find it under the project's properties, Debug, Launch Profiles, HTTP).

```yml
services:
  # MongoDB
  cadmus-PRJ-mongo:
    image: mongo
    container_name: cadmus-PRJ-mongo
    environment:
      - MONGO_DATA_DIR=/data/db
      - MONGO_LOG_DIR=/dev/null
    command: mongod --logpath=/dev/null
    ports:
      - 27017:27017
    networks:
      - cadmus-PRJ-network

  # PostgreSQL
  cadmus-PRJ-pgsql:
    image: postgres
    container_name: cadmus-PRJ-pgsql
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=postgres
    ports:
      - 5432:5432
    networks:
      - cadmus-PRJ-network

  # Biblio API
  # TODO: remove if you are not using it
  cadmus-biblio-api:
    image: vedph2020/cadmus_biblio_api:5.1.1
    container_name: cadmus-biblio-api
    ports:
      - 60058:8080
    depends_on:
      - cadmus-PRJ-mongo
      - cadmus-PRJ-pgsql
    environment:
      - CONNECTIONSTRINGS__DEFAULT=mongodb://cadmus-PRJ-mongo:27017/{0}
      - CONNECTIONSTRINGS__BIBLIO=Server=cadmus-PRJ-pgsql;port=5432;Database={0};User Id=postgres;Password=postgres;Include Error Detail=True
      - SEED__BIBLIODELAY=50
      - SERILOG__CONNECTIONSTRING=mongodb://cadmus-PRJ-mongo:27017/{0}-log
      - STOCKUSERS__0__PASSWORD=P4ss-W0rd!
    networks:
      - cadmus-PRJ-network

  # Cadmus PRJ API
  cadmus-PRJ-api:
    image: vedph2020/cadmus-PRJ-api:0.0.1
    container_name: cadmus-PRJ-api
    ports:
      # TODO: set your port replacing 5052
      - 5052:8080
    depends_on:
      - cadmus-PRJ-mongo
      - cadmus-PRJ-pgsql
    environment:
      - CONNECTIONSTRINGS__DEFAULT=mongodb://cadmus-PRJ-mongo:27017/{0}
      - CONNECTIONSTRINGS__INDEX=Server=cadmus-PRJ-pgsql;port=5432;Database={0};User Id=postgres;Password=postgres;Include Error Detail=True
      - SERILOG__CONNECTIONSTRING=mongodb://cadmus-PRJ-mongo:27017/{0}-log
      - STOCKUSERS__0__PASSWORD=P4ss-W0rd!
      - SEED__INDEXDELAY=25
      - MESSAGING__APIROOTURL=http://cadmusapi.azurewebsites.net
      - MESSAGING__APPROOTURL=http://cadmusapi.com/
      - MESSAGING__SUPPORTEMAIL=support@cadmus.com
    networks:
      - cadmus-PRJ-network

networks:
  cadmus-PRJ-network:
    driver: bridge
```

>API before v8 used MySql:

```yml
  # MySql
  # https://github.com/docker-library/docs/tree/master/mysql#mysql_database
  # https://docs.docker.com/samples/library/mysql/#environment-variables
  # https://github.com/docker-library/mysql/issues/275 (troubleshooting connection)
  cadmus-PRJ-mysql:
    image: mysql
    container_name: cadmus-PRJ-mysql
    # https://github.com/docker-library/mysql/issues/454
    command: --default-authentication-plugin=mysql_native_password
    # https://stackoverflow.com/questions/55559386/how-to-fix-mbind-operation-not-permitted-in-mysql-error-log
    cap_add:
      - SYS_NICE  # CAP_SYS_NICE
    environment:
      # the password that will be set for the MySQL root superuser account
      # Note: use dictionary like here rather than array (- name = value)
      # or you might get MySql connection errors!
      # https://stackoverflow.com/questions/37459031/connecting-to-a-docker-compose-mysql-container-denies-access-but-docker-running/37460872#37460872
      MYSQL_ROOT_PASSWORD: mysql
      MYSQL_ROOT_HOST: '%'
    ports:
      - 3306:3306
    networks:
      - cadmus-PRJ-network
```

(3) add a `.dockerignore` file with this content:

```txt
**/.classpath
**/.dockerignore
**/.env
**/.git
**/.gitignore
**/.project
**/.settings
**/.toolstarget
**/.vs
**/.vscode
**/*.*proj.user
**/*.dbmdl
**/*.jfm
**/azds.yaml
**/bin
**/charts
**/docker-compose*
**/Dockerfile*
**/node_modules
**/npm-debug.log
**/obj
**/secrets.dev.yaml
**/values.dev.yaml
LICENSE
README.md
```

To build a Docker image (replace `PRJ` with your project's name):

```ps1
docker build . -t vedph2020/cadmus-__PRJ__-api:1.0.0 -t vedph2020/cadmus-__PRJ__-api:latest
```

## Readme

Add a readme like this:

```txt
# Cadmus PRJ API

🐋 Quick Docker image build:

    docker build . -t vedph2020/cadmus-__PRJ__-api:0.0.1 -t vedph2020/cadmus-__PRJ__-api:latest

(replace with the current version).

This is a Cadmus API layer customized for the PRJ project. Most of its code is derived from shared Cadmus libraries.
```

🏠 [developer's home](../toc.md)
