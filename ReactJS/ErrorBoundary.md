# Error Boundaries in React - Complete Guide

## 🎯 What is Error Boundary?

**Error Boundary** is a React component that catches JavaScript errors anywhere in the child component tree, logs those errors, and displays a fallback UI instead of crashing the entire application.

### **Think of it as:**
- A **try-catch block for React components**
- A **safety net** that prevents the entire app from crashing
- A **graceful degradation** mechanism for production apps

---

## 🔍 Why Error Boundaries are Useful

### **Without Error Boundary:**
```jsx
// ❌ One error crashes the entire app
const App = () => {
  return (
    <div>
      <Header />
      <Sidebar />
      <BuggyComponent />  {/* Error here crashes everything */}
      <MainContent />
      <Footer />
    </div>
  );
};

// Result: White screen of death! User sees nothing.
```

### **With Error Boundary:**
```jsx
// ✅ Error is contained, rest of app still works
const App = () => {
  return (
    <div>
      <Header />
      <Sidebar />
      <ErrorBoundary fallback={<div>Sidebar failed, but app works</div>}>
        <BuggyComponent />
      </ErrorBoundary>
      <MainContent />
      <Footer />
    </div>
  );
};

// Result: Error message shown, rest of app functional
```

---

## 📋 What Errors Do Error Boundaries Catch?

### **✅ CATCHES:**
- Errors in **render methods**
- Errors in **lifecycle methods**
- Errors in **constructors**
- Errors in **child component tree**

### **❌ DOES NOT CATCH:**
- Errors in **event handlers** (use try-catch)
- Errors in **async code** (setTimeout, promises)
- Errors in **server-side rendering**
- Errors thrown in the **error boundary itself**
- Errors in **event handlers**

---

## 💻 Implementation

### **1. Basic Error Boundary**

```jsx
import React from 'react';

class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { 
      hasError: false,
      error: null,
      errorInfo: null
    };
  }

  // Called when error is thrown in child component
  static getDerivedStateFromError(error) {
    // Update state to show fallback UI
    return { hasError: true };
  }

  // Called after error is caught - for logging
  componentDidCatch(error, errorInfo) {
    // Log error details
    console.error('Error caught by boundary:', error);
    console.error('Error info:', errorInfo);
    
    // Update state with error details
    this.setState({
      error: error,
      errorInfo: errorInfo
    });
    
    // Send to error tracking service
    // logErrorToMyService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // Fallback UI
      return (
        <div style={{ padding: '20px', border: '2px solid red' }}>
          <h2>😔 Something went wrong</h2>
          <p>We're sorry for the inconvenience.</p>
          <details style={{ whiteSpace: 'pre-wrap', marginTop: '10px' }}>
            <summary>Click for error details</summary>
            <p>{this.state.error?.toString()}</p>
            <p>{this.state.errorInfo?.componentStack}</p>
          </details>
        </div>
      );
    }

    // No error, render children normally
    return this.props.children;
  }
}

export default ErrorBoundary;
```

### **2. Usage in Application**

```jsx
import ErrorBoundary from './ErrorBoundary';

const App = () => {
  return (
    <div className="app">
      {/* Wrap entire app */}
      <ErrorBoundary>
        <Header />
        <MainContent />
        <Footer />
      </ErrorBoundary>
    </div>
  );
};

// Or wrap specific components
const Dashboard = () => {
  return (
    <div>
      <ErrorBoundary>
        <Sidebar />
      </ErrorBoundary>
      
      <ErrorBoundary>
        <MainPanel />
      </ErrorBoundary>
      
      <ErrorBoundary>
        <Notifications />
      </ErrorBoundary>
    </div>
  );
};
```

---

## 🚀 Advanced Error Boundary Features

### **3. With Custom Fallback Component**

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    // Log to monitoring service
    if (window.appInsights) {
      window.appInsights.trackException({
        error: error,
        properties: {
          componentStack: errorInfo.componentStack,
          timestamp: new Date().toISOString()
        }
      });
    }
  }

  render() {
    if (this.state.hasError) {
      // Use custom fallback from props
      if (this.props.fallback) {
        return this.props.fallback;
      }
      
      // Or use fallback render prop
      if (this.props.FallbackComponent) {
        return (
          <this.props.FallbackComponent 
            error={this.state.error}
            resetError={() => this.setState({ hasError: false })}
          />
        );
      }
      
      // Default fallback
      return <h2>Something went wrong</h2>;
    }

    return this.props.children;
  }
}

// Usage with custom fallback
<ErrorBoundary fallback={<CustomErrorPage />}>
  <MyComponent />
</ErrorBoundary>

// Usage with fallback component
<ErrorBoundary FallbackComponent={ErrorPage}>
  <MyComponent />
</ErrorBoundary>

// Custom error page component
const ErrorPage = ({ error, resetError }) => {
  return (
    <div className="error-page">
      <h1>Oops! Something went wrong</h1>
      <p>{error?.message}</p>
      <button onClick={resetError}>Try Again</button>
      <button onClick={() => window.location.href = '/'}>Go Home</button>
    </div>
  );
};
```

### **4. Error Boundary with Retry Logic**

```jsx
class ErrorBoundaryWithRetry extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      hasError: false,
      error: null,
      errorInfo: null,
      retryCount: 0
    };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    this.setState({ errorInfo });
    
    console.error('Error:', error);
    console.error('Component Stack:', errorInfo.componentStack);
  }

  resetError = () => {
    this.setState(prevState => ({
      hasError: false,
      error: null,
      errorInfo: null,
      retryCount: prevState.retryCount + 1
    }));
  };

  render() {
    const { hasError, error, retryCount } = this.state;
    const { maxRetries = 3 } = this.props;

    if (hasError) {
      if (retryCount >= maxRetries) {
        return (
          <div className="error-max-retries">
            <h2>Unable to load component</h2>
            <p>Maximum retry attempts reached ({maxRetries})</p>
            <p>Error: {error?.message}</p>
            <button onClick={() => window.location.reload()}>
              Reload Page
            </button>
          </div>
        );
      }

      return (
        <div className="error-retry">
          <h2>Something went wrong</h2>
          <p>Error: {error?.message}</p>
          <p>Retry attempt: {retryCount} / {maxRetries}</p>
          <button onClick={this.resetError}>
            Retry ({maxRetries - retryCount} attempts remaining)
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

// Usage
<ErrorBoundaryWithRetry maxRetries={3}>
  <PotentiallyFailingComponent />
</ErrorBoundaryWithRetry>
```

### **5. Error Boundary with Monitoring Integration**

```jsx
class ErrorBoundaryWithMonitoring extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, errorId: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // Generate unique error ID
    const errorId = `ERR-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
    
    this.setState({ errorId });

    // Log to multiple services
    this.logToServices(error, errorInfo, errorId);
  }

  logToServices = (error, errorInfo, errorId) => {
    // 1. Azure Application Insights
    if (window.appInsights) {
      window.appInsights.trackException({
        error: error,
        severityLevel: 3, // Error
        properties: {
          errorId: errorId,
          componentStack: errorInfo.componentStack,
          userId: this.getUserId(),
          sessionId: this.getSessionId(),
          url: window.location.href,
          userAgent: navigator.userAgent,
          timestamp: new Date().toISOString()
        }
      });
    }

    // 2. Sentry
    if (window.Sentry) {
      window.Sentry.withScope((scope) => {
        scope.setTag('errorId', errorId);
        scope.setContext('componentStack', {
          stack: errorInfo.componentStack
        });
        window.Sentry.captureException(error);
      });
    }

    // 3. Custom logging endpoint (.NET Core API)
    fetch('/api/errors/log', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        errorId,
        message: error.message,
        stack: error.stack,
        componentStack: errorInfo.componentStack,
        timestamp: new Date().toISOString(),
        userAgent: navigator.userAgent,
        url: window.location.href
      })
    }).catch(err => console.error('Failed to log error:', err));

    // 4. Console for development
    if (process.env.NODE_ENV === 'development') {
      console.group(`🔴 Error Boundary Caught Error (ID: ${errorId})`);
      console.error('Error:', error);
      console.error('Component Stack:', errorInfo.componentStack);
      console.groupEnd();
    }
  };

  getUserId = () => {
    // Get from auth context or localStorage
    return localStorage.getItem('userId') || 'anonymous';
  };

  getSessionId = () => {
    return sessionStorage.getItem('sessionId') || 'no-session';
  };

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-boundary-ui">
          <h2>We're experiencing technical difficulties</h2>
          <p>Our team has been notified and is working on a fix.</p>
          <p className="error-id">Error ID: {this.state.errorId}</p>
          <p className="help-text">
            Please contact support with the above Error ID if the issue persists.
          </p>
          <button onClick={() => window.location.reload()}>
            Reload Page
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}
```

---

## 🎯 Real-World Use Cases

### **6. Route-Level Error Boundaries**

```jsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';

const App = () => {
  return (
    <BrowserRouter>
      <Routes>
        {/* Each route has its own error boundary */}
        <Route 
          path="/dashboard" 
          element={
            <ErrorBoundary>
              <Dashboard />
            </ErrorBoundary>
          } 
        />
        
        <Route 
          path="/profile" 
          element={
            <ErrorBoundary>
              <Profile />
            </ErrorBoundary>
          } 
        />
        
        <Route 
          path="/settings" 
          element={
            <ErrorBoundary>
              <Settings />
            </ErrorBoundary>
          } 
        />
      </Routes>
    </BrowserRouter>
  );
};
```

### **7. Granular Error Boundaries**

```jsx
const Dashboard = () => {
  return (
    <div className="dashboard">
      {/* Header can fail independently */}
      <ErrorBoundary fallback={<div>Header unavailable</div>}>
        <DashboardHeader />
      </ErrorBoundary>

      <div className="dashboard-content">
        {/* Sidebar can fail independently */}
        <ErrorBoundary fallback={<div>Sidebar unavailable</div>}>
          <Sidebar />
        </ErrorBoundary>

        {/* Main content can fail independently */}
        <ErrorBoundary fallback={<div>Content unavailable</div>}>
          <MainContent />
        </ErrorBoundary>

        {/* Widgets can fail independently */}
        <ErrorBoundary fallback={<div>Widget unavailable</div>}>
          <AnalyticsWidget />
        </ErrorBoundary>
      </div>

      {/* Footer always works */}
      <Footer />
    </div>
  );
};
```

### **8. Error Boundary for Lazy Loaded Components**

```jsx
import { Suspense, lazy } from 'react';

const LazyDashboard = lazy(() => import('./Dashboard'));
const LazyProfile = lazy(() => import('./Profile'));

const App = () => {
  return (
    <div>
      {/* Handle both loading and error states */}
      <ErrorBoundary 
        fallback={
          <div>
            <h2>Failed to load component</h2>
            <p>Please check your connection and try again</p>
          </div>
        }
      >
        <Suspense fallback={<LoadingSpinner />}>
          <Routes>
            <Route path="/dashboard" element={<LazyDashboard />} />
            <Route path="/profile" element={<LazyProfile />} />
          </Routes>
        </Suspense>
      </ErrorBoundary>
    </div>
  );
};
```

---

## 🛠️ Handling Errors NOT Caught by Error Boundaries

### **9. Event Handler Errors**

```jsx
const ComponentWithEventHandlers = () => {
  const [error, setError] = useState(null);

  const handleClick = () => {
    try {
      // Code that might throw
      riskyOperation();
    } catch (error) {
      // Error boundaries don't catch this - handle manually
      setError(error.message);
      
      // Log to monitoring service
      logError(error);
    }
  };

  const handleAsyncOperation = async () => {
    try {
      await fetchData();
    } catch (error) {
      // Async errors not caught by error boundary
      setError(error.message);
      logError(error);
    }
  };

  if (error) {
    return <div className="error">Error: {error}</div>;
  }

  return (
    <div>
      <button onClick={handleClick}>Risky Operation</button>
      <button onClick={handleAsyncOperation}>Async Operation</button>
    </div>
  );
};
```

### **10. Global Error Handler**

```jsx
// In your index.js or App.js
useEffect(() => {
  // Catch unhandled errors globally
  const handleError = (event) => {
    console.error('Global error caught:', event.error);
    
    // Log to monitoring service
    logErrorToService({
      message: event.error.message,
      stack: event.error.stack,
      type: 'unhandled_error'
    });
  };

  // Catch unhandled promise rejections
  const handleRejection = (event) => {
    console.error('Unhandled promise rejection:', event.reason);
    
    logErrorToService({
      message: event.reason?.message || event.reason,
      type: 'unhandled_rejection'
    });
  };

  window.addEventListener('error', handleError);
  window.addEventListener('unhandledrejection', handleRejection);

  return () => {
    window.removeEventListener('error', handleError);
    window.removeEventListener('unhandledrejection', handleRejection);
  };
}, []);
```

---

## 📊 Best Practices

### **✅ DO:**

1. **Use multiple error boundaries** for granular error handling
```jsx
<ErrorBoundary> {/* App-level */}
  <Header />
  <ErrorBoundary> {/* Feature-level */}
    <ComplexFeature />
  </ErrorBoundary>
</ErrorBoundary>
```

2. **Provide helpful error messages** to users
```jsx
<div>
  <h2>We're sorry, something went wrong</h2>
  <p>Our team has been notified. Please try refreshing the page.</p>
  <button onClick={refresh}>Refresh</button>
</div>
```

3. **Log errors to monitoring services**
```jsx
componentDidCatch(error, errorInfo) {
  // Azure Application Insights
  appInsights.trackException({ error });
  
  // Sentry
  Sentry.captureException(error);
  
  // Custom logging
  logToBackend(error, errorInfo);
}
```

4. **Include retry mechanism**
```jsx
<button onClick={resetError}>Try Again</button>
```

5. **Test error boundaries**
```jsx
// In development, create a component that throws on purpose
const BuggyComponent = () => {
  throw new Error('Test error boundary');
};
```

### **❌ DON'T:**

1. **Don't use one error boundary for everything**
```jsx
// ❌ BAD: One error crashes entire app
<ErrorBoundary>
  <EntireApp />
</ErrorBoundary>
```

2. **Don't ignore error details**
```jsx
// ❌ BAD: No logging
componentDidCatch(error, errorInfo) {
  this.setState({ hasError: true });
  // Missing: Log to monitoring service!
}
```

3. **Don't show technical errors to users**
```jsx
// ❌ BAD: Exposing stack trace
<div>{error.stack}</div>

// ✅ GOOD: User-friendly message
<div>Something went wrong. Please try again.</div>
```

4. **Don't forget development vs production**
```jsx
// Show detailed errors in development
if (process.env.NODE_ENV === 'development') {
  return <DetailedError error={error} />;
}

// User-friendly errors in production
return <SimpleError />;
```

---

## 🎯 Complete Production Example

```jsx
// ProductionErrorBoundary.jsx
import React from 'react';

class ProductionErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      hasError: false,
      error: null,
      errorInfo: null,
      errorId: null,
      retryCount: 0
    };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    const errorId = this.generateErrorId();
    
    this.setState({ errorInfo, errorId });
    
    // Multi-service logging
    this.logToAllServices(error, errorInfo, errorId);
  }

  generateErrorId = () => {
    return `ERR-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  };

  logToAllServices = (error, errorInfo, errorId) => {
    const errorData = {
      errorId,
      message: error.message,
      stack: error.stack,
      componentStack: errorInfo.componentStack,
      url: window.location.href,
      userAgent: navigator.userAgent,
      timestamp: new Date().toISOString(),
      userId: this.getUserId(),
      appVersion: process.env.REACT_APP_VERSION
    };

    // Azure Application Insights
    if (window.appInsights) {
      window.appInsights.trackException({
        error,
        properties: errorData,
        severityLevel: 3
      });
    }

    // .NET Core API logging
    fetch('/api/errors/client', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(errorData)
    }).catch(console.error);

    // Development logging
    if (process.env.NODE_ENV === 'development') {
      console.group('🔴 Error Boundary');
      console.error('Error:', error);
      console.error('Component Stack:', errorInfo.componentStack);
      console.error('Error ID:', errorId);
      console.groupEnd();
    }
  };

  getUserId = () => {
    try {
      return localStorage.getItem('userId') || 'anonymous';
    } catch {
      return 'anonymous';
    }
  };

  handleReset = () => {
    this.setState(prev => ({
      hasError: false,
      error: null,
      errorInfo: null,
      retryCount: prev.retryCount + 1
    }));
  };

  render() {
    const { hasError, error, errorId, retryCount } = this.state;
    const { maxRetries = 3, fallback } = this.props;

    if (hasError) {
      // Custom fallback component
      if (fallback) {
        return fallback;
      }

      // Max retries reached
      if (retryCount >= maxRetries) {
        return (
          <div className="error-boundary-max-retries">
            <h2>Unable to load this section</h2>
            <p>Maximum retry attempts reached.</p>
            <p className="error-id">Error ID: {errorId}</p>
            <button onClick={() => window.location.reload()}>
              Reload Page
            </button>
            <a href="/">Go to Homepage</a>
          </div>
        );
      }

      // Default error UI with retry
      return (
        <div className="error-boundary-ui">
          <div className="error-icon">⚠️</div>
          <h2>Something went wrong</h2>
          <p>We're experiencing technical difficulties with this section.</p>
          
          {process.env.NODE_ENV === 'development' && (
            <details>
              <summary>Error Details (Development Only)</summary>
              <pre>{error?.message}</pre>
              <pre>{error?.stack}</pre>
            </details>
          )}
          
          <div className="error-actions">
            <button onClick={this.handleReset} className="btn-primary">
              Try Again ({maxRetries - retryCount} attempts left)
            </button>
            <button onClick={() => window.location.reload()} className="btn-secondary">
              Reload Page
            </button>
          </div>
          
          <p className="error-id">
            Error ID: <code>{errorId}</code>
          </p>
          <p className="help-text">
            If this problem persists, please contact support with the Error ID above.
          </p>
        </div>
      );
    }

    return this.props.children;
  }
}

export default ProductionErrorBoundary;
```

---

## 🎓 Interview Questions & Answers

### **Q1: Why must Error Boundaries be class components?**
**A:** Error boundaries use lifecycle methods `getDerivedStateFromError` and `componentDidCatch` which are only available in class components. There's currently no Hook equivalent for catching errors in functional components.

### **Q2: What happens if an error boundary itself throws an error?**
**A:** The error will propagate to the next error boundary up the tree. If there's no parent error boundary, the entire app will unmount (white screen).

### **Q3: How do you test error boundaries?**
**A:** Create a component that intentionally throws errors, or use testing libraries:
```jsx
// Test component
const ThrowError = () => {
  throw new Error('Test error');
};

// In test
render(
  <ErrorBoundary>
    <ThrowError />
  </ErrorBoundary>
);

expect(screen.getByText(/something went wrong/i)).toBeInTheDocument();
```

---

## 📝 Summary

**Error Boundaries:**
- ✅ Prevent entire app crashes
- ✅ Provide graceful degradation
- ✅ Enable error logging and monitoring
- ✅ Improve user experience
- ✅ Support granular error handling
- ❌ Must be class components
- ❌ Don't catch event handler errors
- ❌ Don't catch async errors

**Key Takeaway:** Error Boundaries are essential for production React applications, providing a safety net that keeps your app running even when individual components fail.

--- 

# Error Boundaries - Functional Component Alternatives

## 🎯 Why Error Boundaries Must Be Class Components

### **The Core Limitation:**

Error Boundaries **MUST** be class components because:

1. **`getDerivedStateFromError`** - Only available in class components
2. **`componentDidCatch`** - Only available in class components
3. **No Hook equivalent exists** - React team hasn't created `useErrorBoundary` hook yet

```jsx
// ❌ THIS DOESN'T EXIST (yet)
const ErrorBoundary = ({ children }) => {
  const [hasError, setHasError] = useErrorBoundary(); // No such hook!
  // ...
};
```

---

## ✅ Solutions: Functional Component Patterns

While the **Error Boundary itself must be a class**, you can:
1. Use **class-based error boundary** with **functional fallback components**
2. Use **third-party libraries** that provide hooks
3. Use **manual error handling** in functional components
4. Create **hybrid approaches**

---

## 1️⃣ **Recommended: Class Error Boundary + Functional Fallback**

```jsx
// ========================================
// ErrorBoundary.jsx (Class - Required)
// ========================================
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Error:', error, errorInfo);
    this.props.onError?.(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // Use functional component for fallback UI
      return (
        <this.props.FallbackComponent 
          error={this.state.error}
          resetError={() => this.setState({ hasError: false })}
        />
      );
    }

    return this.props.children;
  }
}

// ========================================
// ErrorFallback.jsx (Functional Component)
// ========================================
const ErrorFallback = ({ error, resetError }) => {
  const [retryCount, setRetryCount] = useState(0);
  const [errorDetails, setErrorDetails] = useState(null);

  // Functional component can have all the complex logic!
  useEffect(() => {
    // Log to monitoring service
    if (error) {
      logErrorToService({
        message: error.message,
        stack: error.stack,
        timestamp: new Date().toISOString()
      });

      setErrorDetails({
        errorId: generateErrorId(),
        userAgent: navigator.userAgent,
        url: window.location.href
      });
    }
  }, [error]);

  const handleRetry = () => {
    setRetryCount(prev => prev + 1);
    resetError();
  };

  return (
    <div className="error-fallback">
      <div className="error-icon">⚠️</div>
      <h2>Oops! Something went wrong</h2>
      <p>{error?.message}</p>
      
      {errorDetails && (
        <p className="error-id">Error ID: {errorDetails.errorId}</p>
      )}
      
      <div className="error-actions">
        <button onClick={handleRetry}>
          Try Again (Attempt {retryCount + 1})
        </button>
        <button onClick={() => window.location.href = '/'}>
          Go Home
        </button>
      </div>

      {process.env.NODE_ENV === 'development' && (
        <details>
          <summary>Error Details</summary>
          <pre>{error?.stack}</pre>
        </details>
      )}
    </div>
  );
};

// ========================================
// Usage (Fully Functional!)
// ========================================
const App = () => {
  const handleError = (error, errorInfo) => {
    // This handler is in functional component
    console.error('Caught error:', error);
    
    // Send to Azure Application Insights
    if (window.appInsights) {
      window.appInsights.trackException({ error });
    }
  };

  return (
    <ErrorBoundary 
      FallbackComponent={ErrorFallback}
      onError={handleError}
    >
      <Dashboard />
    </ErrorBoundary>
  );
};

const Dashboard = () => {
  const [data, setData] = useState(null);
  
  // All your functional component logic here
  useEffect(() => {
    fetchData();
  }, []);

  return <div>{/* Your app */}</div>;
};
```

---

## 2️⃣ **Using `react-error-boundary` Library (Best Practice)**

This library provides a **cleaner API** with hooks support:

```bash
npm install react-error-boundary
```

```jsx
import { ErrorBoundary, useErrorHandler } from 'react-error-boundary';

// ========================================
// Functional Fallback Component
// ========================================
const ErrorFallback = ({ error, resetErrorBoundary }) => {
  const [copied, setCopied] = useState(false);

  const copyErrorId = () => {
    const errorId = `ERR-${Date.now()}`;
    navigator.clipboard.writeText(errorId);
    setCopied(true);
  };

  return (
    <div role="alert" className="error-fallback">
      <h2>Something went wrong:</h2>
      <pre style={{ color: 'red' }}>{error.message}</pre>
      
      <div className="button-group">
        <button onClick={resetErrorBoundary}>Try again</button>
        <button onClick={copyErrorId}>
          {copied ? 'Copied!' : 'Copy Error ID'}
        </button>
      </div>
    </div>
  );
};

// ========================================
// App with Error Boundary
// ========================================
const App = () => {
  const handleError = (error, errorInfo) => {
    // Log to your service
    console.error('Logged error:', error);
    logErrorToService(error, errorInfo);
  };

  const handleReset = () => {
    // Called when user clicks retry
    console.log('Error boundary reset');
  };

  return (
    <ErrorBoundary
      FallbackComponent={ErrorFallback}
      onError={handleError}
      onReset={handleReset}
      resetKeys={['someKey']} // Reset when these values change
    >
      <MainApp />
    </ErrorBoundary>
  );
};

// ========================================
// Functional Component with Error Handling
// ========================================
const UserProfile = ({ userId }) => {
  const [user, setUser] = useState(null);
  
  // Hook to manually trigger error boundary
  const handleError = useErrorHandler();

  useEffect(() => {
    fetchUser(userId)
      .then(setUser)
      .catch(error => {
        // Manually trigger error boundary from functional component!
        handleError(error);
      });
  }, [userId, handleError]);

  return (
    <div>
      <h1>{user?.name}</h1>
    </div>
  );
};

// ========================================
// Multiple Boundaries (All Functional!)
// ========================================
const Dashboard = () => {
  return (
    <div className="dashboard">
      {/* Each section has its own error boundary */}
      <ErrorBoundary
        FallbackComponent={SidebarError}
        onReset={() => console.log('Sidebar reset')}
      >
        <Sidebar />
      </ErrorBoundary>

      <ErrorBoundary
        FallbackComponent={MainContentError}
        onReset={() => console.log('Main content reset')}
      >
        <MainContent />
      </ErrorBoundary>

      <ErrorBoundary
        fallbackRender={({ error, resetErrorBoundary }) => (
          <div>
            <p>Widget failed: {error.message}</p>
            <button onClick={resetErrorBoundary}>Retry</button>
          </div>
        )}
      >
        <AnalyticsWidget />
      </ErrorBoundary>
    </div>
  );
};
```

---

## 3️⃣ **Custom Functional Error Handler Hook**

```jsx
// ========================================
// useErrorHandler.js (Custom Hook)
// ========================================
import { useState, useCallback } from 'react';

export const useErrorHandler = () => {
  const [error, setError] = useState(null);

  const handleError = useCallback((error) => {
    setError(error);
    
    // Log to monitoring service
    logToService(error);
  }, []);

  const resetError = useCallback(() => {
    setError(null);
  }, []);

  return { error, handleError, resetError };
};

// ========================================
// Usage in Functional Component
// ========================================
const DataFetchingComponent = ({ apiUrl }) => {
  const [data, setData] = useState(null);
  const { error, handleError, resetError } = useErrorHandler();

  const fetchData = async () => {
    try {
      resetError(); // Clear previous errors
      const response = await fetch(apiUrl);
      
      if (!response.ok) {
        throw new Error(`API Error: ${response.status}`);
      }
      
      const result = await response.json();
      setData(result);
    } catch (err) {
      handleError(err); // Trigger error handling
    }
  };

  useEffect(() => {
    fetchData();
  }, [apiUrl]);

  // Show error UI if error exists
  if (error) {
    return (
      <div className="error-container">
        <h3>Failed to load data</h3>
        <p>{error.message}</p>
        <button onClick={fetchData}>Retry</button>
      </div>
    );
  }

  return (
    <div>
      {data ? <DataDisplay data={data} /> : <Loading />}
    </div>
  );
};
```

---

## 4️⃣ **Advanced: Functional Error Boundary Wrapper Pattern**

```jsx
// ========================================
// withErrorBoundary.jsx (HOC Pattern)
// ========================================
import { ErrorBoundary } from 'react-error-boundary';

// Higher-Order Component that wraps any functional component
export const withErrorBoundary = (Component, errorBoundaryProps = {}) => {
  const WrappedComponent = (props) => {
    const defaultFallback = ({ error, resetErrorBoundary }) => (
      <div className="error-wrapper">
        <h3>Error in {Component.displayName || Component.name}</h3>
        <p>{error.message}</p>
        <button onClick={resetErrorBoundary}>Retry</button>
      </div>
    );

    return (
      <ErrorBoundary
        FallbackComponent={defaultFallback}
        onError={(error, errorInfo) => {
          console.error(`Error in ${Component.name}:`, error);
        }}
        {...errorBoundaryProps}
      >
        <Component {...props} />
      </ErrorBoundary>
    );
  };

  WrappedComponent.displayName = `withErrorBoundary(${Component.displayName || Component.name})`;
  
  return WrappedComponent;
};

// ========================================
// Usage (Clean Functional Component)
// ========================================
const MyComponent = ({ data }) => {
  const [count, setCount] = useState(0);

  if (!data) {
    throw new Error('Data is required!');
  }

  return (
    <div>
      <h1>{data.title}</h1>
      <button onClick={() => setCount(count + 1)}>{count}</button>
    </div>
  );
};

// Wrap with error boundary
export default withErrorBoundary(MyComponent, {
  FallbackComponent: CustomErrorFallback,
  onError: (error) => logToAzure(error)
});

// Or use it directly
const App = () => {
  const SafeComponent = withErrorBoundary(MyComponent);
  
  return <SafeComponent data={someData} />;
};
```

---

## 5️⃣ **Complete Production Example with Functional Components**

```jsx
// ========================================
// ErrorBoundary.jsx (Minimal Class)
// ========================================
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null, errorInfo: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    this.setState({ errorInfo });
    this.props.onError?.(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <this.props.fallback
          error={this.state.error}
          errorInfo={this.state.errorInfo}
          reset={() => this.setState({ hasError: false })}
        />
      );
    }
    return this.props.children;
  }
}

// ========================================
// ErrorFallback.jsx (Full Functional Logic)
// ========================================
const ErrorFallback = ({ error, errorInfo, reset }) => {
  const [retryCount, setRetryCount] = useState(0);
  const [showDetails, setShowDetails] = useState(false);
  const [errorId] = useState(() => `ERR-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`);

  // Log to monitoring services
  useEffect(() => {
    const logError = async () => {
      const errorData = {
        errorId,
        message: error?.message,
        stack: error?.stack,
        componentStack: errorInfo?.componentStack,
        timestamp: new Date().toISOString(),
        url: window.location.href,
        userAgent: navigator.userAgent,
        retryCount
      };

      // Log to Azure Application Insights
      if (window.appInsights) {
        window.appInsights.trackException({
          error,
          properties: errorData,
          severityLevel: 3
        });
      }

      // Log to .NET Core backend
      try {
        await fetch('/api/errors/client', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(errorData)
        });
      } catch (err) {
        console.error('Failed to log error:', err);
      }
    };

    logError();
  }, [error, errorInfo, errorId, retryCount]);

  // Handle retry
  const handleRetry = useCallback(() => {
    setRetryCount(prev => prev + 1);
    reset();
  }, [reset]);

  // Copy error ID to clipboard
  const copyErrorId = useCallback(async () => {
    try {
      await navigator.clipboard.writeText(errorId);
      alert('Error ID copied to clipboard');
    } catch (err) {
      console.error('Failed to copy:', err);
    }
  }, [errorId]);

  return (
    <div className="error-fallback-container">
      <div className="error-content">
        <div className="error-icon">⚠️</div>
        
        <h2>Something went wrong</h2>
        
        <p className="error-message">
          We're sorry for the inconvenience. Our team has been notified.
        </p>

        <div className="error-id-section">
          <code className="error-id">{errorId}</code>
          <button onClick={copyErrorId} className="copy-btn">
            📋 Copy
          </button>
        </div>

        {retryCount > 0 && (
          <p className="retry-info">
            Retry attempts: {retryCount}
          </p>
        )}

        <div className="action-buttons">
          <button onClick={handleRetry} className="btn-primary">
            Try Again
          </button>
          <button onClick={() => window.location.href = '/'} className="btn-secondary">
            Go Home
          </button>
          <button onClick={() => window.location.reload()} className="btn-secondary">
            Reload Page
          </button>
        </div>

        {process.env.NODE_ENV === 'development' && (
          <div className="dev-error-details">
            <button 
              onClick={() => setShowDetails(!showDetails)}
              className="toggle-details"
            >
              {showDetails ? 'Hide' : 'Show'} Error Details
            </button>
            
            {showDetails && (
              <div className="error-details">
                <div className="detail-section">
                  <h4>Error Message:</h4>
                  <pre>{error?.message}</pre>
                </div>
                
                <div className="detail-section">
                  <h4>Stack Trace:</h4>
                  <pre>{error?.stack}</pre>
                </div>
                
                <div className="detail-section">
                  <h4>Component Stack:</h4>
                  <pre>{errorInfo?.componentStack}</pre>
                </div>
              </div>
            )}
          </div>
        )}

        <p className="support-text">
          If this problem persists, please contact support with Error ID: {errorId}
        </p>
      </div>
    </div>
  );
};

// ========================================
// App.jsx (Fully Functional)
// ========================================
const App = () => {
  const handleGlobalError = useCallback((error, errorInfo) => {
    console.error('Global error handler:', error);
    
    // Additional monitoring
    if (window.Sentry) {
      window.Sentry.captureException(error);
    }
  }, []);

  return (
    <ErrorBoundary
      fallback={ErrorFallback}
      onError={handleGlobalError}
    >
      <Router>
        <Layout>
          <Routes>
            <Route path="/" element={<Home />} />
            <Route path="/dashboard" element={
              <ErrorBoundary fallback={DashboardError}>
                <Dashboard />
              </ErrorBoundary>
            } />
            <Route path="/profile" element={
              <ErrorBoundary fallback={ProfileError}>
                <Profile />
              </ErrorBoundary>
            } />
          </Routes>
        </Layout>
      </Router>
    </ErrorBoundary>
  );
};

// ========================================
// Dashboard.jsx (Functional with Local Error Handling)
// ========================================
const Dashboard = () => {
  const [data, setData] = useState(null);
  const [localError, setLocalError] = useState(null);

  // Use custom hook for error handling
  const { handleAsyncError } = useAsyncErrorHandler();

  const fetchDashboardData = useCallback(async () => {
    try {
      setLocalError(null);
      const response = await fetch('/api/dashboard');
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
      }
      
      const result = await response.json();
      setData(result);
    } catch (error) {
      // Handle locally or propagate to error boundary
      if (error.message.includes('404')) {
        setLocalError('Data not found');
      } else {
        handleAsyncError(error); // Throws to error boundary
      }
    }
  }, [handleAsyncError]);

  useEffect(() => {
    fetchDashboardData();
  }, [fetchDashboardData]);

  // Local error handling
  if (localError) {
    return (
      <div className="local-error">
        <p>{localError}</p>
        <button onClick={fetchDashboardData}>Retry</button>
      </div>
    );
  }

  return (
    <div className="dashboard">
      {data ? <DashboardContent data={data} /> : <LoadingSpinner />}
    </div>
  );
};

// ========================================
// Custom Hook for Async Error Handling
// ========================================
const useAsyncErrorHandler = () => {
  const [, setError] = useState();

  const handleAsyncError = useCallback((error) => {
    // This will be caught by error boundary
    setError(() => {
      throw error;
    });
  }, []);

  return { handleAsyncError };
};
```

---

## 📊 Comparison Summary

| Aspect | Class Error Boundary | Functional with Library | Custom Hook |
|--------|---------------------|------------------------|-------------|
| **Error Catching** | ✅ Full (render errors) | ✅ Full (via library) | ⚠️ Manual (async only) |
| **Syntax** | Class syntax | Functional syntax | Functional syntax |
| **Bundle Size** | Minimal | +Small library | No extra |
| **Flexibility** | ✅ Full control | ✅ Excellent API | ✅ Custom logic |
| **Learning Curve** | Higher | Lower | Medium |
| **Best For** | Custom solutions | Most apps | Async errors only |

---

## ✅ Best Practice Recommendation

**For modern React applications:**

1. **Use `react-error-boundary` library** for 90% of cases
2. **Create functional fallback components** with full logic
3. **Keep the class error boundary minimal** (just the wrapper)
4. **Use custom hooks** for async error handling
5. **Combine approaches** for enterprise applications

```jsx
// Perfect modern setup
import { ErrorBoundary } from 'react-error-boundary';

const App = () => {
  return (
    <ErrorBoundary FallbackComponent={FunctionalErrorFallback}>
      <YourFullyFunctionalApp />
    </ErrorBoundary>
  );
};
```

---

## 🎯 Key Takeaway

**You MUST use a class component for the error boundary itself**, but:
- ✅ Fallback UI can be 100% functional
- ✅ Error handlers can be 100% functional
- ✅ All app logic can be 100% functional
- ✅ Use libraries like `react-error-boundary` for cleaner API

The class component is just a **thin wrapper** - all your logic stays functional! 🚀
