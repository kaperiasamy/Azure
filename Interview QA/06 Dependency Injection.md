# Dependency Injection

## Definition of Dependency Injection

**Dependency Injection** is a software design pattern used to implement **Inversion of Control (IoC)**, allowing for the decoupling of components in an application. In DI, an object receives its dependencies from an external source rather than creating them internally, which promotes better modularization and testability.

## Importance of Dependency Injection

- **Decoupling**: DI separates the creation of a dependent object from its usage, allowing changes to be made to dependencies without affecting the dependent class.
- **Testability**: By allowing mock dependencies to be injected, it enhances the ease of unit testing components in isolation.
- **Maintainability**: Changes to dependencies require minimal changes to the consuming classes, improving the overall maintainability of the codebase.

## Types of Dependency Injection

### Constructor Injection:
**Definition**: Dependencies are provided through class constructors.
**Usage**: The dependent class requires the dependencies to be passed during instantiation, making them available throughout the class scope.

**Example**:

```csharp
public class OrderService
{
    private readonly IProductRepository _productRepository;

    public OrderService(IProductRepository productRepository)
    {
        _productRepository = productRepository;
    }
}
```

### Setter Injection (or Property Injection):
**Definition**: Dependencies are provided through public properties or setter methods after the object is constructed.
**Usage**: This allows for optional dependencies that can be set post-instantiation.

**Example**:

```csharp
public class EmailService
{
    public IEmailSender EmailSender { get; set; }

    public void SendEmail(string message)
    {
        EmailSender?.Send(message);
    }
}
```

### Method Injection:
**Definition**: Dependencies are provided through method parameters at the time the method is called.
**Usage**: Useful for methods requiring different dependencies each time they are called without needing the dependencies for the entire lifespan of the class.
    
**Example**:

```csharp
public class NotificationService
{
    public void Notify(IUserNotifier userNotifier)
    {
        userNotifier.Notify("Notification message");
    }
}
```

## Definition of Inversion of Control

Inversion of Control is a design principle that reverses the flow of control in a system. Instead of the application code calling and managing object dependencies, the IoC framework takes over this responsibility. Essentially, IoC allows for greater separation between components and promotes loose coupling within software systems.

### Relationship to Dependency Injection

Dependency Injection is one of the most common forms of IoC. It facilitates the IoC principle by injecting dependencies into classes rather than having those classes instantiate their own dependencies. While IoC is a broader concept, Dependency Injection specifically focuses on the mechanism through which dependencies are supplied.

### Key Benefits of IoC

**Decoupling**: Reduces dependencies between modules, making it easier to change or replace components without affecting the rest of the system.
**Testability**: Improves the ability to unit test components in isolation by allowing dependencies to be mocked or stubbed.
**Maintainability**: Enhances code maintainability by reducing complexity and improving the clarity of component interactions.

**Example Implementation**

In a typical .NET application using an IoC container (like Microsoft's built-in dependency injection in ASP.NET Core), you might register services and let the framework handle the lifecycle.

```csharp

// Registering services in Startup.cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddScoped<IProductRepository, ProductRepository>();
    services.AddScoped<OrderService>(); // IoC manages the dependency
}

// Using OrderService in a controller
public class OrdersController : ControllerBase
{
    private readonly OrderService _orderService;

    public OrdersController(OrderService orderService)
    {
        _orderService = orderService; // Dependency injected by IoC container
    }
}
```

## Definition of Design Patterns

**Design Pattern**: A design pattern is a proven solution to a recurring problem in software design. It is a general, reusable blueprint that can be applied to solve specific design issues in a particular context. Patterns encapsulate best practices and experience gathered from various software development scenarios.
**Example**: Common design patterns include:
- **Singleton**: Ensures a class has only one instance and provides a global point of access to it.
- **Observer**: A pattern where an object maintains a list of dependencies and notifies them of state changes.

## Definition of Design Principles

**Design Principle**: Design principles are foundational guidelines or best practices for software design that provide overarching strategies for creating software systems. They help maintain good quality and ensure that systems are modular, maintainable, and scalable.
**Example**: Key design principles include:
- **SOLID Principles**: A set of five principles that aims to make software designs more understandable, flexible, and maintainable.
- **DRY (Don’t Repeat Yourself)**: Advocates for reducing duplication in code to minimize errors and improve maintainability.

## Key Differences

| Feature    | Design Patterns                                | Design Principles                                  |
|------------|------------------------------------------------|----------------------------------------------------|
| Definition | Reusable solutions to specific design problems | General guidelines for designing software systems  |
| Scope      | Focused on particular design scenarios         | Broad principles applicable to all stages of design|
| Example    | Singleton, Factory Method, Observer            | SOLID, DRY, KISS (Keep It Simple, Stupid)          |
| Usage      | Implemented in code as specific constructs     | Guide decision-making and overall architecture     |
