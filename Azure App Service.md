# Implement Azure App Service web apps

## Explore Azure App Service 
- Azure App Service is an HTTP-based service for hosting web applications, REST APIs, and mobile back ends. 

- App Service Environment is an Azure App Service feature that provides a fully isolated and dedicated environment for running App Service apps. It offers improved security at high scale.

Unlike the App Service offering, where supporting infrastructure is shared, with App Service Environment, compute is dedicated to a single customer.

- In App Service, an app always runs in an App Service plan. An App Service plan defines a set of compute resources for a web app to run. One or more apps can be configured to run on the same computing resources (or in the same App Service plan).

- In the Free and Shared tiers, an app receives CPU minutes on a shared VM instance and can't scale out.

- Automated deployment, or continuous deployment, is a process used to push out new features and bug fixes in a fast and repetitive pattern with minimal effect on end users.

### Authentication and Authorization in App Service
The authentication and authorization middleware component is a feature of the platform that runs on the same VM as your application. When enabled, every incoming HTTP request passes through it before being handled by your application.

The platform middleware handles several things for your app:

  - Authenticates users and clients with the specified identity provider
  - Validates, stores, and refreshes OAuth tokens issued by the configured identity provider
  - Manages the authenticated session
  - Injects identity information into HTTP request headers

The module runs separately from your application code and can be configured using Azure Resource Manager settings or using a configuration file. No SDKs, specific programming languages, or changes to your application code are required.

#### Authentication flow

The authentication flow is the same for all providers, but differs depending on whether you want to sign in with the provider's SDK.

- **Without provider SDK**: The application delegates federated sign-in to App Service. This delegation is typically the case with browser apps, which can present the provider's login page to the user. The server code manages the sign-in process and is referred to as server-directed flow or server flow.

- **With provider SDK**: The application signs users in to the provider manually and then submits the authentication token to App Service for validation. This is typically the case with browser-less apps, which can't present the provider's sign-in page to the user. The application code manages the sign-in process and is referred to as client-directed flow or client flow. This applies to REST APIs, Azure Functions, JavaScript browser clients, and native mobile apps that sign users in using the provider's SDK.

#### Authorization behavior

In the Azure portal, you can configure App Service with many behaviors when an incoming request isn't authenticated.

- **Allow unauthenticated requests**: This option defers authorization of unauthenticated traffic to your application code. For authenticated requests, App Service also passes along authentication information in the HTTP headers. This option provides more flexibility in handling anonymous requests. It lets you present multiple sign-in providers to your users.

- **Require authentication***: This option rejects any unauthenticated traffic to your application. This rejection can be a redirect action to one of the configured identity providers. In these cases, a browser client is redirected to /.auth/login/<provider> for the provider you choose. If the anonymous request comes from a native mobile app, the returned response is an HTTP 401 Unauthorized. You can also configure the rejection to be an HTTP 401 Unauthorized or HTTP 403 Forbidden for all requests.

#### Token store

App Service provides a built-in token store, which is a repository of tokens that are associated with the users of your web apps, APIs, or native mobile apps. When you enable authentication with any provider, this token store is immediately available to your app.

#### Logging and tracing

If you enable application logging, authentication and authorization traces are collected directly in your log files. If you see an authentication error that you didn't expect, you can conveniently find all the details by looking in your existing application logs.

--- 
In Azure, which App Service plan categories provides the maximum scale-out capabilities? 
1. Dedicated compute 
2. Isolated 
3. Shared compute

The correct answer is: **2. Isolated** 

---

### Why Isolated Wins for Scale-Out

The **Isolated** App Service plan—especially the **IsolatedV2** tier—offers the **maximum scale-out capabilities** in Azure App Service. Here's how it stacks up:

| **Plan Category**     | **Max Instances** | **Scale-Out Capability** |
|-----------------------|-------------------|---------------------------|
| Shared Compute        | No scale-out   | Limited, shared resources |
| Dedicated Compute     | Up to 30       | Includes Standard & Premium tiers |
| **Isolated (IsolatedV2)** | **Up to 100**     | Highest scale-out, private VNet integration |

---

### What Makes Isolated Special?

- Runs in a **dedicated Azure Virtual Network**
- Supports **up to 100 instances** for horizontal scaling
- Ideal for **high-performance, secure, enterprise-grade apps**
- Offers **network isolation**, **enhanced security**, and **private endpoints**

--- 

Which of the following networking features of App Service can be used to control outbound network traffic? 
1. App assigened address 
2. Hybrid connections 
3. Service Endpoints

### **2. Hybrid connections **

---

## Configure web app settings 

- In App Service, app settings are variables passed as environment variables to the application code. 

- For ASP.NET and ASP.NET Core developers, setting app settings in App Service is like setting them in <appSettings> in Web.config or appsettings.json, but the values in App Service override the ones in Web.config or appsettings.json. You can keep development settings (for example, local MySQL password) in Web.config or appsettings.json and production secrets (for example, Azure MySQL database password) safely in App Service. The same code uses your development settings when you debug locally, and it uses your production secrets when deployed to Azure.
- App settings are always encrypted when stored (encrypted-at-rest). 

## Scale apps in Azure App Service

#### What is autoscaling?

Autoscaling is a cloud system or process that adjusts available resources based on the current demand. Autoscaling performs scaling in and out, as opposed to scaling up and down.

#### Azure App Service autoscaling

Autoscaling in Azure App Service monitors the resource metrics of a web app as it runs. It detects situations where other resources are required to handle an increasing workload, and ensures those resources are available before the system becomes overloaded.

Autoscaling responds to changes in the environment by adding or removing web servers and balancing the load between them. Autoscaling doesn't have any effect on the CPU power, memory, or storage capacity of the web servers powering the app, it only changes the number of web servers.

- A rule specifies the threshold for a metric, and triggers an autoscale event when this threshold is crossed. 

- Autoscaling works by adding or removing web servers.

#### Metrics for autoscale rules

- **CPU Percentage**. This metric is an indication of the CPU utilization across all instances. A high value shows that instances are becoming CPU-bound, which could cause delays in processing client requests.
- **Memory Percentage**. This metric captures the memory occupancy of the application across all instances. A high value indicates that free memory could be running low, and could cause one or more instances to fail.
- **Disk Queue Length**. This metric is a measure of the number of outstanding I/O requests across all instances. A high value means that disk contention could be occurring.
- **Http Queue Length**. This metric shows how many client requests are waiting for processing by the web app. If this number is large, client requests might fail with HTTP 408 (Timeout) errors.
- **Data In**. This metric is the number of bytes received across all instances.
- **Data Out**. This metric is the number of bytes sent by all instances.

Here’s your content reformatted into clear, skimmable bullet points for easy reference:

---

#### How an Autoscale Rule Analyzes Metrics

- **Autoscaling analyzes metric trends over time** across all instances to determine whether scaling actions are needed.
- The analysis involves **multiple steps**, focusing on both short-term and long-term metric behavior.

---

##### Step 1: Time Grain Aggregation
- Metrics are collected across all instances over a short period called the **time grain**.
- Most metrics use a **1-minute time grain** by default.
- The values are aggregated using a method called **time aggregation**.
- **Time aggregation options** include:
  - `Average`
  - `Minimum`
  - `Maximum`
  - `Sum`
  - `Last`
  - `Count`

---

##### Step 2: Duration-Based Aggregation
- A single minute is often too short to determine meaningful trends.
- Autoscale rules perform a **second aggregation** over a longer, user-defined period called the **Duration**.
- **Minimum Duration** is 5 minutes; common values include 10 minutes or more.
- During this step, the rule aggregates multiple time grain values (e.g., 10 one-minute averages).

---

##### Example Scenario
- If the **time aggregation** is set to `Average` and the metric is **CPU Percentage**:
  - Each minute, the average CPU usage across all instances is calculated.
- If the **Duration** is set to 10 minutes and the aggregation method is `Maximum`:
  - The rule evaluates the **maximum of the 10 average CPU values** to determine if the threshold is crossed.

---

#### Autoscale Actions – Key Concepts

##### Triggering Autoscale
- Autoscale actions are triggered when a **metric crosses a defined threshold**.
- The rule evaluates the metric using **comparison operators**:
  - `Greater than` → typically used for **scale-out** actions.
  - `Less than` → typically used for **scale-in** actions.
  - Other operators include `Equal to`, etc.

---

##### Types of Autoscale Actions
- **Scale-out**: Increases the number of instances.
- **Scale-in**: Decreases the number of instances.
- Autoscale can also **set the instance count to a specific number**, rather than adjusting incrementally.

---

##### Cool Down Period
- After an autoscale action, a **cool down period** is enforced.
- During this time, **no further scaling actions** can be triggered.
- Purpose: Allows the system to **stabilize** and prevents rapid, repeated scaling.
- **Minimum cool down period**: 5 minutes.
- Important because **starting or stopping instances takes time**, and metrics may not reflect changes immediately.

---

#### Explore autoscale best practices

##### Autoscale concepts

- An autoscale setting scales instances horizontally, which is out by increasing the instances and in by decreasing the number of instances. An autoscale setting has a maximum, minimum, and default value of instances.
- An autoscale job always reads the associated metric to scale by, checking if it crossed the configured threshold for scale-out or scale-in.
- All thresholds are calculated at an instance level. For example, "scale out by one instance when average CPU > 80% when instance count is 2", means to scale out when the average CPU across all instances is greater than 80%.
- All autoscale successes and failures are logged to the Activity Log. You can then configure an activity log alert so that you can be notified via email, short message service, or webhooks whenever there's activity.

---

#### Autoscale Best Practices

##### Set Clear Maximum and Minimum Instance Limits
- Ensure **minimum ≠ maximum**; otherwise, no scaling can occur.
- Maintain a **reasonable margin** between min and max values.
- Autoscale operates **within these inclusive boundaries**.

---

##### Select the Right Metric Statistic
- For diagnostics metrics, choose from:
  - `Average` (most common)
  - `Minimum`
  - `Maximum`
  - `Total`
- Pick the statistic that best reflects your workload behavior.

---

##### Define Thoughtful Thresholds
- Use **distinct thresholds** for scale-out and scale-in to avoid confusion.
- Avoid identical thresholds like:
  - Scale-out: Thread Count ≥ 600
  - Scale-in: Thread Count ≤ 600
- Identical thresholds can cause **flapping**—a loop of scaling in and out.

###### Flapping Example
- Start with 2 instances; thread count rises to 625 → scale-out to 3.
- Thread count drops to 575 → autoscale estimates:
  - 575 × 3 = 1,725 / 2 = 862.5 → triggers scale-out again.
- To prevent this, autoscale **skips scale-in** and reevaluates later.

###### Better Threshold Design
- Scale-out: CPU% ≥ 80
- Scale-in: CPU% ≤ 60
- This margin avoids flapping and ensures stable scaling behavior.

---

##### Handle Multiple Rules in a Profile
- **Scale-out** triggers if **any rule** is met.
- **Scale-in** triggers only if **all rules** are met.

###### Example Rule Set
- Scale-in: CPU < 30%, Memory < 50%
- Scale-out: CPU > 75%, Memory > 75%

###### Behavior
- CPU = 76%, Memory = 50% → scale-out
- CPU = 50%, Memory = 76% → scale-out
- CPU = 25%, Memory = 51% → no scale-in
- CPU = 29%, Memory = 49% → scale-in

---

##### Choose a Safe Default Instance Count
- Autoscale uses the **default count** when metrics are unavailable.
- Set a default that can **safely handle your workload**.

---

##### Configure Autoscale Notifications
- Autoscale logs events in the **Activity Log** when:
  - A scale operation is issued
  - A scale action succeeds or fails
  - Metrics are unavailable or recovered
- Use **Activity Log alerts**, **email**, or **webhooks** to monitor autoscale health and actions.

---

