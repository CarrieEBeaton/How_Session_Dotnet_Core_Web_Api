# Creating Services in ASP.NET Core

In this session, we'll learn how to build and use the Services Pattern to abstract away the use of `HowDataContext` from the pages.

We will build two services, `ProductService` for managing Products, and `AzureStorageService` for saving images to Azure Storage.  All pages will interact with the `ProductService` which will consume the `AzureStorageService` for saving product images.

Concepts for this sesssion:

- Interface based programming

- [Dependency Injection](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-3.0#overview-of-dependency-injection)

- [Configuration Management](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-3.0)

- [Options Pattern](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options?view=aspnetcore-3.0)

## Visual Studio Project for Services

In the coming sections, we'll be creating new services and we need a good place to house these new resources.  Not only will we have an implementation class for each service, we'll also need to create interfaces for these implementations.

### Create Service Project

From within Visual Studio, create a new .NET Core Class Library project and call it
`HOW.AspNetCore.Services`.  This project will reference the `HOW.AspNetCore.Data` project and be the only API for managing product related activities.

### Create the following folders

- **Interfaces**

    We will store all Service interfaces within this folder

- **Domains**

    Domain specific Services will live here (e.g. `ProductService`)

- **Storage**

    Our Storage Service implementations will live here (e.g. `AzureStorageService`)

### Add Project Reference to Data Project

Make sure to add a reference to the `HOW.AspNetCore.Data` project.  This will provide access to the `HowDataContext` class using Dependency Injection.

## Dependency Injection Fundamentals

- [Reference Doc](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-3.0#lifetime-and-registration-options)

Before we create our new services, first we need a good example to help explain how Dependency Injection works within ASP.NET Core, such as  lifetime and registration options for types used within the Dependency Injection system.

In order to best demonstrate object lifetimes, we will create an `Operation` class that implements several interfaces used for the different types of registrations.  We will then register an object for each Interface based on a defined lifetime.  

We will also create an `OperationService` class that will ask for these registered interfaces using Dependency Injection.

Finally, we'll create a Razor Page that also askes for these registered interfaces, as well as the `OperationService` registered type, then display the associated Guid for each injected object to demonstrate lifetime.

### Create Interfaces

First we will create the interfaces that will be used for each lifetime registration within the DI System.  Each interface will implement a "root" interface since each of the implementations are used just as a mechanism for registration purposes.

```cs
public interface IOperation
{
    Guid OperationId { get; }
}

public interface IOperationTransient : IOperation
{
}

public interface IOperationScoped : IOperation
{
}

public interface IOperationSingleton : IOperation
{
}

public interface IOperationSingletonInstance : IOperation
{
}
```

### Create Implementation

Next, we'll create an implementation class that inherits from each of the interfaces.  This allows us to use the same implementation class for each lifetime registration.

```cs
public class Operation : IOperationTransient,
    IOperationScoped,
    IOperationSingleton,
    IOperationSingletonInstance
{
    public Operation() : this(Guid.NewGuid())
    {
    }

    public Operation(Guid id)
    {
        OperationId = id;
    }

    public Guid OperationId { get; private set; }
}
```

### Create Service Implementation

We then create a new `OperationService` class that asks for the registered types for each of the interface lifetimes.

```cs
public class OperationService
{
    public OperationService(
        IOperationTransient transientOperation,
        IOperationScoped scopedOperation,
        IOperationSingleton singletonOperation,
        IOperationSingletonInstance instanceOperation)
    {
        TransientOperation = transientOperation;
        ScopedOperation = scopedOperation;
        SingletonOperation = singletonOperation;
        SingletonInstanceOperation = instanceOperation;
    }

    public IOperationTransient TransientOperation { get; }
    public IOperationScoped ScopedOperation { get; }
    public IOperationSingleton SingletonOperation { get; }
    public IOperationSingletonInstance SingletonInstanceOperation { get; }
}
```

### Create Index page to display results

Finally, we create a new `Index.cshtml` Razor Page that askes for the registered types within the DI System.  The IndexModel class will have these registered types injected into the object to demonstrate that the registered types being injected into the PageModel are the same types as the ones injected into the `OperationService` instance.

The Razor page then displays the Guids for each lifetime object to show the matching/unmatching guid results based on lifetime.

```cs
public class IndexModel : PageModel
{
    public IndexModel(
        OperationService operationService,
        IOperationTransient transientOperation,
        IOperationScoped scopedOperation,
        IOperationSingleton singletonOperation,
        IOperationSingletonInstance singletonInstanceOperation)
    {
        OperationService = operationService;
        TransientOperation = transientOperation;
        ScopedOperation = scopedOperation;
        SingletonOperation = singletonOperation;
        SingletonInstanceOperation = singletonInstanceOperation;
    }

    public OperationService OperationService { get; }
    public IOperationTransient TransientOperation { get; }
    public IOperationScoped ScopedOperation { get; }
    public IOperationSingleton SingletonOperation { get; }
    public IOperationSingletonInstance SingletonInstanceOperation { get; }

    public void OnGet()
    {

    }
}
```

```html
@{
    ViewData["Title"] = "Dependency Injection Sample";
}

<h1>@ViewData["Title"]</h1>

<div class="row">
    <div class="panel panel-default">
        <div class="panel-heading">
            <h2 class="panel-title">Operations</h2>
        </div>
        <div class="panel-body">
            <h3>Page Model Operations</h3>
            <dl>
                <dt>Transient</dt>
                <dd>@Model.TransientOperation.OperationId</dd>
                <dt>Scoped</dt>
                <dd>@Model.ScopedOperation.OperationId</dd>
                <dt>Singleton</dt>
                <dd>@Model.SingletonOperation.OperationId</dd>
                <dt>Instance</dt>
                <dd>@Model.SingletonInstanceOperation.OperationId</dd>
            </dl>
            <h3>OperationService Operations</h3>
            <dl>
                <dt>Transient</dt>
                <dd>@Model.OperationService.TransientOperation.OperationId</dd>
                <dt>Scoped</dt>
                <dd>@Model.OperationService.ScopedOperation.OperationId</dd>
                <dt>Singleton</dt>
                <dd>@Model.OperationService.SingletonOperation.OperationId</dd>
                <dt>Instance</dt>
                <dd>@Model.OperationService.SingletonInstanceOperation.OperationId</dd>
            </dl>
        </div>
    </div>
</div>
```


Observe which of the OperationId values vary within a request and between requests:

- Transient objects are always different. The transient OperationId value for both the first and second client requests are different for both OperationService operations and across client requests. A new instance is provided to each service request and client request.
- Scoped objects are the same within a client request but different across client requests.
- Singleton objects are the same for every object and every request regardless of whether an Operation instance is provided in Startup.ConfigureServices.



- Talk about why the SingletonInstance and Singleton registrations have the same guid

- Talk about why the Scoped registrations have the same guids between client requests, but change after each request

- Talk about the transient registration and why the guid changes for each class it's injected into

