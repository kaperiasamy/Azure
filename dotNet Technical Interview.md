## Technical Round – (C# / .NET / SQL)

### 1. **Difference between ref and out**

"In C#, both `ref` and `out` are used to pass arguments by reference, allowing the called method to modify the value of the parameter. However, they differ in intent and usage."

- **Initialization Requirement**:
  - _ref_: The variable must be initialized before it is passed.
  - _out_: The variable does not need to be initialized before passing, but must be assigned inside the method before returning.

- **Use Case**:
  - _ref_ is ideal when the method needs to read and modify the incoming value.
  - _out_ is best when the method is expected to return multiple outputs, and the incoming value is irrelevant.

- **Directionality**:
  - _ref_ is bi-directional —data flows in and out.
  - _out_ is uni-directional —data flows only out.

Example:
```csharp
void ModifyRef(ref int x) {
    x += 10;
}

void SetOut(out int y) {
    y = 42;
}
```

Bonus Tip: You can impress the interviewer by adding:
> "At compile time, both are treated similarly, but at runtime, their behavior diverges. Choosing between them is not just syntactic—it's about clearly communicating the method's intent to other developers."

---

**Question 1: Initialization Requirement**
Which statement is true about variables passed with the ref keyword?

A. They must be assigned inside the method
**B. They must be initialized before being passed**
C. They cannot be modified inside the method
D. They are passed by value

> `ref` requires the variable to be initialized before it's passed to the method.

---

**Question 2: Method Signature**
Which of the following method signatures correctly uses out?

**A. void Calculate(out int result) { result = 10;}**
B. void Calculate(out int result) { Console.WriteLine(result);}
C. void Calculate(out int result) {}
D. void Calculate(out int result) { result += 5;}

> With `out`, the method must assign a value before returning. Options B, C, and D violate that rule.

---

**Question 3: Directionality**
What best describes the data flow of an out parameter?

A. In only
B. In and Out
**C. Out only**
D. Neither in nor out

> `out` parameters are meant for output only. The method doesn’t care about the incoming value.

---

**Question 4: Use Case**
When would you prefer ref over out?

A. When you want to return multiple values
B. When the input value is irrelevant
**C. When you need to read and modify the original value**
D. When you want to discard the original value

> `ref` allows both reading and writing, making it ideal when the input value matters.

---

**Question 5: Compilation Behavior**
What happens if you pass an uninitialized variable to a method using ref?

A. It compiles and runs
B. It throws a runtime exception
**C. It fails to compile**
D. It initializes the variable automatically

> If you try to pass an uninitialized variable with `ref`, the compiler will throw an error.

#### Common mistakes with `ref` and `out`

1. **Not Initializing ref Parameters Before Passing**
- Mistake: Passing a variable with ref that hasn’t been initialized.
- Why it fails: The compiler requires ref parameters to be initialized before use.
- Fix: Always assign a value before passing it with ref.

---

2. **Forgetting to Assign out Parameters Inside the Method**
- Mistake: Declaring an out parameter but not assigning it in the method.
- Why it fails: The compiler enforces that out parameters must be assigned before the method returns.
- Fix: Ensure every execution path in the method assigns a value to the out parameter.

---

3. **Using ref or out When Not Necessary**
- Mistake: Overusing ref/out when a return value or a tuple would be cleaner.
- Why it’s bad: It can make code harder to read and maintain.
- Fix: Use ref/out only when you truly need to modify or return multiple values.

---

4. **Misunderstanding Reference vs. Value Types**
- Mistake: Assuming ref is needed to modify properties of a reference type.
- Why it’s misleading: Reference types can have their properties modified without ref.
- Fix: Use ref only if you need to reassign the reference itself, not just mutate its contents.

---

5. **Mixing ref and out in Confusing Ways**
- Mistake: Using both in the same method without clear intent.
- Why it’s risky: It can confuse other developers and obscure the method’s purpose.
- Fix: Keep method signatures clean and intuitive—document your intent clearly.

---

6. **Ignoring Method Overload Conflicts**
- Mistake: Creating method overloads that differ only by ref vs out.
- Why it’s problematic: It can lead to ambiguous calls and compiler errors.
- Fix: Avoid overloads that rely solely on ref/out distinctions.

#### best practices for using ref and out

Using `ref` and `out` in C# can be powerful—but only when used with care and clarity. Here are the best practices to follow so your code stays clean, maintainable, and interview-worthy:

---

1. **Use `ref` and `out` Only When Necessary**
- Prefer return values or tuples for multiple outputs when possible.
- Use `ref` when you need to read and modify a value.
- Use `out` when you only need to return a value, and the input is irrelevant.

> Tip: Overusing these keywords can make code harder to understand and debug.

---

2. **Always Document Intent Clearly**
- Make sure method names and XML comments explain why `ref` or `out` is used.
- This helps other developers (and future you) understand the method’s purpose.

---

3. **Avoid Mixing `ref` and `out` in the Same Method**
- It can lead to confusion about which parameters are being read vs. written.
- Keep method signatures clean and intuitive.

---

4. **Ensure Proper Initialization**
- `ref` parameters must be initialized before being passed.
- `out` parameters must be assigned inside the method before returning.

> The compiler enforces this, but it’s still good to be mindful of it.

---

5. **Don’t Use `ref` for Reference Type Properties**
- Reference types (like classes) can have their properties modified without `ref`.
- Use `ref` only if you need to reassign the reference itself, not just mutate its contents.

---

6. **Avoid Overloading Methods Based Solely on `ref` vs `out`**
- This can lead to ambiguous calls and compiler confusion.
- Instead, differentiate overloads with parameter types or method names.

---

7. **Consider Alternatives for Clarity**
- Use Tuple, ValueTuple, or custom classes to return multiple values.
- These are often more readable and self-documenting than out parameters.

#### differences between `ref`, `out`, and `in`

1. `ref` — **Read and Write Access**
- **Purpose**: Pass a variable by reference, allowing the method to read and modify it.
- **Initialization**: Must be initialized before being passed.
- **Use Case**: When the method needs to use the existing value and possibly change it.
- Example:
```csharp
  void Update(ref int x) {
      x += 10;
}
```

---

2. `out` — **Write-Only Access**
- **Purpose**: Pass a variable by reference, but the method is expected to assign a value to it.
- **Initialization**: Does not need to be initialized before passing.
- **Use Case**: When returning multiple outputs from a method.
- Example:
```csharp
  void GetValues(out int a, out int b) {
      a = 1;
      b = 2;
}
``` 

---

3. `in` — **Read-Only Access**
- **Purpose**: Pass a variable by reference, but the method cannot modify it.
- **Initialization**: Must be initialized before passing.
- **Use Case**: For performance optimization when passing large structs without copying, while ensuring immutability.
- Example:
```csharp
  void Print(in int x) {
      Console.WriteLine(x);
      // x = 10; // Compile error
}
```

---

### 2. What is async/await?

"Async and await are C# keywords that enable asynchronous programming, allowing methods to run without blocking the calling thread. This is especially useful for I/O-bound operations like web requests, file access, or database calls."

- `async` marks a method as asynchronous and allows the use of the `await` keyword inside it.
- `await` pauses the method execution until the awaited task completes—without blocking the thread.

**Why it matters**:
> "Async/await improves application responsiveness and scalability. For example, in a web application, it allows the server to handle other requests while waiting for a database query to complete."

Example:
```csharp
public async Task<string> FetchDataAsync()
{
    HttpClient client = new HttpClient();
    string result = await client.GetStringAsync("https://example.com");
    return result;
}
```

**Bonus Insight**:
> "Under the hood, async/await uses the Task-based Asynchronous Pattern (TAP). It doesn’t necessarily create new threads—it leverages the SynchronizationContext to resume execution efficiently. This distinction between asynchronous and multithreaded programming is key."

1. **Async vs. Multithreading**
- Async ≠ Multithreading: Async is about non-blocking I/O, not spinning up new threads.
- Tasks may run on the same thread, especially for I/O-bound operations.
- *Interview Tip*: Emphasize: “Async improves scalability, not parallelism. For CPU-bound work, use Task.Run or parallel constructs.”

2. **Exception Handling in Async Code**
- Exceptions thrown in async methods are captured in the returned Task.
- Use try/catch around await to handle them.
- Example:
```csharp
  try {
      await SomeAsyncMethod();
   } catch (Exception ex) {
      // Handle exception
   }
```
- Interview Tip: Mention Task.WhenAll and how it aggregates exceptions.

3. **`Task.WhenAll` vs. `Task.WaitAll`**
- *Task.WhenAll*: Asynchronously waits for multiple tasks to complete.
- *Task.WaitAll*: Blocks the calling thread.
- *Use Case*: Prefer `WhenAll` in `async` methods to maintain non-blocking behavior.
- Example:
```csharp
  await Task.WhenAll(task1, task2);
```

---

### 3. Explain parallel programming

**"Parallel programming in.NET refers to the technique of executing multiple operations simultaneously to improve performance, especially for CPU-bound tasks. It leverages multiple cores of the processor to run code concurrently, reducing overall execution time."**

- **Core Concept**: Instead of running tasks sequentially, parallel programming breaks them into smaller units that can run at the same time.
- **Key API**: The System.Threading.Tasks namespace provides the Parallel class and the Task Parallel Library (TPL), which simplify parallel execution.

Example:
```csharp
Parallel.For(0, 100, i => {
    // Perform computation
});
```

- Benefits:
  - Maximizes CPU utilization
  - Reduces latency for compute-heavy operations
  - Improves scalability in multi-core environments

- Common Tools:
  - Parallel.For and Parallel.ForEach for data parallelism
  - Task and Task.Run for task-based parallelism
  - PLINQ (Parallel LINQ) for parallel query execution

Bonus Insight:
> "Unlike asynchronous programming, which is ideal for I/O-bound operations, parallel programming is best suited for CPU-bound workloads. Understanding when to use each is key to building efficient and responsive applications."

---

### 4. What is a thread?

"**A thread is the smallest unit of execution within a process. In.NET, threads allow a program to perform multiple operations concurrently, which is essential for building responsive and high-performance applications.**"

- Each process has at least one thread, known as the main thread, which executes the application code.
- Multiple threads can be created to run tasks in parallel, enabling better CPU utilization and faster execution of time-consuming operations.

Example:
```csharp
Thread t = new Thread(() => {
    Console.WriteLine("Running on a separate thread");
});
t.Start();
```

**Why it matters:**
> "Threads are especially useful in scenarios like handling background tasks, performing parallel computations, or keeping the UI responsive while processing data."

**Bonus Insight:**
> "While threads offer powerful concurrency, they also introduce complexity—such as race conditions, deadlocks, and synchronization challenges. That's why.NET provides higher-level abstractions like the Task Parallel Library and async/await for safer and more scalable concurrency."

1. **Thread Synchronization**

**Purpose**: Prevent multiple threads from accessing shared resources simultaneously, which can lead to race conditions, deadlocks, or corrupted data.

**Common Techniques**:
- *lock keyword*: Ensures only one thread can access a block of code at a time.
```csharp
  lock(myLockObject) {
      // Critical section
}
```

- *Monitor*: More flexible than lock, allows timeout and pulse mechanisms.
- *Mutex*: Can be used across processes, but heavier than lock.
- *Semaphore / SemaphoreSlim*: Controls access to a resource pool.
- *Interlocked*: Atomic operations for simple numeric updates.

**Best Practice**: Keep critical sections short and avoid nested locks to reduce deadlock risk.

---

2. **Thread Pools**

**What it is**: A managed pool of threads maintained by the.NET runtime to efficiently execute short-lived, concurrent tasks.

**Advantages**:
- Reduces overhead of thread creation.
- Automatically reuses threads.
- Scales better for high-throughput scenarios.

**Usage**:
- `ThreadPool.QueueUserWorkItem` for fire-and-forget tasks.
- `Task.Run` internally uses the thread pool.

**Limitation**: Not ideal for long-running or blocking operations—can exhaust the pool.

---

3. **Threads vs Tasks**
FeatureThreadTaskCreationManual (new Thread)Lightweight (Task.Run)Resource UsageHigh (1MB+ per thread)Low (uses thread pool)Exception HandlingManualBuilt-in (exceptions propagate)CancellationManualSupports `CancellationToken`SchedulingManualManaged by `TaskScheduler`Use CaseLong-running, low-level controlShort-lived, high-level concurrency Key Insight: Tasks are built on top of threads but offer a higher-level abstraction with better scalability and cleaner syntax. Use threads when you need full control; use tasks for most application-level concurrency.

---

### 5. Difference between IEnumerable and IQueryable

"**These interfaces represent different levels of abstraction for working with collections in.NET. Understanding their differences helps in choosing the right tool for performance, flexibility, and clarity.**"

1. **IEnumerable<T>**
- *Definition*: The most basic interface for iteration. It allows forward-only traversal of a collection.
- *Use Case*: Ideal for read-only access and deferred execution, especially when working with LINQ.
- *Importance*: Lightweight and flexible. Best for exposing data without modification.
- *Example*:
```csharp
  IEnumerable<int> numbers = new List<int> { 1, 2, 3};
```

2. **IQueryable<T>**
- *Definition*: Extends IEnumerable<T> and adds query capabilities that can be translated to a data source (like SQL).
- *Use Case*: Used in ORM frameworks like Entity Framework to build database queries.
- *Importance*: Enables deferred execution and server-side filtering, which improves performance.
- *Example*:
```csharp
  IQueryable<User> users = dbContext.Users.Where(u => u.IsActive);
```

3. **ICollection<T>**
- *Definition*: Extends IEnumerable<T> and adds methods for modifying the collection (Add, Remove, Count).
- *Use Case*: Useful when you need to expose a modifiable collection but don’t need indexing.
- *Importance*: Provides a balance between flexibility and control over collection contents.
- *Example*:
```csharp
  ICollection<string> names = new List<string>();
  names.Add("Alice");
```

4. **IList<T>**
- *Definition*: Extends ICollection<T> and adds indexing capabilities.
- *Use Case*: Best when you need random access to elements or maintain order.
- *Importance*: Most commonly used for in-memory collections where order and index matter.
- *Example*:
```csharp
  IList<int> scores = new List<int> { 90, 85, 78};
  int topScore = scores[0];
```

---

**Summary Table**

## Interface Comparison Table

| Interface    | Read | Write | Indexing | Deferred Execution | Best For                    |
|--------------|------|-------|----------|--------------------|-----------------------------|
| IEnumerable  | Yes  | No    | No       | Yes                | Iteration, LINQ queries     |
| IQueryable   | Yes  | No    | No       | Yes (server-side)  | Database queries via LINQ   |
| ICollection  | Yes  | Yes   | No       | No                 | Modifiable collections      |
| IList        | Yes  | Yes   | Yes      | No                 | Ordered, index-based access |


**Bonus Insight**:
> "Choosing the right interface is about intent. If you're exposing data for iteration, use IEnumerable. If you're querying a database, use IQueryable. If you want to allow modification, use ICollection or IList depending on whether indexing is needed."

---

### 6. Difference between Authentication and Authorization

**"Authentication and Authorization are two distinct but closely related concepts in application security. Authentication is the process of verifying the identity of a user, while Authorization determines what that authenticated user is allowed to do."**

**Authentication**
- *Purpose*: Confirms who the user is.
- *How*: Validates credentials like username and password.
- *Example*: Logging into a system using ASP.NET Identity or OAuth.
- In.NET: Implemented using middleware such as UseAuthentication() and services like ASP.NET Core Identity.

**Authorization**
- *Purpose*: Controls access to resources based on identity.
- *How*: Checks roles, claims, or policies to determine permissions.
- *Example*: Allowing only Admin users to access a dashboard.
- In.NET: Configured using UseAuthorization() and attributes like [Authorize(Roles = "Admin")].

---

**Real-World Analogy:**
> "Authentication is like showing your ID at the door. Authorization is the bouncer checking if you're allowed into the VIP section."

**Bonus Insight:**
> "In ASP.NET Core, these are handled separately in the middleware pipeline, allowing flexible and secure access control. Role-based and claims-based authorization are commonly used patterns."

---

### 7. What is JWT and how does it work?

**"JWT stands for JSON Web Token. It's a compact, URL-safe token format used to securely transmit information between parties, typically for authentication and authorization in web applications."**

**How JWT Works:**
1. **Structure**: A JWT consists of three parts:
   - *Header*: Specifies the signing algorithm and token type.
   - *Payload*: Contains claims—statements about the user and metadata.
   - *Signature*: Ensures the token hasn't been tampered with.

   Example format:
   
   *xxxxx.yyyyy.zzzzz*
   

2. **Authentication Flow**:
   - A user logs in with valid credentials.
   - The server generates a JWT and sends it to the client.
   - The client stores the token (usually in local storage or a cookie).
   - For subsequent requests, the token is sent in the Authorization header.
   - The server validates the token and grants access based on its claims.

3. **Statelessness**:
   - JWTs are self-contained, meaning the server doesn't need to store session data.
   - This makes them ideal for scalable, distributed systems like microservices.

**Use Cases:**
- *Authentication*: Verifying user identity.
- *Authorization*: Granting access based on roles or claims.
- *Single Sign-On (SSO)*: Sharing identity across multiple systems.

**Bonus Insight:**
> "JWTs are digitally signed using algorithms like HMAC or RSA. This ensures integrity—if the token is altered, the signature validation fails. You can also set expiration times to enhance security."

---

### 8. What is Middleware?

**"Middleware in ASP.NET Core is software that sits in the request-response pipeline and handles HTTP requests and responses. Each middleware component can perform operations before and after the next component in the pipeline is invoked."**

**Key Concepts:**
- *Pipeline Architecture*: Middleware components are executed in the order they are registered in the Startup.Configure method.
- *Chaining*: Each middleware can either pass the request to the next component or short-circuit the pipeline.
- *Tasks Performed*: Middleware can handle logging, authentication, authorization, error handling, routing, response compression, and more.

Example:
```csharp
public class CustomMiddleware
{
    private readonly RequestDelegate _next;

    public CustomMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        // Pre-processing logic
        await _next(context);
        // Post-processing logic
    }
}
```

Registered in Startup.cs:
```csharp
app.UseMiddleware<CustomMiddleware>();
```

**Bonus Insight:**
> "Middleware promotes modularity and separation of concerns. In ASP.NET Core, the built-in middleware components are highly composable, and developers can easily create custom middleware to tailor the request pipeline to specific application needs."

---

### 9. What is an Interface? (Interviewer expected discussion on Dependency Injection / Loose Coupling)

**"An interface in C# defines a contract that classes can implement. It specifies a set of methods, properties, or events without providing any implementation. Interfaces are central to achieving loose coupling and enabling dependency injection in modern.NET applications."**

**Why Interfaces Matter:**
- *Loose Coupling*: By programming against interfaces rather than concrete classes, components become interchangeable and easier to maintain.
- *Testability*: Interfaces allow mocking dependencies during unit testing.
- *Flexibility*: Behavior can be changed without modifying dependent code.

Example:
```csharp
public interface ILogger {
    void Log(string message);
    }

public class FileLogger: ILogger {
    public void Log(string message) {
        // Write to file
    }
}
```

**Dependency Injection:**
- Instead of creating FileLogger directly, you inject ILogger into the consumer class.
```csharp
public class OrderService {
    private readonly ILogger _logger;

    public OrderService(ILogger logger) {
        _logger = logger;
    }

    public void PlaceOrder() {
        _logger.Log("Order placed.");
    }
}
```

- In ASP.NET Core, you register the dependency in the container:
```csharp
services.AddScoped<ILogger, FileLogger>();
```

**Bonus Insight:**
> "This approach follows the Dependency Inversion Principle—one of the SOLID principles—where high-level modules depend on abstractions, not concrete implementations. It makes the system more modular, testable, and easier to evolve."

---

### 10. Scopes in Dependency Injection (Transient, Scoped, Singleton)

**In.NET, Dependency Injection (DI) allows services to be registered with different lifetimes, which define how long an instance of a service lives. The three main scopes are Transient, Scoped, and Singleton.**

1. **Transient**
- *Lifetime*: A new instance is created every time the service is requested.
- *Use Case*: Lightweight, stateless services like formatters or validators.
- *Example*:
```csharp
  services.AddTransient<IMyService, MyService>();
```

2. **Scoped**
- *Lifetime*: One instance per HTTP request (or scope).
- *Use Case*: Services that maintain state throughout a single request, such as database contexts.
- *Example*:
```csharp
  services.AddScoped<IMyService, MyService>();
```

3. **Singleton**
- *Lifetime*: A single instance is created and shared across the entire application.
- *Use Case*: Services that hold shared state or perform caching, like configuration providers or logging services.
- *Example*:
```csharp
  services.AddSingleton<IMyService, MyService>();
```

---

**Bonus Insight**:
> "Choosing the right scope is critical. For example, injecting a Scoped service into a Singleton can cause runtime errors or unexpected behavior. Understanding scopes helps avoid memory leaks, race conditions, and ensures thread safety."

---

### 11. What is Rate Limiting?

**Rate limiting is a technique used to control how many requests a client can make to a server within a specific time window. It's commonly used in APIs to prevent abuse, ensure fair usage, and protect backend resources from being overwhelmed.**

**Why Rate Limiting Matters:**
- Prevents server overload by limiting excessive traffic.
- Protects against brute-force attacks and denial-of-service attempts.
- Ensures fair access for all users.
- Helps manage cloud costs by reducing unnecessary load.

**How It Works:**
- A server tracks the number of requests from each client (IP address, user ID, or token).
- If the client exceeds the allowed limit, the server rejects further requests with a 429 Too Many Requests status code.
- Limits can be based on fixed windows, sliding windows, or token buckets.

**Rate Limiting in.NET:**
- Starting with.NET 7 and.NET 8, ASP.NET Core includes built-in middleware for rate limiting.
- You can configure policies using AddRateLimiter in Program.cs and apply them to endpoints.

**Example:**
```csharp
builder.Services.AddRateLimiter(options =>
{
    options.AddFixedWindowLimiter("fixed", opt =>
    {
        opt.Window = TimeSpan.FromMinutes(1);
        opt.PermitLimit = 100; // Allow 100 requests per minute
    });
});
```

**Bonus Insight:**
> "Rate limiting is not just a performance tool—it's a security layer. When combined with authentication and logging, it helps build resilient APIs that scale safely under load."

---

## SQL

### 1. Query to select the second highest salary from an employee table

**To select the second highest salary from an employee table, there are multiple ways depending on the database system and performance needs. One of the most reliable and readable methods is using a subquery that excludes the highest salary.**

**Method 1: Using Subquery**
```sql
SELECT MAX(salary)
FROM employee
WHERE salary < (SELECT MAX(salary) FROM employee);
```

- *Explanation*: This query finds the maximum salary that is less than the highest salary—effectively the second highest.

---

If the interviewer is looking for a more advanced or scalable solution, especially when handling ties or ranking, I would use a Common Table Expression (CTE) with a ranking function.

**Method 2: Using CTE and DENSE_RANK (SQL Server, PostgreSQL)**
```sql
WITH SalaryRanks AS (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rank
    FROM employee
)
SELECT salary
FROM SalaryRanks
WHERE rank = 2;
```

- *Explanation*: This method handles duplicate salaries and returns all employees who share the second highest salary.

---

**Bonus Insight:**
> Using `DENSE_RANK` ensures that if multiple employees have the same salary, they are ranked consistently. This approach is also extensible—just change the rank value to get the third, fourth, or nth highest salary.

---

### 2. What is the ACID principle?

**ACID stands for Atomicity, Consistency, Isolation, and Durability. These are the four key properties that ensure reliable and predictable behavior of database transactions. Together, they guarantee that even in the face of errors, crashes, or concurrent access, the database remains accurate and trustworthy.**

---

1. **Atomicity – All or Nothing**
- *Definition*: A transaction must be treated as a single unit. Either all operations within the transaction succeed, or none are applied.
- *Example*: Transferring money from Account A to Account B:
  - Step 1: Deduct $100 from Account A
  - Step 2: Add $100 to Account B
  - If Step 2 fails, Step 1 must be rolled back to prevent data inconsistency.
- *Why it matters*: Prevents partial updates that could corrupt data.

---

2. **Consistency – Valid State Transitions**
- *Definition*: A transaction must bring the database from one valid state to another, maintaining all defined rules, constraints, and relationships.
- *Example*: In a banking system, the total balance across all accounts should remain constant after a transfer.
  - *Before*: Account A = $500, Account B = $200, Total = $700
  - *After*: Account A = $400, Account B = $300, Total = $700
- *Why it matters*: Ensures data integrity by enforcing business rules and constraints.

---

3. **Isolation – No Interference Between Transactions**
- *Definition*: Concurrent transactions must not interfere with each other. The outcome should be the same as if transactions were executed sequentially.
- *Example*: Two users updating the same account balance simultaneously:
  - Without isolation, one user's changes might overwrite the other's.
  - With isolation, each transaction is executed in isolation, preventing dirty reads, non-repeatable reads, or phantom reads.
- *Why it matters*: Prevents data anomalies in multi-user environments.

---

4. **Durability – Changes Persist After Commit**
- *Definition*: Once a transaction is committed, its changes must be permanent—even in the event of a system crash or power failure.
- *Example*: After a successful order placement, the order data must remain in the database even if the server crashes immediately afterward.
- *Why it matters*: Guarantees that committed data is safely stored and recoverable.

---

**Real-World Relevance in.NET:**
- In.NET applications using Entity Framework or ADO.NET, transactions can be managed using TransactionScope or DbContext.Database.BeginTransaction().
- ACID compliance is critical when working with relational databases like SQL Server, PostgreSQL, or Oracle.

---

**Bonus Insight:**
> Understanding ACID is not just about theory—it's about designing systems that are resilient, predictable, and safe under pressure. Whether you're building financial software, e-commerce platforms, or healthcare systems, ACID principles are the backbone of trustworthy data operations.

---

### 3. Query to select duplicate rows

**To select duplicate rows in SQL, we typically use the GROUP BY clause to group records based on one or more columns, and the HAVING clause to filter groups that appear more than once. This helps identify data integrity issues caused by duplicate entries.**

---

#### Step-by-Step Explanation

1. **Identify the Columns That Define a Duplicate**
- Duplicates are based on business logic. For example, two employees with the same name and email might be considered duplicates.

2. **Use GROUP BY and HAVING**
```sql
SELECT name, email, COUNT(*)
FROM employee
GROUP BY name, email
HAVING COUNT(*)> 1;
```
- GROUP BY groups rows with the same name and email.
- COUNT()> 1* filters only those groups that appear more than once.

---

Example Table: employee
idnameemail1Johnjohn@example.com2Alicealice@example.com3Johnjohn@example.com4Bobbob@example.com Result:

name   | email            | count
-------|------------------|------
John   | john@example.com | 2


---

3. **Retrieve Full Duplicate Rows (Optional)**
If you want to see the full rows that are duplicates, use a subquery or a JOIN:

```sql
SELECT e.*
FROM employee e
JOIN (
    SELECT name, email
    FROM employee
    GROUP BY name, email
    HAVING COUNT(*)> 1
) dup ON e.name = dup.name AND e.email = dup.email;
```

---

4. **Finding Duplicates in a Single Column**
```sql
SELECT email, COUNT(*)
FROM employee
GROUP BY email
HAVING COUNT(*)> 1;
```

---

**Bonus Insight:**
> Finding duplicates is not just about writing a query—it's about understanding data quality. In real-world systems, duplicates can lead to incorrect reports, billing errors, or broken relationships between tables. That's why mastering this query is a sign of a developer who cares about clean, reliable data.

---

### 4. Query to get the highest salary from each department (Tables: Employee, Department, Salary)

**To get the highest salary from each department, we need to join the relevant tables and use aggregation to group salaries by department. The goal is to identify the top salary per department, and optionally, the employee(s) who earn it.**

---

**Step-by-Step Breakdown**

Assumed Table Structures:

- Employee(id, name, department_id)
- Department(id, name)
- Salary(employee_id, amount)

---

**Approach 1: Using JOIN and GROUP BY with MAX**

This method retrieves the highest salary per department.

```sql
SELECT d.name AS department_name, MAX(s.amount) AS highest_salary
FROM Department d
JOIN Employee e ON d.id = e.department_id
JOIN Salary s ON e.id = s.employee_id
GROUP BY d.name;
```

*Explanation*:
- JOIN connects all three tables.
- GROUP BY d.name groups data by department.
- MAX(s.amount) finds the highest salary in each group.

---

#### Approach 2: Retrieve Employee(s) with Highest Salary per Department

If the interviewer wants to know who earns the highest salary, use a subquery or window function.

**Option A: Subquery with WHERE IN**

```sql
SELECT d.name AS department_name, e.name AS employee_name, s.amount AS salary
FROM Department d
JOIN Employee e ON d.id = e.department_id
JOIN Salary s ON e.id = s.employee_id
WHERE (d.id, s.amount) IN (
    SELECT e.department_id, MAX(s.amount)
    FROM Employee e
    JOIN Salary s ON e.id = s.employee_id
    GROUP BY e.department_id
);
```

*Explanation*:
- The subquery finds the highest salary per department.
- The outer query filters employees who match that salary.

---

**Option B: Using CTE and RANK()**

This handles ties and is more scalable.

```sql
WITH RankedSalaries AS (
    SELECT d.name AS department_name, e.name AS employee_name, s.amount AS salary,
           RANK() OVER (PARTITION BY d.id ORDER BY s.amount DESC) AS rank
    FROM Department d
    JOIN Employee e ON d.id = e.department_id
    JOIN Salary s ON e.id = s.employee_id
)
SELECT department_name, employee_name, salary
FROM RankedSalaries
WHERE rank = 1;
```

*Explanation*:
- RANK() assigns a rank based on salary within each department.
- PARTITION BY d.id ensures ranking is scoped to each department.
- WHERE rank = 1 filters only top earners.

---

**Bonus Insight:**
> Using window functions like RANK or DENSE_RANK is preferred when multiple employees share the top salary. It also makes the query more readable and extensible for finding second or third highest salaries.

---
