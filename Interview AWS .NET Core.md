# AWS .NET Core API Interview Preparation Guide

## Core AWS Services for API Development

### 1. **AWS Lambda + API Gateway**
**What it is:** Serverless compute service that runs your .NET Core code without managing servers.

**Key concepts you need to know:**
- **Lambda functions** execute your C# code in response to HTTP requests
- **API Gateway** acts as the front door, routing HTTP requests to Lambda functions
- Cold start implications (first invocation takes longer)
- Memory and timeout configurations
- Lambda layers for shared dependencies

**Interview questions to expect:**
- "How would you deploy a .NET Core API as serverless functions?"
- "What are the pros and cons of Lambda vs traditional hosting?"
- "How do you handle cold starts in production?"

**Talking points:**
- You can use AWS Lambda with .NET 6/8 runtime
- Deploy using AWS SAM (Serverless Application Model) or CloudFormation
- Best for event-driven, variable workload APIs
- Cost-effective for sporadic traffic patterns

### 2. **Elastic Container Service (ECS) / Fargate**
**What it is:** Container orchestration for running Dockerized .NET Core APIs.

**Key concepts:**
- **ECS** manages Docker containers at scale
- **Fargate** is serverless compute for containers (no EC2 management)
- Task definitions specify your container configuration
- Application Load Balancer distributes traffic

**Interview questions:**
- "How would you containerize a .NET Core API for AWS?"
- "What's the difference between ECS and Fargate?"
- "How do you handle service discovery and load balancing?"

**Sample answer framework:**
"I'd create a Dockerfile for the .NET Core API, push it to ECR (Elastic Container Registry), define an ECS task with resource requirements, and use Fargate for serverless container execution. I'd place an Application Load Balancer in front for traffic distribution and health checks."

### 3. **Elastic Beanstalk**
**What it is:** Platform-as-a-Service that handles deployment, scaling, and monitoring automatically.

**Key points:**
- Simplest way to deploy .NET Core APIs on AWS
- Automatically provisions EC2, load balancers, auto-scaling
- Supports .NET Core natively
- Good for teams wanting less infrastructure management

**When to mention it:** "For rapid deployment with minimal AWS expertise, Elastic Beanstalk abstracts infrastructure complexity while still allowing customization."

## CI/CD and Deployment

### AWS CodePipeline + CodeBuild + CodeDeploy
**What you should know:**
- **CodePipeline** orchestrates the entire CI/CD workflow
- **CodeBuild** compiles your .NET Core application (uses buildspec.yml)
- **CodeDeploy** handles deployment to EC2, ECS, or Lambda
- Integration with GitHub, BitBucket, or AWS CodeCommit

**Sample buildspec.yml knowledge:**
```yaml
version: 0.2
phases:
  build:
    commands:
      - dotnet restore
      - dotnet build
      - dotnet publish -c Release -o ./output
```

**Alternative:** Mention GitHub Actions, Azure DevOps, or Jenkins can also deploy to AWS using AWS CLI or SDKs.

## Security and Authentication

### 1. **IAM (Identity and Access Management)**
**Critical concepts:**
- IAM roles for services (not hardcoded credentials)
- Least privilege principle
- Service roles for Lambda, ECS tasks

**Interview answer:** "I'd assign IAM roles to Lambda functions or ECS tasks so they can access other AWS services like DynamoDB or S3 without storing credentials in code."

### 2. **API Authentication**
- **Cognito** for user authentication (JWT tokens)
- **API Gateway authorizers** (Lambda authorizers or Cognito)
- **AWS WAF** for API protection

**Sample answer:** "For user authentication, I'd use Amazon Cognito to issue JWT tokens, then configure API Gateway to validate these tokens before forwarding requests to the backend."

### 3. **Secrets Management**
- **AWS Secrets Manager** or **Systems Manager Parameter Store** for connection strings, API keys
- Never hardcode credentials in appsettings.json

## Data Storage Options

### Common choices for .NET Core APIs:
1. **RDS (Relational Database Service)** - SQL Server, PostgreSQL, MySQL
2. **DynamoDB** - NoSQL, serverless database
3. **Aurora Serverless** - Auto-scaling relational database
4. **S3** - Object storage for files

**Know how to discuss:** Connection string management, database migrations in AWS, read replicas for scaling.

## Monitoring and Logging

### CloudWatch
**What to know:**
- **CloudWatch Logs** automatically captures Lambda/ECS logs
- **CloudWatch Metrics** for performance monitoring
- Custom metrics using AWS SDK in your .NET code
- **X-Ray** for distributed tracing

**Interview answer:** "I'd use CloudWatch Logs to centralize application logs, set up CloudWatch Alarms for error rates or latency thresholds, and implement X-Ray for tracing requests across microservices."

## Architecture Patterns to Discuss

### 1. **Microservices Architecture**
- Multiple Lambda functions or ECS services
- API Gateway as single entry point
- Event-driven communication via SNS/SQS

### 2. **API Best Practices on AWS**
- Versioning strategies (URL path vs headers)
- Rate limiting with API Gateway usage plans
- Caching with API Gateway caching or CloudFront
- CORS configuration

### 3. **Scaling Considerations**
- Lambda scales automatically
- ECS auto-scaling based on CPU/memory
- RDS read replicas for database scaling
- ElastiCache (Redis/Memcached) for caching

## Cost Optimization

**Be ready to discuss:**
- Lambda pricing (per request + execution time)
- Reserved instances for predictable workloads
- S3 storage classes for different access patterns
- Right-sizing EC2/ECS resources

## Infrastructure as Code

### AWS CDK (Cloud Development Kit) with C#
**Strong point to mention:** "Since I'm a .NET developer, I can use AWS CDK with C# to define infrastructure in the same language I write application code."

### CloudFormation / Terraform
Basic understanding of declarative infrastructure definitions.

## Quick Win Talking Points

1. **"In Azure, I used [X], which I understand maps to [Y] in AWS"**
   - Azure Functions → AWS Lambda
   - Azure API Management → AWS API Gateway
   - Azure SQL Database → AWS RDS
   - Azure Cosmos DB → AWS DynamoDB

2. **"I'm prepared to learn AWS quickly because..."**
   - Strong .NET fundamentals transfer directly
   - Cloud concepts are similar across providers
   - AWS documentation is comprehensive

3. **Demonstrate enthusiasm:** "I've been exploring AWS documentation on Lambda with .NET and I'm excited about serverless patterns."

## Hands-On Preparation (If You Have Time)

1. **Create a free AWS account** and deploy a simple .NET Core API to Lambda
2. **Follow this path:** Create Lambda function → Add API Gateway trigger → Test with Postman
3. **AWS Free Tier** includes 1M Lambda requests/month

## Red Flag Questions and How to Handle

**"You have Azure experience but not AWS—why should we hire you?"**
"Cloud platforms share fundamental concepts—compute, storage, networking, security. My strong .NET Core expertise and understanding of cloud-native architecture means I can quickly adapt to AWS-specific services. The application logic and API design principles remain the same."

**"Describe a production AWS deployment you've done."**
Be honest: "I haven't deployed to AWS in production yet, but I've architected and deployed APIs on Azure, which has similar services. I understand the AWS equivalents and I'm a fast learner who's already studying AWS documentation and hands-on tutorials."

## Final Tips

- **Be honest** about your AWS experience level but confident in your ability to learn
- **Emphasize** your .NET Core API expertise—that's your strength
- **Show enthusiasm** for learning AWS
- **Prepare 2-3 questions** about their AWS architecture to show genuine interest
- **Review** the AWS Well-Architected Framework briefly (5 pillars: operational excellence, security, reliability, performance efficiency, cost optimization)

Good luck with your interview! Focus on translating your Azure knowledge to AWS concepts and demonstrating strong .NET Core fundamentals.

--- 

# AWS Lambda + API Gateway: Complete Deep Dive for Interviews

## What is AWS Lambda?

**Definition:** AWS Lambda is a serverless compute service that runs your code in response to events without provisioning or managing servers.

**Key characteristics:**
- You upload code, AWS handles everything else (OS, patching, scaling, availability)
- Pay only for compute time consumed (billed per millisecond)
- Automatically scales from zero to thousands of concurrent executions
- Supports .NET 6, .NET 8 runtimes (and others)

**When Lambda runs:**
- HTTP request via API Gateway
- File uploaded to S3
- Message in SQS queue
- Scheduled CloudWatch event
- Database change in DynamoDB

## What is API Gateway?

**Definition:** A fully managed service that creates, publishes, maintains, monitors, and secures REST, HTTP, and WebSocket APIs at any scale.

**Think of it as:** The front door to your Lambda functions. It handles:
- HTTP request routing
- Request/response transformation
- Authentication and authorization
- Rate limiting and throttling
- API versioning
- CORS configuration
- Request validation

## How They Work Together

```
User/Client → API Gateway → Lambda Function → Response → API Gateway → User/Client
                    ↓
            (Optional: DynamoDB, RDS, S3, other services)
```

**Flow:**
1. Client sends HTTP request to API Gateway endpoint
2. API Gateway validates request, checks authentication
3. API Gateway invokes Lambda function with event data
4. Lambda executes your .NET Core code
5. Lambda returns response to API Gateway
6. API Gateway formats and returns response to client

## .NET Core Lambda Function Structure

### Basic Lambda Handler

```csharp
using Amazon.Lambda.Core;
using Amazon.Lambda.APIGatewayEvents;

[assembly: LambdaSerializer(typeof(Amazon.Lambda.Serialization.SystemTextJson.DefaultLambdaJsonSerializer))]

namespace MyApi
{
    public class Function
    {
        // This is the Lambda handler method
        public APIGatewayProxyResponse FunctionHandler(
            APIGatewayProxyRequest request, 
            ILambdaContext context)
        {
            // Log to CloudWatch
            context.Logger.LogLine($"Request: {request.Path}");
            
            return new APIGatewayProxyResponse
            {
                StatusCode = 200,
                Body = "{\"message\": \"Hello from Lambda\"}",
                Headers = new Dictionary<string, string>
                {
                    { "Content-Type", "application/json" }
                }
            };
        }
    }
}
```

**Interview talking points:**
- `APIGatewayProxyRequest` contains HTTP method, path, headers, body, query parameters
- `ILambdaContext` provides runtime information (request ID, remaining time, logging)
- `APIGatewayProxyResponse` must include StatusCode, Body, and Headers
- The `[assembly: LambdaSerializer]` attribute tells Lambda how to serialize JSON

### Parsing Request Data

```csharp
public APIGatewayProxyResponse CreateUser(
    APIGatewayProxyRequest request, 
    ILambdaContext context)
{
    // Get query parameters
    var userId = request.QueryStringParameters?["userId"];
    
    // Get path parameters
    var id = request.PathParameters?["id"];
    
    // Get headers
    var authToken = request.Headers?["Authorization"];
    
    // Parse JSON body
    var user = JsonSerializer.Deserialize<User>(request.Body);
    
    // Your business logic here
    
    return new APIGatewayProxyResponse
    {
        StatusCode = 201,
        Body = JsonSerializer.Serialize(new { id = "123", message = "User created" })
    };
}
```

## Lambda Configuration Details

### 1. **Runtime**
- Select `.NET 6` or `.NET 8` (managed runtime)
- Alternative: Custom runtime using container images

### 2. **Memory Allocation**
- Range: 128 MB to 10,240 MB (10 GB)
- **Important:** CPU power scales with memory
- More memory = faster execution = potentially lower cost
- **Interview tip:** "I'd start with 512 MB and monitor CloudWatch metrics to optimize"

### 3. **Timeout**
- Maximum: 15 minutes
- Default: 3 seconds
- **For APIs:** Usually 5-30 seconds is sufficient
- **Interview answer:** "For synchronous API calls, I'd set timeout to 29 seconds (API Gateway's limit is 30 seconds)"

### 4. **Environment Variables**
```csharp
// Access in code
var connectionString = Environment.GetEnvironmentVariable("CONNECTION_STRING");
var apiKey = Environment.GetEnvironmentVariable("API_KEY");
```

**Use for:** Configuration that changes between environments (dev/staging/prod)

### 5. **IAM Role (Execution Role)**
**Critical concept:** Lambda needs permissions to access other AWS services

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "s3:GetObject"
      ],
      "Resource": "*"
    }
  ]
}
```

**Interview answer:** "I'd create an IAM role with least privilege, granting only the specific permissions needed, like DynamoDB read/write or S3 access."

## API Gateway Types

### 1. **REST API**
- Full-featured API management
- Support for API keys, usage plans, request/response transformations
- More expensive but more control
- **Use when:** You need advanced features like caching, detailed monitoring

### 2. **HTTP API**
- Lighter weight, lower cost (up to 70% cheaper)
- Simpler to set up
- Better performance (lower latency)
- **Use when:** Building simple, high-performance APIs

### 3. **WebSocket API**
- For real-time bidirectional communication
- Chat applications, live dashboards

**Interview tip:** "For most .NET Core REST APIs, I'd start with HTTP API for cost and performance, then migrate to REST API if we need advanced features."

## API Gateway Integration Types

### 1. **Lambda Proxy Integration** (Most Common)
```
Client → API Gateway → Lambda (receives full request) → Response
```

**Characteristics:**
- Lambda receives entire HTTP request as JSON
- Lambda must format response with StatusCode, Headers, Body
- Maximum flexibility

### 2. **Lambda Non-Proxy Integration**
```
Client → API Gateway (transforms request) → Lambda → API Gateway (transforms response) → Client
```

**Characteristics:**
- API Gateway transforms request/response
- Lambda receives simplified input
- More configuration required

**Interview answer:** "I prefer Lambda Proxy Integration for .NET APIs because it gives full control over request/response handling in C# code, which is more maintainable than API Gateway mapping templates."

## Cold Starts - Critical Interview Topic

### What is a Cold Start?

When Lambda hasn't been invoked recently, AWS must:
1. Provision a new execution environment
2. Download your code package
3. Initialize the .NET runtime
4. Execute static constructors and initialization code
5. Finally invoke your handler

**Impact:**
- First request: 2-5 seconds (can be longer for .NET)
- Subsequent requests (warm): 10-100 milliseconds

### Mitigation Strategies

#### 1. **Provisioned Concurrency**
```
Pre-warmed Lambda instances always ready
Cost: Pay for instances even when not handling requests
```

**Interview answer:** "For production APIs with strict latency requirements, I'd configure Provisioned Concurrency for at least the minimum expected load."

#### 2. **Keep Functions Warm**
```csharp
// CloudWatch rule invokes Lambda every 5 minutes
if (request.Headers?.ContainsKey("X-Warmup") == true)
{
    return new APIGatewayProxyResponse { StatusCode = 200 };
}
```

#### 3. **Optimize Code**
```csharp
// BAD: Creates new client on every invocation
public APIGatewayProxyResponse Handler(APIGatewayProxyRequest request)
{
    var dynamoClient = new AmazonDynamoDBClient(); // ❌ Cold start penalty
    // ...
}

// GOOD: Reuse client across invocations
private static readonly AmazonDynamoDBClient _dynamoClient = new AmazonDynamoDBClient(); // ✅

public APIGatewayProxyResponse Handler(APIGatewayProxyRequest request)
{
    // Use _dynamoClient
}
```

**Key principle:** Initialize expensive resources (DB connections, HTTP clients) outside the handler method.

#### 4. **Use Smaller Packages**
- Minimize dependencies
- Use Lambda Layers for shared libraries
- Smaller deployment packages = faster cold starts

**Interview answer:** "I'd minimize the deployment package size, initialize SDK clients as static fields outside the handler, and use Provisioned Concurrency for critical endpoints."

## Lambda Deployment Package

### Creating Deployment Package

```bash
# Navigate to project directory
cd MyLambdaFunction

# Restore dependencies
dotnet restore

# Publish for Lambda (Linux x64)
dotnet publish -c Release -r linux-x64 --self-contained false

# Create zip file
cd bin/Release/net8.0/linux-x64/publish
zip -r ../../../../../function.zip .
```

### What Goes in the Package?
- Compiled DLLs
- Dependencies (NuGet packages)
- Configuration files (if any)

**Size limits:**
- Direct upload: 50 MB (zipped)
- Via S3: 250 MB (unzipped)

## Lambda Layers

**What they are:** Shared libraries, runtimes, or dependencies that multiple Lambda functions can use.

**Benefits:**
- Reduce deployment package size
- Share common code across functions
- Separate business logic from dependencies

**Example structure:**
```
Layer: Common utilities, AWS SDK extensions
├── Function 1: Order API
├── Function 2: User API
└── Function 3: Payment API
```

**Interview answer:** "For a microservices architecture with multiple Lambda functions, I'd create a Lambda Layer containing shared utilities, models, and common dependencies. This reduces each function's package size and improves cold start times."

## API Gateway Routing

### Path-based Routing

```
GET  /users          → ListUsers Lambda
POST /users          → CreateUser Lambda
GET  /users/{id}     → GetUser Lambda
PUT  /users/{id}     → UpdateUser Lambda
DELETE /users/{id}   → DeleteUser Lambda
```

### Resource Configuration
```
Resource: /users/{id}
Method: GET
Integration: Lambda Function (GetUserFunction)
Path parameter: id
```

**In Lambda:**
```csharp
var userId = request.PathParameters["id"];
```

## Authentication & Authorization

### 1. **Lambda Authorizer (Custom)**

```csharp
public class AuthorizerFunction
{
    public APIGatewayCustomAuthorizerResponse Handler(
        APIGatewayCustomAuthorizerRequest request)
    {
        var token = request.Headers["Authorization"];
        
        // Validate token (JWT, API key, etc.)
        bool isValid = ValidateToken(token);
        
        return new APIGatewayCustomAuthorizerResponse
        {
            PrincipalID = "user123",
            PolicyDocument = new APIGatewayCustomAuthorizerPolicy
            {
                Statement = new List<APIGatewayCustomAuthorizerPolicy.IAMPolicyStatement>
                {
                    new APIGatewayCustomAuthorizerPolicy.IAMPolicyStatement
                    {
                        Effect = isValid ? "Allow" : "Deny",
                        Action = new HashSet<string> { "execute-api:Invoke" },
                        Resource = new HashSet<string> { request.MethodArn }
                    }
                }
            }
        };
    }
}
```

**Interview talking point:** "The Lambda Authorizer caches the policy for subsequent requests with the same token, improving performance."

### 2. **Cognito Authorizer**
- API Gateway validates JWT tokens from Cognito
- No custom code needed
- Built-in user management

**Interview answer:** "For user authentication, I'd use Amazon Cognito with API Gateway's built-in Cognito authorizer. Users authenticate with Cognito to receive JWT tokens, which API Gateway validates automatically before invoking Lambda."

### 3. **IAM Authorization**
- AWS Signature Version 4 signing
- Good for service-to-service calls
- Not suitable for public APIs

## Rate Limiting & Throttling

### Usage Plans
```
Usage Plan: Basic Tier
├── Rate: 1000 requests per second
├── Burst: 2000 requests
└── Quota: 1,000,000 requests per month
```

**Configuration:**
1. Create API Key
2. Create Usage Plan with rate/quota limits
3. Associate API Key with Usage Plan
4. Require API Key on API methods

**Interview answer:** "I'd implement Usage Plans with API keys for different customer tiers, setting appropriate rate limits and quotas to prevent abuse and manage costs."

## CORS Configuration

### Handling in API Gateway
```
Enable CORS on resource:
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET,POST,PUT,DELETE
Access-Control-Allow-Headers: Content-Type,Authorization
```

### Handling in Lambda (Alternative)
```csharp
return new APIGatewayProxyResponse
{
    StatusCode = 200,
    Headers = new Dictionary<string, string>
    {
        { "Access-Control-Allow-Origin", "*" },
        { "Access-Control-Allow-Headers", "Content-Type,Authorization" },
        { "Access-Control-Allow-Methods", "GET,POST,PUT,DELETE,OPTIONS" }
    },
    Body = JsonSerializer.Serialize(data)
};
```

## Monitoring & Logging

### CloudWatch Logs
**Automatic logging:**
```csharp
public APIGatewayProxyResponse Handler(
    APIGatewayProxyRequest request, 
    ILambdaContext context)
{
    context.Logger.LogLine($"Processing request: {request.RequestContext.RequestId}");
    context.Logger.LogLine($"Remaining time: {context.RemainingTime.TotalMilliseconds}ms");
    
    // Your logic
}
```

**Logs automatically sent to:** CloudWatch Logs group `/aws/lambda/YourFunctionName`

### CloudWatch Metrics (Automatic)
- **Invocations:** Number of times function executed
- **Duration:** Time function takes to execute
- **Errors:** Number of invocation errors
- **Throttles:** Requests rejected due to concurrency limits
- **ConcurrentExecutions:** Number of function instances processing events

### Custom Metrics
```csharp
using Amazon.CloudWatch;
using Amazon.CloudWatch.Model;

var cloudWatch = new AmazonCloudWatchClient();

await cloudWatch.PutMetricDataAsync(new PutMetricDataRequest
{
    Namespace = "MyAPI",
    MetricData = new List<MetricDatum>
    {
        new MetricDatum
        {
            MetricName = "OrdersProcessed",
            Value = 1,
            Unit = StandardUnit.Count,
            TimestampUtc = DateTime.UtcNow
        }
    }
});
```

### X-Ray Tracing
**Enable distributed tracing:**
```csharp
using Amazon.XRay.Recorder.Core;
using Amazon.XRay.Recorder.Handlers.AwsSdk;

AWSSDKHandler.RegisterXRayForAllServices();

// In handler
AWSXRayRecorder.Instance.BeginSubsegment("Database Query");
try
{
    // Database operation
}
finally
{
    AWSXRayRecorder.Instance.EndSubsegment();
}
```

**Interview answer:** "I'd enable X-Ray tracing to visualize the complete request flow, identify bottlenecks, and troubleshoot issues across Lambda, API Gateway, and downstream services like DynamoDB or RDS."

## Error Handling Best Practices

```csharp
public APIGatewayProxyResponse Handler(APIGatewayProxyRequest request, ILambdaContext context)
{
    try
    {
        // Validate input
        if (string.IsNullOrEmpty(request.Body))
        {
            return new APIGatewayProxyResponse
            {
                StatusCode = 400,
                Body = JsonSerializer.Serialize(new { error = "Request body is required" })
            };
        }
        
        // Business logic
        var result = ProcessRequest(request.Body);
        
        return new APIGatewayProxyResponse
        {
            StatusCode = 200,
            Body = JsonSerializer.Serialize(result)
        };
    }
    catch (ValidationException ex)
    {
        context.Logger.LogLine($"Validation error: {ex.Message}");
        return new APIGatewayProxyResponse
        {
            StatusCode = 400,
            Body = JsonSerializer.Serialize(new { error = ex.Message })
        };
    }
    catch (NotFoundException ex)
    {
        return new APIGatewayProxyResponse
        {
            StatusCode = 404,
            Body = JsonSerializer.Serialize(new { error = "Resource not found" })
        };
    }
    catch (Exception ex)
    {
        context.Logger.LogLine($"Unhandled error: {ex}");
        return new APIGatewayProxyResponse
        {
            StatusCode = 500,
            Body = JsonSerializer.Serialize(new { error = "Internal server error" })
        };
    }
}
```

## Deployment Methods

### 1. **AWS Console (Manual)**
- Upload ZIP file
- Configure manually
- **Use for:** Learning, testing

### 2. **AWS SAM (Serverless Application Model)**

**template.yaml:**
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31

Resources:
  MyApiFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: MyApi::MyApi.Function::FunctionHandler
      Runtime: dotnet8
      CodeUri: ./src/MyApi/
      MemorySize: 512
      Timeout: 30
      Environment:
        Variables:
          TABLE_NAME: !Ref MyTable
      Events:
        GetUsers:
          Type: Api
          Properties:
            Path: /users
            Method: get
        CreateUser:
          Type: Api
          Properties:
            Path: /users
            Method: post
            
  MyTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: Users
      AttributeDefinitions:
        - AttributeName: id
          AttributeType: S
      KeySchema:
        - AttributeName: id
          KeyType: HASH
      BillingMode: PAY_PER_REQUEST
```

**Deploy commands:**
```bash
# Build
sam build

# Deploy
sam deploy --guided
```

**Interview answer:** "I'd use AWS SAM for infrastructure as code. It simplifies Lambda and API Gateway deployment with a single YAML template, supports local testing, and integrates well with CI/CD pipelines."

### 3. **AWS CDK (with C#)**

```csharp
using Amazon.CDK;
using Amazon.CDK.AWS.Lambda;
using Amazon.CDK.AWS.APIGateway;

public class MyApiStack : Stack
{
    public MyApiStack(Construct scope, string id) : base(scope, id)
    {
        var myFunction = new Function(this, "MyFunction", new FunctionProps
        {
            Runtime = Runtime.DOTNET_8,
            Handler = "MyApi::MyApi.Function::FunctionHandler",
            Code = Code.FromAsset("./src/MyApi/bin/Release/net8.0/publish"),
            MemorySize = 512,
            Timeout = Duration.Seconds(30)
        });
        
        var api = new LambdaRestApi(this, "MyApi", new LambdaRestApiProps
        {
            Handler = myFunction,
            Proxy = false
        });
        
        var users = api.Root.AddResource("users");
        users.AddMethod("GET");
        users.AddMethod("POST");
    }
}
```

**Interview talking point:** "As a .NET developer, AWS CDK with C# is appealing because I can define infrastructure using familiar language and tools, with full IntelliSense and type safety."

### 4. **CI/CD with AWS CodePipeline**

**buildspec.yml:**
```yaml
version: 0.2

phases:
  pre_build:
    commands:
      - echo Restore started on `date`
      - dotnet restore
  build:
    commands:
      - echo Build started on `date`
      - dotnet build -c Release
      - dotnet publish -c Release -o ./output
  post_build:
    commands:
      - echo Package Lambda function
      - cd output
      - zip -r ../function.zip .
      
artifacts:
  files:
    - function.zip
    - template.yaml
```

**Pipeline stages:**
1. Source (GitHub/CodeCommit)
2. Build (CodeBuild - compile and package)
3. Deploy (CloudFormation/SAM - update Lambda and API Gateway)

## Concurrency & Scaling

### Automatic Scaling
- Lambda scales automatically from 0 to 1000+ concurrent executions
- Each request gets its own execution environment
- **Concurrency = requests per second × average duration (seconds)**

### Concurrency Limits
- **Account level:** 1000 concurrent executions (default, can be increased)
- **Function level (Reserved Concurrency):** Guarantee minimum capacity
- **Provisioned Concurrency:** Pre-initialized warm instances

**Example:**
```
API receives 500 req/sec
Average function duration: 200ms (0.2 sec)
Required concurrency: 500 × 0.2 = 100 concurrent executions
```

**Interview scenario:** "If I expect 1000 requests per second with 500ms average duration, I need 500 concurrent executions. I'd request a limit increase if necessary and monitor throttling metrics in CloudWatch."

## Cost Optimization

### Pricing Components
1. **Number of requests:** $0.20 per 1 million requests
2. **Compute time:** Based on memory allocated and execution duration
   - $0.0000166667 per GB-second

**Example calculation:**
```
3 million requests/month
512 MB memory
200ms average duration

Requests: 3M × $0.20/1M = $0.60
Compute: 3M × 0.2 sec × 0.5 GB × $0.0000166667 = $5.00
Total: ~$5.60/month
```

### Optimization Strategies

**1. Right-size memory:**
```
Test different memory allocations:
- 512 MB: 300ms execution = $X
- 1024 MB: 150ms execution = $Y (might be cheaper due to faster CPU)
```

**2. Reduce execution time:**
- Optimize code
- Use compiled queries (EF Core)
- Cache frequently accessed data
- Minimize cold starts

**3. Use reserved concurrency only when needed:**
- Reserved concurrency costs money even if unused
- Use only for critical functions

**Interview answer:** "I'd monitor CloudWatch metrics to find the optimal memory allocation where increased memory cost is offset by reduced execution time. I'd also implement caching to reduce database calls and overall execution time."

## Common Interview Questions & Answers

### Q: "How do you handle database connections in Lambda?"

**A:** "Lambda functions should reuse database connections across invocations. I'd create the connection as a static field outside the handler method so it persists in the execution environment:

```csharp
private static SqlConnection _connection;

public APIGatewayProxyResponse Handler(...)
{
    if (_connection == null || _connection.State != ConnectionState.Open)
    {
        _connection = new SqlConnection(connectionString);
        _connection.Open();
    }
    // Use connection
}
```

For RDS, I'd also use RDS Proxy to manage connection pooling and prevent overwhelming the database with too many connections from concurrent Lambda executions."

### Q: "What's the maximum execution time for Lambda with API Gateway?"

**A:** "Lambda supports up to 15 minutes, but API Gateway has a 30-second timeout limit. For APIs, the practical limit is 29 seconds. For longer operations, I'd return a job ID immediately and use asynchronous processing with SQS or Step Functions, then provide a separate endpoint to check job status."

### Q: "How do you handle secrets in Lambda?"

**A:** "I use AWS Secrets Manager or Systems Manager Parameter Store. The secret ARN is stored as an environment variable, and I retrieve it at runtime:

```csharp
private static readonly IAmazonSecretsManager _secretsManager = new AmazonSecretsManagerClient();
private static string _connectionString;

public async Task<APIGatewayProxyResponse> Handler(...)
{
    if (_connectionString == null)
    {
        var secretArn = Environment.GetEnvironmentVariable("DB_SECRET_ARN");
        var response = await _secretsManager.GetSecretValueAsync(new GetSecretValueRequest
        {
            SecretId = secretArn
        });
        _connectionString = response.SecretString;
    }
    // Use connection string
}
```

The Lambda execution role needs `secretsmanager:GetSecretValue` permission."

### Q: "How would you implement API versioning?"

**A:** "I'd use path-based versioning with separate API Gateway stages:

```
/v1/users → Lambda v1 function
/v2/users → Lambda v2 function
```

API Gateway stages (dev, staging, prod) can point to different Lambda versions/aliases. This allows:
- Running multiple versions simultaneously
- Gradual migration of clients
- Easy rollback if issues arise

Alternatively, use Lambda aliases with weighted routing for canary deployments."

### Q: "What happens when Lambda reaches concurrency limits?"

**A:** "When all available concurrency is consumed, new invocations are throttled. API Gateway receives a 429 (Too Many Requests) or 500 error. To handle this:

1. Request a concurrency limit increase from AWS
2. Implement retry logic with exponential backoff on the client side
3. Use SQS as a buffer - API immediately queues requests, separate Lambda processes them at a controlled rate
4. Monitor CloudWatch throttling metrics and set alarms
5. Consider reserved concurrency for critical functions"

### Q: "How do you test Lambda functions locally?"

**A:** "Several approaches:

1. **AWS SAM CLI:** `sam local start-api` - runs Lambda and API Gateway locally with Docker
2. **Unit testing:** Mock `APIGatewayProxyRequest` and test handler logic
3. **Integration testing:** Deploy to a dev/test AWS account
4. **LocalStack:** Open-source tool that emulates AWS services locally

I prefer SAM CLI for local development because it closely mimics the actual AWS environment, then use comprehensive unit tests for business logic."

---

## Final Interview Tips

**Key points to emphasize:**
- Lambda and API Gateway enable building scalable APIs without managing servers
- Cold starts are the main challenge - mitigate with optimization and Provisioned Concurrency
- Proper IAM roles and security practices are critical
- CloudWatch provides comprehensive monitoring out of the box
- Cost-effective for variable workloads, especially with the free tier

**Show enthusiasm:** "I'm excited about the serverless model because it lets developers focus on business logic rather than infrastructure. The automatic scaling and pay-per-use pricing align perfectly with modern application needs."

**Be honest:** "While I'm new to AWS specifically, I understand the Lambda programming model and have strong .NET Core fundamentals. I'm confident I can quickly become productive with AWS Lambda."

Good luck with your interview!

--- 
I am preparing for an interview for an architect role. The topic I am interested in now is `AWS Lambda + API Gateway for hosting .NET Core API to AWS`. In this context, following is the flow, I understood:
1. Client sends HTTP request to API Gateway endpoint (Is HTTPS supported? Is there any configuration for HTTPS, in AWS API Gateway?)
2. API Gateway validates request, checks authentication (How the authentication works?)
3. API Gateway invokes Lambda function with event data (What is event data here?)
4. Lambda executes your .NET Core code
5. Lambda returns response to API Gateway
6. API Gateway formats and returns response to client

Explain me each step, in interview point as well as for understanding, what are the questions may be asked on this? 
--- 

# 🏗️ **AWS Lambda + API Gateway with .NET Core: An Architect's Guide**

You've outlined the exact flow of serverless API processing in AWS! Let's walk through each step with the architectural details, gotchas, and interview-ready insights.

---

## ⚙️ **Step 1: Client Request to API Gateway**

**HTTPS Support:**
- Yes, API Gateway **fully supports HTTPS** and it's the default.
- Two options:
  - Default certificate (automatic for `*.execute-api.{region}.amazonaws.com`)
  - Custom domain with your own certificate via AWS Certificate Manager (ACM)

**Configuration for HTTPS:**
- Edge-optimized endpoints (default) — global CloudFront distribution
- Regional endpoints — better when clients are in the same region
- Private endpoints — for VPC-only access

**Interview Questions:**
- How would you configure custom domains with HTTPS?
- What's the difference between edge-optimized vs. regional endpoints?
- How would you handle TLS certificate rotation?

---

## 🔒 **Step 2: API Gateway Authentication/Validation**

**Authentication Options:**
1. **API Keys** — simple throttling and identification
2. **IAM Roles** — AWS signature-based auth (SigV4)
3. **Lambda Authorizers** — custom auth logic (formerly "Custom Authorizers")
4. **Cognito User Pools** — managed user auth service
5. **JWT/OAuth** — via Lambda Authorizers
6. **Resource Policies** — IP/VPC restrictions

**How Authentication Works (Example: Lambda Authorizer):**
- Request arrives with token in header (e.g., `Authorization: Bearer xyz`)
- API Gateway extracts token, passes to your Authorizer Lambda
- Authorizer returns IAM policy document that allows/denies access
- Result is cached (configurable TTL)

**Interview Questions:**
- Compare API Keys vs. Lambda Authorizers vs. Cognito
- How would you implement role-based access control (RBAC)?
- How do you handle cross-cutting concerns like rate limiting?

---

## 🔄 **Step 3: API Gateway Invokes Lambda with Event Data**

**Event Data Structure:**
The "event" is a JSON object containing:
- HTTP method, path, querystring, headers, body
- Stage variables
- Request context (requestId, accountId, etc.)
- Authorizer context (if auth is used)

Here's a simplified example:
```json
{
  "resource": "/products/{id}",
  "path": "/products/123",
  "httpMethod": "GET",
  "pathParameters": {"id": "123"},
  "headers": {"Accept": "application/json"},
  "queryStringParameters": {"version": "2"},
  "requestContext": { ... },
  "body": null
}
```

**Interview Questions:**
- What's the difference between proxy vs. non-proxy integrations?
- How do you access path parameters from Lambda?
- What happens if the request payload exceeds Lambda limits?

---

## ⚡ **Step 4: Lambda Executes .NET Core Code**

**Handler Types:**
1. **Custom event handler:**
```csharp
public APIGatewayProxyResponse FunctionHandler(APIGatewayProxyRequest request, ILambdaContext context)
{
    return new APIGatewayProxyResponse 
    {
        StatusCode = 200,
        Body = JsonSerializer.Serialize(new { message = "Hello World" })
    };
}
```

2. **ASP.NET Core integration:**
```csharp
// Startup.cs configuration
public class LambdaEntryPoint : Amazon.Lambda.AspNetCoreServer.APIGatewayProxyFunction
{
    protected override void Init(IWebHostBuilder builder)
    {
        builder.UseStartup<Startup>();
    }
}
```

**Architectural Considerations:**
- Cold starts impact (.NET is heavier than Node/Python)
- Memory settings affect CPU allocation
- Timeouts must be coordinated (API Gateway vs Lambda)

**Interview Questions:**
- How would you minimize cold starts for .NET Core Lambdas?
- When would you use container image deployment for .NET Lambdas?
- How do you handle database connections efficiently?

---

## ⏩ **Step 5: Lambda Returns Response to API Gateway**

**Response Structure:**
For proxy integration, Lambda returns:
```json
{
  "statusCode": 200,
  "headers": {
    "Content-Type": "application/json",
    "X-Custom-Header": "Value"
  },
  "body": "{\"message\":\"Success\"}",
  "isBase64Encoded": false
}
```

**Key Considerations:**
- Response size limits (6MB for sync calls)
- Binary content handling (`isBase64Encoded`)
- Error handling patterns

**Interview Questions:**
- How would you handle file uploads/downloads via this architecture?
- What happens if Lambda crashes or times out?
- How do you implement proper error responses?

---

## 🔚 **Step 6: API Gateway Returns Response to Client**

**Response Handling:**
- Response mapping (if non-proxy integration)
- Gateway responses for default error cases
- CORS headers (if configured)
- Caching (if enabled)

**Performance Optimizations:**
- API Gateway caching
- Compression
- CloudFront integration

**Interview Questions:**
- How would you configure CORS properly?
- What caching strategies would you implement?
- How would you handle versioning in your API?

---

## 🏗️ **Architecture-Level Considerations**

### **Deployment Patterns:**
- Infrastructure as Code (CloudFormation, SAM, CDK, Terraform)
- Multi-stage deployments (dev, test, prod)
- Canary releases
- API versioning

### **Security Considerations:**
- Defense in depth (WAF, Shield)
- Least privilege IAM roles
- Secrets management
- API throttling and quotas

### **Monitoring and Observability:**
- CloudWatch Logs/Metrics
- X-Ray tracing
- Structured logging
- Dashboards and alerts

### **Performance Tuning:**
- Provisioned concurrency for Lambdas
- Right-sizing memory
- Gateway caching
- Regional deployment strategy

### **Cost Optimization:**
- Request consolidation
- Caching
- Payload size optimization
- Lambda execution optimization

---

## 💡 **Key Interview Questions to Prepare For:**

1. "How would you migrate an existing monolithic ASP.NET Core API to this serverless architecture?"

2. "Describe the security layers you would implement in this architecture."

3. "How would you handle database connections efficiently with Lambda's stateless nature?"

4. "What are your strategies for minimizing cold starts in .NET Core Lambdas?"

5. "How would you implement a blue/green deployment for this serverless API?"

6. "How would you design the system for high availability across multiple regions?"

7. "Compare containerized Lambda vs. standard Lambda for .NET workloads."

8. "How would you implement proper error handling and logging in this architecture?"

9. "What's your approach to API versioning in this serverless context?"

10. "How would you optimize costs for a high-volume API using this architecture?"

---

This serverless approach is highly scalable and cost-effective but introduces architectural challenges around state management, cold starts, and distributed system design — perfect areas to demonstrate your architect-level understanding in an interview.

--- 

# 🏗️ **AWS Options for Hosting .NET Core Web API**

As an architect, you should know AWS offers multiple paths for hosting .NET Core APIs, each with distinct trade-offs. Here's a comprehensive overview for your interview prep:

---

## 1️⃣ **Compute-Based Options**

### **EC2 (Elastic Compute Cloud)**
- **Description:** Virtual machines where you install and configure everything.
- **Best for:** Maximum control, legacy integration needs, specific Windows dependencies.
- **Architectural considerations:**
  - You manage OS, patching, scaling, load balancing.
  - Good for complex, resource-intensive applications.
  - Requires separate CI/CD setup.

### **Elastic Beanstalk**
- **Description:** PaaS solution that abstracts infrastructure management.
- **Best for:** Traditional web applications, teams familiar with IIS/.NET hosting.
- **Architectural considerations:**
  - Handles deployment, capacity provisioning, load balancing, auto-scaling.
  - .NET Core on Windows or Linux platforms.
  - Less control but simpler operations.

---

## 2️⃣ **Container-Based Options**

### **ECS (Elastic Container Service)**
- **Description:** Container orchestration service.
- **Best for:** Microservices, consistent environments, Docker workflows.
- **Architectural considerations:**
  - Runs containerized .NET Core on EC2 or Fargate.
  - Better resource utilization than VMs.
  - Integrates with ECR, CloudWatch, ALB.

### **EKS (Elastic Kubernetes Service)**
- **Description:** Managed Kubernetes service.
- **Best for:** Complex container orchestration, multi-cloud strategies.
- **Architectural considerations:**
  - Industry-standard Kubernetes for container orchestration.
  - More complex than ECS but more powerful for large-scale deployments.
  - Consistent with Kubernetes deployments elsewhere.

### **App Runner**
- **Description:** Fully managed container service with zero infrastructure management.
- **Best for:** Simpler containerized APIs, rapid deployment needs.
- **Architectural considerations:**
  - Simplest container deployment option.
  - Automatic scaling, CI/CD integration.
  - Less flexible than ECS/EKS but faster to deploy.

---

## 3️⃣ **Serverless Options**

### **Lambda + API Gateway**
- **Description:** Serverless functions exposed via managed API service.
- **Best for:** Event-driven architectures, variable workloads, pay-per-use scenarios.
- **Architectural considerations:**
  - No infrastructure management.
  - Pay only for execution time.
  - Cold starts can impact .NET Core performance.
  - Execution time limits (15-minute max).
  - Different programming model (stateless, function-based).

### **Lambda with ASP.NET Core Integration**
- **Description:** Running ASP.NET Core inside Lambda functions.
- **Best for:** Migrating existing ASP.NET Core apps to serverless.
- **Architectural considerations:**
  - Uses Amazon.Lambda.AspNetCoreServer package.
  - Maintains familiar MVC/Web API programming model.
  - Performance overhead vs. direct Lambda functions.

---

## 4️⃣ **Hybrid and Advanced Options**

### **App Mesh**
- **Description:** Service mesh for microservices architecture.
- **Best for:** Complex microservices deployments.
- **Architectural considerations:**
  - Traffic control, monitoring, security between services.
  - Works with ECS, EKS, EC2, and Fargate.

### **Fargate**
- **Description:** Serverless compute engine for containers.
- **Best for:** Container workloads without server management.
- **Architectural considerations:**
  - Used with ECS or EKS.
  - No EC2 instances to manage.
  - Pay-per-task pricing model.

---

## 🧩 **Selection Criteria and Trade-offs**

| Aspect | EC2 | Elastic Beanstalk | ECS/EKS | Lambda |
|--------|-----|-------------------|---------|--------|
| Control | Highest | Medium | High | Lowest |
| Scaling | Manual/Auto | Auto | Auto | Automatic |
| Operations Effort | High | Low | Medium | Lowest |
| Cold Starts | No | No | Minimal | Yes |
| Cost Model | Hourly | Hourly | Container hours | Per request |
| Max Runtime | Unlimited | Unlimited | Unlimited | 15 minutes |
| Deployment Complexity | High | Low | Medium | Low |

---

## 🔍 **Interview-Ready Insights**

### **When to Choose Each Option:**

- **EC2:** Complex applications needing full control, specific Windows features, or existing .NET Framework components.

- **Elastic Beanstalk:** When you want simplicity but still need a traditional web server model; great for lift-and-shift migrations.

- **Containers (ECS/EKS):** When you need consistent deployments, better resource utilization, and modern CI/CD pipelines.

- **Lambda:** For APIs with variable traffic, when cost optimization is critical, or for event-driven architectures.

### **Key Decision Points:**

1. **Traffic patterns:** Consistent high traffic → Containers or EC2; Variable or low traffic → Lambda.

2. **Team skills:** DevOps expertise → EC2/ECS/EKS; Less infrastructure focus → Beanstalk/Lambda.

3. **Application architecture:** Monolithic → EC2/Beanstalk; Microservices → Containers/Lambda.

4. **Performance requirements:** Low-latency critical → EC2 or containers; Some latency acceptable → Lambda.

5. **Cost model:** Pay-as-you-go → Lambda; Steady-state workloads → reserved EC2 or container capacity.

---

## ✅ **Architect's Summary**

AWS offers a spectrum of hosting options for .NET Core APIs:
- **Traditional hosting** (EC2, Beanstalk) gives familiar environments.
- **Container services** (ECS, EKS, App Runner) offer deployment consistency.
- **Serverless** (Lambda) provides maximum abstraction and pay-per-use pricing.

The best choice depends on your workload characteristics, operational model, and business requirements. Modern architectures often combine multiple approaches, like using Lambda for API endpoints but containers for complex processing.

--- 

# 🏗️ **Best AWS Options for Monolithic .NET Core Web API**

For monolithic .NET Core Web APIs, you'll want options that support traditional web application characteristics: centralized deployments, vertical scaling, and potentially stateful behavior. Here are the optimal hosting choices, ranked from most appropriate to less ideal:

---

## 1️⃣ **Elastic Beanstalk (Top Choice for Monoliths)**

**Why it's preferred for monoliths:**
- Purpose-built for traditional web applications
- Simplest deployment experience (upload code → automatically provisions infrastructure)
- Supports both Windows (IIS) and Linux (Kestrel) environments
- Built-in monitoring, logging, and auto-scaling
- Zero infrastructure management while maintaining predictable performance

**Architectural advantages:**
- Easy blue/green deployments
- Simple configuration via web UI or `.ebextensions`
- Integrated with common CI/CD pipelines
- Managed load balancing and scaling

**Architect's insight:**
"Elastic Beanstalk strikes the ideal balance for monoliths—it abstracts infrastructure complexity while providing the consistent performance and operational model that monolithic apps expect."

---

## 2️⃣ **ECS/Fargate (Modern, Container-Based Option)**

**Why it works well:**
- Containerized deployment is more portable and reproducible
- Fargate handles server management with container-level isolation
- Well-suited for monoliths packaged in containers
- More consistent dev/prod parity
- Easier to migrate toward microservices later

**When to prefer over Beanstalk:**
- Team is already container-focused
- You need more control over networking
- Planning future decomposition into microservices
- Need specialized deployment strategies

**Architect's consideration:**
"Containerization with ECS/Fargate is the 'modernization bridge'—you maintain the monolithic codebase but gain deployment consistency and resource efficiency."

---

## 3️⃣ **EC2 (Traditional but Highest Control)**

**When to choose:**
- Need absolute control over infrastructure
- Have complex Windows/.NET dependencies
- Specific performance tuning requirements
- Existing operational procedures favor direct VM management

**Architectural aspects:**
- You manage the entire stack (OS, runtime, scaling)
- Traditional IIS on Windows Server or Kestrel on Linux
- Requires more operational overhead
- Auto Scaling Groups + Application Load Balancer for availability

**Architect's perspective:**
"EC2 is the traditionalist's choice—most control but most responsibility. Ideal for complex monoliths with specific infrastructure requirements that other services can't accommodate."

---

## ❌ **Not Recommended for Monoliths**

### **Lambda + API Gateway**
- **Why it's problematic:** 
  - 15-minute execution limit
  - Cold start penalties for .NET
  - Not designed for long-running processes
  - Difficult to migrate existing monolithic code structure
  - State management challenges

### **App Runner**
- **Limitations for monoliths:**
  - Less control over networking
  - Limited customization for complex deployment requirements
  - Better for simpler services

---

## 🧠 **Decision Framework for Architects**

| Consideration | Elastic Beanstalk | ECS/Fargate | EC2 |
|---------------|-------------------|-------------|-----|
| **Operational Overhead** | Low | Medium | High |
| **Deployment Simplicity** | Highest | Medium | Lowest |
| **Control Level** | Medium | High | Highest |
| **Cost Predictability** | Good | Better | Best with Reserved Instances |
| **Migration Effort** | Low (upload and deploy) | Medium (containerize first) | High (full infrastructure setup) |
| **.NET Core Integration** | Excellent, preconfigured | Good, custom Dockerfile | Excellent, full control |

---

## 🚀 **Implementation Tips for .NET Monoliths**

### **For Elastic Beanstalk:**
```yaml
# .ebextensions/01-settings.config
option_settings:
  aws:elasticbeanstalk:container:dotnet:
    Target Runtime: .NET Core
  aws:elasticbeanstalk:environment:proxy:
    ProxyServer: nginx
  aws:autoscaling:asg:
    MinSize: 2
    MaxSize: 6
```

### **For ECS/Fargate (Dockerfile):**
```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY ["MyMonolith.csproj", ""]
RUN dotnet restore "MyMonolith.csproj"
COPY . .
RUN dotnet build "MyMonolith.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "MyMonolith.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "MyMonolith.dll"]
```

---

## ✅ **In Short**

For monolithic .NET Core Web APIs:

1. **Elastic Beanstalk** is the preferred option for most scenarios—simplest path with the right balance of management vs. control.

2. **ECS/Fargate** is excellent if you're containerizing your workflow or planning future microservices.

3. **EC2** remains viable when you need absolute control over the environment.

The key is matching your operational model, team skills, and future architecture direction—not just technical capabilities.

--- 

# Elastic Beanstalk

# 🏗️ **AWS Elastic Beanstalk for .NET Core Monolith APIs: Interview Guide**

Elastic Beanstalk is AWS's **Platform-as-a-Service (PaaS)** offering that simplifies deploying and managing applications. It's particularly well-suited for monolithic .NET Core Web APIs, as it abstracts infrastructure complexity while providing the consistent performance that traditional applications require.

---

## ⚙️ **Key Concepts & Terminology**

| Term | Definition |
|------|------------|
| **Environment** | A complete stack of resources (EC2, Load Balancer, Auto Scaling Group) running a version of your application. |
| **Application Version** | A specific, labeled snapshot of deployable code. |
| **Platform** | Runtime environment for a specific language/framework (e.g., .NET on Windows Server or Linux). |
| **Configuration** | Settings defined via console, CLI, or `.ebextensions` that control the environment. |
| **Application** | Collection of environments, versions, and configurations. |
| **Worker Environment** | Environment designed to process tasks from an SQS queue (for background processing). |

---

## 🔍 **Architecture for .NET Core APIs**

Elastic Beanstalk for .NET Core creates:

1. **Load Balancer** (Application Load Balancer by default)
2. **Auto Scaling Group** of EC2 instances
3. **Security Groups** for traffic control
4. **CloudWatch** alarms and logs integration
5. **S3 Bucket** for version storage

For .NET Core specifically, it provides:
- **Windows Platform** with IIS or **.NET on Linux** with Kestrel/Nginx
- **Health monitoring** via `/health` endpoint
- **Configuration system** compatible with ASP.NET Core
- **Deployment mechanism** that understands .NET Core's publish model

---

## 🚀 **Deployment Workflow for .NET Core APIs**

1. **Package your application**:
   ```bash
   dotnet publish -c Release -o ./publish
   # Create deployment bundle (zip file)
   cd ./publish && zip -r ../deploy.zip *
   ```

2. **Deploy via:**
   - **AWS Console** (upload zip)
   - **AWS CLI**:
     ```bash
     aws elasticbeanstalk create-application-version \
       --application-name MyApi \
       --version-label v1 \
       --source-bundle S3Bucket=my-bucket,S3Key=deploy.zip
     
     aws elasticbeanstalk update-environment \
       --environment-name MyApi-prod \
       --version-label v1
     ```
   - **Visual Studio AWS Toolkit**
   - **CI/CD pipeline** (e.g., CodePipeline)

---

## 🧩 **.NET Core Configuration Options**

### **Platform Selection:**
```json
"aws:elasticbeanstalk:container:dotnet:apppool": {
  "Target Runtime": ".NET Core"
}
```

### **Environment Variables:**
```json
"aws:elasticbeanstalk:application:environment": {
  "ASPNETCORE_ENVIRONMENT": "Production",
  "ConnectionStrings__DefaultConnection": "..."
}
```

### **Instance Type:**
```json
"aws:autoscaling:launchconfiguration": {
  "InstanceType": "t3.medium"
}
```

### **Auto Scaling:**
```json
"aws:autoscaling:asg": {
  "MinSize": "2",
  "MaxSize": "6"
}
```

---

## 🎯 **Key Interview Questions & Answers**

### 1. **"What is Elastic Beanstalk and why is it suitable for monolithic .NET Core APIs?"**

**Answer:** 
Elastic Beanstalk is AWS's managed application platform that automates infrastructure provisioning and application deployment. It's ideal for monolithic .NET Core APIs because it:

- Provides a complete, pre-configured stack (load balancer, auto-scaling, monitoring)
- Supports both Windows (IIS) and Linux (Kestrel) runtimes for .NET Core
- Handles capacity provisioning, load balancing, and health monitoring automatically
- Maintains a traditional web server model familiar to .NET developers
- Offers simple deployment workflows (ZIP upload or CLI)
- Enables blue/green deployments for zero-downtime updates
- Abstracts infrastructure management while preserving performance predictability
- Scales vertically (instance types) and horizontally (number of instances)

---

### 2. **"How does Elastic Beanstalk handle scaling for .NET Core applications?"**

**Answer:**
Elastic Beanstalk offers multiple scaling mechanisms:

- **Auto Scaling Groups:** Automatically adjusts capacity based on defined policies.
- **Scaling Triggers:** CPU utilization, network traffic, or custom CloudWatch metrics.
- **Time-Based Scaling:** Schedule capacity changes for predictable traffic patterns.
- **Manual Scaling:** Adjust min/max instance counts through console or API.

For .NET Core specifically, it intelligently handles:
- Request queuing for IIS (Windows platform)
- Proper Kestrel configuration (Linux platform)
- Connection draining during scaling events
- Session affinity options via sticky sessions
- Warm-up periods for new instances (.NET JIT compilation)

Configuration example:
```yaml
# .ebextensions/scaling.config
option_settings:
  aws:autoscaling:asg:
    MinSize: 2
    MaxSize: 10
  aws:autoscaling:trigger:
    UpperThreshold: 70
    LowerThreshold: 40
    MeasureName: CPUUtilization
    Unit: Percent
    UpperBreachScaleIncrement: 1
    LowerBreachScaleIncrement: -1
```

---

### 3. **"Compare .NET on Windows vs Linux platforms in Elastic Beanstalk."**

**Answer:**
| Aspect | Windows Platform | Linux Platform |
|--------|-----------------|----------------|
| **Web Server** | IIS with ASP.NET Core Module | Nginx/Apache + Kestrel |
| **Performance** | Good, more memory overhead | Better, more lightweight |
| **Cost** | Higher (Windows licensing) | Lower |
| **Cold Start** | Slower | Faster |
| **Deployments** | Slightly slower | Slightly faster |
| **Windows-specific APIs** | Fully supported | Limited or unavailable |
| **File System Access** | Local storage, slower | Local storage, faster |
| **Configuration** | web.config + .ebextensions | appsettings.json + .ebextensions |

The choice depends on:
- Cost considerations (Linux is typically 40% less expensive)
- Windows-specific dependencies
- Team familiarity
- Performance requirements (Linux is generally more efficient)

For modern .NET Core APIs without Windows dependencies, Linux is increasingly recommended.

---

### 4. **"How would you configure environment-specific settings for a .NET Core API in Elastic Beanstalk?"**

**Answer:**
Multiple options are available:

1. **Environment Variables:**
   ```yaml
   # .ebextensions/env-vars.config
   option_settings:
     aws:elasticbeanstalk:application:environment:
       ASPNETCORE_ENVIRONMENT: Production
       ConnectionStrings__DefaultConnection: "Server=..."
       ApiKey: "{{resolve:ssm:/myapi/api-key:1}}"
   ```

2. **appsettings.{Environment}.json files:**
   - Include different environment configurations in your deployment package
   - Set ASPNETCORE_ENVIRONMENT to activate the right one

3. **AWS Systems Manager Parameter Store:**
   ```csharp
   // Program.cs
   builder.Configuration.AddSystemsManager("/myapp/config");
   ```

4. **Configuration Files (.ebextensions):**
   ```yaml
   # .ebextensions/app-settings.config
   files:
     "/var/app/staging/appsettings.Production.json":
       content: |
         {
           "Logging": {
             "LogLevel": { "Default": "Information" }
           },
           "ConnectionStrings": {
             "DefaultConnection": "..."
           }
         }
   ```

Best practice is combining environment variables for sensitive values with appsettings files for complex configurations.

---

### 5. **"How does deployment work for .NET Core applications on Elastic Beanstalk?"**

**Answer:**
Elastic Beanstalk deployment for .NET Core follows this process:

1. **Build & Publish:** `dotnet publish -c Release`
2. **Package:** Create a ZIP of the published output
3. **Upload:** Push ZIP to Elastic Beanstalk (directly or via S3)
4. **Deployment Steps:**
   - EB downloads the package to each instance
   - Extracts to staging directory
   - For Windows: Configures IIS application pool and site
   - For Linux: Sets up Nginx/Apache proxy to Kestrel
   - Runs deployment hooks (if configured)
   - Health check on `/health` endpoint
   - If healthy, routes traffic to new version

**Deployment Strategies:**
- **All-at-once:** Updates all instances simultaneously (fastest, but downtime)
- **Rolling:** Updates batches of instances (slower, minimal impact)
- **Rolling with additional batch:** Ensures capacity during deployment (slower, zero downtime)
- **Immutable:** Creates new Auto Scaling group (safest, zero downtime)
- **Blue/Green:** Creates entirely new environment (separate from deployment settings)

The recommended approach for production .NET Core APIs is either Rolling with additional batch or Immutable deployment.

---

### 6. **"How would you handle database migrations with a .NET Core API in Elastic Beanstalk?"**

**Answer:**
Several strategies for handling EF Core migrations with Elastic Beanstalk:

1. **Deployment Hooks:**
   ```yaml
   # .ebextensions/db-migrate.config
   container_commands:
     01_migrations:
       command: "dotnet ef database update"
       leader_only: true
   ```

2. **Separate Migration Process:**
   - Deploy migrations separately via a Lambda function
   - Use deployment pipeline step before application deployment
   - Create a dedicated migration application

3. **On-Start Migration:**
   ```csharp
   // Program.cs
   if (app.Environment.IsProduction())
   {
       using var scope = app.Services.CreateScope();
       var db = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
       db.Database.Migrate();
   }
   ```

4. **Migration Bundles:**
   - Use EF Core's migration bundles to create executables
   - Run as a container command before application start

Best practice is typically using a separate migration step in your CI/CD pipeline, with appropriate transaction handling and rollback strategies.

---

### 7. **"What monitoring and logging options are available for .NET Core APIs on Elastic Beanstalk?"**

**Answer:**
Elastic Beanstalk integrates with AWS monitoring and provides .NET Core-specific options:

1. **CloudWatch Integration:**
   - **Metrics:** CPU, network, disk usage automatically captured
   - **Custom Metrics:** Application metrics via CloudWatch API
   - **Alarms:** Configurable thresholds for scaling or notifications

2. **Logging Options:**
   - **Instance Logs:** `/var/log/eb-engine.log` (deployment)
   - **.NET Logs:** `var/log/web.stdout.log` (standard output)
   - **IIS Logs:** (Windows platform)
   - **Nginx Logs:** (Linux platform)
   - **Log Streaming:** Real-time logs via EB console or CLI
   - **Log Rotation:** Automatic with configurable retention

3. **Health Reporting:**
   - **Basic Health:** Instance and environment status
   - **Enhanced Health:** Deeper visibility into application health
   - **Custom Health Checks:** API endpoints for business logic validation

4. **.NET Core Application Insights:**
   ```csharp
   // Program.cs
   builder.Services.AddApplicationInsightsTelemetry();
   ```

5. **X-Ray Integration:**
   ```csharp
   // Program.cs
   builder.Services.AddAWSXRayRecorder();
   ```

Best practice is to implement structured logging via Serilog or NLog, and stream logs to CloudWatch Logs for centralized analysis.

---

### 8. **"How would you optimize performance for a .NET Core Web API on Elastic Beanstalk?"**

**Answer:**
Performance optimization for .NET Core on Elastic Beanstalk includes:

1. **Instance Optimization:**
   - Right-size EC2 instances (memory-optimized for .NET Core)
   - Enable EC2 enhanced networking
   - Use Graviton processors for Linux platform (better price/performance)

2. **Application Configuration:**
   ```yaml
   # .ebextensions/app-config.config
   option_settings:
     aws:elasticbeanstalk:container:dotnet:
       NumProcesses: 1
       NumThreads: 0 # Auto-based on cores
   ```

3. **GC Optimization:**
   ```
   # .ebextensions/env-vars.config
   option_settings:
     aws:elasticbeanstalk:application:environment:
       DOTNET_gcServer: true
       DOTNET_gcConcurrent: true
   ```

4. **Response Compression:**
   ```csharp
   // Program.cs
   builder.Services.AddResponseCompression();
   ```

5. **Caching Layer:**
   - Add ElastiCache (Redis) for distributed caching
   - Configure output caching for appropriate endpoints

6. **Database Connection Pooling:**
   ```csharp
   // appsettings.json
   "ConnectionStrings": {
     "DefaultConnection": "Server=...;Connection Timeout=30;Max Pool Size=100;"
   }
   ```

7. **Load Balancer Optimization:**
   - Configure sticky sessions if needed
   - Adjust connection idle timeout
   - Enable HTTP/2 support

8. **Static Content:**
   - Offload to CloudFront CDN
   - Configure appropriate caching headers

Best practice combines right-sized infrastructure with application-level optimizations like caching, connection pooling, and response compression.

---

### 9. **"How do you handle secrets and sensitive configuration in .NET Core on Elastic Beanstalk?"**

**Answer:**
Secure configuration management for .NET Core on Elastic Beanstalk:

1. **AWS Systems Manager Parameter Store:**
   ```csharp
   // Program.cs
   builder.Configuration.AddSystemsManager("/myapp/secrets", true);
   ```

2. **AWS Secrets Manager:**
   ```csharp
   // Program.cs
   builder.Configuration.AddSecretsManager();
   ```

3. **Environment Variables with Variable Substitution:**
   ```yaml
   # .ebextensions/secrets.config
   option_settings:
     aws:elasticbeanstalk:application:environment:
       DbPassword: '{{resolve:secretsmanager:MyAppSecrets:SecretString:password}}'
   ```

4. **IAM Roles for EC2 Instances:**
   - Configure instance profile with least-privilege access
   - Grant access to specific Parameter Store paths or Secrets Manager resources

5. **Key Management Service (KMS):**
   - Encrypt sensitive values
   - Decrypt at runtime using KMS API

Never store secrets in:
- Source control
- appsettings.json files in the deployment package
- Unencrypted configuration files

Best practice is to use Systems Manager Parameter Store for configuration and Secrets Manager for highly sensitive values, with appropriate IAM roles.

---

### 10. **"How would you implement CI/CD for a .NET Core application on Elastic Beanstalk?"**

**Answer:**
CI/CD pipeline for .NET Core on Elastic Beanstalk typically includes:

1. **AWS CodePipeline Integration:**
   - **Source:** CodeCommit, GitHub, BitBucket
   - **Build:** CodeBuild with .NET Core SDK
   - **Test:** Unit and integration tests
   - **Deploy:** Direct deployment to Elastic Beanstalk

2. **Build Specification:**
   ```yaml
   # buildspec.yml
   version: 0.2
   phases:
     install:
       runtime-versions:
         dotnet: 8.0
     pre_build:
       commands:
         - dotnet restore
     build:
       commands:
         - dotnet test
         - dotnet publish -c Release -o ./publish
     post_build:
       commands:
         - cd publish && zip -r ../deploy.zip *
   artifacts:
     files:
       - deploy.zip
       - .ebextensions/**/*
   ```

3. **Deployment Strategy:**
   - Blue/Green deployments for zero downtime
   - Environment promotion (Dev → Test → Staging → Prod)
   - Automated rollback on health check failure

4. **Infrastructure as Code:**
   ```yaml
   # cloudformation.yml or Terraform
   Resources:
     Application:
       Type: AWS::ElasticBeanstalk::Application
     Environment:
       Type: AWS::ElasticBeanstalk::Environment
       Properties:
         EnvironmentName: MyApi-prod
         SolutionStackName: "64bit Amazon Linux 2023 v3.0.0 running .NET 8"
   ```

5. **Testing:**
   - Pre-deployment testing in pipeline
   - Post-deployment verification
   - Smoke tests against new environment

Best practice includes infrastructure as code, automated testing at multiple stages, and blue/green deployment approach for production environments.

---

## 🧠 **Best Practices for .NET Core on Elastic Beanstalk**

1. **Use Linux platform** unless Windows-specific features are required
2. **Implement health checks** specific to your application logic
3. **Leverage `.ebextensions`** for advanced configuration
4. **Enable enhanced health monitoring**
5. **Configure appropriate instance types** for .NET Core workloads
6. **Implement proper logging** with structured formats
7. **Use immutable deployments** for production
8. **Externalize configuration** via environment variables or Parameter Store
9. **Optimize for cold starts** by pre-compiling views and initialization
10. **Set up proper auto-scaling** based on application metrics

---

## 🔍 **Troubleshooting Common Issues**

1. **Deployment Failures:**
   - Check deployment logs in EB console
   - Verify ZIP structure matches expected layout
   - Ensure proper publish configuration

2. **Startup Failures:**
   - Verify ASPNETCORE_ENVIRONMENT setting
   - Check web.stdout.log for application errors
   - Confirm IAM permissions for any AWS services

3. **Performance Issues:**
   - Analyze enhanced health metrics
   - Check for memory leaks or high GC pressure
   - Verify instance size is appropriate

4. **Scaling Problems:**
   - Check health check configuration
   - Verify auto-scaling trigger thresholds
   - Look for long startup times affecting scale-out

---

This covers the essential knowledge about Elastic Beanstalk for .NET Core monolithic applications from an architect's perspective. These questions and answers should prepare you well for an architect-level interview on this topic!

--- 

# 🚀 **Creating Web APIs with ASP.NET Core: An Architect's Guide**

As an architect candidate, you'll need to demonstrate both technical depth and strategic vision for .NET Core Web API development. Here's a comprehensive overview of the creation process, architectural considerations, and interview-ready answers.

---

## ⚙️ **ASP.NET Core Web API Architecture**

### **Key Components**
- **Controllers**: Core classes that handle HTTP requests and return responses
- **Models**: Data structures for input/output and domain representation
- **Middleware Pipeline**: Sequential request processors
- **Services**: Business logic components injected via DI
- **Program.cs/Startup.cs**: Application bootstrap and configuration

### **Architecture Patterns in .NET Core Web APIs**
- **Clean/Onion Architecture**: Domain-centric, dependency rule enforcement
- **Vertical Slice Architecture**: Feature-based organization
- **CQRS**: Command-Query Responsibility Segregation
- **Repository Pattern**: Data access abstraction
- **Mediator Pattern**: Decoupling request handlers from controllers

---

## 🔄 **Step-by-Step Process for Creating a .NET Core Web API**

### **1. Project Setup**
```bash
# Create new Web API project
dotnet new webapi -n CompanyName.ProductName.Api

# Add supporting projects (typical architecture)
dotnet new classlib -n CompanyName.ProductName.Core
dotnet new classlib -n CompanyName.ProductName.Infrastructure
```

### **2. Configure Dependency Injection**
```csharp
// Program.cs (in .NET 6+)
builder.Services.AddScoped<IProductService, ProductService>();
builder.Services.AddScoped<IProductRepository, ProductRepository>();
builder.Services.AddDbContext<AppDbContext>(options => 
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
```

### **3. Setup Middleware Pipeline**
```csharp
// Program.cs
app.UseHttpsRedirection();
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();
app.UseExceptionHandler("/error");
app.MapControllers();
```

### **4. Implement API Controllers**
```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IProductService _productService;
    
    public ProductsController(IProductService productService)
    {
        _productService = productService;
    }
    
    [HttpGet]
    public async Task<ActionResult<IEnumerable<ProductDto>>> GetProducts()
    {
        var products = await _productService.GetProductsAsync();
        return Ok(products);
    }
    
    [HttpGet("{id}")]
    public async Task<ActionResult<ProductDto>> GetProduct(int id)
    {
        var product = await _productService.GetProductByIdAsync(id);
        if (product == null)
            return NotFound();
            
        return Ok(product);
    }
    
    [HttpPost]
    public async Task<ActionResult<ProductDto>> CreateProduct(CreateProductDto productDto)
    {
        var createdProduct = await _productService.CreateProductAsync(productDto);
        return CreatedAtAction(nameof(GetProduct), new { id = createdProduct.Id }, createdProduct);
    }
}
```

### **5. Implement Models and DTOs**
```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    // Domain properties
}

public class ProductDto
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    // DTO properties for client consumption
}

public class CreateProductDto
{
    [Required]
    [StringLength(100)]
    public string Name { get; set; }
    
    [Range(0.01, 10000)]
    public decimal Price { get; set; }
    // Properties for creation
}
```

### **6. Implement Services and Repositories**
```csharp
public class ProductService : IProductService
{
    private readonly IProductRepository _repository;
    private readonly IMapper _mapper;
    
    public ProductService(IProductRepository repository, IMapper mapper)
    {
        _repository = repository;
        _mapper = mapper;
    }
    
    public async Task<IEnumerable<ProductDto>> GetProductsAsync()
    {
        var products = await _repository.GetAllAsync();
        return _mapper.Map<IEnumerable<ProductDto>>(products);
    }
    
    // Other implementation methods
}
```

### **7. Configure API Behaviors**
```csharp
// Program.cs
builder.Services.AddControllers()
    .ConfigureApiBehaviorOptions(options =>
    {
        options.SuppressModelStateInvalidFilter = false;
        options.InvalidModelStateResponseFactory = context =>
        {
            var problemDetails = new ValidationProblemDetails(context.ModelState)
            {
                Type = "https://example.com/validation-error",
                Title = "Validation error",
                Status = StatusCodes.Status400BadRequest
            };
            
            return new BadRequestObjectResult(problemDetails);
        };
    });
```

### **8. Add Cross-Cutting Concerns**
```csharp
// Exception handling middleware
app.UseMiddleware<ExceptionHandlingMiddleware>();

// API versioning
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true;
});

// Authentication
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options => 
    {
        // JWT configuration
    });
```

---

## 🧠 **Interview Questions & Answers (Architect Level)**

### **1. What are the key architectural considerations when designing a .NET Core Web API?**

**Answer:**
As an architect, I focus on several critical areas:

- **Separation of Concerns**: Implementing a clean architecture with domain, application, and infrastructure layers to maintain a loosely coupled design.
  
- **API Design Patterns**: Following REST principles or considering GraphQL for complex data requirements. For RESTful APIs, I ensure proper resource naming, HTTP verb usage, and status code responses.

- **Security Architecture**: Implementing a defense-in-depth approach with JWT authentication, authorization policies, CORS configurations, and proper handling of secrets and sensitive data.

- **Performance Considerations**: Implementing response caching, compression, asynchronous programming patterns, and efficient data access strategies.

- **Observability**: Designing comprehensive logging, metrics collection, and distributed tracing to ensure operational visibility.

- **Scalability**: Designing stateless services, implementing proper data partitioning strategies, and considering eventual consistency models for distributed data.

- **Versioning Strategy**: Implementing URL, query string, or header-based versioning to support API evolution without breaking client integrations.

- **Documentation and Discoverability**: Using Swagger/OpenAPI for self-documenting APIs and providing developer-friendly documentation.

I would evaluate these considerations against business requirements, existing systems, and team capabilities to design the most appropriate architecture.

---

### **2. How do you implement proper error handling in a .NET Core Web API?**

**Answer:**
For enterprise-grade Web APIs, I implement a multi-layered error handling strategy:

1. **Global Exception Handling**: Custom middleware to catch unhandled exceptions:
```csharp
public class ExceptionHandlingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<ExceptionHandlingMiddleware> _logger;

    public ExceptionHandlingMiddleware(RequestDelegate next, ILogger<ExceptionHandlingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Unhandled exception");
            await HandleExceptionAsync(context, ex);
        }
    }

    private static Task HandleExceptionAsync(HttpContext context, Exception exception)
    {
        context.Response.ContentType = "application/problem+json";
        
        var statusCode = exception switch
        {
            NotFoundException => StatusCodes.Status404NotFound,
            ValidationException => StatusCodes.Status400BadRequest,
            UnauthorizedAccessException => StatusCodes.Status401Unauthorized,
            _ => StatusCodes.Status500InternalServerError
        };

        context.Response.StatusCode = statusCode;

        var problem = new ProblemDetails
        {
            Status = statusCode,
            Title = GetTitle(exception),
            Detail = exception.Message,
            Instance = context.Request.Path
        };

        // Add custom properties for specific exceptions
        if (exception is ValidationException validationException)
        {
            problem.Extensions["errors"] = validationException.Errors;
        }

        return context.Response.WriteAsJsonAsync(problem);
    }

    private static string GetTitle(Exception exception) =>
        exception switch
        {
            NotFoundException => "Resource not found",
            ValidationException => "Validation error",
            UnauthorizedAccessException => "Unauthorized",
            _ => "An unexpected error occurred"
        };
}
```

2. **Domain-Specific Exceptions**: Custom exception types that map to appropriate HTTP status codes:
```csharp
public class NotFoundException : Exception
{
    public NotFoundException(string message) : base(message) { }
}

public class ValidationException : Exception
{
    public IDictionary<string, string[]> Errors { get; }
    
    public ValidationException(IDictionary<string, string[]> errors) 
        : base("Validation failure")
    {
        Errors = errors;
    }
}
```

3. **Standardized Error Format**: Using RFC 7807 Problem Details standard:
```json
{
  "type": "https://example.com/errors/validation-error",
  "title": "Validation error",
  "status": 400,
  "detail": "The provided data failed validation",
  "instance": "/api/products",
  "errors": {
    "Name": ["Name is required"],
    "Price": ["Price must be greater than zero"]
  }
}
```

4. **Contextual Logging**: Enriching logs with request context and correlation IDs.

5. **Circuit Breakers**: For external service calls to prevent cascading failures.

This approach ensures consistent error responses, proper status codes, detailed error information for clients (without exposing sensitive details), and comprehensive logging for operational support.

---

### **3. Explain the middleware pipeline in ASP.NET Core and how you would architect it for a production API.**

**Answer:**
The ASP.NET Core middleware pipeline is a sequence of request processing components that handle requests and responses in order. Each component can:
- Process incoming requests
- Invoke the next middleware in the pipeline
- Process outgoing responses after the next middleware has completed

For a production-ready Web API, I architect the middleware pipeline with these considerations:

1. **Order Matters**: Middlewares execute in the order they're added, with careful placement for dependencies and logical flow.

2. **Production Middleware Architecture**:
```csharp
// Program.cs in .NET 6+
var app = builder.Build();

// 1. Early termination for non-essential paths
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}
else
{
    // 2. Security headers
    app.UseHsts();
}

// 3. Exception handling - must be early to catch exceptions in other middleware
app.UseMiddleware<ExceptionHandlingMiddleware>();

// 4. Request logging and telemetry
app.UseRequestLogging();
app.UseCorrelationId();

// 5. Basic infrastructure
app.UseHttpsRedirection();
app.UseStaticFiles();

// 6. Routing (must precede endpoint-bound middleware)
app.UseRouting();

// 7. CORS - after routing, before auth
app.UseCors("ApiPolicy");

// 8. Authentication and Authorization
app.UseAuthentication();
app.UseAuthorization();

// 9. Feature middleware
app.UseRateLimiting();
app.UseResponseCaching();
app.UseResponseCompression();

// 10. API request localization
app.UseRequestLocalization();

// 11. Endpoint execution
app.MapControllers();
app.Run();
```

3. **Custom Middleware for Cross-Cutting Concerns**:
   - **Request/Response Logging**: For API observability and diagnostics
   - **Correlation ID**: For request tracking across services
   - **API Key Validation**: For client identification
   - **Rate Limiting**: For resource protection
   - **Tenant Resolution**: For multi-tenant scenarios

4. **Conditional Middleware**:
   - Environment-specific middleware (e.g., Swagger only in development)
   - Feature-flag controlled middleware for gradual rollout

5. **Short-Circuit Patterns**:
```csharp
app.Use(async (context, next) =>
{
    // Pre-processing
    if (ShouldShortCircuit(context))
    {
        // Short-circuit the pipeline
        context.Response.StatusCode = 503;
        return;
    }
    
    // Continue to next middleware
    await next();
    
    // Post-processing
});
```

6. **Performance Considerations**:
   - Limit middleware to what's necessary
   - Place frequently short-circuiting middleware early in the pipeline
   - Use Branch middleware for path-specific handling
   
This architecture ensures a clean request flow, proper error handling, security, and observability while maintaining performance.

---

### **4. How would you design API versioning for a .NET Core Web API?**

**Answer:**
API versioning is critical for evolving APIs without breaking existing clients. As an architect, I implement a comprehensive versioning strategy with these components:

1. **Versioning Approach Selection**:

   I typically prefer **URL path versioning** for public APIs due to its clarity and client compatibility:
   ```
   /api/v1/products
   /api/v2/products
   ```
   
   For internal microservices, I often use **header-based versioning**:
   ```
   GET /api/products
   X-API-Version: 2.0
   ```

2. **Implementation in .NET Core**:
```csharp
// Program.cs
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true;
    options.ApiVersionReader = ApiVersionReader.Combine(
        new UrlSegmentApiVersionReader(),
        new HeaderApiVersionReader("X-API-Version"),
        new QueryStringApiVersionReader("api-version")
    );
});

builder.Services.AddVersionedApiExplorer(options =>
{
    options.GroupNameFormat = "'v'VVV";
    options.SubstituteApiVersionInUrl = true;
});
```

3. **Controller Versioning**:
```csharp
[ApiVersion("1.0")]
[Route("api/v{version:apiVersion}/products")]
public class ProductsV1Controller : ControllerBase
{
    // V1 implementation
}

[ApiVersion("2.0")]
[Route("api/v{version:apiVersion}/products")]
public class ProductsV2Controller : ControllerBase
{
    // V2 implementation
}

// Or with the same controller supporting multiple versions
[ApiController]
[Route("api/v{version:apiVersion}/products")]
[ApiVersion("1.0")]
[ApiVersion("2.0")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    [MapToApiVersion("1.0")]
    public IActionResult GetV1() { /* V1 implementation */ }

    [HttpGet]
    [MapToApiVersion("2.0")]
    public IActionResult GetV2() { /* V2 implementation */ }
}
```

4. **Versioning Strategy**:
   - **Semantic Versioning**: Using Major.Minor.Patch
   - **Major version changes**: Breaking changes requiring client updates
   - **Minor version changes**: Backward-compatible additions
   - **Patch changes**: Bug fixes, no API change

5. **Documentation**:
```csharp
// Configure Swagger for versions
builder.Services.AddSwaggerGen();
builder.Services.ConfigureOptions<ConfigureSwaggerOptions>();

// Separate Swagger docs for each API version
public class ConfigureSwaggerOptions : IConfigureNamedOptions<SwaggerGenOptions>
{
    private readonly IApiVersionDescriptionProvider _provider;

    public ConfigureSwaggerOptions(IApiVersionDescriptionProvider provider)
    {
        _provider = provider;
    }

    public void Configure(SwaggerGenOptions options)
    {
        foreach (var description in _provider.ApiVersionDescriptions)
        {
            options.SwaggerDoc(
                description.GroupName,
                CreateVersionInfo(description));
        }
    }
    
    public void Configure(string name, SwaggerGenOptions options)
    {
        Configure(options);
    }

    private OpenApiInfo CreateVersionInfo(ApiVersionDescription description)
    {
        var info = new OpenApiInfo
        {
            Title = "My API",
            Version = description.ApiVersion.ToString()
        };

        if (description.IsDeprecated)
        {
            info.Description += " This API version has been deprecated.";
        }

        return info;
    }
}
```

6. **Deprecation and Sunsetting**:
   - Marking versions as deprecated
   - Warning headers for deprecated versions
   - Clear documentation on end-of-life dates
   - Monitoring usage of deprecated versions

This approach provides flexibility for clients while allowing the API to evolve, with clear documentation and migration paths.

---

### **5. How do you ensure security in a .NET Core Web API?**

**Answer:**
Security in Web APIs requires a defense-in-depth strategy. Here's how I architect security for .NET Core Web APIs:

1. **Authentication**:
   - **JWT Bearer Authentication**:
   ```csharp
   builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
       .AddJwtBearer(options =>
       {
           options.TokenValidationParameters = new TokenValidationParameters
           {
               ValidateIssuer = true,
               ValidateAudience = true,
               ValidateLifetime = true,
               ValidateIssuerSigningKey = true,
               ValidIssuer = builder.Configuration["Jwt:Issuer"],
               ValidAudience = builder.Configuration["Jwt:Audience"],
               IssuerSigningKey = new SymmetricSecurityKey(
                   Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]))
           };
       });
   ```
   
   - **OAuth 2.0/OpenID Connect** for enterprise scenarios:
   ```csharp
   builder.Services.AddAuthentication(options =>
   {
       options.DefaultScheme = JwtBearerDefaults.AuthenticationScheme;
       options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
   })
   .AddJwtBearer(options =>
   {
       options.Authority = "https://login.microsoftonline.com/{tenant-id}/v2.0";
       options.Audience = "{client-id}";
   });
   ```

2. **Authorization**:
   - **Policy-based authorization**:
   ```csharp
   builder.Services.AddAuthorization(options =>
   {
       options.AddPolicy("AdminOnly", policy => 
           policy.RequireRole("Admin"));
       
       options.AddPolicy("EditProduct", policy =>
           policy.RequireClaim("Permission", "Products.Edit"));
           
       options.AddPolicy("PremiumTier", policy =>
           policy.Requirements.Add(new SubscriptionTierRequirement("Premium")));
   });
   
   // Register handler for custom requirements
   builder.Services.AddSingleton<IAuthorizationHandler, SubscriptionTierHandler>();
   ```
   
   - **Resource-based authorization**:
   ```csharp
   [HttpPut("{id}")]
   public async Task<IActionResult> UpdateProduct(int id, UpdateProductDto dto)
   {
       var product = await _repository.GetByIdAsync(id);
       if (product == null) return NotFound();
       
       // Resource-based authorization check
       var authResult = await _authorizationService.AuthorizeAsync(
           User, product, "ProductOwnerOrAdmin");
           
       if (!authResult.Succeeded)
           return Forbid();
       
       // Update logic
   }
   ```

3. **Data Protection**:
   - **Sensitive Data Encryption**:
   ```csharp
   builder.Services.AddDataProtection()
       .PersistKeysToAzureKeyVault(new Uri("https://myvault.vault.azure.net/keys/mydataprotectionkey"));
   ```
   
   - **Personally Identifiable Information (PII) handling**:
   ```csharp
   // Using data annotation to mark sensitive fields
   public class UserDto
   {
       public int Id { get; set; }
       
       public string Name { get; set; }
       
       [Sensitive]
       [JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingNull)]
       public string Email { get; set; }
   }
   ```

4. **Transport Security**:
   - **HTTPS Enforcement**:
   ```csharp
   // In Program.cs
   app.UseHttpsRedirection();
   
   // In production environments
   app.UseHsts();
   ```
   
   - **CORS Configuration**:
   ```csharp
   builder.Services.AddCors(options =>
   {
       options.AddPolicy("ApiPolicy", policy =>
       {
           policy.WithOrigins("https://trusted-client.com")
                 .WithMethods("GET", "POST", "PUT", "DELETE")
                 .WithHeaders("Authorization", "Content-Type")
                 .AllowCredentials();
       });
   });
   
   // Apply CORS middleware
   app.UseCors("ApiPolicy");
   ```

5. **Input Validation**:
   - **Model Validation**:
   ```csharp
   public class CreateProductDto
   {
       [Required]
       [StringLength(100, MinimumLength = 3)]
       public string Name { get; set; }
       
       [Range(0.01, 10000)]
       public decimal Price { get; set; }
       
       [RegularExpression(@"^[A-Z]{2}-\d{4}$")]
       public string ProductCode { get; set; }
   }
   ```
   
   - **Content Security and Sanitization**:
   ```csharp
   builder.Services.AddControllers(options =>
   {
       options.Filters.Add<ValidateModelFilter>();
   });
   
   // Add HTML sanitization for user-generated content
   builder.Services.AddSingleton<IHtmlSanitizer, HtmlSanitizer>();
   ```

6. **API Security Controls**:
   - **Rate Limiting**:
   ```csharp
   builder.Services.AddRateLimiter(options =>
   {
       options.GlobalLimiter = PartitionedRateLimiter.Create<HttpContext, string>(context =>
       {
           return RateLimitPartition.GetFixedWindowLimiter(
               partitionKey: context.User.Identity?.Name ?? context.Connection.RemoteIpAddress?.ToString() ?? "anonymous",
               factory: partition => new FixedWindowRateLimiterOptions
               {
                   AutoReplenishment = true,
                   PermitLimit = 100,
                   Window = TimeSpan.FromMinutes(1)
               });
       });
       options.RejectionStatusCode = StatusCodes.Status429TooManyRequests;
   });
   ```
   
   - **API Key Validation**:
   ```csharp
   app.UseMiddleware<ApiKeyAuthMiddleware>();
   ```

7. **Sensitive Data Handling**:
   - **Secrets Management**:
   ```csharp
   // Program.cs
   builder.Configuration.AddAzureKeyVault(
       new Uri($"https://{builder.Configuration["KeyVault:Name"]}.vault.azure.net/"),
       new DefaultAzureCredential());
   ```
   
   - **Logging Sanitization**:
   ```csharp
   builder.Services.AddSingleton<ILoggerProvider>(sp =>
   {
       return new SanitizingLoggerProvider(
           new FileLoggerProvider("logs/api.log"),
           new[] { "password", "token", "ssn", "creditcard" });
   });
   ```

8. **Security Headers**:
   ```csharp
   app.Use(async (context, next) =>
   {
       context.Response.Headers.Add("X-Content-Type-Options", "nosniff");
       context.Response.Headers.Add("X-Frame-Options", "DENY");
       context.Response.Headers.Add("Content-Security-Policy", "default-src 'self'");
       context.Response.Headers.Add("X-XSS-Protection", "1; mode=block");
       context.Response.Headers.Add("Referrer-Policy", "strict-origin-when-cross-origin");
       await next();
   });
   ```

This multi-layered approach addresses authentication, authorization, data protection, transport security, input validation, API security, and compliance requirements, creating a robust security architecture.

---

### **6. What patterns would you use for data access in a .NET Core Web API?**

**Answer:**
For data access in .NET Core Web APIs, I implement a layered approach with patterns that support maintainability, testability, and performance:

1. **Repository Pattern**:
   - **Abstracts data access logic** from business logic
   - **Enables unit testing** via repository interfaces
   - **Implementation**:
   
   ```csharp
   public interface IRepository<T> where T : class
   {
       Task<T> GetByIdAsync(int id);
       Task<IReadOnlyList<T>> ListAllAsync();
       Task<T> AddAsync(T entity);
       Task UpdateAsync(T entity);
       Task DeleteAsync(T entity);
   }
   
   public class EfRepository<T> : IRepository<T> where T : class
   {
       private readonly ApplicationDbContext _context;
       
       public EfRepository(ApplicationDbContext context)
       {
           _context = context;
       }
       
       public async Task<T> GetByIdAsync(int id)
       {
           return await _context.Set<T>().FindAsync(id);
       }
       
       // Other implementations
   }
   ```

2. **Unit of Work**:
   - **Manages transactions** across multiple repositories
   - **Ensures consistency** for multi-entity operations
   
   ```csharp
   public interface IUnitOfWork : IDisposable
   {
       IRepository<Product> Products { get; }
       IRepository<Order> Orders { get; }
       Task<int> CompleteAsync();
   }
   
   public class UnitOfWork : IUnitOfWork
   {
       private readonly ApplicationDbContext _context;
       
       public UnitOfWork(ApplicationDbContext context)
       {
           _context = context;
           Products = new EfRepository<Product>(_context);
           Orders = new EfRepository<Order>(_context);
       }
       
       public IRepository<Product> Products { get; }
       public IRepository<Order> Orders { get; }
       
       public async Task<int> CompleteAsync()
       {
           return await _context.SaveChangesAsync();
       }
       
       public void Dispose()
       {
           _context.Dispose();
       }
   }
   ```

3. **Specification Pattern**:
   - **Encapsulates query logic** for complex filtering, sorting, and includes
   - **Reusable query specifications** across the application
   
   ```csharp
   public interface ISpecification<T>
   {
       Expression<Func<T, bool>> Criteria { get; }
       List<Expression<Func<T, object>>> Includes { get; }
       Expression<Func<T, object>> OrderBy { get; }
       Expression<Func<T, object>> OrderByDescending { get; }
       int Take { get; }
       int Skip { get; }
       bool IsPagingEnabled { get; }
   }
   
   public class ProductsWithCategoriesSpecification : BaseSpecification<Product>
   {
       public ProductsWithCategoriesSpecification(ProductSpecParams parameters) 
           : base(x => 
               (string.IsNullOrEmpty(parameters.Search) || x.Name.Contains(parameters.Search)) &&
               (!parameters.CategoryId.HasValue || x.CategoryId == parameters.CategoryId))
       {
           AddInclude(x => x.Category);
           AddOrderBy(x => x.Name);
           ApplyPaging(parameters.PageSize * (parameters.PageIndex - 1), parameters.PageSize);
           
           if (!string.IsNullOrEmpty(parameters.Sort))
           {
               switch (parameters.Sort)
               {
                   case "priceAsc":
                       AddOrderBy(p => p.Price);
                       break;
                   case "priceDesc":
                       AddOrderByDescending(p => p.Price);
                       break;
                   default:
                       AddOrderBy(n => n.Name);
                       break;
               }
           }
       }
   }
   ```

4. **CQRS (Command Query Responsibility Segregation)**:
   - **Separates read and write operations** for optimized data access
   - **Typically implemented with Mediator pattern**
   
   ```csharp
   // Query side
   public record GetProductByIdQuery(int Id) : IRequest<ProductDto>;
   
   public class GetProductByIdHandler : IRequestHandler<GetProductByIdQuery, ProductDto>
   {
       private readonly IReadDbContext _readContext;
       private readonly IMapper _mapper;
       
       public GetProductByIdHandler(IReadDbContext readContext, IMapper mapper)
       {
           _readContext = readContext;
           _mapper = mapper;
       }
       
       public async Task<ProductDto> Handle(GetProductByIdQuery request, CancellationToken cancellationToken)
       {
           var product = await _readContext.Products
               .AsNoTracking()
               .Include(p => p.Category)
               .FirstOrDefaultAsync(p => p.Id == request.Id, cancellationToken);
               
           return _mapper.Map<ProductDto>(product);
       }
   }
   
   // Command side
   public record UpdateProductCommand(int Id, string Name, decimal Price) : IRequest<Unit>;
   
   public class UpdateProductCommandHandler : IRequestHandler<UpdateProductCommand, Unit>
   {
       private readonly IWriteDbContext _writeContext;
       
       public UpdateProductCommandHandler(IWriteDbContext writeContext)
       {
           _writeContext = writeContext;
       }
       
       public async Task<Unit> Handle(UpdateProductCommand request, CancellationToken cancellationToken)
       {
           var product = await _writeContext.Products.FindAsync(request.Id);
           if (product == null) throw new NotFoundException(nameof(Product), request.Id);
           
           product.Name = request.Name;
           product.Price = request.Price;
           
           await _writeContext.SaveChangesAsync(cancellationToken);
           return Unit.Value;
       }
   }
   ```

5. **Optimized Data Access Techniques**:
   - **Projections** for retrieving only necessary fields:
   ```csharp
   public async Task<IEnumerable<ProductSummaryDto>> GetProductSummariesAsync()
   {
       return await _context.Products
           .AsNoTracking()
           .Select(p => new ProductSummaryDto
           {
               Id = p.Id,
               Name = p.Name,
               Price = p.Price
           })
           .ToListAsync();
   }
   ```
   
   - **Compiled Queries** for frequent operations:
   ```csharp
   private static readonly Func<ApplicationDbContext, int, Task<Product>> GetProductByIdQuery =
       EF.CompileAsyncQuery((ApplicationDbContext context, int id) =>
           context.Products.Include(p => p.Category)
                  .FirstOrDefault(p => p.Id == id));
   
   public async Task<Product> GetProductByIdAsync(int id)
   {
       return await GetProductByIdQuery(_context, id);
   }
   ```
   
   - **Efficient Paging** with optimized SQL:
   ```csharp
   public async Task<PagedResult<ProductDto>> GetPagedProductsAsync(int page, int pageSize)
   {
       var totalItems = await _context.Products.CountAsync();
       
       var products = await _context.Products
           .AsNoTracking()
           .OrderBy(p => p.Name)
           .Skip((page - 1) * pageSize)
           .Take(pageSize)
           .Select(p => new ProductDto
           {
               Id = p.Id,
               Name = p.Name,
               Price = p.Price,
               CategoryName = p.Category.Name
           })
           .ToListAsync();
           
       return new PagedResult<ProductDto>
       {
           Items = products,
           TotalItems = totalItems,
           Page = page,
           PageSize = pageSize
       };
   }
   ```

6. **Performance Considerations**:
   - **Appropriate use of `AsNoTracking()`** for read-only queries
   - **Eager loading with `Include()`** to avoid N+1 query problems
   - **Bulk operations** for performance-critical scenarios:
   ```csharp
   public async Task BulkInsertProductsAsync(IEnumerable<Product> products)
   {
       // Using EF Core's AddRange
       _context.Products.AddRange(products);
       await _context.SaveChangesAsync();
       
       // Or with third-party libraries for true bulk operations
       await _context.BulkInsertAsync(products);
   }
   ```

This architecture provides a flexible, maintainable data access layer that can evolve with the application's needs while maintaining performance.

---

### **7. How would you design the API endpoints in a RESTful .NET Core Web API?**

**Answer:**
Designing RESTful API endpoints requires careful consideration of resource modeling, HTTP semantics, and client usability. Here's my approach as an architect:

1. **Resource-Centric URL Design**:
   - Nouns, not verbs in URLs
   - Collection/item hierarchy
   - Consistent pluralization
   
   Examples:
   ```
   /api/products              # Collection
   /api/products/{id}         # Specific item
   /api/customers/{id}/orders # Nested collection
   ```

2. **HTTP Method Usage**:
   ```csharp
   [HttpGet]                      // GET /api/products - List all products
   [HttpGet("{id}")]              // GET /api/products/5 - Get product with ID 5
   [HttpPost]                     // POST /api/products - Create new product
   [HttpPut("{id}")]              // PUT /api/products/5 - Replace product with ID 5
   [HttpPatch("{id}")]            // PATCH /api/products/5 - Partially update product
   [HttpDelete("{id}")]           // DELETE /api/products/5 - Delete product
   [HttpGet("{id}/reviews")]      // GET /api/products/5/reviews - List reviews for product
   ```

3. **Controller Design**:
```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IProductService _productService;
    
    public ProductsController(IProductService productService)
    {
        _productService = productService;
    }
    
    // Collection endpoints
    [HttpGet]
    public async Task<ActionResult<PagedResult<ProductDto>>> GetProducts(
        [FromQuery] ProductParameters parameters)
    {
        return Ok(await _productService.GetProductsAsync(parameters));
    }
    
    [HttpPost]
    public async Task<ActionResult<ProductDto>> CreateProduct(
        [FromBody] CreateProductDto productDto)
    {
        var result = await _productService.CreateProductAsync(productDto);
        return CreatedAtAction(nameof(GetProduct), new { id = result.Id }, result);
    }
    
    // Item endpoints
    [HttpGet("{id}")]
    public async Task<ActionResult<ProductDto>> GetProduct(int id)
    {
        var product = await _productService.GetProductByIdAsync(id);
        if (product == null) return NotFound();
        
        return Ok(product);
    }
    
    [HttpPut("{id}")]
    public async Task<IActionResult> UpdateProduct(
        int id, [FromBody] UpdateProductDto productDto)
    {
        if (id != productDto.Id) return BadRequest();
        
        await _productService.UpdateProductAsync(productDto);
        return NoContent();
    }
    
    [HttpDelete("{id}")]
    public async Task<IActionResult> DeleteProduct(int id)
    {
        await _productService.DeleteProductAsync(id);
        return NoContent();
    }
    
    // Related resource endpoints
    [HttpGet("{id}/reviews")]
    public async Task<ActionResult<IEnumerable<ReviewDto>>> GetProductReviews(int id)
    {
        var reviews = await _productService.GetProductReviewsAsync(id);
        return Ok(reviews);
    }
}
```

4. **Query Parameter Handling**:
```csharp
// Parameter object for collection filtering, sorting, and pagination
public class ProductParameters
{
    // Paging
    private const int MaxPageSize = 50;
    private int _pageSize = 10;
    
    public int PageIndex { get; set; } = 1;
    
    public int PageSize
    {
        get => _pageSize;
        set => _pageSize = Math.Min(value, MaxPageSize);
    }
    
    // Filtering
    public decimal? MinPrice { get; set; }
    public decimal? MaxPrice { get; set; }
    public int? CategoryId { get; set; }
    
    // Searching
    public string Search { get; set; }
    
    // Sorting
    public string SortBy { get; set; } = "name";
    public string SortDirection { get; set; } = "asc";
}

// Usage in controller
[HttpGet]
public async Task<ActionResult<PagedResult<ProductDto>>> GetProducts(
    [FromQuery] ProductParameters parameters)
{
    return Ok(await _productService.GetProductsAsync(parameters));
}
```

5. **HTTP Status Codes**:
```csharp
// Successful responses
return Ok(products);                     // 200 - General success
return Created($"/api/products/{id}", product); // 201 - Resource created
return NoContent();                      // 204 - Success with no response body

// Client errors
return BadRequest(ModelState);           // 400 - Invalid request format
return Unauthorized();                   // 401 - Authentication required
return Forbid();                         // 403 - Authenticated but not authorized
return NotFound();                       // 404 - Resource not found
return Conflict("Product code already exists"); // 409 - Resource conflict
return UnprocessableEntity(errors);      // 422 - Validation errors

// Server errors
return StatusCode(StatusCodes.Status500InternalServerError, 
    "An unexpected error occurred");      // 500 - Server error
```

6. **Response Envelope Patterns**:
```csharp
// Paginated response
public class PagedResult<T>
{
    public IEnumerable<T> Items { get; set; }
    public int TotalItems { get; set; }
    public int PageIndex { get; set; }
    public int PageSize { get; set; }
    public int TotalPages => (int)Math.Ceiling(TotalItems / (double)PageSize);
    public bool HasPreviousPage => PageIndex > 1;
    public bool HasNextPage => PageIndex < TotalPages;
}

// Response with metadata
public class ApiResponse<T>
{
    public T Data { get; set; }
    public bool Success { get; set; }
    public string Message { get; set; }
    public object Meta { get; set; }
}
```

7. **HATEOAS Implementation** (Hypermedia as the Engine of Application State):
```csharp
public class ProductWithLinksDto
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public List<Link> Links { get; set; } = new();
}

public class Link
{
    public string Href { get; set; }
    public string Rel { get; set; }
    public string Method { get; set; }
    
    public Link(string href, string rel, string method)
    {
        Href = href;
        Rel = rel;
        Method = method;
    }
}

// In controller
[HttpGet("{id}")]
public async Task<ActionResult<ProductWithLinksDto>> GetProduct(int id)
{
    var product = await _productService.GetProductByIdAsync(id);
    if (product == null) return NotFound();
    
    var result = _mapper.Map<ProductWithLinksDto>(product);
    
    // Add links to related resources/actions
    result.Links.Add(
        new Link(
            Url.Link(nameof(GetProduct), new { id = product.Id }),
            "self",
            "GET"));
            
    result.Links.Add(
        new Link(
            Url.Link(nameof(UpdateProduct), new { id = product.Id }),
            "update_product",
            "PUT"));
            
    result.Links.Add(
        new Link(
            Url.Link(nameof(DeleteProduct), new { id = product.Id }),
            "delete_product",
            "DELETE"));
            
    result.Links.Add(
        new Link(
            Url.Link(nameof(GetProductReviews), new { id = product.Id }),
            "product_reviews",
            "GET"));
            
    return Ok(result);
}
```

8. **Content Negotiation**:
```csharp
// Configure in Program.cs
builder.Services.AddControllers(options =>
{
    options.RespectBrowserAcceptHeader = true;
    options.ReturnHttpNotAcceptable = true;
})
.AddXmlDataContractSerializerFormatters()
.AddJsonOptions(options =>
{
    options.JsonSerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase;
    options.JsonSerializerOptions.WriteIndented = true;
});
```

This design approach creates a consistent, intuitive API that follows RESTful principles while providing rich functionality to clients.

---

### **8. How do you approach performance optimization in a .NET Core Web API?**

**Answer:**
Performance optimization for .NET Core Web APIs requires a systematic approach across multiple layers. Here's my comprehensive strategy as an architect:

1. **Measurement and Benchmarking**:
   - **Establish Baselines**: Use tools like Apache JMeter, k6, or Azure Load Testing
   - **Profiling**: Use dotTrace, Visual Studio Profiler, or Application Insights
   - **Logging Performance Metrics**:
   ```csharp
   app.Use(async (context, next) =>
   {
       var sw = Stopwatch.StartNew();
       
       try
       {
           await next();
       }
       finally
       {
           sw.Stop();
           var elapsed = sw.ElapsedMilliseconds;
           
           // Log slow requests
           if (elapsed > 500)
           {
               _logger.LogWarning(
                   "Slow request: {Method} {Path} took {ElapsedMs}ms",
                   context.Request.Method,
                   context.Request.Path,
                   elapsed);
           }
           
           // Add response timing header in development
           if (_env.IsDevelopment())
           {
               context.Response.Headers.Add("X-Response-Time-ms", elapsed.ToString());
           }
       }
   });
   ```

2. **Database Optimization**:
   - **Efficient Queries**:
   ```csharp
   // Use AsNoTracking for read-only queries
   var products = await _context.Products
       .AsNoTracking()
       .Where(p => p.CategoryId == categoryId)
       .ToListAsync();
       
   // Select only necessary columns
   var productNames = await _context.Products
       .Where(p => p.IsActive)
       .Select(p => p.Name)
       .ToListAsync();
   ```
   
   - **Query Caching**:
   ```csharp
   // Using LazyCache for query results
   public async Task<List<ProductDto>> GetProductsByCategoryAsync(int categoryId)
   {
       return await _cache.GetOrAddAsync(
           $"products_by_category_{categoryId}",
           async () => 
           {
               var products = await _context.Products
                   .AsNoTracking()
                   .Where(p => p.CategoryId == categoryId)
                   .ToListAsync();
                   
               return _mapper.Map<List<ProductDto>>(products);
           },
           TimeSpan.FromMinutes(10));
   }
   ```
   
   - **Bulk Operations**:
   ```csharp
   // For large data operations
   public async Task ImportProductsAsync(List<ProductImportDto> products)
   {
       var entities = _mapper.Map<List<Product>>(products);
       
       // Using EF Core or third-party library for bulk insert
       await _context.BulkInsertAsync(entities);
   }
   ```
   
   - **Pagination**:
   ```csharp
   [HttpGet]
   public async Task<ActionResult<PagedResult<ProductDto>>> GetProducts(
       [FromQuery] int page = 1, 
       [FromQuery] int pageSize = 10)
   {
       var result = await _productService.GetPagedProductsAsync(page, pageSize);
       return Ok(result);
   }
   ```

3. **Caching Strategies**:
   - **Response Caching**:
   ```csharp
   // In Program.cs
   builder.Services.AddResponseCaching();
   
   // In middleware pipeline
   app.UseResponseCaching();
   
   // On controller or action
   [HttpGet]
   [ResponseCache(Duration = 60, Location = ResponseCacheLocation.Any, VaryByQueryKeys = new[] { "categoryId" })]
   public async Task<ActionResult<List<ProductDto>>> GetProductsByCategory(int categoryId)
   {
       // Implementation
   }
   ```
   
   - **Distributed Caching** with Redis:
   ```csharp
   // In Program.cs
   builder.Services.AddStackExchangeRedisCache(options =>
   {
       options.Configuration = builder.Configuration.GetConnectionString("Redis");
       options.InstanceName = "SampleApp_";
   });
   
   // Usage in service
   public async Task<ProductDto> GetProductByIdAsync(int id)
   {
       var cacheKey = $"product_{id}";
       
       // Try to get from cache
       var cachedProduct = await _distributedCache.GetStringAsync(cacheKey);
       if (!string.IsNullOrEmpty(cachedProduct))
       {
           return JsonSerializer.Deserialize<ProductDto>(cachedProduct);
       }
       
       // If not in cache, get from database
       var product = await _context.Products.FindAsync(id);
       if (product == null) return null;
       
       var result = _mapper.Map<ProductDto>(product);
       
       // Store in cache
       var cacheOptions = new DistributedCacheEntryOptions
       {
           AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10),
           SlidingExpiration = TimeSpan.FromMinutes(2)
       };
       
       await _distributedCache.SetStringAsync(
           cacheKey,
           JsonSerializer.Serialize(result),
           cacheOptions);
           
       return result;
   }
   ```
   
   - **Memory Caching** for frequently accessed data:
   ```csharp
   private readonly IMemoryCache _memoryCache;
   
   public ProductService(ApplicationDbContext context, IMapper mapper, IMemoryCache memoryCache)
   {
       _context = context;
       _mapper = mapper;
       _memoryCache = memoryCache;
   }
   
   public async Task<List<CategoryDto>> GetAllCategoriesAsync()
   {
       if (_memoryCache.TryGetValue("all_categories", out List<CategoryDto> categories))
       {
           return categories;
       }
       
       categories = await _context.Categories
           .AsNoTracking()
           .OrderBy(c => c.Name)
           .Select(c => new CategoryDto
           {
               Id = c.Id,
               Name = c.Name
           })
           .ToListAsync();
           
       _memoryCache.Set("all_categories", categories, TimeSpan.FromHours(1));
       
       return categories;
   }
   ```
   
   - **Cache Invalidation**:
   ```csharp
   public async Task UpdateProductAsync(UpdateProductDto productDto)
   {
       // Update in database
       var product = await _context.Products.FindAsync(productDto.Id);
       if (product == null) throw new NotFoundException(nameof(Product), productDto.Id);
       
       _mapper.Map(productDto, product);
       await _context.SaveChangesAsync();
       
       // Invalidate cache
       await _distributedCache.RemoveAsync($"product_{productDto.Id}");
       await _distributedCache.RemoveAsync($"products_by_category_{product.CategoryId}");
   }
   ```

4. **Optimizing Controllers and Actions**:
   - **Async/Await** for I/O operations:
   ```csharp
   [HttpGet("{id}")]
   public async Task<ActionResult<ProductDto>> GetProduct(int id)
   {
       var product = await _productService.GetProductByIdAsync(id);
       if (product == null) return NotFound();
       
       return Ok(product);
   }
   ```
   
   - **Minimize Serialization Overhead**:
   ```csharp
   // In Program.cs
   builder.Services.AddControllers()
       .AddJsonOptions(options =>
       {
           options.JsonSerializerOptions.DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull;
           options.JsonSerializerOptions.Converters.Add(new JsonStringEnumConverter());
       });
   ```
   
   - **Output Caching**:
   ```csharp
   // In Program.cs
   builder.Services.AddOutputCache(options =>
   {
       options.AddBasePolicy(builder => 
           builder.Expire(TimeSpan.FromMinutes(5)));
       
       options.AddPolicy("ProductsList", builder => 
           builder.Tag("products-list")
                 .SetVaryByQuery("page", "pageSize"));
   });
   
   // In controller
   [HttpGet]
   [OutputCache(PolicyName = "ProductsList")]
   public async Task<ActionResult<PagedResult<ProductDto>>> GetProducts(
       [FromQuery] int page = 1, 
       [FromQuery] int pageSize = 10)
   {
       // Implementation
   }
   ```

5. **Compression and Network Optimizations**:
   - **Response Compression**:
   ```csharp
   // In Program.cs
   builder.Services.AddResponseCompression(options =>
   {
       options.Providers.Add<BrotliCompressionProvider>();
       options.Providers.Add<GzipCompressionProvider>();
       options.EnableForHttps = true;
   });
   
   builder.Services.Configure<BrotliCompressionProviderOptions>(options =>
   {
       options.Level = CompressionLevel.Fastest;
   });
   
   // In middleware pipeline
   app.UseResponseCompression();
   ```
   
   - **Minimize Payload Size**:
   ```csharp
   // DTOs designed for specific use cases to avoid over-fetching
   public class ProductListDto
   {
       public int Id { get; set; }
       public string Name { get; set; }
       public decimal Price { get; set; }
       // Only essential fields for listing
   }
   
   public class ProductDetailDto
   {
       public int Id { get; set; }
       public string Name { get; set; }
       public string Description { get; set; }
       public decimal Price { get; set; }
       public CategoryDto Category { get; set; }
       public List<SpecificationDto> Specifications { get; set; }
       // Full details for single product view
   }
   ```

6. **Application-Level Optimizations**:
   - **Parallel Processing** for independent operations:
   ```csharp
   public async Task<DashboardDto> GetDashboardDataAsync()
   {
       // Execute independent queries in parallel
       var recentOrdersTask = _orderRepository.GetRecentOrdersAsync();
       var topProductsTask = _productRepository.GetTopSellingProductsAsync();
       var salesSummaryTask = _reportService.GetSalesSummaryAsync();
       
       await Task.WhenAll(recentOrdersTask, topProductsTask, salesSummaryTask);
       
       return new DashboardDto
       {
           RecentOrders = await recentOrdersTask,
           TopProducts = await topProductsTask,
           SalesSummary = await salesSummaryTask
       };
   }
   ```
   
   - **Optimize Startup Time**:
   ```csharp
   // Use service fabric model for initialization
   builder.Services.AddSingleton<IProductCatalogService, ProductCatalogService>();
   builder.Services.AddHostedService<ProductCatalogInitializerService>();
   
   // Initializer service
   public class ProductCatalogInitializerService : IHostedService
   {
       private readonly IProductCatalogService _catalogService;
       
       public ProductCatalogInitializerService(IProductCatalogService catalogService)
       {
           _catalogService = catalogService;
       }
       
       public Task StartAsync(CancellationToken cancellationToken)
       {
           // Initialize in background without blocking application startup
           _ = Task.Run(async () =>
           {
               await _catalogService.InitializeAsync();
           }, cancellationToken);
           
           return Task.CompletedTask;
       }
       
       public Task StopAsync(CancellationToken cancellationToken)
       {
           return Task.CompletedTask;
       }
   }
   ```
   
   - **Background Processing** for time-consuming tasks:
   ```csharp
   // In Program.cs
   builder.Services.AddHostedService<QueuedHostedService>();
   builder.Services.AddSingleton<IBackgroundTaskQueue, BackgroundTaskQueue>();
   
   // Usage in controller
   [HttpPost("import")]
   public IActionResult StartImport(ImportRequestDto request)
   {
       // Queue background work
       _backgroundQueue.QueueBackgroundWorkItem(async token =>
       {
           await _importService.ProcessImportAsync(request.FilePath, token);
       });
       
       return Accepted(new { message = "Import started" });
   }
   ```

7. **Monitoring and Continuous Optimization**:
   - **Application Insights Integration**:
   ```csharp
   // In Program.cs
   builder.Services.AddApplicationInsightsTelemetry();
   
   // Custom tracking
   public class ProductService
   {
       private readonly TelemetryClient _telemetryClient;
       
       public ProductService(TelemetryClient telemetryClient)
       {
           _telemetryClient = telemetryClient;
       }
       
       public async Task<List<ProductDto>> SearchProductsAsync(string searchTerm)
       {
           var stopwatch = Stopwatch.StartNew();
           
           try
           {
               var result = await _repository.SearchProductsAsync(searchTerm);
               
               // Track metrics
               _telemetryClient.TrackMetric("ProductSearch.Duration", stopwatch.ElapsedMilliseconds);
               _telemetryClient.TrackMetric("ProductSearch.ResultCount", result.Count);
               
               return result;
           }
           catch (Exception ex)
           {
               _telemetryClient.TrackException(ex, new Dictionary<string, string>
               {
                   ["SearchTerm"] = searchTerm
               });
               throw;
           }
       }
   }
   ```
   
   - **Health Checks** for dependencies:
   ```csharp
   // In Program.cs
   builder.Services.AddHealthChecks()
       .AddDbContextCheck<ApplicationDbContext>()
       .AddRedis(builder.Configuration.GetConnectionString("Redis"))
       .AddCheck<ExternalApiHealthCheck>("external-api");
       
   app.MapHealthChecks("/health");
   ```

This multi-faceted approach addresses performance optimization throughout the API stack, from database access to network delivery, ensuring responsive and scalable applications.

---

### **9. How do you ensure your .NET Core Web API is production-ready?**

**Answer:**
Making a .NET Core Web API production-ready requires addressing multiple aspects beyond just functional requirements. Here's my comprehensive approach as an architect:

1. **Health Monitoring and Diagnostics**:
   - **Health Checks**:
   ```csharp
   // In Program.cs
   builder.Services.AddHealthChecks()
       .AddDbContextCheck<ApplicationDbContext>()
       .AddUrlGroup(new Uri("https://external-dependency.com/health"), "External API")
       .AddCheck<StorageHealthCheck>("Azure Blob Storage")
       .AddCheck("Database Connection", () => 
       {
           using var connection = new SqlConnection(builder.Configuration.GetConnectionString("Default"));
           try
           {
               connection.Open();
               return HealthCheckResult.Healthy();
           }
           catch (Exception ex)
           {
               return HealthCheckResult.Unhealthy(ex.Message);
           }
       });
   
   // Configure health check endpoints
   app.UseHealthChecks("/health", new HealthCheckOptions
   {
       ResponseWriter = UIResponseWriter.WriteHealthCheckUIResponse
   });
   
   app.UseHealthChecks("/health/ready", new HealthCheckOptions
   {
       Predicate = check => check.Tags.Contains("ready"),
       ResponseWriter = UIResponseWriter.WriteHealthCheckUIResponse
   });
   
   app.UseHealthChecks("/health/live", new HealthCheckOptions
   {
       Predicate = _ => false,
       ResponseWriter = UIResponseWriter.WriteHealthCheckUIResponse
   });
   ```
   
   - **Logging and Monitoring**:
   ```csharp
   // In Program.cs
   builder.Services.AddLogging(builder =>
   {
       builder.AddConsole();
       builder.AddDebug();
       builder.AddApplicationInsights();
       
       // Configure log levels
       builder.AddFilter("Microsoft", LogLevel.Warning)
              .AddFilter("System", LogLevel.Warning)
              .AddFilter("MyApi", LogLevel.Information);
   });
   
   // Usage in services
   public class OrderService
   {
       private readonly ILogger<OrderService> _logger;
       
       public OrderService(ILogger<OrderService> logger)
       {
           _logger = logger;
       }
       
       public async Task<Order> CreateOrderAsync(CreateOrderDto orderDto)
       {
           _logger.LogInformation("Creating new order for customer {CustomerId}", orderDto.CustomerId);
           
           try
           {
               // Order creation logic
               
               _logger.LogInformation("Created order {OrderId} for customer {CustomerId}", order.Id, orderDto.CustomerId);
               return order;
           }
           catch (Exception ex)
           {
               _logger.LogError(ex, "Failed to create order for customer {CustomerId}", orderDto.CustomerId);
               throw;
           }
       }
   }
   ```
   
   - **Distributed Tracing**:
   ```csharp
   // In Program.cs
   builder.Services.AddOpenTelemetry()
       .WithTracing(tracerProviderBuilder =>
           tracerProviderBuilder
               .AddSource("MyApi")
               .SetResourceBuilder(ResourceBuilder.CreateDefault().AddService("MyApi"))
               .AddAspNetCoreInstrumentation()
               .AddHttpClientInstrumentation()
               .AddEntityFrameworkCoreInstrumentation()
               .AddJaegerExporter());
   
   // Usage in code
   public class ProductService
   {
       private static readonly ActivitySource _activitySource = new ActivitySource("MyApi");
       
       public async Task<ProductDto> GetProductByIdAsync(int id)
       {
           using var activity = _activitySource.StartActivity("GetProductById");
           activity?.SetTag("productId", id);
           
           // Implementation
       }
   }
   ```

2. **Resilience Patterns**:
   - **Circuit Breakers**:
   ```csharp
   // In Program.cs
   builder.Services.AddHttpClient<IExternalService, ExternalService>()
       .AddPolicyHandler(GetCircuitBreakerPolicy());
   
   static IAsyncPolicy<HttpResponseMessage> GetCircuitBreakerPolicy()
   {
       return HttpPolicyExtensions
           .HandleTransientHttpError()
           .CircuitBreakerAsync(5, TimeSpan.FromMinutes(1));
   }
   ```
   
   - **Retries**:
   ```csharp
   // In Program.cs
   builder.Services.AddHttpClient<IPaymentGatewayService, PaymentGatewayService>()
       .AddPolicyHandler(GetRetryPolicy());
   
   static IAsyncPolicy<HttpResponseMessage> GetRetryPolicy()
   {
       return HttpPolicyExtensions
           .HandleTransientHttpError()
           .OrResult(msg => msg.StatusCode == System.Net.HttpStatusCode.TooManyRequests)
           .WaitAndRetryAsync(3, retryAttempt => TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)));
   }
   ```
   
   - **Timeouts**:
   ```csharp
   builder.Services.AddHttpClient<IExternalService, ExternalService>()
       .AddPolicyHandler(Policy.TimeoutAsync<HttpResponseMessage>(TimeSpan.FromSeconds(10)));
   ```
   
   - **Bulkhead Pattern**:
   ```csharp
   builder.Services.AddHttpClient<IReportingService, ReportingService>()
       .AddPolicyHandler(Policy.BulkheadAsync<HttpResponseMessage>(10, 100));
   ```

3. **Security Hardening**:
   - **HTTPS Enforcement**:
   ```csharp
   // In Program.cs
   builder.Services.AddHttpsRedirection(options =>
   {
       options.RedirectStatusCode = StatusCodes.Status308PermanentRedirect;
       options.HttpsPort = 443;
   });
   
   app.UseHttpsRedirection();
   
   if (!app.Environment.IsDevelopment())
   {
       app.UseHsts();
   }
   ```
   
   - **Content Security Policy**:
   ```csharp
   app.Use(async (context, next) =>
   {
       context.Response.Headers.Add("Content-Security-Policy", 
          "default-src 'self'; script-src 'self' https://trusted-cdn.com");
       
       await next();
   });
   ```
   
   - **Security Headers**:
   ```csharp
   app.Use(async (context, next) =>
   {
       context.Response.Headers.Add("X-Content-Type-Options", "nosniff");
       context.Response.Headers.Add("X-Frame-Options", "DENY");
       context.Response.Headers.Add("X-XSS-Protection", "1; mode=block");
       context.Response.Headers.Add("Referrer-Policy", "strict-origin-when-cross-origin");
       
       await next();
   });
   ```
   
   - **API Key Validation**:
   ```csharp
   public class ApiKeyAuthMiddleware
   {
       private readonly RequestDelegate _next;
       private readonly IConfiguration _configuration;
       
       public ApiKeyAuthMiddleware(RequestDelegate next, IConfiguration configuration)
       {
           _next = next;
           _configuration = configuration;
       }
       
       public async Task InvokeAsync(HttpContext context)
       {
           if (!context.Request.Headers.TryGetValue("X-API-Key", out var apiKeyHeaderValues))
           {
               context.Response.StatusCode = 401;
               await context.Response.WriteAsJsonAsync(new { message = "API Key is missing" });
               return;
           }
           
           var apiKey = apiKeyHeaderValues.FirstOrDefault();
           var validApiKeys = _configuration.GetSection("ApiKeys").Get<List<string>>();
           
           if (apiKey == null || !validApiKeys.Contains(apiKey))
           {
               context.Response.StatusCode = 401;
               await context.Response.WriteAsJsonAsync(new { message = "Invalid API Key" });
               return;
           }
           
           await _next(context);
       }
   }
   ```

4. **Environment Configuration**:
   - **Environment-Specific Settings**:
   ```json
   // appsettings.json
   {
     "ConnectionStrings": {
       "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=MyApiDb;Trusted_Connection=True;"
     }
   }
   
   // appsettings.Production.json
   {
     "ConnectionStrings": {
       "DefaultConnection": "Server=prod-sql;Database=MyApiDb;User Id=app_user;Password=****;"
     },
     "Logging": {
       "LogLevel": {
         "Default": "Information",
         "Microsoft": "Warning",
         "System": "Warning"
       }
     }
   }
   ```
   
   - **Secret Management**:
   ```csharp
   // In Program.cs for development
   if (app.Environment.IsDevelopment())
   {
       builder.Configuration.AddUserSecrets<Program>();
   }
   
   // For production using Key Vault
   if (app.Environment.IsProduction())
   {
       var keyVaultEndpoint = new Uri(builder.Configuration["KeyVaultEndpoint"]);
       builder.Configuration.AddAzureKeyVault(keyVaultEndpoint, new DefaultAzureCredential());
   }
   ```
   
   - **Feature Flags**:
   ```csharp
   // In Program.cs
   builder.Services.AddFeatureManagement();
   
   // In controller
   public class ProductsController : ControllerBase
   {
       private readonly IFeatureManager _featureManager;
       
       public ProductsController(IFeatureManager featureManager)
       {
           _featureManager = featureManager;
       }
       
       [HttpGet("recommendations")]
       public async Task<IActionResult> GetRecommendations()
       {
           if (await _featureManager.IsEnabledAsync("RecommendationEngine"))
           {
               // New recommendation logic
               return Ok(await _recommendationService.GetRecommendationsAsync());
           }
           else
           {
               // Legacy logic
               return Ok(await _productService.GetTopProductsAsync());
           }
       }
   }
   ```

5. **Performance Optimization**:
   - **Response Caching and Compression**:
   ```csharp
   // In Program.cs
   builder.Services.AddResponseCaching();
   builder.Services.AddResponseCompression(options =>
   {
       options.Providers.Add<BrotliCompressionProvider>();
       options.Providers.Add<GzipCompressionProvider>();
       options.EnableForHttps = true;
   });
   
   // In middleware pipeline
   app.UseResponseCaching();
   app.UseResponseCompression();
   ```
   
   - **Async I/O Operations**:
   ```csharp
   [HttpGet]
   public async Task<ActionResult<PagedResult<ProductDto>>> GetProducts([FromQuery] ProductParameters parameters)
   {
       var products = await _productService.GetProductsAsync(parameters);
       return Ok(products);
   }
   ```

6. **API Documentation**:
   - **OpenAPI/Swagger Integration**:
   ```csharp
   // In Program.cs
   if (app.Environment.IsDevelopment() || app.Environment.IsStaging())
   {
       builder.Services.AddEndpointsApiExplorer();
       builder.Services.AddSwaggerGen(options =>
       {
           options.SwaggerDoc("v1", new OpenApiInfo
           {
               Title = "My API",
               Version = "v1",
               Description = "An API for managing products and orders",
               Contact = new OpenApiContact
               {
                   Name = "API Support",
                   Email = "support@example.com"
               }
           });
           
           options.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
           {
               Description = "JWT Authorization header using the Bearer scheme",
               Name = "Authorization",
               In = ParameterLocation.Header,
               Type = SecuritySchemeType.Http,
               Scheme = "bearer"
           });
           
           options.AddSecurityRequirement(new OpenApiSecurityRequirement
           {
               {
                   new OpenApiSecurityScheme
                   {
                       Reference = new OpenApiReference
                       {
                           Type = ReferenceType.SecurityScheme,
                           Id = "Bearer"
                       }
                   },
                   Array.Empty<string>()
               }
           });
           
           // Include XML comments
           var xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
           var xmlPath = Path.Combine(AppContext.BaseDirectory, xmlFile);
           options.IncludeXmlComments(xmlPath);
       });
       
       app.UseSwagger();
       app.UseSwaggerUI(c =>
       {
           c.SwaggerEndpoint("/swagger/v1/swagger.json", "My API v1");
           c.RoutePrefix = "api-docs";
       });
   }
   ```
   
   - **API Documentation in Code**:
   ```csharp
   /// <summary>
   /// Retrieves a product by its unique identifier
   /// </summary>
   /// <param name="id">The product ID</param>
   /// <returns>The product details</returns>
   /// <response code="200">Returns the product</response>
   /// <response code="404">If the product is not found</response>
   [HttpGet("{id}")]
   [ProducesResponseType(typeof(ProductDto), StatusCodes.Status200OK)]
   [ProducesResponseType(StatusCodes.Status404NotFound)]
   public async Task<ActionResult<ProductDto>> GetProduct(int id)
   {
       var product = await _productService.GetProductByIdAsync(id);
       if (product == null) return NotFound();
       
       return Ok(product);
   }
   ```

7. **Rate Limiting and Throttling**:
   - **IP-Based Rate Limiting**:
   ```csharp
   // In Program.cs
   builder.Services.AddRateLimiter(options =>
   {
       options.GlobalLimiter = PartitionedRateLimiter.Create<HttpContext, string>(context =>
       {
           var ipAddress = context.Connection.RemoteIpAddress?.ToString() ?? "unknown";
           return RateLimitPartition.GetFixedWindowLimiter(ipAddress, partition =>
           {
               return new FixedWindowRateLimiterOptions
               {
                   Window = TimeSpan.FromMinutes(1),
                   PermitLimit = 100,
                   AutoReplenishment = true
               };
           });
       });
       options.RejectionStatusCode = StatusCodes.Status429TooManyRequests;
   });
   
   // In middleware pipeline
   app.UseRateLimiter();
   ```
   
   - **User-Based Rate Limiting**:
   ```csharp
   // In Program.cs
   builder.Services.AddRateLimiter(options =>
   {
       options.AddPolicy("authenticated", context =>
       {
           var username = context.User.Identity?.Name ?? context.Request.Headers["X-API-Key"].FirstOrDefault() ?? "anonymous";
           
           return RateLimitPartition.GetConcurrencyLimiter(username, key =>
           {
               // Authenticated users get higher limits
               if (username != "anonymous")
               {
                   return new ConcurrencyLimiterOptions
                   {
                       PermitLimit = 20,
                       QueueLimit = 10,
                       QueueProcessingOrder = QueueProcessingOrder.OldestFirst
                   };
               }
               
               return new ConcurrencyLimiterOptions
               {
                   PermitLimit = 5,
                   QueueLimit = 0
               };
           });
       });
   });
   
   // In controller
   [HttpGet("reports")]
   [EnableRateLimiting("authenticated")]
   public async Task<IActionResult> GenerateReport()
   {
       // Implementation
   }
   ```
   
   - **Endpoint-Specific Limits**:
   ```csharp
   [HttpPost("bulk-import")]
   [EnableRateLimiting("api")]
   public async Task<IActionResult> BulkImport([FromBody] BulkImportRequest request)
   {
       // Resource-intensive operation
   }
   ```

8. **Graceful Shutdown**:
   - **Handling Long-Running Requests**:
   ```csharp
   // In Program.cs
   builder.WebHost.ConfigureKestrel(options =>
   {
       options.Limits.KeepAliveTimeout = TimeSpan.FromMinutes(2);
       options.Limits.RequestHeadersTimeout = TimeSpan.FromSeconds(30);
   });
   
   // Graceful shutdown configuration
   builder.Host.ConfigureHostOptions(options =>
   {
       options.ShutdownTimeout = TimeSpan.FromSeconds(30);
   });
   ```
   
   - **Background Service Shutdown**:
   ```csharp
   public class ProcessingService : BackgroundService
   {
       protected override async Task ExecuteAsync(CancellationToken stoppingToken)
       {
           while (!stoppingToken.IsCancellationRequested)
           {
               try
               {
                   await ProcessWorkItem(stoppingToken);
               }
               catch (OperationCanceledException)
               {
                   // Graceful exit on cancellation
                   break;
               }
               catch (Exception ex)
               {
                   _logger.LogError(ex, "Error in processing service");
               }
               
               await Task.Delay(TimeSpan.FromSeconds(5), stoppingToken);
           }
           
           // Cleanup on shutdown
           await PerformCleanupAsync();
       }
   }
   ```

9. **Deployment Strategy**:
   - **Container-Ready Configuration**:
   ```csharp
   // In Program.cs
   builder.Host.ConfigureAppConfiguration((hostingContext, config) =>
   {
       config.SetBasePath(Directory.GetCurrentDirectory())
             .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
             .AddJsonFile($"appsettings.{hostingContext.HostingEnvironment.EnvironmentName}.json", optional: true)
             .AddEnvironmentVariables(); // For container environment variables
   });
   ```
   
   - **Health Probes for Orchestration**:
   ```csharp
   // Readiness probe - is the app ready to receive traffic?
   app.MapGet("/health/ready", () => Results.Ok())
      .WithMetadata(new EndpointNameMetadata("ReadinessProbe"));
   
   // Liveness probe - is the app running?
   app.MapGet("/health/live", () => Results.Ok())
      .WithMetadata(new EndpointNameMetadata("LivenessProbe"));
   ```
   
   - **Versioned Endpoints for Zero-Downtime Deployment**:
   ```csharp
   [ApiVersion("1.0")]
   [ApiVersion("2.0")]
   [Route("api/v{version:apiVersion}/products")]
   public class ProductsController : ControllerBase
   {
       // Implementations supporting multiple versions
   }
   ```

10. **Observability and Diagnostics**:
    - **Correlation IDs**:
    ```csharp
    // In Program.cs
    builder.Services.AddHttpContextAccessor();
    
    // Middleware
    app.Use(async (context, next) =>
    {
        // Generate correlation ID if not present
        if (!context.Request.Headers.TryGetValue("X-Correlation-ID", out var correlationId))
        {
            correlationId = Guid.NewGuid().ToString();
            context.Request.Headers.Add("X-Correlation-ID", correlationId);
        }
        
        // Add to response
        context.Response.OnStarting(() =>
        {
            context.Response.Headers.Add("X-Correlation-ID", correlationId);
            return Task.CompletedTask;
        });
        
        // Add to logging context
        using (LogContext.PushProperty("CorrelationId", correlationId))
        {
            await next();
        }
    });
    ```
    
    - **Structured Logging**:
    ```csharp
    // In Program.cs
    builder.Host.UseSerilog((context, services, configuration) => configuration
        .ReadFrom.Configuration(context.Configuration)
        .ReadFrom.Services(services)
        .Enrich.FromLogContext()
        .Enrich.WithMachineName()
        .Enrich.WithEnvironmentName()
        .WriteTo.Console(new JsonFormatter())
        .WriteTo.ApplicationInsights(
            services.GetRequiredService<TelemetryConfiguration>(),
            TelemetryConverter.Traces));
    ```

These comprehensive preparations transform a functional Web API into a production-ready system that can handle real-world demands with reliability, security, and observability.

---

### **10. How would you handle database migrations and schema changes in a .NET Core Web API?**

**Answer:**
Managing database migrations is critical for maintaining data integrity while evolving your application. Here's my approach as an architect:

1. **Entity Framework Core Migrations**:
   - **Setting Up Migrations**:
   ```csharp
   // In Program.cs
   builder.Services.AddDbContext<ApplicationDbContext>(options =>
       options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection"),
       sqlOptions =>
       {
           sqlOptions.EnableRetryOnFailure(
               maxRetryCount: 5,
               maxRetryDelay: TimeSpan.FromSeconds(30),
               errorNumbersToAdd: null);
       }));
   ```
   
   - **Creating Migrations**:
   ```bash
   # Command line
   dotnet ef migrations add InitialCreate -o Data/Migrations
   dotnet ef migrations add AddProductCategories
   ```
   
   - **Migration Class**:
   ```csharp
   public partial class AddProductCategories : Migration
   {
       protected override void Up(MigrationBuilder migrationBuilder)
       {
           migrationBuilder.CreateTable(
               name: "Categories",
               columns: table => new
               {
                   Id = table.Column<int>(nullable: false)
                       .Annotation("SqlServer:Identity", "1, 1"),
                   Name = table.Column<string>(maxLength: 100, nullable: false),
                   Description = table.Column<string>(nullable: true)
               },
               constraints: table =>
               {
                   table.PrimaryKey("PK_Categories", x => x.Id);
               });
               
           migrationBuilder.AddColumn<int>(
               name: "CategoryId",
               table: "Products",
               nullable: true);
               
           migrationBuilder.CreateIndex(
               name: "IX_Products_CategoryId",
               table: "Products",
               column: "CategoryId");
               
           migrationBuilder.AddForeignKey(
               name: "FK_Products_Categories_CategoryId",
               table: "Products",
               column: "CategoryId",
               principalTable: "Categories",
               principalColumn: "Id",
               onDelete: ReferentialAction.SetNull);
       }
       
       protected override void Down(MigrationBuilder migrationBuilder)
       {
           migrationBuilder.DropForeignKey(
               name: "FK_Products_Categories_CategoryId",
               table: "Products");
               
           migrationBuilder.DropTable(
               name: "Categories");
               
           migrationBuilder.DropIndex(
               name: "IX_Products_CategoryId",
               table: "Products");
               
           migrationBuilder.DropColumn(
               name: "CategoryId",
               table: "Products");
       }
   }
   ```

2. **Migration Application Strategies**:
   - **Automatic Migrations at Startup** (development/testing):
   ```csharp
   // In Program.cs or Startup.cs
   if (app.Environment.IsDevelopment() || app.Environment.IsStaging())
   {
       using (var scope = app.Services.CreateScope())
       {
           var dbContext = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
           var logger = scope.ServiceProvider.GetRequiredService<ILogger<Program>>();
           
           try
           {
               dbContext.Database.Migrate();
               logger.LogInformation("Database migrated successfully");
               
               // Seed initial data if needed
               await new DatabaseSeeder(dbContext, logger).SeedAsync();
           }
           catch (Exception ex)
           {
               logger.LogError(ex, "An error occurred during database migration");
               throw;
           }
       }
   }
   ```
   
   - **Pre-Deployment Migration Scripts** (production):
   ```csharp
   // Generate SQL script
   dotnet ef migrations script --idempotent --output "migrate.sql"
   
   // In CI/CD pipeline
   // 1. Generate migration script
   // 2. Review script (automated or manual)
   // 3. Apply during maintenance window
   ```
   
   - **Database Migration Service** (large systems):
   ```csharp
   public class MigrationService : IHostedService
   {
       private readonly IServiceProvider _serviceProvider;
       private readonly ILogger<MigrationService> _logger;
       private readonly IConfiguration _configuration;
       
       public MigrationService(
           IServiceProvider serviceProvider,
           ILogger<MigrationService> logger,
           IConfiguration configuration)
       {
           _serviceProvider = serviceProvider;
           _logger = logger;
           _configuration = configuration;
       }
       
       public async Task StartAsync(CancellationToken cancellationToken)
       {
           // Only run migrations if explicitly enabled
           if (!_configuration.GetValue<bool>("Database:RunMigrations"))
           {
               _logger.LogInformation("Database migrations disabled");
               return;
           }
           
           _logger.LogInformation("Starting database migration");
           
           using var scope = _serviceProvider.CreateScope();
           var dbContext = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
           
           try
           {
               await dbContext.Database.MigrateAsync(cancellationToken);
               _logger.LogInformation("Database migration completed successfully");
           }
           catch (Exception ex)
           {
               _logger.LogError(ex, "An error occurred during database migration");
               
               // Decide whether to fail startup based on configuration
               if (_configuration.GetValue<bool>("Database:FailStartupOnMigrationFailure"))
               {
                   throw;
               }
           }
       }
       
       public Task StopAsync(CancellationToken cancellationToken)
       {
           return Task.CompletedTask;
       }
   }
   ```

3. **Managing Database Schema Changes**:
   - **Safe Schema Changes**:
   ```csharp
   public partial class AddProductIndexes : Migration
   {
       protected override void Up(MigrationBuilder migrationBuilder)
       {
           // Create new index without dropping existing ones
           migrationBuilder.CreateIndex(
               name: "IX_Products_Name",
               table: "Products",
               column: "Name");
               
           // Add column with default value
           migrationBuilder.AddColumn<bool>(
               name: "IsActive",
               table: "Products",
               nullable: false,
               defaultValue: true);
       }
       
       protected override void Down(MigrationBuilder migrationBuilder)
       {
           migrationBuilder.DropIndex(
               name: "IX_Products_Name",
               table: "Products");
               
           migrationBuilder.DropColumn(
               name: "IsActive",
               table: "Products");
       }
   }
   ```
   
   - **Custom Migration Operations**:
   ```csharp
   public partial class MigrateProductData : Migration
   {
       protected override void Up(MigrationBuilder migrationBuilder)
       {
           // Add a new column
           migrationBuilder.AddColumn<string>(
               name: "SKU",
               table: "Products",
               nullable: true);
               
           // Custom SQL to populate the new column
           migrationBuilder.Sql(@"
               UPDATE Products
               SET SKU = 'SKU-' + CAST(Id AS VARCHAR(10))
               WHERE SKU IS NULL
           ");
           
           // Make the column non-nullable after populating
           migrationBuilder.AlterColumn<string>(
               name: "SKU",
               table: "Products",
               nullable: false,
               oldNullable: true);
       }
       
       protected override void Down(MigrationBuilder migrationBuilder)
       {
           migrationBuilder.DropColumn(
               name: "SKU",
               table: "Products");
       }
   }
   ```
   
   - **Data Migration for Complex Changes**:
   ```csharp
   public partial class SplitCustomerTable : Migration
   {
       protected override void Up(MigrationBuilder migrationBuilder)
       {
           // Create new tables
           migrationBuilder.CreateTable(
               name: "CustomerDetails",
               columns: table => new
               {
                   Id = table.Column<int>(nullable: false)
                       .Annotation("SqlServer:Identity", "1, 1"),
                   CustomerId = table.Column<int>(nullable: false),
                   Phone = table.Column<string>(nullable: true),
                   Address = table.Column<string>(nullable: true)
               },
               constraints: table =>
               {
                   table.PrimaryKey("PK_CustomerDetails", x => x.Id);
                   table.ForeignKey(
                       name: "FK_CustomerDetails_Customers_CustomerId",
                       column: x => x.CustomerId,
                       principalTable: "Customers",
                       principalColumn: "Id",
                       onDelete: ReferentialAction.Cascade);
               });
           
           // Migrate data with custom SQL
           migrationBuilder.Sql(@"
               INSERT INTO CustomerDetails (CustomerId, Phone, Address)
               SELECT Id, Phone, Address FROM Customers
           ");
           
           // Remove columns from original table
           migrationBuilder.DropColumn(
               name: "Phone",
               table: "Customers");
               
           migrationBuilder.DropColumn(
               name: "Address",
               table: "Customers");
       }
       
       protected override void Down(MigrationBuilder migrationBuilder)
       {
           // Add columns back to original table
           migrationBuilder.AddColumn<string>(
               name: "Phone",
               table: "Customers",
               nullable: true);
               
           migrationBuilder.AddColumn<string>(
               name: "Address",
               table: "Customers",
               nullable: true);
               
           // Move data back
           migrationBuilder.Sql(@"
               UPDATE c
               SET c.Phone = cd.Phone, c.Address = cd.Address
               FROM Customers c
               JOIN CustomerDetails cd ON c.Id = cd.CustomerId
           ");
           
           // Drop the new table
           migrationBuilder.DropTable(
               name: "CustomerDetails");
       }
   }
   ```

4. **Migration Testing and Validation**:
   - **Automated Testing**:
   ```csharp
   public class DatabaseMigrationTests
   {
       [Fact]
       public void CanApplyMigrations()
       {
           // Arrange - Create a new database with a unique name
           var connectionString = "Server=localhost;Database=TestMigrations_" + 
                                 Guid.NewGuid().ToString("N") + ";Trusted_Connection=True;";
                                 
           var optionsBuilder = new DbContextOptionsBuilder<ApplicationDbContext>()
               .UseSqlServer(connectionString);
           
           // Act - Apply migrations
           using (var context = new ApplicationDbContext(optionsBuilder.Options))
           {
               context.Database.EnsureCreated();
               context.Database.Migrate();
           }
           
           // Assert - Verify the database was created successfully
           using (var context = new ApplicationDbContext(optionsBuilder.Options))
           {
               // Check for tables
               Assert.True(context.Model.FindEntityType(typeof(Product)) != null);
               
               // Check for specific schema elements
               var connection = context.Database.GetDbConnection();
               connection.Open();
               using (var command = connection.CreateCommand())
               {
                   command.CommandText = "SELECT COUNT(*) FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_NAME = 'Products'";
                   var result = command.ExecuteScalar();
                   Assert.Equal(1, result);
               }
           }
       }
   }
   ```
   
   - **Versioned Database Rollback Tests**:
   ```csharp
   [Fact]
   public void CanRollbackMigrations()
   {
       // Arrange
       var connectionString = "Server=localhost;Database=TestMigrationRollback_" + 
                             Guid.NewGuid().ToString("N") + ";Trusted_Connection=True;";
                             
       var optionsBuilder = new DbContextOptionsBuilder<ApplicationDbContext>()
           .UseSqlServer(connectionString);
       
       // Act - Apply migrations and then roll back one
       string[] appliedMigrations;
       using (var context = new ApplicationDbContext(optionsBuilder.Options))
       {
           context.Database.Migrate();
           appliedMigrations = context.Database.GetAppliedMigrations().ToArray();
           Assert.True(appliedMigrations.Length > 1); // Ensure we have migrations to roll back
           
           // Roll back the last migration
           var migrator = context.GetService<IMigrator>();
           migrator.Migrate(appliedMigrations[appliedMigrations.Length - 2]);
       }
       
       // Assert
       using (var context = new ApplicationDbContext(optionsBuilder.Options))
       {
           var currentMigrations = context.Database.GetAppliedMigrations().ToArray();
           Assert.Equal(appliedMigrations.Length - 1, currentMigrations.Length);
       }
   }
   ```

5. **Production Deployment Strategies**:
   - **Zero-Downtime Migration** with shadow tables:
   ```csharp
   public partial class AddProductRatingsWithShadowTable : Migration
   {
       protected override void Up(MigrationBuilder migrationBuilder)
       {
           // 1. Create shadow table with new schema
           migrationBuilder.CreateTable(
               name: "Products_New",
               columns: table => new
               {
                   Id = table.Column<int>(nullable: false),
                   Name = table.Column<string>(maxLength: 100, nullable: false),
                   Price = table.Column<decimal>(type: "decimal(18,2)", nullable: false),
                   Rating = table.Column<decimal>(type: "decimal(3,2)", nullable: false, defaultValue: 0.0m)
               },
               constraints: table =>
               {
                   table.PrimaryKey("PK_Products_New", x => x.Id);
               });
               
           // 2. Copy existing data to shadow table
           migrationBuilder.Sql(@"
               INSERT INTO Products_New (Id, Name, Price, Rating)
               SELECT Id, Name, Price, 0.0 as Rating
               FROM Products
           ");
           
           // 3. Rename tables to swap them
           migrationBuilder.Sql(@"
               EXEC sp_rename 'Products', 'Products_Old';
               EXEC sp_rename 'Products_New', 'Products';
           ");
           
           // 4. Create any necessary indexes on new table
           migrationBuilder.CreateIndex(
               name: "IX_Products_Name",
               table: "Products",
               column: "Name");
               
           // 5. Drop old table after successful migration
           migrationBuilder.DropTable(
               name: "Products_Old");
       }
       
       protected override void Down(MigrationBuilder migrationBuilder)
       {
           // Reverse the process
           migrationBuilder.CreateTable(
               name: "Products_Old",
               columns: table => new
               {
                   Id = table.Column<int>(nullable: false),
                   Name = table.Column<string>(maxLength: 100, nullable: false),
                   Price = table.Column<decimal>(type: "decimal(18,2)", nullable: false)
               },
               constraints: table =>
               {
                   table.PrimaryKey("PK_Products_Old", x => x.Id);
               });
               
           migrationBuilder.Sql(@"
               INSERT INTO Products_Old (Id, Name, Price)
               SELECT Id, Name, Price FROM Products
           ");
           
           migrationBuilder.Sql(@"
               EXEC sp_rename 'Products', 'Products_Temp';
               EXEC sp_rename 'Products_Old', 'Products';
               DROP TABLE Products_Temp;
           ");
           
           migrationBuilder.CreateIndex(
               name: "IX_Products_Name",
               table: "Products",
               column: "Name");
       }
   }
   ```
   
   - **Staged Rollout** with feature flags:
   ```csharp
   // In Program.cs
   builder.Services.AddFeatureManagement();
   
   // Service implementation
   public class ProductRepository : IProductRepository
   {
       private readonly ApplicationDbContext _context;
       private readonly IFeatureManager _featureManager;
       
       public ProductRepository(ApplicationDbContext context, IFeatureManager featureManager)
       {
           _context = context;
           _featureManager = featureManager;
       }
       
       public async Task<IEnumerable<Product>> GetAllProductsAsync()
       {
           // Use new database schema if feature is enabled
           if (await _featureManager.IsEnabledAsync("UseNewProductSchema"))
           {
               return await _context.Products
                   .OrderBy(p => p.Name)
                   .ToListAsync();
           }
           else
           {
               // Use old query approach (avoiding new columns)
               return await _context.Products
                   .Select(p => new Product
                   {
                       Id = p.Id,
                       Name = p.Name,
                       Price = p.Price
                       // Exclude new properties
                   })
                   .OrderBy(p => p.Name)
                   .ToListAsync();
           }
       }
   }
   ```
   
   - **Dual-Write Pattern** for critical data:
   ```csharp
   public class OrderService : IOrderService
   {
       private readonly ApplicationDbContext _context;
       private readonly IFeatureManager _featureManager;
       private readonly ILogger<OrderService> _logger;
       
       public OrderService(
           ApplicationDbContext context,
           IFeatureManager featureManager,
           ILogger<OrderService> logger)
       {
           _context = context;
           _featureManager = featureManager;
           _logger = logger;
       }
       
       public async Task<Order> CreateOrderAsync(OrderDto orderDto)
       {
           // Create order in current schema
           var order = new Order
           {
               CustomerId = orderDto.CustomerId,
               OrderDate = DateTime.UtcNow,
               Status = OrderStatus.Pending
           };
           
           _context.Orders.Add(order);
           
           // If using the new schema with order items as a separate table
           if (await _featureManager.IsEnabledAsync("NewOrderSchema"))
           {
               // Add to new schema - OrderItems table
               foreach (var item in orderDto.Items)
               {
                   var orderItem = new OrderItem
                   {
                       Order = order,
                       ProductId = item.ProductId,
                       Quantity = item.Quantity,
                       UnitPrice = item.UnitPrice
                   };
                   
                   _context.OrderItems.Add(orderItem);
               }
               
               // Also write to old schema for backward compatibility during transition
               try
               {
                   var itemsJson = JsonSerializer.Serialize(orderDto.Items);
                   
                   // Use raw SQL to update legacy column
                   await _context.Database.ExecuteSqlRawAsync(
                       "UPDATE Orders SET ItemsJson = {0} WHERE Id = {1}",
                       itemsJson, order.Id);
               }
               catch (Exception ex)
               {
                   _logger.LogError(ex, "Failed to write to legacy schema for Order {OrderId}", order.Id);
                   // Continue anyway - the new schema is primary
               }
           }
           else
           {
               // Old schema only - store items as JSON in single column
               order.ItemsJson = JsonSerializer.Serialize(orderDto.Items);
           }
           
           await _context.SaveChangesAsync();
           return order;
       }
   }
   ```

6. **Database Version Control**:
   - **Migration History Table**:
   ```csharp
   // Creating a custom migration history table
   protected override void OnModelCreating(ModelBuilder modelBuilder)
   {
       base.OnModelCreating(modelBuilder);
       
       // Configure migration history table
       modelBuilder.Entity<HistoryRow>().ToTable("__EFMigrationsHistory", "schema");
   }
   ```
   
   - **Tracking Migration Application**:
   ```csharp
   public class DatabaseMigrationLogger : IHostedService
   {
       private readonly IServiceProvider _serviceProvider;
       private readonly ILogger<DatabaseMigrationLogger> _logger;
       
       public DatabaseMigrationLogger(
           IServiceProvider serviceProvider,
           ILogger<DatabaseMigrationLogger> logger)
       {
           _serviceProvider = serviceProvider;
           _logger = logger;
       }
       
       public async Task StartAsync(CancellationToken cancellationToken)
       {
           using var scope = _serviceProvider.CreateScope();
           var context = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
           
           // Log current migrations
           var migrations = await context.Database.GetAppliedMigrationsAsync(cancellationToken);
           
           _logger.LogInformation(
               "Database is at migration level {MigrationCount}. Latest migration: {LatestMigration}",
               migrations.Count(),
               migrations.LastOrDefault() ?? "No migrations");
           
           // Record migration state
           await context.MigrationLogs.AddAsync(new MigrationLog
           {
               AppliedAt = DateTime.UtcNow,
               Environment = Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT"),
               MigrationName = migrations.LastOrDefault(),
               ServerName = Environment.MachineName
           }, cancellationToken);
           
           await context.SaveChangesAsync(cancellationToken);
       }
       
       public Task StopAsync(CancellationToken cancellationToken)
       {
           return Task.CompletedTask;
       }
   }
   
   // Entity to track migration
   public class MigrationLog
   {
       public int Id { get; set; }
       public string MigrationName { get; set; }
       public DateTime AppliedAt { get; set; }
       public string Environment { get; set; }
       public string ServerName { get; set; }
   }
   ```

7. **Handling Schema Drift and Recovery**:
   - **Schema Consistency Checks**:
   ```csharp
   public class SchemaValidationService : IHostedService
   {
       private readonly IServiceProvider _serviceProvider;
       private readonly ILogger<SchemaValidationService> _logger;
       private readonly IHostApplicationLifetime _appLifetime;
       
       public SchemaValidationService(
           IServiceProvider serviceProvider,
           ILogger<SchemaValidationService> logger,
           IHostApplicationLifetime appLifetime)
       {
           _serviceProvider = serviceProvider;
           _logger = logger;
           _appLifetime = appLifetime;
       }
       
       public async Task StartAsync(CancellationToken cancellationToken)
       {
           try
           {
               using var scope = _serviceProvider.CreateScope();
               var context = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
               
               // Check for pending migrations
               var pendingMigrations = await context.Database.GetPendingMigrationsAsync(cancellationToken);
               if (pendingMigrations.Any())
               {
                   _logger.LogWarning(
                       "Database has {Count} pending migrations: {Migrations}",
                       pendingMigrations.Count(),
                       string.Join(", ", pendingMigrations));
               }
               
               // Check schema consistency
               var modelDiffer = context.GetService<IMigrationsModelDiffer>();
               var designModel = context.GetService<IDesignTimeModel>().Model;
               var databaseModel = await context.GetService<IDatabaseModelFactory>()
                   .CreateModel(context.Database.GetDbConnection());
                   
               var differences = modelDiffer.GetDifferences(databaseModel.GetRelationalModel(), designModel.GetRelationalModel());
               
               if (differences.Any())
               {
                   _logger.LogError(
                       "Database schema differs from expected model. Differences: {DifferenceCount}",
                       differences.Count());
                       
                   // Log detailed differences
                   foreach (var diff in differences.Take(10))
                   {
                       _logger.LogError("Schema difference: {Difference}", diff);
                   }
                   
                   // Optionally stop application if consistency is critical
                   // _appLifetime.StopApplication();
               }
               else
               {
                   _logger.LogInformation("Database schema is consistent with application model");
               }
           }
           catch (Exception ex)
           {
               _logger.LogError(ex, "Error validating database schema");
               // Decide whether to continue or stop based on criticality
           }
       }
       
       public Task StopAsync(CancellationToken cancellationToken)
       {
           return Task.CompletedTask;
       }
   }
   ```
   
   - **Database Repair and Recovery**:
   ```csharp
   // Command handler for database repair
   public class RepairDatabaseCommand : IRequest<RepairResult>
   {
       public bool ApplyFixes { get; set; }
   }
   
   public class RepairDatabaseCommandHandler : IRequestHandler<RepairDatabaseCommand, RepairResult>
   {
       private readonly ApplicationDbContext _context;
       private readonly ILogger<RepairDatabaseCommandHandler> _logger;
       
       public RepairDatabaseCommandHandler(
           ApplicationDbContext context,
           ILogger<RepairDatabaseCommandHandler> logger)
       {
           _context = context;
           _logger = logger;
       }
       
       public async Task<RepairResult> Handle(RepairDatabaseCommand request, CancellationToken cancellationToken)
       {
           var result = new RepairResult();
           
           try
           {
               // 1. Check for missing tables
               var modelDiffer = _context.GetService<IMigrationsModelDiffer>();
               var designModel = _context.GetService<IDesignTimeModel>().Model;
               var databaseModel = await _context.GetService<IDatabaseModelFactory>()
                   .CreateModel(_context.Database.GetDbConnection());
                   
               var differences = modelDiffer.GetDifferences(
                   databaseModel.GetRelationalModel(),
                   designModel.GetRelationalModel());
               
               // 2. Categorize differences
               foreach (var diff in differences)
               {
                   if (diff is AddTableOperation addTable)
                   {
                       result.MissingTables.Add(addTable.Name);
                   }
                   else if (diff is AddColumnOperation addColumn)
                   {
                       result.MissingColumns.Add($"{addColumn.Table}.{addColumn.Name}");
                   }
                   // Other difference types...
               }
               
               // 3. Apply fixes if requested
               if (request.ApplyFixes)
               {
                   if (differences.Any())
                   {
                       _logger.LogWarning("Applying {Count} schema fixes", differences.Count());
                       
                       // Create migration builder and apply differences
                       var migrationBuilder = _context.GetService<IMigrationsSqlGenerator>();
                       var commands = migrationBuilder.Generate(differences);
                       
                       // Execute each command
                       foreach (var command in commands)
                       {
                           await _context.Database.ExecuteSqlRawAsync(command.CommandText);
                       }
                       
                       result.FixesApplied = true;
                   }
               }
               
               return result;
           }
           catch (Exception ex)
           {
               _logger.LogError(ex, "Error during database repair");
               result.Error = ex.Message;
               return result;
           }
       }
   }
   
   public class RepairResult
   {
       public List<string> MissingTables { get; set; } = new();
       public List<string> MissingColumns { get; set; } = new();
       public bool FixesApplied { get; set; }
       public string Error { get; set; }
   }
   ```

This comprehensive approach to database migrations ensures that schema changes can be applied safely, consistently, and with minimal risk across environments, supporting the evolution of your application while maintaining data integrity.

---

## ✅ **In Summary**

Creating a .NET Core Web API involves multiple architectural decisions and best practices. The key steps include:

1. **Setting up proper project structure** with separation of concerns
2. **Configuring the middleware pipeline** for request handling
3. **Implementing controllers and actions** following REST principles
4. **Designing models and DTOs** for input/output
5. **Setting up dependency injection** for services and repositories
6. **Adding cross-cutting concerns** like logging, error handling, and security
7. **Implementing data access** with proper patterns
8. **Testing and optimizing** for performance and reliability
9. **Documenting the API** for consumers
10. **Deploying and monitoring** in production

As an architect, focus on creating maintainable, secure, and performant APIs that can evolve with changing requirements while maintaining backward compatibility.

--- 

