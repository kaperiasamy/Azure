[LMArena](https://lmarena.ai/)


# Microservices Interview Preparation Guide for Senior Roles

## Part 1: Common Microservices Interview Questions for Senior Roles

### **Fundamentals & Architecture**

1. What are microservices? How do they differ from monolithic architecture?
2. What are the key characteristics of microservices architecture?
3. When should you choose microservices over monolithic architecture, and vice versa?
4. What are the main challenges/disadvantages of microservices?
5. Explain the concept of bounded context in Domain-Driven Design (DDD)
6. How do you determine the right size/scope for a microservice?
7. What is the difference between microservices and SOA (Service-Oriented Architecture)?
8. Explain the concept of "database per service" pattern
9. What is the strangler fig pattern for migrating from monolith to microservices?

### **Communication Patterns**

10. What are the different communication patterns between microservices? (Synchronous vs Asynchronous)
11. When would you use REST vs messaging/event-driven communication?
12. Explain the Saga pattern for distributed transactions
13. What is choreography vs orchestration in microservices?
14. How do you handle eventual consistency in microservices?
15. What is the Circuit Breaker pattern and why is it important?
16. Explain the API Gateway pattern and its benefits
17. What is service mesh? When would you use it?
18. How do you handle versioning of APIs in microservices?

### **Data Management**

19. How do you handle distributed transactions across microservices?
20. Explain CQRS (Command Query Responsibility Segregation) pattern
21. What is Event Sourcing and when would you use it?
22. How do you maintain data consistency across services?
23. What strategies can you use for sharing data between microservices?
24. How do you handle database migrations in a microservices environment?

### **Resilience & Reliability**

25. What is the Bulkhead pattern?
26. How do you implement retry logic in microservices?
27. Explain the concept of idempotency and why it's important
28. What is the Timeout pattern?
29. How do you handle cascading failures?
30. What strategies do you use for fault tolerance?

### **Service Discovery & Configuration**

31. What is service discovery and why is it needed?
32. Explain client-side vs server-side service discovery
33. How do you manage configuration in microservices?
34. What is the difference between centralized and distributed configuration?

### **Security**

35. How do you implement authentication and authorization in microservices?
36. What is the BFF (Backend for Frontend) pattern?
37. How do you secure inter-service communication?
38. Explain OAuth2 and OpenID Connect in context of microservices
39. What is mutual TLS (mTLS)?

### **Deployment & DevOps**

40. How do you deploy microservices?
41. What is containerization and why is it important for microservices?
42. Explain the concept of CI/CD in microservices context
43. What is blue-green deployment and canary deployment?
44. How do you manage different environments (dev, staging, prod)?

### **Monitoring & Observability**

45. What is the difference between monitoring and observability?
46. How do you implement distributed tracing?
47. What is correlation ID and why is it important?
48. How do you aggregate logs from multiple microservices?
49. What metrics should you monitor in a microservices architecture?
50. Explain the concept of health checks

### **Azure & .NET Core Specific**

51. What Azure services are commonly used for microservices?
52. How would you implement microservices using Azure Service Fabric vs AKS?
53. What is Azure Service Bus and when would you use it?
54. How do you implement microservices using .NET Core?
55. What is the role of API Management in Azure?

### **Architectural Scenarios (for Architect roles)**

56. How would you design an e-commerce application using microservices?
57. How do you handle a situation where one microservice is a bottleneck?
58. How would you migrate a legacy monolithic application to microservices?
59. How do you ensure data integrity across multiple microservices?
60. What trade-offs would you consider when designing a microservices architecture?

---

## Part 2: Detailed Study Plan (8-Week Program)

### **Week 1: Fundamentals & Core Concepts**

#### **Learning Objectives:**
- Understand what microservices are and when to use them
- Learn the differences from monolithic architecture
- Understand key characteristics and challenges

#### **Study Materials:**
1. **Books to Start:**
   - "Building Microservices" by Sam Newman (Chapters 1-3)
   - Martin Fowler's microservices articles (martinfowler.com/microservices)

2. **Online Resources:**
   - Microsoft's Microservices Architecture Guide (.NET)
   - YouTube: "Microservices Explained" by TechWorld with Nana

3. **Topics to Cover:**
   - Monolithic vs Microservices comparison
   - Characteristics: independently deployable, business-focused, decentralized
   - Challenges: distributed systems complexity, data consistency, testing
   - Domain-Driven Design basics
   - Bounded Context concept

#### **Hands-on Practice:**
- Create a simple monolithic application (e.g., a basic e-commerce with products and orders)
- Document how you would break it into microservices
- Draw architecture diagrams

#### **Weekend Project:**
- Build your first simple microservice using ASP.NET Core Web API
- Create a Product Service with basic CRUD operations
- Use Entity Framework Core with SQL Server

---

### **Week 2: Communication Patterns**

#### **Learning Objectives:**
- Master synchronous and asynchronous communication
- Understand REST, gRPC, and messaging
- Learn API Gateway pattern

#### **Study Materials:**
1. **Reading:**
   - "Building Microservices" Chapter 4 (Integration)
   - Microsoft Docs: Communication in a microservice architecture

2. **Key Concepts:**
   - REST APIs and HTTP communication
   - Message brokers (RabbitMQ, Azure Service Bus)
   - Request/Response vs Event-driven
   - API Gateway pattern (Ocelot, Azure API Management)
   - gRPC basics

#### **Hands-on Practice:**
- Create two microservices (Product Service & Order Service)
- Implement REST-based synchronous communication
- Add Ocelot API Gateway
- Implement basic error handling

#### **Code Example Structure:**
```
ProductService (Port 5001)
  - GET /api/products
  - GET /api/products/{id}
  - POST /api/products

OrderService (Port 5002)
  - Calls ProductService to validate products
  - GET /api/orders
  - POST /api/orders

ApiGateway (Port 5000)
  - Routes to both services
```

---

### **Week 3: Resilience Patterns**

#### **Learning Objectives:**
- Implement Circuit Breaker, Retry, and Timeout patterns
- Handle failures gracefully
- Build resilient microservices

#### **Study Materials:**
1. **Reading:**
   - Microsoft Docs: Handle partial failure with resilience patterns
   - Polly documentation (resilience library for .NET)

2. **Key Patterns:**
   - Circuit Breaker Pattern
   - Retry with exponential backoff
   - Timeout pattern
   - Bulkhead isolation
   - Fallback strategies

#### **Hands-on Practice:**
- Install Polly NuGet package in your services
- Implement Circuit Breaker for inter-service calls
- Add retry logic with exponential backoff
- Simulate failures and test resilience

#### **Code Example:**
```csharp
services.AddHttpClient<IProductService, ProductService>()
    .AddPolicyHandler(GetRetryPolicy())
    .AddPolicyHandler(GetCircuitBreakerPolicy());

static IAsyncPolicy<HttpResponseMessage> GetRetryPolicy()
{
    return HttpPolicyExtensions
        .HandleTransientHttpError()
        .WaitAndRetryAsync(3, retryAttempt => 
            TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)));
}
```

---

### **Week 4: Data Management & Patterns**

#### **Learning Objectives:**
- Understand database per service pattern
- Learn about distributed transactions
- Master CQRS and Event Sourcing basics

#### **Study Materials:**
1. **Reading:**
   - "Building Microservices" Chapter 5 (Splitting the Monolith)
   - Chris Richardson's microservices.io patterns
   - CQRS pattern documentation

2. **Key Concepts:**
   - Database per service
   - Saga pattern (Choreography vs Orchestration)
   - Event Sourcing
   - CQRS pattern
   - Eventual consistency

#### **Hands-on Practice:**
- Implement database per service (separate DBs for Product & Order)
- Create a simple Saga for order placement:
  - Reserve inventory
  - Process payment
  - Create order
- Implement compensating transactions for rollback

#### **Project Enhancement:**
```
Services:
1. Product Service (SQL Server DB)
2. Order Service (SQL Server DB)
3. Inventory Service (SQL Server DB)
4. Payment Service (In-memory for now)

Implement Choreography Saga using events
```

---

### **Week 5: Message Brokers & Async Communication**

#### **Learning Objectives:**
- Master asynchronous messaging
- Learn RabbitMQ and Azure Service Bus
- Implement event-driven architecture

#### **Study Materials:**
1. **Reading:**
   - RabbitMQ tutorials
   - Azure Service Bus documentation
   - Event-driven architecture patterns

2. **Key Concepts:**
   - Publish/Subscribe pattern
   - Message queues vs Topics
   - Message durability and acknowledgments
   - Dead letter queues
   - MassTransit or NServiceBus basics

#### **Hands-on Practice:**
- Set up RabbitMQ locally (Docker)
- Refactor your services to use event-driven communication
- Implement:
  - OrderCreated event
  - ProductUpdated event
  - InventoryReserved event

#### **Implementation:**
```csharp
// Use MassTransit with RabbitMQ
public class OrderCreatedConsumer : IConsumer<OrderCreated>
{
    public async Task Consume(ConsumeContext<OrderCreated> context)
    {
        // Handle the event
        var order = context.Message;
        // Process order logic
    }
}
```

---

### **Week 6: Service Discovery, Configuration & Containerization**

#### **Learning Objectives:**
- Learn service discovery patterns
- Master Docker and containerization
- Understand configuration management

#### **Study Materials:**
1. **Reading:**
   - Docker documentation for .NET
   - Consul or Azure Service Discovery docs
   - 12-Factor App methodology

2. **Key Concepts:**
   - Service Registry pattern
   - Client-side vs Server-side discovery
   - Consul, Eureka
   - Centralized configuration (Azure App Configuration, Consul)
   - Docker fundamentals
   - Docker Compose

#### **Hands-on Practice:**
- Create Dockerfile for each microservice
- Set up Docker Compose for local development
- Implement Consul for service discovery
- Use environment variables for configuration

#### **Docker Compose Example:**
```yaml
version: '3.8'
services:
  productservice:
    build: ./ProductService
    ports:
      - "5001:80"
    environment:
      - ConnectionStrings__DefaultConnection=...
  
  orderservice:
    build: ./OrderService
    ports:
      - "5002:80"
  
  rabbitmq:
    image: rabbitmq:3-management
    ports:
      - "5672:5672"
      - "15672:15672"
```

---

### **Week 7: Security, Monitoring & Observability**

#### **Learning Objectives:**
- Implement authentication and authorization
- Set up distributed tracing
- Configure logging and monitoring

#### **Study Materials:**
1. **Reading:**
   - IdentityServer4 or Azure AD B2C documentation
   - OpenTelemetry documentation
   - Serilog and Seq documentation

2. **Key Concepts:**
   - OAuth 2.0 and OpenID Connect
   - JWT tokens
   - API Gateway authentication
   - Distributed tracing (Jaeger, Zipkin)
   - Structured logging
   - Correlation IDs
   - Health checks

#### **Hands-on Practice:**
- Implement IdentityServer4 or use Azure AD
- Add JWT authentication to your services
- Implement Serilog with Seq
- Add correlation IDs to track requests
- Set up health checks endpoint
- Implement OpenTelemetry or Application Insights

#### **Code Example:**
```csharp
// Add correlation ID middleware
app.Use(async (context, next) =>
{
    var correlationId = context.Request.Headers["X-Correlation-ID"]
        .FirstOrDefault() ?? Guid.NewGuid().ToString();
    
    context.Response.Headers.Add("X-Correlation-ID", correlationId);
    
    using (LogContext.PushProperty("CorrelationId", correlationId))
    {
        await next();
    }
});
```

---

### **Week 8: Azure, Kubernetes & Real-world Architecture**

#### **Learning Objectives:**
- Deploy microservices to Azure
- Learn Kubernetes basics (AKS)
- Design a complete microservices solution

#### **Study Materials:**
1. **Reading:**
   - Azure Kubernetes Service (AKS) documentation
   - Azure Service Fabric documentation
   - Kubernetes basics

2. **Key Azure Services:**
   - Azure Kubernetes Service (AKS)
   - Azure Container Registry (ACR)
   - Azure Service Bus
   - Azure API Management
   - Azure Application Insights
   - Azure Key Vault

#### **Hands-on Practice:**
- Create Azure account (free tier)
- Push Docker images to Azure Container Registry
- Deploy to AKS or Azure App Service
- Set up Azure Service Bus
- Configure Azure API Management
- Implement Application Insights

#### **Final Project:**
Build a complete e-commerce microservices application:

```
Architecture:
- API Gateway (Azure API Management or Ocelot)
- Identity Service (IdentityServer4/Azure AD)
- Product Catalog Service
- Shopping Cart Service
- Order Service
- Inventory Service
- Payment Service (mock)
- Notification Service (email/SMS)

Technologies:
- ASP.NET Core 6/7
- Entity Framework Core
- RabbitMQ or Azure Service Bus
- Redis for caching
- SQL Server
- Docker & Kubernetes
- Azure services
```

---

## Study Schedule (Daily Breakdown)

### **Monday-Friday (2-3 hours/day):**
- **Evening Study (1.5 hours):**
  - Read concepts and documentation
  - Watch tutorial videos
  - Take notes on key patterns

- **Coding Practice (1-1.5 hours):**
  - Hands-on implementation
  - Build/enhance your project

### **Saturday (4-5 hours):**
- Deep dive into weekly topic
- Complete weekend project
- Review and refactor code
- Write documentation

### **Sunday (2-3 hours):**
- Review the week
- Practice interview questions
- Prepare for next week
- Blog about what you learned (optional but recommended)

---

## Essential Tools & Technologies

### **Development Tools:**
- Visual Studio 2022 or VS Code
- Docker Desktop
- Postman or Swagger
- SQL Server Management Studio
- Git

### **Technologies to Practice:**
- ASP.NET Core 6/7 Web API
- Entity Framework Core
- Docker
- RabbitMQ or Azure Service Bus
- Redis
- Ocelot or Azure API Management
- IdentityServer4 or Azure AD
- Polly
- MassTransit or NServiceBus
- Serilog
- xUnit for testing

---

## Recommended Resources

### **Books (in order of priority):**
1. **"Building Microservices" by Sam Newman** (2nd Edition) - THE definitive book
2. **"Microservices Patterns" by Chris Richardson** - Excellent for patterns
3. **".NET Microservices: Architecture for Containerized .NET Applications"** by Microsoft (Free eBook)

### **Online Courses:**
1. **Pluralsight:**
   - "Microservices Architecture" by Rag Dhiman
   - "Getting Started with ASP.NET Core Microservices" by Steve Gordon

2. **Udemy:**
   - "Microservices Architecture and Implementation on .NET 5" by Mehmet Ozkaya
   - Look for courses with hands-

--- 

I’m a software developer with over 16 years of experience. I began my career with FoxPro, then worked with VB6 and other languages, and transitioned to .NET around 12 years ago. I’m currently seeking a job change, aiming for roles such as Lead Developer, Technical Architect, or Solution Architect. My goal is to move into modern technologies like .NET Core, ASP.NET Core, Web API, Microservices, and Azure.

However, my knowledge in these areas—especially Microservices—is currently limited, which is making it difficult to clear technical interviews for senior roles. I want to focus on one topic at a time, starting with **Microservices**.

Please provide:

1. A list of common **Microservices interview questions** for senior roles (Lead, Architect).
2. A **detailed study plan** to learn Microservices thoroughly and prepare for interviews, including concepts, best practices, architecture patterns, and hands-on skills.

I have to be ready within a week of time.

--- 

# Intensive 1-Week Microservices Interview Crash Course

Given your 16+ years of experience, you already understand core software development principles. This accelerated plan focuses on **critical microservices concepts** and **high-impact interview preparation**.

---

## Part 1: Priority Interview Questions (Must Know)

### **Tier 1: CRITICAL (80% of interviews will cover these)**

1. What are microservices and how do they differ from monolithic architecture?
2. When should you use microservices vs monolith?
3. What are the main challenges of microservices?
4. Explain synchronous vs asynchronous communication patterns
5. What is the Circuit Breaker pattern and why is it important?
6. How do you handle distributed transactions? (Saga pattern)
7. Explain the "database per service" pattern
8. What is service discovery and why is it needed?
9. How do you ensure data consistency across microservices?
10. Explain the API Gateway pattern
11. What is the strangler fig pattern for migration?
12. How do you handle versioning of APIs?
13. What is the difference between choreography and orchestration?
14. Explain eventual consistency
15. What Azure services support microservices? (Service Bus, AKS, API Management)

### **Tier 2: IMPORTANT (Design discussions)**

16. How would you design an e-commerce system using microservices?
17. How do you implement security (Authentication/Authorization) across services?
18. What is service mesh? (Istio, Linkerd)
19. Explain CQRS pattern
20. What is Event Sourcing?
21. How do you monitor and trace distributed systems?
22. Explain the BFF (Backend for Frontend) pattern
23. How do you handle cascading failures?
24. What is the Bulkhead pattern?
25. How do you structure teams around microservices? (Conway's Law)

### **Tier 3: NICE-TO-KNOW (If time permits)**

26. Explain idempotency and why it matters
27. What is correlation ID in distributed tracing?
28. How do you implement health checks?
29. Explain blue-green and canary deployments
30. What is the relationship between DDD and microservices?

---

## Part 2: Intensive 7-Day Study Plan

### **Day 1: Monday - Fundamentals & Architecture (4-5 hours)**

#### **Morning (2 hours): Core Concepts**

**Read (1 hour):**
- Martin Fowler's Microservices article: https://martinfowler.com/articles/microservices.html
- Bookmark: https://microservices.io/patterns/index.html (Chris Richardson's site)

**Key takeaways to remember:**
```
MICROSERVICES = Small independent services that:
✓ Run in separate processes
✓ Own their data (database per service)
✓ Communicate via well-defined APIs
✓ Are independently deployable
✓ Are organized around business capabilities

MONOLITH vs MICROSERVICES:
Monolith: Single codebase, shared database, scaled as single unit
Microservices: Multiple codebases, separate DBs, scale independently
```

**Why Microservices?**
- Independent deployment
- Technology diversity
- Scale what needs scaling
- Fault isolation

**Challenges:**
- Network latency & failures (don't assume synchronous calls work)
- Data consistency (no distributed transactions)
- Operational complexity (more moving parts)
- Testing difficulty
- Debugging distributed systems

**Afternoon (1.5 hours): When to Use Microservices**

**Decision Matrix to memorize:**

| Factor | Use Microservices | Use Monolith |
|--------|------------------|-------------|
| Team size | Large (6+ teams) | Small (<6) |
| Scalability needs | Variable per feature | Uniform |
| Deployment frequency | High (multiple/week) | Low (monthly) |
| Technology diversity | Needed | Not needed |
| Maturity | Mature org, DevOps ready | Early stage |
| Team distribution | Multiple locations | Co-located |
| Performance critical | If latency acceptable | If latency critical |

**Lecture video (30 min):**
- YouTube: "Microservices by Sam Newman" (O'Reilly)

**Evening (1 hour): Bounded Context & Domain-Driven Design**

**Key concept:**
```
BOUNDED CONTEXT = Clear boundary around a microservice
Each service has ONE reason to change (Single Responsibility)

Example - E-commerce:
- Order Service (bounded context: orders)
- Product Service (bounded context: products)
- Inventory Service (bounded context: stock)
- Shipping Service (bounded context: logistics)

Each service owns its model and database for that domain
```

**Day 1 Deliverable:** Be able to answer:
- "What are 3 key differences between microservices and monolith?"
- "List 2 scenarios where microservices are WRONG choice"

---

### **Day 2: Tuesday - Communication Patterns (4-5 hours)**

#### **Morning (2.5 hours): Synchronous vs Asynchronous**

**Critical Decision:**

| Aspect | Synchronous (REST) | Asynchronous (Events) |
|--------|-------------------|----------------------|
| How? | HTTP/REST calls | Message Broker |
| Speed | Immediate response | Fire and forget |
| Coupling | Tight coupling | Loose coupling |
| Reliability | Call fails if service down | Message stored, processed later |
| Latency | Higher | Lower perceived latency |
| Consistency | Immediate | Eventual |
| Best for | Read operations, immediate feedback | Notifications, background tasks |

**Synchronous Example:**
```
OrderService makes REST call to ProductService
POST /api/orders
{
  "productId": 123,
  "quantity": 2
}

OrderService → REST Call → ProductService
ProductService checks inventory, returns immediately
```

**Asynchronous Example:**
```
OrderService publishes OrderCreated event
Message broker stores it
InventoryService subscribes to OrderCreated
InventoryService processes it in background
Eventual consistency
```

**When to use each:**
- **Sync REST:** Shopping cart validation, displaying product info, login
- **Async messaging:** Order confirmation emails, inventory updates, notifications

**Afternoon (1.5 hours): API Gateway Pattern**

**Key concept:**
```
API Gateway = Single entry point for all client requests
Advantages:
✓ Single URL for clients
✓ Cross-cutting concerns (auth, logging)
✓ Version management
✓ Service protection

Tools: Ocelot, Kong, Azure API Management, AWS API Gateway
```

**Architecture diagram to understand:**
```
Client
  ↓
API Gateway (Port 80)
  ├→ ProductService (5001)
  ├→ OrderService (5002)
  ├→ UserService (5003)
  └→ InventoryService (5004)
```

**Evening (1 hour): gRPC Basics**

**Quick intro:**
```
gRPC = Google Remote Procedure Call
- Binary protocol (faster than JSON)
- HTTP/2 (multiplexing)
- Strongly typed (Protocol Buffers)
- Better performance than REST

When to use gRPC:
✓ High-performance inter-service communication
✓ Real-time streaming
✗ Public APIs (doesn't work with browsers directly)
✗ Simple CRUD operations (overkill)
```

**Day 2 Deliverable:** Be able to explain:
- "When would you use REST vs messaging?"
- "Draw a simple API Gateway architecture"
- "What is gRPC and when to use it?"

---

### **Day 3: Wednesday - Data Management & Resilience (5 hours)**

#### **Morning (2 hours): Database Per Service Pattern**

**Core principle:**
```
RULE: Each microservice owns its database
NO shared databases across services

Why?
✓ Independent scalability
✓ Technology choice flexibility
✓ Loose coupling
✓ Independent deployment

Challenge: How do you get data from other services?
Answer: Call their APIs or subscribe to events
```

**Database choice per service:**
```
Product Service → SQL Server (structured data)
User Sessions → Redis (fast reads)
Search/Analytics → Elasticsearch
Documents → MongoDB
File Storage → Object Store (Azure Blob)
```

**Afternoon (2 hours): Handling Distributed Transactions - SAGA PATTERN**

**THIS IS CRITICAL FOR INTERVIEWS**

**The Problem:**
```
Traditional transaction with 2PC (Two-Phase Commit):
Order Service: BEGIN TRANSACTION
  → Reserve inventory
  → Process payment
  → Create order
COMMIT

Problem: Service fails mid-transaction, no rollback!
```

**Solution: SAGA PATTERN**

```
SAGA = Business process that spans multiple services
Uses compensating transactions (rollback logic)

TWO TYPES:

1. CHOREOGRAPHY (Event-driven)
   OrderService publishes OrderCreated
   ↓ InventoryService receives, reserves stock, publishes InventoryReserved
   ↓ PaymentService receives, processes payment, publishes PaymentProcessed
   ↓ ShippingService receives, prepares shipment

   Advantage: Loose coupling
   Disadvantage: Complex to visualize, circular dependencies

2. ORCHESTRATION (Workflow engine)
   OrderOrchestrator coordinates:
   1. Call InventoryService → ReserveStock()
   2. If success, call PaymentService → ProcessPayment()
   3. If success, call ShippingService → PrepareShipment()
   4. If any fails, execute compensating transactions

   Advantage: Clear flow, easier to understand
   Disadvantage: Central orchestrator (single point of failure, tighter coupling)
```

**Real Example:**
```csharp
// Choreography example
public class OrderService
{
    public async Task CreateOrder(CreateOrderRequest request)
    {
        var order = new Order { ... };
        await _repository.Save(order);
        
        // Publish event - other services listen
        await _eventBus.PublishAsync(new OrderCreatedEvent { OrderId = order.Id });
    }
}

public class InventoryService // Listens for OrderCreatedEvent
{
    public async Task Handle(OrderCreatedEvent @event)
    {
        try
        {
            await ReserveInventory(@event.OrderId, @event.Items);
            await _eventBus.PublishAsync(new InventoryReservedEvent { OrderId = @event.OrderId });
        }
        catch
        {
            // Compensating transaction - reverse the order
            await _eventBus.PublishAsync(new InventoryReservationFailedEvent { OrderId = @event.OrderId });
        }
    }
}
```

**Evening (1 hour): Circuit Breaker Pattern**

**MUST KNOW FOR INTERVIEWS**

```
CIRCUIT BREAKER = Prevent cascading failures

States:
1. CLOSED (normal) - requests pass through
2. OPEN (failure detected) - requests blocked immediately (fail fast)
3. HALF-OPEN (recovery test) - allow few requests to test service recovery

Implementation with Polly (NuGet):
```

```csharp
var circuitBreakerPolicy = Policy
    .Handle<HttpRequestException>()
    .OrResult<HttpResponseMessage>(r => !r.IsSuccessStatusCode)
    .CircuitBreakerAsync(
        handledEventsAllowedBeforeBreaking: 3,
        durationOfBreak: TimeSpan.FromSeconds(10),
        onBreak: (outcome, duration) =>
        {
            Console.WriteLine($"Circuit breaker opened for {duration}");
        },
        onReset: () =>
        {
            Console.WriteLine("Circuit breaker reset");
        }
    );

var response = await circuitBreakerPolicy.ExecuteAsync(
    () => httpClient.GetAsync("https://api.example.com")
);
```

**When to use:**
- Calls to external services
- Database calls
- Any I/O operation that can fail

**Day 3 Deliverable:** Be able to explain:
- "Why is each service having its own database important?"
- "Explain SAGA pattern with a real example"
- "When would you use Circuit Breaker?"

---

### **Day 4: Thursday - Key Patterns & Consistency (4-5 hours)**

#### **Morning (1.5 hours): Eventual Consistency**

**The Paradigm Shift:**
```
TRADITIONAL (Monolith):
✓ Strong consistency immediately
"Write order → Read order → You get latest data immediately"

MICROSERVICES:
✓ Eventual consistency
"Write order to Service A → Propagates to Service B → After few seconds B has data"

Why? Network, independent databases, asynchronous communication
```

**Example:**
```
Time 0:00 - Customer places order in OrderService
Time 0:01 - Event published: OrderCreated
Time 0:02 - InventoryService receives event, updates stock
Time 0:03 - AnalyticsService receives event, updates dashboard

If customer queries inventory at 0:01, they see OLD data
This is acceptable in most business scenarios
```

**Afternoon (2 hours): Critical Patterns Summary**

**Pattern 1: Strangler Fig (Migration)**
```
How to migrate monolith to microservices:

Phase 1: Proxy layer in front of monolith
Phase 2: Extract one microservice (e.g., Auth)
Phase 3: Route AuthService requests through proxy
Phase 4: Old code still works, new code routes to microservice
Phase 5: Gradually extract more services

Advantage: Zero downtime, gradual migration
```

**Pattern 2: CQRS (Command Query Responsibility Segregation)**
```
CQRS = Separate read and write models

Simple: One service, one database
        All writes and reads on same DB

CQRS: 
  - Write model (normalized) for commands
  - Read model (denormalized) for queries
  - Eventual consistency between them
  
Example:
  POST /orders (write) → OrderService (normalized DB)
  GET /orders (read) → ReportService (denormalized cache/DB)
  
  Event: OrderCreated
  ↓ Updates read model asynchronously

Use when: High read volume, complex queries, read/write patterns differ
```

**Pattern 3: BFF (Backend for Frontend)**
```
BFF = API optimized for specific client type

Instead of:
Client → General API → Multiple services

BFF approach:
Mobile Client → Mobile BFF → Multiple services
Web Client → Web BFF → Multiple services
Desktop → Desktop BFF → Multiple services

Each BFF optimized for its client:
- Mobile BFF: Lightweight, minimal data
- Web BFF: Full features, rich data
- Desktop BFF: Advanced features
```

**Evening (1 hour): Service Discovery**

**The Problem:**
```
Microservices run on different servers/containers
IP addresses are dynamic (containers restart, get new IPs)
How does Service A find Service B?
```

**Solution 1: Client-Side Discovery**
```
Service Registry: Central place storing all services + locations

Client-side:
OrderService needs ProductService
1. Query Service Registry: "Where is ProductService?"
2. Registry returns: "ProductService at 192.168.1.5:5001"
3. OrderService calls directly

Tool: Consul, Eureka
```

**Solution 2: Server-Side Discovery**
```
Load Balancer maintains registry

OrderService needs ProductService
1. OrderService calls Load Balancer: "I need ProductService"
2. Load Balancer finds ProductService instance
3. Load Balancer forwards request

Tool: Kubernetes Service Discovery, AWS ELB
```

**In Azure/Kubernetes:**
```
Kubernetes handles service discovery automatically!
Services can reference each other by name:
http://product-service:80/api/products
(Kubernetes DNS resolves the name)
```

**Day 4 Deliverable:** Be able to explain:
- "What is eventual consistency?"
- "When would you use CQRS?"
- "How does service discovery work?"

---

### **Day 5: Friday - Security, Monitoring & Azure (5 hours)**

#### **Morning (2 hours): Security Patterns**

**Layer 1: API Gateway Authentication**
```
All requests go through API Gateway
API Gateway validates token/credentials
Only authenticated requests reach microservices

Tool: OAuth2/OpenID Connect

Flow:
1. Client sends credentials to API Gateway
2. API Gateway calls Identity Service
3. Identity Service returns JWT token
4. Client includes JWT in Authorization header
5. API Gateway validates JWT
6. Request passes to microservice
7. Microservice can optionally validate JWT again
```

**Layer 2: Inter-Service Security**
```
How does ServiceA call ServiceB securely?

Option 1: Mutual TLS (mTLS)
- Both services have certificates
- Services verify each other's certificates
- Encrypted communication

Option 2: Service-to-Service JWT
- ServiceA calls Identity Service with client credentials
- Gets JWT token specific for ServiceB
- Includes token in call to ServiceB
- ServiceB validates token

Option 3: API Gateway forwards auth context
- Client authenticated at API Gateway
- API Gateway adds user context header
- Microservices trust API Gateway headers
```

**Quick Implementation Overview:**
```csharp
// Add JWT authentication
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.Authority = "https://identityserver";
        options.Audience = "api1";
    });

// Protect endpoint
[Authorize]
[HttpGet("orders/{id}")]
public async Task<IActionResult> GetOrder(int id)
{
    // Only authenticated users can call this
}
```

**Afternoon (2.5 hours): Monitoring, Logging & Tracing**

**The Challenge:**
```
Monolith: One log file, one trace
Microservices: Logs scattered across 10+ services!

How do you trace a request through 5 microservices?
```

**Solution 1: Correlation ID (MUST KNOW)**
```
CORRELATION ID = Unique ID attached to entire request chain

Flow:
1. Client makes request
2. API Gateway generates Correlation ID: "ABC-123-XYZ"
3. API Gateway passes to OrderService with header
4. OrderService logs with "ABC-123-XYZ"
5. OrderService calls InventoryService, passes same ID
6. InventoryService logs with "ABC-123-XYZ"
7. You can grep all logs for "ABC-123-XYZ" to see entire flow!

Implementation:
Header: X-Correlation-ID: ABC-123-XYZ
```

**Solution 2: Distributed Tracing (Jaeger, Application Insights)**
```
Tools: Application Insights (Azure), Jaeger, Zipkin

Shows:
- Full request flow across services
- Timing at each step
- Failures and errors
- Performance bottlenecks
```

**Solution 3: Structured Logging (Serilog)**
```
Not: log.Info("Order created")
Yes: log.Information("Order created. OrderId: {OrderId}, UserId: {UserId}, Amount: {Amount}", 
                      orderId, userId, amount)

Output includes: Timestamp, Level, OrderId, UserId, Amount
Easy to search and correlate
```

**Key Metrics to Monitor:**
```
1. Request Rate - Requests per second
2. Error Rate - % of failed requests
3. Latency - Response time (p50, p95, p99)
4. Service Health - % availability
5. Resource Usage - CPU, Memory, Disk per service
6. Business Metrics - Orders/hour, Revenue/day
```

**Evening (0.5 hours): Azure Microservices Services**

**MUST KNOW FOR AZURE INTERVIEWS**

```
Azure Service Bus
├─ Asynchronous messaging
├─ Topics & Subscriptions
├─ Dead-letter queues
└─ Use for: Event publishing, async workflows

Azure Kubernetes Service (AKS)
├─ Container orchestration
├─ Auto-scaling
├─ Service discovery (built-in)
└─ Use for: Running containerized microservices

Azure API Management
├─ Single entry point
├─ Rate limiting, throttling
├─ API versioning
├─ Use for: Managing public APIs

Azure Application Insights
├─ Distributed tracing
├─ Performance monitoring
├─ Log aggregation
└─ Use for: Monitoring and observability

Azure Key Vault
├─ Secrets management
├─ Connection strings storage
└─ Use for: Secure configuration

Azure Container Registry (ACR)
├─ Private Docker registry
└─ Use for: Storing container images

Entity Framework Core + SQL Azure
├─ Database per service
└─ Use for: Data persistence
```

**Day 5 Deliverable:** Be able to explain:
- "How would you authenticate across microservices?"
- "What is a Correlation ID?"
- "Which Azure services would you use for microservices?"

---

### **Day 6: Saturday - Case Study & Design Exercise (5 hours)**

#### **Complete Project: Design an E-Commerce Microservices System**

**Requirements:**
```
1. Customers browse products
2. Add to cart
3. Checkout (order)
4. Pay
5. Get notifications
6. View order history
```

**Your Task - Design Answer:**

**Services to Create:**
```
1. API Gateway (Ocelot/Azure API Management)
   - Single entry point
   - Authentication validation
   - Route to appropriate service

2. Identity Service
   - User registration/login
   - JWT token generation
   - OAuth2 endpoint

3. Product Service
   - Get products
   - Filter, search
   - Database: SQL Server
   - Cache: Redis

4. Cart Service
   - Add/remove items
   - Validate products via Product Service (REST)
   - Store in Redis or Database per user

5. Order Service
   - Create order
   - Track order status
   - Database: SQL Server (separate from Product)
   - Publish OrderCreated event

6. Payment Service
   - Process payments
   - Integrate with payment gateway
   - Publish PaymentProcessed event
   - Compensating transaction: Refund

7. Inventory Service
   - Reserve stock
   - Update stock
   - Database: SQL Server
   - Subscribe to OrderCreated (Saga pattern)
   - If reservation fails, publish event to rollback

8. Notification Service
   - Send emails/SMS
   - Subscribe to OrderCreated, PaymentProcessed
   - No database needed (just sends notifications)

9. Reporting Service (Optional)
   - Read-only replica of data (CQRS)
   - Reports, analytics
   - Denormalized database
```

**Communication Flow:**

```
1. BROWSING PRODUCTS
   Browser → API Gateway → Product Service (REST)
   Response: Product list

2. LOGIN
   Browser → API Gateway → Identity Service (REST)
   Response: JWT token

3. ADD TO CART
   Browser (includes JWT) → API Gateway → Cart Service (REST)
   Cart Service: GET http://product-service/api/products/{id} (validate)
   Store in Redis

4. CHECKOUT
   Browser → API Gateway → Order Service (REST)
   Order Service:
   a) Create Order in DB
   b) Publish OrderCreated event (async)

5. SAGA EXECUTION (Async)
   OrderCreated event → Message Broker (RabbitMQ/Service Bus)
   
   InventoryService receives:
   - Try to reserve stock
   - If success: Publish InventoryReserved event
   - If fail: Publish InventoryReservationFailed event
   
   PaymentService receives InventoryReserved:
   - Try to process payment
   - If success: Publish PaymentProcessed event
   - If fail: Publish PaymentFailed event (triggers compensation)
   
   NotificationService receives all events:
   - Publish PaymentProcessed → Send order confirmation email
   - Publish PaymentFailed → Send failure notification

6. FAILURE SCENARIO
   If payment fails:
   - Publish PaymentFailed event
   - InventoryService receives: Release reserved stock
   - OrderService receives: Mark order as Failed
   - NotificationService: Send failure email
```

**Deployment Architecture:**

```
Azure Deployment:

┌─────────────────────────────────────┐
│         Azure Kubernetes (AKS)      │
├─────────────────────────────────────┤
│ ┌─────────────────────────────────┐ │
│ │ Containers (Docker)             │ │
│ ├─────────────────────────────────┤ │
│ │ API Gateway (Ocelot)            │ │
│ │ Identity Service                │ │
│ │ Product Service                 │ │
│ │ Order Service                   │ │
│ │ Payment Service                 │ │
│ │ Inventory Service               │ │
│ │ Notification Service            │ │
│ │ Reporting Service               │ │
│ └─────────────────────────────────┘ │
└─────────────────────────────────────┘
        ↓
┌─────────────────────────────────────┐
│    Backing Services                 │
├─────────────────────────────────────┤
│ Azure SQL Database (databases)      │
│ Azure Cache for Redis               │
│ Azure Service Bus (messaging)       │
│ Azure Key Vault (secrets)           │
│ Azure Application Insights          │
│ Azure Container Registry (images)   │
└─────────────────────────────────────┘
```

**Data Storage:**

```
Product Service:
  Database: Azure SQL (shared read-only access)
  Cache: Redis
  
Order Service:
  Database: Azure SQL (separate instance)
  
Cart Service:
  Database: Redis (session data)
  
Inventory Service:
  Database: Azure SQL (separate instance)
  
Reporting Service:
  Database: Azure SQL (denormalized copy)
```

**Security:**

```
1. All requests through API Gateway
2. JWT token validation at Gateway
3. Correlation ID for tracing
4. Secrets in Key Vault
5. mTLS between services (if high security needed)
6. Rate limiting at Gateway
```

**Monitoring:**

```
Application Insights:
- Distributed tracing
- Request/response logging
- Performance metrics
- Error tracking

Correlation IDs:
- X-Correlation-ID header
- Passed through all service calls
- Visible in logs for end-to-end tracing
```

**Write this out on paper or document - Practice explaining it**

**Day 6 Deliverable:** Document for E-Commerce system:
1. Service breakdown (what each does)
2. Data flow diagram
3. Saga workflow for order placement
4. Failure scenarios and compensation
5. Deployment on Azure

---

### **Day 7: Sunday - Mock Interview Practice (3-4 hours)**

#### **Part 1: Self-Interview (1.5 hours)**

Record yourself answering these 10 questions (2-3 min each):

1. "Design a microservices system for an online banking application. How would you structure it?"
2. "How would you handle payment failure in a distributed transaction?"
3. "A microservice is experiencing 50% failure rate. What's your approach?"
4. "Explain how you'd implement authentication across 10 microservices"
5. "Draw the architecture of your e-commerce system and explain each component"
6. "How would you migrate a monolithic 10-year-old system to microservices?"
7. "What are the top 3 biggest challenges you'd face building microservices?"
8. "How would you monitor and debug a slow request across 5 microservices?"
9. "When should you NOT use microservices?"
10. "Explain eventual consistency and give a real example"

**Record on your phone, watch playback, improve delivery**

#### **Part 2: Real Practice (1.5-2 hours)**

**Use these resources:**
- Grab a colleague or friend, have them ask questions
- Use chatGPT to generate follow-up questions
- Practice on Pramp.com (free mock interviews)

**Focus on:**
- Speaking clearly (not mumbling)
- Structure your answer (Problem → Solution → Trade-offs)
- Give concrete examples
- Ask clarifying questions first
- Draw diagrams when explaining

#### **Part 3: Quick Review of Tier 1 Questions (1 hour)**

Go through all 15 Tier 1 questions again:
- Answer each in 1-2 minutes
- Write bullet points if needed
- Focus on clarity and confidence

**Day 7 Deliverable:** Be confident answering any Tier 1 question

---

## Quick Reference Card (Study Guide for Last Minute)

Print this and carry it:

```
========== MICROSERVICES QUICK REFERENCE ==========

WHEN TO USE:
✓ Large teams (6+)
✓ Need independent deployment
✓ Different parts scale differently
✗ Team < 6 people
✗ Co-located team, rare deployments

COMMUNICATION:
Sync (REST): Immediate response needed
Async (Events): Notifications, background work

PATTERNS TO KNOW:
1. Circuit Breaker - Fail fast, prevent cascading failures
2. Saga - Distributed transactions with compensation
3. API Gateway - Single entry point
4. Service Discovery - Find services dynamically
5. Database per Service - Own your data
6. Eventual Consistency - Accept delay for availability
7. Strangler Fig - Migrate monolith gradually
8. BFF - Optimize API for client type
9. CQRS - Separate read/write models

RESILIENCE:
- Polly: Circuit breaker, retry
- TimeoutPolicy: Set timeouts
- BulkheadPolicy: Isolate failures
- FallbackPolicy: Graceful degradation

DATA CONSISTENCY:
No 2PC!
Use Saga: Choreography (event-driven) or Orchestration (workflow)
Accept eventual consistency

SECURITY:
- OAuth2/OIDC at API Gateway
- JWT tokens
- Correlation IDs for tracing
- Secrets in Key Vault
- Optional: mTLS between services

MONITORING:
- Correlation ID in every request
- Structured logging (Serilog)
- Distributed tracing (Application Insights)
- Health checks on every service
- Alert on: Error rate, latency, availability

AZURE SERVICES:
- AKS: Run containers
- Service Bus: Async messaging
- API Management: Gateway
- Application Insights: Monitoring
- Key Vault: Secrets
- Container Registry: Store images

FAILURE SCENARIO RESPONSE:
1. Identify: Use correlation ID
2. Isolate: Circuit breaker stops cascading
3. Compensate: Saga triggers rollback
4. Log: Structured logging for debugging
5. Monitor: Alert on patterns
```

---

## Interview Day Tips

### **Before Interview:**
- Get 8 hours sleep night before
- Review this guide one more time
- Take a practice test on a whiteboard
- Test your audio/video if remote

### **During Interview:**
- **Listen carefully** - Understand the question fully
- **Ask clarifying questions first** - "Is this for startup or enterprise?"
- **Structure your answer:**
  ```
  1. Restate problem
  2. Propose solution
  3. List trade-offs
  4. Give example
  5. Ask: "Would you like me to elaborate?"
  ```
- **Draw diagrams** - Visual > verbal
- **Be honest about gaps** - "I haven't worked with X, but I understand it's..."
- **Connect to your experience** - Reference 16 years of development
- **Show thought process** - Don't just recite

### **Example Answer Format:**

**Q: "Design a microservices system for uber?"**

```
A: Great question. Let me clarify first:
   - Are we talking ride-matching, payment, or entire system?
   - Expected scale? (1 city vs worldwide)
   
   Assuming worldwide, multi-feature system:
   
   SERVICES I'D CREATE:
   1. User Service (authentication, profiles)
   2. Ride Matching Service (algorithm, matching)
   3. Driver Location Service (real-time GPS)
   4. Payment Service (transactions)
   5. Notification Service (SMS/push)
   6. Pricing Service (surge pricing)
   
   COMMUNICATION:
   - REST for immediate needs (request ride - need immediate response)
   - Async events for notifications (payment confirmed → send SMS)
   
   DATA:
   - Each service owns database
   - High-frequency data (location) → NoSQL (Redis)
   - Transactional data (payments) → SQL (with Saga for consistency)
   
   DEPLOYMENT:
   - Containerized on Kubernetes (AKS if Azure)
   - Auto-scaling based on demand
   - API Gateway for routing
   
   KEY CHALLENGES:
   - Real-time location updates (might use WebSockets)
   - Consistency in payment (Saga pattern with compensation)
   - Scale under surge demand (separate scaling per service)
   
   TRADEOFF: Complexity increases vs benefits of independent scaling
   
   Does this align with what you're looking for?
```

---

## Final Checklist (Before Interview)

- [ ] Understand 15 Tier 1 questions deeply
- [ ] Can draw simple architecture diagram
- [ ] Know SAGA pattern with example
- [ ] Can explain Circuit Breaker in 2 minutes
- [ ] Know when to use microservices
- [ ] Understand eventual consistency
- [ ] Can list top Azure services for microservices
- [ ] Have one complete design exercise memorized (e-commerce)
- [ ] Practice speaking answers aloud
- [ ] Understand when NOT to use microservices

---

## Last-Minute Cramming (If you have only a few hours)

Priority order:
1. **Tier 1 Questions** (must know these cold)
2. **E-commerce design exercise** (write it down, memorize flow)
3. **SAGA pattern** (choreography vs orchestration with example)
4. **Circuit Breaker** (when and why)
5. **API Gateway** (simple diagram)

**You've got this!** 

Your 16 years of experience means you understand distributed systems thinking. Microservices are just applying old patterns to modern problems. Focus on the concepts, not the tools.

Good luck! 🚀

--- 
# Microservices Interview Q&A with Code Examples & Diagrams

---

## Q1: What are Microservices? How do they differ from Monolithic Architecture?

### Answer:

**Definition:**
Microservices is an architectural approach where an application is built as a collection of small, independent services that communicate over well-defined APIs. Each service runs in its own process, owns its data, and can be deployed independently.

**Key Differences:**

```
MONOLITHIC ARCHITECTURE:
┌─────────────────────────────────┐
│    Single Application           │
├─────────────────────────────────┤
│  ├─ User Management             │
│  ├─ Product Catalog             │
│  ├─ Order Processing            │
│  └─ Notification System         │
├─────────────────────────────────┤
│    Shared Database              │
└─────────────────────────────────┘

MICROSERVICES ARCHITECTURE:
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│  User Svc    │  │ Product Svc  │  │  Order Svc   │
├──────────────┤  ├──────────────┤  ├──────────────┤
│  User DB     │  │  Product DB  │  │  Order DB    │
└──────────────┘  └──────────────┘  └──────────────┘
       ↑                ↑                   ↑
       └────────────────┴───────────────────┘
         (APIs / Events / Messages)
```

**C# Code Example:**

```csharp
// MONOLITHIC APPROACH
public class OrderService
{
    public async Task<Order> CreateOrder(CreateOrderRequest request)
    {
        // Everything in one service
        var user = await _userRepository.GetUser(request.UserId);
        var products = await _productRepository.GetProducts(request.ProductIds);
        var order = new Order { UserId = user.Id, Items = products };
        await _orderRepository.Save(order);
        await SendNotification(user.Email, "Order created");
        return order;
    }
}

// MICROSERVICES APPROACH
// Service 1: Order Service
public class OrderService
{
    private readonly IOrderRepository _orderRepository;
    private readonly IEventPublisher _eventBus;
    
    public async Task<Order> CreateOrder(CreateOrderRequest request)
    {
        var order = new Order { UserId = request.UserId, Items = request.Items };
        await _orderRepository.Save(order);
        
        // Publish event - other services handle rest
        await _eventBus.PublishAsync(new OrderCreatedEvent 
        { 
            OrderId = order.Id, 
            UserId = request.UserId 
        });
        return order;
    }
}

// Service 2: Notification Service (listens for OrderCreatedEvent)
public class NotificationConsumer : IConsumer<OrderCreatedEvent>
{
    public async Task Consume(ConsumeContext<OrderCreatedEvent> context)
    {
        var email = await _userService.GetUserEmail(context.Message.UserId);
        await _emailSender.SendAsync(email, "Order confirmation");
    }
}
```

**Key Differences Table:**

| Aspect | Monolithic | Microservices |
|--------|-----------|---------------|
| Deployment | All or nothing | Independent |
| Database | Shared | Per service |
| Scaling | Scale entire app | Scale by service |
| Technology | Single tech stack | Polyglot possible |
| Team Structure | One team | Multiple teams |
| Complexity | Low initially | High initially |

---

## Q2: When Should You Use Microservices? When NOT to Use?

### Answer:

**Decision Matrix:**

```
USE MICROSERVICES WHEN:
✓ Team size > 6 people (teams can work independently)
✓ Need frequent, independent deployments
✓ Different parts need different scalability
✓ Organization is mature with DevOps practices
✓ Distributed team across locations

DO NOT USE MICROSERVICES WHEN:
✗ Team size < 6 (overhead > benefit)
✗ Startup phase (focus on MVP)
✗ Latency-critical system (network overhead)
✗ Simple CRUD application
✗ No DevOps/Container infrastructure
```

**Decision Tree Diagram:**

```
START: Should we use microservices?
  │
  ├─→ Is team size < 6?
  │   └─→ NO → Continue
  │   └─→ YES → ❌ Use Monolith
  │
  ├─→ Do we need frequent independent deployments?
  │   └─→ NO → ❌ Use Monolith
  │   └─→ YES → Continue
  │
  ├─→ Do different components scale differently?
  │   └─→ NO → ❌ Use Monolith
  │   └─→ YES → Continue
  │
  ├─→ Is this latency-critical (< 50ms)?
  │   └─→ YES → ❌ Use Monolith (network adds latency)
  │   └─→ NO → Continue
  │
  ├─→ Do we have DevOps/Container infrastructure?
  │   └─→ NO → ❌ Invest in infrastructure first
  │   └─→ YES → ✅ Use Microservices
```

**C# Code: When to Choose What**

```csharp
public class ArchitectureDecision
{
    public static string DecideArchitecture(ProjectMetrics metrics)
    {
        // Early-stage startup
        if (metrics.IsStartup && metrics.TeamSize < 5)
            return "MonolithicArchitecture";
        
        // Mature company, independent scaling needs
        if (metrics.TeamSize >= 6 && metrics.HasDevOps && 
            metrics.NeedsIndependentScaling && metrics.DeploymentFrequency > 1)
            return "MicroservicesArchitecture";
        
        // Real-time latency-critical (stock trading, gaming)
        if (metrics.MaxLatencyMs < 50)
            return "MonolithicArchitecture"; // Or extreme optimization needed
        
        // Simple CRUD app
        if (metrics.Complexity == Complexity.Low)
            return "MonolithicArchitecture";
        
        return "Modular Monolith"; // Middle ground
    }
}

// Example metrics
var metrics = new ProjectMetrics
{
    TeamSize = 12,
    IsStartup = false,
    HasDevOps = true,
    NeedsIndependentScaling = true,
    DeploymentFrequency = 5, // 5 times per week
    Complexity = Complexity.High,
    MaxLatencyMs = 200
};
```

---

## Q3: Explain Synchronous vs Asynchronous Communication

### Answer:

**Diagram:**

```
SYNCHRONOUS (REST/HTTP):
RequestService          ResponseService
    │                        │
    ├──── HTTP Request ─────→ │
    │     (block here)        │
    │                    (process)
    │ ←──── HTTP Response ──┤
    │   (get response)       │
    ✓ (continue)             │

Caller blocks until response arrives
Example: Get user details, validate product stock


ASYNCHRONOUS (Messaging):
PublisherService    MessageBroker    SubscriberService
    │                    │                  │
    ├── Publish Event ──→ │                  │
    │ (no wait)          │                  │
    ✓ (continue)         │                  │
                         ├─ Store Message ──│
                         │                  │
                         │     (Subscriber processes)
                         │     ←─ Acknowledge ──
                         ✓ (message deleted)

Publisher doesn't wait for response
Example: Send order confirmation email, update analytics
```

**C# Code Example:**

```csharp
// SYNCHRONOUS - REST API Call
public class OrderService
{
    private readonly HttpClient _httpClient;
    
    public async Task<Order> CreateOrderSync(CreateOrderRequest request)
    {
        var order = new Order { UserId = request.UserId };
        await _repository.SaveAsync(order);
        
        // BLOCK HERE - Wait for inventory service response
        var inventoryResponse = await _httpClient.GetAsync(
            $"https://inventory-service/api/check-stock?productId={request.ProductId}"
        );
        
        if (!inventoryResponse.IsSuccessStatusCode)
            throw new Exception("Stock not available"); // Immediate response
        
        return order;
    }
}

// ASYNCHRONOUS - Event-Driven
public class OrderServiceAsync
{
    private readonly IEventBus _eventBus;
    
    public async Task<Order> CreateOrderAsync(CreateOrderRequest request)
    {
        var order = new Order { UserId = request.UserId };
        await _repository.SaveAsync(order);
        
        // FIRE AND FORGET - Don't wait for response
        await _eventBus.PublishAsync(new OrderCreatedEvent 
        { 
            OrderId = order.Id, 
            ProductId = request.ProductId 
        });
        
        return order; // Return immediately
    }
}

// Subscriber processes asynchronously
public class InventoryConsumer : IConsumer<OrderCreatedEvent>
{
    public async Task Consume(ConsumeContext<OrderCreatedEvent> context)
    {
        // Process in background, no caller waiting
        await ReserveInventoryAsync(context.Message.ProductId);
    }
}
```

**When to Use What:**

| Scenario | Use | Reason |
|----------|-----|--------|
| Get user profile | Sync (REST) | Need immediate response |
| Validate stock before order | Sync | Need immediate feedback |
| Send confirmation email | Async | Can wait, no blocker |
| Update analytics | Async | Fire and forget |
| Display product details | Sync | User waiting |
| Log audit trail | Async | Background task |

---

## Q4: What is the Circuit Breaker Pattern?

### Answer:

**Concept:**
Circuit Breaker prevents cascading failures by stopping requests to a failing service. Similar to electrical circuit breaker that cuts power when overload detected.

**Three States:**

```
                    ┌─────────────────────┐
                    │    CIRCUIT STATES   │
                    └─────────────────────┘
                             │
         ┌───────────────────┼───────────────────┐
         │                   │                   │
    ┌─────────┐         ┌──────────┐       ┌─────────┐
    │ CLOSED  │         │ HALF-OPEN│       │  OPEN   │
    ├─────────┤         ├──────────┤       ├─────────┤
    │Normal op│         │Testing   │       │Blocking │
    │Requests │         │recovery  │       │requests │
    │allowed  │         │attempts  │       │Fail fast│
    └────┬────┘         └────┬─────┘       └────┬────┘
         │                   │                   │
         │ Failure           │ Failure           │ Timeout
         │ threshold         │ in test           │ expires
         │ reached           │ requests          │
         └──→ OPEN ←─────────────              │
                                              │
                                 Recover? Yes │
                                              ↓
                                         HALF-OPEN
                                              ↑
                                              │
                                      Success?
                                       Resume
                                       Normal
```

**C# Code with Polly:**

```csharp
public class ResilientHttpClientService
{
    private readonly HttpClient _httpClient;
    private readonly IAsyncPolicy<HttpResponseMessage> _circuitBreakerPolicy;
    
    public ResilientHttpClientService(HttpClient httpClient)
    {
        _httpClient = httpClient;
        
        // Configure Circuit Breaker
        _circuitBreakerPolicy = Policy
            .Handle<HttpRequestException>()
            .OrResult<HttpResponseMessage>(r => !r.IsSuccessStatusCode)
            .CircuitBreakerAsync<HttpResponseMessage>(
                handledEventsAllowedBeforeBreaking: 3, // 3 failures before opening
                durationOfBreak: TimeSpan.FromSeconds(10), // Keep open for 10 sec
                onBreak: (outcome, duration) =>
                {
                    Console.WriteLine($"Circuit breaker OPENED for {duration}");
                },
                onReset: () =>
                {
                    Console.WriteLine("Circuit breaker CLOSED - service recovered");
                }
            );
    }
    
    public async Task<string> GetProductDetailsAsync(int productId)
    {
        // Request goes through circuit breaker
        var response = await _circuitBreakerPolicy.ExecuteAsync(
            () => _httpClient.GetAsync($"https://product-service/api/products/{productId}")
        );
        
        return await response.Content.ReadAsStringAsync();
    }
}

// Usage
var service = new ResilientHttpClientService(httpClient);
try
{
    var details = await service.GetProductDetailsAsync(123);
}
catch (BrokenCircuitException)
{
    Console.WriteLine("Service temporarily unavailable - try later");
}
```

**State Transitions:**

```
STATE: CLOSED (Normal Operation)
Request → Service responds ✓ → Continue in CLOSED
Request → Service fails ✗ → Increment failure counter
Failure counter = 3? → OPEN

STATE: OPEN (Failing)
Request → Immediately fail (don't call service) ✓ Fail Fast!
Timeout expires (10 sec)? → HALF-OPEN

STATE: HALF-OPEN (Testing Recovery)
Request → Call service (test if recovered) 
Success ✓ → CLOSED (resume normal)
Failure ✗ → OPEN (still failing, wait more)
```

---

## Q5: Explain the SAGA Pattern for Distributed Transactions

### Answer:

**Problem:**
Traditional transactions don't work across microservices (no 2-Phase Commit). How do you maintain data consistency across services?

**Solution: SAGA Pattern**

```
SCENARIO: Create Order
1. Reserve Inventory
2. Process Payment  
3. Create Shipment

SAGA = Series of local transactions with compensating transactions

HAPPY PATH (All succeed):
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│  Order Svc   │      │ Inventory Svc│      │ Payment Svc  │
├──────────────┤      ├──────────────┤      ├──────────────┤
│ Create Order │      │ Reserve Stock│      │Process Payment│
│     ✓        │ ────→│      ✓       │ ────→│      ✓       │
│ (Pending)    │      │ (Reserved)   │      │ (Authorized) │
└──────────────┘      └──────────────┘      └──────────────┘

FAILURE PATH (Payment fails):
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│  Order Svc   │      │ Inventory Svc│      │ Payment Svc  │
├──────────────┤      ├──────────────┤      ├──────────────┤
│ Create Order │      │ Reserve Stock│      │Process Payment│
│     ✓        │ ────→│      ✓       │ ────→│      ✗       │
│              │      │              │      │   FAILED     │
│              │ ←────│ Release Stock│      │              │
│              │      │ (Compensate) │      │              │
│ Cancel Order │      │              │      │              │
└──────────────┘      └──────────────┘      └──────────────┘
(Compensate)         (Compensating Txn)     (No rollback needed)
```

**Two Implementation Approaches:**

### **Approach 1: Choreography (Event-Driven)**

```csharp
// Each service publishes events, others listen
public class OrderChoreography
{
    private readonly IEventBus _eventBus;
    
    public async Task CreateOrder(CreateOrderRequest request)
    {
        var order = new Order { UserId = request.UserId };
        await _repository.SaveAsync(order);
        
        // Publish event
        await _eventBus.PublishAsync(new OrderCreatedEvent 
        { 
            OrderId = order.Id, 
            Items = request.Items 
        });
    }
}

// Inventory Service listens
public class InventoryChoreography : IConsumer<OrderCreatedEvent>
{
    public async Task Consume(ConsumeContext<OrderCreatedEvent> context)
    {
        try
        {
            // Reserve inventory
            await ReserveInventoryAsync(context.Message.Items);
            
            // Success - publish next event
            await context.Publish(new InventoryReservedEvent 
            { 
                OrderId = context.Message.OrderId 
            });
        }
        catch
        {
            // Failure - publish compensation event
            await context.Publish(new OrderCancelledEvent 
            { 
                OrderId = context.Message.OrderId, 
                Reason = "Inventory unavailable" 
            });
        }
    }
}

// Payment Service listens
public class PaymentChoreography : IConsumer<InventoryReservedEvent>
{
    public async Task Consume(ConsumeContext<InventoryReservedEvent> context)
    {
        try
        {
            // Process payment
            await ProcessPaymentAsync(context.Message.OrderId);
            
            // Success
            await context.Publish(new PaymentProcessedEvent 
            { 
                OrderId = context.Message.OrderId 
            });
        }
        catch
        {
            // Failure - trigger compensation
            await context.Publish(new PaymentFailedEvent 
            { 
                OrderId = context.Message.OrderId 
            });
        }
    }
}

// Compensation: Release reserved inventory
public class CompensationHandler : IConsumer<PaymentFailedEvent>
{
    public async Task Consume(ConsumeContext<PaymentFailedEvent> context)
    {
        // Release reserved inventory
        await ReleaseInventoryAsync(context.Message.OrderId);
    }
}
```

### **Approach 2: Orchestration (Workflow)**

```csharp
// Central orchestrator controls the saga
public class OrderOrchestrator
{
    private readonly IOrderService _orderService;
    private readonly IInventoryService _inventoryService;
    private readonly IPaymentService _paymentService;
    
    public async Task ExecuteOrderSaga(CreateOrderRequest request)
    {
        var order = new Order { UserId = request.UserId };
        
        try
        {
            // Step 1: Create order
            await _orderService.CreateAsync(order);
            
            // Step 2: Reserve inventory
            await _inventoryService.ReserveAsync(request.Items);
            
            // Step 3: Process payment
            await _paymentService.ProcessAsync(order.Id);
            
            // Success
            await _orderService.MarkCompleteAsync(order.Id);
        }
        catch (Exception ex)
        {
            // Compensation - rollback in reverse order
            await _paymentService.RefundAsync(order.Id); // If needed
            await _inventoryService.ReleaseAsync(request.Items);
            await _orderService.CancelAsync(order.Id);
            
            throw;
        }
    }
}
```

**Choreography vs Orchestration:**

| Aspect | Choreography | Orchestration |
|--------|-------------|---------------|
| Control | Distributed | Centralized |
| Coupling | Loose | Tighter |
| Testing | Complex | Easier |
| Understanding Flow | Hard (events everywhere) | Easy (clear workflow) |
| Performance | Fast (parallel possible) | Can be sequential |

---

## Q6: What is the API Gateway Pattern?

### Answer:

**Concept:**
API Gateway is a single entry point for all client requests. It routes requests to appropriate microservices and handles cross-cutting concerns.

**Architecture:**

```
WITHOUT API GATEWAY (Problematic):
┌─────────────────────────────────┐
│         Clients                 │
│ (Web, Mobile, Desktop)          │
└────┬────────────────┬───────────┘
     │                │
     │                ├─────────────────┐
     │                │                 │
     ↓                ↓                 ↓
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│ Product Svc  │ │  Order Svc   │ │  User Svc    │
└──────────────┘ └──────────────┘ └──────────────┘

Problems:
- Clients know all service URLs
- Duplicate auth logic in each service
- No rate limiting at entry point
- Hard to add cross-cutting concerns

WITH API GATEWAY (Best Practice):
┌─────────────────────────────────┐
│         Clients                 │
└────────────────┬────────────────┘
                 │
                 ↓
         ┌──────────────────┐
         │  API Gateway     │
         │ (Single Entry)   │
         ├──────────────────┤
         │ - Authentication │
         │ - Rate Limiting  │
         │ - Logging        │
         │ - Version Mgmt   │
         └────────┬─────────┘
                  │
        ┌─────────┼──────────┐
        │         │          │
        ↓         ↓          ↓
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│ Product Svc  │ │  Order Svc   │ │  User Svc    │
└──────────────┘ └──────────────┘ └──────────────┘

Benefits:
- Single entry point
- Centralized auth
- Rate limiting
- Easy versioning
```

**C# Code with Ocelot:**

```csharp
// ocelot.json configuration
{
  "Routes": [
    {
      "DownstreamPathTemplate": "/api/products/{everything}",
      "DownstreamScheme": "http",
      "DownstreamHostAndPorts": [
        { "Host": "product-service", "Port": 5001 }
      ],
      "UpstreamPathTemplate": "/api/products/{everything}",
      "UpstreamHttpMethod": [ "Get", "Post" ],
      "AuthenticationOptions": {
        "AuthenticationProviderKey": "Bearer",
        "AllowedScopes": []
      }
    }
  ],
  "GlobalConfiguration": {
    "BaseUrl": "https://api.example.com"
  }
}

// Startup configuration
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddOcelot(_configuration)
            .AddConsul() // Service discovery
            .AddPolly(); // Resilience
        
        // Add authentication
        services.AddAuthentication("Bearer")
            .AddJwtBearer("Bearer", options =>
            {
                options.Authority = "https://identity-server";
                options.Audience = "api";
            });
    }
    
    public async void Configure(IApplicationBuilder app)
    {
        await app.UseOcelot();
    }
}
```

**API Gateway Responsibilities:**

```csharp
public class ApiGatewayMiddleware
{
    public async Task Invoke(HttpContext context)
    {
        // 1. Authentication
        if (!User.Identity.IsAuthenticated)
            return Unauthorized();
        
        // 2. Rate Limiting
        if (!await _rateLimiter.AllowRequestAsync(User.Id))
            return TooManyRequests();
        
        // 3. Logging / Correlation ID
        context.Request.Headers.Add("X-Correlation-ID", 
            Guid.NewGuid().ToString());
        
        // 4. Request Transformation
        
        // 5. Routing
        
        // 6. Response Transformation
        
        // 7. Caching
    }
}
```

---

## Q7: Explain Database Per Service Pattern

### Answer:

**Concept:**
Each microservice has its own database. No shared databases between services.

**Diagram:**

```
ANTI-PATTERN (Shared Database):
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ Product Svc  │  │  Order Svc   │  │ Inventory Svc│
└──────┬───────┘  └──────┬───────┘  └──────┬───────┘
       │                 │                 │
       └─────────────────┼─────────────────┘
                         │
                    ┌────▼────┐
                    │  Shared  │
                    │ Database │
                    └──────────┘

Problems:
✗ Tight coupling
✗ Can't scale independently
✗ Database becomes bottleneck
✗ Hard to change database technology

BEST PRACTICE (Database Per Service):
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ Product Svc  │  │  Order Svc   │  │ Inventory Svc│
└──────┬───────┘  └──────┬───────┘  └──────┬───────┘
       │                 │                 │
       ↓                 ↓                 ↓
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│ SQL Server   │ │ SQL Server   │ │   Redis      │
│ (Products)   │ │ (Orders)     │ │ (Inventory)  │
└──────────────┘ └──────────────┘ └──────────────┘

Benefits:
✓ Independent scaling
✓ Technology flexibility
✓ Loose coupling
✓ Independent deployment
```

**C# Implementation:**

```csharp
// Product Service - SQL Server
public class ProductService
{
    private readonly ProductDbContext _productDb;
    
    public async Task<Product> GetProductAsync(int id)
    {
        return await _productDb.Products
            .FirstOrDefaultAsync(p => p.Id == id);
    }
}

// Order Service - SQL Server (Different DB)
public class OrderService
{
    private readonly OrderDbContext _orderDb; // Different DB context!
    
    public async Task<Order> CreateOrderAsync(CreateOrderRequest request)
    {
        // Order service doesn't directly access product database
        // It calls Product Service API instead
        var productDetails = await _productService.GetProductAsync(
            request.ProductId);
        
        var order = new Order 
        { 
            UserId = request.UserId, 
            ProductId = request.ProductId 
        };
        
        await _orderDb.Orders.AddAsync(order);
        await _orderDb.SaveChangesAsync();
        return order;
    }
}

// Inventory Service - Redis (Different Technology)
public class InventoryService
{
    private readonly IDistributedCache _cache; // Redis cache
    
    public async Task<int> GetStockAsync(int productId)
    {
        var stock = await _cache.GetStringAsync($"stock:{productId}");
        return int.Parse(stock ?? "0");
    }
}

// Startup - Multiple DB contexts
public void ConfigureServices(IServiceCollection services)
{
    // Product DB
    services.AddDbContext<ProductDbContext>(
        options => options.UseSqlServer(
            "Server=prod-server;Database=ProductDb"));
    
    // Order DB
    services.AddDbContext<OrderDbContext>(
        options => options.UseSqlServer(
            "Server=order-server;Database=OrderDb"));
    
    // Inventory Cache
    services.AddStackExchangeRedisCache(options =>
    {
        options.Configuration = "inventory-redis:6379";
    });
}
```

**Handling Data Access Across Services:**

```csharp
// Get data from other service via API
public class OrderService
{
    private readonly HttpClient _httpClient;
    
    public async Task<Order> CreateOrderWithProductDetailsAsync(
        CreateOrderRequest request)
    {
        // Call Product Service API (NOT direct DB access)
        var product = await _httpClient.GetFromJsonAsync<Product>(
            $"https://product-service/api/products/{request.ProductId}");
        
        if (product == null)
            throw new Exception("Product not found");
        
        // Create order in ORDER database (not product database)
        var order = new Order 
        { 
            UserId = request.UserId, 
            ProductId = product.Id,
            Price = product.Price 
        };
        
        await _orderDb.SaveAsync(order);
        return order;
    }
}
```

---

## Q8: What is Service Discovery?

### Answer:

**Problem:**
In containerized environments, services run on dynamic IP addresses. How does ServiceA find ServiceB?

**Diagram:**

```
STATIC WORLD (No Service Discovery):
OrderService knows: "ProductService is at 192.168.1.5:5001"
But containers restart → New IP: 192.168.1.10:5001
Hard-coded URL breaks!

DYNAMIC WORLD (With Service Discovery):
┌─────────────────────────────────────────┐
│      Service Registry                   │
│  ┌────────────────────────────────────┐ │
│  │ Service Name    →  IP:Port         │ │
│  │ product-service → 192.168.1.5:5001│ │
│  │ order-service   → 192.168.1.6:5002│ │
│  │ user-service    → 192.168.1.7:5003│ │
│  └────────────────────────────────────┘ │
└──────────────────┬──────────────────────┘
                   │
        ┌──────────┼──────────┐
        │          │          │
    ┌───▼──┐   ┌──▼───┐  ┌──▼───┐
    │Order │   │Prod  │  │User  │
    │Svc   │   │Svc   │  │Svc   │
    └──────┘   └──────┘  └──────┘
```

**Two Approaches:**

### **Approach 1: Client-Side Discovery**

```csharp
// Client queries registry directly
public class OrderService
{
    private readonly IServiceDiscovery _serviceDiscovery;
    private readonly HttpClient _httpClient;
    
    public async Task<Product> GetProductAsync(int productId)
    {
        // Step 1: Discover ProductService location
        var productServiceUrl = await _serviceDiscovery
            .ResolveAsync("product-service");
        
        // Step 2: Call the discovered URL
        var response = await _httpClient.GetAsync(
            $"{productServiceUrl}/api/products/{productId}");
        
        return await response.Content.ReadAsAsync<Product>();
    }
}

// Service Discovery implementation (Consul)
public class ConsulServiceDiscovery : IServiceDiscovery
{
    private readonly IConsulClient _consul;
    
    public async Task<string> ResolveAsync(string serviceName)
    {
        // Query Consul for service location
        var services = await _consul.Catalog.Service(serviceName);
        
        if (!services.Response.Any())
            throw new Exception($"Service {serviceName} not found");
        
        // Pick first instance (or load balance)
        var service = services.Response.First();
        
        return $"http://{service.ServiceAddress}:{service.ServicePort}";
    }
}
```

### **Approach 2: Server-Side Discovery (Kubernetes)**

```csharp
// Kubernetes DNS handles discovery automatically!
public class OrderService
{
    private readonly HttpClient _httpClient;
    
    public async Task<Product> GetProductAsync(int productId)
    {
        // Kubernetes automatically resolves this DNS name
        // No need to query registry!
        var response = await _httpClient.GetAsync(
            "http://product-service:80/api/products/{productId}");
        
        return await response.Content.ReadAsAsync<Product>();
    }
}

// Kubernetes Service automatically discovers endpoints:
/*
apiVersion: v1
kind: Service
metadata:
  name: product-service
spec:
  selector:
    app: product-service
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
*/
```

**Service Registration:**

```csharp
// Services register themselves with registry on startup
public class ServiceStartup
{
    public void RegisterServiceWithConsul(IApplicationBuilder app,
        IHostApplicationLifetime lifetime,
        IConsulClient consul)
    {
        var serviceId = Guid.NewGuid().ToString();
        var registration = new AgentServiceRegistration()
        {
            ID = serviceId,
            Name = "order-service",
            Address = "192.168.1.6",
            Port = 5002,
            Check = new AgentServiceCheck()
            {
                HTTP = "http://192.168.1.6:5002/health",
                Interval = TimeSpan.FromSeconds(10)
            }
        };
        
        consul.Agent.ServiceRegister(registration);
        
        // Deregister on shutdown
        lifetime.ApplicationStopping.Register(() =>
        {
            consul.Agent.ServiceDeregister(serviceId);
        });
    }
}
```

---

## Q9: How do you Handle Eventual Consistency?

### Answer:

**Concept:**
In microservices, data across services is eventually consistent (not immediately consistent like traditional databases).

**Timeline Diagram:**

```
TRADITIONAL (Immediate Consistency):
Time:     0ms    1ms    2ms    3ms
         │ │     │ │    │ │    │ │
Write ──→ DB updated
         └─────────────────────────
Read  ──→                     Get latest data ✓

MICROSERVICES (Eventual Consistency):
Time:     0ms    100ms  200ms  300ms  400ms
         │ │     │ │    │ │    │ │    │ │
Write ──→ Service A (Order created)
         └─────────────────────────────────
Event ───→ Message Broker
          └─────────────────────────────────
Service B ──────→ (Still processing)
          ├──────┤ (has old data)
                 │
Service C ──────────→ (Still processing)
          ├──────────┤
                     │
Service D ──────────────→ (Finally received)
          ├───────────────┤ (now consistent)
                          └──→ All consistent at 300ms+
```

**C# Code Example:**

```csharp
// Order Service: Create order and publish event
public class OrderService
{
    private readonly IOrderRepository _orderRepo;
    private readonly IEventBus _eventBus;
    
    public async Task<Order> CreateOrderAsync(CreateOrderRequest request)
    {
        var order = new Order { UserId = request.UserId, Status = "Pending" };
        await _orderRepo.SaveAsync(order); // Saved to ORDER DB
        
        // Publish event for other services
        await _eventBus.PublishAsync(new OrderCreatedEvent 
        { 
            OrderId = order.Id, 
            UserId = request.UserId 
        });
        
        // WARNING: Other services don't have this data yet!
        return order;
    }
}

// Analytics Service: Eventually receives data
public class AnalyticsService : IConsumer<OrderCreatedEvent>
{
    private readonly IAnalyticsRepository _analyticsRepo;
    
    public async Task Consume(ConsumeContext<OrderCreatedEvent> context)
    {
        // Delay of 100-500ms common
        var orderData = new AnalyticsRecord 
        { 
            OrderId = context.Message.OrderId,
            CreatedAt = DateTime.UtcNow 
        };
        
        await _analyticsRepo.SaveAsync(orderData);
    }
}

// Notification Service: Independently receives
public class NotificationService : IConsumer<OrderCreatedEvent>
{
    private readonly INotificationRepository _notifRepo;
    
    public async Task Consume(ConsumeContext<OrderCreatedEvent> context)
    {
        // May arrive at different time than Analytics
        var notification = new Notification 
        { 
            OrderId = context.Message.OrderId,
            Type = "OrderConfirmation" 
        };
        
        await _notifRepo.SaveAsync(notification);
    }
}

// IMPORTANT: Design for eventual consistency
public class ConsistencyStrategy
{
    // DON'T do this:
    public async Task<Order> GetOrderWithAnalyticsWrong(int orderId)
    {
        var order = await _orderService.GetAsync(orderId);
        var analytics = await _analyticsService.GetAsync(orderId);
        
        // Analytics might not exist yet! Exception likely!
        return order; // Broken
    }
    
    // DO this:
    public async Task<Order> GetOrderWithAnalyticsCorrect(int orderId)
    {
        var order = await _orderService.GetAsync(orderId);
        
        // Analytics may or may not be available - handle gracefully
        var analytics = await _analyticsService.GetAsync(orderId);
        
        if (analytics == null)
        {
            // Data not available yet, retry later or use default
            return order;
        }
        
        return order;
    }
}
```

**Handling Eventual Consistency:**

```csharp
// Strategy 1: Polling (Retry logic)
public async Task<Order> GetOrderWithRetryAsync(int orderId)
{
    var retries = 0;
    const int maxRetries = 5;
    
    while (retries < maxRetries)
    {
        try
        {
            var order = await _orderService.GetAsync(orderId);
            var analytics = await _analyticsService.GetAsync(orderId);
            
            if (analytics != null)
                return order;
            
            await Task.Delay(1000 * (retries + 1)); // Exponential backoff
            retries++;
        }
        catch
        {
            retries++;
        }
    }
    
    throw new Exception("Data consistency timeout");
}

// Strategy 2: Make it transparent to UI
public async Task<OrderViewModel> GetOrderViewAsync(int orderId)
{
    var order = await _orderService.GetAsync(orderId);
    
    var viewModel = new OrderViewModel 
    { 
        OrderId = order.Id,
        Status = order.Status,
        // Show "Loading..." for analytics if not ready
        AnalyticsLoaded = false,
        Analytics = null
    };
    
    // Load analytics in background (eventually)
    _ = Task.Run(async () =>
    {
        await Task.Delay(500); // Account for delay
        var analytics = await _analyticsService.GetAsync(orderId);
        if (analytics != null)
            viewModel.Analytics = analytics;
    });
    
    return viewModel;
}
```

---

## Q10: Explain Choreography vs Orchestration in Saga Pattern

### Answer:

**Choreography (Event-Driven):**

```
ORDER SAGA - CHOREOGRAPHY FLOW:

OrderService creates order
       │
       └──→ Publish: OrderCreated
              │
              └──→ Message Broker stores
                     │
                     ├─→ InventoryService listens
                     │   ├─ Reserve stock
                     │   ├─ Publish: InventoryReserved
                     │   │
                     │   └─→ Message Broker
                     │
                     ├─→ PaymentService listens
                     │   ├─ Process payment
                     │   ├─ Publish: PaymentProcessed
                     │   │
                     │   └─→ Message Broker
                     │
                     └─→ NotificationService listens
                         ├─ Send confirmation email
                         └─ Done

VISUAL SEQUENCE:
OrderService ──→ InventoryService ──→ PaymentService ──→ Notification
(Create Order) (Reserve Stock)    (Process Payment)   (Send Email)
      ├─────────────────────────────────────────────────────────│
      │ (Chain of events, each triggers next)                   │
      └────────────────────────────────────────────────────────→

Each service listens for events and publishes its own events
No central orchestrator
```

**Orchestration (Workflow Engine):**

```
ORDER SAGA - ORCHESTRATION FLOW:

OrderOrchestrator (Central Controller)
      │
      ├─→ Step 1: Call OrderService.CreateOrder()
      │   └─ Wait for response
      │
      ├─→ Step 2: Call InventoryService.Reserve()
      │   └─ Wait for response
      │
      ├─→ Step 3: Call PaymentService.Process()
      │   └─ Wait for response
      │
      └─→ If any fails: Rollback in reverse order

VISUAL SEQUENCE:
            ┌─────────────────────┐
            │  OrderOrchestrator  │
            │ (Central Control)   │
            └──────────┬──────────┘
                       │
       ┌───────────────┼───────────────┐
       │               │               │
       ↓               ↓               ↓
  OrderService  InventoryService  PaymentService
  (Passive)     (Passive)         (Passive)

Orchestrator calls each service synchronously
Each service just executes, doesn't coordinate
Orchestrator handles flow logic
```

**C# Code Comparison:**

```csharp
// ============ CHOREOGRAPHY ============
public class OrderChoreography
{
    private readonly IEventBus _eventBus;
    
    public async Task CreateOrder(CreateOrderRequest request)
    {
        var order = new Order { UserId = request.UserId };
        await _repository.SaveAsync(order);
        
        // Just publish - don't care who listens
        await _eventBus.PublishAsync(new OrderCreatedEvent 
        { 
            OrderId = order.Id 
        });
    }
}

// Multiple subscribers (choreography)
public class InventoryConsumer : IConsumer<OrderCreatedEvent>
{
    public async Task Consume(ConsumeContext<OrderCreatedEvent> context)
    {
        // Reserve inventory
        await ReserveAsync(context.Message.OrderId);
        
        // Publish next event
        await context.Publish(new InventoryReservedEvent 
        { 
            OrderId = context.Message.OrderId 
        });
    }
}

public class PaymentConsumer : IConsumer<InventoryReservedEvent>
{
    public async Task Consume(ConsumeContext<InventoryReservedEvent> context)
    {
        // Process payment
        await ProcessPaymentAsync(context.Message.OrderId);
        
        // Publish next event
        await context.Publish(new PaymentProcessedEvent 
        { 
            OrderId = context.Message.OrderId 
        });
    }
}

// ============ ORCHESTRATION ============
public class OrderOrchestration
{
    private readonly IOrderService _orderService;
    private readonly IInventoryService _inventoryService;
    private readonly IPaymentService _paymentService;
    
    public async Task ExecuteOrderSaga(CreateOrderRequest request)
    {
        try
        {
            // Step 1: Create order
            var order = await _orderService.CreateAsync(request);
            
            // Step 2: Reserve inventory (orchestrator calls directly)
            await _inventoryService.ReserveAsync(order.Id);
            
            // Step 3: Process payment (orchestrator calls directly)
            await _paymentService.ProcessAsync(order.Id);
            
            // All success
            await _orderService.MarkCompleteAsync(order.Id);
        }
        catch (Exception ex)
        {
            // Orchestrator handles compensation
            await _paymentService.RefundAsync(request.OrderId);
            await _inventoryService.ReleaseAsync(request.OrderId);
            await _orderService.CancelAsync(request.OrderId);
            
            throw;
        }
    }
}
```

**Comparison Table:**

| Aspect | Choreography | Orchestration |
|--------|-------------|---------------|
| Central Controller | No | Yes |
| Coupling | Loose | Tighter |
| Visibility | Hard (events scattered) | Easy (clear flow) |
| Testing | Complex | Simpler |
| Performance | Can parallelize | Sequential |
| Debugging | Difficult | Easier |
| Scalability | Better | Bottleneck at orchestrator |
| Implementation | Event-driven (MassTransit) | Workflow (Saga pattern) |

---

## Q11: What is the BFF (Backend for Frontend) Pattern?

### Answer:

**Problem:**
Different clients (Web, Mobile, Desktop) have different needs. One API for all = inefficient.

**Architecture:**

```
WITHOUT BFF (One-Size-Fits-All):
┌──────────────────────────────────┐
│  Web | Mobile | Desktop | TV     │ (All clients)
└──────────┬───────────────────────┘
           │
           ↓
    ┌─────────────────────┐
    │  General Purpose    │
    │  API                │
    │ Returns full data   │
    │ (Inefficient for    │
    │  mobile!)           │
    └────────┬────────────┘
             │
    ┌────────┴────────────┐
    │ Microservices       │

WITH BFF (Optimized per client):
┌─────────────┐  ┌──────────────┐  ┌──────────────┐
│ Web Client  │  │Mobile Client │  │Desktop Client│
└──────┬──────┘  └──────┬───────┘  └──────┬───────┘
       │                │                │
       ↓                ↓                ↓
┌─────────────────────────────────────────────────┐
│         BFF Layer (Optimized APIs)              │
├──────────────┬──────────────┬──────────────┐    │
│ Web BFF      │ Mobile BFF   │Desktop BFF   │    │
│ Full data    │ Minimal data │Rich features │    │
│ Rich UI      │ Fast, lite   │Advanced      │    │
└──────┬───────┴──────┬───────┴──────┬──────┘    │
       │              │              │           │
       └──────────────┴──────────────┴────────┐   │
                                    │        │
                          ┌─────────▼────────▼──┐
                          │  Microservices     │
                          │ (Core business     │
                          │  logic)            │
                          └────────────────────┘
```

**C# Implementation:**

```csharp
// ============ WEB BFF ============
[ApiController]
[Route("api/web")]
public class WebBffController
{
    private readonly IProductService _productService;
    private readonly IOrderService _orderService;
    
    [HttpGet("products")]
    public async Task<IEnumerable<ProductWebDto>> GetProducts()
    {
        var products = await _productService.GetAllAsync();
        
        // Web gets full details for rich UI
        return products.Select(p => new ProductWebDto
        {
            Id = p.Id,
            Name = p.Name,
            Description = p.Description, // Full description for web
            Price = p.Price,
            ImageUrl = p.ImageUrl,
            DetailedSpecs = p.Specifications, // Rich content
            Reviews = p.Reviews,
            RelatedProducts = p.Related
        });
    }
}

// ============ MOBILE BFF ============
[ApiController]
[Route("api/mobile")]
public class MobileBffController
{
    private readonly IProductService _productService;
    
    [HttpGet("products")]
    public async Task<IEnumerable<ProductMobileDto>> GetProducts()
    {
        var products = await _productService.GetAllAsync();
        
        // Mobile gets minimal data (bandwidth conscious)
        return products.Select(p => new ProductMobileDto
        {
            Id = p.Id,
            Name = p.Name,
            Price = p.Price,
            ThumbnailUrl = p.ThumbnailUrl // Compressed image only
            // No description, specs, reviews - too much data!
        });
    }
}

// ============ DTOs ============
public class ProductWebDto
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    public decimal Price { get; set; }
    public string ImageUrl { get; set; }
    public List<string> DetailedSpecs { get; set; }
    public List<Review> Reviews { get; set; }
    public List<Product> RelatedProducts { get; set; }
}

public class ProductMobileDto
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public string ThumbnailUrl { get; set; }
}
```

**Benefits:**
- Optimized for each client's needs
- Reduced bandwidth for mobile
- Different authentication per client
- Easy to A/B test different UIs
- Independent scaling per BFF

---

## Q12: How do you Monitor Distributed Microservices?

### Answer:

**Challenge:**
With 10+ services, logs are scattered. How do you trace a request?

**Solution: Correlation ID + Distributed Tracing**

```
REQUEST FLOW WITH CORRELATION ID:

┌─────────────────────────────────┐
│ Client Request                  │
│ GET /orders                     │
└────────────────┬────────────────┘
                 │
                 ↓
    ┌────────────────────────────┐
    │ API Gateway                │
    │ Generate: X-Correlation-ID │
    │ ABC-123-XYZ                │
    └───────┬───────┬────────────┘
            │       │
            │       └──→ [Log 1] Received request ABC-123-XYZ
            │
            ↓
    ┌────────────────────────────┐
    │ Order Service              │
    │ Receives: ABC-123-XYZ      │
    └───────┬───────┬────────────┘
            │       │
            │       └──→ [Log 2] Creating order ABC-123-XYZ
            │
            ↓ (calls Product Service)
    ┌────────────────────────────┐
    │ Product Service            │
    │ Receives: ABC-123-XYZ      │
    └───────┬───────┬────────────┘
            │       │
            │       └──→ [Log 3] Validating product ABC-123-XYZ
            │
            ↓ (response)
    ┌────────────────────────────┐
    │ Order Service              │
    │ Correlating: ABC-123-XYZ   │
    └───────┬───────┬────────────┘
            │       │
            │       └──→ [Log 4] Order created ABC-123-XYZ
            │
            ↓
    ┌────────────────────────────┐
    │ API Gateway                │
    │ Returns response           │
    └───────────────────────────┘

GREP ALL LOGS:
grep "ABC-123-XYZ" *.log
[Log 1] Received request ABC-123-XYZ
[Log 2] Creating order ABC-123-XYZ
[Log 3] Validating product ABC-123-XYZ
[Log 4] Order created ABC-123-XYZ
→ Full request trace in one place!
```

**C# Implementation:**

```csharp
// Step 1: API Gateway generates Correlation ID
public class CorrelationIdMiddleware
{
    private readonly RequestDelegate _next;
    
    public CorrelationIdMiddleware(RequestDelegate next)
    {
        _next = next;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        var correlationId = context.Request.Headers
            .FirstOrDefault(h => h.Key == "X-Correlation-ID")
            .Value.FirstOrDefault() ?? Guid.NewGuid().ToString();
        
        context.Response.Headers.Add("X-Correlation-ID", correlationId);
        
        using (LogContext.PushProperty("CorrelationId", correlationId))
        {
            // Pass to next middleware
            await _next(context);
        }
    }
}

// Step 2: Serilog structured logging
public void ConfigureLogging(ILoggingBuilder logging)
{
    Log.Logger = new LoggerConfiguration()
        .Enrich.FromLogContext() // Includes CorrelationId
        .WriteTo.Console(
            outputTemplate: 
            "[{Timestamp:yyyy-MM-dd HH:mm:ss}] {Level:u3} {CorrelationId} {Message:lj}{NewLine}")
        .CreateLogger();
    
    logging.AddSerilog();
}

// Step 3: Services include Correlation ID in calls
public class OrderService
{
    private readonly HttpClient _httpClient;
    private readonly ILogger<OrderService> _logger;
    
    public async Task<Order> CreateOrderAsync(CreateOrderRequest request)
    {
        _logger.LogInformation("Creating order for user {UserId}", request.UserId);
        
        var order = new Order { UserId = request.UserId };
        await _repository.SaveAsync(order);
        
        // Include Correlation ID in outbound call
        var requestMessage = new HttpRequestMessage(HttpMethod.Get,
            $"https://product-service/api/products/{request.ProductId}");
        
        requestMessage.Headers.Add("X-Correlation-ID",
            context.Request.Headers["X-Correlation-ID"]);
        
        var response = await _httpClient.SendAsync(requestMessage);
        
        _logger.LogInformation("Order created: {OrderId}", order.Id);
        return order;
    }
}

// Step 4: Use Application Insights for visualization
public void ConfigureServices(IServiceCollection services)
{
    services.AddApplicationInsightsTelemetry();
    
    // Distributed tracing automatically enabled
}
```

**Log Output:**
```
[2024-01-15 10:30:45] INF ABC-123-XYZ Creating order for user 42
[2024-01-15 10:30:46] INF ABC-123-XYZ Calling product service
[2024-01-15 10:30:47] INF ABC-123-XYZ Product validated
[2024-01-15 10:30:48] INF ABC-123-XYZ Order created: 789

All logs tied together with ABC-123-XYZ!
```

---

## Q13: How do you Implement Security in Microservices?

### Answer:

**Architecture:**

```
SECURITY LAYERS:

┌─────────────────────────────────────┐
│  Clients (Web, Mobile, API)         │
└──────────────────┬──────────────────┘
                   │
                   ↓ (HTTPS only)
        ┌──────────────────────┐
        │ API Gateway          │
        │ - Validate JWT       │
        │ - Rate limiting      │
        │ - TLS termination    │
        └─────┬────────────────┘
              │
        ┌─────▼────────────────┐
        │ Identity Service     │
        │ (OAuth2/OpenID)      │
        │ - Issue JWT tokens   │
        │ - Validate creds     │
        └──────────────────────┘
              │
        ┌─────▼────────────────────────┐
        │ Microservices                │
        │ (Behind API Gateway)         │
        │ - Service-to-service auth    │
        │ - Optional JWT validation    │
        └──────────────────────────────┘
              │
        ┌─────▼────────────────┐
        │ Azure Key Vault      │
        │ - Secrets storage    │
        │ - Connection strings │
        └──────────────────────┘
```

**C# Implementation:**

```csharp
// ============ CLIENT AUTHENTICATION ============
public class AuthController : ControllerBase
{
    private readonly IIdentityService _identityService;
    
    [HttpPost("login")]
    public async Task<IActionResult> Login(LoginRequest request)
    {
        var user = await _identityService.AuthenticateAsync(
            request.Email, request.Password);
        
        if (user == null)
            return Unauthorized();
        
        // Generate JWT token
        var token = _identityService.GenerateToken(user);
        
        return Ok(new { Token = token }); // Client stores this
    }
}

// ============ API GATEWAY VALIDATION ============
public class AuthenticationMiddleware
{
    private readonly RequestDelegate _next;
    
    public AuthenticationMiddleware(RequestDelegate next)
    {
        _next = next;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        var token = context.Request.Headers["Authorization"]
            .FirstOrDefault()?.Split(" ").Last();
        
        if (token != null)
        {
            try
            {
                // Validate JWT
                var principal = _jwtHandler.ValidateToken(token);
                context.User = principal;
                context.Items["User"] = principal.Identity.Name;
            }
            catch
            {
                return context.Response.WriteAsync("Invalid token");
            }
        }
        
        await _next(context);
    }
}

// ============ PROTECTED ENDPOINT ============
[Authorize]
[ApiController]
[Route("api/[controller]")]
public class OrderController : ControllerBase
{
    [HttpPost]
    public async Task<IActionResult> CreateOrder(CreateOrderRequest request)
    {
        var userId = User.FindFirst("sub").Value; // From JWT token
        
        var order = new Order 
        { 
            UserId = int.Parse(userId), 
            Items = request.Items 
        };
        
        await _repository.SaveAsync(order);
        
        return CreatedAtAction(nameof(GetOrder), 
            new { id = order.Id }, order);
    }
}

// ============ SERVICE-TO-SERVICE COMMUNICATION ============
public class ServiceAuthClient
{
    private readonly HttpClient _httpClient;
    private readonly IIdentityService _identityService;
    
    public async Task<T> GetAsync<T>(string serviceName, string endpoint)
    {
        // Service authenticates using client credentials
        var serviceToken = await _identityService
            .GetServiceTokenAsync("order-service", "secret123");
        
        var request = new HttpRequestMessage(HttpMethod.Get, endpoint);
        request.Headers.Authorization = 
            new AuthenticationHeaderValue("Bearer", serviceToken);
        
        var response = await _httpClient.SendAsync(request);
        return await response.Content.ReadAsAsync<T>();
    }
}

// ============ SECRETS MANAGEMENT ============
public void ConfigureServices(IServiceCollection services)
{
    // Load secrets from Key Vault
    var keyVaultUrl = "https://myvault.vault.azure.net/";
    var credential = new ClientSecretCredential(
        tenantId, clientId, clientSecret);
    var client = new SecretClient(new Uri(keyVaultUrl), credential);
    
    services.AddSingleton(client);
    
    // Use secrets
    var connectionString = client.GetSecret("db-connection-string").Value.Value;
    services.AddDbContext<OrderContext>(options =>
        options.UseSqlServer(connectionString)
    );
}
```

---

## Q14: Design an E-Commerce Microservices System

### Answer:

**Requirements:**
- Browse products
- Add to cart
- Checkout & pay
- Track orders
- Notifications

**Architecture Diagram:**

```
┌────────────────────────────────────────────────────────────┐
│                    Client Layer                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   Web App    │  │  Mobile App  │  │ Desktop App  │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
└─────────────────────┬──────────────────────────────────────┘
                      │ (HTTPS)
                      ↓
         ┌────────────────────────────┐
         │   API Gateway              │
         │ (Ocelot/Azure API Mgmt)    │
         │ - Auth validation          │
         │ - Rate limiting            │
         │ - Logging                  │
         └────────────────────────────┘
                 │      │      │      │
    ┌────────────┴──┬───┴──────┴──┬──┴────────────────┐
    │               │              │                  │
    ↓               ↓              ↓                  ↓
┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│ Identity │  │ Product  │  │  Order   │  │ Inventory│
│ Service  │  │ Service  │  │ Service  │  │ Service  │
├──────────┤  ├──────────┤  ├──────────┤  ├──────────┤
│JWT/OAuth2│  │ Catalog  │  │ Orders   │  │ Stock    │
│          │  │ Search   │  │ Tracking │  │ Mgmt     │
└──────────┘  └──────────┘  └──────────┘  └──────────┘
    │              │              │              │
    │         ┌────┴──────┐       │         ┌────┴──────┐
    │         │  Caching  │       │         │  Events   │
    │         │  (Redis)  │       │         │  (Bus)    │
    │         └───────────┘       │         └───────────┘
    │                             │
    └─────────────────────────────┼─────────────────────────┐
                                  │                         │
                    ┌─────────────┴──────────────┐           │
                    │  Payment Service           │           │
                    │  - Process payments        │           │
                    │  - Stripe/PayPal           │           │
                    └────────────────────────────┘           │
                                                             │
                    ┌────────────────────────────┐           │
                    │  Notification Service      │ ←─────────┘
                    │  - Email                   │  (Events)
                    │  - SMS                     │
                    │  - Push notifications      │
                    └────────────────────────────┘

┌───────────────────────────────────────────────────────────┐
│              Backing Services                             │
├───────────────────────────────────────────────────────────┤
│ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐     │
│ │ SQL DB   │ │  SQL DB  │ │ SQL DB   │ │ SQL DB   │     │
│ │(Identity)│ │(Product) │ │ (Orders) │ │(Inventory)    │
│ └──────────┘ └──────────┘ └──────────┘ └──────────┘     │
│                                                           │
│ ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│ │ Service Bus  │  │ App Insights │  │  Key Vault   │    │
│ │ (Messaging)  │  │ (Monitoring) │  │ (Secrets)    │    │
│ └──────────────┘  └──────────────┘  └──────────────┘    │
└───────────────────────────────────────────────────────────┘
```

**Data Flow: Order Creation**

```
1. USER BROWSING:
   Client → API Gateway → Product Service
   Product Service queries Redis cache
   Returns product list

2. USER ADDS TO CART:
   Client → API Gateway → Cart Service
   Cart Service stores in Redis (per session)
   No persistence needed yet

3. USER CHECKOUT:
   Client → API Gateway → Order Service
   
   ┌─ Order Service creates order in DB
   │
   └─→ Publishes: OrderCreatedEvent
       │
       ├─→ InventoryService receives
       │   └─ Tries to reserve stock
       │      ├─ Success → InventoryReservedEvent
       │      └─ Fail → InventoryFailedEvent
       │
       ├─→ PaymentService receives InventoryReservedEvent
       │   └─ Tries to process payment
       │      ├─ Success → PaymentProcessedEvent
       │      └─ Fail → PaymentFailedEvent
       │
       └─→ NotificationService receives
           ├─ PaymentProcessedEvent → Send order confirmation email
           └─ PaymentFailedEvent → Send payment failed email

4. TRACKING:
   Client → API Gateway → Order Service
   Returns order status with tracking info
```

**C# Code Skeleton:**

```csharp
// ========== ORCHESTRATION (Main Flow) ==========
public class OrderSaga
{
    private readonly IOrderService _orderService;
    private readonly IInventoryService _inventoryService;
    private readonly IPaymentService _paymentService;
    
    public async Task ExecuteAsync(CreateOrderRequest request)
    {
        var order = await _orderService.CreateAsync(
            new Order { UserId = request.UserId });
        
        try
        {
            // Reserve inventory
            await _inventoryService.ReserveAsync(
                order.Id, request.Items);
            
            // Process payment
            await _paymentService.ProcessAsync(
                order.Id, request.Amount);
            
            // Mark complete
            await _orderService.MarkCompleteAsync(order.Id);
        }
        catch
        {
            // Compensation
            await _paymentService.RefundAsync(order.Id);
            await _inventoryService.ReleaseAsync(order.Id);
            await _orderService.CancelAsync(order.Id);
            throw;
        }
    }
}

// ========== NOTIFICATION SAGA ==========
public class NotificationSaga
{
    private readonly INotificationService _notificationService;
    
    [Subscribe]
    public async Task OnOrderCreated(OrderCreatedEvent evt)
    {
        await _notificationService.SendEmailAsync(
            evt.UserEmail, 
            "Order confirmation", 
            $"Your order {evt.OrderId} has been received");
    }
    
    [Subscribe]
    public async Task OnPaymentProcessed(PaymentProcessedEvent evt)
    {
        await _notificationService.SendEmailAsync(
            evt.UserEmail, 
            "Payment successful", 
            $"Payment for order {evt.OrderId} processed");
    }
}

// ========== DEPLOYMENT ==========
// Kubernetes manifests handle deployment:
// - Each service as separate pod
// - Horizontal scaling per service
// - Rolling updates
// - Health checks
```

---

## Q15: How do you Handle Service Failures and Cascading Failures?

### Answer:

**Problem:**
If ProductService goes down, OrderService can't process orders. How do you prevent cascading failures?

```
CASCADE DIAGRAM:

SCENARIO: ProductService fails
    │
    ├─ OrderService calls ProductService
    │   └─ No response (timeout)
    │
    ├─ OrderService waits for response
    │   └─ Accumulates threads/connections
    │
    ├─ OrderService thread pool exhausted
    │   └─ Can't process new requests
    │
    ├─ API Gateway can't route requests
    │   └─ Returns errors to clients
    │
    └─ System cascades (many services affected)

SOLUTION LAYERS:

Layer 1: Circuit Breaker (Fail Fast)
    ├─ Detects ProductService is down
    ├─ Opens circuit
    └─ Returns error immediately (no waiting)

Layer 2: Timeout
    ├─ Don't wait indefinitely
    └─ Return error after N seconds

Layer 3: Bulkhead (Isolation)
    ├─ Separate thread pools per service
    └─ One service down ≠ system down

Layer 4: Fallback
    ├─ Use cached/default data
    └─ Graceful degradation

Layer 5: Retry with Backoff
    ├─ Retry with exponential backoff
    └─ Don't overwhelm recovering service
```

**C# Implementation:**

```csharp
public class ResilientServiceCalls
{
    private readonly HttpClient _httpClient;
    private readonly IAsyncPolicy<HttpResponseMessage> _policy;
    
    public ResilientServiceCalls(HttpClient httpClient)
    {
        _httpClient = httpClient;
        
        // LAYER 1 & 2: Circuit Breaker + Timeout
        var circuitBreakerPolicy = Policy
            .Handle<HttpRequestException>()
            .OrResult<HttpResponseMessage>(r => !r.IsSuccessStatusCode)
            .CircuitBreakerAsync<HttpResponseMessage>(
                handledEventsAllowedBeforeBreaking: 3,
                durationOfBreak: TimeSpan.FromSeconds(10)
            );
        
        // LAYER 5: Retry with exponential backoff
        var retryPolicy = Policy
            .Handle<HttpRequestException>()
            .OrResult<HttpResponseMessage>(r => !r.IsSuccessStatusCode)
            .WaitAndRetryAsync<HttpResponseMessage>(
                retryCount: 3,
                sleepDurationProvider: attempt =>
                    TimeSpan.FromSeconds(Math.Pow(2, attempt))
            );
        
        // LAYER 2: Timeout
        var timeoutPolicy = Policy
            .TimeoutAsync<HttpResponseMessage>(
                TimeSpan.FromSeconds(5)
            );
        
        // Combine policies
        _policy = Policy.WrapAsync<HttpResponseMessage>(
            circuitBreakerPolicy,
            retryPolicy,
            timeoutPolicy
        );
    }
    
    // LAYER 4: Fallback
    public async Task<Product> GetProductWithFallbackAsync(int productId)
    {
        try
        {
            var response = await _policy.ExecuteAsync(
                () => _httpClient.GetAsync(
                    $"https://product-service/api/products/{productId}")
            );
            
            return await response.Content.ReadAsAsync<Product>();
        }
        catch (BrokenCircuitException)
        {
            // Service is down, use fallback
            Console.WriteLine("ProductService unavailable, using cached data");
            
            return new Product
            {
                Id = productId,
                Name = "Product Details Unavailable",
                Price = 0,
                IsFromCache = true
            };
        }
    }
    
    // LAYER 3: Bulkhead (Separate thread pools)
    public void ConfigureBulkhead(IServiceCollection services)
    {
        // Separate thread pool for ProductService
        var productServicePolicy = Policy.BulkheadAsync(
            maxParallelization: 10, // Max 10 parallel calls
            maxQueuingActions: 20    // Max 20 queued
        );
        
        // Separate thread pool for PaymentService
        var paymentServicePolicy = Policy.BulkheadAsync(
            maxParallelization: 5,
            maxQueuingActions: 10
        );
        
        // If ProductService exhausts its threads,
        // PaymentService still works!
    }
}
```

**Monitoring Failure Patterns:**

```csharp
public class FailureMonitoring
{
    private readonly ILogger<FailureMonitoring> _logger;
    
    public void MonitorServiceHealth()
    {
        // Alert on patterns
        while (true)
        {
            var circuitState = _circuitBreaker.State;
            
            if (circuitState == CircuitState.Open)
                _logger.LogError("Circuit OPEN - Service down!");
            
            if (circuitState == CircuitState.HalfOpen)
                _logger.LogWarning("Circuit HALF-OPEN - Recovering?");
            
            Task.Delay(5000).Wait();
        }
    }
}
```

---

## Summary Table: Key Patterns

| Pattern | Problem | Solution | When |
|---------|---------|----------|------|
| Circuit Breaker | Cascading failures | Fail fast, prevent calls | Service down |
| Timeout | Hanging requests | Return error after N sec | Network slow |
| Bulkhead | Resource exhaustion | Isolate thread pools | Many requests |
| Fallback | Service unavailable | Use cached/default data | Best effort |
| Retry | Transient failures | Retry with backoff | Network blip |
| Saga | Distributed transactions | Compensating transactions | Multi-service |
| API Gateway | No entry point | Single URL for all | Routing |
| Circuit Breaker | Cascading failures | Fail fast | Service down |
| Service Discovery | Hard-coded URLs | Dynamic resolution | Container env |
| Correlation ID | No tracing | Track request end-to-end | Debugging |

---

# 50 Advanced Microservices Interview Q&A with Code Examples

---

## Q16: What is CQRS (Command Query Responsibility Segregation) Pattern?

### Answer:

**Concept:**
Separate read and write operations into different models. Reads use a denormalized view; writes update the source of truth.

**Diagram:**

```
TRADITIONAL (Single Model):
┌──────────────────────────────┐
│      Product Database        │
│   (Normalized Structure)     │
├──────────────────────────────┤
│ ProductId | Name | Category  │
│ 1         | Laptop | Tech   │
│ 2         | Phone  | Tech   │
└────────┬─────────────────────┘
         │
    ┌────┴─────┐
    │           │
Write Command  Read Query
(INSERT/UPDATE) (SELECT)

Problem: Complex queries on normalized data are slow!

CQRS (Separated Models):
┌──────────────────────────┐      ┌──────────────────────────┐
│  WRITE MODEL             │      │  READ MODEL              │
│  (Normalized)            │      │  (Denormalized)          │
├──────────────────────────┤      ├──────────────────────────┤
│ ProductId | Name | Price │      │ ProductId | Name | Price │
│ 1         | Laptop | 999 │──────→ 1         | Laptop | 999 │
│ (Source of Truth)        │      │ 2         | Phone | 599  │
│                          │      │ 3         | Tablet | 399 │
└──────────────────────────┘      │ (Optimized for reads)    │
                                   └──────────────────────────┘
                                            ↑
                                   Event: ProductCreated
                                   EventStream updates
                                   read model asynchronously
```

**C# Implementation:**

```csharp
// ========== WRITE SIDE (Command Handler) ==========
public class CreateProductCommandHandler
{
    private readonly IProductWriteRepository _writeRepository;
    private readonly IEventBus _eventBus;
    
    public async Task<int> HandleAsync(CreateProductCommand command)
    {
        // Validate and save to WRITE database (normalized)
        var product = new Product
        {
            Name = command.Name,
            Price = command.Price,
            Category = command.Category
        };
        
        var productId = await _writeRepository.SaveAsync(product);
        
        // Publish event for read model to consume
        await _eventBus.PublishAsync(new ProductCreatedEvent
        {
            ProductId = productId,
            Name = command.Name,
            Price = command.Price,
            Category = command.Category
        });
        
        return productId;
    }
}

// ========== READ SIDE (Query Handler) ==========
public class GetProductsQueryHandler
{
    private readonly IProductReadRepository _readRepository;
    
    public async Task<List<ProductReadModel>> HandleAsync(
        GetProductsQuery query)
    {
        // Query from READ database (denormalized, optimized)
        // This is FAST because data is pre-denormalized
        return await _readRepository.GetAllAsync();
    }
}

// ========== EVENT CONSUMER (Updates Read Model) ==========
public class ProductCreatedEventHandler : IConsumer<ProductCreatedEvent>
{
    private readonly IProductReadRepository _readRepository;
    
    public async Task Consume(ConsumeContext<ProductCreatedEvent> context)
    {
        // When product is created, update READ model
        var readModel = new ProductReadModel
        {
            ProductId = context.Message.ProductId,
            Name = context.Message.Name,
            Price = context.Message.Price,
            Category = context.Message.Category,
            // Additional denormalized data for fast reads
            DisplayPrice = FormatPrice(context.Message.Price),
            SearchKeywords = GenerateSearchKeywords(context.Message.Name)
        };
        
        await _readRepository.SaveAsync(readModel);
    }
}

// ========== REPOSITORIES ==========
// WRITE Repository (Normalized)
public interface IProductWriteRepository
{
    Task<int> SaveAsync(Product product);
    Task UpdateAsync(Product product);
    Task DeleteAsync(int productId);
}

// READ Repository (Denormalized)
public interface IProductReadRepository
{
    Task<List<ProductReadModel>> GetAllAsync();
    Task<ProductReadModel> GetByIdAsync(int productId);
    Task<List<ProductReadModel>> SearchByNameAsync(string name);
    Task<List<ProductReadModel>> GetByPriceRangeAsync(
        decimal minPrice, decimal maxPrice);
}

// ========== DTOs ==========
public class Product // Write model (normalized)
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public int CategoryId { get; set; }
}

public class ProductReadModel // Read model (denormalized)
{
    public int ProductId { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public string Category { get; set; }
    public string DisplayPrice { get; set; } // Pre-formatted
    public List<string> SearchKeywords { get; set; }
    public int ViewCount { get; set; }
    public double AverageRating { get; set; }
}
```

**Benefits:**
- Read queries are fast (pre-denormalized)
- Can scale read and write independently
- Write database optimized for writes
- Read database optimized for queries

---

## Q17: What is Event Sourcing?

### Answer:

**Concept:**
Don't store current state. Store all events that led to that state. Rebuild state by replaying events.

**Diagram:**

```
TRADITIONAL STATE STORAGE:
┌─────────────────────┐
│ Current Product     │
│ State               │
├─────────────────────┤
│ Price: $100         │
│ Stock: 50           │
│ Status: Active      │
└─────────────────────┘

Problem:
- Can't see history
- Concurrent updates conflict
- Hard to debug "what happened?"

EVENT SOURCING:
┌─────────────────────────────────────┐
│ Event Stream (Immutable Log)        │
├─────────────────────────────────────┤
│ 1. ProductCreated                   │
│    Id: 1, Name: Laptop, Price: $100 │
│                                     │
│ 2. PriceChanged                     │
│    Id: 1, NewPrice: $90             │
│                                     │
│ 3. InventoryAdded                   │
│    Id: 1, Quantity: +10             │
│                                     │
│ 4. ProductPriceChanged              │
│    Id: 1, NewPrice: $85             │
│                                     │
│ 5. InventoryReserved                │
│    Id: 1, Quantity: -5              │
└─────────────────────────────────────┘
        │
        └─→ Replay events
            │
            ├─ ProductCreated
            ├─ Apply: Price=$100, Stock=0
            │
            ├─ PriceChanged
            ├─ Apply: Price=$90
            │
            ├─ InventoryAdded
            ├─ Apply: Stock=10
            │
            ├─ PriceChanged
            ├─ Apply: Price=$85
            │
            ├─ InventoryReserved
            ├─ Apply: Stock=5
            │
            ↓
        CURRENT STATE:
        Price: $85
        Stock: 5
        Status: Active

BENEFIT: Complete audit trail! Can see all changes.
```

**C# Implementation:**

```csharp
// ========== EVENT CLASSES ==========
public abstract class DomainEvent
{
    public int AggregateId { get; set; }
    public DateTime OccurredAt { get; set; }
    public int Version { get; set; }
}

public class ProductCreatedEvent : DomainEvent
{
    public string Name { get; set; }
    public decimal Price { get; set; }
    public int InitialStock { get; set; }
}

public class PriceChangedEvent : DomainEvent
{
    public decimal NewPrice { get; set; }
    public string Reason { get; set; }
}

public class InventoryAddedEvent : DomainEvent
{
    public int Quantity { get; set; }
}

public class InventoryReservedEvent : DomainEvent
{
    public int Quantity { get; set; }
    public int OrderId { get; set; }
}

// ========== AGGREGATE ROOT ==========
public class Product
{
    public int Id { get; private set; }
    public string Name { get; private set; }
    public decimal Price { get; private set; }
    public int Stock { get; private set; }
    public int Version { get; private set; }
    
    private List<DomainEvent> _uncommittedEvents = new();
    
    public static Product Create(int id, string name, 
        decimal price, int stock)
    {
        var product = new Product { Id = id, Version = 1 };
        
        product._uncommittedEvents.Add(new ProductCreatedEvent
        {
            AggregateId = id,
            Name = name,
            Price = price,
            InitialStock = stock,
            Version = 1
        });
        
        product.Name = name;
        product.Price = price;
        product.Stock = stock;
        
        return product;
    }
    
    public void ChangePrice(decimal newPrice, string reason)
    {
        this.Price = newPrice;
        this.Version++;
        
        _uncommittedEvents.Add(new PriceChangedEvent
        {
            AggregateId = Id,
            NewPrice = newPrice,
            Reason = reason,
            Version = Version
        });
    }
    
    public void ReserveInventory(int quantity, int orderId)
    {
        if (Stock < quantity)
            throw new Exception("Insufficient stock");
        
        Stock -= quantity;
        Version++;
        
        _uncommittedEvents.Add(new InventoryReservedEvent
        {
            AggregateId = Id,
            Quantity = quantity,
            OrderId = orderId,
            Version = Version
        });
    }
    
    // Replay events to rebuild state
    public void LoadFromHistory(List<DomainEvent> events)
    {
        foreach (var @event in events.OrderBy(e => e.Version))
        {
            ApplyEvent(@event);
        }
    }
    
    private void ApplyEvent(DomainEvent @event)
    {
        switch (@event)
        {
            case ProductCreatedEvent created:
                Id = created.AggregateId;
                Name = created.Name;
                Price = created.Price;
                Stock = created.InitialStock;
                Version = created.Version;
                break;
                
            case PriceChangedEvent priceChanged:
                Price = priceChanged.NewPrice;
                Version = priceChanged.Version;
                break;
                
            case InventoryReservedEvent reserved:
                Stock -= reserved.Quantity;
                Version = reserved.Version;
                break;
        }
    }
    
    public List<DomainEvent> GetUncommittedEvents()
    {
        return _uncommittedEvents;
    }
    
    public void MarkEventsAsCommitted()
    {
        _uncommittedEvents.Clear();
    }
}

// ========== EVENT STORE ==========
public class EventStore
{
    private readonly DbContext _context;
    
    public async Task SaveEventsAsync(int aggregateId, 
        List<DomainEvent> events)
    {
        foreach (var @event in events)
        {
            var eventRecord = new StoredEvent
            {
                AggregateId = aggregateId,
                EventType = @event.GetType().Name,
                EventData = JsonConvert.SerializeObject(@event),
                OccurredAt = @event.OccurredAt
            };
            
            await _context.StoredEvents.AddAsync(eventRecord);
        }
        
        await _context.SaveChangesAsync();
    }
    
    public async Task<List<DomainEvent>> GetEventsAsync(int aggregateId)
    {
        var storedEvents = await _context.StoredEvents
            .Where(e => e.AggregateId == aggregateId)
            .OrderBy(e => e.Version)
            .ToListAsync();
        
        var events = new List<DomainEvent>();
        
        foreach (var stored in storedEvents)
        {
            var eventType = Type.GetType($"YourNamespace.{stored.EventType}");
            var @event = JsonConvert.DeserializeObject(
                stored.EventData, eventType) as DomainEvent;
            events.Add(@event);
        }
        
        return events;
    }
}

// ========== REPOSITORY ==========
public class ProductRepository
{
    private readonly EventStore _eventStore;
    
    public async Task<Product> GetByIdAsync(int id)
    {
        var events = await _eventStore.GetEventsAsync(id);
        
        if (!events.Any())
            return null;
        
        var product = new Product();
        product.LoadFromHistory(events); // Replay events
        
        return product;
    }
    
    public async Task SaveAsync(Product product)
    {
        var events = product.GetUncommittedEvents();
        await _eventStore.SaveEventsAsync(product.Id, events);
        product.MarkEventsAsCommitted();
    }
}
```

**Benefits:**
- Complete audit trail
- Easy to debug (replay events)
- Can rebuild state at any point in time
- Supports temporal queries

---

## Q18: What is idempotency and why is it important in microservices?

### Answer:

**Concept:**
Idempotent operation: executing it multiple times has the same effect as executing it once.

**Why Important:**
Network failures cause retries. Without idempotency, retries create duplicates.

**Diagram:**

```
NON-IDEMPOTENT (Problem):
┌─────────────────────────────────┐
│ Client: Transfer $100            │
└─────────────────────────────────┘
         │
         ↓
    ┌─────────────────────┐
    │ Service receives    │
    │ (Process $100)      │
    │ Result: SUCCESS     │
    │ Response: OK        │
    └─────────────────────┘
         │
    Response lost (network fail)
    Client doesn't receive "OK"
    │
    ├─ Client retries: Transfer $100
    │  │
    │  ↓
    │ ┌─────────────────────┐
    │ │ Service receives    │
    │ │ (Process $100)      │
    │ │ (AGAIN!)            │
    │ │ Result: SUCCESS     │
    │ └─────────────────────┘
    │
    └─ Now transferred $200 instead of $100!
       Bug: Double charging!

IDEMPOTENT (Solution):
┌─────────────────────────────────┐
│ Client: Transfer $100            │
│ RequestId: ABC-123              │
└─────────────────────────────────┘
         │
         ↓
    ┌─────────────────────────┐
    │ Service receives        │
    │ Check: ABC-123 exists?  │ NO
    │ (Process $100)          │
    │ Store: RequestId ABC-123│
    │ Result: SUCCESS         │
    │ Response: OK            │
    └─────────────────────────┘
         │
    Response lost (network fail)
    │
    ├─ Client retries: Transfer $100 (RequestId: ABC-123)
    │  │
    │  ↓
    │ ┌─────────────────────────┐
    │ │ Service receives        │
    │ │ Check: ABC-123 exists?  │ YES!
    │ │ Already processed       │
    │ │ Return: OK (no charge)  │
    │ └─────────────────────────┘
    │
    └─ Still only $100 transferred!
       Correct behavior!
```

**C# Implementation:**

```csharp
// ========== IDEMPOTENCY KEY TRACKING ==========
public class IdempotencyRecord
{
    public string IdempotencyKey { get; set; }
    public string RequestData { get; set; }
    public string Response { get; set; }
    public DateTime CreatedAt { get; set; }
    public bool IsProcessed { get; set; }
}

public class IdempotencyStore
{
    private readonly DbContext _context;
    
    public async Task<IdempotencyRecord> GetByKeyAsync(
        string idempotencyKey)
    {
        return await _context.IdempotencyRecords
            .FirstOrDefaultAsync(r => r.IdempotencyKey == idempotencyKey);
    }
    
    public async Task StoreAsync(string idempotencyKey, 
        string requestData)
    {
        var record = new IdempotencyRecord
        {
            IdempotencyKey = idempotencyKey,
            RequestData = requestData,
            CreatedAt = DateTime.UtcNow,
            IsProcessed = false
        };
        
        await _context.IdempotencyRecords.AddAsync(record);
        await _context.SaveChangesAsync();
    }
    
    public async Task MarkProcessedAsync(string idempotencyKey, 
        string response)
    {
        var record = await GetByKeyAsync(idempotencyKey);
        
        if (record != null)
        {
            record.Response = response;
            record.IsProcessed = true;
            await _context.SaveChangesAsync();
        }
    }
}

// ========== IDEMPOTENT PAYMENT SERVICE ==========
[ApiController]
[Route("api/payments")]
public class PaymentController
{
    private readonly IPaymentService _paymentService;
    private readonly IdempotencyStore _idempotencyStore;
    private readonly ILogger<PaymentController> _logger;
    
    [HttpPost("transfer")]
    public async Task<IActionResult> TransferMoney(
        [FromBody] TransferRequest request,
        [FromHeader(Name = "Idempotency-Key")] string idempotencyKey)
    {
        // STEP 1: Check if already processed
        var existing = await _idempotencyStore
            .GetByKeyAsync(idempotencyKey);
        
        if (existing != null && existing.IsProcessed)
        {
            _logger.LogInformation(
                "Request already processed: {IdempotencyKey}", 
                idempotencyKey);
            
            // Return same response as before
            return Ok(JsonConvert.DeserializeObject(
                existing.Response));
        }
        
        // STEP 2: Store request (before processing)
        await _idempotencyStore.StoreAsync(
            idempotencyKey, 
            JsonConvert.SerializeObject(request));
        
        try
        {
            // STEP 3: Process payment
            var result = await _paymentService.TransferAsync(
                request.FromAccount, 
                request.ToAccount, 
                request.Amount);
            
            // STEP 4: Store response
            var response = JsonConvert.SerializeObject(result);
            await _idempotencyStore.MarkProcessedAsync(
                idempotencyKey, response);
            
            return Ok(result);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, 
                "Transfer failed: {IdempotencyKey}", 
                idempotencyKey);
            
            return BadRequest(new { Error = ex.Message });
        }
    }
}

// ========== IDEMPOTENT MIDDLEWARE ==========
public class IdempotencyMiddleware
{
    private readonly RequestDelegate _next;
    private readonly IdempotencyStore _idempotencyStore;
    
    public IdempotencyMiddleware(RequestDelegate next, 
        IdempotencyStore idempotencyStore)
    {
        _next = next;
        _idempotencyStore = idempotencyStore;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        var idempotencyKey = context.Request.Headers
            .FirstOrDefault(h => h.Key == "Idempotency-Key")
            .Value.FirstOrDefault();
        
        if (string.IsNullOrEmpty(idempotencyKey))
        {
            await _next(context);
            return;
        }
        
        // Check if already processed
        var existing = await _idempotencyStore
            .GetByKeyAsync(idempotencyKey);
        
        if (existing?.IsProcessed == true)
        {
            context.Response.StatusCode = 200;
            await context.Response.WriteAsync(existing.Response);
            return;
        }
        
        await _next(context);
    }
}
```

**Client Usage:**

```csharp
// Client includes Idempotency-Key header
var request = new HttpRequestMessage(HttpMethod.Post, 
    "https://payment-service/api/payments/transfer");

request.Content = new StringContent(
    JsonConvert.SerializeObject(new { Amount = 100 }),
    Encoding.UTF8, "application/json");

// Generate unique key (usually UUID)
var idempotencyKey = Guid.NewGuid().ToString();

request.Headers.Add("Idempotency-Key", idempotencyKey);

var response = await httpClient.SendAsync(request);
// Client can retry with same key safely!
```

---

## Q19: What is the Bulkhead Pattern?

### Answer:

**Concept:**
Isolate resources (threads, connections) per service to prevent one failing service from taking down others.

**Diagram:**

```
WITHOUT BULKHEAD (Problem):
┌────────────────────────────────────┐
│ Thread Pool (100 threads)          │
├────────────────────────────────────┤
│ ┌──────────┐ ┌──────────┐         │
│ │ Service A│ │ Service B│         │
│ │  99 reqs │ │   1 req  │         │
│ └──────────┘ └──────────┘         │
│                                    │
│ If Service A gets slow:            │
│ ├─ Uses 99 threads                │
│ ├─ Service B only gets 1 thread   │
│ └─ Service B fails (can't respond) │
└────────────────────────────────────┘

WITH BULKHEAD (Solution):
┌────────────────────────────────────┐
│ Thread Pools (Isolated)            │
├────────────────────────────────────┤
│ ┌──────────────────┐               │
│ │ Service A Pool   │               │
│ │ (50 threads)     │               │
│ │ ┌──────┬──────┐  │               │
│ │ │ Req 1│ Req 2│  │               │
│ │ └──────┴──────┘  │               │
│ └──────────────────┘               │
│                                    │
│ ┌──────────────────┐               │
│ │ Service B Pool   │               │
│ │ (50 threads)     │               │
│ │ ┌──────┬──────┐  │               │
│ │ │ Req 1│ Req 2│  │               │
│ │ └──────┴──────┘  │               │
│ └──────────────────┘               │
│                                    │
│ If Service A gets slow:            │
│ ├─ Uses its 50 threads            │
│ ├─ Service B still has 50 threads │
│ └─ Service B responds normally!    │
└────────────────────────────────────┘
```

**C# Implementation:**

```csharp
// ========== BULKHEAD WITH POLLY ==========
public class BulkheadConfiguration
{
    public void ConfigureServices(IServiceCollection services)
    {
        // Separate bulkheads for different services
        
        // Service A: 10 parallel, 20 queued
        var serviceABulkhead = Policy.BulkheadAsync(
            maxParallelization: 10,
            maxQueuingActions: 20,
            onBulkheadRejectedAsync: context =>
            {
                Console.WriteLine("Service A bulkhead saturated");
                return Task.CompletedTask;
            }
        );
        
        // Service B: 5 parallel, 10 queued
        var serviceBBulkhead = Policy.BulkheadAsync(
            maxParallelization: 5,
            maxQueuingActions: 10,
            onBulkheadRejectedAsync: context =>
            {
                Console.WriteLine("Service B bulkhead saturated");
                return Task.CompletedTask;
            }
        );
        
        services.AddSingleton(serviceABulkhead);
        services.AddSingleton(serviceBBulkhead);
    }
}

// ========== USAGE ==========
public class ResilientServiceClient
{
    private readonly IAsyncPolicy _serviceABulkhead;
    private readonly IAsyncPolicy _serviceBBulkhead;
    
    public async Task CallServiceAAsync()
    {
        try
        {
            await _serviceABulkhead.ExecuteAsync(async () =>
            {
                // Call Service A
                await _httpClient.GetAsync(
                    "https://service-a/api/data");
            });
        }
        catch (BulkheadRejectedException)
        {
            Console.WriteLine("Service A queue full - circuit open");
        }
    }
    
    public async Task CallServiceBAsync()
    {
        try
        {
            await _serviceBBulkhead.ExecuteAsync(async () =>
            {
                // Call Service B
                await _httpClient.GetAsync(
                    "https://service-b/api/data");
            });
        }
        catch (BulkheadRejectedException)
        {
            Console.WriteLine("Service B queue full - circuit open");
        }
    }
}

// ========== THREAD POOL ISOLATION ==========
public class ThreadPoolBulkhead
{
    private readonly Executor _serviceAExecutor;
    private readonly Executor _serviceBExecutor;
    
    public ThreadPoolBulkhead()
    {
        // Separate thread pools
        _serviceAExecutor = new Executor(
            name: "ServiceA", 
            threadCount: 10);
        
        _serviceBExecutor = new Executor(
            name: "ServiceB", 
            threadCount: 5);
    }
    
    public async Task CallServiceAAsync(Action work)
    {
        await _serviceAExecutor.ExecuteAsync(work);
    }
    
    public async Task CallServiceBAsync(Action work)
    {
        await _serviceBExecutor.ExecuteAsync(work);
    }
}

// ========== CUSTOM EXECUTOR ==========
public class Executor
{
    private readonly string _name;
    private readonly BlockingCollection<Func<Task>> _queue;
    private readonly Task[] _workers;
    
    public Executor(string name, int threadCount)
    {
        _name = name;
        _queue = new BlockingCollection<Func<Task>>(100);
        _workers = new Task[threadCount];
        
        for (int i = 0; i < threadCount; i++)
        {
            _workers[i] = Task.Run(() => WorkerLoop());
        }
    }
    
    public async Task ExecuteAsync(Func<Task> work)
    {
        if (!_queue.TryAdd(work, TimeSpan.FromSeconds(5)))
        {
            throw new Exception(
                $"Bulkhead {_name} queue is full");
        }
    }
    
    private async Task WorkerLoop()
    {
        foreach (var work in _queue.GetConsumingEnumerable())
        {
            try
            {
                await work();
            }
            catch (Exception ex)
            {
                Console.WriteLine(
                    $"Error in {_name}: {ex.Message}");
            }
        }
    }
}
```

---

## Q20: How do you implement Rate Limiting in Microservices?

### Answer:

**Concept:**
Control request rate to prevent service overload.

```
RATE LIMITING STRATEGIES:

1. FIXED WINDOW:
   ┌─ 0s ──┬─ 5s ──┬─ 10s ─┬─ 15s ─┐
   │ Window 1 │ Window 2 │ Window 3│
   │ Max: 100 │ Max: 100 │ Max: 100│
   │ Reqs: 95 │ Reqs: 120 (REJECT 20)
   │ ✓        │ ✗ (overflow)

2. SLIDING WINDOW:
   ┌──────────────────────────────────┐
   │ Last 60 seconds max 1000 reqs    │
   ├──────────────────────────────────┤
   │ ...[old reqs] [new reqs]...      │
   │ Sliding window always last 60s   │
   └──────────────────────────────────┘

3. TOKEN BUCKET:
   ┌─────────────────────────┐
   │ Tokens: [●●●●●●●●●●]    │
   │ Rate: 100 tokens/min    │
   │ Each request costs 1    │
   ├─────────────────────────┤
   │ Request arrives         │
   │ ├─ Token available? ✓   │
   │ ├─ Consume token        │
   │ └─ Process request      │
   │                         │
   │ Burst capacity: refill  │
   │ gradually                │
   └─────────────────────────┘

4. LEAKY BUCKET:
   ┌──────────────────┐
   │ Requests in      │
   │ [●●●●●●]        │
   ├──────────────────┤
   │ Queue buffer     │
   │ Max: 100 reqs    │
   ├──────────────────┤
   │ Process: 10/sec  │
   │ Leak out steady  │
   └──────────────────┘
```

**C# Implementation:**

```csharp
// ========== TOKEN BUCKET ALGORITHM ==========
public class TokenBucketRateLimiter
{
    private readonly int _capacity;
    private readonly double _refillRate; // tokens per second
    private double _tokens;
    private DateTime _lastRefillTime;
    private readonly object _lock = new object();
    
    public TokenBucketRateLimiter(int capacity, double refillRate)
    {
        _capacity = capacity;
        _refillRate = refillRate;
        _tokens = capacity;
        _lastRefillTime = DateTime.UtcNow;
    }
    
    public bool AllowRequest(int tokensNeeded = 1)
    {
        lock (_lock)
        {
            RefillTokens();
            
            if (_tokens >= tokensNeeded)
            {
                _tokens -= tokensNeeded;
                return true;
            }
            
            return false;
        }
    }
    
    private void RefillTokens()
    {
        var now = DateTime.UtcNow;
        var timePassed = (now - _lastRefillTime).TotalSeconds;
        var tokensToAdd = timePassed * _refillRate;
        
        _tokens = Math.Min(_capacity, _tokens + tokensToAdd);
        _lastRefillTime = now;
    }
}

// ========== SLIDING WINDOW RATE LIMITER ==========
public class SlidingWindowRateLimiter
{
    private readonly int _maxRequests;
    private readonly TimeSpan _window;
    private readonly ConcurrentQueue<DateTime> _requestTimestamps;
    
    public SlidingWindowRateLimiter(int maxRequests, TimeSpan window)
    {
        _maxRequests = maxRequests;
        _window = window;
        _requestTimestamps = new ConcurrentQueue<DateTime>();
    }
    
    public bool AllowRequest()
    {
        var now = DateTime.UtcNow;
        var cutoffTime = now.Subtract(_window);
        
        // Remove old requests outside window
        while (_requestTimestamps.TryPeek(out var timestamp))
        {
            if (timestamp < cutoffTime)
            {
                _requestTimestamps.TryDequeue(out _);
            }
            else
            {
                break;
            }
        }
        
        // Check if under limit
        if (_requestTimestamps.Count < _maxRequests)
        {
            _requestTimestamps.Enqueue(now);
            return true;
        }
        
        return false;
    }
}

// ========== RATE LIMITING MIDDLEWARE ==========
public class RateLimitingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly Dictionary<string, TokenBucketRateLimiter> 
        _limiters;
    
    public RateLimitingMiddleware(RequestDelegate next)
    {
        _next = next;
        _limiters = new Dictionary<string, TokenBucketRateLimiter>();
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        // Get user/API key
        var userId = context.User?.FindFirst("sub")?.Value 
            ?? "anonymous";
        
        // Get or create limiter for user
        if (!_limiters.ContainsKey(userId))
        {
            _limiters[userId] = new TokenBucketRateLimiter(
                capacity: 100,
                refillRate: 10); // 10 requests per second
        }
        
        var limiter = _limiters[userId];
        
        if (!limiter.AllowRequest())
        {
            context.Response.StatusCode = 429; // Too Many Requests
            context.Response.Headers.Add(
                "Retry-After", "1");
            await context.Response.WriteAsync(
                "Rate limit exceeded");
            return;
        }
        
        // Add rate limit headers
        context.Response.Headers.Add(
            "X-RateLimit-Limit", "100");
        context.Response.Headers.Add(
            "X-RateLimit-Remaining", "75");
        context.Response.Headers.Add(
            "X-RateLimit-Reset", 
            DateTimeOffset.UtcNow.AddSeconds(60)
            .ToUnixTimeSeconds().ToString());
        
        await _next(context);
    }
}

// ========== USAGE IN STARTUP ==========
public class Startup
{
    public void Configure(IApplicationBuilder app)
    {
        app.UseMiddleware<RateLimitingMiddleware>();
    }
}
```

**API Gateway Rate Limiting:**

```csharp
// Ocelot configuration
public class RateLimitOptions
{
    public void Configure(IApplicationBuilder app)
    {
        var config = new FileConfiguration();
        config.Routes = new List<FileRoute>
        {
            new FileRoute
            {
                DownstreamPathTemplate = "/api/orders",
                UpstreamPathTemplate = "/orders",
                RateLimitOptions = new FileRateLimitOptions
                {
                    ClientWhitelist = new List<string>(),
                    EnableRateLimiting = true,
                    Period = "1m", // per minute
                    PeriodTimespan = 60,
                    Limit = 100    // max 100 requests
                }
            }
        };
    }
}
```

---

## Q21: What is the Retry Pattern and How to Implement It?

### Answer:

**Concept:**
Automatically retry failed requests with exponential backoff and jitter.

```
RETRY STRATEGY:

IMMEDIATE RETRY (Bad):
Request ──→ Fail ──→ Immediate Retry ──→ Fail again
                    (Same condition)

RETRY WITH DELAY:
Request ──→ Fail ──→ Wait 1s ──→ Retry
                        ✓ (Often recovers)

EXPONENTIAL BACKOFF:
Attempt 1: Fail ──→ Wait 1s
Attempt 2: Fail ──→ Wait 2s (2^1)
Attempt 3: Fail ──→ Wait 4s (2^2)
Attempt 4: Fail ──→ Wait 8s (2^3)
Attempt 5: Give up ──→ Circuit Open

JITTER (Prevents thundering herd):
Without jitter: All clients retry at same time (spike)
With jitter: Random delay ± variance
           Spreads retries over time (smooth)
```

**C# Implementation:**

```csharp
// ========== RETRY POLICY WITH POLLY ==========
public class RetryConfiguration
{
    public static IAsyncPolicy<HttpResponseMessage> 
        GetRetryPolicy()
    {
        // Simple retry
        return HttpPolicyExtensions
            .HandleTransientHttpError()
            .WaitAndRetryAsync(
                retryCount: 3,
                sleepDurationProvider: attempt =>
                    TimeSpan.FromSeconds(
                        Math.Pow(2, attempt))); // 1s, 2s, 4s
    }
    
    public static IAsyncPolicy<HttpResponseMessage> 
        GetRetryPolicyWithJitter()
    {
        var random = new Random();
        
        // Retry with exponential backoff + jitter
        return HttpPolicyExtensions
            .HandleTransientHttpError()
            .WaitAndRetryAsync(
                retryCount: 3,
                sleepDurationProvider: attempt =>
                {
                    var baseDelay = Math.Pow(2, attempt);
                    var jitter = (random.NextDouble() - 0.5) * 0.5;
                    var delay = baseDelay + jitter;
                    
                    return TimeSpan.FromSeconds(delay);
                },
                onRetry: (outcome, timespan, retryCount, context) =>
                {
                    Console.WriteLine(
                        $"Retry {retryCount} after {timespan.TotalSeconds}s");
                }
            );
    }
}

// ========== MANUAL RETRY IMPLEMENTATION ==========
public class RetryService
{
    private readonly HttpClient _httpClient;
    private readonly ILogger<RetryService> _logger;
    
    public async Task<T> GetWithRetryAsync<T>(
        string url, 
        int maxRetries = 3)
    {
        var attempt = 0;
        var random = new Random();
        
        while (attempt < maxRetries)
        {
            try
            {
                var response = await _httpClient
                    .GetAsync(url);
                
                if (response.IsSuccessStatusCode)
                {
                    return await response.Content
                        .ReadAsAsync<T>();
                }
                
                // Transient error (5xx)
                if ((int)response.StatusCode >= 500)
                {
                    throw new HttpRequestException();
                }
            }
            catch (HttpRequestException ex)
            {
                attempt++;
                
                if (attempt >= maxRetries)
                {
                    throw;
                }
                
                // Calculate delay
                var baseDelay = Math.Pow(2, attempt);
                var jitter = random.NextDouble() * 0.1;
                var delay = TimeSpan.FromSeconds(
                    baseDelay + jitter);
                
                _logger.LogWarning(
                    "Request failed. Retry {Attempt} of {MaxRetries} " +
                    "after {Delay}ms",
                    attempt, maxRetries, delay.TotalMilliseconds);
                
                await Task.Delay(delay);
            }
        }
        
        throw new Exception("Max retries exceeded");
    }
}

// ========== IDEMPOTENT + RETRY ==========
public class IdempotentRetryService
{
    private readonly HttpClient _httpClient;
    
    public async Task<T> PostWithRetryAsync<T>(
        string url, 
        object data, 
        int maxRetries = 3)
    {
        var idempotencyKey = Guid.NewGuid().ToString();
        
        for (int attempt = 0; attempt < maxRetries; attempt++)
        {
            try
            {
                var request = new HttpRequestMessage(
                    HttpMethod.Post, url);
                
                // Add idempotency key
                request.Headers.Add(
                    "Idempotency-Key", 
                    idempotencyKey);
                
                request.Content = new StringContent(
                    JsonConvert.SerializeObject(data),
                    Encoding.UTF8, 
                    "application/json");
                
                var response = await _httpClient
                    .SendAsync(request);
                
                if (response.IsSuccessStatusCode)
                {
                    return await response.Content
                        .ReadAsAsync<T>();
                }
                
                if (attempt < maxRetries - 1)
                {
                    var delay = TimeSpan.FromSeconds(
                        Math.Pow(2, attempt));
                    
                    await Task.Delay(delay);
                }
            }
            catch (Exception ex)
            {
                if (attempt >= maxRetries - 1)
                    throw;
                
                var delay = TimeSpan.FromSeconds(
                    Math.Pow(2, attempt));
                
                await Task.Delay(delay);
            }
        }
        
        throw new Exception("Max retries exceeded");
    }
}

// ========== RETRY POLICIES COMBINED ==========
public class ResilientHttpClient
{
    private readonly IAsyncPolicy<HttpResponseMessage> 
        _policy;
    private readonly HttpClient _httpClient;
    
    public ResilientHttpClient(HttpClient httpClient)
    {
        _httpClient = httpClient;
        
        // Retry policy
        var retryPolicy = HttpPolicyExtensions
            .HandleTransientHttpError()
            .WaitAndRetryAsync(
                retryCount: 3,
                sleepDurationProvider: attempt =>
                    TimeSpan.FromSeconds(
                        Math.Pow(2, attempt)));
        
        // Circuit breaker policy
        var circuitBreakerPolicy = HttpPolicyExtensions
            .HandleTransientHttpError()
            .CircuitBreakerAsync(
                handledEventsAllowedBeforeBreaking: 5,
                durationOfBreak: TimeSpan.FromSeconds(10));
        
        // Combine
        _policy = Policy.WrapAsync(
            retryPolicy, 
            circuitBreakerPolicy);
    }
    
    public async Task<T> GetAsync<T>(string url)
    {
        var response = await _policy.ExecuteAsync(
            () => _httpClient.GetAsync(url));
        
        return await response.Content.ReadAsAsync<T>();
    }
}
```

---

## Q22: Explain the concept of Dead Letter Queue (DLQ)

### Answer:

**Concept:**
Messages that fail processing repeatedly are moved to Dead Letter Queue for manual inspection.

```
MESSAGE FLOW:

NORMAL FLOW:
Message ──→ Queue ──→ Consumer ──→ Process ──→ Success
                                                  │
                                          Remove from queue

FAILURE FLOW (with DLQ):
Message ──→ Queue ──→ Consumer ──→ Process ──→ FAIL
                                                  │
                                    Retry (attempt 1) ──→ FAIL
                                    Retry (attempt 2) ──→ FAIL
                                    Retry (attempt 3) ──→ FAIL
                                    Max retries exceeded
                                                  │
                                                  ↓
                                            Dead Letter Queue
                                            (Move here)
                                                  │
                                    ┌─────────────┴─────────────┐
                                    │                           │
                            Admin review              Analyze failure
                                    │                           │
                            Fix issue                 Log error details
                                    │                           │
                            Requeue message           Alert ops team
```

**C# Implementation:**

```csharp
// ========== AZURE SERVICE BUS DLQ ==========
public class ServiceBusMessageHandler
{
    private readonly IAsyncPolicy<bool> _retryPolicy;
    
    public ServiceBusMessageHandler()
    {
        // Retry 3 times, then move to DLQ
        _retryPolicy = Policy
            .Handle<Exception>()
            .FallbackAsync(false)
            .WrapAsync(
                Policy
                    .Handle<Exception>()
                    .RetryAsync<bool>(3)
            );
    }
}

// ========== RABBITMQ DLQ SETUP ==========
public class RabbitMqDlqSetup
{
    public void ConfigureQueues(IModel channel)
    {
        const string mainQueue = "order-queue";
        const string dlQueue = "order-dlq";
        const string dlExchange = "order-dlx";
        
        // Declare main queue with DLX routing
        channel.QueueDeclare(
            queue: mainQueue,
            durable: true,
            exclusive: false,
            autoDelete: false,
            arguments: new Dictionary<string, object>
            {
                // Route failed messages to DLX
                { "x-dead-letter-exchange", dlExchange },
                // After 3 retries
                { "x-max-length", 1000 }
            }
        );
        
        // Declare DLX (Dead Letter Exchange)
        channel.ExchangeDeclare(
            exchange: dlExchange,
            type: ExchangeType.Direct,
            durable: true
        );
        
        // Declare DLQ
        channel.QueueDeclare(
            queue: dlQueue,
            durable: true,
            exclusive: false,
            autoDelete: false
        );
        
        // Bind DLX to DLQ
        channel.QueueBind(
            queue: dlQueue,
            exchange: dlExchange,
            routingKey: ""
        );
    }
}

// ========== CONSUMER WITH DLQ HANDLING ==========
public class OrderConsumer : IConsumer<OrderCreatedEvent>
{
    private readonly IOrderRepository _repository;
    private readonly ILogger<OrderConsumer> _logger;
    private const int MaxRetries = 3;
    
    public async Task Consume(
        ConsumeContext<OrderCreatedEvent> context)
    {
        var message = context.Message;
        var retryCount = context.Headers
            .Get<int>("x-retry-count", 0);
        
        try
        {
            // Process order
            await _repository.SaveAsync(
                new Order 
                { 
                    Id = message.OrderId,
                    Items = message.Items 
                });
            
            _logger.LogInformation(
                "Order processed: {OrderId}", 
                message.OrderId);
        }
        catch (Exception ex)
        {
            retryCount++;
            
            if (retryCount >= MaxRetries)
            {
                // Move to DLQ
                _logger.LogError(
                    "Order failed after {RetryCount} retries: {OrderId}. " +
                    "Moving to DLQ. Error: {Error}",
                    retryCount, 
                    message.OrderId, 
                    ex.Message);
                
                // Send to DLQ
                await context.Forward(
                    new Uri("rabbitmq://localhost//order-dlq"));
            }
            else
            {
                _logger.LogWarning(
                    "Order processing failed: {OrderId}. " +
                    "Retry {RetryCount} of {MaxRetries}",
                    message.OrderId, 
                    retryCount, 
                    MaxRetries);
                
                // Retry with backoff
                context.Headers.Set(
                    "x-retry-count", retryCount);
                
                throw;
            }
        }
    }
}

// ========== DLQ MONITORING & PROCESSING ==========
[ApiController]
[Route("api/dlq")]
public class DlqController
{
    private readonly IMessageReceiver _messageReceiver;
    private readonly ILogger<DlqController> _logger;
    
    [HttpGet("messages")]
    public async Task<IActionResult> GetDlqMessages()
    {
        var messages = new List<object>();
        
        // Read messages from DLQ (don't delete yet)
        var message = await _messageReceiver
            .PeekAsync();
        
        while (message != null)
        {
            messages.Add(new
            {
                MessageId = message.MessageId,
                Body = Encoding.UTF8.GetString(
                    message.Body),
                DeadLetterReason = 
                    message.DeadLetterReason,
                DeadLetterErrorDescription = 
                    message.DeadLetterErrorDescription,
                DeliveryCount = 
                    message.DeliveryCount
            });
            
            message = await _messageReceiver
                .PeekAsync();
        }
        
        return Ok(messages);
    }
    
    [HttpPost("retry/{messageId}")]
    public async Task<IActionResult> RetryMessage(
        string messageId)
    {
        try
        {
            // Get message from DLQ
            var message = await _messageReceiver
                .ReceiveAsync();
            
            // Reprocess
            _logger.LogInformation(
                "Reprocessing DLQ message: {MessageId}", 
                messageId);
            
            // Send back to main queue
            await _messageReceiver.CompleteAsync(
                message);
            
            return Ok(new 
            { 
                Message = "Retrying message" 
            });
        }
        catch (Exception ex)
        {
            return BadRequest(new 
            { 
                Error = ex.Message 
            });
        }
    }
}
```

---

## Q23: What is Health Check and How to Implement?

### Answer:

**Concept:**
Endpoint that reports if service is healthy (running, dependencies available).

```
HEALTH CHECK FLOW:

┌─────────────────────────────┐
│ Health Check Endpoint       │
│ GET /health                 │
└──────────────┬──────────────┘
               │
       ┌───────┴────────┐
       │                │
   Ready?           Responsive?
       │                │
   ┌───▼──────┐    ┌────▼────┐
   │ DB alive?│    │ Memory OK?│
   │ Cache OK?│    │ CPU OK?  │
   │ Network? │    │ Disk OK? │
   └───┬──────┘    └────┬────┘
       │                │
       └────────┬───────┘
                │
    ┌───────────▼────────────┐
    │ Return Status          │
    ├────────────────────────┤
    │ {                      │
    │   "status": "Healthy"  │
    │   "checks": {          │
    │     "database": "OK",  │
    │     "memory": "OK"     │
    │   }                    │
    │ }                      │
    └────────────────────────┘
```

**Health Check States:**

```
Healthy (200):
├─ Service is running
├─ All dependencies OK
└─ Ready to handle requests

Degraded (503):
├─ Service running
├─ Some dependencies down (cache)
└─ Can still handle requests (reduced functionality)

Unhealthy (503):
├─ Service not responding
├─ Critical dependencies down (database)
└─ Can't handle requests
```

**C# Implementation:**

```csharp
// ========== HEALTH CHECK MODELS ==========
public class HealthCheckResponse
{
    [JsonPropertyName("status")]
    public string Status { get; set; }
    
    [JsonPropertyName("timestamp")]
    public DateTime Timestamp { get; set; }
    
    [JsonPropertyName("checks")]
    public Dictionary<string, string> Checks { get; set; }
    
    [JsonPropertyName("totalDuration")]
    public long TotalDurationMs { get; set; }
}

// ========== CUSTOM HEALTH CHECK ==========
public class DatabaseHealthCheck : IHealthCheck
{
    private readonly DbContext _context;
    
    public DatabaseHealthCheck(DbContext context)
    {
        _context = context;
    }
    
    public async Task<HealthCheckResult> CheckHealthAsync(
        HealthCheckContext context,
        CancellationToken cancellationToken = default)
    {
        try
        {
            // Try to query database
            await _context.Database.ExecuteSqlAsync(
                new RawSqlString("SELECT 1"));
            
            return HealthCheckResult.Healthy(
                "Database is accessible");
        }
        catch (Exception ex)
        {
            return HealthCheckResult.Unhealthy(
                "Database check failed",
                exception: ex);
        }
    }
}

public class CacheHealthCheck : IHealthCheck
{
    private readonly IDistributedCache _cache;
    
    public CacheHealthCheck(IDistributedCache cache)
    {
        _cache = cache;
    }
    
    public async Task<HealthCheckResult> CheckHealthAsync(
        HealthCheckContext context,
        CancellationToken cancellationToken = default)
    {
        try
        {
            // Try to set/get value in cache
            var testKey = "health-check-test";
            await _cache.SetStringAsync(testKey, "ok");
            var value = await _cache.GetStringAsync(testKey);
            
            if (value == "ok")
            {
                return HealthCheckResult.Healthy(
                    "Cache is working");
            }
            
            return HealthCheckResult.Unhealthy(
                "Cache returned unexpected value");
        }
        catch (Exception ex)
        {
            return HealthCheckResult.Degraded(
                "Cache is degraded",
                exception: ex);
        }
    }
}

public class ExternalServiceHealthCheck : IHealthCheck
{
    private readonly HttpClient _httpClient;
    
    public ExternalServiceHealthCheck(HttpClient httpClient)
    {
        _httpClient = httpClient;
    }
    
    public async Task<HealthCheckResult> CheckHealthAsync(
        HealthCheckContext context,
        CancellationToken cancellationToken = default)
    {
        try
        {
            var response = await _httpClient
                .GetAsync("https://payment-service/health");
            
            if (response.IsSuccessStatusCode)
            {
                return HealthCheckResult.Healthy(
                    "Payment service is healthy");
            }
            
            return HealthCheckResult.Degraded(
                "Payment service returned error");
        }
        catch (Exception ex)
        {
            return HealthCheckResult.Unhealthy(
                "Payment service is unreachable",
                exception: ex);
        }
    }
}

// ========== STARTUP CONFIGURATION ==========
public void ConfigureServices(IServiceCollection services)
{
    services
        .AddHealthChecks()
        .AddCheck<DatabaseHealthCheck>(
            "Database",
            tags: new[] { "ready" })
        .AddCheck<CacheHealthCheck>(
            "Cache",
            tags: new[] { "live" })
        .AddCheck<ExternalServiceHealthCheck>(
            "PaymentService",
            tags: new[] { "live" });
}

public void Configure(IApplicationBuilder app)
{
    // Map health check endpoints
    app.UseHealthChecks("/health/live", new HealthCheckOptions
    {
        Predicate = check => check.Tags.Contains("live"),
        ResponseWriter = WriteResponse
    });
    
    app.UseHealthChecks("/health/ready", new HealthCheckOptions
    {
        Predicate = check => check.Tags.Contains("ready"),
        ResponseWriter = WriteResponse
    });
}

// ========== CUSTOM RESPONSE WRITER ==========
private static async Task WriteResponse(HttpContext context,
    HealthReport report)
{
    context.Response.ContentType = "application/json";
    
    var response = new HealthCheckResponse
    {
        Status = report.Status.ToString(),
        Timestamp = DateTime.UtcNow,
        Checks = new Dictionary<string, string>(),
        TotalDurationMs = (long)report.TotalDuration
            .TotalMilliseconds
    };
    
    foreach (var entry in report.Entries)
    {
        response.Checks.Add(
            entry.Key,
            entry.Value.Status.ToString());
    }
    
    if (report.Status == HealthStatus.Unhealthy)
    {
        context.Response.StatusCode = 503;
    }
    
    await context.Response.WriteAsJsonAsync(response);
}

// ========== KUBERNETES PROBE INTEGRATION ==========
/*
Kubernetes uses health checks for:

1. Liveness Probe: Is container alive?
   Command: curl http://localhost:8080/health/live
   Failure: Kill and restart container

2. Readiness Probe: Can container handle traffic?
   Command: curl http://localhost:8080/health/ready
   Failure: Remove from load balancer

3. Startup Probe: Did startup complete?
   Command: curl http://localhost:8080/health/startup
   Failure: Kill container

apiVersion: v1
kind: Pod
metadata:
  name: order-service
spec:
  containers:
  - name: order-service
    image: order-service:latest
    livenessProbe:
      httpGet:
        path: /health/live
        port: 8080
      initialDelaySeconds: 10
      periodSeconds: 10
    readinessProbe:
      httpGet:
        path: /health/ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
*/
```

---

## Q24: How do you Implement Caching in Microservices?

### Answer:

**Caching Strategy Levels:**

```
L1: CLIENT CACHE (Browser)
    ├─ HTTP Cache headers
    └─ SessionStorage/LocalStorage

L2: CDN CACHE (Cloudflare, Azure CDN)
    ├─ Static content
    └─ Geographic distribution

L3: API GATEWAY CACHE
    ├─ Response caching
    └─ Request aggregation

L4: SERVICE CACHE (Redis, Memcached)
    ├─ Database query results
    └─ Session data

L5: DATABASE CACHE
    └─ Connection pooling
```

**C# Implementation:**

```csharp
// ========== CACHE STRATEGY PATTERNS ==========

// 1. CACHE ASIDE (Lazy Loading)
public class CacheAsidePattern
{
    private readonly IDistributedCache _cache;
    private readonly IRepository _repository;
    private readonly ILogger<CacheAsidePattern> _logger;
    
    public async Task<Product> GetProductAsync(int productId)
    {
        var cacheKey = $"product:{productId}";
        
        // Try cache first
        var cachedData = await _cache.GetStringAsync(cacheKey);
        
        if (!string.IsNullOrEmpty(cachedData))
        {
            _logger.LogInformation("Cache HIT: {Key}", cacheKey);
            return JsonConvert.DeserializeObject<Product>(
                cachedData);
        }
        
        _logger.LogInformation("Cache MISS: {Key}", cacheKey);
        
        // Get from database
        var product = await _repository.GetAsync(productId);
        
        if (product != null)
        {
            // Store in cache
            await _cache.SetStringAsync(
                cacheKey,
                JsonConvert.SerializeObject(product),
                new DistributedCacheEntryOptions
                {
                    AbsoluteExpirationRelativeToNow = 
                        TimeSpan.FromHours(1)
                });
        }
        
        return product;
    }
}

// 2. WRITE-THROUGH CACHE
public class WriteThroughCache
{
    private readonly IDistributedCache _cache;
    private readonly IRepository _repository;
    
    public async Task<int> CreateProductAsync(Product product)
    {
        // Write to database first
        var productId = await _repository
            .CreateAsync(product);
        
        product.Id = productId;
        
        // Then update cache
        await _cache.SetStringAsync(
            $"product:{productId}",
            JsonConvert.SerializeObject(product),
            new DistributedCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = 
                    TimeSpan.FromHours(1)
            });
        
        return productId;
    }
}

// 3. WRITE-BEHIND (Write-Back) CACHE
public class WriteBehindCache
{
    private readonly IDistributedCache _cache;
    private readonly IRepository _repository;
    private readonly Queue<(int id, Product product)> 
        _writeQueue;
    
    public async Task<int> CreateProductAsync(Product product)
    {
        // Write to cache only (fast)
        var productId = Guid.NewGuid().GetHashCode();
        product.Id = productId;
        
        await _cache.SetStringAsync(
            $"product:{productId}",
            JsonConvert.SerializeObject(product));
        
        // Queue write to DB (asynchronous)
        _writeQueue.Enqueue((productId, product));
        
        // Background worker processes queue
        // Benefits: Very fast writes
        // Risks: Data loss if cache fails before write
        
        return productId;
    }
    
    // Background service
    [BackgroundJob]
    public async Task ProcessWriteQueueAsync()
    {
        while (_writeQueue.TryDequeue(out var item))
        {
            try
            {
                await _repository.CreateAsync(item.product);
            }
            catch (Exception ex)
            {
                // Log and retry logic
            }
        }
    }
}

// ========== DISTRIBUTED CACHING (Redis) ==========
public class RedisCache
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddStackExchangeRedisCache(options =>
        {
            options.Configuration = 
                "redis-cache:6379";
            options.InstanceName = "order-service:";
        });
    }
}

// ========== CACHE INVALIDATION ==========
public class CacheInvalidationPatterns
{
    private readonly IDistributedCache _cache;
    private readonly IEventBus _eventBus;
    
    // 1. Time-Based (TTL)
    public async Task SetWithExpirationAsync(string key, string value)
    {
        await _cache.SetStringAsync(
            key,
            value,
            new DistributedCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = 
                    TimeSpan.FromMinutes(5),
                SlidingExpiration = TimeSpan.FromMinutes(1)
            });
    }
    
    // 2. Event-Based Invalidation
    public async Task OnProductUpdatedAsync(
        ProductUpdatedEvent @event)
    {
        // When product updated, invalidate cache
        await _cache.RemoveAsync($"product:{@event.ProductId}");
        
        // Cascade invalidation
        await _cache.RemoveAsync("products:all");
        await _cache.RemoveAsync(
            $"category:{@event.CategoryId}");
    }
    
    // 3. Manual Invalidation
    public async Task InvalidateCacheAsync(string key)
    {
        await _cache.RemoveAsync(key);
    }
    
    // 4. Pattern-Based Invalidation (Redis SCAN)
    public async Task InvalidatePatternAsync(string pattern)
    {
        var redis = ConnectionMultiplexer
            .Connect("redis-cache:6379");
        var db = redis.GetDatabase();
        
        var server = redis.GetServer(
            redis.GetEndPoints().First());
        
        foreach (var key in server.Keys(
            pattern: $"order-service:{pattern}*"))
        {
            await db.KeyDeleteAsync(key);
        }
    }
}

// ========== CACHE WARMING ==========
public class CacheWarmingService : IHostedService
{
    private readonly IDistributedCache _cache;
    private readonly IRepository _repository;
    
    public async Task StartAsync(CancellationToken token)
    {
        // Warm cache on startup
        var products = await _repository
            .GetTopProductsAsync(100);
        
        foreach (var product in products)
        {
            await _cache.SetStringAsync(
                $"product:{product.Id}",
                JsonConvert.SerializeObject(product),
                new DistributedCacheEntryOptions
                {
                    AbsoluteExpirationRelativeToNow = 
                        TimeSpan.FromHours(24)
                });
        }
    }
    
    public Task StopAsync(CancellationToken token)
    {
        return Task.CompletedTask;
    }
}

// ========== CACHE MONITORING ==========
public class CacheMetrics
{
    private int _hits;
    private int _misses;
    
    public double GetHitRatio()
    {
        var total = _hits + _misses;
        return total == 0 ? 0 : (double)_hits / total;
    }
}
```

---

## Q25: What is Message Idempotency Key and How to Use It?

### Answer:

**Already detailed in Q18 with comprehensive code examples.**

---

## Q26-50: Rapid-Fire Q&A

Let me continue with more concise Q&As:

---

## Q26: What is the Bulkhead Pattern used with?

### Answer:
```csharp
// Combines with:
// 1. Thread pool isolation
// 2. Connection pooling
// 3. Resource limits
// 4. Timeout policies

var policy = Policy.BulkheadAsync(
    maxParallelization: 10,  // Max 10 parallel
    maxQueuingActions: 20);  // Max 20 queued

// When one service fails:
// - Only its bulkhead affected
// - Other services continue
// - Prevents cascading failures
```

---

## Q27: How do you handle Timeout in Microservices?

### Answer:
```csharp
// Client timeout
var timeout = Policy
    .TimeoutAsync<HttpResponseMessage>(
        TimeSpan.FromSeconds(5));

// Server timeout
app.UseRequestTimeouts(options =>
{
    options.DefaultPolicy = new RequestTimeoutPolicy 
    { 
        Timeout = TimeSpan.FromSeconds(10) 
    };
});
```

---

## Q28: What is Bulkhead vs Circuit Breaker?

### Answer:
| Aspect | Bulkhead | Circuit Breaker |
|--------|----------|-----------------|
| Purpose | Isolate resources | Fail fast |
| Control | Threads/connections | Request flow |
| When fails | Queues requests | Blocks requests |
| Recovery | Automatic | After timeout |

---

## Q29: How do you implement Cross-Cutting Concerns?

### Answer:
```csharp
// Middleware for cross-cutting concerns
public class LoggingMiddleware
{
    public async Task InvokeAsync(HttpContext context)
    {
        // Logging
        // Correlation ID
        // Metrics
        // Authorization
    }
}
```

---

## Q30: What is Request Aggregation?

### Answer:
```csharp
// BFF aggregates multiple service calls
public class OrderBffController
{
    [HttpGet("order/{id}")]
    public async Task<OrderAggregated> GetOrder(int id)
    {
        var order = await _orderService.GetAsync(id);
        var products = await _productService
            .GetByIdsAsync(order.ProductIds);
        var customer = await _customerService
            .GetAsync(order.CustomerId);
        
        return new OrderAggregated 
        { 
            Order = order,
            Products = products,
            Customer = customer 
        };
    }
}
```

---

## Q31: How do you implement Timeout + Retry?

### Answer:
```csharp
var timeout = Policy
    .TimeoutAsync<HttpResponseMessage>(
        TimeSpan.FromSeconds(5));

var retry = Policy
    .Handle<HttpRequestException>()
    .WaitAndRetryAsync(3, 
        attempt => TimeSpan.FromSeconds(
            Math.Pow(2, attempt)));

var combined = Policy.WrapAsync(timeout, retry);
```

---

## Q32: What is Transactional Messaging?

### Answer:
```csharp
// Combine database transaction with message publishing
public async Task CreateOrderWithMessaging(Order order)
{
    using (var transaction = 
        await _context.Database.BeginTransactionAsync())
    {
        try
        {
            // Save order
            await _context.Orders.AddAsync(order);
            await _context.SaveChangesAsync();
            
            // Publish event (same transaction)
            await _eventBus.PublishAsync(
                new OrderCreatedEvent { OrderId = order.Id });
            
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

---

## Q33: What is the Outbox Pattern?

### Answer:
```
Solves: Dual-write problem (DB + Message Queue)

Problem:
Save to DB ✓ → Publish event ✗ → Lost event!

Solution:
Save to DB + Insert event in OUTBOX table (same transaction)
├─ Commit = guaranteed
└─ Separate process publishes from OUTBOX
   └─ Retry until successful
```

```csharp
public async Task CreateOrderWithOutbox(Order order)
{
    using (var transaction = 
        await _context.Database.BeginTransactionAsync())
    {
        // Save order
        await _context.Orders.AddAsync(order);
        
        // Save event to OUTBOX
        var @event = new OutboxEvent
        {
            EventType = "OrderCreated",
            AggregateId = order.Id,
            Payload = JsonConvert.SerializeObject(order),
            CreatedAt = DateTime.UtcNow,
            IsPublished = false
        };
        
        await _context.OutboxEvents.AddAsync(@event);
        await _context.SaveChangesAsync();
        await transaction.CommitAsync();
    }
    
    // Separate service publishes events
    // Even if app crashes, events will be published
    // when app restarts
}
```

---

## Q34: How do you handle Network Partitions?

### Answer:
```
CAP Theorem: Can't have Consistency + Availability + Partition tolerance

During network partition:
├─ Option 1: Reject writes (Consistency)
├─ Option 2: Accept writes, sync later (Availability)
└─ Choose based on use case
```

```csharp
public class NetworkPartitionHandler
{
    public async Task<Order> CreateOrderAsync(Order order)
    {
        try
        {
            // Try to call other services
            var available = await _inventoryService
                .CheckAsync(order.Items);
            
            if (!available)
                throw new Exception("Inventory unavailable");
        }
        catch (HttpRequestException)
        {
            // Network partition detected
            // Option: Accept order, process later
            order.Status = "Pending";
            
            // Store for later reconciliation
            await _pendingRepository.SaveAsync(order);
        }
    }
}
```

---

## Q35: What is Compensating Transaction?

### Answer:
```
Undo operations in reverse order

SAGA Compensation:
1. Reserve inventory (Success)
2. Process payment (FAILS)
   └─ Trigger compensation
3. Release inventory (Compensating Txn)

Example:
Reserve(100) → Released() // Undo
ProcessPayment(100) → Refund(100) // Undo
CreateShipment() → CancelShipment() // Undo
```

---

## Q36: How do you implement Distributed Locking?

### Answer:
```csharp
// Use Redis or database for distributed lock
public class DistributedLock
{
    private readonly IDistributedCache _cache;
    
    public async Task<bool> AcquireLockAsync(
        string lockKey, 
        string lockValue,
        TimeSpan timeout)
    {
        var acquired = await _cache.GetStringAsync(lockKey);
        
        if (acquired == null)
        {
            await _cache.SetStringAsync(
                lockKey, 
                lockValue,
                new DistributedCacheEntryOptions
                {
                    AbsoluteExpirationRelativeToNow = timeout
                });
            
            return true;
        }
        
        return false;
    }
    
    public async Task ReleaseLockAsync(string lockKey)
    {
        await _cache.RemoveAsync(lockKey);
    }
}

// Usage
if (await _lock.AcquireLockAsync("order:123", 
    Guid.NewGuid().ToString(), 
    TimeSpan.FromSeconds(5)))
{
    try
    {
        // Critical section
        await ProcessOrderAsync();
    }
    finally
    {
        await _lock.ReleaseLockAsync("order:123");
    }
}
```

---

## Q37: What is Strangler Fig Pattern Steps?

### Answer:
```
Phase 1: Create Proxy
Client → Proxy → Monolith

Phase 2: Extract Service (Auth Service)
Client → Proxy → AuthService + Monolith

Phase 3: Route Requests
Client → Proxy 
  ├→ /auth → AuthService
  └→ /orders → Monolith

Phase 4: Migrate Data
Monolith data copied to AuthService

Phase 5: Remove Old Code
Monolith Auth code removed

Phase 6: Complete
All services migrated

Benefits: Zero downtime, gradual migration
```

---

## Q38: How do you handle Versioning in Microservices?

### Answer:
```csharp
// URL Versioning
[Route("api/v1/[controller]")]
public class ProductsV1Controller { }

[Route("api/v2/[controller]")]
public class ProductsV2Controller { }

// Header Versioning
[ApiVersion("1.0")]
[Route("api/[controller]")]
public class ProductsController { }

// Query Parameter Versioning
GET /api/products?api-version=1.0

// API Gateway handles routing
{
  "Routes": [{
    "UpstreamPathTemplate": "/v1/products",
    "DownstreamPathTemplate": "/api/products-v1"
  }]
}
```

---

## Q39: What is Webhook Pattern in Microservices?

### Answer:
```csharp
// Service B registers for notifications from Service A
public class WebhookRegistration
{
    [HttpPost("register-webhook")]
    public async Task RegisterWebhookAsync(
        WebhookRequest request)
    {
        // Store callback URL
        await _webhookRepository.SaveAsync(
            new Webhook 
            { 
                Url = request.CallbackUrl,
                Event = request.EventType,
                Service = request.ServiceName 
            });
    }
}

// When event occurs, call webhook
public class OrderCreatedEventHandler
{
    public async Task OnOrderCreatedAsync(
        OrderCreatedEvent @event)
    {
        var webhooks = await _webhookRepository
            .GetByEventAsync("OrderCreated");
        
        foreach (var webhook in webhooks)
        {
            // Call webhook URL
            await _httpClient.PostAsJsonAsync(
                webhook.Url, @event);
        }
    }
}
```

---

## Q40: How do you implement Transactional Outbox?

### Answer:
```
Problem: Dual-write (save + publish)
Solution: Write both to database in transaction

Outbox Table:
┌────┬────────────┬─────────┬───────────┐
│ Id │ EventType  │ Payload │Published  │
├────┼────────────┼─────────┼───────────┤
│ 1  │ OrderCr... │ {...}   │ False     │
└────┴────────────┴─────────┴───────────┘

Service polls OUTBOX:
├─ Find unpublished events
├─ Publish to message broker
└─ Mark as published
```

---

## Q41: What is Polyglot Persistence?

### Answer:
```
Different services use different databases

Order Service → SQL (relational)
Product Service → MongoDB (document)
Cache Service → Redis (key-value)
Analytics → Elasticsearch (search)
FileStore → S3 (object storage)

Benefits:
✓ Optimal DB per use case
✓ Technology flexibility
✓ Scale independently

Challenges:
✗ Complex transactions
✗ Data consistency
```

---

## Q42: How do you implement API Versioning at Gateway?

### Answer:
```csharp
public class ApiVersioningGateway
{
    public static void Configure(IApplicationBuilder app)
    {
        var config = new FileConfiguration();
        config.Routes = new List<FileRoute>
        {
            new FileRoute
            {
                UpstreamPathTemplate = "/api/v1/products",
                DownstreamPathTemplate = "/api/products-v1",
                DownstreamHostAndPorts = new List<FileHostAndPort>
                {
                    new FileHostAndPort 
                    { 
                        Host = "product-service-v1", 
                        Port = 5001 
                    }
                }
            },
            new FileRoute
            {
                UpstreamPathTemplate = "/api/v2/products",
                DownstreamPathTemplate = "/api/products-v2",
                DownstreamHostAndPorts = new List<FileHostAndPort>
                {
                    new FileHostAndPort 
                    { 
                        Host = "product-service-v2", 
                        Port = 5001 
                    }
                }
            }
        };
    }
}
```

---

## Q43: What is Canary Deployment?

### Answer:
```
Gradual rollout to subset of users:

Version 1 → 90% traffic
Version 2 → 10% traffic (Canary)
        ↓ (Monitor)
   No errors?
        ↓ YES
Version 1 → 50% traffic
Version 2 → 50% traffic
        ↓ (Monitor)
   No errors?
        ↓ YES
Version 1 → 0% traffic
Version 2 → 100% traffic (Complete)

If errors in canary:
Quick rollback to Version 1!
```

---

## Q44: What is Blue-Green Deployment?

### Answer:
```
Two identical environments:

BLUE (Current production)
├─ 100% traffic
└─ Stable

GREEN (New version)
├─ 0% traffic
├─ Test thoroughly
└─ Deploy new version

Switch:
BLUE → 0% traffic
GREEN → 100% traffic

Instant rollback if needed!
```

---

## Q45: How do you handle Request Timeouts?

### Answer:
```csharp
public class TimeoutHandling
{
    // Connection timeout
    var httpClient = new HttpClient
    {
        Timeout = TimeSpan.FromSeconds(5)
    };
    
    // Polly timeout policy
    var timeoutPolicy = Policy
        .TimeoutAsync<HttpResponseMessage>(
            TimeSpan.FromSeconds(5),
            TimeoutStrategy.Optimistic);
    
    // Request timeout middleware
    app.UseRequestTimeouts(options =>
    {
        options.AddPolicy("DefaultTimeout",
            new RequestTimeoutPolicy 
            { 
                Timeout = TimeSpan.FromSeconds(30) 
            });
    });
}
```

---

## Q46: What is the Saga Pattern vs Distributed Transactions?

### Answer:
| Aspect | Saga | Distributed Transaction |
|--------|------|------------------------|
| Coupling | Loose | Tight |
| Consistency | Eventual | Strong |
| Rollback | Compensating | 2PC |
| Performance | Better | Slower |
| Complexity | High | Medium |
| Use Case | Async processes | Strong consistency needed |

---

## Q47: How do you implement Metrics Collection?

### Answer:
```csharp
public class MetricsCollection
{
    private readonly ILogger<MetricsCollection> _logger;
    
    public void RecordMetric(string name, 
        double value, 
        Dictionary<string, string> tags)
    {
        // Log to Application Insights
        _logger.LogInformation(
            "Metric {Metric} = {Value}, Tags = {Tags}",
            name, value, string.Join(",", tags));
        
        // Or use Prometheus
        var metric = _registry.CreateGauge(
            name, "help", tags.Keys.ToArray());
        
        metric.WithLabels(tags.Values.ToArray())
            .Set(value);
    }
}
```

---

## Q48: What is Tracing vs Logging vs Monitoring?

### Answer:
```
LOGGING:
├─ Discrete events
├─ "User login failed"
└─ Used for debugging

TRACING:
├─ Request path through services
├─ Request ID: ABC-123
│  ├─ OrderService: 100ms
│  ├─ InventoryService: 50ms
│  └─ PaymentService: 200ms
└─ Used for performance

MONITORING:
├─ Metrics + alerting
├─ CPU: 75%, Memory: 80%
├─ Error rate: 0.5%
└─ Used for SLA/health
```

---

## Q49: How do you implement Graceful Shutdown?

### Answer:
```csharp
public class GracefulShutdown
{
    public static async Task Main(string[] args)
    {
        var host = Host.CreateDefaultBuilder(args)
            .ConfigureServices(services =>
            {
                services.AddSingleton<IGracefulShutdown, 
                    GracefulShutdownService>();
            })
            .Build();
        
        var shutdown = host.Services
            .GetRequiredService<IGracefulShutdown>();
        
        await host.RunAsync();
    }
}

public class GracefulShutdownService : IHostedService
{
    private readonly ILogger<GracefulShutdownService> _logger;
    
    public async Task StartAsync(CancellationToken token)
    {
        _logger.LogInformation("Service starting");
        return Task.CompletedTask;
    }
    
    public async Task StopAsync(CancellationToken token)
    {
        // Graceful shutdown
        _logger.LogInformation("Graceful shutdown starting");
        
        // Stop accepting new requests
        // Complete in-flight requests
        // Close connections
        // Release resources
        
        await Task.Delay(5000, token); // Grace period
        
        _logger.LogInformation("Shutdown complete");
    }
}
```

---

## Q50: How do you implement Circuit Breaker State Transitions?

### Answer:

Already detailed in Q4. Here's final summary:

```csharp
public class CircuitBreakerStateMachine
{
    public enum State { Closed, Open, HalfOpen }
    
    public State CurrentState { get; private set; } = State.Closed;
    private int _failureCount = 0;
    private DateTime _lastFailureTime;
    private const int FailureThreshold = 5;
    private const int TimeoutSeconds = 60;
    
    public bool AllowRequest()
    {
        switch (CurrentState)
        {
            case State.Closed:
                return true; // Normal operation
                
            case State.Open:
                // Check if timeout expired
                if ((DateTime.UtcNow - _lastFailureTime)
                    .TotalSeconds > TimeoutSeconds)
                {
                    CurrentState = State.HalfOpen;
                    _failureCount = 0;
                    return true; // Allow test request
                }
                return false; // Reject
                
            case State.HalfOpen:
                return true; // Allow test request
        }
        return false;
    }
    
    public void RecordSuccess()
    {
        _failureCount = 0;
        
        if (CurrentState == State.HalfOpen)
        {
            CurrentState = State.Closed; // Recovered!
        }
    }
    
    public void RecordFailure()
    {
        _failureCount++;
        _lastFailureTime = DateTime.UtcNow;
        
        if (_failureCount >= FailureThreshold)
        {
            CurrentState = State.Open;
        }
    }
}
```

---

## Final Summary Table

| Pattern | Problem | Solution |
|---------|---------|----------|
| Q1 | Monolith issues | Microservices |
| Q2 | When to migrate | Decision matrix |
| Q3 | Communication | Sync vs Async |
| Q4 | Cascading failures | Circuit Breaker |
| Q5 | Distributed txns | Saga pattern |
| Q6 | Multiple services | API Gateway |
| Q7 | Data coupling | Database per service |
| Q8 | Dynamic IPs | Service discovery |
| Q9 | Immediate consistency | Eventual consistency |
| Q10 | Orchestration | Choreography |
| Q11 | Different clients | BFF |
| Q12 | Distributed tracing | Correlation ID |
| Q13 | Security | JWT + OAuth2 |
| Q14 | System design | E-commerce architecture |
| Q15 | Failures | Resilience patterns |

---

