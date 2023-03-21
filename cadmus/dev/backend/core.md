---
layout: page
title: Creating Core Backend
subtitle: "Cadmus Backend Development"
---

- [Creating Solution](#creating-solution)
- [Adding Parts or Fragments](#adding-parts-or-fragments)
- [Adding Part or Fragment Seeders](#adding-part-or-fragment-seeders)
- [Adding Services](#adding-services)
- [Publishing Packages](#publishing-packages)

📌 Create a set of backend custom Cadmus models (parts/fragments) with their unit tests and mock data seeders.

1. **core**
2. [parts](parts.md)
3. [part seeders](part-seeders.md)
4. [fragments](fragments.md)
5. [fragment seeders](fragment-seeders.md)
6. [services](services.md)
7. [API](api.md)

The backend is a set of C# libraries, built with VS. This step is required only if you have new data models (parts or fragments) specific to your project.

The following procedure will:

- create a new Visual Studio solution for the backend components (business layer).
- add to it a library for parts/fragments (if required), another library for their mock data seeders, and a third library for API services. Also, each model-related library will have its unit tests library.

In what follows, `<PRJ>` represents the short name you chose for your project.

## Creating Solution

(1) launch VS and create a new _blank solution_ named `Cadmus<PRJ>`.

(2) add to this solution a _C# .NET 7 class library_, named `Cadmus.<PRJ>.Parts`. This will hold parts and fragments specific to your projects. Usually a single library is enough, but you are free to distribute components across several libraries should you need more granularity for their reuse. Once created, delete the empty `Class1.cs` file from it.

![adding new project](../../../img/cadmus/a01_add-new-project.png)

![adding new project](../../../img/cadmus/a02_add-new-project.png)

(3) add another _C# .NET 7 class library_ named `Cadmus.Seed.<PRJ>.Parts` to provide the mock data seeders for your components. This is not strictly a requirement, but it's suggested to let you play with the editor while building it. Once created, delete the empty `Class1.cs` file from it.

(4) add another _C# .NET 7 class library_ named `Cadmus.<PRJ>.Services` to provide some API services to plug into your API. Once created, delete the empty `Class1.cs` file from it.

(5) add a _XUnit Test Project_ named `Cadmus.<PRJ>.Parts.Test` to contain the tests for the `Cadmus.<PRJ>.Parts` library. Alternatively, any other unit test framework can be used; this just reflects my preferences, and is suggested as the test templates I provide use XUnit. Once created, delete the empty `UnitTest1.cs` class.

![adding new project](../../../img/cadmus/a03_add-new-xunit-project.png)

(6) add a _XUnit Test Project_ named `Cadmus.Seed.<PRJ>.Parts.Test` to contain the tests for the `Cadmus.Seed.<PRJ>.Parts` library. Alternatively, any other unit test framework can be used; this just reflects my preferences, and is suggested as the test templates I provide use XUnit. Once created, delete the empty `UnitTest1.cs` class.

Your solution should now look like this (here `<PRJ>` is `Pura`):

![adding new project](../../../img/cadmus/a04_solution.png)

(7) add references across projects in the solution, according to this schema:

- Cadmus.PRJ.Parts.Test depends on:
  - Cadmus.PRJ.Parts
  - Cadmus.Seed.PRJ.Parts
- Cadmus.PRJ.Services depends on:
  - Cadmus.PRJ.Parts
  - Cadmus.Seed.PRJ.Parts
- Cadmus.Seed.PRJ.Parts depends on:
  - Cadmus.PRJ.Parts
- Cadmus.Seed.PRJ.Parts.Test depends on:
  - Cadmus.Seed.PRJ.Parts

Adding a project reference can be done by right clicking the `Dependencies` node under the test project, selecting `Add Project Reference` from the popup menu, and checking the target project in the list which appears. Finally close the dialog with `OK`.

![adding new project](../../../img/cadmus/a05_project-deps.png)

![adding new project](../../../img/cadmus/a06_project-deps.png)

Alternatively, just edit the `csproj` XML file and add a line in an `ItemGroup` element like in this sample (replace the path with the correct one):

```xml
<ItemGroup>
  <ProjectReference Include="..\Cadmus.Pura.Parts\Cadmus.Pura.Parts.csproj" />
</ItemGroup>
```

## Adding Parts or Fragments

You can now add as many parts and fragments as required to the `Cadmus.<PRJ>.Parts` project.

(1) add a reference to the Cadmus core components to this project. This can be done in the VS UI, by adding a new NuGet package named `Cadmus.Core`, or by editing the `csproj` project XML file, and adding this line under an `<ItemGroup>` element (replace the version number with the latest available version):

```xml
<ItemGroup>
  <PackageReference Include="Cadmus.Core" Version="6.0.2" />
</ItemGroup>
```

Should you need existing components to build your own (e.g. to extend or integrate them), add their packages in the imports too.

(2) add a plain C# class for each part or fragment, representing its data model. Please refer to these pages for details:

- [adding parts](./parts.md)
- [adding fragments](./fragments.md)

## Adding Part or Fragment Seeders

For each part or fragment you should provide a corresponding mock data seeder to the `Cadmus.Seed.<PRJ>.Parts` project. This is extremely useful to let developers and users play with the editor.

(1) add a reference to the Cadmus core seed components to this project, as explained above. Also, a reference to the `Bogus` package is useful to leverage the power of this library rather than creating mock data from scratch. The package references are listed below (replace the version number with the latest available version):

```xml
<ItemGroup>
  <PackageReference Include="Bogus" Version="34.0.2" />
  <PackageReference Include="Cadmus.Core" Version="6.0.2" />
  <PackageReference Include="Cadmus.Seed" Version="6.0.2" />
</ItemGroup>
```

(2) add a plain C# class for each part or fragment seeder. Please refer to these pages for details:

- [adding parts](./backend-part.md)
- [adding part seeders](./backend-part-seeder.md)
- [adding fragments](./backend-fragment.md)

## Adding Services

Every Cadmus backend project using its own data models requires a couple of services:

- **repository provider**: this provides a Cadmus repository, used to edit the database, including all the models required for your project.
- **part seeder factory provider**: this provides a parts seeder factory, which provides the factory for generating part seeders. A part seeder is used to generate mock data to play with when developing the UI.

Please refer to [this page](./backend-core-svc.md) about adding those services.

## Publishing Packages

Once your parts, seeders, and services are ready, typically you should package them and publish the package so that it is available to yourself and to the community. Alternatively, you will just add a reference to the compiled library in your consumer projects.

To package the libraries for NuGet (you must have a free account for it), you should do this just once:

(1) not required, but suggested: ensure that you have added these to the PropertyGroup of each csproj to be packaged:

```xml
<IncludeSymbols>true</IncludeSymbols>
<SymbolPackageFormat>snupkg</SymbolPackageFormat>
```

This ensures that symbols are included when building the package.

(2) insert the package metadata by right clicking the project and picking `Properties`: author, license, version, etc.

Once you have setup your projects in this way, just publish them like in this batch:

```bat
@echo off
echo BUILD Cadmus Pura Packages
del .\Cadmus.Pura.Parts\bin\Debug\*.nupkg

cd .\Cadmus.Pura.Parts
dotnet pack -c Debug -p:IncludeSymbols=true -p:SymbolPackageFormat=snupkg
cd..

cd .\Cadmus.Pura.Services
dotnet pack -c Debug -p:IncludeSymbols=true -p:SymbolPackageFormat=snupkg
cd..

cd .\Cadmus.Seed.Pura.Parts
dotnet pack -c Debug -p:IncludeSymbols=true -p:SymbolPackageFormat=snupkg
cd..

pause
```

(in this sample I'm publishing the Debug versions for diagnostic purposes, but you should pick the Release version once you are comfortable with it).

🏠 [developer's home](../toc.md)

▶️ next: [parts](parts.md)
