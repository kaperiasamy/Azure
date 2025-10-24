# Protected Routes in React - Interview Guide

## What are Protected Routes?

Protected routes (also called private routes) are routes in a React application that can only be accessed by authenticated or authorized users. If a user tries to access a protected route without proper credentials, they're redirected to a login page or shown an error.

## Why Use Protected Routes?

- **Security**: Prevent unauthorized access to sensitive pages
- **User Experience**: Guide users to login before accessing restricted content
- **Access Control**: Implement role-based permissions (admin, user, guest)

---

## Implementation Methods

### 1. **Basic Protected Route Component (React Router v6)**

```jsx
import { Navigate } from 'react-router-dom';

const ProtectedRoute = ({ children }) => {
  const isAuthenticated = localStorage.getItem('token'); // or use auth context
  
  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }
  
  return children;
};

// Usage
<Route 
  path="/dashboard" 
  element={
    <ProtectedRoute>
      <Dashboard />
    </ProtectedRoute>
  } 
/>
```

### 2. **Using Context API (Recommended)**

```jsx
// AuthContext.js
import { createContext, useContext, useState } from 'react';

const AuthContext = createContext(null);

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  
  const login = (userData) => {
    setUser(userData);
    localStorage.setItem('token', userData.token);
  };
  
  const logout = () => {
    setUser(null);
    localStorage.removeItem('token');
  };
  
  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => useContext(AuthContext);
```

```jsx
// ProtectedRoute.js
import { Navigate, useLocation } from 'react-router-dom';
import { useAuth } from './AuthContext';

const ProtectedRoute = ({ children }) => {
  const { user } = useAuth();
  const location = useLocation();
  
  if (!user) {
    // Redirect to login, but save the location they were trying to access
    return <Navigate to="/login" state={{ from: location }} replace />;
  }
  
  return children;
};
```

### 3. **Role-Based Protected Routes**

```jsx
const ProtectedRoute = ({ children, allowedRoles }) => {
  const { user } = useAuth();
  const location = useLocation();
  
  if (!user) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }
  
  if (allowedRoles && !allowedRoles.includes(user.role)) {
    return <Navigate to="/unauthorized" replace />;
  }
  
  return children;
};

// Usage
<Route 
  path="/admin" 
  element={
    <ProtectedRoute allowedRoles={['admin']}>
      <AdminPanel />
    </ProtectedRoute>
  } 
/>
```

### 4. **Complete App Structure**

```jsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import { AuthProvider } from './AuthContext';

function App() {
  return (
    <AuthProvider>
      <BrowserRouter>
        <Routes>
          {/* Public routes */}
          <Route path="/login" element={<Login />} />
          <Route path="/register" element={<Register />} />
          <Route path="/" element={<Home />} />
          
          {/* Protected routes */}
          <Route 
            path="/dashboard" 
            element={
              <ProtectedRoute>
                <Dashboard />
              </ProtectedRoute>
            } 
          />
          
          <Route 
            path="/profile" 
            element={
              <ProtectedRoute>
                <Profile />
              </ProtectedRoute>
            } 
          />
          
          {/* Admin only route */}
          <Route 
            path="/admin" 
            element={
              <ProtectedRoute allowedRoles={['admin']}>
                <AdminPanel />
              </ProtectedRoute>
            } 
          />
          
          <Route path="/unauthorized" element={<Unauthorized />} />
        </Routes>
      </BrowserRouter>
    </AuthProvider>
  );
}
```

---

## Common Interview Questions & Answers

### Q1: **How do you implement protected routes?**
**Answer**: Create a wrapper component that checks authentication status (usually from Context or Redux) and either renders the requested component or redirects to login using `<Navigate>` from React Router.

### Q2: **Where do you store authentication tokens?**
**Answer**: 
- **localStorage**: Persists across sessions, vulnerable to XSS
- **sessionStorage**: Cleared when tab closes
- **httpOnly cookies**: Most secure, immune to XSS but requires backend setup
- **Memory (state)**: Most secure but lost on refresh

### Q3: **How do you handle route redirect after login?**
```jsx
// Save intended destination
<Navigate to="/login" state={{ from: location }} replace />

// Login component
const login = async () => {
  await authenticate();
  const from = location.state?.from?.pathname || '/dashboard';
  navigate(from, { replace: true });
};
```

### Q4: **What's the difference between authentication and authorization?**
- **Authentication**: Verifying who the user is (login)
- **Authorization**: Verifying what the user can access (permissions/roles)

### Q5: **How do you handle token expiration?**
```jsx
import axios from 'axios';

axios.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      // Token expired
      localStorage.removeItem('token');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);
```

---

## Best Practices

1. ✅ **Use Context API or Redux** for global auth state
2. ✅ **Validate tokens server-side**, never trust client-side validation
3. ✅ **Handle loading states** while checking authentication
4. ✅ **Implement token refresh** for better UX
5. ✅ **Use HTTPS** in production
6. ✅ **Implement proper error handling**
7. ✅ **Clear sensitive data on logout**

## Advanced: Loading State Handling

```jsx
const ProtectedRoute = ({ children }) => {
  const { user, loading } = useAuth();
  
  if (loading) {
    return <LoadingSpinner />;
  }
  
  if (!user) {
    return <Navigate to="/login" replace />;
  }
  
  return children;
};
```

This comprehensive guide covers everything you need for interviews! 🚀