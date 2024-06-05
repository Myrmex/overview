---
layout: page
title: Creating Core Backend
subtitle: "Cadmus Backend Development"
---

- [Creating Solution](#creating-solution)
- [Adding Parts/Fragments](#adding-partsfragments)
  - [Adding Parts or Fragments](#adding-parts-or-fragments)
  - [Adding Part or Fragment Seeders](#adding-part-or-fragment-seeders)
- [Adding Services](#adding-services)
- [Publishing Packages](#publishing-packages)

📌 Create a set of backend custom Cadmus models (parts/fragments) with their unit tests, mock data seeders and services.

1. **core**
2. [parts](parts.md)
3. [part seeders](part-seeders.md)
4. [fragments](fragments.md)
5. [fragment seeders](fragment-seeders.md)
6. [services](services.md)
7. [API](api.md)

The backend is a set of C# libraries, built with VS.

The following procedure will:

- create a new Visual Studio solution for the backend components (business layer).
- add to it these libraries:
  - library for parts/fragments (if required), plus a test library.
  - library for parts/fragments mock data seeders, plus a test library.
  - library for API services.

In what follows, `PRJ` represents the short name you chose for your project.

## Creating Solution

(1) launch VS and create a new _blank solution_ named `CadmusPRJ`. Alternatively, use this command (replace `PRJ` with your project's name):

```bash
dotnet new sln -n CadmusPRJ
```

## Adding Parts/Fragments

If you have project-specific parts or fragments, follow the steps in this section. Otherwise, just go to the [next section](#adding-services).

(2) add to this solution a _C# .NET class library_, named `Cadmus.PRJ.Parts`:

```bash
dotnet new classlib -n Cadmus.PRJ.Parts
dotnet sln CadmusPRJ.sln add Cadmus.PRJ.Parts/Cadmus.PRJ.Parts.csproj
```

 This will hold parts and fragments specific to your projects. Usually a single library is enough, but you are free to distribute components across several libraries should you need more granularity for their reuse. Once created, delete the empty `Class1.cs` file from it.

![adding new project](../../../img/cadmus/a01_add-new-project.png)

![adding new project](../../../img/cadmus/a02_add-new-project.png)

(3) add another _C# .NET 8 class library_ named `Cadmus.Seed.PRJ.Parts` to provide the mock data seeders for your components. This is not strictly a requirement, but it's suggested to let you play with the editor while building it. Once created, delete the empty `Class1.cs` file from it.

```bash
dotnet new classlib -n Cadmus.Seed.PRJ.Parts
dotnet sln CadmusPRJ.sln add Cadmus.Seed.PRJ.Parts/Cadmus.Seed.PRJ.Parts.csproj
```

(4) add another _C# .NET 8 class library_ named `Cadmus.PRJ.Services` to provide some API services to plug into your API. Once created, delete the empty `Class1.cs` file from it.

```bash
dotnet new classlib -n Cadmus.PRJ.Services
dotnet sln CadmusPRJ.sln add Cadmus.PRJ.Services/Cadmus.PRJ.Services.csproj
```

(5) add a _XUnit Test Project_ named `Cadmus.PRJ.Parts.Test` to contain the tests for the `Cadmus.PRJ.Parts` library. Alternatively, any other unit test framework can be used; this just reflects my preferences, and is suggested as the test templates I provide use XUnit. Once created, delete the empty `UnitTest1.cs` class.

```bash
dotnet new xunit -n Cadmus.PRJ.Parts.Test
dotnet sln CadmusPRJ.sln add Cadmus.PRJ.Parts.Test/Cadmus.PRJ.Parts.Test.csproj
```

![adding new project](../../../img/cadmus/a03_add-new-xunit-project.png)

(6) add a _XUnit Test Project_ named `Cadmus.Seed.PRJ.Parts.Test` to contain the tests for the `Cadmus.Seed.PRJ.Parts` library. Alternatively, any other unit test framework can be used; this just reflects my preferences, and is suggested as the test templates I provide use XUnit. Once created, delete the empty `UnitTest1.cs` class.

```bash
dotnet new xunit -n Cadmus.Seed.PRJ.Parts.Test
dotnet sln CadmusPRJ.sln add Cadmus.Seed.PRJ.Parts.Test/Cadmus.Seed.PRJ.Parts.Test.csproj
```

Your solution should now look like this (here `PRJ` is `Pura`):

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

### Adding Parts or Fragments

You can now add as many parts and fragments as required to the `Cadmus.PRJ.Parts` project.

(1) add a reference to the Cadmus core components to this project. This can be done in the VS UI, by adding a new NuGet package named `Cadmus.Core`; from the command line, as shown below; or by editing the `csproj` project XML file, and adding this line under an `<ItemGroup>` element (replace the version number with the latest available version):

- command line:

```bash
cd Cadmus.PRJ.Parts
dotnet add package Cadmus.Core
```

- CS project file:

```xml
<ItemGroup>
  <PackageReference Include="Cadmus.Core" Version="7.0.3" />
</ItemGroup>
```

Should you need existing components to build your own (e.g. to extend or integrate them), add their packages in the imports too.

(2) add a plain C# class for each part or fragment, representing its data model. Please refer to these pages for details:

- [adding parts](./parts.md)
- [adding fragments](./fragments.md)

💡 If you have several parts/fragments using the `StandardDataPinTextFilter` to filter pin string values, you have better use a helper class providing a singleton for it, like in the following code:

```cs
static internal class DataPinHelper
{
    private static StandardDataPinTextFilter? _filter;

    /// <summary>
    /// Gets the default filter used for pins.
    /// This improves performance, as we can share this filter
    /// among several parts.
    /// </summary>
    static public IDataPinTextFilter DefaultFilter
    {
        get { return _filter ??= new StandardDataPinTextFilter(); }
    }
}
```

### Adding Part or Fragment Seeders

For each part or fragment you should provide a corresponding mock data seeder to the `Cadmus.Seed.PRJ.Parts` project. This is extremely useful to let developers and users play with the editor.

(1) add a reference to the Cadmus core seed components to this project, as explained above. Also, a reference to the `Bogus` package is useful to leverage the power of this library rather than creating mock data from scratch. The package references are listed below (replace the version number with the latest available version):

- command line:

```bash
cd Cadmus.Seed.PRJ.Parts
dotnet add package Bogus
dotnet add package Cadmus.Core
dotnet add package Cadmus.Seed
```

- CS project file:

```xml
<ItemGroup>
  <PackageReference Include="Bogus" Version="35.5.1" />
  <PackageReference Include="Cadmus.Core" Version="7.0.3" />
  <PackageReference Include="Cadmus.Seed" Version="7.0.2" />
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

Please refer to [this page](services.md) about adding those services.

## Publishing Packages

Once your parts, seeders, and services are ready, typically you should package them and publish the package so that it is available to yourself and to the community. Alternatively, you will just add a reference to the compiled library in your consumer projects.

To package the libraries for NuGet (you must have a free account for it), you should do this just once:

(1) not required, but suggested: ensure that you have added these to the `PropertyGroup` of each csproj to be packaged:

```xml
<IncludeSymbols>true</IncludeSymbols>
<SymbolPackageFormat>snupkg</SymbolPackageFormat>
```

This ensures that symbols are included when building the package.

(2) insert the package metadata by right clicking the project and picking `Properties`: author, license, version, etc.

Once you have setup your projects in this way, just publish them like in this batch:

```bat
@echo off
echo BUILD Cadmus PRJ Packages
del .\Cadmus.PRJ.Parts\bin\Debug\*.nupkg

cd .\Cadmus.PRJ.Parts
dotnet pack -c Debug -p:IncludeSymbols=true -p:SymbolPackageFormat=snupkg
cd..

cd .\Cadmus.PRJ.Services
dotnet pack -c Debug -p:IncludeSymbols=true -p:SymbolPackageFormat=snupkg
cd..

cd .\Cadmus.Seed.PRJ.Parts
dotnet pack -c Debug -p:IncludeSymbols=true -p:SymbolPackageFormat=snupkg
cd..

pause
```

(in this sample I'm publishing the Debug versions for diagnostic purposes, but you should pick the Release version once you are comfortable with it).

🏠 [developer's home](../toc.md)

▶️ next: [parts](parts.md)
