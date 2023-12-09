---
layout: page
title: Cadmus Development
subtitle: History - Logging Configuration
---

## Logging Configuration

📆 Date: 2023-12-06

### Rationale

Provide more granular logging options, especially useful when you need to troubleshoot your server setup.

### Affected Products

All the backend API projects.

### Upgrade Path

More granular options have been added to the API. To include them in your project, follow these steps:

(1) add package `Serilog.Sinks.Postgresql.Alternative` if you want to include also the option to log into a PostgreSQL database.

(2) add these entries to your `appsettings.json`, enabling and disabling the various targets at will:

```json
  "Auditing": {
    "File": true,
    "Mongo": true,
    "Postgres": true,
    "Console": true
  },
```

In the same file, change `Log` into `MongoLog` in `ConnectionStrings`; and add PostgresLog if you want PostgreSQL output (replace `PRJ` with your short project name):

```json
  "ConnectionStrings": {
    ...
    "MongoLog": "mongodb://localhost:27017/cadmus-PRJ-log",
    "PostgresLog": "Server=localhost;Database=cadmus-PRJ-log;User Id=postgres;Password=postgres;Include Error Detail=True"
  },
```

Finally, ensure that your `Serilog` entry in this file has entries in `Using` (remove the ones you do not use):

```json
"Using": [
    "Serilog.Sinks.Console",
    "Serilog.Sinks.File",
    "Serilog.Sinks.MongoDB",
    "Serilog.Sinks.Postgresql.Alternative"
],
```

(3) in `Program.cs` add this code:

```cs
private static bool IsAuditEnabledFor(IConfiguration config, string key)
{
    bool? value = config.GetValue<bool?>($"Auditing:{key}");
    return value != null && value != false;
}

private static void ConfigurePostgreLogging(HostBuilderContext context,  loggerConfiguration)
{
    string cs = context.Configuration.GetConnectionString("PostgresLog");
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
```

(4) also, in your `Program.cs` `Main` method, replace the host creation with this:

```cs
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
            string cs = hostingContext.Configuration
                .GetConnectionString("MongoLog");

            if (!string.IsNullOrEmpty(cs))
            {
                loggerConfiguration.WriteTo.MongoDBCapped(cs,
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
            ConfigurePostgreLogging(hostingContext, loggerConfiguration);
        }

        if (IsAuditEnabledFor(hostingContext.Configuration, "Console"))
        {
            Console.WriteLine("Logging to console enabled");
            loggerConfiguration.WriteTo.Console();
        }
    })
    .Build()
    .SeedAsync(); // see Services/HostSeedExtension
```
