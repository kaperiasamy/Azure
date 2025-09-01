# Constructor

## Definition of Constructor

Constructor: A constructor is a special method in a class that is automatically invoked when an instance (or object) of that class is created. It typically has the same name as the class and does not have a return type, not even void.

## Purpose of a Constructor

Initialization of Objects: The primary purpose of constructors is to initialize the new object’s properties and allocate resources required by the object. It ensures that the object is in a valid and usable state immediately upon creation.

## Why We Use Constructors

### 1. Setting Default Values:
Constructors allow you to define default values for object properties. This ensures that all instances of a class start with a consistent state. For example:

```csharp
public class Product
{
    public string Name { get; set; }
    public decimal Price { get; set; }

    // Constructor with default values
    public Product()
    {
        Name = "Default Product";
        Price = 0.0m;
    }
}
```

### 2. Overloading Options:
In .NET, you can have multiple constructors with different parameter lists (overloading). This flexibility allows you to create instances of the class in various ways, depending on the constructor invoked:

```csharp
public class Employee
{
    public string Name { get; private set; }
    public string Role { get; private set; }

    // Constructor for initializing only name
    public Employee(string name)
    {
        Name = name;
        Role = "Unknown";
    }

    // Overloaded constructor for full initialization
    public Employee(string name, string role)
    {
        Name = name;
        Role = role;
    }
}
```

### 3. Encapsulation of Creation Logic:
Constructors encapsulate the logic necessary to create an object. This keeps object initialization processes separate from other methods, maintaining a clean and organized class design.

### 4. Dependency Injection:
In applications utilizing Dependency Injection (DI), constructors are often used to inject dependencies into a class. This promotes loose coupling and enhances testability by allowing mock dependencies to be passed during instantiation:

```csharp
public class OrderService
{
    private readonly IProductRepository _productRepository;

    // Constructor injection
    public OrderService(IProductRepository productRepository)
    {
        _productRepository = productRepository;
    }
}
```
