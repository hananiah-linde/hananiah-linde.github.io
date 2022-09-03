---
title: Using Entity Framework Core in Separate Class Library Using SQL Server
date: 2022-08-13 16:30:00 -500
categories: [software]
tags: [csharp,entityframework,.net]
---

# Using Entity Framework Core in Separate Class Library Using SQL Server

In the post, let's walk through using Entity Framework Core in a separate class library using SQL Server and an ASP.NET Core WebAPI in .NET 6. 

I like to get straight to the point of this post so if you want to learn more about Entity Framework, you can check out Microsoft's documentation [here](https://docs.microsoft.com/en-us/ef/core/). 

You can check out the Github repo for this post [here](https://github.com/hananiah-linde/EfcoreSqlServerApp).

I'm going to use the command line tools to create the project and folder structure instead of using Visual Studio 2022 but if you are following along in VS, feel free to just use that to create the project.

Let's start by creating a Solution file that will hold both the WebAPI and the Class Library projects:

I'm using PowerShell in Windows Terminal:
```terminal
dotnet new sln --output EfcoreSqlServerApp
```

The *--output* option creates a sln file in a new folder created with the same name. So the result is that we have a folder called EfcoreApp with the file EfcoreApp.sln inside.

 Change directory into the *EfcoreApp* folder:
 ``` terminal
 cd EfcoreSqlServerApp
 ```

 Now we can add our projects to our Solution. Let's add the WebAPI project first:
 ```terminal
dotnet new webapi -o API
 ```
 Now the class library:
 ```terminal
dotnet new classlib -n Database
 ```
 Now we need to add the two projects to our solution file:
 ```terminal
dotnet sln add API
dotnet sln add Database
 ```

 Lastly, we need to add a reference from our API project to the Class Library:

 ``` terminal
dotnet add API/API.csproj reference Database/Database.csproj
 ```

 Now we can start adding code. I'm using Visual Studio 2022. In this app, we are going to setup ASP.NET Core Identity for the API. This provides us a number of Tables that allow for managing Users in the database. We will have one model that extends the IdentityUser class that is part of ASP.NET Core Identity.

 Let's start in the Class Library. The class library template comes with a file called class1.cs. Delete that file and add a new Class called DatabaseContext.cs:

 ``` c#
 namespace Database;
public class DataContext
{

}
 ```

 In order to utilize ASP.NET Core Identity, this class needs to inherit from IdentityDbContext which is part of the Microsoft.AspNetCore.Identity.EntityFrameworkCore namespace. So we need to add the Nuget Package for this.

 Back in the terminal in the Database folder, run the following command to add the package:
 ``` terminal
dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
 ```

Now we can inherhit from IdentityDbContext and use IdentityUser as the generic type.
Add the following using statements to include the namespaces for those classes:
``` c#
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Identity.EntityFrameworkCore;

namespace Database;
public class DatabaseContext : IdentityDbContext<IdentityUser>
{

}
```

When our API registers the DatabaseContext object (more on that later), it needs a contructor with a DbContextOptions<DatabaseContext<DatabaseContext>> parameter:
``` c#
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Identity.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore;

namespace Database;
public class DatabaseContext : IdentityDbContext<IdentityUser>
{
    public DatabaseContext(DbContextOptions options) : base(options)
    {
    }
}

```
Visual Studio 2022 automatically adding the using statement for Microsoft.EntityFrameworkCore.

Now let's add our Model. I'll create a folder in the Database project called Entities and add a class named ApiUser.cs there:
``` c#
using Microsoft.AspNetCore.Identity;

namespace Database.Entities;
public class ApiUser : IdentityUser
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
}
```
Here's our model. We've extended the IdentityUser class by adding two properties FirstName and LastName which aren't included in the base class. 

Now because our DatabaseContext class is inheriting from IdentityDbContext<IdentityUser<IdentityUser>>, we don't need to add a DbSet<> to register our model like you normal do.

So our Database Project is all set for now.

Now let's turn our attention to the API project.
There's two things we need to do.
1. Add EntityFrameworkCore
2. Add ASP.NET Core Identity

I prefer keeping my Program.cs file lean so I'm going to add a new folder that will contain extension methods for adding these services.

Add a new folder called Startup and create a class called ServiceExtensions.cs.

Add the following code:
``` c#
using Database;
using Microsoft.AspNetCore.Identity;

namespace API.Startup;
public static class ServiceExtensions
{
    public static void ConfigureEntityFrameworkCore(this IServiceCollection services, IConfiguration configuration)
    {
        var connectionString = configuration.GetConnectionString("Default");
        services.AddDbContext<DatabaseContext>(options =>
            options.UseSqlServer(connectionString));
    }

    public static void ConfigureIdentityCore(this IServiceCollection services)
    {
        var builder = services.AddIdentityCore<IdentityUser>(q => q.User.RequireUniqueEmail = true);

        builder = new IdentityBuilder(builder.UserType, typeof(IdentityRole), services);
        builder.AddEntityFrameworkStores<DatabaseContext>().AddDefaultTokenProviders();
    }
}
```

Add the following Nuget Package in the API directory for the missing namespace:
``` terminal
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
```

The ConfigureEntityFrameworkCore extension method will register the DatabaseContext class as a service. We need to supply the connection string to the database. In this demo, I've put the connection string in appsettings.json but this is not the best practice for real applications.

Add the following to appsettings.json:

``` json
{
  "ConnectionStrings": {
    "Default": "Server=(localdb)\\mssqllocaldb;Database=EfCoreAppSqlServer;Trusted_Connection=True;MultipleActiveResultSets=true"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

The ConfigureIdentityCore extension method adds ASP.NET Core Identity.

In Program.cs we need to call those extension methods in addition to the UseAuthentication():
```c#
using API.Startup;

var builder = WebApplication.CreateBuilder(args);

builder.Services.ConfigureEntityFrameworkCore(builder.Configuration);
builder.Services.AddAuthentication();
builder.Services.ConfigureIdentityCore();
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

app.UseSwagger();
app.UseSwaggerUI();
app.UseHttpsRedirection();
app.UseAuthorization();
app.UseAuthentication();
app.MapControllers();

app.Run();
```

Now we can add out first migration. We need to add the following package to our API project.

From the API Directory add the following package:
```terminal
dotnet add package Microsoft.EntityFrameworkCore.Design
```

Now we can change directory back to Database and run the following command to add our first Migration:
```terminal
dotnet ef migrations add initial --startup-project ..\API\API.csproj
```

Output:

```terminal
Build started...
Build succeeded.
info: Microsoft.EntityFrameworkCore.Infrastructure[10403]
      Entity Framework Core 6.0.8 initialized 'DatabaseContext' using provider 'Microsoft.EntityFrameworkCore.SqlServer:6.0.8' with options: None
Done. To undo this action, use 'ef migrations remove'
```

This created the migration in the Database Project in a folder named Migrations. However, this code is generated by the EF Core tools and does not include all the required packages for the code that was generated.
If you look at the DatabaseContextModelSnapshot.cs file, you'll see that there are some errors. These can be resolved by adding the following packages to the Database project:

```terminal
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
```

Now all the errors have been resolved, let's update the database:
``` terminal
dotnet ef database update --startup-project ..\API\API.csproj
```

Now we have successfully separated our DatabaseContext from our UI. This allows us to share the DataContext/EF Core with other projects in the same solution in a clean manner. 