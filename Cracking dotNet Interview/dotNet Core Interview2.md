## **ARCHITECTURAL DESIGN QUESTIONS**

### **Layered Architecture**
19. **N-Tier Architecture**
    - How do you structure a Web API project with proper separation of concerns?
    - Show Presentation → Business → Data layer implementation.
    - How do you handle cross-cutting concerns across layers?

---
# Layered Architecture: A Comprehensive Guide

## Definition and Core Principles

Layered architecture is a software architectural pattern that organizes code into horizontal layers, where each layer has specific responsibilities and communicates only with adjacent layers. This pattern follows the principle of **separation of concerns**, creating a clear hierarchy where higher-level layers depend on lower-level layers, but not vice versa.

### Core Principles:
1. **Unidirectional Dependencies** - Upper layers can depend on lower layers, never the reverse
2. **Layer Isolation** - Each layer should be replaceable without affecting other layers
3. **Single Responsibility** - Each layer handles one specific aspect of the application
4. **Abstraction** - Layers communicate through well-defined interfaces

## Common Layers and Their Responsibilities

### 1. Presentation Layer (UI/Web Layer)
**Responsibilities:**
- User interface components and user interaction handling
- Input validation and formatting
- Request routing and response formatting
- Authentication and authorization UI flows

**Technologies:** Web frameworks (Spring MVC, ASP.NET MVC), REST controllers, GraphQL resolvers, UI components

### 2. Business/Application Layer (Service Layer)
**Responsibilities:**
- Business logic implementation
- Transaction coordination
- Application workflows and orchestration
- Business rule validation
- Use case implementation

**Key Components:** Services, workflows, business rule engines, application facades

### 3. Data Access Layer (DAL/Repository Layer)
**Responsibilities:**
- Data persistence abstraction
- Query construction and execution
- Data mapping between domain objects and database entities
- Connection management

**Technologies:** ORMs (Hibernate, Entity Framework), repositories, DAOs, query builders

### 4. Database Layer
**Responsibilities:**
- Data storage and retrieval
- Data integrity and consistency
- Performance optimization through indexing
- Backup and recovery

**Components:** Relational databases, NoSQL stores, data warehouses

## Advantages

### Separation of Concerns
Each layer focuses on a specific responsibility, making the codebase more organized and understandable. Changes in UI don't affect business logic, and database changes don't impact presentation.

### Maintainability
Clear boundaries make it easier to locate and modify specific functionality. Bugs are typically contained within their respective layers, simplifying debugging.

### Testability
Layers can be tested in isolation using mocks or stubs for dependencies. Unit testing becomes more straightforward with well-defined interfaces between layers.

### Team Specialization
Different teams can work on different layers simultaneously - UI/UX teams on presentation, backend developers on business logic, DBAs on database optimization.

### Technology Independence
Each layer can use different technologies. You might use React for presentation, Java for business logic, and PostgreSQL for data storage.

## Disadvantages

### Performance Overhead
Multiple layer transitions can introduce latency. Each layer adds processing time and memory usage, especially problematic in high-performance scenarios.

### Potential for Tight Coupling
Without careful design, layers can become tightly coupled through shared data structures or overly specific interfaces, reducing flexibility.

### Complexity in Cross-Cutting Concerns
Features like logging, security, and caching span multiple layers, making them challenging to implement consistently without code duplication.

### Over-Engineering Risk
Simple applications may become unnecessarily complex when forced into a layered structure, leading to the "architecture astronaut" anti-pattern.

## When to Use Layered Architecture vs Alternatives

### Use Layered Architecture When:
- Building traditional enterprise applications with clear separation needs
- Working with teams that have specialized skills (UI, business logic, database)
- Requirements are well-understood and stable
- Performance is not the primary concern
- Compliance requires clear audit trails and separation

### Consider Alternatives When:

**Hexagonal Architecture** for:
- Domain-driven design approaches
- Need for high testability and port/adapter flexibility
- Complex business domains requiring isolation from external concerns

**Clean Architecture** for:
- Independence from frameworks and external dependencies
- Long-term maintainability over decades
- Complex business rules that shouldn't be influenced by external factors

**Microservices** for:
- Large, distributed teams
- Need for independent deployability
- Different scaling requirements for different features

## Real-World Implementation Challenges and Solutions

### Challenge 1: Anemic Domain Model
**Problem:** Business logic scattered across layers, domain objects become mere data containers.

**Solution:** 
```java
// Instead of anemic model
public class Order {
    private List<OrderItem> items;
    private OrderStatus status;
    // Only getters/setters
}

// Rich domain model
public class Order {
    private List<OrderItem> items;
    private OrderStatus status;
    
    public void addItem(Product product, int quantity) {
        validateProduct(product);
        items.add(new OrderItem(product, quantity));
        recalculateTotal();
    }
    
    public boolean canBeCanceled() {
        return status == OrderStatus.PENDING || status == OrderStatus.CONFIRMED;
    }
}
```

### Challenge 2: Layer Skip Temptation
**Problem:** Direct communication between non-adjacent layers for performance or convenience.

**Solution:** Implement strict layer enforcement through dependency injection and architectural testing tools like ArchUnit.

### Challenge 3: Data Transfer Object Proliferation
**Problem:** Multiple DTOs for the same entity across layers.

**Solution:** Use mapping libraries like MapStruct or implement a shared DTO strategy with clear versioning.

## Handling Cross-Cutting Concerns

### 1. Aspect-Oriented Programming (AOP)
```java
@Aspect
public class SecurityAspect {
    @Around("@annotation(Secured)")
    public Object checkSecurity(ProceedingJoinPoint joinPoint) throws Throwable {
        // Security logic here
        return joinPoint.proceed();
    }
}
```

### 2. Decorator Pattern
```java
public class CachingOrderService implements OrderService {
    private final OrderService delegate;
    private final CacheManager cache;
    
    @Override
    public Order getOrder(Long id) {
        return cache.get(id, () -> delegate.getOrder(id));
    }
}
```

### 3. Infrastructure Layer
Add a dedicated infrastructure layer to handle cross-cutting concerns like logging, monitoring, and configuration.

## Design Patterns by Layer

### Presentation Layer Patterns:
- **Model-View-Controller (MVC)** - Separates presentation logic
- **Model-View-Presenter (MVP)** - Better testability than MVC
- **Front Controller** - Centralized request handling

### Business Layer Patterns:
- **Service Layer** - Encapsulates business operations
- **Domain Model** - Rich objects with business behavior
- **Transaction Script** - Simple procedural approach for straightforward logic

### Data Access Layer Patterns:
- **Repository Pattern** - Encapsulates data access logic
- **Data Mapper** - Separates domain objects from database schema
- **Active Record** - Domain objects handle their own persistence
- **Unit of Work** - Maintains list of objects affected by transactions

## Evolution: Traditional N-Tier to Modern Layered Approaches

### Traditional N-Tier (Early 2000s)
- Physical separation across network boundaries
- Each tier often on separate servers
- Heavy reliance on stored procedures
- Limited flexibility due to network constraints

### Modern Layered Architecture
- Logical separation within the same application
- Microservices influence on layer independence
- Domain-driven design integration
- Cloud-native considerations (stateless layers, horizontal scaling)

### Current Trends
- **Hybrid Approaches** - Combining layered with hexagonal concepts
- **Event-Driven Layers** - Asynchronous communication between layers
- **CQRS Integration** - Separate read and write layer implementations

## Key Interview Preparation Points

### E-Commerce Platform Design
```
Presentation Layer: Web controllers, mobile APIs, admin dashboards
Business Layer: Order processing, inventory management, pricing engines
Data Access Layer: Product repositories, user repositories, order DAOs
Database Layer: Customer data, product catalog, transaction records
```

### Strict vs Relaxed Layering
- **Strict:** Each layer only communicates with the layer directly below
- **Relaxed:** Layers can skip intermediate layers for performance (with careful consideration)

### Preventing Anemic Domain Model
- Push business logic into domain entities
- Use value objects for complex data types
- Implement domain services for multi-entity operations
- Keep application services thin - they orchestrate, don't implement business rules

## Conclusion

Layered architecture remains a fundamental pattern for enterprise applications, providing clear structure and separation of concerns. While it has limitations, understanding when and how to apply it effectively is crucial for software architects. The key is balancing the benefits of clear separation with the practical needs of performance and maintainability, often leading to hybrid approaches that combine layered architecture with modern patterns like hexagonal architecture or event-driven design.

Success with layered architecture comes from strict adherence to its principles while remaining pragmatic about real-world constraints and evolution toward more sophisticated architectural patterns as applications grow in complexity.

---
I'll provide you with a comprehensive overview of layered architecture from a solution-oriented architect's perspective, perfect for your technical interview preparation.This comprehensive guide covers all the aspects you requested for your .NET technical interview preparation. The content is structured from a solution-oriented architect's perspective, emphasizing practical implementation strategies and real-world challenges.

Key highlights of this guide:

**For Interview Success:**
- **Business Focus:** Each section connects technical concepts to business value
- **Problem-Solution Approach:** Challenges are paired with concrete solutions
- **Modern Context:** Covers evolution from traditional to modern approaches
- **Practical Examples:** Simple, focused code snippets that demonstrate concepts

**Architecture Decision Framework:**
The guide helps you articulate when to choose layered architecture versus alternatives, which is crucial for senior architect roles. You can discuss trade-offs intelligently and show understanding of the broader architectural landscape.

**Real-world Insights:**
The challenges and solutions section prepares you for follow-up questions about implementation difficulties you've faced, demonstrating hands-on experience with enterprise-scale applications.

# Layered Architecture Guide for .NET Architects

## Definition and Core Principles

**Layered Architecture** is a structural pattern that organizes code into horizontal layers, where each layer has specific responsibilities and communicates only with adjacent layers. It's based on the principle of **separation of concerns** and follows a top-down dependency flow.

### Core Principles:
1. **Unidirectional Dependencies** - Upper layers depend on lower layers, never the reverse
2. **Layer Abstraction** - Each layer provides services to the layer above through well-defined interfaces
3. **Encapsulation** - Internal implementation details are hidden within each layer
4. **Single Responsibility** - Each layer has one primary concern

## Common Layers and Responsibilities

### 1. Presentation Layer (UI/Controllers)
**Responsibility:** Handle user interaction and presentation logic
- Receive user input and display output
- Format data for presentation
- Handle HTTP requests/responses
- Manage user session and authentication tokens

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductController : ControllerBase
{
    private readonly IProductService _productService;
    
    public ProductController(IProductService productService)
    {
        _productService = productService;
    }
    
    [HttpGet("{id}")]
    public async Task<ActionResult<ProductDto>> GetProduct(int id)
    {
        var product = await _productService.GetProductAsync(id);
        return Ok(product);
    }
}
```

### 2. Business/Application Layer (Services)
**Responsibility:** Implement business logic and orchestrate operations
- Enforce business rules and validations
- Coordinate between different domain entities
- Handle workflow and process management
- Implement use cases and business operations

```csharp
public class ProductService : IProductService
{
    private readonly IProductRepository _repository;
    private readonly IInventoryService _inventoryService;
    
    public async Task<ProductDto> GetProductAsync(int id)
    {
        var product = await _repository.GetByIdAsync(id);
        if (product == null) 
            throw new ProductNotFoundException(id);
            
        // Business logic: Check inventory
        var inventory = await _inventoryService.GetInventoryAsync(id);
        
        return new ProductDto 
        { 
            Id = product.Id, 
            Name = product.Name,
            IsAvailable = inventory.Quantity > 0 
        };
    }
}
```

### 3. Data Access Layer (Repositories/DAL)
**Responsibility:** Handle data persistence and retrieval
- Abstract database operations
- Implement data access patterns
- Handle database connections and transactions
- Convert between domain models and data models

```csharp
public class ProductRepository : IProductRepository
{
    private readonly ApplicationDbContext _context;
    
    public async Task<Product> GetByIdAsync(int id)
    {
        return await _context.Products
            .FirstOrDefaultAsync(p => p.Id == id);
    }
    
    public async Task<Product> AddAsync(Product product)
    {
        _context.Products.Add(product);
        await _context.SaveChangesAsync();
        return product;
    }
}
```

### 4. Database Layer
**Responsibility:** Physical data storage
- Database schema and tables
- Stored procedures and functions
- Database-specific optimizations
- Data integrity constraints

## Advantages

### 1. Separation of Concerns
Each layer focuses on a specific aspect of the application, making code more organized and understandable.

### 2. Maintainability
Changes in one layer have minimal impact on others, making maintenance easier and reducing regression risks.

### 3. Testability
Each layer can be independently unit tested using mocks and stubs for dependencies.

```csharp
[Test]
public async Task GetProduct_ValidId_ReturnsProduct()
{
    // Arrange
    var mockRepo = new Mock<IProductRepository>();
    var mockInventory = new Mock<IInventoryService>();
    mockRepo.Setup(r => r.GetByIdAsync(1)).ReturnsAsync(new Product { Id = 1, Name = "Test" });
    
    var service = new ProductService(mockRepo.Object, mockInventory.Object);
    
    // Act
    var result = await service.GetProductAsync(1);
    
    // Assert
    Assert.That(result.Name, Is.EqualTo("Test"));
}
```

### 4. Team Specialization
Different teams can work on different layers simultaneously - UI team, business logic team, database team.

## Disadvantages

### 1. Performance Overhead
Multiple layer calls can introduce latency, especially in distributed scenarios.

**Solution:** Implement caching strategies and consider async patterns for I/O operations.

### 2. Potential for Tight Coupling
Layers can become tightly coupled if not properly abstracted.

**Solution:** Use dependency injection and interface-based programming.

### 3. Complexity in Cross-cutting Concerns
Features like logging, security, and caching span multiple layers.

**Solution:** Implement using Aspect-Oriented Programming (AOP) or middleware patterns.

## When to Use Layered Architecture vs Alternatives

### Use Layered Architecture When:
- Building traditional CRUD applications
- Team has varied skill levels
- Clear separation between UI, business logic, and data access is needed
- Compliance requirements demand clear audit trails

### Consider Alternatives When:
- **Hexagonal Architecture:** Need better testability and independence from external systems
- **Clean Architecture:** Require strict dependency inversion and domain-centric design
- **Microservices:** Need to scale different parts independently
- **Event-Driven:** Handling complex workflows with asynchronous processing

## Real-world Implementation Challenges and Solutions

### Challenge 1: Circular Dependencies
**Problem:** Business layer needs data layer, but data layer needs domain models from business layer.

**Solution:** Create a separate Domain/Models project that both layers reference.

### Challenge 2: Performance in Deep Call Stacks
**Problem:** Multiple layer calls create performance bottlenecks.

**Solution:** Implement strategic caching and use async/await patterns.

```csharp
public class CachedProductService : IProductService
{
    private readonly IProductService _productService;
    private readonly IMemoryCache _cache;
    
    public async Task<ProductDto> GetProductAsync(int id)
    {
        var cacheKey = $"product_{id}";
        if (_cache.TryGetValue(cacheKey, out ProductDto cachedProduct))
            return cachedProduct;
            
        var product = await _productService.GetProductAsync(id);
        _cache.Set(cacheKey, product, TimeSpan.FromMinutes(5));
        return product;
    }
}
```

### Challenge 3: Transaction Management Across Layers
**Problem:** Managing database transactions that span multiple service calls.

**Solution:** Implement Unit of Work pattern or use distributed transaction coordinators.

## Handling Cross-cutting Concerns

### 1. Logging
Use structured logging with dependency injection:

```csharp
public class ProductService : IProductService
{
    private readonly ILogger<ProductService> _logger;
    
    public async Task<ProductDto> GetProductAsync(int id)
    {
        _logger.LogInformation("Fetching product with ID: {ProductId}", id);
        // Implementation
    }
}
```

### 2. Security
Implement at multiple layers:
- **Presentation:** Authentication and authorization
- **Business:** Permission checking and data filtering
- **Data:** Row-level security and encryption

### 3. Caching
Implement using decorator pattern or middleware:

```csharp
// In Startup.cs
services.AddScoped<ProductService>();
services.Decorate<IProductService, CachedProductService>();
```

## Design Patterns by Layer

### Presentation Layer
- **MVC/MVP Pattern** - Separate presentation logic
- **DTO Pattern** - Data transfer objects for API responses
- **Command Pattern** - Handle user actions

### Business Layer
- **Service Layer Pattern** - Encapsulate business operations
- **Domain Model Pattern** - Rich domain objects with behavior
- **Specification Pattern** - Encapsulate business rules

### Data Access Layer
- **Repository Pattern** - Abstract data access
- **Unit of Work Pattern** - Manage transactions
- **Data Mapper Pattern** - Convert between domain and data models

## Evolution: Traditional N-tier to Modern Layered Approaches

### Traditional N-tier (Legacy)
- Tight coupling between layers
- Database-centric design
- Limited testability
- Monolithic deployment

### Modern Layered Architecture
- Interface-based abstraction
- Domain-centric design
- Dependency injection
- Microservice-ready structure

### Key Evolutionary Changes:

1. **From Concrete Dependencies to Interfaces**
```csharp
// Traditional
public class ProductService
{
    private ProductRepository _repo = new ProductRepository();
}

// Modern
public class ProductService
{
    private readonly IProductRepository _repo;
    public ProductService(IProductRepository repo) => _repo = repo;
}
```

2. **From Database-First to Domain-First**
- Traditional: Design database schema first, generate entities
- Modern: Design domain models first, map to database

3. **From Synchronous to Asynchronous**
```csharp
// Traditional
public Product GetProduct(int id) => _repository.GetById(id);

// Modern
public async Task<Product> GetProductAsync(int id) => 
    await _repository.GetByIdAsync(id);
```

## Best Practices for Modern Implementation

1. **Use Dependency Injection Container**
2. **Implement Async/Await Throughout**
3. **Apply SOLID Principles**
4. **Use Generic Repository with Specifications**
5. **Implement Proper Error Handling and Logging**
6. **Consider CQRS for Complex Read/Write Scenarios**

## Interview Tips

When discussing layered architecture in interviews:

1. **Emphasize the "Why"** - Explain business benefits, not just technical features
2. **Discuss Trade-offs** - Show you understand when NOT to use it
3. **Provide Real Examples** - Reference projects where you've implemented it
4. **Address Modern Concerns** - Microservices, cloud-native considerations
5. **Show Evolution Understanding** - How layered architecture fits in modern architectures

Remember: Layered architecture is not just about organizing code - it's about creating maintainable, scalable systems that support business objectives while enabling team productivity and system evolution.

I'll provide a comprehensive elaboration on these architectural patterns, focusing on how they compare to layered architecture and when to choose each approach.This comprehensive guide covers all five architectural patterns you requested, with detailed explanations, practical code examples, and decision frameworks. Here are the key takeaways for your interview preparation:

**For Senior Architect Interviews:**

1. **Pattern Recognition**: Understand that each architecture solves different problems. Be ready to explain the "why" behind choosing one over another.

2. **Trade-off Analysis**: Every architecture has benefits and drawbacks. Show you understand the cost-benefit analysis for each pattern.

3. **Real-world Context**: Be prepared to discuss how these patterns work in practice, including implementation challenges and hybrid approaches.

4. **Evolution Path**: Understand how applications can evolve from one architecture to another (e.g., starting with layered architecture and evolving to microservices).

**Key Interview Points:**

- **Hexagonal**: Emphasize ports/adapters and testability benefits
- **Clean**: Focus on dependency inversion and business rule protection
- **Microservices**: Discuss organizational implications and distributed system challenges
- **Event-Driven**: Highlight scalability and eventual consistency concepts
- **Onion**: Emphasize domain-centric approach and business rule encapsulation

**Common Interview Questions You'll Be Ready For:**
- "When would you choose microservices over a monolith?"
- "How do you handle cross-cutting concerns in Clean Architecture?"
- "What are the challenges of implementing event-driven architecture?"
- "How does Onion Architecture differ from traditional layered architecture?"

The comparison table and decision framework will help you articulate architectural choices confidently during technical discussions.

# Advanced Architecture Patterns for .NET Architects

## 1. Hexagonal Architecture (Ports and Adapters)

### Definition and Core Principles
Hexagonal Architecture, created by Alistair Cockburn, isolates the core business logic from external concerns through **ports** (interfaces) and **adapters** (implementations). The application core sits at the center, surrounded by ports that define contracts for interaction.

### Key Components:
- **Application Core:** Contains business logic and domain models
- **Ports:** Interfaces that define how the core communicates with the outside world
- **Adapters:** Implementations that connect ports to external systems
- **Primary/Driving Adapters:** Initiate actions (UI, API controllers, tests)
- **Secondary/Driven Adapters:** Respond to actions (databases, external APIs, message queues)

### Structure Example:
```csharp
// Core Domain
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    
    public void ApplyDiscount(decimal percentage)
    {
        if (percentage < 0 || percentage > 100)
            throw new InvalidOperationException("Invalid discount percentage");
        Price *= (1 - percentage / 100);
    }
}

// Primary Port (Application Service Interface)
public interface IProductService
{
    Task<Product> GetProductAsync(int id);
    Task<Product> CreateProductAsync(CreateProductCommand command);
}

// Secondary Port (Repository Interface)
public interface IProductRepository
{
    Task<Product> GetByIdAsync(int id);
    Task<Product> SaveAsync(Product product);
}

// Application Core (Service Implementation)
public class ProductService : IProductService
{
    private readonly IProductRepository _repository;
    private readonly IEmailNotificationService _emailService;
    
    public ProductService(IProductRepository repository, IEmailNotificationService emailService)
    {
        _repository = repository;
        _emailService = emailService;
    }
    
    public async Task<Product> CreateProductAsync(CreateProductCommand command)
    {
        var product = new Product 
        { 
            Name = command.Name, 
            Price = command.Price 
        };
        
        var savedProduct = await _repository.SaveAsync(product);
        await _emailService.SendProductCreatedNotificationAsync(savedProduct);
        
        return savedProduct;
    }
}

// Primary Adapter (API Controller)
[ApiController]
[Route("api/[controller]")]
public class ProductController : ControllerBase
{
    private readonly IProductService _productService;
    
    public ProductController(IProductService productService)
    {
        _productService = productService;
    }
    
    [HttpPost]
    public async Task<IActionResult> CreateProduct(CreateProductRequest request)
    {
        var command = new CreateProductCommand { Name = request.Name, Price = request.Price };
        var product = await _productService.CreateProductAsync(command);
        return CreatedAtAction(nameof(GetProduct), new { id = product.Id }, product);
    }
}

// Secondary Adapter (Database Repository)
public class SqlProductRepository : IProductRepository
{
    private readonly ApplicationDbContext _context;
    
    public SqlProductRepository(ApplicationDbContext context)
    {
        _context = context;
    }
    
    public async Task<Product> SaveAsync(Product product)
    {
        _context.Products.Add(product);
        await _context.SaveChangesAsync();
        return product;
    }
}
```

### Advantages:
- **Testability:** Easy to mock external dependencies
- **Technology Independence:** Core logic doesn't depend on frameworks
- **Flexibility:** Easy to swap implementations (SQL to NoSQL, REST to GraphQL)
- **Maintainability:** Clear separation between business logic and infrastructure

### When to Use:
- Applications with complex business logic
- Need for high testability and flexibility
- Multiple external integrations
- Long-term maintenance requirements

## 2. Clean Architecture (Uncle Bob's Architecture)

### Definition and Core Principles
Clean Architecture, proposed by Robert Martin (Uncle Bob), organizes code in concentric circles with dependencies pointing inward. The inner circles contain business logic, while outer circles contain implementation details.

### Layers (from inside out):
1. **Entities:** Enterprise business rules and core domain objects
2. **Use Cases/Interactors:** Application-specific business rules
3. **Interface Adapters:** Convert data between use cases and external systems
4. **Frameworks & Drivers:** External tools, databases, web frameworks

### Structure Example:
```csharp
// 1. Entities (Innermost layer)
public class Order
{
    public int Id { get; private set; }
    public DateTime OrderDate { get; private set; }
    public decimal TotalAmount { get; private set; }
    public OrderStatus Status { get; private set; }
    private readonly List<OrderItem> _items = new();
    
    public IReadOnlyList<OrderItem> Items => _items.AsReadOnly();
    
    public void AddItem(Product product, int quantity)
    {
        if (Status != OrderStatus.Draft)
            throw new InvalidOperationException("Cannot add items to non-draft order");
            
        _items.Add(new OrderItem(product, quantity));
        RecalculateTotal();
    }
    
    public void Submit()
    {
        if (!_items.Any())
            throw new InvalidOperationException("Cannot submit empty order");
            
        Status = OrderStatus.Submitted;
        OrderDate = DateTime.UtcNow;
    }
    
    private void RecalculateTotal()
    {
        TotalAmount = _items.Sum(item => item.Price * item.Quantity);
    }
}

// 2. Use Cases (Application Business Rules)
public interface ICreateOrderUseCase
{
    Task<OrderDto> ExecuteAsync(CreateOrderRequest request);
}

public class CreateOrderUseCase : ICreateOrderUseCase
{
    private readonly IOrderRepository _orderRepository;
    private readonly IProductRepository _productRepository;
    private readonly IEmailService _emailService;
    
    public CreateOrderUseCase(
        IOrderRepository orderRepository,
        IProductRepository productRepository,
        IEmailService emailService)
    {
        _orderRepository = orderRepository;
        _productRepository = productRepository;
        _emailService = emailService;
    }
    
    public async Task<OrderDto> ExecuteAsync(CreateOrderRequest request)
    {
        var order = new Order();
        
        foreach (var item in request.Items)
        {
            var product = await _productRepository.GetByIdAsync(item.ProductId);
            if (product == null)
                throw new ProductNotFoundException(item.ProductId);
                
            order.AddItem(product, item.Quantity);
        }
        
        order.Submit();
        await _orderRepository.SaveAsync(order);
        await _emailService.SendOrderConfirmationAsync(order);
        
        return new OrderDto 
        { 
            Id = order.Id, 
            TotalAmount = order.TotalAmount,
            Status = order.Status.ToString()
        };
    }
}

// 3. Interface Adapters (Controllers, Presenters, Gateways)
[ApiController]
[Route("api/[controller]")]
public class OrderController : ControllerBase
{
    private readonly ICreateOrderUseCase _createOrderUseCase;
    
    public OrderController(ICreateOrderUseCase createOrderUseCase)
    {
        _createOrderUseCase = createOrderUseCase;
    }
    
    [HttpPost]
    public async Task<ActionResult<OrderResponse>> CreateOrder(CreateOrderApiRequest request)
    {
        var useCaseRequest = new CreateOrderRequest
        {
            Items = request.Items.Select(i => new CreateOrderItemRequest
            {
                ProductId = i.ProductId,
                Quantity = i.Quantity
            }).ToList()
        };
        
        var result = await _createOrderUseCase.ExecuteAsync(useCaseRequest);
        
        return CreatedAtAction(nameof(GetOrder), 
            new { id = result.Id }, 
            new OrderResponse { Id = result.Id, Total = result.TotalAmount });
    }
}
```

### Advantages:
- **Independence:** Framework, database, and UI agnostic
- **Testability:** Business logic can be tested without external dependencies
- **Flexibility:** Easy to change external tools without affecting business rules
- **Maintainability:** Clear dependency direction makes code easier to understand

### When to Use:
- Complex business domains
- Long-lived applications
- Multiple delivery mechanisms (web, mobile, desktop)
- Strong requirement for testability and maintainability

## 3. Microservices Architecture

### Definition and Core Principles
Microservices architecture decomposes an application into small, independent services that communicate over network protocols. Each service owns its data and business capability.

### Key Characteristics:
- **Single Responsibility:** Each service handles one business capability
- **Decentralized:** Independent deployment, scaling, and technology choices
- **Fault Isolation:** Failure in one service doesn't bring down the entire system
- **Data Ownership:** Each service manages its own database

### Implementation Example:
```csharp
// Product Service
[ApiController]
[Route("api/[controller]")]
public class ProductController : ControllerBase
{
    private readonly IProductService _productService;
    
    [HttpGet("{id}")]
    public async Task<ActionResult<Product>> GetProduct(int id)
    {
        var product = await _productService.GetByIdAsync(id);
        return product != null ? Ok(product) : NotFound();
    }
}

// Order Service
[ApiController]
[Route("api/[controller]")]
public class OrderController : ControllerBase
{
    private readonly IOrderService _orderService;
    private readonly IProductServiceClient _productClient;
    
    [HttpPost]
    public async Task<ActionResult<Order>> CreateOrder(CreateOrderRequest request)
    {
        // Validate products exist via Product Service
        var productIds = request.Items.Select(i => i.ProductId).ToList();
        var products = await _productClient.GetProductsByIdsAsync(productIds);
        
        if (products.Count != productIds.Count)
            return BadRequest("Some products not found");
        
        var order = await _orderService.CreateOrderAsync(request, products);
        
        // Publish order created event
        await _messageBus.PublishAsync(new OrderCreatedEvent 
        { 
            OrderId = order.Id,
            CustomerId = order.CustomerId,
            TotalAmount = order.TotalAmount
        });
        
        return CreatedAtAction(nameof(GetOrder), new { id = order.Id }, order);
    }
}

// Inter-service communication
public interface IProductServiceClient
{
    Task<List<Product>> GetProductsByIdsAsync(List<int> productIds);
}

public class ProductServiceClient : IProductServiceClient
{
    private readonly HttpClient _httpClient;
    private readonly IConfiguration _configuration;
    
    public ProductServiceClient(HttpClient httpClient, IConfiguration configuration)
    {
        _httpClient = httpClient;
        _configuration = configuration;
    }
    
    public async Task<List<Product>> GetProductsByIdsAsync(List<int> productIds)
    {
        var baseUrl = _configuration["Services:ProductService:BaseUrl"];
        var idsQuery = string.Join(",", productIds);
        var response = await _httpClient.GetAsync($"{baseUrl}/api/products/batch?ids={idsQuery}");
        
        if (response.IsSuccessStatusCode)
        {
            var json = await response.Content.ReadAsStringAsync();
            return JsonSerializer.Deserialize<List<Product>>(json);
        }
        
        throw new ProductServiceException("Failed to retrieve products");
    }
}

// Service Registration (Program.cs)
builder.Services.AddHttpClient<IProductServiceClient, ProductServiceClient>();
builder.Services.AddScoped<IOrderService, OrderService>();
```

### Advantages:
- **Independent Scaling:** Scale services based on individual demand
- **Technology Diversity:** Different services can use different technologies
- **Team Autonomy:** Teams can work independently on different services
- **Fault Isolation:** Issues in one service don't affect others
- **Continuous Deployment:** Deploy services independently

### Challenges:
- **Network Complexity:** Inter-service communication overhead
- **Data Consistency:** Managing transactions across services
- **Service Discovery:** Finding and connecting to services
- **Monitoring:** Distributed tracing and logging complexity

### When to Use:
- Large, complex applications
- Multiple development teams
- Different scaling requirements for different features
- Organizational readiness for distributed systems complexity

## 4. Event-Driven Architecture

### Definition and Core Principles
Event-Driven Architecture (EDA) uses events as the primary mechanism for communication between components. Systems react to events asynchronously, promoting loose coupling and scalability.

### Key Components:
- **Event Producers:** Generate events when state changes occur
- **Event Consumers:** React to events and perform actions
- **Event Bus/Broker:** Routes events from producers to consumers
- **Event Store:** Persists events for replay and auditing

### Implementation Example:
```csharp
// Domain Event
public abstract class DomainEvent
{
    public Guid Id { get; } = Guid.NewGuid();
    public DateTime OccurredAt { get; } = DateTime.UtcNow;
}

public class OrderCreatedEvent : DomainEvent
{
    public int OrderId { get; set; }
    public int CustomerId { get; set; }
    public decimal TotalAmount { get; set; }
    public List<OrderItem> Items { get; set; } = new();
}

public class OrderCancelledEvent : DomainEvent
{
    public int OrderId { get; set; }
    public string CancellationReason { get; set; }
}

// Event Bus Interface
public interface IEventBus
{
    Task PublishAsync<T>(T @event) where T : DomainEvent;
    void Subscribe<T, TH>() where T : DomainEvent where TH : IEventHandler<T>;
}

// Event Handler Interface
public interface IEventHandler<in T> where T : DomainEvent
{
    Task HandleAsync(T @event);
}

// Specific Event Handlers
public class SendOrderConfirmationHandler : IEventHandler<OrderCreatedEvent>
{
    private readonly IEmailService _emailService;
    private readonly ICustomerRepository _customerRepository;
    
    public SendOrderConfirmationHandler(IEmailService emailService, ICustomerRepository customerRepository)
    {
        _emailService = emailService;
        _customerRepository = customerRepository;
    }
    
    public async Task HandleAsync(OrderCreatedEvent orderCreatedEvent)
    {
        var customer = await _customerRepository.GetByIdAsync(orderCreatedEvent.CustomerId);
        
        await _emailService.SendOrderConfirmationAsync(new EmailRequest
        {
            To = customer.Email,
            Subject = "Order Confirmation",
            OrderId = orderCreatedEvent.OrderId,
            TotalAmount = orderCreatedEvent.TotalAmount
        });
    }
}

public class UpdateInventoryHandler : IEventHandler<OrderCreatedEvent>
{
    private readonly IInventoryService _inventoryService;
    
    public UpdateInventoryHandler(IInventoryService inventoryService)
    {
        _inventoryService = inventoryService;
    }
    
    public async Task HandleAsync(OrderCreatedEvent orderCreatedEvent)
    {
        foreach (var item in orderCreatedEvent.Items)
        {
            await _inventoryService.ReserveStockAsync(item.ProductId, item.Quantity);
        }
    }
}

// Event Sourcing Example
public class OrderAggregate
{
    private readonly List<DomainEvent> _events = new();
    public int Id { get; private set; }
    public OrderStatus Status { get; private set; }
    public decimal TotalAmount { get; private set; }
    
    public IReadOnlyList<DomainEvent> Events => _events.AsReadOnly();
    
    public void CreateOrder(int customerId, List<OrderItem> items)
    {
        var totalAmount = items.Sum(i => i.Price * i.Quantity);
        
        var @event = new OrderCreatedEvent
        {
            OrderId = Id,
            CustomerId = customerId,
            TotalAmount = totalAmount,
            Items = items
        };
        
        ApplyEvent(@event);
    }
    
    public void CancelOrder(string reason)
    {
        if (Status == OrderStatus.Delivered)
            throw new InvalidOperationException("Cannot cancel delivered order");
        
        var @event = new OrderCancelledEvent
        {
            OrderId = Id,
            CancellationReason = reason
        };
        
        ApplyEvent(@event);
    }
    
    private void ApplyEvent(DomainEvent @event)
    {
        _events.Add(@event);
        
        switch (@event)
        {
            case OrderCreatedEvent orderCreated:
                Status = OrderStatus.Created;
                TotalAmount = orderCreated.TotalAmount;
                break;
            case OrderCancelledEvent:
                Status = OrderStatus.Cancelled;
                break;
        }
    }
}

// Service Integration
public class OrderService
{
    private readonly IEventBus _eventBus;
    private readonly IOrderRepository _orderRepository;
    
    public async Task<Order> CreateOrderAsync(CreateOrderRequest request)
    {
        var aggregate = new OrderAggregate();
        aggregate.CreateOrder(request.CustomerId, request.Items);
        
        // Save to repository
        await _orderRepository.SaveAsync(aggregate);
        
        // Publish events
        foreach (var @event in aggregate.Events)
        {
            await _eventBus.PublishAsync(@event);
        }
        
        return new Order { /* map from aggregate */ };
    }
}
```

### Advantages:
- **Loose Coupling:** Components don't need to know about each other
- **Scalability:** Asynchronous processing improves performance
- **Extensibility:** Easy to add new event handlers
- **Auditability:** Event sourcing provides complete audit trail
- **Resilience:** Eventual consistency and retry mechanisms

### When to Use:
- Systems requiring high scalability and performance
- Complex business workflows with multiple steps
- Integration with multiple external systems
- Need for audit trails and event replay
- Microservices architectures

## 5. Onion Architecture

### Definition and Core Principles
Onion Architecture, created by Jeffrey Palermo, is similar to Clean Architecture but emphasizes the domain model at the core, surrounded by concentric layers. Dependencies point inward, and the domain layer has no dependencies on external concerns.

### Layers (from inside out):
1. **Domain Model:** Core business entities and value objects
2. **Domain Services:** Complex business logic that doesn't belong to entities
3. **Application Services:** Use cases and application-specific logic
4. **Infrastructure:** External concerns (database, web services, file system)

### Structure Example:
```csharp
// 1. Domain Model (Core)
public class Customer
{
    public int Id { get; private set; }
    public string Email { get; private set; }
    public CustomerStatus Status { get; private set; }
    public DateTime CreatedAt { get; private set; }
    
    private readonly List<Order> _orders = new();
    public IReadOnlyList<Order> Orders => _orders.AsReadOnly();
    
    public Customer(string email)
    {
        if (string.IsNullOrWhiteSpace(email))
            throw new ArgumentException("Email cannot be empty");
            
        Email = email;
        Status = CustomerStatus.Active;
        CreatedAt = DateTime.UtcNow;
    }
    
    public void PlaceOrder(Order order)
    {
        if (Status != CustomerStatus.Active)
            throw new InvalidOperationException("Inactive customer cannot place orders");
            
        _orders.Add(order);
    }
    
    public void Deactivate()
    {
        Status = CustomerStatus.Inactive;
    }
    
    public bool IsEligibleForPremium()
    {
        return _orders.Count >= 10 && 
               _orders.Sum(o => o.TotalAmount) >= 1000m;
    }
}

// 2. Domain Services
public interface ICustomerDomainService
{
    Task<bool> IsEmailUniqueAsync(string email);
    decimal CalculateCustomerLifetimeValue(Customer customer);
}

public class CustomerDomainService : ICustomerDomainService
{
    private readonly ICustomerRepository _customerRepository;
    
    public CustomerDomainService(ICustomerRepository customerRepository)
    {
        _customerRepository = customerRepository;
    }
    
    public async Task<bool> IsEmailUniqueAsync(string email)
    {
        var existingCustomer = await _customerRepository.GetByEmailAsync(email);
        return existingCustomer == null;
    }
    
    public decimal CalculateCustomerLifetimeValue(Customer customer)
    {
        var monthsSinceRegistration = (DateTime.UtcNow - customer.CreatedAt).Days / 30.0;
        var totalSpent = customer.Orders.Sum(o => o.TotalAmount);
        
        return monthsSinceRegistration > 0 
            ? totalSpent / (decimal)monthsSinceRegistration * 12 
            : 0;
    }
}

// 3. Application Services
public interface ICustomerApplicationService
{
    Task<CustomerDto> CreateCustomerAsync(CreateCustomerCommand command);
    Task<CustomerDto> GetCustomerAsync(int id);
    Task<List<CustomerDto>> GetPremiumEligibleCustomersAsync();
}

public class CustomerApplicationService : ICustomerApplicationService
{
    private readonly ICustomerRepository _customerRepository;
    private readonly ICustomerDomainService _domainService;
    private readonly IEventBus _eventBus;
    
    public CustomerApplicationService(
        ICustomerRepository customerRepository,
        ICustomerDomainService domainService,
        IEventBus eventBus)
    {
        _customerRepository = customerRepository;
        _domainService = domainService;
        _eventBus = eventBus;
    }
    
    public async Task<CustomerDto> CreateCustomerAsync(CreateCustomerCommand command)
    {
        if (!await _domainService.IsEmailUniqueAsync(command.Email))
            throw new DuplicateEmailException(command.Email);
        
        var customer = new Customer(command.Email);
        await _customerRepository.SaveAsync(customer);
        
        await _eventBus.PublishAsync(new CustomerCreatedEvent 
        { 
            CustomerId = customer.Id,
            Email = customer.Email 
        });
        
        return new CustomerDto 
        {
            Id = customer.Id,
            Email = customer.Email,
            Status = customer.Status.ToString()
        };
    }
    
    public async Task<List<CustomerDto>> GetPremiumEligibleCustomersAsync()
    {
        var allCustomers = await _customerRepository.GetAllActiveAsync();
        var eligibleCustomers = allCustomers.Where(c => c.IsEligibleForPremium()).ToList();
        
        return eligibleCustomers.Select(c => new CustomerDto
        {
            Id = c.Id,
            Email = c.Email,
            Status = c.Status.ToString(),
            LifetimeValue = _domainService.CalculateCustomerLifetimeValue(c)
        }).ToList();
    }
}

// 4. Infrastructure Layer
public class SqlCustomerRepository : ICustomerRepository
{
    private readonly ApplicationDbContext _context;
    
    public SqlCustomerRepository(ApplicationDbContext context)
    {
        _context = context;
    }
    
    public async Task<Customer> GetByIdAsync(int id)
    {
        return await _context.Customers
            .Include(c => c.Orders)
            .FirstOrDefaultAsync(c => c.Id == id);
    }
    
    public async Task<Customer> SaveAsync(Customer customer)
    {
        if (customer.Id == 0)
            _context.Customers.Add(customer);
        else
            _context.Customers.Update(customer);
            
        await _context.SaveChangesAsync();
        return customer;
    }
}

// Dependency Injection Setup
public void ConfigureServices(IServiceCollection services)
{
    // Infrastructure
    services.AddDbContext<ApplicationDbContext>(options => 
        options.UseSqlServer(connectionString));
    services.AddScoped<ICustomerRepository, SqlCustomerRepository>();
    
    // Domain Services
    services.AddScoped<ICustomerDomainService, CustomerDomainService>();
    
    // Application Services
    services.AddScoped<ICustomerApplicationService, CustomerApplicationService>();
}
```

### Advantages:
- **Domain-Centric:** Business logic is the core focus
- **Testability:** Domain logic can be tested in isolation
- **Flexibility:** Easy to change infrastructure without affecting business rules
- **Clear Dependencies:** Dependency direction is always inward

### When to Use:
- Domain-rich applications with complex business rules
- Applications requiring high testability
- Long-lived applications with evolving requirements
- Team with strong domain modeling expertise

## Comparison Summary

| Architecture | Complexity | Testability | Flexibility | Performance | Best For |
|--------------|------------|-------------|-------------|-------------|-----------|
| **Layered** | Low | Medium | Low | High | Simple CRUD, Traditional apps |
| **Hexagonal** | Medium | High | High | Medium | Complex business logic, High testability needs |
| **Clean** | High | Very High | Very High | Medium | Enterprise apps, Long-term maintenance |
| **Microservices** | Very High | Medium | Very High | Variable | Large teams, Independent scaling |
| **Event-Driven** | High | Medium | High | High | High scalability, Async workflows |
| **Onion** | Medium-High | High | High | Medium | Domain-rich applications |

## Decision Framework

### Choose **Layered Architecture** when:
- Building simple CRUD applications
- Team is new to software architecture
- Time-to-market is critical
- Performance is the primary concern

### Choose **Hexagonal Architecture** when:
- Need high testability
- Multiple external integrations
- Technology flexibility is important
- Medium complexity business logic

### Choose **Clean Architecture** when:
- Building enterprise-grade applications
- Long-term maintenance is critical
- Multiple UI channels (web, mobile, desktop)
- Complex business rules

### Choose **Microservices** when:
- Large development teams
- Different scaling requirements
- Independent deployment needs
- Organizational alignment with service boundaries

### Choose **Event-Driven Architecture** when:
- High scalability requirements
- Complex workflows and integrations
- Need for audit trails and event replay
- Asynchronous processing is beneficial

### Choose **Onion Architecture** when:
- Domain-driven design approach
- Complex business domain
- Need for domain model isolation
- Strong domain expertise in the team

Remember: Architecture decisions should align with business requirements, team capabilities, and organizational constraints. Often, hybrid approaches combining multiple patterns work best for real-world applications.

---

20. **Clean Architecture**
    - How do you implement Clean Architecture in .NET Core Web API?
    - Show dependency flow and folder structure.
    - How do you keep business logic independent of frameworks?

---






---

### **API Design Patterns**
21. **MVC Pattern**
    - How does Web API implement MVC pattern differently from traditional MVC?
    - Show controller, model, and view separation in API context.

---
# MVC Pattern: Solution-Oriented Architect's Guide for .NET Professionals

## Core MVC Pattern: Separation and Responsibilities

### Model-View-Controller Fundamentals

The MVC pattern divides application logic into three interconnected components, each with distinct responsibilities:

**Model (Data + Business Logic)**
```csharp
// Rich Domain Model Example
public class Order
{
    public int Id { get; private set; }
    public DateTime OrderDate { get; private set; }
    public OrderStatus Status { get; private set; }
    private readonly List<OrderItem> _items = new();
    public IReadOnlyCollection<OrderItem> Items => _items.AsReadOnly();

    public decimal TotalAmount => _items.Sum(item => item.Subtotal);

    public void AddItem(Product product, int quantity)
    {
        if (Status != OrderStatus.Draft)
            throw new InvalidOperationException("Cannot modify confirmed orders");
        
        var existingItem = _items.FirstOrDefault(i => i.ProductId == product.Id);
        if (existingItem != null)
            existingItem.UpdateQuantity(existingItem.Quantity + quantity);
        else
            _items.Add(new OrderItem(product, quantity));
    }

    public void Confirm()
    {
        ValidateForConfirmation();
        Status = OrderStatus.Confirmed;
        OrderDate = DateTime.UtcNow;
        // Raise domain event
        DomainEvents.Raise(new OrderConfirmedEvent(this));
    }
}
```

**View (Presentation Layer)**
```csharp
// Razor View with strongly-typed model
@model OrderViewModel
@{
    ViewData["Title"] = "Order Details";
}

<div class="order-container">
    <h2>Order #@Model.OrderNumber</h2>
    <div class="order-status status-@Model.Status.ToString().ToLower()">
        @Model.Status
    </div>
    
    @foreach (var item in Model.Items)
    {
        <div class="order-item">
            <span>@item.ProductName</span>
            <span>Qty: @item.Quantity</span>
            <span>@item.Subtotal.ToString("C")</span>
        </div>
    }
    
    <div class="order-total">
        Total: @Model.TotalAmount.ToString("C")
    </div>
</div>
```

**Controller (Flow Coordination)**
```csharp
[ApiController]
[Route("api/[controller]")]
public class OrderController : ControllerBase
{
    private readonly IOrderService _orderService;
    private readonly IMapper _mapper;

    public OrderController(IOrderService orderService, IMapper mapper)
    {
        _orderService = orderService;
        _mapper = mapper;
    }

    [HttpGet("{id}")]
    public async Task<ActionResult<OrderViewModel>> GetOrder(int id)
    {
        var order = await _orderService.GetOrderAsync(id);
        if (order == null)
            return NotFound();

        var viewModel = _mapper.Map<OrderViewModel>(order);
        return Ok(viewModel);
    }

    [HttpPost("{id}/confirm")]
    public async Task<ActionResult> ConfirmOrder(int id)
    {
        try
        {
            await _orderService.ConfirmOrderAsync(id);
            return NoContent();
        }
        catch (OrderNotFoundException)
        {
            return NotFound();
        }
        catch (InvalidOperationException ex)
        {
            return BadRequest(ex.Message);
        }
    }
}
```

## MVC Variations: Architectural Evolution

### MVP (Model-View-Presenter): Enhanced Testability

**Key Characteristics:**
- View is passive and doesn't directly interact with Model
- Presenter handles all UI logic and coordinates between View and Model
- Better testability due to passive View

```csharp
// MVP Implementation
public interface IOrderView
{
    event EventHandler<int> OrderRequested;
    event EventHandler<int> ConfirmOrderRequested;
    
    void DisplayOrder(OrderViewModel order);
    void ShowError(string message);
    void ShowLoading(bool isLoading);
}

public class OrderPresenter
{
    private readonly IOrderView _view;
    private readonly IOrderService _orderService;

    public OrderPresenter(IOrderView view, IOrderService orderService)
    {
        _view = view;
        _orderService = orderService;
        
        _view.OrderRequested += OnOrderRequested;
        _view.ConfirmOrderRequested += OnConfirmOrderRequested;
    }

    private async void OnOrderRequested(object sender, int orderId)
    {
        try
        {
            _view.ShowLoading(true);
            var order = await _orderService.GetOrderAsync(orderId);
            var viewModel = MapToViewModel(order);
            _view.DisplayOrder(viewModel);
        }
        catch (Exception ex)
        {
            _view.ShowError(ex.Message);
        }
        finally
        {
            _view.ShowLoading(false);
        }
    }
}
```

### MVVM (Model-View-ViewModel): Data Binding Focus

**Optimal for:** WPF, UWP, Xamarin, Blazor applications with two-way data binding

```csharp
// MVVM Implementation
public class OrderViewModel : INotifyPropertyChanged
{
    private Order _order;
    private bool _isLoading;

    public Order Order
    {
        get => _order;
        set
        {
            _order = value;
            OnPropertyChanged();
            OnPropertyChanged(nameof(TotalAmount));
        }
    }

    public decimal TotalAmount => Order?.TotalAmount ?? 0;

    public bool IsLoading
    {
        get => _isLoading;
        set
        {
            _isLoading = value;
            OnPropertyChanged();
        }
    }

    public ICommand ConfirmOrderCommand { get; }
    public ICommand AddItemCommand { get; }

    public OrderViewModel(IOrderService orderService)
    {
        ConfirmOrderCommand = new AsyncRelayCommand(ConfirmOrderAsync);
        AddItemCommand = new RelayCommand<Product>(AddItem);
    }

    private async Task ConfirmOrderAsync()
    {
        try
        {
            IsLoading = true;
            await _orderService.ConfirmOrderAsync(Order.Id);
            // Update UI state
        }
        catch (Exception ex)
        {
            // Handle error
        }
        finally
        {
            IsLoading = false;
        }
    }

    public event PropertyChangedEventHandler PropertyChanged;
    
    protected virtual void OnPropertyChanged([CallerMemberName] string propertyName = null)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }
}
```

## Framework-Specific Implementations

### ASP.NET Core MVC: Enterprise-Grade Implementation

```csharp
// Startup Configuration
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddDbContext<ApplicationDbContext>(options =>
            options.UseSqlServer(connectionString));
        
        services.AddScoped<IOrderService, OrderService>();
        services.AddScoped<IOrderRepository, OrderRepository>();
        
        services.AddAutoMapper(typeof(OrderProfile));
        
        services.AddMvc(options =>
        {
            options.Filters.Add<GlobalExceptionFilter>();
            options.Filters.Add<ValidationFilter>();
        });
    }
}

// Advanced Controller with Action Filters
[Route("api/[controller]")]
[ApiController]
[Authorize]
public class OrderController : ControllerBase
{
    private readonly IMediator _mediator;

    public OrderController(IMediator mediator)
    {
        _mediator = mediator;
    }

    [HttpPost]
    [ValidateModel]
    [ProducesResponseType(typeof(OrderViewModel), StatusCodes.Status201Created)]
    public async Task<ActionResult<OrderViewModel>> CreateOrder(CreateOrderCommand command)
    {
        var order = await _mediator.Send(command);
        return CreatedAtAction(nameof(GetOrder), new { id = order.Id }, order);
    }

    [HttpGet("{id}")]
    [ResponseCache(Duration = 300)]
    public async Task<ActionResult<OrderViewModel>> GetOrder(int id)
    {
        var query = new GetOrderQuery { OrderId = id };
        var order = await _mediator.Send(query);
        return Ok(order);
    }
}
```

### Modern Web API with CQRS Pattern

```csharp
// Command Handler
public class CreateOrderHandler : IRequestHandler<CreateOrderCommand, OrderViewModel>
{
    private readonly IOrderRepository _repository;
    private readonly IUnitOfWork _unitOfWork;
    private readonly IMapper _mapper;

    public async Task<OrderViewModel> Handle(CreateOrderCommand request, CancellationToken cancellationToken)
    {
        var order = new Order(request.CustomerId);
        
        foreach (var item in request.Items)
        {
            order.AddItem(item.Product, item.Quantity);
        }

        _repository.Add(order);
        await _unitOfWork.SaveChangesAsync(cancellationToken);

        return _mapper.Map<OrderViewModel>(order);
    }
}

// Query Handler
public class GetOrderHandler : IRequestHandler<GetOrderQuery, OrderViewModel>
{
    private readonly IOrderRepository _repository;
    private readonly IMapper _mapper;
    private readonly IMemoryCache _cache;

    public async Task<OrderViewModel> Handle(GetOrderQuery request, CancellationToken cancellationToken)
    {
        var cacheKey = $"order-{request.OrderId}";
        
        if (_cache.TryGetValue(cacheKey, out OrderViewModel cachedOrder))
            return cachedOrder;

        var order = await _repository.GetByIdAsync(request.OrderId);
        if (order == null)
            throw new OrderNotFoundException(request.OrderId);

        var viewModel = _mapper.Map<OrderViewModel>(order);
        _cache.Set(cacheKey, viewModel, TimeSpan.FromMinutes(15));
        
        return viewModel;
    }
}
```

## MVC in Modern Web Applications and SPAs

### Hybrid MVC-SPA Architecture

```csharp
// API Controller for SPA Backend
[Route("api/[controller]")]
[ApiController]
public class OrderApiController : ControllerBase
{
    [HttpGet]
    public async Task<ActionResult<PagedResult<OrderSummaryViewModel>>> GetOrders(
        [FromQuery] OrderFilterParameters parameters)
    {
        var orders = await _orderService.GetOrdersAsync(parameters);
        return Ok(orders);
    }

    [HttpGet("{id}")]
    public async Task<ActionResult<OrderDetailViewModel>> GetOrderDetail(int id)
    {
        var order = await _orderService.GetOrderDetailAsync(id);
        return Ok(order);
    }
}

// SignalR Hub for Real-time Updates
public class OrderHub : Hub
{
    public async Task JoinOrderGroup(int orderId)
    {
        await Groups.AddToGroupAsync(Context.ConnectionId, $"Order-{orderId}");
    }

    public async Task NotifyOrderStatusChanged(int orderId, OrderStatus newStatus)
    {
        await Clients.Group($"Order-{orderId}")
            .SendAsync("OrderStatusChanged", new { OrderId = orderId, Status = newStatus });
    }
}
```

### Blazor Server-Side MVC Integration

```csharp
// Blazor Component with MVC-like separation
@page "/orders/{OrderId:int}"
@inject IOrderService OrderService
@inject NavigationManager Navigation

<div class="order-detail-container">
    @if (Order != null)
    {
        <OrderDetailComponent Order="Order" OnOrderConfirmed="HandleOrderConfirmed" />
    }
    else if (IsLoading)
    {
        <LoadingSpinner />
    }
    else
    {
        <ErrorMessage Message="Order not found" />
    }
</div>

@code {
    [Parameter] public int OrderId { get; set; }
    
    private Order Order { get; set; }
    private bool IsLoading { get; set; }

    protected override async Task OnInitializedAsync()
    {
        IsLoading = true;
        try
        {
            Order = await OrderService.GetOrderAsync(OrderId);
        }
        catch (Exception ex)
        {
            // Handle error
        }
        finally
        {
            IsLoading = false;
        }
    }

    private async Task HandleOrderConfirmed()
    {
        await OrderService.ConfirmOrderAsync(OrderId);
        Navigation.NavigateTo("/orders");
    }
}
```

## Advantages: Architectural Benefits

### 1. Separation of Concerns
```csharp
// Clear responsibility boundaries
public class OrderController : ControllerBase  // HTTP concerns only
{
    private readonly IOrderApplicationService _orderService; // Business orchestration
    
    public async Task<ActionResult> ConfirmOrder(int id)
    {
        var result = await _orderService.ConfirmOrderAsync(id);
        return result.IsSuccess ? Ok() : BadRequest(result.Error);
    }
}

public class OrderApplicationService : IOrderApplicationService  // Business workflow
{
    public async Task<Result> ConfirmOrderAsync(int orderId)
    {
        var order = await _repository.GetByIdAsync(orderId);
        order.Confirm(); // Domain logic encapsulated
        await _unitOfWork.SaveChangesAsync();
        await _eventPublisher.PublishAsync(new OrderConfirmedEvent(order));
        return Result.Success();
    }
}
```

### 2. Enhanced Testability
```csharp
// Unit Testing Controllers
[Test]
public async Task ConfirmOrder_ValidOrder_ReturnsOk()
{
    // Arrange
    var mockService = new Mock<IOrderApplicationService>();
    mockService.Setup(x => x.ConfirmOrderAsync(123))
           .ReturnsAsync(Result.Success());
    
    var controller = new OrderController(mockService.Object);

    // Act
    var result = await controller.ConfirmOrder(123);

    // Assert
    Assert.IsInstanceOf<OkResult>(result);
    mockService.Verify(x => x.ConfirmOrderAsync(123), Times.Once);
}

// Integration Testing
[Test]
public async Task GetOrder_ExistingOrder_ReturnsCorrectData()
{
    // Arrange
    using var factory = new WebApplicationFactory<Program>();
    var client = factory.CreateClient();
    
    // Act
    var response = await client.GetAsync("/api/orders/1");
    
    // Assert
    response.EnsureSuccessStatusCode();
    var order = await response.Content.ReadFromJsonAsync<OrderViewModel>();
    Assert.That(order.Id, Is.EqualTo(1));
}
```

### 3. Parallel Development
```csharp
// Contract-First Development
public interface IOrderService
{
    Task<OrderViewModel> GetOrderAsync(int id);
    Task<Result> ConfirmOrderAsync(int id);
    Task<PagedResult<OrderSummaryViewModel>> GetOrdersAsync(OrderFilterParameters parameters);
}

// Teams can work in parallel:
// - Frontend team implements UI against interface
// - Backend team implements service
// - Database team designs optimal schema
```

## Disadvantages and Mitigation Strategies

### 1. Fat Controller Anti-Pattern

**Problem:**
```csharp
// BAD: Fat Controller
public class OrderController : ControllerBase
{
    public async Task<ActionResult> CreateOrder(CreateOrderRequest request)
    {
        // Validation logic
        if (string.IsNullOrEmpty(request.CustomerEmail))
            return BadRequest("Email is required");
        
        // Business logic
        var customer = await _dbContext.Customers.FirstOrDefaultAsync(c => c.Email == request.CustomerEmail);
        if (customer == null)
        {
            customer = new Customer { Email = request.CustomerEmail, CreatedDate = DateTime.Now };
            _dbContext.Customers.Add(customer);
        }
        
        // Complex calculation
        var total = request.Items.Sum(i => i.Price * i.Quantity * (1 - i.Discount));
        if (total > 10000) total *= 0.95m; // Bulk discount
        
        // Persistence
        var order = new Order { CustomerId = customer.Id, Total = total };
        _dbContext.Orders.Add(order);
        await _dbContext.SaveChangesAsync();
        
        // Email notification
        await _emailService.SendOrderConfirmationAsync(customer.Email, order);
        
        return Ok(order);
    }
}
```

**Solution: Thin Controller with Rich Services**
```csharp
// GOOD: Thin Controller
public class OrderController : ControllerBase
{
    private readonly IMediator _mediator;

    public async Task<ActionResult<OrderViewModel>> CreateOrder(CreateOrderCommand command)
    {
        var result = await _mediator.Send(command);
        return CreatedAtAction(nameof(GetOrder), new { id = result.Id }, result);
    }
}

// Rich Application Service
public class CreateOrderHandler : IRequestHandler<CreateOrderCommand, OrderViewModel>
{
    private readonly IOrderDomainService _domainService;
    private readonly ICustomerService _customerService;
    private readonly INotificationService _notificationService;

    public async Task<OrderViewModel> Handle(CreateOrderCommand request, CancellationToken cancellationToken)
    {
        var customer = await _customerService.GetOrCreateCustomerAsync(request.CustomerEmail);
        var order = await _domainService.CreateOrderAsync(customer, request.Items);
        await _notificationService.SendOrderConfirmationAsync(order);
        
        return _mapper.Map<OrderViewModel>(order);
    }
}
```

### 2. Complexity for Simple Applications

**Mitigation: Gradual Complexity Introduction**
```csharp
// Start Simple
public class SimpleOrderController : ControllerBase
{
    private readonly ApplicationDbContext _context;

    [HttpGet("{id}")]
    public async Task<Order> GetOrder(int id)
    {
        return await _context.Orders.FindAsync(id);
    }
}

// Evolve to Complex when needed
public class EnterpriseOrderController : ControllerBase
{
    private readonly IMediator _mediator;
    
    [HttpGet("{id}")]
    [ResponseCache(Duration = 300)]
    [Authorize]
    public async Task<ActionResult<OrderViewModel>> GetOrder([FromRoute] GetOrderQuery query)
    {
        var result = await _mediator.Send(query);
        return Ok(result);
    }
}
```

## Best Practices: Keeping Controllers Thin and Models Rich

### 1. Rich Domain Models

```csharp
// Rich Domain Model with Business Logic
public class Order
{
    private readonly List<OrderItem> _items = new();
    
    public decimal CalculateTotal()
    {
        var subtotal = _items.Sum(i => i.Subtotal);
        var discount = CalculateDiscount(subtotal);
        var tax = CalculateTax(subtotal - discount);
        return subtotal - discount + tax;
    }

    public void ApplyPromotion(IPromotionStrategy promotion)
    {
        promotion.Apply(this);
        RecalculateTotal();
    }

    public bool CanBeModified()
    {
        return Status == OrderStatus.Draft && 
               CreatedDate > DateTime.UtcNow.AddHours(-24);
    }

    private decimal CalculateDiscount(decimal subtotal)
    {
        if (subtotal > 1000) return subtotal * 0.1m;
        if (subtotal > 500) return subtotal * 0.05m;
        return 0;
    }
}
```

### 2. Application Service Pattern

```csharp
public class OrderApplicationService : IOrderApplicationService
{
    private readonly IOrderRepository _orderRepository;
    private readonly IDomainEventPublisher _eventPublisher;
    private readonly IUnitOfWork _unitOfWork;

    public async Task<Result<OrderViewModel>> ProcessOrderAsync(ProcessOrderCommand command)
    {
        using var transaction = await _unitOfWork.BeginTransactionAsync();
        
        try
        {
            var order = await _orderRepository.GetByIdAsync(command.OrderId);
            if (order == null)
                return Result<OrderViewModel>.Failure("Order not found");

            order.Process(command.ProcessingInstructions);
            
            await _orderRepository.UpdateAsync(order);
            await _eventPublisher.PublishAsync(new OrderProcessedEvent(order));
            await transaction.CommitAsync();
            
            return Result<OrderViewModel>.Success(_mapper.Map<OrderViewModel>(order));
        }
        catch (Exception ex)
        {
            await transaction.RollbackAsync();
            return Result<OrderViewModel>.Failure(ex.Message);
        }
    }
}
```

## Handling Cross-Cutting Concerns

### 1. Action Filters for AOP
```csharp
public class AuditActionFilter : IActionFilter
{
    private readonly IAuditService _auditService;

    public void OnActionExecuting(ActionExecutingContext context)
    {
        var auditInfo = new AuditInfo
        {
            UserId = context.HttpContext.User.GetUserId(),
            Action = context.ActionDescriptor.DisplayName,
            Parameters = JsonSerializer.Serialize(context.ActionArguments),
            Timestamp = DateTime.UtcNow
        };
        
        context.HttpContext.Items["AuditInfo"] = auditInfo;
    }

    public void OnActionExecuted(ActionExecutedContext context)
    {
        var auditInfo = (AuditInfo)context.HttpContext.Items["AuditInfo"];
        auditInfo.Success = context.Exception == null;
        
        _auditService.LogAsync(auditInfo);
    }
}
```

### 2. Middleware for Global Concerns
```csharp
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestLoggingMiddleware> _logger;

    public async Task InvokeAsync(HttpContext context)
    {
        var correlationId = Guid.NewGuid().ToString();
        context.Items["CorrelationId"] = correlationId;
        
        using var scope = _logger.BeginScope(new Dictionary<string, object>
        {
            ["CorrelationId"] = correlationId,
            ["UserId"] = context.User.GetUserId()
        });

        var stopwatch = Stopwatch.StartNew();
        
        try
        {
            await _next(context);
        }
        finally
        {
            stopwatch.Stop();
            _logger.LogInformation(
                "Request {Method} {Path} completed in {ElapsedMs}ms with status {StatusCode}",
                context.Request.Method,
                context.Request.Path,
                stopwatch.ElapsedMilliseconds,
                context.Response.StatusCode);
        }
    }
}
```

## MVC in Microservices Architecture

### 1. API Gateway Pattern with MVC
```csharp
// API Gateway Controller
[Route("api/gateway/[controller]")]
public class OrderGatewayController : ControllerBase
{
    private readonly IOrderServiceClient _orderService;
    private readonly IInventoryServiceClient _inventoryService;
    private readonly IPaymentServiceClient _paymentService;

    [HttpPost]
    public async Task<ActionResult<OrderViewModel>> CreateOrder(CreateOrderRequest request)
    {
        // Orchestrate multiple microservices
        var inventoryCheck = await _inventoryService.CheckAvailabilityAsync(request.Items);
        if (!inventoryCheck.AllAvailable)
            return BadRequest("Some items are not available");

        var order = await _orderService.CreateOrderAsync(request);
        
        // Async processing for payment
        _ = Task.Run(() => _paymentService.ProcessPaymentAsync(order.Id, request.PaymentInfo));

        return CreatedAtAction(nameof(GetOrder), new { id = order.Id }, order);
    }
}
```

### 2. Service-to-Service Communication
```csharp
// HTTP Client with Polly for Resilience
public class OrderServiceClient : IOrderServiceClient
{
    private readonly HttpClient _httpClient;
    private readonly IAsyncPolicy<HttpResponseMessage> _retryPolicy;

    public OrderServiceClient(HttpClient httpClient)
    {
        _httpClient = httpClient;
        _retryPolicy = Policy
            .HandleResult<HttpResponseMessage>(r => !r.IsSuccessStatusCode)
            .WaitAndRetryAsync(3, retryAttempt => 
                TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)));
    }

    public async Task<OrderViewModel> GetOrderAsync(int orderId)
    {
        var response = await _retryPolicy.ExecuteAsync(() => 
            _httpClient.GetAsync($"/api/orders/{orderId}"));
        
        response.EnsureSuccessStatusCode();
        return await response.Content.ReadFromJsonAsync<OrderViewModel>();
    }
}
```

## Performance Optimization Strategies

### 1. Caching Strategies
```csharp
public class CachedOrderService : IOrderService
{
    private readonly IOrderService _decorated;
    private readonly IMemoryCache _cache;
    private readonly IDistributedCache _distributedCache;

    public async Task<OrderViewModel> GetOrderAsync(int orderId)
    {
        var cacheKey = $"order-{orderId}";
        
        // L1 Cache (Memory)
        if (_cache.TryGetValue(cacheKey, out OrderViewModel cachedOrder))
            return cachedOrder;

        // L2 Cache (Redis)
        var distributedOrder = await _distributedCache.GetAsync(cacheKey);
        if (distributedOrder != null)
        {
            var order = JsonSerializer.Deserialize<OrderViewModel>(distributedOrder);
            _cache.Set(cacheKey, order, TimeSpan.FromMinutes(5));
            return order;
        }

        // Fallback to service
        var result = await _decorated.GetOrderAsync(orderId);
        
        // Update caches
        var serialized = JsonSerializer.SerializeToUtf8Bytes(result);
        await _distributedCache.SetAsync(cacheKey, serialized, new DistributedCacheEntryOptions
        {
            SlidingExpiration = TimeSpan.FromMinutes(30)
        });
        _cache.Set(cacheKey, result, TimeSpan.FromMinutes(5));
        
        return result;
    }
}
```

### 2. Async/Await Best Practices
```csharp
public class OptimizedOrderController : ControllerBase
{
    [HttpGet("{id}")]
    public async Task<ActionResult<OrderDetailViewModel>> GetOrderWithDetails(int id)
    {
        // Parallel execution for independent operations
        var orderTask = _orderService.GetOrderAsync(id);
        var customerTask = _customerService.GetCustomerAsync(id);
        var shipmentsTask = _shippingService.GetShipmentsAsync(id);

        await Task.WhenAll(orderTask, customerTask, shipmentsTask);

        var order = await orderTask;
        var customer = await customerTask;
        var shipments = await shipmentsTask;

        return Ok(new OrderDetailViewModel
        {
            Order = order,
            Customer = customer,
            Shipments = shipments
        });
    }
}
```

## Interview Success Framework

### Key Technical Points for Senior .NET Architects:

1. **MVC Evolution Understanding**: Traditional MVC → Web API → Minimal APIs → Microservices
2. **SOLID Principles Application**: How MVC naturally supports dependency inversion and single responsibility
3. **Performance Considerations**: When to use caching, async patterns, and connection pooling
4. **Testing Strategies**: Unit testing controllers, integration testing APIs, contract testing for microservices
5. **Enterprise Patterns**: CQRS, Event Sourcing, Domain Events in MVC context

### Sample Interview Question Responses:

**Q: How do you prevent tight coupling between layers in MVC?**

A: "I implement dependency inversion through interfaces, use the mediator pattern for complex workflows, apply DTOs for data transfer, and leverage dependency injection containers. For example, controllers depend on abstractions like `IOrderService` rather than concrete implementations, enabling easy testing and swapping of implementations."

**Q: How do you handle validation in a distributed MVC application?**

A: "I implement multi-layered validation: client-side for UX, model validation attributes for basic rules, FluentValidation for complex business rules, and domain model validation for invariants. Each layer serves a specific purpose and can be tested independently."

This comprehensive guide positions you as a solution-oriented architect who understands both the theoretical foundations and practical implementations of MVC patterns in enterprise .NET applications.

---

22. **REST Architecture**
    - How do you design RESTful APIs following REST principles?
    - Show proper HTTP verbs, status codes, and resource naming.

23. **API Versioning**
    - How do you implement API versioning without breaking existing clients?
    - Show URL, header, and query parameter versioning strategies.

## **SCENARIO-BASED QUESTIONS**

### **Real-World Application Design**
24. **E-Commerce API Scenario**
    - "Design a Web API for an e-commerce platform with products, orders, payments, and inventory management. What design patterns would you use and why?"
    - Follow-up: How would you handle concurrent order processing?
    - Follow-up: How would you implement different payment gateways?

25. **Banking System Scenario**
    - "Design a banking API with account management, transactions, and fraud detection. How would you ensure data consistency and security?"
    - Follow-up: How would you implement transaction rollback?
    - Follow-up: How would you handle audit logging?

26. **Notification System Scenario**
    - "Design a notification service that can send emails, SMS, and push notifications. How would you make it extensible for new notification types?"
    - Follow-up: How would you handle notification failures and retries?

27. **File Processing Scenario**
    - "Design an API that processes uploaded files (CSV, Excel, PDF). How would you handle different file types and large file uploads?"
    - Follow-up: How would you implement background processing?
    - Follow-up: How would you handle file validation and error reporting?

### **Performance & Scalability**
28. **Caching Strategy**
    - "Your API is experiencing high load. How would you implement caching at different levels?"
    - Show in-memory, distributed, and HTTP caching strategies.

29. **Database Performance**
    - "Your API is slow due to database queries. How would you optimize it using design patterns?"
    - Show Repository with caching, query optimization, and connection management.

30. **Microservices Communication**
    - "How would you design communication between microservices in your Web API?"
    - Show synchronous vs asynchronous communication patterns.

### **Error Handling & Resilience**
31. **Global Error Handling**
    - "How would you implement consistent error handling across your entire Web API?"
    - Show middleware, filters, and exception handling patterns.

32. **Retry Pattern**
    - "How would you implement retry logic for external API calls?"
    - Show exponential backoff and circuit breaker patterns.

33. **Validation Patterns**
    - "How would you implement input validation that's reusable and maintainable?"
    - Show model validation, custom attributes, and FluentValidation.

### **Security Patterns**
34. **Authentication & Authorization**
    - "How would you implement JWT-based authentication with role-based authorization?"
    - Show token generation, validation, and middleware implementation.

35. **API Security**
    - "How would you protect your API from common security threats?"
    - Show rate limiting, input sanitization, and CORS implementation.

### **Testing & Maintainability**
36. **Testable Design**
    - "How do you design your Web API to be easily testable?"
    - Show dependency injection, mocking, and integration testing strategies.

37. **Configuration Management**
    - "How would you manage different configurations for dev, staging, and production?"
    - Show Options pattern, environment-specific settings, and configuration validation.

### **Advanced Scenarios**
38. **Event Sourcing**
    - "How would you implement an audit trail system using event sourcing?"
    - Show event store, replay capability, and CQRS integration.

39. **Multi-tenant Architecture**
    - "How would you design a Web API that serves multiple tenants with data isolation?"
    - Show tenant identification, data separation strategies, and configuration.

40. **API Gateway Pattern**
    - "How would you implement cross-cutting concerns like logging, authentication, and rate limiting across multiple APIs?"
    - Show gateway implementation and service orchestration.

## **FOLLOW-UP QUESTIONS (Expect These After Your Initial Answer)**

- **"How would you test this pattern?"**
- **"What are the disadvantages of this approach?"**
- **"How would this scale with increased load?"**
- **"How would you handle failures in this design?"**
- **"What metrics would you monitor for this implementation?"**
- **"How would you migrate from the old design to this new one?"**
- **"What are the performance implications?"**
- **"How does this pattern affect maintainability?"**

## **KEY SUCCESS TIPS**

1. **Always start with the problem** the pattern/principle solves
2. **Show concrete code examples** (10-15 lines max)
3. **Discuss trade-offs** - every pattern has pros and cons
4. **Connect to business value** - how does it help the organization?
5. **Mention testing strategy** for each pattern you discuss
6. **Be ready to critique your own design** and suggest improvements

--- 

