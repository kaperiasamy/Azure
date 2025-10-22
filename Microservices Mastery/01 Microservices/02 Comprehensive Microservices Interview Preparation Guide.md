# Comprehensive Microservices Interview Preparation Guide

## Part 1: Top 50 Microservices Interview Questions with Answers

### **Foundation & Architecture (Questions 1-10)**

---

#### **1. What are Microservices and how do they differ from Monolithic architecture?**

**Answer:** Microservices are an architectural style where applications are built as a collection of small, autonomous services, each running in its own process and communicating via well-defined APIs.

**Key Differences:**
- **Deployment**: Independent vs. Single unit
- **Scalability**: Service-level vs. Application-level
- **Technology Stack**: Polyglot vs. Uniform
- **Fault Isolation**: Isolated failures vs. System-wide impact
- **Development**: Parallel teams vs. Sequential coordination

**Follow-up:** "How do you decide when to use Microservices over Monolithic?"
- Consider team size, project complexity, scalability requirements, and organizational maturity.

---

#### **2. What are the key principles of Microservices architecture?**

**Answer:** 
1. **Single Responsibility**: Each service handles one business capability
2. **Autonomous**: Services are independently deployable
3. **Decentralized**: Distributed data management and governance
4. **Failure Isolation**: Failures don't cascade
5. **Observable**: Comprehensive monitoring and logging
6. **Domain-Driven**: Organized around business domains

**C# Example - Service Interface:**
```csharp
public interface IOrderService
{
    Task<Order> CreateOrderAsync(OrderDto orderDto);
    Task<Order> GetOrderAsync(Guid orderId);
    Task UpdateOrderStatusAsync(Guid orderId, OrderStatus status);
}

public class OrderService : IOrderService
{
    private readonly IOrderRepository _repository;
    private readonly IEventBus _eventBus;
    
    public async Task<Order> CreateOrderAsync(OrderDto orderDto)
    {
        var order = new Order(orderDto);
        await _repository.SaveAsync(order);
        await _eventBus.PublishAsync(new OrderCreatedEvent(order));
        return order;
    }
}
```

**Follow-up:** "How do you ensure service autonomy?"

---

#### **3. Explain the concept of Bounded Context in Microservices.**

**Answer:** Bounded Context is a DDD concept defining explicit boundaries within which a domain model exists. Each microservice typically represents one bounded context, ensuring:
- Clear ownership and responsibility
- Independent evolution
- Reduced coupling
- Consistent internal model

**Example Contexts:**
- Order Management Service
- Inventory Service
- Payment Service
- Customer Service

**Follow-up:** "How do you identify bounded contexts in an existing monolith?"

---

#### **4. What are the advantages and disadvantages of Microservices?**

**Answer:**

**Advantages:**
- Independent deployment and scaling
- Technology diversity
- Fault isolation
- Team autonomy
- Better alignment with business

**Disadvantages:**
- Distributed system complexity
- Network latency
- Data consistency challenges
- Operational overhead
- Testing complexity

**Follow-up:** "How do you mitigate the disadvantages?"

---

#### **5. Explain the Database per Service pattern.**

**Answer:** Each microservice owns its database, ensuring:
- Service autonomy
- Independent schema evolution
- Technology flexibility
- Performance isolation

**C# Implementation Example:**
```csharp
// OrderService with its own DbContext
public class OrderDbContext : DbContext
{
    public DbSet<Order> Orders { get; set; }
    public DbSet<OrderItem> OrderItems { get; set; }
    
    protected override void OnConfiguring(DbContextOptionsBuilder options)
        => options.UseSqlServer("Server=orderdb;Database=Orders;");
}

// ProductService with different database
public class ProductDbContext : DbContext
{
    public DbSet<Product> Products { get; set; }
    
    protected override void OnConfiguring(DbContextOptionsBuilder options)
        => options.UsePostgreSql("Host=productdb;Database=Products;");
}
```

**Follow-up:** "How do you handle transactions across services?"

---

### **Communication Patterns (Questions 6-15)**

---

#### **6. What are the different types of communication in Microservices?**

**Answer:**

**Synchronous:**
- REST APIs
- gRPC
- GraphQL

**Asynchronous:**
- Message Queues (RabbitMQ, Azure Service Bus)
- Event Streaming (Kafka, Event Hubs)
- Pub/Sub patterns

**C# REST API Example:**
```csharp
[ApiController]
[Route("api/[controller]")]
public class OrderController : ControllerBase
{
    private readonly IOrderService _orderService;
    
    [HttpPost]
    public async Task<IActionResult> CreateOrder(OrderDto order)
    {
        var result = await _orderService.CreateOrderAsync(order);
        return CreatedAtAction(nameof(GetOrder), 
            new { id = result.Id }, result);
    }
    
    [HttpGet("{id}")]
    public async Task<IActionResult> GetOrder(Guid id)
    {
        var order = await _orderService.GetOrderAsync(id);
        return order != null ? Ok(order) : NotFound();
    }
}
```

**Follow-up:** "When would you choose async over sync communication?"

---

#### **7. Explain the API Gateway pattern.**

**Answer:** API Gateway acts as a single entry point for all client requests, providing:
- Request routing
- Authentication/Authorization
- Rate limiting
- Request/Response transformation
- Load balancing
- Caching

**C# Ocelot Configuration Example:**
```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddOcelot()
            .AddCacheManager()
            .AddPolly();
        services.AddAuthentication("Bearer")
            .AddJwtBearer("Bearer", options => {
                options.Authority = "https://identity-server";
                options.RequireHttpsMetadata = false;
            });
    }
    
    public void Configure(IApplicationBuilder app)
    {
        app.UseAuthentication();
        app.UseOcelot().Wait();
    }
}
```

**Follow-up:** "What are the alternatives to API Gateway?"

---

#### **8. What is Service Discovery and how is it implemented?**

**Answer:** Service Discovery enables services to find and communicate with each other without hard-coding hostnames or ports.

**Types:**
- **Client-side**: Client queries registry (Consul, Eureka)
- **Server-side**: Load balancer queries registry

**C# Consul Integration:**
```csharp
public class ServiceDiscovery
{
    private readonly IConsulClient _consulClient;
    
    public async Task<string> GetServiceUrlAsync(string serviceName)
    {
        var services = await _consulClient.Health.Service(serviceName);
        var service = services.Response?.FirstOrDefault();
        
        if (service != null)
        {
            return $"http://{service.Service.Address}:{service.Service.Port}";
        }
        throw new ServiceNotFoundException(serviceName);
    }
}
```

**Follow-up:** "How do you handle service registration and deregistration?"

---

#### **9. Explain the Circuit Breaker pattern.**

**Answer:** Circuit Breaker prevents cascading failures by monitoring for failures and temporarily blocking requests to failing services.

**States:**
- **Closed**: Normal operation
- **Open**: Requests fail immediately
- **Half-Open**: Limited requests to test recovery

**C# Polly Implementation:**
```csharp
public class ResilientHttpClient
{
    private readonly HttpClient _httpClient;
    private readonly IAsyncPolicy<HttpResponseMessage> _retryPolicy;
    
    public ResilientHttpClient()
    {
        _retryPolicy = Policy
            .HandleResult<HttpResponseMessage>(r => !r.IsSuccessStatusCode)
            .CircuitBreakerAsync(
                handledEventsAllowedBeforeBreaking: 3,
                durationOfBreak: TimeSpan.FromSeconds(30),
                onBreak: (result, duration) => Log.Warning("Circuit opened"),
                onReset: () => Log.Information("Circuit closed"));
    }
    
    public async Task<T> GetAsync<T>(string url)
    {
        var response = await _retryPolicy.ExecuteAsync(
            () => _httpClient.GetAsync(url));
        return await response.Content.ReadAsAsync<T>();
    }
}
```

**Follow-up:** "What other resilience patterns work with Circuit Breaker?"

---

#### **10. What is the Saga pattern and when do you use it?**

**Answer:** Saga manages distributed transactions across multiple services through a sequence of local transactions, with compensating transactions for rollback.

**Types:**
- **Choreography**: Services communicate via events
- **Orchestration**: Central coordinator manages the flow

**C# Orchestration Example:**
```csharp
public class OrderSaga
{
    public async Task<bool> ProcessOrderAsync(Order order)
    {
        try
        {
            await _inventoryService.ReserveItemsAsync(order.Items);
            await _paymentService.ProcessPaymentAsync(order.Payment);
            await _shippingService.CreateShipmentAsync(order);
            await _orderService.ConfirmOrderAsync(order.Id);
            return true;
        }
        catch (Exception ex)
        {
            // Compensating transactions
            await _inventoryService.ReleaseItemsAsync(order.Items);
            await _paymentService.RefundPaymentAsync(order.Payment);
            return false;
        }
    }
}
```

**Follow-up:** "How do you handle partial failures in Saga?"

---

### **Data Management (Questions 11-20)**

---

#### **11. How do you handle distributed transactions in Microservices?**

**Answer:** Avoid distributed transactions when possible. Use:
1. **Saga Pattern**: Sequential local transactions
2. **Event Sourcing**: Store events, not state
3. **CQRS**: Separate read/write models
4. **Eventual Consistency**: Accept temporary inconsistency

**C# Event Sourcing Example:**
```csharp
public class OrderAggregate
{
    private List<IEvent> _events = new List<IEvent>();
    public Guid Id { get; private set; }
    public OrderStatus Status { get; private set; }
    
    public void CreateOrder(OrderDetails details)
    {
        var @event = new OrderCreatedEvent(Guid.NewGuid(), details);
        Apply(@event);
        _events.Add(@event);
    }
    
    private void Apply(OrderCreatedEvent @event)
    {
        Id = @event.OrderId;
        Status = OrderStatus.Created;
    }
}
```

**Follow-up:** "What are the challenges with eventual consistency?"

---

#### **12. Explain CQRS pattern in Microservices context.**

**Answer:** CQRS (Command Query Responsibility Segregation) separates read and write operations, allowing:
- Independent scaling
- Optimized data models
- Different consistency requirements
- Performance optimization

**C# Implementation:**
```csharp
// Command side
public interface ICommandHandler<TCommand>
{
    Task HandleAsync(TCommand command);
}

public class CreateOrderCommandHandler : ICommandHandler<CreateOrderCommand>
{
    public async Task HandleAsync(CreateOrderCommand command)
    {
        var order = new Order(command);
        await _repository.SaveAsync(order);
        await _eventBus.PublishAsync(new OrderCreatedEvent(order));
    }
}

// Query side
public interface IQueryHandler<TQuery, TResult>
{
    Task<TResult> HandleAsync(TQuery query);
}

public class GetOrderQueryHandler : IQueryHandler<GetOrderQuery, OrderDto>
{
    public async Task<OrderDto> HandleAsync(GetOrderQuery query)
    {
        return await _readRepository.GetOrderAsync(query.OrderId);
    }
}
```

**Follow-up:** "How do you synchronize command and query models?"

---

#### **13. What is Event Sourcing and how does it work with Microservices?**

**Answer:** Event Sourcing stores all changes as a sequence of events rather than current state, providing:
- Complete audit trail
- Temporal queries
- Event replay capability
- Natural integration with CQRS

**C# Event Store Example:**
```csharp
public class EventStore
{
    private readonly IEventRepository _repository;
    
    public async Task SaveEventsAsync(Guid aggregateId, 
        IEnumerable<IEvent> events, int expectedVersion)
    {
        var eventList = events.ToList();
        foreach (var @event in eventList)
        {
            var eventData = new EventData
            {
                AggregateId = aggregateId,
                EventType = @event.GetType().Name,
                Data = JsonSerializer.Serialize(@event),
                Version = ++expectedVersion,
                Timestamp = DateTime.UtcNow
            };
            await _repository.SaveAsync(eventData);
        }
    }
}
```

**Follow-up:** "How do you handle event schema evolution?"

---

#### **14. How do you manage data consistency across Microservices?**

**Answer:** Strategies include:
1. **Eventual Consistency**: Accept temporary inconsistencies
2. **Distributed Sagas**: Coordinate transactions
3. **Event-Driven Architecture**: Publish domain events
4. **Compensation**: Undo operations on failure
5. **Idempotency**: Ensure operations can be safely retried

**C# Idempotent Operation:**
```csharp
public class IdempotentOrderService
{
    private readonly IIdempotencyStore _idempotencyStore;
    
    public async Task<Order> CreateOrderAsync(string idempotencyKey, 
        OrderDto orderDto)
    {
        var existingResult = await _idempotencyStore
            .GetResultAsync<Order>(idempotencyKey);
        if (existingResult != null)
            return existingResult;
            
        var order = await ProcessOrderAsync(orderDto);
        await _idempotencyStore.StoreResultAsync(idempotencyKey, order);
        return order;
    }
}
```

**Follow-up:** "How long should idempotency keys be stored?"

---

#### **15. Explain the Outbox pattern.**

**Answer:** The Outbox pattern ensures reliable message publishing by storing events in the same database transaction as the business operation, then publishing them separately.

**C# Implementation:**
```csharp
public class OrderServiceWithOutbox
{
    public async Task CreateOrderAsync(OrderDto orderDto)
    {
        using var transaction = await _dbContext.Database
            .BeginTransactionAsync();
        try
        {
            // Save order
            var order = new Order(orderDto);
            _dbContext.Orders.Add(order);
            
            // Save event to outbox
            var outboxEvent = new OutboxEvent
            {
                EventType = "OrderCreated",
                Payload = JsonSerializer.Serialize(order),
                CreatedAt = DateTime.UtcNow
            };
            _dbContext.OutboxEvents.Add(outboxEvent);
            
            await _dbContext.SaveChangesAsync();
            await transaction.CommitAsync();
        }
        catch
        {
            await transaction.RollbackAsync();
            throw;
        }
    }
}
```

**Follow-up:** "How do you process outbox events?"

---

### **Security & Authentication (Questions 16-25)**

---

#### **16. How do you implement authentication in Microservices?**

**Answer:** Common approaches:
1. **Centralized Identity Server**: OAuth 2.0/OpenID Connect
2. **JWT Tokens**: Stateless authentication
3. **API Gateway Authentication**: Single authentication point
4. **Service-to-Service**: mTLS or API keys

**C# JWT Implementation:**
```csharp
public class JwtService
{
    private readonly string _secret;
    
    public string GenerateToken(User user)
    {
        var tokenHandler = new JwtSecurityTokenHandler();
        var key = Encoding.ASCII.GetBytes(_secret);
        var tokenDescriptor = new SecurityTokenDescriptor
        {
            Subject = new ClaimsIdentity(new[]
            {
                new Claim(ClaimTypes.Name, user.Username),
                new Claim(ClaimTypes.Role, user.Role)
            }),
            Expires = DateTime.UtcNow.AddHours(1),
            SigningCredentials = new SigningCredentials(
                new SymmetricSecurityKey(key),
                SecurityAlgorithms.HmacSha256Signature)
        };
        var token = tokenHandler.CreateToken(tokenDescriptor);
        return tokenHandler.WriteToken(token);
    }
}
```

**Follow-up:** "How do you handle token refresh?"

---

#### **17. What is Service Mesh and what problems does it solve?**

**Answer:** Service Mesh is a dedicated infrastructure layer that handles service-to-service communication, providing:
- Traffic management
- Security (mTLS)
- Observability
- Resilience patterns
- Policy enforcement

**Popular Service Meshes:**
- Istio
- Linkerd
- Consul Connect

**Configuration Example:**
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: order-service
spec:
  hosts:
  - order-service
  http:
  - match:
    - headers:
        canary:
          exact: "true"
    route:
    - destination:
        host: order-service
        subset: v2
      weight: 100
  - route:
    - destination:
        host: order-service
        subset: v1
```

**Follow-up:** "When would you not use a service mesh?"

---

#### **18. How do you handle authorization across Microservices?**

**Answer:** Approaches include:
1. **Token-based**: Include permissions in JWT
2. **Policy-based**: Centralized policy engine (OPA)
3. **Service-level**: Each service manages its authorization
4. **API Gateway**: Centralized authorization checks

**C# Policy-based Authorization:**
```csharp
public class OrderAuthorizationHandler : 
    AuthorizationHandler<OrderOperationRequirement, Order>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        OrderOperationRequirement requirement,
        Order resource)
    {
        var userId = context.User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
        
        if (requirement.Name == "Delete" && 
            context.User.IsInRole("Admin"))
        {
            context.Succeed(requirement);
        }
        else if (resource.UserId == userId)
        {
            context.Succeed(requirement);
        }
        
        return Task.CompletedTask;
    }
}
```

**Follow-up:** "How do you audit authorization decisions?"

---

#### **19. Explain Zero Trust Security in Microservices.**

**Answer:** Zero Trust assumes no implicit trust and requires:
- Continuous verification
- Least privilege access
- Micro-segmentation
- Encrypted communication (mTLS)
- Strong authentication
- Regular security audits

**C# mTLS Configuration:**
```csharp
public class SecureHttpClientFactory
{
    public HttpClient CreateSecureClient(string certPath, string keyPath)
    {
        var handler = new HttpClientHandler();
        var certificate = new X509Certificate2(certPath, keyPath);
        handler.ClientCertificates.Add(certificate);
        handler.ServerCertificateCustomValidationCallback = 
            (sender, cert, chain, errors) =>
            {
                // Custom validation logic
                return chain.ChainStatus.Length == 0;
            };
        
        return new HttpClient(handler);
    }
}
```

**Follow-up:** "How do you rotate certificates in production?"

---

#### **20. How do you secure sensitive data in Microservices?**

**Answer:** Security measures:
1. **Encryption at rest**: Database encryption
2. **Encryption in transit**: TLS/mTLS
3. **Secret management**: Azure Key Vault, HashiCorp Vault
4. **Data masking**: Hide sensitive data in logs
5. **Tokenization**: Replace sensitive data with tokens

**C# Azure Key Vault Integration:**
```csharp
public class SecretManager
{
    private readonly SecretClient _secretClient;
    
    public SecretManager(string keyVaultUrl)
    {
        _secretClient = new SecretClient(
            new Uri(keyVaultUrl),
            new DefaultAzureCredential());
    }
    
    public async Task<string> GetSecretAsync(string secretName)
    {
        var secret = await _secretClient.GetSecretAsync(secretName);
        return secret.Value.Value;
    }
}
```

**Follow-up:** "How do you handle secret rotation?"

---

### **Deployment & DevOps (Questions 21-30)**

---

#### **21. Explain Container Orchestration in Microservices.**

**Answer:** Container orchestration automates deployment, scaling, and management of containerized applications. Key features:
- Service discovery
- Load balancing
- Auto-scaling
- Self-healing
- Rolling updates
- Secret management

**Kubernetes Deployment Example:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: order-service
  template:
    metadata:
      labels:
        app: order-service
    spec:
      containers:
      - name: order-service
        image: myregistry/order-service:v1.0
        ports:
        - containerPort: 80
        env:
        - name: ConnectionString
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: connection-string
```

**Follow-up:** "How do you handle stateful services in Kubernetes?"

---

#### **22. What is Blue-Green Deployment?**

**Answer:** Blue-Green deployment maintains two identical production environments, allowing zero-downtime deployments by switching traffic between them.

**C# Health Check for Blue-Green:**
```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddHealthChecks()
            .AddCheck("database", () =>
            {
                using var conn = new SqlConnection(connectionString);
                conn.Open();
                return HealthCheckResult.Healthy();
            })
            .AddCheck("dependencies", async () =>
            {
                var client = new HttpClient();
                var response = await client.GetAsync("http://dependency/health");
                return response.IsSuccessStatusCode 
                    ? HealthCheckResult.Healthy() 
                    : HealthCheckResult.Unhealthy();
            });
    }
}
```

**Follow-up:** "How do you handle database migrations in Blue-Green?"

---

#### **23. Explain Canary Deployment strategy.**

**Answer:** Canary deployment gradually rolls out changes to a small subset of users before full deployment, allowing:
- Risk mitigation
- Performance validation
- User feedback
- Quick rollback

**Implementation Approach:**
```csharp
public class CanaryMiddleware
{
    private readonly RequestDelegate _next;
    private readonly int _canaryPercentage;
    
    public async Task InvokeAsync(HttpContext context)
    {
        var random = new Random();
        var isCanary = random.Next(100) < _canaryPercentage;
        
        if (isCanary)
        {
            context.Request.Headers.Add("X-Canary", "true");
            // Route to canary version
        }
        
        await _next(context);
    }
}
```

**Follow-up:** "How do you monitor canary deployments?"

---

#### **24. How do you implement CI/CD for Microservices?**

**Answer:** CI/CD pipeline components:
1. **Source Control**: Git with branching strategy
2. **Build**: Compile and unit tests
3. **Containerization**: Docker image creation
4. **Integration Tests**: Service integration tests
5. **Security Scanning**: Vulnerability checks
6. **Deployment**: Automated deployment to environments
7. **Smoke Tests**: Post-deployment validation

**Azure DevOps Pipeline Example:**
```yaml
trigger:
  branches:
    include:
    - main

stages:
- stage: Build
  jobs:
  - job: BuildJob
    steps:
    - task: DotNetCoreCLI@2
      inputs:
        command: 'build'
    - task: DotNetCoreCLI@2
      inputs:
        command: 'test'
    - task: Docker@2
      inputs:
        command: 'buildAndPush'
        repository: 'order-service'
        tags: |
          $(Build.BuildId)
          latest

- stage: Deploy
  jobs:
  - deployment: DeployToAKS
    environment: 'production'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: KubernetesManifest@0
            inputs:
              action: 'deploy'
              manifests: 'k8s/*.yaml'
```

**Follow-up:** "How do you handle database migrations in CI/CD?"

---

#### **25. What is GitOps and how does it apply to Microservices?**

**Answer:** GitOps uses Git as the single source of truth for declarative infrastructure and applications, enabling:
- Version control for infrastructure
- Automated deployments
- Easy rollbacks
- Audit trail
- Consistency across environments

**ArgoCD Application Example:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: order-service
spec:
  source:
    repoURL: https://github.com/company/microservices
    targetRevision: HEAD
    path: order-service/k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

**Follow-up:** "How do you handle secrets in GitOps?"

---

### **Monitoring & Observability (Questions 26-35)**

---

#### **26. What are the three pillars of Observability?**

**Answer:**
1. **Metrics**: Quantitative measurements (CPU, memory, request rate)
2. **Logs**: Detailed event records
3. **Traces**: Request flow across services

**C# OpenTelemetry Implementation:**
```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddOpenTelemetryTracing(builder =>
        {
            builder
                .SetResourceBuilder(ResourceBuilder.CreateDefault()
                    .AddService("OrderService"))
                .AddAspNetCoreInstrumentation()
                .AddHttpClientInstrumentation()
                .AddSqlClientInstrumentation()
                .AddJaegerExporter(options =>
                {
                    options.AgentHost = "jaeger";
                    options.AgentPort = 6831;
                });
        });
    }
}
```

**Follow-up:** "How do you correlate logs, metrics, and traces?"

---

#### **27. Explain Distributed Tracing in Microservices.**

**Answer:** Distributed tracing tracks requests across multiple services, providing:
- End-to-end latency analysis
- Service dependency mapping
- Performance bottleneck identification
- Error propagation tracking

**C# Custom Trace Context:**
```csharp
public class TracingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<TracingMiddleware> _logger;
    
    public async Task InvokeAsync(HttpContext context)
    {
        var traceId = context.Request.Headers["X-Trace-Id"].FirstOrDefault() 
            ?? Guid.NewGuid().ToString();
        context.Items["TraceId"] = traceId;
        
        using (_logger.BeginScope(new { TraceId = traceId }))
        {
            _logger.LogInformation("Request started");
            await _next(context);
            _logger.LogInformation("Request completed");
        }
    }
}
```

**Follow-up:** "What tools do you use for distributed tracing?"

---

#### **28. How do you implement centralized logging?**

**Answer:** Centralized logging aggregates logs from all services for:
- Unified view
- Correlation analysis
- Search and filtering
- Alerting
- Compliance

**C# Serilog with Elasticsearch:**
```csharp
public class Program
{
    public static void Main(string[] args)
    {
        Log.Logger = new LoggerConfiguration()
            .Enrich.FromLogContext()
            .Enrich.WithProperty("Service", "OrderService")
            .Enrich.WithMachineName()
            .WriteTo.Elasticsearch(new ElasticsearchSinkOptions(
                new Uri("http://elasticsearch:9200"))
            {
                IndexFormat = "microservices-{0:yyyy.MM.dd}",
                AutoRegisterTemplate = true
            })
            .CreateLogger();
            
        CreateHostBuilder(args).Build().Run();
    }
}
```

**Follow-up:** "How do you handle log retention and rotation?"

---

#### **29. What metrics should you monitor in Microservices?**

**Answer:** Key metrics categories:

**Business Metrics:**
- Transaction volume
- Revenue impact
- User engagement

**Application Metrics:**
- Request rate
- Error rate
- Response time (P50, P95, P99)

**Infrastructure Metrics:**
- CPU/Memory usage
- Network I/O
- Disk usage

**C# Custom Metrics with Prometheus:**
```csharp
public class MetricsService
{
    private readonly Counter _requestCounter;
    private readonly Histogram _requestDuration;
    
    public MetricsService()
    {
        _requestCounter = Metrics.CreateCounter(
            "order_requests_total", 
            "Total number of order requests");
            
        _requestDuration = Metrics.CreateHistogram(
            "order_request_duration_seconds",
            "Order request duration");
    }
    
    public async Task<T> TrackMetrics<T>(Func<Task<T>> operation)
    {
        _requestCounter.Inc();
        using (_requestDuration.NewTimer())
        {
            return await operation();
        }
    }
}
```

**Follow-up:** "How do you set up alerting based on metrics?"

---

#### **30. Explain the concept of SLI, SLO, and SLA.**

**Answer:**
- **SLI (Service Level Indicator)**: Quantitative measure of service behavior
- **SLO (Service Level Objective)**: Target value for SLI
- **SLA (Service Level Agreement)**: Contract with consequences

**Example:**
- SLI: Request success rate
- SLO: 99.9% success rate
- SLA: 99.5% uptime with penalties

**C# SLO Monitoring:**
```csharp
public class SloMonitor
{
    private readonly IMetricsCollector _metrics;
    private const double SLO_TARGET = 0.999;
    
    public async Task<bool> CheckSlo(DateTime startTime, DateTime endTime)
    {
        var totalRequests = await _metrics.GetTotalRequests(startTime, endTime);
        var successfulRequests = await _metrics.GetSuccessfulRequests(
            startTime, endTime);
        
        var successRate = (double)successfulRequests / totalRequests;
        var sloMet = successRate >= SLO_TARGET;
        
        if (!sloMet)
        {
            await _alerting.SendAlert($"SLO breach: {successRate:P2}");
        }
        
        return sloMet;
    }
}
```

**Follow-up:** "How do you calculate error budgets?"

---

### **Testing Strategies (Questions 31-40)**

---

#### **31. What is Contract Testing in Microservices?**

**Answer:** Contract testing verifies that services can communicate correctly by testing the contract (API specification) between consumer and provider independently.

**C# Pact Consumer Test:**
```csharp
[Test]
public async Task GetOrder_ReturnsCorrectOrder()
{
    var pact = new PactBuilder()
        .ServiceConsumer("OrderUI")
        .HasPactWith("OrderService");
        
    pact.UponReceiving("a request for order")
        .Given("order exists")
        .WithRequest(HttpMethod.Get, "/api/orders/123")
        .WillRespond()
        .WithStatus(200)
        .WithJsonBody(new { id = "123", status = "Confirmed" });
        
    await pact.VerifyAsync(async () =>
    {
        var client = new OrderServiceClient(pact.MockServerUri);
        var order = await client.GetOrderAsync("123");
        Assert.AreEqual("Confirmed", order.Status);
    });
}
```

**Follow-up:** "How do you version API contracts?"

---

#### **32. How do you perform Integration Testing in Microservices?**

**Answer:** Integration testing strategies:
1. **Service Virtualization**: Mock external dependencies
2. **TestContainers**: Spin up real services in containers
3. **In-memory implementations**: For databases/message brokers
4. **Sandbox environments**: Dedicated test environments

**C# TestContainers Example:**
```csharp
public class OrderServiceIntegrationTests : IAsyncLifetime
{
    private readonly TestcontainersContainer _postgres;
    private readonly TestcontainersContainer _redis;
    
    public OrderServiceIntegrationTests()
    {
        _postgres = new TestcontainersBuilder<TestcontainersContainer>()
            .WithImage("postgres:13")
            .WithEnvironment("POSTGRES_PASSWORD", "test")
            .Build();
            
        _redis = new TestcontainersBuilder<TestcontainersContainer>()
            .WithImage("redis:6")
            .Build();
    }
    
    [Fact]
    public async Task CreateOrder_StoresInDatabase()
    {
        var factory = new WebApplicationFactory<Startup>()
            .WithWebHostBuilder(builder =>
            {
                builder.ConfigureServices(services =>
                {
                    services.AddDbContext<OrderContext>(options =>
                        options.UseNpgsql(_postgres.ConnectionString));
                });
            });
            
        var client = factory.CreateClient();
        var response = await client.PostAsJsonAsync("/api/orders", 
            new OrderDto());
        Assert.Equal(HttpStatusCode.Created, response.StatusCode);
    }
}
```

**Follow-up:** "How do you handle test data management?"

---

#### **33. What is Chaos Engineering and how do you implement it?**

**Answer:** Chaos Engineering intentionally injects failures to test system resilience:
- Network latency
- Service failures
- Resource exhaustion
- Data corruption

**C# Chaos Middleware:**
```csharp
public class ChaosMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ChaosSettings _settings;
    private readonly Random _random = new();
    
    public async Task InvokeAsync(HttpContext context)
    {
        if (_settings.Enabled && _random.NextDouble() < _settings.FailureRate)
        {
            var chaosType = _settings.ChaosTypes[
                _random.Next(_settings.ChaosTypes.Count)];
                
            switch (chaosType)
            {
                case "latency":
                    await Task.Delay(_settings.LatencyMs);
                    break;
                case "error":
                    context.Response.StatusCode = 500;
                    return;
                case "timeout":
                    await Task.Delay(TimeSpan.FromMinutes(5));
                    break;
            }
        }
        
        await _next(context);
    }
}
```

**Follow-up:** "How do you ensure chaos experiments are safe?"

---

#### **34. Explain the Testing Pyramid for Microservices.**

**Answer:** The testing pyramid for microservices includes:

1. **Unit Tests** (Base - Most): Fast, isolated, numerous
2. **Integration Tests**: Service integration, database tests
3. **Component Tests**: Single service with mocked dependencies
4. **Contract Tests**: API contracts between services
5. **End-to-End Tests** (Top - Least): Full system flow

**Test Distribution Example:**
```csharp
// Unit Test (70%)
[Test]
public void CalculateOrderTotal_AppliesDiscount()
{
    var order = new Order();
    order.AddItem(new Item { Price = 100 });
    order.ApplyDiscount(10);
    Assert.AreEqual(90, order.Total);
}

// Integration Test (20%)
[Test]
public async Task SaveOrder_PersistsToDatabase()
{
    var order = new Order();
    await _repository.SaveAsync(order);
    var saved = await _repository.GetAsync(order.Id);
    Assert.NotNull(saved);
}

// E2E Test (10%)
[Test]
public async Task CompleteOrderFlow_WorksEndToEnd()
{
    var orderId = await CreateOrder();
    await ProcessPayment(orderId);
    await ShipOrder(orderId);
    var status = await GetOrderStatus(orderId);
    Assert.AreEqual("Shipped", status);
}
```

**Follow-up:** "How do you balance test coverage and execution time?"

---

#### **35. How do you implement Service Virtualization?**

**Answer:** Service virtualization creates simulated services for testing when real services are:
- Unavailable
- Expensive to use
- Difficult to configure
- Not yet built

**C# WireMock Example:**
```csharp
public class PaymentServiceVirtualization
{
    private WireMockServer _server;
    
    public void Start()
    {
        _server = WireMockServer.Start();
        
        _server
            .Given(Request.Create()
                .WithPath("/api/payment/process")
                .UsingPost())
            .RespondWith(Response.Create()
                .WithStatusCode(200)
                .WithBodyAsJson(new { 
                    transactionId = Guid.NewGuid(),
                    status = "Approved" 
                })
                .WithDelay(TimeSpan.FromMilliseconds(100)));
    }
    
    public string ServiceUrl => _server.Urls[0];
}
```

**Follow-up:** "When would you use virtualization vs real services?"

---

### **Performance & Scalability (Questions 36-45)**

---

#### **36. How do you handle Rate Limiting in Microservices?**

**Answer:** Rate limiting strategies:
1. **Token Bucket**: Fixed rate with burst capacity
2. **Sliding Window**: Requests per time window
3. **Concurrent Requests**: Limit parallel requests
4. **API Gateway**: Centralized rate limiting

**C# Rate Limiting Implementation:**
```csharp
public class RateLimitingMiddleware
{
    private readonly IMemoryCache _cache;
    private readonly int _limit = 100;
    private readonly TimeSpan _window = TimeSpan.FromMinutes(1);
    
    public async Task InvokeAsync(HttpContext context, RequestDelegate next)
    {
        var key = $"rate_limit_{context.User.Identity.Name}";
        var requestCount = _cache.Get<int>(key);
        
        if (requestCount >= _limit)
        {
            context.Response.StatusCode = 429; // Too Many Requests
            await context.Response.WriteAsync("Rate limit exceeded");
            return;
        }
        
        _cache.Set(key, requestCount + 1, _window);
        await next(context);
    }
}
```

**Follow-up:** "How do you implement distributed rate limiting?"

---

#### **37. Explain different caching strategies in Microservices.**

**Answer:** Caching strategies:

1. **Cache-Aside**: Application manages cache
2. **Read-Through**: Cache loads data on miss
3. **Write-Through**: Write to cache and database
4. **Write-Behind**: Write to cache, async to database
5. **Distributed Cache**: Shared across services (Redis)

**C# Distributed Cache Example:**
```csharp
public class CachedOrderService
{
    private readonly IDistributedCache _cache;
    private readonly IOrderRepository _repository;
    
    public async Task<Order> GetOrderAsync(Guid orderId)
    {
        var cacheKey = $"order:{orderId}";
        var cached = await _cache.GetStringAsync(cacheKey);
        
        if (!string.IsNullOrEmpty(cached))
            return JsonSerializer.Deserialize<Order>(cached);
        
        var order = await _repository.GetAsync(orderId);
        if (order != null)
        {
            await _cache.SetStringAsync(cacheKey, 
                JsonSerializer.Serialize(order),
                new DistributedCacheEntryOptions
                {
                    SlidingExpiration = TimeSpan.FromMinutes(5)
                });
        }
        
        return order;
    }
}
```

**Follow-up:** "How do you handle cache invalidation?"

---

#### **38. How do you implement Auto-scaling for Microservices?**

**Answer:** Auto-scaling types:

1. **Horizontal Pod Autoscaling (HPA)**: Scale based on metrics
2. **Vertical Pod Autoscaling (VPA)**: Adjust resource limits
3. **Cluster Autoscaling**: Add/remove nodes
4. **Custom Metrics**: Business-specific scaling

**Kubernetes HPA Configuration:**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: order-service-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: order-service
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Pods
    pods:
      metric:
        name: pending_orders
      target:
        type: AverageValue
        averageValue: "30"
```

**Follow-up:** "How do you prevent scaling thrashing?"

---

#### **39. What is Backpressure and how do you handle it?**

**Answer:** Backpressure occurs when a service receives requests faster than it can process them. Handling strategies:

1. **Rate limiting**: Limit incoming requests
2. **Circuit breakers**: Fail fast
3. **Buffering**: Queue requests
4. **Load shedding**: Drop low-priority requests
5. **Reactive streams**: Built-in backpressure

**C# Reactive Extensions Example:**
```csharp
public class BackpressureHandler
{
    public IObservable<ProcessedOrder> ProcessOrders(
        IObservable<Order> orders)
    {
        return orders
            .Buffer(TimeSpan.FromSeconds(1), 100) // Buffer max 100 items/sec
            .Where(buffer => buffer.Count > 0)
            .SelectMany(buffer => 
                buffer.ToObservable()
                    .Select(ProcessOrder)
                    .Merge(maxConcurrent: 5)) // Process max 5 concurrently
            .Retry(3)
            .Catch<ProcessedOrder, Exception>(ex =>
            {
                _logger.LogError(ex, "Order processing failed");
                return Observable.Empty<ProcessedOrder>();
            });
    }
}
```

**Follow-up:** "How do you monitor backpressure?"

---

#### **40. How do you optimize database access in Microservices?**

**Answer:** Optimization strategies:

1. **Connection pooling**: Reuse connections
2. **Read replicas**: Distribute read load
3. **Caching**: Reduce database hits
4. **Query optimization**: Indexes, query plans
5. **CQRS**: Separate read/write models
6. **Database sharding**: Horizontal partitioning

**C# Connection Pooling Configuration:**
```csharp
public class OptimizedDbContext : DbContext
{
    protected override void OnConfiguring(DbContextOptionsBuilder options)
    {
        options.UseSqlServer(connectionString, sqlOptions =>
        {
            sqlOptions.EnableRetryOnFailure(
                maxRetryCount: 3,
                maxRetryDelay: TimeSpan.FromSeconds(5),
                errorNumbersToAdd: null);
        });
        
        options.UseQueryTrackingBehavior(QueryTrackingBehavior.NoTracking);
    }
    
    public async Task<List<Order>> GetOrdersOptimized(int userId)
    {
        return await Orders
            .AsNoTracking()
            .Include(o => o.Items)
            .Where(o => o.UserId == userId)
            .OrderByDescending(o => o.CreatedDate)
            .Take(10)
            .ToListAsync();
    }
}
```

**Follow-up:** "How do you handle database connection failures?"

---

### **Advanced Patterns & Best Practices (Questions 41-50)**

---

#### **41. Explain the Strangler Fig pattern for Microservices migration.**

**Answer:** The Strangler Fig pattern gradually replaces a monolithic application with microservices by:
1. Creating new functionality as services
2. Gradually redirecting traffic
3. Decommissioning old code
4. Avoiding big-bang rewrites

**C# Implementation with Feature Flags:**
```csharp
public class StranglerFigRouter
{
    private readonly IFeatureManager _featureManager;
    private readonly ILegacyOrderService _legacy;
    private readonly IModernOrderService _modern;
    
    public async Task<Order> GetOrderAsync(Guid orderId)
    {
        var useModernService = await _featureManager
            .IsEnabledAsync("UseModernOrderService");
            
        if (useModernService)
        {
            try
            {
                return await _modern.GetOrderAsync(orderId);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Modern service failed, falling back");
                return await _legacy.GetOrderAsync(orderId);
            }
        }
        
        return await _legacy.GetOrderAsync(orderId);
    }
}
```

**Follow-up:** "How do you ensure data consistency during migration?"

---

#### **42. What is the Bulkhead pattern?**

**Answer:** The Bulkhead pattern isolates resources to prevent failures from cascading:
- Separate thread pools
- Connection pool isolation
- Resource partitioning
- Failure containment

**C# Bulkhead Implementation:**
```csharp
public class BulkheadService
{
    private readonly SemaphoreSlim _criticalOperations;
    private readonly SemaphoreSlim _normalOperations;
    
    public BulkheadService()
    {
        _criticalOperations = new SemaphoreSlim(10, 10); // Max 10 concurrent
        _normalOperations = new SemaphoreSlim(50, 50);   // Max 50 concurrent
    }
    
    public async Task<T> ExecuteCriticalAsync<T>(Func<Task<T>> operation)
    {
        await _criticalOperations.WaitAsync();
        try
        {
            return await operation();
        }
        finally
        {
            _criticalOperations.Release();
        }
    }
}
```

**Follow-up:** "How do you determine bulkhead sizes?"

---

#### **43. Explain the Sidecar pattern.**

**Answer:** The Sidecar pattern deploys helper components alongside the main service container, providing:
- Logging/Monitoring
- Security/Authentication
- Configuration management
- Service mesh proxy
- Circuit breaking

**Kubernetes Sidecar Example:**
```yaml
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: order-service
    image: order-service:v1
    ports:
    - containerPort: 8080
    
  - name: logging-sidecar
    image: fluentd:latest
    volumeMounts:
    - name: logs
      mountPath: /var/log
      
  - name: envoy-proxy
    image: envoyproxy/envoy:latest
    ports:
    - containerPort: 9901
    volumeMounts:
    - name: envoy-config
      mountPath: /etc/envoy
```

**Follow-up:** "What are the drawbacks of the sidecar pattern?"

---

#### **44. How do you implement the Compensation pattern?**

**Answer:** The Compensation pattern provides a way to undo operations when distributed transactions fail:

**C# Compensation Implementation:**
```csharp
public interface ICompensableAction
{
    Task ExecuteAsync();
    Task CompensateAsync();
}

public class OrderSagaOrchestrator
{
    private readonly Stack<ICompensableAction> _executedActions = new();
    
    public async Task<bool> ExecuteSagaAsync(
        List<ICompensableAction> actions)
    {
        foreach (var action in actions)
        {
            try
            {
                await action.ExecuteAsync();
                _executedActions.Push(action);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Action failed, compensating...");
                await CompensateAsync();
                return false;
            }
        }
        return true;
    }
    
    private async Task CompensateAsync()
    {
        while (_executedActions.Count > 0)
        {
            var action = _executedActions.Pop();
            try
            {
                await action.CompensateAsync();
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Compensation failed");
            }
        }
    }
}
```

**Follow-up:** "How do you handle partial compensation failures?"

---

#### **45. What is the Backend for Frontend (BFF) pattern?**

**Answer:** BFF creates separate backend services for different frontend applications, providing:
- Optimized APIs for each client
- Reduced over-fetching
- Client-specific aggregation
- Simplified frontend logic

**C# BFF Implementation:**
```csharp
// Mobile BFF
[ApiController]
[Route("api/mobile/orders")]
public class MobileOrderController : ControllerBase
{
    public async Task<MobileOrderDto> GetOrder(Guid id)
    {
        var order = await _orderService.GetOrderAsync(id);
        var customer = await _customerService.GetCustomerAsync(order.CustomerId);
        
        // Mobile-optimized response
        return new MobileOrderDto
        {
            OrderId = order.Id,
            Status = order.Status,
            CustomerName = customer.Name,
            Total = order.Total
        };
    }
}

// Web BFF with more details
[ApiController]
[Route("api/web/orders")]
public class WebOrderController : ControllerBase
{
    public async Task<WebOrderDto> GetOrder(Guid id)
    {
        var order = await _orderService.GetOrderAsync(id);
        var customer = await _customerService.GetCustomerAsync(order.CustomerId);
        var items = await _inventoryService.GetItemsAsync(order.ItemIds);
        
        return new WebOrderDto
        {
            Order = order,
            Customer = customer,
            Items = items,
            RecommendedProducts = await _recommendationService
                .GetRecommendationsAsync(order)
        };
    }
}
```

**Follow-up:** "How do you avoid code duplication in BFF?"

---

#### **46. Explain Event-Driven Architecture in Microservices.**

**Answer:** Event-Driven Architecture uses events to trigger actions across services:
- Loose coupling
- Asynchronous communication
- Event sourcing capability
- Real-time processing

**C# Event Bus Implementation:**
```csharp
public interface IEventBus
{
    Task PublishAsync<T>(T @event) where T : IEvent;
    Task SubscribeAsync<T>(Func<T, Task> handler) where T : IEvent;
}

public class AzureServiceBusEventBus : IEventBus
{
    private readonly ServiceBusClient _client;
    
    public async Task PublishAsync<T>(T @event) where T : IEvent
    {
        var sender = _client.CreateSender(typeof(T).Name);
        var message = new ServiceBusMessage(JsonSerializer.Serialize(@event))
        {
            Subject = typeof(T).Name,
            MessageId = Guid.NewGuid().ToString()
        };
        await sender.SendMessageAsync(message);
    }
    
    public async Task SubscribeAsync<T>(Func<T, Task> handler) where T : IEvent
    {
        var processor = _client.CreateProcessor(typeof(T).Name);
        processor.ProcessMessageAsync += async args =>
        {
            var @event = JsonSerializer.Deserialize<T>(args.Message.Body);
            await handler(@event);
            await args.CompleteMessageAsync(args.Message);
        };
        await processor.StartProcessingAsync();
    }
}
```

**Follow-up:** "How do you ensure event ordering?"

---

#### **47. How do you handle Distributed Caching in Microservices?**

**Answer:** Distributed caching strategies:
1. **Shared cache** (Redis/Memcached)
2. **Cache-aside pattern**
3. **Cache invalidation strategies**
4. **Cache warming**
5. **Multi-level caching**

**C# Redis Implementation with Patterns:**
```csharp
public class DistributedCacheService
{
    private readonly IConnectionMultiplexer _redis;
    private readonly IDatabase _db;
    
    public async Task<T> GetOrSetAsync<T>(string key, 
        Func<Task<T>> factory, 
        TimeSpan? expiry = null)
    {
        var cached = await _db.StringGetAsync(key);
        if (cached.HasValue)
        {
            return JsonSerializer.Deserialize<T>(cached);
        }
        
        var value = await factory();
        await _db.StringSetAsync(key, 
            JsonSerializer.Serialize(value), 
            expiry ?? TimeSpan.FromMinutes(5));
        
        // Publish invalidation event for other instances
        await _db.PublishAsync("cache:invalidation", key);
        
        return value;
    }
    
    public async Task InvalidateAsync(string pattern)
    {
        var server = _redis.GetServer(_redis.GetEndPoints().First());
        var keys = server.Keys(pattern: pattern);
        await _db.KeyDeleteAsync(keys.ToArray());
    }
}
```

**Follow-up:** "How do you handle cache stampede?"

---

#### **48. What is the Ambassador pattern?**

**Answer:** The Ambassador pattern creates a helper service that sends network requests on behalf of the application, providing:
- Retry logic
- Circuit breaking
- Logging/Monitoring
- Request/Response transformation
- Authentication

**C# Ambassador Implementation:**
```csharp
public class AmbassadorHttpClient
{
    private readonly HttpClient _httpClient;
    private readonly IAsyncPolicy<HttpResponseMessage> _policy;
    
    public AmbassadorHttpClient()
    {
        _policy = Policy.WrapAsync(
            Policy.HandleResult<HttpResponseMessage>(r => !r.IsSuccessStatusCode)
                .CircuitBreakerAsync(3, TimeSpan.FromSeconds(30)),
            Policy.HandleResult<HttpResponseMessage>(r => !r.IsSuccessStatusCode)
                .WaitAndRetryAsync(3, retryAttempt => 
                    TimeSpan.FromSeconds(Math.Pow(2, retryAttempt))));
    }
    
    public async Task<T> SendRequestAsync<T>(HttpRequestMessage request)
    {
        // Add authentication
        request.Headers.Authorization = 
            new AuthenticationHeaderValue("Bearer", await GetTokenAsync());
        
        // Add correlation ID
        request.Headers.Add("X-Correlation-Id", Guid.NewGuid().ToString());
        
        var response = await _policy.ExecuteAsync(
            () => _httpClient.SendAsync(request));
        
        response.EnsureSuccessStatusCode();
        return await response.Content.ReadAsAsync<T>();
    }
}
```

**Follow-up:** "How is Ambassador different from Sidecar?"

---

#### **49. How do you implement Multi-tenancy in Microservices?**

**Answer:** Multi-tenancy strategies:
1. **Shared Database, Shared Schema**: Tenant ID in tables
2. **Shared Database, Separate Schema**: Schema per tenant
3. **Database per Tenant**: Complete isolation
4. **Hybrid**: Mix based on tenant tier

**C# Multi-tenant Implementation:**
```csharp
public class TenantDbContext : DbContext
{
    private readonly ITenantService _tenantService;
    
    public TenantDbContext(ITenantService tenantService)
    {
        _tenantService = tenantService;
    }
    
    protected override void OnConfiguring(DbContextOptionsBuilder options)
    {
        var tenant = _tenantService.GetCurrentTenant();
        var connectionString = tenant.Tier switch
        {
            "Premium" => GetDedicatedDbConnection(tenant.Id),
            "Standard" => GetSharedDbConnection(),
            _ => throw new InvalidOperationException()
        };
        
        options.UseSqlServer(connectionString);
    }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Order>().HasQueryFilter(
            o => o.TenantId == _tenantService.GetCurrentTenant().Id);
    }
    
    public override Task<int> SaveChangesAsync(CancellationToken token)
    {
        foreach (var entry in ChangeTracker.Entries<ITenantEntity>())
        {
            if (entry.State == EntityState.Added)
            {
                entry.Entity.TenantId = _tenantService.GetCurrentTenant().Id;
            }
        }
        return base.SaveChangesAsync(token);
    }
}
```

**Follow-up:** "How do you handle tenant-specific customizations?"

---

#### **50. What are the best practices for Microservices in production?**

**Answer:** Key best practices:

**Design:**
- Domain-driven design
- API versioning
- Idempotent operations
- Backward compatibility

**Development:**
- Automated testing
- Code reviews
- Documentation
- Dependency management

**Deployment:**
- CI/CD pipelines
- Infrastructure as Code
- Feature flags
- Progressive rollouts

**Operations:**
- Comprehensive monitoring
- Centralized logging
- Distributed tracing
- Incident response procedures

**Security:**
- Zero trust architecture
- Secret management
- Regular security audits
- Compliance automation

**Example Production Checklist:**
```csharp
public class ProductionReadinessChecker
{
    public async Task<bool> IsServiceProductionReady(string serviceName)
    {
        var checks = new List<Func<Task<bool>>>
        {
            () => HasHealthEndpoint(serviceName),
            () => HasMetricsEndpoint(serviceName),
            () => HasDocumentation(serviceName),
            () => HasTests(serviceName, minCoverage: 80),
            () => HasSecurityScanning(serviceName),
            () => HasLoadTesting(serviceName),
            () => HasDisasterRecoveryPlan(serviceName),
            () => HasRunbook(serviceName),
            () => HasSLOs(serviceName),
            () => HasAlerts(serviceName)
        };
        
        var results = await Task.WhenAll(checks.Select(c => c()));
        return results.All(r => r);
    }
}
```

**Follow-up:** "How do you prioritize these practices for a startup vs enterprise?"

---

## Part 2: Comprehensive Study Plan for Microservices Mastery

### **Phase 1: Foundation (Weeks 1-4)**

#### Week 1-2: Core Concepts
- **Study Topics:**
  - Microservices principles and characteristics
  - Monolithic vs Microservices architecture
  - Domain-Driven Design basics
  - Bounded contexts
  
- **Hands-on Labs:**
  - Break down a sample monolithic application into services
  - Create your first microservice with ASP.NET Core
  - Implement basic REST APIs

- **Resources:**
  - Book: "Building Microservices" by Sam Newman
  - Pluralsight: "Microservices: The Big Picture"

#### Week 3-4: Communication Patterns
- **Study Topics:**
  - Synchronous vs Asynchronous communication
  - REST, gRPC, GraphQL
  - Message queues and event buses
  - Service discovery

- **Hands-on Labs:**
  - Implement REST communication between services
  - Set up RabbitMQ/Azure Service Bus
  - Create pub/sub messaging

### **Phase 2: Advanced Patterns (Weeks 5-8)**

#### Week 5-6: Data Management
- **Study Topics:**
  - Database per service pattern
  - CQRS and Event Sourcing
  - Distributed transactions
  - Saga pattern

- **Hands-on Labs:**
  - Implement CQRS with MediatR
  - Create an event-sourced aggregate
  - Build a choreography-based saga

#### Week 7-8: Resilience & Reliability
- **Study Topics:**
  - Circuit breaker pattern
  - Retry policies
  - Bulkhead pattern
  - Health checks

- **Hands-on Labs:**
  - Implement Polly for resilience
  - Create custom health checks
  - Build a circuit breaker

### **Phase 3: Platform & Infrastructure (Weeks 9-12)**

#### Week 9-10: Containerization & Orchestration
- **Study Topics:**
  - Docker fundamentals
  - Kubernetes concepts
  - Service mesh (Istio/Linkerd)
  - Container security

- **Hands-on Labs:**
  - Containerize microservices
  - Deploy to Kubernetes
  - Configure auto-scaling

#### Week 11-12: Cloud & DevOps
- **Study Topics:**
  - Azure services for microservices
  - CI/CD pipelines
  - Infrastructure as Code
  - GitOps

- **Hands-on Labs:**
  - Deploy to Azure Kubernetes Service
  - Create Azure DevOps pipelines
  - Implement monitoring with Application Insights

### **Phase 4: Production Readiness (Weeks 13-16)**

#### Week 13-14: Observability & Monitoring
- **Study Topics:**
  - Distributed tracing
  - Centralized logging
  - Metrics and alerting
  - APM tools

- **Hands-on Labs:**
  - Implement OpenTelemetry
  - Set up ELK stack
  - Create dashboards in Grafana

#### Week 15-16: Security & Testing
- **Study Topics:**
  - OAuth 2.0 and OpenID Connect
  - API Gateway security
  - Testing strategies
  - Chaos engineering

- **Hands-on Labs:**
  - Implement IdentityServer
  - Create contract tests with Pact
  - Perform chaos experiments

### **Practical Project Recommendations**

Build a complete e-commerce platform with:
1. Product Catalog Service
2. Order Management Service
3. Payment Service
4. Inventory Service
5. Notification Service
6. User Service

Each iteration should add complexity:
- **v1**: Basic CRUD operations
- **v2**: Add messaging and events
- **v3**: Implement resilience patterns
- **v4**: Add monitoring and security
- **v5**: Deploy to cloud with full DevOps

### **Key Skills to Demonstrate in Interviews**

1. **Architecture Design**: Draw and explain microservices architecture
2. **Pattern Knowledge**: Know when and why to use specific patterns
3. **Problem Solving**: Discuss trade-offs and alternatives
4. **Real-world Experience**: Share challenges and solutions
5. **Tool Proficiency**: Demonstrate knowledge of industry tools

### **Recommended Certifications**
- Microsoft Azure Developer Associate (AZ-204)
- Microsoft Azure Solutions Architect Expert (AZ-305)
- Kubernetes certifications (CKA/CKAD)

### **Interview Preparation Tips**

1. **Practice System Design**: Use platforms like Pramp or System Design Interview
2. **Code Examples**: Prepare small, working examples of patterns
3. **Case Studies**: Study real-world microservices implementations (Netflix, Uber, Amazon)
4. **Mock Interviews**: Practice with peers or mentors
5. **Stay Current**: Follow microservices blogs and conferences

Remember, becoming proficient in microservices is a journey. Focus on understanding the "why" behind patterns and practices, not just the "how". Good luck with your career transition!