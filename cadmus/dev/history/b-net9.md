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
    <PackageReference Include="Cadmus.Api.Config" Version="10.1.1" />
    <PackageReference Include="Scalar.AspNetCore" Version="1.2.39" />
    ```

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

7. replace `Program.cs` with the code from the [new API v10+ solution](https://github.com/vedph/cadmus-api/blob/master/CadmusApi/Program.cs). The only change required refers to the name of the repository and part seeder factory providers for your project, in `Program.ConfigureAppServices`: `__PRJ__RepositoryProvider` and `__PRJ__PartSeederFactoryProvider` (see `Startup.ConfigureServices`).
8. remove `Startup.cs`.
9. in the API project Debug properties, change the startup route from `swagger` to `scalar/v1`.