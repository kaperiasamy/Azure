# Advanced C# Interview Questions - Tech Lead/Solution Architect Preparation

## .NET Framework & CLR Concepts

### 1. Explain the .NET program execution flow and the role of Intermediate Language (IL).

**Answer:** The .NET program execution flow involves several stages:
1. Source code is compiled to Intermediate Language (IL) by language-specific compilers
2. IL code is stored in assemblies with metadata
3. At runtime, the CLR's Just-In-Time (JIT) compiler converts IL to native machine code
4. Native code is executed by the processor

**Explanation:** IL is a CPU-independent instruction set that allows language interoperability. When you compile C# code, it's not directly converted to machine code but to IL. This enables the "write once, run anywhere" philosophy within the .NET ecosystem.

```csharp
// C# Code
public int Add(int a, int b)
{
    return a + b;
}

// Becomes IL (simplified)
.method public hidebysig instance int32 Add(int32 a, int32 b) cil managed
{
    .maxstack 2
    ldarg.1
    ldarg.2
    add
    ret
}
```

**Follow-up Questions:**
- What are the advantages of JIT compilation over AOT compilation?
- How does the CLR optimize code during JIT compilation?

### 2. What is the difference between managed and unmanaged code, and how do they interact?

**Answer:** Managed code runs under the control of the CLR, which provides services like garbage collection, type safety, and exception handling. Unmanaged code runs directly on the operating system without CLR oversight.

**Explanation:** Managed code benefits from automatic memory management and type safety, while unmanaged code offers direct system access and potentially better performance for specific scenarios.

```csharp
// Managed code
public class ManagedClass
{
    private string data; // Memory managed by GC
    
    public void ProcessData()
    {
        data = "Hello World"; // GC handles allocation
    } // GC handles cleanup
}

// Interacting with unmanaged code
[DllImport("kernel32.dll")]
public static extern IntPtr GetModuleHandle(string lpModuleName);
```

**Follow-up Questions:**
- How does P/Invoke work for calling unmanaged code?
- What are the security implications of mixing managed and unmanaged code?

### 3. Explain the Garbage Collector in detail, including its generations and collection algorithms.

**Answer:** The .NET Garbage Collector is a generational, mark-and-sweep collector that automatically manages memory. It divides objects into three generations (0, 1, 2) based on their lifetime expectations.

**Explanation:** 
- **Generation 0:** Newly allocated objects, collected most frequently
- **Generation 1:** Objects that survived one GC cycle
- **Generation 2:** Long-lived objects, collected least frequently

```csharp
public class GCExample
{
    public void DemonstrateGC()
    {
        // These objects start in Gen 0
        var shortLived = new List<int>();
        var anotherShortLived = new StringBuilder();
        
        // Force GC to see generation promotion
        GC.Collect(0); // Collect Gen 0
        
        // Objects that survive move to Gen 1
        Console.WriteLine($"Generation of shortLived: {GC.GetGeneration(shortLived)}");
        
        // Can also check GC statistics
        Console.WriteLine($"Gen 0 collections: {GC.CollectionCount(0)}");
    }
}
```

**Follow-up Questions:**
- When should you explicitly call GC.Collect() and why is it generally discouraged?
- How do weak references work and when would you use them?

### 4. What is an AppDomain and how does it provide isolation?

**Answer:** An AppDomain is a logical isolation boundary within a process that provides security, versioning, and fault isolation for applications running in the same process.

**Explanation:** AppDomains allow multiple applications to run in a single process while maintaining isolation. Each AppDomain has its own security context, configuration, and can be unloaded independently.

```csharp
public class AppDomainExample
{
    public void CreateAndUseAppDomain()
    {
        // Create new AppDomain
        AppDomain newDomain = AppDomain.CreateDomain("IsolatedDomain");
        
        try
        {
            // Load and execute code in the new domain
            var proxy = (RemoteClass)newDomain.CreateInstanceAndUnwrap(
                typeof(RemoteClass).Assembly.FullName,
                typeof(RemoteClass).FullName);
                
            proxy.DoWork();
        }
        finally
        {
            // Clean up
            AppDomain.Unload(newDomain);
        }
    }
}

public class RemoteClass : MarshalByRefObject
{
    public void DoWork()
    {
        Console.WriteLine($"Working in domain: {AppDomain.CurrentDomain.FriendlyName}");
    }
}
```

**Follow-up Questions:**
- How has .NET Core changed the AppDomain model?
- What are the performance implications of cross-AppDomain calls?

## Advanced C# Concepts

### 5. Explain the difference between delegates, events, and Func/Action types.

**Answer:** 
- **Delegate:** Type-safe function pointer that can hold references to methods
- **Event:** Special delegate that provides publisher-subscriber pattern with access restrictions
- **Func/Action:** Generic delegate types; Func returns a value, Action doesn't

**Explanation:** Delegates enable functional programming concepts, events provide encapsulation for notifications, and Func/Action are predefined generic delegates for common scenarios.

```csharp
// Custom delegate
public delegate void CustomDelegate(string message);

public class DelegateExample
{
    // Event based on delegate
    public event CustomDelegate OnMessage;
    
    // Using Func and Action
    public void DemonstrateTypes()
    {
        // Custom delegate
        CustomDelegate del = Console.WriteLine;
        del += (msg) => Debug.WriteLine(msg);
        
        // Action (void return)
        Action<string> action = Console.WriteLine;
        
        // Func (with return value)
        Func<int, int, int> func = (a, b) => a + b;
        
        // Event usage
        OnMessage += (msg) => Console.WriteLine($"Event: {msg}");
        OnMessage?.Invoke("Hello");
    }
}
```

**Follow-up Questions:**
- How do multicast delegates handle exceptions?
- What's the difference between events and observable patterns?

### 6. Explain async/await and Task-based asynchronous programming in detail.

**Answer:** Async/await enables writing asynchronous code that reads like synchronous code. Tasks represent asynchronous operations and can be awaited to get results without blocking threads.

**Explanation:** The async keyword marks a method as asynchronous, and await unwraps Task results while yielding control back to the caller until the operation completes.

```csharp
public class AsyncExample
{
    public async Task<string> FetchDataAsync()
    {
        using var client = new HttpClient();
        
        // This doesn't block the calling thread
        var response = await client.GetStringAsync("https://api.example.com/data");
        
        // Process the response
        var processed = await ProcessDataAsync(response);
        return processed;
    }
    
    private async Task<string> ProcessDataAsync(string data)
    {
        await Task.Delay(1000); // Simulate async work
        return data.ToUpper();
    }
    
    // Multiple async operations
    public async Task<string[]> FetchMultipleAsync()
    {
        var task1 = FetchDataAsync();
        var task2 = FetchDataAsync();
        var task3 = FetchDataAsync();
        
        // Wait for all to complete
        return await Task.WhenAll(task1, task2, task3);
    }
}
```

**Follow-up Questions:**
- What happens if you don't await a Task?
- How does ConfigureAwait(false) affect execution context?

### 7. Demonstrate advanced generic concepts including constraints and variance.

**Answer:** Generics provide type safety and performance benefits. Constraints limit generic types, while variance (covariance/contravariance) enables flexible type relationships.

**Explanation:** Generic constraints ensure type safety at compile time. Covariance (out) allows returning more derived types, contravariance (in) allows accepting more base types.

```csharp
// Generic class with constraints
public class Repository<T> where T : class, IEntity, new()
{
    private List<T> items = new List<T>();
    
    public void Add(T item)
    {
        items.Add(item);
    }
    
    public T CreateNew()
    {
        return new T(); // Possible due to new() constraint
    }
}

// Variance examples
public interface ICovariant<out T> // Covariant
{
    T Get();
}

public interface IContravariant<in T> // Contravariant
{
    void Set(T value);
}

// Usage
public class VarianceExample
{
    public void DemonstrateVariance()
    {
        // Covariance - can assign more derived to less derived
        ICovariant<string> covariantString = GetStringProvider();
        ICovariant<object> covariantObject = covariantString; // Valid
        
        // Contravariance - can assign less derived to more derived
        IContravariant<object> contravariantObject = GetObjectConsumer();
        IContravariant<string> contravariantString = contravariantObject; // Valid
    }
    
    private ICovariant<string> GetStringProvider() => null;
    private IContravariant<object> GetObjectConsumer() => null;
}
```

**Follow-up Questions:**
- When would you use generic methods vs generic classes?
- How do generic constraints affect JIT compilation?

### 8. Explain LINQ internals and deferred execution.

**Answer:** LINQ provides a unified query syntax across different data sources. Deferred execution means queries are not executed until results are enumerated.

**Explanation:** LINQ queries return IEnumerable that implements lazy evaluation. The query is built as an expression tree and executed only when enumerated.

```csharp
public class LinqExample
{
    public void DemonstrateLinq()
    {
        var numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
        
        // Deferred execution - query not executed yet
        var evenNumbers = numbers.Where(n => 
        {
            Console.WriteLine($"Processing {n}");
            return n % 2 == 0;
        });
        
        Console.WriteLine("Query defined, not executed");
        
        // Now the query executes
        foreach (var num in evenNumbers)
        {
            Console.WriteLine($"Result: {num}");
        }
        
        // Custom LINQ extension
        var customFiltered = numbers.CustomWhere(n => n > 5);
    }
}

public static class LinqExtensions
{
    public static IEnumerable<T> CustomWhere<T>(this IEnumerable<T> source, Func<T, bool> predicate)
    {
        foreach (var item in source)
        {
            if (predicate(item))
                yield return item;
        }
    }
}
```

**Follow-up Questions:**
- What's the difference between IEnumerable and IQueryable?
- How do you optimize LINQ queries for performance?

### 9. Explain reflection and its use cases, including performance considerations.

**Answer:** Reflection allows examining and manipulating types, members, and objects at runtime. It's powerful but has performance implications due to runtime type checking.

**Explanation:** Reflection enables dynamic programming scenarios like serialization, dependency injection, and attribute-based programming, but should be used judiciously due to performance costs.

```csharp
public class ReflectionExample
{
    public void DemonstrateReflection()
    {
        var type = typeof(Person);
        
        // Get type information
        Console.WriteLine($"Type name: {type.Name}");
        Console.WriteLine($"Namespace: {type.Namespace}");
        
        // Create instance
        var person = Activator.CreateInstance(type);
        
        // Get and set properties
        var nameProperty = type.GetProperty("Name");
        nameProperty?.SetValue(person, "John Doe");
        
        // Get and invoke methods
        var greetMethod = type.GetMethod("Greet");
        greetMethod?.Invoke(person, null);
        
        // Work with attributes
        var attributes = type.GetCustomAttributes(typeof(SerializableAttribute), false);
        Console.WriteLine($"Is serializable: {attributes.Length > 0}");
    }
    
    // Performance-optimized reflection using delegates
    private static readonly Func<Person, string> GetNameFast = CreateGetterDelegate<Person, string>("Name");
    
    private static Func<T, TProperty> CreateGetterDelegate<T, TProperty>(string propertyName)
    {
        var property = typeof(T).GetProperty(propertyName);
        var parameter = Expression.Parameter(typeof(T), "obj");
        var propertyAccess = Expression.Property(parameter, property);
        return Expression.Lambda<Func<T, TProperty>>(propertyAccess, parameter).Compile();
    }
}

[Serializable]
public class Person
{
    public string Name { get; set; }
    
    public void Greet()
    {
        Console.WriteLine($"Hello, I'm {Name}");
    }
}
```

**Follow-up Questions:**
- How can you cache reflection results for better performance?
- What are the security implications of reflection?

### 10. Explain exception handling best practices and custom exceptions.

**Answer:** Exception handling should be used for exceptional conditions, not control flow. Custom exceptions should provide meaningful context and follow .NET naming conventions.

**Explanation:** Proper exception handling improves application robustness. Custom exceptions should inherit from appropriate base classes and provide relevant information.

```csharp
// Custom exception hierarchy
public class BusinessLogicException : Exception
{
    public string ErrorCode { get; }
    
    public BusinessLogicException(string errorCode, string message) : base(message)
    {
        ErrorCode = errorCode;
    }
    
    public BusinessLogicException(string errorCode, string message, Exception innerException) 
        : base(message, innerException)
    {
        ErrorCode = errorCode;
    }
}

public class ValidationException : BusinessLogicException
{
    public Dictionary<string, string> ValidationErrors { get; }
    
    public ValidationException(Dictionary<string, string> errors) 
        : base("VALIDATION_ERROR", "Validation failed")
    {
        ValidationErrors = errors;
    }
}

public class ExceptionHandlingExample
{
    public async Task<Result<T>> SafeExecuteAsync<T>(Func<Task<T>> operation)
    {
        try
        {
            var result = await operation();
            return Result<T>.Success(result);
        }
        catch (ValidationException ex)
        {
            // Log specific validation errors
            LogValidationErrors(ex.ValidationErrors);
            return Result<T>.Failure($"Validation failed: {ex.Message}");
        }
        catch (BusinessLogicException ex)
        {
            // Log business logic errors
            LogError(ex.ErrorCode, ex.Message);
            return Result<T>.Failure(ex.Message);
        }
        catch (Exception ex)
        {
            // Log unexpected errors
            LogError("UNEXPECTED_ERROR", ex.Message);
            return Result<T>.Failure("An unexpected error occurred");
        }
    }
    
    private void LogValidationErrors(Dictionary<string, string> errors) { /* Implementation */ }
    private void LogError(string errorCode, string message) { /* Implementation */ }
}

// Result pattern for better error handling
public class Result<T>
{
    public bool IsSuccess { get; private set; }
    public T Value { get; private set; }
    public string Error { get; private set; }
    
    private Result(bool isSuccess, T value, string error)
    {
        IsSuccess = isSuccess;
        Value = value;
        Error = error;
    }
    
    public static Result<T> Success(T value) => new Result<T>(true, value, null);
    public static Result<T> Failure(string error) => new Result<T>(false, default(T), error);
}
```

**Follow-up Questions:**
- When should you use checked vs unchecked exceptions?
- How do you handle exceptions in async methods?

## Multithreading and Concurrency

### 11. Explain thread safety and different synchronization mechanisms.

**Answer:** Thread safety ensures that shared data remains consistent when accessed by multiple threads. .NET provides various synchronization primitives like lock, Monitor, Mutex, Semaphore, and concurrent collections.

**Explanation:** Different synchronization mechanisms serve different purposes: locks for mutual exclusion, semaphores for resource counting, and concurrent collections for lock-free operations.

```csharp
public class ThreadSafetyExample
{
    private readonly object lockObject = new object();
    private readonly ReaderWriterLockSlim readerWriterLock = new ReaderWriterLockSlim();
    private readonly SemaphoreSlim semaphore = new SemaphoreSlim(3, 3); // Max 3 concurrent operations
    private readonly ConcurrentDictionary<string, int> concurrentDict = new ConcurrentDictionary<string, int>();
    
    private int counter = 0;
    private List<string> data = new List<string>();
    
    // Basic lock synchronization
    public void IncrementCounter()
    {
        lock (lockObject)
        {
            counter++;
        }
    }
    
    // Reader-writer lock for read-heavy scenarios
    public void ReadData()
    {
        readerWriterLock.EnterReadLock();
        try
        {
            // Multiple readers can access simultaneously
            var count = data.Count;
            Console.WriteLine($"Data count: {count}");
        }
        finally
        {
            readerWriterLock.ExitReadLock();
        }
    }
    
    public void WriteData(string item)
    {
        readerWriterLock.EnterWriteLock();
        try
        {
            // Only one writer at a time
            data.Add(item);
        }
        finally
        {
            readerWriterLock.ExitWriteLock();
        }
    }
    
    // Semaphore for resource limiting
    public async Task ProcessWithSemaphoreAsync()
    {
        await semaphore.WaitAsync();
        try
        {
            // Only 3 operations can run concurrently
            await Task.Delay(1000);
            Console.WriteLine("Processing completed");
        }
        finally
        {
            semaphore.Release();
        }
    }
    
    // Lock-free operations with concurrent collections
    public void UpdateConcurrentData(string key, int value)
    {
        concurrentDict.AddOrUpdate(key, value, (k, oldValue) => oldValue + value);
    }
}
```

**Follow-up Questions:**
- What are the performance differences between different lock types?
- When would you use volatile keyword vs locks?

### 12. Explain the Task Parallel Library (TPL) and parallel programming patterns.

**Answer:** TPL provides high-level abstractions for parallel and concurrent programming, including Parallel class, PLINQ, and advanced task scheduling.

**Explanation:** TPL automatically manages thread creation, work distribution, and synchronization, making parallel programming more accessible and efficient.

```csharp
public class ParallelProgrammingExample
{
    public void DemonstrateParallelPatterns()
    {
        var numbers = Enumerable.Range(1, 1000000).ToArray();
        
        // Parallel.For
        var results = new int[numbers.Length];
        Parallel.For(0, numbers.Length, i =>
        {
            results[i] = ExpensiveOperation(numbers[i]);
        });
        
        // Parallel.ForEach with partitioning
        var partitioner = Partitioner.Create(numbers, true);
        Parallel.ForEach(partitioner, number =>
        {
            ProcessNumber(number);
        });
        
        // PLINQ (Parallel LINQ)
        var parallelQuery = numbers
            .AsParallel()
            .WithDegreeOfParallelism(Environment.ProcessorCount)
            .Where(n => IsPrime(n))
            .Select(n => n * n)
            .OrderBy(n => n);
            
        // Task-based parallelism
        var tasks = new List<Task<int>>();
        for (int i = 0; i < 10; i++)
        {
            int localI = i; // Capture loop variable
            tasks.Add(Task.Run(() => ExpensiveOperation(localI)));
        }
        
        var taskResults = await Task.WhenAll(tasks);
        
        // Producer-Consumer pattern with BlockingCollection
        using var collection = new BlockingCollection<int>();
        
        // Producer task
        var producer = Task.Run(() =>
        {
            for (int i = 0; i < 100; i++)
            {
                collection.Add(i);
                Thread.Sleep(10);
            }
            collection.CompleteAdding();
        });
        
        // Consumer tasks
        var consumers = Enumerable.Range(0, 3)
            .Select(i => Task.Run(() =>
            {
                foreach (var item in collection.GetConsumingEnumerable())
                {
                    ProcessItem(item);
                }
            }))
            .ToArray();
            
        await Task.WhenAll(consumers);
    }
    
    private int ExpensiveOperation(int input) => input * input; // Simulate work
    private void ProcessNumber(int number) { /* Implementation */ }
    private void ProcessItem(int item) { /* Implementation */ }
    private bool IsPrime(int number) { /* Implementation */ return true; }
}
```

**Follow-up Questions:**
- How does work-stealing work in the TPL?
- What are the trade-offs between data parallelism and task parallelism?

### 13. Explain memory models and the volatile keyword.

**Answer:** The memory model defines how threads interact through shared memory. The volatile keyword prevents certain compiler optimizations and ensures visibility of changes across threads.

**Explanation:** Modern processors and compilers can reorder operations for performance. Volatile ensures that reads/writes are not optimized away and provides ordering guarantees.

```csharp
public class MemoryModelExample
{
    private bool isComplete = false;
    private volatile bool isCompleteVolatile = false;
    private int result = 0;
    
    // Without volatile - may not work as expected
    public void ProducerNonVolatile()
    {
        result = 42;
        isComplete = true; // Compiler might reorder this
    }
    
    public void ConsumerNonVolatile()
    {
        while (!isComplete)
        {
            Thread.Yield();
        }
        Console.WriteLine($"Result: {result}"); // Might be 0!
    }
    
    // With volatile - ensures visibility
    public void ProducerVolatile()
    {
        result = 42;
        isCompleteVolatile = true; // Cannot be reordered past this point
    }
    
    public void ConsumerVolatile()
    {
        while (!isCompleteVolatile)
        {
            Thread.Yield();
        }
        Console.WriteLine($"Result: {result}"); // Will be 42
    }
    
    // Better approach using memory barriers
    public void ProducerWithBarrier()
    {
        result = 42;
        Thread.MemoryBarrier(); // Full memory barrier
        isComplete = true;
    }
    
    public void ConsumerWithBarrier()
    {
        while (!isComplete)
        {
            Thread.Yield();
        }
        Thread.MemoryBarrier(); // Ensure we see latest values
        Console.WriteLine($"Result: {result}");
    }
    
    // Modern approach with Interlocked
    private int atomicValue = 0;
    
    public void AtomicOperations()
    {
        // Thread-safe increment
        Interlocked.Increment(ref atomicValue);
        
        // Thread-safe compare and swap
        int originalValue = atomicValue;
        int newValue = originalValue + 10;
        Interlocked.CompareExchange(ref atomicValue, newValue, originalValue);
        
        // Thread-safe read
        int currentValue = Interlocked.Read(ref atomicValue);
    }
}
```

**Follow-up Questions:**
- How do memory barriers affect performance?
- What's the difference between volatile and Interlocked operations?

## Advanced Design Patterns and Architecture

### 14. Implement a thread-safe Singleton pattern with lazy initialization.

**Answer:** The Singleton pattern ensures only one instance exists. Thread safety and lazy initialization require careful implementation to avoid performance issues.

**Explanation:** Multiple approaches exist: double-checked locking, Lazy<T>, and static initialization. Each has different characteristics regarding thread safety and performance.

```csharp
// Approach 1: Thread-safe with double-checked locking
public sealed class SingletonDoubleCheck
{
    private static volatile SingletonDoubleCheck instance;
    private static readonly object lockObject = new object();
    
    private SingletonDoubleCheck() { }
    
    public static SingletonDoubleCheck Instance
    {
        get
        {
            if (instance == null)
            {
                lock (lockObject)
                {
                    if (instance == null)
                        instance = new SingletonDoubleCheck();
                }
            }
            return instance;
        }
    }
}

// Approach 2: Using Lazy<T> (Recommended)
public sealed class SingletonLazy
{
    private static readonly Lazy<SingletonLazy> lazy = 
        new Lazy<SingletonLazy>(() => new SingletonLazy());
    
    private SingletonLazy() { }
    
    public static SingletonLazy Instance => lazy.Value;
}

// Approach 3: Static initialization
public sealed class SingletonStatic
{
    private static readonly SingletonStatic instance = new SingletonStatic();
    
    static SingletonStatic() { }
    private SingletonStatic() { }
    
    public static SingletonStatic Instance => instance;
}

// Generic Singleton base class
public abstract class Singleton<T> where T : class, new()
{
    private static readonly Lazy<T> instance = new Lazy<T>(() => new T());
    
    public static T Instance => instance.Value;
}

// Usage example
public class DatabaseConnection : Singleton<DatabaseConnection>
{
    private string connectionString;
    
    public DatabaseConnection()
    {
        connectionString = "Data Source=...";
    }
    
    public void Connect()
    {
        Console.WriteLine($"Connecting with: {connectionString}");
    }
}
```

**Follow-up Questions:**
- What are the disadvantages of the Singleton pattern?
- How do you unit test code that uses Singletons?

### 15. Implement the Observer pattern with weak references to prevent memory leaks.

**Answer:** The Observer pattern defines a one-to-many dependency between objects. Weak references prevent memory leaks by allowing observers to be garbage collected.

**Explanation:** Traditional event handling can create strong references that prevent garbage collection. Weak references solve this while maintaining the observer functionality.

```csharp
public interface IObserver<T>
{
    void OnNext(T value);
    void OnError(Exception error);
    void OnCompleted();
}

public class WeakObservableList<T> : IDisposable
{
    private readonly List<WeakReference<IObserver<T>>> observers = new List<WeakReference<IObserver<T>>>();
    private readonly object lockObject = new object();
    
    public IDisposable Subscribe(IObserver<T> observer)
    {
        lock (lockObject)
        {
            observers.Add(new WeakReference<IObserver<T>>(observer));
            CleanupDeadReferences();
        }
        
        return new Unsubscriber<T>(this, observer);
    }
    
    public void NotifyNext(T value)
    {
        var liveObservers = GetLiveObservers();
        foreach (var observer in liveObservers)
        {
            try
            {
                observer.OnNext(value);
            }
            catch (Exception ex)
            {
                observer.OnError(ex);
            }
        }
    }
    
    public void NotifyCompleted()
    {
        var liveObservers = GetLiveObservers();
        foreach (var observer in liveObservers)
        {
            observer.OnCompleted();
        }
        
        lock (lockObject)
        {
            observers.Clear();
        }
    }
    
    private List<IObserver<T>> GetLiveObservers()
    {
        var liveObservers = new List<IObserver<T>>();
        
        lock (lockObject)
        {
            for (int i = observers.Count - 1; i >= 0; i--)
            {
                if (observers[i].TryGetTarget(out var observer))
                {
                    liveObservers.Add(observer);
                }
                else
                {
                    observers.RemoveAt(i); // Clean up dead reference
                }
            }
        }
        
        return liveObservers;
    }
    
    private void CleanupDeadReferences()
    {
        for (int i = observers.Count - 1; i >= 0; i--)
        {
            if (!observers[i].TryGetTarget(out _))
            {
                observers.RemoveAt(i);
            }
        }
    }
    
    internal void Unsubscribe(IObserver<T> observer)
    {
        lock (lockObject)
        {
            for (int i = observers.Count - 1; i >= 0; i--)
            {
                if (observers[i].TryGetTarget(out var target) && ReferenceEquals(target, observer))
                {
                    observers.RemoveAt(i);
                    break;
                }
            }
        }
    }
    
    public void Dispose()
    {
        lock (lockObject)
        {
            observers.Clear();
        }
    }
}

internal class Unsubscriber<T> : IDisposable
{
    private readonly WeakObservableList<T> observable;
    private readonly IObserver<T> observer;
    
    public Unsubscriber(WeakObservableList<T> observable, IObserver<T> observer)
    {
        this.observable = observable;
        this.observer = observer;
    }
    
    public void Dispose()
    {
        observable.Unsubscribe(observer);
    }
}

// Usage example
public class TemperatureSensor : WeakObservableList<double>
{
    public void ReportTemperature(double temperature)
    {
        NotifyNext(temperature);
    }
}

public class TemperatureDisplay : IObserver<double>
{
    public void OnNext(double temperature)
    {
        Console.WriteLine($"Temperature: {temperature}°C");
    }
    
    public void OnError(Exception error)
    {
        Console.WriteLine($"Error: {error.Message}");
    }
    
    public void OnCompleted()
    {
        Console.WriteLine("Temperature monitoring completed");
    }
}
```

**Follow-up Questions:**
- How does this compare to .NET's built-in IObservable<T>?
- What are the performance implications of weak references?

### 16. Implement a Command pattern with undo/redo functionality.

**Answer:** The Command pattern encapsulates requests as objects, enabling parameterization, queuing, logging, and undo operations.

**Explanation:** Commands separate the object making the request from the object handling it. This enables complex operations like macro recording, undo/redo, and transactional behavior.

```csharp
public interface ICommand
{
    void Execute();
    void Undo();
    string Description { get; }
}

public interface IUndoableCommand : ICommand
{
    bool CanUndo { get; }
}

// Concrete command example
public class TransferMoneyCommand : IUndoableCommand
{
    private readonly Account fromAccount;
    private readonly Account toAccount;
    private readonly decimal amount;
    private bool executed = false;
    
    public string Description => $"Transfer ${amount} from {fromAccount.Name} to {toAccount.Name}";
    public bool CanUndo => executed;
    
    public TransferMoneyCommand(Account from, Account to, decimal amount)
    {
        fromAccount = from;
        toAccount = to;
        this.amount = amount;
    }
    
    public void Execute()
    {
        if (fromAccount.Balance < amount)
            throw new InvalidOperationException("Insufficient funds");
            
        fromAccount.Withdraw(amount);
        toAccount.Deposit(amount);
        executed = true;
    }
    
    public void Undo()
    {
        if (!CanUndo)
            throw new InvalidOperationException("Cannot undo command that hasn't been executed");
            
        toAccount.Withdraw(amount);
        fromAccount.Deposit(amount);
        executed = false;
    }
}

// Command manager with undo/redo
public class CommandManager
{
    private readonly Stack<IUndoableCommand> undoStack = new Stack<IUndoableCommand>();
    private readonly Stack<IUndoableCommand> redoStack = new Stack<IUndoableCommand>();
    private readonly int maxHistorySize;
    
    public CommandManager(int maxHistorySize = 50)
    {
        this.maxHistorySize = maxHistorySize;
    }
    
    public void ExecuteCommand(IUndoableCommand command)
    {
        try
        {
            command.Execute();
            undoStack.Push(command);
            redoStack.Clear(); // Clear redo stack when new command is executed
            
            // Limit history size
            if (undoStack.Count > maxHistorySize)
            {
                var commands = undoStack.ToArray();
                undoStack.Clear();
                for (int i = commands.Length - maxHistorySize; i < commands.Length; i++)
                {
                    undoStack.Push(commands[i]);
                }
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Command execution failed: {ex.Message}");
            throw;
        }
    }
    
    public bool CanUndo => undoStack.Count > 0;
    public bool CanRedo => redoStack.Count > 0;
    
    public void Undo()
    {
        if (!CanUndo)
            throw new InvalidOperationException("Nothing to undo");
            
        var command = undoStack.Pop();
        command.Undo();
        redoStack.Push(command);
    }
    
    public void Redo()
    {
        if (!CanRedo)
            throw new InvalidOperationException("Nothing to redo");
            
        var command = redoStack.Pop();
        command.Execute();
        undoStack.Push(command);
    }
    
    public IEnumerable<string> GetUndoHistory()
    {
        return undoStack.Select(cmd => cmd.Description);
    }
}

// Macro command for composite operations
public class MacroCommand : IUndoableCommand
{
    private readonly List<IUndoableCommand> commands = new List<IUndoableCommand>();
    private readonly List<IUndoableCommand> executedCommands = new List<IUndoableCommand>();
    
    public string Description { get; private set; }
    public bool CanUndo => executedCommands.Count > 0;
    
    public MacroCommand(string description)
    {
        Description = description;
    }
    
    public void AddCommand(IUndoableCommand command)
    {
        commands.Add(command);
    }
    
    public void Execute()
    {
        foreach (var command in commands)
        {
            try
            {
                command.Execute();
                executedCommands.Add(command);
            }
            catch
            {
                // Undo already executed commands
                Undo();
                throw;
            }
        }
    }
    
    public void Undo()
    {
        // Undo in reverse order
        for (int i = executedCommands.Count - 1; i >= 0; i--)
        {
            executedCommands[i].Undo();
        }
        executedCommands.Clear();
    }
}

public class Account
{
    public string Name { get; }
    public decimal Balance { get; private set; }
    
    public Account(string name, decimal initialBalance)
    {
        Name = name;
        Balance = initialBalance;
    }
    
    public void Deposit(decimal amount)
    {
        Balance += amount;
    }
    
    public void Withdraw(decimal amount)
    {
        Balance -= amount;
    }
}
```

**Follow-up Questions:**
- How would you implement command serialization for persistence?
- What are the memory implications of keeping unlimited command history?

### 17. Explain dependency injection patterns and implement a simple IoC container.

**Answer:** Dependency Injection is a design pattern that implements Inversion of Control, allowing dependencies to be injected rather than created internally. This improves testability and flexibility.

**Explanation:** DI containers manage object lifetimes and resolve dependencies automatically. Different lifetime scopes (Singleton, Transient, Scoped) serve different purposes.

```csharp
public interface IContainer
{
    void Register<TInterface, TImplementation>(ServiceLifetime lifetime = ServiceLifetime.Transient)
        where TImplementation : class, TInterface;
    void Register<T>(Func<IContainer, T> factory, ServiceLifetime lifetime = ServiceLifetime.Transient);
    T Resolve<T>();
    object Resolve(Type type);
}

public enum ServiceLifetime
{
    Transient,
    Singleton,
    Scoped
}

public class ServiceDescriptor
{
    public Type ServiceType { get; set; }
    public Type ImplementationType { get; set; }
    public Func<IContainer, object> Factory { get; set; }
    public ServiceLifetime Lifetime { get; set; }
    public object SingletonInstance { get; set; }
}

public class SimpleContainer : IContainer, IDisposable
{
    private readonly Dictionary<Type, ServiceDescriptor> services = new Dictionary<Type, ServiceDescriptor>();
    private readonly Dictionary<Type, object> scopedInstances = new Dictionary<Type, object>();
    private readonly object lockObject = new object();
    
    public void Register<TInterface, TImplementation>(ServiceLifetime lifetime = ServiceLifetime.Transient)
        where TImplementation : class, TInterface
    {
        lock (lockObject)
        {
            services[typeof(TInterface)] = new ServiceDescriptor
            {
                ServiceType = typeof(TInterface),
                ImplementationType = typeof(TImplementation),
                Lifetime = lifetime
            };
        }
    }
    
    public void Register<T>(Func<IContainer, T> factory, ServiceLifetime lifetime = ServiceLifetime.Transient)
    {
        lock (lockObject)
        {
            services[typeof(T)] = new ServiceDescriptor
            {
                ServiceType = typeof(T),
                Factory = container => factory(container),
                Lifetime = lifetime
            };
        }
    }
    
    public T Resolve<T>()
    {
        return (T)Resolve(typeof(T));
    }
    
    public object Resolve(Type type)
    {
        if (!services.TryGetValue(type, out var serviceDescriptor))
        {
            throw new InvalidOperationException($"Service of type {type.Name} is not registered");
        }
        
        return serviceDescriptor.Lifetime switch
        {
            ServiceLifetime.Singleton => GetSingleton(serviceDescriptor),
            ServiceLifetime.Scoped => GetScoped(serviceDescriptor),
            ServiceLifetime.Transient => CreateInstance(serviceDescriptor),
            _ => throw new ArgumentOutOfRangeException()
        };
    }
    
    private object GetSingleton(ServiceDescriptor descriptor)
    {
        if (descriptor.SingletonInstance == null)
        {
            lock (lockObject)
            {
                if (descriptor.SingletonInstance == null)
                {
                    descriptor.SingletonInstance = CreateInstance(descriptor);
                }
            }
        }
        return descriptor.SingletonInstance;
    }
    
    private object GetScoped(ServiceDescriptor descriptor)
    {
        if (!scopedInstances.TryGetValue(descriptor.ServiceType, out var instance))
        {
            instance = CreateInstance(descriptor);
            scopedInstances[descriptor.ServiceType] = instance;
        }
        return instance;
    }
    
    private object CreateInstance(ServiceDescriptor descriptor)
    {
        if (descriptor.Factory != null)
        {
            return descriptor.Factory(this);
        }
        
        var constructors = descriptor.ImplementationType.GetConstructors();
        var constructor = constructors.OrderByDescending(c => c.GetParameters().Length).First();
        
        var parameters = constructor.GetParameters()
            .Select(p => Resolve(p.ParameterType))
            .ToArray();
            
        return Activator.CreateInstance(descriptor.ImplementationType, parameters);
    }
    
    public IContainer CreateScope()
    {
        return new SimpleContainer { services = this.services };
    }
    
    public void Dispose()
    {
        foreach (var instance in scopedInstances.Values.OfType<IDisposable>())
        {
            instance.Dispose();
        }
        scopedInstances.Clear();
    }
}

// Example usage
public interface IRepository<T>
{
    Task<T> GetByIdAsync(int id);
    Task SaveAsync(T entity);
}

public class DatabaseRepository<T> : IRepository<T>
{
    private readonly IDbContext dbContext;
    
    public DatabaseRepository(IDbContext dbContext)
    {
        this.dbContext = dbContext;
    }
    
    public async Task<T> GetByIdAsync(int id)
    {
        // Implementation
        return default(T);
    }
    
    public async Task SaveAsync(T entity)
    {
        // Implementation
    }
}

public interface IDbContext
{
    // Database context interface
}

public class ApplicationDbContext : IDbContext
{
    // Implementation
}

// Container setup
public class ContainerSetup
{
    public static IContainer ConfigureServices()
    {
        var container = new SimpleContainer();
        
        container.Register<IDbContext, ApplicationDbContext>(ServiceLifetime.Scoped);
        container.Register(typeof(IRepository<>), typeof(DatabaseRepository<>), ServiceLifetime.Transient);
        
        // Factory registration
        container.Register<IComplexService>(c => 
            new ComplexService(c.Resolve<IRepository<User>>(), "configuration"), 
            ServiceLifetime.Singleton);
            
        return container;
    }
}
```

**Follow-up Questions:**
- How do you handle circular dependencies in DI containers?
- What are the performance implications of reflection-based DI?

### 18. Implement a generic Repository pattern with Unit of Work.

**Answer:** The Repository pattern encapsulates data access logic, while Unit of Work maintains a list of objects affected by a business transaction and coordinates writing changes.

**Explanation:** This pattern provides abstraction over data access and ensures consistency by treating multiple operations as a single transaction.

```csharp
public interface IEntity
{
    int Id { get; set; }
}

public interface IRepository<T> where T : class, IEntity
{
    Task<T> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task<IEnumerable<T>> FindAsync(Expression<Func<T, bool>> predicate);
    Task AddAsync(T entity);
    void Update(T entity);
    void Remove(T entity);
    Task<bool> ExistsAsync(int id);
}

public interface IUnitOfWork : IDisposable
{
    IRepository<T> Repository<T>() where T : class, IEntity;
    Task<int> SaveChangesAsync();
    Task BeginTransactionAsync();
    Task CommitTransactionAsync();
    Task RollbackTransactionAsync();
}

public class GenericRepository<T> : IRepository<T> where T : class, IEntity
{
    protected readonly DbContext context;
    protected readonly DbSet<T> dbSet;
    
    public GenericRepository(DbContext context)
    {
        this.context = context;
        dbSet = context.Set<T>();
    }
    
    public async Task<T> GetByIdAsync(int id)
    {
        return await dbSet.FindAsync(id);
    }
    
    public async Task<IEnumerable<T>> GetAllAsync()
    {
        return await dbSet.ToListAsync();
    }
    
    public async Task<IEnumerable<T>> FindAsync(Expression<Func<T, bool>> predicate)
    {
        return await dbSet.Where(predicate).ToListAsync();
    }
    
    public async Task AddAsync(T entity)
    {
        await dbSet.AddAsync(entity);
    }
    
    public void Update(T entity)
    {
        dbSet.Update(entity);
    }
    
    public void Remove(T entity)
    {
        dbSet.Remove(entity);
    }
    
    public async Task<bool> ExistsAsync(int id)
    {
        return await dbSet.AnyAsync(e => e.Id == id);
    }
}

public class UnitOfWork : IUnitOfWork
{
    private readonly DbContext context;
    private readonly Dictionary<Type, object> repositories;
    private IDbContextTransaction transaction;
    
    public UnitOfWork(DbContext context)
    {
        this.context = context;
        repositories = new Dictionary<Type, object>();
    }
    
    public IRepository<T> Repository<T>() where T : class, IEntity
    {
        var type = typeof(T);
        if (!repositories.ContainsKey(type))
        {
            repositories[type] = new GenericRepository<T>(context);
        }
        return (IRepository<T>)repositories[type];
    }
    
    public async Task<int> SaveChangesAsync()
    {
        try
        {
            return await context.SaveChangesAsync();
        }
        catch (DbUpdateConcurrencyException ex)
        {
            // Handle concurrency conflicts
            foreach (var entry in ex.Entries)
            {
                var proposedValues = entry.CurrentValues;
                var databaseValues = await entry.GetDatabaseValuesAsync();
                
                if (databaseValues == null)
                {
                    // Entity was deleted
                    entry.State = EntityState.Detached;
                }
                else
                {
                    // Handle conflict resolution
                    entry.OriginalValues.SetValues(databaseValues);
                }
            }
            
            return await context.SaveChangesAsync();
        }
    }
    
    public async Task BeginTransactionAsync()
    {
        transaction = await context.Database.BeginTransactionAsync();
    }
    
    public async Task CommitTransactionAsync()
    {
        if (transaction != null)
        {
            await transaction.CommitAsync();
            await transaction.DisposeAsync();
            transaction = null;
        }
    }
    
    public async Task RollbackTransactionAsync()
    {
        if (transaction != null)
        {
            await transaction.RollbackAsync();
            await transaction.DisposeAsync();
            transaction = null;
        }
    }
    
    public void Dispose()
    {
        transaction?.Dispose();
        context?.Dispose();
    }
}

// Domain entities
public class User : IEntity
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    public List<Order> Orders { get; set; } = new List<Order>();
}

public class Order : IEntity
{
    public int Id { get; set; }
    public int UserId { get; set; }
    public User User { get; set; }
    public DateTime OrderDate { get; set; }
    public decimal Total { get; set; }
}

// Service layer using Repository and Unit of Work
public class UserService
{
    private readonly IUnitOfWork unitOfWork;
    
    public UserService(IUnitOfWork unitOfWork)
    {
        this.unitOfWork = unitOfWork;
    }
    
    public async Task<User> CreateUserWithOrderAsync(string name, string email, decimal orderTotal)
    {
        await unitOfWork.BeginTransactionAsync();
        
        try
        {
            // Create user
            var user = new User { Name = name, Email = email };
            await unitOfWork.Repository<User>().AddAsync(user);
            await unitOfWork.SaveChangesAsync(); // Save to get user ID
            
            // Create order
            var order = new Order 
            { 
                UserId = user.Id, 
                OrderDate = DateTime.UtcNow, 
                Total = orderTotal 
            };
            await unitOfWork.Repository<Order>().AddAsync(order);
            await unitOfWork.SaveChangesAsync();
            
            await unitOfWork.CommitTransactionAsync();
            return user;
        }
        catch
        {
            await unitOfWork.RollbackTransactionAsync();
            throw;
        }
    }
    
    public async Task<IEnumerable<User>> GetActiveUsersAsync()
    {
        // Example of using repository with LINQ
        return await unitOfWork.Repository<User>()
            .FindAsync(u => u.Orders.Any(o => o.OrderDate > DateTime.UtcNow.AddMonths(-6)));
    }
}
```

**Follow-up Questions:**
- How do you handle lazy loading with the repository pattern?
- What are the pros and cons of generic repositories vs specific repositories?

### 19. Implement caching strategies with cache-aside and write-through patterns.

**Answer:** Caching strategies determine how data flows between the application, cache, and data store. Cache-aside puts the application in control, while write-through ensures cache consistency.

**Explanation:** Different patterns suit different scenarios: cache-aside for read-heavy workloads, write-through for consistency requirements, and write-behind for performance optimization.

```csharp
public interface ICacheService
{
    Task<T> GetAsync<T>(string key);
    Task SetAsync<T>(string key, T value, TimeSpan? expiration = null);
    Task RemoveAsync(string key);
    Task<bool> ExistsAsync(string key);
}

public class DistributedCacheService : ICacheService
{
    private readonly IDistributedCache distributedCache;
    private readonly ILogger<DistributedCacheService> logger;
    private readonly JsonSerializerOptions jsonOptions;
    
    public DistributedCacheService(IDistributedCache distributedCache, ILogger<DistributedCacheService> logger)
    {
        this.distributedCache = distributedCache;
        this.logger = logger;
        jsonOptions = new JsonSerializerOptions
        {
            PropertyNamingPolicy = JsonNamingPolicy.CamelCase
        };
    }
    
    public async Task<T> GetAsync<T>(string key)
    {
        try
        {
            var json = await distributedCache.GetStringAsync(key);
            return json == null ? default(T) : JsonSerializer.Deserialize<T>(json, jsonOptions);
        }
        catch (Exception ex)
        {
            logger.LogError(ex, "Error retrieving cache key: {Key}", key);
            return default(T);
        }
    }
    
    public async Task SetAsync<T>(string key, T value, TimeSpan? expiration = null)
    {
        try
        {
            var json = JsonSerializer.Serialize(value, jsonOptions);
            var options = new DistributedCacheEntryOptions();
            
            if (expiration.HasValue)
                options.SetAbsoluteExpiration(expiration.Value);
                
            await distributedCache.SetStringAsync(key, json, options);
        }
        catch (Exception ex)
        {
            logger.LogError(ex, "Error setting cache key: {Key}", key);
        }
    }
    
    public async Task RemoveAsync(string key)
    {
        try
        {
            await distributedCache.RemoveAsync(key);
        }
        catch (Exception ex)
        {
            logger.LogError(ex, "Error removing cache key: {Key}", key);
        }
    }
    
    public async Task<bool> ExistsAsync(string key)
    {
        var value = await GetAsync<object>(key);
        return value != null;
    }
}

// Cache-Aside Pattern Implementation
public class CacheAsideUserRepository : IRepository<User>
{
    private readonly IRepository<User> baseRepository;
    private readonly ICacheService cacheService;
    private readonly TimeSpan cacheExpiration = TimeSpan.FromMinutes(30);
    
    public CacheAsideUserRepository(IRepository<User> baseRepository, ICacheService cacheService)
    {
        this.baseRepository = baseRepository;
        this.cacheService = cacheService;
    }
    
    public async Task<User> GetByIdAsync(int id)
    {
        string cacheKey = $"user:{id}";
        
        // Try to get from cache first
        var cachedUser = await cacheService.GetAsync<User>(cacheKey);
        if (cachedUser != null)
        {
            return cachedUser;
        }
        
        // Not in cache, get from database
        var user = await baseRepository.GetByIdAsync(id);
        if (user != null)
        {
            // Store in cache for future requests
            await cacheService.SetAsync(cacheKey, user, cacheExpiration);
        }
        
        return user;
    }
    
    public async Task AddAsync(User entity)
    {
        await baseRepository.AddAsync(entity);
        
        // Optionally cache the newly created entity
        if (entity.Id != 0)
        {
            string cacheKey = $"user:{entity.Id}";
            await cacheService.SetAsync(cacheKey, entity, cacheExpiration);
        }
    }
    
    public async Task UpdateAsync(User entity)
    {
        await baseRepository.UpdateAsync(entity);
        
        // Invalidate cache
        string cacheKey = $"user:{entity.Id}";
        await cacheService.RemoveAsync(cacheKey);
    }
    
    public async Task RemoveAsync(int id)
    {
        await baseRepository.RemoveAsync(id);
        
        // Remove from cache
        string cacheKey = $"user:{id}";
        await cacheService.RemoveAsync(cacheKey);
    }
}

// Write-Through Pattern Implementation
public class WriteThroughUserRepository : IRepository<User>
{
    private readonly IRepository<User> baseRepository;
    private readonly ICacheService cacheService;
    private readonly TimeSpan cacheExpiration = TimeSpan.FromMinutes(60);
    
    public WriteThroughUserRepository(IRepository<User> baseRepository, ICacheService cacheService)
    {
        this.baseRepository = baseRepository;
        this.cacheService = cacheService;
    }
    
    public async Task<User> GetByIdAsync(int id)
    {
        string cacheKey = $"user:{id}";
        
        // Try cache first
        var cachedUser = await cacheService.GetAsync<User>(cacheKey);
        if (cachedUser != null)
        {
            return cachedUser;
        }
        
        // Load from database and cache
        var user = await baseRepository.GetByIdAsync(id);
        if (user != null)
        {
            await cacheService.SetAsync(cacheKey, user, cacheExpiration);
        }
        
        return user;
    }
    
    public async Task AddAsync(User entity)
    {
        // Write to database first
        await baseRepository.AddAsync(entity);
        
        // Then write to cache
        string cacheKey = $"user:{entity.Id}";
        await cacheService.SetAsync(cacheKey, entity, cacheExpiration);
    }
    
    public async Task UpdateAsync(User entity)
    {
        // Write to database first
        await baseRepository.UpdateAsync(entity);
        
        // Then update cache
        string cacheKey = $"user:{entity.Id}";
        await cacheService.SetAsync(cacheKey, entity, cacheExpiration);
    }
}

// Advanced Circuit Breaker with metrics and configurable policies
public class AdvancedCircuitBreaker : IDisposable
{
    private readonly CircuitBreakerPolicy policy;
    private readonly IMetricsCollector metricsCollector;
    private readonly Timer monitoringTimer;
    private CircuitBreakerMetrics metrics = new CircuitBreakerMetrics();
    
    public AdvancedCircuitBreaker(CircuitBreakerPolicy policy, IMetricsCollector metricsCollector = null)
    {
        this.policy = policy;
        this.metricsCollector = metricsCollector;
        
        // Monitor circuit breaker health
        monitoringTimer = new Timer(ReportMetrics, null, TimeSpan.FromSeconds(10), TimeSpan.FromSeconds(10));
    }
    
    public async Task<T> ExecuteAsync<T>(Func<Task<T>> operation, string operationName = null)
    {
        var stopwatch = Stopwatch.StartNew();
        
        try
        {
            CheckState();
            
            var result = await operation().TimeoutAfter(policy.Timeout);
            
            stopwatch.Stop();
            RecordSuccess(stopwatch.Elapsed);
            
            return result;
        }
        catch (CircuitBreakerException)
        {
            stopwatch.Stop();
            RecordRejection();
            throw;
        }
        catch (Exception ex)
        {
            stopwatch.Stop();
            RecordFailure(ex, stopwatch.Elapsed);
            throw;
        }
    }
    
    private void CheckState()
    {
        lock (metrics)
        {
            switch (metrics.State)
            {
                case CircuitBreakerState.Open:
                    if (DateTime.UtcNow >= metrics.LastFailureTime.Add(policy.RetryTimeout))
                    {
                        TransitionToHalfOpen();
                    }
                    else
                    {
                        throw new CircuitBreakerException($"Circuit breaker is open. Next retry at: {metrics.LastFailureTime.Add(policy.RetryTimeout)}");
                    }
                    break;
            }
        }
    }
    
    private void RecordSuccess(TimeSpan duration)
    {
        lock (metrics)
        {
            metrics.SuccessCount++;
            metrics.TotalRequests++;
            metrics.LastSuccessTime = DateTime.UtcNow;
            
            if (metrics.State == CircuitBreakerState.HalfOpen)
            {
                TransitionToClosed();
            }
        }
    }
    
    private void RecordFailure(Exception exception, TimeSpan duration)
    {
        lock (metrics)
        {
            metrics.FailureCount++;
            metrics.TotalRequests++;
            metrics.LastFailureTime = DateTime.UtcNow;
            metrics.LastException = exception;
            
            var failureRate = (double)metrics.FailureCount / metrics.TotalRequests;
            
            if ((metrics.State == CircuitBreakerState.Closed || metrics.State == CircuitBreakerState.HalfOpen) &&
                metrics.TotalRequests >= policy.MinimumThroughput &&
                failureRate >= policy.FailureThreshold)
            {
                TransitionToOpen();
            }
        }
    }
    
    private void RecordRejection()
    {
        lock (metrics)
        {
            metrics.RejectionCount++;
        }
    }
    
    private void TransitionToOpen()
    {
        metrics.State = CircuitBreakerState.Open;
        metrics.StateChangedTime = DateTime.UtcNow;
        metricsCollector?.RecordStateChange("Open", metrics.StateChangedTime);
    }
    
    private void TransitionToHalfOpen()
    {
        metrics.State = CircuitBreakerState.HalfOpen;
        metrics.StateChangedTime = DateTime.UtcNow;
        metricsCollector?.RecordStateChange("HalfOpen", metrics.StateChangedTime);
    }
    
    private void TransitionToClosed()
    {
        metrics.State = CircuitBreakerState.Closed;
        metrics.StateChangedTime = DateTime.UtcNow;
        metrics.FailureCount = 0; // Reset failure count
        metricsCollector?.RecordStateChange("Closed", metrics.StateChangedTime);
    }
    
    private void ReportMetrics(object state)
    {
        lock (metrics)
        {
            metricsCollector?.ReportMetrics(new CircuitBreakerSnapshot
            {
                State = metrics.State,
                SuccessCount = metrics.SuccessCount,
                FailureCount = metrics.FailureCount,
                RejectionCount = metrics.RejectionCount,
                TotalRequests = metrics.TotalRequests,
                FailureRate = metrics.TotalRequests > 0 ? (double)metrics.FailureCount / metrics.TotalRequests : 0
            });
        }
    }
    
    public void Dispose()
    {
        monitoringTimer?.Dispose();
    }
}

public class CircuitBreakerPolicy
{
    public int MinimumThroughput { get; set; } = 10;
    public double FailureThreshold { get; set; } = 0.5; // 50% failure rate
    public TimeSpan Timeout { get; set; } = TimeSpan.FromSeconds(10);
    public TimeSpan RetryTimeout { get; set; } = TimeSpan.FromMinutes(1);
}

public class CircuitBreakerMetrics
{
    public CircuitBreakerState State { get; set; } = CircuitBreakerState.Closed;
    public long SuccessCount { get; set; }
    public long FailureCount { get; set; }
    public long RejectionCount { get; set; }
    public long TotalRequests { get; set; }
    public DateTime LastSuccessTime { get; set; }
    public DateTime LastFailureTime { get; set; }
    public DateTime StateChangedTime { get; set; } = DateTime.UtcNow;
    public Exception LastException { get; set; }
}

public class CircuitBreakerSnapshot
{
    public CircuitBreakerState State { get; set; }
    public long SuccessCount { get; set; }
    public long FailureCount { get; set; }
    public long RejectionCount { get; set; }
    public long TotalRequests { get; set; }
    public double FailureRate { get; set; }
}

// Extension method for timeout functionality
public static class TaskExtensions
{
    public static async Task<T> TimeoutAfter<T>(this Task<T> task, TimeSpan timeout)
    {
        using var timeoutCancellationTokenSource = new CancellationTokenSource();
        var completedTask = await Task.WhenAny(task, Task.Delay(timeout, timeoutCancellationTokenSource.Token));
        
        if (completedTask == task)
        {
            timeoutCancellationTokenSource.Cancel();
            return await task;
        }
        else
        {
            throw new TimeoutException($"The operation has timed out after {timeout}.");
        }
    }
    
    public static async Task TimeoutAfter(this Task task, TimeSpan timeout)
    {
        using var timeoutCancellationTokenSource = new CancellationTokenSource();
        var completedTask = await Task.WhenAny(task, Task.Delay(timeout, timeoutCancellationTokenSource.Token));
        
        if (completedTask == task)
        {
            timeoutCancellationTokenSource.Cancel();
            await task;
        }
        else
        {
            throw new TimeoutException($"The operation has timed out after {timeout}.");
        }
    }
}

// Usage example
public class ExternalServiceClient
{
    private readonly HttpClient httpClient;
    private readonly AdvancedCircuitBreaker circuitBreaker;
    
    public ExternalServiceClient(HttpClient httpClient, IMetricsCollector metricsCollector)
    {
        this.httpClient = httpClient;
        
        var policy = new CircuitBreakerPolicy
        {
            FailureThreshold = 0.6, // 60% failure rate
            MinimumThroughput = 5,
            Timeout = TimeSpan.FromSeconds(5),
            RetryTimeout = TimeSpan.FromSeconds(30)
        };
        
        circuitBreaker = new AdvancedCircuitBreaker(policy, metricsCollector);
    }
    
    public async Task<string> GetDataAsync(string endpoint)
    {
        return await circuitBreaker.ExecuteAsync(async () =>
        {
            var response = await httpClient.GetAsync(endpoint);
            response.EnsureSuccessStatusCode();
            return await response.Content.ReadAsStringAsync();
        }, $"GetData-{endpoint}");
    }
}

public interface IMetricsCollector
{
    void RecordStateChange(string newState, DateTime timestamp);
    void ReportMetrics(CircuitBreakerSnapshot snapshot);
}
```

**Follow-up Questions:**
- How do you implement circuit breaker pattern in a microservices architecture?
- What metrics should you monitor to tune circuit breaker parameters?

## SQL and Data Access

### 21. Explain SQL execution plans and how to optimize query performance.

**Answer:** SQL execution plans show how the database engine executes queries. Understanding plans helps identify performance bottlenecks like table scans, missing indexes, or expensive joins.

**Explanation:** Query optimization involves analyzing execution plans, understanding index usage, join strategies, and statistics. Different plan operators have different performance characteristics.

```csharp
public class QueryOptimizationExample
{
    public async Task DemonstrateQueryOptimization()
    {
        using var connection = new SqlConnection(connectionString);
        await connection.OpenAsync();
        
        // Bad query - table scan
        var badQuery = @"
            SELECT u.Name, u.Email, COUNT(o.Id) as OrderCount
            FROM Users u
            LEFT JOIN Orders o ON u.Id = o.UserId
            WHERE u.Email LIKE '%@gmail.com'
            GROUP BY u.Id, u.Name, u.Email
            ORDER BY COUNT(o.Id) DESC";
            
        // Optimized query with proper indexing
        var optimizedQuery = @"
            SELECT u.Name, u.Email, 
                   ISNULL(o.OrderCount, 0) as OrderCount
            FROM Users u
            LEFT JOIN (
                SELECT UserId, COUNT(*) as OrderCount
                FROM Orders WITH(INDEX(IX_Orders_UserId))
                GROUP BY UserId
            ) o ON u.Id = o.UserId
            WHERE u.EmailDomain = 'gmail.com'  -- Computed column with index
            ORDER BY ISNULL(o.OrderCount, 0) DESC";
            
        // Use query hints when necessary
        var queryWithHints = @"
            SELECT /*+ USE_INDEX(Users, IX_Users_Email) */ 
                   u.Name, u.Email
            FROM Users u WITH(INDEX(IX_Users_Email))
            WHERE u.Email = @email
            OPTION (OPTIMIZE FOR (@email = 'user@example.com'))";
            
        // Parameterized queries to prevent SQL injection and enable plan reuse
        using var command = new SqlCommand(optimizedQuery, connection);
        command.Parameters.Add("@domain", SqlDbType.VarChar, 50).Value = "gmail.com";
        
        var results = await command.ExecuteReaderAsync();
    }
    
    // Method to analyze query performance
    public async Task<QueryPerformanceInfo> AnalyzeQueryPerformance(string query, Dictionary<string, object> parameters = null)
    {
        using var connection = new SqlConnection(connectionString);
        await connection.OpenAsync();
        
        // Enable execution plan and statistics
        await connection.ExecuteAsync("SET STATISTICS IO ON");
        await connection.ExecuteAsync("SET STATISTICS TIME ON");
        
        var stopwatch = Stopwatch.StartNew();
        
        using var command = new SqlCommand(query, connection);
        if (parameters != null)
        {
            foreach (var param in parameters)
            {
                command.Parameters.AddWithValue(param.Key, param.Value);
            }
        }
        
        var results = await command.ExecuteReaderAsync();
        var rowCount = 0;
        
        while (await results.ReadAsync())
        {
            rowCount++;
        }
        
        stopwatch.Stop();
        
        return new QueryPerformanceInfo
        {
            ExecutionTime = stopwatch.Elapsed,
            RowsReturned = rowCount,
            Query = query
        };
    }
}

public class QueryPerformanceInfo
{
    public TimeSpan ExecutionTime { get; set; }
    public int RowsReturned { get; set; }
    public string Query { get; set; }
}

// Index management helper
public class IndexManagementService
{
    private readonly string connectionString;
    
    public IndexManagementService(string connectionString)
    {
        this.connectionString = connectionString;
    }
    
    public async Task<List<MissingIndexInfo>> GetMissingIndexesAsync()
    {
        var query = @"
            SELECT 
                mid.statement as TableName,
                migs.avg_total_user_cost * (migs.avg_user_impact / 100.0) * (migs.user_seeks + migs.user_scans) AS improvement_measure,
                'CREATE INDEX IX_' + REPLACE(REPLACE(REPLACE(mid.statement, '[', ''), ']', ''), '.', '_') + 
                '_' + REPLACE(REPLACE(mid.equality_columns + ISNULL('_' + mid.inequality_columns, ''), '[', ''), ']', '') + 
                ' ON ' + mid.statement + ' (' + ISNULL(mid.equality_columns, '') + 
                CASE WHEN mid.equality_columns IS NOT NULL AND mid.inequality_columns IS NOT NULL THEN ',' ELSE '' END +
                ISNULL(mid.inequality_columns, '') + ')' + 
                ISNULL(' INCLUDE (' + mid.included_columns + ')', '') AS create_index_statement
            FROM sys.dm_db_missing_index_groups mig
            INNER JOIN sys.dm_db_missing_index_group_stats migs ON migs.group_handle = mig.index_group_handle
            INNER JOIN sys.dm_db_missing_index_details mid ON mig.index_handle = mid.index_handle
            WHERE migs.avg_total_user_cost * (migs.avg_user_impact / 100.0) * (migs.user_seeks + migs.user_scans) > 10
            ORDER BY improvement_measure DESC";
            
        using var connection = new SqlConnection(connectionString);
        return (await connection.QueryAsync<MissingIndexInfo>(query)).ToList();
    }
    
    public async Task<List<IndexUsageInfo>> GetIndexUsageStatsAsync()
    {
        var query = @"
            SELECT 
                OBJECT_NAME(ius.object_id) as TableName,
                i.name as IndexName,
                ius.user_seeks,
                ius.user_scans,
                ius.user_lookups,
                ius.user_updates,
                CASE 
                    WHEN ius.user_seeks + ius.user_scans + ius.user_lookups = 0 THEN 'Consider dropping'
                    WHEN ius.user_updates > (ius.user_seeks + ius.user_scans + ius.user_lookups) * 2 THEN 'High maintenance cost'
                    ELSE 'Good usage'
                END as Usage_Assessment
            FROM sys.dm_db_index_usage_stats ius
            INNER JOIN sys.indexes i ON ius.object_id = i.object_id AND ius.index_id = i.index_id
            WHERE ius.database_id = DB_ID()
            ORDER BY ius.user_seeks + ius.user_scans + ius.user_lookups DESC";
            
        using var connection = new SqlConnection(connectionString);
        return (await connection.QueryAsync<IndexUsageInfo>(query)).ToList();
    }
}

public class MissingIndexInfo
{
    public string TableName { get; set; }
    public double ImprovementMeasure { get; set; }
    public string CreateIndexStatement { get; set; }
}

public class IndexUsageInfo
{
    public string TableName { get; set; }
    public string IndexName { get; set; }
    public long UserSeeks { get; set; }
    public long UserScans { get; set; }
    public long UserLookups { get; set; }
    public long UserUpdates { get; set; }
    public string UsageAssessment { get; set; }
}
```

**Follow-up Questions:**
- How do you handle query plan caching and parameter sniffing?
- What are the trade-offs between covering indexes and included columns?

### 22. Implement connection pooling and demonstrate ADO.NET best practices.

**Answer:** Connection pooling reuses database connections to improve performance and resource utilization. ADO.NET provides automatic pooling with configurable parameters.

**Explanation:** Proper connection management involves using connection strings with pooling settings, disposing connections properly, and handling connection failures gracefully.

```csharp
public class ConnectionPoolingExample
{
    // Connection string with pooling configuration
    private readonly string connectionString = @"
        Data Source=server;
        Initial Catalog=database;
        Integrated Security=true;
        Connection Timeout=30;
        Command Timeout=60;
        Pooling=true;
        Min Pool Size=5;
        Max Pool Size=100;
        Connection Lifetime=300;
        Application Name=MyApplication";
    
    // Connection factory with retry logic
    public async Task<SqlConnection> CreateConnectionAsync()
    {
        var connection = new SqlConnection(connectionString);
        
        var retryPolicy = new RetryPolicy(
            retryCount: 3,
            delay: TimeSpan.FromSeconds(1));
            
        await retryPolicy.ExecuteAsync(async () =>
        {
            await connection.OpenAsync();
        });
        
        return connection;
    }
    
    // Efficient data access patterns
    public async Task<IEnumerable<User>> GetUsersWithOrdersAsync()
    {
        const string sql = @"
            SELECT u.Id, u.Name, u.Email,
                   o.Id as OrderId, o.OrderDate, o.Total
            FROM Users u
            LEFT JOIN Orders o ON u.Id = o.UserId
            ORDER BY u.Id";
            
        using var connection = await CreateConnectionAsync();
        
        var userDictionary = new Dictionary<int, User>();
        
        await connection.QueryAsync<User, Order, User>(sql, (user, order) =>
        {
            if (!userDictionary.TryGetValue(user.Id, out var existingUser))
            {
                existingUser = user;
                existingUser.Orders = new List<Order>();
                userDictionary.Add(user.Id, existingUser);
            }
            
            if (order != null)
            {
                existingUser.Orders.Add(order);
            }
            
            return existingUser;
        }, splitOn: "OrderId");
        
        return userDictionary.Values;
    }
    
    // Bulk operations for better performance
    public async Task BulkInsertUsersAsync(IEnumerable<User> users)
    {
        using var connection = await CreateConnectionAsync();
        using var transaction = connection.BeginTransaction();
        
        try
        {
            // Method 1: Using SqlBulkCopy for large datasets
            var dataTable = ConvertToDataTable(users);
            using var bulkCopy = new SqlBulkCopy(connection, SqlBulkCopyOptions.Default, transaction);
            bulkCopy.DestinationTableName = "Users";
            bulkCopy.BatchSize = 1000;
            bulkCopy.BulkCopyTimeout = 300;
            
            await bulkCopy.WriteToServerAsync(dataTable);
            
            transaction.Commit();
        }
        catch
        {
            transaction.Rollback();
            throw;
        }
    }
    
    public async Task BulkUpdateUsersAsync(IEnumerable<User> users)
    {
        using var connection = await CreateConnectionAsync();
        
        // Method 2: Using table-valued parameters
        var userTable = CreateUserTableParameter(users);
        
        const string sql = @"
            UPDATE u SET 
                Name = ut.Name,
                Email = ut.Email
            FROM Users u
            INNER JOIN @UserTable ut ON u.Id = ut.Id";
            
        await connection.ExecuteAsync(sql, new { UserTable = userTable });
    }
    
    // Async streaming for large result sets
    public async IAsyncEnumerable<User> StreamUsersAsync()
    {
        using var connection = await CreateConnectionAsync();
        using var command = new SqlCommand("SELECT Id, Name, Email FROM Users ORDER BY Id", connection);
        
        using var reader = await command.ExecuteReaderAsync();
        
        while (await reader.ReadAsync())
        {
            yield return new User
            {
                Id = reader.GetInt32("Id"),
                Name = reader.GetString("Name"),
                Email = reader.GetString("Email")
            };
        }
    }
    
    // Connection health monitoring
    public async Task<ConnectionHealthInfo> CheckConnectionHealthAsync()
    {
        var healthInfo = new ConnectionHealthInfo();
        
        try
        {
            using var connection = await CreateConnectionAsync();
            var stopwatch = Stopwatch.StartNew();
            
            await connection.ExecuteScalarAsync("SELECT 1");
            
            stopwatch.Stop();
            healthInfo.IsHealthy = true;
            healthInfo.ResponseTime = stopwatch.Elapsed;
            healthInfo.PoolInfo = GetPoolInfo();
        }
        catch (Exception ex)
        {
            healthInfo.IsHealthy = false;
            healthInfo.Error = ex.Message;
        }
        
        return healthInfo;
    }
    
    private ConnectionPoolInfo GetPoolInfo()
    {
        // Access connection pool performance counters
        return new ConnectionPoolInfo
        {
            // These would require performance counter access or custom tracking
            ActiveConnections = 0, // Placeholder
            PooledConnections = 0, // Placeholder
            TotalConnections = 0   // Placeholder
        };
    }
    
    private DataTable ConvertToDataTable(IEnumerable<User> users)
    {
        var dataTable = new DataTable();
        dataTable.Columns.Add("Id", typeof(int));
        dataTable.Columns.Add("Name", typeof(string));
        dataTable.Columns.Add("Email", typeof(string));
        
        foreach (var user in users)
        {
            dataTable.Rows.Add(user.Id, user.Name, user.Email);
        }
        
        return dataTable;
    }
    
    private DataTable CreateUserTableParameter(IEnumerable<User> users)
    {
        var table = new DataTable();
        table.Columns.Add("Id", typeof(int));
        table.Columns.Add("Name", typeof(string));
        table.Columns.Add("Email", typeof(string));
        
        foreach (var user in users)
        {
            table.Rows.Add(user.Id, user.Name, user.Email);
        }
        
        return table;
    }
}

// Retry policy implementation
public class RetryPolicy
{
    private readonly int retryCount;
    private readonly TimeSpan delay;
    
    public RetryPolicy(int retryCount, TimeSpan delay)
    {
        this.retryCount = retryCount;
        this.delay = delay;
    }
    
    public async Task ExecuteAsync(Func<Task> operation)
    {
        var currentRetry = 0;
        
        while (true)
        {
            try
            {
                await operation();
                break;
            }
            catch (Exception ex) when (IsTransientError(ex) && currentRetry < retryCount)
            {
                currentRetry++;
                await Task.Delay(delay * currentRetry); // Exponential backoff
            }
        }
    }
    
    private bool IsTransientError(Exception ex)
    {
        // Check for transient SQL errors
        if (ex is SqlException sqlEx)
        {
            return sqlEx.Number switch
            {
                -2 => true,      // Timeout
                2 => true,       // Connection timeout
                53 => true,      // Network error
                121 => true,     // Semaphore timeout
                1205 => true,    // Deadlock
                _ => false
            };
        }
        
        return false;
    }
}

public class ConnectionHealthInfo
{
    public bool IsHealthy { get; set; }
    public TimeSpan ResponseTime { get; set; }
    public string Error { get; set; }
    public ConnectionPoolInfo PoolInfo { get; set; }
}

public class ConnectionPoolInfo
{
    public int ActiveConnections { get; set; }
    public int PooledConnections { get; set; }
    public int TotalConnections { get; set; }
}
```

**Follow-up Questions:**
- How do you handle connection pool exhaustion?
- What are the performance implications of different isolation levels?

## Web API and HTTP

### 23. Implement comprehensive Web API error handling and logging.

**Answer:** Comprehensive error handling in Web API involves global exception handling, structured logging, and consistent error responses while maintaining security.

**Explanation:** Good error handling provides meaningful responses to clients while protecting sensitive information and enabling effective debugging through structured logging.

```csharp
// Global exception handling middleware
public class GlobalExceptionMiddleware
{
    private readonly RequestDelegate next;
    private readonly ILogger<GlobalExceptionMiddleware> logger;
    private readonly IWebHostEnvironment environment;
    
    public GlobalExceptionMiddleware(RequestDelegate next, ILogger<GlobalExceptionMiddleware> logger, IWebHostEnvironment environment)
    {
        this.next = next;
        this.logger = logger;
        this.environment = environment;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await next(context);
        }
        catch (Exception ex)
        {
            await HandleExceptionAsync(context, ex);
        }
    }
    
    private async Task HandleExceptionAsync(HttpContext context, Exception exception)
    {
        var correlationId = context.TraceIdentifier;
        
        logger.LogError(exception, 
            "Unhandled exception occurred. CorrelationId: {CorrelationId}, Path: {Path}, Method: {Method}",
            correlationId, context.Request.Path, context.Request.Method);
        
        var response = CreateErrorResponse(exception, correlationId);
        
        context.Response.ContentType = "application/json";
        context.Response.StatusCode = response.StatusCode;
        
        var jsonResponse = JsonSerializer.Serialize(response, new JsonSerializerOptions
        {
            PropertyNamingPolicy = JsonNamingPolicy.CamelCase
        });
        
        await context.Response.WriteAsync(jsonResponse);
    }
    
    private ApiErrorResponse CreateErrorResponse(Exception exception, string correlationId)
    {
        return exception switch
        {
            ValidationException validationEx => new ApiErrorResponse
            {
                StatusCode = 400,
                Title = "Validation Error",
                Detail = "One or more validation errors occurred.",
                CorrelationId = correlationId,
                Errors = validationEx.ValidationErrors
            },
            NotFoundException notFoundEx => new ApiErrorResponse
            {
                StatusCode = 404,
                Title = "Resource Not Found",
                Detail = notFoundEx.Message,
                CorrelationId = correlationId
            },
            UnauthorizedException => new ApiErrorResponse
            {
                StatusCode = 401,
                Title = "Unauthorized",
                Detail = "Authentication is required to access this resource.",
                CorrelationId = correlationId
            },
            ForbiddenException forbiddenEx => new ApiErrorResponse
            {
                StatusCode = 403,
                Title = "Forbidden",
                Detail = forbiddenEx.Message,
                CorrelationId = correlationId
            },
            BusinessLogicException businessEx => new ApiErrorResponse
            {
                StatusCode = 400,
                Title = "Business Rule Violation",
                Detail = businessEx.Message,
                CorrelationId = correlationId,
                ErrorCode = businessEx.ErrorCode
            },
            _ => new ApiErrorResponse
            {
                StatusCode = 500,
                Title = "Internal Server Error",
                Detail = environment.IsDevelopment() ? exception.Message : "An unexpected error occurred.",
                CorrelationId = correlationId
            }
        };
    }
}

public class ApiErrorResponse
{
    public int StatusCode { get; set; }
    public string Title { get; set; }
    public string Detail { get; set; }
    public string CorrelationId { get; set; }
    public string ErrorCode { get; set; }
    public Dictionary<string, string> Errors { get; set; }
    public DateTime Timestamp { get; set; } = DateTime.UtcNow;
}

// Custom exception types
public class NotFoundException : Exception
{
    public NotFoundException(string message) : base(message) { }
    public NotFoundException(string resourceType, object id) : base($"{resourceType} with id '{id}' was not found.") { }
}

public class UnauthorizedException : Exception
{
    public UnauthorizedException() : base("Unauthorized access.") { }
    public UnauthorizedException(string message) : base(message) { }
}

public class ForbiddenException : Exception
{
    public ForbiddenException(string message) : base(message) { }
}

// Structured logging with Serilog integration
public class StructuredLoggingService
{
    private readonly ILogger<StructuredLoggingService> logger;
    
    public StructuredLoggingService(ILogger<StructuredLoggingService> logger)
    {
        this.logger = logger;
    }
    
    public void LogApiRequest(HttpContext context, TimeSpan duration)
    {
        logger.LogInformation(
            "API Request completed: {Method} {Path} {StatusCode} in {Duration}ms. User: {UserId}",
            context.Request.Method,
            context.Request.Path,
            context.Response.StatusCode,
            duration.TotalMilliseconds,
            context.User?.FindFirst("sub")?.Value ?? "Anonymous");
    }
    
    public void LogBusinessOperation(string operation, object parameters, TimeSpan duration, bool success)
    {
        if (success)
        {
            logger.LogInformation(
                "Business operation completed: {Operation} with parameters {@Parameters} in {Duration}ms",
                operation, parameters, duration.TotalMilliseconds);
        }
        else
        {
            logger.LogWarning(
                "Business operation failed: {Operation} with parameters {@Parameters} in {Duration}ms",
                operation, parameters, duration.TotalMilliseconds);
        }
    }
    
    public void LogPerformanceMetrics(string operation, Dictionary<string, object> metrics)
    {
        logger.LogInformation(
            "Performance metrics for {Operation}: {@Metrics}",
            operation, metrics);
    }
}

// Request/Response logging middleware
public class RequestResponseLoggingMiddleware
{
    private readonly RequestDelegate next;
    private readonly StructuredLoggingService loggingService;
    
    public RequestResponseLoggingMiddleware(RequestDelegate next, StructuredLoggingService loggingService)
    {
        this.next = next;
        this.loggingService = lod caching with cache warming and batch operations
public class AdvancedCacheManager<T> where T : class, IEntity
{
    private readonly IRepository<T> repository;
    private readonly ICacheService cacheService;
    private readonly SemaphoreSlim semaphore = new SemaphoreSlim(1, 1);
    
    public AdvancedCacheManager(IRepository<T> repository, ICacheService cacheService)
    {
        this.repository = repository;
        this.cacheService = cacheService;
    }
    
    // Cache warming strategy
    public async Task WarmCacheAsync(IEnumerable<int> ids)
    {
        var tasks = ids.Select(async id =>
        {
            string cacheKey = GetCacheKey(id);
            if (!await cacheService.ExistsAsync(cacheKey))
            {
                var entity = await repository.GetByIdAsync(id);
                if (entity != null)
                {
                    await cacheService.SetAsync(cacheKey, entity, TimeSpan.FromHours(1));
                }
            }
        });
        
        await Task.WhenAll(tasks);
    }
    
    // Batch cache operations
    public async Task<Dictionary<int, T>> GetMultipleAsync(IEnumerable<int> ids)
    {
        var result = new Dictionary<int, T>();
        var missedIds = new List<int>();
        
        // Check cache first
        foreach (var id in ids)
        {
            var cached = await cacheService.GetAsync<T>(GetCacheKey(id));
            if (cached != null)
            {
                result[id] = cached;
            }
            else
            {
                missedIds.Add(id);
            }
        }
        
        // Load missing items from database
        if (missedIds.Any())
        {
            var missedEntities = await repository.FindAsync(e => missedIds.Contains(e.Id));
            foreach (var entity in missedEntities)
            {
                result[entity.Id] = entity;
                await cacheService.SetAsync(GetCacheKey(entity.Id), entity, TimeSpan.FromHours(1));
            }
        }
        
        return result;
    }
    
    private string GetCacheKey(int id) => $"{typeof(T).Name.ToLower()}:{id}";
}
```

**Follow-up Questions:**
- How do you handle cache invalidation in a distributed system?
- What are the trade-offs between different cache eviction policies?

### 20. Explain and implement the Circuit Breaker pattern for fault tolerance.

**Answer:** The Circuit Breaker pattern prevents an application from repeatedly trying to execute an operation that's likely to fail, allowing it to continue operating despite failures in dependent services.

**Explanation:** The pattern has three states: Closed (normal operation), Open (failing fast), and Half-Open (testing if the service has recovered). This protects both the calling service and the failing service.

```csharp
public enum CircuitBreakerState
{
    Closed,    // Normal operation
    Open,      // Failing fast
    HalfOpen   // Testing if service recovered
}

public class CircuitBreakerException : Exception
{
    public CircuitBreakerException(string message) : base(message) { }
}

public class CircuitBreaker
{
    private readonly int failureThreshold;
    private readonly TimeSpan timeout;
    private readonly TimeSpan retryTimeout;
    
    private int failureCount;
    private DateTime lastFailureTime;
    private CircuitBreakerState state = CircuitBreakerState.Closed;
    private readonly object lockObject = new object();
    
    public CircuitBreaker(int failureThreshold = 5, TimeSpan timeout = default, TimeSpan retryTimeout = default)
    {
        this.failureThreshold = failureThreshold;
        this.timeout = timeout == default ? TimeSpan.FromSeconds(10) : timeout;
        this.retryTimeout = retryTimeout == default ? TimeSpan.FromMinutes(1) : retryTimeout;
    }
    
    public CircuitBreakerState State
    {
        get
        {
            lock (lockObject)
            {
                return state;
            }
        }
    }
    
    public async Task<T> ExecuteAsync<T>(Func<Task<T>> operation)
    {
        CheckCircuitBreakerState();
        
        try
        {
            var task = operation();
            var result = await task.TimeoutAfter(timeout);
            OnSuccess();
            return result;
        }
        catch (TimeoutException)
        {
            OnFailure();
            throw new CircuitBreakerException("Operation timed out");
        }
        catch (Exception)
        {
            OnFailure();
            throw;
        }
    }
    
    public async Task ExecuteAsync(Func<Task> operation)
    {
        CheckCircuitBreakerState();
        
        try
        {
            var task = operation();
            await task.TimeoutAfter(timeout);
            OnSuccess();
        }
        catch (TimeoutException)
        {
            OnFailure();
            throw new CircuitBreakerException("Operation timed out");
        }
        catch (Exception)
        {
            OnFailure();
            throw;
        }
    }
    
    private void CheckCircuitBreakerState()
    {
        lock (lockObject)
        {
            switch (state)
            {
                case CircuitBreakerState.Open:
                    if (DateTime.UtcNow >= lastFailureTime.Add(retryTimeout))
                    {
                        state = CircuitBreakerState.HalfOpen;
                    }
                    else
                    {
                        throw new CircuitBreakerException("Circuit breaker is open");
                    }
                    break;
                    
                case CircuitBreakerState.HalfOpen:
                    // Allow one request through to test the service
                    break;
                    
                case CircuitBreakerState.Closed:
                    // Normal operation
                    break;
            }
        }
    }
    
    private void OnSuccess()
    {
        lock (lockObject)
        {
            failureCount = 0;
            state = CircuitBreakerState.Closed;
        }
    }
    
    private void OnFailure()
    {
        lock (lockObject)
        {
            failureCount++;
            lastFailureTime = DateTime.UtcNow;
            
            if (failureCount >= failureThreshold)
            {
                state = CircuitBreakerState.Open;
            }
        }
    }
    
    public void Reset()
    {
        lock (lockObject)
        {
            failureCount = 0;
            state = CircuitBreakerState.Closed;
        }
    }
}

// Advance