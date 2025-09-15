# C# – Advanced Interview Questions (1–25)

---

### 1. What is the difference between `ref` and `out` parameters?

**Answer**:  
- `ref` requires the variable to be initialized before passing.  
- `out` does not require initialization but must be assigned inside the method.

**Explanation**:  
Both allow passing arguments by reference, but `ref` is for input/output, while `out` is strictly for output.

**Example**:
```csharp
void AddRef(ref int x) {
    x += 10;
}

void AddOut(out int y) {
    y = 20;
}

int a = 5;
AddRef(ref a); // a becomes 15

int b;
AddOut(out b); // b becomes 20
```

**Follow-up**:  
- Can you use `ref` and `out` with properties or async methods?

---

### 2. What is the difference between `Task` and `Thread`?

**Answer**:  
- `Thread` is a low-level construct for parallel execution.  
- `Task` is higher-level and uses thread pooling, ideal for async operations.

**Explanation**:  
`Task` is part of the Task Parallel Library and integrates with `async/await`.

**Example**:
```csharp
Task.Run(() => Console.WriteLine("Running with Task"));

new Thread(() => Console.WriteLine("Running with Thread")).Start();
```

**Follow-up**:  
- How does `Task` handle exceptions compared to `Thread`?

---

### 3. What is boxing and unboxing?

**Answer**:  
- **Boxing**: Converting a value type to `object`.  
- **Unboxing**: Extracting the value type from `object`.

**Explanation**:  
Boxing stores value types on the heap, which can impact performance.

**Example**:
```csharp
int num = 42;
object boxed = num; // Boxing
int unboxed = (int)boxed; // Unboxing
```

**Follow-up**:  
- How can you avoid unnecessary boxing in collections?

---

### 4. What is the difference between `IEnumerable` and `IQueryable`?

**Answer**:  
- `IEnumerable` executes queries in memory.  
- `IQueryable` builds expression trees and executes on the data source.

**Explanation**:  
Use `IQueryable` for deferred execution and database-side filtering.

**Example**:
```csharp
IEnumerable<Product> products = db.Products.ToList(); // In-memory
IQueryable<Product> query = db.Products.Where(p => p.Price > 100); // SQL executed
```

**Follow-up**:  
- What happens if you call `.ToList()` on an `IQueryable`?

---

### 5. What is the difference between `abstract` class and `interface`?

**Answer**:  
- Abstract classes can have implementation.  
- Interfaces define only contracts (until C# 8).

**Explanation**:  
Use abstract classes for shared base logic; interfaces for polymorphism.

**Example**:
```csharp
abstract class Animal {
    public abstract void Speak();
}

interface IAnimal {
    void Speak();
}
```

**Follow-up**:  
- Can a class implement multiple interfaces and inherit an abstract class?

---

### 6. What is the use of `yield return`?

**Answer**:  
It enables deferred execution and simplifies iterator logic.

**Explanation**:  
`yield return` lets you return values one at a time without building a collection.

**Example**:
```csharp
public IEnumerable<int> GetNumbers() {
    yield return 1;
    yield return 2;
    yield return 3;
}
```

**Follow-up**:  
- How does `yield return` affect memory and performance?

---

### 7. What is the difference between `readonly` and `const`?

**Answer**:  
- `const` is compile-time constant.  
- `readonly` is runtime constant, set in constructor.

**Explanation**:
```csharp
const int Max = 100;
readonly int Min;

public MyClass() {
    Min = 10;
}
```

**Follow-up**:  
- Can `readonly` be static?

---

### 8. What is the difference between `==` and `.Equals()`?

**Answer**:  
- `==` compares references unless overloaded.  
- `.Equals()` compares values.

**Explanation**:
```csharp
string a = "hello";
string b = new string("hello".ToCharArray());

Console.WriteLine(a == b);       // True
Console.WriteLine(a.Equals(b));  // True
```

**Follow-up**:  
- How do you override `Equals()` and `GetHashCode()`?

---

### 9. What is a delegate?

**Answer**:  
A delegate is a type-safe reference to a method.

**Explanation**:
```csharp
delegate int MathOperation(int x, int y);

MathOperation add = (a, b) => a + b;
Console.WriteLine(add(2, 3)); // Output: 5
```

**Follow-up**:  
- How are delegates used in event handling?

---

### 10. What is an event in C#?

**Answer**:  
An event is a wrapper around a delegate that restricts invocation to the declaring class.

**Explanation**:
```csharp
public event Action OnClick;

public void Trigger() {
    OnClick?.Invoke();
}
```

**Follow-up**:  
- How do you unsubscribe from an event?

---

### 11. What is the difference between `Func`, `Action`, and `Predicate`?

**Answer**:  
- `Func`: returns a value.  
- `Action`: returns void.  
- `Predicate`: returns a boolean.

**Explanation**:
```csharp
Func<int, int> square = x => x * x;
Action<string> greet = name => Console.WriteLine($"Hello {name}");
Predicate<int> isEven = x => x % 2 == 0;
```

**Follow-up**:  
- Can you use these with LINQ?

---

### 12. What is the difference between `var`, `dynamic`, and `object`?

**Answer**:  
- `var`: compile-time type inference.  
- `dynamic`: runtime type resolution.  
- `object`: base type of all types.

**Explanation**:
```csharp
var x = 5;           // int
dynamic y = "hello"; // runtime
object z = 3.14;     // boxed double
```

**Follow-up**:  
- What are the risks of using `dynamic`?

---

### 13. What is the difference between `is` and `as`?

**Answer**:  
- `is`: checks type compatibility.  
- `as`: performs safe casting.

**Explanation**:
```csharp
object obj = "text";
if (obj is string) { Console.WriteLine("It's a string"); }

string str = obj as string;
```

**Follow-up**:  
- What happens if `as` fails?

---

### 14. What is the difference between `override`, `new`, and `virtual`?

**Answer**:  
- `virtual`: allows method to be overridden.  
- `override`: replaces base method.  
- `new`: hides base method.

**Explanation**:
```csharp
class Base {
    public virtual void Show() => Console.WriteLine("Base");
}

class Derived : Base {
    public override void Show() => Console.WriteLine("Derived");
}
```

**Follow-up**:  
- What happens if you use `new` instead of `override`?

---

### 15. What is the difference between `static` and `singleton`?

**Answer**:  
- `static`: class cannot be instantiated.  
- `singleton`: ensures one instance.

**Explanation**:
```csharp
public sealed class Singleton {
    private static readonly Singleton _instance = new Singleton();
    public static Singleton Instance => _instance;
    private Singleton() { }
}
```

**Follow-up**:  
- How do you make singleton thread-safe?

---

### 16. What is the difference between `throw` and `throw ex`?

**Answer**:  
- `throw` preserves the original stack trace.  
- `throw ex` resets the stack trace, making debugging harder.

**Explanation**:  
Use `throw` to rethrow exceptions without losing the original context.

**Example**:
```csharp
try {
    // some code
} catch (Exception ex) {
    throw;       // preserves stack trace
    // throw ex; // resets stack trace
}
```

**Follow-up**: Why is preserving the stack trace important?  
**Answer**: It helps identify the exact location and sequence of method calls that led to the exception, which is crucial for debugging.

---

### 17. What is the difference between `null` and `default`?

**Answer**:  
- `null` represents the absence of a value.  
- `default` returns the default value of a type.

**Explanation**:
```csharp
int? x = null;              // nullable int with no value
int y = default(int);       // y = 0
string s = default(string); // s = null
```

**Follow-up**: What is `default` for custom structs?  
**Answer**: It initializes all fields to their default values (e.g., numbers to 0, references to null).

---

### 18. What is the difference between `GetType()` and `typeof()`?

**Answer**:  
- `GetType()` is used at runtime on an object.  
- `typeof()` is used at compile time with a type name.

**Explanation**:
```csharp
object obj = "hello";
Console.WriteLine(obj.GetType());     // System.String
Console.WriteLine(typeof(string));    // System.String
```

**Follow-up**: Can you compare types using `==`?  
**Answer**: Yes. `typeof(string) == obj.GetType()` returns true if the object is a string.

---

### 19. What is the difference between `Array` and `ArrayList`?

**Answer**:  
- `Array` is strongly typed.  
- `ArrayList` stores objects and requires casting.

**Explanation**:
```csharp
int[] numbers = new int[] { 1, 2, 3 };
ArrayList list = new ArrayList();
list.Add(1);
int x = (int)list[0]; // requires casting
```

**Follow-up**: Why is `ArrayList` considered obsolete?  
**Answer**: Because it lacks type safety and performance benefits of generics. Use `List<T>` instead.

---

### 20. What is the difference between `List<T>` and `Dictionary<TKey, TValue>`?

**Answer**:  
- `List<T>` stores items sequentially.  
- `Dictionary` stores key-value pairs for fast lookup.

**Explanation**:
```csharp
List<string> names = new List<string> { "Alice", "Bob" };
Dictionary<int, string> users = new Dictionary<int, string> {
    {1, "Alice"}, {2, "Bob"}
};
```

**Follow-up**: What happens with duplicate keys in a dictionary?  
**Answer**: It throws an exception (`ArgumentException`). Keys must be unique.

---

### 21. What is the difference between `lock` and `Monitor`?

**Answer**:  
- `lock` is syntactic sugar for `Monitor.Enter/Exit`.  
- `Monitor` provides more control (e.g., timeout, try-enter).

**Explanation**:
```csharp
lock(obj) {
    // thread-safe code
}

// Equivalent using Monitor
Monitor.Enter(obj);
try {
    // thread-safe code
} finally {
    Monitor.Exit(obj);
}
```

**Follow-up**: What happens if a thread throws inside a `lock` block?  
**Answer**: The lock is released automatically. With `Monitor`, you must ensure `Exit()` is called in `finally`.

---

### 22. What is the difference between `Thread.Sleep()` and `Task.Delay()`?

**Answer**:  
- `Thread.Sleep()` blocks the thread.  
- `Task.Delay()` is asynchronous and does not block.

**Explanation**:
```csharp
await Task.Delay(1000); // non-blocking
Thread.Sleep(1000);     // blocks thread
```

**Follow-up**: What happens if you use `Sleep` in a UI thread?  
**Answer**: It freezes the UI, making the application unresponsive.

---

### 23. What is the difference between `readonly` and `static readonly`?

**Answer**:  
- `readonly`: instance-level, set in constructor.  
- `static readonly`: type-level, set in static constructor.

**Explanation**:
```csharp
readonly int instanceValue;
static readonly int staticValue;

public MyClass() {
    instanceValue = 10;
}

static MyClass() {
    staticValue = 20;
}
```

**Follow-up**: Can you assign `readonly` outside the constructor?  
**Answer**: No. It must be assigned during declaration or in the constructor.

---

### 24. What is the difference between `sealed` and `abstract`?

**Answer**:  
- `sealed` prevents inheritance.  
- `abstract` enforces inheritance and cannot be instantiated.

**Explanation**:
```csharp
sealed class FinalClass { }

abstract class BaseClass {
    public abstract void DoWork();
}
```

**Follow-up**: Can a class be both `sealed` and `abstract`?  
**Answer**: No. `sealed` means no inheritance; `abstract` requires inheritance.

---

### 25. What is the difference between `delegate` and `event`?

**Answer**:  
- `delegate` is a type-safe function pointer.  
- `event` wraps a delegate to restrict invocation to the declaring class.

**Explanation**:
```csharp
public delegate void Notify();
public event Notify OnNotify;

public void Trigger() {
    OnNotify?.Invoke();
}
```

**Follow-up**: How do you raise an event safely?  
**Answer**: Use null-conditional operator: `OnNotify?.Invoke();` to avoid `NullReferenceException`.

---
Absolutely! Let’s dive into the **ASP.NET MVC** section with the same depth and clarity as before. Below are **25 advanced ASP.NET MVC interview questions (26–50)**, each with:

- ✅ A correct answer  
- 💡 Clear explanation  
- 🧪 Example code  
- 🔄 Follow-up question  
- ✅ Follow-up answer  

---

## ASP.NET MVC – Advanced Interview Questions (26–50)

---

### 26. What is the MVC design pattern?

**Answer**:  
MVC stands for Model-View-Controller, a design pattern that separates concerns.

**Explanation**:  
- **Model**: Handles business logic and data.  
- **View**: Displays data to the user.  
- **Controller**: Manages user input and coordinates between model and view.

**Example**:
```csharp
public class Product {
    public int Id { get; set; }
    public string Name { get; set; }
}

public class ProductController : Controller {
    public ActionResult Details() {
        var product = new Product { Id = 1, Name = "Laptop" };
        return View(product);
    }
}
```

**Follow-up**: How does MVC differ from MVVM?  
**Answer**: MVVM introduces a ViewModel between View and Model, mainly used in WPF for data binding and command handling.

---

### 27. What is routing in ASP.NET MVC?

**Answer**:  
Routing maps incoming URLs to controller actions.

**Explanation**:  
Defined in `RouteConfig.cs` using `routes.MapRoute`.

**Example**:
```csharp
routes.MapRoute(
    name: "Default",
    url: "{controller}/{action}/{id}",
    defaults: new { controller = "Home", action = "Index", id = UrlParameter.Optional }
);
```

**Follow-up**: What is attribute routing?  
**Answer**: Attribute routing uses route attributes directly on controller actions:
```csharp
[Route("products/{id}")]
public ActionResult Details(int id) { ... }
```

---

### 28. What are Action Results?

**Answer**:  
Action Results represent the outcome of a controller action.

**Explanation**:  
Common types include `ViewResult`, `JsonResult`, `RedirectResult`, `ContentResult`.

**Example**:
```csharp
public ActionResult Show() {
    return View(); // ViewResult
}

public ActionResult GetJson() {
    return Json(new { Name = "Laptop" }, JsonRequestBehavior.AllowGet);
}
```

**Follow-up**: When would you use `PartialViewResult`?  
**Answer**: When rendering a portion of a view asynchronously or inside another view.

---

### 29. What is the difference between TempData, ViewData, and ViewBag?

**Answer**:  
- `TempData`: persists across redirects.  
- `ViewData`: dictionary for passing data to views.  
- `ViewBag`: dynamic wrapper over `ViewData`.

**Explanation**:
```csharp
ViewData["Message"] = "Hello";
ViewBag.Message = "Hello";
TempData["Notice"] = "Saved!";
```

**Follow-up**: Which one is best for passing data between actions?  
**Answer**: `TempData` is best because it survives redirects.

---

### 30. What are filters in MVC?

**Answer**:  
Filters allow logic to run before or after controller actions.

**Explanation**:  
Types include `AuthorizationFilter`, `ActionFilter`, `ResultFilter`, `ExceptionFilter`.

**Example**:
```csharp
public class LogActionFilter : ActionFilterAttribute {
    public override void OnActionExecuting(ActionExecutingContext filterContext) {
        // Logging logic
    }
}
```

**Follow-up**: How do you apply a filter globally?  
**Answer**: Register it in `FilterConfig.cs`:
```csharp
filters.Add(new LogActionFilter());
```

---

### 31. What is model binding?

**Answer**:  
Model binding maps HTTP request data to action method parameters.

**Explanation**:
```csharp
public ActionResult Create(Product product) {
    // Automatically binds form fields to Product properties
}
```

**Follow-up**: How do you customize model binding?  
**Answer**: Implement `IModelBinder` and register it in `ModelBinderConfig`.

---

### 32. What is validation in MVC?

**Answer**:  
Validation ensures data integrity using attributes or custom logic.

**Explanation**:
```csharp
public class Product {
    [Required]
    [Range(1, 10000)]
    public decimal Price { get; set; }
}
```

**Follow-up**: How do you enable client-side validation?  
**Answer**: Include `jquery.validate` and `jquery.validate.unobtrusive` scripts.

---

### 33. What is the role of `HtmlHelper`?

**Answer**:  
`HtmlHelper` generates HTML elements in Razor views.

**Explanation**:
```csharp
@Html.TextBoxFor(m => m.Name)
@Html.ValidationMessageFor(m => m.Name)
```

**Follow-up**: How do you create a custom HTML helper?  
**Answer**:
```csharp
public static class MyHelpers {
    public static MvcHtmlString Bold(this HtmlHelper html, string text) {
        return new MvcHtmlString($"<b>{text}</b>");
    }
}
```

---

### 34. What is Razor syntax?

**Answer**:  
Razor is a markup syntax for embedding C# in HTML.

**Explanation**:
```csharp
@model Product
<p>@Model.Name</p>
```

**Follow-up**: How does Razor differ from Web Forms?  
**Answer**: Razor is cleaner, uses `@` for code, and avoids verbose tags like `<% %>`.

---

### 35. What is the difference between `RenderPartial` and `RenderAction`?

**Answer**:  
- `RenderPartial`: renders a partial view.  
- `RenderAction`: invokes a controller action and renders its result.

**Explanation**:
```csharp
@Html.Partial("_ProductPartial", Model)
@Html.Action("Summary", "Product")
```

**Follow-up**: Which is better for dynamic content?  
**Answer**: `RenderAction` is better when the partial view needs controller logic.

---

### 36. What is the purpose of `BundleConfig`?

**Answer**:  
Combines and minifies CSS and JS files to improve performance.

**Explanation**:
```csharp
bundles.Add(new ScriptBundle("~/bundles/jquery").Include(
    "~/Scripts/jquery-{version}.js"));
```

**Follow-up**: How do you enable bundling in production?  
**Answer**: Set `BundleTable.EnableOptimizations = true;` in `Global.asax`.

---

### 37. What is the role of `Global.asax`?

**Answer**:  
Handles application-level events like `Application_Start`.

**Explanation**:
```csharp
protected void Application_Start() {
    RouteConfig.RegisterRoutes(RouteTable.Routes);
}
```

**Follow-up**: What replaces `Global.asax` in .NET Core?  
**Answer**: `Program.cs` and `Startup.cs` handle app configuration in .NET Core.

---

### 38. What is scaffolding in MVC?

**Answer**:  
Scaffolding auto-generates code for CRUD operations.

**Explanation**:  
Use Visual Studio or CLI to scaffold controllers and views.

**Follow-up**: Can you customize scaffolding templates?  
**Answer**: Yes, by modifying T4 templates in the `CodeTemplates` folder.

---

### 39. What is the difference between synchronous and asynchronous controllers?

**Answer**:  
Async controllers use `async/await` for non-blocking operations.

**Explanation**:
```csharp
public async Task<ActionResult> Index() {
    var data = await service.GetDataAsync();
    return View(data);
}
```

**Follow-up**: When should you avoid async controllers?  
**Answer**: For CPU-bound operations or simple logic where async adds complexity.

---

### 40. What is anti-forgery token?

**Answer**:  
Prevents CSRF attacks by validating form submissions.

**Explanation**:
```csharp
@Html.AntiForgeryToken()
[ValidateAntiForgeryToken]
public ActionResult Submit(FormModel model) { ... }
```

**Follow-up**: How does MVC validate the token?  
**Answer**: It compares the hidden form token with the cookie token.

---

### 41. What is the difference between `HttpGet` and `HttpPost`?

**Answer**:  
- `HttpGet`: retrieves data.  
- `HttpPost`: submits data.

**Explanation**:
```csharp
[HttpGet]
public ActionResult Create() { return View(); }

[HttpPost]
public ActionResult Create(Product model) { ... }
```

**Follow-up**: Can a single action handle both verbs?  
**Answer**: Yes, but it's discouraged. Use method overloading or separate actions.

---

### 42. What is the role of `RouteConfig.cs`?

**Answer**:  
Defines URL patterns and default routes.

**Explanation**:
```csharp
routes.MapRoute("Default", "{controller}/{action}/{id}",
    new { controller = "Home", action = "Index", id = UrlParameter.Optional });
```

**Follow-up**: How do you enable attribute routing?  
**Answer**: Call `routes.MapMvcAttributeRoutes()` in `RouteConfig.cs`.

---

Thanks for pointing that out! Let’s pick up right where we left off and complete the remaining ASP.NET MVC questions (43–50), each with a correct answer, explanation, example code, and follow-up question with its answer.

---

### 43. What is the difference between `View()` and `RedirectToAction()`?

**Answer**:  
- `View()` returns a view directly.  
- `RedirectToAction()` performs an HTTP redirect to another action.

**Explanation**:  
Use `View()` when rendering a view in the same request. Use `RedirectToAction()` when following the Post-Redirect-Get (PRG) pattern.

**Example**:
```csharp
public ActionResult SubmitForm() {
    // Save logic
    return RedirectToAction("Confirmation");
}

public ActionResult Confirmation() {
    return View();
}
```

**Follow-up**: What is the PRG pattern and why is it useful?  
**Answer**: Post-Redirect-Get avoids duplicate form submissions by redirecting after a POST, ensuring the user sees a GET response.

---

### 44. What is the role of `ControllerBase`?

**Answer**:  
`ControllerBase` is the base class for all controllers in ASP.NET MVC.

**Explanation**:  
It provides core functionality like `Request`, `Response`, `ModelState`, and action result methods.

**Example**:
```csharp
public class MyController : Controller {
    public ActionResult Index() {
        if (!ModelState.IsValid) return BadRequest();
        return View();
    }
}
```

**Follow-up**: How does `Controller` extend `ControllerBase`?  
**Answer**: `Controller` adds support for views and Razor rendering, while `ControllerBase` is suitable for APIs or headless controllers.

---

### 45. What is the difference between `PartialView` and `ViewComponent`?

**Answer**:  
- `PartialView` is a reusable view fragment.  
- `ViewComponent` encapsulates both logic and rendering, similar to a mini-controller.

**Explanation**:
```csharp
// PartialView
@Html.Partial("_ProductPartial", Model)

// ViewComponent
@await Component.InvokeAsync("ProductSummary", new { id = 5 })
```

**Follow-up**: When should you prefer `ViewComponent` over `PartialView`?  
**Answer**: Use `ViewComponent` when you need logic, data fetching, or reusability beyond simple markup.

---

### 46. What is the role of `ModelState`?

**Answer**:  
`ModelState` tracks the validity of model binding and validation.

**Explanation**:
```csharp
public ActionResult Save(Product model) {
    if (!ModelState.IsValid) return View(model);
    // Save logic
}
```

**Follow-up**: How do you add custom errors to `ModelState`?  
**Answer**:
```csharp
ModelState.AddModelError("Price", "Price must be greater than zero.");
```

---

### 47. What is the difference between `Area` and `Controller`?

**Answer**:  
- `Area` is a way to group related controllers and views.  
- `Controller` handles specific request logic.

**Explanation**:  
Areas help organize large applications into modules like Admin, User, etc.

**Example**:
```csharp
// In Admin Area
public class DashboardController : Controller {
    public ActionResult Index() => View();
}
```

**Follow-up**: How do you register an Area?  
**Answer**: Use `AreaRegistration`:
```csharp
public class AdminAreaRegistration : AreaRegistration {
    public override void RegisterArea(AreaRegistrationContext context) {
        context.MapRoute("Admin_default", "Admin/{controller}/{action}/{id}", 
            new { action = "Index", id = UrlParameter.Optional });
    }
}
```

---

### 48. What is the role of `ViewEngine`?

**Answer**:  
`ViewEngine` is responsible for locating and rendering views.

**Explanation**:  
ASP.NET MVC uses `RazorViewEngine` by default to parse `.cshtml` files.

**Follow-up**: Can you create a custom `ViewEngine`?  
**Answer**: Yes, by inheriting from `IViewEngine` and registering it in `ViewEngines.Engines`.

---

### 49. What is the difference between `Server.Transfer` and `Response.Redirect`?

**Answer**:  
- `Server.Transfer` transfers execution on the server without changing the URL.  
- `Response.Redirect` sends a 302 redirect to the browser.

**Explanation**:  
`Server.Transfer` is not available in MVC; use `RedirectToAction()` instead.

**Follow-up**: What is the MVC alternative to `Server.Transfer`?  
**Answer**: Use `RedirectToAction()` or `Redirect()` for navigation.

---

### 50. What is the role of `UrlHelper`?

**Answer**:  
`UrlHelper` generates URLs based on routing configuration.

**Explanation**:
```csharp
@Url.Action("Details", "Product", new { id = 5 }) // /Product/Details/5
@Url.Content("~/Content/site.css") // Resolves to absolute path
```

**Follow-up**: How do you use `UrlHelper` in a controller?  
**Answer**:
```csharp
string link = Url.Action("Edit", "Product", new { id = 10 });
```

---



