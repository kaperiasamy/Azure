# Microservices Fundamentals & Architecture: Service Boundaries Definition

---

## Question 1: How Do You Define Service Boundaries Using Domain-Driven Design (DDD) Principles in Azure Microservices?

### Detailed Answer:

Service boundary definition is the cornerstone of microservices architecture. In Domain-Driven Design (DDD), we identify **Bounded Contexts**—distinct areas of your business domain where specific terminology, rules, and entities apply. Each Bounded Context should map to a microservice, ensuring loose coupling and high cohesion. For example, in an e-commerce system, the "Order Management" Bounded Context differs fundamentally from the "Inventory Management" Bounded Context—they have different vocabularies (Order vs. Stock Item), rules, and responsibilities.

In Azure, this translates to separate services deployed independently, often as Azure Container Instances, App Services, or Azure Functions. Each service owns its data store (database per service pattern), communicates via Azure Service Bus or Event Grid, and exposes contracts through REST APIs or gRPC endpoints. The key is identifying where your domain "language" changes, where teams naturally split, or where deployment cycles differ—these are your service boundaries.

When defining boundaries poorly (e.g., by technical layers instead of business capabilities), you create "chatty" microservices that depend heavily on each other, defeating the purpose of microservices. Tools like Event Storming workshops help teams collaboratively identify Bounded Contexts and establish clear service boundaries before coding begins.

### Explanation:

**Importance & Use Cases:**
- **Prevents coupling**: Service boundaries based on DDD ensure services evolve independently without tight interdependencies.
- **Team autonomy**: Clear boundaries allow different teams to own and deploy services without coordination overhead.
- **Scalability**: Each service scales independently based on actual demand for that business capability.
- **Database independence**: Enables polyglot persistence—each service can choose its data store (SQL, NoSQL, etc.) in Azure.

**Benefits & Trade-offs:**
- **Benefit**: Reduced complexity, independent deployments, faster time-to-market.
- **Trade-off**: Added operational complexity (distributed tracing, eventual consistency challenges, network latency).
- **Interview perspective**: Demonstrates understanding of business architecture, not just technical implementation.

### C# Implementation:

```csharp
using Azure.Messaging.ServiceBus;
using Microsoft.Azure.Functions.Worker;
using Microsoft.Extensions.Logging;

// OrderBoundedContext - owns order-specific logic and data
public class OrderService
{
    private readonly ServiceBusClient _serviceBusClient;
    private readonly ILogger _logger;

    public OrderService(ServiceBusClient serviceBusClient, ILogger logger)
    {
        _serviceBusClient = serviceBusClient;
        _logger = logger;
    }

    // Order domain model - isolated to this boundary
    public record Order(string OrderId, string CustomerId, decimal TotalAmount);

    // When order created, publish domain event for other bounded contexts
    public async Task CreateOrderAsync(Order order)
    {
        _logger.LogInformation($"Order {order.OrderId} created in OrderBoundedContext");
        
        // Publish cross-boundary event (loose coupling)
        var sender = _serviceBusClient.CreateSender("order-events");
        var message = new ServiceBusMessage($"OrderCreated:{order.OrderId}");
        await sender.SendMessageAsync(message);
    }
}

// InventoryBoundedContext - owns inventory-specific logic
public class InventoryService
{
    // Different terminology and rules - separate bounded context
    public record StockItem(string Sku, int QuantityOnHand);
    
    // Subscribes to OrderCreated event (published by Order service)
    [Function("ProcessOrderCreatedEvent")]
    public async Task ProcessOrderEventAsync(
        [ServiceBusTrigger("order-events", Connection = "ServiceBusConnection")] 
        ServiceBusReceivedMessage message,
        IAsyncCollector<ServiceBusMessage> inventoryQueue)
    {
        var body = message.Body.ToString();
        Console.WriteLine($"InventoryBoundedContext received: {body}");
        
        // Inventory responds independently - no direct call to Order service
        await inventoryQueue.AddAsync(new ServiceBusMessage("StockReserved"));
    }
}
```

### Follow-Up Questions:

**Q1: How do you handle data consistency across service boundaries in Azure?**
* **Answer:** Use the Saga pattern via Azure Service Bus or Durable Functions. For the Order → Inventory → Payment flow, each service listens to events and performs compensating transactions on failure. Azure Durable Functions orchestrate multi-step workflows reliably. Alternatively, use event sourcing with Azure Event Hubs to maintain an immutable log. Accept eventual consistency as a trade-off—synchronous calls across boundaries defeat microservices independence. Implement idempotency keys to handle retries safely.

**Q2: What tools or techniques help identify Bounded Contexts during architecture design?**
* **Answer:** Event Storming is the primary technique—gather domain experts, developers, and stakeholders to map business processes using color-coded sticky notes representing events, commands, and aggregates. This reveals where language shifts, indicating context boundaries. Other approaches include analyzing user stories for vocabulary changes, reviewing team organizational structure (Conway's Law), and examining deployment frequency differences. In Azure, use Azure Boards to organize user stories by Bounded Context, making boundaries visible.

**Q3: Should each Bounded Context have its own database? What are the implications for Azure?**
* **Answer:** Yes, the "database per service" pattern follows DDD principles and prevents cross-service data coupling. However, this complicates querying across contexts—avoid joining tables from different services. In Azure, use Azure Cosmos DB for eventually consistent reads across services, materialized views, or read models. Trade-offs include increased storage, backup complexity, and licensing costs. For financial or audit-critical contexts, stronger consistency guarantees may override this rule.

**Q4: How do you prevent service boundaries from becoming too fine-grained?**
* **Answer:** Avoid the "microservice per entity" anti-pattern. Group related entities sharing the same lifecycle and business rules into one service. If services constantly communicate synchronously, they're likely over-partitioned. Monitor Azure Application Insights for high inter-service call frequency—this signals boundaries need realignment. Use team ownership as a gauge: can one team comfortably own and deploy the service? If not, it's too large; if easy, consider merging with adjacent services.

**Q5: How would you model a shared reference data (e.g., product catalog) across multiple Bounded Contexts?**
* **Answer:** Use a **Shared Kernel** pattern—a dedicated lightweight reference service or cache (Azure Cache for Redis) that both Order and Inventory contexts consume. Alternatively, each context maintains its own copy synchronized via events published by a central Product Catalog service. Avoid tight coupling by never calling another service for data during critical operations; instead, denormalize necessary data into your service's database during event processing, accepting eventual consistency.

---

## Question 2: What Strategies Help You Define Service Boundaries When Business Requirements Are Ambiguous or Rapidly Evolving?

### Detailed Answer:

Defining service boundaries in dynamic environments requires flexible strategies beyond static DDD analysis. When requirements evolve rapidly (common in startups or digital transformations), rigidly-defined boundaries become liabilities. A pragmatic approach is **Strangler Pattern deployment** in Azure—start with a monolithic application and gradually extract services as boundaries become clearer. This reduces upfront architectural risk while maintaining functionality.

Another strategy is **using organizational structure as a guide** (Conway's Law principle)—align service boundaries to team structures, as each team naturally optimizes communication within their service. In Azure, this means cross-functional teams can own full end-to-end services in Azure Container Instances or App Services. Additionally, adopt **continuous boundary refinement**—implement comprehensive logging via Azure Monitor and Application Insights to identify coupling hot-spots. If services communicate excessively or share responsibilities, consolidate them. If a single service has conflicting scaling requirements (e.g., one endpoint needs 1000 RPS while another needs 10 RPS), split them.

Finally, design services with **internal extensibility** initially—use plugin architectures (strategy pattern, dependency injection) to defer boundary decisions. As patterns stabilize and deployment independence becomes valuable, extract boundaries formally. This hybrid approach balances YAGNI (You Aren't Gonna Need It) principles with future scalability.

### Explanation:

**Importance & Use Cases:**
- **Risk mitigation**: Avoids over-architecture that creates unmaintainable boundaries.
- **Agile alignment**: Supports iterative refinement as business understanding deepens.
- **Cost optimization**: Prevents premature scaling investments in uncertain areas.
- **Team dynamics**: Reflects real organizational communication patterns, improving autonomy.

**Benefits & Trade-offs:**
- **Benefit**: Reduced technical debt, faster time-to-value, adaptive architecture.
- **Trade-off**: Requires discipline to refactor boundaries; refactoring distributed systems is harder than monoliths. Demands strong monitoring and observability investments.
- **Interview perspective**: Shows pragmatism, risk awareness, and understanding that architecture evolves—not a sign of poor initial design.

### C# Implementation:

```csharp
using Azure.Monitor.OpenTelemetry;
using Azure.Data.Tables;
using Microsoft.ApplicationInsights;
using Microsoft.ApplicationInsights.DataContracts;

// Strategy 1: Strangler Pattern - Gradually extract service boundaries
public class OrderFacadeService
{
    private readonly HttpClient _legacyMonolithClient;
    private readonly OrderMicroservice _newOrderService;
    private readonly TelemetryClient _telemetryClient;

    public OrderFacadeService(
        HttpClient legacyClient, 
        OrderMicroservice orderService,
        TelemetryClient telemetry)
    {
        _legacyMonolithClient = legacyClient;
        _newOrderService = orderService;
        _telemetryClient = telemetry;
    }

    public async Task<Order> GetOrderAsync(string orderId, bool useNewService = false)
    {
        // Gradually migrate traffic to new microservice
        // Track which code path is used for observability
        using var operation = _telemetryClient.StartOperation<RequestTelemetry>("GetOrder");
        operation.Telemetry.Properties.Add("service", useNewService ? "microservice" : "monolith");

        if (useNewService)
        {
            return await _newOrderService.GetOrderAsync(orderId);
        }
        else
        {
            // Legacy path still works; gradually reduce percentage
            var response = await _legacyMonolithClient.GetAsync($"/orders/{orderId}");
            return await response.Content.ReadAsAsync<Order>();
        }
    }
}

// Strategy 2: Monitor coupling metrics to refine boundaries
public class ServiceBoundaryCouplingAnalyzer
{
    private readonly TableClient _couplingTableClient;

    public ServiceBoundaryCouplingAnalyzer(TableClient tableClient)
    {
        _couplingTableClient = tableClient;
    }

    // Log every inter-service call to identify boundary issues
    public async Task LogServiceCallAsync(
        string sourceService, 
        string targetService, 
        int responseTimeMs)
    {
        var entity = new TableEntity(sourceService, targetService)
        {
            { "timestamp", DateTime.UtcNow },
            { "responseTimeMs", responseTimeMs },
            { "callCount", 1 }
        };

        await _couplingTableClient.UpsertEntityAsync(entity);
        
        // Alert if services are too chatty
        if (responseTimeMs > 500)
        {
            Console.WriteLine(
                $"⚠️ High latency: {sourceService} → {targetService} ({responseTimeMs}ms). " +
                $"Consider boundary consolidation.");
        }
    }
}

// Strategy 3: Plugin architecture for deferred boundary decisions
public interface IOrderProcessor
{
    Task ProcessAsync(Order order);
}

public class OrderService
{
    private readonly List<IOrderProcessor> _processors;

    public OrderService(IEnumerable<IOrderProcessor> processors)
    {
        // Plugins can be replaced/extracted later as services
        _processors = new List<IOrderProcessor>(processors);
    }

    public async Task CreateOrderAsync(Order order)
    {
        foreach (var processor in _processors)
        {
            await processor.ProcessAsync(order);
        }
    }
}

// Later, extract ValidateInventory as separate service
public class InventoryValidationPlugin : IOrderProcessor
{
    public async Task ProcessAsync(Order order)
    {
        // Can be extracted to microservice when mature
        Console.WriteLine($"Validating inventory for order {order.OrderId}");
    }
}
```

### Follow-Up Questions:

**Q1: How do you decide between the Strangler Pattern and a complete greenfield microservices rewrite?**
* **Answer:** Strangler is preferable for existing systems where business continuity is critical and risk tolerance is low. It allows parallel operation of old/new systems, enabling gradual validation of boundaries. Greenfield rewrites only work if you can afford a complete outage window and have deep business domain knowledge upfront. In Azure, Strangler via Azure API Management's routing policies or Azure App Gateway allows traffic splitting. Monitor both paths in Application Insights before fully migrating to catch boundary issues early.

**Q2: What metrics in Azure Monitor indicate service boundaries are poorly defined?**
* **Answer:** High inter-service latency (>500ms), synchronous call chains exceeding 3 hops, circular dependencies detected via dependency graphs, and uneven scaling (one service always throttles others). Also monitor for high transaction failure rates when services coordinate—this suggests eventual consistency is needed. If a single business operation requires coordinating 5+ services, boundaries are too fine-grained. Use Application Insights' dependency graph and custom events to surface these metrics.

**Q3: Can you apply DDD Bounded Contexts incrementally within a monolith before extracting services?**
* **Answer:** Absolutely. This is excellent practice—organize your monolith code by Bounded Context (separate folders/namespaces), establish clear contracts between contexts (anti-corruption layers), and implement event buses internally (MediatR or similar). This prepares the codebase for eventual service extraction without major rewrites. When a context matures and needs independent scaling/deployment, extract it to a service with minimal changes—the internal event bus becomes Azure Service Bus.

**Q4: How do you handle service boundary changes after deployment to production?**
* **Answer:** Use feature flags (Azure App Configuration) to enable/disable routing between old and new boundaries without redeployment. Implement data migration strategies with backward compatibility—never delete columns immediately, just stop writing to them. Use Azure Service Bus dead-letter queues to handle messages from deprecated boundaries. Plan boundary changes in quarterly architecture reviews rather than ad-hoc, minimizing churn. Communicate changes to dependent teams early via RACI matrices.

**Q5: Should boundaries align with deployment frequency or business capability first?**
* **Answer:** Business capability should be primary—align to how the business thinks and organizes. Deployment frequency is secondary but important: if one business capability needs daily deploys while another is quarterly, split them to enable independent velocity. However, don't over-fragment for minor differences. In Azure, use deployment slots and blue-green deployments to decouple business capability boundaries from deployment schedules initially, merging or splitting services only when operational pain becomes evident.

---

## Summary Table: Service Boundary Definition Strategies

| Strategy | When to Use | Azure Tools | Trade-offs |
|----------|------------|-------------|-----------|
| **DDD Bounded Contexts** | Mature domain understanding | Event Grid, Service Bus | Requires upfront analysis |
| **Strangler Pattern** | Legacy system modernization | API Management, App Gateway | Temporary complexity (old+new) |
| **Team Ownership** | Large organizations | Azure DevOps, Boards | May not align with domain |
| **Monitoring-Driven Refinement** | Evolving requirements | Application Insights, Monitor | Reactive, requires discipline |
| **Plugin Architecture** | High uncertainty | Dependency Injection | Complex until extraction |

---
# Microservices Fundamentals & Architecture: Service Boundaries Definition (Continued)

---

## Question 3: How Do You Establish Clear Service Contracts and Communication Patterns Within Defined Boundaries?

### Detailed Answer:

Once service boundaries are defined, establishing explicit contracts prevents hidden dependencies and enables autonomous development. A **service contract** encompasses the API specification (REST, gRPC, or messaging), data schemas, versioning strategy, and error handling conventions. In Azure microservices, contracts are typically defined using OpenAPI/Swagger for REST APIs or Protocol Buffers for gRPC. The critical principle is **contract-first design**—define contracts before implementation, allowing client and server teams to work in parallel.

Azure provides multiple patterns for contract enforcement. For **synchronous communication**, use Azure API Management to version APIs, enforce schemas, and provide API contracts via OpenAPI documentation. For **asynchronous messaging**, define message schemas in Azure Service Bus or Event Grid using JSON Schema, ensuring consumers can handle contract evolution. A robust contract includes not just happy-path behavior but also **failure contracts**—documented exceptions, retry policies, timeouts, and circuit-breaker thresholds. For example, if the Inventory service has a 2-second timeout, the Order service must know this and handle timeouts appropriately.

**Backward compatibility** is essential for service boundary health. When a service publishes a new API version, older consumers must continue functioning. Achieve this through additive-only changes (new optional fields), deprecation headers (HTTP 410 Gone with migration guidance), and API versioning strategies (URL path versioning `/v1/orders`, header versioning, or media type versioning). In Azure, use Azure API Management policies to map multiple API versions to backend implementations, ensuring smooth transitions without coordinated deployments.

### Explanation:

**Importance & Use Cases:**
- **Prevents contract violations**: Clear contracts catch integration bugs early, reducing production incidents.
- **Enables parallel development**: Teams work independently when contracts are agreed upon upfront.
- **Supports safe evolution**: Versioning strategies allow boundaries to evolve without breaking consumers.
- **Reduces debugging time**: Well-documented contracts provide debugging context in distributed systems.

**Benefits & Trade-offs:**
- **Benefit**: Reduced coupling, faster development velocity, predictable integration points.
- **Trade-off**: Initial overhead in defining comprehensive contracts; requires discipline to maintain versioning schemes.
- **Interview perspective**: Demonstrates understanding of distributed system resilience, not just architecture theory.

### C# Implementation:

```csharp
using Azure.Core.Serialization;
using Azure.Messaging.ServiceBus;
using Microsoft.AspNetCore.Mvc;
using System.ComponentModel.DataAnnotations;
using System.Text.Json.Serialization;

// ============ SERVICE CONTRACT DEFINITION ============

/// <summary>
/// Order Service Contract - RESTful API
/// Version: 1.0
/// Deprecation: This version will be unsupported after 2025-12-31
/// </summary>
[ApiVersion("1.0")]
[Route("api/v{version:apiVersion}/orders")]
[ApiController]
public class OrdersController : ControllerBase
{
    private readonly IOrderRepository _repository;

    public OrdersController(IOrderRepository repository)
    {
        _repository = repository;
    }

    /// <summary>
    /// Create a new order
    /// </summary>
    /// <remarks>
    /// **Contract Details:**
    /// - Request schema validates all fields
    /// - Response includes Location header (RFC 7231)
    /// - Returns 409 Conflict if order ID already exists
    /// - Timeout: 5 seconds
    /// - Retry Policy: Idempotent (safe to retry with same IdempotencyKey)
    /// </remarks>
    [HttpPost]
    [ProducesResponseType(typeof(OrderResponse), StatusCodes.Status201Created)]
    [ProducesResponseType(typeof(ErrorResponse), StatusCodes.Status400BadRequest)]
    [ProducesResponseType(typeof(ErrorResponse), StatusCodes.Status409Conflict)]
    public async Task<IActionResult> CreateOrderAsync(
        [FromBody] CreateOrderRequest request,
        [FromHeader(Name = "Idempotency-Key")] string? idempotencyKey)
    {
        if (!ModelState.IsValid)
        {
            return BadRequest(new ErrorResponse
            {
                ErrorCode = "INVALID_REQUEST",
                Message = "Request validation failed",
                Details = ModelState.Values.SelectMany(v => v.Errors).Select(e => e.ErrorMessage)
            });
        }

        try
        {
            var order = await _repository.CreateOrderAsync(request, idempotencyKey);
            return CreatedAtAction(nameof(GetOrderAsync), new { id = order.OrderId }, 
                new OrderResponse(order));
        }
        catch (OrderAlreadyExistsException ex)
        {
            return Conflict(new ErrorResponse
            {
                ErrorCode = "ORDER_ALREADY_EXISTS",
                Message = ex.Message,
                CorrelationId = HttpContext.TraceIdentifier
            });
        }
    }

    /// <summary>
    /// Retrieve order by ID
    /// </summary>
    /// <remarks>
    /// - Returns 404 if order not found
    /// - Supports ETag for caching (RFC 7232)
    /// - Timeout: 3 seconds
    /// </remarks>
    [HttpGet("{id}")]
    [ProducesResponseType(typeof(OrderResponse), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<OrderResponse>> GetOrderAsync(string id)
    {
        var order = await _repository.GetOrderAsync(id);
        if (order == null)
            return NotFound();

        return Ok(new OrderResponse(order));
    }
}

// ============ REQUEST/RESPONSE CONTRACTS ============

/// <summary>
/// Contract for creating an order
/// Schema validation rules enforced
/// </summary>
public record CreateOrderRequest(
    [Required(ErrorMessage = "CustomerId is required")]
    [StringLength(50, MinimumLength = 1)]
    string CustomerId,

    [Required]
    [Range(0.01, double.MaxValue, ErrorMessage = "TotalAmount must be positive")]
    decimal TotalAmount,

    [JsonPropertyName("line_items")]
    List<LineItemRequest> LineItems
);

public record LineItemRequest(
    [Required]
    string Sku,

    [Range(1, int.MaxValue)]
    int Quantity
);

/// <summary>
/// Standard response contract for all Order endpoints
/// </summary>
public record OrderResponse(
    [JsonPropertyName("order_id")]
    string OrderId,

    [JsonPropertyName("customer_id")]
    string CustomerId,

    [JsonPropertyName("total_amount")]
    decimal TotalAmount,

    [JsonPropertyName("status")]
    string Status,

    [JsonPropertyName("created_at")]
    DateTime CreatedAt
)
{
    public OrderResponse(Order order) : this(
        order.OrderId,
        order.CustomerId,
        order.TotalAmount,
        order.Status,
        order.CreatedAt)
    {
    }
}

/// <summary>
/// Standard error response contract
/// Used consistently across all services for debugging
/// </summary>
public record ErrorResponse
{
    [JsonPropertyName("error_code")]
    public string ErrorCode { get; init; }

    [JsonPropertyName("message")]
    public string Message { get; init; }

    [JsonPropertyName("details")]
    public IEnumerable<string>? Details { get; init; }

    [JsonPropertyName("correlation_id")]
    public string? CorrelationId { get; init; }

    [JsonPropertyName("timestamp")]
    public DateTime Timestamp { get; init; } = DateTime.UtcNow;
}

// ============ MESSAGING CONTRACT (ASYNC) ============

/// <summary>
/// Domain event contract published to Azure Service Bus
/// Versioned via Type property to enable backward compatibility
/// </summary>
public record OrderCreatedEvent
{
    [JsonPropertyName("event_type")]
    public string EventType { get; } = "OrderCreated";

    [JsonPropertyName("schema_version")]
    public string SchemaVersion { get; } = "1.0";

    [JsonPropertyName("order_id")]
    public required string OrderId { get; init; }

    [JsonPropertyName("customer_id")]
    public required string CustomerId { get; init; }

    [JsonPropertyName("total_amount")]
    public required decimal TotalAmount { get; init; }

    [JsonPropertyName("timestamp")]
    public DateTime Timestamp { get; init; } = DateTime.UtcNow;

    [JsonPropertyName("metadata")]
    public Dictionary<string, string>? Metadata { get; init; }
}

// ============ CONTRACT ENFORCEMENT & VERSIONING ============

public class ContractEnforcementMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<ContractEnforcementMiddleware> _logger;

    public ContractEnforcementMiddleware(RequestDelegate next, ILogger<ContractEnforcementMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        // Enforce required headers per contract
        if (!context.Request.Headers.TryGetValue("X-Api-Version", out var apiVersion))
        {
            context.Request.Headers.Add("X-Api-Version", "1.0");
        }

        // Log contract violations for monitoring
        context.Response.OnStarting(async () =>
        {
            if (context.Response.StatusCode >= 400)
            {
                _logger.LogWarning(
                    "Contract violation detected: {Method} {Path} returned {StatusCode}",
                    context.Request.Method,
                    context.Request.Path,
                    context.Response.StatusCode);
            }
        });

        await _next(context);
    }
}

// ============ ASYNC MESSAGE PUBLISHER WITH CONTRACT VALIDATION ============

public class OrderEventPublisher
{
    private readonly ServiceBusClient _serviceBusClient;
    private readonly JsonObjectSerializer _serializer;

    public OrderEventPublisher(ServiceBusClient serviceBusClient)
    {
        _serviceBusClient = serviceBusClient;
        _serializer = new JsonObjectSerializer();
    }

    public async Task PublishOrderCreatedAsync(OrderCreatedEvent @event)
    {
        var sender = _serviceBusClient.CreateSender("order-events");

        // Serialize with contract schema validation
        var eventBody = _serializer.Serialize(@event);
        var message = new ServiceBusMessage(eventBody)
        {
            Subject = "OrderCreated",
            ContentType = "application/json",
            // Enable idempotent publishing
            MessageId = Guid.NewGuid().ToString(),
        };

        // Set message expiration per contract
        message.TimeToLive = TimeSpan.FromHours(24);

        await sender.SendMessageAsync(message);
        Console.WriteLine($"✓ OrderCreated event published (Contract v1.0): {@event.OrderId}");
    }
}

// ============ MODELS ============

public class Order
{
    public string OrderId { get; set; }
    public string CustomerId { get; set; }
    public decimal TotalAmount { get; set; }
    public string Status { get; set; }
    public DateTime CreatedAt { get; set; }
}

public interface IOrderRepository
{
    Task<Order> CreateOrderAsync(CreateOrderRequest request, string? idempotencyKey);
    Task<Order?> GetOrderAsync(string id);
}

// ============ EXCEPTIONS ============

public class OrderAlreadyExistsException : Exception
{
    public OrderAlreadyExistsException(string message) : base(message) { }
}
```

### Follow-Up Questions:

**Q1: How do you handle breaking changes to service contracts without coordinating deployment with all consumers?**
* **Answer:** Use the **Strangler Pattern with versioning**—deploy new API version alongside old version (e.g., `/v2/orders` alongside `/v1/orders`). In Azure API Management, route requests to appropriate backend versions. Publish deprecation timeline in API documentation and response headers (`Deprecation: true`, `Sunset: Wed, 21 Dec 2025 23:59:59 GMT`). Implement feature flags to gradually shift traffic to new version. Maintain backward compatibility window (typically 6–12 months), then retire old version. Use Application Insights to track adoption of new vs. old versions before sunset date.

**Q2: How do you validate message contracts on Azure Service Bus to prevent integration failures?**
* **Answer:** Define JSON Schema and validate incoming messages before processing. Use Azure Functions with System.Text.Json or Newtonsoft.Json to deserialize and validate messages. Store schemas in Azure Blob Storage or GitHub and version them. For failed validations, route messages to dead-letter queues for manual review. Alternatively, use Azure Event Grid with schema validation enabled, or implement a schema registry pattern (similar to Kafka Schema Registry) using Azure Container Registry to store versioned schemas. Always log contract mismatches with correlation IDs for debugging.

**Q3: What happens when an older service consumer doesn't understand a new field in a service response?**
* **Answer:** Modern serializers like System.Text.Json ignore unknown properties by default (with `JsonSerializerOptions.PropertyNamingPolicy`), so adding new optional fields is safe. Always add new fields as optional (nullable or with default values). For critical changes, bump schema version and maintain both versions temporarily. Monitor your consumers—if you see 400 Bad Request errors suddenly spike after a deployment, a consumer likely can't parse your new response format. Document field deprecation clearly: mark old fields with `[Obsolete]` attributes and provide migration guides in API documentation.

**Q4: How do you version API contracts in a way that scales to 20+ microservices?**
* **Answer:** Centralize contract management using **Azure API Management (APIM)** as the API gateway. Define versioning strategy consistently across all services: use URL path versioning (`/v1/`, `/v2/`) for clarity, not header-based versioning which is harder to debug. Store OpenAPI specs in a shared repository (GitHub/Azure Repos) organized by service and version. Use APIM policies to handle version routing, transformations, and backward compatibility (e.g., adding legacy fields to new responses). Enforce contract reviews in pull requests before merging. Document sunset dates in a shared spreadsheet, coordinating deprecation across teams via quarterly architecture meetings.

**Q5: Should service contracts include SLA/timeout details, or is that infrastructure configuration?**
* **Answer:** Contracts should include **both**. Document expected timeouts, retry policies, circuit-breaker thresholds, and rate limits in API documentation as part of the contract—consumers need this to implement resilience correctly. However, actual timeout values should be configurable in Azure (via code or App Configuration) so they can be tuned based on real production metrics without redeploying consumers. For example, document "Inventory service timeout: typically 2 seconds, max 5 seconds" but implement it as a configurable value. Use Application Insights alerts to notify if timeouts consistently approach limits, indicating need to adjust either timeout or service performance.

---

## Question 4: How Do You Manage Data Consistency and Prevent Data Leakage Across Service Boundaries?

### Detailed Answer:

One of the most critical aspects of service boundary definition is **data ownership and isolation**. Each service owns its data store exclusively; no other service should directly query another service's database. This principle, called the **database-per-service pattern**, prevents tight coupling and enables independent scaling and technology choices. However, it introduces complexity: if Order service needs customer data owned by Customer service, it cannot query the Customer database directly. Instead, it must either request data via Customer service API (synchronous) or maintain its own denormalized copy synchronized via events (asynchronous).

**Data consistency challenges** emerge at boundaries. When an order spans multiple services (Order → Inventory → Payment), ensuring atomicity becomes impossible in traditional ACID terms. Instead, adopt **eventual consistency** patterns. The **Saga pattern** orchestrates multi-step transactions, where each step is a local transaction in one service, and failures trigger compensating transactions. In Azure, implement Sagas via **Durable Functions** (orchestration pattern) or **event-driven choreography** using Azure Service Bus or Event Grid. For example, OrderCreated event triggers inventory reservation; if reservation fails, a compensating OrderCancelled event is published, triggering refunds.

**Data leakage**—exposing internal implementation details through service APIs—undermines boundary integrity. A common anti-pattern is returning database IDs or internal domain models in APIs. Instead, define public API contracts separate from internal models. Use **anti-corruption layers** (adapters) that translate between internal representations and API contracts. For instance, internal Order entity might contain internal ProductId, but the API contract exposes only ProductName and SKU, preventing coupling to the Product service's ID scheme.

### Explanation:

**Importance & Use Cases:**
- **Service autonomy**: Independent data stores mean services scale and evolve independently.
- **Reduces cascading failures**: Data corruption in one service doesn't spread to others.
- **Enables polyglot persistence**: One service uses PostgreSQL, another uses Cosmos DB—optimal for each boundary.
- **Audit and compliance**: Data ownership is unambiguous, simplifying GDPR/HIPAA compliance.

**Benefits & Trade-offs:**
- **Benefit**: Resilience, scalability, independent team autonomy, technology flexibility.
- **Trade-off**: Complexity (eventual consistency, saga pattern implementation), higher storage/infrastructure costs, harder querying across services.
- **Interview perspective**: Shows understanding of distributed systems trade-offs and practical problem-solving.

### C# Implementation:

```csharp
using Azure.Data.Cosmos;
using Azure.Messaging.ServiceBus;
using Microsoft.DurableTask;
using Microsoft.DurableTask.Client;
using System.Transactions;

// ============ DATABASE-PER-SERVICE PATTERN ============

// Order Service - owns its own data
public interface IOrderRepository
{
    Task<Order> CreateOrderAsync(Order order);
    Task<Order> GetOrderAsync(string orderId);
    Task UpdateOrderStatusAsync(string orderId, string status);
}

public class OrderService
{
    private readonly IOrderRepository _orderRepository;
    private readonly ServiceBusClient _serviceBusClient;

    public OrderService(IOrderRepository orderRepository, ServiceBusClient serviceBusClient)
    {
        _orderRepository = orderRepository;
        _serviceBusClient = serviceBusClient;
    }

    // DO NOT directly query Customer or Inventory databases
    // Instead, publish events and respond to events asynchronously
    public async Task<Order> CreateOrderAsync(CreateOrderRequest request)
    {
        var order = new Order
        {
            OrderId = Guid.NewGuid().ToString(),
            CustomerId = request.CustomerId,
            TotalAmount = request.TotalAmount,
            Status = "Pending",
            CreatedAt = DateTime.UtcNow
        };

        // Store in Order service database
        await _orderRepository.CreateOrderAsync(order);

        // Publish event—other services listen and act independently
        await PublishOrderCreatedEventAsync(order);

        return order;
    }

    private async Task PublishOrderCreatedEventAsync(Order order)
    {
        var sender = _serviceBusClient.CreateSender("order-events");
        var eventMessage = new ServiceBusMessage(
            System.Text.Json.JsonSerializer.Serialize(new
            {
                EventType = "OrderCreated",
                OrderId = order.OrderId,
                CustomerId = order.CustomerId,
                TotalAmount = order.TotalAmount
            }))
        {
            Subject = "OrderCreated"
        };

        await sender.SendMessageAsync(eventMessage);
    }
}

// Inventory Service - owns its own data (separate database)
public interface IInventoryRepository
{
    Task<bool> ReserveInventoryAsync(string orderId, List<ReservationItem> items);
    Task ReleaseInventoryAsync(string orderId);
}

public class InventoryService
{
    private readonly IInventoryRepository _inventoryRepository;
    private readonly ServiceBusClient _serviceBusClient;

    public InventoryService(
        IInventoryRepository inventoryRepository,
        ServiceBusClient serviceBusClient)
    {
        _inventoryRepository = inventoryRepository;
        _serviceBusClient = serviceBusClient;
    }

    // Process OrderCreated event from Order service
    public async Task ProcessOrderCreatedAsync(string orderId, List<ReservationItem> items)
    {
        try
        {
            // Reserve inventory in Inventory service database (local transaction)
            bool reserved = await _inventoryRepository.ReserveInventoryAsync(orderId, items);

            if (reserved)
            {
                await PublishInventoryReservedEventAsync(orderId);
            }
            else
            {
                await PublishInventoryReservationFailedEventAsync(orderId);
            }
        }
        catch (Exception ex)
        {
            // Failure triggers compensating transaction
            await PublishInventoryReservationFailedEventAsync(orderId);
        }
    }

    private async Task PublishInventoryReservedEventAsync(string orderId)
    {
        var sender = _serviceBusClient.CreateSender("inventory-events");
        var message = new ServiceBusMessage($"{{\"EventType\":\"InventoryReserved\",\"OrderId\":\"{orderId}\"}}")
        {
            Subject = "InventoryReserved"
        };
        await sender.SendMessageAsync(message);
    }

    private async Task PublishInventoryReservationFailedEventAsync(string orderId)
    {
        var sender = _serviceBusClient.CreateSender("order-events");
        var message = new ServiceBusMessage($"{{\"EventType\":\"InventoryReservationFailed\",\"OrderId\":\"{orderId}\"}}")
        {
            Subject = "InventoryReservationFailed"
        };
        await sender.SendMessageAsync(message);
    }
}

// ============ SAGA PATTERN VIA DURABLE FUNCTIONS ============

/// <summary>
/// Orchestrates multi-service transaction via Azure Durable Functions
/// Implements Saga pattern for distributed transactions across service boundaries
/// </summary>
public class OrderFulfillmentOrchestrator
{
    [Function(nameof(OrderFulfillmentSaga))]
    public async Task OrderFulfillmentSaga(
        [OrchestrationTrigger] TaskOrchestrationContext context,
        string orderId)
    {
        var orderData = context.GetInput<OrderData>();
        var compensation = new List<Func<Task>>();

        try
        {
            // Step 1: Reserve inventory (local transaction in Inventory service)
            await context.CallActivityAsync(
                nameof(ReserveInventoryActivity),
                new { OrderId = orderId, Items = orderData.Items });
            
            compensation.Add(async () => 
                await context.CallActivityAsync(nameof(ReleaseInventoryActivity), orderId));

            // Step 2: Process payment (local transaction in Payment service)
            await context.CallActivityAsync(
                nameof(ProcessPaymentActivity),
                new { OrderId = orderId, Amount = orderData.TotalAmount });
            
            compensation.Add(async () =>
                await context.CallActivityAsync(nameof(RefundPaymentActivity), orderId));

            // Step 3: Confirm order (local transaction in Order service)
            await context.CallActivityAsync(
                nameof(ConfirmOrderActivity),
                orderId);

            // Success - saga completed
        }
        catch (Exception ex)
        {
            // Compensation: undo in reverse order (like rollback)
            foreach (var comp in Enumerable.Reverse(compensation))
            {
                try
                {
                    await comp();
                }
                catch (Exception compEx)
                {
                    // Log compensation failure for manual intervention
                    context.CreateTimer(
                        context.CurrentUtcDateTime.AddSeconds(5),
                        CancellationToken.None);
                }
            }

            throw;
        }
    }

    [Function(nameof(ReserveInventoryActivity))]
    public async Task ReserveInventoryActivity(
        [ActivityTrigger] IDictionary<string, object> input,
        ILogger log)
    {
        var orderId = input["OrderId"].ToString();
        log.LogInformation($"Reserving inventory for order {orderId}");
        // Call Inventory service API or trigger via Service Bus
    }

    [Function(nameof(ReleaseInventoryActivity))]
    public async Task ReleaseInventoryActivity([ActivityTrigger] string orderId)
    {
        Console.WriteLine($"Compensating: Releasing inventory for order {orderId}");
    }

    [Function(nameof(ProcessPaymentActivity))]
    public async Task ProcessPaymentActivity([ActivityTrigger] IDictionary<string, object> input)
    {
        var orderId = input["OrderId"].ToString();
        var amount = decimal.Parse(input["Amount"].ToString());
        Console.WriteLine($"Processing payment of {amount} for order {orderId}");
    }

    [Function(nameof(RefundPaymentActivity))]
    public async Task RefundPaymentActivity([ActivityTrigger] string orderId)
    {
        Console.WriteLine($"Compensating: Refunding payment for order {orderId}");
    }

    [Function(nameof(ConfirmOrderActivity))]
    public async Task ConfirmOrderActivity([ActivityTrigger] string orderId)
    {
        Console.WriteLine($"Confirming order {orderId}");
    }
}

// ============ ANTI-CORRUPTION LAYER (DATA BOUNDARY PROTECTION) ============

/// <summary>
/// Anti-corruption layer prevents internal data representation from leaking through API boundaries
/// </summary>
public class OrderApiModel
{
    // Public API contract - never exposes internal IDs or implementation details
    public string OrderId { get; set; }
    public string CustomerName { get; set; }  // NOT CustomerId (internal detail)
    public decimal TotalAmount { get; set; }
    public List<OrderItemApiModel> Items { get; set; }
}

public class OrderItemApiModel
{
    public string ProductName { get; set; }   // NOT ProductId (external boundary)
    public string Sku { get; set; }
    public int Quantity { get; set; }
}

/// <summary>
/// Adapter translates internal Order domain model to public API contract
/// Prevents coupling between internal representation and external consumers
/// </summary>
public class OrderApiAdapter
{
    private readonly IProductService _productService;

    public OrderApiAdapter(IProductService productService)
    {
        _productService = productService;
    }

    public async Task<OrderApiModel> ConvertToApiModelAsync(Order internalOrder)
    {
        // Fetch denormalized product names (don't expose ProductIds)
        var itemsWithNames = new List<OrderItemApiModel>();
        foreach (var item in internalOrder.Items)
        {
            var productName = await _productService.GetProductNameAsync(item.ProductId);
            itemsWithNames.Add(new OrderItemApiModel
            {
                ProductName = productName,
                Sku = item.Sku,
                Quantity = item.Quantity
            });
        }

        return new OrderApiModel
        {
            OrderId = internalOrder.OrderId,
            CustomerName = internalOrder.CustomerName,  // Fetch via API, not expose ID
            TotalAmount = internalOrder.TotalAmount,
            Items = itemsWithNames
        };
    }
}

// ============ DATA MODELS ============

public class Order
{
    public string OrderId { get; set; }
    public string CustomerId { get; set; }
    public string CustomerName { get; set; }
    public decimal TotalAmount { get; set; }
    public string Status { get; set; }
    public List<OrderItem> Items { get; set; } = new();
    public DateTime CreatedAt { get; set; }
}

public class OrderItem
{
    public string Sku { get; set; }
    public string ProductId { get; set; }  // Internal ID—never exposed in API
    public int Quantity { get; set; }
}

public record OrderData(string OrderId, decimal TotalAmount, List<ReservationItem> Items);
public record ReservationItem(string Sku, int Quantity);

public interface IProductService
{
    Task<string> GetProductNameAsync(string productId);
}
```

### Follow-Up Questions:

**Q1: How do you handle denormalization and caching of data from other services without creating tight coupling?**
* **Answer:** Use **eventual consistency with event-driven synchronization**. When Product service updates a product, it publishes ProductUpdated event. Order service subscribes and updates its denormalized product cache (in its database or Redis via Azure Cache for Redis). This respects service boundaries—Order never directly queries Product database. Cache invalidation is handled via event timestamps; if Order's cached data is stale, periodically refresh. For critical reads requiring fresh data, call the owning service API synchronously with a timeout and circuit-breaker. Document cache freshness SLAs in service contracts (e.g., "product names may be 5 minutes stale").

**Q2: What happens if a Saga fails midway—how do you ensure compensation logic runs reliably?**
* **Answer:** Use **Azure Durable Functions** for reliable orchestration—it persists orchestration state so retries survive service restarts. If compensation fails, the saga goes into a "Failed" state; implement monitoring (Application Insights) to alert operators, who manually trigger compensation or rollback via admin endpoints. Store failed sagas in a dead-letter queue (Azure Service Bus) for audit. For critical transactions (payments), implement **idempotency**—track each transaction step with a unique ID; replaying steps with same ID returns the same result without duplicating charges. Document compensation SLAs: if compensation takes >24 hours, escalate to manual review.

**Q3: How do you prevent "chatty" services that request too much data across boundaries, impacting performance?**
* **Answer:** **Batch requests and use GraphQL or composite APIs**. Instead of Order service making 100 requests to Product service (one per item), batch them: `POST /products/batch {"ids": ["p1","p2",...]}`. Alternatively, use **API composition** via Azure API Management or a dedicated composition service that aggregates data from multiple services efficiently. Cache responses in Redis (5-minute TTL). Monitor inter-service call counts in Application Insights; if calls/request ratio >3, it indicates poor boundary design—consolidate services or redesign data flow. For reporting queries spanning multiple services, use **read models/CQRS pattern** where a dedicated read service denormalizes data from multiple sources.

**Q4: How do you handle GDPR "right to be forgotten" across service boundaries when data is distributed?**
* **Answer:** **Establish a data purge contract**—when Customer service receives a delete request, it publishes `CustomerDeleted` event. All services listening (Order, Inventory, Payment) receive this event and delete associated records from their databases within an agreed SLA (e.g., 48 hours). Implement a **data purge orchestrator** (Durable Function) that tracks deletion status across services and alerts if any service doesn't acknowledge deletion. Store audit logs separately (immutable, for compliance) but purge personal data. For compliance audits, each service maintains a purge log with deletion timestamps. Use encryption keys—if a service can't delete data, rotate encryption keys to make old data unreadable.

**Q5: Should read-heavy queries use a separate database per service, or share a read replica?**
* **Answer:** **Use separate read models per service boundary**. Each service owns a read database (possibly a read replica of its write database) for performance. This respects boundaries—Order service doesn't query Inventory's read database. For complex reporting spanning multiple services, implement a **CQRS read database** (separate from operational databases) that's fed by events from all services. In Azure, use Azure Synapse Analytics or Data Lake for this central reporting database, fed by event streams. It's read-only and eventually consistent, suitable for analytics but not operational queries. Document the "freshness window"—data may be 5–30 minutes stale. For operational queries requiring real-time accuracy, call service APIs directly despite latency cost.

---

## Summary: Service Boundary Data Management Strategies

| Pattern | Use Case | Azure Tools | Trade-offs |
|---------|----------|------------|-----------|
| **Database-per-Service** | Independent scaling, polyglot persistence | Cosmos DB, SQL Database, PostgreSQL | Complex eventual consistency |
| **Saga Pattern (Orchestration)** | Multi-service transactions with rollback | Durable Functions | Higher complexity, latency |
| **Saga Pattern (Choreography)** | Event-driven distributed transactions | Service Bus, Event Grid | Harder to visualize flow |
| **Anti-Corruption Layer** | Boundary data protection | Custom adapters, APIM policies | Maintenance overhead |
| **CQRS Read Models** | Complex multi-service queries | Synapse Analytics, Data Lake | Eventual consistency gap |
| **API Composition** | Reduce inter-service chatter | API Management, Composite APIs | Single point of failure risk |

---

## Comparison Table: Questions 1–4

| Question | Focus | Key Pattern | Azure Service |
|----------|-------|-------------|---------------|
| **Q1: Defining Boundaries** | DDD, team structure, refactoring | Strangler, Bounded Contexts | Event Grid, Service Bus |
| **Q2: Ambiguous Requirements** | Dynamic evolution, monitoring | Strangler, Plugin Architecture | Application Insights, App Configuration |
| **Q3: Service Contracts** | API design, versioning, backward compatibility | Contract-First, API Versioning | API Management, OpenAPI |
| **Q4: Data Consistency** | Saga, database isolation, GDPR | Database-per-Service, CQRS | Durable Functions, Cosmos DB |

---

# Microservices Fundamentals & Architecture: Service Boundaries Definition (Continued)

---

## Question 5: How Do You Handle Cross-Cutting Concerns (Logging, Monitoring, Security) Across Service Boundaries Without Creating Hidden Dependencies?

### Detailed Answer:

Cross-cutting concerns—logging, monitoring, authentication, authorization, rate limiting, and tracing—span all services. If implemented inconsistently or tightly coupled to business logic, they undermine service autonomy. The key is **separating concerns via infrastructure, not code**. In Azure, use **managed services and middleware patterns** to enforce cross-cutting concerns uniformly across all boundaries without services knowing about each other.

**Distributed tracing** is critical in microservices. When a user request flows through Order → Inventory → Payment services, you need end-to-end visibility. Azure Application Insights provides **automatic instrumentation** via Azure Monitor's OpenTelemetry support. Each service propagates a **correlation ID** (trace ID) through HTTP headers (`traceparent` header per W3C Trace Context standard) and message metadata. Services don't explicitly depend on each other for tracing—the infrastructure (Application Insights SDK) automatically captures the flow. This respects boundaries: Order service doesn't know it calls Inventory; the tracing layer observes and records the interaction.

**Authentication and authorization** must be centralized to prevent duplicate logic across services. Use **Azure AD/Microsoft Entra ID** as the identity provider and **Azure API Management** as the policy enforcement point. APIM validates tokens, enforces scopes, and translates identities without services implementing security logic directly. This is crucial: services shouldn't authenticate every incoming request themselves; the API gateway handles it, allowing services to trust authenticated claims. For service-to-service calls, use **managed identities** (Azure VMs, Functions, App Services have built-in identities) rather than shared secrets scattered across services.

**Observability** without tight coupling requires standardized contracts for logs and metrics. All services emit logs to a centralized Log Analytics workspace (via Application Insights). Use structured logging (JSON format) with consistent fields: `timestamp`, `service_name`, `trace_id`, `correlation_id`, `severity`, `message`. Avoid service-specific logging logic; instead, use middleware and decorators to inject context. Similarly, emit metrics (counters, gauges, histograms) to Application Insights using standard naming conventions (e.g., `order_create_duration_ms`, `inventory_reserve_success_rate`).

### Explanation:

**Importance & Use Cases:**
- **Observability at scale**: With 20+ services, centralized logging/monitoring prevents debugging chaos.
- **Security posture**: Centralized auth prevents security gaps where one service bypasses auth.
- **Operational consistency**: Standard tracing/logging makes incident response faster and more reliable.
- **Compliance**: Audit trails and data classification consistent across all services.

**Benefits & Trade-offs:**
- **Benefit**: Reduced code duplication, consistent security, faster debugging, compliance-ready.
- **Trade-off**: Dependency on central infrastructure (if Application Insights is down, observability is limited); slight latency overhead from instrumentation.
- **Interview perspective**: Demonstrates understanding of operational excellence and security in distributed systems beyond just architecture.

### C# Implementation:

```csharp
using Azure.Identity;
using Azure.Monitor.OpenTelemetry;
using Microsoft.ApplicationInsights;
using Microsoft.ApplicationInsights.DataContracts;
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.AspNetCore.Authorization;
using Microsoft.Identity.Web;
using OpenTelemetry;
using OpenTelemetry.Trace;
using System.Diagnostics;

// ============ DISTRIBUTED TRACING & CORRELATION ============

/// <summary>
/// Middleware to propagate correlation IDs across service boundaries
/// Implements W3C Trace Context standard (traceparent header)
/// </summary>
public class CorrelationIdMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<CorrelationIdMiddleware> _logger;

    public CorrelationIdMiddleware(RequestDelegate next, ILogger<CorrelationIdMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        // Extract or generate correlation ID from W3C traceparent header
        string correlationId = context.Request.Headers.TryGetValue("traceparent", out var traceParent)
            ? traceParent.ToString()
            : Activity.Current?.Id ?? context.TraceIdentifier;

        // Make available throughout request pipeline
        context.Items["CorrelationId"] = correlationId;

        // Ensure response also includes correlation ID for client-side debugging
        context.Response.Headers.Add("X-Correlation-ID", correlationId);

        _logger.LogInformation(
            "Incoming request: {Method} {Path} | Correlation ID: {CorrelationId}",
            context.Request.Method,
            context.Request.Path,
            correlationId);

        await _next(context);
    }
}

/// <summary>
/// HTTP client factory that automatically propagates correlation IDs
/// Prevents services from explicitly managing trace context
/// </summary>
public class CorrelationIdHttpClientHandler : DelegatingHandler
{
    private readonly IHttpContextAccessor _httpContextAccessor;

    public CorrelationIdHttpClientHandler(IHttpContextAccessor httpContextAccessor)
    {
        _httpContextAccessor = httpContextAccessor;
    }

    protected override async Task<HttpResponseMessage> SendAsync(
        HttpRequestMessage request,
        CancellationToken cancellationToken)
    {
        // Automatically add correlation ID to outgoing requests
        if (_httpContextAccessor.HttpContext?.Items.TryGetValue("CorrelationId", out var correlationId) == true)
        {
            request.Headers.Add("traceparent", correlationId.ToString());
            request.Headers.Add("X-Correlation-ID", correlationId.ToString());
        }

        return await base.SendAsync(request, cancellationToken);
    }
}

// ============ STRUCTURED LOGGING WITH CONTEXT ============

/// <summary>
/// Standardized log context enricher for all services
/// Injects service name, environment, trace ID consistently
/// </summary>
public class LogEnricher
{
    private readonly string _serviceName;
    private readonly string _environment;

    public LogEnricher(string serviceName, string environment)
    {
        _serviceName = serviceName;
        _environment = environment;
    }

    public void EnrichLog(
        LogLevel level,
        string message,
        ILogger logger,
        string? correlationId = null,
        Dictionary<string, object>? customProperties = null)
    {
        // Structured logging with consistent schema
        var logEntry = new Dictionary<string, object>
        {
            { "timestamp", DateTime.UtcNow },
            { "service_name", _serviceName },
            { "environment", _environment },
            { "level", level.ToString() },
            { "message", message },
            { "trace_id", Activity.Current?.Id ?? correlationId ?? "unknown" },
            { "correlation_id", correlationId ?? "unknown" }
        };

        if (customProperties != null)
        {
            foreach (var prop in customProperties)
            {
                logEntry[prop.Key] = prop.Value;
            }
        }

        // Log as structured JSON for Log Analytics parsing
        logger.LogInformation(
            "Log Entry: {@LogEntry}",
            logEntry);
    }
}

// ============ CENTRALIZED AUTHENTICATION & AUTHORIZATION ============

/// <summary>
/// Extension method to configure Azure AD authentication uniformly across all services
/// Prevents duplicate auth logic in each service
/// </summary>
public static class AuthenticationExtensions
{
    public static IServiceCollection AddCentralizedAuthentication(
        this IServiceCollection services,
        IConfiguration configuration)
    {
        services
            .AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
            .AddMicrosoftIdentityWebApi(configuration.GetSection("AzureAd"));

        // Add authorization with role-based access control
        services.AddAuthorizationBuilder()
            .AddPolicy("OrderManager", policy =>
                policy.RequireRole("Order.Manager").RequireScope("api://orders/manage"))
            .AddPolicy("InventoryViewer", policy =>
                policy.RequireRole("Inventory.Viewer").RequireScope("api://inventory/read"));

        return services;
    }
}

/// <summary>
/// Authorization policy for service-to-service calls using managed identities
/// No shared secrets needed—Azure AD handles identity verification
/// </summary>
public class ManagedIdentityAuthHandler
{
    public static async Task<string> GetServiceTokenAsync(
        string targetResourceId)
    {
        // Automatic token acquisition using managed identity
        var credential = new DefaultAzureCredential();
        var token = await credential.GetTokenAsync(
            new Azure.Core.TokenRequestContext(new[] { $"{targetResourceId}/.default" }));

        return token.Token;
    }
}

// ============ SERVICE-TO-SERVICE COMMUNICATION WITH MANAGED IDENTITIES ============

public class OrderServiceClient
{
    private readonly HttpClient _httpClient;
    private readonly ILogger<OrderServiceClient> _logger;

    public OrderServiceClient(HttpClient httpClient, ILogger<OrderServiceClient> logger)
    {
        _httpClient = httpClient;
        _logger = logger;
    }

    public async Task<InventoryResponse> CheckInventoryAsync(string orderId)
    {
        try
        {
            // Acquire managed identity token automatically
            var credential = new DefaultAzureCredential();
            var token = await credential.GetTokenAsync(
                new Azure.Core.TokenRequestContext(
                    new[] { "https://inventory-service/.default" }));

            // No username/password—identity verified via Azure AD
            _httpClient.DefaultRequestHeaders.Authorization =
                new System.Net.Http.Headers.AuthenticationHeaderValue("Bearer", token.Token);

            var response = await _httpClient.PostAsync(
                "https://inventory-service.azurewebsites.net/api/reserve",
                new StringContent(
                    System.Text.Json.JsonSerializer.Serialize(new { OrderId = orderId }),
                    System.Text.Encoding.UTF8,
                    "application/json"));

            _logger.LogInformation(
                "Inventory service call succeeded: Order {OrderId}",
                orderId);

            return await response.Content.ReadAsAsync<InventoryResponse>();
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Inventory service call failed for order {OrderId}", orderId);
            throw;
        }
    }
}

// ============ CENTRALIZED METRICS & OBSERVABILITY ============

/// <summary>
/// Decorator pattern for capturing metrics without modifying service logic
/// Cross-cutting concern isolated from business code
/// </summary>
public class MetricsInterceptor<T> : IAsyncInterceptor where T : class
{
    private readonly TelemetryClient _telemetryClient;
    private readonly ILogger<MetricsInterceptor<T>> _logger;

    public MetricsInterceptor(
        TelemetryClient telemetryClient,
        ILogger<MetricsInterceptor<T>> logger)
    {
        _telemetryClient = telemetryClient;
        _logger = logger;
    }

    public void InterceptSynchronous(
        IInvocation invocation)
    {
        ExecuteWithMetrics(invocation);
    }

    public void InterceptAsynchronous(IInvocation invocation)
    {
        invocation.ReturnValue = InterceptAsynchronousCore(invocation);
    }

    public void InterceptAsynchronous<TResult>(IInvocation invocation)
    {
        invocation.ReturnValue = InterceptAsynchronousCore<TResult>(invocation);
    }

    private void ExecuteWithMetrics(IInvocation invocation)
    {
        var stopwatch = Stopwatch.StartNew();
        var methodName = $"{typeof(T).Name}.{invocation.Method.Name}";

        try
        {
            invocation.Proceed();
            stopwatch.Stop();

            // Track successful call duration
            _telemetryClient.TrackEvent("MethodSuccess", new Dictionary<string, string>
            {
                { "method", methodName },
                { "duration_ms", stopwatch.ElapsedMilliseconds.ToString() }
            });

            _logger.LogInformation(
                "Method {Method} completed in {DurationMs}ms",
                methodName,
                stopwatch.ElapsedMilliseconds);
        }
        catch (Exception ex)
        {
            stopwatch.Stop();

            // Track failed call
            _telemetryClient.TrackException(ex, new Dictionary<string, string>
            {
                { "method", methodName },
                { "duration_ms", stopwatch.ElapsedMilliseconds.ToString() }
            });

            _logger.LogError(ex, "Method {Method} failed after {DurationMs}ms", methodName, stopwatch.ElapsedMilliseconds);
            throw;
        }
    }

    private async Task InterceptAsynchronousCore(IInvocation invocation)
    {
        var stopwatch = Stopwatch.StartNew();
        var methodName = $"{typeof(T).Name}.{invocation.Method.Name}";

        try
        {
            invocation.Proceed();
            var task = (Task)invocation.ReturnValue;
            await task.ConfigureAwait(false);
            stopwatch.Stop();

            _telemetryClient.TrackEvent("AsyncMethodSuccess", new Dictionary<string, string>
            {
                { "method", methodName },
                { "duration_ms", stopwatch.ElapsedMilliseconds.ToString() }
            });
        }
        catch (Exception ex)
        {
            stopwatch.Stop();
            _telemetryClient.TrackException(ex);
            throw;
        }
    }

    private async Task<TResult> InterceptAsynchronousCore<TResult>(IInvocation invocation)
    {
        var stopwatch = Stopwatch.StartNew();
        var methodName = $"{typeof(T).Name}.{invocation.Method.Name}";

        try
        {
            invocation.Proceed();
            var task = (Task<TResult>)invocation.ReturnValue;
            var result = await task.ConfigureAwait(false);
            stopwatch.Stop();

            _telemetryClient.TrackEvent("AsyncMethodSuccess", new Dictionary<string, string>
            {
                { "method", methodName },
                { "duration_ms", stopwatch.ElapsedMilliseconds.ToString() }
            });

            return result;
        }
        catch (Exception ex)
        {
            stopwatch.Stop();
            _telemetryClient.TrackException(ex);
            throw;
        }
    }
}

// ============ STARTUP CONFIGURATION ============

public static class ServiceCollectionExtensions
{
    public static IServiceCollection AddCrossCuttingConcerns(
        this IServiceCollection services,
        IConfiguration configuration,
        string serviceName)
    {
        // Centralized authentication (Azure AD)
        services.AddCentralizedAuthentication(configuration);

        // Distributed tracing via Application Insights
        services.AddApplicationInsightsTelemetry();
        services.AddOpenTelemetry()
            .WithTracing(builder => builder
                .AddAspNetCoreInstrumentation()
                .AddHttpClientInstrumentation()
                .AddAzureMonitorTraceExporter());

        // Structured logging
        services.AddLogging(builder => builder
            .ClearProviders()
            .AddApplicationInsights()
            .AddConsole());

        // Correlation ID middleware
        services.AddScoped<CorrelationIdMiddleware>();
        services.AddScoped<IHttpContextAccessor, HttpContextAccessor>();
        services.AddHttpClient<OrderServiceClient>()
            .AddHttpMessageHandler<CorrelationIdHttpClientHandler>();

        // Metrics interceptor
        services.AddScoped(typeof(MetricsInterceptor<>));

        // Log enricher
        services.AddSingleton(_ => new LogEnricher(serviceName, "Production"));

        return services;
    }
}

// ============ CONTROLLER EXAMPLE ============

[ApiController]
[Route("api/v1/orders")]
[Authorize(Policy = "OrderManager")]
public class OrdersController : ControllerBase
{
    private readonly OrderService _orderService;
    private readonly LogEnricher _logEnricher;
    private readonly ILogger<OrdersController> _logger;

    public OrdersController(
        OrderService orderService,
        LogEnricher logEnricher,
        ILogger<OrdersController> logger)
    {
        _orderService = orderService;
        _logEnricher = logEnricher;
        _logger = logger;
    }

    [HttpPost]
    public async Task<IActionResult> CreateOrderAsync([FromBody] CreateOrderRequest request)
    {
        var correlationId = HttpContext.Items["CorrelationId"]?.ToString();

        // Cross-cutting concerns handled transparently
        try
        {
            _logEnricher.EnrichLog(
                LogLevel.Information,
                "Creating order",
                _logger,
                correlationId,
                new Dictionary<string, object>
                {
                    { "customer_id", request.CustomerId },
                    { "total_amount", request.TotalAmount }
                });

            var order = await _orderService.CreateOrderAsync(request);

            // User identity automatically available from Azure AD
            var userId = User.FindFirst("sub")?.Value ?? "unknown";
            _logEnricher.EnrichLog(
                LogLevel.Information,
                "Order created successfully",
                _logger,
                correlationId,
                new Dictionary<string, object>
                {
                    { "order_id", order.OrderId },
                    { "created_by", userId }
                });

            return CreatedAtAction(nameof(GetOrderAsync), new { id = order.OrderId }, order);
        }
        catch (Exception ex)
        {
            _logEnricher.EnrichLog(
                LogLevel.Error,
                "Failed to create order",
                _logger,
                correlationId,
                new Dictionary<string, object>
                {
                    { "error", ex.Message }
                });
            throw;
        }
    }

    [HttpGet("{id}")]
    [Authorize(Policy = "InventoryViewer")]
    public async Task<ActionResult<Order>> GetOrderAsync(string id)
    {
        var order = await _orderService.GetOrderAsync(id);
        if (order == null)
            return NotFound();

        return Ok(order);
    }
}

// ============ MODELS ============

public record CreateOrderRequest(string CustomerId, decimal TotalAmount);
public record Order(string OrderId, string CustomerId, decimal TotalAmount);
public record InventoryResponse(bool Available, int QuantityOnHand);
```

### Follow-Up Questions:

**Q1: How do you implement rate limiting and circuit breakers across service boundaries without tight coupling?**
* **Answer:** Use **Azure API Management (APIM)** policies to enforce rate limiting centrally—no code changes needed in services. Configure rate limit policies (e.g., "max 1000 requests/minute per API key") in APIM; requests exceeding limits are rejected before reaching services. For circuit breaking, use **Polly library** (resilience library for .NET) with Application Insights integration. When a service dependency fails repeatedly, Polly automatically stops sending requests for a timeout window, fast-failing subsequent requests. Polly policies are configured per HttpClient (e.g., retry 3 times, then circuit break), not mixed into business logic. This respects boundaries: services don't know about circuit breakers; resilience is infrastructure-level.

**Q2: How do you correlate logs across services when requests span multiple deployments (VMs, AKS, Functions)?**
* **Answer:** **W3C Trace Context standard** ensures correlation regardless of deployment model. Every service propagates the `traceparent` header (format: `00-trace-id-parent-id-trace-flags`). Application Insights automatically indexes logs by trace_id, allowing end-to-end visibility in Azure Portal under "Transaction Search" or "Application Map." For custom correlation, extract trace_id from baggage headers and store in Log Analytics. If services run in AKS, enable Application Insights Profiler for runtime insights. For serverless (Functions), managed instrumentation captures correlation automatically. Use Application Insights' dependency map to visualize service interactions—if Order → Inventory call fails, dependency map highlights the failure path.

**Q3: What's the difference between authentication and authorization, and how do they apply across service boundaries?**
* **Answer:** **Authentication** verifies identity ("who are you?"); **authorization** verifies permissions ("what can you do?"). In Azure AD, authentication happens at API Gateway (APIM) where JWT tokens are validated. Authorization happens in two places: (1) APIM policies enforce scopes (e.g., "api://orders/manage"), and (2) services validate role claims in JWT (`roles` claim). Services don't need to validate tokens (gateway did it); they trust the token and extract claims. For service-to-service calls, use managed identities instead of secrets—Azure AD handles authentication transparently. Authorization rules (which service can call which service) are enforced via API Management policies or network policies (Service-to-Service Networking in AKS), not hard-coded in services.

**Q4: How do you ensure sensitive data (API keys, connection strings) isn't leaked in logs across service boundaries?**
* **Answer:** Use **Azure Key Vault** for secrets management—no secrets stored in code or configs. Application Insights has built-in data masking for common sensitive patterns (credit cards, SSNs), but don't rely on it. Instead, implement **secret scrubbing middleware** that sanitizes logs before sending to Application Insights—mask parameters like `apiKey`, `password`, `token`. Use Application Insights' sampling to reduce cost while maintaining observability; sampled telemetry shouldn't contain raw data anyway. For compliance (PCI-DSS, HIPAA), enable data retention limits in Log Analytics (default 30 days, configurable). Never log request/response bodies containing PII; log only headers and result codes. For debugging sensitive flows, use Application Insights' Snapshot Debugger on pre-prod only.

**Q5: How do you monitor service boundaries health without creating dependency alerts that fire constantly?**
* **Answer:** Implement **smart alerting** in Application Insights: alert on error rate >5% sustained for 10 minutes (not instant), not on every failure. Use **dynamic thresholds** which learn normal patterns and alert only on anomalies. Create alerts by service boundary: "Inventory service latency spike," "Order-to-Inventory call failure rate >10%." Avoid alerting on individual timeouts (transient); instead, alert if timeout rate exceeds SLA (e.g., p99 latency >2 seconds). Use Azure Monitor's multi-resource rules to correlate failures across services—if Order and Inventory both spike, it might be shared infrastructure (database) failing, not service-specific. Set up actionable runbooks: when alert fires, provide remediation steps. Use PagerDuty or Azure Incident Management to route critical alerts, avoiding alert fatigue.

---

## Question 6: What Patterns and Tools Help You Identify When Service Boundaries Need Refactoring or Reorganization?

### Detailed Answer:

Service boundaries are not static. As business requirements evolve, team structures change, and deployment patterns shift, boundaries that were optimal become sub-optimal. Identifying when and how to refactor boundaries requires both quantitative metrics and qualitative signals. **Quantitative signals** come from observability: if Service A consistently makes 90% of its calls to Service B, they're likely too fine-grained and should be merged. If a "simple" order creation request spans 8 service calls with 500ms+ latency, boundaries are too fragmented. **Qualitative signals** include team complaints ("we can't deploy without coordinating with three teams"), frequent cross-team PRs for dependency updates, and circular dependencies in service call graphs.

**Conway's Law** states that system architecture mirrors organizational structure. If your org restructures but service boundaries don't, friction increases. Conversely, if teams consistently collaborate on two services, consider aligning those services to team boundaries. Tools like **Azure Application Insights Dependency Map** visualize service interactions; clustering services in the map shows collaboration patterns. **Azure DevOps Analytics** reveals deployment frequency, cycle time, and team ownership—compare these with service boundaries to identify misalignments.

**Refactoring strategies** depend on the issue. If services are too fine-grained, **consolidate** them (merge databases, combine APIs). If a service has conflicting scaling requirements (some endpoints scale differently), **split** it using the Strangler Pattern. If services have circular dependencies, **introduce an intermediary service** that both depend on (breaking the cycle). Azure supports all these patterns: use **Azure Container Registry** to manage multiple images during refactoring, **Azure Traffic Manager** or **API Management** to route traffic during transitions, and **Azure Data Factory** for data migration during consolidation.

### Explanation:

**Importance & Use Cases:**
- **Prevents technical debt**: Outdated boundaries create unnecessary complexity and slow deployments.
- **Improves team autonomy**: Boundaries aligned with org structure reduce coordination overhead.
- **Optimizes performance**: Reducing inter-service calls improves latency and reduces infrastructure costs.
- **Enables scaling**: Refactored boundaries can scale independently per business need.

**Benefits & Trade-offs:**
- **Benefit**: Faster feature delivery, reduced operational complexity, improved team velocity.
- **Trade-off**: Refactoring distributed systems is risky; requires comprehensive testing and potentially downtime (or complex blue-green deployments).
- **Interview perspective**: Shows understanding of living architecture, not one-time design; demonstrates ability to balance theory with pragmatic evolution.

### C# Implementation:

```csharp
using Azure.Data.Tables;
using Azure.Monitor.OpenTelemetry;
using Microsoft.ApplicationInsights;
using Microsoft.ApplicationInsights.DataContracts;
using System.Collections.Generic;
using System.Diagnostics;
using System.Linq;

// ============ SERVICE BOUNDARY HEALTH ANALYZER ============

/// <summary>
/// Analyzes service interactions via Application Insights to detect boundary issues
/// Identifies when services should be merged, split, or reorganized
/// </summary>
public class ServiceBoundaryHealthAnalyzer
{
    private readonly TelemetryClient _telemetryClient;
    private readonly TableClient _metricsTableClient;
    private readonly ILogger<ServiceBoundaryHealthAnalyzer> _logger;

    public ServiceBoundaryHealthAnalyzer(
        TelemetryClient telemetryClient,
        TableClient metricsTableClient,
        ILogger<ServiceBoundaryHealthAnalyzer> logger)
    {
        _telemetryClient = telemetryClient;
        _metricsTableClient = metricsTableClient;
        _logger = logger;
    }

    /// <summary>
    /// Metric 1: High inter-service call frequency indicates over-fragmentation
    /// </summary>
    public async Task<ServiceDependencyMetrics> AnalyzeDependencyIntensityAsync(
        string sourceService,
        string targetService,
        TimeSpan analysisWindow)
    {
        // Query Application Insights for dependency calls
        var callCount = await GetDependencyCallCountAsync(
            sourceService,
            targetService,
            analysisWindow);

        var totalRequests = await GetTotalRequestsAsync(sourceService, analysisWindow);
        var dependencyRatio = totalRequests > 0
            ? (double)callCount / totalRequests
            : 0;

        var avgLatency = await GetAverageDependencyLatencyAsync(
            sourceService,
            targetService,
            analysisWindow);

        var failureRate = await GetDependencyFailureRateAsync(
            sourceService,
            targetService,
            analysisWindow);

        var metrics = new ServiceDependencyMetrics
        {
            SourceService = sourceService,
            TargetService = targetService,
            CallCount = callCount,
            DependencyRatio = dependencyRatio,
            AverageLatencyMs = avgLatency,
            FailureRate = failureRate,
            AnalysisTimestamp = DateTime.UtcNow
        };

        // Alert if dependency ratio > 70% (indicator of poor boundary)
        if (dependencyRatio > 0.7)
        {
            _logger.LogWarning(
                "⚠️ High dependency ratio: {SourceService} depends on {TargetService} {Ratio:P}. " +
                "Consider consolidation.",
                sourceService,
                targetService,
                dependencyRatio);

            _telemetryClient.TrackEvent("BoundaryHealthIssue", new Dictionary<string, string>
            {
                { "issue_type", "high_dependency_ratio" },
                { "source_service", sourceService },
                { "target_service", targetService },
                { "ratio", dependencyRatio.ToString("P") }
            });
        }

        // Alert if average latency > 500ms (performance impact)
        if (avgLatency > 500)
        {
            _logger.LogWarning(
                "⚠️ High latency: {SourceService} → {TargetService} ({LatencyMs}ms). " +
                "Consider merging or optimizing communication.",
                sourceService,
                targetService,
                avgLatency);
        }

        await _metricsTableClient.UpsertEntityAsync(
            new ServiceDependencyMetricsEntity(metrics));

        return metrics;
    }

    /// <summary>
    /// Metric 2: Circular dependencies indicate boundary design problems
    /// </summary>
    public async Task<CircularDependencyReport> DetectCircularDependenciesAsync()
    {
        var dependencies = await GetServiceDependencyGraphAsync();
        var cycles = FindCycles(dependencies);

        if (cycles.Any())
        {
            _logger.LogError(
                "❌ Circular dependencies detected: {Cycles}",
                string.Join(", ", cycles.Select(c => string.Join("→", c))));

            foreach (var cycle in cycles)
            {
                _telemetryClient.TrackEvent("CircularDependencyDetected", new Dictionary<string, string>
                {
                    { "cycle", string.Join("→", cycle) }
                });
            }
        }

        return new CircularDependencyReport
        {
            CirclesDetected = cycles,
            AnalysisTimestamp = DateTime.UtcNow
        };
    }

    /// <summary>
    /// Metric 3: Deployment coupling - frequent coordinated deployments indicate tight coupling
    /// </summary>
    public async Task<DeploymentCouplingReport> AnalyzeDeploymentCouplingAsync(TimeSpan window)
    {
        // Query Azure DevOps or deployment logs for deployment times
        var deploymentEvents = await GetDeploymentEventsAsync(window);

        // Group deployments by timestamp proximity (within 1 hour)
        var coupledDeployments = deploymentEvents
            .GroupBy(d => d.Timestamp.Date)
            .Where(g => g.Count() > 1)
            .SelectMany(g => g)
            .ToList();

        var couplingPercentage = coupledDeployments.Count > 0
            ? (double)coupledDeployments.Count / deploymentEvents.Count
            : 0;

        if (couplingPercentage > 0.3) // >30% coordinated deployments
        {
            _logger.LogWarning(
                "⚠️ High deployment coupling: {CouplingPercentage:P} of deployments are coordinated. " +
                "Consider boundary realignment.",
                couplingPercentage);

            _telemetryClient.TrackEvent("HighDeploymentCoupling", new Dictionary<string, string>
            {
                { "coupling_percentage", couplingPercentage.ToString("P") }
            });
        }

        return new DeploymentCouplingReport
        {
            CoordinatedDeploymentPercentage = couplingPercentage,
            AnalysisTimestamp = DateTime.UtcNow
        };
    }

    /// <summary>
    /// Metric 4: Scaling patterns - independent scaling needs suggest boundary split
    /// </summary>
    public async Task<ScalingAnalysisReport> AnalyzeScalingPatternsAsync(
        string serviceName,
        TimeSpan window)
    {
        var cpuUsage = await GetMetricAsync($"{serviceName}_cpu_usage", window);
        var memoryUsage = await GetMetricAsync($"{serviceName}_memory_usage", window);
        var requestsPerSecond = await GetMetricAsync($"{serviceName}_requests_per_second", window);

        // Calculate coefficient of variation for each metric
        var cpuVariation = CalculateCoefficientOfVariation(cpuUsage);
        var memoryVariation = CalculateCoefficientOfVariation(memoryUsage);
        var requestsVariation = CalculateCoefficientOfVariation(requestsPerSecond);

        // If one endpoint is bursty while others are stable, split might be beneficial
        var isScalingUneven = cpuVariation > 0.5 && memoryVariation < 0.3; // Conflicting patterns
        if (isScalingUneven)
        {
            _logger.LogWarning(
                "⚠️ Uneven scaling patterns in {ServiceName}: " +
                "CPU varies {CpuVar:P}, Memory varies {MemVar:P}. " +
                "Consider splitting service.",
                serviceName,
                cpuVariation,
                memoryVariation);
        }

        return new ScalingAnalysisReport
        {
            ServiceName = serviceName,
            CpuVariation = cpuVariation,
            MemoryVariation = memoryVariation,
            RequestsVariation = requestsVariation,
            RecommendedAction = isScalingUneven ? "Consider service split" : "Current boundaries acceptable",
            AnalysisTimestamp = DateTime.UtcNow
        };
    }

    /// <summary>
    /// Generate consolidated health report
    /// </summary>
    public async Task<ServiceBoundaryHealthReport> GenerateHealthReportAsync()
    {
        var dependencyIssues = new List<ServiceDependencyMetrics>();
        var circularDependencies = await DetectCircularDependenciesAsync();
        var deploymentCoupling = await AnalyzeDeploymentCouplingAsync(TimeSpan.FromDays(30));

        var report = new ServiceBoundaryHealthReport
        {
            CircularDependencies = circularDependencies,
            DeploymentCoupling = deploymentCoupling,
            HighRiskBoundaries = dependencyIssues.Where(d => d.DependencyRatio > 0.7).ToList(),
            PerformanceHotspots = dependencyIssues.Where(d => d.AverageLatencyMs > 500).ToList(),
            RecommendedRefactorings = GenerateRefactoringRecommendations(
                circularDependencies,
                deploymentCoupling,
                dependencyIssues),
            GeneratedAt = DateTime.UtcNow
        };

        return report;
    }

    // ============ HELPER METHODS ============

    private async Task<int> GetDependencyCallCountAsync(
        string source,
        string target,
        TimeSpan window)
    {
        // Mock implementation - query Application Insights in production
        await Task.Delay(10);
        return 1500; // Example: 1500 calls in window
    }

    private async Task<int> GetTotalRequestsAsync(string service, TimeSpan window)
    {
        await Task.Delay(10);
        return 2000;
    }

    private async Task<double> GetAverageDependencyLatencyAsync(
        string source,
        string target,
        TimeSpan window)
    {
        await Task.Delay(10);
        return 450; // milliseconds
    }

    private async Task<double> GetDependencyFailureRateAsync(
        string source,
        string target,
        TimeSpan window)
    {
        await Task.Delay(10);
        return 0.02; // 2% failure rate
    }

    private async Task<Dictionary<string, List<string>>> GetServiceDependencyGraphAsync()
    {
        await Task.Delay(10);
        return new Dictionary<string, List<string>>
        {
            { "OrderService", new List<string> { "InventoryService", "PaymentService" } },
            { "InventoryService", new List<string> { "ProductService" } },
            { "PaymentService", new List<string> { "OrderService" } } // Circular!
        };
    }

    private List<List<string>> FindCycles(Dictionary<string, List<string>> graph)
    {
        var cycles = new List<List<string>>();
        var visited = new HashSet<string>();
        var recursionStack = new HashSet<string>();
        var path = new Stack<string>();

        foreach (var node in graph.Keys)
        {
            if (!visited.Contains(node))
            {
                DfsForCycles(node, graph, visited, recursionStack, path, cycles);
            }
        }

        return cycles;
    }

    private void DfsForCycles(
        string node,
        Dictionary<string, List<string>> graph,
        HashSet<string> visited,
        HashSet<string> recursionStack,
        Stack<string> path,
        List<List<string>> cycles)
    {
        visited.Add(node);
        recursionStack.Add(node);
        path.Push(node);

        if (graph.TryGetValue(node, out var neighbors))
        {
            foreach (var neighbor in neighbors)
            {
                if (!visited.Contains(neighbor))
                {
                    DfsForCycles(neighbor, graph, visited, recursionStack, path, cycles);
                }
                else if (recursionStack.Contains(neighbor))
                {
                    // Cycle found
                    var cycleList = new List<string>();
                    foreach (var item in path)
                    {
                        cycleList.Add(item);
                        if (item == neighbor) break;
                    }
                    cycles.Add(cycleList);
                }
            }
        }

        path.Pop();
        recursionStack.Remove(node);
    }

    private async Task<List<DeploymentEvent>> GetDeploymentEventsAsync(TimeSpan window)
    {
        await Task.Delay(10);
        return new List<DeploymentEvent>
        {
            new DeploymentEvent { Service = "OrderService", Timestamp = DateTime.UtcNow },
            new DeploymentEvent { Service = "InventoryService", Timestamp = DateTime.UtcNow.AddMinutes(5) }
        };
    }

    private async Task<List<double>> GetMetricAsync(string metricName, TimeSpan window)
    {
        await Task.Delay(10);
        var random = new Random();
        return Enumerable.Range(0, 24)
            .Select(_ => random.NextDouble() * 100)
            .ToList();
    }

    private double CalculateCoefficientOfVariation(List<double> values)
    {
        if (values.Count == 0) return 0;
        var mean = values.Average();
        var variance = values.Average(v => Math.Pow(v - mean, 2));
        var stdDev = Math.Sqrt(variance);
        return mean > 0 ? stdDev / mean : 0;
    }

    private List<string> GenerateRefactoringRecommendations(
        CircularDependencyReport circular,
        DeploymentCouplingReport deployment,
        List<ServiceDependencyMetrics> dependencies)
    {
        var recommendations = new List<string>();

        if (circular.CirclesDetected.Any())
        {
            recommendations.Add("🔴 CRITICAL: Resolve circular dependencies by introducing intermediary service");
        }

        if (deployment.CoordinatedDeploymentPercentage > 0.3)
        {
            recommendations.Add("🟡 WARNING: High deployment coupling suggests boundary misalignment");
        }

        var problematicDependencies = dependencies.Where(d => d.DependencyRatio > 0.7);
        foreach (var dep in problematicDependencies)
        {
            recommendations.Add(
                $"🟠 Consider merging {dep.SourceService} and {dep.TargetService} " +
                $"({dep.DependencyRatio:P} call ratio)");
        }

        return recommendations;
    }
}

// ============ DATA MODELS ============

public record ServiceDependencyMetrics
{
    public string SourceService { get; set; }
    public string TargetService { get; set; }
    public int CallCount { get; set; }
    public double DependencyRatio { get; set; }
    public double AverageLatencyMs { get; set; }
    public double FailureRate { get; set; }
    public DateTime AnalysisTimestamp { get; set; }
}

public record CircularDependencyReport
{
    public List<List<string>> CirclesDetected { get; set; }
    public DateTime AnalysisTimestamp { get; set; }
}

public record DeploymentCouplingReport
{
    public double CoordinatedDeploymentPercentage { get; set; }
    public DateTime AnalysisTimestamp { get; set; }
}

public record ScalingAnalysisReport
{
    public string ServiceName { get; set; }
    public double CpuVariation { get; set; }
    public double MemoryVariation { get; set; }
    public double RequestsVariation { get; set; }
    public string RecommendedAction { get; set; }
    public DateTime AnalysisTimestamp { get; set; }
}

public record ServiceBoundaryHealthReport
{
    public CircularDependencyReport CircularDependencies { get; set; }
    public DeploymentCouplingReport DeploymentCoupling { get; set; }
    public List<ServiceDependencyMetrics> HighRiskBoundaries { get; set; }
    public List<ServiceDependencyMetrics> PerformanceHotspots { get; set; }
    public List<string> RecommendedRefactorings { get; set; }
    public DateTime GeneratedAt { get; set; }
}

public record DeploymentEvent
{
    public string Service { get; set; }
    public DateTime Timestamp { get; set; }
}

public class ServiceDependencyMetricsEntity : ITableEntity
{
    public ServiceDependencyMetricsEntity() { }

    public ServiceDependencyMetricsEntity(ServiceDependencyMetrics metrics)
    {
        PartitionKey = metrics.SourceService;
        RowKey = $"{metrics.TargetService}_{metrics.AnalysisTimestamp:yyyyMMddHHmmss}";
        DependencyRatio = metrics.DependencyRatio;
        AverageLatencyMs = metrics.AverageLatencyMs;
        FailureRate = metrics.FailureRate;
        CallCount = metrics.CallCount;
    }

    public string PartitionKey { get; set; }
    public string RowKey { get; set; }
    public DateTimeOffset? Timestamp { get; set; }
    public ETag ETag { get; set; }

    public double DependencyRatio { get; set; }
    public double AverageLatencyMs { get; set; }
    public double FailureRate { get; set; }
    public int CallCount { get; set; }
}

// ============ REFACTORING EXECUTION HELPER ============

public class ServiceBoundaryRefactoringExecutor
{
    private readonly ILogger<ServiceBoundaryRefactoringExecutor> _logger;

    public ServiceBoundaryRefactoringExecutor(ILogger<ServiceBoundaryRefactoringExecutor> logger)
    {
        _logger = logger;
    }

    /// <summary>
    /// Execute Strangler Pattern refactoring to merge two services
    /// </summary>
    public async Task ExecuteMergeRefactoringAsync(
        string service1,
        string service2,
        string trafficPercentageToNewService)
    {
        _logger.LogInformation(
            "Starting merge refactoring: {Service1} + {Service2}",
            service1,
            service2);

        // Phase 1: Deploy combined service alongside originals
        _logger.LogInformation("Phase 1: Deploying combined service");

        // Phase 2: Gradually shift traffic using feature flags
        _logger.LogInformation(
            "Phase 2: Shifting {Percentage}% traffic to combined service",
            trafficPercentageToNewService);

        // Phase 3: Monitor for issues
        _logger.LogInformation("Phase 3: Monitoring metrics...");

        // Phase 4: Retire original services
        _logger.LogInformation("Phase 4: Retiring original services");
    }

    /// <summary>
    /// Execute split refactoring to separate conflicting concerns
    /// </summary>
    public async Task ExecuteSplitRefactoringAsync(
        string originalService,
        string newService1,
        string newService2)
    {
        _logger.LogInformation(
            "Starting split refactoring: {OriginalService} → {NewService1} + {NewService2}",
            originalService,
            newService1,
            newService2);

        // Similar phased approach: deploy, route, monitor, retire
    }
}
```

### Follow-Up Questions:

**Q1: How do you safely execute a boundary refactoring (merge/split) without downtime?**
* **Answer:** Use the **Strangler Pattern with feature flags and gradual traffic shifting**. Deploy the new service boundary alongside existing services (no immediate cutover). Use Azure API Management or App Gateway to route percentage-based traffic: 5% to new service, 95% to old initially. Monitor error rates and latency with Application Insights; if metrics spike, revert traffic immediately. Gradually increase percentage (5% → 25% → 50% → 100%) over days or weeks. For data migration, use Azure Data Factory to copy/transform data; validate consistency before fully switching. Only retire old services after 2-4 weeks at 100% to ensure no issues. Use deployment slots and blue-green deployments for per-service cutover if Strangler isn't feasible.

**Q2: How do you handle team organization changes that misalign with service boundaries?**
* **Answer:** **Align service boundaries to team structure (Conway's Law)**. If teams reorganize but services don't, create friction and slowdowns. Schedule a "boundary alignment" architecture review: map current services to current team structure. If a team owns both Order and Inventory services but there's another team handling only Payment, consider consolidating Order + Inventory or splitting Payment. Use Azure DevOps to visualize team ownership of services in the project structure. Communicate boundary changes ahead of time—give teams 2-4 weeks notice before moving responsibility. Use this as an opportunity to refactor boundaries gradually (Strangler Pattern) rather than disruptively.

**Q3: What's the risk of over-monitoring for boundary issues—how do you avoid false positives?**
* **Answer:** Use **anomaly detection and dynamic thresholds** instead of fixed thresholds. Application Insights' intelligent alerting learns normal patterns (Monday traffic is higher than Sunday; spike at 8 AM is normal). Alert only when behavior deviates from established baseline, reducing false positives. Set alert thresholds high enough to catch real issues (e.g., error rate >10%, not >1%). Require sustained anomalies (e.g., high latency for 10 minutes) before alerting, not instant spikes. Create "info-level" vs. "critical-level" alerts: info-level suggests monitoring but doesn't page on-call; critical-level means immediate action needed. Review alert effectiveness quarterly—disable alerts that consistently fire on non-issues.

**Q4: How do you balance refactoring efforts against shipping new features?**
* **Answer:** **Reserve 20% of sprint capacity for technical debt/refactoring** (rough guideline; adjust per team). Prioritize refactoring when metrics clearly indicate boundary issues (circular dependencies, >70% call ratios, deployment coupling). Schedule refactoring during slower business cycles (Q4, post-holiday) to reduce interference with feature delivery. Use feature flags to decouple refactoring from business releases—merge boundaries behind feature flag, enable gradually after stabilization. For major refactoring, dedicate a full sprint with cross-team participation rather than scattering effort. Track refactoring benefits post-deployment (reduced deployment time, faster feature delivery, fewer incidents) to justify investment; if refactoring didn't improve metrics, question whether it was worth the effort.

**Q5: How do you handle legacy services that don't fit modern boundary definitions but are too risky to refactor?**
* **Answer:** **Treat legacy services as-is; don't force refactoring if risk outweighs benefit**. Instead, create an **anti-corruption layer** that translates between legacy service contracts and modern services. New services call the anti-corruption layer, which adapts requests/responses for legacy service. This quarantines legacy oddities, preventing them from infecting new services. For critical legacy services, prioritize observability (extensive logging, distributed tracing) to understand behavior before attempting changes. Consider **strangling** the legacy service by gradually replacing functionality in new services; redirect traffic incrementally until legacy service is no longer needed. If strangling is infeasible (financial/time constraints), accept that legacy service is a boundary exception and document why; this prevents future architects from attempting misguided refactoring.

---

# Microservices Fundamentals & Architecture: Service Boundaries Definition (Continued)

---

## Question 7: How Do You Implement Service Discovery and Dynamic Routing When Service Instances Are Ephemeral or Scaled Dynamically?

### Detailed Answer:

In traditional monolithic systems, services run on known IP addresses or hostnames that rarely change. Microservices architectures, especially in Azure, introduce **ephemeral infrastructure**: containers spin up and down based on demand, instances get replaced during updates, and Azure App Service slots swap instantly. If services hardcode URLs (e.g., `http://inventory-service.contoso.com:8080`), they break when that instance is replaced. **Service discovery** solves this by maintaining a dynamic registry of service locations, allowing consumers to locate available instances at runtime.

Azure provides multiple service discovery mechanisms. **Azure DNS** and **Azure Private DNS Zones** offer DNS-based discovery where services register under FQDNs (e.g., `order-service.azureservices.net`); Azure handles load balancing transparently. For **AKS (Azure Kubernetes Service)**, Kubernetes' built-in DNS service automatically registers services: `order-service.default.svc.cluster.local` always resolves to available replicas. **Azure Service Bus** provides logical addressing—services don't need to know each other's addresses for asynchronous communication; messages are routed by queue/topic name. For more advanced scenarios, **Azure API Management (APIM)** acts as a service mesh gateway, maintaining a registry of backend services and handling routing, versioning, and load balancing.

**Routing patterns** determine how requests reach the right service instance. **Client-side discovery** (client retrieves available instances from registry and chooses one) is complex in microservices but gives clients fine-grained control. **Server-side discovery** (client sends request to API gateway/load balancer; gateway discovers and routes) is more common and simplifies client logic. **Health checking** is critical: if a service instance fails, the registry must detect this and remove it from the available pool, preventing requests from failing. Azure Load Balancer and APIM include health probes; containers in AKS expose liveness/readiness probes. Service discovery must be **resilient**—if the registry is temporarily unavailable, services should either fall back to cached instance lists or queue requests for retry.

### Explanation:

**Importance & Use Cases:**
- **Resilience to failures**: If one instance fails, requests route to healthy instances automatically.
- **Seamless scaling**: New instances are discovered and receive traffic without manual configuration.
- **Zero-downtime deployments**: Old instances are gracefully drained; new instances take over seamlessly.
- **Multi-region support**: Routing can intelligently select nearest region or failover to secondary region.

**Benefits & Trade-offs:**
- **Benefit**: Automatic instance management, reduced operational overhead, resilience to failures.
- **Trade-off**: Complexity in setup and debugging (distributed system problems harder to diagnose); latency from service discovery lookups (mitigated with caching); potential single point of failure if registry goes down.
- **Interview perspective**: Demonstrates understanding of cloud-native architecture patterns and operational excellence at scale.

### C# Implementation:

```csharp
using Azure.Identity;
using Azure.Messaging.ServiceBus;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Diagnostics.HealthChecks;
using Microsoft.Extensions.Diagnostics.HealthChecks;
using System.Net;

// ============ SERVICE DISCOVERY VIA AZURE DNS ============

/// <summary>
/// Service discovery using Azure DNS with automatic load balancing
/// All instances register under same FQDN; DNS returns all healthy IPs
/// </summary>
public class AzureDnsServiceDiscovery
{
    private readonly ILogger<AzureDnsServiceDiscovery> _logger;
    private readonly HttpClient _httpClient;

    public AzureDnsServiceDiscovery(HttpClient httpClient, ILogger<AzureDnsServiceDiscovery> logger)
    {
        _httpClient = httpClient;
        _logger = logger;
    }

    /// <summary>
    /// Resolve service endpoint via Azure DNS
    /// Example: inventory-service.azureservices.net → multiple IPs
    /// </summary>
    public async Task<ServiceInstanceInfo[]> DiscoverServiceInstancesAsync(
        string serviceName,
        string dnsSuffix = "azureservices.net")
    {
        var fqdn = $"{serviceName}.{dnsSuffix}";
        
        try
        {
            // DNS resolution returns all healthy instances
            var addresses = await Dns.GetHostAddressesAsync(fqdn);
            
            _logger.LogInformation(
                "Service discovery for {ServiceName}: Found {InstanceCount} instances",
                serviceName,
                addresses.Length);

            return addresses
                .Select(ip => new ServiceInstanceInfo
                {
                    ServiceName = serviceName,
                    InstanceId = Guid.NewGuid().ToString(), // Generated per lookup
                    IpAddress = ip.ToString(),
                    Port = 443, // HTTPS
                    Scheme = "https",
                    Fqdn = fqdn,
                    DiscoveredAt = DateTime.UtcNow,
                    IsHealthy = true
                })
                .ToArray();
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Service discovery failed for {ServiceName}", serviceName);
            throw;
        }
    }

    /// <summary>
    /// Resolve and call service with automatic retry on different instances
    /// </summary>
    public async Task<T> CallServiceWithDiscoveryAsync<T>(
        string serviceName,
        string endpoint,
        Func<HttpClient, Task<T>> requestFunc)
    {
        var instances = await DiscoverServiceInstancesAsync(serviceName);
        
        if (instances.Length == 0)
        {
            throw new InvalidOperationException($"No healthy instances of {serviceName} found");
        }

        // Try each instance; circuit break after all fail
        Exception? lastException = null;
        foreach (var instance in instances)
        {
            try
            {
                var baseUrl = $"{instance.Scheme}://{instance.Fqdn}";
                _httpClient.BaseAddress = new Uri(baseUrl);
                
                var result = await requestFunc(_httpClient);
                
                _logger.LogInformation(
                    "Successfully called {ServiceName} on instance {InstanceId}",
                    serviceName,
                    instance.InstanceId);
                
                return result;
            }
            catch (Exception ex)
            {
                _logger.LogWarning(
                    ex,
                    "Failed to call {ServiceName} on instance {InstanceId}. Trying next instance.",
                    serviceName,
                    instance.InstanceId);
                
                lastException = ex;
            }
        }

        throw new InvalidOperationException(
            $"All instances of {serviceName} failed",
            lastException);
    }
}

// ============ SERVICE DISCOVERY VIA AZURE API MANAGEMENT ============

/// <summary>
/// Service discovery via API Management (server-side discovery pattern)
/// APIM maintains backend pool and routes requests intelligently
/// </summary>
public class ApiManagementServiceDiscovery
{
    private readonly HttpClient _httpClient;
    private readonly ILogger<ApiManagementServiceDiscovery> _logger;
    private readonly string _apimEndpoint;
    private readonly string _apiKey;

    public ApiManagementServiceDiscovery(
        HttpClient httpClient,
        ILogger<ApiManagementServiceDiscovery> logger,
        string apimEndpoint,
        string apiKey)
    {
        _httpClient = httpClient;
        _logger = logger;
        _apimEndpoint = apimEndpoint;
        _apiKey = apiKey;
    }

    /// <summary>
    /// Call service through APIM gateway
    /// APIM handles instance discovery, health checking, and routing
    /// </summary>
    public async Task<HttpResponseMessage> CallServiceThroughApimAsync(
        string apiPath,
        HttpMethod method,
        string? requestBody = null)
    {
        var requestUri = $"{_apimEndpoint}/{apiPath}";
        
        _logger.LogInformation(
            "Routing request to {ApiPath} via APIM",
            apiPath);

        using var request = new HttpRequestMessage(method, requestUri);
        request.Headers.Add("Ocp-Apim-Subscription-Key", _apiKey);

        if (requestBody != null)
        {
            request.Content = new StringContent(
                requestBody,
                System.Text.Encoding.UTF8,
                "application/json");
        }

        // APIM gateway handles:
        // - Service instance discovery
        // - Load balancing
        // - Health probe checks
        // - Retry on unhealthy instance
        // - Rate limiting
        var response = await _httpClient.SendAsync(request);

        if (!response.IsSuccessStatusCode)
        {
            _logger.LogWarning(
                "APIM gateway returned {StatusCode} for {ApiPath}",
                response.StatusCode,
                apiPath);
        }

        return response;
    }

    /// <summary>
    /// Register service backend with APIM
    /// Called during service startup
    /// </summary>
    public async Task RegisterBackendAsync(
        string backendName,
        string backendUrl,
        HealthCheckOptions healthCheckOptions)
    {
        _logger.LogInformation(
            "Registering backend {BackendName} at {BackendUrl}",
            backendName,
            backendUrl);

        // In production, call APIM REST API to register backend
        // For now, mock implementation
        await Task.CompletedTask;
    }
}

// ============ AKS SERVICE DISCOVERY (KUBERNETES DNS) ============

/// <summary>
/// Service discovery in Azure Kubernetes Service via Kubernetes DNS
/// Kubernetes automatically maintains service registry
/// </summary>
public class KubernetesServiceDiscovery
{
    private readonly ILogger<KubernetesServiceDiscovery> _logger;
    private readonly HttpClient _httpClient;

    public KubernetesServiceDiscovery(
        HttpClient httpClient,
        ILogger<KubernetesServiceDiscovery> logger)
    {
        _httpClient = httpClient;
        _logger = logger;
    }

    /// <summary>
    /// Call service via Kubernetes DNS
    /// Format: servicename.namespace.svc.cluster.local
    /// Kubernetes DNS automatically load balances across pods
    /// </summary>
    public async Task<T> CallKubernetesServiceAsync<T>(
        string serviceName,
        string kubernetesNamespace,
        string endpoint,
        Func<HttpResponseMessage, Task<T>> parseResponseFunc)
    {
        // Kubernetes DNS format
        var kuberenetesDnsName = $"{serviceName}.{kubernetesNamespace}.svc.cluster.local";
        var requestUrl = $"http://{kuberenetesDnsName}/{endpoint}";

        try
        {
            _logger.LogInformation(
                "Calling Kubernetes service: {ServiceDns} at {Endpoint}",
                kuberenetesDnsName,
                endpoint);

            var response = await _httpClient.GetAsync(requestUrl);
            
            if (!response.IsSuccessStatusCode)
            {
                _logger.LogWarning(
                    "Kubernetes service call failed: {StatusCode}",
                    response.StatusCode);
                
                throw new HttpRequestException($"Service returned {response.StatusCode}");
            }

            return await parseResponseFunc(response);
        }
        catch (Exception ex)
        {
            _logger.LogError(
                ex,
                "Failed to call Kubernetes service {ServiceDns}",
                kuberenetesDnsName);
            
            throw;
        }
    }
}

// ============ HEALTH CHECKING FOR SERVICE DISCOVERY ============

/// <summary>
/// Health check endpoint that service discovery systems probe
/// Liveness probe: is service running?
/// Readiness probe: is service ready to receive traffic?
/// </summary>
public class ServiceHealthCheckController : ControllerBase
{
    private readonly IHealthCheckService _healthCheckService;
    private readonly ILogger<ServiceHealthCheckController> _logger;

    public ServiceHealthCheckController(
        IHealthCheckService healthCheckService,
        ILogger<ServiceHealthCheckController> logger)
    {
        _healthCheckService = healthCheckService;
        _logger = logger;
    }

    /// <summary>
    /// Liveness probe: 200 OK if service is running
    /// Service discovery removes instance if this returns non-200
    /// </summary>
    [HttpGet("/health/live")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status503ServiceUnavailable)]
    public async Task<IActionResult> LivenessProbeAsync()
    {
        // Simple check: is process alive?
        _logger.LogDebug("Liveness probe called");
        
        return Ok(new { status = "alive", timestamp = DateTime.UtcNow });
    }

    /// <summary>
    /// Readiness probe: 200 OK if service can handle traffic
    /// Checks dependencies (database, cache, etc.)
    /// Service not added to load balancer until this passes
    /// </summary>
    [HttpGet("/health/ready")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status503ServiceUnavailable)]
    public async Task<IActionResult> ReadinessProbeAsync()
    {
        var result = await _healthCheckService.CheckHealthAsync();

        _logger.LogInformation(
            "Readiness probe: {HealthStatus}",
            result.Status);

        if (result.Status == HealthStatus.Healthy)
        {
            return Ok(new
            {
                status = "ready",
                checks = result.Entries.Select(e => new
                {
                    name = e.Key,
                    status = e.Value.Status.ToString()
                })
            });
        }

        return StatusCode(503, new
        {
            status = "not_ready",
            reason = result.Entries
                .Where(e => e.Value.Status != HealthStatus.Healthy)
                .Select(e => new { name = e.Key, reason = e.Value.Description })
        });
    }
}

// ============ CONFIGURABLE SERVICE DISCOVERY ============

/// <summary>
/// Adapter that allows switching between discovery mechanisms
/// via configuration (DNS, APIM, or Kubernetes)
/// </summary>
public interface IServiceDiscoveryProvider
{
    Task<ServiceInstanceInfo[]> DiscoverAsync(string serviceName);
    Task<HttpResponseMessage> CallServiceAsync(
        string serviceName,
        string endpoint,
        HttpMethod method,
        string? requestBody = null);
}

public class ConfigurableServiceDiscovery : IServiceDiscoveryProvider
{
    private readonly string _discoveryMode; // "dns", "apim", or "kubernetes"
    private readonly AzureDnsServiceDiscovery? _dnsDiscovery;
    private readonly ApiManagementServiceDiscovery? _apimDiscovery;
    private readonly KubernetesServiceDiscovery? _kubeDiscovery;
    private readonly ILogger<ConfigurableServiceDiscovery> _logger;

    public ConfigurableServiceDiscovery(
        IConfiguration configuration,
        AzureDnsServiceDiscovery? dnsDiscovery,
        ApiManagementServiceDiscovery? apimDiscovery,
        KubernetesServiceDiscovery? kubeDiscovery,
        ILogger<ConfigurableServiceDiscovery> logger)
    {
        _discoveryMode = configuration["ServiceDiscovery:Mode"] ?? "dns";
        _dnsDiscovery = dnsDiscovery;
        _apimDiscovery = apimDiscovery;
        _kubeDiscovery = kubeDiscovery;
        _logger = logger;

        _logger.LogInformation("Service discovery mode: {Mode}", _discoveryMode);
    }

    public async Task<ServiceInstanceInfo[]> DiscoverAsync(string serviceName)
    {
        return _discoveryMode switch
        {
            "dns" => await _dnsDiscovery.DiscoverServiceInstancesAsync(serviceName),
            "apim" => throw new NotImplementedException("APIM discovery not directly exposed"),
            "kubernetes" => new[] { new ServiceInstanceInfo
            {
                ServiceName = serviceName,
                IpAddress = $"{serviceName}.default.svc.cluster.local",
                Port = 80,
                Scheme = "http"
            }},
            _ => throw new InvalidOperationException($"Unknown discovery mode: {_discoveryMode}")
        };
    }

    public async Task<HttpResponseMessage> CallServiceAsync(
        string serviceName,
        string endpoint,
        HttpMethod method,
        string? requestBody = null)
    {
        _logger.LogInformation(
            "Calling {ServiceName}/{Endpoint} via {DiscoveryMode}",
            serviceName,
            endpoint,
            _discoveryMode);

        return _discoveryMode switch
        {
            "dns" => await CallViaDnsAsync(serviceName, endpoint, method, requestBody),
            "apim" => await _apimDiscovery.CallServiceThroughApimAsync(
                $"{serviceName}/{endpoint}", method, requestBody),
            "kubernetes" => await CallViaKubernetesAsync(
                serviceName, endpoint, method, requestBody),
            _ => throw new InvalidOperationException($"Unknown discovery mode: {_discoveryMode}")
        };
    }

    private async Task<HttpResponseMessage> CallViaDnsAsync(
        string serviceName,
        string endpoint,
        HttpMethod method,
        string? requestBody)
    {
        var instances = await _dnsDiscovery.DiscoverServiceInstancesAsync(serviceName);
        if (instances.Length == 0)
            throw new InvalidOperationException($"No instances of {serviceName} found");

        var instance = instances[0]; // Simple round-robin
        var url = $"{instance.Scheme}://{instance.Fqdn}/{endpoint}";

        using var request = new HttpRequestMessage(method, url);
        if (requestBody != null)
        {
            request.Content = new StringContent(
                requestBody,
                System.Text.Encoding.UTF8,
                "application/json");
        }

        return await new HttpClient().SendAsync(request);
    }

    private async Task<HttpResponseMessage> CallViaKubernetesAsync(
        string serviceName,
        string endpoint,
        HttpMethod method,
        string? requestBody)
    {
        var kubeServiceName = $"{serviceName}.default.svc.cluster.local";
        var url = $"http://{kubeServiceName}/{endpoint}";

        using var request = new HttpRequestMessage(method, url);
        if (requestBody != null)
        {
            request.Content = new StringContent(
                requestBody,
                System.Text.Encoding.UTF8,
                "application/json");
        }

        return await new HttpClient().SendAsync(request);
    }
}

// ============ DEPENDENCY INJECTION SETUP ============

public static class ServiceDiscoveryExtensions
{
    public static IServiceCollection AddServiceDiscovery(
        this IServiceCollection services,
        IConfiguration configuration)
    {
        var discoveryMode = configuration["ServiceDiscovery:Mode"] ?? "dns";

        services.AddHttpClient<AzureDnsServiceDiscovery>();
        services.AddSingleton<ApiManagementServiceDiscovery>(provider =>
            new ApiManagementServiceDiscovery(
                provider.GetRequiredService<HttpClient>(),
                provider.GetRequiredService<ILogger<ApiManagementServiceDiscovery>>(),
                configuration["APIM:Endpoint"] ?? throw new InvalidOperationException("APIM:Endpoint not configured"),
                configuration["APIM:ApiKey"] ?? throw new InvalidOperationException("APIM:ApiKey not configured")
            ));
        services.AddHttpClient<KubernetesServiceDiscovery>();
        services.AddSingleton<IServiceDiscoveryProvider, ConfigurableServiceDiscovery>();

        return services;
    }
}

// ============ DATA MODELS ============

public record ServiceInstanceInfo
{
    public string ServiceName { get; set; }
    public string InstanceId { get; set; }
    public string IpAddress { get; set; }
    public int Port { get; set; }
    public string Scheme { get; set; }
    public string Fqdn { get; set; }
    public DateTime DiscoveredAt { get; set; }
    public bool IsHealthy { get; set; }
}

public record HealthCheckOptions
{
    public string Path { get; set; } = "/health";
    public int IntervalSeconds { get; set; } = 10;
    public int TimeoutSeconds { get; set; } = 5;
    public int HealthyThreshold { get; set; } = 2;
    public int UnhealthyThreshold { get; set; } = 3;
}
```

### Follow-Up Questions:

**Q1: How does service discovery handle cascading failures where a downstream service is slow, causing timeouts in service discovery lookup?**
* **Answer:** Implement **caching of discovery results** with TTL (time-to-live). Cache last-known healthy instances locally; if discovery service is slow/down, use cached list instead of blocking. However, stale instances in cache may be unhealthy—complement with **circuit breakers**: if an instance fails repeatedly, remove from local cache. Use **health check probes** (liveness/readiness checks) to detect failures early before caching stale data. For critical services, implement **backup DNS servers** or **fallback discovery providers** (e.g., primary is DNS, fallback is hardcoded list). In Application Insights, track discovery latency; if >1 second, investigate—service discovery itself shouldn't be a bottleneck.

**Q2: In a multi-region deployment, how does service discovery handle failover from one region to another?**
* **Answer:** Use **Azure Traffic Manager** or **Azure Front Door** for multi-region routing. Both support geographic routing (route based on user location) and failover routing (if primary region is down, route to secondary). Service discovery within each region (Azure DNS or Kubernetes DNS) handles local instance discovery; Traffic Manager/Front Door handles inter-region failover. For consistency, replicate service registries across regions (e.g., replicated DNS zones). Test failover regularly—use chaos engineering (Azure Chaos Studio) to simulate region failures and verify automatic failover works. Document RTO/RPO for failover; be explicit about data consistency expectations (eventual consistency acceptable, or do you need strong consistency?).

**Q3: What happens when service discovery itself fails—how do you ensure services can still route requests?**
* **Answer:** **Fallback to cached instance list** (see Q1). Additionally, implement **local service registry fallback**—if centralized discovery is down, services fall back to a local config file or environment variables with backup instance lists (updated periodically). For Kubernetes, use **DNS caching at pod level** (kubelet's DNS cache); if Kubernetes DNS service temporarily fails, local DNS cache continues answering. For Azure DNS, multiple redundant DNS servers ensure availability; client-side DNS resolution typically retries across multiple servers. Avoid single points of failure in discovery itself—use managed services (Azure DNS, Kubernetes DNS) rather than custom solutions. Monitor discovery service health in Application Insights; if it goes down, alert immediately.

**Q4: How do you handle uneven load distribution when service discovery returns all instances as equal but some are overloaded?**
* **Answer:** Implement **weighted load balancing** where discovery returns instances with weights (e.g., 80% traffic to instance A, 20% to instance B). Client uses weights when load balancing. Alternatively, use **APIM or load balancer** that monitors real-time instance load and adjusts routing—this is server-side discovery advantage. In Kubernetes, **Horizontal Pod Autoscaler (HPA)** automatically scales based on CPU/memory metrics; more replicas = lighter load per instance. For client-side discovery, use **least-connections algorithm** (route to instance with fewest active connections) instead of round-robin. Monitor instance load via Application Insights metrics; if one instance consistently overloaded while others have spare capacity, it suggests scaling issue or algorithm inefficiency—investigate.

**Q5: Should services cache discovery results or always do fresh lookups? What's the trade-off?**
* **Answer:** **Cache with short TTL** (10–30 seconds). Fresh lookups every request add latency and query load on discovery service; caching reduces this. However, stale cache may contain failed instances, causing request failures—mitigate with health checks and circuit breakers. For critical paths, use **adaptive caching**: if discovery is fast and reliable, use short cache TTL; if discovery is slow/unreliable, use longer TTL even if data may be stale. In Kubernetes, DNS caching at pod level (configurable via `ndots` option) provides built-in caching. Measure cache hit rate in Application Insights; if <80%, consider longer TTL or more frequent cache refresh.

---

## Question 8: How Do You Design Service Boundaries to Support Multi-Tenancy and Isolation Requirements?

### Detailed Answer:

Multi-tenancy adds complexity to service boundaries: you must balance shared infrastructure efficiency with tenant isolation and data security. There are two primary multi-tenancy models, each with different boundary implications. **Silo model** (dedicated resources per tenant) is simplest for isolation but least efficient—each tenant gets separate database, service instances, even separate Azure subscriptions. **Pool model** (shared resources with logical isolation) maximizes efficiency but requires robust isolation mechanisms. Most production systems use a **hybrid approach**: shared service tier (API Gateway, core services) with per-tenant databases or data partitioning.

**Data isolation** is the most critical aspect of multi-tenant boundaries. At the service boundary level, isolation manifests as: (1) **Database per tenant** (highest isolation, lowest efficiency), (2) **Shared database with per-tenant schema** (moderate isolation via schema separation), or (3) **Shared database with row-level security (RLS)** or data partitioning (highest efficiency, requires careful access control). Azure SQL Database supports RLS via security policies that automatically filter rows by tenant; Azure Cosmos DB supports logical partitioning by tenant ID. The key principle: **every query and API call must be scoped to the authenticated tenant**—no query should ever expose data from another tenant, even accidentally.

**Service boundary design for multi-tenancy** requires thoughtful contract design. REST APIs should include tenant context (via JWT claims, header, or URL path) automatically. For example, `GET /api/v1/orders` for single-tenant becomes `GET /api/v1/tenants/{tenantId}/orders` for multi-tenant (though tenantId can be implicit from JWT). Service-to-service calls must propagate tenant context via headers or JWT claims; if Order service calls Inventory service, it must pass the tenant ID so Inventory service queries only that tenant's data. **Blast radius** must be bounded: if one tenant's data is compromised due to a bug, other tenants are unaffected. Achieve this through data partitioning, network policies, and access controls at service boundaries.

**Compliance and regulatory requirements** differ by tenant (GDPR for EU customers, HIPAA for healthcare). Design boundaries with flexibility for tenant-specific policies. For example, one tenant might require on-premises data residency; implement a boundary that routes that tenant to a separate on-premises service while others use Azure cloud. Use **feature flags** per tenant to enable/disable functionality; use **policy engines** at service boundaries to enforce tenant-specific rules (encryption, data retention, audit logging).

### Explanation:

**Importance & Use Cases:**
- **Scalability**: Multi-tenant architecture serves many customers from shared infrastructure, reducing per-customer cost.
- **Regulatory compliance**: Flexibility in data residency, encryption, audit logging for diverse compliance requirements.
- **Data security**: Proper isolation prevents accidental or malicious cross-tenant data leakage.
- **Operational efficiency**: Centralized updates benefit all tenants; no per-tenant maintenance burden.

**Benefits & Trade-offs:**
- **Benefit**: Cost efficiency, scalability, centralized operations, faster feature rollouts.
- **Trade-off**: Complexity in isolation mechanisms, regulatory compliance management, potential performance impact of row-level filtering, harder debugging when issues affect only specific tenants.
- **Interview perspective**: Demonstrates understanding of SaaS architecture and operational complexity beyond single-tenant systems.

### C# Implementation:

```csharp
using Azure.Data.Tables;
using Azure.Security.KeyVault.Secrets;
using Microsoft.AspNetCore.Authorization;
using Microsoft.Identity.Web;
using System.Security.Claims;

// ============ MULTI-TENANT CONTEXT PROPAGATION ============

/// <summary>
/// Extract tenant context from JWT or HTTP headers
/// Injected into every request to ensure tenant scoping
/// </summary>
public interface ITenantContext
{
    string TenantId { get; }
    string UserId { get; }
    string[] Roles { get; }
    bool IsValidTenant { get; }
}

public class HttpContextTenantProvider : ITenantContext
{
    private readonly IHttpContextAccessor _httpContextAccessor;
    private readonly ILogger<HttpContextTenantProvider> _logger;
    private string? _cachedTenantId;

    public HttpContextTenantProvider(
        IHttpContextAccessor httpContextAccessor,
        ILogger<HttpContextTenantProvider> logger)
    {
        _httpContextAccessor = httpContextAccessor;
        _logger = logger;
    }

    public string TenantId
    {
        get
        {
            if (_cachedTenantId != null)
                return _cachedTenantId;

            var context = _httpContextAccessor.HttpContext;
            if (context == null)
                throw new InvalidOperationException("HttpContext is null");

            // Extract tenant ID from JWT claim (preferred)
            var tenantIdClaim = context.User.FindFirst("tid"); // Azure AD tenant ID
            if (tenantIdClaim != null)
            {
                _cachedTenantId = tenantIdClaim.Value;
                return _cachedTenantId;
            }

            // Fallback: extract from URL route
            if (context.Request.RouteValues.TryGetValue("tenantId", out var routeTenantId))
            {
                _cachedTenantId = routeTenantId?.ToString();
                return _cachedTenantId;
            }

            // Fallback: extract from custom header
            if (context.Request.Headers.TryGetValue("X-Tenant-ID", out var headerTenantId))
            {
                _cachedTenantId = headerTenantId.ToString();
                return _cachedTenantId;
            }

            throw new InvalidOperationException("Unable to determine tenant ID from request");
        }
    }

    public string UserId => _httpContextAccessor.HttpContext?.User.FindFirst(ClaimTypes.NameIdentifier)?.Value
        ?? throw new InvalidOperationException("Unable to determine user ID");

    public string[] Roles => _httpContextAccessor.HttpContext?.User
        .FindAll(ClaimTypes.Role)
        .Select(c => c.Value)
        .ToArray() ?? Array.Empty<string>();

    public bool IsValidTenant
    {
        get
        {
            try
            {
                var tenantId = TenantId;
                // Validate tenant exists in multi-tenant registry (or cache)
                return !string.IsNullOrEmpty(tenantId);
            }
            catch
            {
                return false;
            }
        }
    }
}

/// <summary>
/// Middleware to enforce tenant context in every request
/// Prevents accidentally querying wrong tenant data
/// </summary>
public class TenantScopingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<TenantScopingMiddleware> _logger;

    public TenantScopingMiddleware(RequestDelegate next, ILogger<TenantScopingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context, ITenantContext tenantContext)
    {
        try
        {
            // Validate tenant context exists
            if (!tenantContext.IsValidTenant)
            {
                _logger.LogWarning("Request without valid tenant context: {Path}", context.Request.Path);
                context.Response.StatusCode = StatusCodes.Status401Unauthorized;
                await context.Response.WriteAsync("Invalid tenant context");
                return;
            }

            // Store tenant ID for access in services
            context.Items["TenantId"] = tenantContext.TenantId;
            context.Items["UserId"] = tenantContext.UserId;

            _logger.LogInformation(
                "Request scoped to tenant {TenantId} by user {UserId}",
                tenantContext.TenantId,
                tenantContext.UserId);

            await _next(context);
        }
        catch (InvalidOperationException ex)
        {
            _logger.LogError(ex, "Tenant context error");
            context.Response.StatusCode = StatusCodes.Status400BadRequest;
            await context.Response.WriteAsync($"Tenant context error: {ex.Message}");
        }
    }
}

// ============ MULTI-TENANT DATA ISOLATION ============

/// <summary>
/// Data repository with automatic tenant-scoping
/// Every query is automatically filtered by tenant
/// </summary>
public interface IMultiTenantRepository<T> where T : IMultiTenantEntity
{
    Task<T?> GetByIdAsync(string tenantId, string id);
    Task<IEnumerable<T>> ListAsync(string tenantId);
    Task CreateAsync(string tenantId, T entity);
    Task UpdateAsync(string tenantId, T entity);
    Task DeleteAsync(string tenantId, string id);
}

public interface IMultiTenantEntity
{
    string Id { get; }
    string TenantId { get; set; }
    DateTime CreatedAt { get; set; }
}

public class MultiTenantOrderRepository : IMultiTenantRepository<Order>
{
    private readonly IDbContext _dbContext;
    private readonly ITenantContext _tenantContext;
    private readonly ILogger<MultiTenantOrderRepository> _logger;

    public MultiTenantOrderRepository(
        IDbContext dbContext,
        ITenantContext tenantContext,
        ILogger<MultiTenantOrderRepository> logger)
    {
        _dbContext = dbContext;
        _tenantContext = tenantContext;
        _logger = logger;
    }

    public async Task<Order?> GetByIdAsync(string tenantId, string id)
    {
        // Verify caller owns this tenant
        ValidateTenantAccess(tenantId);

        _logger.LogInformation(
            "Retrieving order {OrderId} for tenant {TenantId}",
            id,
            tenantId);

        // Query automatically scoped to tenant
        var order = await _dbContext.Orders
            .Where(o => o.TenantId == tenantId && o.Id == id)
            .FirstOrDefaultAsync();

        if (order == null)
        {
            _logger.LogWarning(
                "Order {OrderId} not found for tenant {TenantId}",
                id,
                tenantId);
        }

        return order;
    }

    public async Task<IEnumerable<Order>> ListAsync(string tenantId)
    {
        ValidateTenantAccess(tenantId);

        _logger.LogInformation("Listing orders for tenant {TenantId}", tenantId);

        return await _dbContext.Orders
            .Where(o => o.TenantId == tenantId)
            .ToListAsync();
    }

    public async Task CreateAsync(string tenantId, Order entity)
    {
        ValidateTenantAccess(tenantId);

        // Ensure entity is scoped to tenant
        entity.TenantId = tenantId;
        entity.CreatedAt = DateTime.UtcNow;

        _logger.LogInformation(
            "Creating order for tenant {TenantId}",
            tenantId);

        _dbContext.Orders.Add(entity);
        await _dbContext.SaveChangesAsync();
    }

    public async Task UpdateAsync(string tenantId, Order entity)
    {
        ValidateTenantAccess(tenantId);

        // Verify entity belongs to tenant
        if (entity.TenantId != tenantId)
        {
            throw new UnauthorizedAccessException(
                $"Order belongs to tenant {entity.TenantId}, not {tenantId}");
        }

        _logger.LogInformation(
            "Updating order {OrderId} for tenant {TenantId}",
            entity.Id,
            tenantId);

        _dbContext.Orders.Update(entity);
        await _dbContext.SaveChangesAsync();
    }

    public async Task DeleteAsync(string tenantId, string id)
    {
        ValidateTenantAccess(tenantId);

        var order = await GetByIdAsync(tenantId, id);
        if (order == null)
            throw new InvalidOperationException($"Order {id} not found");

        _logger.LogInformation(
            "Deleting order {OrderId} for tenant {TenantId}",
            id,
            tenantId);

        _dbContext.Orders.Remove(order);
        await _dbContext.SaveChangesAsync();
    }

    private void ValidateTenantAccess(string requestedTenantId)
    {
        if (_tenantContext.TenantId != requestedTenantId)
        {
            _logger.LogError(
                "Unauthorized tenant access attempt: {RequestedTenantId} by {ActualTenantId}",
                requestedTenantId,
                _tenantContext.TenantId);

            throw new UnauthorizedAccessException(
                $"You do not have access to tenant {requestedTenantId}");
        }
    }
}

// ============ MULTI-TENANT API CONTROLLER ============

[ApiController]
[Route("api/v1/tenants/{tenantId}/orders")]
[Authorize]
public class MultiTenantOrdersController : ControllerBase
{
    private readonly IMultiTenantRepository<Order> _orderRepository;
    private readonly ITenantContext _tenantContext;
    private readonly ILogger<MultiTenantOrdersController> _logger;

    public MultiTenantOrdersController(
        IMultiTenantRepository<Order> orderRepository,
        ITenantContext tenantContext,
        ILogger<MultiTenantOrdersController> logger)
    {
        _orderRepository = orderRepository;
        _tenantContext = tenantContext;
        _logger = logger;
    }

    /// <summary>
    /// List orders for authenticated tenant
    /// URL includes tenantId; middleware verifies it matches JWT
    /// </summary>
    [HttpGet]
    [ProducesResponseType(typeof(IEnumerable<OrderResponse>), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status401Unauthorized)]
    public async Task<ActionResult<IEnumerable<OrderResponse>>> ListOrdersAsync(string tenantId)
    {
        _logger.LogInformation(
            "Listing orders for tenant {TenantId}",
            tenantId);

        var orders = await _orderRepository.ListAsync(tenantId);

        return Ok(orders.Select(o => new OrderResponse(o)));
    }

    /// <summary>
    /// Get specific order
    /// Implicitly scoped to authenticated tenant
    /// </summary>
    [HttpGet("{orderId}")]
    [ProducesResponseType(typeof(OrderResponse), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    [ProducesResponseType(StatusCodes.Status401Unauthorized)]
    public async Task<ActionResult<OrderResponse>> GetOrderAsync(string tenantId, string orderId)
    {
        var order = await _orderRepository.GetByIdAsync(tenantId, orderId);

        if (order == null)
        {
            _logger.LogWarning(
                "Order {OrderId} not found for tenant {TenantId}",
                orderId,
                tenantId);

            return NotFound();
        }

        return Ok(new OrderResponse(order));
    }

    /// <summary>
    /// Create order for authenticated tenant
    /// </summary>
    [HttpPost]
    [ProducesResponseType(typeof(OrderResponse), StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<ActionResult<OrderResponse>> CreateOrderAsync(
        string tenantId,
        [FromBody] CreateOrderRequest request)
    {
        if (!ModelState.IsValid)
            return BadRequest(ModelState);

        var order = new Order
        {
            Id = Guid.NewGuid().ToString(),
            TenantId = tenantId,
            CustomerId = request.CustomerId,
            TotalAmount = request.TotalAmount,
            Status = "Pending"
        };

        await _orderRepository.CreateAsync(tenantId, order);

        _logger.LogInformation(
            "Order {OrderId} created for tenant {TenantId}",
            order.Id,
            tenantId);

        return CreatedAtAction(
            nameof(GetOrderAsync),
            new { tenantId, orderId = order.Id },
            new OrderResponse(order));
    }
}

// ============ SERVICE-TO-SERVICE MULTI-TENANT CALLS ============

/// <summary>
/// HTTP client that automatically propagates tenant context in inter-service calls
/// Ensures downstream services query correct tenant data
/// </summary>
public class MultiTenantHttpClientHandler : DelegatingHandler
{
    private readonly ITenantContext _tenantContext;
    private readonly ILogger<MultiTenantHttpClientHandler> _logger;

    public MultiTenantHttpClientHandler(
        ITenantContext tenantContext,
        ILogger<MultiTenantHttpClientHandler> logger)
    {
        _tenantContext = tenantContext;
        _logger = logger;
    }

    protected override async Task<HttpResponseMessage> SendAsync(
        HttpRequestMessage request,
        CancellationToken cancellationToken)
    {
        // Automatically add tenant ID to outgoing service-to-service calls
        request.Headers.Add("X-Tenant-ID", _tenantContext.TenantId);
        request.Headers.Add("X-User-ID", _tenantContext.UserId);

        _logger.LogInformation(
            "Service-to-service call to {Url} for tenant {TenantId}",
            request.RequestUri,
            _tenantContext.TenantId);

        return await base.SendAsync(request, cancellationToken);
    }
}

public class InventoryServiceClient
{
    private readonly HttpClient _httpClient;
    private readonly ITenantContext _tenantContext;

    public InventoryServiceClient(
        HttpClient httpClient,
        ITenantContext tenantContext)
    {
        _httpClient = httpClient;
        _tenantContext = tenantContext;
    }

    public async Task<bool> IsInventoryAvailableAsync(string sku, int quantity)
    {
        // Tenant ID automatically added by MultiTenantHttpClientHandler
        var response = await _httpClient.PostAsync(
            $"https://inventory-service/api/v1/check-availability",
            new StringContent(
                System.Text.Json.JsonSerializer.Serialize(new { sku, quantity }),
                System.Text.Encoding.UTF8,
                "application/json"));

        return response.IsSuccessStatusCode;
    }
}

// ============ MULTI-TENANT CONFIGURATION & SETUP ============

public static class MultiTenancyExtensions
{
    public static IServiceCollection AddMultiTenancy(
        this IServiceCollection services,
        IConfiguration configuration)
    {
        // Add tenant context provider
        services.AddScoped<ITenantContext, HttpContextTenantProvider>();
        services.AddHttpContextAccessor();

        // Add multi-tenant middleware
        services.AddScoped<TenantScopingMiddleware>();

        // Add tenant-scoped repositories
        services.AddScoped(typeof(IMultiTenantRepository<>), typeof(MultiTenantOrderRepository));

        // Add tenant-aware HTTP client
        services.AddHttpClient<InventoryServiceClient>()
            .AddHttpMessageHandler<MultiTenantHttpClientHandler>();

        return services;
    }
}

public static class IApplicationBuilderExtensions
{
    public static IApplicationBuilder UseMultiTenancy(this IApplicationBuilder app)
    {
        app.UseMiddleware<TenantScopingMiddleware>();
        return app;
    }
}

// ============ DATA MODELS ============

public class Order : IMultiTenantEntity
{
    public string Id { get; set; }
    public string TenantId { get; set; }
    public string CustomerId { get; set; }
    public decimal TotalAmount { get; set; }
    public string Status { get; set; }
    public DateTime CreatedAt { get; set; }
}

public record OrderResponse(string OrderId, string CustomerId, decimal TotalAmount, string Status);

public record CreateOrderRequest(string CustomerId, decimal TotalAmount);

public interface IDbContext
{
    DbSet<Order> Orders { get; }
    Task SaveChangesAsync();
}
```

### Follow-Up Questions:

**Q1: How do you prevent accidentally querying another tenant's data due to bugs or security oversights?**
* **Answer:** Implement **defense-in-depth** with multiple layers: (1) Application layer—automatic tenant scoping in repository patterns (as shown above); always validate `TenantId` before querying. (2) Database layer—use Azure SQL Row-Level Security (RLS) or Cosmos DB logical partitioning as secondary enforcement; even if app layer is bypassed, database denies cross-tenant access. (3) Network layer—use network policies/firewalls to restrict service-to-service calls; one tenant's isolated service instance can't call another's. (4) Testing—automated tests that verify queries don't leak data; run tests as different tenants, verify isolation. (5) Code review—mandatory security review for any code touching tenant data. Use static analysis tools to flag unsafe patterns. Annual penetration testing confirms no cross-tenant leakage.

**Q2: How do you handle compliance requirements where different tenants need different data residency, encryption, or audit policies?**
* **Answer:** **Tenant-specific policy engine at service boundaries**. Store tenant configuration (data residency, encryption key, audit requirements) in a configuration service. At service boundaries, check tenant config and route accordingly: EU tenant requests → European data center, on-premises tenant requests → on-premises service instance, healthcare tenant → service with HIPAA-compliant audit logging. Use **conditional infrastructure**: deploy encryption, audit, and storage services per tenant. For truly demanding tenants (financial, healthcare), consider **single-tenant deployment** (dedicated service instances) even though less efficient. Track all data access via audit logs (queryable per tenant). Use Azure Policy to enforce organizational compliance; tag resources by tenant and compliance requirement.

**Q3: Does multi-tenancy require separate databases or can you use shared databases with logical isolation?**
* **Answer:** Both work; trade-offs differ. **Shared database with row-level security (RLS)** is most efficient—one database serves all tenants, Azure SQL RLS automatically filters rows by tenant. However, schema updates affect all tenants simultaneously (risk of breaking one tenant). **Database per tenant** offers strongest isolation—one tenant's schema change doesn't affect others; highest operational overhead (backup/restore per database). **Hybrid approach**: core/shared services use shared databases; high-volume or compliance-sensitive services use databases per tenant. In Azure, use **Elastic Database pools** to share compute/storage among per-tenant databases, reducing overhead. For cost-sensitive workloads, shared database with RLS; for compliance-sensitive workloads, database per tenant.

**Q4: How do you test multi-tenant applications to ensure isolation is working correctly?**
* **Answer:** Implement **tenant-isolation test suite**: (1) Create two test tenants (TenantA, TenantB). (2) Create data for TenantA. (3) Query as TenantB—verify no TenantA data returned. (4) Attempt to delete TenantA data as TenantB—verify authorization error. (5) Directly query database as TenantB—verify RLS filters apply. (6) Call service-to-service APIs—verify tenant context propagated. (7) Chaos test—inject faults in one tenant's data store; verify other tenants unaffected. Use **BDD (Behavior-Driven Development)** scenarios: "When TenantB queries TenantA's orders, then authorization error is returned." Automate these tests in CI/CD pipeline; fail build if any isolation violation detected. Run tests against both shared and per-tenant database models to validate both approaches.

**Q5: What's the impact of multi-tenancy on service boundary design—do boundaries need to change?**
* **Answer:** **Service boundaries remain conceptually the same (business capabilities) but implementation changes.** Order service still owns "order creation and management," but now every method must accept and validate `TenantId`. New services may emerge for multi-tenant concerns (tenant registry, billing/metering, compliance enforcement). Data sharing boundaries become stricter—Order service can't directly query Inventory's database (even for same tenant) because RLS would allow both services to see each other's data if misconfigured. Instead, both call each other's APIs with tenant context. Consider a **Platform/Infrastructure service** responsible for multi-tenant policies, encryption key management, audit logging—this service becomes a new boundary that all business services depend on. Design boundaries with tenant isolation in mind from the start, not retrofitted later (retrofitting multi-tenancy is expensive and error-prone).

---

# Microservices Fundamentals & Architecture: Service Boundaries Definition (Continued)

---

## Question 9: How Do You Handle Backward Compatibility and Graceful Degradation When Service Boundaries Evolve or Dependencies Fail?

### Detailed Answer:

Service boundaries are not immutable; APIs evolve, services are retired, and dependencies fail. Graceful degradation ensures that when a service boundary changes or a dependency fails, the entire system doesn't cascade into failure. **Backward compatibility** allows consumers to continue functioning with older API versions; **graceful degradation** means the system offers reduced functionality rather than complete failure. Together, they decouple services temporally (consumers and producers don't need to deploy simultaneously) and operationally (services can fail without bringing down dependent systems).

**Contract evolution** is where backward compatibility matters most. When Order service adds a new required field to its API response, old consumers expecting the old schema fail. Prevent this through **additive-only changes**: add new fields as optional (nullable with defaults), never remove or change field types, never make previously-optional fields required. If you need to break contract, introduce a new API version (`/v2/orders`) alongside the old one (`/v1/orders`). Both versions exist for a deprecation window (e.g., 12 months), giving consumers time to migrate. Azure API Management handles version routing transparently; consumers can migrate at their own pace.

**Graceful degradation patterns** handle dependency failures without total system collapse. The **Bulkhead pattern** isolates failures—if Inventory service fails, Order service can still create orders without inventory validation, flagging them for manual review. The **Circuit Breaker pattern** stops making requests to a failing service after N consecutive failures, fast-failing new requests rather than timing out. The **Retry pattern** with exponential backoff automatically retries transient failures. **Fallback strategies** provide degraded responses: if product recommendations service fails, return empty recommendations rather than error. In Azure, implement these via **Polly library** (resilience policies), **Azure Service Bus Dead Letter Queues** (capture failed messages for manual handling), and **Azure Functions Durable Functions** (retries and compensation).

**Feature flags** enable graceful rollout of boundary changes. Deploy new code with feature flags disabled; gradually enable for canary users (1% → 10% → 100%) while monitoring. If errors spike, disable the flag immediately. This decouples deployment from feature activation, reducing risk. Azure App Configuration integrates feature flags with Azure DevOps and provides real-time flag updates without redeployment.

### Explanation:

**Importance & Use Cases:**
- **Decoupling deployment timelines**: Consumers and producers deploy independently; old consumer works with new producer and vice versa.
- **Operational resilience**: Transient failures (network timeout, service restart) are retried automatically; permanent failures degrade gracefully.
- **User experience**: Reduced functionality is better than error page; users proceed with what's available.
- **Compliance and audit**: Failed requests are logged for audit; manual review processes ensure no data is lost.

**Benefits & Trade-offs:**
- **Benefit**: Reduced coordination between teams, operational resilience, predictable failure modes.
- **Trade-off**: Complexity (multiple code paths for different API versions, fallback logic); potential stale data (if fallback uses cached data); testing burden (must test all combinations of versions).
- **Interview perspective**: Shows maturity in designing for failure and operational realities of distributed systems.

### C# Implementation:

```csharp
using Microsoft.FeatureManagement;
using Polly;
using Polly.CircuitBreaker;
using Polly.Retry;
using Polly.Timeout;
using System.Diagnostics;

// ============ BACKWARD-COMPATIBLE API VERSIONING ============

/// <summary>
/// Order API with versioning support
/// V1 (legacy): minimal fields
/// V2 (current): additional optional fields
/// Both versions coexist for backward compatibility
/// </summary>

[ApiController]
[Route("api/v{version:apiVersion}/orders")]
[ApiVersion("1.0")]
[ApiVersion("2.0")]
[Authorize]
public class OrdersController : ControllerBase
{
    private readonly OrderService _orderService;
    private readonly ILogger<OrdersController> _logger;

    public OrdersController(OrderService orderService, ILogger<OrdersController> logger)
    {
        _orderService = orderService;
        _logger = logger;
    }

    /// <summary>
    /// GET /api/v1/orders/{id} - Legacy endpoint (read-only format)
    /// Returns minimal fields for backward compatibility
    /// </summary>
    [HttpGet("{id}", Name = "GetOrderV1")]
    [MapToApiVersion("1.0")]
    [ProducesResponseType(typeof(OrderResponseV1), StatusCodes.Status200OK)]
    public async Task<ActionResult<OrderResponseV1>> GetOrderV1Async(string id)
    {
        var order = await _orderService.GetOrderAsync(id);
        if (order == null)
            return NotFound();

        // Return only V1 fields
        return Ok(new OrderResponseV1
        {
            OrderId = order.OrderId,
            CustomerId = order.CustomerId,
            TotalAmount = order.TotalAmount,
            Status = order.Status
        });
    }

    /// <summary>
    /// GET /api/v2/orders/{id} - Current endpoint
    /// Returns all fields including new optional ones
    /// </summary>
    [HttpGet("{id}", Name = "GetOrderV2")]
    [MapToApiVersion("2.0")]
    [ProducesResponseType(typeof(OrderResponseV2), StatusCodes.Status200OK)]
    public async Task<ActionResult<OrderResponseV2>> GetOrderV2Async(string id)
    {
        var order = await _orderService.GetOrderAsync(id);
        if (order == null)
            return NotFound();

        // Return all fields including new ones (backward compatible)
        return Ok(new OrderResponseV2
        {
            OrderId = order.OrderId,
            CustomerId = order.CustomerId,
            TotalAmount = order.TotalAmount,
            Status = order.Status,
            ShippingAddress = order.ShippingAddress,        // New optional field
            EstimatedDeliveryDate = order.EstimatedDeliveryDate, // New optional field
            TrackingNumber = order.TrackingNumber            // New optional field
        });
    }

    /// <summary>
    /// POST /api/v1/orders - Legacy create (minimal fields)
    /// </summary>
    [HttpPost(Name = "CreateOrderV1")]
    [MapToApiVersion("1.0")]
    [ProducesResponseType(typeof(OrderResponseV1), StatusCodes.Status201Created)]
    public async Task<ActionResult<OrderResponseV1>> CreateOrderV1Async(
        [FromBody] CreateOrderRequestV1 request)
    {
        if (!ModelState.IsValid)
            return BadRequest(ModelState);

        var order = await _orderService.CreateOrderAsync(
            request.CustomerId,
            request.TotalAmount);

        return CreatedAtRoute("GetOrderV1", new { id = order.OrderId }, 
            new OrderResponseV1(order));
    }

    /// <summary>
    /// POST /api/v2/orders - Current create (with optional fields)
    /// </summary>
    [HttpPost(Name = "CreateOrderV2")]
    [MapToApiVersion("2.0")]
    [ProducesResponseType(typeof(OrderResponseV2), StatusCodes.Status201Created)]
    public async Task<ActionResult<OrderResponseV2>> CreateOrderV2Async(
        [FromBody] CreateOrderRequestV2 request)
    {
        if (!ModelState.IsValid)
            return BadRequest(ModelState);

        var order = await _orderService.CreateOrderAsync(
            request.CustomerId,
            request.TotalAmount,
            request.ShippingAddress);      // New optional field

        return CreatedAtRoute("GetOrderV2", new { id = order.OrderId },
            new OrderResponseV2(order));
    }
}

// ============ API CONTRACTS ============

// V1 Response (minimal)
public record OrderResponseV1(
    string OrderId,
    string CustomerId,
    decimal TotalAmount,
    string Status
);

// V2 Response (current - backward compatible extension)
public record OrderResponseV2(
    string OrderId,
    string CustomerId,
    decimal TotalAmount,
    string Status,
    string? ShippingAddress = null,           // New optional field
    DateTime? EstimatedDeliveryDate = null,   // New optional field
    string? TrackingNumber = null             // New optional field
)
{
    public OrderResponseV2(Order order) : this(
        order.OrderId,
        order.CustomerId,
        order.TotalAmount,
        order.Status,
        order.ShippingAddress,
        order.EstimatedDeliveryDate,
        order.TrackingNumber)
    {
    }
};

// V1 Request (minimal)
public record CreateOrderRequestV1(
    string CustomerId,
    decimal TotalAmount
);

// V2 Request (current - backward compatible extension)
public record CreateOrderRequestV2(
    string CustomerId,
    decimal TotalAmount,
    string? ShippingAddress = null  // New optional field
);

// ============ GRACEFUL DEGRADATION WITH RESILIENCE POLICIES ============

/// <summary>
/// Resilience policies for graceful degradation
/// Implements retry, circuit breaker, timeout, and fallback
/// </summary>
public class ResilientInventoryService
{
    private readonly HttpClient _httpClient;
    private readonly IAsyncPolicy<HttpResponseMessage> _resilientPolicy;
    private readonly ILogger<ResilientInventoryService> _logger;

    public ResilientInventoryService(
        HttpClient httpClient,
        ILogger<ResilientInventoryService> logger)
    {
        _httpClient = httpClient;
        _logger = logger;

        // Build composite policy: timeout → retry → circuit breaker → fallback
        _resilientPolicy = CreateResilientPolicy();
    }

    /// <summary>
    /// Build composite Polly policy
    /// </summary>
    private IAsyncPolicy<HttpResponseMessage> CreateResilientPolicy()
    {
        // 1. Timeout policy: abort if response takes >2 seconds
        var timeoutPolicy = Policy.TimeoutAsync<HttpResponseMessage>(
            TimeSpan.FromSeconds(2),
            TimeoutStrategy.Optimistic); // Optimistic = timeout doesn't abort operation

        // 2. Retry policy: retry 3 times on transient failures (500, 503, 408)
        var retryPolicy = Policy
            .Handle<HttpRequestException>()
            .Or<OperationCanceledException>()
            .OrResult<HttpResponseMessage>(r =>
                r.StatusCode == System.Net.HttpStatusCode.InternalServerError ||
                r.StatusCode == System.Net.HttpStatusCode.ServiceUnavailable ||
                r.StatusCode == System.Net.HttpStatusCode.RequestTimeout)
            .WaitAndRetryAsync(
                retryCount: 3,
                sleepDurationProvider: attempt =>
                    TimeSpan.FromMilliseconds(Math.Pow(2, attempt) * 100), // Exponential backoff
                onRetry: (outcome, timespan, retryCount, context) =>
                {
                    _logger.LogWarning(
                        "Retry {RetryCount} after {DelayMs}ms due to: {Outcome}",
                        retryCount,
                        timespan.TotalMilliseconds,
                        outcome.Exception?.Message ?? outcome.Result?.StatusCode.ToString());
                });

        // 3. Circuit breaker: stop calling after 5 failures in 30 seconds
        var circuitBreakerPolicy = Policy
            .Handle<HttpRequestException>()
            .Or<OperationCanceledException>()
            .OrResult<HttpResponseMessage>(r => !r.IsSuccessStatusCode)
            .CircuitBreakerAsync<HttpResponseMessage>(
                handledEventsAllowedBeforeBreaking: 5,
                durationOfBreak: TimeSpan.FromSeconds(30),
                onBreak: (outcome, duration) =>
                {
                    _logger.LogError(
                        "Circuit breaker opened for {Duration}s due to repeated failures",
                        duration.TotalSeconds);
                },
                onReset: () =>
                {
                    _logger.LogInformation("Circuit breaker reset; resuming requests");
                });

        // 4. Fallback policy: return cached/degraded response
        var fallbackPolicy = Policy
            .Handle<HttpRequestException>()
            .Or<OperationCanceledException>()
            .Or<BrokenCircuitException>()
            .OrResult<HttpResponseMessage>(r => !r.IsSuccessStatusCode)
            .FallbackAsync<HttpResponseMessage>(
                fallbackAction: (outcome, context) =>
                {
                    _logger.LogWarning("Inventory service unavailable; using fallback");
                    
                    // Fallback: return "assume inventory available" to avoid blocking orders
                    return Task.FromResult(new HttpResponseMessage(System.Net.HttpStatusCode.OK)
                    {
                        Content = new StringContent(
                            System.Text.Json.JsonSerializer.Serialize(new
                            {
                                available = true,
                                isFallback = true,
                                reason = "Service degraded; assuming inventory available"
                            }),
                            System.Text.Encoding.UTF8,
                            "application/json")
                    });
                });

        // Combine all policies: wrap in order from outermost to innermost
        return Policy.WrapAsync(timeoutPolicy, retryPolicy, circuitBreakerPolicy, fallbackPolicy);
    }

    /// <summary>
    /// Check inventory with graceful degradation
    /// Returns true even if service fails (assumes available)
    /// </summary>
    public async Task<InventoryCheckResult> CheckInventoryAsync(string sku, int quantity)
    {
        var stopwatch = Stopwatch.StartNew();
        
        try
        {
            var response = await _resilientPolicy.ExecuteAsync(async () =>
            {
                return await _httpClient.PostAsync(
                    "https://inventory-service/api/v1/check",
                    new StringContent(
                        System.Text.Json.JsonSerializer.Serialize(new { sku, quantity }),
                        System.Text.Encoding.UTF8,
                        "application/json"));
            });

            stopwatch.Stop();

            if (response.IsSuccessStatusCode)
            {
                var content = await response.Content.ReadAsStringAsync();
                var result = System.Text.Json.JsonSerializer.Deserialize<InventoryCheckResult>(content);
                
                _logger.LogInformation(
                    "Inventory check succeeded in {ElapsedMs}ms: SKU={SKU}, Available={Available}",
                    stopwatch.ElapsedMilliseconds,
                    sku,
                    result?.Available);
                
                return result ?? new InventoryCheckResult { Available = true };
            }
            else
            {
                _logger.LogWarning(
                    "Inventory service returned {StatusCode}; degrading gracefully",
                    response.StatusCode);

                // Fallback triggered above; assume available
                return new InventoryCheckResult
                {
                    Available = true,
                    IsDegraded = true,
                    Reason = "Service unavailable; assuming inventory available"
                };
            }
        }
        catch (BrokenCircuitException ex)
        {
            _logger.LogWarning(ex, "Inventory service circuit breaker open; degrading gracefully");
            
            // Degrade gracefully
            return new InventoryCheckResult
            {
                Available = true,
                IsDegraded = true,
                Reason = "Service temporarily unavailable; circuit breaker engaged"
            };
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Unexpected error checking inventory; degrading gracefully");
            
            // Final fallback
            return new InventoryCheckResult
            {
                Available = true,
                IsDegraded = true,
                Reason = $"Error checking inventory: {ex.Message}"
            };
        }
    }
}

// ============ FEATURE FLAGS FOR GRACEFUL ROLLOUT ============

/// <summary>
/// Service using feature flags to control rollout of boundary changes
/// </summary>
public class FeatureFlaggedOrderService
{
    private readonly OrderService _orderService;
    private readonly IFeatureManager _featureManager;
    private readonly ILogger<FeatureFlaggedOrderService> _logger;

    public FeatureFlaggedOrderService(
        OrderService orderService,
        IFeatureManager featureManager,
        ILogger<FeatureFlaggedOrderService> logger)
    {
        _orderService = orderService;
        _featureManager = featureManager;
        _logger = logger;
    }

    /// <summary>
    /// Create order with feature flag controlling new behavior
    /// </summary>
    public async Task<Order> CreateOrderWithFeatureFlagAsync(
        string customerId,
        decimal totalAmount,
        string? shippingAddress = null)
    {
        // Check if new shipping feature is enabled
        var useShippingFeature = await _featureManager.IsEnabledAsync("NewShippingFeature");

        _logger.LogInformation(
            "Creating order with NewShippingFeature={Enabled}",
            useShippingFeature);

        if (useShippingFeature)
        {
            // New behavior: calculate shipping address
            var calculatedAddress = shippingAddress ?? await CalculateDefaultShippingAsync(customerId);
            
            return await _orderService.CreateOrderWithShippingAsync(
                customerId,
                totalAmount,
                calculatedAddress);
        }
        else
        {
            // Legacy behavior: ignore shipping
            return await _orderService.CreateOrderAsync(customerId, totalAmount);
        }
    }

    /// <summary>
    /// Get order with optional enrichment based on feature flag
    /// </summary>
    public async Task<OrderResponse> GetOrderWithOptionalEnrichmentAsync(string orderId)
    {
        var order = await _orderService.GetOrderAsync(orderId);
        if (order == null)
            return null;

        // Check if enrichment feature is enabled (gradually enable for users)
        var useEnrichment = await _featureManager.IsEnabledAsync("OrderEnrichmentFeature");

        if (useEnrichment)
        {
            // Enrich with additional data (recommendations, similar orders)
            var enriched = await EnrichOrderAsync(order);
            return new OrderResponse(enriched);
        }
        else
        {
            // Return plain order
            return new OrderResponse(order);
        }
    }

    private async Task<string> CalculateDefaultShippingAsync(string customerId)
    {
        // Calculate default shipping address
        return "123 Default St, Default City";
    }

    private async Task<Order> EnrichOrderAsync(Order order)
    {
        // Add enrichment data (recommendations, etc.)
        return order;
    }
}

/// <summary>
/// Feature flag configuration for canary rollout
/// </summary>
public class FeatureFlagOptions
{
    public Dictionary<string, FeatureFlagConfig> Features { get; set; } = new();
}

public class FeatureFlagConfig
{
    public bool Enabled { get; set; }
    public double PercentageEnabled { get; set; } // Gradual rollout: 0-100%
    public List<string>? EnabledUsers { get; set; }
    public List<string>? EnabledTenants { get; set; }
}

// ============ DATA MODELS ============

public class Order
{
    public string OrderId { get; set; }
    public string CustomerId { get; set; }
    public decimal TotalAmount { get; set; }
    public string Status { get; set; }
    public string? ShippingAddress { get; set; }
    public DateTime? EstimatedDeliveryDate { get; set; }
    public string? TrackingNumber { get; set; }
}

public record OrderResponse(Order Order);

public record InventoryCheckResult
{
    public bool Available { get; set; }
    public bool IsDegraded { get; set; }
    public string? Reason { get; set; }
}

public interface IOrderService
{
    Task<Order> GetOrderAsync(string id);
    Task<Order> CreateOrderAsync(string customerId, decimal totalAmount);
    Task<Order> CreateOrderWithShippingAsync(string customerId, decimal totalAmount, string shippingAddress);
}

public class OrderService : IOrderService
{
    public async Task<Order> GetOrderAsync(string id)
    {
        return new Order { OrderId = id, CustomerId = "C123", TotalAmount = 100 };
    }

    public async Task<Order> CreateOrderAsync(string customerId, decimal totalAmount)
    {
        return new Order
        {
            OrderId = Guid.NewGuid().ToString(),
            CustomerId = customerId,
            TotalAmount = totalAmount,
            Status = "Pending"
        };
    }

    public async Task<Order> CreateOrderWithShippingAsync(
        string customerId,
        decimal totalAmount,
        string shippingAddress)
    {
        return new Order
        {
            OrderId = Guid.NewGuid().ToString(),
            CustomerId = customerId,
            TotalAmount = totalAmount,
            Status = "Pending",
            ShippingAddress = shippingAddress
        };
    }
}

// ============ DEPENDENCY INJECTION ============

public static class BackwardCompatibilityExtensions
{
    public static IServiceCollection AddBackwardCompatibilitySupport(
        this IServiceCollection services,
        IConfiguration configuration)
    {
        // Add API versioning
        services.AddApiVersioning(options =>
        {
            options.DefaultApiVersion = new ApiVersion(2, 0);
            options.AssumeDefaultVersionWhenUnspecified = true;
            options.ReportApiVersions = true;
        })
        .AddApiExplorer(options =>
        {
            options.GroupNameFormat = "'v'VVV";
            options.SubstituteApiVersionInUrl = true;
        });

        // Add resilience patterns
        services.AddHttpClient<ResilientInventoryService>();

        // Add feature flags
        services.AddFeatureManagement(configuration.GetSection("FeatureManagement"));

        // Add order services
        services.AddScoped<OrderService>();
        services.AddScoped<FeatureFlaggedOrderService>();

        return services;
    }
}
```

### Follow-Up Questions:

**Q1: How do you version database schemas when service boundaries evolve without breaking existing data?**
* **Answer:** Use **zero-downtime migrations** with backward-compatible schema changes. Steps: (1) Deploy code that reads from both old and new columns (if renaming); (2) Run migration to add new column/schema, keeping old; (3) Update code to write to both; (4) Backfill historical data; (5) Update code to read only from new column; (6) Remove old column in future release. For Azure SQL, use **feature flags to control which schema version is active**; gradually shift traffic to new schema version. Never remove columns immediately; deprecate first (mark as not used in app), then remove in next major version. For data versioning, store version number in each row; code handles both versions. Use Azure Data Factory for large-scale migrations without locking tables.

**Q2: How do you handle API deprecation when you have thousands of external consumers using legacy versions?**
* **Answer:** Follow **HTTP Sunset header** (RFC 8594): include `Sunset: Wed, 21 Dec 2025 23:59:59 GMT` in responses to deprecated versions, notifying consumers of retirement date. Send email notifications and post API deprecation roadmap 6–12 months before sunset. Provide migration guides and tools (SDK updates, code examples). Use analytics to identify still-active consumers of deprecated version; reach out directly if <10% of traffic uses it. Consider **extended support window** for paying customers; keep deprecated version alive longer at premium price. In Azure, use Application Insights to track usage of deprecated endpoints; set alerts if usage drops below expected (may indicate customer left or didn't migrate). Have dedicated team to support deprecation migration; don't force users to migrate without help.

**Q3: What's the right balance between backward compatibility and forward progress—how long to maintain old versions?**
* **Answer:** Common rule: **maintain current and N-1 previous versions**; retire oldest version when N+1 releases. For SaaS: typically 12–24 months of backward compatibility; give customers time to plan migrations. For internal services: 3–6 months (faster pace). Consider customer impact: changes breaking contract deserve longer support window (18+ months); additive changes (new optional fields) need shorter window (6 months). Use **semantic versioning**: major version breaks compatibility, minor version is backward-compatible. Example: v1.0 → v1.1 (optional field added), consumers don't need to upgrade; v2.0 (breaking changes), consumers must upgrade within 12 months. Document support timeline clearly; surprises anger customers. For critical services, maintain LTS (long-term support) versions with extended support periods.

**Q4: How do you test backward compatibility across multiple API versions?**
* **Answer:** Create **contract tests** for each API version; test that both V1 and V2 endpoints work, return appropriate fields. Use **consumer-driven contracts**: consumers define what fields they need; contract tests verify older and newer versions satisfy these needs. Implement **pact tests** (CDCTs) for service-to-service contracts: Order service writes pact, Inventory service verifies it can fulfill the contract in both old and new versions. Use **property-based testing** (QuickCheck, Hypothesis) to generate random data; verify both versions handle it correctly. Create matrix test: test all combinations (V1 consumer + V2 producer, V2 consumer + V1 producer, etc.). For databases, test schema migrations: backup prod database, run migration on backup, verify data integrity before applying to prod. Include backward-compatibility tests in CI/CD pipeline; fail if old version client can't communicate with new service or vice versa.

**Q5: When should you use feature flags vs. API versioning vs. gradual rollout—how do you choose?**
* **Answer:** Use all three together: **Feature flags** control which code paths activate (internal teams, canary users, percentage-based); **API versioning** provides external interface stability (backward compatibility for external consumers); **gradual rollout** (canary, blue-green) deploys safely with monitoring. Example workflow: (1) Deploy new code behind feature flag (disabled for 99% of users), (2) Enable for 5% canary users while monitoring errors in Application Insights, (3) If healthy, enable for 50%, then 100%, (4) Keep old API version running for 12 months, (5) Add deprecation warnings to old version, (6) Retire old version after sunset date. Feature flags are faster to revert (no rollback deploy); API versioning is for long-term compatibility; gradual rollout manages risk. Don't use feature flags for long-term compatibility—disable them after rollout. Use API versioning for permanent interfaces.

---

## Question 10: How Do You Design Service Boundaries for Cost Optimization While Maintaining Operational Efficiency and Service Level Agreements (SLAs)?

### Detailed Answer:

Service boundaries fundamentally impact cloud infrastructure costs. Finely-grained services create overhead (more deployments, more instances, more networking, more monitoring); coarse-grained services create inefficiency (components scaling together that scale independently, shared databases limiting partitioning). Optimal boundary design balances business needs (independent deployment, team autonomy) with cost and operational efficiency (minimal infrastructure overhead, appropriate scaling granularity).

**Infrastructure costs** scale with service count: each service requires monitoring agents, logging, health checks, and base instance hours. On Azure, deploying 20 microservices costs significantly more than 5 services, even with same total compute. Consolidate related services if they always scale together, share dependencies, or lack independent deployment drivers. However, don't over-consolidate to save costs—consolidation prevents independent scaling and increases failure blast radius (one service's spike affects others).

**Scaling costs** are where boundary design decisions show. If Order and Inventory services have completely different traffic patterns (Order spikes 10x during sales, Inventory is steady), separate them to scale independently. If they have correlated patterns, combining them into one service with unified autoscaling is more cost-efficient. Analyze traffic patterns using Application Insights; if services scale in lockstep, consider consolidation. Use **reserved capacity** (Azure Reserved Instances) for predictable baseline load; use **spot instances** or **dynamic scaling** for burst traffic. Design services so each scales independently to match its actual demand pattern.

**SLAs and reliability** directly affect costs. A 99.99% SLA (four 9s) requires redundancy across availability zones, active-passive failover, and extensive testing—expensive. A 99.9% SLA (three 9s) is simpler and cheaper. Design service boundaries by reliability requirement: separate critical-path services (authentication, payment processing) requiring high SLA from non-critical services (recommendations, personalization) where occasional downtime is acceptable. This allows per-service SLA design: pay for four 9s only for services that need it.

**Data transfer costs** are significant but often overlooked. Each inter-service call across networks has data transfer cost (negligible within region, ~$0.01/GB between regions). Boundaries that require frequent cross-region communication are expensive. Design boundaries to minimize cross-boundary chatter; co-locate services that communicate heavily in same region. Use **data residency boundaries**: separate per-region services instead of one global service making cross-region calls.

### Explanation:

**Importance & Use Cases:**
- **Cost control in SaaS**: Optimize infrastructure spend per customer to maintain profitability.
- **Predictable spend**: Right-sized services with appropriate SLAs allow accurate cost forecasting.
- **Efficiency at scale**: As customer base grows, cost-optimized boundaries scale cost-efficiently.
- **Competitive advantage**: Better margins enable aggressive pricing or feature investment.

**Benefits & Trade-offs:**
- **Benefit**: Lower infrastructure costs, predictable scaling, appropriate SLAs by service.
- **Trade-off**: Requires continuous monitoring and adjustment; may constrain architectural freedom; risk of under-provisioning and SLA violations if cost optimization is too aggressive.
- **Interview perspective**: Shows business acumen and practical understanding of cloud economics—critical for production SaaS systems.

### C# Implementation:

```csharp
using Azure.Monitor.OpenTelemetry;
using Microsoft.ApplicationInsights;
using Microsoft.AspNetCore.Mvc;
using System.Diagnostics;

// ============ COST-AWARE SERVICE DESIGN ============

/// <summary>
/// Service with cost optimization and SLA enforcement
/// Scales independently based on actual demand
/// Implements SLA monitoring and alerting
/// </summary>
public class CostOptimizedOrderService
{
    private readonly IOrderRepository _orderRepository;
    private readonly TelemetryClient _telemetryClient;
    private readonly ILogger<CostOptimizedOrderService> _logger;

    public CostOptimizedOrderService(
        IOrderRepository orderRepository,
        TelemetryClient telemetryClient,
        ILogger<CostOptimizedOrderService> logger)
    {
        _orderRepository = orderRepository;
        _telemetryClient = telemetryClient;
        _logger = logger;
    }

    /// <summary>
    /// Create order with cost tracking
    /// Measures operation cost (CPU, memory, data transfer)
    /// </summary>
    public async Task<Order> CreateOrderAsync(string customerId, decimal totalAmount)
    {
        var stopwatch = Stopwatch.StartNew();
        var operationCost = new OperationCostMetrics();

        try
        {
            // Measure operation
            var order = new Order
            {
                OrderId = Guid.NewGuid().ToString(),
                CustomerId = customerId,
                TotalAmount = totalAmount,
                Status = "Pending",
                CreatedAt = DateTime.UtcNow
            };

            // Measure database operation cost
            var dbStopwatch = Stopwatch.StartNew();
            await _orderRepository.CreateAsync(order);
            dbStopwatch.Stop();

            operationCost.DatabaseOperationTimeMs = dbStopwatch.ElapsedMilliseconds;
            operationCost.DatabaseOperationCost = EstimateDatabaseCost(order);

            stopwatch.Stop();

            // Track operation cost
            operationCost.TotalTimeMs = stopwatch.ElapsedMilliseconds;
            operationCost.EstimatedComputeCost = EstimateComputeCost(stopwatch.ElapsedMilliseconds);

            // Log cost metrics for billing and optimization
            _telemetryClient.TrackEvent("OrderCreated", new Dictionary<string, string>
            {
                { "order_id", order.OrderId },
                { "customer_id", customerId },
                { "total_amount", totalAmount.ToString() }
            }, new Dictionary<string, double>
            {
                { "duration_ms", operationCost.TotalTimeMs },
                { "db_operation_time_ms", operationCost.DatabaseOperationTimeMs },
                { "estimated_compute_cost_usd", operationCost.EstimatedComputeCost },
                { "estimated_db_cost_usd", operationCost.DatabaseOperationCost }
            });

            _logger.LogInformation(
                "Order created (Cost: ${ComputeCost:F4} compute + ${DbCost:F4} DB)",
                operationCost.EstimatedComputeCost,
                operationCost.DatabaseOperationCost);

            return order;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to create order");
            _telemetryClient.TrackException(ex);
            throw;
        }
    }

    private double EstimateComputeCost(long elapsedMs)
    {
        // Estimate cost: 1 vCPU per 1000ms = $0.0002 per vCPU-ms (rough estimate)
        const double cpuCostPerMs = 0.0000002;
        return elapsedMs * cpuCostPerMs;
    }

    private double EstimateDatabaseCost(Order order)
    {
        // Estimate cost: DTU-based pricing
        // Azure SQL Database: ~$0.50 per DTU-day
        // Rough estimate: 1 write operation = 0.001 DTUs
        const double dtusPerWrite = 0.001;
        const double costPerDtuDay = 0.50;
        const double minutesPerDay = 24 * 60;
        
        return (dtusPerWrite * costPerDtuDay) / minutesPerDay;
    }
}

// ============ SCALING STRATEGY BY SLA REQUIREMENT ============

/// <summary>
/// Service configuration based on SLA requirement
/// Higher SLA = more expensive (redundancy, failover)
/// </summary>
public enum ServiceSla
{
    /// <summary>
    /// Single instance, no redundancy
    /// 99% uptime = 3.65 days downtime/year
    /// </summary>
    Standard = 99,

    /// <summary>
    /// Multi-instance with load balancing
    /// 99.9% uptime = 8.77 hours downtime/year
    /// </summary>
    Premium = 999,

    /// <summary>
    /// Multi-region, active-active failover
    /// 99.99% uptime = 52.6 minutes downtime/year
    /// </summary>
    Enterprise = 9999
}

public class SlaBasedServiceConfiguration
{
    public ServiceSla RequiredSla { get; set; }
    public int MinimumInstances { get; set; }
    public int MaximumInstances { get; set; }
    public int ScaleCpuThresholdPercent { get; set; }
    public string AzureRegions { get; set; }
    public double EstimatedAnnualCostUsd { get; set; }

    public static SlaBasedServiceConfiguration ForSla(ServiceSla sla)
    {
        return sla switch
        {
            // Standard: single instance, acceptable downtime
            ServiceSla.Standard => new SlaBasedServiceConfiguration
            {
                RequiredSla = sla,
                MinimumInstances = 1,
                MaximumInstances = 3,
                ScaleCpuThresholdPercent = 70,
                AzureRegions = "Single region",
                EstimatedAnnualCostUsd = 5000
            },

            // Premium: multi-instance, zone redundancy
            ServiceSla.Premium => new SlaBasedServiceConfiguration
            {
                RequiredSla = sla,
                MinimumInstances = 2,
                MaximumInstances = 10,
                ScaleCpuThresholdPercent = 60,
                AzureRegions = "Single region, 2+ availability zones",
                EstimatedAnnualCostUsd = 15000
            },

            // Enterprise: multi-region active-active
            ServiceSla.Enterprise => new SlaBasedServiceConfiguration
            {
                RequiredSla = sla,
                MinimumInstances = 3,
                MaximumInstances = 20,
                ScaleCpuThresholdPercent = 50,
                AzureRegions = "Multiple regions (primary + backup)",
                EstimatedAnnualCostUsd = 50000
            },

            _ => throw new ArgumentException($"Unknown SLA: {sla}")
        };
    }
}

// ============ SLA MONITORING AND ENFORCEMENT ============

/// <summary>
/// SLA monitoring to ensure service meets commitments
/// Tracks availability, latency, and error rate
/// </summary>
public class SlaMonitor
{
    private readonly TelemetryClient _telemetryClient;
    private readonly ILogger<SlaMonitor> _logger;
    private readonly ServiceSla _requiredSla;

    public SlaMonitor(
        TelemetryClient telemetryClient,
        ILogger<SlaMonitor> logger,
        ServiceSla requiredSla)
    {
        _telemetryClient = telemetryClient;
        _logger = logger;
        _requiredSla = requiredSla;
    }

    /// <summary>
    /// Middleware to measure SLA metrics
    /// </summary>
    public async Task<IActionResult> MeasureSlaAsync(
        string serviceName,
        Func<Task<IActionResult>> operation)
    {
        var stopwatch = Stopwatch.StartNew();
        IActionResult? result = null;
        Exception? exception = null;

        try
        {
            result = await operation();
            return result;
        }
        catch (Exception ex)
        {
            exception = ex;
            throw;
        }
        finally
        {
            stopwatch.Stop();

            // Record SLA metrics
            var isSuccess = exception == null && (result?.GetType().Name == "OkResult" || 
                                                  result?.GetType().Name.Contains("Ok") == true);

            var slaMetrics = new SlaMetrics
            {
                ServiceName = serviceName,
                ResponseTimeMs = stopwatch.ElapsedMilliseconds,
                IsSuccess = isSuccess,
                Timestamp = DateTime.UtcNow
            };

            _telemetryClient.TrackEvent("SlaMetric", new Dictionary<string, string>
            {
                { "service", serviceName },
                { "success", isSuccess.ToString() }
            }, new Dictionary<string, double>
            {
                { "response_time_ms", slaMetrics.ResponseTimeMs }
            });

            // Check SLA violation
            CheckSlaViolation(slaMetrics);
        }
    }

    private void CheckSlaViolation(SlaMetrics metrics)
    {
        var latencySlaMs = _requiredSla switch
        {
            ServiceSla.Standard => 5000,    // 5s acceptable
            ServiceSla.Premium => 1000,     // 1s acceptable
            ServiceSla.Enterprise => 500,   // 500ms acceptable
            _ => 5000
        };

        if (metrics.ResponseTimeMs > latencySlaMs)
        {
            _logger.LogWarning(
                "SLA latency violation: {ServiceName} took {ActualMs}ms, SLA is {SlaMs}ms",
                metrics.ServiceName,
                metrics.ResponseTimeMs,
                latencySlaMs);

            _telemetryClient.TrackEvent("SlaViolation", new Dictionary<string, string>
            {
                { "service", metrics.ServiceName },
                { "violation_type", "latency" },
                { "sla", _requiredSla.ToString() }
            }, new Dictionary<string, double>
            {
                { "actual_ms", metrics.ResponseTimeMs },
                { "sla_ms", latencySlaMs }
            });
        }
    }
}

// ============ COST ANALYSIS AND OPTIMIZATION ============

/// <summary>
/// Analyze service costs and identify optimization opportunities
/// </summary>
public class ServiceCostAnalyzer
{
    private readonly TelemetryClient _telemetryClient;
    private readonly ILogger<ServiceCostAnalyzer> _logger;

    public ServiceCostAnalyzer(
        TelemetryClient telemetryClient,
        ILogger<ServiceCostAnalyzer> logger)
    {
        _telemetryClient = telemetryClient;
        _logger = logger;
    }

    /// <summary>
    /// Analyze service efficiency and recommend optimizations
    /// </summary>
    public CostOptimizationReport AnalyzeCosts(
        string serviceName,
        ServiceMetrics metrics,
        SlaBasedServiceConfiguration slaConfig)
    {
        var report = new CostOptimizationReport
        {
            ServiceName = serviceName,
            AnalysisDate = DateTime.UtcNow,
            RequiredSla = slaConfig.RequiredSla.ToString()
        };

        // 1. Analyze resource utilization
        if (metrics.AverageCpuUsagePercent < 30)
        {
            report.OptimizationOpportunities.Add(
                $"Low CPU utilization ({metrics.AverageCpuUsagePercent}%). " +
                $"Consider reducing minimum instance count or consolidating with other services.");
        }

        if (metrics.AverageCpuUsagePercent > 80)
        {
            report.OptimizationOpportunities.Add(
                $"High CPU utilization ({metrics.AverageCpuUsagePercent}%). " +
                $"Consider increasing instance size or splitting service.");
        }

        // 2. Analyze error rate and SLA
        if (metrics.ErrorRatePercent > 0.1)
        {
            report.OptimizationOpportunities.Add(
                $"High error rate ({metrics.ErrorRatePercent}%). " +
                $"Investigate and fix; may indicate service not meeting SLA.");
        }

        // 3. Analyze traffic patterns
        if (metrics.HasPeakTrafficVariance)
        {
            report.OptimizationOpportunities.Add(
                $"High traffic variance detected. " +
                $"Consider implementing scheduled scaling or reserved capacity for baseline.");
        }

        // 4. Analyze inter-service communication
        if (metrics.AverageInterServiceLatencyMs > 500)
        {
            report.OptimizationOpportunities.Add(
                $"High inter-service latency ({metrics.AverageInterServiceLatencyMs}ms). " +
                $"Consider co-locating services or consolidating boundaries.");
        }

        // Estimate cost impact
        var currentAnnualCost = slaConfig.EstimatedAnnualCostUsd;
        var potentialSavings = 0.0;

        if (metrics.AverageCpuUsagePercent < 30)
            potentialSavings += currentAnnualCost * 0.20; // 20% savings

        if (metrics.ErrorRatePercent < 0.01)
            potentialSavings += currentAnnualCost * 0.10; // 10% savings

        report.EstimatedAnnualCostUsd = currentAnnualCost;
        report.PotentialAnnualSavingsUsd = potentialSavings;
        report.SavingsPercentage = potentialSavings / currentAnnualCost * 100;

        _logger.LogInformation(
            "Cost analysis for {ServiceName}: ${Current}/year, " +
            "potential savings: ${Savings}/year ({SavingsPercent}%)",
            serviceName,
            currentAnnualCost,
            potentialSavings,
            report.SavingsPercentage);

        return report;
    }
}

// ============ DATA MODELS ============

public record OperationCostMetrics
{
    public double TotalTimeMs { get; set; }
    public double DatabaseOperationTimeMs { get; set; }
    public double EstimatedComputeCost { get; set; }
    public double DatabaseOperationCost { get; set; }
}

public record SlaMetrics
{
    public string ServiceName { get; set; }
    public long ResponseTimeMs { get; set; }
    public bool IsSuccess { get; set; }
    public DateTime Timestamp { get; set; }
}

public record ServiceMetrics
{
    public string ServiceName { get; set; }
    public double AverageCpuUsagePercent { get; set; }
    public double AverageMemoryUsagePercent { get; set; }
    public double ErrorRatePercent { get; set; }
    public double AverageLatencyMs { get; set; }
    public double AverageInterServiceLatencyMs { get; set; }
    public int AverageInstanceCount { get; set; }
    public int PeakInstanceCount { get; set; }
    public bool HasPeakTrafficVariance { get; set; }
}

public record CostOptimizationReport
{
    public string ServiceName { get; set; }
    public DateTime AnalysisDate { get; set; }
    public string RequiredSla { get; set; }
    public List<string> OptimizationOpportunities { get; set; } = new();
    public double EstimatedAnnualCostUsd { get; set; }
    public double PotentialAnnualSavingsUsd { get; set; }
    public double SavingsPercentage { get; set; }
}

public class Order
{
    public string OrderId { get; set; }
    public string CustomerId { get; set; }
    public decimal TotalAmount { get; set; }
    public string Status { get; set; }
    public DateTime CreatedAt { get; set; }
}

public interface IOrderRepository
{
    Task CreateAsync(Order order);
}
```

### Follow-Up Questions:

**Q1: How do you determine the right SLA for each service—how does it affect boundary decisions?**
* **Answer:** SLA should match business criticality and customer tolerance for downtime. **Critical path services** (authentication, payment, order placement) need high SLA (99.9%+); **supporting services** (recommendations, analytics) can have lower SLA (99%). SLA affects boundary decisions: if combining two services, they must meet the higher SLA of either (expensive if one needs 99.9% and other 99%). Better to keep them separate, each with appropriate SLA. Track SLA violations in Application Insights; if service consistently violates SLA, either upgrade infrastructure (expensive) or relax SLA (business decision). Document SLA clearly in service contract; if you promise 99.9%, failing to deliver 99.9% is a SLA breach (legal/financial consequences).

**Q2: What's the cost difference between 99%, 99.9%, and 99.99% SLA—how does it factor into boundary design?**
* **Answer:** Roughly exponential: 99% SLA might cost $X; 99.9% costs 3-5X (multi-instance, zone redundancy); 99.99% costs 10-20X (multi-region, active-active, extensive testing). Example: $5K/year for 99%, $20K/year for 99.9%, $50K+/year for 99.99%. For service boundaries, this means: consolidate high-SLA services to amortize cost, separate low-SLA services to avoid overpaying. Don't force all services to same SLA; tailor per service. For most SaaS, 99.9% is sweet spot—customers accept occasional downtime, cost is manageable. Reserve 99.99% for payment processing, critical user data (where downtime has severe consequences). Calculate SLA ROI: if 99.99% SLA costs 5X more but prevents customer churn on 0.09% of outages, is it worth it? Model financially before deciding.

**Q3: How do you optimize costs when you have unpredictable traffic spikes without violating SLA?**
* **Answer:** Use **spot instances or burstable SKUs** for non-critical components; they're 50-70% cheaper but can be preempted (acceptable for non-critical workloads). For critical components, use **scheduled scaling**: if spikes are predictable (end-of-month, holiday sales), scale up proactively before spike using scheduled rules. Use **reserved capacity** for baseline (always-on minimum), spot instances for burst. In Azure, combine **Reserved Instances** (for baseline) + **Spot VMs** (for burst) + **Auto Scaling** (responsive scaling). Implement **load shedding**: if traffic exceeds capacity, serve degraded experience (fewer results, delayed processing) rather than error, avoiding SLA violation. Use feature flags to disable non-critical features during high load (recommendations, personalization), freeing capacity for critical features (ordering, checkout).

**Q4: How do you balance consolidation for cost savings with maintaining service independence and deployment agility?**
* **Answer:** Use **cost-independence matrix**: map services by coupling (how much they communicate) and deployment frequency (how often they change). Services with high coupling and low deployment frequency can consolidate (low benefit to independence). Services with low coupling or high deployment frequency should stay separate. Example: Order and Inventory (independent deployment, different scaling needs, occasional communication) should stay separate. Order and OrderHistory (always deployed together, same scaling needs, tightly coupled data) should consolidate. For cost, calculate: consolidation savings vs. independence benefit (time to market, team autonomy, debugging ease). If independence benefit > consolidation savings, keep separate. Use **cost as a tiebreaker**: when in doubt, don't consolidate—independence has long-term value, cost optimization is short-term.

**Q5: How do you implement cost showback or chargeback in multi-team environments with shared infrastructure?**
* **Answer:** Implement **usage-based pricing model** within organization: track cost per service per team using tags in Azure (tag resources by service, team, cost center). Use **Azure Cost Management + Billing** to allocate costs; generate reports showing each team's spend. Implement **chargeback** (team is billed for usage) or **showback** (team is informed of cost but not billed). For chargeback, use internal billing system fed by Azure Cost Management data. Encourage cost awareness by making costs visible in dashboards; teams see their AWS bill and optimize naturally. For shared services (infrastructure, platform), allocate cost proportionally or per-request (track request metrics in Application Insights, multiply by per-request cost). Be transparent: show cost breakdown (compute, storage, data transfer, licenses); help teams understand what drives cost and where to optimize.

---

# Comprehensive Service Boundaries Definition Framework
## Decision Trees, Architectural Recommendations, and Implementation Guide

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Service Boundary Definition Decision Tree](#service-boundary-definition-decision-tree)
3. [Architecture Pattern Selection Guide](#architecture-pattern-selection-guide)
4. [Multi-Dimensional Decision Matrix](#multi-dimensional-decision-matrix)
5. [Implementation Roadmap](#implementation-roadmap)
6. [Common Anti-Patterns and Corrections](#common-anti-patterns-and-corrections)
7. [Azure Service Selection Guide](#azure-service-selection-guide)
8. [Operational Excellence Checklist](#operational-excellence-checklist)
9. [Cost Optimization Strategies](#cost-optimization-strategies)
10. [Interview Question Quick Reference](#interview-question-quick-reference)

---

## Executive Summary

Service boundary definition is the **foundational architectural decision** in microservices systems. Poor boundaries create technical debt (tight coupling, complex deployments), operational challenges (debugging cascading failures), and business impact (slow feature delivery, high infrastructure costs). This framework provides a systematic approach to defining, evolving, and optimizing service boundaries in Azure-based microservices architectures.

### Key Principles:

1. **Boundaries follow business capabilities** (not technical layers)
2. **Optimize for team autonomy** (Conway's Law)
3. **Balance independence vs. efficiency** (granularity sweet spot)
4. **Design for observable failure** (distributed systems perspective)
5. **Iterate based on metrics** (continuous refinement)

---

## Service Boundary Definition Decision Tree

```
┌─────────────────────────────────────────────────────────────────────┐
│ START: Defining Service Boundaries                                  │
└──────────────────────────┬──────────────────────────────────────────┘
                           │
                ┌──────────▼──────────┐
                │ What is the scope?  │
                └──────────┬──────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼
   GREENFIELD        EXISTING SYSTEM    HYBRID/EVOLUTION
   (New build)       (Monolith)         (Mixed state)
        │                  │                  │
        │         ┌─────────▼────────┐      │
        │         │ Use Strangler    │      │
        │         │ Pattern:         │      │
        │         │ - Extract one    │      │
        │         │   service        │      │
        │         │ - Route traffic  │      │
        │         │ - Monitor        │      │
        │         │ - Iterate        │      │
        │         └──────────────────┘      │
        │                  │                │
        └──────────────────┼────────────────┘
                           │
              ┌────────────▼────────────┐
              │ Identify Bounded        │
              │ Contexts (DDD)          │
              │ Ask: Where does         │
              │ language change?        │
              └────────────┬────────────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
    BUSINESS            TEAM          DEPLOYMENT
    CAPABILITY          STRUCTURE      FREQUENCY
    (Primary)           (Secondary)    (Tertiary)
          │                │                │
          └────────────────┼────────────────┘
                           │
              ┌────────────▼────────────┐
              │ Define Initial          │
              │ Boundaries              │
              │ (3-7 services ideal)    │
              └────────────┬────────────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
    TOO COARSE        APPROPRIATE      TOO FINE-GRAINED
    (<3 services)    (3-7 services)   (>15 services)
    │                │                │
    │ Re-split by    │ ✓ PROCEED      │ Reconsider:
    │ business       │ to contract    │ - Consolidate
    │ capability     │ design         │ - Check coupling
    │                │                │
    └────────┬───────┴────────┬───────┴─────────┘
             │                │
             └────────┬───────┘
                      │
         ┌────────────▼────────────┐
         │ Define Service          │
         │ Contracts               │
         │ (API, messaging,        │
         │ versioning)             │
         └────────────┬────────────┘
                      │
        ┌─────────────┴──────────────┐
        │                            │
        ▼                            ▼
   SYNCHRONOUS            ASYNCHRONOUS
   (REST/gRPC)            (Events/Queues)
        │                            │
        │ ┌──────────────────────────┤
        │ │ Versioning strategy:     │
        │ │ - URL path (/v1, /v2)    │
        │ │ - Header-based           │
        │ │ - Media type             │
        │ │ - Deprecation timeline   │
        │ │   (12-24 months)         │
        │ │                          │
        │ └──────────────────────────┤
        │                            │
        └────────────┬───────────────┘
                     │
        ┌────────────▼────────────┐
        │ Design Data Isolation   │
        │ Strategy                │
        └────────────┬────────────┘
                     │
    ┌────────────────┼────────────────┐
    │                │                │
    ▼                ▼                ▼
DATABASE PER    SHARED DB +     HYBRID
SERVICE         ROW-LEVEL       (per-context)
                SECURITY
    │                │                │
    │ Highest       │ Balanced       │ Flexible
    │ isolation,    │ cost/          │
    │ highest cost  │ isolation      │
    │                │                │
    └────────────────┼────────────────┘
                     │
        ┌────────────▼────────────┐
        │ Design Consistency      │
        │ Model                   │
        └────────────┬────────────┘
                     │
    ┌────────────────┼────────────────┐
    │                │                │
    ▼                ▼                ▼
STRONG          EVENTUAL           CAUSAL
CONSISTENCY     CONSISTENCY       CONSISTENCY
(Synchronous    (Event-driven     (Causal
 coordination)   Saga pattern)      ordering)
    │                │                │
    │ Limited to    │ Scalable,       │ Compromises
    │ single        │ resilient,      │
    │ service       │ complex         │
    │                │                │
    └────────────────┼────────────────┘
                     │
        ┌────────────▼────────────┐
        │ Design Cross-Cutting    │
        │ Concerns                │
        │ (Logging, auth,         │
        │  tracing, security)     │
        └────────────┬────────────┘
                     │
    ┌────────────────┴────────────────┐
    │                                 │
    ▼                                 ▼
CENTRALIZED                    DECENTRALIZED
(API Gateway,                 (Service mesh,
 Infrastructure)              Sidecars)
    │                                 │
    │ ┌──────────────────────────────┤
    │ │ Select Azure services:       │
    │ │ - API Management (APIM)      │
    │ │ - Application Insights       │
    │ │ - Azure AD (identity)        │
    │ │ - Azure Service Bus (async)  │
    │ │                              │
    │ └──────────────────────────────┤
    │                                 │
    └────────────┬────────────────────┘
                 │
      ┌──────────▼──────────┐
      │ Establish Service   │
      │ Discovery Strategy  │
      └────────────┬────────┘
                   │
    ┌──────────────┼──────────────┐
    │              │              │
    ▼              ▼              ▼
AZURE DNS    API MANAGEMENT   KUBERNETES DNS
(DNS-based)  (APIM gateway)   (AKS clusters)
    │              │              │
    │ Simplest,    │ Full-featured,│ Cloud-native,
    │ managed      │ comprehensive │ auto-scaling
    │              │              │
    └──────────────┼──────────────┘
                   │
      ┌────────────▼────────────┐
      │ Design for Failure &    │
      │ Graceful Degradation    │
      └────────────┬────────────┘
                   │
    ┌──────────────┼──────────────┐
    │              │              │
    ▼              ▼              ▼
CIRCUIT      RETRY +        FALLBACK
BREAKER      EXPONENTIAL    STRATEGY
             BACKOFF
    │              │              │
    │ Stop calling │ Transient    │ Degraded
    │ failing      │ error        │ response vs.
    │ service      │ recovery     │ error
    │              │              │
    └──────────────┼──────────────┘
                   │
      ┌────────────▼────────────┐
      │ Plan Cost Optimization  │
      │ & SLA Requirements      │
      └────────────┬────────────┘
                   │
    ┌──────────────┼──────────────┐
    │              │              │
    ▼              ▼              ▼
SLA BY       SCALING         COST
SERVICE      STRATEGY        ALLOCATION
    │              │              │
    │ Different    │ Independent  │ Reserved +
    │ SLAs per     │ per service  │ spot
    │ criticality  │              │ instances
    │              │              │
    └──────────────┼──────────────┘
                   │
      ┌────────────▼────────────┐
      │ ✓ IMPLEMENTATION READY   │
      │                          │
      │ Next: Create contracts,  │
      │ deploy infrastructure,   │
      │ implement services,      │
      │ establish monitoring     │
      └──────────────────────────┘
```

---

## Architecture Pattern Selection Guide

### Pattern 1: Business Capability-Based Boundaries (Recommended)

**When to use:** Always start here for greenfield systems

**Decision criteria:**
- Clear business domain model (DDD)
- Cross-functional teams available
- Independent deployment needed
- Feature velocity important

**Benefits:**
- Aligns with organizational structure
- Enables team autonomy
- Supports rapid feature delivery
- Clear ownership

**Trade-offs:**
- Requires clear business understanding upfront
- May not align with technical constraints initially

**Azure Services:**
- Azure DevOps (team organization)
- Azure Service Bus (async communication)
- Event Grid (event publishing)
- API Management (contract versioning)

**Implementation Example:**

```csharp
// Business capability-based services
public enum BusinessCapability
{
    OrderManagement,      // Handles order creation, updates, cancellation
    InventoryManagement,  // Manages stock levels
    CustomerManagement,   // Customer data, preferences
    BillingAndPayments,   // Payment processing, invoicing
    Fulfillment,          // Shipping, tracking
    Analytics             // Reporting, insights
}

// Teams align to capabilities
public class OrganizationStructure
{
    public BusinessCapability Capability { get; set; }
    public List<string> TeamMembers { get; set; }
    public string ServiceName { get; set; } // "order-service", etc.
    public List<string> DeploymentRegions { get; set; }
}
```

---

### Pattern 2: Strangler Fig Pattern (For Monolith Migration)

**When to use:** Migrating from monolithic system

**Implementation phases:**

```
Phase 1: Deploy new service alongside monolith
├─ Create new service (Order microservice)
├─ Deploy in parallel
└─ New service fully tested

Phase 2: Gradual traffic routing
├─ Route 5% of traffic to new service (canary)
├─ Monitor errors/latency in Application Insights
├─ Gradually increase: 5% → 25% → 50% → 100%
└─ Maintain old monolith during transition

Phase 3: Data migration
├─ Replicate historical data to new service database
├─ Validate consistency
├─ Maintain dual-write (monolith + new service)
└─ Switch reads to new service once validated

Phase 4: Retire legacy component
├─ Confirm 100% traffic on new service
├─ Monitor for issues over 2-4 weeks
├─ Archive monolith component
└─ Document lessons learned
```

**Azure Services:**
- Azure API Management (routing)
- Azure App Gateway (traffic splitting)
- Azure Data Factory (data migration)
- Azure DevOps (coordinated deployments)

---

### Pattern 3: Saga Pattern (For Distributed Transactions)

**When to use:** Coordinating multi-service transactions

**Implementation approaches:**

```
Option A: Orchestration (Centralized)
├─ Order Service → [Orchestrator] ← Inventory Service
│                 ← Payment Service
│
├─ Orchestrator controls flow
├─ Orchestrator handles failures
├─ Easier to understand
├─ Orchestrator can become bottleneck
└─ Use: Azure Durable Functions

Option B: Choreography (Event-driven)
├─ Order Service creates order → [Event: OrderCreated]
├─ Inventory Service listens → reserves inventory → [Event: InventoryReserved]
├─ Payment Service listens → processes payment → [Event: PaymentProcessed]
├─ Fully distributed, no bottleneck
├─ Harder to debug flow
└─ Use: Azure Service Bus, Event Grid

Compensating transactions for failures:
├─ Payment fails → emit PaymentFailed
├─ Inventory service listens → releases reservation
├─ Order service listens → cancels order
```

**Azure Services:**
- Azure Durable Functions (orchestration)
- Azure Service Bus (choreography)
- Event Grid (event publishing)

---

### Pattern 4: Multi-Tenant SaaS Boundaries

**When to use:** Building SaaS applications

**Data isolation strategies:**

```
Strategy A: Database per Tenant (Highest isolation)
├─ Tenant A → Database A
├─ Tenant B → Database B
├─ Strongest isolation
├─ Highest operational overhead (backup/restore/migration per tenant)
└─ Cost: O(n) where n = number of tenants

Strategy B: Shared Database + Row-Level Security (RLS)
├─ All tenants → Single database
├─ RLS policies automatically filter rows by tenant
├─ Balanced cost/isolation
├─ Schema updates affect all tenants simultaneously (risk)
└─ Cost: Minimal overhead

Strategy C: Hybrid (Recommended for most SaaS)
├─ Shared core/platform services (auth, billing)
├─ Database per tenant for high-value/compliance-sensitive tenants
├─ Shared database + RLS for standard tenants
└─ Flexible per customer needs
```

**Implementation:**

```csharp
// Tenant context propagation
public interface ITenantContext
{
    string TenantId { get; }
    bool IsAuthorized(string requestedTenantId);
}

// Every query automatically scoped to tenant
public class MultiTenantRepository<T> where T : IMultiTenantEntity
{
    public async Task<T> GetAsync(string tenantId, string id)
    {
        ValidateTenantAccess(tenantId);
        return await _dbContext.Set<T>()
            .Where(x => x.TenantId == tenantId && x.Id == id)
            .FirstOrDefaultAsync();
    }
}
```

---

### Pattern 5: Anti-Corruption Layer (For Boundary Protection)

**When to use:** Integrating with external systems or legacy components

**Structure:**

```
External System         Your Services
       │                    │
       └──────┬──────────────┘
              │
    ┌─────────▼─────────┐
    │ Anti-Corruption   │
    │ Layer             │
    │ ─────────────────│
    │ - Translate       │
    │ - Validate        │
    │ - Transform       │
    │ - Adapt contracts │
    └─────────┬─────────┘
              │
    ┌─────────▼──────────────┐
    │ Your Domain Model      │
    │ (Decoupled from        │
    │  external representation)
    └───────────────────────┘
```

**Example:**

```csharp
// External system (legacy)
public class LegacyOrderResponse
{
    public int ORDER_ID { get; set; }  // Legacy naming
    public string CUST_ID { get; set; }
    public decimal AMT { get; set; }
}

// Anti-corruption layer
public class LegacyOrderAdapter
{
    public Order ConvertToOurModel(LegacyOrderResponse legacy)
    {
        return new Order
        {
            OrderId = legacy.ORDER_ID.ToString(),
            CustomerId = legacy.CUST_ID,
            TotalAmount = legacy.AMT,
            // Add any validation/transformation here
        };
    }
}

// Your service uses clean model
public class OrderService
{
    private readonly LegacyOrderAdapter _adapter;
    
    public async Task<Order> GetOrderAsync(int legacyOrderId)
    {
        var legacyOrder = await _legacySystem.GetOrder(legacyOrderId);
        return _adapter.ConvertToOurModel(legacyOrder);
    }
}
```

---

## Multi-Dimensional Decision Matrix

### Service Granularity Decision Matrix

```
┌─────────────────────┬───────────┬────────────┬──────────────────┐
│ Characteristic      │ Coarse    │ Balanced   │ Fine-grained     │
│                     │ (1-2)     │ (3-7)      │ (10+)            │
├─────────────────────┼───────────┼────────────┼──────────────────┤
│ Number of services  │ 1-2       │ 3-7        │ 10+              │
│                     │           │            │                  │
│ Deployment          │ Single    │ Independent│ Highly           │
│ independence        │ deploy    │ per        │ independent      │
│                     │ for all   │ service    │ deploys per      │
│                     │           │            │ service          │
│                     │           │            │                  │
│ Scaling             │ Monolithic│ Selective  │ Fine-grained     │
│                     │ (all or   │ scaling    │ scaling per      │
│                     │ nothing)  │ by need    │ endpoint         │
│                     │           │            │                  │
│ Data isolation      │ Shared    │ Mixed      │ Per-service      │
│                     │ database  │ approach   │ databases        │
│                     │           │            │                  │
│ Operational         │ Simple    │ Moderate   │ Complex          │
│ complexity          │ (few      │ (some      │ (many            │
│                     │ deployments│ coordination│deployments,     │
│                     │)          │)           │coordination)     │
│                     │           │            │                  │
│ Cross-service       │ Local     │ Async +    │ Mostly async,    │
│ communication       │ calls     │ occasional │ complex routing  │
│                     │           │ sync       │                  │
│                     │           │            │                  │
│ Team organization   │ Single    │ 3-7 teams  │ Many small teams │
│                     │ large team│ or mixed   │                  │
│                     │           │            │                  │
│ Feature velocity    │ Slower    │ Optimal    │ Can be slower    │
│                     │ (coord    │ (independent│ (debugging,      │
│                     │ required) │ teams)     │ complexity)      │
│                     │           │            │                  │
│ Infrastructure cost │ Low       │ Moderate   │ High (per-service│
│                     │           │            │ overhead)        │
│                     │           │            │                  │
│ Recommended for     │ Simple    │ Most SaaS  │ Large teams,     │
│                     │ internal  │ systems,   │ extensive        │
│                     │ apps      │ startups   │ domain           │
│                     │           │            │ complexity       │
└─────────────────────┴───────────┴────────────┴──────────────────┘
```

### Data Consistency vs. Performance Trade-off

```
                      CONSISTENCY
                           ▲
                           │
                    Strong consistency
                           │  (ACID)
                           │  - Single service
                           │  - Traditional DB
                           │  ╱
                          ╱│
                         ╱ │
                        ╱  │
    Causal consistency ╱   │
                      ╱    │  (Hybrid)
                     ╱     │  - Saga with retries
                    ╱      │  - Event sourcing
                   ╱       │
                  ╱        │
                 ╱         │
        Eventual consistency (Async)
       │         │      ╱  Event-driven
       │         │     ╱   - Services independent
       │         │    ╱    - No coordination
       │         │   ╱
       └─────────┼──▶ PERFORMANCE
                 │
            Distributed systems reality:
            Choose your trade-off point
            based on business requirements
```

### Deployment Topology Decision Matrix

```
┌──────────────────────┬──────────┬─────────────┬──────────────────┐
│ Deployment Target    │ App      │ Containers  │ Kubernetes       │
│                      │ Service  │ (ACI, ACS)  │ (AKS)            │
├──────────────────────┼──────────┼─────────────┼──────────────────┤
│ Complexity           │ Low      │ Moderate    │ High             │
│ Learning curve       │ Gentle   │ Medium      │ Steep            │
│ Auto-scaling         │ Good     │ Good        │ Excellent        │
│ Cost per service     │ Moderate │ Low         │ Very low (scaled)│
│ Service discovery    │ DNS      │ Manual/DNS  │ Native (DNS SVC) │
│ Network policies     │ NSGs     │ Container   │ Network policies │
│ Recommended for      │ Small    │ Containerized│ Large clusters, │
│                      │ teams,   │ workloads,  │ multi-region,    │
│                      │ <10      │ on-premises │ complex routing  │
│                      │ services │             │                  │
│ Azure tool           │ App      │ Container   │ AKS              │
│                      │ Service  │ Instances   │                  │
└──────────────────────┴──────────┴─────────────┴──────────────────┘
```

---

## Implementation Roadmap

### Phase 1: Assessment (Weeks 1-2)

**Objectives:**
- Understand current system architecture
- Identify stakeholders and teams
- Define business capabilities
- Assess operational maturity

**Activities:**

```csharp
// Document current state
public class CurrentStateAssessment
{
    public string SystemName { get; set; }
    public ArchitectureType CurrentArchitecture { get; set; } // Monolith, SOA, Microservices
    public List<string> Teams { get; set; }
    public List<DeploymentPipeline> CurrentDeployments { get; set; }
    public List<Database> DatabasesAndDataStores { get; set; }
    public List<ThirdPartyIntegration> ExternalIntegrations { get; set; }
    
    // Pain points
    public List<string> CurrentChallenges { get; set; }
    public List<string> DeploymentFrequencies { get; set; }
    public List<string> KnownBottlenecks { get; set; }
}

// Identify business capabilities through event storming
public class EventStormingSession
{
    public List<DomainEvent> Events { get; set; }      // Sticky notes: orange
    public List<Command> Commands { get; set; }        // Sticky notes: blue
    public List<Aggregate> Aggregates { get; set; }    // Sticky notes: yellow
    public List<BoundedContext> BoundedContexts { get; set; } // Identified contexts
    public List<string> ActorsAndSystems { get; set; } // Sticky notes: green/pink
}
```

**Deliverables:**
- Current state architecture diagram
- List of 3-7 proposed bounded contexts
- Team organization structure
- Initial pain points documentation

---

### Phase 2: Design (Weeks 3-4)

**Objectives:**
- Define service boundaries
- Design contracts
- Plan data strategy
- Design for failure

**Activities:**

```csharp
// Define service specifications
public class ServiceSpecification
{
    public string ServiceName { get; set; }
    public string BusinessCapability { get; set; }
    public List<TeamMember> OwningTeam { get; set; }
    
    // API Contract
    public List<RestEndpoint> RestApi { get; set; }
    public List<AsyncEvent> PublishedEvents { get; set; }
    public List<AsyncEvent> SubscribedEvents { get; set; }
    
    // Data Strategy
    public DataIsolationStrategy DataIsolation { get; set; }
    public string DatabaseTechnology { get; set; } // SQL, CosmosDB, Redis
    public ConsistencyModel ConsistencyModel { get; set; }
    
    // Deployment
    public DeploymentStrategy Deployment { get; set; }
    public int MinInstanceCount { get; set; }
    public int MaxInstanceCount { get; set; }
    public ServiceSla RequiredSla { get; set; }
    
    // Failure Handling
    public List<ResiliencePolicy> ResiliencePolicies { get; set; }
    public FallbackStrategy FallbackStrategy { get; set; }
}

// Document contracts
public class ServiceContract
{
    // REST API specification (OpenAPI/Swagger)
    public OpenApiSpec RestApiSpec { get; set; }
    
    // Versioning strategy
    public VersioningStrategy VersioningStrategy { get; set; } // URL, header, media-type
    public int CurrentMajorVersion { get; set; }
    public string DeprecationTimeline { get; set; }
    
    // Async messaging specification
    public List<MessageSchema> MessageSchemas { get; set; }
    public List<TopicOrQueueDefinition> MessagingEndpoints { get; set; }
    
    // SLA
    public ServiceSla Sla { get; set; }
    public Dictionary<string, long> LatencySlas { get; set; } // p95, p99
}
```

**Deliverables:**
- Service boundary definitions (3-7 services)
- Service specifications for each
- API contract definitions (OpenAPI/Swagger)
- Data isolation strategy document
- Resilience patterns design

---

### Phase 3: Build Infrastructure (Weeks 5-8)

**Objectives:**
- Set up Azure resources
- Configure CI/CD pipelines
- Implement service discovery
- Deploy monitoring

**Activities:**

```csharp
// Infrastructure as Code (IaC) template
public class AzureInfrastructureDefinition
{
    // API Management
    public ApiManagementService ApimInstance { get; set; }
    
    // Service hosting
    public List<AzureAppService> ServiceInstances { get; set; }
    public AzureKubernetesService AksCluster { get; set; } // Optional
    
    // Data
    public List<AzureSqlDatabase> SqlDatabases { get; set; }
    public List<CosmosDbAccount> CosmosDbAccounts { get; set; }
    
    // Messaging
    public ServiceBusNamespace ServiceBusNamespace { get; set; }
    public EventGridTopic EventGridTopic { get; set; }
    
    // Monitoring
    public ApplicationInsightsInstance ApplicationInsights { get; set; }
    public LogAnalyticsWorkspace LogAnalyticsWorkspace { get; set; }
    
    // Security
    public KeyVaultInstance KeyVault { get; set; }
    public MicrosoftEntraId EntraIdConfig { get; set; }
    
    // Networking
    public VirtualNetwork Vnet { get; set; }
    public List<NetworkSecurityGroup> NetworkSecurityGroups { get; set; }
}

// Service discovery configuration
public class ServiceDiscoveryConfiguration
{
    public DiscoveryMode Mode { get; set; } // AzureDns, ApiManagement, Kubernetes
    
    // Health checks
    public HealthCheckConfig LivenessProbe { get; set; }
    public HealthCheckConfig ReadinessProbe { get; set; }
    
    // Service registration
    public List<RegisteredService> RegisteredServices { get; set; }
}

// CI/CD pipeline configuration
public class CiCdConfiguration
{
    public string RepositoryUrl { get; set; }
    public List<PipelineStage> BuildPipeline { get; set; }
    public List<PipelineStage> DeploymentPipeline { get; set; }
    public AutomatedTestStrategy TestStrategy { get; set; }
}
```

**Deliverables:**
- ARM templates or Terraform scripts for all resources
- CI/CD pipelines (Azure DevOps/GitHub Actions)
- Health check endpoints
- Monitoring dashboards (Application Insights)

---

### Phase 4: Implement Services (Weeks 9-16)

**Objectives:**
- Develop individual microservices
- Implement contracts
- Implement resilience patterns
- Comprehensive testing

**Activities:**

```csharp
// Service template with all best practices
public class MicroserviceTemplate
{
    // Startup and configuration
    public class ServiceStartup
    {
        public void ConfigureServices(IServiceCollection services, IConfiguration config)
        {
            // Dependency injection
            services.AddScoped<ITenantContext, HttpContextTenantProvider>();
            services.AddScoped<OrderService>();
            
            // HTTP clients with resilience
            services.AddHttpClient<ResilientInventoryClient>()
                .AddPolicyHandler(GetRetryPolicy())
                .AddPolicyHandler(GetCircuitBreakerPolicy());
            
            // Service discovery
            services.AddServiceDiscovery(config);
            
            // API versioning
            services.AddApiVersioning();
            
            // Authorization
            services.AddCentralizedAuthentication(config);
            
            // Messaging
            services.AddServiceBusMessaging(config);
            
            // Monitoring
            services.AddApplicationInsightsTelemetry();
            services.AddOpenTelemetry();
        }
        
        public void Configure(IApplicationBuilder app)
        {
            // Middleware
            app.UseCorrelationIdMiddleware();
            app.UseMultiTenancy();
            app.UseAuthentication();
            app.UseAuthorization();
            
            // Endpoints
            app.UseEndpoints(endpoints => endpoints.MapControllers());
            
            // Health checks
            app.UseHealthChecks("/health/live");
            app.UseHealthChecks("/health/ready", new HealthCheckOptions { ... });
        }
    }
    
    // Contracts
    [ApiController]
    [Route("api/v{version:apiVersion}/[controller]")]
    public class ResourceController : ControllerBase
    {
        [HttpGet("{id}")]
        [ProducesResponseType(typeof(ResourceResponse), 200)]
        [ProducesResponseType(404)]
        public async Task<ActionResult<ResourceResponse>> GetAsync(string id) { }
        
        [HttpPost]
        [Authorize(Policy = "WriterRole")]
        [ProducesResponseType(typeof(ResourceResponse), 201)]
        public async Task<ActionResult<ResourceResponse>> CreateAsync(
            [FromBody] CreateResourceRequest request) { }
    }
    
    // Domain logic
    public class DomainService
    {
        private readonly IRepository<Entity> _repository;
        private readonly ILogger<DomainService> _logger;
        private readonly ResilientDownstreamClient _downstream;
        
        public async Task<Entity> DoSomethingAsync(CreateRequest request)
        {
            // Business logic with resilience
            var downstreamData = await _downstream.GetDataWithFallbackAsync(request.Id);
            var entity = new Entity { /* ... */ };
            await _repository.CreateAsync(entity);
            return entity;
        }
    }
    
    // Resilience patterns
    private static IAsyncPolicy<HttpResponseMessage> GetRetryPolicy()
    {
        return HttpPolicyExtensions
            .HandleTransientHttpError()
            .WaitAndRetryAsync(
                retryCount: 3,
                sleepDurationProvider: attempt => TimeSpan.FromSeconds(Math.Pow(2, attempt)));
    }
    
    private static IAsyncPolicy<HttpResponseMessage> GetCircuitBreakerPolicy()
    {
        return HttpPolicyExtensions
            .HandleTransientHttpError()
            .CircuitBreakerAsync(
                handledEventsAllowedBeforeBreaking: 5,
                durationOfBreak: TimeSpan.FromSeconds(30));
    }
}

// Testing
public class ServiceTests
{
    [TestClass]
    public class ContractTests
    {
        [TestMethod]
        public async Task Get_ReturnsV1Contract_WhenCalledWithV1()
        {
            // Test backward compatibility
        }
        
        [TestMethod]
        public async Task Get_ReturnsV2Contract_WhenCalledWithV2()
        {
            // Test new version features
        }
    }
    
    [TestClass]
    public class ResilienceTests
    {
        [TestMethod]
        public async Task DownstreamServiceDown_DegradeGracefully()
        {
            // Test circuit breaker, fallback
        }
        
        [TestMethod]
        public async Task TransientFailure_RetrySucceeds()
        {
            // Test retry logic
        }
    }
    
    [TestClass]
    public class MultiTenancyTests
    {
        [TestMethod]
        public async Task TenantAData_NotAccessibleByTenantB()
        {
            // Test isolation
        }
    }
}
```

**Deliverables:**
- Implemented microservices
- Comprehensive test suites
- Deployed to staging environments
- Documentation and runbooks

---

### Phase 5: Deploy & Monitor (Weeks 17-20)

**Objectives:**
- Production deployment
- Establish monitoring
- Incident response procedures
- Continuous optimization

**Activities:**

```csharp
// Deployment strategy
public class ProductionDeploymentStrategy
{
    // Canary deployment
    public class CanaryDeployment
    {
        public decimal InitialTrafficPercentage { get; set; } // 5%
        public decimal IncrementPercentage { get; set; }      // 10%
        public TimeSpan IncrementInterval { get; set; }       // 1 hour
        public MetricThreshold ErrorRateThreshold { get; set; }
        public MetricThreshold LatencyThreshold { get; set; }
        
        // Auto-rollback if thresholds exceeded
        public bool AutoRollbackOnThresholdExceeded { get; set; } = true;
    }
    
    // Blue-green deployment
    public class BlueGreenDeployment
    {
        public Environment BlueEnvironment { get; set; }  // Current
        public Environment GreenEnvironment { get; set; } // New
        
        public async Task ExecuteAsync()
        {
            // 1. Deploy to Green
            // 2. Run tests against Green
            // 3. Switch traffic Blue -> Green
            // 4. Monitor for errors
            // 5. Keep Blue as rollback
        }
    }
}

// Monitoring and alerting
public class MonitoringStrategy
{
    public class DashboardDefinition
    {
        public List<Metric> ServiceHealthMetrics { get; set; }
        public List<Metric> BusinessMetrics { get; set; }
        public List<Metric> InfrastructureMetrics { get; set; }
    }
    
    public class AlertingRules
    {
        public List<AlertRule> CriticalAlerts { get; set; } // Page on-call
        public List<AlertRule> WarningAlerts { get; set; }  // Email
        public List<AlertRule> InfoAlerts { get; set; }     // Log
    }
}

// Incident response
public class IncidentResponsePlan
{
    public List<RunbookDefinition> Runbooks { get; set; }
    public EscalationMatrix EscalationMatrix { get; set; }
    public PostMortemProcess PostMortemProcess { get; set; }
}

// Continuous optimization
public class ContinuousOptimization
{
    // Weekly reviews
    public class WeeklyReview
    {
        public List<Metric> MetricsTrend { get; set; }
        public List<string> IssuesIdentified { get; set; }
        public List<string> OptimizationActions { get; set; }
    }
    
    // Monthly architecture review
    public class MonthlyArchitectureReview
    {
        public ServiceBoundaryHealthReport BoundaryHealth { get; set; }
        public CostOptimizationReport CostAnalysis { get; set; }
        public List<string> ProposedChanges { get; set; }
    }
    
    // Quarterly refinement
    public class QuarterlyRefinement
    {
        public List<BoundaryRefactoringProposal> RefactoringProposals { get; set; }
        public List<ArchitectureDecisionRecord> DecisionRecords { get; set; }
    }
}
```

---

## Common Anti-Patterns and Corrections

### Anti-Pattern 1: Chatty Microservices (Too Many Inter-Service Calls)

**Symptoms:**
- Single user request triggers 50+ service calls
- Average latency >2 seconds
- Network bandwidth between services exceeds intra-service bandwidth
- High Application Insights dependency call counts

**Root Causes:**
- Services over-fragmented (each CRUD operation is separate service)
- Lack of caching/denormalization
- Poor API design (requires multiple calls to accomplish one task)

**Corrections:**

```csharp
// ❌ WRONG: Chatty design
public class OrderCheckoutService
{
    public async Task CheckoutAsync(string customerId, string[] productIds)
    {
        // Call 1: Get customer
        var customer = await _customerService.GetCustomer(customerId);
        
        // Calls 2-N: Get each product
        var products = new List<Product>();
        foreach (var productId in productIds)
        {
            products.Add(await _productService.GetProduct(productId));
        }
        
        // Call N+1: Get inventory for each product
        foreach (var product in products)
        {
            var inventory = await _inventoryService.GetInventory(product.Sku);
        }
        
        // ... etc
        // Total: 50+ calls for checkout
    }
}

// ✓ CORRECT: Batch and cache
public class OptimizedOrderCheckoutService
{
    public async Task CheckoutAsync(string customerId, string[] productIds)
    {
        // Call 1: Batch get customer + products + inventory
        var checkoutData = await _compositeCheckoutService.GetCheckoutDataAsync(
            new CheckoutRequest
            {
                CustomerId = customerId,
                ProductIds = productIds
            });
        
        // Use cached/denormalized data
        var total = checkoutData.Products.Sum(p => p.Price * p.RequestedQuantity);
        
        // Proceed with order
    }
}

// Composite service that aggregates data efficiently
public class CompositeCheckoutService
{
    public async Task<CheckoutData> GetCheckoutDataAsync(CheckoutRequest request)
    {
        // Parallel calls instead of sequential
        var customerTask = _customerService.GetCustomer(request.CustomerId);
        var productsTask = _productService.GetProductsBatch(request.ProductIds); // Batch call
        var inventoryTask = _inventoryService.CheckInventoryBatch(request.ProductIds); // Batch call
        
        await Task.WhenAll(customerTask, productsTask, inventoryTask);
        
        return new CheckoutData
        {
            Customer = await customerTask,
            Products = await productsTask,
            Inventory = await inventoryTask
        };
    }
}
```

**Prevention:**
- Monitor Application Insights for dependency call ratios (>3 calls per request)
- Design APIs for common use cases (batch operations)
- Use API composition/BFF patterns
- Denormalize data appropriately

---

### Anti-Pattern 2: Circular Dependencies

**Symptoms:**
- Service A calls Service B, Service B calls Service A
- Deployment order issues
- Deadlock potential
- Difficult to understand data flow

**Root Causes:**
- Boundaries defined by technical layers instead of business capability
- Database joins across service boundaries
- Unclear data ownership

**Corrections:**

```csharp
// ❌ WRONG: Circular dependency
public class OrderService
{
    // Order → Inventory → Order (cycle!)
    public async Task CreateOrderAsync(Order order)
    {
        // Check if items available
        var available = await _inventoryService.CheckAvailability(order.Items);
        
        if (available)
            await _repository.CreateAsync(order);
    }
}

public class InventoryService
{
    // Inventory → Order → Inventory (cycle!)
    public async Task ReserveItemsAsync(string[] items, string orderId)
    {
        // Update order status
        await _orderService.UpdateOrderStatus(orderId, "Reserved");
    }
}

// ✓ CORRECT: Introduce intermediary or use events
public class CheckoutOrchestrator // New intermediary service
{
    public async Task CheckoutAsync(CheckoutRequest request)
    {
        // Orchestrate without cycles
        var inventoryAvailable = await _inventoryService.CheckAvailability(request.Items);
        
        if (inventoryAvailable)
        {
            var order = await _orderService.CreateOrderAsync(request);
            await _inventoryService.ReserveItemsAsync(request.Items, order.Id);
            
            // Publish event; both services listen but don't call each other
            await _eventPublisher.PublishAsync(new OrderCreatedEvent { OrderId = order.Id });
        }
    }
}

// Or use event-driven (choreography)
public class InventoryServiceEventDriven
{
    [ServiceBusTrigger("order-events")]
    public async Task HandleOrderCreatedAsync(OrderCreatedEvent @event)
    {
        // Listen for event, don't call Order service
        await _repository.ReserveItems(@event.OrderId, @event.Items);
        
        // Publish own event
        await _eventPublisher.PublishAsync(
            new InventoryReservedEvent { OrderId = @event.OrderId });
    }
}
```

**Prevention:**
- Visualize service dependency graph (Application Insights dependency map)
- Run dependency cycle detection in CI/CD
- Review circular dependency as architecture review blocker

---

### Anti-Pattern 3: Shared Database Across Services

**Symptoms:**
- Multiple services directly query same database
- Hard to scale individual services
- Data model changes require coordination
- Tight coupling through database schema

**Root Causes:**
- Database-first design instead of service-first
- Fear of duplication (over-normalized)
- Legacy system inheritance

**Corrections:**

```csharp
// ❌ WRONG: Shared database
public class OrderService
{
    public async Task<Order> GetOrderAsync(string id)
    {
        // Direct query on shared database
        return await _sharedDbContext.Orders
            .Include(o => o.OrderItems)
                .ThenInclude(oi => oi.Product) // Couples to Product schema!
            .FirstOrDefaultAsync(o => o.Id == id);
    }
}

public class InventoryService
{
    public async Task<Inventory> GetInventoryAsync(string productId)
    {
        // Same shared database; both services coupled through schema
        return await _sharedDbContext.Inventory
            .FirstOrDefaultAsync(i => i.ProductId == productId);
    }
}

// ✓ CORRECT: Database per service
public class OrderService
{
    private readonly OrderDbContext _orderDb;
    
    public async Task<Order> GetOrderAsync(string id)
    {
        // Own database; isolated schema
        return await _orderDb.Orders
            .FirstOrDefaultAsync(o => o.Id == id);
    }
    
    public async Task EnrichOrderWithProductNamesAsync(Order order)
    {
        // Call Product service API to get names (not direct DB access)
        var productNames = await _productServiceClient.GetProductNamesAsync(
            order.Items.Select(i => i.ProductId).ToArray());
        
        // Denormalize into order response
        foreach (var item in order.Items)
        {
            item.ProductName = productNames[item.ProductId];
        }
    }
}

public class InventoryService
{
    private readonly InventoryDbContext _inventoryDb;
    
    public async Task<Inventory> GetInventoryAsync(string productId)
    {
        // Own database; can evolve schema independently
        return await _inventoryDb.Inventory
            .FirstOrDefaultAsync(i => i.ProductId == productId);
    }
}
```

**Prevention:**
- Enforce "database per service" as architecture principle
- Code review: flag direct cross-service DB access
- Use API calls for cross-service data access

---

### Anti-Pattern 4: "God Service" (Catch-All Service)

**Symptoms:**
- One service handles multiple unrelated capabilities
- Service has 50+ endpoints
- Different teams claim ownership
- High deployment frequency for unrelated changes

**Root Causes:**
- Unclear business domain (DDD not applied)
- Fear of splitting (over-consolidation)
- Gradual accumulation over time

**Corrections:**

```csharp
// ❌ WRONG: God service
[ApiController]
[Route("api/v1/[controller]")]
public class MasterService : ControllerBase
{
    // Order endpoints
    [HttpPost("orders")]
    public async Task<IActionResult> CreateOrder([FromBody] CreateOrderRequest req) { }
    
    [HttpGet("orders/{id}")]
    public async Task<IActionResult> GetOrder(string id) { }
    
    // Inventory endpoints
    [HttpPost("inventory/reserve")]
    public async Task<IActionResult> ReserveInventory([FromBody] ReserveRequest req) { }
    
    // Payment endpoints
    [HttpPost("payments")]
    public async Task<IActionResult> ProcessPayment([FromBody] PaymentRequest req) { }
    
    // Shipping endpoints
    [HttpPost("shipments")]
    public async Task<IActionResult> CreateShipment([FromBody] ShipmentRequest req) { }
    
    // Notifications endpoints
    [HttpPost("notifications/send")]
    public async Task<IActionResult> SendNotification([FromBody] NotificationRequest req) { }
    
    // ... 50+ endpoints
}

// ✓ CORRECT: Split into services
[ApiController]
[Route("api/v1/orders")]
public class OrderService : ControllerBase
{
    [HttpPost]
    public async Task<IActionResult> CreateOrder([FromBody] CreateOrderRequest req) { }
    
    [HttpGet("{id}")]
    public async Task<IActionResult> GetOrder(string id) { }
}

[ApiController]
[Route("api/v1/inventory")]
public class InventoryService : ControllerBase
{
    [HttpPost("reserve")]
    public async Task<IActionResult> ReserveInventory([FromBody] ReserveRequest req) { }
}

[ApiController]
[Route("api/v1/payments")]
public class PaymentService : ControllerBase
{
    [HttpPost]
    public async Task<IActionResult> ProcessPayment([FromBody] PaymentRequest req) { }
}

// ... etc
```

**Prevention:**
- Use Bounded Contexts to define service scope
- Limit service to <20 public endpoints (rule of thumb)
- Assign clear team ownership per service

---

## Azure Service Selection Guide

### Comprehensive Service Selection Decision Tree

```
┌──────────────────────────────────────────────────────────────┐
│ I need to build microservices. Which Azure services do I use?│
└────────────────────────┬─────────────────────────────────────┘
                         │
        ┌────────────────┴────────────────┐
        │                                 │
        ▼                                 ▼
   Where will services               How will they
   run?                              communicate?
        │                                 │
    ┌───┴─────────────┐            ┌──────┴──────────┐
    │                 │            │                 │
    ▼                 ▼            ▼                 ▼
  PaaS          Containers      Synchronous      Asynchronous
  (App           (Docker)        (REST/gRPC)      (Events/Queues)
  Service)       
    │                 │            │                 │
    │           ┌─────┴─────┐      │                 │
    │           │           │      │                 │
    ▼           ▼           ▼      ▼                 ▼
   App        Container  Kubernetes API          Service Bus
   Service    Instances  (AKS)    Management      │
                                                  ├─ Queues
                                                  ├─ Topics/Subs
                                                  └─ Sessions
                                    │
                              ┌─────┴─────────┐
                              │               │
                              ▼               ▼
                         Managed          Event Grid
                         (APIM)
                         Service
                         Facade


For Data Storage:
    ├─ Relational → Azure SQL Database or PostgreSQL
    ├─ Document → Azure Cosmos DB
    ├─ Cache/Session → Azure Cache for Redis
    ├─ Blob → Azure Blob Storage
    └─ Time-series → Azure Data Explorer

For Observability:
    ├─ Logging → Application Insights / Log Analytics
    ├─ Tracing → Application Insights (OpenTelemetry)
    ├─ Metrics → Azure Monitor
    └─ Dashboards → Azure Portal / Grafana

For Security:
    ├─ Secrets → Azure Key Vault
    ├─ Identity → Microsoft Entra ID
    ├─ Network → Network Security Groups, VNet
    └─ Certificate → Azure Key Vault (Certificates)

For Deployment:
    ├─ Infrastructure → Azure Resource Manager (ARM)
    ├─ CI/CD → Azure DevOps or GitHub Actions
    ├─ Artifact Storage → Azure Container Registry
    └─ Policy → Azure Policy
```

### Service Selection Decision Matrix

```
┌──────────────────┬─────────────────────┬────────────────────┬──────────────────┐
│ Requirement      │ Azure App Service   │ Container Instances│ AKS              │
├──────────────────┼─────────────────────┼────────────────────┼──────────────────┤
│ Ease of use      │ ★★★★★ (Easiest)    │ ★★★ (Moderate)    │ ★☆ (Complex)     │
│ Cost             │ ★★ (Higher base)   │ ★★★ (Per-second)  │ ★★★★ (Scaled)    │
│ Auto-scaling     │ ★★★★ (Good)        │ ★★ (Limited)      │ ★★★★★ (Excellent)│
│ Multi-tenancy    │ ★★★ (Via slots)    │ ★★★★ (Via ACI)    │ ★★★★★ (Native)  │
│ Deployment       │ ★★★★ (Fast)        │ ★★★★ (Fast)       │ ★★★ (Complex)    │
│ Best for         │ Simple services,    │ Containerized      │ Large clusters,  │
│                  │ rapid development   │ workloads,         │ multi-region,    │
│                  │                     │ batch jobs         │ auto-scaling     │
│                  │                     │                    │                  │
│ Example use case │ REST API for        │ One-off data       │ Global SaaS      │
│                  │ small team          │ processing,        │ with 100+ pods   │
│                  │                     │ scheduled tasks    │                  │
└──────────────────┴─────────────────────┴────────────────────┴──────────────────┘

┌──────────────────┬──────────────────┬───────────────────┬──────────────────┐
│ Requirement      │ API Management   │ Service Bus       │ Event Grid       │
├──────────────────┼──────────────────┼───────────────────┼──────────────────┤
│ Complexity       │ Moderate         │ Moderate          │ Simple           │
│ Message latency  │ N/A (sync)       │ <1 second         │ Milliseconds     │
│ Ordering         │ N/A              │ ✓ (FIFO queues)   │ Best-effort      │
│ Durability       │ N/A              │ ✓ (Persisted)     │ At-least-once    │
│ Dead-lettering   │ N/A              │ ✓ (Built-in)      │ Retry policy     │
│ TTL              │ N/A              │ ✓ (Configurable)  │ N/A              │
│ Best for         │ API facade,      │ Reliable message  │ Real-time events │
│                  │ versioning,      │ delivery between  │ (webhooks)       │
│                  │ rate limiting    │ services          │                  │
└──────────────────┴──────────────────┴───────────────────┴──────────────────┘

┌──────────────────┬──────────────────┬───────────────────┬──────────────────┐
│ Requirement      │ SQL Database     │ Cosmos DB         │ Cache for Redis  │
├──────────────────┼──────────────────┼───────────────────┼──────────────────┤
│ Data model       │ Relational       │ Document/Graph    │ Key-value        │
│ Consistency      │ ACID             │ Eventual/Tunable  │ Eventual         │
│ Query language   │ T-SQL            │ SQL/Gremlin       │ Redis commands   │
│ Scale (write)    │ Single region    │ Global multi-master│ Single node      │
│ Use case         │ Transactions,    │ Global SaaS,      │ Sessions,        │
│                  │ complex queries  │ IoT data          │ caching          │
└──────────────────┴──────────────────┴───────────────────┴──────────────────┘
```

---

## Operational Excellence Checklist

### Pre-Deployment Checklist

```csharp
public class PreDeploymentChecklist
{
    // ✓ Architecture & Design
    public bool BoundariesClearlyDefined { get; set; }              // DDD applied
    public bool ContractsDocumented { get; set; }                   // OpenAPI specs
    public bool DataIsolationStrategy { get; set; }                 // Per-service DB
    public bool FailureHandlingDesigned { get; set; }               // Circuit breaker, retry
    public bool SlaDefinedPerService { get; set; }                  // Different SLAs
    
    // ✓ Code Quality
    public bool CodeReviewProcess { get; set; }                     // Peer review
    public bool UnitTestCoverage { get; set; }                      // >80% coverage
    public bool IntegrationTests { get; set; }                      // Consumer-driven contracts
    public bool LoadTesting { get; set; }                           // Performance baseline
    public bool SecurityScanning { get; set; }                      // SAST/DAST
    
    // ✓ Observability
    public bool CentralizedLogging { get; set; }                    // Application Insights
    public bool DistributedTracing { get; set; }                    // Correlation IDs
    public bool MetricsInstrumented { get; set; }                   // Custom metrics
    public bool HealthCheckEndpoints { get; set; }                  // Liveness/Readiness
    public bool SlaMonitoring { get; set; }                         // Error rate, latency
    
    // ✓ Security
    public bool AuthenticationConfigured { get; set; }              // Azure AD
    public bool SecretsManaged { get; set; }                        // Key Vault
    public bool NetworkPoliciesEnforced { get; set; }               // NSGs, WAF
    public bool DataEncryption { get; set; }                        // TLS, at-rest
    public bool ComplianceReviewed { get; set; }                    // GDPR, etc.
    
    // ✓ Deployment
    public bool CiCdPipelineConfigured { get; set; }                // Automated builds
    public bool DeploymentStrategyChosen { get; set; }              // Canary/Blue-green
    public bool RollbackPlanDocumented { get; set; }                // Quick rollback
    public bool DeploymentApprovedByArchitect { get; set; }         // Governance
    
    // ✓ Runbooks & Documentation
    public bool RunbooksForCommonIssues { get; set; }               // Troubleshooting guides
    public bool IncidentResponsePlan { get; set; }                  // Escalation matrix
    public bool ArchitectureDecisions { get; set; }                 // ADR documentation
    public bool ApiDocumentation { get; set; }                      // Swagger/OpenAPI
}

// Validation
public class DeploymentValidation
{
    public bool ValidateChecklist()
    {
        var checklist = new PreDeploymentChecklist();
        var properties = typeof(PreDeploymentChecklist).GetProperties();
        
        var allPassed = properties.All(p => (bool)p.GetValue(checklist));
        
        if (!allPassed)
        {
            var failed = properties
                .Where(p => !(bool)p.GetValue(checklist))
                .Select(p => p.Name)
                .ToList();
            
            throw new InvalidOperationException(
                $"Pre-deployment checklist failed:\n" +
                string.Join("\n", failed));
        }
        
        return true;
    }
}
```

### Production Monitoring Checklist

```
┌─────────────────────────────────────────────────────────────┐
│ WEEKLY MONITORING CHECKLIST                                 │
├─────────────────────────────────────────────────────────────┤
│ [ ] Error rates < acceptable threshold for all services     │
│ [ ] P95/P99 latency within SLA                              │
│ [ ] No unexpected traffic pattern changes                   │
│ [ ] Alert threshold tuning (reducing false positives)       │
│ [ ] Review incident reports; identify trends               │
│ [ ] Check deployment success rates (>95%)                   │
│ [ ] Review inter-service dependency health                 │
│ [ ] Resource utilization reasonable (CPU <80%, RAM <85%)    │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ MONTHLY MONITORING CHECKLIST                                │
├─────────────────────────────────────────────────────────────┤
│ [ ] Comprehensive cost analysis per service                 │
│ [ ] Service boundary health report                          │
│ [ ] SLA attainment review (target vs. actual)              │
│ [ ] Database performance (slow queries)                     │
│ [ ] Cache hit ratios analysis                               │
│ [ ] Network bandwidth analysis (cross-region transfer)      │
│ [ ] Security audit (new vulnerabilities)                    │
│ [ ] Data pipeline health (ETL jobs)                         │
│ [ ] Customer incident postmortems                           │
│ [ ] Dependency updates/patches applied                      │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ QUARTERLY MONITORING CHECKLIST                              │
├─────────────────────────────────────────────────────────────┤
│ [ ] Architecture review (boundary alignment)                │
│ [ ] Technical debt assessment                               │
│ [ ] Scaling capacity for next quarter                       │
│ [ ] Disaster recovery drill                                 │
│ [ ] Security assessment (penetration testing)               │
│ [ ] Compliance audit (GDPR, HIPAA, etc.)                    │
│ [ ] Team retrospective (what worked, what didn't)           │
│ [ ] Document lessons learned                                │
│ [ ] Update architecture decision records (ADRs)             │
│ [ ] Plan boundary refactoring (if needed)                   │
└─────────────────────────────────────────────────────────────┘
```

---

## Cost Optimization Strategies

### Cost Optimization Decision Matrix

```
┌────────────────────┬───────────────────┬─────────────────┬──────────────────┐
│ Optimization Area  │ Impact            │ Effort          │ Recommendation   │
├────────────────────┼───────────────────┼─────────────────┼──────────────────┤
│ Reserved Instances │ 30-35% savings    │ Low (plan ahead)│ Always for       │
│ (for predictable   │ on compute        │                 │ baseline load    │
│ load)              │                   │                 │                  │
│                    │                   │                 │                  │
│ Spot instances     │ 70% savings on    │ Moderate        │ For non-critical │
│ (for bursty load)  │ compute           │ (retry logic)   │ workloads        │
│                    │                   │                 │                  │
│ Auto-scaling       │ 20-40% savings    │ Moderate        │ Essential for    │
│ (scale down on     │ on off-peak hours │ (config tuning) │ dynamic workloads│
│ low demand)        │                   │                 │                  │
│                    │                   │                 │                  │
│ Service            │ 10-20% savings    │ Low (per-       │ Consolidate      │
│ consolidation      │ (fewer services)  │ service ROI)    │ loosely coupled  │
│                    │                   │                 │ services         │
│                    │                   │                 │                  │
│ Database optimizer │ 15-25% savings    │ Low (managed    │ Enable auto-tune │
│                    │ on queries        │ service)        │                  │
│                    │                   │                 │                  │
│ CDN for static     │ 50-80% savings    │ Low             │ Essential for    │
│ content            │ on egress         │ (configuration) │ global apps      │
│                    │                   │                 │                  │
│ Regional pricing   │ 10-40% savings    │ High (re-arch)  │ For multi-region │
│ arbitrage          │ based on location │                 │ apps, evaluate   │
│                    │                   │                 │                  │
│ Rightsize         │ 20-30% savings    │ Moderate        │ Review annually; │
│ instances         │ on under-utilized │ (testing,       │ scale down if    │
│                    │ compute           │ monitoring)     │ over-provisioned │
└────────────────────┴───────────────────┴─────────────────┴──────────────────┘
```

### Cost Modeling Template

```csharp
public class ServiceCostModel
{
    public string ServiceName { get; set; }
    
    // Compute
    public class ComputeCosts
    {
        public int BaselineInstanceCount { get; set; }           // Min instances
        public int PeakInstanceCount { get; set; }               // Max instances
        public decimal InstanceCostPerHourUsd { get; set; }      // e.g., $0.25/hour
        public int AvgHoursAtBaseline { get; set; }              // 720h/month baseline
        public int AvgHoursAtPeak { get; set; }                  // 48h/month peak
        
        public decimal MonthlyCost =>
            (BaselineInstanceCount * AvgHoursAtBaseline +
             PeakInstanceCount * AvgHoursAtPeak) * InstanceCostPerHourUsd;
    }
    
    // Storage
    public class StorageCosts
    {
        public long BaselineStorageGb { get; set; }
        public decimal StorageCostPerGbMonth { get; set; }
        public decimal MonthlyCost => BaselineStorageGb * StorageCostPerGbMonth;
    }
    
    // Data transfer
    public class DataTransferCosts
    {
        public long IntraRegionTransferGb { get; set; }          // Free
        public long InterRegionTransferGb { get; set; }
        public decimal InterRegionCostPerGbUsd { get; set; }     // $0.02/GB
        
        public decimal MonthlyCost =>
            InterRegionTransferGb * InterRegionCostPerGbUsd;
    }
    
    // Database
    public class DatabaseCosts
    {
        public string TierName { get; set; }                     // e.g., "S2"
        public decimal MonthlyCostPerTierUsd { get; set; }
        public long StorageGbIncluded { get; set; }
        public long StorageGbUsed { get; set; }
        public decimal ExtraStorageCostPerGbUsd { get; set; }
        
        public decimal MonthlyCost =>
            MonthlyCostPerTierUsd +
            (Math.Max(0, StorageGbUsed - StorageGbIncluded) * ExtraStorageCostPerGbUsd);
    }
    
    public ComputeCosts Compute { get; set; }
    public StorageCosts Storage { get; set; }
    public DataTransferCosts DataTransfer { get; set; }
    public DatabaseCosts Database { get; set; }
    
    public decimal TotalMonthlyCostUsd =>
        Compute.MonthlyCost +
        Storage.MonthlyCost +
        DataTransfer.MonthlyCost +
        Database.MonthlyCost;
    
    public decimal TotalAnnualCostUsd => TotalMonthlyCostUsd * 12;
    
    // Cost per unit (for benchmarking)
    public decimal CostPerMillionRequestsUsd(long monthlyRequests)
    {
        if (monthlyRequests == 0) return 0;
        return (TotalMonthlyCostUsd / monthlyRequests) * 1_000_000;
    }
}

// Example usage
var orderServiceCost = new ServiceCostModel
{
    ServiceName = "OrderService",
    Compute = new ServiceCostModel.ComputeCosts
    {
        BaselineInstanceCount = 2,
        PeakInstanceCount = 10,
        InstanceCostPerHourUsd = 0.25m,
        AvgHoursAtBaseline = 720,
        AvgHoursAtPeak = 48
    },
    Database = new ServiceCostModel.DatabaseCosts
    {
        TierName = "S2",
        MonthlyCostPerTierUsd = 50,
        StorageGbIncluded = 250,
        StorageGbUsed = 150,
        ExtraStorageCostPerGbUsd = 0.10m
    },
    // ... other costs
};

// Monthly: $612 (baseline compute: $360 + peak: $60 + DB: $50)
// Annual: $7,344
// Cost per million requests: $X (if 1M requests/month)
```

---

## Interview Question Quick Reference

### Quick Reference Table

```
┌─────┬────────────────────────────────────┬──────────────────┬─────────────────────────┐
│ Q # │ Topic                              │ Primary Pattern  │ Key Azure Service       │
├─────┼────────────────────────────────────┼──────────────────┼─────────────────────────┤
│  1  │ Defining boundaries with DDD       │ Business         │ Event Grid, Service Bus │
│     │                                    │ Capability       │                         │
│  2  │ Evolving boundaries dynamically    │ Strangler,       │ App Config, APIM        │
│     │                                    │ Monitoring       │                         │
│  3  │ Service contracts & versioning     │ Contract-First   │ API Management          │
│  4  │ Data consistency & isolation       │ Saga, RLS        │ Durable Functions,      │
│     │                                    │                  │ SQL DB                  │
│  5  │ Cross-cutting concerns             │ Infrastructure   │ App Insights, Azure AD  │
│  6  │ Identifying refactoring needs      │ Metrics-driven   │ App Insights, Monitor   │
│  7  │ Service discovery & routing        │ Cloud-native     │ APIM, Azure DNS, AKS    │
│  8  │ Multi-tenant isolation             │ SaaS patterns    │ SQL DB + RLS, Cosmos DB │
│  9  │ Backward compatibility             │ API versioning   │ APIM, Feature flags     │
│ 10  │ Cost optimization & SLAs           │ Cost-aware       │ Cost Management,        │
│     │                                    │ design           │ Reserved Instances      │
└─────┴────────────────────────────────────┴──────────────────┴─────────────────────────┘

Common Interview Patterns:
├─ "Describe your approach to defining service boundaries"
│  └─ Answer: DDD + Strangler pattern + Conway's Law
│
├─ "How do you handle service failures?"
│  └─ Answer: Circuit breaker + retry + fallback + graceful degradation
│
├─ "Tell me about your experience with multi-tenancy"
│  └─ Answer: Database per tenant vs. RLS strategy, data isolation mechanisms
│
├─ "How do you ensure API backward compatibility?"
│  └─ Answer: Versioning strategy, deprecation timeline, contract-first design
│
├─ "What metrics do you monitor for service health?"
│  └─ Answer: Error rate, latency (p95/p99), dependency health, SLA compliance
│
├─ "How do you optimize cloud costs?"
│  └─ Answer: Reserved instances + spot, right-sizing, scaling strategy by SLA
│
└─ "Walk me through a production incident you handled"
   └─ Answer: Root cause (distributed tracing), mitigation (circuit break), resolution
```

### Elevator Pitch Examples

**For "Tell me about your experience with microservices":**

> I've designed service boundaries using Domain-Driven Design, identifying 5-7 Bounded Contexts per system. I use the Strangler Pattern to migrate from monoliths, gradually extracting services while maintaining backward compatibility. I employ event-driven communication for asynchronous coordination and API Management for synchronous contract enforcement. For observability, I implement distributed tracing with correlation IDs and Application Insights monitoring. I optimize costs by using Reserved Instances for baseline load and Spot instances for burst capacity, tailoring SLAs per service based on criticality.

**For "How do you handle distributed system complexity?":**

> I use three strategies: (1) **Observability** - centralized logging, distributed tracing, and metrics in Application Insights; (2) **Resilience** - circuit breakers, retries with exponential backoff, and graceful degradation using Polly policies; (3) **Governance** - architecture reviews, contract-based testing, and automated CI/CD validation. I've found that combining strong observability with well-designed failure modes makes systems comprehensible and predictable, even at scale.

**For "What's your approach to API design?":**

> I use contract-first design: define OpenAPI specs before implementation. I version APIs using URL path versioning (/v1, /v2) for clarity, maintain backward compatibility through additive-only changes, and document deprecation timelines (typically 12-24 months). I implement in Azure API Management, which handles versioning, routing, rate limiting, and policy enforcement transparently. This decouples producer and consumer deployments—consumers can upgrade independently on their schedule.

---

## Architecture Decision Records (ADR) Template

### Recording Service Boundary Decisions

```markdown
# ADR-001: Service Boundary Definition Strategy

## Status
Accepted

## Context
We need to design microservices for our e-commerce platform. 
Current monolith has 500K LOC across order management, inventory, 
billing, and fulfillment capabilities. Deployment takes 4 hours; 
teams cannot move independently. We want faster feature delivery 
and team autonomy.

## Decision
We will define service boundaries using Domain-Driven Design (DDD) 
Bounded Contexts. We identified 5 primary contexts:
1. Order Service (Order creation, tracking, cancellation)
2. Inventory Service (Stock management, reservations)
3. Payment Service (Payment processing, refunds)
4. Fulfillment Service (Shipping, tracking)
5. Customer Service (Customer profiles, preferences)

We will use the Strangler Pattern to migrate from monolith.

## Rationale
- DDD aligns boundaries with business capabilities
- 5 services match our 5 cross-functional teams
- Each service has independent deployment cycle
- Clear business rules per context (order vs. inventory logic is distinct)
- Strangler minimizes migration risk; no big-bang rewrite

## Consequences
- **Positive**: Faster deployments (30 min → 5 min per service), team autonomy
- **Positive**: Independent scaling per service
- **Negative**: New operational complexity (distributed tracing, coordination)
- **Negative**: Increased infrastructure costs (per-service monitoring, databases)
- **Risk**: Over-fragmentation if not careful (addressed by ADR-003)

## Alternatives Considered
1. **Monolith (status quo)** - Rejected: Doesn't solve deployment bottleneck
2. **Technical-layer microservices** - Rejected: Would couple data layer across services
3. **Hybrid (5 services + shared platform)** - Considered, similar to chosen approach

## Approval
- Architecture Review Board: ✓ Approved
- Chief Architect: ✓ Approved
- Date: 2024-01-15

---

# ADR-002: Data Isolation Strategy

## Status
Accepted

## Decision
We will use database-per-service pattern for high-isolation services 
(Order, Payment, Fulfillment) and shared database + Row-Level Security 
(RLS) for lower-risk services (Customer, Product Catalog).

## Rationale
- Critical services need complete isolation; database-per-service ensures this
- Lower-risk services can share infrastructure for cost efficiency
- RLS provides automatic row filtering by tenant at database level

## Consequences
- Order service can evolve schema independently (+ feature velocity)
- Payment service has isolated data in case of breach (+ security)
- Customer service shares infrastructure (- cost)
- Must implement data sync for cross-service queries via APIs (+ complexity)

---

# ADR-003: API Versioning Strategy

## Status
Accepted

## Decision
- URL path versioning (/api/v1/orders, /api/v2/orders)
- Maintain 2 concurrent major versions (v1 support for 12 months from v2 release)
- Deprecation header: `Deprecation: true` + `Sunset: <ISO-8601 date>`

## Rationale
- URL versioning is explicit and easy to debug
- 12-month support window allows consumers time to migrate
- Deprecation headers follow HTTP standards (RFC 8594)

## Consequences
- Multiple code paths to maintain during transition
- Clients may ignore deprecation warnings (need active migration support)
```

---

## Final Recommendations by Scenario

### Scenario 1: Greenfield SaaS (New Product)

**Recommended approach:**
```
1. DEFINE: 3-5 bounded contexts using Event Storming
2. IMPLEMENT: Azure App Service + SQL Database per service
3. COMMUNICATION: Service Bus (async) + APIM (sync)
4. DATA: Database per service + RLS for multi-tenancy
5. COST: Start with standard tier; right-size after 3 months
6. DEPLOYMENT: Canary deployment; auto-rollback on error spike
7. MONITORING: Application Insights centralized logging; 
             track cost per tenant
```

**Azure Services:**
- App Service (host services)
- SQL Database (per-service databases)
- Service Bus (async messaging)
- API Management (API versioning)
- Application Insights (observability)
- Key Vault (secrets management)

**Cost estimate:** $15-25K/month (small team, 3-5 services)

---

### Scenario 2: Monolith Migration

**Recommended approach:**
```
1. ASSESS: Map monolith dependencies; identify tight coupling points
2. PRIORITIZE: Extract highest-value service first (e.g., Order Service)
3. STRANGLER: Deploy new service alongside monolith; gradual traffic shift
4. AUTOMATE: Extensive testing; automated rollback on failure
5. MIGRATE: Dual-write during transition; validate consistency
6. RETIRE: Once 100% traffic on new service, retire monolith endpoint
7. ITERATE: Repeat for remaining services (2-3 per quarter)
```

**Timeline:** 12-24 months for complete migration (depends on monolith size)

**Risk mitigation:**
- Maintain monolith for 2+ quarters after service extraction
- Keep detailed rollback procedures
- Test failure scenarios before production

---

### Scenario 3: Scaling Existing Microservices

**Recommended approach:**
```
1. AUDIT: Run boundary health analysis (Q6 analysis)
2. IDENTIFY: Over-fragmented or under-fragmented areas
3. CONSOLIDATE: Merge loosely-coupled services (reduce overhead)
4. SPLIT: Services with conflicting scaling needs or SLAs
5. COST OPTIMIZE: Use Reserved Instances + Spot instances
6. MONITOR: Track cost per service; quarterly review
```

**Expected improvements:**
- 20-30% cost reduction through consolidation
- 40-60% faster deployments
- Reduced operational complexity

---

## Conclusion

Service boundary definition is **not a one-time decision** but an **ongoing architectural practice**. Use this framework to:

1. **Establish initial boundaries** based on business capabilities and team structure (Q1-Q2)
2. **Implement supporting infrastructure** for communication, observability, and resilience (Q3-Q5)
3. **Monitor continuously** and refactor based on metrics (Q6)
4. **Evolve with the business** as requirements change (Q7-Q10)

### Key Success Metrics:

| Metric | Target | Measurement |
|--------|--------|-------------|
| Deployment frequency | 2+ per week per service | DevOps metrics |
| Lead time for change | <1 week | DevOps metrics |
| Change failure rate | <15% | DevOps metrics |
| Mean time to recovery | <1 hour | Incident response |
| API backward compatibility | 100% within deprecation window | Contract tests |
| Service availability | >99.9% for critical, >99% for supporting | SLA monitoring |
| Cost per request | Decreases quarterly | Cost analysis |
| Mean time to understand incident | <15 minutes | Distributed tracing |

---

**This framework provides the complete blueprint for designing, implementing, and optimizing service boundaries in Azure microservices architectures. Use it as a reference guide for architectural decisions, interview preparation, and production system design.**

---

---

## Summary Table: Cost Optimization by Service Boundary Pattern

| Pattern | Cost Implication | Scaling Efficiency | SLA Support | Recommendation |
|---------|------------------|-------------------|-------------|-----------------|
| **Many fine-grained services** | High (overhead per service) | Excellent (independent scaling) | Per-service SLA | Use when independent scaling critical |
| **Few coarse-grained services** | Low (minimal overhead) | Poor (scale together) | Uniform SLA | Use for cost-sensitive workloads |
| **Cost-optimized hybrid** | Moderate | Good (scale by group) | Mixed SLA | Sweet spot for most organizations |
| **Multi-region active-active** | Very high (2-3X single region) | Excellent + failover | 99.99%+ | Only for critical, global services |
| **Single-region with burst** | Moderate (reserved + spot) | Good (burst scaling) | 99.9% | Ideal for SaaS with predictable spikes |

---

## Final Integration: Questions 1–10 Complete Framework

### Service Boundary Design Lifecycle:

1. **Q1–Q2**: Define initial boundaries (DDD, team structure, monitoring-driven refinement)
2. **Q3**: Establish clear contracts and versioning strategy
3. **Q4**: Design data isolation and consistency mechanisms
4. **Q5**: Implement cross-cutting concerns (observability, security, tracing)
5. **Q6**: Monitor boundaries; identify refactoring needs
6. **Q7**: Implement dynamic discovery and routing for cloud-native deployment
7. **Q8**: Add multi-tenancy support if building SaaS
8. **Q9**: Design for backward compatibility and graceful degradation
9. **Q10**: Optimize costs while maintaining SLAs and operational efficiency

### Architecture Decision Framework:

| Decision Point | Considerations | Questions to Ask |
|----------------|-----------------|-----------------|
| **Boundary Definition** | Business capability, team structure, deployment frequency | Q1, Q2 |
| **Service Contract** | API versioning, backward compatibility, evolution strategy | Q3, Q9 |
| **Data Management** | Isolation, consistency, GDPR compliance, multi-tenancy | Q4, Q8 |
| **Operations** | Observability, security, discovery, routing | Q5, Q7 |
| **Evolution & Health** | Monitoring, refactoring, graceful degradation | Q6, Q9 |
| **Business Constraints** | Cost, SLA, compliance, regional requirements | Q10 |

This comprehensive 10-question framework covers the **complete lifecycle of service boundary management** in Azure microservices, from initial definition through operational excellence and cost optimization.

---

## Summary Table: Service Discovery & Multi-Tenancy Patterns

| Pattern | Use Case | Pros | Cons | Azure Tool |
|---------|----------|------|------|------------|
| **DNS-based Discovery** | Azure App Service, VMs | Simple, low latency | Limited health checks | Azure DNS, Private DNS |
| **APIM Gateway** | All services | Centralized, rich features | Single point of failure | API Management |
| **Kubernetes DNS** | AKS deployments | Automatic, scalable | Kubernetes-specific | AKS, Kubernetes DNS |
| **Database per Tenant** | High compliance needs | Strongest isolation | Operational overhead | Azure SQL Databases |
| **Shared DB + RLS** | Cost-sensitive multi-tenant | Efficient, shared compute | Complex RLS policies | Azure SQL |
| **Silo Model** | Single-tenant per infrastructure | Complete isolation | Lowest efficiency | VM/Container per tenant |
| **Pool Model** | Multi-tenant SaaS | Cost-efficient | Requires careful isolation | Shared infrastructure + RLS |

---

## Combined Q7-Q8 Interview Guide

**When service discovery and multi-tenancy converge:**

- **Service discovery per tenant**: In highly isolated environments, each tenant may have dedicated service instances; discovery must route tenant-specific requests to tenant-specific services. Use traffic routing rules in APIM or service mesh.
- **Tenant context in discovery**: Service discovery calls must include tenant context; discovered instances should be tenant-specific or partition-aware (know which tenant data they serve).
- **Cascading tenant context**: When Order service discovers Inventory service, it passes tenant context; Inventory uses this to return only that tenant's data. Discovery doesn't leak cross-tenant instances.
- **Compliance-driven discovery**: Tenant's compliance policy might require on-premises discovery; route those tenants to on-premises service registry instead of Azure-based discovery.

---

This comprehensive treatment of Questions 7 and 8 covers the advanced scenarios of **dynamic, cloud-native infrastructure** (service discovery) and **business model complexity** (multi-tenancy)—both critical for modern Azure microservices architectures in production.

---

## Summary Table: Service Boundary Health Signals and Recommended Actions

| Signal | Severity | Cause | Recommended Action | Azure Tools |
|--------|----------|-------|-------------------|-------------|
| **Dependency Ratio >70%** | High | Over-fragmentation | Merge services | API Management, Strangler Pattern |
| **Circular Dependencies** | Critical | Design flaw | Introduce intermediary service | Dependency Analyzer, Application Insights |
| **Average Latency >500ms** | Medium | Excessive inter-service calls | Cache, batch requests, or merge | Application Insights, Azure Cache |
| **Deployment Coupling >30%** | High | Boundary misalignment | Realign to team structure | DevOps Analytics |
| **Uneven Scaling Patterns** | Medium | Conflicting concerns | Split service by scaling needs | Azure Monitor, Autoscale rules |
| **Frequent Timeouts** | High | Service overload or latency | Scale service or optimize boundary | Application Insights, Azure Autoscale |

---

## Final Comparison: Questions 1–6 (Complete Service Boundary Coverage)

| Question | Primary Pattern | Key Focus | Decision Lever |
|----------|-----------------|-----------|-----------------|
| **Q1: Defining Boundaries** | DDD, Strangler | Business capability, team structure | Domain language boundaries |
| **Q2: Ambiguous Requirements** | Iterative refinement | Monitoring-driven evolution | Coupling metrics, deployment frequency |
| **Q3: Service Contracts** | Contract-first, versioning | API design, backward compatibility | Contract stability, consumer count |
| **Q4: Data Consistency** | Saga, database-per-service | Eventual consistency, GDPR | Data ownership clarity, compliance needs |
| **Q5: Cross-Cutting Concerns** | Infrastructure patterns | Observability, security, tracing | Centralization via platform services |
| **Q6: Boundary Refactoring** | Health metrics, Conway's Law | Detecting mis-boundaries | Call ratios, deployment coupling, team structure |

---

This comprehensive set of questions covers the full lifecycle of service boundary management in Azure microservices: definition, evolution, contract enforcement, data isolation, operational concerns, and ongoing health. Each question is grounded in practical patterns, real Azure services, and decision frameworks useful in actual interviews and architecture decisions.

---

You are a senior .NET architect and technical interview trainer specializing in **Azure-based Microservices Architecture**.
Under the topic **“Microservices Fundamentals & Architecture”**, generate **7–10 interview questions** specifically focused on:
> **Service boundaries definition**
For **each question**, use the following structure:
---
## Question X: [Title]
### Detailed Answer:
* Write a clear, 2–3 paragraph explanation covering core concepts and Azure/.NET relevance.
### Explanation:
* Explain importance, use cases, benefits, and trade-offs from an architecture/interview perspective.
### C# Implementation:
* Provide clean, modern C# code (10–25 lines) with comments.
* Focus on Azure-based scenarios (e.g., Azure Functions, Service Bus, APIs).
### Follow-Up Questions:
* Add 3–5 related follow-up questions in this format:
  **Q1: [Question]**
  * **Answer:** [5–10 sentence focused answer]
---
**Additional Guidelines:**
* Include database schema where applicable.
* Stick to **Azure** context only (no AWS/GCP).
* Use **.NET 6+/C# 10+** coding standards.
* Prioritize practical, applied answers — not theory-heavy responses.

Generate 1 or 2 questions with full structure described above.