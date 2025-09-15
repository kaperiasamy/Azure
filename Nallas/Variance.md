# 1. What is Variance?

**Variance** means: *How subtyping relationships between types affect subtyping relationships between their generic constructions.*

Put simply:
If `Dog` derives from `Animal`, then what about `List<Dog>` vs `List<Animal>`?

There are three possible answers → **Covariance, Contravariance, Invariance**.

---

# 2. Covariance (`out`)

**Definition**:
A generic type is **covariant** if the subtyping is preserved in the **same direction**.

Formally:
If `Dog : Animal` → then `IEnumerable<Dog> : IEnumerable<Animal>`.

**Real meaning**:
Covariance means "you can substitute a more derived type safely when producing values."

**Example**:

```csharp
IEnumerable<Dog> dogs = new List<Dog>();
IEnumerable<Animal> animals = dogs; // ✅ covariance
```

---

# 3. Contravariance (`in`)

**Definition**:
A generic type is **contravariant** if the subtyping is reversed.

Formally:
If `Dog : Animal` → then `IComparer<Animal> : IComparer<Dog>`.

**Real meaning**:
Contravariance means "you can substitute a more general type safely when consuming values."

**Example**:

```csharp
Action<Animal> actAnimal = a => Console.WriteLine(a.GetType().Name);
Action<Dog> actDog = actAnimal; // ✅ contravariance
```

---

# 4. Invariance

**Definition**:
A generic type is **invariant** if no subtyping relationship is preserved.

Formally:
If `Dog : Animal` → then `List<Dog>` is **not** related to `List<Animal>`.

**Real meaning**:
Invariance means "no variance is applied; the types must match exactly."

**Example**:

```csharp
List<Dog> dogs = new List<Dog>();
List<Animal> animals = dogs; // ❌ compile-time error
```

---

# 5. Why This Matters in Solution Architecture

* **Covariance**: For **data producers** (e.g., repositories, queries, APIs returning objects).
* **Contravariance**: For **data consumers** (e.g., event handlers, processors, comparers).
* **Invariance**: For **mutable structures** (e.g., `List<T>`), because allowing variance breaks type safety.

**Rule of thumb for architecture**:

* Use **covariant interfaces** (`IEnumerable<out T>`, `IReadOnlyList<out T>`) for APIs returning data.
* Use **contravariant interfaces** (`IComparer<in T>`, `IEqualityComparer<in T>`) for APIs consuming data.
* Keep **invariant collections** (`List<T>`, `Dictionary<TKey, TValue>`) inside implementation details.

---

# 6. ASCII Diagram

```
Animal
  ▲
  │
Dog

Covariance:    IEnumerable<Dog>   --->  IEnumerable<Animal>
Contravariance: Action<Animal>    --->  Action<Dog>
Invariance:    List<Dog>    !=    List<Animal>
```

---

# 7. Follow-up Questions

**Q1: Why is List<T> invariant?**
A: Because it both consumes and produces, so neither covariance nor contravariance is safe.

**Q2: Can a type be both covariant and contravariant?**
A: No, a single generic parameter can’t be both. But an interface can declare multiple type parameters, some covariant, some contravariant.

**Q3: How does variance affect API versioning?**
A: By using variance correctly (`out` and `in`), you make APIs **more extensible** without breaking existing consumers.

---

**Summary (Interview phrasing)**:

* **Variance** is about how inheritance works with generics.
* **Covariance (`out`)**: subtype preserved (Dog → Animal). Use for producers.
* **Contravariance (`in`)**: subtype reversed (Animal → Dog). Use for consumers.
* **Invariance**: no relationship. Use when mutation makes variance unsafe.

---

