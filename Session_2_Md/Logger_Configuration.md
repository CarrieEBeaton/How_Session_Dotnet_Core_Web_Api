[ApplicationInsights](https://docs.microsoft.com/en-us/azure/azure-monitor/app/ilogger)


# Logging in .NET Core and ASP.NET Core

.NET Core supports a logging API that works with a variety of built-in and third-party logging providers. This article shows how to use the logging API with built-in providers.

Most of the code examples shown in this article are from ASP.NET Core apps. The logging-specific parts of these code snippets apply to any .NET Core app that uses the [Generic Host](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/host/generic-host?view=aspnetcore-3.1). For an example of how to use the Generic Host in a non-web console app, see the Program.cs file of the Background Tasks sample app (Background tasks with hosted services in ASP.NET Core).

Logging code for apps without Generic Host differs in the way [providers are added](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging/?view=aspnetcore-3.1#add-providers) and [loggers are created](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging/?view=aspnetcore-3.1#create-logs). Non-host code examples are shown in those sections of the article.

[View or download sample code (how to download)](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/logging/index/samples)

## Add providers
A logging provider displays or stores logs. For example, the Console provider displays logs on the console, and the Azure Application Insights provider stores them in Azure Application Insights. Logs can be sent to multiple destinations by adding multiple providers.

To add a provider in an app that uses Generic Host, call the provider's Add{provider name} extension method in _Program.cs_:

C#  | Copy
----|-----

    public static IHostBuilder CreateHostBuilder(string[] args) =>
     Host.CreateDefaultBuilder(args)
            .ConfigureLogging(logging =>
        {
            logging.ClearProviders();
            logging.AddConsole();
        })
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });
In a non-host console app, call the provider's Add{provider name} extension method while creating a LoggerFactory:

C#  | Copy
----|-----
    var loggerFactory = LoggerFactory.Create(builder =>
    {
      builder
          .AddFilter("Microsoft", LogLevel.Warning)
          .AddFilter("System", LogLevel.Warning)
          .AddFilter("LoggingConsoleApp.Program", LogLevel.Debug)
          .AddConsole()
           .AddEventLog();
    });
    ILogger logger = loggerFactory.CreateLogger<Program>();
    logger.LogInformation("Example log message");

`LoggerFactory` and `AddConsole` require a using statement for `Microsoft.Extensions.Logging`.

The default ASP.NET Core project templates call CreateDefaultBuilder, which adds the following logging providers:

- Console
- Debug
- EventSource
- EventLog (only when running on Windows)

You can replace the default providers with your own choices. Call - ClearProviders, and add the providers you want.

C#  | Copy
----|-----
    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureLogging(logging =>
            {
                logging.ClearProviders();
               logging.AddConsole();
            })
            .ConfigureWebHostDefaults(webBuilder =>
           {
               webBuilder.UseStartup<Startup>();
           });
Learn more about built-in logging providers and third-party logging providers later in the article.

## Create logs
To create logs, use an [`ILogger<TCategoryName>`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.logging.ilogger-1) object. In a web app or hosted service, get an ILogger from dependency injection (DI). In non-host console apps, use the LoggerFactory to create an ILogger.

The following ASP.NET Core example creates a logger with TodoApiSample.Pages.AboutModel as the category. The log category is a string that is associated with each log. The ILogger<T> instance provided by DI creates logs that have the fully qualified name of type T as the category.

C#  | Copy
----|-----
public class AboutModel : PageModel
{
    private readonly ILogger _logger;

    public AboutModel(ILogger<AboutModel> logger)
    {
        _logger = logger;
    }
The following non-host console app example creates a logger with LoggingConsoleApp.Program as the category.

C#

Copy
var loggerFactory = LoggerFactory.Create(builder =>
{
    builder
        .AddFilter("Microsoft", LogLevel.Warning)
        .AddFilter("System", LogLevel.Warning)
        .AddFilter("LoggingConsoleApp.Program", LogLevel.Debug)
        .AddConsole()
        .AddEventLog();
});
ILogger logger = loggerFactory.CreateLogger<Program>();
logger.LogInformation("Example log message");
In the following ASP.NET Core and console app examples, the logger is used to create logs with Information as the level. The Log level indicates the severity of the logged event.

C#

Copy
public void OnGet()
{
    Message = $"About page visited at {DateTime.UtcNow.ToLongTimeString()}";
    _logger.LogInformation("Message displayed: {Message}", Message);
}
C#

Copy
var loggerFactory = LoggerFactory.Create(builder =>
{
    builder
        .AddFilter("Microsoft", LogLevel.Warning)
        .AddFilter("System", LogLevel.Warning)
        .AddFilter("LoggingConsoleApp.Program", LogLevel.Debug)
        .AddConsole()
        .AddEventLog();
});
ILogger logger = loggerFactory.CreateLogger<Program>();
logger.LogInformation("Example log message");
Levels and categories are explained in more detail later in this article.

Create logs in the Program class
To write logs in the Program class of an ASP.NET Core app, get an ILogger instance from DI after building the host:

C#

Copy
public static void Main(string[] args)
{
    var host = CreateHostBuilder(args).Build();

    var todoRepository = host.Services.GetRequiredService<ITodoRepository>();
    todoRepository.Add(new Core.Model.TodoItem() { Name = "Feed the dog" });
    todoRepository.Add(new Core.Model.TodoItem() { Name = "Walk the dog" });

    var logger = host.Services.GetRequiredService<ILogger<Program>>();
    logger.LogInformation("Seeded the database.");

    IMyService myService = host.Services.GetRequiredService<IMyService>();
    myService.WriteLog("Logged from MyService.");

    host.Run();
}

public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });
Logging during host construction isn't directly supported. However, a separate logger can be used. In the following example, a Serilog logger is used to log in CreateHostBuilder. AddSerilog uses the static configuration specified in Log.Logger:

C#

Copy
using System;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.Hosting;
using Microsoft.Extensions.Logging;

public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args)
    {
        var builtConfig = new ConfigurationBuilder()
            .AddJsonFile("appsettings.json")
            .AddCommandLine(args)
            .Build();

        Log.Logger = new LoggerConfiguration()
            .WriteTo.Console()
            .WriteTo.File(builtConfig["Logging:FilePath"])
            .CreateLogger();

        try
        {
            return Host.CreateDefaultBuilder(args)
                .ConfigureServices((context, services) =>
                {
                    services.AddRazorPages();
                })
                .ConfigureAppConfiguration((hostingContext, config) =>
                {
                    config.AddConfiguration(builtConfig);
                })
                .ConfigureLogging(logging =>
                {   
                    logging.AddSerilog();
                })
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>();
                });
        }
        catch (Exception ex)
        {
            Log.Fatal(ex, "Host builder error");

            throw;
        }
        finally
        {
            Log.CloseAndFlush();
        }
    }
}
Create logs in the Startup class
To write logs in the Startup.Configure method of an ASP.NET Core app, include an ILogger parameter in the method signature:

C#

Copy
public void Configure(IApplicationBuilder app, IHostEnvironment env, ILogger<Startup> logger)
{
    if (env.IsDevelopment())
    {
        logger.LogInformation("In Development environment");
        app.UseDeveloperExceptionPage();
    }
    else
    {
        app.UseExceptionHandler("/Error");
        app.UseHsts();
    }

    app.UseHttpsRedirection();
    app.UseStaticFiles();

    app.UseRouting();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers();
        endpoints.MapRazorPages();
    });
}
Writing logs before completion of the DI container setup in the Startup.ConfigureServices method is not supported:

Logger injection into the Startup constructor is not supported.
Logger injection into the Startup.ConfigureServices method signature is not supported
The reason for this restriction is that logging depends on DI and on configuration, which in turns depends on DI. The DI container isn't set up until ConfigureServices finishes.

Constructor injection of a logger into Startup works in earlier versions of ASP.NET Core because a separate DI container is created for the Web Host. For information about why only one container is created for the Generic Host, see the breaking change announcement.

If you need to configure a service that depends on ILogger<T>, you can still do that by using constructor injection or by providing a factory method. The factory method approach is recommended only if there is no other option. For example, suppose you need to fill a property with a service from DI:

C#

Copy
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();
    services.AddRazorPages();

    services.AddSingleton<IMyService>((container) =>
    {
        var logger = container.GetRequiredService<ILogger<MyService>>();
        return new MyService() { Logger = logger };
    });

    services.AddSingleton<ITodoRepository, TodoRepository>();
}
The preceding highlighted code is a Func that runs the first time the DI container needs to construct an instance of MyService. You can access any of the registered services in this way.

No asynchronous logger methods
Logging should be so fast that it isn't worth the performance cost of asynchronous code. If your logging data store is slow, don't write to it directly. Consider writing the log messages to a fast store initially, then move them to the slow store later. For example, if you're logging to SQL Server, you don't want to do that directly in a Log method, since the Log methods are synchronous. Instead, synchronously add log messages to an in-memory queue and have a background worker pull the messages out of the queue to do the asynchronous work of pushing data to SQL Server. For more information, see this GitHub issue.

Configuration
Logging provider configuration is provided by one or more configuration providers:

File formats (INI, JSON, and XML).
Command-line arguments.
Environment variables.
In-memory .NET objects.
The unencrypted Secret Manager storage.
An encrypted user store, such as Azure Key Vault.
Custom providers (installed or created).
For example, logging configuration is commonly provided by the Logging section of app settings files. The following example shows the contents of a typical appsettings.Development.json file:

JSON

Copy
{
  "Logging": {
    "LogLevel": {
      "Default": "Debug",
      "System": "Information",
      "Microsoft": "Information"
    },
    "Console":
    {
      "IncludeScopes": true
    }
  }
}
The Logging property can have LogLevel and log provider properties (Console is shown).

The LogLevel property under Logging specifies the minimum level to log for selected categories. In the example, System and Microsoft categories log at Information level, and all others log at Debug level.

Other properties under Logging specify logging providers. The example is for the Console provider. If a provider supports log scopes, IncludeScopes indicates whether they're enabled. A provider property (such as Console in the example) may also specify a LogLevel property. LogLevel under a provider specifies levels to log for that provider.

If levels are specified in Logging.{providername}.LogLevel, they override anything set in Logging.LogLevel.

The Logging API doesn't include a scenario to change log levels while an app is running. However, some configuration providers are capable of reloading configuration, which takes immediate effect on logging configuration. For example, the File Configuration Provider, which is added by CreateDefaultBuilder to read settings files, reloads logging configuration by default. If configuration is changed in code while an app is running, the app can call IConfigurationRoot.Reload to update the app's logging configuration.

For information on implementing configuration providers, see Configuration in ASP.NET Core.

Sample logging output
With the sample code shown in the preceding section, logs appear in the console when the app is run from the command line. Here's an example of console output:

console

Copy
info: Microsoft.AspNetCore.Hosting.Diagnostics[1]
      Request starting HTTP/1.1 GET http://localhost:5000/api/todo/0
info: Microsoft.AspNetCore.Hosting.Diagnostics[2]
      Request finished in 84.26180000000001ms 307
info: Microsoft.AspNetCore.Hosting.Diagnostics[1]
      Request starting HTTP/2 GET https://localhost:5001/api/todo/0
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[0]
      Executing endpoint 'TodoApiSample.Controllers.TodoController.GetById (TodoApiSample)'
info: Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker[3]
      Route matched with {action = "GetById", controller = "Todo", page = ""}. Executing controller action with signature Microsoft.AspNetCore.Mvc.IActionResult GetById(System.String) on controller TodoApiSample.Controllers.TodoController (TodoApiSample).
info: TodoApiSample.Controllers.TodoController[1002]
      Getting item 0
warn: TodoApiSample.Controllers.TodoController[4000]
      GetById(0) NOT FOUND
info: Microsoft.AspNetCore.Mvc.StatusCodeResult[1]
      Executing HttpStatusCodeResult, setting HTTP status code 404
The preceding logs were generated by making an HTTP Get request to the sample app at http://localhost:5000/api/todo/0.

Here's an example of the same logs as they appear in the Debug window when you run the sample app in Visual Studio:

console

Copy
Microsoft.AspNetCore.Hosting.Diagnostics: Information: Request starting HTTP/2.0 GET https://localhost:44328/api/todo/0  
Microsoft.AspNetCore.Routing.EndpointMiddleware: Information: Executing endpoint 'TodoApiSample.Controllers.TodoController.GetById (TodoApiSample)'
Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker: Information: Route matched with {action = "GetById", controller = "Todo", page = ""}. Executing controller action with signature Microsoft.AspNetCore.Mvc.IActionResult GetById(System.String) on controller TodoApiSample.Controllers.TodoController (TodoApiSample).
TodoApiSample.Controllers.TodoController: Information: Getting item 0
TodoApiSample.Controllers.TodoController: Warning: GetById(0) NOT FOUND
Microsoft.AspNetCore.Mvc.StatusCodeResult: Information: Executing HttpStatusCodeResult, setting HTTP status code 404
Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker: Information: Executed action TodoApiSample.Controllers.TodoController.GetById (TodoApiSample) in 34.167ms
Microsoft.AspNetCore.Routing.EndpointMiddleware: Information: Executed endpoint 'TodoApiSample.Controllers.TodoController.GetById (TodoApiSample)'
Microsoft.AspNetCore.Hosting.Diagnostics: Information: Request finished in 98.41300000000001ms 404
The logs that are created by the ILogger calls shown in the preceding section begin with "TodoApiSample". The logs that begin with "Microsoft" categories are from ASP.NET Core framework code. ASP.NET Core and application code are using the same logging API and providers.

The remainder of this article explains some details and options for logging.

NuGet packages
The ILogger and ILoggerFactory interfaces are in Microsoft.Extensions.Logging.Abstractions, and default implementations for them are in Microsoft.Extensions.Logging.

Log category
When an ILogger object is created, a category is specified for it. That category is included with each log message created by that instance of ILogger. The category may be any string, but the convention is to use the class name, such as "TodoApi.Controllers.TodoController".

Use ILogger<T> to get an ILogger instance that uses the fully qualified type name of T as the category:

C#

Copy
public class TodoController : Controller
{
    private readonly ITodoRepository _todoRepository;
    private readonly ILogger _logger;

    public TodoController(ITodoRepository todoRepository,
        ILogger<TodoController> logger)
    {
        _todoRepository = todoRepository;
        _logger = logger;
    }
To explicitly specify the category, call ILoggerFactory.CreateLogger:

C#

Copy
public class TodoController : Controller
{
    private readonly ITodoRepository _todoRepository;
    private readonly ILogger _logger;

    public TodoController(ITodoRepository todoRepository,
        ILoggerFactory logger)
    {
        _todoRepository = todoRepository;
        _logger = logger.CreateLogger("TodoApiSample.Controllers.TodoController");
    }
ILogger<T> is equivalent to calling CreateLogger with the fully qualified type name of T.

Log level
Every log specifies a LogLevel value. The log level indicates the severity or importance. For example, you might write an Information log when a method ends normally and a Warning log when a method returns a 404 Not Found status code.

The following code creates Information and Warning logs:

C#

Copy
public IActionResult GetById(string id)
{
    _logger.LogInformation(LoggingEvents.GetItem, "Getting item {Id}", id);
    var item = _todoRepository.Find(id);
    if (item == null)
    {
        _logger.LogWarning(LoggingEvents.GetItemNotFound, "GetById({Id}) NOT FOUND", id);
        return NotFound();
    }
    return new ObjectResult(item);
}
In the preceding code, the first parameter is the Log event ID. The second parameter is a message template with placeholders for argument values provided by the remaining method parameters. The method parameters are explained in the message template section later in this article.

Log methods that include the level in the method name (for example, LogInformation and LogWarning) are extension methods for ILogger. These methods call a Log method that takes a LogLevel parameter. You can call the Log method directly rather than one of these extension methods, but the syntax is relatively complicated. For more information, see ILogger and the logger extensions source code.

ASP.NET Core defines the following log levels, ordered here from lowest to highest severity.

Trace = 0

For information that's typically valuable only for debugging. These messages may contain sensitive application data and so shouldn't be enabled in a production environment. Disabled by default.

Debug = 1

For information that may be useful in development and debugging. Example: Entering method Configure with flag set to true. Enable Debug level logs in production only when troubleshooting, due to the high volume of logs.

Information = 2

For tracking the general flow of the app. These logs typically have some long-term value. Example: Request received for path /api/todo

Warning = 3

For abnormal or unexpected events in the app flow. These may include errors or other conditions that don't cause the app to stop but might need to be investigated. Handled exceptions are a common place to use the Warning log level. Example: FileNotFoundException for file quotes.txt.

Error = 4

For errors and exceptions that cannot be handled. These messages indicate a failure in the current activity or operation (such as the current HTTP request), not an app-wide failure. Example log message: Cannot insert record due to duplicate key violation.

Critical = 5

For failures that require immediate attention. Examples: data loss scenarios, out of disk space.

Use the log level to control how much log output is written to a particular storage medium or display window. For example:

In production:
Logging at the Trace through Information levels produces a high-volume of detailed log messages. To control costs and not exceed data storage limits, log Trace through Information level messages to a high-volume, low-cost data store.
Logging at Warning through Critical levels typically produces fewer, smaller log messages. Therefore, costs and storage limits usually aren't a concern, which results in greater flexibility of data store choice.
During development:
Log Warning through Critical messages to the console.
Add Trace through Information messages when troubleshooting.
The Log filtering section later in this article explains how to control which log levels a provider handles.

ASP.NET Core writes logs for framework events. The log examples earlier in this article excluded logs below Information level, so no Debug or Trace level logs were created. Here's an example of console logs produced by running the sample app configured to show Debug logs:

console

Copy
info: Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker[3]
      Route matched with {action = "GetById", controller = "Todo", page = ""}. Executing controller action with signature Microsoft.AspNetCore.Mvc.IActionResult GetById(System.String) on controller TodoApiSample.Controllers.TodoController (TodoApiSample).
dbug: Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker[1]
      Execution plan of authorization filters (in the following order): None
dbug: Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker[1]
      Execution plan of resource filters (in the following order): Microsoft.AspNetCore.Mvc.ViewFeatures.Filters.SaveTempDataFilter
dbug: Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker[1]
      Execution plan of action filters (in the following order): Microsoft.AspNetCore.Mvc.Filters.ControllerActionFilter (Order: -2147483648), Microsoft.AspNetCore.Mvc.ModelBinding.UnsupportedContentTypeFilter (Order: -3000)
dbug: Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker[1]
      Execution plan of exception filters (in the following order): None
dbug: Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker[1]
      Execution plan of result filters (in the following order): Microsoft.AspNetCore.Mvc.ViewFeatures.Filters.SaveTempDataFilter
dbug: Microsoft.AspNetCore.Mvc.ModelBinding.ParameterBinder[22]
      Attempting to bind parameter 'id' of type 'System.String' ...
dbug: Microsoft.AspNetCore.Mvc.ModelBinding.Binders.SimpleTypeModelBinder[44]
      Attempting to bind parameter 'id' of type 'System.String' using the name 'id' in request data ...
dbug: Microsoft.AspNetCore.Mvc.ModelBinding.Binders.SimpleTypeModelBinder[45]
      Done attempting to bind parameter 'id' of type 'System.String'.
dbug: Microsoft.AspNetCore.Mvc.ModelBinding.ParameterBinder[23]
      Done attempting to bind parameter 'id' of type 'System.String'.
dbug: Microsoft.AspNetCore.Mvc.ModelBinding.ParameterBinder[26]
      Attempting to validate the bound parameter 'id' of type 'System.String' ...
dbug: Microsoft.AspNetCore.Mvc.ModelBinding.ParameterBinder[27]
      Done attempting to validate the bound parameter 'id' of type 'System.String'.
info: TodoApiSample.Controllers.TodoController[1002]
      Getting item 0
warn: TodoApiSample.Controllers.TodoController[4000]
      GetById(0) NOT FOUND
info: Microsoft.AspNetCore.Mvc.StatusCodeResult[1]
      Executing HttpStatusCodeResult, setting HTTP status code 404
info: Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker[2]
      Executed action TodoApiSample.Controllers.TodoController.GetById (TodoApiSample) in 32.690400000000004ms
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[1]
      Executed endpoint 'TodoApiSample.Controllers.TodoController.GetById (TodoApiSample)'
info: Microsoft.AspNetCore.Hosting.Diagnostics[2]
      Request finished in 176.9103ms 404
Log event ID
Each log can specify an event ID. The sample app does this by using a locally defined LoggingEvents class:

C#

Copy
public IActionResult GetById(string id)
{
    _logger.LogInformation(LoggingEvents.GetItem, "Getting item {Id}", id);
    var item = _todoRepository.Find(id);
    if (item == null)
    {
        _logger.LogWarning(LoggingEvents.GetItemNotFound, "GetById({Id}) NOT FOUND", id);
        return NotFound();
    }
    return new ObjectResult(item);
}
C#

Copy
public class LoggingEvents
{
    public const int GenerateItems = 1000;
    public const int ListItems = 1001;
    public const int GetItem = 1002;
    public const int InsertItem = 1003;
    public const int UpdateItem = 1004;
    public const int DeleteItem = 1005;

    public const int GetItemNotFound = 4000;
    public const int UpdateItemNotFound = 4001;
}
An event ID associates a set of events. For example, all logs related to displaying a list of items on a page might be 1001.

The logging provider may store the event ID in an ID field, in the logging message, or not at all. The Debug provider doesn't show event IDs. The console provider shows event IDs in brackets after the category:

console

Copy
info: TodoApi.Controllers.TodoController[1002]
      Getting item invalidid
warn: TodoApi.Controllers.TodoController[4000]
      GetById(invalidid) NOT FOUND
Log message template
Each log specifies a message template. The message template can contain placeholders for which arguments are provided. Use names for the placeholders, not numbers.

C#

Copy
public IActionResult GetById(string id)
{
    _logger.LogInformation(LoggingEvents.GetItem, "Getting item {Id}", id);
    var item = _todoRepository.Find(id);
    if (item == null)
    {
        _logger.LogWarning(LoggingEvents.GetItemNotFound, "GetById({Id}) NOT FOUND", id);
        return NotFound();
    }
    return new ObjectResult(item);
}
The order of placeholders, not their names, determines which parameters are used to provide their values. In the following code, notice that the parameter names are out of sequence in the message template:

C#

Copy
string p1 = "parm1";
string p2 = "parm2";
_logger.LogInformation("Parameter values: {p2}, {p1}", p1, p2);
This code creates a log message with the parameter values in sequence:

text

Copy
Parameter values: parm1, parm2
The logging framework works this way so that logging providers can implement semantic logging, also known as structured logging. The arguments themselves are passed to the logging system, not just the formatted message template. This information enables logging providers to store the parameter values as fields. For example, suppose logger method calls look like this:

C#

Copy
_logger.LogInformation("Getting item {Id} at {RequestTime}", id, DateTime.Now);
If you're sending the logs to Azure Table Storage, each Azure Table entity can have ID and RequestTime properties, which simplifies queries on log data. A query can find all logs within a particular RequestTime range without parsing the time out of the text message.

Logging exceptions
The logger methods have overloads that let you pass in an exception, as in the following example:

C#

Copy
catch (Exception ex)
{
    _logger.LogWarning(LoggingEvents.GetItemNotFound, ex, "GetById({Id}) NOT FOUND", id);
    return NotFound();
}
return new ObjectResult(item);
Different providers handle the exception information in different ways. Here's an example of Debug provider output from the code shown above.

text

Copy
TodoApiSample.Controllers.TodoController: Warning: GetById(55) NOT FOUND

System.Exception: Item not found exception.
   at TodoApiSample.Controllers.TodoController.GetById(String id) in C:\TodoApiSample\Controllers\TodoController.cs:line 226
Log filtering
You can specify a minimum log level for a specific provider and category or for all providers or all categories. Any logs below the minimum level aren't passed to that provider, so they don't get displayed or stored.

To suppress all logs, specify LogLevel.None as the minimum log level. The integer value of LogLevel.None is 6, which is higher than LogLevel.Critical (5).

Create filter rules in configuration
The project template code calls CreateDefaultBuilder to set up logging for the Console, Debug, and EventSource (ASP.NET Core 2.2 or later) providers. The CreateDefaultBuilder method sets up logging to look for configuration in a Logging section, as explained earlier in this article.

The configuration data specifies minimum log levels by provider and category, as in the following example:

JSON

Copy
{
  "Logging": {
    "Debug": {
      "LogLevel": {
        "Default": "Information"
      }
    },
    "Console": {
      "IncludeScopes": false,
      "LogLevel": {
        "Microsoft.AspNetCore.Mvc.Razor.Internal": "Warning",
        "Microsoft.AspNetCore.Mvc.Razor.Razor": "Debug",
        "Microsoft.AspNetCore.Mvc.Razor": "Error",
        "Default": "Information"
      }
    },
    "LogLevel": {
      "Default": "Debug"
    }
  }
}
This JSON creates six filter rules: one for the Debug provider, four for the Console provider, and one for all providers. A single rule is chosen for each provider when an ILogger object is created.

Filter rules in code
The following example shows how to register filter rules in code:

C#

Copy
.ConfigureLogging(logging =>
    logging.AddFilter("System", LogLevel.Debug)
           .AddFilter<DebugLoggerProvider>("Microsoft", LogLevel.Trace))
The second AddFilter specifies the Debug provider by using its type name. The first AddFilter applies to all providers because it doesn't specify a provider type.

How filtering rules are applied
The configuration data and the AddFilter code shown in the preceding examples create the rules shown in the following table. The first six come from the configuration example and the last two come from the code example.

Number	Provider	Categories that begin with ...	Minimum log level
1	Debug	All categories	Information
2	Console	Microsoft.AspNetCore.Mvc.Razor.Internal	Warning
3	Console	Microsoft.AspNetCore.Mvc.Razor.Razor	Debug
4	Console	Microsoft.AspNetCore.Mvc.Razor	Error
5	Console	All categories	Information
6	All providers	All categories	Debug
7	All providers	System	Debug
8	Debug	Microsoft	Trace
When an ILogger object is created, the ILoggerFactory object selects a single rule per provider to apply to that logger. All messages written by an ILogger instance are filtered based on the selected rules. The most specific rule possible for each provider and category pair is selected from the available rules.

The following algorithm is used for each provider when an ILogger is created for a given category:

Select all rules that match the provider or its alias. If no match is found, select all rules with an empty provider.
From the result of the preceding step, select rules with longest matching category prefix. If no match is found, select all rules that don't specify a category.
If multiple rules are selected, take the last one.
If no rules are selected, use MinimumLevel.
With the preceding list of rules, suppose you create an ILogger object for category "Microsoft.AspNetCore.Mvc.Razor.RazorViewEngine":

For the Debug provider, rules 1, 6, and 8 apply. Rule 8 is most specific, so that's the one selected.
For the Console provider, rules 3, 4, 5, and 6 apply. Rule 3 is most specific.
The resulting ILogger instance sends logs of Trace level and above to the Debug provider. Logs of Debug level and above are sent to the Console provider.

Provider aliases
Each provider defines an alias that can be used in configuration in place of the fully qualified type name. For the built-in providers, use the following aliases:

Console
Debug
EventSource
EventLog
TraceSource
AzureAppServicesFile
AzureAppServicesBlob
ApplicationInsights
Default minimum level
There's a minimum level setting that takes effect only if no rules from configuration or code apply for a given provider and category. The following example shows how to set the minimum level:

C#

Copy
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureLogging(logging => logging.SetMinimumLevel(LogLevel.Warning))
If you don't explicitly set the minimum level, the default value is Information, which means that Trace and Debug logs are ignored.

Filter functions
A filter function is invoked for all providers and categories that don't have rules assigned to them by configuration or code. Code in the function has access to the provider type, category, and log level. For example:

C#

Copy
.ConfigureLogging(logBuilder =>
{
    logBuilder.AddFilter((provider, category, logLevel) =>
    {
        if (provider == "Microsoft.Extensions.Logging.Console.ConsoleLoggerProvider" &&
            category == "TodoApiSample.Controllers.TodoController")
        {
            return false;
        }
        return true;
    });
})
System categories and levels
Here are some categories used by ASP.NET Core and Entity Framework Core, with notes about what logs to expect from them:

Category	Notes
Microsoft.AspNetCore	General ASP.NET Core diagnostics.
Microsoft.AspNetCore.DataProtection	Which keys were considered, found, and used.
Microsoft.AspNetCore.HostFiltering	Hosts allowed.
Microsoft.AspNetCore.Hosting	How long HTTP requests took to complete and what time they started. Which hosting startup assemblies were loaded.
Microsoft.AspNetCore.Mvc	MVC and Razor diagnostics. Model binding, filter execution, view compilation, action selection.
Microsoft.AspNetCore.Routing	Route matching information.
Microsoft.AspNetCore.Server	Connection start, stop, and keep alive responses. HTTPS certificate information.
Microsoft.AspNetCore.StaticFiles	Files served.
Microsoft.EntityFrameworkCore	General Entity Framework Core diagnostics. Database activity and configuration, change detection, migrations.
Log scopes
A scope can group a set of logical operations. This grouping can be used to attach the same data to each log that's created as part of a set. For example, every log created as part of processing a transaction can include the transaction ID.

A scope is an IDisposable type that's returned by the BeginScope method and lasts until it's disposed. Use a scope by wrapping logger calls in a using block:

C#

Copy
public IActionResult GetById(string id)
{
    TodoItem item;
    using (_logger.BeginScope("Message attached to logs created in the using block"))
    {
        _logger.LogInformation(LoggingEvents.GetItem, "Getting item {Id}", id);
        item = _todoRepository.Find(id);
        if (item == null)
        {
            _logger.LogWarning(LoggingEvents.GetItemNotFound, "GetById({Id}) NOT FOUND", id);
            return NotFound();
        }
    }
    return new ObjectResult(item);
}
The following code enables scopes for the console provider:

Program.cs:

C#

Copy
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureLogging((hostingContext, logging) =>
        {
            logging.ClearProviders();
            logging.AddConsole(options => options.IncludeScopes = true);
            logging.AddDebug();
        })
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });
 Note

Configuring the IncludeScopes console logger option is required to enable scope-based logging.

For information on configuration, see the Configuration section.

Each log message includes the scoped information:


Copy
info: TodoApiSample.Controllers.TodoController[1002]
      => RequestId:0HKV9C49II9CK RequestPath:/api/todo/0 => TodoApiSample.Controllers.TodoController.GetById (TodoApi) => Message attached to logs created in the using block
      Getting item 0
warn: TodoApiSample.Controllers.TodoController[4000]
      => RequestId:0HKV9C49II9CK RequestPath:/api/todo/0 => TodoApiSample.Controllers.TodoController.GetById (TodoApi) => Message attached to logs created in the using block
      GetById(0) NOT FOUND
Built-in logging providers
ASP.NET Core ships the following providers:

Console
Debug
EventSource
EventLog
TraceSource
AzureAppServicesFile
AzureAppServicesBlob
ApplicationInsights
For information on stdout and debug logging with the ASP.NET Core Module, see Troubleshoot ASP.NET Core on Azure App Service and IIS and ASP.NET Core Module.

Console provider
The Microsoft.Extensions.Logging.Console provider package sends log output to the console.

C#

Copy
logging.AddConsole();
To see console logging output, open a command prompt in the project folder and run the following command:

.NET Core CLI

Copy
dotnet run
Debug provider
The Microsoft.Extensions.Logging.Debug provider package writes log output by using the System.Diagnostics.Debug class (Debug.WriteLine method calls).

On Linux, this provider writes logs to /var/log/message.

C#

Copy
logging.AddDebug();
