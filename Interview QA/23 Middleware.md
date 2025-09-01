# Middleware

## Definition:
Middleware is a software component that is assembled into an application pipeline to handle requests and responses in an ASP.NET Core application. Each piece of middleware can perform operations before and after the next component in the pipeline is invoked.
It allows you to add custom processing to requests and responses globally or for specific routes.

## Purpose:
- **Request Handling**: Middleware is designed to handle aspects of request processing, such as routing, authentication, logging, error handling, and modifying requests or responses.
- **Pipeline Management**: Middleware components execute in a defined order, allowing you to compose functionalities in a modular way which keeps your application architecture clean and maintainable.

## How Middleware Works:
- In ASP.NET Core, middleware components are executed in the order they are registered in the Startup.cs file, specifically within the Configure method. Each middleware can either process the request, pass it to the next component, or generate a response directly.
- The Invoke or InvokeAsync methods are used to define the logic executed for each request. Each middleware can modify the HTTP context, and it is important to call await _next(context); to pass control to the next middleware in the pipeline.

## Example of Middleware:
Let's consider a simple example of a custom logging middleware that logs the execution time of incoming requests:

```csharp
public class RequestTimingMiddleware
{
    private readonly RequestDelegate _next;

    public RequestTimingMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        var watch = System.Diagnostics.Stopwatch.StartNew();

        await _next(context); // Call the next middleware

        watch.Stop();
        var elapsed = watch.ElapsedMilliseconds;
        Console.WriteLine($"Request {context.Request.Method} {context.Request.Path} took {elapsed} ms");
    }
}
```

You register this middleware in your Startup.cs file:

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    app.UseMiddleware<RequestTimingMiddleware>(); // Registering the middleware
    // Other middleware registrations e.g., app.UseRouting(), app.UseEndpoints(), etc.
}
```
## Usage Scenarios:
- Middleware is beneficial in scenarios where you need to handle cross-cutting concerns such as:
    - *Authentication and Authorization*: Validating user credentials and access permissions.
    - *Logging and Monitoring*: Tracking request metrics or application performance.
    - *Error Handling*: Global error catching to return user-friendly errors.
    - *CORS Handling*: Enabling Cross-Origin Resource Sharing for API calls.
