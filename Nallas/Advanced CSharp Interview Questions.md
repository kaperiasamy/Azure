## Advanced C# Interview Questions (1–50) – Print-Friendly Format

| No | Question                                      | Answer Summary                                | Key Explanation / Example                    | Follow-up Question                          |
|----|-----------------------------------------------|-----------------------------------------------|----------------------------------------------|---------------------------------------------|
| 1  | ref vs out parameters                         | ref: needs init; out: must be assigned        | ref passes by reference; out for output only | Can you use ref/out in async methods?       |
| 2  | Task vs Thread                                | Task: high-level; Thread: low-level           | Task uses TPL; Thread is manual              | How does Task handle exceptions?            |
| 3  | IEnumerable vs IQueryable                     | IEnumerable: in-memory; IQueryable: remote    | IQueryable builds SQL; IEnumerable runs in .NET | When to prefer IEnumerable?             |
| 4  | Boxing vs Unboxing                            | Boxing: value to object; Unboxing: reverse    | Impacts performance; avoid in loops          | How to avoid unnecessary boxing?            |
| 5  | Abstract class vs Interface                   | Abstract: partial impl; Interface: contract   | Interface allows multiple inheritance        | How did C# 8 change interfaces?             |
| 6  | yield return                                  | Enables deferred execution                    | Simplifies custom iterators                  | How does yield affect memory usage?         |
| 7  | readonly vs const                             | const: compile-time; readonly: runtime        | readonly set in constructor                  | Can readonly be static?                     |
| 8  | == vs Equals()                                | ==: reference/value; Equals: value            | Equals can be overridden                     | How to override Equals and GetHashCode?     |
| 9  | static vs singleton                           | static: no instance; singleton: one instance  | Singleton uses private constructor           | How to make singleton thread-safe?          |
| 10 | SOLID principles                              | SRP, OCP, LSP, ISP, DIP                        | Promotes maintainable, scalable code         | Example of violating SRP?                   |
| 11 | Dispose vs Finalize                           | Dispose: manual cleanup; Finalize: GC cleanup | Dispose via IDisposable; Finalize via GC     | How does using relate to Dispose()?         |
| 12 | async vs await                                | async marks method; await pauses execution    | Enables non-blocking code                    | What if you forget to use await?            |
| 13 | var vs dynamic vs object                      | var: inferred; dynamic: runtime; object: base | dynamic skips compile-time checks            | Risks of using dynamic?                     |
| 14 | Covariance vs Contravariance                  | Covariance: return; Contravariance: input     | Used in delegates and generics               | Where is this used in delegates?            |
| 15 | Thread.Sleep vs Task.Delay                    | Sleep blocks thread; Delay is async           | Prefer Delay in async methods                | What if Sleep is used in UI thread?         |
| 16 | Struct vs Class                               | Struct: value type; Class: reference type     | Structs on stack; Classes on heap            | Can structs have constructors?              |
| 17 | Access modifiers                              | public, private, protected, internal          | Controls visibility and encapsulation        | What is protected internal?                 |
| 18 | Interface vs Delegate                         | Interface: contract; Delegate: method pointer | Interface for polymorphism; Delegate for callbacks | Can you use delegates with events?     |
| 19 | Event vs Delegate                             | Event wraps delegate for safety               | Events restrict external invocation          | How to unsubscribe from an event?           |
| 20 | Static vs Instance members                    | Static: type-level; Instance: object-level    | Static shared across instances               | Can static methods access instance members? |
| 21 | override vs new vs virtual                    | virtual: allow override; new: hide; override: replace | Enables polymorphism               | What if you forget override?                |
| 22 | is vs as operators                            | is: type check; as: safe cast                 | as returns null if cast fails                | What if as fails?                           |
| 23 | params keyword                                | Allows variable arguments                     | Simplifies method calls                      | Can you use multiple params?                |
| 24 | sealed vs abstract                            | sealed: no inheritance; abstract: must inherit| sealed is final; abstract is incomplete      | Can a class be both sealed and abstract?    |
| 25 | try-catch-finally vs using                    | try-catch handles errors; using disposes      | using is shorthand for Dispose               | Can you nest using blocks?                  |
| 26 | List vs Dictionary                            | List: sequence; Dictionary: key-value         | Dictionary for fast lookup                   | What if you add duplicate keys?             |
| 27 | Array vs ArrayList                            | Array: typed; ArrayList: object-based         | Prefer List<T> over ArrayList                | Why is ArrayList obsolete?                  |
| 28 | lock vs Monitor                               | lock is shorthand for Monitor.Enter/Exit      | Both ensure thread safety                    | What if exception occurs inside lock?       |
| 29 | Task.Wait vs await                            | Wait blocks; await is non-blocking            | await avoids deadlocks                       | Can you use await in constructors?          |
| 30 | Func vs Action vs Predicate                   | Func: returns; Action: void; Predicate: bool  | Used in LINQ and delegates                   | Can you chain Func delegates?               |
| 31 | GetType vs typeof                             | GetType: runtime; typeof: compile-time        | Used for reflection and comparison           | Can you compare types using ==?             |
| 32 | Equals vs ReferenceEquals                     | Equals: value; ReferenceEquals: memory        | Equals can be overridden                     | When do Equals and ReferenceEquals differ?  |
| 33 | delegate vs lambda                            | Lambda is concise delegate syntax             | Lambdas simplify anonymous methods           | Can lambdas capture variables?              |
| 34 | null vs default                               | null: no value; default: type default         | default(int) = 0; default(string) = null     | What is default for structs?                |
| 35 | throw vs throw ex                             | throw preserves stack; throw ex resets it     | Use throw for better debugging               | Why preserve stack trace?                   |
| 36 | Sync vs Async programming                     | Sync blocks; Async allows continuation        | Async improves scalability                   | How does async help server throughput?      |
| 37 | Value vs Reference types                      | Value: direct data; Reference: memory address | Value types on stack; Reference on heap      | What happens when passing value types?      |
| 38 | Shallow vs Deep copy                          | Shallow: references; Deep: full copy          | Deep copy duplicates nested objects          | How to implement deep copy?                 |
| 39 | Nullable vs Default values                    | Nullable allows null; default returns default | int? x = null; int y = default;              | How to check if nullable has value?         |
| 40 | Implicit vs Explicit conversion               | Implicit: safe; Explicit: cast required       | Explicit needed for narrowing conversions    | What happens during narrowing?              |
| 41 | Extension vs Static helper methods            | Extension appears as instance method          | Uses this keyword in static class            | Can you override extension methods?         |
| 42 | IEnumerable vs IEnumerator                    | IEnumerable: provides; IEnumerator: iterates  | foreach uses IEnumerable                     | How to implement custom iteration?          |
| 43 | Static vs Instance constructor                | Static: type init; Instance: object init      | Static runs once per type                    | Can you overload static constructors?       |
| 44 | Method hiding vs overriding                   | hiding: new; overriding: override             | Overriding supports polymorphism             | What if base reference calls hidden method? |
| 45 | Early vs Late binding                         | Early: compile-time; Late: runtime            | Late uses reflection or dynamic              | Performance impact of late binding?         |
| 46 | Thread safety vs Reentrancy                   | Thread safety: no race; Reentrancy: safe re-entry | Reentrant avoids shared state            | How to make a method reentrant?             |
| 47 | Sealed class vs Sealed method                 | Class: no inheritance; Method: no override    | Use sealed to finalize behavior              | Can you seal method in non-sealed class?    |
| 48 | GC.Collect vs automatic GC                    | GC.Collect forces cleanup; auto is managed    | Manual GC discouraged                        | What are GC generations?                    |
| 49 | AppDomain vs Process                          | AppDomain: .NET isolation; Process: OS unit   | AppDomain deprecated in .NET Core            | Is AppDomain supported in .NET Core?        |
| 50 | Boxing vs Unboxing                            | Boxing: value to object; Unboxing: reverse    | Impacts performance; avoid in loops          | How to avoid unnecessary boxing?            |

---
