# Facade Design Pattern

You're making a smart move—**Facade Design Pattern** is a favorite in interviews because it demonstrates your ability to simplify complexity. Let’s break it down into digestible parts so you can confidently explain it and even code it on the fly.

---

## What Is the Facade Design Pattern?

The **Facade Design Pattern** is a **structural pattern** that provides a **simplified interface** to a complex subsystem. Instead of exposing all the internal components and their interactions, the facade offers a single, unified entry point.

Think of it like a **hotel concierge**: you don’t need to talk to the chef, the housekeeper, and the manager separately—you just talk to the concierge, and they handle everything behind the scenes.

---

## When and Why to Use It

### Use Cases:
- When working with **complex libraries or APIs** that have multiple interdependent classes.
- To **decouple client code** from the internal workings of a subsystem.
- To **improve readability and maintainability** by hiding unnecessary complexity.
- In **legacy systems**, where you want to wrap old code with a cleaner interface.

### When to Use:
- When you want to **simplify client interaction** with a system.
- When you need to **organize startup logic** or **group related operations**.
- When multiple subsystems need to be coordinated in a **predictable sequence**.

---

## Real-World Analogy

Imagine placing an order in an e-commerce app:
- You need to check inventory
- Process payment
- Generate invoice
- Send confirmation email

Instead of calling each service individually, you use a **`OrderFacade`** that handles all of this for you.

---

## C# Example: Pizza Ordering System

### Subsystems

```csharp
public class PizzaService
{
    public void GetVegPizza() => Console.WriteLine("Preparing Veg Pizza");
    public void GetNonVegPizza() => Console.WriteLine("Preparing Non-Veg Pizza");
}

public class BreadService
{
    public void GetGarlicBread() => Console.WriteLine("Preparing Garlic Bread");
    public void GetCheesyBread() => Console.WriteLine("Preparing Cheesy Garlic Bread");
}
```

### Facade Class

```csharp
public class RestaurantFacade
{
    private readonly PizzaService _pizzaService;
    private readonly BreadService _breadService;

    public RestaurantFacade()
    {
        _pizzaService = new PizzaService();
        _breadService = new BreadService();
    }

    public void OrderVegMeal()
    {
        _pizzaService.GetVegPizza();
        _breadService.GetGarlicBread();
    }

    public void OrderNonVegMeal()
    {
        _pizzaService.GetNonVegPizza();
        _breadService.GetCheesyBread();
    }
}
```

### Client Code

```csharp
class Program
{
    static void Main()
    {
        var restaurant = new RestaurantFacade();
        restaurant.OrderVegMeal();
        restaurant.OrderNonVegMeal();
    }
}
```

---

## Benefits

- **Simplifies usage** for clients
- **Reduces coupling** between client and subsystem
- **Improves code readability**
- **Encapsulates complexity**

---

## Interview Tip

When asked about Facade, emphasize:
- It’s about **simplification**, not functionality.
- It doesn’t restrict access to subsystems—it just **organizes** it.
- It’s often used in **startup configuration**, **API wrappers**, and **service orchestration**.

---

## Technical Analogy: **Startup Configuration in ASP.NET Core**

In ASP.NET Core, when your application starts, you need to configure:
- Logging
- Dependency Injection
- Middleware
- Routing
- Authentication

Each of these components has its own configuration logic and dependencies. If you expose all of this directly to the client code (like `Program.cs` or `Startup.cs`), it becomes messy and hard to maintain.

### Enter the Facade

You create a **`StartupFacade`** or **`AppConfigurator`** class that wraps all these configurations into clean, reusable methods like:

```csharp
public class AppConfigurator
{
    public static void ConfigureServices(IServiceCollection services)
    {
        services.AddLogging();
        services.AddAuthentication();
        services.AddControllers();
        // Other service setups
    }

    public static void ConfigureMiddleware(IApplicationBuilder app)
    {
        app.UseRouting();
        app.UseAuthentication();
        app.UseAuthorization();
        app.UseEndpoints(endpoints => {
            endpoints.MapControllers();
        });
    }
}
```

### Client Code (Simplified)

```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        AppConfigurator.ConfigureServices(services);
    }

    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        AppConfigurator.ConfigureMiddleware(app);
    }
}
```

---

## Benefits in This Context

- **Cleaner Startup.cs**: You avoid cluttering your entry point with low-level configuration.
- **Reusability**: You can reuse `AppConfigurator` across multiple projects or microservices.
- **Testability**: Easier to unit test configuration logic in isolation.
- **Decoupling**: Your startup logic is decoupled from the actual implementation of services.

---

## Interview Insight

If asked in an interview, you could say:

> “In ASP.NET Core, I often use the Facade pattern to wrap startup configuration logic. Instead of configuring services and middleware directly in `Startup.cs`, I delegate that to a facade class. This keeps the entry point clean and makes the configuration reusable and testable.”

---

## Facade Pattern in Microservices Architecture

In a microservices ecosystem, services are often **highly specialized**, but they can also be **interdependent** and **complex to orchestrate**. The **Facade Pattern** helps by introducing a **simplified interface** that coordinates multiple services or hides internal complexity.

### Common Use Cases

| Scenario | Facade Role |
|---------|-------------|
| **API Gateway** | Acts as a facade to route requests to appropriate microservices, aggregate responses, and handle cross-cutting concerns like auth, logging, rate limiting. |
| **Service Orchestration** | A facade service can coordinate multiple microservices to fulfill a business process (e.g., placing an order triggers inventory, payment, and shipping services). |
| **Legacy Integration** | Facade wraps legacy systems with a clean interface so modern services can interact without dealing with outdated protocols. |
| **External Dependency Shielding** | Facade microservices abstract away third-party APIs (e.g., payment gateways, mapping services) to reduce coupling and simplify changes. |

### Example

Imagine a **Travel Booking System**:
- Microservices: FlightService, HotelService, PaymentService
- Facade: `BookingFacadeService`

Instead of the client calling each service individually, it calls `BookingFacadeService.BookTrip()`, which internally:
- Reserves flight
- Books hotel
- Processes payment
- Sends confirmation

This keeps the client code clean and shields it from the orchestration logic.

---

## Comparison: Facade vs Adapter vs Proxy

Let’s break it down with purpose, behavior, and analogy:

| Pattern | Purpose | Behavior | Analogy |
|--------|---------|----------|--------|
| **Facade** | Simplifies access to a complex system | Aggregates and delegates calls to subsystems | Universal remote that controls TV, sound system, lights |
| **Adapter** | Makes incompatible interfaces work together | Translates one interface to another | HDMI-to-VGA converter |
| **Proxy** | Controls access to an object | Adds logic before/after access (e.g., caching, auth) | Security guard at a vault |

### Key Differences

- **Facade** is about **simplicity** and **abstraction**.
- **Adapter** is about **compatibility** between mismatched interfaces.
- **Proxy** is about **control**—it may restrict, enhance, or monitor access.

---

## Interview Tip

If asked to compare them, you could say:

> “Facade simplifies a complex system for the client. Adapter bridges incompatible interfaces. Proxy adds a layer of control over access. In microservices, I’ve used Facade to wrap orchestration logic, Adapter to integrate legacy APIs, and Proxy to implement lazy loading and security checks.”

---

One of my technical interview, I want to learn Facade Design Pattern. Explain me Facade Design Pattern with proper examples in C#, use cases and when & how to use that pattern. Use some technical analogy for explaining the concepts. Give more examples in C# and include inline comments for the code. Also explain the followup questions. 

---

I've created a comprehensive guide to the Facade Design Pattern with multiple C# examples and technical analogies. The guide covers:

## **Key Features:**

1. **Clear Technical Analogies**: Hotel concierge, API Gateway, power plug converter
2. **Three Detailed Examples**: 
   - Home Theater (classic example)
   - Banking System (enterprise scenario)
   - File Processing (batch operations)
3. **Extensive Inline Comments**: Every code section is well-documented
4. **Real-world Use Cases**: When to use and when to avoid
5. **Common Interview Questions**: With detailed answers and code examples

## **What Makes This Interview-Ready:**

- **Progressive Complexity**: Starting simple, building to enterprise scenarios
- **Error Handling Patterns**: Shows robust error management in facades
- **Testing Considerations**: Discusses unit testing implications
- **Performance Awareness**: Mentions potential overhead and mitigation
- **Best Practices**: Interface segregation, composition over inheritance

The examples demonstrate how Facade pattern is commonly used in .NET enterprise applications - from simple device coordination to complex banking operations that involve multiple subsystems like authentication, validation, transaction processing, and notifications.

This should give you a solid foundation for discussing Facade pattern in technical interviews, with concrete examples you can reference and explain the trade-offs involved.

--- 

# Facade Design Pattern - Complete Guide

## What is the Facade Pattern?

The Facade Pattern provides a **simplified interface** to a complex subsystem. Think of it like a **hotel concierge** - instead of you having to know which department handles room service, housekeeping, maintenance, and billing, you just call the concierge who coordinates with all the different departments for you.

**Technical Analogy:** Like a **REST API Gateway** that sits in front of multiple microservices. Clients don't need to know about individual service endpoints, authentication protocols, or data transformation - they just call one simple API.

## Key Benefits

- **Simplifies complex systems** by providing a single entry point
- **Reduces coupling** between clients and subsystems
- **Hides complexity** from clients
- **Provides a stable interface** even when subsystems change

## Example 1 - Home Theater System

```csharp
// Complex subsystems that the client shouldn't need to know about
public class DVDPlayer
{
    public void TurnOn()
    {
        Console.WriteLine("DVD Player: Powering on...");
    }
    
    public void PlayMovie(string movie)
    {
        Console.WriteLine($"DVD Player: Playing '{movie}'");
    }
    
    public void TurnOff()
    {
        Console.WriteLine("DVD Player: Powering off...");
    }
}

public class SurroundSound
{
    public void TurnOn()
    {
        Console.WriteLine("Surround Sound: Powering on...");
    }
    
    public void SetVolume(int level)
    {
        Console.WriteLine($"Surround Sound: Volume set to {level}");
    }
    
    public void TurnOff()
    {
        Console.WriteLine("Surround Sound: Powering off...");
    }
}

public class Projector
{
    public void TurnOn()
    {
        Console.WriteLine("Projector: Powering on...");
    }
    
    public void SetInput(string input)
    {
        Console.WriteLine($"Projector: Input set to {input}");
    }
    
    public void TurnOff()
    {
        Console.WriteLine("Projector: Powering off...");
    }
}

public class Lights
{
    public void DimLights(int level)
    {
        Console.WriteLine($"Lights: Dimmed to {level}%");
    }
    
    public void TurnOnLights()
    {
        Console.WriteLine("Lights: Full brightness");
    }
}

// FACADE - This is the simplified interface
public class HomeTheaterFacade
{
    // Facade aggregates all the complex subsystems
    private readonly DVDPlayer _dvdPlayer;
    private readonly SurroundSound _surroundSound;
    private readonly Projector _projector;
    private readonly Lights _lights;
    
    public HomeTheaterFacade(DVDPlayer dvdPlayer, SurroundSound surroundSound, 
                           Projector projector, Lights lights)
    {
        _dvdPlayer = dvdPlayer;
        _surroundSound = surroundSound;
        _projector = projector;
        _lights = lights;
    }
    
    // Simple method that orchestrates complex operations
    public void WatchMovie(string movie)
    {
        Console.WriteLine("Getting ready to watch a movie...");
        
        // Facade handles all the complexity
        _lights.DimLights(10);           // Dim lights to 10%
        _projector.TurnOn();             // Turn on projector
        _projector.SetInput("DVD");      // Set input to DVD
        _surroundSound.TurnOn();         // Turn on sound system
        _surroundSound.SetVolume(7);     // Set volume to 7
        _dvdPlayer.TurnOn();             // Turn on DVD player
        _dvdPlayer.PlayMovie(movie);     // Play the movie
        
        Console.WriteLine("Enjoy your movie!\n");
    }
    
    // Another simple method for cleanup
    public void EndMovie()
    {
        Console.WriteLine("Shutting down movie theater...");
        
        _dvdPlayer.TurnOff();
        _surroundSound.TurnOff();
        _projector.TurnOff();
        _lights.TurnOnLights();
        
        Console.WriteLine("Movie theater is off.\n");
    }
}

// Client code - Notice how simple this is!
public class Client
{
    public static void Main()
    {
        // Create all the complex subsystems
        var dvd = new DVDPlayer();
        var sound = new SurroundSound();
        var projector = new Projector();
        var lights = new Lights();
        
        // Create the facade
        var homeTheater = new HomeTheaterFacade(dvd, sound, projector, lights);
        
        // Client uses simple interface - no need to know about subsystems
        homeTheater.WatchMovie("The Matrix");
        homeTheater.EndMovie();
    }
}
```

---

## Example 2 - Banking System (Real-world Enterprise Scenario)

```csharp
// Complex subsystems in a banking application
public class AccountService
{
    public bool ValidateAccount(string accountNumber)
    {
        Console.WriteLine($"AccountService: Validating account {accountNumber}");
        // Complex validation logic here
        return true;
    }
    
    public decimal GetBalance(string accountNumber)
    {
        Console.WriteLine($"AccountService: Retrieving balance for {accountNumber}");
        // Database call to get balance
        return 1000.50m;
    }
}

public class SecurityService
{
    public bool AuthenticateUser(string userId, string pin)
    {
        Console.WriteLine($"SecurityService: Authenticating user {userId}");
        // Complex authentication logic
        return true;
    }
    
    public bool AuthorizeTransaction(string userId, decimal amount)
    {
        Console.WriteLine($"SecurityService: Authorizing transaction of ${amount}");
        // Authorization logic
        return amount <= 5000; // Simple rule: max $5000 per transaction
    }
}

public class TransactionService
{
    public bool ProcessWithdrawal(string accountNumber, decimal amount)
    {
        Console.WriteLine($"TransactionService: Processing withdrawal of ${amount}");
        // Complex transaction processing
        return true;
    }
    
    public void LogTransaction(string accountNumber, string transactionType, decimal amount)
    {
        Console.WriteLine($"TransactionService: Logging {transactionType} of ${amount}");
        // Audit logging
    }
}

public class NotificationService
{
    public void SendSMS(string phoneNumber, string message)
    {
        Console.WriteLine($"NotificationService: SMS sent to {phoneNumber}: {message}");
    }
    
    public void SendEmail(string email, string subject, string body)
    {
        Console.WriteLine($"NotificationService: Email sent to {email}");
    }
}

// FACADE for Banking Operations
public class BankingFacade
{
    private readonly AccountService _accountService;
    private readonly SecurityService _securityService;
    private readonly TransactionService _transactionService;
    private readonly NotificationService _notificationService;
    
    public BankingFacade()
    {
        // Facade creates and manages all subsystem instances
        _accountService = new AccountService();
        _securityService = new SecurityService();
        _transactionService = new TransactionService();
        _notificationService = new NotificationService();
    }
    
    // Simple interface for complex withdrawal operation
    public bool WithdrawMoney(string userId, string pin, string accountNumber, 
                             decimal amount, string phoneNumber)
    {
        Console.WriteLine($"=== Starting withdrawal process for ${amount} ===");
        
        try
        {
            // Step 1: Authenticate user
            if (!_securityService.AuthenticateUser(userId, pin))
            {
                Console.WriteLine("Authentication failed!");
                return false;
            }
            
            // Step 2: Validate account
            if (!_accountService.ValidateAccount(accountNumber))
            {
                Console.WriteLine("Invalid account!");
                return false;
            }
            
            // Step 3: Check balance
            decimal balance = _accountService.GetBalance(accountNumber);
            if (balance < amount)
            {
                Console.WriteLine("Insufficient funds!");
                return false;
            }
            
            // Step 4: Authorize transaction
            if (!_securityService.AuthorizeTransaction(userId, amount))
            {
                Console.WriteLine("Transaction not authorized!");
                return false;
            }
            
            // Step 5: Process withdrawal
            if (!_transactionService.ProcessWithdrawal(accountNumber, amount))
            {
                Console.WriteLine("Transaction processing failed!");
                return false;
            }
            
            // Step 6: Log transaction
            _transactionService.LogTransaction(accountNumber, "WITHDRAWAL", amount);
            
            // Step 7: Send notification
            _notificationService.SendSMS(phoneNumber, 
                $"Withdrawal of ${amount} successful. New balance: ${balance - amount}");
            
            Console.WriteLine("=== Withdrawal completed successfully! ===\n");
            return true;
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error during withdrawal: {ex.Message}");
            return false;
        }
    }
    
    // Another simple interface for balance inquiry
    public decimal CheckBalance(string userId, string pin, string accountNumber)
    {
        Console.WriteLine("=== Checking account balance ===");
        
        if (!_securityService.AuthenticateUser(userId, pin))
        {
            throw new UnauthorizedAccessException("Authentication failed");
        }
        
        if (!_accountService.ValidateAccount(accountNumber))
        {
            throw new ArgumentException("Invalid account");
        }
        
        decimal balance = _accountService.GetBalance(accountNumber);
        Console.WriteLine($"Current balance: ${balance}\n");
        return balance;
    }
}

// Client code - ATM machine or mobile app
public class ATMClient
{
    public static void Main()
    {
        // Client only needs to know about the facade
        var banking = new BankingFacade();
        
        try
        {
            // Simple method calls hide all the complexity
            decimal balance = banking.CheckBalance("user123", "1234", "ACC001");
            
            bool success = banking.WithdrawMoney("user123", "1234", "ACC001", 
                                               200.00m, "+1234567890");
            
            if (success)
            {
                Console.WriteLine("Please take your cash and receipt.");
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Operation failed: {ex.Message}");
        }
    }
}
```

---

## Example 3 - File Processing System

```csharp
// Complex file processing subsystems
public class FileReader
{
    public string ReadFile(string filePath)
    {
        Console.WriteLine($"FileReader: Reading file from {filePath}");
        // Simulate file reading
        return "Sample file content with data...";
    }
}

public class DataValidator
{
    public bool ValidateData(string data)
    {
        Console.WriteLine("DataValidator: Validating data format and integrity");
        // Complex validation rules
        return !string.IsNullOrEmpty(data) && data.Length > 10;
    }
}

public class DataTransformer
{
    public string TransformData(string rawData)
    {
        Console.WriteLine("DataTransformer: Transforming data format");
        // Complex transformation logic
        return rawData.ToUpper().Replace(" ", "_");
    }
}

public class DatabaseWriter
{
    public bool WriteToDatabase(string transformedData)
    {
        Console.WriteLine("DatabaseWriter: Inserting data into database");
        // Database insertion logic
        Console.WriteLine($"Data inserted: {transformedData.Substring(0, Math.Min(50, transformedData.Length))}...");
        return true;
    }
}

public class AuditLogger
{
    public void LogOperation(string operation, bool success)
    {
        Console.WriteLine($"AuditLogger: {operation} - Status: {(success ? "SUCCESS" : "FAILED")}");
    }
}

// FACADE for File Processing
public class FileProcessingFacade
{
    private readonly FileReader _fileReader;
    private readonly DataValidator _dataValidator;
    private readonly DataTransformer _dataTransformer;
    private readonly DatabaseWriter _databaseWriter;
    private readonly AuditLogger _auditLogger;
    
    public FileProcessingFacade()
    {
        // Initialize all subsystems
        _fileReader = new FileReader();
        _dataValidator = new DataValidator();
        _dataTransformer = new DataTransformer();
        _databaseWriter = new DatabaseWriter();
        _auditLogger = new AuditLogger();
    }
    
    // Single method that orchestrates the entire process
    public bool ProcessFile(string filePath)
    {
        Console.WriteLine($"=== Processing file: {filePath} ===");
        
        try
        {
            // Step 1: Read file
            string rawData = _fileReader.ReadFile(filePath);
            
            // Step 2: Validate data
            if (!_dataValidator.ValidateData(rawData))
            {
                _auditLogger.LogOperation($"File Processing: {filePath}", false);
                Console.WriteLine("Data validation failed!");
                return false;
            }
            
            // Step 3: Transform data
            string transformedData = _dataTransformer.TransformData(rawData);
            
            // Step 4: Write to database
            bool writeSuccess = _databaseWriter.WriteToDatabase(transformedData);
            
            // Step 5: Log the operation
            _auditLogger.LogOperation($"File Processing: {filePath}", writeSuccess);
            
            if (writeSuccess)
            {
                Console.WriteLine("=== File processed successfully! ===\n");
                return true;
            }
            else
            {
                Console.WriteLine("=== File processing failed at database write ===\n");
                return false;
            }
        }
        catch (Exception ex)
        {
            _auditLogger.LogOperation($"File Processing: {filePath}", false);
            Console.WriteLine($"Error processing file: {ex.Message}");
            return false;
        }
    }
    
    // Batch processing method
    public void ProcessMultipleFiles(string[] filePaths)
    {
        Console.WriteLine($"=== Batch processing {filePaths.Length} files ===");
        
        int successCount = 0;
        foreach (string filePath in filePaths)
        {
            if (ProcessFile(filePath))
            {
                successCount++;
            }
        }
        
        Console.WriteLine($"Batch processing complete: {successCount}/{filePaths.Length} files processed successfully");
    }
}

// Client code
public class FileProcessingClient
{
    public static void Main()
    {
        var processor = new FileProcessingFacade();
        
        // Process single file
        processor.ProcessFile("C:\\data\\customer_data.csv");
        
        // Process multiple files
        string[] files = {
            "C:\\data\\orders.csv",
            "C:\\data\\products.csv",
            "C:\\data\\inventory.csv"
        };
        
        processor.ProcessMultipleFiles(files);
    }
}
```

---

## When to Use Facade Pattern

### **Good Use Cases:**

1. **Complex APIs**: When you have a complex library or framework that requires many steps to use
2. **Legacy System Integration**: Wrapping old systems with modern, clean interfaces
3. **Microservices**: Creating API gateways that aggregate multiple services
4. **Third-party Libraries**: Simplifying complex third-party integrations
5. **Layered Architecture**: Creating service layers that hide business logic complexity

### **Avoid When:**

1. **Over-simplification**: Don't hide functionality that clients actually need
2. **Single Responsibility**: If your subsystem is already simple, facade adds unnecessary complexity
3. **Performance Critical**: Extra layer might add unwanted overhead
4. **Frequently Changing Requirements**: If clients need different combinations of subsystem features

---

## Common Interview Follow-up Questions

### Q1 - What's the difference between Facade and Adapter patterns?

**Answer:**
- **Facade**: Simplifies a **complex interface** by providing a higher-level interface
- **Adapter**: Makes an **incompatible interface** compatible with what the client expects
- **Analogy**: Facade is like a hotel concierge (simplifies), Adapter is like a power plug converter (makes compatible)

### Q2 - Can Facade pattern violate Single Responsibility Principle?

**Answer:**
Yes, if not designed carefully. A facade can become a "God object" that does too much. **Solution:**
- Keep facades focused on specific domains (e.g., separate facades for Authentication, Payment, Notification)
- Use composition to break down large facades
- Each facade should have a single, well-defined purpose

### Q3 - How does Facade pattern affect unit testing?

**Answer:**
- **Pros**: Easier to test client code because you only need to mock the facade
- **Cons**: Harder to test individual subsystems in isolation
- **Best Practice**: Create interfaces for facades to enable easier mocking:

```csharp
public interface IBankingFacade
{
    bool WithdrawMoney(string userId, string pin, string accountNumber, decimal amount, string phoneNumber);
    decimal CheckBalance(string userId, string pin, string accountNumber);
}

public class BankingFacade : IBankingFacade
{
    // Implementation...
}
```

### Q4 - What are the disadvantages of Facade pattern?

**Answer:**
1. **Additional Layer**: Adds another layer of abstraction (potential performance impact)
2. **Limited Flexibility**: Clients can't access subsystem features not exposed by facade
3. **Tight Coupling Risk**: Facade can become tightly coupled to all subsystems
4. **Over-simplification**: Might hide important details that clients need to know

### Q5 - How do you handle errors in Facade pattern?

**Answer:**

```csharp
public class RobustBankingFacade
{
    public Result<decimal> WithdrawMoney(WithdrawalRequest request)
    {
        try
        {
            // Facade operations...
            return Result<decimal>.Success(newBalance);
        }
        catch (AuthenticationException ex)
        {
            return Result<decimal>.Failure("Authentication failed", ex);
        }
        catch (InsufficientFundsException ex)
        {
            return Result<decimal>.Failure("Insufficient funds", ex);
        }
        catch (Exception ex)
        {
            // Log the error but don't expose internal details
            _logger.LogError(ex, "Withdrawal failed for account {Account}", request.AccountNumber);
            return Result<decimal>.Failure("Transaction failed. Please try again.");
        }
    }
}

// Result wrapper class for better error handling
public class Result<T>
{
    public bool IsSuccess { get; private set; }
    public T Value { get; private set; }
    public string ErrorMessage { get; private set; }
    public Exception Exception { get; private set; }

    public static Result<T> Success(T value) => new Result<T> { IsSuccess = true, Value = value };
    public static Result<T> Failure(string error, Exception ex = null) => 
        new Result<T> { IsSuccess = false, ErrorMessage = error, Exception = ex };
}
```

This approach provides better error handling while maintaining the simplicity that Facade pattern offers.