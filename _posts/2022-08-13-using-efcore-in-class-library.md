---
title: DRAFT Using Entity Framework Core in Separate Class Library
date: 2022-08-13 16:30:00 -500
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
