This is a classic distributed systems security question that tests your understanding of service-to-service authentication patterns. Here's how to answer it comprehensively:

## The Core Challenge
When Service1 receives authentication details, Services 2 and 3 need to verify that incoming requests are legitimate without having direct access to the original user credentials. This is the "transitive trust" problem in microservices.

## Primary Solutions (Choose Based on Context)

### 1. **JWT Token Propagation** (Most Common)
Service1 validates the user credentials and creates a JWT token containing user identity and permissions. This token is passed through the call chain.

**Implementation:**
- Service1: Validates credentials → generates JWT → calls Service2 with JWT in Authorization header
- Service2: Validates JWT signature → extracts user context → calls Service3 with same JWT
- Service3: Validates JWT → processes request

**Advantages:** Stateless, contains user context, widely supported
**Trade-offs:** Token size, token lifecycle management

### 2. **Service Mesh with mTLS** (Enterprise/Cloud-Native)
Use a service mesh (Istio, Linkerd) that handles service-to-service authentication automatically through mutual TLS certificates.

**Implementation:**
- Each service gets a unique certificate from the service mesh
- Service mesh handles certificate rotation and validation
- Services communicate over encrypted, authenticated channels

### 3. **API Gateway Pattern**
Centralize authentication at an API gateway that forwards requests with service-specific tokens or headers.

## Advanced Considerations to Mention

**Token Types:**
- Access tokens for user identity
- Service tokens for service-to-service calls
- Refresh token handling for long-running processes

**Security Best Practices:**
- Principle of least privilege (each service only gets necessary permissions)
- Token expiration and rotation
- Audit logging across all services
- Network-level security (VPC, security groups)

**Scalability Patterns:**
- Token caching strategies
- Circuit breakers for auth service failures
- Async token validation

## How to Present This Answer

1. **Start with the problem:** "This is about establishing transitive trust in a distributed system..."

2. **Present the JWT solution first** (most interviewers expect this)

3. **Show depth:** Mention token structure, validation flow, and security considerations

4. **Demonstrate experience:** "In my experience, I've implemented this using [specific technology] and handled challenges like token expiration during long-running processes..."

5. **Consider alternatives:** Briefly mention service mesh or OAuth2 client credentials flow for service-to-service auth

6. **Address edge cases:** "We also need to consider what happens if Service2 fails - does Service3 need to validate the original user context independently?"

This approach shows you understand both the theoretical concepts and practical implementation challenges, which is what distinguishes senior engineers in technical interviews.

--- 

# The Transitive Trust Problem in Microservices

## What is Transitive Trust?

**Transitive trust** is a security challenge in distributed systems where Service A trusts Service B, and Service B trusts Service C, but Service C needs to verify that the original request came from a legitimate source through Service A, not just from Service B directly.

The problem: **How does Service C know that Service B's request originated from an authenticated user, and not from a malicious actor who compromised Service B or is impersonating it?**

## Real-World Scenario

Let's say we have an e-commerce system:
- **Service A (API Gateway)**: Handles user authentication
- **Service B (Order Service)**: Processes orders
- **Service C (Inventory Service)**: Updates stock levels

When a user places an order:
1. User authenticates with Service A
2. Service A calls Service B to process the order
3. Service B calls Service C to reserve inventory

**The Challenge**: Service C receives a call from Service B, but how does it know:
- The original request came from an authenticated user?
- Service B itself is legitimate and not compromised?
- What permissions the original user had?Now, let's solve this with JWT-based authentication:## Detailed Explanation of the Solution

### How JWT Solves Transitive Trust

1. **Cryptographic Verification**: Each service independently validates the JWT signature using a shared secret or public key, ensuring the token hasn't been tampered with.

2. **Self-Contained Identity**: The JWT contains all necessary information about the user (ID, role, permissions), eliminating the need for database lookups.

3. **Token Propagation**: The same JWT token flows through the entire call chain, maintaining the original user context.

4. **Stateless Authentication**: Services don't need to maintain session state or call back to an authentication server.

## Common Follow-up Interview Questions

### 1. "What if the JWT secret key is compromised?"

**Answer**: 
- Implement **key rotation** strategy with multiple valid keys
- Use **asymmetric keys** (RS256) instead of symmetric (HS256) - services only need the public key
- Implement **token revocation** using a blacklist or short-lived tokens with refresh tokens
- Monitor for unusual token usage patterns
- Consider using **service mesh** with automatic certificate rotation

### 2. "How do you handle token expiration in long-running processes?"

**Answer**:
```csharp
// Token refresh mechanism
public class TokenRefreshService
{
    public async Task<string> RefreshTokenIfNeeded(string token)
    {
        var handler = new JwtSecurityTokenHandler();
        var jsonToken = handler.ReadJwtToken(token);
        
        // Check if token expires in next 5 minutes
        if (jsonToken.ValidTo < DateTime.UtcNow.AddMinutes(5))
        {
            // Call refresh token endpoint
            return await RefreshToken(token);
        }
        
        return token;
    }
}
```

### 3. "What about performance? JWT validation on every request seems expensive."

**Answer**:
- **Token Caching**: Cache validated tokens in memory with expiration
- **Asymmetric Keys**: Public key validation is fast
- **Token Introspection**: For high-security scenarios, validate against auth server periodically
- **Service Mesh**: Offload JWT validation to sidecar proxies

### 4. "How do you handle service-to-service authentication vs user authentication?"

**Answer**:
```csharp
// Different token types for different contexts
public class TokenService
{
    public string GenerateUserToken(User user) { /* User context token */ }
    
    public string GenerateServiceToken(string serviceName, List<string> scopes)
    {
        var claims = new List<Claim>
        {
            new Claim("service_name", serviceName),
            new Claim("token_type", "service"),
            new Claim(JwtRegisteredClaimNames.Sub, serviceName)
        };
        
        scopes.ForEach(scope => claims.Add(new Claim("scope", scope)));
        // Generate token...
    }
}
```

### 5. "What if Service B is compromised? Can it impersonate users?"

**Answer**:
- **Principle of Least Privilege**: Each service only gets tokens with minimal required permissions
- **Network Segmentation**: Services communicate over private networks
- **Request Signing**: Add request signatures for critical operations
- **Audit Logging**: Log all token usage for anomaly detection
- **Token Scoping**: Limit token scope to specific operations/resources

### 6. "How do you implement fine-grained authorization?"

**Answer**:
```csharp
[JwtAuthorize("inventory:reserve")]
public async Task<IActionResult> ReserveInventory([FromBody] ReserveInventoryRequest request)
{
    var tokenInfo = (TokenInfo)HttpContext.Items["TokenInfo"];
    
    // Resource-level authorization
    if (request.ProductId == SensitiveProductId && tokenInfo.Role != "Admin")
        return Forbid("Insufficient privileges for this product");
        
    // Rate limiting per user
    if (await _rateLimitService.IsRateLimited(tokenInfo.UserId, "inventory:reserve"))
        return StatusCode(429, "Rate limit exceeded");
        
    // Business rule authorization
    if (request.Quantity > 100 && !tokenInfo.Permissions.Contains("bulk_order"))
        return Forbid("Bulk orders require special permission");
}
```

### 7. "What about token size and bandwidth concerns?"

**Answer**:
- **Minimal Claims**: Only include essential information in JWT
- **Token Compression**: Use JWT compression for large tokens
- **Reference Tokens**: Use opaque tokens with server-side lookup for complex scenarios
- **Selective Claims**: Different token types for different use cases

## Alternative Solutions to Mention

### 1. **OAuth 2.0 Client Credentials Flow**
For service-to-service authentication without user context.

### 2. **Service Mesh (Istio/Linkerd)**
Automatic mTLS and service identity management.

### 3. **API Gateway Pattern**
Centralized authentication with downstream service tokens.

### 4. **Zero Trust Architecture**
Every service validates every request regardless of source.

This comprehensive explanation demonstrates deep understanding of distributed systems security, practical implementation skills, and awareness of real-world challenges - exactly what technical interviewers are looking for.

--- 
```csharp
// PROBLEMATIC APPROACH - Shows the transitive trust problem

// Service A - API Gateway
[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    private readonly IOrderService _orderService;
    private readonly IUserService _userService;

    public AuthController(IOrderService orderService, IUserService userService)
    {
        _orderService = orderService;
        _userService = userService;
    }

    [HttpPost("login")]
    public async Task<IActionResult> Login([FromBody] LoginRequest request)
    {
        // Validate user credentials
        var user = await _userService.ValidateCredentials(request.Username, request.Password);
        if (user == null)
            return Unauthorized("Invalid credentials");

        // Store user in session or some mechanism
        HttpContext.Session.SetString("UserId", user.Id.ToString());
        HttpContext.Session.SetString("UserRole", user.Role);

        return Ok(new { Message = "Login successful", UserId = user.Id });
    }

    [HttpPost("place-order")]
    public async Task<IActionResult> PlaceOrder([FromBody] OrderRequest orderRequest)
    {
        // Check if user is authenticated (simplified)
        var userIdStr = HttpContext.Session.GetString("UserId");
        if (string.IsNullOrEmpty(userIdStr))
            return Unauthorized("User not authenticated");

        var userId = int.Parse(userIdStr);
        
        // PROBLEM: We're calling Service B without any way for downstream 
        // services to verify the original user's identity or permissions
        var result = await _orderService.ProcessOrderAsync(orderRequest, userId);
        
        return Ok(result);
    }
}

// Service B - Order Service
[ApiController]
[Route("api/[controller]")]
public class OrderController : ControllerBase
{
    private readonly IInventoryService _inventoryService;
    private readonly IOrderRepository _orderRepository;

    public OrderController(IInventoryService inventoryService, 
                           IOrderRepository orderRepository)
    {
        _inventoryService = inventoryService;
        _orderRepository = orderRepository;
    }

    [HttpPost("process")]
    public async Task<IActionResult> ProcessOrder([FromBody] OrderRequest orderRequest, 
                                                  int userId)
    {
        // PROBLEM: How do we know this userId is legitimate?
        // What if someone directly calls this endpoint with any userId?
        
        // Create order
        var order = new Order
        {
            UserId = userId,
            ProductId = orderRequest.ProductId,
            Quantity = orderRequest.Quantity,
            OrderDate = DateTime.UtcNow
        };

        // PROBLEM: When we call Service C, we're not passing any authentication context
        // Service C has no way to verify this request is legitimate
        var inventoryResult = await _inventoryService.ReserveInventoryAsync(
            orderRequest.ProductId, 
            orderRequest.Quantity);

        if (!inventoryResult.Success)
            return BadRequest("Insufficient inventory");

        await _orderRepository.SaveOrderAsync(order);
        return Ok(new { OrderId = order.Id, Message = "Order processed successfully" });
    }
}

// Service C - Inventory Service  
[ApiController]
[Route("api/[controller]")]
public class InventoryController : ControllerBase
{
    private readonly IInventoryRepository _inventoryRepository;

    public InventoryController(IInventoryRepository inventoryRepository)
    {
        _inventoryRepository = inventoryRepository;
    }

    [HttpPost("reserve")]
    public async Task<IActionResult> ReserveInventory(
                 [FromBody] ReserveInventoryRequest request)
    {
        // MAJOR PROBLEM: We have no idea who is making this request!
        // - Is it coming from legitimate Service B?
        // - Is the original user authorized to make this reservation?
        // - Could this be a malicious actor directly calling our endpoint?
        // - What if someone is trying to reserve inventory without placing an order?

        var currentStock = await _inventoryRepository.GetStockAsync(request.ProductId);
        
        if (currentStock < request.Quantity)
        {
            return Ok(new InventoryResult 
            { 
                Success = false, 
                Message = "Insufficient stock",
                AvailableQuantity = currentStock
            });
        }

        // We're blindly trusting this request and updating inventory
        await _inventoryRepository.UpdateStockAsync(request.ProductId, -request.Quantity);
        
        return Ok(new InventoryResult 
        { 
            Success = true, 
            Message = "Inventory reserved successfully",
            ReservedQuantity = request.Quantity
        });
    }
}

// Supporting Classes
public class LoginRequest
{
    public string Username { get; set; }
    public string Password { get; set; }
}

public class OrderRequest
{
    public int ProductId { get; set; }
    public int Quantity { get; set; }
}

public class ReserveInventoryRequest
{
    public int ProductId { get; set; }
    public int Quantity { get; set; }
}

public class Order
{
    public int Id { get; set; }
    public int UserId { get; set; }
    public int ProductId { get; set; }
    public int Quantity { get; set; }
    public DateTime OrderDate { get; set; }
}

public class User
{
    public int Id { get; set; }
    public string Username { get; set; }
    public string Role { get; set; }
}

public class InventoryResult
{
    public bool Success { get; set; }
    public string Message { get; set; }
    public int ReservedQuantity { get; set; }
    public int AvailableQuantity { get; set; }
}

/* 
PROBLEMS WITH THIS APPROACH:

1. NO AUTHENTICATION CONTEXT: Service C has 
   no way to verify the request came from an authenticated user

2. NO AUTHORIZATION: Even if we passed userId, Service C can't verify user permissions

3. DIRECT ACCESS VULNERABILITY: Malicious actors can directly call Service B or C endpoints

4. NO REQUEST TRACING: Can't trace requests back to original user for audit purposes

5. NO SERVICE AUTHENTICATION: Services can't verify each other's identity

6. SESSION DEPENDENCY: Service A relies on session state, doesn't scale 
   in distributed environment

7. NO TOKEN EXPIRATION: No mechanism for token/session expiration

8. REPLAY ATTACKS: No protection against request replay attacks

9. NO RATE LIMITING: Can't implement per-user rate limiting in downstream services

10. NO FINE-GRAINED PERMISSIONS: Can't implement resource-level 
    permissions in downstream services
*/
```


```csharp
// JWT-BASED SOLUTION - Solves the transitive trust problem

using Microsoft.IdentityModel.Tokens;
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using System.Text;

// JWT Service for token generation and validation
public class JwtService
{
    private readonly string _secretKey;
    private readonly string _issuer;
    private readonly int _expirationMinutes;

    public JwtService(IConfiguration configuration)
    {
        _secretKey = configuration["Jwt:SecretKey"];
        _issuer = configuration["Jwt:Issuer"];
        _expirationMinutes = int.Parse(configuration["Jwt:ExpirationMinutes"]);
    }

    public string GenerateToken(User user, List<string> permissions = null)
    {
        var tokenHandler = new JwtSecurityTokenHandler();
        var key = Encoding.ASCII.GetBytes(_secretKey);

        var claims = new List<Claim>
        {
            new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
            new Claim(ClaimTypes.Name, user.Username),
            new Claim(ClaimTypes.Role, user.Role),
            new Claim("user_id", user.Id.ToString()),
            new Claim("iat", DateTimeOffset.UtcNow.ToUnixTimeSeconds().ToString(), 
                             ClaimValueTypes.Integer64),
            new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()) // Unique token ID
        };

        // Add custom permissions if provided
        if (permissions != null)
        {
            foreach (var permission in permissions)
            {
                claims.Add(new Claim("permission", permission));
            }
        }

        var tokenDescriptor = new SecurityTokenDescriptor
        {
            Subject = new ClaimsIdentity(claims),
            Expires = DateTime.UtcNow.AddMinutes(_expirationMinutes),
            Issuer = _issuer,
            Audience = _issuer,
            SigningCredentials = new SigningCredentials(new SymmetricSecurityKey(key), 
                SecurityAlgorithms.HmacSha256Signature)
        };

        var token = tokenHandler.CreateToken(tokenDescriptor);
        return tokenHandler.WriteToken(token);
    }

    public ClaimsPrincipal ValidateToken(string token)
    {
        try
        {
            var tokenHandler = new JwtSecurityTokenHandler();
            var key = Encoding.ASCII.GetBytes(_secretKey);

            var validationParameters = new TokenValidationParameters
            {
                ValidateIssuerSigningKey = true,
                IssuerSigningKey = new SymmetricSecurityKey(key),
                ValidateIssuer = true,
                ValidIssuer = _issuer,
                ValidateAudience = true,
                ValidAudience = _issuer,
                ValidateLifetime = true,
                ClockSkew = TimeSpan.Zero
            };

            var principal = tokenHandler.ValidateToken(token, validationParameters, 
                            out SecurityToken validatedToken);
            return principal;
        }
        catch (Exception ex)
        {
            // Log the exception
            throw new UnauthorizedAccessException("Invalid token", ex);
        }
    }

    public TokenInfo ExtractTokenInfo(string token)
    {
        var principal = ValidateToken(token);
        return new TokenInfo
        {
            UserId = int.Parse(principal.FindFirst("user_id")?.Value ?? "0"),
            Username = principal.FindFirst(ClaimTypes.Name)?.Value,
            Role = principal.FindFirst(ClaimTypes.Role)?.Value,
            Permissions = principal.FindAll("permission").Select(c => c.Value).ToList(),
            TokenId = principal.FindFirst(JwtRegisteredClaimNames.Jti)?.Value,
            IssuedAt = DateTimeOffset.FromUnixTimeSeconds(
                long.Parse(principal.FindFirst("iat")?.Value ?? "0"))
        };
    }
}

// Custom Authorization Attribute
public class JwtAuthorizeAttribute : Attribute, IAuthorizationFilter
{
    private readonly string[] _requiredPermissions;

    public JwtAuthorizeAttribute(params string[] requiredPermissions)
    {
        _requiredPermissions = requiredPermissions ?? Array.Empty<string>();
    }

    public void OnAuthorization(AuthorizationFilterContext context)
    {
        var jwtService = context.HttpContext.RequestServices.GetService<JwtService>();
        
        if (!context.HttpContext.Request.Headers.ContainsKey("Authorization"))
        {
            context.Result = new UnauthorizedResult();
            return;
        }

        var authHeader = context.HttpContext.Request.Headers["Authorization"]
                                .FirstOrDefault();
        if (authHeader == null || !authHeader.StartsWith("Bearer "))
        {
            context.Result = new UnauthorizedResult();
            return;
        }

        var token = authHeader.Substring("Bearer ".Length).Trim();
        
        try
        {
            var tokenInfo = jwtService.ExtractTokenInfo(token);
            
            // Check required permissions
            if (_requiredPermissions.Any() && 
                !_requiredPermissions.All(p => tokenInfo.Permissions.Contains(p)))
            {
                context.Result = new ForbidResult("Insufficient permissions");
                return;
            }

            // Add token info to HttpContext for use in controllers
            context.HttpContext.Items["TokenInfo"] = tokenInfo;
            context.HttpContext.Items["JWT"] = token; // Pass token for downstream calls
        }
        catch (UnauthorizedAccessException)
        {
            context.Result = new UnauthorizedResult();
        }
    }
}

// HTTP Client service for making authenticated requests to other services
public class AuthenticatedHttpClientService
{
    private readonly HttpClient _httpClient;

    public AuthenticatedHttpClientService(HttpClient httpClient)
    {
        _httpClient = httpClient;
    }

    public async Task<HttpResponseMessage> PostAsync(string requestUri, 
                                       object content, string jwtToken)
    {
        var json = JsonSerializer.Serialize(content);
        var httpContent = new StringContent(json, Encoding.UTF8, "application/json");
        
        // Forward the JWT token to downstream service
        _httpClient.DefaultRequestHeaders.Authorization = 
            new System.Net.Http.Headers.AuthenticationHeaderValue("Bearer", jwtToken);

        return await _httpClient.PostAsync(requestUri, httpContent);
    }
}

// Service A - API Gateway (FIXED)
[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    private readonly IUserService _userService;
    private readonly JwtService _jwtService;
    private readonly AuthenticatedHttpClientService _httpClient;

    public AuthController(IUserService userService, JwtService jwtService, 
        AuthenticatedHttpClientService httpClient)
    {
        _userService = userService;
        _jwtService = jwtService;
        _httpClient = httpClient;
    }

    [HttpPost("login")]
    public async Task<IActionResult> Login([FromBody] LoginRequest request)
    {
        var user = await _userService.ValidateCredentials(request.Username, 
                                                          request.Password);
        if (user == null)
            return Unauthorized("Invalid credentials");

        // Define user permissions based on role
        var permissions = GetUserPermissions(user.Role);

        // Generate JWT token with user identity and permissions
        var token = _jwtService.GenerateToken(user, permissions);

        return Ok(new LoginResponse 
        { 
            Token = token, 
            ExpiresIn = 3600, // 1 hour
            TokenType = "Bearer",
            User = new UserInfo { Id = user.Id, Username = user.Username, 
                                  Role = user.Role }
        });
    }

    [HttpPost("place-order")]
    [JwtAuthorize("place_order")] // Requires specific permission
    public async Task<IActionResult> PlaceOrder([FromBody] OrderRequest orderRequest)
    {
        // Extract token info (set by JwtAuthorize attribute)
        var tokenInfo = (TokenInfo)HttpContext.Items["TokenInfo"];
        var jwtToken = (string)HttpContext.Items["JWT"];

        // SOLUTION: Pass the JWT token to Service B
        // This allows Service B and C to verify the original user's identity and permissions
        var response = await _httpClient.PostAsync(
            "https://order-service/api/order/process", 
            new { OrderRequest = orderRequest, UserId = tokenInfo.UserId }, 
            jwtToken);

        if (response.IsSuccessStatusCode)
        {
            var result = await response.Content.ReadAsStringAsync();
            return Ok(JsonSerializer.Deserialize<OrderResult>(result));
        }

        return StatusCode((int)response.StatusCode, "Order processing failed");
    }

    private List<string> GetUserPermissions(string role)
    {
        return role switch
        {
            "Admin" => new List<string> { "place_order", "cancel_order", "view_all_orders",
             "manage_inventory" },
            "Customer" => new List<string> { "place_order", "view_own_orders" },
            "Guest" => new List<string> { "view_products" },
            _ => new List<string>()
        };
    }
}

// Service B - Order Service (FIXED)
[ApiController]
[Route("api/[controller]")]
public class OrderController : ControllerBase
{
    private readonly AuthenticatedHttpClientService _httpClient;
    private readonly IOrderRepository _orderRepository;
    private readonly ILogger<OrderController> _logger;

    public OrderController(AuthenticatedHttpClientService httpClient, 
        IOrderRepository orderRepository, ILogger<OrderController> logger)
    {
        _httpClient = httpClient;
        _orderRepository = orderRepository;
        _logger = logger;
    }

    [HttpPost("process")]
    [JwtAuthorize("place_order")] // SOLUTION: Verify JWT token and required permission
    public async Task<IActionResult> ProcessOrder([FromBody] ProcessOrderRequest request)
    {
        // SOLUTION: Extract verified user information from JWT
        var tokenInfo = (TokenInfo)HttpContext.Items["TokenInfo"];
        var jwtToken = (string)HttpContext.Items["JWT"];

        _logger.LogInformation($"Processing order for user {tokenInfo.UserId} 
                                ({tokenInfo.Username})");

        // Verify the user ID matches the token (additional security check)
        if (request.UserId != tokenInfo.UserId)
        {
            _logger.LogWarning($"User ID mismatch: token={tokenInfo.UserId}, 
                                 request={request.UserId}");
            return Forbid("User ID mismatch");
        }

        var order = new Order
        {
            UserId = tokenInfo.UserId, // Use verified user ID from token
            ProductId = request.OrderRequest.ProductId,
            Quantity = request.OrderRequest.Quantity,
            OrderDate = DateTime.UtcNow
        };

        // SOLUTION: Forward the same JWT token to Service C
        // Service C can now verify the original user's identity and permissions
        var inventoryResponse = await _httpClient.PostAsync(
            "https://inventory-service/api/inventory/reserve",
            new ReserveInventoryRequest 
            { 
                ProductId = request.OrderRequest.ProductId, 
                Quantity = request.OrderRequest.Quantity,
                OrderId = order.Id,
                RequestedBy = tokenInfo.UserId
            },
            jwtToken); // Forward the JWT token

        if (!inventoryResponse.IsSuccessStatusCode)
        {
            _logger.LogError($"Inventory reservation failed for order {order.Id}");
            return BadRequest("Insufficient inventory or reservation failed");
        }

        await _orderRepository.SaveOrderAsync(order);
        
        _logger.LogInformation($"Order {order.Id} processed successfully 
                                 for user {tokenInfo.UserId}");
        
        return Ok(new OrderResult 
        { 
            OrderId = order.Id, 
            Message = "Order processed successfully",
            UserId = tokenInfo.UserId,
            ProcessedAt = DateTime.UtcNow
        });
    }
}

// Service C - Inventory Service (FIXED)
[ApiController]
[Route("api/[controller]")]
public class InventoryController : ControllerBase
{
    private readonly IInventoryRepository _inventoryRepository;
    private readonly ILogger<InventoryController> _logger;

    public InventoryController(IInventoryRepository inventoryRepository, 
                               ILogger<InventoryController> logger)
    {
        _inventoryRepository = inventoryRepository;
        _logger = logger;
    }

    [HttpPost("reserve")]
    [JwtAuthorize("place_order")] // SOLUTION: Verify JWT token and permission
    public async Task<IActionResult> ReserveInventory([FromBody] ReserveInventoryRequest 
    request)
    {
        // SOLUTION: Extract verified user information from JWT
        var tokenInfo = (TokenInfo)HttpContext.Items["TokenInfo"];

        _logger.LogInformation($"Inventory reservation request from user 
        {tokenInfo.UserId} ({tokenInfo.Username}) " +
                             $"for product {request.ProductId}, quantity {request.Quantity}");

        // SOLUTION: Verify the requesting user matches the token
        if (request.RequestedBy != tokenInfo.UserId)
        {
            _logger.LogWarning($"User ID mismatch in inventory request: 
                token={tokenInfo.UserId}, request={request.RequestedBy}");
            return Forbid("User ID mismatch");
        }

        // SOLUTION: We now have full context about who is making this request
        // - We know the original authenticated user
        // - We've verified their permissions
        // - We can implement user-specific business logic

        var currentStock = await _inventoryRepository.GetStockAsync(request.ProductId);
        
        if (currentStock < request.Quantity)
        {
            _logger.LogWarning($"Insufficient stock for product {request.ProductId}: 
               available={currentStock}, requested={request.Quantity}");
            return Ok(new InventoryResult 
            { 
                Success = false, 
                Message = "Insufficient stock",
                AvailableQuantity = currentStock
            });
        }

        // Additional business logic based on user role
        if (tokenInfo.Role == "Guest")
        {
            return Forbid("Guests cannot reserve inventory");
        }

        // Check if user has exceeded their reservation limit (example business rule)
        var userReservations = await _inventoryRepository
                .GetUserReservationsAsync(tokenInfo.UserId);
        if (userReservations.Count >= 10 && tokenInfo.Role == "Customer")
        {
            return BadRequest("User has exceeded maximum reservation limit");
        }

        // Reserve inventory
        await _inventoryRepository.UpdateStockAsync(request.ProductId, -request.Quantity);
        await _inventoryRepository.CreateReservationAsync(new InventoryReservation
        {
            ProductId = request.ProductId,
            Quantity = request.Quantity,
            UserId = tokenInfo.UserId,
            OrderId = request.OrderId,
            ReservedAt = DateTime.UtcNow,
            ExpiresAt = DateTime.UtcNow.AddMinutes(15) // Reservation expires in 15 minutes
        });

        _logger.LogInformation($"Successfully reserved {request.Quantity} units of product 
                  {request.ProductId} " +
                             $"for user {tokenInfo.UserId}");
        
        return Ok(new InventoryResult 
        { 
            Success = true, 
            Message = "Inventory reserved successfully",
            ReservedQuantity = request.Quantity,
            ReservationId = Guid.NewGuid().ToString()
        });
    }
}

// Supporting Classes
public class TokenInfo
{
    public int UserId { get; set; }
    public string Username { get; set; }
    public string Role { get; set; }
    public List<string> Permissions { get; set; } = new();
    public string TokenId { get; set; }
    public DateTimeOffset IssuedAt { get; set; }
}

public class LoginResponse
{
    public string Token { get; set; }
    public int ExpiresIn { get; set; }
    public string TokenType { get; set; }
    public UserInfo User { get; set; }
}

public class UserInfo
{
    public int Id { get; set; }
    public string Username { get; set; }
    public string Role { get; set; }
}

public class ProcessOrderRequest
{
    public OrderRequest OrderRequest { get; set; }
    public int UserId { get; set; }
}

public class ReserveInventoryRequest
{
    public int ProductId { get; set; }
    public int Quantity { get; set; }
    public int OrderId { get; set; }
    public int RequestedBy { get; set; }
}

public class OrderResult
{
    public int OrderId { get; set; }
    public string Message { get; set; }
    public int UserId { get; set; }
    public DateTime ProcessedAt { get; set; }
}

public class InventoryReservation
{
    public int Id { get; set; }
    public int ProductId { get; set; }
    public int Quantity { get; set; }
    public int UserId { get; set; }
    public int OrderId { get; set; }
    public DateTime ReservedAt { get; set; }
    public DateTime ExpiresAt { get; set; }
}

/* 
HOW THIS SOLVES THE TRANSITIVE TRUST PROBLEM:

1. AUTHENTICATED CONTEXT: Every service receives and validates the JWT token,
   ensuring they know the original authenticated user.

2. VERIFIED IDENTITY: Each service can extract user ID, role, and permissions
   from the cryptographically signed JWT token.

3. AUTHORIZATION: Services can check if the user has required permissions
   before processing requests.

4. AUDIT TRAIL: Full traceability of requests back to the original user.

5. TAMPERING PROTECTION: JWT signature ensures token hasn't been modified.

6. EXPIRATION: Tokens expire, limiting the window of potential misuse.

7. STATELESS: No need for central session storage or database lookups.

8. SERVICE-TO-SERVICE TRUST: Services trust the JWT signature rather than
   blindly trusting upstream services.

9. FINE-GRAINED PERMISSIONS: Can implement resource-level and action-level
   authorization in any service.

10. SCALABILITY: Stateless approach scales well in distributed systems.
*/
```