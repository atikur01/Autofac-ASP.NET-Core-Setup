# Autofac Flexible Module System in ASP.NET Core

This repository demonstrates how to integrate Autofac, a powerful IoC (Inversion of Control) container, into an ASP.NET Core 9 web application using a flexible module system.

## Table of Contents
- [Installation](#installation)
- [Setup](#setup)
- [Creating and Using Autofac Modules](#creating-and-using-autofac-modules)
- [Implementing a Plugin Architecture](#implementing-a-plugin-architecture)
- [Best Practices](#best-practices)
- [References](#references)

## Installation

To install Autofac in your ASP.NET Core 9 project, run the following command:

```sh
 dotnet add package Autofac.Extensions.DependencyInjection
```

## Setup

Modify your `Program.cs` to use Autofac as the service provider factory:

```csharp
using Autofac;
using Autofac.Extensions.DependencyInjection;

var builder = WebApplication.CreateBuilder(args);

// Use Autofac as the service provider factory
builder.Host.UseServiceProviderFactory(new AutofacServiceProviderFactory());
builder.Host.ConfigureContainer<ContainerBuilder>(containerBuilder =>
{
    // Register Autofac modules
    containerBuilder.RegisterModule(new YourAutofacModule());
});

// Add services
builder.Services.AddControllersWithViews();

var app = builder.Build();

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

## Creating and Using Autofac Modules

Autofac modules allow better organization of service registrations. Here’s how to create a module:

```csharp
using Autofac;

public class YourAutofacModule : Module
{
    protected override void Load(ContainerBuilder builder)
    {
        // Register your services
        builder.RegisterType<YourService>().As<IYourService>().InstancePerLifetimeScope();
    }
}
```

Register the module inside `Program.cs`:

```csharp
builder.Host.ConfigureContainer<ContainerBuilder>(containerBuilder =>
{
    containerBuilder.RegisterModule(new YourAutofacModule());
});
```

## Implementing a Plugin Architecture

To implement a plugin system, first define discoverable plugins:

```csharp
[AttributeUsage(AttributeTargets.Class)]
public class DiscoverablePluginAttribute : Attribute
{
}

[DiscoverablePlugin]
public class SamplePlugin : IPlugin
{
    public void Execute()
    {
        // Plugin logic
    }
}
```

Then, dynamically load and register plugins:

```csharp
var builder = new ContainerBuilder();
var assemblies = AppDomain.CurrentDomain.GetAssemblies();
var pluginTypes = assemblies.SelectMany(a => a.GetTypes())
                            .Where(t => t.GetCustomAttributes<DiscoverablePluginAttribute>().Any());

foreach (var pluginType in pluginTypes)
{
    builder.RegisterType(pluginType).As<IPlugin>().SingleInstance();
}

var container = builder.Build();
```

## Best Practices

- **Organize Registrations**: Use modules to structure dependency registration.
- **Manage Lifetimes Properly**: Use appropriate instance scopes.
- **Avoid Service Locators**: Use constructor injection instead of manually resolving dependencies.

## References

- [Autofac Official Documentation](https://docs.autofac.org/en/stable/integration/aspnetcore.html)
- [Plugin Architecture in ASP.NET Core](https://www.codeproject.com/Articles/5365839/Plugin-Architecture-in-ASP-NET-Core-How-To-Master)
- [YouTube - Autofac in ASP.NET Core](https://www.youtube.com/watch?v=FJziGcvuzmo)

