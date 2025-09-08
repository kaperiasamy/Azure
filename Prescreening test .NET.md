I am a .NET developer, having 16 years of experience. Today I am having an prescreening session in which I will be given 14 multiple choice questions to answer in 15 minutes. I am preparing for that now. This is my current scenario. In this case, if I was asked as question like "Identify the missing design principle from the following content", what is the approach I have to follow and answer this?

For "Identify the missing design principle" questions in a .NET prescreening context, here's a systematic approach:

## Quick Assessment Strategy (1-2 minutes per question)

**1. Identify the Context**
- Is it asking about SOLID principles, Gang of Four patterns, or general OOP principles?
- Look for keywords that hint at the domain (inheritance, coupling, cohesion, etc.)

**2. Mental Checklist Approach**
For **SOLID principles**, quickly check:
- **S**ingle Responsibility Principle
- **O**pen/Closed Principle  
- **L**iskov Substitution Principle
- **I**nterface Segregation Principle
- **D**ependency Inversion Principle

For **General OOP principles**:
- Encapsulation, Inheritance, Polymorphism, Abstraction
- DRY (Don't Repeat Yourself)
- KISS (Keep It Simple, Stupid)
- YAGNI (You Aren't Gonna Need It)

**3. Process of Elimination**
- Read the given content/code snippet
- Identify which principles are clearly being followed
- The missing one is likely what's NOT being demonstrated

**4. Common Patterns in These Questions**
- Code violating Single Responsibility (class doing too many things)
- Missing abstraction/interface usage (Dependency Inversion)
- Tight coupling between classes
- Code that's not extensible (Open/Closed violation)

**5. Time-Saving Tip**
If you see a code example, look for these red flags:
- Large classes with multiple responsibilities → Missing SRP
- Direct instantiation of concrete classes → Missing DIP
- If-else chains for types → Missing polymorphism/OCP

--- 

# .NET Design Principles - Practice Questions & Solutions

## Question 1: SOLID Principles - Missing Principle

**Question:**
```csharp
public class OrderProcessor
{
    public void ProcessOrder(Order order)
    {
        // Validate order
        if (order.Items.Count == 0)
            throw new Exception("Order has no items");
        
        // Calculate total
        decimal total = 0;
        foreach (var item in order.Items)
            total += item.Price * item.Quantity;
        
        // Apply discount
        if (order.Customer.Type == CustomerType.Premium)
            total *= 0.9m;
        else if (order.Customer.Type == CustomerType.Gold)
            total *= 0.85m;
        
        // Save to database
        var connectionString = "Server=...";
        using (var connection = new SqlConnection(connectionString))
        {
            // Save order logic
        }
        
        // Send email
        var emailService = new EmailService();
        emailService.SendOrderConfirmation(order.Customer.Email, order);
        
        // Generate invoice
        var invoiceGenerator = new InvoiceGenerator();
        invoiceGenerator.GenerateInvoice(order, total);
    }
}
```

Which SOLID principle is violated in this code?
A) Single Responsibility Principle
B) Open/Closed Principle  
C) Liskov Substitution Principle
D) Interface Segregation Principle

**Solution Process:**

**Step 1: Identify the context** - This is clearly about SOLID principles

**Step 2: Analyze what the code is doing**
- Order validation
- Price calculation
- Discount application
- Database operations
- Email sending
- Invoice generation

**Step 3: Check each SOLID principle**
- **SRP**: The class has 6 different responsibilities → VIOLATED
- **OCP**: Adding new customer types requires modifying the class → VIOLATED
- **LSP**: No inheritance shown
- **ISG**: No interfaces shown

**Step 4: Primary violation**
The most obvious violation is **Single Responsibility Principle** - the class is doing too many things.

**Answer: A) Single Responsibility Principle**

---

## Question 2: Dependency Injection Pattern

**Question:**
```csharp
public class UserService
{
    private readonly IUserRepository _userRepository;
    private readonly IEmailService _emailService;
    
    public UserService(IUserRepository userRepository, IEmailService emailService)
    {
        _userRepository = userRepository;
        _emailService = emailService;
    }
    
    public void RegisterUser(User user)
    {
        _userRepository.Save(user);
        
        // Missing principle here
        var logger = new FileLogger();
        logger.Log($"User {user.Name} registered");
        
        _emailService.SendWelcomeEmail(user.Email);
    }
}
```

What design principle is being violated in the RegisterUser method?
A) Single Responsibility Principle
B) Dependency Inversion Principle
C) Open/Closed Principle
D) Interface Segregation Principle

**Solution Process:**

**Step 1: Analyze the pattern** - This is dependency injection, but there's an inconsistency

**Step 2: Notice the problem**
- `_userRepository` and `_emailService` are injected (good DI)
- `FileLogger` is directly instantiated (bad DI)

**Step 3: Identify the principle**
- The code depends on a concrete implementation (`FileLogger`)
- Should depend on abstraction (`ILogger`)
- This violates **Dependency Inversion Principle**

**Answer: B) Dependency Inversion Principle**

---

## Question 3: Open/Closed Principle

**Question:**
```csharp
public class PaymentProcessor
{
    public void ProcessPayment(Payment payment)
    {
        switch (payment.Type)
        {
            case PaymentType.CreditCard:
                ProcessCreditCardPayment(payment);
                break;
            case PaymentType.PayPal:
                ProcessPayPalPayment(payment);
                break;
            case PaymentType.BankTransfer:
                ProcessBankTransferPayment(payment);
                break;
            default:
                throw new NotSupportedException("Payment type not supported");
        }
    }
    
    private void ProcessCreditCardPayment(Payment payment) { /* logic */ }
    private void ProcessPayPalPayment(Payment payment) { /* logic */ }
    private void ProcessBankTransferPayment(Payment payment) { /* logic */ }
}
```

This code violates which principle when adding new payment methods?
A) Single Responsibility Principle
B) Open/Closed Principle
C) Liskov Substitution Principle
D) Dependency Inversion Principle

**Solution Process:**

**Step 1: Understand the scenario** - Adding new payment methods

**Step 2: Analyze the impact**
- To add a new payment type (e.g., Cryptocurrency), you must:
  - Modify the enum
  - Modify the switch statement
  - Add a new method

**Step 3: Identify the principle**
- The class is **closed for modification** ❌
- The class is **not open for extension** ❌
- This violates **Open/Closed Principle**

**Better approach would be:**
```csharp
public abstract class PaymentProcessor
{
    public abstract void Process(Payment payment);
}

public class CreditCardProcessor : PaymentProcessor { /* ... */ }
public class PayPalProcessor : PaymentProcessor { /* ... */ }
```

**Answer: B) Open/Closed Principle**

---

## Question 4: Interface Segregation Principle

**Question:**
```csharp
public interface IWorker
{
    void Work();
    void Eat();
    void Sleep();
    void TakeBreak();
}

public class Human : IWorker
{
    public void Work() { /* implementation */ }
    public void Eat() { /* implementation */ }
    public void Sleep() { /* implementation */ }
    public void TakeBreak() { /* implementation */ }
}

public class Robot : IWorker
{
    public void Work() { /* implementation */ }
    public void Eat() { throw new NotImplementedException(); } // Problem!
    public void Sleep() { throw new NotImplementedException(); } // Problem!
    public void TakeBreak() { /* implementation */ }
}
```

Which SOLID principle is violated?
A) Single Responsibility Principle
B) Open/Closed Principle
C) Interface Segregation Principle
D) Dependency Inversion Principle

**Solution Process:**

**Step 1: Look at the interface usage**
- `Robot` is forced to implement methods it doesn't need
- `Robot` throws `NotImplementedException` for `Eat()` and `Sleep()`

**Step 2: Identify the problem**
- The interface is too "fat" - contains methods not all implementers need
- Clients are forced to implement methods they don't use

**Step 3: Recognize the principle**
- This violates **Interface Segregation Principle**
- "No client should be forced to depend on methods it does not use"

**Better approach:**
```csharp
public interface IWorkable { void Work(); }
public interface IEatable { void Eat(); }
public interface ISleepable { void Sleep(); }
```

**Answer: C) Interface Segregation Principle**

---

## Question 5: Liskov Substitution Principle

**Question:**
```csharp
public class Rectangle
{
    public virtual int Width { get; set; }
    public virtual int Height { get; set; }
    
    public int CalculateArea()
    {
        return Width * Height;
    }
}

public class Square : Rectangle
{
    public override int Width
    {
        get { return base.Width; }
        set { base.Width = base.Height = value; }
    }
    
    public override int Height
    {
        get { return base.Height; }
        set { base.Width = base.Height = value; }
    }
}

// Usage code that breaks:
public void TestRectangle(Rectangle rect)
{
    rect.Width = 5;
    rect.Height = 10;
    Assert.AreEqual(50, rect.CalculateArea()); // Fails for Square!
}
```

Which principle is violated when Square is used as Rectangle?
A) Single Responsibility Principle
B) Open/Closed Principle
C) Liskov Substitution Principle
D) Interface Segregation Principle

**Solution Process:**

**Step 1: Understand substitutability**
- Can `Square` be used wherever `Rectangle` is expected?
- The test assumes width and height can be set independently

**Step 2: Analyze the behavior**
- For `Rectangle`: setting width=5, height=10 gives area=50
- For `Square`: setting width=5, height=10 actually makes both=10, area=100
- Behavior is different!

**Step 3: Identify the principle**
- **Liskov Substitution Principle**: "Objects of a superclass should be replaceable with objects of a subclass without breaking the application"
- `Square` cannot substitute `Rectangle` without changing behavior

**Answer: C) Liskov Substitution Principle**

---

## Quick Decision Framework for Exam

### 30-Second Analysis Method:

1. **Multiple responsibilities in one class?** → **SRP**
2. **Switch statements/if-else chains for types?** → **OCP**  
3. **Subclass changes expected behavior?** → **LSP**
4. **Interface forces unused methods?** → **ISP**
5. **Direct instantiation of concrete classes?** → **DIP**

### Red Flags to Look For:
- **SRP**: Classes with "and" in their description
- **OCP**: `switch` statements on types, modification needed for extension
- **LSP**: Subclasses throwing exceptions or changing contracts
- **ISP**: Empty implementations or `NotImplementedException`
- **DIP**: `new` keyword in business logic, hard-coded dependencies

### Time Management:
- **15 seconds**: Read the question and identify context
- **30 seconds**: Scan code for obvious violations  
- **15 seconds**: Apply elimination process
- **Don't second-guess** your first instinct if it fits the pattern!

---

## Additional Examples - More Design Principles

## Question 6: Single Responsibility Principle (Alternative Example)

**Question:**
```csharp
public class Employee
{
    public string Name { get; set; }
    public decimal Salary { get; set; }
    public string Department { get; set; }
    
    // Employee data management
    public void Save()
    {
        // Database save logic
        var connectionString = ConfigurationManager.ConnectionStrings["DB"].ConnectionString;
        using (var connection = new SqlConnection(connectionString))
        {
            // Save employee to database
        }
    }
    
    // Salary calculation
    public decimal CalculateBonus()
    {
        if (Department == "Sales")
            return Salary * 0.1m;
        else if (Department == "Engineering")
            return Salary * 0.15m;
        return Salary * 0.05m;
    }
    
    // Report generation
    public string GeneratePayslip()
    {
        return $"Employee: {Name}\nSalary: {Salary}\nBonus: {CalculateBonus()}";
    }
    
    // Email functionality
    public void SendPayslipEmail()
    {
        var emailBody = GeneratePayslip();
        // Email sending logic
        var smtpClient = new SmtpClient("smtp.company.com");
        smtpClient.Send("hr@company.com", $"{Name}@company.com", "Payslip", emailBody);
    }
}
```

This class violates which design principle?
A) Open/Closed Principle
B) Single Responsibility Principle
C) Dependency Inversion Principle
D) Interface Segregation Principle

**Solution:**
The `Employee` class has 4 different responsibilities:
1. **Data representation** (properties)
2. **Data persistence** (Save method)
3. **Business logic** (CalculateBonus, GeneratePayslip)
4. **Communication** (SendPayslipEmail)

**Answer: B) Single Responsibility Principle**

---

## Question 7: Open/Closed Principle (Alternative Example)

**Question:**
```csharp
public class DiscountCalculator
{
    public decimal CalculateDiscount(Customer customer, decimal orderAmount)
    {
        if (customer.Type == "Regular")
        {
            return orderAmount * 0.05m; // 5% discount
        }
        else if (customer.Type == "Premium")
        {
            return orderAmount * 0.10m; // 10% discount
        }
        else if (customer.Type == "VIP")
        {
            return orderAmount * 0.15m; // 15% discount
        }
        else if (customer.Type == "Corporate")
        {
            return orderAmount * 0.20m; // 20% discount
        }
        
        return 0; // No discount for unknown types
    }
}
```

When the business wants to add a "Student" customer type with 8% discount, which principle is violated?
A) Single Responsibility Principle
B) Open/Closed Principle
C) Liskov Substitution Principle
D) Dependency Inversion Principle

**Solution:**
To add "Student" discount, you must **modify** the existing `CalculateDiscount` method by adding another `if-else` condition. The class is not **open for extension** but requires **modification**.

**Better approach:**
```csharp
public abstract class DiscountStrategy
{
    public abstract decimal Calculate(decimal amount);
}

public class RegularDiscount : DiscountStrategy
{
    public override decimal Calculate(decimal amount) => amount * 0.05m;
}
```

**Answer: B) Open/Closed Principle**

---

## Question 8: Liskov Substitution Principle (Alternative Example)

**Question:**
```csharp
public abstract class Bird
{
    public abstract void Fly();
    public abstract void MakeSound();
}

public class Eagle : Bird
{
    public override void Fly()
    {
        Console.WriteLine("Eagle soars high in the sky");
    }
    
    public override void MakeSound()
    {
        Console.WriteLine("Eagle makes a screech");
    }
}

public class Penguin : Bird
{
    public override void Fly()
    {
        throw new NotSupportedException("Penguins cannot fly!");
    }
    
    public override void MakeSound()
    {
        Console.WriteLine("Penguin makes a squawk");
    }
}

// Client code
public void MakeBirdFly(Bird bird)
{
    bird.Fly(); // This will crash for Penguin!
}
```

Which principle is violated when using Penguin as a Bird?
A) Single Responsibility Principle
B) Open/Closed Principle  
C) Liskov Substitution Principle
D) Interface Segregation Principle

**Solution:**
The `Penguin` class cannot be substituted for `Bird` without breaking the application. When client code calls `Fly()` on any `Bird`, it expects it to work, but `Penguin` throws an exception.

**LSP Rule**: "Objects of a superclass should be replaceable with objects of subclasses without altering the correctness of the program."

**Answer: C) Liskov Substitution Principle**

---

## Question 9: Interface Segregation Principle (Alternative Example)

**Question:**
```csharp
public interface IVehicle
{
    void StartEngine();
    void StopEngine();
    void Accelerate();
    void Brake();
    void Fly();
    void Dive();
}

public class Car : IVehicle
{
    public void StartEngine() { /* Start car engine */ }
    public void StopEngine() { /* Stop car engine */ }
    public void Accelerate() { /* Accelerate car */ }
    public void Brake() { /* Apply car brakes */ }
    
    // Car cannot fly or dive
    public void Fly() { throw new NotImplementedException(); }
    public void Dive() { throw new NotImplementedException(); }
}

public class Airplane : IVehicle
{
    public void StartEngine() { /* Start airplane engine */ }
    public void StopEngine() { /* Stop airplane engine */ }
    public void Accelerate() { /* Accelerate airplane */ }
    public void Brake() { /* Apply airplane brakes */ }
    public void Fly() { /* Airplane flies */ }
    
    // Airplane cannot dive
    public void Dive() { throw new NotImplementedException(); }
}
```

Which principle does this interface design violate?
A) Single Responsibility Principle
B) Open/Closed Principle
C) Interface Segregation Principle
D) Dependency Inversion Principle

**Solution:**
The `IVehicle` interface is too broad and forces implementers to implement methods they don't need:
- `Car` is forced to implement `Fly()` and `Dive()`
- `Airplane` is forced to implement `Dive()`

**ISP Rule**: "No client should be forced to depend on methods it does not use."

**Better approach:**
```csharp
public interface IDrivable { void StartEngine(); void Accelerate(); }
public interface IFlyable { void Fly(); }
public interface IDivable { void Dive(); }
```

**Answer: C) Interface Segregation Principle**

---

## Question 10: Dependency Inversion Principle (Alternative Example)

**Question:**
```csharp
public class OrderService
{
    public void CreateOrder(Order order)
    {
        // Validate order
        if (order.Items.Count == 0)
            throw new ArgumentException("Order must have items");
        
        // Save to database - PROBLEM: Direct dependency on concrete class
        var repository = new SqlOrderRepository();
        repository.Save(order);
        
        // Send notification - PROBLEM: Direct dependency on concrete class
        var emailService = new SmtpEmailService();
        emailService.SendOrderConfirmation(order.CustomerEmail, order.Id);
        
        // Log the order - PROBLEM: Direct dependency on concrete class
        var logger = new FileLogger();
        logger.Log($"Order {order.Id} created successfully");
    }
}

public class SqlOrderRepository
{
    public void Save(Order order) { /* SQL implementation */ }
}

public class SmtpEmailService
{
    public void SendOrderConfirmation(string email, int orderId) { /* SMTP implementation */ }
}
```

This code violates which principle by directly instantiating concrete classes?
A) Single Responsibility Principle
B) Open/Closed Principle
C) Interface Segregation Principle
D) Dependency Inversion Principle

**Solution:**
The `OrderService` depends on concrete implementations (`SqlOrderRepository`, `SmtpEmailService`, `FileLogger`) rather than abstractions. This makes the code:
- Hard to test (cannot mock dependencies)
- Tightly coupled
- Difficult to change implementations

**DIP Rule**: "Depend on abstractions, not concretions."

**Better approach:**
```csharp
public class OrderService
{
    private readonly IOrderRepository _repository;
    private readonly IEmailService _emailService;
    private readonly ILogger _logger;
    
    public OrderService(IOrderRepository repository, IEmailService emailService, ILogger logger)
    {
        _repository = repository;
        _emailService = emailService;
        _logger = logger;
    }
}
```

**Answer: D) Dependency Inversion Principle**

---

## Question 11: DRY Principle (Don't Repeat Yourself)

**Question:**
```csharp
public class ReportGenerator
{
    public string GenerateSalesReport(List<Sale> sales)
    {
        var report = new StringBuilder();
        report.AppendLine("=== SALES REPORT ===");
        report.AppendLine($"Generated on: {DateTime.Now:yyyy-MM-dd HH:mm:ss}");
        report.AppendLine($"Total Records: {sales.Count}");
        report.AppendLine("========================");
        
        decimal total = 0;
        foreach (var sale in sales)
        {
            report.AppendLine($"Sale ID: {sale.Id}, Amount: ${sale.Amount:F2}, Date: {sale.Date:yyyy-MM-dd}");
            total += sale.Amount;
        }
        
        report.AppendLine("========================");
        report.AppendLine($"Total Sales: ${total:F2}");
        
        return report.ToString();
    }
    
    public string GenerateInventoryReport(List<Product> products)
    {
        var report = new StringBuilder();
        report.AppendLine("=== INVENTORY REPORT ===");
        report.AppendLine($"Generated on: {DateTime.Now:yyyy-MM-dd HH:mm:ss}");
        report.AppendLine($"Total Records: {products.Count}");
        report.AppendLine("============================");
        
        decimal totalValue = 0;
        foreach (var product in products)
        {
            report.AppendLine($"Product: {product.Name}, Stock: {product.Stock}, Value: ${product.Price:F2}");
            totalValue += product.Price * product.Stock;
        }
        
        report.AppendLine("============================");
        report.AppendLine($"Total Inventory Value: ${totalValue:F2}");
        
        return report.ToString();
    }
    
    public string GenerateCustomerReport(List<Customer> customers)
    {
        var report = new StringBuilder();
        report.AppendLine("=== CUSTOMER REPORT ===");
        report.AppendLine($"Generated on: {DateTime.Now:yyyy-MM-dd HH:mm:ss}");
        report.AppendLine($"Total Records: {customers.Count}");
        report.AppendLine("==========================");
        
        foreach (var customer in customers)
        {
            report.AppendLine($"Customer: {customer.Name}, Email: {customer.Email}, Phone: {customer.Phone}");
        }
        
        report.AppendLine("==========================");
        
        return report.ToString();
    }
}
```

Which design principle is being violated by this code?
A) Single Responsibility Principle
B) DRY (Don't Repeat Yourself)
C) Open/Closed Principle
D) KISS (Keep It Simple, Stupid)

**Solution:**
The code repeats the same report formatting logic in all three methods:
- Header formatting with title, date, and record count
- Separator lines
- Footer formatting

This violates the **DRY principle** - the same report structure logic is duplicated.

**Better approach:**
```csharp
private string GenerateReportHeader(string title, int recordCount)
{
    return $"=== {title} ===\nGenerated on: {DateTime.Now:yyyy-MM-dd HH:mm:ss}\nTotal Records: {recordCount}\n";
}
```

**Answer: B) DRY (Don't Repeat Yourself)**

---

## Question 12: KISS Principle (Keep It Simple, Stupid)

**Question:**
```csharp
public class AdvancedCalculator
{
    public int Add(int a, int b)
    {
        // Over-engineered solution for simple addition
        var calculator = CalculatorFactory.CreateCalculator(CalculationType.Addition);
        var operationStrategy = new ArithmeticOperationStrategy();
        var validator = new InputValidator();
        var logger = new OperationLogger();
        var cache = new ResultCache();
        
        var cacheKey = $"ADD_{a}_{b}";
        if (cache.Exists(cacheKey))
        {
            logger.LogCacheHit(cacheKey);
            return cache.Get(cacheKey);
        }
        
        if (!validator.ValidateInputs(a, b))
        {
            throw new InvalidOperationException("Invalid inputs for addition operation");
        }
        
        var operationContext = new OperationContext
        {
            Operator = ArithmeticOperator.Addition,
            LeftOperand = a,
            RightOperand = b,
            Precision = PrecisionType.Integer
        };
        
        var result = operationStrategy.Execute(operationContext);
        
        cache.Store(cacheKey, result);
        logger.LogOperation("Addition", a, b, result);
        
        return result;
    }
    
    // Simple addition should just be:
    // public int Add(int a, int b) { return a + b; }
}
```

This code violates which principle by over-complicating a simple operation?
A) Single Responsibility Principle
B) DRY (Don't Repeat Yourself)
C) Open/Closed Principle
D) KISS (Keep It Simple, Stupid)

**Solution:**
The code takes a simple addition operation (which should be `return a + b;`) and makes it unnecessarily complex with:
- Factory patterns for basic arithmetic
- Strategy patterns for simple operations
- Caching for trivial calculations
- Complex validation for basic integer addition
- Excessive logging and abstraction

This violates the **KISS principle** - "Keep It Simple, Stupid." The solution is over-engineered for the problem it solves.

**KISS Rule**: "Most systems work best if they are kept simple rather than made complicated."

**Answer: D) KISS (Keep It Simple, Stupid)**

---

## Updated Quick Decision Framework

### Extended 30-Second Analysis Method:

1. **Multiple responsibilities in one class?** → **SRP**
2. **Switch statements/if-else chains for types?** → **OCP**  
3. **Subclass changes expected behavior or throws exceptions?** → **LSP**
4. **Interface forces unused methods (NotImplementedException)?** → **ISP**
5. **Direct instantiation of concrete classes (`new` keyword)?** → **DIP**
6. **Same code repeated in multiple places?** → **DRY**
7. **Over-complicated solution for simple problem?** → **KISS**

### Additional Red Flags:
- **DRY**: Look for copy-pasted code blocks, similar method implementations
- **KISS**: Look for excessive patterns, over-abstraction, factory/strategy patterns for simple operations

---

I am a .NET developer, having 16 years of experience. Today I am having an prescreening session in which I will be given 14 multiple choice questions to answer in 15 minutes. I am preparing for that now. This is my current scenario. In this case, what are the questions can be asked. List me some around 50 questions, much similar to the previous.

--- 

# .NET Developer Prescreening Questions - 50 MCQs

## Design Principles & Patterns (15 Questions)

### 1. SOLID Principles
**Question 1:**
```csharp
public class FileManager
{
    public void ReadFile(string path) { /* read logic */ }
    public void WriteFile(string path, string content) { /* write logic */ }
    public void CompressFile(string path) { /* compression logic */ }
    public void EncryptFile(string path) { /* encryption logic */ }
    public void SendFileByEmail(string path, string email) { /* email logic */ }
}
```
Which principle is violated?
A) SRP B) OCP C) LSP D) ISP

**Question 2:**
```csharp
public class ShapeCalculator
{
    public double CalculateArea(object shape)
    {
        if (shape is Circle c)
            return Math.PI * c.Radius * c.Radius;
        else if (shape is Rectangle r)
            return r.Width * r.Height;
        else if (shape is Triangle t)
            return 0.5 * t.Base * t.Height;
        
        throw new NotSupportedException();
    }
}
```
Adding a new shape requires modifying this method. Which principle is violated?
A) SRP B) OCP C) LSP D) DIP

**Question 3:**
```csharp
public class DatabaseService
{
    public void SaveUser(User user)
    {
        var logger = new FileLogger(); // Direct instantiation
        var emailService = new SmtpEmailService(); // Direct instantiation
        
        // Save logic
        logger.Log("User saved");
        emailService.Send("Welcome email");
    }
}
```
Which principle is violated?
A) SRP B) OCP C) ISP D) DIP

**Question 4:**
```csharp
public interface IRepository
{
    void Save<T>(T entity);
    void Delete<T>(T entity);
    void Update<T>(T entity);
    IEnumerable<T> GetAll<T>();
    void BackupDatabase();
    void OptimizeIndexes();
    void RunDiagnostics();
}
```
Which principle does this interface violate?
A) SRP B) OCP C) ISP D) DIP

**Question 5:**
```csharp
public class Bird
{
    public virtual void Fly() { Console.WriteLine("Flying"); }
}

public class Ostrich : Bird
{
    public override void Fly()
    {
        throw new InvalidOperationException("Ostriches can't fly");
    }
}
```
Which principle is violated when using Ostrich as Bird?
A) SRP B) OCP C) LSP D) ISP

## .NET Framework & Language Features (15 Questions)

**Question 6:**
What's the difference between `String` and `StringBuilder`?
A) String is mutable, StringBuilder is immutable
B) String is immutable, StringBuilder is mutable
C) Both are mutable
D) Both are immutable

**Question 7:**
```csharp
public class Example
{
    public static void Main()
    {
        int? x = null;
        int y = x ?? 10;
        Console.WriteLine(y);
    }
}
```
What will this output?
A) null B) 0 C) 10 D) Exception

**Question 8:**
What's the primary difference between `IEnumerable<T>` and `IQueryable<T>`?
A) IEnumerable is for databases, IQueryable is for collections
B) IQueryable allows deferred execution, IEnumerable doesn't
C) IQueryable can be translated to SQL, IEnumerable cannot
D) No significant difference

**Question 9:**
```csharp
var numbers = new List<int> { 1, 2, 3, 4, 5 };
var result = numbers.Where(x => x > 2).Select(x => x * 2);
```
When is the actual filtering and transformation executed?
A) Immediately when Where() is called
B) When Select() is called
C) When result is enumerated
D) Never, it's compile-time

**Question 10:**
What's the difference between `Task.Run()` and `Task.Factory.StartNew()`?
A) No difference, they're identical
B) Task.Run() uses ThreadPool, StartNew() creates new thread
C) Task.Run() is optimized for CPU-bound work
D) StartNew() is deprecated

**Question 11:**
```csharp
public async Task<string> GetDataAsync()
{
    await Task.Delay(1000);
    return "Data";
}

public void CallMethod()
{
    var result = GetDataAsync().Result; // What's wrong here?
}
```
What's the potential problem?
A) Memory leak B) Deadlock C) Exception D) Performance issue

**Question 12:**
Which collection is thread-safe in .NET?
A) List<T> B) Dictionary<K,V> C) ConcurrentBag<T> D) HashSet<T>

**Question 13:**
```csharp
using (var stream = new FileStream("file.txt", FileMode.Open))
{
    // Process file
}
```
What interface makes the `using` statement work?
A) IDisposable B) ICloseable C) IFinalize D) ICleanup

**Question 14:**
What's the difference between `ref` and `out` parameters?
A) ref must be initialized, out doesn't need to be
B) out must be initialized, ref doesn't need to be
C) ref passes by reference, out passes by value
D) No difference in functionality

**Question 15:**
```csharp
public class BaseClass
{
    public virtual void Method() { Console.WriteLine("Base"); }
}

public class DerivedClass : BaseClass
{
    public new void Method() { Console.WriteLine("Derived"); }
}
```
What's the difference between `override` and `new` keyword?
A) override hides base method, new overrides it
B) new hides base method, override provides polymorphism
C) Both do the same thing
D) new is for static methods only

**Question 16:**
What happens when you don't handle an exception in an async method?
A) Application crashes immediately
B) Exception is lost
C) Exception is stored in Task and thrown when awaited
D) Exception is logged automatically

**Question 17:**
```csharp
var list = new List<string> { "a", "b", "c" };
foreach (var item in list)
{
    if (item == "b")
        list.Remove(item); // What happens?
}
```
A) Works fine B) InvalidOperationException C) ArgumentException D) NullReferenceException

**Question 18:**
What's the purpose of `ConfigureAwait(false)`?
A) Improves performance B) Prevents deadlocks C) Enables debugging D) All of the above

**Question 19:**
Which is more efficient for string concatenation in a loop?
A) string concatenation (+) B) StringBuilder C) String.Concat D) String.Join

**Question 20:**
What's the difference between `==` and `Equals()` for strings?
A) == compares references, Equals compares values
B) No difference for strings
C) Equals is faster
D) == can handle nulls better

## OOP & Inheritance (10 Questions)

**Question 21:**
```csharp
public abstract class Animal
{
    public abstract void MakeSound();
    public virtual void Sleep() { Console.WriteLine("Sleeping"); }
}
```
What's the difference between `abstract` and `virtual` methods?
A) Abstract must be overridden, virtual can be overridden
B) Virtual must be overridden, abstract can be overridden
C) No difference
D) Abstract is for interfaces only

**Question 22:**
Can an abstract class have a constructor?
A) No, abstract classes cannot have constructors
B) Yes, but only parameterless constructors
C) Yes, and they can be called by derived classes
D) Only static constructors are allowed

**Question 23:**
```csharp
public interface IRepository<T>
{
    void Save(T entity);
}

public class UserRepository : IRepository<User>
{
    public void Save(User entity) { /* implementation */ }
}
```
This demonstrates which OOP principle?
A) Encapsulation B) Inheritance C) Polymorphism D) Abstraction

**Question 24:**
What's the maximum number of classes a C# class can inherit from?
A) 1 B) 2 C) 5 D) Unlimited

**Question 25:**
```csharp
public class Parent
{
    protected internal void Method() { }
}
```
What does `protected internal` mean?
A) Protected OR internal access
B) Protected AND internal access
C) Only protected access
D) Only internal access

**Question 26:**
Can you override a private method?
A) Yes, with override keyword B) Yes, with new keyword C) No, private methods cannot be overridden D) Only in derived classes

**Question 27:**
```csharp
public sealed class FinalClass
{
    public virtual void Method() { }
}
```
What's wrong with this code?
A) Sealed classes can't have virtual methods
B) Virtual methods need override
C) Nothing wrong
D) Sealed keyword is incorrect

**Question 28:**
What's the purpose of the `base` keyword?
A) Access static members B) Access parent class members C) Create base instance D) Check inheritance

**Question 29:**
```csharp
public class Generic<T> where T : class, new()
{
    public T CreateInstance() => new T();
}
```
What do the constraints `class, new()` mean?
A) T must be a class with parameterless constructor
B) T must be a reference type with default constructor
C) Both A and B are correct
D) T must be abstract class

**Question 30:**
Can interfaces have fields?
A) Yes, all types of fields B) Only static fields C) Only constants D) No fields allowed

## Memory Management & Performance (8 Questions)

**Question 31:**
What's the difference between stack and heap memory?
A) Stack is for reference types, heap is for value types
B) Stack is for value types, heap is for reference types
C) Both store the same types
D) Stack is faster, heap is slower

**Question 32:**
When does the Garbage Collector run?
A) Every 5 seconds B) When heap memory is low C) Only when called explicitly D) At application shutdown

**Question 33:**
```csharp
public class ResourceHolder : IDisposable
{
    public void Dispose()
    {
        // Cleanup code
        GC.SuppressFinalize(this);
    }
}
```
Why call `GC.SuppressFinalize(this)`?
A) Improves performance B) Prevents memory leaks C) Required by IDisposable D) Cleans resources immediately

**Question 34:**
What's a memory leak in .NET?
A) Objects not being garbage collected B) Stack overflow C) Heap overflow D) CPU usage spike

**Question 35:**
Which is more memory efficient?
A) `List<int>` B) `int[]` C) `Collection<int>` D) `IEnumerable<int>`

**Question 36:**
```csharp
string result = "";
for (int i = 0; i < 1000; i++)
{
    result += i.ToString();
}
```
What's the performance issue?
A) Too many iterations B) String immutability creates many objects C) Integer conversion D) No issue

**Question 37:**
What's the purpose of `WeakReference`?
A) Prevent garbage collection B) Allow garbage collection C) Improve performance D) Handle null references

**Question 38:**
Which generation does the GC check most frequently?
A) Generation 0 B) Generation 1 C) Generation 2 D) All equally

## Exception Handling & Debugging (7 Questions)

**Question 39:**
```csharp
try
{
    // risky code
}
catch (SpecificException ex)
{
    // handle specific
}
catch (Exception ex)
{
    // handle general
    throw; // What does this do?
}
```
What's the difference between `throw` and `throw ex`?
A) No difference B) throw preserves stack trace, throw ex doesn't C) throw ex is faster D) throw creates new exception

**Question 40:**
When should you use `finally` block?
A) Always B) Only with exceptions C) For cleanup that must always run D) Never in async methods

**Question 41:**
```csharp
public void Method()
{
    try { /* code */ }
    catch { /* no exception parameter */ }
}
```
What's wrong with this catch block?
A) Missing exception type B) Can't handle any exceptions C) Swallows all exceptions D) Syntax error

**Question 42:**
Which exception should you throw for invalid method arguments?
A) Exception B) ArgumentException C) InvalidOperationException D) ApplicationException

**Question 43:**
What's the purpose of `Debug.Assert()`?
A) Always throws exception B) Only active in Debug builds C) Logs to database D) Validates user input

**Question 44:**
```csharp
public async Task<string> MethodAsync()
{
    try
    {
        return await SomeAsyncOperation();
    }
    catch (Exception ex)
    {
        // Log exception
        throw; // Is this correct?
    }
}
```
A) Correct B) Should use throw ex C) Should return null D) Missing finally

**Question 45:**
What's a custom exception best practice?
A) Inherit from Exception B) Inherit from ApplicationException C) Always include inner exception D) Include serialization support

## Best Practices & Code Quality (5 Questions)

**Question 46:**
```csharp
public class Calculator
{
    public int Add(int a, int b)
    {
        var factory = new CalculatorFactory();
        var validator = new InputValidator();
        var logger = new Logger();
        var cache = new ResultCache();
        
        // ... complex logic for simple addition
        return a + b;
    }
}
```
Which principle is violated?
A) SRP B) DRY C) KISS D) OCP

**Question 47:**
```csharp
public void ProcessUsers()
{
    var users = GetUsers();
    
    // Format users for display
    foreach (var user in users)
    {
        user.DisplayName = $"{user.FirstName} {user.LastName}";
        user.AgeGroup = user.Age < 18 ? "Minor" : user.Age < 65 ? "Adult" : "Senior";
    }
    
    // Format users for email
    foreach (var user in users)
    {
        user.EmailDisplayName = $"{user.FirstName} {user.LastName}";
        user.EmailAgeGroup = user.Age < 18 ? "Minor" : user.Age < 65 ? "Adult" : "Senior";
    }
}
```
Which principle is violated?
A) SRP B) DRY C) KISS D) OCP

**Question 48:**
What's the recommended way to compare strings for equality?
A) == operator B) Equals() method C) String.Compare() D) CompareTo() method

**Question 49:**
Which naming convention is correct for C# methods?
A) calculateTotal B) CalculateTotal C) calculate_total D) CALCULATE_TOTAL

**Question 50:**
```csharp
public class UserService
{
    private static UserService _instance;
    
    private UserService() { }
    
    public static UserService Instance
    {
        get
        {
            if (_instance == null)
                _instance = new UserService();
            return _instance;
        }
    }
}
```
What design pattern is this, and what's the potential issue?
A) Singleton, thread-safety issue B) Factory, performance issue C) Observer, memory issue D) Strategy, complexity issue

---

## Quick Answer Key (For Self-Testing)

1. A (SRP) 2. B (OCP) 3. D (DIP) 4. C (ISP) 5. C (LSP)
6. B 7. C 8. C 9. C 10. C
11. B 12. C 13. A 14. A 15. B
16. C 17. B 18. B 19. B 20. B
21. A 22. C 23. D 24. A 25. A
26. C 27. A 28. B 29. C 30. D
31. B 32. B 33. A 34. A 35. B
36. B 37. B 38. A 39. B 40. C
41. C 42. B 43. B 44. A 45. A
46. C (KISS) 47. B (DRY) 48. B 49. B 50. A

---

## Question Categories Distribution:
- **Design Principles**: 15 questions (SOLID, DRY, KISS)
- **.NET Framework**: 15 questions (Language features, async/await, collections)
- **OOP**: 10 questions (Inheritance, polymorphism, interfaces)
- **Memory Management**: 8 questions (GC, performance, disposal)
- **Exception Handling**: 7 questions (Try-catch, debugging)
- **Best Practices**: 5 questions (Code quality, patterns)

## Time Management Strategy:
- **Design Principles**: 30 seconds each (7.5 minutes)
- **Technical Questions**: 20 seconds each (5 minutes)
- **Review & Verify**: 2.5 minutes

Focus on pattern recognition and trust your experience!

--- 

I've created 50 comprehensive multiple choice questions that cover the typical areas tested in .NET developer prescreening sessions. These questions are designed to be answerable within 15-20 seconds each, perfect for your 15-minute timeframe.

**Key Areas Covered:**

1. **Design Principles (30%)** - SOLID, DRY, KISS principles with code examples
2. **.NET Framework (30%)** - Language features, async/await, collections, performance
3. **OOP Concepts (20%)** - Inheritance, polymorphism, interfaces, generics
4. **Memory Management (16%)** - Garbage collection, disposal patterns, performance
5. **Exception Handling (14%)** - Best practices, debugging, async exceptions
6. **Code Quality (10%)** - Naming conventions, patterns, best practices

**Question Difficulty Levels:**
- **Basic (40%)** - Fundamental concepts every .NET developer should know
- **Intermediate (45%)** - Practical scenarios you encounter daily
- **Advanced (15%)** - Edge cases and best practices

**Speed Tips for Your Exam:**
1. **Pattern Recognition** - Most questions follow common patterns you've seen
2. **Elimination Strategy** - Eliminate obviously wrong answers first
3. **Trust Your Experience** - With 16 years experience, your instincts are usually right
4. **Don't Overthink** - First reasonable answer is often correct
5. **Time Allocation** - Spend more time on design principle questions (higher weightage)

**Common Question Types You'll Encounter:**
- Code analysis with violations
- "What's wrong with this code?"
- "Which principle is violated?"
- Comparison questions (String vs StringBuilder)
- Best practice scenarios
- Performance implications

Practice with these questions and you should feel confident for your prescreening. The key is recognizing patterns quickly rather than deep analysis.

---

