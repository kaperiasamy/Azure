# Dependency Injection Lifetimes

When registering services in a DI container, you define how long those services should be retained. The three common lifetimes are **Transient**, **Scoped**, and **Singleton**.

1. **Transient**

**Definition**: A Transient service is created each time it is requested. Every time a class or function requires this service, a new instance is provided.
**Use Cases**: Use Transient services for lightweight, stateless services where each request can be treated independently. They are ideal for non-shared services where data does not need to be preserved between requests.
**Example**:

```csharp
services.AddTransient<IEmailService, EmailService>(); // A new instance for each injection
```

2. Scoped

**Definition**: A Scoped service is created once per request (or per scope). In web applications, this typically means one instance per HTTP request, shared across all components involved in handling that request.
**Use Cases**: This is useful for services that need to maintain state throughout the lifecycle of a single request but should not be shared between different requests. They fit well for database contexts (like DbContext in Entity Framework), where you want to ensure that all operations on that context are conducted in a single transaction.
**Example**:

```csharp
services.AddScoped<IProductRepository, ProductRepository>(); // A single instance per request
```

3. Singleton

**Definition**: A Singleton service is created once and shared throughout the application's lifetime. There is only one instance per application, and it is reused across all requests.
**Use Cases**: Use Singleton for stateless services or services that manage shared resources and do not hold any state that is specific to a single request. Singleton services are ideal for cache design, shared configurations, and logging services where a consistent instance is needed.
**Example**:

```csharp
services.AddSingleton<ILoggingService, LoggingService>(); // A single instance for the application's lifetime
```

## Summary of Differences

| Lifetime  | Instance Creation            | Request Scope                      | Use Cases                                |
|-----------|------------------------------|------------------------------------|------------------------------------------|
| Transient | New instance on each request | Multiple instances across requests | Lightweight, stateless services          |
| Scoped    | One instance per request     | Shared across a single request     | Services needing stateful context        |
| Singleton | One instance for lifetime    | Shared across all requests         | Shared resources, configuration, logging |
