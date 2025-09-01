# Authorization Explained: When to Use RBAC, ABAC, ACL & More

**Authorization** is the step that comes after authentication.

Once a login request is approved, meaning the system knows who the user is, the next question becomes: What are they allowed to do?

That’s where authorization kicks in. It’s the foundation of controlling access, enforcing security, and protecting privacy across modern systems.

In this post, you’ll learn how real applications manage permissions using the 3 most common authorization models — RBAC, ABAC, and ACL — plus how tools like OAuth2 and JWTs help enforce these rules in production.

If you prefer video format, I cover all of this in detail here: Watch the full Authorization Explained video on YouTube
What Is Authorization?

Authorization is the process of determining which actions or resources a user is allowed to access after they’ve been authenticated.

- **Authentication** = Who is this user?
- **Authorization** = What are they allowed to do?

## The 3 Main Authorization Models

Most systems use one or a combination of the following models:

### 1. Role-Based Access Control (RBAC)

In RBAC, users are assigned roles, and roles have specific permissions.

Example:

    Admin → full access
    Editor → can update content
    Viewer → read-only

RBAC is widely used because it’s simple, scalable, and easy to manage. You’ll see it in systems like Stripe dashboards, CMS tools, and most admin panels.

### 2. Attribute-Based Access Control (ABAC)

ABAC defines access based on user or resource attributes, and even contextual factors like time or location.

Example:

allow if user.department === 'HR' && time < 6PM

This makes it highly flexible. You could:

    Restrict access to features by department
    Limit actions to certain hours
    Combine user and resource metadata for precise policies

Downside? Complexity. You’ll need a policy engine and a way to manage rules at scale.

### 3. Access Control Lists (ACL)

In ACL-based systems, each resource has its own list of permissions for specific users or groups.

Example: In Google Drive, every file has its own ACL defining who can view, comment, or edit.

This approach is highly granular, but harder to scale when managing millions of resources — unless well-abstracted.
How Real Apps Use Authorization

Here’s how popular platforms implement these models:

    GitHub: Combines RBAC (owner, collaborator) with repo-level permissions
    Stripe: Defines roles like developer, support, or billing admin
    Firebase: Uses a rule-based engine where developers define access conditions for each resource (a flexible mix of RBAC + ABAC)

Most large systems mix models to balance flexibility and control.

### Implementing Authorization in Practice

Now that you understand the models, let’s talk about the mechanisms.

This is where things like OAuth2, JWTs, and access tokens come in.

#### OAuth2: Delegated Authorization

OAuth2 is a protocol that lets users grant access to their data in one system to another system without sharing their credentials.

Example: You let a third-party app read your GitHub repos. GitHub doesn’t give them your password; it gives them a token with specific permissions.
OAuth2 defines that flow securely.

This is known as delegated authorization, and it’s used in “Login with Google”, “Connect to Slack”, and many APIs.

#### Token-Based Authorization (JWTs, Bearer Tokens)

After a user logs in, most systems issue an access token (often a JWT) that contains key information like:

    User ID
    Roles or scopes
    Expiration time

When the client sends this token with a request, the backend:

    Verifies it
    Reads the claims
    Checks what actions the user is allowed to perform (using RBAC, ABAC, etc.)

Tokens carry identity, but your backend defines permission logic.

Tokens are a transport mechanism, not a decision-making engine.

#### Putting It All Together

Authorization isn’t just about who got in, it’s about controlling what happens after.

To recap:

    RBAC: Define roles and assign permissions (simple, scalable)
    ABAC: Use attributes and conditions for fine-grained rules
    ACL: Attach access lists to each resource (granular, but harder to scale)
    OAuth2 and JWTs: Enforce those rules across systems and clients

Most real systems blend these models to balance flexibility, performance, and security.

