# IndexAttribute for EntityFramework Core  
[![Build status](https://ci.appveyor.com/api/projects/status/dv0et0b80da5mwys?svg=true)](https://ci.appveyor.com/project/jsakamoto/entityframeworkcore-indexattribute) [![NuGet Package](https://img.shields.io/nuget/v/Toolbelt.EntityFrameworkCore.IndexAttribute.svg)](https://www.nuget.org/packages/Toolbelt.EntityFrameworkCore.IndexAttribute/)

## What's this?

Revival of `[Index]` attribute for EF Core. (with extension for model building.)


### Attention

EF Core team said:

> _"We didn't bring this (= IndexAttribute) over from EF6.x **because it had a lot of issues**"_  
> (https://github.com/aspnet/EntityFrameworkCore/issues/4050)

Therefore, you should consider well before use this package.

## How to use?

1. Add [`Toolbelt.EntityFrameworkCore.IndexAttribute`](https://www.nuget.org/packages/Toolbelt.EntityFrameworkCore.IndexAttribute/) package to your project.

```shell
> dotnet add package Toolbelt.EntityFrameworkCore.IndexAttribute
```

### Supported Versions

EF Core version | This package version
----------------|-------------------------
v.3.1, v.5.0 Prev1  | v.3.1, v.3.2
v.3.0           | v.3.0, v.3.1
v.2.0, 2.1, 2.2 | v.2.0.x

If you want to use `IsClustered=true` and/or `Includes` index features on a SQL Server, you have to add [`Toolbelt.EntityFrameworkCore.IndexAttribute.SqlServer`](https://www.nuget.org/packages/Toolbelt.EntityFrameworkCore.IndexAttribute.SqlServer/) package to your project, instead.

```shell
> dotnet add package Toolbelt.EntityFrameworkCore.IndexAttribute.SqlServer
```

2. Annotate your model with `[Index]` attribute that lives in `Toolbelt.ComponentModel.DataAnnotations.Schema` namespace.

```csharp
using Toolbelt.ComponentModel.DataAnnotations.Schema;

public class Person
{
    public int Id { get; set; }

    [Index] // <- Here!
    public string Name { get; set; }
}
```

3. **[Important]** Override `OnModelCreating()` method of your DbContext class, and call `BuildIndexesFromAnnotations()` extension method which lives in `Toolbelt.ComponentModel.DataAnnotations` namespace.

```csharp
using Microsoft.EntityFrameworkCore;
using Toolbelt.ComponentModel.DataAnnotations;

public class MyDbContext : DbContext
{
    ...
    // Override "OnModelCreating", ...
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        // .. and invoke "BuildIndexesFromAnnotations"!
        modelBuilder.BuildIndexesFromAnnotations();
    }
}
```

If you use SQL Server and `IsClustered=true` and/or `Includes = new[]{"Foo", "Bar"}` features, you need to call `BuildIndexesFromAnnotationsForSqlServer()` extension method instead of `BuildIndexesFromAnnotations()` extension method.

```csharp
    ...
    // Override "OnModelCreating", ...
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        // Invoke "BuildIndexesFromAnnotationsForSqlServer"
        // instead of "BuildIndexesFromAnnotations".
        modelBuilder.BuildIndexesFromAnnotationsForSqlServer();
    }
```

That's all!

`BuildIndexesFromAnnotations()` (or, `BuildIndexesFromAnnotationsForSqlServer()`) extension method scans the DbContext with .NET Reflection technology, and detects `[Index]` attributes, then build models related to indexing.

After doing that, the database which created by EF Core, contains indexes that are specified by `[Index]` attributes.

## Appendix A - Suppress "NotSupportedException"

You will run into "NotSupportedException" when you call `BuildIndexesFromAnnotations()` with the model which is annotated with the `[Index]` attribute that's "IsClustered" property is true, or "Includes" property is not empty.

If you have to call `BuildIndexesFromAnnotations()` in this case (for example, share the model for different Database products), you can suppress this behavior with configuration, like this.

```csharp
  ...
  protected override void OnModelCreating(ModelBuilder modelBuilder)
  {
    base.OnModelCreating(modelBuilder);

    // Suppress "NotSupportedException" for "IsClustered" and "Includes" feature.
    modelBuilder.BuildIndexesFromAnnotations(options => {
      options.SuppressNotSupportedException.IsClustered = true;
      options.SuppressNotSupportedException.Includes = true;
    });
  }
}
```

## Appendix B -  Notice for using "IsClustered=true"

If you annotate the model with "IsClustered=true" index simply like this,

```csharp
public class Employee {
  public int Id { get; set; }

  [Index(IsClustered = true)]
  public string EmployeeCode { get; set; }
}
```

You will run into 'System.Data.SqlClient.SqlException' like this.

```
System.Data.SqlClient.SqlException :
Cannot create more than one clustered index on table '(table name)'.
Drop the existing clustered index '(index name)' before creating another.
```

In this case, you need to annotate a primary key property with `[PrimaryKey(IsClustered = false)]` attribute explicitly  for suppress auto generated primary key to be clustered index.

```csharp
public class Employee {
  [PrimaryKey(IsClustered = false)] // <- Add this line!
  public int Id { get; set; }

  [Index(IsClustered = true)]
  public string EmployeeCode { get; set; }
}
```

## Appendix C -  If you want to use only "IndexAttribute" without any dependencies...

If you want to use only "IndexAttribute" class without any dependencies, you can use [Toolbelt.EntityFrameworkCore.IndexAttribute.Attribute](https://www.nuget.org/packages/Toolbelt.EntityFrameworkCore.IndexAttribute.Attribute) NuGet package.


## For More Detail...

This library is designed to have the same syntax as EF 6.x `[Index]` annotation.

Please visit document site of EF 6.x and `[Index]` attribute for EF 6.x.

- [MSDN Document - Entity Framework Code First Data Annotations](https://msdn.microsoft.com/en-us/library/jj591583%28v=vs.113%29.aspx?f=255&MSPPError=-2147217396)
- [MSDN Document - IndexAttribute class](https://msdn.microsoft.com/library/system.componentmodel.dataannotations.schema.indexattribute(v=vs.113).aspx)

## Limitations

`[Index]` attribute with `IsClustered=true` can apply only not owned entity types.

## Release Notes

- [Toolbelt.EntityFrameworkCore.IndexAttribute.Attibute - Release Notes](https://github.com/jsakamoto/EntityFrameworkCore.IndexAttribute/blob/master/EFCore.IndexAttribute.Attribute/RELEASE-NOTES.txt)
- [Toolbelt.EntityFrameworkCore.IndexAttribute - Release Notes](https://github.com/jsakamoto/EntityFrameworkCore.IndexAttribute/blob/master/EFCore.IndexAttribute/RELEASE-NOTES.txt)
- [Toolbelt.EntityFrameworkCore.IndexAttribute.SqlServer - Release Notes](https://github.com/jsakamoto/EntityFrameworkCore.IndexAttribute/blob/master/EFCore.IndexAttribute.SqlServer/RELEASE-NOTES.txt)

## License

[MIT License](https://github.com/jsakamoto/EntityFrameworkCore.IndexAttribute/blob/master/LICENSE)

