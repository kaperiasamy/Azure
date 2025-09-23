I may have an interview in a day or two. The main areas covered will be .NET Core and Web API. Mostly the questions will be on how I design an application, what are the design patterns used, how that design patterns are implemented, what is the purpose of the design pattern used for (which problem it solved), what are the design principles used, how those design principles are implemented, something like this. Get me the list of questions (full list of most important questions) that may be asked, which includes scenario-based questions.

--- 

Here's a comprehensive list of **interview questions for .NET Core & Web API** focusing on design patterns, principles, and application architecture:## **MOST CRITICAL QUESTIONS TO PREPARE** (High Probability)

### **Top 10 Must-Know Questions:**

1. **Dependency Injection in .NET Core** - Service lifetimes and implementation
2. **Repository Pattern with Entity Framework** - Generic vs Specific repositories
3. **SOLID Principles with Web API examples** - Especially SRP and DIP
4. **Clean Architecture structure** - Layers and dependency flow
5. **Error handling middleware** - Global exception handling
6. **Authentication/Authorization patterns** - JWT implementation
7. **API versioning strategies** - URL vs header versioning
8. **Caching patterns** - In-memory, distributed, HTTP caching
9. **CQRS implementation** - Command and Query separation
10. **Unit of Work pattern** - Transaction management

### **Scenario-Based Preparation Strategy:**

**Pick 2-3 domains you know well** (e-commerce, banking, inventory management) and be ready to:
- Design complete API architecture
- Explain pattern choices and alternatives
- Show code implementations
- Discuss scalability and performance
- Address security concerns
- Explain testing approach

--- 
Here's a comprehensive **ERP Domain Scenario-Based Preparation Strategy** for your .NET Core Web API interview:## **Additional ERP-Specific Interview Questions to Prepare**

### **Advanced Scenario Questions:**

1. **"How would you handle a scenario where payroll processing fails halfway through 10,000 employees?"**
   - Discuss idempotent operations, checkpoint recovery, and transaction compensation

2. **"Design an API that handles complex approval workflows for purchase orders with multiple approval levels."**
   - Show State Machine pattern, workflow orchestration, and notification systems

3. **"How would you implement financial period closing that affects multiple modules simultaneously?"**
   - Discuss distributed transactions, saga patterns, and eventual consistency

4. **"Design a multi-currency ERP system where exchange rates change daily."**
   - Show strategy pattern for currency conversion, temporal data patterns, and audit trails

### **Performance Deep-Dive Questions:**

1. **"Your financial reports are taking 30+ seconds to generate. How would you optimize?"**
   - Discuss CQRS with read replicas, materialized views, and background report generation

2. **"How would you handle concurrent inventory updates from multiple warehouses?"**
   - Show optimistic concurrency control, event sourcing, and conflict resolution

### **Quick Preparation Checklist:**

✅ **Understand ERP Business Flows**: Procure-to-Pay, Order-to-Cash, Hire-to-Retire
✅ **Know Multi-Tenant Patterns**: Data isolation strategies and tenant resolution
✅ **Master Transaction Management**: ACID properties, distributed transactions, saga patterns
✅ **Practice Security Implementation**: JWT, role-based access, data encryption
✅ **Understand Compliance Requirements**: Audit trails, data retention, regulatory reporting
✅ **Know Integration Patterns**: API gateways, event-driven architecture, message queues

This ERP scenario preparation gives you a complete framework to handle any enterprise-level API design question with confidence!

# ERP Domain - Scenario-Based Interview Preparation Strategy

## **CORE ERP SCENARIO**
**"Design a Web API for an ERP system that handles Employee Management, Payroll Processing, Inventory Management, and Financial Reporting. The system must support multiple companies (multi-tenant), handle high transaction volumes, ensure data consistency, and provide real-time reporting."**

---

## **1. COMPLETE API ARCHITECTURE DESIGN**

### **Clean Architecture Structure**
```
ERP.Web.API/                 (Presentation Layer)
├── Controllers/
├── Middleware/
├── Filters/
└── Program.cs

ERP.Application/             (Application Layer)
├── Commands/               (CQRS Write Operations)
├── Queries/               (CQRS Read Operations)  
├── Services/              (Application Services)
├── DTOs/                  (Data Transfer Objects)
├── Interfaces/            (Application Contracts)
└── Validators/            (FluentValidation)

ERP.Domain/                  (Domain Layer)
├── Entities/              (Business Entities)
├── ValueObjects/          (Value Objects)
├── DomainEvents/          (Domain Events)
├── Interfaces/            (Domain Contracts)
└── Specifications/        (Business Rules)

ERP.Infrastructure/          (Infrastructure Layer)
├── Data/                  (Entity Framework)
├── Repositories/          (Data Access)
├── Services/              (External Services)
├── Caching/              (Redis Implementation)
└── Messaging/            (Event Bus)

ERP.Shared/                  (Shared Kernel)
├── Constants/
├── Enums/
├── Exceptions/
└── Extensions/
```

### **Microservices Decomposition**
- **Identity Service**: Authentication, Authorization, User Management
- **Employee Service**: HR Management, Employee Records
- **Payroll Service**: Salary Processing, Tax Calculations
- **Inventory Service**: Stock Management, Procurement
- **Finance Service**: Accounting, Financial Reporting
- **Notification Service**: Email, SMS, Push Notifications

---

## **2. PATTERN CHOICES & ALTERNATIVES**

### **Primary Patterns Used**

#### **CQRS (Command Query Responsibility Segregation)**
```csharp
// Command - Write Operations
public class CreateEmployeeCommand : IRequest<CreateEmployeeResponse>
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Email { get; set; }
    public int DepartmentId { get; set; }
    public decimal Salary { get; set; }
    public int TenantId { get; set; }
}

// Query - Read Operations  
public class GetEmployeeQuery : IRequest<EmployeeDto>
{
    public int EmployeeId { get; set; }
    public int TenantId { get; set; }
}

// Handler Implementation
public class CreateEmployeeCommandHandler : IRequestHandler<CreateEmployeeCommand, CreateEmployeeResponse>
{
    private readonly IEmployeeRepository _repository;
    private readonly IEventBus _eventBus;
    
    public async Task<CreateEmployeeResponse> Handle(CreateEmployeeCommand request, CancellationToken cancellationToken)
    {
        var employee = Employee.Create(request.FirstName, request.LastName, request.Email, request.TenantId);
        await _repository.AddAsync(employee);
        
        // Publish domain event
        await _eventBus.PublishAsync(new EmployeeCreatedEvent(employee.Id, employee.TenantId));
        
        return new CreateEmployeeResponse { EmployeeId = employee.Id };
    }
}
```

**Why CQRS?**
- Separate read/write models for complex ERP operations
- Different optimization strategies for queries vs commands
- Better scalability with separate read/write databases

**Alternative**: Traditional Repository pattern (simpler but less scalable)

#### **Multi-Tenant Strategy Pattern**
```csharp
// Tenant Resolution Strategy
public interface ITenantResolver
{
    int ResolveTenantId(HttpContext context);
}

public class HeaderTenantResolver : ITenantResolver
{
    public int ResolveTenantId(HttpContext context)
    {
        return int.Parse(context.Request.Headers["X-Tenant-Id"].FirstOrDefault() ?? "0");
    }
}

public class SubdomainTenantResolver : ITenantResolver
{
    public int ResolveTenantId(HttpContext context)
    {
        var host = context.Request.Host.Host;
        var subdomain = host.Split('.').FirstOrDefault();
        return GetTenantIdFromSubdomain(subdomain);
    }
}

// Multi-tenant middleware
public class TenantMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ITenantResolver _tenantResolver;
    
    public async Task InvokeAsync(HttpContext context, ITenantResolver tenantResolver)
    {
        var tenantId = tenantResolver.ResolveTenantId(context);
        context.Items["TenantId"] = tenantId;
        await _next(context);
    }
}
```

#### **Unit of Work with Repository Pattern**
```csharp
public interface IUnitOfWork : IDisposable
{
    IEmployeeRepository Employees { get; }
    IPayrollRepository Payrolls { get; }
    IInventoryRepository Inventory { get; }
    Task<int> CommitAsync();
    Task RollbackAsync();
}

public class UnitOfWork : IUnitOfWork
{
    private readonly ERPDbContext _context;
    private readonly ITenantContext _tenantContext;
    
    public UnitOfWork(ERPDbContext context, ITenantContext tenantContext)
    {
        _context = context;
        _tenantContext = tenantContext;
    }
    
    public async Task<int> CommitAsync()
    {
        using var transaction = await _context.Database.BeginTransactionAsync();
        try
        {
            var result = await _context.SaveChangesAsync();
            await transaction.CommitAsync();
            return result;
        }
        catch
        {
            await transaction.RollbackAsync();
            throw;
        }
    }
}
```

---

## **3. CODE IMPLEMENTATIONS**

### **Employee Management API**
```csharp
[ApiController]
[Route("api/v1/employees")]
[Authorize]
public class EmployeesController : ControllerBase
{
    private readonly IMediator _mediator;
    private readonly ITenantContext _tenantContext;
    
    [HttpPost]
    [HasPermission("employees.create")]
    public async Task<ActionResult<CreateEmployeeResponse>> CreateEmployee(CreateEmployeeCommand command)
    {
        command.TenantId = _tenantContext.TenantId;
        var result = await _mediator.Send(command);
        return CreatedAtAction(nameof(GetEmployee), new { id = result.EmployeeId }, result);
    }
    
    [HttpGet("{id}")]
    [HasPermission("employees.read")]
    [ResponseCache(Duration = 300, VaryByHeader = "X-Tenant-Id")]
    public async Task<ActionResult<EmployeeDto>> GetEmployee(int id)
    {
        var query = new GetEmployeeQuery { EmployeeId = id, TenantId = _tenantContext.TenantId };
        var result = await _mediator.Send(query);
        return Ok(result);
    }
}
```

### **Payroll Processing with Saga Pattern**
```csharp
// Saga for complex payroll processing
public class PayrollProcessingSaga : ISaga<PayrollInitiatedEvent>, 
                                    ISaga<SalaryCalculatedEvent>,
                                    ISaga<TaxCalculatedEvent>
{
    public async Task Handle(PayrollInitiatedEvent @event)
    {
        // Step 1: Calculate base salary
        await _mediator.Send(new CalculateSalaryCommand(@event.PayrollId));
    }
    
    public async Task Handle(SalaryCalculatedEvent @event)
    {
        // Step 2: Calculate taxes
        await _mediator.Send(new CalculateTaxCommand(@event.PayrollId, @event.BaseSalary));
    }
    
    public async Task Handle(TaxCalculatedEvent @event)
    {
        // Step 3: Generate pay slip and send notification
        await _mediator.Send(new GeneratePaySlipCommand(@event.PayrollId));
        await _mediator.Send(new SendPayrollNotificationCommand(@event.EmployeeId));
    }
}
```

---

## **4. SCALABILITY & PERFORMANCE**

### **Horizontal Scaling Strategies**

#### **Database Sharding by Tenant**
```csharp
public class TenantDbContextFactory : IDbContextFactory<ERPDbContext>
{
    private readonly IConfiguration _configuration;
    
    public ERPDbContext CreateDbContext(int tenantId)
    {
        var shardKey = tenantId % 4; // 4 database shards
        var connectionString = _configuration.GetConnectionString($"Shard{shardKey}");
        
        var options = new DbContextOptionsBuilder<ERPDbContext>()
            .UseSqlServer(connectionString)
            .Options;
            
        return new ERPDbContext(options, tenantId);
    }
}
```

#### **Redis Distributed Caching**
```csharp
public class CachedEmployeeRepository : IEmployeeRepository
{
    private readonly IEmployeeRepository _repository;
    private readonly IDistributedCache _cache;
    private readonly ITenantContext _tenantContext;
    
    public async Task<Employee> GetByIdAsync(int id)
    {
        var cacheKey = $"employee:{_tenantContext.TenantId}:{id}";
        var cached = await _cache.GetStringAsync(cacheKey);
        
        if (cached != null)
            return JsonSerializer.Deserialize<Employee>(cached);
            
        var employee = await _repository.GetByIdAsync(id);
        
        await _cache.SetStringAsync(cacheKey, JsonSerializer.Serialize(employee),
            new DistributedCacheEntryOptions { AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(15) });
            
        return employee;
    }
}
```

#### **Background Job Processing**
```csharp
// Payroll processing background job
[AutomaticRetry(Attempts = 3)]
public class PayrollProcessingJob : IBackgroundJob
{
    public async Task Execute(PayrollProcessingRequest request)
    {
        var employees = await _employeeRepository.GetActiveEmployeesAsync(request.TenantId);
        
        var tasks = employees.Select(async employee => 
        {
            await _mediator.Send(new ProcessEmployeePayrollCommand 
            { 
                EmployeeId = employee.Id, 
                PayPeriod = request.PayPeriod 
            });
        });
        
        await Task.WhenAll(tasks);
    }
}
```

---

## **5. SECURITY CONCERNS**

### **JWT Authentication with Role-Based Authorization**
```csharp
[HasPermission("payroll.process")]
[RequiresTenant]
public class PayrollController : ControllerBase
{
    [HttpPost("process")]
    [RateLimiting(MaxRequests = 5, WindowMinutes = 60)] // Prevent abuse
    public async Task<IActionResult> ProcessPayroll(ProcessPayrollCommand command)
    {
        // Audit logging
        _auditLogger.LogAction("PayrollProcessing", HttpContext.User.Identity.Name, 
                              _tenantContext.TenantId, command);
        
        var result = await _mediator.Send(command);
        return Ok(result);
    }
}

// Custom authorization attribute
public class HasPermissionAttribute : AuthorizeAttribute, IAuthorizationRequirement
{
    public string Permission { get; }
    
    public HasPermissionAttribute(string permission)
    {
        Permission = permission;
        Policy = permission;
    }
}

// Permission handler
public class PermissionAuthorizationHandler : AuthorizationHandler<HasPermissionAttribute>
{
    protected override async Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        HasPermissionAttribute requirement)
    {
        var tenantId = context.Resource as int?;
        var userPermissions = await _permissionService.GetUserPermissionsAsync(
            context.User.Identity.Name, tenantId);
            
        if (userPermissions.Contains(requirement.Permission))
        {
            context.Succeed(requirement);
        }
    }
}
```

### **Data Encryption & PII Protection**
```csharp
// Sensitive data encryption
public class Employee
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    
    [Encrypted] // Custom attribute for automatic encryption
    public string SSN { get; set; }
    
    [Encrypted]
    public decimal Salary { get; set; }
    
    public int TenantId { get; set; }
}

// Encryption interceptor
public class EncryptionInterceptor : SaveChangesInterceptor
{
    private readonly IDataProtector _protector;
    
    public override ValueTask<InterceptionResult<int>> SavingChangesAsync(
        DbContextEventData eventData, 
        InterceptionResult<int> result, 
        CancellationToken cancellationToken = default)
    {
        EncryptProperties(eventData.Context);
        return base.SavingChangesAsync(eventData, result, cancellationToken);
    }
}
```

---

## **6. TESTING APPROACH**

### **Unit Testing with Domain-Driven Design**
```csharp
[TestFixture]
public class EmployeeTests
{
    [Test]
    public void Employee_Create_Should_Raise_EmployeeCreatedEvent()
    {
        // Arrange
        var firstName = "John";
        var lastName = "Doe";
        var email = "john@company.com";
        var tenantId = 1;
        
        // Act
        var employee = Employee.Create(firstName, lastName, email, tenantId);
        
        // Assert
        employee.FirstName.Should().Be(firstName);
        employee.DomainEvents.Should().ContainSingle()
            .Which.Should().BeOfType<EmployeeCreatedEvent>();
    }
    
    [Test]
    public async Task CreateEmployeeCommandHandler_Should_Save_Employee_And_Publish_Event()
    {
        // Arrange
        var command = new CreateEmployeeCommand 
        { 
            FirstName = "John", 
            LastName = "Doe", 
            Email = "john@test.com",
            TenantId = 1 
        };
        
        var mockRepository = new Mock<IEmployeeRepository>();
        var mockEventBus = new Mock<IEventBus>();
        var handler = new CreateEmployeeCommandHandler(mockRepository.Object, mockEventBus.Object);
        
        // Act
        var result = await handler.Handle(command, CancellationToken.None);
        
        // Assert
        mockRepository.Verify(x => x.AddAsync(It.IsAny<Employee>()), Times.Once);
        mockEventBus.Verify(x => x.PublishAsync(It.IsAny<EmployeeCreatedEvent>()), Times.Once);
    }
}
```

### **Integration Testing with TestContainers**
```csharp
[TestFixture]
public class EmployeeApiIntegrationTests
{
    private WebApplicationFactory<Program> _factory;
    private HttpClient _client;
    private PostgreSqlContainer _dbContainer;
    
    [OneTimeSetUp]
    public async Task OneTimeSetUp()
    {
        _dbContainer = new PostgreSqlBuilder()
            .WithImage("postgres:13")
            .WithDatabase("erp_test")
            .Build();
            
        await _dbContainer.StartAsync();
        
        _factory = new WebApplicationFactory<Program>()
            .WithWebHostBuilder(builder =>
            {
                builder.ConfigureTestServices(services =>
                {
                    services.AddDbContext<ERPDbContext>(options =>
                        options.UseNpgsql(_dbContainer.GetConnectionString()));
                });
            });
            
        _client = _factory.CreateClient();
    }
    
    [Test]
    public async Task CreateEmployee_Should_Return_Created_Status()
    {
        // Arrange
        var command = new CreateEmployeeCommand 
        { 
            FirstName = "John", 
            LastName = "Doe", 
            Email = "john@test.com" 
        };
        
        _client.DefaultRequestHeaders.Add("X-Tenant-Id", "1");
        _client.DefaultRequestHeaders.Authorization = 
            new AuthenticationHeaderValue("Bearer", await GetValidJwtToken());
        
        // Act
        var response = await _client.PostAsJsonAsync("/api/v1/employees", command);
        
        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.Created);
        var result = await response.Content.ReadFromJsonAsync<CreateEmployeeResponse>();
        result.EmployeeId.Should().BeGreaterThan(0);
    }
}
```

### **Performance Testing Strategy**
```csharp
[TestFixture]
public class PayrollPerformanceTests
{
    [Test]
    public async Task ProcessPayroll_Should_Handle_1000_Employees_Within_30_Seconds()
    {
        // Arrange
        var employees = GenerateTestEmployees(1000);
        var stopwatch = Stopwatch.StartNew();
        
        // Act
        var result = await _payrollService.ProcessBulkPayrollAsync(employees, DateTime.Now);
        
        // Assert
        stopwatch.Stop();
        stopwatch.Elapsed.Should().BeLessThan(TimeSpan.FromSeconds(30));
        result.ProcessedCount.Should().Be(1000);
        result.FailedCount.Should().Be(0);
    }
}
```

---

## **INTERVIEW RESPONSE STRUCTURE**

### **When Asked: "How would you design an ERP system API?"**

1. **Start with Domain Understanding**:
   "ERP systems have complex business workflows, multi-tenant requirements, and need strong consistency. I'd use Clean Architecture with CQRS to separate complex read/write operations..."

2. **Explain Pattern Choices**:
   "I chose CQRS because payroll processing involves complex calculations but reporting needs optimized queries. The Unit of Work pattern ensures transaction consistency across multiple entities..."

3. **Address Scalability**:
   "For scaling, I'd implement tenant-based sharding, Redis caching for frequently accessed data like employee details, and background job processing for long-running operations like payroll..."

4. **Security Focus**:
   "ERP systems handle sensitive PII and financial data. I'd implement JWT with fine-grained permissions, encrypt sensitive fields at rest, and maintain comprehensive audit logs..."

5. **Testing Strategy**:
   "I'd focus on domain logic unit tests, integration tests with TestContainers for database scenarios, and performance tests to ensure payroll processing meets SLA requirements..."

---

## **KEY INTERVIEW TIPS**

1. **Always mention multi-tenancy** - ERP systems serve multiple companies
2. **Emphasize data consistency** - Financial data accuracy is critical
3. **Discuss audit trails** - Regulatory compliance requirements
4. **Address bulk operations** - Payroll processes thousands of records
5. **Consider background processing** - Long-running financial calculations
6. **Mention reporting performance** - Executives need fast dashboards

This comprehensive strategy demonstrates deep understanding of enterprise-level API design with real-world ERP complexity.

---

### **Code Examples to Practice:**

Prepare **10-15 line code snippets** for:
- DI container registration
- Repository pattern implementation  
- Middleware creation
- Custom attribute validation
- Strategy pattern for business rules
- Factory pattern for service creation

**Pro Tip**: For each pattern, know:
1. **What problem it solves**
2. **When NOT to use it**
3. **How to test it**
4. **Performance implications**
5. **Real-world alternatives**

This comprehensive list covers 95% of what you'll encounter in .NET Core/Web API design-focused interviews. Focus on the "Most Critical Questions" first, then expand to scenario-based questions.

Here are **interview-ready code snippets** with comprehensive explanations for each pattern:## **DETAILED EXPLANATIONS FOR EACH PATTERN**

---

## **1. DEPENDENCY INJECTION CONTAINER REGISTRATION**

### **What Problem It Solves:**
- **Tight Coupling**: Eliminates hard-coded dependencies between classes
- **Testing Difficulty**: Enables easy mocking of dependencies for unit tests  
- **Object Lifetime Management**: Automatically manages object creation and disposal
- **Configuration Centralization**: All dependencies configured in one place

### **When NOT to Use It:**
- **Simple Applications**: Overkill for small applications with few dependencies
- **Performance-Critical Paths**: DI container resolution adds minimal but measurable overhead
- **Static Utilities**: Don't inject stateless utility classes (Math functions, string helpers)
- **Value Objects**: Simple data containers shouldn't use DI

### Code Snippet
```csharp
// =============================================================================
// 1. DEPENDENCY INJECTION CONTAINER REGISTRATION
// =============================================================================

// Program.cs - Service Registration
var builder = WebApplication.CreateBuilder(args);

// Different service lifetimes
builder.Services.AddScoped<IUserRepository, UserRepository>();        // Per HTTP request
builder.Services.AddTransient<IEmailService, EmailService>();         // New instance every time
builder.Services.AddSingleton<ICacheService, RedisCacheService>();    // Single instance for app lifetime

// Generic registration with implementation factory
builder.Services.AddScoped<IRepository<T>, Repository<T>>();
builder.Services.AddScoped<IUnitOfWork>(provider => 
    new UnitOfWork(provider.GetService<AppDbContext>()));

// Configuration binding
builder.Services.Configure<EmailSettings>(builder.Configuration.GetSection("EmailSettings"));

// Conditional registration based on environment
if (builder.Environment.IsDevelopment())
    builder.Services.AddScoped<IPaymentService, MockPaymentService>();
else
    builder.Services.AddScoped<IPaymentService, StripePaymentService>();

var app = builder.Build();
```

### **How to Test It:**
```csharp
[Test]
public void Should_Resolve_Scoped_Service_Same_Instance_Within_Scope()
{
    // Arrange
    var services = new ServiceCollection();
    services.AddScoped<IUserRepository, UserRepository>();
    var provider = services.BuildServiceProvider();
    
    // Act
    using var scope = provider.CreateScope();
    var service1 = scope.ServiceProvider.GetService<IUserRepository>();
    var service2 = scope.ServiceProvider.GetService<IUserRepository>();
    
    // Assert
    Assert.AreSame(service1, service2); // Same instance within scope
}
```

### **Performance Implications:**
- **Resolution Overhead**: ~10-50 nanoseconds per resolution
- **Memory Usage**: Singleton services remain in memory for application lifetime
- **Scoped Services**: Created/disposed per HTTP request - can impact GC pressure
- **Transient Services**: Highest memory overhead, new instance every time

### **Real-World Alternatives:**
- **Service Locator Pattern**: `ServiceProvider.GetService<T>()` (anti-pattern, but sometimes necessary)
- **Manual Factory Classes**: Explicit object creation for better control
- **Static Classes**: For stateless operations (extension methods, utilities)
- **Pure Functions**: For simple transformations without dependencies

---

## **2. REPOSITORY PATTERN IMPLEMENTATION**

### **What Problem It Solves:**
- **Data Access Abstraction**: Hides database implementation details from business logic
- **Testability**: Easy to mock for unit testing without database
- **Consistency**: Standardized data access methods across entities
- **Multiple Data Sources**: Can switch between SQL, NoSQL, APIs transparently

### **When NOT to Use It:**
- **Simple CRUD Applications**: Entity Framework DbContext might be sufficient
- **Complex Queries**: Repository can't expose all ORM capabilities efficiently
- **Performance-Critical Scenarios**: Additional abstraction layer adds overhead
- **When Using CQRS**: Query side often benefits from direct data access

### Code snippet
```csharp
// =============================================================================
// 2. REPOSITORY PATTERN IMPLEMENTATION
// =============================================================================

// Generic Repository Interface
public interface IRepository<T> where T : class
{
    Task<T> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task<T> AddAsync(T entity);
    Task UpdateAsync(T entity);
    Task DeleteAsync(int id);
}

// Repository Implementation
public class Repository<T> : IRepository<T> where T : class
{
    private readonly DbContext _context;
    private readonly DbSet<T> _dbSet;

    public Repository(DbContext context)
    {
        _context = context;
        _dbSet = context.Set<T>();
    }

    public async Task<T> GetByIdAsync(int id) => await _dbSet.FindAsync(id);
    public async Task<IEnumerable<T>> GetAllAsync() => await _dbSet.ToListAsync();
    public async Task<T> AddAsync(T entity) { _dbSet.Add(entity); await _context.SaveChangesAsync(); return entity; }
    public async Task UpdateAsync(T entity) { _context.Entry(entity).State = EntityState.Modified; await _context.SaveChangesAsync(); }
    public async Task DeleteAsync(int id) { var entity = await GetByIdAsync(id); _dbSet.Remove(entity); await _context.SaveChangesAsync(); }
}
```

### **How to Test It:**
```csharp
[Test]
public async Task GetByIdAsync_Should_Return_Correct_Entity()
{
    // Arrange
    var options = new DbContextOptionsBuilder<TestDbContext>()
        .UseInMemoryDatabase(databaseName: "TestDb")
        .Options;
    
    using var context = new TestDbContext(options);
    var repository = new Repository<User>(context);
    var user = new User { Id = 1, Name = "Test User" };
    await context.Users.AddAsync(user);
    await context.SaveChangesAsync();
    
    // Act
    var result = await repository.GetByIdAsync(1);
    
    // Assert
    Assert.AreEqual("Test User", result.Name);
}
```

### **Performance Implications:**
- **Additional Layer Overhead**: ~1-5% performance penalty
- **Query Efficiency**: May not expose optimal query methods (Include, projection)
- **Memory Usage**: Generic repositories can prevent query optimization
- **Caching Challenges**: Harder to implement entity-specific caching strategies

### **Real-World Alternatives:**
- **Direct DbContext Usage**: For simple applications with straightforward data access
- **CQRS with MediatR**: Separate command/query handlers instead of repositories  
- **Dapper with Query Objects**: Lightweight ORM with specific query classes
- **Entity-Specific Services**: Domain services that encapsulate business logic with data access

---

## **3. MIDDLEWARE CREATION**

### **What Problem It Solves:**
- **Cross-Cutting Concerns**: Handles logging, authentication, error handling globally
- **Request Pipeline Control**: Process requests before/after reaching controllers
- **Separation of Concerns**: Keeps controllers focused on business logic
- **Reusability**: Same middleware can be used across multiple applications

### **When NOT to Use It:**
- **Business Logic**: Don't put domain logic in middleware
- **Controller-Specific Logic**: Use action filters instead for controller-specific concerns
- **Complex Conditional Logic**: Middleware runs for every request - avoid heavy processing
- **State Management**: Middleware should be stateless

### Code snippet
```csharp
// =============================================================================
// 3. MIDDLEWARE CREATION
// =============================================================================

// Custom Exception Handling Middleware
public class ExceptionHandlingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<ExceptionHandlingMiddleware> _logger;

    public ExceptionHandlingMiddleware(RequestDelegate next, ILogger<ExceptionHandlingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "An unhandled exception occurred");
            await HandleExceptionAsync(context, ex);
        }
    }

    private static async Task HandleExceptionAsync(HttpContext context, Exception exception)
    {
        context.Response.ContentType = "application/json";
        context.Response.StatusCode = exception switch
        {
            ValidationException => 400,
            NotFoundException => 404,
            UnauthorizedAccessException => 401,
            _ => 500
        };

        var response = new { message = exception.Message, statusCode = context.Response.StatusCode };
        await context.Response.WriteAsync(JsonSerializer.Serialize(response));
    }
}

// Registration: app.UseMiddleware<ExceptionHandlingMiddleware>();
```

### **How to Test It:**
```csharp
[Test]
public async Task Middleware_Should_Handle_Exception_And_Return_500()
{
    // Arrange
    var context = new DefaultHttpContext();
    context.Response.Body = new MemoryStream();
    var logger = new Mock<ILogger<ExceptionHandlingMiddleware>>();
    
    RequestDelegate next = (ctx) => throw new Exception("Test exception");
    var middleware = new ExceptionHandlingMiddleware(next, logger.Object);
    
    // Act
    await middleware.InvokeAsync(context);
    
    // Assert
    Assert.AreEqual(500, context.Response.StatusCode);
    Assert.AreEqual("application/json", context.Response.ContentType);
}
```

### **Performance Implications:**
- **Pipeline Overhead**: Each middleware adds to request processing time
- **Exception Handling Cost**: Try-catch blocks have minimal performance impact until exception occurs
- **Memory Allocation**: Avoid creating objects in middleware hot paths
- **Async/Await Overhead**: Proper async implementation crucial for scalability

### **Real-World Alternatives:**
- **Action Filters**: For controller-specific cross-cutting concerns
- **Global Exception Filters**: ASP.NET Core built-in exception handling
- **Middleware Libraries**: Serilog, Application Insights for logging/monitoring
- **API Gateway**: External solutions like Azure API Management, Kong

---

## **4. CUSTOM ATTRIBUTE VALIDATION**

### **What Problem It Solves:**
- **Reusable Validation Logic**: Same validation rules across multiple models
- **Declarative Validation**: Clear, self-documenting validation requirements
- **Framework Integration**: Automatic validation in ASP.NET Core model binding
- **Custom Business Rules**: Implement domain-specific validation logic

### **When NOT to Use It:**
- **Complex Cross-Field Validation**: Use IValidatableObject or FluentValidation
- **Database-Dependent Validation**: Validation attributes should be stateless
- **Dynamic Validation Rules**: When rules change based on runtime conditions
- **Performance-Critical Paths**: Attribute instantiation/reflection has overhead

### Code snippet
```csharp
// =============================================================================
// 4. CUSTOM ATTRIBUTE VALIDATION
// =============================================================================

// Custom Validation Attribute
public class ValidEmailDomainAttribute : ValidationAttribute
{
    private readonly string[] _allowedDomains;

    public ValidEmailDomainAttribute(params string[] allowedDomains)
    {
        _allowedDomains = allowedDomains;
    }

    protected override ValidationResult IsValid(object value, ValidationContext validationContext)
    {
        if (value is string email && !string.IsNullOrEmpty(email))
        {
            var domain = email.Split('@').LastOrDefault();
            if (_allowedDomains.Contains(domain, StringComparer.OrdinalIgnoreCase))
                return ValidationResult.Success;
            
            return new ValidationResult($"Email domain must be one of: {string.Join(", ", _allowedDomains)}");
        }
        return new ValidationResult("Email is required");
    }
}

// Usage in Model
public class CreateUserRequest
{
    [Required]
    [ValidEmailDomain("company.com", "enterprise.com")]
    public string Email { get; set; }
    
    [Required]
    [StringLength(50, MinimumLength = 2)]
    public string FirstName { get; set; }
}
```

### **How to Test It:**
```csharp
[Test]
public void ValidEmailDomainAttribute_Should_Validate_Allowed_Domain()
{
    // Arrange
    var attribute = new ValidEmailDomainAttribute("company.com");
    var context = new ValidationContext(new object());
    
    // Act
    var result = attribute.GetValidationResult("user@company.com", context);
    
    // Assert
    Assert.AreEqual(ValidationResult.Success, result);
}

[Test]
public void ValidEmailDomainAttribute_Should_Fail_Invalid_Domain()
{
    // Arrange
    var attribute = new ValidEmailDomainAttribute("company.com");
    var context = new ValidationContext(new object());
    
    // Act
    var result = attribute.GetValidationResult("user@invalid.com", context);
    
    // Assert
    Assert.IsNotNull(result);
    Assert.IsTrue(result.ErrorMessage.Contains("Email domain must be"));
}
```

### **Performance Implications:**
- **Reflection Overhead**: Attribute discovery and instantiation uses reflection
- **Validation Cost**: Runs on every model binding operation
- **Memory Allocation**: Each validation creates error message objects
- **Caching**: .NET Core caches validation attributes to improve performance

### **Real-World Alternatives:**
- **FluentValidation**: More powerful, testable validation library
- **Manual Validation**: Explicit validation in services/controllers
- **Data Annotations**: Built-in attributes (Required, Range, StringLength)
- **JSON Schema Validation**: For API payload validation

---

## **5. STRATEGY PATTERN FOR BUSINESS RULES**

### **What Problem It Solves:**
- **Algorithm Variation**: Multiple ways to perform the same operation
- **Open/Closed Principle**: Add new strategies without modifying existing code
- **Runtime Selection**: Choose strategy based on runtime conditions
- **Testability**: Each strategy can be tested in isolation

### **When NOT to Use It:**
- **Simple Conditional Logic**: Basic if/else statements might be clearer
- **Stable Algorithms**: When business rules rarely change
- **Performance-Critical Code**: Strategy selection adds indirection overhead
- **Few Variations**: Two strategies might not justify the pattern complexity

### Code snippet
```csharp
// =============================================================================
// 5. STRATEGY PATTERN FOR BUSINESS RULES
// =============================================================================

// Payment Strategy Interface
public interface IPaymentStrategy
{
    Task<PaymentResult> ProcessPaymentAsync(PaymentRequest request);
    bool CanProcess(PaymentMethod method);
}

// Concrete Strategies
public class CreditCardPaymentStrategy : IPaymentStrategy
{
    public async Task<PaymentResult> ProcessPaymentAsync(PaymentRequest request)
    {
        // Credit card processing logic
        return new PaymentResult { Success = true, TransactionId = Guid.NewGuid().ToString() };
    }
    public bool CanProcess(PaymentMethod method) => method == PaymentMethod.CreditCard;
}

public class PayPalPaymentStrategy : IPaymentStrategy
{
    public async Task<PaymentResult> ProcessPaymentAsync(PaymentRequest request)
    {
        // PayPal processing logic  
        return new PaymentResult { Success = true, TransactionId = $"PP_{Guid.NewGuid()}" };
    }
    public bool CanProcess(PaymentMethod method) => method == PaymentMethod.PayPal;
}

// Context Class
public class PaymentProcessor
{
    private readonly IEnumerable<IPaymentStrategy> _strategies;

    public PaymentProcessor(IEnumerable<IPaymentStrategy> strategies) => _strategies = strategies;

    public async Task<PaymentResult> ProcessAsync(PaymentRequest request)
    {
        var strategy = _strategies.FirstOrDefault(s => s.CanProcess(request.PaymentMethod));
        return strategy != null ? await strategy.ProcessPaymentAsync(request) 
                               : new PaymentResult { Success = false, Error = "Payment method not supported" };
    }
}
```

### **How to Test It:**
```csharp
[Test]
public async Task PaymentProcessor_Should_Use_Correct_Strategy()
{
    // Arrange
    var strategies = new List<IPaymentStrategy>
    {
        new CreditCardPaymentStrategy(),
        new PayPalPaymentStrategy()
    };
    var processor = new PaymentProcessor(strategies);
    var request = new PaymentRequest { PaymentMethod = PaymentMethod.PayPal };
    
    // Act
    var result = await processor.ProcessAsync(request);
    
    // Assert
    Assert.IsTrue(result.Success);
    Assert.IsTrue(result.TransactionId.StartsWith("PP_"));
}
```

### **Performance Implications:**
- **Strategy Selection Overhead**: LINQ FirstOrDefault adds ~10-50 nanoseconds
- **Interface Dispatch**: Virtual method calls have minimal overhead
- **Memory Usage**: All strategies loaded in memory (consider lazy loading)
- **DI Resolution**: Strategy enumeration resolved from DI container

### **Real-World Alternatives:**
- **Switch Expressions**: For simple, stable conditional logic
- **Chain of Responsibility**: When multiple handlers might process the same request
- **State Pattern**: When behavior changes based on object state
- **Command Pattern**: When you need to encapsulate requests as objects

---

## **6. FACTORY PATTERN FOR SERVICE CREATION**

### **What Problem It Solves:**
- **Object Creation Complexity**: Encapsulates complex object instantiation logic
- **Runtime Type Selection**: Create different types based on runtime parameters
- **Dependency Management**: Handle constructor dependencies centrally
- **Abstraction**: Client doesn't need to know concrete implementation details

### **When NOT to Use It:**
- **Simple Object Creation**: Basic constructors don't need factories
- **DI Container Sufficient**: When DI container handles all creation needs
- **Single Implementation**: No need for factory with only one concrete type
- **Performance-Critical Creation**: Direct instantiation is faster

### Code snippet
```csharp
// =============================================================================
// 6. FACTORY PATTERN FOR SERVICE CREATION
// =============================================================================

// Abstract Factory Interface
public interface INotificationServiceFactory
{
    INotificationService CreateNotificationService(NotificationType type);
}

// Concrete Factory
public class NotificationServiceFactory : INotificationServiceFactory
{
    private readonly IServiceProvider _serviceProvider;

    public NotificationServiceFactory(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }

    public INotificationService CreateNotificationService(NotificationType type)
    {
        return type switch
        {
            NotificationType.Email => _serviceProvider.GetService<IEmailNotificationService>(),
            NotificationType.SMS => _serviceProvider.GetService<ISmsNotificationService>(),
            NotificationType.Push => _serviceProvider.GetService<IPushNotificationService>(),
            _ => throw new ArgumentException($"Notification type {type} is not supported")
        };
    }
}

// Usage in Controller
[ApiController]
public class NotificationController : ControllerBase
{
    private readonly INotificationServiceFactory _factory;

    [HttpPost("send")]
    public async Task<IActionResult> SendNotification(NotificationRequest request)
    {
        var service = _factory.CreateNotificationService(request.Type);
        await service.SendAsync(request.Message, request.Recipients);
        return Ok();
    }
}

// Supporting Classes
public enum PaymentMethod { CreditCard, PayPal, BankTransfer }
public enum NotificationType { Email, SMS, Push }
public class PaymentRequest { public PaymentMethod PaymentMethod { get; set; } public decimal Amount { get; set; } }
public class PaymentResult { public bool Success { get; set; } public string TransactionId { get; set; } public string Error { get; set; } }
public class NotificationRequest { public NotificationType Type { get; set; } public string Message { get; set; } public List<string> Recipients { get; set; } }
```

### **How to Test It:**
```csharp
[Test]
public void Factory_Should_Create_Email_Service_For_Email_Type()
{
    // Arrange
    var services = new ServiceCollection();
    services.AddTransient<IEmailNotificationService, EmailNotificationService>();
    var serviceProvider = services.BuildServiceProvider();
    var factory = new NotificationServiceFactory(serviceProvider);
    
    // Act
    var service = factory.CreateNotificationService(NotificationType.Email);
    
    // Assert
    Assert.IsInstanceOf<IEmailNotificationService>(service);
}

[Test]
public void Factory_Should_Throw_For_Unsupported_Type()
{
    // Arrange
    var factory = new NotificationServiceFactory(Mock.Of<IServiceProvider>());
    
    // Act & Assert
    Assert.Throws<ArgumentException>(() => 
        factory.CreateNotificationService((NotificationType)999));
}
```

### **Performance Implications:**
- **Factory Method Overhead**: Additional method call and switch statement
- **DI Resolution Cost**: ServiceProvider.GetService adds resolution overhead
- **Memory Usage**: Factory instances consume memory even when not creating objects
- **Caching Opportunities**: Can cache created instances if appropriate

### **Real-World Alternatives:**
- **Generic Factory**: `IFactory<T>` for type-safe creation
- **Abstract Factory**: When you need families of related objects
- **Builder Pattern**: For complex objects requiring step-by-step construction
- **DI Container Features**: Named registrations, factory delegates in modern containers
- **Function Factories**: `Func<TParam, TResult>` delegates for simple factory scenarios

---

## **INTERVIEW SUCCESS TIPS**

1. **Start with the Problem**: Always explain what problem the pattern solves first
2. **Show Trade-offs**: Mention when NOT to use it - demonstrates mature thinking
3. **Performance Awareness**: Show you understand the performance implications
4. **Testing Strategy**: Explain how you would test the implementation
5. **Real-world Experience**: Mention alternatives you've used in production
6. **Keep It Concise**: 10-15 lines max for code examples, detailed verbal explanation

These patterns and explanations will help you demonstrate deep technical understanding in your interviews!

--- 

# .NET Core & Web API Interview Questions - Design Patterns & Architecture

## **DESIGN PATTERNS QUESTIONS**

### **Creational Patterns**
1. **Dependency Injection (DI)**
   - How do you implement DI in .NET Core? What are the different service lifetimes?
   - What problems does DI solve? How does it improve testability?
   - Explain Scoped vs Transient vs Singleton with real examples.
   - How would you resolve circular dependencies in DI?

---
I'll provide you with a comprehensive guide to Dependency Injection (DI) from a solution-oriented architect's perspective, perfect for your technical interview preparation.This comprehensive guide covers all aspects of Dependency Injection from an architect's perspective. Here are the key points to remember for your interview:

**Core Interview Points:**
1. **Problem-Solution Focus**: Always start by explaining the tight coupling problem DI solves
2. **SOLID Principles**: Emphasize how DI directly implements Dependency Inversion Principle
3. **Lifetime Management**: Be crystal clear about Singleton vs Scoped vs Transient with real examples
4. **Circular Dependencies**: Show you understand architectural design by explaining resolution strategies

**Practical Demonstration:**
- Be ready to write constructor injection code on a whiteboard
- Explain how you'd mock dependencies in unit tests
- Discuss performance implications and when to use each lifetime
- Show how DI enables Clean Architecture patterns

**Advanced Topics:**
- Factory pattern integration with DI
- Event-driven architecture to resolve circular dependencies
- Cloud-native considerations for service lifetimes
- Thread safety implications

The guide provides interview-ready code examples (10-15 lines each) and real-world scenarios that demonstrate your architectural thinking. Focus on explaining the "why" behind each decision, as senior interviews often test your reasoning more than your coding ability.

# Dependency Injection (DI) Pattern - Complete Guide

## Core Category of Pattern
**Creational Pattern** - Structural Implementation Pattern
- Falls under IoC (Inversion of Control) container patterns
- Architectural pattern for managing object dependencies

## What is this Pattern?
Dependency Injection is a design pattern that implements Inversion of Control (IoC) for resolving dependencies. Instead of objects creating their own dependencies, an external entity (IoC container) injects the required dependencies.

**Key Components:**
- **Service**: The dependency being injected
- **Client**: The class that depends on the service
- **Injector**: The IoC container that provides the dependency
- **Interface**: Contract defining the service

## What Problem Does It Solve?
1. **Tight Coupling**: Eliminates direct instantiation of dependencies
2. **Hard-to-Test Code**: Makes unit testing easier with mock injections
3. **Violation of SOLID Principles**: Ensures adherence to DIP and SRP
4. **Configuration Management**: Centralizes dependency configuration
5. **Object Lifecycle Management**: Handles creation, scope, and disposal

## Why Should I Use This Pattern?
- **Testability**: Easy mocking and unit testing
- **Maintainability**: Loose coupling between components
- **Flexibility**: Easy to swap implementations
- **Separation of Concerns**: Clear responsibility boundaries
- **Configuration Centralization**: Single place to manage dependencies

## When Should I Use This Pattern?
**Better fit than:**
- **Factory Pattern**: When you need lifecycle management
- **Service Locator**: When you want to avoid service location anti-pattern
- **Manual instantiation**: In complex applications with many dependencies

**Use when:**
- Building enterprise applications
- Need different implementations (dev/prod)
- Require comprehensive testing
- Building microservices
- Implementing Clean Architecture

## Use Cases of This Pattern
1. **Database Access**: Injecting repository implementations
2. **External Services**: API clients, email services
3. **Configuration**: Settings and environment-specific configs
4. **Logging**: Logger implementations
5. **Caching**: Cache providers
6. **Authentication**: Identity services
7. **Business Logic**: Domain services

## Implementation in .NET Framework vs .NET Core

### .NET Framework (Using Autofac/Unity)
```csharp
// Registration
var builder = new ContainerBuilder();
builder.RegisterType<UserRepository>().As<IUserRepository>();
builder.RegisterType<UserService>().As<IUserService>();
var container = builder.Build();

// Usage
public class UserController : Controller
{
    private readonly IUserService _userService;
    
    public UserController(IUserService userService)
    {
        _userService = userService;
    }
}
```

### .NET Core (Built-in DI)
```csharp
// Startup.cs / Program.cs
services.AddScoped<IUserRepository, UserRepository>();
services.AddScoped<IUserService, UserService>();

// Usage
public class UserController : ControllerBase
{
    private readonly IUserService _userService;
    
    public UserController(IUserService userService)
    {
        _userService = userService;
    }
}
```

## Architecture Relevance
- **Clean Architecture**: Supports dependency rule (inner layers don't depend on outer)
- **Hexagonal Architecture**: Enables port/adapter pattern
- **Microservices**: Facilitates service boundaries and testing
- **Domain-Driven Design**: Supports repository and domain service patterns
- **CQRS**: Separates command and query handlers

## SOLID Principles Connection

### Single Responsibility Principle (SRP)
Classes focus on their core responsibility, not dependency creation

### Open/Closed Principle (OCP)
Easy to extend with new implementations without modifying existing code

### Liskov Substitution Principle (LSP)
Any implementation can be substituted through interfaces

### Interface Segregation Principle (ISP)
Dependencies are injected through focused interfaces

### Dependency Inversion Principle (DIP)
**Direct Implementation** - High-level modules depend on abstractions, not concretions

## UML Diagram

```
┌─────────────────┐    depends on    ┌──────────────────┐
│   UserService   │ ───────────────→ │ IUserRepository  │
│                 │                  │                  │
│ + GetUser()     │                  │ + GetById()      │
│ + CreateUser()  │                  │ + Save()         │
└─────────────────┘                  └──────────────────┘
         ↑                                    ↑
         │                                    │
    injected by                          implements
         │                                    │
         ↓                                    ↓
┌─────────────────┐                  ┌──────────────────┐
│  DI Container   │                  │ UserRepository   │
│                 │                  │                  │
│ + Resolve<T>()  │                  │ + GetById()      │
└─────────────────┘                  │ + Save()         │
                                     └──────────────────┘
```

## Concurrency and Cloud Context

### Concurrency Considerations
```csharp
// Thread-safe singleton
services.AddSingleton<IThreadSafeService, ThreadSafeService>();

// Scoped per request (thread-safe in web context)
services.AddScoped<IRequestScopedService, RequestScopedService>();
```

### Cloud-Native Benefits
- **Scalability**: Easy to swap implementations for different environments
- **Configuration**: Environment-specific services
- **Health Checks**: Inject health check services
- **Distributed Tracing**: Inject telemetry services

## Trade-offs & Misuse

### Benefits
- ✅ Testability and maintainability
- ✅ Loose coupling
- ✅ Configuration flexibility
- ✅ Lifecycle management

### Drawbacks
- ❌ Runtime errors for missing dependencies
- ❌ Performance overhead
- ❌ Increased complexity
- ❌ Potential memory leaks with incorrect scoping

### Common Misuse
1. **Over-injection**: Injecting simple values instead of using configuration
2. **Wrong lifetimes**: Using singleton for stateful services
3. **Circular dependencies**: Poor design leading to circular refs
4. **Service Locator anti-pattern**: Injecting container itself

## Risks in Multi-thread/Cloud Native World

### Multi-threading Risks
```csharp
// ❌ Dangerous - Singleton with mutable state
services.AddSingleton<ICounterService, CounterService>(); // If not thread-safe

// ✅ Safe approaches
services.AddSingleton<IImmutableConfigService, ConfigService>();
services.AddScoped<IRequestContextService, RequestContextService>();
```

### Cloud-Native Risks
- **Memory leaks**: Incorrect singleton usage
- **Resource exhaustion**: Not disposing services properly
- **Configuration drift**: Different environments with different services

## Drawbacks of Not Using DI

1. **Testing Nightmare**: Hard to mock dependencies
2. **Tight Coupling**: Changes cascade through codebase
3. **Configuration Scattered**: Dependencies created everywhere
4. **Violation of SOLID**: Especially DIP and SRP
5. **Maintenance Issues**: Hard to change implementations
6. **No Lifecycle Management**: Manual memory management

## Code Examples

### Basic Implementation
```csharp
public interface IEmailService
{
    Task SendAsync(string to, string subject, string body);
}

public class EmailService : IEmailService
{
    private readonly IConfiguration _config;
    
    public EmailService(IConfiguration config) => _config = config;
    
    public async Task SendAsync(string to, string subject, string body)
    {
        // Implementation using _config
        await Task.CompletedTask;
    }
}

// Registration
services.AddScoped<IEmailService, EmailService>();
```

### Constructor Injection
```csharp
public class OrderService
{
    private readonly IOrderRepository _repo;
    private readonly IEmailService _email;
    private readonly ILogger<OrderService> _logger;
    
    public OrderService(IOrderRepository repo, IEmailService email, ILogger<OrderService> logger)
    {
        _repo = repo;
        _email = email;
        _logger = logger;
    }
    
    public async Task ProcessOrderAsync(Order order)
    {
        await _repo.SaveAsync(order);
        await _email.SendAsync(order.CustomerEmail, "Order Confirmed", "Thank you!");
        _logger.LogInformation("Order {OrderId} processed", order.Id);
    }
}
```

## Scoped vs Transient vs Singleton with Real Examples

### Singleton - One instance for application lifetime
```csharp
// ✅ Good for: Configuration, thread-safe caches
services.AddSingleton<IApplicationSettings, ApplicationSettings>();
services.AddSingleton<IMemoryCache, MemoryCache>();

public class ApplicationSettings : IApplicationSettings
{
    public string ConnectionString { get; } = "Server=...";
    public int MaxRetries { get; } = 3;
}
```

### Scoped - One instance per request/scope
```csharp
// ✅ Good for: Database contexts, request-specific services
services.AddScoped<IUserRepository, UserRepository>();
services.AddScoped<DbContext, ApplicationDbContext>();

public class UserRepository : IUserRepository
{
    private readonly DbContext _context;
    
    public UserRepository(DbContext context) => _context = context;
    
    public async Task<User> GetByIdAsync(int id) => 
        await _context.Users.FindAsync(id);
}
```

### Transient - New instance every time
```csharp
// ✅ Good for: Lightweight, stateless services
services.AddTransient<IEmailService, EmailService>();
services.AddTransient<IMapper, AutoMapper.Mapper>();

public class NotificationService
{
    public async Task SendWelcomeEmail(User user)
    {
        // New EmailService instance each time
        var emailService = serviceProvider.GetService<IEmailService>();
        await emailService.SendAsync(user.Email, "Welcome", "Welcome to our app!");
    }
}
```

## Resolving Circular Dependencies

### Problem Example
```csharp
// ❌ Circular dependency
public class ServiceA
{
    public ServiceA(ServiceB serviceB) { }
}

public class ServiceB  
{
    public ServiceB(ServiceA serviceA) { }
}
```

### Solution 1: Redesign (Preferred)
```csharp
// ✅ Extract common dependency
public interface ISharedService { }

public class ServiceA
{
    public ServiceA(ISharedService shared) { }
}

public class ServiceB
{
    public ServiceB(ISharedService shared) { }
}
```

### Solution 2: Factory Pattern
```csharp
// ✅ Use factory to break cycle
public interface IServiceBFactory
{
    ServiceB Create();
}

public class ServiceA
{
    private readonly IServiceBFactory _serviceBFactory;
    
    public ServiceA(IServiceBFactory serviceBFactory) => 
        _serviceBFactory = serviceBFactory;
    
    public void DoWork()
    {
        var serviceB = _serviceBFactory.Create();
        // Use serviceB
    }
}
```

### Solution 3: Event-Driven Architecture
```csharp
// ✅ Use events to decouple
public class ServiceA
{
    private readonly IEventBus _eventBus;
    
    public ServiceA(IEventBus eventBus) => _eventBus = eventBus;
    
    public void DoWork()
    {
        _eventBus.Publish(new ServiceAEvent());
    }
}

public class ServiceB
{
    public ServiceB(IEventBus eventBus) => 
        eventBus.Subscribe<ServiceAEvent>(Handle);
    
    private void Handle(ServiceAEvent evt) { /* Handle event */ }
}
```

## Interview Tips

1. **Start with the problem**: Always explain what problem DI solves
2. **Show SOLID connection**: Emphasize DIP implementation
3. **Demonstrate lifetime understanding**: Know when to use each scope
4. **Discuss testing benefits**: Show how DI enables mocking
5. **Address performance**: Acknowledge overhead but explain benefits
6. **Show circular dependency resolution**: Demonstrate architectural thinking
7. **Connect to real scenarios**: Use examples from your experience

## Key Takeaways for Architects

- DI is fundamental to modern .NET applications
- Choose appropriate lifetimes based on state and performance needs
- Design for testability from the start
- Avoid circular dependencies through better design
- Use DI containers as composition roots
- Consider cloud-native implications for service lifetimes
---

2. **Factory Pattern**
   - When would you use Factory pattern in Web API? Show implementation.
   - How is Factory different from Abstract Factory? Give scenarios.
   - How do you combine Factory with DI container?

---
I'll provide you with a comprehensive, architect-level overview of the Factory Pattern that will prepare you for senior .NET technical interviews.This comprehensive guide covers all the aspects you requested for the Factory Pattern from a solution-oriented architect's perspective. The content is structured to help you ace technical interviews at the senior level with:

**Key Interview Strengths:**
- Real-world enterprise examples (payment processing, cloud services)
- SOLID principles integration with code examples
- Threading and cloud-native considerations
- Practical Web API implementation with DI
- Clear distinction between Factory variants
- Architecture trade-offs and decision criteria

**Interview-Ready Features:**
- Short, focused code examples (10-15 lines) with detailed explanations
- UML diagram using Mermaid syntax
- Common misuses and anti-patterns
- Enterprise use cases you'll encounter in real projects

The examples use modern C# features (pattern matching, async/await, nullable reference types) and follow current .NET best practices. Each code block includes detailed comments explaining the architectural decisions and benefits.

# Factory Pattern - Complete .NET Interview Guide

## Core Category of Pattern
**Creational Design Pattern** - Part of the Gang of Four (GoF) design patterns, specifically focused on object creation mechanisms.

## What is Factory Pattern?
The Factory Pattern provides an interface for creating objects without specifying their exact classes. It encapsulates object creation logic in a separate method or class, promoting loose coupling between client code and concrete implementations.

**Types of Factory Pattern:**
- Simple Factory (Factory Method)
- Factory Method Pattern
- Abstract Factory Pattern

## What Problem Does It Solve?

### Primary Problems:
1. **Tight Coupling**: Direct instantiation creates dependencies on concrete classes
2. **Violates Open/Closed Principle**: Adding new types requires modifying existing code
3. **Complex Object Creation**: When object creation involves complex logic or dependencies
4. **Runtime Type Determination**: When the exact type to instantiate is determined at runtime

### Example Problem:
```csharp
// Problematic approach - tight coupling
public class PaymentProcessor
{
    public void ProcessPayment(string type, decimal amount)
    {
        if (type == "Credit")
        {
            var payment = new CreditCardPayment(); // Direct dependency
            payment.Process(amount);
        }
        else if (type == "PayPal")
        {
            var payment = new PayPalPayment(); // Direct dependency
            payment.Process(amount);
        }
        // Adding new payment types requires modifying this method
    }
}
```

## Why Should I Use This Pattern?

### Architecture Benefits:
1. **Separation of Concerns**: Object creation is separated from business logic
2. **Maintainability**: Easy to add new types without modifying existing code
3. **Testability**: Can inject mock factories for unit testing
4. **Flexibility**: Runtime decision making for object creation
5. **Code Reusability**: Factory logic can be reused across different contexts

## When Should I Use This Pattern?

### Better Fit Scenarios:
- **Over Direct Instantiation**: When you have multiple related classes
- **Over Conditional Logic**: Replace complex if-else chains for object creation
- **Over Static Methods**: When you need polymorphic behavior
- **With Strategy Pattern**: Factory creates strategies based on context
- **In Plugin Architecture**: Dynamic loading of implementations

### Red Flags for Usage:
- Simple object creation without complexity
- Only one implementation exists
- Performance-critical scenarios where factory overhead matters

## Use Cases of Factory Pattern

### Enterprise Scenarios:
1. **Database Connection Management**: Create different database providers
2. **Cloud Service Abstraction**: Switch between AWS, Azure, GCP services
3. **Message Queue Providers**: RabbitMQ, Azure Service Bus, Amazon SQS
4. **Authentication Providers**: JWT, OAuth, SAML
5. **File Storage Services**: Local, Azure Blob, AWS S3
6. **Logging Frameworks**: Serilog, NLog, log4net
7. **Payment Gateways**: Stripe, PayPal, Square
8. **Notification Services**: Email, SMS, Push notifications

## Implementation in .NET Framework and .NET Core

### Simple Factory Implementation:
```csharp
// Product interface
public interface IPaymentProcessor
{
    Task<PaymentResult> ProcessAsync(decimal amount);
}

// Concrete implementations
public class CreditCardProcessor : IPaymentProcessor
{
    public async Task<PaymentResult> ProcessAsync(decimal amount)
    {
        // Credit card specific processing logic
        await Task.Delay(100); // Simulate API call
        return new PaymentResult { Success = true, TransactionId = Guid.NewGuid().ToString() };
    }
}

public class PayPalProcessor : IPaymentProcessor
{
    public async Task<PaymentResult> ProcessAsync(decimal amount)
    {
        // PayPal specific processing logic
        await Task.Delay(150); // Simulate API call
        return new PaymentResult { Success = true, TransactionId = Guid.NewGuid().ToString() };
    }
}

// Factory class - encapsulates creation logic
public static class PaymentProcessorFactory
{
    // Factory method - returns interface, not concrete type
    public static IPaymentProcessor CreateProcessor(PaymentType type)
    {
        return type switch
        {
            PaymentType.CreditCard => new CreditCardProcessor(),
            PaymentType.PayPal => new PayPalProcessor(),
            _ => throw new ArgumentException($"Unsupported payment type: {type}")
        };
    }
}

// Usage - client code is decoupled from concrete implementations
public class PaymentService
{
    public async Task<PaymentResult> ProcessPaymentAsync(PaymentType type, decimal amount)
    {
        // Factory handles object creation complexity
        var processor = PaymentProcessorFactory.CreateProcessor(type);
        return await processor.ProcessAsync(amount);
    }
}
```

### Factory Method Pattern (More Flexible):
```csharp
// Abstract creator class
public abstract class PaymentProcessorCreator
{
    // Factory method - subclasses decide which class to instantiate
    protected abstract IPaymentProcessor CreateProcessor();
    
    // Template method using factory method
    public async Task<PaymentResult> ProcessPaymentAsync(decimal amount)
    {
        var processor = CreateProcessor(); // Delegation to subclass
        return await processor.ProcessAsync(amount);
    }
}

// Concrete creators
public class CreditCardProcessorCreator : PaymentProcessorCreator
{
    protected override IPaymentProcessor CreateProcessor()
    {
        return new CreditCardProcessor(); // Specific implementation
    }
}

public class PayPalProcessorCreator : PaymentProcessorCreator
{
    protected override IPaymentProcessor CreateProcessor()
    {
        return new PayPalProcessor(); // Specific implementation
    }
}
```

## Architecture Relevance

### Microservices Architecture:
- **Service Discovery**: Factory pattern for creating service clients
- **Configuration Management**: Different configurations per environment
- **Cross-cutting Concerns**: Logging, monitoring, security implementations

### Clean Architecture:
- **Infrastructure Layer**: Factories for external service implementations
- **Application Layer**: Use case specific object creation
- **Domain Layer**: Domain service factories

### Event-Driven Architecture:
- **Event Handler Factories**: Create appropriate handlers for event types
- **Message Serializer Factories**: Choose serialization strategy

## SOLID Principles Connection

### Single Responsibility Principle (SRP):
- Factory has single responsibility: object creation
- Business logic separated from creation logic

### Open/Closed Principle (OCP):
```csharp
// Open for extension - add new payment types
public class DigitalWalletProcessor : IPaymentProcessor
{
    public async Task<PaymentResult> ProcessAsync(decimal amount)
    {
        // Digital wallet implementation
        return await Task.FromResult(new PaymentResult { Success = true });
    }
}

// Closed for modification - extend factory without changing existing code
public static class PaymentProcessorFactory
{
    public static IPaymentProcessor CreateProcessor(PaymentType type)
    {
        return type switch
        {
            PaymentType.CreditCard => new CreditCardProcessor(),
            PaymentType.PayPal => new PayPalProcessor(),
            PaymentType.DigitalWallet => new DigitalWalletProcessor(), // New addition
            _ => throw new ArgumentException($"Unsupported payment type: {type}")
        };
    }
}
```

### Liskov Substitution Principle (LSP):
- All concrete implementations are substitutable through common interface

### Interface Segregation Principle (ISP):
- Factories depend on abstractions, not concrete implementations

### Dependency Inversion Principle (DIP):
- High-level modules depend on abstractions (interfaces)
- Factory provides concrete implementations

## UML Diagram

```mermaid
classDiagram
    class Client {
        +ProcessPayment()
    }
    
    class PaymentProcessorFactory {
        +CreateProcessor(type: PaymentType): IPaymentProcessor
    }
    
    class IPaymentProcessor {
        <<interface>>
        +ProcessAsync(amount: decimal): Task~PaymentResult~
    }
    
    class CreditCardProcessor {
        +ProcessAsync(amount: decimal): Task~PaymentResult~
    }
    
    class PayPalProcessor {
        +ProcessAsync(amount: decimal): Task~PaymentResult~
    }
    
    class PaymentType {
        <<enumeration>>
        CreditCard
        PayPal
    }
    
    Client --> PaymentProcessorFactory : uses
    PaymentProcessorFactory --> IPaymentProcessor : creates
    PaymentProcessorFactory ..> PaymentType : depends on
    CreditCardProcessor ..|> IPaymentProcessor : implements
    PayPalProcessor ..|> IPaymentProcessor : implements
    PaymentProcessorFactory ..> CreditCardProcessor : creates
    PaymentProcessorFactory ..> PayPalProcessor : creates
```

## Concurrency and Cloud Context

### Thread Safety Considerations:
```csharp
// Thread-safe singleton factory
public sealed class ThreadSafePaymentFactory
{
    private static readonly ConcurrentDictionary<PaymentType, Func<IPaymentProcessor>> 
        _processors = new();
    
    static ThreadSafePaymentFactory()
    {
        // Initialize factory functions - thread-safe
        _processors[PaymentType.CreditCard] = () => new CreditCardProcessor();
        _processors[PaymentType.PayPal] = () => new PayPalProcessor();
    }
    
    public static IPaymentProcessor CreateProcessor(PaymentType type)
    {
        // Thread-safe lookup and creation
        return _processors.TryGetValue(type, out var factory) 
            ? factory() 
            : throw new ArgumentException($"Unsupported payment type: {type}");
    }
}
```

### Cloud-Native Considerations:
- **Configuration-driven**: Use cloud configuration services
- **Service Discovery**: Integration with service meshes
- **Resilience**: Circuit breaker pattern integration
- **Observability**: Distributed tracing and metrics

## Trade-offs & Misuse

### Trade-offs:
**Pros:**
- Flexibility and extensibility
- Loose coupling
- Testability
- Maintainability

**Cons:**
- Additional complexity for simple scenarios
- Performance overhead
- More classes to maintain
- Potential over-engineering

### Common Misuses:
1. **Over-engineering**: Using factory for single implementation
2. **God Factory**: Factory that creates too many different types
3. **Leaky Abstraction**: Factory returning concrete types instead of interfaces
4. **State Management**: Factories shouldn't maintain state

## Risks in Multi-thread/Cloud Native World

### Multi-threading Risks:
1. **Race Conditions**: Non-thread-safe factory implementations
2. **Shared State**: Factories maintaining mutable state
3. **Resource Contention**: Multiple threads creating expensive objects

### Cloud Native Risks:
1. **Configuration Drift**: Different configurations across environments
2. **Service Discovery**: Factory unable to handle dynamic service endpoints
3. **Resilience**: No fallback mechanisms for failed object creation
4. **Memory Leaks**: Not properly disposing created objects in containerized environments

## Drawbacks of Not Using Factory Pattern

### Technical Debt:
1. **Tight Coupling**: Difficult to change implementations
2. **Violates SOLID**: Especially OCP and DIP
3. **Testing Challenges**: Hard to mock dependencies
4. **Maintenance Overhead**: Changes require modifications in multiple places

### Business Impact:
1. **Slow Feature Delivery**: Ripple effects across codebase
2. **Higher Bug Risk**: Changes affect multiple components
3. **Integration Complexity**: Difficult to switch third-party services

## Web API Implementation Example

```csharp
// Startup.cs or Program.cs (.NET 6+)
public void ConfigureServices(IServiceCollection services)
{
    // Register factory and implementations
    services.AddScoped<IPaymentProcessorFactory, PaymentProcessorFactory>();
    services.AddScoped<CreditCardProcessor>();
    services.AddScoped<PayPalProcessor>();
}

// Factory interface for DI
public interface IPaymentProcessorFactory
{
    IPaymentProcessor CreateProcessor(PaymentType type);
}

// Factory implementation with DI
public class PaymentProcessorFactory : IPaymentProcessorFactory
{
    private readonly IServiceProvider _serviceProvider;
    
    public PaymentProcessorFactory(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }
    
    public IPaymentProcessor CreateProcessor(PaymentType type)
    {
        // Use DI container to resolve dependencies
        return type switch
        {
            PaymentType.CreditCard => _serviceProvider.GetRequiredService<CreditCardProcessor>(),
            PaymentType.PayPal => _serviceProvider.GetRequiredService<PayPalProcessor>(),
            _ => throw new ArgumentException($"Unsupported payment type: {type}")
        };
    }
}

// API Controller
[ApiController]
[Route("api/[controller]")]
public class PaymentController : ControllerBase
{
    private readonly IPaymentProcessorFactory _factory;
    
    public PaymentController(IPaymentProcessorFactory factory)
    {
        _factory = factory;
    }
    
    [HttpPost("process")]
    public async Task<ActionResult<PaymentResult>> ProcessPayment(
        [FromBody] PaymentRequest request)
    {
        try
        {
            // Factory creates appropriate processor based on request
            var processor = _factory.CreateProcessor(request.Type);
            var result = await processor.ProcessAsync(request.Amount);
            return Ok(result);
        }
        catch (ArgumentException ex)
        {
            return BadRequest($"Invalid payment type: {ex.Message}");
        }
    }
}
```

## Factory vs Abstract Factory

### Factory Pattern:
- **Purpose**: Create single product
- **Scope**: One product hierarchy
- **Complexity**: Simple to moderate

```csharp
// Factory - creates single type of object
public interface IPaymentProcessorFactory
{
    IPaymentProcessor CreateProcessor(PaymentType type);
}
```

### Abstract Factory Pattern:
- **Purpose**: Create families of related products
- **Scope**: Multiple product hierarchies
- **Complexity**: More complex

```csharp
// Abstract Factory - creates families of related objects
public interface IPaymentSuiteFactory
{
    IPaymentProcessor CreateProcessor();
    IPaymentValidator CreateValidator();
    IPaymentLogger CreateLogger();
}

// Concrete factory for Stripe suite
public class StripePaymentSuiteFactory : IPaymentSuiteFactory
{
    public IPaymentProcessor CreateProcessor() => new StripeProcessor();
    public IPaymentValidator CreateValidator() => new StripeValidator();
    public IPaymentLogger CreateLogger() => new StripeLogger();
}

// Concrete factory for PayPal suite
public class PayPalPaymentSuiteFactory : IPaymentSuiteFactory
{
    public IPaymentProcessor CreateProcessor() => new PayPalProcessor();
    public IPaymentValidator CreateValidator() => new PayPalValidator();
    public IPaymentLogger CreateLogger() => new PayPalLogger();
}
```

### When to Use Each:
- **Factory**: Different implementations of same interface
- **Abstract Factory**: Complete ecosystem of related components

## Combining Factory with DI Container

### Advanced DI Integration:
```csharp
// Factory using DI with configuration
public class ConfigurablePaymentFactory : IPaymentProcessorFactory
{
    private readonly IServiceProvider _serviceProvider;
    private readonly IConfiguration _configuration;
    private readonly ILogger<ConfigurablePaymentFactory> _logger;
    
    public ConfigurablePaymentFactory(
        IServiceProvider serviceProvider,
        IConfiguration configuration,
        ILogger<ConfigurablePaymentFactory> logger)
    {
        _serviceProvider = serviceProvider;
        _configuration = configuration;
        _logger = logger;
    }
    
    public IPaymentProcessor CreateProcessor(PaymentType type)
    {
        // Get processor configuration from appsettings.json
        var processorConfig = _configuration.GetSection($"PaymentProcessors:{type}");
        
        var processor = type switch
        {
            PaymentType.CreditCard => CreateCreditCardProcessor(processorConfig),
            PaymentType.PayPal => CreatePayPalProcessor(processorConfig),
            _ => throw new ArgumentException($"Unsupported payment type: {type}")
        };
        
        _logger.LogInformation("Created payment processor for type: {Type}", type);
        return processor;
    }
    
    private IPaymentProcessor CreateCreditCardProcessor(IConfigurationSection config)
    {
        // Create with specific configuration and dependencies from DI
        var httpClient = _serviceProvider.GetRequiredService<HttpClient>();
        var options = config.Get<CreditCardOptions>();
        return new CreditCardProcessor(httpClient, options);
    }
    
    private IPaymentProcessor CreatePayPalProcessor(IConfigurationSection config)
    {
        var httpClient = _serviceProvider.GetRequiredService<HttpClient>();
        var options = config.Get<PayPalOptions>();
        return new PayPalProcessor(httpClient, options);
    }
}

// Registration
services.AddScoped<IPaymentProcessorFactory, ConfigurablePaymentFactory>();
services.Configure<CreditCardOptions>(configuration.GetSection("PaymentProcessors:CreditCard"));
services.Configure<PayPalOptions>(configuration.GetSection("PaymentProcessors:PayPal"));
```

## Key Takeaways for Architects

### Design Decisions:
1. **Use Simple Factory**: For straightforward object creation
2. **Use Factory Method**: When subclasses should decide creation
3. **Use Abstract Factory**: For families of related objects
4. **Combine with DI**: For enterprise applications with complex dependencies

### Architecture Guidelines:
1. **Keep Factories Stateless**: Avoid maintaining state in factory classes
2. **Use Configuration**: Drive factory behavior through configuration
3. **Plan for Extension**: Design factories to be easily extensible
4. **Monitor Performance**: Consider caching for expensive object creation
5. **Handle Failures Gracefully**: Implement fallback mechanisms

### Anti-patterns to Avoid:
1. **God Factory**: Single factory creating unrelated objects
2. **Leaky Factory**: Exposing internal creation details
3. **Stateful Factory**: Maintaining state between creations
4. **Hard-coded Factory**: No configuration or extension points

The Factory Pattern is essential for building maintainable, extensible .NET applications. It promotes loose coupling, supports SOLID principles, and provides flexibility for changing business requirements while maintaining clean architecture boundaries.

--- 

3. **Builder Pattern**
   - How would you implement Builder pattern for complex API configurations?
   - When is Builder better than constructor overloading?

---
I'll provide a comprehensive breakdown of the Builder Pattern from an architect's perspective, covering all the points you've requested.This comprehensive guide covers the Builder Pattern from an architect's perspective with detailed explanations, practical examples, and real-world considerations. Here are the key highlights:

## Quick Interview-Ready Summary:

**What**: Creational pattern that constructs complex objects step-by-step using a fluent interface
**Why**: Solves telescoping constructor problem, enables validation, supports immutability
**When**: Use for objects with 4+ parameters, optional parameters, or complex validation
**How**: Separate Builder class with fluent methods returning `this`, final `Build()` method with validation

## Critical Points for Senior .NET Interviews:

1. **SOLID Adherence**: Builder pattern naturally follows SRP and OCP
2. **Thread Safety**: Use immutable builders or proper synchronization for concurrent scenarios
3. **Performance Trade-off**: Slight overhead but massive readability/maintainability gains
4. **Cloud-Native**: Essential for complex configuration objects in microservices
5. **Testing**: Makes creating test data much easier and more readable

The examples provided are production-ready and demonstrate best practices including validation, error handling, and proper separation of concerns. Each code snippet is interview-appropriate (10-15 lines) but includes detailed comments explaining the architectural decisions.

# Builder Pattern - Complete Architect's Guide

## 1. Core Category of Pattern
**Creational Design Pattern** - Part of the Gang of Four (GoF) patterns, specifically focused on object construction and instantiation.

## 2. What is this Pattern?
The Builder Pattern is a creational design pattern that provides a flexible solution to construct complex objects step by step. It separates the construction of a complex object from its representation, allowing the same construction process to create different representations.

**Key Components:**
- **Builder Interface**: Defines construction steps
- **Concrete Builder**: Implements construction steps
- **Director**: Orchestrates the construction process
- **Product**: The complex object being built

## 3. What Problem Does It Solve?
- **Telescoping Constructor Problem**: Eliminates the need for multiple constructor overloads
- **Complex Object Creation**: Handles objects with many optional parameters
- **Immutable Objects**: Enables creation of immutable objects with validation
- **Construction Logic Separation**: Separates construction logic from business logic
- **Parameter Validation**: Centralizes validation during object construction

## 4. Why Should I Use This Pattern?
- **Improved Readability**: Fluent interface makes code self-documenting
- **Flexibility**: Easy to add new construction variations without changing existing code
- **Validation**: Centralized validation before object creation
- **Immutability**: Perfect for creating immutable objects
- **Testing**: Easier to test complex object creation scenarios
- **Maintenance**: Changes to construction logic are localized

## 5. When Should I Use This Pattern?
**Use Builder when:**
- Object has 4+ constructor parameters
- Many parameters are optional
- Construction requires validation logic
- Need different representations of the same object
- Creating immutable objects
- Complex initialization sequences

**Better than:**
- **Constructor Overloading**: When you have many optional parameters
- **Factory Pattern**: When construction process is more important than object families
- **Prototype Pattern**: When object creation is more complex than cloning

## 6. Use Cases of This Pattern
- **Configuration Objects**: API clients, database connections
- **DTOs/Models**: Complex data transfer objects
- **Query Builders**: SQL, NoSQL query construction
- **Test Data Builders**: Creating test objects with various configurations
- **Document Builders**: PDF, XML, JSON document generation
- **UI Component Builders**: Complex UI controls with many properties

## 7. Implementation in .NET Framework and .NET Core

### Basic Builder Implementation
```csharp
// Product: Complex object being built
public class DatabaseConnection
{
    public string ConnectionString { get; }
    public int Timeout { get; }
    public bool EnableRetry { get; }
    public int MaxRetries { get; }
    
    // Private constructor forces use of builder
    internal DatabaseConnection(string connectionString, int timeout, bool enableRetry, int maxRetries)
    {
        ConnectionString = connectionString ?? throw new ArgumentNullException(nameof(connectionString));
        Timeout = timeout;
        EnableRetry = enableRetry;
        MaxRetries = maxRetries;
    }
}

// Builder: Provides fluent interface for construction
public class DatabaseConnectionBuilder
{
    private string _connectionString;
    private int _timeout = 30; // Default values
    private bool _enableRetry = false;
    private int _maxRetries = 3;
    
    public DatabaseConnectionBuilder WithConnectionString(string connectionString)
    {
        _connectionString = connectionString; // Store parameter for later use
        return this; // Return this for method chaining (fluent interface)
    }
    
    public DatabaseConnectionBuilder WithTimeout(int timeout)
    {
        if (timeout <= 0) throw new ArgumentException("Timeout must be positive"); // Validation during building
        _timeout = timeout;
        return this;
    }
    
    public DatabaseConnectionBuilder WithRetry(int maxRetries = 3)
    {
        _enableRetry = true; // Enable retry when this method is called
        _maxRetries = maxRetries;
        return this;
    }
    
    public DatabaseConnection Build()
    {
        // Final validation before object creation
        if (string.IsNullOrEmpty(_connectionString))
            throw new InvalidOperationException("Connection string is required");
        
        // Create immutable object with validated parameters
        return new DatabaseConnection(_connectionString, _timeout, _enableRetry, _maxRetries);
    }
}

// Usage Example
var connection = new DatabaseConnectionBuilder()
    .WithConnectionString("Server=localhost;Database=MyDB")
    .WithTimeout(60)
    .WithRetry(5)
    .Build(); // Clean, readable object construction
```

### Advanced Builder with Director
```csharp
// Director: Encapsulates construction algorithms
public class DatabaseConnectionDirector
{
    private readonly DatabaseConnectionBuilder _builder;
    
    public DatabaseConnectionDirector(DatabaseConnectionBuilder builder)
    {
        _builder = builder; // Dependency injection for flexibility
    }
    
    public DatabaseConnection CreateProductionConnection(string server, string database)
    {
        // Encapsulates production-specific construction logic
        return _builder
            .WithConnectionString($"Server={server};Database={database};Integrated Security=true")
            .WithTimeout(60) // Production timeout
            .WithRetry(5) // Production retry policy
            .Build();
    }
    
    public DatabaseConnection CreateDevelopmentConnection(string server, string database)
    {
        // Encapsulates development-specific construction logic
        return _builder
            .WithConnectionString($"Server={server};Database={database};User Id=dev;Password=dev")
            .WithTimeout(30) // Shorter timeout for dev
            .Build(); // No retry for development
    }
}
```

## 8. Architecture Relevance
- **Layered Architecture**: Builders typically live in Infrastructure/Data Access layers
- **Domain-Driven Design**: Builders can enforce domain invariants during construction
- **Microservices**: Excellent for building configuration objects for service communication
- **Clean Architecture**: Helps maintain dependency inversion by abstracting complex construction
- **CQRS**: Perfect for building complex command/query objects

## 9. SOLID Principles Connection

### Single Responsibility Principle (SRP)
```csharp
// ✅ Good: Builder has single responsibility - object construction
public class EmailBuilder
{
    // Only responsible for building Email objects
    public EmailBuilder To(string recipient) { /* ... */ return this; }
    public EmailBuilder Subject(string subject) { /* ... */ return this; }
    public Email Build() { /* validation & construction */ }
}
```

### Open/Closed Principle (OCP)
```csharp
// ✅ Good: Can extend builder without modifying existing code
public class EmailBuilder
{
    // Base functionality
    public virtual EmailBuilder WithPriority(Priority priority) { /* ... */ return this; }
}

public class SecureEmailBuilder : EmailBuilder
{
    // Extended functionality - closed for modification, open for extension
    public SecureEmailBuilder WithEncryption(EncryptionType type) { /* ... */ return this; }
}
```

### Interface Segregation Principle (ISP)
```csharp
// ✅ Good: Segregated interfaces for different building concerns
public interface IEmailContentBuilder
{
    IEmailContentBuilder Subject(string subject);
    IEmailContentBuilder Body(string body);
}

public interface IEmailDeliveryBuilder
{
    IEmailDeliveryBuilder To(string recipient);
    IEmailDeliveryBuilder Priority(EmailPriority priority);
}
```

### Dependency Inversion Principle (DIP)
```csharp
// ✅ Good: Depend on abstractions, not concretions
public interface IConfigurationBuilder<T>
{
    T Build();
}

public class ServiceConfiguration
{
    public ServiceConfiguration(IConfigurationBuilder<ServiceConfig> builder)
    {
        _config = builder.Build(); // Depends on abstraction, not concrete builder
    }
}
```

## 10. UML Diagram

```mermaid
classDiagram
    class Director {
        -builder: IBuilder
        +construct(): Product
    }
    
    class IBuilder {
        <<interface>>
        +buildPartA()
        +buildPartB()
        +getResult(): Product
    }
    
    class ConcreteBuilder {
        -product: Product
        +buildPartA()
        +buildPartB()
        +getResult(): Product
    }
    
    class Product {
        -partA: string
        -partB: string
        +operation()
    }
    
    Director --> IBuilder
    IBuilder <|-- ConcreteBuilder
    ConcreteBuilder --> Product
    
    note for Director "Orchestrates construction\nusing builder interface"
    note for ConcreteBuilder "Implements construction\nsteps and maintains\nproduct state"
```

## 11. Concurrency and Cloud Context

### Thread-Safe Builder Implementation
```csharp
public class ThreadSafeBuilder
{
    private readonly object _lock = new object(); // Thread synchronization
    private readonly Dictionary<string, object> _properties = new();
    
    public ThreadSafeBuilder WithProperty(string key, object value)
    {
        lock (_lock) // Ensure thread safety during property setting
        {
            _properties[key] = value;
            return this;
        }
    }
    
    public T Build<T>()
    {
        lock (_lock) // Ensure thread safety during build
        {
            // Create immutable copy of properties
            var props = new Dictionary<string, object>(_properties);
            return CreateInstance<T>(props); // Build with copied state
        }
    }
}
```

### Cloud-Native Configuration Builder
```csharp
public class CloudConfigBuilder
{
    public CloudConfigBuilder FromEnvironmentVariables() { /* ... */ return this; }
    public CloudConfigBuilder FromKeyVault(string vaultUrl) { /* ... */ return this; }
    public CloudConfigBuilder FromConfigMap(string configMapName) { /* ... */ return this; }
    
    public async Task<CloudConfig> BuildAsync()
    {
        // Async construction for cloud resources
        var config = new CloudConfig();
        await LoadCloudResourcesAsync(config); // Non-blocking resource loading
        return config;
    }
}
```

## 12. Trade-offs & Misuse

### ✅ Advantages
- **Flexibility**: Easy to modify construction without affecting existing code
- **Readability**: Self-documenting fluent interface
- **Validation**: Centralized validation logic
- **Immutability**: Perfect for creating immutable objects
- **Testability**: Easy to create test objects with specific configurations

### ❌ Disadvantages
- **Complexity**: Adds extra classes and complexity for simple objects
- **Memory Overhead**: Additional objects during construction phase
- **Performance**: Slight performance overhead due to method chaining
- **Learning Curve**: Developers need to understand the pattern

### Common Misuse Patterns
```csharp
// ❌ Bad: Using builder for simple objects
public class PersonBuilder // Overkill for simple 2-property object
{
    public PersonBuilder WithName(string name) { /* ... */ return this; }
    public PersonBuilder WithAge(int age) { /* ... */ return this; }
    public Person Build() { /* ... */ }
}

// ✅ Good: Simple constructor is sufficient
public class Person
{
    public Person(string name, int age) // Simple construction
    {
        Name = name;
        Age = age;
    }
}
```

## 13. Risks in Multi-threaded/Cloud-Native World

### Thread Safety Risks
- **Shared State**: Builders with mutable state can cause race conditions
- **Validation Issues**: Concurrent access during validation can lead to inconsistent state
- **Resource Leaks**: Improper cleanup in cloud builders can cause resource leaks

### Mitigation Strategies
```csharp
// ✅ Good: Immutable builder approach
public class ImmutableBuilder
{
    private readonly ImmutableDictionary<string, object> _properties;
    
    private ImmutableBuilder(ImmutableDictionary<string, object> properties)
    {
        _properties = properties; // Immutable state - thread safe by design
    }
    
    public static ImmutableBuilder Create() => 
        new(ImmutableDictionary<string, object>.Empty);
    
    public ImmutableBuilder With(string key, object value) =>
        new(_properties.SetItem(key, value)); // Return new instance, don't modify existing
}
```

## 14. Drawbacks of Not Using Builder Pattern

### Constructor Hell
```csharp
// ❌ Bad: Telescoping constructor anti-pattern
public class ApiClient
{
    public ApiClient(string baseUrl) { /* ... */ }
    public ApiClient(string baseUrl, int timeout) { /* ... */ }
    public ApiClient(string baseUrl, int timeout, string apiKey) { /* ... */ }
    public ApiClient(string baseUrl, int timeout, string apiKey, bool enableRetry) { /* ... */ }
    public ApiClient(string baseUrl, int timeout, string apiKey, bool enableRetry, int maxRetries) { /* ... */ }
    // ... continues indefinitely
}
```

### Poor Maintainability
- **Parameter Confusion**: Easy to pass parameters in wrong order
- **Breaking Changes**: Adding parameters breaks existing code
- **Testing Complexity**: Hard to create test objects with specific configurations
- **Validation Scatter**: Validation logic scattered across multiple constructors

## 15. Complex API Configuration Example

```csharp
public class ApiClientBuilder
{
    private string _baseUrl;
    private readonly Dictionary<string, string> _headers = new();
    private readonly List<IInterceptor> _interceptors = new();
    private TimeSpan _timeout = TimeSpan.FromSeconds(30);
    private RetryPolicy _retryPolicy;
    
    public ApiClientBuilder WithBaseUrl(string baseUrl)
    {
        if (!Uri.IsWellFormedUriString(baseUrl, UriKind.Absolute))
            throw new ArgumentException("Invalid base URL format");
        _baseUrl = baseUrl;
        return this;
    }
    
    public ApiClientBuilder WithAuthentication(string scheme, string parameter)
    {
        _headers["Authorization"] = $"{scheme} {parameter}"; // Add auth header
        return this;
    }
    
    public ApiClientBuilder WithRetry(int maxAttempts, TimeSpan delay)
    {
        _retryPolicy = new RetryPolicy(maxAttempts, delay); // Configure retry behavior
        return this;
    }
    
    public ApiClientBuilder WithInterceptor(IInterceptor interceptor)
    {
        _interceptors.Add(interceptor); // Add request/response interceptor
        return this;
    }
    
    public async Task<ApiClient> BuildAsync()
    {
        if (string.IsNullOrEmpty(_baseUrl))
            throw new InvalidOperationException("Base URL is required");
            
        // Create HTTP client with all configurations
        var httpClient = new HttpClient { BaseAddress = new Uri(_baseUrl), Timeout = _timeout };
        
        // Apply headers
        foreach (var header in _headers)
            httpClient.DefaultRequestHeaders.Add(header.Key, header.Value);
            
        // Create configured API client
        return new ApiClient(httpClient, _retryPolicy, _interceptors);
    }
}
```

## 16. Builder vs Constructor Overloading

### When Builder is Better

**Complex Configuration (4+ parameters):**
```csharp
// ❌ Bad: Too many constructor parameters
public EmailService(string smtpHost, int port, bool enableSsl, string username, 
                   string password, int timeout, bool enableRetry, int maxRetries) { }

// ✅ Good: Builder pattern
var emailService = new EmailServiceBuilder()
    .WithSmtpServer("smtp.gmail.com", 587)
    .WithCredentials("user@gmail.com", "password")
    .WithSsl(true)
    .WithTimeout(TimeSpan.FromMinutes(2))
    .WithRetryPolicy(3, TimeSpan.FromSeconds(5))
    .Build();
```

**Optional Parameters with Validation:**
```csharp
// ❌ Bad: Constructor overloading becomes unwieldy
public class DatabaseQuery
{
    public DatabaseQuery(string sql) { }
    public DatabaseQuery(string sql, int timeout) { }
    public DatabaseQuery(string sql, int timeout, bool enableCache) { }
    // Validation scattered across constructors
}

// ✅ Good: Builder with centralized validation
var query = new DatabaseQueryBuilder()
    .WithSql("SELECT * FROM Users WHERE Age > @age")
    .WithParameter("@age", 18)
    .WithTimeout(30)
    .WithCaching(TimeSpan.FromMinutes(5))
    .Build(); // All validation happens here
```

## 17. Key Takeaways for Architects

### Design Decisions
1. **Use Sparingly**: Don't over-apply - simple objects don't need builders
2. **Immutability**: Combine with immutable objects for thread safety
3. **Validation Strategy**: Centralize validation in Build() method
4. **Interface Design**: Design fluent interfaces that guide correct usage
5. **Performance**: Consider object creation overhead vs. readability benefits

### Architecture Integration
- **Dependency Injection**: Register builders as transient services
- **Configuration**: Perfect for complex configuration objects
- **Testing**: Create builder extensions for test-specific configurations
- **Documentation**: Builders serve as living documentation of object requirements

### Best Practices
- **Fail Fast**: Validate early and provide clear error messages
- **Method Chaining**: Ensure all builder methods return the builder instance
- **Immutable Results**: Build() should return immutable objects when possible
- **Resource Management**: Implement IDisposable when builders manage resources
- **Async Support**: Provide async Build methods for I/O intensive construction

### Anti-Patterns to Avoid
- **God Builder**: Don't create builders that build too many different types
- **Leaky Abstraction**: Don't expose internal builder state
- **Incomplete State**: Always validate required properties before building
- **Mutable Products**: Avoid returning mutable objects from builders in multi-threaded scenarios
---

4. **Singleton Pattern**
   - How does .NET Core DI implement Singleton? What are the thread-safety concerns?
   - When should you avoid Singleton in Web API applications?

---
I'll provide a comprehensive breakdown of the Singleton Pattern from an architect's perspective, covering all the points you've requested with special focus on .NET Core DI and modern architectural concerns.This comprehensive guide covers the Singleton Pattern from an architect's perspective with detailed focus on modern .NET development concerns. Here are the key highlights for your technical interviews:

## Critical Interview Points:

### **Traditional vs. DI Singleton**
- **Traditional**: Manual instance control, harder to test, tight coupling
- **DI Singleton**: Container-managed, testable, follows SOLID principles
- **Recommendation**: Prefer DI singletons in modern .NET applications

### **Thread Safety Essentials**
- Use `Lazy<T>` for thread-safe initialization
- Use `ConcurrentDictionary` and other thread-safe collections
- Avoid double-check locking (use `Lazy<T>` instead)
- Be aware of memory visibility issues with `volatile`

### **Web API Specific Concerns**
- **Avoid**: Stateful singletons that maintain user-specific data
- **Risk**: Race conditions in multi-threaded web environments
- **Performance**: Singletons can become bottlenecks
- **Testing**: Traditional singletons are hard to mock

### **.NET Core DI Implementation**
- Container uses `ConcurrentDictionary` for thread-safe instance storage
- Automatic disposal handling for `IDisposable` singletons
- Factory-based registration for complex initialization
- Service lifetime scope: Application lifetime vs. request scope

### **Cloud-Native Considerations**
- Each container gets its own singleton instance
- Use distributed caching for truly global state
- Monitor resource usage across container instances
- Plan for horizontal scaling implications

The code examples provided are production-ready and demonstrate best practices including proper thread safety, error handling, and modern .NET patterns. Each example includes detailed architectural reasoning that will help you articulate design decisions during interviews.

# Singleton Pattern - Complete Architect's Guide

## 1. Core Category of Pattern
**Creational Design Pattern** - Part of the Gang of Four (GoF) patterns, specifically focused on controlling object instantiation and ensuring single instance existence.

## 2. What is this Pattern?
The Singleton Pattern ensures that a class has only one instance and provides a global point of access to that instance. It's responsible for creating its own unique instance and ensuring no other instance can be created.

**Key Characteristics:**
- **Single Instance**: Only one object instance exists throughout application lifetime
- **Global Access**: Provides a globally accessible point to the instance
- **Self-Instantiation**: The class controls its own instantiation process
- **Lazy or Eager Loading**: Instance can be created on-demand or at startup

## 3. What Problem Does It Solve?
- **Resource Management**: Controls access to shared resources (database connections, file systems)
- **Global State**: Provides controlled global state management
- **Configuration Management**: Ensures single source of truth for application configuration
- **Service Coordination**: Coordinates actions across the system
- **Memory Optimization**: Prevents multiple instances of expensive objects
- **Thread Synchronization**: Provides centralized coordination point

## 4. Why Should I Use This Pattern?
- **Resource Conservation**: Saves memory and computational resources
- **Global Access**: Provides easy access to shared functionality
- **Consistency**: Ensures consistent state across the application
- **Coordination**: Enables system-wide coordination and communication
- **Initialization Control**: Controls when and how expensive initialization occurs
- **State Management**: Maintains application-wide state consistently

## 5. When Should I Use This Pattern?
**Use Singleton when:**
- Only one instance should exist (database connection pool, logger, cache)
- Global access point is needed
- Expensive initialization should happen only once
- Coordinating actions across the system
- Managing shared resources

**Better than:**
- **Static Classes**: When you need inheritance, interfaces, or dependency injection
- **Global Variables**: Provides controlled access and initialization
- **Factory Pattern**: When you need exactly one instance, not multiple variants

**Avoid when:**
- Multiple instances might be needed in the future
- Testing requires different instances
- The class has dependencies that change
- In stateless web applications (usually)

## 6. Use Cases of This Pattern
- **Logging Services**: Application-wide logging coordination
- **Configuration Managers**: Application settings and configuration
- **Database Connection Pools**: Managing database connection resources
- **Cache Managers**: In-memory caching systems
- **Thread Pool Managers**: Managing worker thread pools
- **Service Locators**: Dependency resolution (though DI is preferred)
- **Hardware Interface**: Printer spoolers, device drivers
- **License Managers**: Software licensing validation

## 7. Implementation in .NET Framework and .NET Core

### Thread-Safe Lazy Singleton (Recommended)
```csharp
public sealed class ConfigurationManager
{
    // Lazy<T> provides thread-safe, lazy initialization
    private static readonly Lazy<ConfigurationManager> _instance = 
        new Lazy<ConfigurationManager>(() => new ConfigurationManager());
    
    // Dictionary to store configuration values
    private readonly ConcurrentDictionary<string, string> _settings;
    
    // Private constructor prevents external instantiation
    private ConfigurationManager()
    {
        _settings = new ConcurrentDictionary<string, string>();
        LoadConfiguration(); // Initialize configuration on first access
    }
    
    // Public static property provides global access point
    public static ConfigurationManager Instance => _instance.Value;
    
    public string GetSetting(string key) => _settings.GetValueOrDefault(key);
    
    public void SetSetting(string key, string value) => _settings[key] = value;
    
    private void LoadConfiguration()
    {
        // Load configuration from file, database, etc.
        // This expensive operation happens only once
    }
}
```

### Double-Check Locking Pattern (Legacy Approach)
```csharp
public sealed class LegacySingleton
{
    private static volatile LegacySingleton _instance; // volatile ensures proper memory ordering
    private static readonly object _lock = new object(); // Lock object for synchronization
    
    private LegacySingleton() { } // Private constructor
    
    public static LegacySingleton Instance
    {
        get
        {
            if (_instance == null) // First check without locking (performance optimization)
            {
                lock (_lock) // Enter critical section
                {
                    if (_instance == null) // Second check after acquiring lock
                        _instance = new LegacySingleton(); // Create instance only once
                }
            }
            return _instance;
        }
    }
}
```

### Generic Singleton Base Class
```csharp
public abstract class Singleton<T> where T : class, new()
{
    // Generic lazy singleton base class for reusability
    private static readonly Lazy<T> _instance = new Lazy<T>(() => new T());
    
    protected Singleton() // Protected constructor allows inheritance
    {
        if (_instance.IsValueCreated)
            throw new InvalidOperationException($"Instance of {typeof(T).Name} already exists");
    }
    
    public static T Instance => _instance.Value; // Generic instance access
}

// Usage: Inherit from generic singleton
public class DatabaseManager : Singleton<DatabaseManager>
{
    public void ExecuteQuery(string sql) { /* Implementation */ }
}
```

## 8. Architecture Relevance
- **Microservices**: Each service may have singleton managers for coordination
- **Layered Architecture**: Often used in Infrastructure and Application Service layers
- **Domain-Driven Design**: Domain services that coordinate across aggregates
- **Event-Driven Architecture**: Event buses and message coordinators
- **Clean Architecture**: Infrastructure concerns like logging, caching
- **CQRS**: Command/Query dispatchers and handlers registration

## 9. SOLID Principles Connection

### Single Responsibility Principle (SRP)
```csharp
// ✅ Good: Logger has single responsibility
public sealed class Logger : Singleton<Logger>
{
    private readonly ILogWriter _writer;
    
    private Logger() 
    {
        _writer = new FileLogWriter(); // Single responsibility: logging
    }
    
    public void Log(string message, LogLevel level) 
    {
        _writer.Write($"[{level}] {DateTime.Now}: {message}");
    }
}
```

### Open/Closed Principle (OCP)
```csharp
// ✅ Good: Extensible through strategy pattern
public sealed class NotificationManager : Singleton<NotificationManager>
{
    private readonly List<INotificationStrategy> _strategies = new();
    
    public void AddStrategy(INotificationStrategy strategy) // Open for extension
    {
        _strategies.Add(strategy);
    }
    
    public void Notify(string message) // Closed for modification
    {
        foreach (var strategy in _strategies)
            strategy.Send(message);
    }
}
```

### Dependency Inversion Principle (DIP)
```csharp
// ❌ Bad: Singleton violates DIP by creating hard dependencies
public sealed class BadOrderService
{
    public void ProcessOrder(Order order)
    {
        Logger.Instance.Log("Processing order"); // Hard dependency on concrete class
        EmailService.Instance.SendConfirmation(order.Email); // Tight coupling
    }
}

// ✅ Good: Use DI instead of singleton for better testability
public class GoodOrderService
{
    private readonly ILogger _logger;
    private readonly IEmailService _emailService;
    
    public GoodOrderService(ILogger logger, IEmailService emailService) // Depend on abstractions
    {
        _logger = logger;
        _emailService = emailService;
    }
    
    public void ProcessOrder(Order order)
    {
        _logger.Log("Processing order"); // Loosely coupled through interface
        _emailService.SendConfirmation(order.Email);
    }
}
```

## 10. UML Diagram

```mermaid
classDiagram
    class Singleton {
        -instance: Singleton$
        -Singleton()
        +getInstance()$ Singleton
        +businessMethod()
    }
    
    class Client {
        +operation()
    }
    
    Client --> Singleton : uses
    Singleton --> Singleton : creates
    
    note for Singleton "- Private constructor prevents\n  external instantiation\n- Static instance variable\n- Static getInstance() method\n- Lazy or eager initialization"
    
    note for Client "Cannot instantiate Singleton\ndirectly, must use getInstance()"
```

### Singleton with Dependency Injection
```mermaid
classDiagram
    class IService {
        <<interface>>
        +performOperation()
    }
    
    class SingletonService {
        -instance: SingletonService$
        -SingletonService()
        +getInstance()$ SingletonService
        +performOperation()
    }
    
    class DIContainer {
        +registerSingleton(type, instance)
        +resolve(type) T
    }
    
    class Client {
        -service: IService
        +Client(service: IService)
        +doWork()
    }
    
    IService <|-- SingletonService
    DIContainer --> SingletonService : manages lifecycle
    DIContainer --> Client : injects dependency
    Client --> IService : uses
```

## 11. Concurrency and Cloud Context

### Thread-Safe Implementation Considerations
```csharp
public sealed class ThreadSafeCache
{
    private static readonly Lazy<ThreadSafeCache> _instance = 
        new Lazy<ThreadSafeCache>(() => new ThreadSafeCache());
    
    // Use concurrent collections for thread safety
    private readonly ConcurrentDictionary<string, object> _cache = new();
    private readonly ReaderWriterLockSlim _lock = new(); // For complex operations
    
    private ThreadSafeCache() { }
    
    public static ThreadSafeCache Instance => _instance.Value;
    
    public T Get<T>(string key)
    {
        _lock.EnterReadLock(); // Multiple readers allowed
        try
        {
            return _cache.TryGetValue(key, out var value) ? (T)value : default(T);
        }
        finally
        {
            _lock.ExitReadLock();
        }
    }
    
    public void Set<T>(string key, T value)
    {
        _lock.EnterWriteLock(); // Exclusive write access
        try
        {
            _cache[key] = value; // Thread-safe update
        }
        finally
        {
            _lock.ExitWriteLock();
        }
    }
}
```

### Cloud-Native Considerations
```csharp
public sealed class CloudConfigurationManager
{
    private static readonly Lazy<CloudConfigurationManager> _instance = 
        new Lazy<CloudConfigurationManager>(() => new CloudConfigurationManager());
    
    private readonly IMemoryCache _cache;
    private readonly string _instanceId; // Track container instance
    
    private CloudConfigurationManager()
    {
        _cache = new MemoryCache(new MemoryCacheOptions());
        _instanceId = Environment.GetEnvironmentVariable("HOSTNAME") ?? Guid.NewGuid().ToString();
        // In cloud environments, each container gets its own singleton instance
    }
    
    public static CloudConfigurationManager Instance => _instance.Value;
    public string InstanceId => _instanceId; // Useful for distributed debugging
    
    public async Task<string> GetConfigurationAsync(string key)
    {
        if (_cache.TryGetValue(key, out string cachedValue))
            return cachedValue;
            
        // Fetch from cloud configuration service
        var value = await FetchFromCloudAsync(key);
        _cache.Set(key, value, TimeSpan.FromMinutes(5)); // Cache with TTL
        return value;
    }
    
    private async Task<string> FetchFromCloudAsync(string key) 
    { 
        // Implementation for cloud config service
        await Task.Delay(100); // Simulate network call
        return $"CloudValue-{key}";
    }
}
```

## 12. Trade-offs & Misuse

### ✅ Advantages
- **Memory Efficiency**: Single instance reduces memory footprint
- **Global Access**: Easy access throughout the application
- **Initialization Control**: Expensive initialization happens only once
- **Resource Management**: Controls access to limited resources
- **State Consistency**: Maintains consistent global state

### ❌ Disadvantages
- **Global State**: Creates hidden dependencies and makes testing difficult
- **Tight Coupling**: Classes become tightly coupled to singleton instances
- **Scalability Issues**: Can become bottleneck in multi-threaded applications
- **Testing Complexity**: Difficult to mock and test in isolation
- **Violation of SRP**: Often accumulates too many responsibilities

### Common Misuse Patterns
```csharp
// ❌ Bad: God Object Singleton
public sealed class ApplicationManager // Anti-pattern: too many responsibilities
{
    public void ManageDatabase() { }
    public void HandleLogging() { }
    public void ProcessPayments() { }
    public void SendEmails() { }
    public void ValidateInput() { }
    // ... 50 more methods
}

// ❌ Bad: Singleton for Stateless Operations
public sealed class MathUtils // Should be static class instead
{
    private static readonly Lazy<MathUtils> _instance = new(() => new MathUtils());
    public static MathUtils Instance => _instance.Value;
    
    public int Add(int a, int b) => a + b; // Stateless operation doesn't need singleton
    public int Multiply(int a, int b) => a * b;
}

// ✅ Good: Static class for stateless utilities
public static class MathUtils
{
    public static int Add(int a, int b) => a + b; // Simple static method
    public static int Multiply(int a, int b) => a * b;
}
```

## 13. Risks in Multi-threaded/Cloud-Native World

### Multi-threading Risks
- **Race Conditions**: Multiple threads creating instances simultaneously
- **Memory Visibility**: Changes not visible across threads without proper synchronization
- **Deadlocks**: Improper locking can cause deadlocks
- **Performance Bottlenecks**: Synchronized access can become performance bottleneck

### Cloud-Native Risks
```csharp
// ❌ Risk: Singleton assumptions in containerized environments
public sealed class ProblematicSingleton
{
    private static int _requestCount = 0; // This counter is per-container instance
    
    public void IncrementRequests()
    {
        Interlocked.Increment(ref _requestCount); // Only counts requests in this container
    }
    
    public int GetTotalRequests() => _requestCount; // Misleading in multi-container deployment
}

// ✅ Better: Use distributed state management
public sealed class CloudAwareSingleton
{
    private readonly IDistributedCache _distributedCache;
    
    private CloudAwareSingleton()
    {
        _distributedCache = new RedisCache(/* connection */);
    }
    
    public async Task<int> GetGlobalRequestCountAsync()
    {
        var countStr = await _distributedCache.GetStringAsync("global_request_count");
        return int.TryParse(countStr, out var count) ? count : 0;
    }
}
```

## 14. Drawbacks of Not Using Singleton

### Resource Waste
```csharp
// ❌ Bad: Multiple expensive instances
public class ExpensiveDatabaseConnectionManager
{
    private readonly List<IDbConnection> _connections = new();
    
    public ExpensiveDatabaseConnectionManager()
    {
        // Expensive initialization happens multiple times
        for (int i = 0; i < 10; i++)
        {
            _connections.Add(CreateConnection()); // Resource waste
        }
    }
}
```

### Inconsistent State
```csharp
// ❌ Bad: Multiple configuration instances with different states
public class ConfigurationService
{
    private readonly Dictionary<string, string> _settings = new();
    
    public void LoadFromFile(string fileName) 
    {
        // Different instances might load different configuration files
        // Leading to inconsistent application behavior
    }
}
```

## 15. .NET Core DI Implementation of Singleton

### How .NET Core DI Implements Singleton
```csharp
// ServiceCollection registration
public void ConfigureServices(IServiceCollection services)
{
    // AddSingleton creates a single instance for the entire application lifetime
    services.AddSingleton<IMyService, MyService>();
    
    // Alternative: Register existing instance
    services.AddSingleton<IConfigService>(new ConfigService());
    
    // Factory-based registration with singleton lifetime
    services.AddSingleton<IExpensiveService>(serviceProvider =>
    {
        var config = serviceProvider.GetRequiredService<IConfiguration>();
        return new ExpensiveService(config); // Instance created once
    });
}

// Internal DI implementation concept (simplified)
public class ServiceContainer
{
    private readonly ConcurrentDictionary<Type, object> _singletonInstances = new();
    private readonly Dictionary<Type, ServiceDescriptor> _services = new();
    
    public T GetService<T>()
    {
        var serviceType = typeof(T);
        
        if (_services[serviceType].Lifetime == ServiceLifetime.Singleton)
        {
            // GetOrAdd ensures thread-safe single instance creation
            return (T)_singletonInstances.GetOrAdd(serviceType, 
                _ => CreateInstance(serviceType)); // Factory creates instance once
        }
        
        return CreateInstance<T>(); // Create new instance for other lifetimes
    }
}
```

### Thread-Safety Concerns in .NET Core DI
```csharp
// ✅ Good: Thread-safe singleton service
[Service(ServiceLifetime.Singleton)]
public class ThreadSafeService : IMyService
{
    private readonly ConcurrentDictionary<string, int> _counters = new();
    private readonly ILogger<ThreadSafeService> _logger;
    
    public ThreadSafeService(ILogger<ThreadSafeService> logger) // DI handles thread-safe construction
    {
        _logger = logger; // Logger itself is thread-safe
    }
    
    public void IncrementCounter(string key)
    {
        var newValue = _counters.AddOrUpdate(key, 1, (_, oldValue) => oldValue + 1);
        _logger.LogInformation("Counter {Key} incremented to {Value}", key, newValue);
    }
}

// ❌ Risk: Non-thread-safe singleton service
[Service(ServiceLifetime.Singleton)]
public class ProblematicSingletonService : IMyService
{
    private int _counter = 0; // Race condition risk
    private readonly Dictionary<string, object> _cache = new(); // Not thread-safe
    
    public void IncrementCounter()
    {
        _counter++; // Race condition: multiple threads can increment simultaneously
    }
    
    public void CacheValue(string key, object value)
    {
        _cache[key] = value; // Race condition: Dictionary is not thread-safe
    }
}
```

### Singleton Disposal in DI
```csharp
public class DisposableSingletonService : IMyService, IDisposable
{
    private readonly IDbConnection _connection;
    private bool _disposed = false;
    
    public DisposableSingletonService(IDbConnection connection)
    {
        _connection = connection; // DI injects dependencies
    }
    
    public void Dispose()
    {
        if (!_disposed)
        {
            _connection?.Dispose(); // DI container calls Dispose on app shutdown
            _disposed = true;
        }
    }
    
    // DI container automatically handles disposal of singleton services
    // when the application shuts down
}
```

## 16. When to Avoid Singleton in Web API Applications

### Stateful Singletons in Web APIs
```csharp
// ❌ Avoid: Singleton with user-specific state in Web API
[Service(ServiceLifetime.Singleton)]
public class ProblematicUserService
{
    private string _currentUserId; // Shared across all requests - major problem!
    
    public void SetCurrentUser(string userId)
    {
        _currentUserId = userId; // Race condition: user data mixed between requests
    }
    
    public string GetCurrentUser() => _currentUserId; // Returns wrong user data
}

// ✅ Better: Use scoped lifetime for request-specific data
[Service(ServiceLifetime.Scoped)]
public class UserService
{
    private string _currentUserId; // Scoped per request - safe
    
    public void SetCurrentUser(string userId) => _currentUserId = userId;
    public string GetCurrentUser() => _currentUserId;
}
```

### Performance Bottlenecks
```csharp
// ❌ Avoid: Singleton that becomes bottleneck
[Service(ServiceLifetime.Singleton)]
public class CentralizedProcessor
{
    private readonly object _lock = new object();
    
    public async Task<string> ProcessRequestAsync(string input)
    {
        lock (_lock) // All requests serialize through this lock
        {
            // Expensive processing that blocks all other requests
            Thread.Sleep(1000); // Simulates blocking operation
            return $"Processed: {input}";
        }
    }
}

// ✅ Better: Use scoped or transient for parallel processing
[Service(ServiceLifetime.Transient)]
public class ParallelProcessor
{
    public async Task<string> ProcessRequestAsync(string input)
    {
        // Each request gets its own instance - parallel processing
        await Task.Delay(1000); // Non-blocking async operation
        return $"Processed: {input}";
    }
}
```

### Testing and Mocking Issues
```csharp
// ❌ Difficult to test: Singleton with external dependencies
public sealed class PaymentProcessor
{
    private static readonly Lazy<PaymentProcessor> _instance = 
        new Lazy<PaymentProcessor>(() => new PaymentProcessor());
    
    public static PaymentProcessor Instance => _instance.Value;
    
    private PaymentProcessor() { }
    
    public bool ProcessPayment(decimal amount)
    {
        // Direct call to external service - hard to mock in tests
        return ExternalPaymentGateway.Charge(amount);
    }
}

// ✅ Testable: DI with interfaces
public class PaymentProcessor : IPaymentProcessor
{
    private readonly IPaymentGateway _gateway;
    
    public PaymentProcessor(IPaymentGateway gateway) // Dependency injection
    {
        _gateway = gateway;
    }
    
    public bool ProcessPayment(decimal amount)
    {
        return _gateway.Charge(amount); // Easy to mock in tests
    }
}
```

## 17. Best Practices and Recommendations

### When to Use Traditional Singleton
```csharp
// ✅ Good use case: Application-wide configuration
public sealed class AppConfiguration
{
    private static readonly Lazy<AppConfiguration> _instance = 
        new Lazy<AppConfiguration>(() => new AppConfiguration());
    
    private readonly IConfigurationRoot _config;
    
    private AppConfiguration()
    {
        _config = new ConfigurationBuilder()
            .AddJsonFile("appsettings.json")
            .AddEnvironmentVariables()
            .Build(); // Expensive one-time initialization
    }
    
    public static AppConfiguration Instance => _instance.Value;
    public string GetConnectionString(string name) => _config.GetConnectionString(name);
}
```

### When to Use DI Singleton
```csharp
// ✅ Preferred: DI singleton for better testability
public interface ICacheService
{
    T Get<T>(string key);
    void Set<T>(string key, T value, TimeSpan expiration);
}

public class MemoryCacheService : ICacheService
{
    private readonly IMemoryCache _cache;
    
    public MemoryCacheService(IMemoryCache cache) // DI manages singleton lifetime
    {
        _cache = cache;
    }
    
    public T Get<T>(string key) => _cache.Get<T>(key);
    public void Set<T>(string key, T value, TimeSpan expiration) => 
        _cache.Set(key, value, expiration);
}

// Registration
services.AddSingleton<ICacheService, MemoryCacheService>(); // DI handles lifetime
```

## 18. Key Takeaways for Architects

### Design Decisions
1. **Prefer DI Singletons**: Use dependency injection over traditional singleton pattern
2. **Thread Safety**: Always ensure thread safety in singleton implementations
3. **Stateless Design**: Keep singletons stateless or use thread-safe state management
4. **Interface Segregation**: Use interfaces to reduce coupling
5. **Testability**: Design for testability with dependency injection

### Architecture Integration
- **Microservices**: Each service has its own singleton instances
- **Cloud-Native**: Be aware of container-per-instance implications
- **Performance**: Monitor for bottlenecks in high-concurrency scenarios
- **Monitoring**: Add metrics and logging to singleton services
- **Graceful Shutdown**: Implement proper disposal for resource cleanup

### Modern Alternatives
- **Dependency Injection**: Preferred over traditional singleton
- **Static Classes**: For stateless utility functions
- **Scoped Services**: For request-specific state in web applications
- **Distributed Caching**: For shared state across multiple instances
- **Configuration Services**: For application settings management
---

### **Structural Patterns**
5. **Repository Pattern**
   - How do you implement Repository pattern with Entity Framework Core?
   - What problems does Repository solve over direct DbContext usage?
   - Show Generic Repository vs Specific Repository implementation.
   - How do you handle Unit of Work with Repository?

---
I'll provide you with a comprehensive, solution-oriented architect's guide to the Repository Pattern for your .NET technical interview preparation.I've created a comprehensive interview guide covering all aspects of the Repository Pattern from a solution-oriented architect's perspective. This guide includes:

**Key highlights for your interview preparation:**

1. **Pattern Classification**: Architectural/Data Access Layer pattern with DDD alignment
2. **Problem-Solution Mapping**: Clear understanding of what problems it solves vs. when to avoid it
3. **SOLID Principles**: Detailed connection showing how Repository supports each principle
4. **Practical Examples**: Short but interview-ready code examples with detailed explanations
5. **EF Core Integration**: Specific implementation details for Entity Framework Core
6. **Trade-offs Analysis**: Honest assessment of when to use vs. avoid this pattern
7. **Enterprise Context**: Cloud-native and concurrency considerations for modern applications

**Interview-ready talking points:**

- **"Repository acts as an in-memory collection of domain objects"** - This is the key conceptual understanding
- **Generic vs Specific Repository trade-offs** - Know when to use each approach
- **Unit of Work coordination** - Essential for maintaining transactional consistency
- **SOLID principles alignment** - Demonstrate architectural thinking
- **Testing benefits** - Major reason for adoption in enterprise applications

The guide is structured to help you explain not just *how* to implement the pattern, but *why* it exists and *when* it provides value - which is exactly what senior technical interviews focus on.

# Repository Pattern - Complete Interview Guide for .NET Architects

## 1. Core Category of Pattern
**Architectural Pattern** (Data Access Layer Pattern)
- **Type**: Structural/Creational hybrid pattern
- **Layer**: Data Access Layer abstraction
- **Domain**: Domain-Driven Design (DDD) tactical pattern

## 2. What is this Pattern?

The Repository Pattern is a design pattern that **encapsulates the logic needed to access data sources**. It centralizes common data access functionality, providing better maintainability and decoupling the infrastructure or technology used to access databases from the domain model layer.

**Key Characteristics:**
- Acts as an **in-memory collection** of domain objects
- Provides a **uniform interface** for accessing domain objects
- **Abstracts data access technology** from business logic
- Implements **collection-like interface** for domain aggregates

## 3. What Problem Does It Solve?

### Primary Problems:
1. **Tight Coupling**: Direct database access couples business logic to data access technology
2. **Testability**: Hard to unit test business logic with direct database dependencies
3. **Technology Lock-in**: Difficult to switch between different data access technologies
4. **Code Duplication**: Repetitive CRUD operations across different parts of application
5. **Complex Queries**: Business logic mixed with complex database queries
6. **Domain Model Pollution**: Infrastructure concerns leak into domain layer

### Secondary Problems:
- **Inconsistent Data Access**: Different parts using different approaches
- **Transaction Management**: Difficult to coordinate multiple operations
- **Caching Strategy**: No centralized place for data access optimizations

## 4. Why Should I Use This Pattern?

### Business Benefits:
- **Separation of Concerns**: Clean separation between business and data access logic
- **Maintainability**: Centralized data access logic easier to maintain
- **Flexibility**: Easy to switch data sources without affecting business logic
- **Team Productivity**: Multiple developers can work on business logic and data access independently

### Technical Benefits:
- **Unit Testing**: Mock repositories for isolated business logic testing
- **Caching**: Centralized place for implementing caching strategies
- **Query Optimization**: Single place to optimize data access patterns
- **Domain Alignment**: Repository per aggregate root aligns with DDD principles

## 5. When Should I Use This Pattern?

### **Better Fit Than:**

| **Repository Pattern** | **Direct DbContext**   | **When to Choose Repository** |
|------------------------|------------------------|-------------------------------|
| Complex domain logic   | Simple CRUD operations | Domain-driven applications    |
| Multiple data sources  | Single database        | Multi-tenant scenarios        |
| High testability needs | Rapid prototyping      | Enterprise applications       |
| Team collaboration     | Small projects         | Long-term maintainability     |

### **Ideal Scenarios:**
- **Domain-Driven Design** implementations
- **Multi-layered architectures** (Clean Architecture, Onion Architecture)
- **Enterprise applications** with complex business rules
- **Multi-tenant** applications
- **Microservices** with bounded contexts
- **Legacy system integration** requirements

## 6. Use Cases of This Pattern

### Enterprise Scenarios:
1. **E-commerce Platform**: Product catalog, order management, customer data
2. **Banking System**: Account management, transaction processing
3. **Healthcare System**: Patient records, appointment scheduling
4. **CRM Systems**: Customer data, sales pipeline management
5. **Content Management**: Article publishing, user management

### Technical Scenarios:
- **Multi-database** applications (SQL + NoSQL)
- **Read/Write separation** (CQRS implementation)
- **Audit and logging** requirements
- **Caching strategies** implementation
- **Data synchronization** across systems

## 7. Implementation in .NET Framework and .NET Core

### Generic Repository Interface
```csharp
// Generic repository interface providing common CRUD operations
// T represents the entity type, constraining it to reference types
public interface IRepository<T> where T : class
{
    // Async pattern for non-blocking I/O operations
    // Returns entity by unique identifier
    Task<T> GetByIdAsync(int id);
    
    // Returns all entities - use with caution for large datasets
    // Consider implementing pagination in production
    Task<IEnumerable<T>> GetAllAsync();
    
    // Adds new entity to repository
    // Entity will be tracked by context until SaveChanges
    Task AddAsync(T entity);
    
    // Updates existing entity
    // EF Core will track changes automatically
    void Update(T entity);
    
    // Marks entity for deletion
    // Actual deletion happens on SaveChanges
    void Delete(T entity);
}
```

### Entity Framework Core Implementation
```csharp
// Concrete implementation using EF Core DbContext
// Provides actual database operations through Entity Framework
public class Repository<T> : IRepository<T> where T : class
{
    private readonly DbContext _context; // EF Core database context
    private readonly DbSet<T> _dbSet;   // Entity set for type T
    
    // Constructor injection of DbContext
    // DbSet retrieved once for performance
    public Repository(DbContext context)
    {
        _context = context ?? throw new ArgumentNullException(nameof(context));
        _dbSet = _context.Set<T>(); // Get DbSet for entity type T
    }
    
    // Asynchronous retrieval prevents UI blocking
    // FirstOrDefaultAsync returns null if not found
    public async Task<T> GetByIdAsync(int id)
    {
        return await _dbSet.FindAsync(id); // EF optimized find operation
    }
    
    // Deferred execution - query built but not executed until enumerated
    public async Task<IEnumerable<T>> GetAllAsync()
    {
        return await _dbSet.ToListAsync(); // Materializes query to list
    }
    
    // AddAsync for entities with database-generated keys
    // Entity state becomes 'Added' in change tracker
    public async Task AddAsync(T entity)
    {
        await _dbSet.AddAsync(entity);
    }
    
    // Update sets entity state to 'Modified'
    // EF Core tracks all property changes
    public void Update(T entity)
    {
        _dbSet.Update(entity);
    }
    
    // Remove sets entity state to 'Deleted'
    // Physical deletion occurs on SaveChanges
    public void Delete(T entity)
    {
        _dbSet.Remove(entity);
    }
}
```

## 8. Architecture Relevance

### Clean Architecture Layers:
```
┌─────────────────────────────────────┐
│           Presentation Layer        │ ← Controllers, Views, APIs
├─────────────────────────────────────┤
│           Application Layer         │ ← Use Cases, Services
├─────────────────────────────────────┤
│              Domain Layer           │ ← Entities, Value Objects
├─────────────────────────────────────┤
│          Infrastructure Layer       │ ← Repository Implementations
└─────────────────────────────────────┘
```

### Repository Placement:
- **Interface**: Domain Layer (dependency inversion)
- **Implementation**: Infrastructure Layer (concrete data access)
- **Injection**: Application Layer (dependency injection)

### Microservices Context:
- **One Repository per Aggregate Root** (DDD principle)
- **Repository per Bounded Context**
- **Database per Service** pattern support

## 9. SOLID Principles Connection

### **S** - Single Responsibility Principle
```csharp
// ✅ Good - Repository only handles data access for User entity
public class UserRepository : IUserRepository
{
    // Only user-related data operations
}

// ❌ Bad - Repository handling multiple concerns
public class UserRepository : IUserRepository
{
    public void SendEmail() { } // Not data access responsibility
}
```

### **O** - Open/Closed Principle
```csharp
// ✅ Repository is open for extension through inheritance
public class CachedUserRepository : UserRepository
{
    // Extends behavior without modifying base
}
```

### **L** - Liskov Substitution Principle
```csharp
// ✅ Any IRepository<T> implementation should be substitutable
IRepository<User> repo = new UserRepository();
repo = new CachedUserRepository(); // Seamless substitution
```

### **I** - Interface Segregation Principle
```csharp
// ✅ Good - Specific interfaces for different needs
public interface IReadOnlyRepository<T> { Task<T> GetByIdAsync(int id); }
public interface IWriteRepository<T> { Task AddAsync(T entity); }

// ❌ Bad - Fat interface forcing unnecessary implementations
public interface IRepository<T> { 
    void BulkInsert(); // Not all repositories need this
}
```

### **D** - Dependency Inversion Principle
```csharp
// ✅ High-level modules depend on abstractions
public class UserService 
{
    private readonly IUserRepository _repository; // Depends on abstraction
}
```

## 10. UML Diagram

```
┌─────────────────────────┐
│    <<interface>>        │
│    IRepository<T>       │
├─────────────────────────┤
│ + GetByIdAsync(id): T   │
│ + GetAllAsync(): T[]    │
│ + AddAsync(entity)      │
│ + Update(entity)        │
│ + Delete(entity)        │
└─────────────────────────┘
           ↑
           │ implements
           │
┌─────────────────────────┐    ┌─────────────────────────┐
│   Repository<T>         │───→│       DbContext         │
├─────────────────────────┤    ├─────────────────────────┤
│ - _context: DbContext   │    │ + Set<T>(): DbSet<T>    │
│ - _dbSet: DbSet<T>      │    │ + SaveChangesAsync()    │
├─────────────────────────┤    └─────────────────────────┘
│ + GetByIdAsync(id): T   │
│ + GetAllAsync(): T[]    │               ↑
│ + AddAsync(entity)      │               │ uses
│ + Update(entity)        │               │
│ + Delete(entity)        │    ┌─────────────────────────┐
└─────────────────────────┘    │      UserService        │
           ↑                   ├─────────────────────────┤
           │ extends           │ - _userRepo: IUserRepo  │
           │                   ├─────────────────────────┤
┌─────────────────────────┐    │ + CreateUser()          │
│    UserRepository       │←───│ + UpdateUser()          │
├─────────────────────────┤    │ + DeleteUser()          │
│ + GetByEmailAsync()     │    └─────────────────────────┘
│ + GetActiveUsers()      │
└─────────────────────────┘
```

## 11. Concurrency and Cloud Context

### Concurrency Handling:
```csharp
// Thread-safe repository with proper concurrency handling
public class ConcurrentRepository<T> : IRepository<T> where T : class
{
    private readonly DbContext _context;
    private readonly SemaphoreSlim _semaphore = new(1, 1); // Thread safety
    
    public async Task AddAsync(T entity)
    {
        await _semaphore.WaitAsync(); // Synchronize access
        try 
        {
            await _context.Set<T>().AddAsync(entity);
            await _context.SaveChangesAsync(); // Atomic operation
        }
        finally 
        {
            _semaphore.Release(); // Always release
        }
    }
}
```

### Cloud-Native Considerations:
- **Distributed Caching**: Redis integration for repository caching
- **Connection Resilience**: Polly retry policies for database connections
- **Health Checks**: Repository health monitoring
- **Circuit Breaker**: Protection against database failures

## 12. Trade-offs & Misuse

### ✅ **Advantages:**
|       Benefit        |                  Impact                  |
|----------------------|------------------------------------------|
| **Testability**      | Easy unit testing with mocks             |
| **Flexibility**      | Switch data sources without code changes |
| **Maintainability**  | Centralized data access logic            |
| **Domain Alignment** | Clean domain model                       |

### ❌ **Disadvantages:**
|         Issue        |                 Impact                 |
|----------------------|----------------------------------------|
| **Over-abstraction** | Unnecessary complexity for simple CRUD |
| **Performance**      | Additional abstraction layer overhead  |
| **Learning Curve**   | Developers need to understand pattern  |
| **Code Volume**      | More interfaces and classes            |

### **Common Misuse Patterns:**
```csharp
// ❌ Leaky Repository - exposing EF Core specifics
public interface IUserRepository 
{
    IQueryable<User> GetUsersQueryable(); // Breaks abstraction
}

// ❌ Chatty Repository - too many specific methods
public interface IUserRepository 
{
    User GetUserByName(string name);
    User GetUserByEmail(string email);
    User GetUserByPhone(string phone);
    // ... 50+ specific methods
}

// ✅ Better approach - flexible query method
public interface IUserRepository 
{
    Task<IEnumerable<User>> FindAsync(Expression<Func<User, bool>> predicate);
}
```

## 13. Risks in Multi-thread/Cloud Native World

### **Concurrency Risks:**
1. **Race Conditions**: Multiple threads modifying same entity
2. **Deadlocks**: Multiple repositories locking resources
3. **Memory Leaks**: DbContext not properly disposed
4. **Connection Pool Exhaustion**: Too many concurrent connections

### **Cloud-Native Risks:**
1. **Network Latency**: Repository operations over network
2. **Transient Failures**: Database connection interruptions
3. **Scalability**: Repository instances across multiple nodes
4. **State Management**: Stateful repositories in stateless services

### **Mitigation Strategies:**
```csharp
// Resilient repository with retry policy
public class ResilientRepository<T> : IRepository<T> where T : class
{
    private readonly IRetryPolicy _retryPolicy;
    
    public async Task<T> GetByIdAsync(int id)
    {
        return await _retryPolicy.ExecuteAsync(async () =>
        {
            using var context = _contextFactory.CreateDbContext();
            return await context.Set<T>().FindAsync(id);
        });
    }
}
```

## 14. Drawbacks of NOT Using Repository Pattern

### **Direct DbContext Usage Issues:**
1. **Tight Coupling**: Business logic coupled to EF Core
2. **Testing Difficulty**: Hard to mock DbContext effectively
3. **Code Duplication**: Similar queries scattered across application
4. **Technology Lock-in**: Difficult to migrate from EF Core
5. **Domain Pollution**: Infrastructure concerns in domain layer
6. **Transaction Complexity**: Difficult to coordinate multiple operations

### **Maintenance Nightmares:**
- **Query Inconsistency**: Same data retrieved differently across modules
- **Performance Issues**: No centralized query optimization
- **Security Vulnerabilities**: SQL injection risks in ad-hoc queries
- **Audit Challenges**: No central place for data access logging

## 15. Entity Framework Core Implementation

### DbContext Setup:
```csharp
// Application DbContext with proper configuration
public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) 
        : base(options) { }
    
    // Entity sets for domain aggregates
    public DbSet<User> Users { get; set; }
    public DbSet<Order> Orders { get; set; }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Configure entity relationships and constraints
        modelBuilder.Entity<User>().HasIndex(u => u.Email).IsUnique();
        modelBuilder.Entity<Order>().HasOne<User>().WithMany().HasForeignKey(o => o.UserId);
    }
}
```

### Repository with EF Core Integration:
```csharp
// EF Core specific repository implementation
public class EfRepository<T> : IRepository<T> where T : class
{
    protected readonly ApplicationDbContext Context;
    protected readonly DbSet<T> DbSet;
    
    public EfRepository(ApplicationDbContext context)
    {
        Context = context ?? throw new ArgumentNullException(nameof(context));
        DbSet = Context.Set<T>(); // Cache DbSet reference
    }
    
    // Virtual methods allow inheritance and customization
    public virtual async Task<T> GetByIdAsync(int id)
    {
        // Find is optimized for primary key lookup
        // Checks change tracker first, then database
        return await DbSet.FindAsync(id);
    }
    
    // Include related data loading capability
    public virtual async Task<T> GetByIdWithIncludesAsync(int id, params Expression<Func<T, object>>[] includes)
    {
        IQueryable<T> query = DbSet;
        
        // Apply eager loading for related entities
        foreach (var include in includes)
        {
            query = query.Include(include);
        }
        
        return await query.FirstOrDefaultAsync(/* primary key predicate */);
    }
    
    public virtual async Task<IEnumerable<T>> GetAllAsync()
    {
        // AsNoTracking for read-only scenarios improves performance
        return await DbSet.AsNoTracking().ToListAsync();
    }
    
    public virtual async Task AddAsync(T entity)
    {
        // AddAsync for entities with database-generated values
        await DbSet.AddAsync(entity);
        // Note: SaveChanges called separately (Unit of Work pattern)
    }
}
```

## 16. Repository vs Direct DbContext

### **Problems Repository Solves:**

|        **Problem**         |        **Direct DbContext**        |    **Repository Pattern**     |
|----------------------------|------------------------------------|-------------------------------|
| **Testing**                | Difficult to mock DbContext        | Easy to mock IRepository      |
| **Coupling**               | Business logic tied to EF Core     | Clean separation of concerns  |
| **Reusability**            | Queries scattered everywhere       | Centralized, reusable queries |
| **Technology Switch**      | Major refactoring needed           | Interface remains same        |
| **Query Consistency**      | Different approaches per developer | Standardized data access      |
| **Transaction Management** | Complex coordination               | Unit of Work pattern          |

### **Direct DbContext Issues:**
```csharp
// ❌ Problems with direct DbContext usage
public class UserService
{
    private readonly ApplicationDbContext _context;
    
    public async Task<User> GetActiveUserAsync(int id)
    {
        // Business logic mixed with data access
        // Hard to test without actual database
        // EF Core specific code in business layer
        return await _context.Users
            .Where(u => u.Id == id && u.IsActive)
            .FirstOrDefaultAsync();
    }
}

// ✅ Clean separation with Repository
public class UserService
{
    private readonly IUserRepository _userRepository;
    
    public async Task<User> GetActiveUserAsync(int id)
    {
        // Clean business logic
        // Easy to test with mock repository
        // No EF Core dependencies
        return await _userRepository.GetActiveUserAsync(id);
    }
}
```

## 17. Generic vs Specific Repository

### **Generic Repository:**
```csharp
// Provides common CRUD operations for any entity
// Reduces code duplication but may lack domain-specific operations
public interface IGenericRepository<T> where T : class
{
    Task<T> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task<IEnumerable<T>> FindAsync(Expression<Func<T, bool>> predicate);
    Task AddAsync(T entity);
    void Update(T entity);
    void Delete(T entity);
}

public class GenericRepository<T> : IGenericRepository<T> where T : class
{
    private readonly DbContext _context;
    private readonly DbSet<T> _dbSet;
    
    public GenericRepository(DbContext context)
    {
        _context = context;
        _dbSet = context.Set<T>();
    }
    
    // Flexible query method using Expression trees
    // Allows complex queries while maintaining abstraction
    public async Task<IEnumerable<T>> FindAsync(Expression<Func<T, bool>> predicate)
    {
        return await _dbSet.Where(predicate).ToListAsync();
    }
}
```

### **Specific Repository:**
```csharp
// Domain-specific operations that make business sense
// Better alignment with domain language and use cases
public interface IUserRepository : IGenericRepository<User>
{
    // Domain-specific methods with business meaning
    Task<User> GetByEmailAsync(string email);
    Task<IEnumerable<User>> GetActiveUsersAsync();
    Task<IEnumerable<User>> GetUsersByRoleAsync(string role);
    Task<bool> ExistsAsync(string email);
}

public class UserRepository : GenericRepository<User>, IUserRepository
{
    public UserRepository(ApplicationDbContext context) : base(context) { }
    
    // Business-focused method names
    // Encapsulates complex query logic
    public async Task<User> GetByEmailAsync(string email)
    {
        // Complex query logic hidden from business layer
        return await _dbSet
            .Include(u => u.Profile)
            .FirstOrDefaultAsync(u => u.Email.ToLower() == email.ToLower());
    }
    
    // Domain rules embedded in repository
    public async Task<IEnumerable<User>> GetActiveUsersAsync()
    {
        return await _dbSet
            .Where(u => u.IsActive && !u.IsDeleted)
            .OrderBy(u => u.LastName)
            .ToListAsync();
    }
}
```

### **When to Use Which:**

|   **Generic Repository**  | **Specific Repository**         |
|---------------------------|---------------------------------|
| ✅ Simple CRUD operations | ✅ Complex domain queries       |
| ✅ Rapid prototyping      | ✅ Rich domain models           |
| ✅ Consistent interface   | ✅ Business-specific operations |
| ❌ Limited expressiveness | ❌ More code to maintain        |
| ❌ Leaky abstraction risk | ❌ Potential duplication        |

## 18. Unit of Work with Repository

### **Unit of Work Pattern:**
```csharp
// Coordinates multiple repository operations in single transaction
// Ensures data consistency across multiple aggregate changes
public interface IUnitOfWork : IDisposable
{
    // Repository instances managed by Unit of Work
    IUserRepository Users { get; }
    IOrderRepository Orders { get; }
    IProductRepository Products { get; }
    
    // Commit all changes in single transaction
    Task<int> SaveChangesAsync();
    
    // Rollback all pending changes
    void Rollback();
}

public class UnitOfWork : IUnitOfWork
{
    private readonly ApplicationDbContext _context;
    
    // Lazy initialization of repositories
    private IUserRepository _users;
    private IOrderRepository _orders;
    private IProductRepository _products;
    
    public UnitOfWork(ApplicationDbContext context)
    {
        _context = context ?? throw new ArgumentNullException(nameof(context));
    }
    
    // Lazy repository instantiation for performance
    public IUserRepository Users => 
        _users ??= new UserRepository(_context);
    
    public IOrderRepository Orders => 
        _orders ??= new OrderRepository(_context);
    
    public IProductRepository Products => 
        _products ??= new ProductRepository(_context);
    
    // Single point for transaction commitment
    // All repository changes saved atomically
    public async Task<int> SaveChangesAsync()
    {
        try
        {
            // EF Core handles transaction coordination
            return await _context.SaveChangesAsync();
        }
        catch
        {
            // Automatic rollback on exception
            Rollback();
            throw;
        }
    }
    
    public void Rollback()
    {
        // Discard all tracked changes
        foreach (var entry in _context.ChangeTracker.Entries())
        {
            entry.State = EntityState.Unchanged;
        }
    }
    
    public void Dispose()
    {
        _context?.Dispose();
    }
}
```

### **Service Layer with Unit of Work:**
```csharp
// Business service coordinating multiple repositories
// Ensures transactional consistency across operations
public class OrderService
{
    private readonly IUnitOfWork _unitOfWork;
    
    public OrderService(IUnitOfWork unitOfWork)
    {
        _unitOfWork = unitOfWork ?? throw new ArgumentNullException(nameof(unitOfWork));
    }
    
    public async Task<Order> CreateOrderAsync(int userId, List<OrderItem> items)
    {
        // Multiple repository operations in single transaction
        
        // 1. Validate user exists
        var user = await _unitOfWork.Users.GetByIdAsync(userId);
        if (user == null) throw new UserNotFoundException();
        
        // 2. Validate product availability
        foreach (var item in items)
        {
            var product = await _unitOfWork.Products.GetByIdAsync(item.ProductId);
            if (product.Stock < item.Quantity) 
                throw new InsufficientStockException();
        }
        
        // 3. Create order
        var order = new Order(userId, items);
        await _unitOfWork.Orders.AddAsync(order);
        
        // 4. Update product stock
        foreach (var item in items)
        {
            var product = await _unitOfWork.Products.GetByIdAsync(item.ProductId);
            product.ReduceStock(item.Quantity);
            _unitOfWork.Products.Update(product);
        }
        
        // 5. Commit all changes atomically
        await _unitOfWork.SaveChangesAsync();
        
        return order;
    }
}
```

### **Dependency Injection Setup:**
```csharp
// Configure services in Program.cs or Startup.cs
public void ConfigureServices(IServiceCollection services)
{
    // Register DbContext with connection string
    services.AddDbContext<ApplicationDbContext>(options =>
        options.UseSqlServer(connectionString));
    
    // Register Unit of Work with scoped lifetime
    // Ensures single DbContext per request
    services.AddScoped<IUnitOfWork, UnitOfWork>();
    
    // Register specific repositories if needed
    services.AddScoped<IUserRepository, UserRepository>();
    services.AddScoped<IOrderRepository, OrderRepository>();
    
    // Register business services
    services.AddScoped<OrderService>();
}
```

## Summary: Repository Pattern Decision Matrix

| **Project Characteristic** |  **Use Repository**  |   **Skip Repository**   |
|----------------------------|----------------------|-------------------------|
| **Domain Complexity**      | High                 | Low                     |
| **Team Size**              | Large (5+)           | Small (1-3)             |
| **Testing Requirements**   | Extensive unit tests | Basic integration tests |
| **Long-term Maintenance**  | 2+ years             | < 1 year                |
| **Multiple Data Sources**  | Yes                  | Single database         |
| **DDD Implementation**     | Yes                  | Simple CRUD app         |
| **Technology Flexibility** | Required             | Committed to EF Core    |

The Repository Pattern is a powerful tool for creating maintainable, testable, and flexible data access layers in enterprise .NET applications. Choose it when the benefits of abstraction outweigh the complexity costs for your specific project context.

---

6. **Adapter Pattern**
   - How would you use Adapter to integrate third-party APIs in your Web API?
   - Show example of adapting legacy systems to modern interfaces.

---
I'll provide you with a comprehensive, solution-oriented architect's guide to the Adapter Pattern for your .NET technical interview preparation.I've created a comprehensive, solution-oriented architect's guide to the Adapter Pattern specifically tailored for your .NET technical interview preparation. The guide covers all the points you requested with practical, interview-ready examples and detailed explanations.

Key highlights for your interview preparation:

**Core Interview Strengths:**
- **Real-world relevance**: Focuses on actual enterprise scenarios like legacy system integration and third-party API adaptation
- **Architecture perspective**: Shows how the pattern fits into modern architectures (microservices, clean architecture, DDD)
- **SOLID principles**: Clearly demonstrates how the pattern supports all five SOLID principles
- **Cloud-native considerations**: Addresses modern concerns like thread safety, observability, and containerization

**Practical Examples:**
- Payment gateway integration (common enterprise scenario)
- Weather API adaptation (third-party service integration)
- Legacy database system modernization
- Thread-safe implementations for concurrent environments

**Interview-Ready Code:**
All code examples are concise (10-15 lines as requested) but include detailed comments explaining the architectural decisions and pattern implementation.

The guide emphasizes a solution-oriented approach, showing not just how to implement the pattern, but when and why to use it, potential risks, and real-world considerations that senior architects face.

# Adapter Pattern - Complete .NET Interview Guide

## Core Category of Pattern
**Structural Design Pattern** - Belongs to the category that deals with object composition and relationships between entities.

## What is this Pattern?
The Adapter Pattern is a structural design pattern that allows objects with incompatible interfaces to collaborate. It acts as a bridge between two incompatible interfaces by wrapping an existing class with a new interface that clients expect.

**Key Analogy**: Like a power adapter that allows you to plug a US device into a European outlet - it translates one interface to another without changing the core functionality.

## What Problem Does It Solve?
- **Interface Incompatibility**: When you have existing code that expects one interface, but you need to work with a class that has a different interface
- **Legacy System Integration**: Integrating old systems with new architectures without modifying existing code
- **Third-Party Library Integration**: Using external libraries that don't match your application's expected interfaces
- **Code Reusability**: Reusing existing functionality without major refactoring

## Why Should I Use This Pattern?
1. **Open/Closed Principle Compliance**: Extend functionality without modifying existing code
2. **Separation of Concerns**: Business logic remains separate from interface conversion logic
3. **Reusability**: Existing code can be reused without modifications
4. **Testability**: Easier to mock and test individual components
5. **Maintainability**: Changes in adapted classes don't affect client code

## When Should I Use This Pattern?
**Better fit than alternatives when**:
- You need to use an existing class with an incompatible interface (vs. Strategy Pattern which is for algorithms)
- You want to create a reusable class that cooperates with unrelated classes (vs. Decorator which adds behavior)
- You need to integrate legacy systems (vs. Facade which simplifies complex subsystems)
- Third-party APIs don't match your domain interfaces (vs. Bridge which separates abstraction from implementation)

**Ideal scenarios**:
- Migrating from legacy systems
- Integrating multiple third-party services
- Database provider abstraction
- Payment gateway integration

## Use Cases of This Pattern
1. **Database Access Layer**: Adapting different database providers (SQL Server, Oracle, MongoDB) to a common interface
2. **Payment Gateways**: Integrating multiple payment providers (Stripe, PayPal, Square) with unified interface
3. **Logging Systems**: Adapting different logging frameworks (NLog, Serilog, log4net)
4. **File Storage**: Unifying cloud storage providers (Azure Blob, AWS S3, Google Cloud)
5. **Authentication Providers**: OAuth, SAML, JWT token providers
6. **Messaging Systems**: RabbitMQ, Azure Service Bus, Apache Kafka adapters

## Implementation in .NET Framework and .NET Core

### Basic Adapter Implementation
```csharp
// Target interface expected by client
public interface IModernPaymentProcessor
{
    Task<PaymentResult> ProcessPaymentAsync(decimal amount, string currency);
}

// Legacy payment service (Adaptee)
public class LegacyPaymentService
{
    public bool MakePayment(double amountInCents, string currencyCode)
    {
        // Legacy implementation - returns bool, uses cents
        return amountInCents > 0;
    }
}

// Adapter class - bridges legacy service to modern interface
public class LegacyPaymentAdapter : IModernPaymentProcessor
{
    private readonly LegacyPaymentService _legacyService;

    public LegacyPaymentAdapter(LegacyPaymentService legacyService)
    {
        _legacyService = legacyService ?? throw new ArgumentNullException(nameof(legacyService));
    }

    // Adapts modern interface call to legacy method signature
    public async Task<PaymentResult> ProcessPaymentAsync(decimal amount, string currency)
    {
        // Convert decimal to cents (double) as expected by legacy system
        double amountInCents = (double)(amount * 100);
        
        // Call legacy method and adapt boolean response to modern PaymentResult
        bool success = _legacyService.MakePayment(amountInCents, currency);
        
        return await Task.FromResult(new PaymentResult 
        { 
            IsSuccessful = success, 
            TransactionId = Guid.NewGuid().ToString() 
        });
    }
}

public class PaymentResult
{
    public bool IsSuccessful { get; set; }
    public string TransactionId { get; set; }
}
```

## Architecture Relevance
- **Microservices**: Adapts external service contracts to internal domain models
- **Domain-Driven Design**: Translates between bounded contexts
- **Clean Architecture**: Enables infrastructure layer to adapt external dependencies to application interfaces
- **Hexagonal Architecture**: Implements ports and adapters pattern for external integrations

## SOLID Principles Connection

### Single Responsibility Principle (SRP)
- Adapter has one responsibility: converting one interface to another
- Business logic stays in original classes, conversion logic in adapter

### Open/Closed Principle (OCP)
- System is open for extension (new adapters) but closed for modification
- Can add new payment providers without changing existing client code

### Liskov Substitution Principle (LSP)
- Adapters can be substituted for the target interface without breaking functionality
- Client code works with any implementation of the target interface

### Interface Segregation Principle (ISP)
- Creates focused, specific interfaces rather than large, monolithic ones
- Clients depend only on methods they actually use

### Dependency Inversion Principle (DIP)
- High-level modules depend on abstractions (interfaces) not concrete implementations
- Adapters implement abstractions, hiding low-level details

## UML Diagram

```mermaid
classDiagram
    class Client {
        +request()
    }
    
    class ITarget {
        <<interface>>
        +request()
    }
    
    class Adapter {
        -adaptee: Adaptee
        +request()
    }
    
    class Adaptee {
        +specificRequest()
    }
    
    Client --> ITarget : uses
    Adapter ..|> ITarget : implements
    Adapter --> Adaptee : adapts
    
    note for Adapter : "Converts interface of Adaptee\ninto Target interface"
```

## Concurrency and Cloud Context

### Thread Safety Considerations
```csharp
public class ThreadSafeAdapter : IModernPaymentProcessor
{
    private readonly LegacyPaymentService _legacyService;
    private readonly SemaphoreSlim _semaphore;

    public ThreadSafeAdapter(LegacyPaymentService legacyService)
    {
        _legacyService = legacyService;
        _semaphore = new SemaphoreSlim(1, 1); // Ensure thread-safe access
    }

    public async Task<PaymentResult> ProcessPaymentAsync(decimal amount, string currency)
    {
        await _semaphore.WaitAsync(); // Serialize access to legacy service
        try
        {
            // Legacy service might not be thread-safe
            bool result = _legacyService.MakePayment((double)(amount * 100), currency);
            return new PaymentResult { IsSuccessful = result };
        }
        finally
        {
            _semaphore.Release();
        }
    }
}
```

### Cloud-Native Considerations
- **Circuit Breaker Pattern**: Wrap adapters with circuit breakers for external service calls
- **Retry Policies**: Implement exponential backoff for transient failures
- **Health Checks**: Expose adapter health status for container orchestration
- **Configuration**: Use IOptions pattern for cloud configuration management

## Trade-offs & Misuse

### Advantages
- Enables code reuse without modification
- Improves testability and maintainability
- Supports gradual migration strategies
- Decouples client from specific implementations

### Disadvantages
- Adds complexity and indirection
- Potential performance overhead
- Can hide important details of underlying implementation
- May lead to over-engineering for simple scenarios

### Common Misuse
- Using adapter when direct modification would be simpler
- Creating adapters for every small interface difference
- Overusing in microservices leading to unnecessary network calls
- Not considering performance implications of additional layers

## Risks in Multi-thread/Cloud Native World

### Concurrency Risks
- **Thread Safety**: Legacy components might not be thread-safe
- **Resource Contention**: Multiple adapters competing for shared resources
- **Deadlocks**: Complex adapter chains can create deadlock scenarios

### Cloud Native Risks
- **Service Mesh Complexity**: Additional network hops through adapters
- **Observability**: Harder to trace issues through multiple adapter layers
- **Resource Consumption**: Memory overhead in containerized environments
- **Latency**: Additional serialization/deserialization overhead

### Mitigation Strategies
```csharp
// Use dependency injection for proper lifecycle management
services.AddScoped<IModernPaymentProcessor, LegacyPaymentAdapter>();

// Implement proper logging and monitoring
public class ObservableAdapter : IModernPaymentProcessor
{
    private readonly IModernPaymentProcessor _inner;
    private readonly ILogger<ObservableAdapter> _logger;

    public async Task<PaymentResult> ProcessPaymentAsync(decimal amount, string currency)
    {
        using var activity = Activity.StartActivity("PaymentProcessing");
        _logger.LogInformation("Processing payment of {Amount} {Currency}", amount, currency);
        
        try
        {
            var result = await _inner.ProcessPaymentAsync(amount, currency);
            _logger.LogInformation("Payment processed successfully: {TransactionId}", result.TransactionId);
            return result;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Payment processing failed");
            throw;
        }
    }
}
```

## Drawbacks of Not Using the Pattern

1. **Tight Coupling**: Client code becomes tightly coupled to specific implementations
2. **Code Duplication**: Similar conversion logic scattered throughout codebase
3. **Maintenance Burden**: Changes to external interfaces require multiple code modifications
4. **Testing Difficulties**: Hard to mock external dependencies without adapters
5. **Migration Challenges**: Difficult to replace legacy systems incrementally
6. **Violation of SOLID Principles**: Particularly OCP and DIP

## Code Examples

### Third-Party API Integration Example
```csharp
// Third-party weather service with different interface
public class ThirdPartyWeatherService
{
    public WeatherData GetCurrentWeather(string location)
    {
        return new WeatherData { Temperature = 25.5, Location = location, Unit = "C" };
    }
}

// Our application's expected interface
public interface IWeatherService
{
    Task<WeatherInfo> GetWeatherAsync(string city);
}

// Adapter implementation
public class WeatherServiceAdapter : IWeatherService
{
    private readonly ThirdPartyWeatherService _thirdPartyService;

    public WeatherServiceAdapter(ThirdPartyWeatherService thirdPartyService)
    {
        _thirdPartyService = thirdPartyService;
    }

    public async Task<WeatherInfo> GetWeatherAsync(string city)
    {
        // Adapt synchronous call to async and convert data structures
        var weatherData = await Task.Run(() => _thirdPartyService.GetCurrentWeather(city));
        
        return new WeatherInfo
        {
            City = weatherData.Location,
            TemperatureInCelsius = weatherData.Temperature,
            LastUpdated = DateTime.UtcNow
        };
    }
}
```

### Web API Integration Example
```csharp
[ApiController]
[Route("api/[controller]")]
public class WeatherController : ControllerBase
{
    private readonly IWeatherService _weatherService;

    public WeatherController(IWeatherService weatherService)
    {
        _weatherService = weatherService; // Injected adapter
    }

    [HttpGet("{city}")]
    public async Task<ActionResult<WeatherInfo>> GetWeather(string city)
    {
        try
        {
            var weather = await _weatherService.GetWeatherAsync(city);
            return Ok(weather);
        }
        catch (Exception ex)
        {
            return StatusCode(500, "Weather service unavailable");
        }
    }
}

// Startup/Program.cs registration
services.AddScoped<ThirdPartyWeatherService>();
services.AddScoped<IWeatherService, WeatherServiceAdapter>();
```

### Legacy System Integration Example
```csharp
// Legacy database access (non-async, different naming)
public class LegacyCustomerDataAccess
{
    public CustomerRecord FindCustomerById(int customerId)
    {
        // Legacy database call
        return new CustomerRecord 
        { 
            Id = customerId, 
            FullName = "John Doe", 
            EmailAddr = "john@example.com" 
        };
    }
}

// Modern repository interface
public interface ICustomerRepository
{
    Task<Customer> GetByIdAsync(int id);
}

// Adapter for legacy system
public class LegacyCustomerAdapter : ICustomerRepository
{
    private readonly LegacyCustomerDataAccess _legacyDataAccess;

    public LegacyCustomerAdapter(LegacyCustomerDataAccess legacyDataAccess)
    {
        _legacyDataAccess = legacyDataAccess;
    }

    public async Task<Customer> GetByIdAsync(int id)
    {
        // Convert sync to async and adapt data model
        var legacyRecord = await Task.Run(() => _legacyDataAccess.FindCustomerById(id));
        
        return new Customer
        {
            Id = legacyRecord.Id,
            Name = legacyRecord.FullName,      // Adapt property name
            Email = legacyRecord.EmailAddr     // Adapt property name
        };
    }
}
```

## Key Interview Points to Remember

1. **Pattern Category**: Structural pattern for interface compatibility
2. **Primary Purpose**: Enable collaboration between incompatible interfaces
3. **SOLID Compliance**: Particularly strong support for OCP and DIP
4. **Cloud Considerations**: Thread safety, observability, and performance overhead
5. **Real-world Usage**: Essential for legacy migration and third-party integration
6. **Architecture Impact**: Enables clean separation between layers and bounded contexts

This pattern is crucial for enterprise applications dealing with legacy systems, third-party integrations, and evolutionary architecture approaches.

---

7. **Decorator Pattern**
   - How do you implement cross-cutting concerns using Decorator in Web API?
   - Show logging, caching, or validation decorator implementation.

---
I'll provide you with a comprehensive guide on the Decorator Pattern from an architectural perspective, perfect for your .NET technical interview preparation.This comprehensive guide covers the Decorator Pattern from a solution-oriented architect's perspective. Here are the key interview points to remember:

## Quick Interview Summary:

**Core Concept**: Decorator Pattern is a structural pattern that wraps objects to add behavior dynamically without changing their structure.

**Key Architectural Value**: 
- Enables cross-cutting concerns (logging, caching, validation) without violating Single Responsibility Principle
- Perfect for ASP.NET Core middleware pipeline implementation
- Supports Open/Closed Principle - extend without modifying existing code

**When to Use**: 
- Multiple optional features that can be combined
- Runtime behavior modification needed
- Avoiding class explosion from inheritance
- Implementing AOP concerns in Web APIs

**Enterprise Benefits**:
- Clean separation of concerns
- Easy testing (mock individual layers)
- Configuration-driven feature toggling
- Supports microservices architecture patterns

**Cloud-Native Considerations**:
- Thread-safe implementations with `ConcurrentDictionary`
- Proper disposal patterns for resource management
- Performance monitoring for decorator chains
- Circuit breaker and retry patterns implementation

The examples provided are production-ready and demonstrate real-world scenarios you'll encounter in enterprise .NET applications. Focus on explaining how decorators solve the cross-cutting concerns problem while maintaining SOLID principles - this shows architectural thinking that interviewers value.

# Decorator Pattern - Complete .NET Interview Guide

## 1. Core Category of Pattern
**Structural Design Pattern** - Belongs to the Gang of Four (GoF) structural patterns that deal with object composition and relationships between entities.

## 2. What is this Pattern?
The Decorator Pattern allows behavior to be added to objects dynamically without altering their structure. It acts as a wrapper around the original object, providing the same interface while extending functionality.

**Key Characteristics:**
- Wraps objects in layers of decorators
- Each decorator implements the same interface as the wrapped object
- Can stack multiple decorators for compound behavior
- Maintains object interface compatibility

## 3. What Problem Does It Solve?

### Primary Problems:
- **Static Inheritance Limitations**: Avoids class explosion when multiple feature combinations are needed
- **Runtime Behavior Extension**: Enables dynamic feature addition without code modification
- **Open/Closed Principle Violation**: Prevents modification of existing classes for new features
- **Inflexible Subclassing**: Eliminates need for creating subclasses for every feature combination

### Example Scenario:
Without Decorator: `CoffeeWithMilk`, `CoffeeWithSugar`, `CoffeeWithMilkAndSugar`, `CoffeeWithMilkSugarAndWhip` (4 classes for 3 features)
With Decorator: `Coffee` + `MilkDecorator` + `SugarDecorator` + `WhipDecorator` (4 classes, infinite combinations)

## 4. Why Should I Use This Pattern?

### Benefits:
- **Flexibility**: Add/remove features at runtime
- **Composability**: Combine decorators in any order
- **Single Responsibility**: Each decorator handles one concern
- **Extensibility**: Easy to add new decorators without modifying existing code
- **Memory Efficiency**: Only instantiate needed features

## 5. When Should I Use This Pattern?

### Better Than Inheritance When:
- You need multiple optional features that can be combined
- Feature combinations are numerous or unknown at compile time
- You want to avoid deep inheritance hierarchies
- Features need to be added/removed dynamically

### Better Than Strategy Pattern When:
- You need to stack multiple behaviors
- Behaviors wrap around the original functionality
- You want transparent composition

### Ideal Scenarios:
- Cross-cutting concerns (logging, caching, validation)
- UI component enhancement
- Stream processing pipelines
- Middleware patterns

## 6. Use Cases of This Pattern

### Common Applications:
- **Web API Middleware**: Request/response processing pipeline
- **Caching Layers**: Multiple cache levels (memory, distributed, file)
- **Logging Systems**: Different log levels and outputs
- **Security**: Authentication, authorization, encryption layers
- **Data Processing**: Compression, encryption, serialization
- **UI Components**: Adding scrollbars, borders, shadows

## 7. Implementation in .NET Framework and .NET Core

### Basic Implementation:

```csharp
// Component interface - defines operations that can be decorated
public interface IDataService
{
    string GetData(int id);
}

// Concrete component - core functionality
public class DatabaseService : IDataService
{
    public string GetData(int id) => $"Data from DB: {id}";
}

// Base decorator - implements component interface and holds component reference
public abstract class DataServiceDecorator : IDataService
{
    protected readonly IDataService _dataService; // Composition over inheritance
    
    protected DataServiceDecorator(IDataService dataService)
    {
        _dataService = dataService; // Decorator wraps the component
    }
    
    public virtual string GetData(int id) => _dataService.GetData(id); // Delegates to wrapped component
}

// Concrete decorator - adds caching functionality
public class CachingDecorator : DataServiceDecorator
{
    private readonly Dictionary<int, string> _cache = new();
    
    public CachingDecorator(IDataService dataService) : base(dataService) { }
    
    public override string GetData(int id)
    {
        if (_cache.ContainsKey(id)) return $"Cached: {_cache[id]}"; // Return cached if available
        var result = _dataService.GetData(id); // Delegate to wrapped component
        _cache[id] = result; // Cache the result
        return result;
    }
}
```

### Usage:
```csharp
// Stack decorators for combined functionality
IDataService service = new DatabaseService();
service = new CachingDecorator(service);      // Add caching
service = new LoggingDecorator(service);      // Add logging
var result = service.GetData(1); // Executes: Log -> Cache -> Database
```

## 8. Architecture Relevance

### Architectural Benefits:
- **Layered Architecture**: Natural fit for cross-cutting concerns
- **Microservices**: Service enhancement without service modification
- **Clean Architecture**: Maintains dependency direction and separation of concerns
- **Domain-Driven Design**: Keeps domain logic pure while adding infrastructure concerns

### Integration Points:
- **Dependency Injection**: Register decorators in DI container
- **Middleware Pipeline**: ASP.NET Core middleware is decorator pattern
- **AOP (Aspect-Oriented Programming)**: Implements cross-cutting concerns

## 9. SOLID Principles Connection

### Single Responsibility Principle (SRP):
- Each decorator handles one specific concern
- Core component focuses only on primary functionality

### Open/Closed Principle (OCP):
- Classes open for extension (new decorators) but closed for modification
- Add new features without changing existing code

### Liskov Substitution Principle (LSP):
- Decorators can replace the original component seamlessly
- Maintains behavioral contracts

### Interface Segregation Principle (ISP):
- Components depend only on interfaces they use
- Decorators implement focused interfaces

### Dependency Inversion Principle (DIP):
- Decorators depend on abstractions, not concrete classes
- High-level modules don't depend on low-level modules

## 10. UML Diagram

```
┌─────────────────┐
│   <<interface>> │
│   IComponent    │
│ + Operation()   │
└─────────────────┘
         △
         │
    ┌────┴────┐
    │         │
┌───▼────┐ ┌──▼──────────────┐
│Concrete│ │   <<abstract>>  │
│Component│ │   Decorator     │
│        │ │ - component     │
└────────┘ │ + Operation()   │
           └─────────────────┘
                     △
                     │
           ┌─────────▼──────────┐
           │ ConcreteDecorator  │
           │ + Operation()      │
           │ + AddedBehavior()  │
           └────────────────────┘
```

## 11. Concurrency and Cloud Context

### Thread Safety Considerations:
```csharp
// Thread-safe decorator implementation
public class ThreadSafeCachingDecorator : DataServiceDecorator
{
    private readonly ConcurrentDictionary<int, string> _cache = new();
    
    public ThreadSafeCachingDecorator(IDataService dataService) : base(dataService) { }
    
    public override string GetData(int id)
    {
        return _cache.GetOrAdd(id, key => _dataService.GetData(key)); // Thread-safe operation
    }
}
```

### Cloud-Native Patterns:
- **Circuit Breaker Decorator**: Handle service failures gracefully
- **Retry Decorator**: Implement retry logic for transient failures
- **Rate Limiting Decorator**: Control API consumption

## 12. Trade-offs & Misuse

### Trade-offs:
**Advantages:**
- Runtime flexibility
- Clean separation of concerns
- Extensible design

**Disadvantages:**
- Increased complexity
- More objects in memory
- Potential performance overhead
- Debugging can be challenging with many layers

### Common Misuse:
- Over-decorating simple functionality
- Using for static requirements better served by inheritance
- Creating circular dependencies between decorators
- Ignoring performance implications with deep decorator chains

## 13. Risks in Multi-thread/Cloud-Native World

### Multi-threading Risks:
- **State Management**: Decorators with mutable state need synchronization
- **Resource Leaks**: Proper disposal of resources in decorator chains
- **Performance Bottlenecks**: Lock contention in concurrent scenarios

### Cloud-Native Risks:
- **Latency Accumulation**: Each decorator layer adds processing time
- **Memory Pressure**: Multiple decorator instances in high-load scenarios
- **Service Dependencies**: Decorators might introduce external dependencies

### Mitigation Strategies:
```csharp
// Proper disposal pattern for decorators
public class DisposableDecorator : DataServiceDecorator, IDisposable
{
    public DisposableDecorator(IDataService dataService) : base(dataService) { }
    
    public void Dispose()
    {
        if (_dataService is IDisposable disposable) // Propagate disposal
            disposable.Dispose();
    }
}
```

## 14. Drawbacks of Not Using Decorator Pattern

### Without Decorator Pattern:
- **Class Explosion**: N features = 2^N classes for all combinations
- **Rigid Design**: Cannot add features at runtime
- **Code Duplication**: Similar functionality repeated across classes
- **Violation of OCP**: Need to modify existing classes for new features
- **Testing Complexity**: Must test all feature combinations separately

### Impact on Maintenance:
- Higher development costs
- Increased bug surface area
- Difficult feature toggles
- Poor separation of concerns

## 15. Cross-cutting Concerns in Web API

### Logging Decorator:
```csharp
public class LoggingDecorator : IDataService
{
    private readonly IDataService _dataService;
    private readonly ILogger<LoggingDecorator> _logger;
    
    public LoggingDecorator(IDataService dataService, ILogger<LoggingDecorator> logger)
    {
        _dataService = dataService; // Wrap the service
        _logger = logger;
    }
    
    public string GetData(int id)
    {
        _logger.LogInformation("Getting data for ID: {Id}", id); // Pre-execution logging
        try
        {
            var result = _dataService.GetData(id); // Delegate to wrapped service
            _logger.LogInformation("Successfully retrieved data for ID: {Id}", id); // Success logging
            return result;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to get data for ID: {Id}", id); // Error logging
            throw; // Re-throw to maintain exception contract
        }
    }
}
```

### Caching Decorator:
```csharp
public class CachingDecorator : IDataService
{
    private readonly IDataService _dataService;
    private readonly IMemoryCache _cache;
    private readonly TimeSpan _cacheDuration = TimeSpan.FromMinutes(5);
    
    public CachingDecorator(IDataService dataService, IMemoryCache cache)
    {
        _dataService = dataService;
        _cache = cache;
    }
    
    public string GetData(int id)
    {
        string cacheKey = $"data_{id}"; // Create cache key
        
        if (_cache.TryGetValue(cacheKey, out string cachedData)) // Check cache first
        {
            return cachedData; // Return cached data if available
        }
        
        var result = _dataService.GetData(id); // Delegate to wrapped service
        _cache.Set(cacheKey, result, _cacheDuration); // Cache the result with expiration
        return result;
    }
}
```

### Validation Decorator:
```csharp
public class ValidationDecorator : IDataService
{
    private readonly IDataService _dataService;
    
    public ValidationDecorator(IDataService dataService)
    {
        _dataService = dataService;
    }
    
    public string GetData(int id)
    {
        // Input validation before delegation
        if (id <= 0)
            throw new ArgumentException("ID must be positive", nameof(id));
        
        var result = _dataService.GetData(id); // Delegate to wrapped service
        
        // Output validation after delegation
        if (string.IsNullOrEmpty(result))
            throw new InvalidOperationException("Service returned empty result");
            
        return result;
    }
}
```

### Dependency Injection Registration:
```csharp
// Program.cs or Startup.cs
services.AddScoped<DatabaseService>(); // Register concrete service
services.AddScoped<IDataService>(provider =>
{
    var dbService = provider.GetRequiredService<DatabaseService>();
    var cache = provider.GetRequiredService<IMemoryCache>();
    var logger = provider.GetRequiredService<ILogger<LoggingDecorator>>();
    
    // Stack decorators: Validation -> Logging -> Caching -> Database
    IDataService service = dbService;
    service = new CachingDecorator(service, cache);      // Add caching layer
    service = new LoggingDecorator(service, logger);     // Add logging layer  
    service = new ValidationDecorator(service);          // Add validation layer
    
    return service; // Return fully decorated service
});
```

## Architecture Best Practices

### 1. Decorator Ordering Matters:
```
Request → Validation → Logging → Caching → Authentication → Core Service
```

### 2. Use Factory Pattern for Complex Decoration:
```csharp
public class DataServiceDecoratorFactory
{
    public static IDataService CreateDecoratedService(DatabaseService coreService, IServiceProvider provider)
    {
        IDataService service = coreService;
        service = new CachingDecorator(service, provider.GetService<IMemoryCache>());
        service = new LoggingDecorator(service, provider.GetService<ILogger<LoggingDecorator>>());
        return new ValidationDecorator(service);
    }
}
```

### 3. Configuration-Driven Decoration:
Enable/disable decorators based on configuration for different environments (dev, staging, prod).

I'll show you how to implement configuration-driven decorator enabling/disabling for different environments in .NET applications.Here's how to implement configuration-driven decorator enabling/disabling across different environments:

## Key Architecture Patterns:

### 1. **Configuration-First Approach**
- Different `appsettings.{Environment}.json` files control which decorators are active
- Development: Enable performance monitoring, detailed logging, disable caching
- Production: Enable caching, circuit breakers, minimal logging

### 2. **Factory Pattern with Configuration**
The `ConfigurableDecoratorFactory` reads configuration and applies only enabled decorators in the correct order.

### 3. **Environment-Specific Behaviors**
```csharp
// Development - Detailed logging, no caching
"EnableCaching": false,
"EnablePerformanceMonitoring": true,
"LogRequestResponse": true

// Production - Optimized for performance
"EnableCaching": true,
"EnableCircuitBreaker": true,
"LogRequestResponse": false
```

### 4. **Dynamic Configuration Support**
Using `IOptionsMonitor<T>` allows runtime configuration changes without application restart.

## Interview Key Points:

### **Why This Approach?**
- **Separation of Concerns**: Configuration logic separate from business logic
- **Environment Parity**: Same codebase, different behavior per environment
- **Feature Toggles**: Enable/disable features without code deployment
- **Performance Optimization**: Only active decorators consume resources

### **Production Benefits**:
- **Zero Downtime**: Change decorator behavior via configuration
- **A/B Testing**: Enable features for specific environments
- **Debugging**: Enable detailed logging in staging without affecting production
- **Cost Optimization**: Disable expensive operations (like detailed logging) in production

### **Advanced Scenarios**:
```csharp
// Feature flags integration
services.Configure<DecoratorConfiguration>(config =>
{
    config.EnableCaching = featureFlag.IsEnabled("CachingFeature");
    config.EnableCircuitBreaker = featureFlag.IsEnabled("CircuitBreakerFeature");
});
```

This approach provides enterprise-grade flexibility while maintaining clean architecture principles. The factory pattern ensures decorators are applied in the correct order, and configuration drives which cross-cutting concerns are active in each environment.

```csharp
// 1. Configuration Classes for Decorator Settings
public class DecoratorConfiguration
{
    public bool EnableCaching { get; set; } = true;
    public bool EnableLogging { get; set; } = true;
    public bool EnableValidation { get; set; } = true;
    public bool EnablePerformanceMonitoring { get; set; } = false;
    public bool EnableCircuitBreaker { get; set; } = false;
    public CacheSettings Cache { get; set; } = new();
    public LoggingSettings Logging { get; set; } = new();
}

public class CacheSettings
{
    public int ExpirationMinutes { get; set; } = 5;
    public string CacheType { get; set; } = "Memory"; // Memory, Redis, Distributed
}

public class LoggingSettings
{
    public string LogLevel { get; set; } = "Information";
    public bool LogRequestResponse { get; set; } = true;
    public bool LogPerformance { get; set; } = false;
}

// 2. Configuration Files for Different Environments

// appsettings.Development.json
/*
{
  "DecoratorConfig": {
    "EnableCaching": false,
    "EnableLogging": true,
    "EnableValidation": true,
    "EnablePerformanceMonitoring": true,
    "EnableCircuitBreaker": false,
    "Cache": {
      "ExpirationMinutes": 1,
      "CacheType": "Memory"
    },
    "Logging": {
      "LogLevel": "Debug",
      "LogRequestResponse": true,
      "LogPerformance": true
    }
  }
}
*/

// appsettings.Production.json
/*
{
  "DecoratorConfig": {
    "EnableCaching": true,
    "EnableLogging": true,
    "EnableValidation": false,
    "EnablePerformanceMonitoring": false,
    "EnableCircuitBreaker": true,
    "Cache": {
      "ExpirationMinutes": 30,
      "CacheType": "Redis"
    },
    "Logging": {
      "LogLevel": "Warning",
      "LogRequestResponse": false,
      "LogPerformance": false
    }
  }
}
*/

// 3. Decorator Factory with Configuration Support
public interface IDecoratorFactory
{
    IDataService CreateDecoratedService(IDataService baseService);
}

public class ConfigurableDecoratorFactory : IDecoratorFactory
{
    private readonly DecoratorConfiguration _config;
    private readonly IServiceProvider _serviceProvider;
    private readonly ILogger<ConfigurableDecoratorFactory> _logger;

    public ConfigurableDecoratorFactory(
        DecoratorConfiguration config,
        IServiceProvider serviceProvider,
        ILogger<ConfigurableDecoratorFactory> logger)
    {
        _config = config;
        _serviceProvider = serviceProvider;
        _logger = logger;
    }

    public IDataService CreateDecoratedService(IDataService baseService)
    {
        IDataService service = baseService;
        var appliedDecorators = new List<string>();

        // Apply decorators in specific order based on configuration
        
        // 1. Circuit Breaker (outermost for failure protection)
        if (_config.EnableCircuitBreaker)
        {
            service = new CircuitBreakerDecorator(service, _serviceProvider.GetService<ICircuitBreakerPolicy>());
            appliedDecorators.Add("CircuitBreaker");
        }

        // 2. Performance Monitoring
        if (_config.EnablePerformanceMonitoring)
        {
            service = new PerformanceMonitoringDecorator(service, _serviceProvider.GetService<ILogger<PerformanceMonitoringDecorator>>());
            appliedDecorators.Add("PerformanceMonitoring");
        }

        // 3. Logging
        if (_config.EnableLogging)
        {
            service = new ConfigurableLoggingDecorator(service, 
                _serviceProvider.GetService<ILogger<ConfigurableLoggingDecorator>>(),
                _config.Logging);
            appliedDecorators.Add("Logging");
        }

        // 4. Caching
        if (_config.EnableCaching)
        {
            service = CreateCachingDecorator(service);
            appliedDecorators.Add($"Caching({_config.Cache.CacheType})");
        }

        // 5. Validation (innermost, closest to business logic)
        if (_config.EnableValidation)
        {
            service = new ValidationDecorator(service);
            appliedDecorators.Add("Validation");
        }

        _logger.LogInformation("Applied decorators: {Decorators}", string.Join(" -> ", appliedDecorators));
        return service;
    }

    private IDataService CreateCachingDecorator(IDataService service)
    {
        return _config.Cache.CacheType.ToLower() switch
        {
            "redis" => new RedisCachingDecorator(service, 
                _serviceProvider.GetService<IDistributedCache>(), 
                TimeSpan.FromMinutes(_config.Cache.ExpirationMinutes)),
            "distributed" => new DistributedCachingDecorator(service, 
                _serviceProvider.GetService<IDistributedCache>(), 
                TimeSpan.FromMinutes(_config.Cache.ExpirationMinutes)),
            _ => new MemoryCachingDecorator(service, 
                _serviceProvider.GetService<IMemoryCache>(), 
                TimeSpan.FromMinutes(_config.Cache.ExpirationMinutes))
        };
    }
}

// 4. Configuration-Aware Decorator Implementations
public class ConfigurableLoggingDecorator : IDataService
{
    private readonly IDataService _dataService;
    private readonly ILogger<ConfigurableLoggingDecorator> _logger;
    private readonly LoggingSettings _settings;

    public ConfigurableLoggingDecorator(IDataService dataService, ILogger<ConfigurableLoggingDecorator> logger, LoggingSettings settings)
    {
        _dataService = dataService;
        _logger = logger;
        _settings = settings;
    }

    public async Task<string> GetDataAsync(int id)
    {
        var stopwatch = _settings.LogPerformance ? Stopwatch.StartNew() : null;

        if (_settings.LogRequestResponse)
        {
            _logger.LogInformation("Request: GetData for ID: {Id}", id);
        }

        try
        {
            var result = await _dataService.GetDataAsync(id);

            if (_settings.LogRequestResponse)
            {
                _logger.LogInformation("Response: Data retrieved for ID: {Id}, Length: {Length}", id, result?.Length ?? 0);
            }

            if (_settings.LogPerformance && stopwatch != null)
            {
                _logger.LogInformation("Performance: GetData({Id}) took {ElapsedMs}ms", id, stopwatch.ElapsedMilliseconds);
            }

            return result;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error retrieving data for ID: {Id}", id);
            throw;
        }
    }
}

// 5. Environment-Specific Decorator (Example: Performance Monitoring)
public class PerformanceMonitoringDecorator : IDataService
{
    private readonly IDataService _dataService;
    private readonly ILogger<PerformanceMonitoringDecorator> _logger;

    public PerformanceMonitoringDecorator(IDataService dataService, ILogger<PerformanceMonitoringDecorator> logger)
    {
        _dataService = dataService;
        _logger = logger;
    }

    public async Task<string> GetDataAsync(int id)
    {
        var stopwatch = Stopwatch.StartNew();
        var memoryBefore = GC.GetTotalMemory(false);

        try
        {
            var result = await _dataService.GetDataAsync(id);
            
            stopwatch.Stop();
            var memoryAfter = GC.GetTotalMemory(false);
            var memoryUsed = memoryAfter - memoryBefore;

            _logger.LogInformation("Performance Metrics - Method: GetData, ID: {Id}, Duration: {Duration}ms, Memory: {Memory} bytes", 
                id, stopwatch.ElapsedMilliseconds, memoryUsed);

            return result;
        }
        catch (Exception ex)
        {
            stopwatch.Stop();
            _logger.LogError(ex, "Performance Metrics - Method: GetData, ID: {Id}, Duration: {Duration}ms, Status: Failed", 
                id, stopwatch.ElapsedMilliseconds);
            throw;
        }
    }
}

// 6. Circuit Breaker Decorator (Production Environment)
public class CircuitBreakerDecorator : IDataService
{
    private readonly IDataService _dataService;
    private readonly ICircuitBreakerPolicy _circuitBreaker;

    public CircuitBreakerDecorator(IDataService dataService, ICircuitBreakerPolicy circuitBreaker)
    {
        _dataService = dataService;
        _circuitBreaker = circuitBreaker;
    }

    public async Task<string> GetDataAsync(int id)
    {
        return await _circuitBreaker.ExecuteAsync(async () => await _dataService.GetDataAsync(id));
    }
}

// 7. Dependency Injection Configuration
public class Startup
{
    public void ConfigureServices(IServiceCollection services, IConfiguration configuration)
    {
        // Register configuration
        var decoratorConfig = new DecoratorConfiguration();
        configuration.GetSection("DecoratorConfig").Bind(decoratorConfig);
        services.AddSingleton(decoratorConfig);

        // Register base service
        services.AddScoped<DatabaseService>();

        // Register decorator factory
        services.AddScoped<IDecoratorFactory, ConfigurableDecoratorFactory>();

        // Register the decorated service
        services.AddScoped<IDataService>(provider =>
        {
            var baseService = provider.GetRequiredService<DatabaseService>();
            var factory = provider.GetRequiredService<IDecoratorFactory>();
            return factory.CreateDecoratedService(baseService);
        });

        // Register caching services based on configuration
        if (decoratorConfig.EnableCaching)
        {
            if (decoratorConfig.Cache.CacheType.Equals("Redis", StringComparison.OrdinalIgnoreCase))
            {
                services.AddStackExchangeRedisCache(options =>
                {
                    options.Configuration = configuration.GetConnectionString("Redis");
                });
            }
            else
            {
                services.AddMemoryCache();
            }
        }

        // Register circuit breaker if enabled
        if (decoratorConfig.EnableCircuitBreaker)
        {
            services.AddScoped<ICircuitBreakerPolicy, CircuitBreakerPolicy>();
        }
    }
}

// 8. Feature Toggle Extension Method
public static class ServiceCollectionExtensions
{
    public static IServiceCollection AddConditionalDecorator<TInterface, TDecorator>(
        this IServiceCollection services,
        Func<DecoratorConfiguration, bool> condition)
        where TInterface : class
        where TDecorator : class, TInterface
    {
        services.AddScoped<TInterface>(provider =>
        {
            var config = provider.GetRequiredService<DecoratorConfiguration>();
            var baseService = provider.GetRequiredService<TInterface>();

            if (condition(config))
            {
                return ActivatorUtilities.CreateInstance<TDecorator>(provider, baseService);
            }

            return baseService;
        });

        return services;
    }
}

// Usage Example:
// services.AddConditionalDecorator<IDataService, CachingDecorator>(config => config.EnableCaching);

// 9. Runtime Configuration Change Support (Advanced)
public class DynamicDecoratorFactory : IDecoratorFactory
{
    private readonly IOptionsMonitor<DecoratorConfiguration> _configMonitor;
    private readonly IServiceProvider _serviceProvider;

    public DynamicDecoratorFactory(IOptionsMonitor<DecoratorConfiguration> configMonitor, IServiceProvider serviceProvider)
    {
        _configMonitor = configMonitor;
        _serviceProvider = serviceProvider;
    }

    public IDataService CreateDecoratedService(IDataService baseService)
    {
        var currentConfig = _configMonitor.CurrentValue;
        // Use currentConfig to create decorators
        // Configuration changes are automatically picked up
        return CreateServiceWithCurrentConfig(baseService, currentConfig);
    }

    private IDataService CreateServiceWithCurrentConfig(IDataService baseService, DecoratorConfiguration config)
    {
        // Implementation similar to ConfigurableDecoratorFactory
        // but uses the current configuration value
        return baseService; // Simplified for brevity
    }
}

// 10. Health Check Integration
public class DecoratorHealthCheck : IHealthCheck
{
    private readonly DecoratorConfiguration _config;

    public DecoratorHealthCheck(DecoratorConfiguration config)
    {
        _config = config;
    }

    public Task<HealthCheckResult> CheckHealthAsync(HealthCheckContext context, CancellationToken cancellationToken = default)
    {
        var enabledDecorators = new List<string>();
        
        if (_config.EnableCaching) enabledDecorators.Add("Caching");
        if (_config.EnableLogging) enabledDecorators.Add("Logging");
        if (_config.EnableValidation) enabledDecorators.Add("Validation");
        if (_config.EnablePerformanceMonitoring) enabledDecorators.Add("PerformanceMonitoring");
        if (_config.EnableCircuitBreaker) enabledDecorators.Add("CircuitBreaker");

        var data = new Dictionary<string, object>
        {
            ["EnabledDecorators"] = enabledDecorators,
            ["CacheType"] = _config.Cache.CacheType,
            ["LogLevel"] = _config.Logging.LogLevel
        };

        return Task.FromResult(HealthCheckResult.Healthy("Decorators configured successfully", data));
    }
}
```

This comprehensive guide covers all aspects of the Decorator Pattern from an enterprise architecture perspective, providing you with interview-ready knowledge and practical implementation examples for .NET environments.

---

### **Behavioral Patterns**
8. **Strategy Pattern**
   - How would you implement different payment gateways using Strategy pattern?
   - Show strategy selection based on runtime conditions.

---


---

9. **Command Pattern**
   - How do you implement CQRS using Command pattern in Web API?
   - Show command validation and error handling.

10. **Observer Pattern**
    - How would you implement event-driven architecture using Observer?
    - Show domain events implementation in .NET Core.

## **DESIGN PRINCIPLES QUESTIONS**

### **SOLID Principles**
11. **Single Responsibility Principle (SRP)**
    - Show a controller that violates SRP and how to fix it.
    - How do you ensure each class has only one reason to change?

12. **Open/Closed Principle (OCP)**
    - How do you design extensible API endpoints without modifying existing code?
    - Show plugin architecture implementation.

13. **Liskov Substitution Principle (LSP)**
    - Give example of LSP violation in inheritance hierarchies.
    - How do you design proper abstractions that follow LSP?

14. **Interface Segregation Principle (ISP)**
    - How do you avoid fat interfaces in service design?
    - Show splitting large interfaces into focused ones.

15. **Dependency Inversion Principle (DIP)**
    - How does .NET Core DI implement DIP?
    - Show high-level modules depending on abstractions, not concretions.

### **Other Design Principles**
16. **DRY (Don't Repeat Yourself)**
    - How do you eliminate code duplication in Web API controllers?
    - Show base controller implementation and attribute usage.

17. **KISS (Keep It Simple, Stupid)**
    - How do you balance simplicity with flexibility in API design?
    - When is over-engineering harmful?

18. **YAGNI (You Aren't Gonna Need It)**
    - How do you avoid premature optimization in Web API design?
    - Show iterative development approach.

## **ARCHITECTURAL DESIGN QUESTIONS**

### **Layered Architecture**
19. **N-Tier Architecture**
    - How do you structure a Web API project with proper separation of concerns?
    - Show Presentation → Business → Data layer implementation.
    - How do you handle cross-cutting concerns across layers?

20. **Clean Architecture**
    - How do you implement Clean Architecture in .NET Core Web API?
    - Show dependency flow and folder structure.
    - How do you keep business logic independent of frameworks?

### **API Design Patterns**
21. **MVC Pattern**
    - How does Web API implement MVC pattern differently from traditional MVC?
    - Show controller, model, and view separation in API context.

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

