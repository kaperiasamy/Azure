# Real-time Data Updates: Front-end React to Back-end .NET Core

## Table of Contents
1. [Overview & Architecture](#overview--architecture)
2. [Best Practice Solutions](#best-practice-solutions)
3. [WebSockets with SignalR](#websockets-with-signalr) ⭐ Recommended
4. [Server-Sent Events (SSE)](#server-sent-events-sse)
5. [Polling](#polling)
6. [Comparison](#comparison)
7. [Complete Implementation](#complete-implementation)

---

## Overview & Architecture

### The Problem

```
.NET Core Backend          React Frontend
┌──────────────────┐      ┌──────────────────┐
│ Data Updated     │      │ UI Shows Old     │
│ State Changed    │  ??  │ Data             │
│ Event Triggered  │─────▶│ User Confused    │
└──────────────────┘      └──────────────────┘
```

### Requirements for Real-time Updates

- **Low Latency**: Minimize delay between backend change and UI update
- **Efficiency**: Don't overwhelm the network with constant requests
- **Scalability**: Handle multiple concurrent connections
- **Reliability**: Ensure updates don't get lost
- **User Experience**: Smooth, seamless updates

---

## Best Practice Solutions

| Solution | Latency | Efficiency | Complexity | Scalability | Use Case |
|----------|---------|-----------|-----------|------------|----------|
| **WebSockets (SignalR)** | ✅ Low | ✅ High | ⚠️ Medium | ✅ Excellent | Real-time, bidirectional |
| **Server-Sent Events** | ✅ Low | ✅ High | ✅ Simple | ⚠️ Good | One-way server → client |
| **Long Polling** | ⚠️ Medium | ⚠️ Medium | ✅ Simple | ❌ Poor | Fallback option |
| **Short Polling** | ❌ High | ❌ Low | ✅ Simple | ❌ Poor | Legacy systems |

---

## WebSockets with SignalR ⭐ Recommended

### Why SignalR?

```
✅ Automatic fallback (WebSockets → SSE → Long Polling)
✅ Real-time bidirectional communication
✅ Built-in reconnection logic
✅ Message grouping and broadcasting
✅ Type-safe with TypeScript
✅ Reduced bandwidth
✅ Low latency
```

---

## Complete Implementation

### Architecture Diagram

```
┌─────────────────────────────────────────────────┐
│                  .NET Core Backend              │
│                                                 │
│  ┌────────────────────────────────────────┐    │
│  │ SignalR Hub                            │    │
│  │ - BroadcastDataChange()                │    │
│  │ - NotifyUsers()                        │    │
│  │ - SendUpdateToGroup()                  │    │
│  └────────────────────────────────────────┘    │
│           ▲                                     │
│           │ WebSocket/SSE                      │
│           ▼                                     │
└─────────────────────────────────────────────────┘
           │
           │ Persistent Connection
           │
┌─────────────────────────────────────────────────┐
│                React Frontend                   │
│                                                 │
│  ┌────────────────────────────────────────┐    │
│  │ useSignalR Hook                        │    │
│  │ - Connect to Hub                       │    │
│  │ - Listen for messages                  │    │
│  │ - Update UI State                      │    │
│  └────────────────────────────────────────┘    │
│                                                 │
│  ┌────────────────────────────────────────┐    │
│  │ Components                             │    │
│  │ - Receive updates automatically        │    │
│  │ - Render new data                      │    │
│  └────────────────────────────────────────┘    │
└─────────────────────────────────────────────────┘
```

---

## 1. Backend (.NET Core) Setup

### Step 1: Install NuGet Packages

```bash
dotnet add package Microsoft.AspNetCore.SignalR
dotnet add package Microsoft.AspNetCore.SignalR.Core
```

### Step 2: Create SignalR Hub

```csharp
// Hubs/DataHub.cs
using Microsoft.AspNetCore.SignalR;
using System.Threading.Tasks;
using System.Collections.Generic;

namespace YourApi.Hubs
{
    /// <summary>
    /// SignalR Hub for real-time data updates
    /// </summary>
    public class DataHub : Hub
    {
        // Store connected users/groups
        private static Dictionary<string, UserConnection> UserConnections 
            = new Dictionary<string, UserConnection>();

        /// <summary>
        /// Called when a client connects
        /// </summary>
        public override async Task OnConnectedAsync()
        {
            var connectionId = Context.ConnectionId;
            var userId = Context.User?.FindFirst("sub")?.Value ?? "anonymous";
            
            UserConnections[connectionId] = new UserConnection 
            { 
                ConnectionId = connectionId, 
                UserId = userId 
            };

            // Add user to a group (optional)
            await Groups.AddToGroupAsync(connectionId, $"user_{userId}");
            await Groups.AddToGroupAsync(connectionId, "all_users");

            // Notify all clients that user connected
            await Clients.All.SendAsync("UserConnected", new 
            { 
                userId, 
                connectionId,
                timestamp = DateTime.UtcNow 
            });

            await base.OnConnectedAsync();
        }

        /// <summary>
        /// Called when a client disconnects
        /// </summary>
        public override async Task OnDisconnectedAsync(Exception exception)
        {
            if (UserConnections.TryGetValue(Context.ConnectionId, out var user))
            {
                UserConnections.Remove(Context.ConnectionId);
                
                await Clients.All.SendAsync("UserDisconnected", new 
                { 
                    userId = user.UserId,
                    timestamp = DateTime.UtcNow 
                });
            }

            await base.OnDisconnectedAsync(exception);
        }

        /// <summary>
        /// Broadcast data change to all connected clients
        /// </summary>
        public async Task BroadcastDataChange(DataChangeMessage message)
        {
            await Clients.All.SendAsync("DataUpdated", message);
        }

        /// <summary>
        /// Send update to specific user
        /// </summary>
        public async Task SendToUser(string userId, string messageType, object data)
        {
            await Clients.Group($"user_{userId}")
                .SendAsync(messageType, data);
        }

        /// <summary>
        /// Send update to specific group
        /// </summary>
        public async Task SendToGroup(string groupName, string messageType, object data)
        {
            await Clients.Group(groupName)
                .SendAsync(messageType, data);
        }

        /// <summary>
        /// Join a group (e.g., project group, team group)
        /// </summary>
        public async Task JoinGroup(string groupName)
        {
            await Groups.AddToGroupAsync(Context.ConnectionId, groupName);
            await Clients.Group(groupName).SendAsync("UserJoinedGroup", new 
            { 
                userId = Context.User?.FindFirst("sub")?.Value,
                groupName,
                timestamp = DateTime.UtcNow 
            });
        }

        /// <summary>
        /// Leave a group
        /// </summary>
        public async Task LeaveGroup(string groupName)
        {
            await Groups.RemoveFromGroupAsync(Context.ConnectionId, groupName);
            await Clients.Group(groupName).SendAsync("UserLeftGroup", new 
            { 
                userId = Context.User?.FindFirst("sub")?.Value,
                groupName,
                timestamp = DateTime.UtcNow 
            });
        }

        /// <summary>
        /// Request full data sync (e.g., after reconnection)
        /// </summary>
        public async Task RequestDataSync(string dataType)
        {
            // Implement logic to send full data
            await Clients.Caller.SendAsync("DataSyncRequested", 
                new { dataType, timestamp = DateTime.UtcNow });
        }
    }

    // Models
    public class UserConnection
    {
        public string ConnectionId { get; set; }
        public string UserId { get; set; }
        public DateTime ConnectedAt { get; set; } = DateTime.UtcNow;
    }

    public class DataChangeMessage
    {
        public string ChangeType { get; set; } // "Create", "Update", "Delete"
        public string EntityType { get; set; } // "User", "Product", "Order", etc.
        public int EntityId { get; set; }
        public object NewData { get; set; }
        public string ChangedBy { get; set; }
        public DateTime Timestamp { get; set; } = DateTime.UtcNow;
    }
}
```

### Step 3: Register SignalR in Startup

```csharp
// Program.cs (Startup.cs for older versions)
using YourApi.Hubs;

var builder = WebApplicationBuilder.CreateBuilder(args);

// Add services
builder.Services.AddSignalR(options =>
{
    // Configure SignalR options
    options.MaximumReceiveMessageSize = 1024 * 1024; // 1MB
    options.StreamBufferCapacity = 10;
    options.KeepAliveInterval = TimeSpan.FromSeconds(15);
    options.ClientTimeoutInterval = TimeSpan.FromSeconds(30);
    options.HandshakeTimeout = TimeSpan.FromSeconds(15);
});

builder.Services.AddCors(options =>
{
    options.AddPolicy("CorsPolicy", policy =>
    {
        policy
            .WithOrigins("http://localhost:3000", "https://yourdomain.com")
            .AllowAnyMethod()
            .AllowAnyHeader()
            .AllowCredentials(); // Important for SignalR
    });
});

var app = builder.Build();

app.UseRouting();
app.UseCors("CorsPolicy");

// Map SignalR hub
app.MapHub<DataHub>("/api/hubs/data");

app.Run();
```

### Step 4: Service to Broadcast Changes

```csharp
// Services/DataChangeService.cs
using Microsoft.AspNetCore.SignalR;
using YourApi.Hubs;

namespace YourApi.Services
{
    public interface IDataChangeService
    {
        Task NotifyDataChange(DataChangeMessage message);
        Task NotifyUser(string userId, string messageType, object data);
        Task NotifyGroup(string groupName, string messageType, object data);
    }

    public class DataChangeService : IDataChangeService
    {
        private readonly IHubContext<DataHub> _hubContext;
        private readonly ILogger<DataChangeService> _logger;

        public DataChangeService(
            IHubContext<DataHub> hubContext,
            ILogger<DataChangeService> logger)
        {
            _hubContext = hubContext;
            _logger = logger;
        }

        /// <summary>
        /// Broadcast data change to all connected clients
        /// </summary>
        public async Task NotifyDataChange(DataChangeMessage message)
        {
            try
            {
                _logger.LogInformation(
                    "Broadcasting {EntityType} {ChangeType} to all clients",
                    message.EntityType,
                    message.ChangeType);

                await _hubContext.Clients.All
                    .SendAsync("DataUpdated", message);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error broadcasting data change");
                throw;
            }
        }

        /// <summary>
        /// Send update to specific user
        /// </summary>
        public async Task NotifyUser(string userId, string messageType, object data)
        {
            try
            {
                await _hubContext.Clients.Group($"user_{userId}")
                    .SendAsync(messageType, data);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error notifying user {UserId}", userId);
                throw;
            }
        }

        /// <summary>
        /// Send update to group
        /// </summary>
        public async Task NotifyGroup(string groupName, string messageType, object data)
        {
            try
            {
                await _hubContext.Clients.Group(groupName)
                    .SendAsync(messageType, data);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error notifying group {GroupName}", groupName);
                throw;
            }
        }
    }
}
```

### Step 5: Use in Controllers

```csharp
// Controllers/ProductsController.cs
using Microsoft.AspNetCore.Mvc;
using YourApi.Services;

namespace YourApi.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class ProductsController : ControllerBase
    {
        private readonly IProductService _productService;
        private readonly IDataChangeService _dataChangeService;
        private readonly ILogger<ProductsController> _logger;

        public ProductsController(
            IProductService productService,
            IDataChangeService dataChangeService,
            ILogger<ProductsController> logger)
        {
            _productService = productService;
            _dataChangeService = dataChangeService;
            _logger = logger;
        }

        /// <summary>
        /// Update a product and broadcast change
        /// </summary>
        [HttpPut("{id}")]
        public async Task<IActionResult> UpdateProduct(int id, UpdateProductDto dto)
        {
            try
            {
                var product = await _productService.UpdateProduct(id, dto);

                // Notify all connected clients about the change
                var changeMessage = new DataChangeMessage
                {
                    ChangeType = "Update",
                    EntityType = "Product",
                    EntityId = id,
                    NewData = product,
                    ChangedBy = User?.FindFirst("sub")?.Value ?? "system",
                    Timestamp = DateTime.UtcNow
                };

                await _dataChangeService.NotifyDataChange(changeMessage);

                // Also notify specific group (e.g., product managers)
                await _dataChangeService.NotifyGroup(
                    "product_managers",
                    "ProductUpdated",
                    product);

                return Ok(product);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error updating product {Id}", id);
                return BadRequest(ex.Message);
            }
        }

        /// <summary>
        /// Create a product and broadcast change
        /// </summary>
        [HttpPost]
        public async Task<IActionResult> CreateProduct(CreateProductDto dto)
        {
            try
            {
                var product = await _productService.CreateProduct(dto);

                var changeMessage = new DataChangeMessage
                {
                    ChangeType = "Create",
                    EntityType = "Product",
                    EntityId = product.Id,
                    NewData = product,
                    ChangedBy = User?.FindFirst("sub")?.Value ?? "system",
                    Timestamp = DateTime.UtcNow
                };

                await _dataChangeService.NotifyDataChange(changeMessage);

                return CreatedAtAction(nameof(GetProduct), 
                    new { id = product.Id }, product);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error creating product");
                return BadRequest(ex.Message);
            }
        }

        /// <summary>
        /// Delete a product and broadcast change
        /// </summary>
        [HttpDelete("{id}")]
        public async Task<IActionResult> DeleteProduct(int id)
        {
            try
            {
                await _productService.DeleteProduct(id);

                var changeMessage = new DataChangeMessage
                {
                    ChangeType = "Delete",
                    EntityType = "Product",
                    EntityId = id,
                    ChangedBy = User?.FindFirst("sub")?.Value ?? "system",
                    Timestamp = DateTime.UtcNow
                };

                await _dataChangeService.NotifyDataChange(changeMessage);

                return NoContent();
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error deleting product {Id}", id);
                return BadRequest(ex.Message);
            }
        }

        [HttpGet("{id}")]
        public async Task<IActionResult> GetProduct(int id)
        {
            var product = await _productService.GetProduct(id);
            return Ok(product);
        }
    }
}
```

---

## 2. Frontend (React) Setup

### Step 1: Install SignalR Client

```bash
npm install @microsoft/signalr
```

### Step 2: Create Custom Hook for SignalR

```typescript
// hooks/useSignalR.ts
import { useEffect, useRef, useCallback, useState } from 'react';
import * as SignalR from '@microsoft/signalr';

interface UseSignalROptions {
  url: string;
  accessToken?: string;
  onConnected?: () => void;
  onDisconnected?: () => void;
  onError?: (error: Error) => void;
  autoReconnect?: boolean;
  reconnectTimeout?: number;
}

interface UseSignalRReturn {
  connection: SignalR.HubConnection | null;
  isConnected: boolean;
  isConnecting: boolean;
  error: Error | null;
  connect: () => Promise<void>;
  disconnect: () => Promise<void>;
  on: (methodName: string, callback: (...args: any[]) => void) => void;
  off: (methodName: string, callback?: (...args: any[]) => void) => void;
  invoke: (methodName: string, ...args: any[]) => Promise<any>;
  send: (methodName: string, ...args: any[]) => Promise<void>;
}

/**
 * Custom hook for SignalR connection management
 * 
 * @param options - Configuration options
 * @returns Connection state and methods
 * 
 * @example
 * const { connection, isConnected, on, invoke } = useSignalR({
 *   url: 'http://localhost:5000/api/hubs/data',
 *   accessToken: authToken
 * });
 * 
 * useEffect(() => {
 *   if (!isConnected) return;
 *   
 *   on('DataUpdated', (data) => {
 *     console.log('Data updated:', data);
 *   });
 * }, [isConnected]);
 */
export const useSignalR = ({
  url,
  accessToken,
  onConnected,
  onDisconnected,
  onError,
  autoReconnect = true,
  reconnectTimeout = 5000,
}: UseSignalROptions): UseSignalRReturn => {
  const connectionRef = useRef<SignalR.HubConnection | null>(null);
  const [isConnected, setIsConnected] = useState(false);
  const [isConnecting, setIsConnecting] = useState(false);
  const [error, setError] = useState<Error | null>(null);
  const reconnectTimeoutRef = useRef<NodeJS.Timeout>();

  // Create connection
  const createConnection = useCallback(() => {
    const connection = new SignalR.HubConnectionBuilder()
      .withUrl(url, {
        accessTokenFactory: () => accessToken || '',
        skipNegotiation: false,
        transport:
          SignalR.HttpTransportType.WebSockets |
          SignalR.HttpTransportType.ServerSentEvents |
          SignalR.HttpTransportType.LongPolling,
      })
      .withAutomaticReconnect({
        nextRetryDelayInMilliseconds: (retryCount) => {
          // Exponential backoff: 1s, 3s, 5s, 10s, 30s, 60s, 120s
          if (retryCount === 0) return 1000;
          if (retryCount === 1) return 3000;
          if (retryCount === 2) return 5000;
          if (retryCount === 3) return 10000;
          if (retryCount === 4) return 30000;
          if (retryCount === 5) return 60000;
          return 120000;
        },
      })
      .configureLogging(
        process.env.NODE_ENV === 'development'
          ? SignalR.LogLevel.Information
          : SignalR.LogLevel.Error
      )
      .build();

    // Connection event handlers
    connection.onreconnecting((error) => {
      console.log('SignalR reconnecting...', error);
      setIsConnecting(true);
    });

    connection.onreconnected((connectionId) => {
      console.log('SignalR reconnected', connectionId);
      setIsConnecting(false);
      setIsConnected(true);
      setError(null);
      onConnected?.();
    });

    connection.onclose((error) => {
      console.log('SignalR connection closed', error);
      setIsConnected(false);
      setIsConnecting(false);
      onDisconnected?.();

      // Auto-reconnect
      if (autoReconnect) {
        reconnectTimeoutRef.current = setTimeout(() => {
          console.log('Attempting to reconnect...');
          connection.start().catch((err) => {
            console.error('Reconnection failed:', err);
            onError?.(err);
            setError(err);
          });
        }, reconnectTimeout);
      }
    });

    return connection;
  }, [url, accessToken, autoReconnect, reconnectTimeout, onConnected, onDisconnected, onError]);

  // Connect to hub
  const connect = useCallback(async () => {
    if (isConnected || isConnecting) {
      console.warn('Already connected or connecting');
      return;
    }

    setIsConnecting(true);
    setError(null);

    try {
      if (!connectionRef.current) {
        connectionRef.current = createConnection();
      }

      await connectionRef.current.start();
      setIsConnected(true);
      setIsConnecting(false);
      setError(null);
      onConnected?.();

      console.log('SignalR connected');
    } catch (err) {
      const error = err instanceof Error ? err : new Error(String(err));
      console.error('SignalR connection failed:', error);
      setError(error);
      setIsConnecting(false);
      onError?.(error);
    }
  }, [isConnected, isConnecting, createConnection, onConnected, onError]);

  // Disconnect from hub
  const disconnect = useCallback(async () => {
    if (!connectionRef.current) return;

    try {
      await connectionRef.current.stop();
      setIsConnected(false);
      setError(null);
      onDisconnected?.();
      console.log('SignalR disconnected');
    } catch (err) {
      const error = err instanceof Error ? err : new Error(String(err));
      console.error('Error disconnecting:', error);
      setError(error);
      onError?.(error);
    }
  }, [onDisconnected, onError]);

  // Listen to server method
  const on = useCallback(
    (methodName: string, callback: (...args: any[]) => void) => {
      if (!connectionRef.current) {
        console.warn('Connection not established');
        return;
      }
      connectionRef.current.on(methodName, callback);
    },
    []
  );

  // Stop listening to server method
  const off = useCallback(
    (methodName: string, callback?: (...args: any[]) => void) => {
      if (!connectionRef.current) return;
      if (callback) {
        connectionRef.current.off(methodName, callback);
      } else {
        connectionRef.current.off(methodName);
      }
    },
    []
  );

  // Invoke server method and get result
  const invoke = useCallback(
    async (methodName: string, ...args: any[]) => {
      if (!connectionRef.current || !isConnected) {
        throw new Error('Connection not established');
      }
      return connectionRef.current.invoke(methodName, ...args);
    },
    [isConnected]
  );

  // Send message to server without waiting for response
  const send = useCallback(
    async (methodName: string, ...args: any[]) => {
      if (!connectionRef.current || !isConnected) {
        throw new Error('Connection not established');
      }
      await connectionRef.current.send(methodName, ...args);
    },
    [isConnected]
  );

  // Connect on mount
  useEffect(() => {
    connect();

    // Cleanup on unmount
    return () => {
      if (reconnectTimeoutRef.current) {
        clearTimeout(reconnectTimeoutRef.current);
      }
      disconnect();
    };
  }, [connect, disconnect]);

  return {
    connection: connectionRef.current,
    isConnected,
    isConnecting,
    error,
    connect,
    disconnect,
    on,
    off,
    invoke,
    send,
  };
};

export default useSignalR;
```

### Step 3: Create Context for Global State

```typescript
// contexts/RealtimeContext.tsx
import React, { createContext, useContext, useEffect, useState } from 'react';
import useSignalR from '../hooks/useSignalR';

interface DataChangeMessage {
  changeType: 'Create' | 'Update' | 'Delete';
  entityType: string;
  entityId: number;
  newData?: any;
  changedBy: string;
  timestamp: string;
}

interface RealtimeContextType {
  isConnected: boolean;
  isConnecting: boolean;
  error: Error | null;
  lastUpdate: DataChangeMessage | null;
  subscribe: (eventName: string, callback: (...args: any[]) => void) => void;
  unsubscribe: (eventName: string, callback?: (...args: any[]) => void) => void;
  send: (methodName: string, ...args: any[]) => Promise<void>;
  invoke: (methodName: string, ...args: any[]) => Promise<any>;
}

const RealtimeContext = createContext<RealtimeContextType | undefined>(undefined);

export const RealtimeProvider: React.FC<{ 
  children: React.ReactNode;
  hubUrl: string;
  accessToken?: string;
}> = ({ children, hubUrl, accessToken }) => {
  const [lastUpdate, setLastUpdate] = useState<DataChangeMessage | null>(null);

  const { isConnected, isConnecting, error, on, off, send, invoke } = useSignalR({
    url: hubUrl,
    accessToken,
    onConnected: () => console.log('Realtime connection established'),
    onDisconnected: () => console.log('Realtime connection lost'),
    onError: (error) => console.error('Realtime error:', error),
  });

  // Listen to data updates
  useEffect(() => {
    if (!isConnected) return;

    on('DataUpdated', (message: DataChangeMessage) => {
      setLastUpdate(message);
      console.log('Data updated:', message);
    });

    return () => {
      off('DataUpdated');
    };
  }, [isConnected, on, off]);

  const value: RealtimeContextType = {
    isConnected,
    isConnecting,
    error,
    lastUpdate,
    subscribe: on,
    unsubscribe: off,
    send,
    invoke,
  };

  return (
    <RealtimeContext.Provider value={value}>
      {children}
    </RealtimeContext.Provider>
  );
};

export const useRealtime = () => {
  const context = useContext(RealtimeContext);
  if (!context) {
    throw new Error('useRealtime must be used within RealtimeProvider');
  }
  return context;
};
```

### Step 4: Use in App

```typescript
// App.tsx
import React from 'react';
import { RealtimeProvider } from './contexts/RealtimeContext';
import Dashboard from './pages/Dashboard';

const App: React.FC = () => {
  const token = localStorage.getItem('authToken');

  return (
    <RealtimeProvider
      hubUrl="http://localhost:5000/api/hubs/data"
      accessToken={token}
    >
      <Dashboard />
    </RealtimeProvider>
  );
};

export default App;
```

### Step 5: Use in Components

```typescript
// components/ProductsList.tsx
import React, { useState, useEffect } from 'react';
import { useRealtime } from '../contexts/RealtimeContext';

interface Product {
  id: number;
  name: string;
  price: number;
  updatedAt: string;
}

const ProductsList: React.FC = () => {
  const [products, setProducts] = useState<Product[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const { isConnected, lastUpdate, subscribe, unsubscribe } = useRealtime();

  // Load initial products
  useEffect(() => {
    const loadProducts = async () => {
      setLoading(true);
      try {
        const response = await fetch('http://localhost:5000/api/products');
        const data = await response.json();
        setProducts(data);
        setError(null);
      } catch (err) {
        setError(err instanceof Error ? err.message : 'Failed to load products');
      } finally {
        setLoading(false);
      }
    };

    loadProducts();
  }, []);

  // Listen for real-time updates
  useEffect(() => {
    if (!isConnected) return;

    // Listen to data updates
    const handleDataUpdated = (message: any) => {
      if (message.entityType !== 'Product') return;

      switch (message.changeType) {
        case 'Create':
          setProducts((prev) => [...prev, message.newData]);
          break;

        case 'Update':
          setProducts((prev) =>
            prev.map((p) =>
              p.id === message.entityId
                ? { ...p, ...message.newData }
                : p
            )
          );
          break;

        case 'Delete':
          setProducts((prev) => prev.filter((p) => p.id !== message.entityId));
          break;

        default:
          break;
      }

      // Show notification
      showNotification(message);
    };

    subscribe('DataUpdated', handleDataUpdated);

    return () => {
      unsubscribe('DataUpdated', handleDataUpdated);
    };
  }, [isConnected, subscribe, unsubscribe]);

  const showNotification = (message: any) => {
    const notification = document.createElement('div');
    notification.className = 'notification';
    notification.textContent = `Product ${message.changeType.toLowerCase()}: ${message.newData?.name || message.entityId}`;
    notification.style.cssText = `
      position: fixed;
      top: 20px;
      right: 20px;
      padding: 15px 20px;
      background: #4CAF50;
      color: white;
      border-radius: 4px;
      z-index: 1000;
      animation: slideIn 0.3s ease-out;
    `;
    document.body.appendChild(notification);
    setTimeout(() => notification.remove(), 3000);
  };

  if (loading) return <div>Loading products...</div>;
  if (error) return <div className="error">{error}</div>;

  return (
    <div>
      <div className="status">
        SignalR Status:{' '}
        <span className={isConnected ? 'connected' : 'disconnected'}>
          {isConnected ? '🟢 Connected' : '🔴 Disconnected'}
        </span>
      </div>

      <h2>Products</h2>
      <table>
        <thead>
          <tr>
            <th>ID</th>
            <th>Name</th>
            <th>Price</th>
            <th>Updated</th>
          </tr>
        </thead>
        <tbody>
          {products.map((product) => (
            <tr
              key={product.id}
              className={
                lastUpdate?.entityId === product.id &&
                lastUpdate?.entityType === 'Product'
                  ? 'highlight'
                  : ''
              }
            >
              <td>{product.id}</td>
              <td>{product.name}</td>
              <td>${product.price}</td>
              <td>{new Date(product.updatedAt).toLocaleString()}</td>
            </tr>
          ))}
        </tbody>
      </table>

      <style>{`
        .status {
          padding: 10px;
          margin-bottom: 20px;
          background: #f5f5f5;
          border-radius: 4px;
        }

        .connected {
          color: green;
          font-weight: bold;
        }

        .disconnected {
          color: red;
          font-weight: bold;
        }

        table {
          width: 100%;
          border-collapse: collapse;
        }

        th, td {
          padding: 10px;
          text-align: left;
          border-bottom: 1px solid #ddd;
        }

        th {
          background-color: #f2f2f2;
          font-weight: bold;
        }

        tr.highlight {
          background-color: #fff3cd;
          animation: highlight 0.5s ease-out;
        }

        @keyframes highlight {
          from {
            background-color: #90EE90;
          }
          to {
            background-color: #fff3cd;
          }
        }

        .error {
          color: red;
          padding: 10px;
          background: #ffe0e0;
          border-radius: 4px;
        }

        .notification {
          @keyframes slideIn {
            from {
              transform: translateX(100%);
              opacity: 0;
            }
            to {
              transform: translateX(0);
              opacity: 1;
            }
          }
        }
      `}</style>
    </div>
  );
};

export default ProductsList;
```

### Step 6: Advanced Component with Subscriptions

```typescript
// components/OrderMonitor.tsx
import React, { useEffect, useState } from 'react';
import { useRealtime } from '../contexts/RealtimeContext';

const OrderMonitor: React.FC<{ orderId: number }> = ({ orderId }) => {
  const [order, setOrder] = useState<any>(null);
  const { isConnected, invoke } = useRealtime();

  useEffect(() => {
    if (!isConnected) return;

    // Join order-specific group on connect
    invoke('JoinGroup', `order_${orderId}`)
      .then(() => console.log(`Joined order_${orderId} group`))
      .catch((err) => console.error('Failed to join group:', err));

    return () => {
      // Leave group on unmount
      invoke('LeaveGroup', `order_${orderId}`)
        .catch((err) => console.error('Failed to leave group:', err));
    };
  }, [isConnected, orderId, invoke]);

  return (
    <div>
      <h3>Order #{orderId}</h3>
      {order ? (
        <div>
          <p>Status: {order.status}</p>
          <p>Last Updated: {new Date(order.updatedAt).toLocaleString()}</p>
        </div>
      ) : (
        <p>Loading order details...</p>
      )}
    </div>
  );
};

export default OrderMonitor;
```

---

## Server-Sent Events (SSE)

### When to use SSE:
- One-way communication (server → client)
- Simpler than WebSockets
- Real-time updates needed
- Don't need bidirectional communication

### Backend Implementation

```csharp
// Controllers/EventsController.cs
[ApiController]
[Route("api/[controller]")]
public class EventsController : ControllerBase
{
    private static readonly ConcurrentDictionary<string, ChannelWriter<string>> 
        Clients = new();

    [HttpGet("subscribe/{userId}")]
    public async Task Subscribe(string userId, CancellationToken cancellationToken)
    {
        Response.ContentType = "text/event-stream";
        Response.Headers.Add("Cache-Control", "no-cache");

        var channel = Channel.CreateUnbounded<string>();
        Clients.TryAdd(userId, channel.Writer);

        try
        {
            await foreach (var message in channel.Reader.ReadAllAsync(cancellationToken))
            {
                await Response.WriteAsync($"data: {message}\n\n", cancellationToken);
                await Response.Body.FlushAsync(cancellationToken);
            }
        }
        finally
        {
            Clients.TryRemove(userId, out _);
        }
    }

    // Broadcast method
    public static async Task BroadcastToUser(string userId, string message)
    {
        if (Clients.TryGetValue(userId, out var writer))
        {
            await writer.WriteAsync(message);
        }
    }
}
```

### Frontend Implementation

```typescript
// hooks/useSSE.ts
import { useEffect, useState } from 'react';

export const useSSE = (url: string, userId: string) => {
  const [data, setData] = useState<any>(null);
  const [isConnected, setIsConnected] = useState(false);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    const eventSource = new EventSource(`${url}?userId=${userId}`);

    eventSource.onopen = () => {
      setIsConnected(true);
      setError(null);
    };

    eventSource.onmessage = (event) => {
      try {
        const message = JSON.parse(event.data);
        setData(message);
      } catch (err) {
        console.error('Failed to parse SSE message:', err);
      }
    };

    eventSource.onerror = () => {
      setIsConnected(false);
      setError(new Error('SSE connection failed'));
    };

    return () => {
      eventSource.close();
    };
  }, [url, userId]);

  return { data, isConnected, error };
};
```

---

## Comparison

| Feature | SignalR (WebSocket) | SSE | Long Polling |
|---------|------------------|-----|--------------|
| **Latency** | ⚠️ ~100ms | ⚠️ ~100ms | ❌ ~1-5s |
| **Bandwidth** | ✅ Low | ✅ Low | ❌ High |
| **Complexity** | ⚠️ Medium | ✅ Simple | ✅ Simple |
| **Bidirectional** | ✅ Yes | ❌ One-way | ⚠️ Possible |
| **Browser Support** | ✅ Excellent | ✅ Excellent | ✅ Full |
| **Firewall Friendly** | ⚠️ Port 443 | ✅ HTTP | ✅ HTTP |
| **Scalability** | ✅ Excellent | ✅ Good | ❌ Limited |
| **Connection Reuse** | ✅ Yes | ❌ New per client | ❌ New per client |
| **Best Use Case** | Real-time dashboards, chat, multiplayer games | Server notifications, real-time updates | Legacy systems, simple notifications |

---

## Best Practices Checklist

### ✅ Backend
- [ ] Implement proper error handling
- [ ] Add authentication/authorization to hub
- [ ] Use groups for targeted messaging
- [ ] Implement reconnection logic
- [ ] Handle client disconnects
- [ ] Log all significant events
- [ ] Use dependency injection for services
- [ ] Test with multiple concurrent connections

### ✅ Frontend
- [ ] Implement automatic reconnection
- [ ] Handle connection errors gracefully
- [ ] Unsubscribe from events on cleanup
- [ ] Show connection status to users
- [ ] Debounce rapid updates
- [ ] Implement retry logic
- [ ] Use TypeScript for type safety
- [ ] Test with network throttling

### ✅ Network
- [ ] Use secure WebSocket (WSS) in production
- [ ] Configure CORS properly
- [ ] Set appropriate timeouts
- [ ] Monitor connection health
- [ ] Implement rate limiting
- [ ] Use load balancing for scale
- [ ] Configure sticky sessions if load balancing

### ✅ Monitoring
- [ ] Log connection events
- [ ] Monitor active connections
- [ ] Track message throughput
- [ ] Alert on connection failures
- [ ] Monitor network latency
- [ ] Track error rates

---

## Production Deployment Checklist

```csharp
// Production-ready setup
public void ConfigureSignalR(IServiceCollection services)
{
    services.AddSignalR(options =>
    {
        // Reduce message size
        options.MaximumReceiveMessageSize = 5 * 1024 * 1024;
        
        // Increase timeouts for poor networks
        options.KeepAliveInterval = TimeSpan.FromSeconds(30);
        options.ClientTimeoutInterval = TimeSpan.FromSeconds(60);
        options.HandshakeTimeout = TimeSpan.FromSeconds(30);
        
        // Disable negotiation for better performance
        options.TransportMaxBufferSize = 10 * 1024 * 1024;
        options.StreamBufferCapacity = 20;
    });

    // Use Azure SignalR Service for cloud scale
    services.AddSignalR()
        .AddAzureSignalR(options =>
        {
            options.ServerStickyMode = ServerStickyMode.Required;
        });

    // Add logging
    services.AddLogging(builder =>
    {
        builder.AddApplicationInsights();
    });
}
```

---

## Summary

### Recommended Solution: **SignalR** ⭐

**Why SignalR is best for real-time React + .NET Core:**

1. **True bidirectional communication** - Essential for complex apps
2. **Automatic transport fallback** - Works everywhere
3. **Built-in reconnection** - Handles network issues automatically
4. **Groups/broadcasting** - Easy to implement complex messaging patterns
5. **Performance** - Minimal overhead, high throughput
6. **Developer experience** - Easy to learn and use
7. **Production-ready** - Used by Microsoft and thousands of companies
8. **Scalability** - Can scale horizontally with Azure SignalR Service

### Quick Start:
1. Install `@microsoft/signalr` on frontend
2. Create SignalR Hub in .NET Core
3. Register in Startup
4. Use `useSignalR` hook in React components
5. Listen and update UI state

This approach provides the best balance of performance, reliability, and developer experience for real-time updates in React applications! 🚀