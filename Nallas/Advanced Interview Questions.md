Absolutely! You're preparing for a high-impact role, and this kind of deep technical readiness will set you apart. Below is a curated set of **50 advanced C# interview questions**, each with:

- A correct answer  
- A detailed explanation with examples  
- Follow-up questions to deepen your understanding or anticipate interviewer probes  

---

# Advanced C# Interview Questions (1–10)

### 1. What is the difference between `ref` and `out` parameters in C#?

**Answer**:  
- `ref` requires the variable to be initialized before passing.  
- `out` does not require initialization before passing, but must be assigned inside the method.

**Explanation**:  
```csharp
void ModifyRef(ref int x) { x += 10; }
void ModifyOut(out int y) { y = 20; }

int a = 5;
ModifyRef(ref a); // a becomes 15

int b;
ModifyOut(out b); // b becomes 20
```

**Follow-up**:  
- Can you use both `ref` and `out` in async methods?  
- What are the implications for method overloading?

---

### 2. What is the difference between `Task` and `Thread`?

**Answer**:  
- `Thread` is a lower-level construct for creating and managing threads manually.  
- `Task` is part of the Task Parallel Library (TPL) and abstracts threading with better scalability and control.

**Explanation**:  
```csharp
Task.Run(() => Console.WriteLine("Using Task"));
new Thread(() => Console.WriteLine("Using Thread")).Start();
```

**Follow-up**:  
- How does `Task` handle exceptions compared to `Thread`?  
- What is the role of `async/await` with `Task`?

---

### 3. What is the difference between `IEnumerable` and `IQueryable`?

**Answer**:  
- `IEnumerable` executes queries in-memory.  
- `IQueryable` executes queries on the data source (e.g., SQL Server).

**Explanation**:  
```csharp
IEnumerable<Product> products = db.Products.ToList(); // In-memory
IQueryable<Product> query = db.Products.Where(p => p.Price > 100); // SQL executed
```

**Follow-up**:  
- When would you prefer `IEnumerable` over `IQueryable`?

---

### 4. What is boxing and unboxing in C#?

**Answer**:  
- **Boxing**: Converting a value type to `object`.  
- **Unboxing**: Extracting the value type from `object`.

**Explanation**:  
```csharp
int i = 123;
object obj = i; // Boxing
int j = (int)obj; // Unboxing
```

**Follow-up**:  
- What are the performance implications of boxing?

---

### 5. What is the difference between `abstract` class and `interface`?

**Answer**:  
- Abstract class can have implementation; interface cannot (prior to C# 8).  
- A class can inherit multiple interfaces but only one abstract class.

**Explanation**:  
```csharp
abstract class Animal { public abstract void Speak(); }
interface IAnimal { void Speak(); }
```

**Follow-up**:  
- How did C# 8 change interface capabilities?

---

### 6. What is the use of `yield return`?

**Answer**:  
It enables deferred execution and simplifies iterator implementation.

**Explanation**:  
```csharp
public IEnumerable<int> GetNumbers()
{
    yield return 1;
    yield return 2;
}
```

**Follow-up**:  
- How does `yield return` affect memory usage?

---

### 7. What is the difference between `readonly` and `const`?

**Answer**:  
- `const` is compile-time constant.  
- `readonly` is runtime constant, set in constructor.

**Explanation**:  
```csharp
const int Max = 100;
readonly int Min = 10;
```

**Follow-up**:  
- Can you use `readonly` with static fields?

---

### 8. What is the difference between `==` and `.Equals()`?

**Answer**:  
- `==` compares reference for objects unless overloaded.  
- `.Equals()` compares values.

**Explanation**:  
```csharp
string a = "hello";
string b = new string("hello".ToCharArray());
Console.WriteLine(a == b); // True
Console.WriteLine(a.Equals(b)); // True
```

**Follow-up**:  
- How do you override `Equals()` and `GetHashCode()`?

---

### 9. What is the difference between `static` and `singleton`?

**Answer**:  
- `static` class cannot be instantiated.  
- `singleton` is a design pattern that ensures one instance.

**Explanation**:  
```csharp
public sealed class Singleton
{
    private static readonly Singleton _instance = new Singleton();
    public static Singleton Instance => _instance;
    private Singleton() { }
}
```

**Follow-up**:  
- How do you make a thread-safe singleton?

---

### 10. What are SOLID principles?

**Answer**:  
- **S**: Single Responsibility  
- **O**: Open/Closed  
- **L**: Liskov Substitution  
- **I**: Interface Segregation  
- **D**: Dependency Inversion

**Explanation**:  
Each principle promotes maintainable, scalable, and testable code.

**Follow-up**:  
- Can you give a real-world example of violating SRP?

Absolutely! Here's a full set of 40 additional advanced C# interview questions, each with a correct answer, detailed explanation, and relevant follow-up questions. These are tailored for your Tech Lead and Solution Architect interview prep, especially with your .NET Core and SOLID expertise.

---

# Advanced C# Interview Questions (11–50)

### 11. What is the difference between `Dispose()` and `Finalize()`?

**Answer**:  
`Dispose()` is called explicitly to release unmanaged resources.  
`Finalize()` is called by the garbage collector before object destruction.

**Explanation**:  
Use `Dispose()` via `IDisposable` for deterministic cleanup.  
`Finalize()` is overridden in a destructor and is non-deterministic.

**Follow-up**:  
- How does `using` relate to `Dispose()`?

---

### 12. What is the difference between `async` and `await`?

**Answer**:  
`async` marks a method as asynchronous.  
`await` pauses execution until the awaited task completes.

**Explanation**:  
```csharp
public async Task<int> GetDataAsync() {
    int result = await Task.Run(() => 5);
    return result;
}
```

**Follow-up**:  
- What happens if you forget to use `await`?

---

### 13. What is the difference between `var`, `dynamic`, and `object`?

**Answer**:  
- `var`: compile-time type inference  
- `dynamic`: runtime type resolution  
- `object`: base type of all types

**Explanation**:  
`var x = 5;` is int  
`dynamic x = 5; x = "hello";`  
`object x = 5; x.ToString();`

**Follow-up**:  
- What are the risks of using `dynamic`?

---

### 14. What is covariance and contravariance?

**Answer**:  
Covariance allows a method to return a more derived type.  
Contravariance allows a method to accept a less derived type.

**Explanation**:  
```csharp
IEnumerable<string> strList = new List<string>(); // Covariance
Action<object> act = (obj) => Console.WriteLine(obj); // Contravariance
```

**Follow-up**:  
- Where is this used in delegates?

---

### 15. What is the difference between `Thread.Sleep()` and `Task.Delay()`?

**Answer**:  
`Thread.Sleep()` blocks the thread.  
`Task.Delay()` is asynchronous and does not block.

**Explanation**:  
Use `Task.Delay()` in async methods to avoid blocking.

**Follow-up**:  
- What happens if you use `Thread.Sleep()` in UI thread?

---

### 16. What is the difference between `Struct` and `Class`?

**Answer**:  
Structs are value types; classes are reference types.

**Explanation**:  
Structs are stored on the stack, classes on the heap.

**Follow-up**:  
- Can structs have constructors?

---

### 17. What is the difference between `public`, `private`, `protected`, and `internal`?

**Answer**:  
- `public`: accessible everywhere  
- `private`: accessible within the class  
- `protected`: accessible in derived classes  
- `internal`: accessible within the same assembly

**Explanation**:  
Use access modifiers to enforce encapsulation.

**Follow-up**:  
- What is `protected internal`?

---

### 18. What is the difference between `interface` and `delegate`?

**Answer**:  
Interface defines a contract; delegate defines a method signature.

**Explanation**:  
Use interfaces for polymorphism; delegates for callbacks.

**Follow-up**:  
- Can you use delegates with events?

---

### 19. What is the difference between `event` and `delegate`?

**Answer**:  
`event` is a wrapper around a delegate to restrict access.

**Explanation**:  
```csharp
public event Action OnClick;
```

**Follow-up**:  
- How do you unsubscribe from an event?

---

### 20. What is the difference between `static` and `instance` members?

**Answer**:  
Static members belong to the type; instance members belong to the object.

**Explanation**:  
Use static for shared data or utility methods.

**Follow-up**:  
- Can static methods access instance members?

---

### 21. What is the difference between `override`, `new`, and `virtual`?

**Answer**:  
- `virtual`: allows method to be overridden  
- `override`: overrides base method  
- `new`: hides base method

**Explanation**:  
Use `override` for polymorphism; `new` for hiding.

**Follow-up**:  
- What happens if you forget `override`?

---

### 22. What is the difference between `is` and `as` operators?

**Answer**:  
`is` checks type compatibility; `as` performs safe casting.

**Explanation**:  
```csharp
if (obj is string) { }  
string s = obj as string;
```

**Follow-up**:  
- What happens if `as` fails?

---

### 23. What is the difference between `params` and `params object[]`?

**Answer**:  
`params` allows variable number of arguments.  
`params object[]` allows any type.

**Explanation**:  
```csharp
void Print(params int[] numbers) { }
```

**Follow-up**:  
- Can you use multiple `params`?

---

### 24. What is the difference between `sealed` and `abstract`?

**Answer**:  
`sealed` prevents inheritance; `abstract` enforces inheritance.

**Explanation**:  
Use `sealed` for security or final implementations.

**Follow-up**:  
- Can a class be both `sealed` and `abstract`?

---

### 25. What is the difference between `try-catch-finally` and `using`?

**Answer**:  
`try-catch-finally` handles exceptions.  
`using` ensures disposal of resources.

**Explanation**:  
Use `using` for `IDisposable` objects.

**Follow-up**:  
- Can you nest `using` blocks?

---

### 26. What is the difference between `List<T>` and `Dictionary<TKey, TValue>`?

**Answer**:  
`List<T>` stores items sequentially.  
`Dictionary` stores key-value pairs.

**Explanation**:  
Use `Dictionary` for fast lookup by key.

**Follow-up**:  
- What happens if you add duplicate keys?

---

### 27. What is the difference between `Array` and `ArrayList`?

**Answer**:  
`Array` is strongly typed.  
`ArrayList` stores objects and requires casting.

**Explanation**:  
Prefer `List<T>` over `ArrayList` in modern C#.

**Follow-up**:  
- Why is `ArrayList` considered obsolete?

---

### 28. What is the difference between `lock` and `Monitor`?

**Answer**:  
`lock` is syntactic sugar for `Monitor.Enter/Exit`.

**Explanation**:  
Use `lock` for thread safety.

**Follow-up**:  
- What happens if a thread throws inside a `lock`?

---

### 29. What is the difference between `Task.Wait()` and `await`?

**Answer**:  
`Task.Wait()` blocks; `await` is non-blocking.

**Explanation**:  
Use `await` in async methods to avoid deadlocks.

**Follow-up**:  
- Can you use `await` in constructors?

---

### 30. What is the difference between `Func`, `Action`, and `Predicate`?

**Answer**:  
- `Func`: returns a value  
- `Action`: no return  
- `Predicate`: returns bool

**Explanation**:  
```csharp
Func<int, int> square = x => x * x;
```

**Follow-up**:  
- Can you chain multiple `Func` delegates?

---

### 31. What is the difference between `GetType()` and `typeof()`?

**Answer**:  
`GetType()` is runtime; `typeof()` is compile-time.

**Explanation**:  
```csharp
obj.GetType();  
typeof(string);
```

**Follow-up**:  
- Can you compare types using `==`?

---

### 32. What is the difference between `Equals()` and `ReferenceEquals()`?

**Answer**:  
`Equals()` compares values; `ReferenceEquals()` compares memory addresses.

**Explanation**:  
Use `ReferenceEquals()` for object identity.

**Follow-up**:  
- When would `Equals()` return true but `ReferenceEquals()` false?

---

### 33. What is the difference between `delegate` and `lambda`?

**Answer**:  
Lambda is a concise syntax for anonymous delegates.

**Explanation**:  
```csharp
delegate int MyDelegate(int x);  
MyDelegate d = x => x * x;
```

**Follow-up**:  
- Can lambdas capture variables?

---

### 34. What is the difference between `null` and `default`?

**Answer**:  
`null` is absence of value.  
`default` returns default value of type.

**Explanation**:  
```csharp
default(int); // 0  
default(string); // null
```

**Follow-up**:  
- What is `default` for structs?

---

### 35. What is the difference between `throw` and `throw ex`?

**Answer**:  
`throw` preserves stack trace.  
`throw ex` resets it.

**Explanation**:  
Prefer `throw` to maintain debugging info.

**Follow

Absolutely! Let's continue from where we left off. Here are the remaining advanced C# interview questions (36–50), each with a correct answer, detailed explanation, and relevant follow-up questions.

---

### 36. What is the difference between throw and throw ex?

**Answer**:  
`throw` preserves the original stack trace.  
`throw ex` resets the stack trace, making debugging harder.

**Explanation**:  
```csharp
try
{
    // some code
}
catch (Exception ex)
{
    throw;       // preserves stack trace
    // throw ex; // resets stack trace
}
```

**Follow-up**:  
- Why is preserving the stack trace important in production debugging?

---

### 37. What is the difference between synchronous and asynchronous programming?

**Answer**:  
Synchronous code blocks execution until completion.  
Asynchronous code allows other operations to run while waiting.

**Explanation**:  
Use `async/await` for non-blocking I/O operations, improving scalability.

**Follow-up**:  
- How does async improve server throughput?

---

### 38. What is the difference between value types and reference types?

**Answer**:  
Value types store data directly; reference types store a reference to the data.

**Explanation**:  
Value types: int, bool, struct  
Reference types: class, string, array

**Follow-up**:  
- What happens when you pass a value type to a method?

---

### 39. What is the difference between shallow copy and deep copy?

**Answer**:  
Shallow copy duplicates references; deep copy duplicates objects recursively.

**Explanation**:  
Use serialization or manual cloning for deep copy.

**Follow-up**:  
- How do you implement deep copy in C#?

---

### 40. What is the difference between nullable types and default values?

**Answer**:  
Nullable types allow value types to hold null.  
Default values are type-specific defaults.

**Explanation**:  
```csharp
int? x = null;  
int y = default; // 0
```

**Follow-up**:  
- How do you check if a nullable has a value?

---

### 41. What is the difference between implicit and explicit conversion?

**Answer**:  
Implicit conversion is automatic; explicit requires casting.

**Explanation**:  
```csharp
int x = 10;  
double y = x; // implicit  
int z = (int)y; // explicit
```

**Follow-up**:  
- What happens during narrowing conversions?

---

### 42. What is the difference between extension methods and static helper methods?

**Answer**:  
Extension methods appear as instance methods on the extended type.

**Explanation**:  
```csharp
public static class StringExtensions
{
    public static bool IsNullOrEmpty(this string str) => string.IsNullOrEmpty(str);
}
```

**Follow-up**:  
- Can you override extension methods?

---

### 43. What is the difference between IEnumerable and IEnumerator?

**Answer**:  
`IEnumerable` provides an enumerator; `IEnumerator` performs the iteration.

**Explanation**:  
Use `foreach` with `IEnumerable`; `IEnumerator` is used internally.

**Follow-up**:  
- How do you implement custom iteration?

---

### 44. What is the difference between static constructor and instance constructor?

**Answer**:  
Static constructor initializes static members; instance constructor initializes object state.

**Explanation**:  
Static constructor runs once per type, before any instance is created.

**Follow-up**:  
- Can you overload static constructors?

---

### 45. What is the difference between method hiding and method overriding?

**Answer**:  
Hiding uses `new`; overriding uses `override`.

**Explanation**:  
Overriding enables polymorphism; hiding does not.

**Follow-up**:  
- What happens when you call a hidden method via base reference?

---

### 46. What is the difference between early binding and late binding?

**Answer**:  
Early binding resolves method calls at compile time; late binding at runtime.

**Explanation**:  
Late binding uses reflection or `dynamic`.

**Follow-up**:  
- What are performance implications of late binding?

---

### 47. What is the difference between thread safety and reentrancy?

**Answer**:  
Thread safety ensures correct behavior in multithreaded access.  
Reentrancy allows a method to be safely interrupted and re-entered.

**Explanation**:  
Thread-safe code uses locks; reentrant code avoids shared state.

**Follow-up**:  
- How do you make a method reentrant?

---

### 48. What is the difference between sealed class and sealed method?

**Answer**:  
Sealed class cannot be inherited.  
Sealed method cannot be overridden further.

**Explanation**:  
Use sealed to prevent extension or override.

**Follow-up**:  
- Can you seal a method in a non-sealed class?

---

### 49. What is the difference between GC.Collect() and automatic garbage collection?

**Answer**:  
`GC.Collect()` forces garbage collection.  
Automatic GC runs based on memory pressure.

**Explanation**:  
Manual collection is discouraged unless necessary.

**Follow-up**:  
- What are the generations in .NET GC?

---

### 50. What is the difference between AppDomain and Process?

**Answer**:  
AppDomain is a lightweight isolation unit within a process.  
Process is the OS-level execution unit.

**Explanation**:  
AppDomains allow multiple applications in one process (used in .NET Framework).

**Follow-up**:  
- Is AppDomain supported in .NET Core?

---
Absolutely! Here's the complete, print-friendly table of **Advanced C# Interview Questions (1–50)**, formatted to fit within approximately 120 characters per row—ideal for printing on A4 paper in portrait mode.

---

