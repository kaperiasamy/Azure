IEnumerable, ICollection and IList:
The difference, use cases and importance.

**IEnumerable**:
- Supports simple iteration over a collection.

```csharp
IEnumerable<string> languages = new List<string> {"C#", "Java", "Python"};
foreach(string language in languages)
    Console.WriteLine(language);
```

**ICollection**:
- Supports adding, removing, and counting elements

```csharp
ICollection<string> languages = new List<string>();
languages.Add("C#");
languages.Add("Java");
languages.Add("Python");
int count = languages.Count;
```

**IList**:
- Supports indexed access and modification

```csharp
IList<string> languages = new List<string> {"C#", "Java", "Python"};
languages[1] = "JavaScript";
```