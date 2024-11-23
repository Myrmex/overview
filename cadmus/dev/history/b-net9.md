---
layout: page
title: Cadmus Development
subtitle: History - Backend API Update
---

## Backend Graph Upgrade

📆 Date: 2024-11-18

### Rationale

The upgrade to .NET 9 implies some relevant changes:

- drop Swagger (whose development was left behind) for Scalar, integrating the new OpenAPI capabilities from .NET 9.
- drop MongoDB-based authentication for RDBMS-based authentication via PostgreSQL. This does not affect the system, except for the requirement to migrate user accounts from MongoDB to PostgreSQL, which is best accomplished by just recreating them in the new environment.
- refactor the API code to reflect the more modern code style already in use from past versions of .NET, removing the `Startup` class. This also allows moving most of the API app code to a new library, `Cadmus.Api.Config`, thus making the API code much more streamlined.

### Affected Products

⚠️ All the API libraries and apps.

### Upgrade Path

Provided that you have upgraded to .NET 9 your models (if any):

1. upgrade to .NET 9 (including the Dockerfile references to .NET images).
2. update all packages.
3. add packages:

    ```xml
    <PackageReference Include="Cadmus.Api.Config" Version="10.1.2" />
    <PackageReference Include="Microsoft.AspNetCore.OpenApi" Version="9.0.0" />
    <PackageReference Include="Scalar.AspNetCore" Version="1.2.43" />
    ```

    >Update versions of these packages if newer are available.

4. remove packages `AspNetCore.Identity.Mongo` and `Swashbuckle.AspNetCore`.
5. add the `Auth` connection string to `appconfig.json` under section `ConnectionStrings`: `"Auth": "Server=localhost;Database={0};User Id=postgres;Password=postgres;Include Error Detail=True",`.
6. in `appconfig.json`, add rate limiter configuration so that it is disabled by default:

    ```json
    "RateLimit": {
      "IsDisabled": true,
      "PermitLimit": 100,
      "QueueLimit": 0,
      "TimeWindow": "00:01:00"
    },
    ```

7. in `appconfig.json`, change `Seed:IndexDelay` entry to `Seed:Delay`. Also change the environment variable name in Docker compose accordingly to `SEED__DELAY`.
8. replace `Program.cs` with the code from the [new API v10+ solution](https://github.com/vedph/cadmus-api/blob/master/CadmusApi/Program.cs). The only change required refers to the name of the repository and part seeder factory providers for your project, in `Program.ConfigureAppServices`: `__PRJ__RepositoryProvider` and `__PRJ__PartSeederFactoryProvider` (see `Startup.ConfigureServices`).
9. remove `Startup.cs`.
10. in the API project Debug properties, change the startup route from `swagger` to `scalar/v1`.
11. update the .NET image versions to 9 in your Dockerfile, ensuring that 8080 is the exposed port.
12. in your Docker compose:
    - add the environment variable to override the connection to the AUTH database:
    - ensure that ASP.NET core port is set to 8080 and that the environment variable `ASPNETCORE_URLS` is properly set.
    - ensure that `SEED__DELAY` (renamed from `SEED__INDEXDELAY`) is set if you are going to seed mock items, so that there is a pause before the API starts using the database service when this is still starting.

Example:

```yml
  cadmus-vela-api:
    image: vedph2020/cadmus-vela-api:4.0.1
    container_name: cadmus-vela-api
    ports:
      - 5080:8080
    depends_on:
      - cadmus-vela-mongo
      - cadmus-vela-pgsql
    environment:
      # for Windows use : as separator, for non Windows use __
      # (see https://github.com/aspnet/Configuration/issues/469)
      - ASPNETCORE_URLS=http://+:8080
      - CONNECTIONSTRINGS__DEFAULT=mongodb://cadmus-vela-mongo:27017/{0}
      - CONNECTIONSTRINGS__AUTH=Server=cadmus-vela-pgsql;port=5432;Database={0};User Id=postgres;Password=postgres;Include Error Detail=True
      - CONNECTIONSTRINGS__INDEX=Server=cadmus-vela-pgsql;port=5432;Database={0};User Id=postgres;Password=postgres;Include Error Detail=True
      - SERILOG__CONNECTIONSTRING=mongodb://cadmus-vela-mongo:27017/{0}-log
      - STOCKUSERS__0__PASSWORD=P4ss-W0rd!
      - SEED__DELAY=20
      - MESSAGING__APIROOTURL=http://cadmusapi.azurewebsites.net
      - MESSAGING__APPROOTURL=http://cadmusapi.com/
      - MESSAGING__SUPPORTEMAIL=support@cadmus.com
    networks:
      - cadmus-vela-network
```
