Interview Questions (All were asked to WRITE the code/steps, not just explain):

1. Introduction

Start with: “Introduce yourself and your role in the project.”

2. API & Backend
(Q1) Write the complete steps/code for creating an HTTP POST API – taking a request and returning response in JSON.
(Q2) Show code/example for Dependency Injection and explain its lifecycle with implementation.
(Q3) Write a Custom Middleware (with Invoke method).
(Q4) Write different ways for API Versioning in .NET Core.
(Q5) Write code/steps to Upload & Retrieve an image from Azure Blob Storage.
(Q6) Write steps to create a gRPC service in .NET (with .proto file, service, and implementation).
(Q7) Write code to call a gRPC service from a .NET client.

3. C# & LINQ
(Q8) Write LINQ query to get 2nd Highest Salary (and other approaches).
(Q9) Difference with examples: IQueryable vs IEnumerable.
(Q10) Show sample code using ref and out parameters.

4. SQL
(Q11) Write steps/queries for SQL Performance tuning.
(Q12) Write query for Nth Highest Salary in SQL.
(Q13) Extended scenario → If multiple employees have same salary → write using Dense_Rank().

5. Angular (Frontend)
(Q14) Write basic Angular component code and show how components interact.

--- 

- In a 3-tier architecture performing CRUD operations, the Data Access Layer interacts with the database using ADO.NET commands and stored procedures.
- Because SqlConnection, SqlCommand, and ExecuteNonQuery are common across all CRUD methods, there’s a high chance of code duplication.
- Question: How can we design the DAL to avoid repeating the same code or commands for every CRUD method? 💡💬

--- 
𝗗𝗼𝘁 𝗡𝗲𝘁 𝗜𝗻𝘁𝗲𝗿𝘃𝗶𝗲𝘄 𝗧𝗶𝗽 𝗼𝗳 𝘁𝗵𝗲 𝗗𝗮𝘆
𝗤𝘂𝗲𝘀𝘁𝗶𝗼𝗻:
 What is garbage collection in .NET and how does it work? Why is it important?
𝗔𝗻𝘀𝘄𝗲𝗿:
 In .NET, garbage collection (GC) is a form of automatic memory management that helps developers by reclaiming memory occupied by objects that are no longer accessible in the application.
𝗛𝗼𝘄 𝗚𝗮𝗿𝗯𝗮𝗴𝗲 𝗖𝗼𝗹𝗹𝗲𝗰𝘁𝗶𝗼𝗻 𝗪𝗼𝗿𝗸𝘀:
The .NET runtime uses a tracing garbage collector, which periodically checks for objects on the managed heap that are no longer referenced by any part of the application.
When such objects are found, the GC automatically frees their memory, making space for new allocations.
The GC process is organized in "generations" (Gen 0, Gen 1, Gen 2), optimizing collection efficiency based on the expected lifetime of objects.
𝗕𝗲𝗻𝗲𝗳𝗶𝘁𝘀 𝗼𝗳 𝗚𝗮𝗿𝗯𝗮𝗴𝗲 𝗖𝗼𝗹𝗹𝗲𝗰𝘁𝗶𝗼𝗻:
Prevents memory leaks by reclaiming unused memory.
Reduces developer effort—no need for manual memory deallocation.
Improves application stability and performance over time.
𝗣𝗿𝗮𝗰𝘁𝗶𝗰𝗮𝗹 𝗘𝘅𝗮𝗺𝗽𝗹𝗲:
 When an object like this:

csharp
MyClass obj = new MyClass();
is no longer referenced in your code, the GC will eventually detect and clean it up automatically.
𝗧𝗶𝗽:
 Although GC is automatic, always dispose unmanaged resources (like file handles, database connections) by implementing IDisposable and calling .Dispose() or using using statements for optimal resource management.

--- 
