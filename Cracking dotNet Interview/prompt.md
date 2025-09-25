I've created a comprehensive technical interview preparation template for senior/principal architect positions. This template covers all the areas you requested with detailed preparation prompts that will help you develop interview-ready responses.

The template is structured to help you prepare for the depth and breadth that senior architect interviews typically explore. Each section includes:

1. **Detailed preparation prompts** that cover both theoretical knowledge and practical implementation
2. **Key interview questions** to practice with
3. **Real-world scenarios** that test architectural thinking
4. **Trade-offs and decision-making frameworks** that interviewers look for

The template emphasizes not just knowing the concepts, but being able to:
- Explain the "why" behind architectural decisions
- Discuss trade-offs and alternatives
- Provide real-world implementation details
- Address non-functional requirements (performance, security, maintainability)
- Demonstrate systems thinking and business impact awareness

You can use each section's preparation prompt to either study independently or ask me to elaborate on any specific topic. The template is designed to help you think like a senior architect who can balance technical excellence with business needs and practical constraints.

# Senior/Principal Architect Technical Interview Preparation Template

## Instructions for Use
Use this comprehensive prompt template to prepare detailed, interview-ready responses for senior/principal architect positions. Each section includes key concepts, practical scenarios, and common interview questions that assess architectural thinking, decision-making, and real-world problem-solving abilities.

---

## 1. ARCHITECTURAL DESIGN

### 1.1 Layered Architecture

**Preparation Prompt:**
```
Hi, I am a .NET resource of having 16+ years of experience. I am preparing for technical interview on .NET. 
Explain layered architecture in detail covering:
- Definition and core principles of layered architecture
- Common layers (Presentation, Business/Application, Data Access, Database) and their responsibilities
- Advantages: separation of concerns, maintainability, testability, team specialization
- Disadvantages: performance overhead, potential for tight coupling, complexity in cross-cutting concerns
- When to use layered architecture vs alternatives (hexagonal, clean architecture)
- Real-world implementation challenges and solutions
- How to handle cross-cutting concerns (logging, security, caching) in layered architecture
- Design patterns commonly used within each layer
- Evolution from traditional N-tier to modern layered approaches
Please get me the details in solution-oriented architect approach, include more explanation and keep the code short and simple. 
```

**Key Interview Questions to Prepare:**
- How would you design a layered architecture for an e-commerce platform?
- What are the trade-offs between strict layering vs relaxed layering?
- How do you prevent the anemic domain model anti-pattern in layered architecture?

### 1.2 API Design Patterns

#### MVC Pattern
**Preparation Prompt:**
```
Hi, I am a .NET resource of having 16+ years of experience. I am preparing for technical interview on .NET. 
Provide comprehensive coverage of MVC pattern including:
- Model-View-Controller separation and responsibilities
- Variations: MVP (Model-View-Presenter), MVVM (Model-View-ViewModel)
- Implementation in different frameworks (Spring MVC, ASP.NET MVC, Express.js)
- How MVC fits into modern web applications and SPAs
- Advantages: separation of concerns, testability, parallel development
- Disadvantages: complexity for simple applications, potential for fat controllers
- Best practices for keeping controllers thin and models rich
- Handling of cross-cutting concerns in MVC
- MVC in microservices architecture
Please get me the details in solution-oriented architect approach, include more explanation and keep the code short and simple. 
```

#### REST / SOAP / RESTful Architecture
**Preparation Prompt:**
```
Hi, I am a .NET resource of having 16+ years of experience. I am preparing for technical interview on .NET. 
Explain web service architectures comprehensively:

**REST:**
- REST principles: stateless, cacheable, uniform interface, layered system, client-server, HATEOAS
- HTTP methods and their proper usage (GET, POST, PUT, DELETE, PATCH)
- Resource identification and URI design best practices
- Status codes and their appropriate usage
- Content negotiation and media types
- RESTful API design patterns and anti-patterns
- Richardson Maturity Model (levels 0-3)

**SOAP:**
- SOAP protocol structure and XML-based messaging
- WSDL (Web Services Description Language) and service contracts
- WS-* standards (WS-Security, WS-ReliableMessaging, WS-Transaction)
- When to choose SOAP over REST (enterprise integration, formal contracts)
- SOAP vs REST comparison in terms of performance, complexity, tooling

**RESTful Architecture:**
- Designing RESTful services at scale
- Resource modeling and hierarchical relationships
- Handling complex operations that don't fit CRUD model
- Bulk operations and batch processing in REST
- Error handling and consistent response formats
- API documentation and discoverability
Please get me the details in solution-oriented architect approach, include more explanation and keep the code short and simple. 
```

#### API Versioning
**Preparation Prompt:**
```
Hi, I am a .NET resource of having 16+ years of experience. I am preparing for technical interview on .NET. 
Cover API versioning strategies comprehensively:
- Why API versioning is necessary (breaking changes, backward compatibility)
- Versioning strategies:
  * URI versioning (/v1/users, /v2/users)
  * Header versioning (Accept: application/vnd.api+json;version=1)
  * Query parameter versioning (?version=1)
  * Content negotiation versioning
- Semantic versioning for APIs (major.minor.patch)
- Deprecation strategies and sunset policies
- Managing multiple API versions in production
- Database schema evolution with API versioning
- Client migration strategies
- Tools and frameworks for API version management
- GraphQL approach to API evolution
- Contract testing across API versions
Please get me the details in solution-oriented architect approach, include more explanation and keep the code short and simple. 
```

---

## 2. SCENARIO-BASED QUESTIONS

**Preparation Prompt:**
```
Hi, I am a .NET resource of having 16+ years of experience. I am preparing for technical interview on .NET. 
Prepare detailed architectural solutions for these common scenarios:

**E-commerce Platform:**
- Design a scalable e-commerce system handling millions of products and users
- Address: product catalog, inventory management, order processing, payment integration
- Consider: peak traffic handling, data consistency, search functionality, recommendation engine

**Real-time Chat Application:**
- Architecture for supporting millions of concurrent users
- WebSocket vs Server-Sent Events vs HTTP polling trade-offs
- Message delivery guarantees and offline message handling
- Horizontal scaling of real-time components

**Content Management System:**
- Multi-tenant CMS with customizable workflows
- Content versioning and audit trails
- Performance optimization for content delivery
- Integration with CDN and caching strategies

**Financial Trading System:**
- Low-latency, high-throughput trading platform
- Risk management and compliance integration
- Real-time market data processing
- Disaster recovery and business continuity

**IoT Data Processing Platform:**
- Ingesting and processing data from millions of IoT devices
- Stream processing vs batch processing decisions
- Data lake vs data warehouse architecture
- Real-time analytics and alerting

For each scenario, address:
- System requirements and constraints
- High-level architecture diagrams
- Technology stack recommendations
- Scalability and performance considerations
- Security implications
- Data modeling approach
- Integration patterns
- Monitoring and observability
- Cost optimization strategies
Please get me the details in solution-oriented architect approach, include more explanation and keep the code short and simple. 
```

---

## 3. PERFORMANCE & SCALABILITY

### 3.1 Caching Strategy

**Preparation Prompt:**
```
Hi, I am a .NET resource of having 16+ years of experience. I am preparing for technical interview on .NET. 
Provide comprehensive caching strategy knowledge:

**Caching Levels:**
- Browser caching (HTTP headers, service workers)
- CDN caching (edge locations, cache invalidation)
- Reverse proxy caching (Nginx, Varnish)
- Application-level caching (in-memory, distributed)
- Database caching (query result caching, buffer pools)

**Caching Patterns:**
- Cache-aside (lazy loading)
- Write-through caching
- Write-behind (write-back) caching
- Refresh-ahead caching
- Cache invalidation strategies (TTL, manual, event-driven)

**Distributed Caching:**
- Redis vs Memcached comparison
- Cache partitioning and sharding
- Cache coherence in distributed systems
- Cache warming strategies
- Handling cache failures and fallbacks

**Cache Design Considerations:**
- What to cache vs what not to cache
- Cache key design and naming conventions
- Cache hit ratio optimization
- Memory management and eviction policies (LRU, LFU, FIFO)
- Cache monitoring and metrics
- Hot key problems and solutions
- Cache stampede prevention
Please get me the details in solution-oriented architect approach, include more explanation and keep the code short and simple. 
```

### 3.2 Database Performance

**Preparation Prompt:**
```
Hi, I am a .NET resource of having 16+ years of experience. I am preparing for technical interview on .NET. 
Cover database performance optimization comprehensively:

**Indexing Strategies:**
- B-tree, hash, bitmap, and full-text indexes
- Composite indexes and index order importance
- Partial and filtered indexes
- Index maintenance overhead and trade-offs
- Query execution plan analysis

**Query Optimization:**
- SQL query optimization techniques
- Query rewriting and restructuring
- Subquery vs JOIN performance
- Understanding and reading execution plans
- Statistics and cardinality estimation

**Database Design:**
- Normalization vs denormalization trade-offs
- Partitioning strategies (horizontal, vertical, functional)
- Sharding patterns and challenges
- Read replicas and write scaling
- ACID properties vs performance trade-offs

**NoSQL Performance:**
- Document database optimization (MongoDB)
- Key-value store performance (Redis, DynamoDB)
- Column-family optimization (Cassandra, HBase)
- Graph database performance (Neo4j)
- Eventual consistency implications

**Connection Management:**
- Connection pooling best practices
- Connection pool sizing and configuration
- Prepared statements and query caching
- Database connection monitoring
- Handling connection failures and retries
Please get me the details in solution-oriented architect approach, include more explanation and keep the code short and simple. 
```

### 3.3 Microservices Communication

**Preparation Prompt:**
```
Hi, I am a .NET resource of having 16+ years of experience. I am preparing for technical interview on .NET. 
Explain microservices communication patterns in detail:

**Synchronous Communication:**
- HTTP/REST API communication
- gRPC and Protocol Buffers
- Service discovery mechanisms (Consul, Eureka, Kubernetes DNS)
- Load balancing strategies (round-robin, weighted, circuit breaker)
- Timeout and retry configurations
- Service mesh architecture (Istio, Linkerd)

**Asynchronous Communication:**
- Message queues (RabbitMQ, Apache Kafka, Amazon SQS)
- Event-driven architecture patterns
- Publish-subscribe vs point-to-point messaging
- Message ordering and partitioning
- Dead letter queues and error handling
- Event sourcing and CQRS patterns

**Data Consistency:**
- Distributed transactions and two-phase commit
- Saga pattern for managing distributed transactions
- Eventual consistency patterns
- Compensating actions and rollback strategies
- Event choreography vs orchestration

**Performance Considerations:**
- Network latency optimization
- Batch processing and bulk operations
- Connection multiplexing and HTTP/2
- Compression and serialization formats
- Monitoring inter-service communication
- Distributed tracing and observability
Please get me the details in solution-oriented architect approach, include more explanation and keep the code short and simple. 
```

---

## 4. ERROR HANDLING & RESILIENCE

### 4.1 Global Error Handling

**Preparation Prompt:**
```
Hi, I am a .NET resource of having 16+ years of experience. I am preparing for technical interview on .NET. 
Cover comprehensive error handling strategies:

**Error Classification:**
- System errors vs business errors
- Recoverable vs non-recoverable errors
- Transient vs permanent failures
- User errors vs application errors

**Global Exception Handling:**
- Centralized error handling middleware
- Error propagation in layered architectures
- Context preservation during error handling
- Error logging and correlation IDs
- Error response standardization

**Error Response Design:**
- HTTP status code usage guidelines
- Error response payload structure
- Localization and internationalization of error messages
- Error codes and error catalogs
- Client-friendly error messages

**Logging and Monitoring:**
- Structured logging for errors
- Error rate monitoring and alerting
- Error aggregation and analysis
- Integration with APM tools
- Error reporting to external systems

**Resilience Patterns:**
- Graceful degradation strategies
- Fallback mechanisms
- Error boundaries in component architecture
- Health checks and readiness probes
- Chaos engineering principles
Please get me the details in solution-oriented architect approach, include more explanation and keep the code short and simple. 
```

### 4.2 Retry Pattern

**Preparation Prompt:**
```
Hi, I am a .NET resource of having 16+ years of experience. I am preparing for technical interview on .NET. 
Explain retry patterns comprehensively:

**Retry Strategies:**
- Fixed interval retry
- Exponential backoff with jitter
- Linear backoff
- Custom backoff strategies
- Maximum retry limits and timeout configuration

**When to Retry:**
- Identifying transient vs permanent failures
- HTTP status codes suitable for retry (5xx, specific 4xx)
- Database connection failures and timeouts
- Network-related errors
- Third-party service failures

**Retry Implementation:**
- Implementing retry logic in different languages/frameworks
- Async retry patterns
- Retry with circuit breaker combination
- Distributed retry coordination
- Retry state management

**Advanced Patterns:**
- Bulkhead pattern for isolating failures
- Circuit breaker pattern integration
- Retry storm prevention
- Backpressure handling
- Dead letter queue integration

**Monitoring and Observability:**
- Retry metrics and monitoring
- Retry attempt logging
- Success rate tracking after retries
- Performance impact of retries
- Cost implications of retry strategies
Please get me the details in solution-oriented architect approach, include more explanation and keep the code short and simple. 
```

### 4.3 Validation Patterns

**Preparation Prompt:**
```
Hi, I am a .NET resource of having 16+ years of experience. I am preparing for technical interview on .NET. 
Cover validation patterns at different architectural levels:

**Input Validation:**
- Client-side vs server-side validation
- Data type validation, format validation, business rule validation
- Sanitization vs validation
- Validation frameworks and libraries
- Custom validation rules and composability

**API Validation:**
- Request payload validation
- Parameter validation (path, query, header)
- Content-type validation
- Rate limiting and throttling
- API key and authentication validation

**Business Logic Validation:**
- Domain model validation
- Cross-field validation rules
- Validation in aggregates and entities
- Validation error collection and reporting
- Conditional validation rules

**Data Layer Validation:**
- Database constraints (NOT NULL, UNIQUE, CHECK)
- Trigger-based validation
- Stored procedure validation
- Data integrity validation
- Migration validation strategies

**Distributed Validation:**
- Validation in microservices architecture
- Shared validation rules across services
- Validation result caching
- Validation service patterns
- Event-driven validation
Please get me the details in solution-oriented architect approach, include more explanation and keep the code short and simple. 
```

---

## 5. SECURITY PATTERNS

### 5.1 Authentication & Authorization

**Preparation Prompt:**
```
Hi, I am a .NET resource of having 16+ years of experience. I am preparing for technical interview on .NET. 
Provide comprehensive security authentication and authorization coverage:

**Authentication Methods:**
- Username/password authentication
- Multi-factor authentication (MFA, 2FA)
- Certificate-based authentication
- Biometric authentication
- Social authentication (OAuth providers)
- Single Sign-On (SSO) patterns

**Token-based Authentication:**
- JWT (JSON Web Tokens) structure and validation
- Access tokens vs refresh tokens
- Token expiration and renewal strategies
- Token storage security (httpOnly cookies vs localStorage)
- Token revocation mechanisms
- Stateless vs stateful token approaches

**Authorization Patterns:**
- Role-Based Access Control (RBAC)
- Attribute-Based Access Control (ABAC)
- Permission-based authorization
- Resource-based authorization
- Claims-based authorization
- Hierarchical role structures

**Enterprise Integration:**
- Active Directory integration
- LDAP authentication
- SAML 2.0 for enterprise SSO
- OpenID Connect implementation
- Kerberos authentication
- Federation and trust relationships

**Security Best Practices:**
- Password policies and hashing (bcrypt, Argon2)
- Account lockout and brute force protection
- Session management and security
- Cross-site request forgery (CSRF) protection
- Audit logging for authentication events
Please get me the details in solution-oriented architect approach, include more explanation and keep the code short and simple. 
```

### 5.2 API Security

**Preparation Prompt:**
```
Hi, I am a .NET resource of having 16+ years of experience. I am preparing for technical interview on .NET. 
Cover API security comprehensively:

**API Authentication:**
- API key management and rotation
- OAuth 2.0 flows (authorization code, client credentials, implicit)
- Bearer token authentication
- Mutual TLS (mTLS) authentication
- HMAC signature-based authentication
- API gateway authentication patterns

**Input Security:**
- Input validation and sanitization
- SQL injection prevention
- Cross-site scripting (XSS) prevention
- Command injection prevention
- File upload security
- JSON/XML parsing security

**Transport Security:**
- HTTPS/TLS configuration
- Certificate management and rotation
- HTTP security headers (HSTS, CSP, X-Frame-Options)
- API versioning security implications
- CORS configuration and security

**Rate Limiting and Throttling:**
- Request rate limiting strategies
- IP-based vs user-based throttling
- Distributed rate limiting
- DDoS protection patterns
- Resource quota management
- Burst handling and token bucket algorithms

**API Security Monitoring:**
- API abuse detection
- Anomaly detection in API usage
- Security logging and SIEM integration
- API security testing (OWASP ZAP, Burp Suite)
- Vulnerability scanning and assessment
- Security incident response for APIs
Please get me the details in solution-oriented architect approach, include more explanation and keep the code short and simple. 
```

---

## 6. TESTING & MAINTAINABILITY

### 6.1 Testable Design

**Preparation Prompt:**
```
Hi, I am a .NET resource of having 16+ years of experience. I am preparing for technical interview on .NET. 
Explain testable architecture design principles:

**Dependency Injection:**
- Constructor injection vs property injection
- Service locator pattern vs dependency injection
- IoC container configuration and lifecycle management
- Testing with mock dependencies
- Circular dependency resolution

**Test Pyramid Strategy:**
- Unit tests: scope, isolation, and mocking strategies
- Integration tests: database, external services, component integration
- End-to-end tests: user journey testing, browser automation
- Contract tests: API contract verification, schema validation
- Performance tests: load testing, stress testing integration

**Design for Testability:**
- SOLID principles application for testing
- Avoiding static dependencies and globals
- Pure functions and immutable objects
- Test-friendly error handling
- Database testing strategies (in-memory, test containers)

**Testing Patterns:**
- Arrange-Act-Assert (AAA) pattern
- Given-When-Then (BDD) pattern
- Test fixtures and data builders
- Page Object Model for UI testing
- Test data management and cleanup

**Mocking and Stubbing:**
- When to mock vs when to use real implementations
- Creating maintainable mocks
- Verifying interactions vs state
- Testing external service integrations
- Database mocking strategies
Please get me the details in solution-oriented architect approach, include more explanation and keep the code short and simple. 
```

### 6.2 Configuration Management

**Preparation Prompt:**
```
Hi, I am a .NET resource of having 16+ years of experience. I am preparing for technical interview on .NET. 
Cover configuration management comprehensively:

**Configuration Sources:**
- Environment variables
- Configuration files (JSON, YAML, XML, Properties)
- Database configuration storage
- External configuration services (Consul, etcd)
- Command-line arguments
- Cloud provider configuration services

**Environment-specific Configuration:**
- Development, staging, production configuration separation
- Configuration promotion pipelines
- Environment variable hierarchy and overrides
- Secrets management (passwords, API keys, certificates)
- Configuration validation and type safety

**Dynamic Configuration:**
- Hot reload and configuration refresh
- Feature flags and toggles implementation
- A/B testing configuration
- Circuit breaker configuration
- Rate limiting configuration updates

**Security in Configuration:**
- Configuration encryption at rest and in transit
- Secret rotation strategies
- Access control for configuration changes
- Configuration audit trails
- Separation of sensitive and non-sensitive configuration

**Configuration Architecture:**
- Centralized vs distributed configuration
- Configuration service patterns
- Configuration caching strategies
- Configuration change propagation
- Rollback strategies for configuration changes
Please get me the details in solution-oriented architect approach, include more explanation and keep the code short and simple. 
```

---

## 7. ADVANCED SCENARIOS

### 7.1 Event Sourcing

**Preparation Prompt:**
```
Hi, I am a .NET resource of having 16+ years of experience. I am preparing for technical interview on .NET. 
Explain event sourcing pattern comprehensively:

**Core Concepts:**
- Event sourcing definition and principles
- Events vs commands vs queries
- Event store design and implementation
- Event serialization and versioning
- Aggregate root and event sourcing relationship

**Event Store Implementation:**
- Event stream storage strategies
- Event ordering and concurrent writes
- Event store performance optimization
- Event store scaling patterns
- Snapshot strategies for performance

**CQRS Integration:**
- Command Query Responsibility Segregation with event sourcing
- Read model projections and view materialization
- Eventual consistency between command and query sides
- Multiple read models from single event stream
- Projection rebuilding strategies

**Event Versioning:**
- Schema evolution strategies
- Backward compatibility maintenance
- Event upcasting and downcasting
- Migration strategies for event format changes
- Handling deprecated events

**Challenges and Solutions:**
- Complex business logic reconstruction
- Event replay performance issues
- External system integration with event sourcing
- Audit and compliance with event sourcing
- Testing strategies for event-sourced systems
- When not to use event sourcing
Please get me the details in solution-oriented architect approach, include more explanation and keep the code short and simple. 
```

### 7.2 Multi-tenant Architecture

**Preparation Prompt:**
```
Hi, I am a .NET resource of having 16+ years of experience. I am preparing for technical interview on .NET. 
Cover multi-tenancy patterns in detail:

**Tenancy Models:**
- Single-tenant architecture (dedicated instances)
- Multi-tenant with shared application, shared database
- Multi-tenant with shared application, separate databases
- Multi-tenant with separate applications and databases
- Hybrid tenancy models

**Data Isolation Strategies:**
- Row-level security and tenant ID filtering
- Schema-based separation (separate schemas per tenant)
- Database-based separation (separate databases per tenant)
- Encryption-based data isolation
- Data residency and compliance requirements

**Application-level Multi-tenancy:**
- Tenant context management and propagation
- Tenant-specific configuration and customization
- Feature flags per tenant
- Tenant-specific business logic
- Resource quotas and billing per tenant

**Scaling Multi-tenant Systems:**
- Tenant-aware load balancing
- Resource allocation per tenant
- Performance isolation between tenants
- Horizontal scaling strategies
- Tenant migration strategies

**Security in Multi-tenancy:**
- Tenant data leak prevention
- Cross-tenant security testing
- Tenant authentication and authorization
- Audit trails per tenant
- Compliance and data governance
Please get me the details in solution-oriented architect approach, include more explanation and keep the code short and simple. 
```

### 7.3 API Gateway Pattern

**Preparation Prompt:**
```
Hi, I am a .NET resource of having 16+ years of experience. I am preparing for technical interview on .NET. 
Explain API Gateway architecture comprehensively:

**Core Functions:**
- Request routing and load balancing
- Authentication and authorization
- Rate limiting and throttling
- Request/response transformation
- Protocol translation (HTTP to gRPC, REST to GraphQL)
- API composition and aggregation

**Implementation Patterns:**
- Backend for Frontend (BFF) pattern
- Micro-gateway vs centralized gateway
- API gateway clustering and high availability
- Gateway-per-service vs single gateway
- Service mesh integration with API gateway

**Advanced Features:**
- Circuit breaker integration
- Caching at gateway level
- Request/response logging and monitoring
- A/B testing and canary deployments
- API versioning through gateway
- WebSocket and Server-Sent Events handling

**Performance Considerations:**
- Gateway performance optimization
- Connection pooling and keep-alive
- Compression and content optimization
- Edge deployment strategies
- CDN integration with API gateway
- Latency minimization techniques

**Operational Aspects:**
- API gateway monitoring and observability
- Configuration management for gateways
- Security scanning and vulnerability management
- Gateway deployment strategies
- Disaster recovery for API gateway
- Cost optimization in cloud environments

**Popular Implementations:**
- Kong, Zuul, Ambassador, Istio Gateway
- Cloud-managed gateways (AWS API Gateway, Azure API Management)
- Implementation trade-offs and selection criteria
Please get me the details in solution-oriented architect approach, include more explanation and keep the code short and simple. 
```

---

## Interview Preparation Strategy

1. **For each topic above, prepare:**
   - 2-3 real-world examples from your experience
   - Detailed technical implementation approach
   - Trade-offs and alternative solutions
   - Performance and scalability implications
   - Security considerations
   - Monitoring and operational aspects

2. **Practice articulating:**
   - High-level architecture diagrams
   - Step-by-step implementation approach
   - Decision-making rationale
   - Risk mitigation strategies
   - Technology stack recommendations

3. **Be ready to dive deep into:**
   - Specific technology implementations
   - Performance bottlenecks and solutions
   - Security vulnerabilities and mitigations
   - Cost optimization strategies
   - Team and organizational impact of architectural decisions


Would you like me to dive deeper into any specific section or help you practice with particular scenarios?

---

Please create a comprehensive prompt tempate for preparing technical interview for senior architect / solution architect on `Architectural design`, `Scenario-based questions`, `Performance & Scalability`, `Error Handling & Resilience`, `Security Patterns`, `Testing & Maintainability`, and `Advanced Scenarios`. 
Under `Architectural design`, I would like to include:
1. Layered Architecture
2. API Design Patterns(MVC Pattern, REST / SOAP / RESTFul Architecture, and API Versioning)
Under `Performance & Scalability`, I would like to include: 
1. Caching Strategy
2. Database Performance
3. Microservices Communication
Under `Error Handling & Resilience`, I would like to include: 
1. Global Error Handling
2. Retry Pattern
3. Validation Patterns
Under `Security Patterns`, I would like to include: 
1. Authentication & Authorization
2. API Security
Under `Testing & Maintainability`, I would like to include: 
1. Testable Design
2. Configuration Management
Under `Advanced Scenarios`, I would like to include: 
1. Event Sourcing
2. Multi-tenant Architecture
3. API Gateway Pattern
The prompt template should help me to get the most detailed and interview-ready information on each topic. Structure the template to cover all aspects that that senior/principal architect interviews typically explore.

---

Hi, I’m a .NET developer with over 16 years of experience primarily working with C#, .NET, .NET Core, Web APIs, and Azure. I’m currently preparing for an online screening test that comes before a technical interview for a senior individual contributor role in the banking domain. I'm not sure what to expect from the screening test. Could you please shed some light on the types of questions that are typically asked, so I can prepare accordingly?

