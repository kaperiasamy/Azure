# 50 Advanced C# Interview Questions for Tech Lead/Solution Architect

## .NET Framework & Core Questions

### 1. Explain the difference between .NET Framework and .NET Core architecture
**Answer:** .NET Framework is Windows-only and monolithic, while .NET Core (now .NET 5+) is cross-platform, modular, and uses CoreCLR. .NET Core has better performance, supports side-by-side versioning, and uses a package-based deployment model.

**Explanation:** .NET Core was rebuilt from scratch to be cloud-optimized with smaller footprint, removing legacy dependencies and Windows-specific components like WPF initially.

**Follow-up:** How does the unified .NET 5+ platform combine both frameworks? What migration challenges have you faced moving from Framework to Core?

### 2. How does the CLR handle memory management for value types vs reference types?
**Answer:** Value types are allocated on the stack (when local variables) or inline within objects on the heap. Reference types are always allocated on the heap. The GC manages heap memory but not stack memory, which is automatically reclaimed when methods exit.

**Explanation:** This distinction affects performance - stack allocation is faster but limited in size. Boxing/unboxing occurs when converting between value and reference types.

**Follow-up:** What is boxing overhead? How can you minimize boxing in generic collections?

### 3. Describe the three generations in .NET Garbage Collection
**Answer:** Gen 0 (short-lived objects, collected frequently), Gen 1 (buffer between Gen 0 and Gen 2), Gen 2 (long-lived objects, collected rarely). Objects surviving collection are promoted to the next generation.

**Explanation:** This generational approach optimizes GC performance since most objects die young. Large objects (>85KB) go directly to Large Object Heap (LOH).

**Follow-up:** What triggers a Gen 2 collection? How does the LOH affect application performance?

### 4. What is an AppDomain and how does it provide isolation?
**Answer:** AppDomain is a logical isolation boundary within a process that provides security, versioning, and reliability isolation. Each AppDomain has its own assemblies, security context, and configuration.

**Explanation:** AppDomains enable running multiple applications in a single process with fault isolation - if one crashes, others continue running.

**Follow-up:** How do AppDomains differ in .NET Core? What replaced them for isolation?

### 5. Explain strong-named assemblies and their benefits
**Answer:** Strong-named assemblies have a unique identity using public key cryptography, including name, version, culture, and public key token. They prevent assembly spoofing and enable side-by-side versioning in GAC.

**Explanation:** Strong naming ensures assembly integrity and enables the runtime to enforce version binding policies.

**Follow-up:** What is delay signing? When would you use it in development?

## C# Language Advanced Questions

### 6. How do covariance and contravariance work in C# generics?
**Answer:** Covariance (out keyword) allows using a more derived type than specified (IEnumerable<Derived> to IEnumerable<Base>). Contravariance (in keyword) allows using a less derived type (Action<Base> to Action<Derived>).

**Explanation:** This enables more flexible type relationships while maintaining type safety. Only works with interfaces and delegates, not classes.

**Follow-up:** Why can't we have covariance with List<T>? Give an example where contravariance is useful.

### 7. Explain the difference between Task.Run and Task.Factory.StartNew
**Answer:** Task.Run is simplified API for CPU-bound work using default scheduler and options. Task.Factory.StartNew offers more control over creation options, scheduler, and cancellation but requires careful configuration.

**Explanation:** Task.Run internally calls Task.Factory.StartNew with safe defaults. StartNew with LongRunning option creates a dedicated thread.

**Follow-up:** When would you choose StartNew over Run? What are the dangers of using StartNew incorrectly?

### 8. How does async state machine transformation work?
**Answer:** Compiler transforms async methods into state machines implementing IAsyncStateMachine. Each await point becomes a state, with continuations scheduled when tasks complete.

**Explanation:** The state machine tracks local variables, current state, and manages continuation scheduling through the SynchronizationContext or TaskScheduler.

**Follow-up:** What is the performance cost of async methods? How does ValueTask optimize this?

### 9. Explain memory management differences between Span<T> and Memory<T>
**Answer:** Span<T> is a ref struct (stack-only) providing zero-cost abstraction over contiguous memory. Memory<T> is a regular struct that can be stored on heap and represents a lease to memory that can create Spans.

**Explanation:** Span<T> enables high-performance scenarios without allocations but can't be stored in fields or async methods. Memory<T> bridges this gap.

**Follow-up:** When would you use ReadOnlySpan<T>? How does stackalloc work with Span?

### 10. What are expression trees and how do they enable LINQ providers?
**Answer:** Expression trees represent code as data structures that can be examined and transformed at runtime. LINQ providers like EF Core traverse these trees to generate SQL or other query languages.

**Explanation:** Expression<Func<T>> captures lambda structure rather than compiled delegate, enabling runtime analysis and translation.

**Follow-up:** How do you build expression trees dynamically? What's the performance difference between compiled expressions and regular delegates?

### 11. Describe the difference between volatile, lock, and Interlocked
**Answer:** Volatile ensures reads/writes aren't optimized away and provides acquire-release semantics. Lock provides mutual exclusion. Interlocked provides atomic operations without locks.

**Explanation:** Volatile doesn't provide atomicity, just visibility. Interlocked is lock-free and faster for simple operations. Lock is needed for complex critical sections.

**Follow-up:** When is volatile insufficient? How do memory barriers work in .NET?

### 12. How do ConfigureAwait(false) and SynchronizationContext interact?
**Answer:** ConfigureAwait(false) prevents capturing the current SynchronizationContext, allowing continuation on any thread pool thread. This avoids deadlocks and improves performance in library code.

**Explanation:** UI frameworks post continuations back to UI thread via SynchronizationContext. ConfigureAwait(false) bypasses this, but loses thread affinity.

**Follow-up:** When should library code use ConfigureAwait(false)? What happens with nested async calls?

### 13. Explain weak references and when to use them
**Answer:** WeakReference allows holding a reference to an object without preventing garbage collection. Useful for caches where memory pressure should allow collection.

**Explanation:** The GC can collect weakly referenced objects during memory pressure. You must check if the target is still alive before use.

**Follow-up:** What's the difference between short and long weak references? How do they interact with resurrection?

### 14. How does pattern matching enhance switch statements in C# 8+?
**Answer:** Pattern matching allows switching on types, properties, tuples, and complex conditions. Switch expressions provide concise syntax with exhaustiveness checking.

**Explanation:** Patterns include type patterns, property patterns, positional patterns, and relational patterns, enabling powerful conditional logic.

**Follow-up:** How do you handle exhaustiveness in switch expressions? What are recursive patterns?

### 15. Explain the purpose and implementation of IAsyncEnumerable
**Answer:** IAsyncEnumerable enables asynchronous iteration over data streams, yielding items as they become available. Combines async/await with iterator patterns.

**Explanation:** Useful for processing data streams, pagination, or real-time data without blocking or loading everything into memory.

**Follow-up:** How does cancellation work with IAsyncEnumerable? What's the difference from Observable patterns?

## SQL & Database Questions

### 16. What's the difference between clustered and non-clustered indexes?
**Answer:** Clustered index determines physical data order (one per table), while non-clustered indexes are separate structures with pointers to data (multiple allowed).

**Explanation:** Clustered index IS the table data. Non-clustered indexes contain key values and row locators (either clustered key or RID).

**Follow-up:** When would you use included columns in an index? What is index fragmentation?

### 17. Explain isolation levels and their trade-offs
**Answer:** Read Uncommitted (dirty reads), Read Committed (default, no dirty reads), Repeatable Read (no phantom reads in row), Serializable (no phantom reads), Snapshot (versioning-based).

**Explanation:** Higher isolation reduces concurrency but prevents anomalies. Snapshot isolation uses row versioning to avoid locks.

**Follow-up:** What is the difference between pessimistic and optimistic concurrency? How does RCSI work?

### 18. How do execution plans help optimize queries?
**Answer:** Execution plans show how SQL Server executes queries, including join strategies, index usage, and cost estimates. Actual plans show runtime statistics.

**Explanation:** Plans reveal performance issues like table scans, missing indexes, or inefficient joins. Statistics drive plan selection.

**Follow-up:** What causes parameter sniffing issues? How do you force plan recompilation?

### 19. Describe the differences between INNER, LEFT, RIGHT, and CROSS joins
**Answer:** INNER returns matching rows, LEFT includes all left table rows, RIGHT includes all right table rows, CROSS produces Cartesian product.

**Explanation:** Join type affects result set size and null handling. Performance varies based on indexes and data distribution.

**Follow-up:** When would you use CROSS APPLY vs LEFT JOIN? What is a hash join vs nested loop join?

### 20. What are CTEs and how do they differ from temp tables?
**Answer:** Common Table Expressions are named temporary result sets within a query scope. Temp tables are physical structures in tempdb with statistics and indexes.

**Explanation:** CTEs are readable and support recursion but don't persist. Temp tables can be indexed and reused but require cleanup.

**Follow-up:** When do recursive CTEs cause performance issues? What are table variables vs temp tables?

## Web API Questions

### 21. Explain content negotiation in Web API
**Answer:** Content negotiation selects response format based on Accept header, using MediaTypeFormatters. Supports JSON, XML, and custom formats.

**Explanation:** Client specifies preferred formats via Accept header. Server selects best match or returns 406 if unsupported.

**Follow-up:** How do you create custom media type formatters? What is the role of content type in request/response?

### 22. How does Web API routing differ from MVC routing?
**Answer:** Web API uses HTTP verb-based routing by default, matching action names to verbs. Supports attribute routing for fine control.

**Explanation:** Convention-based routing maps HTTP methods to action prefixes (GetXxx, PostXxx). Attribute routing provides explicit control.

**Follow-up:** How do route constraints work? What is route ordering precedence?

### 23. Describe OAuth 2.0 flow implementation in Web API
**Answer:** OAuth 2.0 involves authorization server issuing tokens after user consent. Web API validates bearer tokens in Authorization header using middleware.

**Explanation:** Common flows include Authorization Code (web apps), Implicit (SPAs), Client Credentials (service-to-service).

**Follow-up:** What is the difference between authentication and authorization? How do refresh tokens work?

### 24. How do you implement API versioning strategies?
**Answer:** URL versioning (/api/v1/), query string (?version=1), header versioning (api-version: 1), or media type versioning (application/vnd.company.v1+json).

**Explanation:** Each strategy has trade-offs for caching, routing complexity, and client implementation. URL versioning is most visible.

**Follow-up:** How do you handle breaking changes? What is semantic versioning for APIs?

### 25. Explain CORS and preflight requests
**Answer:** Cross-Origin Resource Sharing allows browsers to make cross-domain requests. Preflight OPTIONS requests check if actual request is allowed.

**Explanation:** Browser sends Origin header, server responds with Access-Control headers. Complex requests trigger preflight.

**Follow-up:** What triggers a preflight request? How do you configure CORS policies in .NET Core?

## ADO.NET Questions

### 26. Compare connected vs disconnected architecture in ADO.NET
**Answer:** Connected maintains open connection during operations (DataReader). Disconnected fetches data into DataSet/DataTable and works offline.

**Explanation:** Connected is faster, memory-efficient for forward-only reading. Disconnected enables offline work but uses more memory.

**Follow-up:** When would you choose DataReader over DataSet? How does connection pooling work?

### 27. Explain the role of DataAdapter in ADO.NET
**Answer:** DataAdapter bridges between DataSet and database, providing Fill() to populate and Update() to persist changes using SelectCommand, InsertCommand, UpdateCommand, DeleteCommand.

**Explanation:** Automatically generates commands using CommandBuilder or allows custom commands for complex scenarios.

**Follow-up:** How does optimistic concurrency work with DataAdapter? What is batch updating?

### 28. How do you prevent SQL injection in ADO.NET?
**Answer:** Use parameterized queries with SqlParameter objects. Never concatenate user input into SQL strings. Parameters are sent separately from SQL text.

**Explanation:** Parameters ensure data is treated as values, not executable code. Also validates data types and handles special characters.

**Follow-up:** What's the difference between Parameters.Add and Parameters.AddWithValue? Why avoid AddWithValue?

### 29. Describe transaction management in ADO.NET
**Answer:** SqlTransaction provides ACID transactions within a connection. Begin with BeginTransaction(), execute commands with transaction, then Commit() or Rollback().

**Explanation:** All commands must use the same connection and transaction. Nested transactions aren't truly nested in SQL Server.

**Follow-up:** How do distributed transactions work with TransactionScope? What is transaction isolation level?

### 30. What is the purpose of connection pooling?
**Answer:** Connection pooling reuses physical database connections to avoid overhead of creating/destroying connections. Managed automatically by ADO.NET.

**Explanation:** Pools are per connection string. Connections return to pool on Close/Dispose. Min/Max pool size configurable.

**Follow-up:** How do you monitor connection pool usage? What causes pool fragmentation?

## ASP.NET MVC Questions

### 31. Explain the MVC request pipeline and action execution
**Answer:** Request → Routing → Controller Selection → Action Selection → Model Binding → Action Filters → Action Execution → Result Filters → Result Execution → Response.

**Explanation:** Each stage is extensible through interfaces. Filters provide cross-cutting concerns at different pipeline stages.

**Follow-up:** How do you create custom action filters? What's the order of filter execution?

### 32. How does model binding work with complex types?
**Answer:** Model binder recursively maps request data (form, route, query) to action parameters using naming conventions. Supports nested objects and collections.

**Explanation:** Default binder uses property names to match data. Custom binders handle special types. Validation occurs after binding.

**Follow-up:** How do you create custom model binders? What is over-posting and how do you prevent it?

### 33. Describe ViewBag, ViewData, and TempData differences
**Answer:** ViewBag is dynamic wrapper over ViewData dictionary. Both last for single request. TempData persists across redirects using session/cookies.

**Explanation:** ViewBag provides property syntax but no compile-time checking. TempData marks data as read after access unless Keep() called.

**Follow-up:** When should you use strongly-typed view models instead? How does TempData provider work?

### 34. Explain Razor view engine compilation and caching
**Answer:** Razor views compile to C# classes inheriting from WebViewPage. First request triggers compilation, then cached. Precompilation available for performance.

**Explanation:** Runtime compilation allows view changes without rebuild. Compiled views are assemblies loaded into AppDomain.

**Follow-up:** How do you enable view precompilation? What is Razor runtime compilation in .NET Core?

### 35. How do HTML helpers differ from Tag helpers?
**Answer:** HTML helpers are C# methods generating HTML. Tag helpers are attributes on HTML elements processed server-side, providing cleaner syntax.

**Explanation:** Tag helpers maintain HTML-like syntax while adding server-side behavior. HTML helpers use lambda expressions.

**Follow-up:** How do you create custom tag helpers? Can you use both in the same view?

### 36. Describe areas and their routing implications
**Answer:** Areas partition MVC apps into functional groups with own controllers, views, and routes. Each area has separate folder structure and routing namespace.

**Explanation:** Areas enable modular application structure. Route must specify area name to disambiguate controllers with same names.

**Follow-up:** How do you link between areas? What are area route constraints?

### 37. Explain action results and custom result types
**Answer:** ActionResult is base class for all results. Common types: ViewResult, JsonResult, RedirectResult, FileResult. Custom results implement ExecuteResult.

**Explanation:** Results encapsulate response generation logic. Separation enables testing without HTTP context.

**Follow-up:** What's the difference between JsonResult and returning object in Web API? How do you stream large files?

### 38. How does ASP.NET MVC handle security and CSRF?
**Answer:** Anti-forgery tokens prevent CSRF by requiring token validation. Forms include hidden token, cookies store second token, both must match.

**Explanation:** ValidateAntiForgeryToken attribute enforces checking. Token generation/validation customizable. AJAX requires header token.

**Follow-up:** How do you implement CSRF protection for AJAX calls? What is double submit cookie pattern?

## Advanced Architecture & Design Questions

### 39. How do you implement dependency injection in .NET Core?
**Answer:** Built-in DI container registers services in ConfigureServices with lifetimes (Transient, Scoped, Singleton). Constructor injection preferred, resolved recursively.

**Explanation:** Scoped services live per request, Singleton per app lifetime, Transient per resolution. Container manages lifecycles and disposal.

**Follow-up:** When would you use factory pattern with DI? How do you handle circular dependencies?

### 40. Explain middleware pipeline in ASP.NET Core
**Answer:** Middleware components form a pipeline processing requests/responses. Each middleware calls next() or short-circuits. Order matters for behavior.

**Explanation:** Request flows through middleware in registration order, response returns in reverse. Middleware can modify both.

**Follow-up:** How do you conditionally apply middleware? What's the difference between Use, Run, and Map?

### 41. Describe async patterns evolution from APM to TAP
**Answer:** APM (BeginXxx/EndXxx with callbacks), EAP (event-based with XxxAsync/XxxCompleted), TAP (Task-based with async/await).

**Explanation:** TAP simplified async programming with composability and exception handling. Async/await provides synchronous-looking code.

**Follow-up:** How do you wrap APM methods in Tasks? What is the async disposal pattern?

### 42. How do you implement circuit breaker pattern?
**Answer:** Circuit breaker prevents cascading failures by tracking failures and opening circuit after threshold. Includes Closed, Open, Half-Open states.

**Explanation:** Closed allows requests, Open fails fast, Half-Open tests recovery. Polly library provides implementation.

**Follow-up:** What's the difference from retry pattern? How do you implement bulkhead isolation?

### 43. Explain event sourcing and CQRS patterns
**Answer:** Event sourcing stores state changes as events. CQRS separates read/write models. Often combined for complex domains.

**Explanation:** Events provide audit trail and temporal queries. CQRS optimizes read/write sides independently. Eventually consistent.

**Follow-up:** How do you handle event versioning? What are projections in event sourcing?

### 44. Describe microservices communication patterns
**Answer:** Synchronous (HTTP/gRPC), asynchronous (message queues/event bus), service mesh for discovery/routing. Each has trade-offs.

**Explanation:** Sync is simpler but creates coupling. Async enables resilience but adds complexity. Service mesh handles cross-cutting concerns.

**Follow-up:** How do you handle distributed transactions? What is saga pattern?

### 45. How does memory caching differ from distributed caching?
**Answer:** Memory caching (IMemoryCache) stores in-process, fast but not shared. Distributed caching (Redis/SQL) shared across instances, slower but scalable.

**Explanation:** Memory cache lost on restart, doesn't scale horizontally. Distributed cache survives restarts, enables session affinity.

**Follow-up:** What are cache invalidation strategies? How do you prevent cache stampede?

## Performance & Optimization Questions

### 46. Explain object pooling and when to use it
**Answer:** Object pooling reuses expensive objects instead of creating/destroying. ArrayPool, MemoryPool, and custom pools reduce GC pressure.

**Explanation:** Effective for frequently allocated objects with expensive initialization. Must handle cleanup between uses.

**Follow-up:** How does ArrayPool work internally? What are the risks of pooling?

### 47. Describe ETW and performance profiling in .NET
**Answer:** Event Tracing for Windows provides low-overhead instrumentation. EventSource enables custom events. PerfView/dotnet-trace analyze traces.

**Explanation:** ETW captures detailed runtime events without debugger overhead. Enables production diagnostics.

**Follow-up:** How do you create custom EventSource providers? What is BenchmarkDotNet?

### 48. How do you optimize LINQ queries for performance?
**Answer:** Avoid multiple enumeration, use appropriate collection types, consider PLINQ for CPU-bound work, be aware of closure allocations.

**Explanation:** LINQ creates iterator chains that can be inefficient. Understanding execution model helps optimize.

**Follow-up:** When does PLINQ help vs hurt performance? What is query syntax vs method syntax performance?

### 49. Explain JIT compilation tiers and optimization
**Answer:** Tiered compilation starts with quick JIT (Tier 0), then optimizes hot paths (Tier 1). ReadyToRun provides AOT for startup.

**Explanation:** Balances startup time with runtime performance. Profiling guides optimization decisions.

**Follow-up:** What is PGO (Profile-Guided Optimization)? How does crossgen work?

### 50. Describe garbage collection tuning strategies
**Answer:** Workstation vs Server GC, concurrent vs background GC, LOH compaction, GC regions. Configuration via runtime settings.

**Explanation:** Server GC uses multiple threads but higher memory. Background GC reduces pause times. Regions improve heap management.

**Follow-up:** How do you diagnose GC performance issues? What are pinned objects and their impact?

---

## Interview Success Tips

1. **Be specific with examples** - Always ready with real-world scenarios you've encountered
2. **Admit unknowns gracefully** - Show how you'd research and solve unfamiliar problems
3. **Demonstrate depth** - Go beyond surface-level answers to show true understanding
4. **Ask clarifying questions** - Shows analytical thinking and attention to requirements
5. **Discuss trade-offs** - Every technical decision has pros and cons

Good luck with your interview! These questions cover the full technical stack you mentioned and should prepare you well for both Tech Lead and Solution Architect discussions.