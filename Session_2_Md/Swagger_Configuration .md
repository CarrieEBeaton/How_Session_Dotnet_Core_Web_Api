## Package installation
Swashbuckle can be added with the following approaches:

Visual Studio

- From the Package Manager Console window:

    - Go to View > Other Windows > Package Manager Console

    - Navigate to the directory in which the TodoApi.csproj file exists

    - Execute the following command:

PowerShell | Copy
----|-----
        Install-Package Swashbuckle.AspNetCore -Version 5.0.0-rc5
-  From the Manage NuGet Packages dialog:

- Right-click the project in Solution Explorer > Manage NuGet Packages
- Set the Package source to "nuget.org"
- Ensure the "Include prerelease" option is enabled
- Enter "Swashbuckle.AspNetCore" in the search box
- Select the latest "Swashbuckle.AspNetCore" package from the Browse tab and click Install
## Add and configure Swagger middleware
In the Startup class, import the following namespace to use the OpenApiInfo class:
C# | Copy
----|-----
```cs
        using Microsoft.OpenApi.Models;
```
Add the Swagger generator to the services collection in the Startup.ConfigureServices method:

C# | Copy
----|-----
```cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddDbContext<TodoContext>(opt =>
        opt.UseInMemoryDatabase("TodoList"));
    services.AddControllers();

    // Register the Swagger generator, defining 1 or more Swagger documents
    services.AddSwaggerGen(c =>
    {
        c.SwaggerDoc("v1", new OpenApiInfo { Title = "My API", Version = "v1" });
    });
}
```
In the Startup.Configure method, enable the middleware for serving the generated JSON document and the Swagger UI:

C# | Copy
----|-----
```cs
public void Configure(IApplicationBuilder app)
{
    // Enable middleware to serve generated Swagger as a JSON endpoint.
    app.UseSwagger();

    // Enable middleware to serve swagger-ui (HTML, JS, CSS, etc.),
    // specifying the Swagger JSON endpoint.
    app.UseSwaggerUI(c =>
    {
        c.SwaggerEndpoint("/swagger/v1/swagger.json", "My API V1");
    });

    app.UseRouting();
    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers();
    });
}
```
The preceding UseSwaggerUI method call enables the Static File Middleware. If targeting .NET Framework or .NET Core 1.x, add the Microsoft.AspNetCore.StaticFiles NuGet package to the project.

Launch the app, and navigate to http://localhost:<port>/swagger/v1/swagger.json. The generated document describing the endpoints appears as shown in Swagger specification (swagger.json).

The Swagger UI can be found at http://localhost:<port>/swagger. Explore the API via Swagger UI and incorporate it in other programs.

 >**Tip**: 
>To serve the Swagger UI at the app's root (http://localhost:<port>/), set the RoutePrefix property to an empty string:

C# | Copy
----|-----
```cs
app.UseSwaggerUI(c =>
{
    c.SwaggerEndpoint("/swagger/v1/swagger.json", "My API V1");
    c.RoutePrefix = string.Empty;
});
```

If using directories with IIS or a reverse proxy, set the Swagger endpoint to a relative path using the ./ prefix. For example, ./swagger/v1/swagger.json. Using /swagger/v1/swagger.json instructs the app to look for the JSON file at the true root of the URL (plus the route prefix, if used). For example, use http://localhost:<port>/<route_prefix>/swagger/v1/swagger.json instead of http://localhost:<port>/<virtual_directory>/<route_prefix>/swagger/v1/swagger.json.

Customize and extend
Swagger provides options for documenting the object model and customizing the UI to match your theme.

In the Startup class, add the following namespaces:

C# | Copy
----|-----
```cs
        using System;
        using System.Reflection;
        using System.IO;
        using Swashbuckle.AspNetCore.Swagger;
```

## API info and description
The configuration action passed to the AddSwaggerGen method adds information such as the author, license, and description:

C# | Copy
----|-----
```cs
        // Register the Swagger generator, defining 1 or more Swagger documents
        services.AddSwaggerGen(c =>
        {
            c.SwaggerDoc("v1", new Info
            {
                Version = "v1",
                Title = "ToDo API",
                Description = "A simple example ASP.NET Core Web API",
                TermsOfService = new Uri("https://example.com/terms"),
                Contact = new OpenApiContact
                {
                    Name = "Shayne Boyer",
                    Email = string.Empty,
                    Url = new Uri("https://twitter.com/spboyer"),
                },
                License = new OpenApiLicense
                {
                    Name = "Use under LICX",
                    Url = new Uri("https://example.com/license"),
                }
            });
        });
```
The Swagger UI displays the version's information:

Swagger UI with version information: description, author, and see more link

## XML comments
XML comments can be enabled with the following approaches:

Visual Studio
Visual Studio for Mac
Visual Studio Code
.NET Core CLI
Right-click the project in Solution Explorer and select Edit <project_name>.csproj.
Manually add the highlighted lines to the .csproj file:
XML | Copy
----|-----

```xml
        <PropertyGroup>
         <GenerateDocumentationFile>true</GenerateDocumentationFile>
         <NoWarn>$(NoWarn);1591</NoWarn>
        </PropertyGroup>
```
Enabling XML comments provides debug information for undocumented public types and members. Undocumented types and members are indicated by the warning message. For example, the following message indicates a violation of warning code 1591:
text | Copy
----|-----

```text
    warning CS1591: Missing XML comment for publicly visible type or member 'TodoController.GetAll()'
```
To suppress warnings project-wide, define a semicolon-delimited list of warning codes to ignore in the project file. Appending the warning codes to $(NoWarn); applies the C# default values too.

XML | Copy
----|-----
```xml
        <PropertyGroup>
         <GenerateDocumentationFile>true</GenerateDocumentationFile>
         <NoWarn>$(NoWarn);1591</NoWarn>
        </PropertyGroup>
```
To suppress warnings only for specific members, enclose the code in #pragma warning preprocessor directives. This approach is useful for code that shouldn't be exposed via the API docs. In the following example, warning code CS1591 is ignored for the entire Program class. Enforcement of the warning code is restored at the close of the class definition. Specify multiple warning codes with a comma-delimited list.

C# | Copy
----|-----

```cs
        namespace TodoApi
        {
        #pragma warning disable CS1591
           public class Program
          {
             public static void Main(string[] args) =>
                BuildWebHost(args).Run();

                public static IWebHost BuildWebHost(string[] args) =>
                  WebHost.CreateDefaultBuilder(args)
                      .UseStartup<Startup>()
                      .Build();
        }
        #pragma warning restore CS1591 
        }

```
    
Configure Swagger to use the XML file that's generated with the preceding instructions. For Linux or non-Windows operatingsystems, file names and paths can be case-sensitive. For example, a TodoApi.XML file is valid on Windows but not CentOS.



C# | Copy
----|-----
       
```cs
        public void ConfigureServices(IServiceCollection services)
        {
           services.AddDbContext<TodoContext>(opt =>
               opt.UseInMemoryDatabase("TodoList"));
           services.AddControllers();

           // Register the Swagger generator, defining 1 or more Swagger documents
           services.AddSwaggerGen(c =>
           {
              c.SwaggerDoc("v1", new OpenApiInfo
              {
              Version = "v1",
                  Title = "ToDo API",
                  Description = "A simple example ASP.NET Core Web API",
                  TermsOfService = new Uri("https://example.com/terms"),
                  Contact = new OpenApiContact
                  {
                      Name = "Shayne Boyer",
                      Email = string.Empty,
                      Url = new Uri("https://twitter.com/spboyer"),
                  },
                  License = new OpenApiLicense
                  {
                      Name = "Use under LICX",
                      Url = new Uri("https://example.com/license"),
                  }
              });

              // Set the comments path for the Swagger JSON and UI.
              var xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
              var xmlPath = Path.Combine(AppContext.BaseDirectory, xmlFile);
              c.IncludeXmlComments(xmlPath);
          });

```          
In the preceding code, Reflection is used to build an XML file name matching that of the web API project. The AppContext.BaseDirectory property is used to construct a path to the XML file. Some Swagger features `(for example, schemata of input parameters or HTTP methods and response codes from the respective attributes)` work without the use of an XML documentation file. For most features, namely method summaries and the descriptions of parameters and response codes, the use of an XML file is mandatory.

Adding triple-slash comments to an action enhances the Swagger UI by adding the description to the section header. Add a <summary> element above the Delete action:

C# | Copy
----|-----

```cs
    /// <summary>
    /// Deletes a specific TodoItem.
    /// </summary>
    /// <param name="id"></param>        
    [HttpDelete("{id}")]
    public IActionResult Delete(long id)
    {
        var todo = _context.TodoItems.Find(id);

        if (todo == null)
        {
            return NotFound();
        }

        _context.TodoItems.Remove(todo);
        _context.SaveChanges();

        return NoContent();
    }

```
The Swagger UI displays the inner text of the preceding code's <summary> element:

Swagger UI showing XML comment 'Deletes a specific TodoItem.' for the DELETE method

The UI is driven by the generated JSON schema:

JSON | Copy
----|-----

```js 
        "delete": {
        "tags": [
        "Todo"
       ],
          "summary": "Deletes a specific TodoItem.",
          "operationId": "ApiTodoByIdDelete",
          "consumes": [],
           "produces": [],
          "parameters": [
          {
              "name": "id",
              "in": "path",
              "description": "",
              "required": true,
              "type": "integer",
              "format": "int64"
          }
       ],
       "responses": {
           "200": {
               "description": "Success"
           }
       }
    }
    
```
Add a <remarks> element to the Create action method documentation. It supplements information specified in the <summary> element and provides a more robust Swagger UI. The <remarks> element content can consist of text, JSON, or XML.

C# | Copy
----|-----

```cs
    /// <summary>
    /// Creates a TodoItem.
    /// </summary>
    /// <remarks>
    /// Sample request:
    ///
    ///     POST /Todo
    ///     {
    ///        "id": 1,
    ///        "name": "Item1",
    ///        "isComplete": true
    ///     }
    ///
    /// </remarks>
    /// <param name="item"></param>
    /// <returns>A newly created TodoItem</returns>
    /// <response code="201">Returns the newly created item</response>
    /// <response code="400">If the item is null</response>            
    [HttpPost]
    [ProducesResponseType(StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public ActionResult<TodoItem> Create(TodoItem item)
    {
        _context.TodoItems.Add(item);
        _context.SaveChanges();

        return CreatedAtRoute("GetTodo", new { id = item.Id }, item);
    }
```
Notice the UI enhancements with these additional comments:

Swagger UI with additional comments shown

## Data annotations
Mark the model with attributes, found in the System.ComponentModel.DataAnnotations namespace, to help drive the Swagger UI components.

Add the [Required] attribute to the Name property of the TodoItem class:

C# | Copy
----|-----

```cs
    using System.ComponentModel;
    using System.ComponentModel.DataAnnotations;

    namespace TodoApi.Models
    {
        public class TodoItem
        {
            public long Id { get; set; }

            [Required]
            public string Name { get; set; }

            [DefaultValue(false)]
            public bool IsComplete { get; set; }
        }
    }
```
The presence of this attribute changes the UI behavior and alters the underlying JSON schema:


JSON | Copy
----|-----

```js
        "definitions": {
            "TodoItem": {
                "required": [
                    "name"
                ],
                "type": "object",
                "properties": {
                    "id": {
                        "format": "int64",
                        "type": "integer"
                    },
                    "name": {
                        "type": "string"
                    },
                    "isComplete": {
                        "default": false,
                        "type": "boolean"
                    }
                }
            }
        },
```
Add the [Produces("application/json")] attribute to the API controller. Its purpose is to declare that the controller's actions support a response content type of application/json:

C#  | Copy
----|-----

```cs
    [Produces("application/json")]
    [Route("api/[controller]")]
    [ApiController]
    public class TodoController : ControllerBase
    {
        private readonly TodoContext _context;
```

The Response Content Type drop-down selects this content type as the default for the controller's GET actions:

Swagger UI with default response content type

As the usage of data annotations in the web API increases, the UI and API help pages become more descriptive and useful.

Describe response types
Developers consuming a web API are most concerned with what's returned—specifically response types and error codes (if not standard). The response types and error codes are denoted in the XML comments and data annotations.

The Create action returns an HTTP 201 status code on success. An HTTP 400 status code is returned when the posted request body is null. Without proper documentation in the Swagger UI, the consumer lacks knowledge of these expected outcomes. Fix that problem by adding the highlighted lines in the following example:

C#  | Copy
----|-----

```cs
     /// <summary>
     /// Creates a TodoItem.
     /// </summary>
     /// <remarks>
     /// Sample request:
     ///
     ///     POST /Todo
     ///     {
     ///        "id": 1,
     ///        "name": "Item1",
     ///        "isComplete": true
     ///     }
     ///
     /// </remarks>
     /// <param name="item"></param>
     /// <returns>A newly created TodoItem</returns>
     /// <response code="201">Returns the newly created item</response>
     /// <response code="400">If the item is null</response>            
     [HttpPost]
     [ProducesResponseType(StatusCodes.Status201Created)]
     [ProducesResponseType(StatusCodes.Status400BadRequest)]
     public ActionResult<TodoItem> Create(TodoItem item)
     {
         _context.TodoItems.Add(item);
         _context.SaveChanges();

         return CreatedAtRoute("GetTodo", new { id = item.Id }, item);
        }

```
The Swagger UI now clearly documents the expected HTTP response codes:


Swagger UI showing POST Response Class description 'Returns the newly created Todo item' and '400 - If the item is null' for status code and reason under Response Messages

In ASP.NET Core 2.2 or later, conventions can be used as an alternative to explicitly decorating individual actions with [ProducesResponseType]. For more information, see Use web API conventions.

## Customize the UI
The stock UI is both functional and presentable. However, API documentation pages should represent your brand or theme. Branding the Swashbuckle components requires adding the resources to serve static files and building the folder structure to host those files.

If targeting .NET Framework or .NET Core 1.x, add the Microsoft.AspNetCore.StaticFiles NuGet package to the project:
XML | Copy
----|-----
```xml
<PackageReference Include="Microsoft.AspNetCore.StaticFiles" Version="2.0.0" />
```
The preceding NuGet package is already installed if targeting .NET Core 2.x and using the metapackage.

Enable Static File Middleware:

C#  | Copy
----|-----


```cs
        public void Configure(IApplicationBuilder app)
        {
            app.UseStaticFiles();

            // Enable middleware to serve generated Swagger as a JSON endpoint.
            app.UseSwagger();

            // Enable middleware to serve swagger-ui (HTML, JS, CSS, etc.),
            // specifying the Swagger JSON endpoint.
            app.UseSwaggerUI(c =>
            {
                c.SwaggerEndpoint("/swagger/v1/swagger.json", "My API V1");
            });

            app.UseRouting();
            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllers();
            });
        }
```
Acquire the contents of the dist folder from the Swagger UI GitHub repository. This folder contains the necessary assets for the Swagger UI page.

Create a wwwroot/swagger/ui folder, and copy into it the contents of the dist folder.

Create a custom.css file, in wwwroot/swagger/ui, with the following CSS to customize the page header:

css | Copy
----|-----
```css
.swagger-ui .topbar {
    background-color: #000;
    border-bottom: 3px solid #547f00;
}
```
Reference custom.css in the index.html file inside ui folder, after any other CSS files:
HTML | Copy
-----|-----
```html
<link href="https://fonts.googleapis.com/css?family=Open+Sans:400,700|Source+Code+Pro:300,600|Titillium+Web:400,600,700" rel="stylesheet">
<link rel="stylesheet" type="text/css" href="./swagger-ui.css">
<link rel="stylesheet" type="text/css" href="custom.css">
```
Browse to the index.html page at http://localhost:<port>/swagger/ui/index.html. Enter https://localhost:<port>/swagger/v1/swagger.json in the header's textbox, and click the Explore button. The resulting page looks as follows: