---
layout: page
title: Cadmus Development
subtitle: Summary
---

- [Guide](#guide)
- [Development Environment](#development-environment)
  - [Angular CLI](#angular-cli)
  - [MongoDB](#mongodb)
  - [MySql](#mysql)

## Guide

To create a Cadmus editor for your project, you need a backend and a frontend. For the most part, you will end up just customizing the provided code templates, so you do not need advanced programming skills to setup an editor. Of course, a minimal familiarity with the languages and technologies involved is required.

You start with creating backend core libraries, eventually adding your own parts and fragments, with their seeders and tests. Then, you add a couple of services which put all the pieces together, and are consumed by the API layer. Once done, you create the backend API by just assembling the pieces you created with those coming from Cadmus infrastructure.

Then, you create an Angular workspace for your frontend app, eventually adding the editors corresponding to your own parts and fragments. Apart from this, the app is built by just assembling pieces from the Cadmus infrastructure.

1. [backend](backend.md)
   1. [core data models](backend-core.md) (optional)
      1. [parts](backend-part.md)
      2. [part seeders](backend-part-seeder.md)
      3. [fragments](backend-fragment.md)
      4. [fragment seeders](backend-fragment-seeder.md)
      5. [services](backend-core-svc.md)
   2. [backend services](backend-core-svc.md)
   3. [backend API](backend-api.md)
2. [frontend](frontend.md)
   1. [parts](frontend-part.md) (optional)
   2. fragments (optional)

## Development Environment

When developing on a single machine, I setup my environment as follows (if you are starting from scratch, it's easier if you follow this order - at any rate, Angular requires NodeJS):

1. [Docker](../docker-setup.md).
2. a Docker container for MongoDB, named `mongo`.
3. a Docker container for MySql, named `mysql`.
4. [Visual Studio Community Edition](https://visualstudio.microsoft.com/vs/community/) for the backend.
5. [NodeJS](https://nodejs.org/en/download/): use the LTS version.
6. [Angular CLI](https://angular.io/cli): this requires NodeJS. Both NodeJS and Angular are used for the frontend.
7. [VSCode](https://code.visualstudio.com/download) to be used as the frontend editor.

Note that you can either install MySql and MongoDB the usual way, or use a Docker-based service. I prefer the latter because it gives you more freedom, and does not pollute my workstation with stuff I have then to maintain; yet, you can also go with the former solution, which might be easier. In this case, skip (1), replace (2-3) with a traditional setup, and you are ready to go without requiring Docker.

>If you are installing your databases rather than using Docker containers for them, just accept all the defaults during setup. For MySql you also need to setup a root user; I usually add an internal root user named `root` with password=`mysql`; these are the defaults in Cadmus. Of course you can change them (in the connection strings); but it's easier if you stick to this convention. This won't hurt anyway, because you will use MySql only in your local machine for development.

### Angular CLI

To install Angular CLI globally:

```bash
npm install -g @angular/cli@latest
```

You can check that the setup worked by just invoking `ng version`.

Note that in Windows you will need to allow Powershell script execution for this. Start Windows PowerShell with "Run as Administrator", and then enable running unsigned scripts like:

```ps1
Set-ExecutionPolicy remotesigned
```

This will allow running unsigned scripts that you write on your local computer and signed scripts from Internet.

To update Angular when required, you can use this command:

```bash
npm update -g @angular/cli
```

### MongoDB

**Windows**: to run a mongo container persisting data in the host via a volume in `c:\users\<USERNAME>\data\mongo`:

```bash
docker run --name mongo -d -p 27017:27017 --volume c:/users/dfusi/dockerVolMongo/db:/data/db mongo --noauth
```

You can now connect to `localhost`.

Some database clients:

- [Studio3T](https://studio3t.com/)
- [MongoDB Compass](https://www.mongodb.com/products/compass)

### MySql

**Windows**: to run a mysql container storing data in host through a volume in `c:\data\mysql`:

```bash
docker run --name mysql -d -e MYSQL_ROOT_PASSWORD=mysql -v c:/data/mysql:/var/lib/mysql -p 3306:3306 mysql --default-authentication-plugin=mysql_native_password
```

Once the container has been created, you can stop and restart it using `docker container stop mysql` and `docker container start mysql`. You can inspect it with `docker container inspect mysql`.

You can now connect to 127.0.0.1 with user=`root` and password=`mysql`.

Some database clients:

- [DBeaver](https://dbeaver.io/download/)
- [MySql Workbench](https://dev.mysql.com/downloads/workbench/)
