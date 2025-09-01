# Action filter

## Definition:
An action filter is a type of attribute in ASP.NET MVC and ASP.NET Core that allows you to run code before or after an action method executes. They are primarily used to add functionality such as logging, authentication, validation, or modifying the result returned by actions.

## Purpose:
- **Separation of Concerns**: Action filters promote clean code by separating cross-cutting concerns from your business logic. This helps maintain clear, readable, and maintainable code.
- **Reusable Functionality**: Filters can be applied to multiple action methods and controllers, allowing you to reuse code conveniently across your application.
- **Centralized Logic**: They provide a centralized place to handle responsibilities that pertain to multiple actions, making your application easier to manage.

## Types of Action Filters:
- **Authorization Filters**: Handle user authentication and authorization.
- **Action Filters**: Allow logic to run before and after the action method executions.
- **Result Filters**: Allow logic to run before and after the action result is executed.
- **Exception Filters**: Handle exceptions thrown by action methods.

## Real-Time Example: Logging User Actions
Consider a scenario where you want to log user actions across your MVC application for analytics or auditing purposes. Instead of adding logging code to every controller action, you can create a custom action filter to handle this behavior.
Here’s how you can create a simple logging action filter:

```csharp
public class LogActionFilter : ActionFilterAttribute
{
    public override void OnActionExecuting(ActionExecutingContext context)
    {
        // Logic before the action executes
        var actionName = context.ActionDescriptor.ActionName;
        var userId = context.HttpContext.User.Identity.Name;
        Console.WriteLine($"User {userId} is executing action {actionName}");
        base.OnActionExecuting(context);
    }

    public override void OnActionExecuted(ActionExecutedContext context)
    {
        // Logic after the action executes
        Console.WriteLine($"Action Executed: {context.ActionDescriptor.ActionName}");
        base.OnActionExecuted(context);
    }
}
```

**Usage**: You can apply this filter globally or on specific controllers/actions:

```csharp
[ServiceFilter(typeof(LogActionFilter))]
public class HomeController : Controller
{
    public ActionResult Index()
    {
        return View();
    }
}
```

In this example, every time the Index action of HomeController executes, the log action filter will log which user is executing the action as well as when it has been executed.

## Benefits:
- **Improved Maintainability**: You keep logging responsibilities isolated from your application logic, making changes to logging behavior easy and localized.
- **Reusable Code**: If you need logging in multiple controllers, adding the same filter is straightforward and avoids code duplication.
- **Enhanced Scalability**: Adding new features (like different logging mechanisms) does not require changes in business logic or action methods.
