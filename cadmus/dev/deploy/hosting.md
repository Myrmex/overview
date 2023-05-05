---
layout: page
title: Hosting Cadmus
subtitle: Cadmus Deployment
---

- [Services](#services)
- [API Settings](#api-settings)
  - [Overriding Settings](#overriding-settings)
  - [Security](#security)
    - [AllowedOrigins](#allowedorigins)
    - [Jwt](#jwt)
    - [Server](#server)
    - [StockUsers](#stockusers)
    - [Auditing and Privacy](#auditing-and-privacy)
  - [Database](#database)
    - [External Databases](#external-databases)
  - [Other Services](#other-services)
- [App Settings](#app-settings)

In this section you will find some general advice for hosting Cadmus on a server. Even though each configuration has its own requirements and peculiarities, you can use this guide as a starting point.

Essentially, to host Cadmus on a server you should just customize the default _Docker compose script_, which is designed for hosting the system on a local machine. This usually resolves to changing some URIs, and replacing mock security data.

## Services

The default script skeleton includes 4 services:

- `cadmus-mongo`: MongoDB service.
- `cadmus-index`: MySql service used for indexes and semantic graphs.
- `cadmus-api`: API backend service. This is exposed at some port in `localhost`. This is ASP.NET 7 served by [Kestrel](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel).
- `cadmus-web`: web app service, exposed at port 4200 in `localhost`. This is Angular served by [NGINX](https://www.nginx.com/).

>⚠️ Please notice that if you want your data to persist when containers are destroyed, you must use Docker _volumes_ for databases. You can find a ready-to-use sample of this in a variant of the default Docker compose script.

## API Settings

All the API settings are defined in `appsettings.json`. This is packed in the Docker image, but you can override any of its values by just providing an _environment variable_ in the Docker compose script. So, you don't need to change the Docker image, but just setup some environment variables in your host.

As `appsettings.json` is a JSON document, overriding any of its properties means that you must use an expression to point to it. In the ASP.NET convention, used here, you just have to append the name of each property (case insensitive; conventionally, use uppercase for environment variables) starting from the root object. Each name is separated by a double underscore (or by `:` in Windows hosts: see [this post](https://github.com/aspnet/Configuration/issues/469)).

>👉 The double underscore is supported by all platforms, and is automatically converted into a colon. Paths are case insensitive.

For instance, say your configuration document has a `ConnectionStrings` object like this:

```json
{
  "ConnectionStrings": {
    "Default": "mongodb://localhost:27017/{0}",
    "Index": "Server=localhost;Database={0};Uid=root;Pwd=mysql;"
  }
}
```

The path to the `Default` connection property will be `CONNECTIONSTRINGS__DEFAULT` (notice the 2 underscores between its words). This is the name of the environment variable to set for overriding the value of this setting.

When using arrays, use the index of each item in it to refer to that item. So, in an array like this:

```json
"AllowedOrigins": [
  "http://localhost:4200",
  "https://www.fusisoft.it/"
],
```

the first item of the array will be targeted by `ALLOWEDORIGINS__0`; the second item by `ALLOWEDORIGINS__1`; and so forth. The default items are used for testing.

In this section I list the most relevant settings from the perspective of a server setup. The settings marked with 🚩 _must_ be changed to adapt to the hosting environment.

### Overriding Settings

To override settings you typically use environment variables in the host. Every sensitive setting is systematically overridden; other settings can be overridden at will, to customize the API's behavior and fit it into its hosting environment.

The Cadmus API uses the standard ASP.NET Core 3 [default configuration](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-3.1#default-configuration) order, namely:

1. settings from `appsettings.json`;
2. settings from the environment-specific versions of `appsettings.json` (named like `appsettings.{Environment}.json`);
3. secret manager (in the `Development` environment); this relies on a user secrets file, a JSON file stored on the local developer's machine, outside of the source directory (and thus of source control);
4. environment variables;
5. command-line arguments.

The last override wins. Thus, the command line arguments have the highest precedence, followed by environment variables, etc.

As for [command line](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-3.1#command-line-configuration-provider), these are the main points:

- the argument is a key=value pair. The value (which can be empty) either follows the `=` sign, or a space when the key is prefixed with `--` or `/`.
- do not mix the two alternative syntaxes (with `=` or prefix).

Samples from the quoted documentation:

```bash
dotnet run CommandLineKey1=value1 --CommandLineKey2=value2 /CommandLineKey3=value3
dotnet run --CommandLineKey1 value1 /CommandLineKey2 value2
dotnet run CommandLineKey1= CommandLineKey2=value2
```

### Security

Security settings comprise the most relevant changes in your configuration. Please be sure to carefully follow these advices, as failing in doing so might compromise your system's security.

#### AllowedOrigins

🚩 Cadmus API implement [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) for cross-origin access. You can configure the allowed origins for CORS under the `AllowedOrigins` section of the API settings. This is an array of strings, listing all the allowed CORS origins, like:

```json
"AllowedOrigins": [
  "http://localhost:4200",
],
```

Here, you must replace the `localhost` origin (`ALLOWEDORIGINS__0`) with the URL of your host, like:

```yml
  environment:
    - ALLOWEDORIGINS__0=http://100.101.102.103
```

#### Jwt

🚩 This object contains the configuration for the authentication JWT token. Among its properties, the relevant one is `SecureKey`, which contains a seed for generating the token. This must be a string whose length _must_ be a multiple of 8. Use some long and complex string here; it should at least be 32 characters long, but the longer the better. For instance:

```yml
  environment:
    - JWT__SECUREKEY=Rh+m(dkh_Rn6DhOD-wKcd;>P=]Q*T}J/MPbnfenDKOL[1y4I_1Oy1JAU./V98Zex
```

#### Server

This object contains the configuration for HTTPS. You might need this if you are configuring HTTPS at the level of the Docker containers. If instead you are using a reverse proxy to handle HTTPS, and dealing with HTTP-only containerized services (which is the easiest and most common configuration), this is not necessary.

```json
"Server": {
  "UseHSTS": false,
  "UseHttpsRedirection": false
}
```

You can turn these settings `true` to enforce HTTPS in Kestrel.

#### StockUsers

🚩 This array contains all the stock users, i.e. those users which are seeded together with the database. Typically here we have at least 1 admin user, to ensure that there always is an account for configuring the system.

Each stock user object has its username (which must be unique), password, email address, roles, and first and last name, like in this example:

```json
"StockUsers": [
    {
      "UserName": "zeus",
      "Password": "P4ss-W0rd!",
      "Email": "zeus@gmail.com",
      "Roles": [
        "admin",
        "editor"
      ],
      "FirstName": "Daniele",
      "LastName": "Fusi"
    }
  ],
```

Here, you must at least replace the default stock user _password_. Of course, you can change all the other settings at will; but the password provided in the image is the same mock password used in developing, and thus published in the code repository. So, as for any other security related settings, you must change this sensitive data. For instance:

```yml
  environment:
    - STOCKUSERS__0__PASSWORD=2R$M&o*4ej4@
```

>⚠️ Please ensure that you are complying with all the requirements setup in Cadmus for a password, or you will get errors at startup. The password must include at least 8 characters, uppercase and lowercase letters, digits, and symbols like dash, stop, parentheses, etc.

#### Auditing and Privacy

Cadmus has a granular auditing policy for data being edited. Most edits are logged in the auditing log (hosted in a MongoDB database), and the full editing history of each datum is stored in the data themselves. Please take the appropriate measures to periodically check this log for suspected activities, and protect it to comply with privacy requirements.

Anyway, no personal data is directly found in the log, as it just stores user names, which usually mean nothing outside a team. For instance, my user name for testing is "zeus"; and the mapping between zeus and my real name, if any, is found only in another database, related to user accounts. Of course, it's up to you to decide whether you want to map user names to real names, or just use first names, fake names, or whatever you prefer.

In any case, all sensitive operations on data are logged with user names and their IP address. The log is cyclic, so that it won't grow indefinitely (usually it's limited to 10 MB); you can anyway control its options via `Serilog`-related settings in the configuration.

Data history instead never gets pruned, as it's part of the data themselves and can be used to recover from errors or other accidents, and track the evolution of data being entered.

### Database

The database-related settings essentially refer to connection strings. In `appsettings.json`, they are (here `__PRJ__` is a placeholder for your project's short name):

```json
{
  "ConnectionStrings": {
    "Default": "mongodb://localhost:27017/{0}",
    "Index": "Server=localhost;Database={0};Uid=root;Pwd=mysql;"
  },
  "DatabaseNames": {
    "Auth": "cadmus-__PRJ__auth",
    "Data": "cadmus-__PRJ__"
  },
  "Seed": {
    "ProfileSource": "%wwwroot%/seed-profile.json",
    "ItemCount": 100,
    "IndexDelay": 0
  }
}
```

>💥 Please remember that the containerized API layer requiring the database service connects to it within a Docker network. This means that the server name is just the name of the container, and that the port is the default port for its database service. Even when you eventually remap the database service port, e.g. to avoid a clash with the same service (PostgreSQL at 5432) on the host, nonetheless the containerized API will continue to connect to the default port 5432.

- `ConnectionStrings` 🚩 This object contains the connection string template to the MongoDB database (`Default`), and the index database (`Index`). You must ensure that the strings use the database service as named in the Docker stack (or, if you prefer, any other external database service, in this case removing it from the bottom of the Docker stack). Typically, they are as follows:

```yml
environment:
  - CONNECTIONSTRINGS__DEFAULT=mongodb://cadmus-mongo:27017/{0}
  - CONNECTIONSTRINGS__INDEX=Server=cadmus-index;port=3306;Database={0};Uid=root;Pwd=mysql
```

>💡 Note that here the hostnames (like `cadmus-mongo` or `cadmus-index`) are just the names of the database containerized services in the Docker compose script.

- `DatabaseNames`: the names of the authentication and data databases. These are not found in the connection string templates, where there is a placeholder `{0}` representing them, as the program needs to know both the full connection string and the database name alone.
- `Seed`:
  - `ProfileSource` (`SEED__PROFILESOURCE`): the source of the profile file. Typically you should not change this.
  - `ItemCount` (`SEED__ITEMCOUNT`): the count of the mock items you might want to seed into your database when creating it. On startup, the API service creates and seeds the database if not found.
  - `IndexDelay` (`SEED__INDEXDELAY`): an optional delay in milliseconds. When greater than 0, the API service waits the specified amount of time before starting to seed the database index. This may be required if the underlying database service takes some time to startup in your host.

#### External Databases

Should you prefer to use a database service outside the Docker compose stack, you must slightly modify the script.

The original script uses container names as data server names in the connection strings. In this scenario, you will rather replace these with your data service URI.

⚠️ As the containers in the compose script all refer to a Docker virtual network, you must use `extra_hosts` to address a service outside this network, as it is the case for your external data services. Thus, you should end up adding an `extra_hosts` section to each API service using a database, having an arbitrary name for your database service, and its mapping to an IP address. Once you have this, just replace the server name in the connection string with the arbitrary name chosen for your database service.

To sum up, the API section should look like this:

```yml
services:
  cadmus-api:
    image: ...
    environment:
      - CONNECTIONSTRINGS__DEFAULT=Server=somedb;...
    networks:
      - cadmus-network
    # added for mapping to IP outside the Docker virtual network
    extra_hosts:
      - "somedb:130.140.150.200"
```

In this API service the connection string points to the `somedb` server (`Server=somedb`), which is mapped to the IP 130.140.150.200 in its `extra_hosts` section.

### Other Services

- `Messaging`: this object is used to build automatic messages in the API service, mostly related to users management. The messaging object is like:

```json
"Messaging": {
  "AppName": "Cadmus",
  "ApiRootUrl": "https://cadmus.azurewebsites.net/api/",
  "AppRootUrl": "https://fusisoft.it/apps/cadmus/",
  "SupportEmail": "webmaster@fusisoft.net"
},
```

The settings here are used to fill some placeholders in the body of the messages being built. The provided values are mock values. Feel free to change these settings as you prefer, or just leave them as they are if you are not going to use email.

- `Mailer`: this object is used to configure an SMTP service to be consumed by the API service, when managing users. For instance, a user might want to recover his/her password, which here requires an email message to reset the forgotten password and setup a new one. Anyway, this is not a requirement, especially if you just manage a few users managed by an admin. The object is like this for a standard SMTP server:

```json
"Mailer": {
  "IsEnabled": false,
  "SenderEmail": "webmaster@fusisoft.net",
  "SenderName": "Cadmus",
  "TestRecipient": "you@somewhere.com",
  "Host": "YourSmtpHost",
  "Port": 587,
  "UseSsl": true,
  "UserName": "SmtpUserName",
  "Password": "SmtpPassword"
}
```

All the messaging services share these options:

- `IsEnabled`: a boolean used to enable or disable messaging.
- `SenderEmail`: the sender email address.
- `SenderName`: the sender human-friendly name.
- `TestRecipient`: the recipient email address to be used when sending a test email message. If not specified, no mail message will be sent.

Other options are specific for each service. Other mailing services can be used (e.g. SendGrid, MailJet, etc.); this requires just swapping the default service with another in the API startup code.

>💡 The messaging section is separated from the mailer section because building a text message is a process potentially shared among different messaging systems (email, SMS, etc.). Messages are built according to a set of HTML templates in the API layer (in folder `wwwroot/messages` of its source code).

## App Settings

When hosting on a server, the web application needs to know the address of the API service, which is no more found at `localhost`.

To avoid having to recompile the Angular app when changing this address, the app has an uncompiled `env.json` file including the base URIs accessed by it. So, you can just change the URIs inside this file and you are ready to go.

>💡 Also remember that your web app too will be relocated on a server, so it will no more be running at `localhost`. This means that you should ensure to include its new location under the [allowed CORS origins](#allowedorigins) of the API layer.

This is the default content of `env.js` (the version number may vary):

```js
(function (window) {
  window.__env = window.__env || {};
  window.__env.apiUrl = "http://localhost:29557/api/";
  window.__env.version = "0.0.10";
})(this);
```

As you can see, the only URI to change is `apiUrl`. Eventually, you can also change the version label, which gets displayed at the bottom of the page.

The typical procedure for changing the app image is:

1. download the app repository.
2. replace the URIs in `env.js`.
3. build the app (following the directions in `README.md`), and then create an _ad-hoc_ Docker image for your server environment.

Then, in your Docker compose script:

1. replace the _app image name_, so to use the newly created image.
2. change the exposed app _port number_ from 4200 to the standard HTTP / HTTPS port 80 / 443, like e.g.:

```yml
  cursus-app:
    image: yourrepo/yourimage:1.0.0-prod
    ports:
      - 80:80
```

>Alternatively, you may might "hack" the Docker image by directly changing the file's content in the container. Anyway, manually changing a Docker image is not a good practice.

▶️ next: [configuring HTTPS](https.md)