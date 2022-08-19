---
title: Using Entity Framework Core in Separate Class Library
date: 2022-08-19 16:30:00 -500
categories: [software]
tags: [csharp,entityframework,.net]
---

# Using Entity Framework Core in Separate Class Library

In the post, let's walk through using Entity Framework Core in a separate class library using Blazor Server as the UI. 

I like to get straight to the point of this post so if you want to learn more about Entity Framework, you can check out Microsoft's documentation [here](https://docs.microsoft.com/en-us/ef/core/). 

You can check out the Github repo for this post [here]().

I'm going to use the command line tools to create the project and folder structure instead of using Visual Studio 2022 but if you are following along in VS, feel free to just use that to create the project.

Let's start by creating a Solution file that will hold both the Blazor Server project and the Class Library project:

I'm using PowerShell in Windows Terminal:
```terminal
dotnet new sln --output EfcoreApp
```

The *--output* option creates a sln file in a new folder created with the same name. So the result is that we have a folder called EfcoreApp with the file EfcoreApp.sln inside.

 Change directory into the *EfcoreApp* folder:
 ``` terminal
 cd EfcoreApp
 ```

 Now we can add our projects to our Solution. Let's add the Blazor Server project first:
 ```terminal
dotnet new blazorserver -o BlazorServerUI
 ```
 Now the class library:
 ```terminal
dotnet new classlib -n EfcoreLibrary
 ```
 Now we need to add the two projects to our solution file:
 ```terminal
dotnet sln add BlazorServerUI
dotnet sln add EfcoreLibrary
 ```

 Lastly, we need to add a reference from our Blazor Server project to the Class Library:

 ``` terminal
dotnet add BlazorServerUI/BlazorServerUI.csproj reference EfcoreLibrary/EfcoreLibrary.csproj
 ```

 Now we can start adding code. I'm using Visual Studio 2022. This application is going to be a Contacts app. Since this is a blog post about using Ef Core in a separate class library, we aren't going to build out the application, but we do need a model for our Data Context so it's helpful to have some context to what we are doing. 

 Let's start in the Class Library. The class library template comes with a file called class1.cs. Delete that file and add a new Class called DataContext.cs:

 ``` c#
 namespace EfcoreLibrary;
public class DataContext
{

}
 ```

 This class needs to inherit from DbContext which is part of the Microsoft.EntityFrameworkCore namespace. So we need to add the Nuget Package for Microsoft.EntityFrameworkCore.

 Back in the terminal in the EfcoreLibrary folder, run the following command to add the package:
 ``` terminal
dotnet add package Microsoft.EntityFrameworkCore
 ```

Now we can inherhit from DbContext and add the using statement for the namespace:
``` c#
using Microsoft.EntityFrameworkCore;

namespace EfcoreLibrary;
public class DataContext : DbContext
{

}
```

When our BlazorServer UI registers the DataContext object (more on that later), it needs a contructor with a DbContextOptions<DataContext<DataContext>> parameter:
``` c#
using Microsoft.EntityFrameworkCore;

namespace EfcoreLibrary;
public class DataContext : DbContext
{
    public DataContext(DbContextOptions<DataContext> options) : base(options)
    {
    }
}
```

Now let's add our Model. I'll create a folder in the EfcoreLibrary project called Models and add the Contact class there:
``` c#
namespace EfcoreLibrary.Models;
public class Contact
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string PhoneNumber { get; set; }
    public string Email { get; set; }
    public string Address { get; set; }
}
```

Now let's add this model to the Context class to register it as an entity:
``` c#
using EfcoreLibrary.Models;
using Microsoft.EntityFrameworkCore;

namespace EfcoreLibrary;
public class DataContext : DbContext
{
    public DataContext(DbContextOptions<DataContext> options) : base(options)
    {
    }

    public DbSet<Contact> Contacts { get; set; }
}
```
For the purposes of this post, I am not going to configure the entity and just let EF define the table. I do not reccomend this for real applications. You should always define/configure the entity to create the table exactly how you need it.

Now let's turn our attention to the UI project. We can start by adding the DbContext as a service in the Program.cs file.
In this demo, I'm going to use PostgreSQL for the database. We'll need to add a few packages from Nuget. Change directory into the BlazorServerUI folder and run the following command:
``` terminal
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
```

Now let's add the code to add the DbContext to our services in Program.cs:
```c#
using BlazorServerUI.Data;
using EfcoreLibrary;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddRazorPages();
builder.Services.AddServerSideBlazor();
builder.Services.AddSingleton<WeatherForecastService>();
builder.Services.AddDbContextFactory<DataContext>(options =>
{
    options.UseNpgsql(builder.Configuration.GetConnectionString("DefaultConnection"));
});

```

Here we are using the AddDbContextFactory instead of AddDbContext becuase we are using Blazor Server. For more information on why we are doing that, you can check out this [article](https://docs.microsoft.com/en-us/ef/core/dbcontext-configuration/#using-a-dbcontext-factory-eg-for-blazor).
We are also getting the connection string from appsettings.json. Here's what that looks like:
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost; Port=5432; Database=DemoEfcore; User Id=postgres; Password=fakepassword"
  }
}
```

Now we can look at adding out first migration, but before we do that, we need to add a few more NuGet Packages.

From the BlazorServerUI Directory add the following package:
```terminal
dotnet add package Microsoft.EntityFrameworkCore.Design
```

Now we can change directory back to EfcoreLibrary and run the following command to add our first Migration:
```terminal
dotnet ef migrations add initial --startup-project ..\BlazorServerUI\BlazorServerUI.csproj
```

Output:

```terminal
Build started...
Build succeeded.
info: Microsoft.EntityFrameworkCore.Infrastructure[10403]
      Entity Framework Core 6.0.8 initialized 'DataContext' using provider 'Npgsql.EntityFrameworkCore.PostgreSQL:6.0.6+6fa8f3c27a7c241a66e72a6c09e0b252509215d0' with options: None
Done. To undo this action, use 'ef migrations remove'
```

This created the migration in the Libary Project in a folder named Migrations. However, if you go look at the migration, you'll see that there are a lot of errors in code that was generated and you won't be able to apply the migration. Trying to apply the migration will result in the following:
``` terminal
 dotnet ef database update --startup-project ..\BlazorServerUI\BlazorServerUI.csproj
Build started...
Build failed. Use dotnet build to see the errors.
```
This is because we are still missing a few more Nuget Packages.

Let's add the following the Library project:
```terminal
dotnet add package Microsoft.EntityFrameworkCore.Relational
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
```

Now all the build errors have been resolved, let's update the database:
``` terminal
dotnet ef database update --startup-project ..\BlazorServerUI\BlazorServerUI.csproj
```

Now we successfully separated our DataContext from our UI. This allows us to share the DataContext/EF Core with other projects in the same solution in a clean manner. 