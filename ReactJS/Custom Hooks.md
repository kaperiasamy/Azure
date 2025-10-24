# Custom Hooks in React: Complete Guide

## Table of Contents
1. [What are Custom Hooks?](#what-are-custom-hooks)
2. [When to Create Custom Hooks?](#when-to-create-custom-hooks)
3. [Rules for Custom Hooks](#rules-for-custom-hooks)
4. [How to Create Custom Hooks](#how-to-create-custom-hooks)
5. [Custom Hook Examples](#custom-hook-examples)
6. [Best Practices](#best-practices)
7. [Testing Custom Hooks](#testing-custom-hooks)

---

## What are Custom Hooks?

Custom hooks are **reusable functions that let you extract component logic into reusable functions**. They're just JavaScript functions that use React hooks internally.

### Key Points:
- Custom hooks must start with `use` (e.g., `useFetch`, `useAuth`)
- They can use other React hooks
- They can accept arguments and return values
- They allow logic sharing between components without render props or HOCs

---

## When to Create Custom Hooks?

### ✅ Create a custom hook when:

| Scenario | Example |
|----------|---------|
| **Logic is repeated** | Multiple components fetch user data the same way |
| **Complex state logic** | Managing multiple related state variables |
| **Side effects management** | Subscription, animations, event listeners |
| **Form handling** | Form validation, input management across components |
| **API calls** | Fetching data with loading/error handling |
| **Local storage sync** | Reading/writing to localStorage |
| **Authentication** | Auth state, permissions, login/logout |
| **Animations** | Reusable animation logic |
| **Geolocation tracking** | Location-based features |

### ❌ Don't create a custom hook when:

```jsx
// ❌ BAD - Too simple, not worth extracting
const useOne = () => {
  return 1;
};

// ❌ BAD - Just renaming built-in hooks
const useMyState = (initial) => {
  return useState(initial);
};

// ❌ BAD - Not reusable logic
const useSpecificFeature = () => {
  // Logic specific to one component
};
```

---

## Rules for Custom Hooks

### ✅ The Two Essential Rules:

```jsx
// Rule 1: Always call hooks at the top level
const useData = (url) => {
  const [data, setData] = useState(null); // ✅ GOOD
  
  useEffect(() => {
    fetch(url);
  }, [url]); // ✅ GOOD
  
  return data;
};

// ❌ BAD - Conditional hooks
const useBadHook = (shouldUse) => {
  if (shouldUse) {
    useState(0); // ❌ Don't do this!
  }
};

// ✅ GOOD - Condition inside the hook
const useConditional = (shouldUse) => {
  const [state, setState] = useState(0);
  
  if (shouldUse) {
    // Use state inside hook, not define it inside condition
    setState(1);
  }
  
  return state;
};
```

```jsx
// Rule 2: Only call hooks from React functions
// ✅ Call from function components
const MyComponent = () => {
  const data = useData();
  return <div>{data}</div>;
};

// ✅ Call from custom hooks
const useCustomLogic = () => {
  const data = useData();
  return data;
};

// ❌ Don't call from regular functions
function regularFunction() {
  const data = useData(); // ❌ Not allowed!
}
```

---

## How to Create Custom Hooks

### Step-by-Step Process:

```jsx
// Step 1: Identify repeatable logic
// Multiple components need to fetch user data

// Step 2: Extract the logic
const useFetch = (url) => {
  // Step 3: Use React hooks
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  // Step 4: Implement the logic
  useEffect(() => {
    const fetchData = async () => {
      try {
        setLoading(true);
        const response = await fetch(url);
        if (!response.ok) throw new Error('Failed to fetch');
        const json = await response.json();
        setData(json);
        setError(null);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };
    
    fetchData();
  }, [url]);
  
  // Step 5: Return state and functions
  return { data, loading, error };
};

// Step 6: Use in components
const UserList = () => {
  const { data: users, loading, error } = useFetch('/api/users');
  
  if (loading) return <Spinner />;
  if (error) return <Error message={error} />;
  
  return <ul>{users.map(u => <li key={u.id}>{u.name}</li>)}</ul>;
};
```

---

## Custom Hook Examples

### 1. ✅ useFetch - Data Fetching Hook

**Best Practice Implementation:**

```jsx
// hooks/useFetch.js
import { useState, useEffect, useCallback, useRef } from 'react';

/**
 * Custom hook for fetching data with loading, error, and refetch states
 * 
 * @param {string} url - The URL to fetch from
 * @param {Object} options - Fetch options
 * @param {Object} options.headers - Request headers
 * @param {string} options.method - HTTP method (default: GET)
 * @param {boolean} options.skip - Skip fetching initially
 * @returns {Object} - { data, loading, error, refetch }
 */
const useFetch = (
  url,
  {
    headers = {},
    method = 'GET',
    skip = false,
    onSuccess = null,
    onError = null,
  } = {}
) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(!skip);
  const [error, setError] = useState(null);
  
  // Use ref to prevent infinite loops
  const abortControllerRef = useRef(null);

  // Memoized fetch function
  const fetchData = useCallback(async () => {
    // Cancel previous request if exists
    if (abortControllerRef.current) {
      abortControllerRef.current.abort();
    }

    abortControllerRef.current = new AbortController();

    setLoading(true);
    setError(null);

    try {
      const response = await fetch(url, {
        method,
        headers: {
          'Content-Type': 'application/json',
          ...headers,
        },
        signal: abortControllerRef.current.signal,
      });

      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }

      const result = await response.json();
      setData(result);
      setError(null);

      // Call success callback
      onSuccess?.(result);
    } catch (err) {
      // Don't handle abort errors
      if (err.name !== 'AbortError') {
        setError(err.message);
        onError?.(err);
      }
    } finally {
      setLoading(false);
    }
  }, [url, method, headers, onSuccess, onError]);

  // Initial fetch
  useEffect(() => {
    if (!skip) {
      fetchData();
    }

    // Cleanup: cancel request on unmount
    return () => {
      if (abortControllerRef.current) {
        abortControllerRef.current.abort();
      }
    };
  }, [url, skip, fetchData]);

  return {
    data,
    loading,
    error,
    refetch: fetchData,
    reset: () => {
      setData(null);
      setError(null);
      setLoading(false);
    },
  };
};

export default useFetch;
```

**Usage:**

```jsx
// components/UserProfile.jsx
import useFetch from '../hooks/useFetch';

const UserProfile = ({ userId }) => {
  const {
    data: user,
    loading,
    error,
    refetch,
  } = useFetch(`/api/users/${userId}`, {
    onSuccess: (data) => console.log('User loaded:', data),
    onError: (error) => console.error('Failed to load user:', error),
  });

  if (loading) return <div>Loading user...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return <div>No user found</div>;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
      <button onClick={refetch}>Refresh</button>
    </div>
  );
};

export default UserProfile;
```

---

### 2. ✅ useForm - Form Handling Hook

**Best Practice Implementation:**

```jsx
// hooks/useForm.js
import { useState, useCallback } from 'react';

/**
 * Custom hook for managing form state and validation
 * 
 * @param {Object} initialValues - Initial form values
 * @param {Function} onSubmit - Callback on form submission
 * @param {Function} validate - Validation function
 * @returns {Object} - Form state and handlers
 */
const useForm = (initialValues, onSubmit, validate = null) => {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [submitError, setSubmitError] = useState(null);

  // Handle field value change
  const handleChange = useCallback((e) => {
    const { name, value, type, checked } = e.target;
    const newValue = type === 'checkbox' ? checked : value;

    setValues((prev) => ({
      ...prev,
      [name]: newValue,
    }));

    // Clear error when user starts typing
    if (errors[name]) {
      setErrors((prev) => ({
        ...prev,
        [name]: '',
      }));
    }
  }, [errors]);

  // Handle field blur
  const handleBlur = useCallback((e) => {
    const { name } = e.target;
    
    setTouched((prev) => ({
      ...prev,
      [name]: true,
    }));

    // Validate on blur
    if (validate) {
      const fieldError = validate(name, values[name]);
      setErrors((prev) => ({
        ...prev,
        [name]: fieldError || '',
      }));
    }
  }, [values, validate]);

  // Validate all fields
  const validateForm = useCallback(() => {
    if (!validate) return true;

    const newErrors = {};
    let isValid = true;

    Object.keys(values).forEach((field) => {
      const error = validate(field, values[field]);
      if (error) {
        newErrors[field] = error;
        isValid = false;
      }
    });

    setErrors(newErrors);
    return isValid;
  }, [values, validate]);

  // Handle form submission
  const handleSubmit = useCallback(
    async (e) => {
      e.preventDefault();
      setSubmitError(null);

      // Mark all fields as touched
      const touchedFields = Object.keys(values).reduce((acc, key) => {
        acc[key] = true;
        return acc;
      }, {});
      setTouched(touchedFields);

      // Validate
      if (!validateForm()) {
        return;
      }

      setIsSubmitting(true);

      try {
        await onSubmit(values);
      } catch (error) {
        setSubmitError(error.message);
      } finally {
        setIsSubmitting(false);
      }
    },
    [values, onSubmit, validateForm]
  );

  // Reset form
  const resetForm = useCallback(() => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
    setSubmitError(null);
  }, [initialValues]);

  // Set field value programmatically
  const setFieldValue = useCallback((name, value) => {
    setValues((prev) => ({
      ...prev,
      [name]: value,
    }));
  }, []);

  return {
    values,
    errors,
    touched,
    isSubmitting,
    submitError,
    handleChange,
    handleBlur,
    handleSubmit,
    resetForm,
    setFieldValue,
    setErrors,
  };
};

export default useForm;
```

**Usage:**

```jsx
// components/RegistrationForm.jsx
import useForm from '../hooks/useForm';

const RegistrationForm = () => {
  const validateForm = (fieldName, value) => {
    switch (fieldName) {
      case 'email':
        return !value ? 'Email is required' :
               !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value) ? 'Invalid email' :
               '';
      case 'password':
        return !value ? 'Password is required' :
               value.length < 8 ? 'Password must be at least 8 characters' :
               '';
      case 'name':
        return !value ? 'Name is required' : '';
      default:
        return '';
    }
  };

  const {
    values,
    errors,
    touched,
    isSubmitting,
    submitError,
    handleChange,
    handleBlur,
    handleSubmit,
    resetForm,
  } = useForm(
    { name: '', email: '', password: '' },
    async (values) => {
      await api.register(values);
      alert('Registration successful!');
      resetForm();
    },
    validateForm
  );

  return (
    <form onSubmit={handleSubmit}>
      {submitError && <div className="error">{submitError}</div>}

      <div>
        <input
          type="text"
          name="name"
          value={values.name}
          onChange={handleChange}
          onBlur={handleBlur}
          placeholder="Full Name"
        />
        {touched.name && errors.name && (
          <span className="error">{errors.name}</span>
        )}
      </div>

      <div>
        <input
          type="email"
          name="email"
          value={values.email}
          onChange={handleChange}
          onBlur={handleBlur}
          placeholder="Email"
        />
        {touched.email && errors.email && (
          <span className="error">{errors.email}</span>
        )}
      </div>

      <div>
        <input
          type="password"
          name="password"
          value={values.password}
          onChange={handleChange}
          onBlur={handleBlur}
          placeholder="Password"
        />
        {touched.password && errors.password && (
          <span className="error">{errors.password}</span>
        )}
      </div>

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Registering...' : 'Register'}
      </button>
      <button type="button" onClick={resetForm}>
        Reset
      </button>
    </form>
  );
};

export default RegistrationForm;
```

---

### 3. ✅ useLocalStorage - Persistent State Hook

**Best Practice Implementation:**

```jsx
// hooks/useLocalStorage.js
import { useState, useEffect, useCallback } from 'react';

/**
 * Custom hook for syncing state with localStorage
 * 
 * @param {string} key - localStorage key
 * @param {*} initialValue - Initial value if not in localStorage
 * @returns {Array} - [value, setValue]
 */
const useLocalStorage = (key, initialValue) => {
  // Initialize state with localStorage value or initialValue
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(`Error reading localStorage key "${key}":`, error);
      return initialValue;
    }
  });

  // Sync state with localStorage
  const setValue = useCallback(
    (value) => {
      try {
        // Allow value to be a function for same API as useState
        const valueToStore =
          value instanceof Function ? value(storedValue) : value;

        setStoredValue(valueToStore);
        window.localStorage.setItem(key, JSON.stringify(valueToStore));

        // Dispatch custom event so other tabs/windows can sync
        window.dispatchEvent(
          new Event('local-storage', { detail: { key, value: valueToStore } })
        );
      } catch (error) {
        console.error(`Error setting localStorage key "${key}":`, error);
      }
    },
    [key, storedValue]
  );

  // Sync across tabs/windows
  useEffect(() => {
    const handleStorageChange = (e) => {
      if (e.key === key && e.newValue) {
        try {
          setStoredValue(JSON.parse(e.newValue));
        } catch (error) {
          console.error(`Error syncing localStorage key "${key}":`, error);
        }
      }
    };

    window.addEventListener('storage', handleStorageChange);
    return () => window.removeEventListener('storage', handleStorageChange);
  }, [key]);

  // Remove from localStorage
  const removeValue = useCallback(() => {
    try {
      window.localStorage.removeItem(key);
      setStoredValue(initialValue);
    } catch (error) {
      console.error(`Error removing localStorage key "${key}":`, error);
    }
  }, [key, initialValue]);

  return [storedValue, setValue, removeValue];
};

export default useLocalStorage;
```

**Usage:**

```jsx
// components/ThemeToggle.jsx
import useLocalStorage from '../hooks/useLocalStorage';

const ThemeToggle = () => {
  const [theme, setTheme, resetTheme] = useLocalStorage('theme', 'light');

  const toggleTheme = () => {
    setTheme((prevTheme) => (prevTheme === 'light' ? 'dark' : 'light'));
  };

  return (
    <div>
      <p>Current theme: {theme}</p>
      <button onClick={toggleTheme}>Toggle Theme</button>
      <button onClick={resetTheme}>Reset to Default</button>
    </div>
  );
};

// Usage in multiple components
const UserPreferences = () => {
  const [sidebarCollapsed, setSidebarCollapsed] = useLocalStorage(
    'sidebarCollapsed',
    false
  );

  return (
    <button onClick={() => setSidebarCollapsed((prev) => !prev)}>
      {sidebarCollapsed ? 'Expand' : 'Collapse'} Sidebar
    </button>
  );
};
```

---

### 4. ✅ useAuth - Authentication Hook

**Best Practice Implementation:**

```jsx
// hooks/useAuth.js
import { useState, useCallback, useEffect, useContext } from 'react';
import { AuthContext } from '../contexts/AuthContext';

/**
 * Custom hook for authentication
 * 
 * @returns {Object} - Auth state and functions
 */
const useAuth = () => {
  const context = useContext(AuthContext);

  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }

  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);
  const { user, setUser, token, setToken } = context;

  // Login function
  const login = useCallback(async (email, password) => {
    setIsLoading(true);
    setError(null);

    try {
      const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password }),
      });

      if (!response.ok) {
        throw new Error('Login failed');
      }

      const data = await response.json();
      setUser(data.user);
      setToken(data.token);

      // Store token in localStorage
      localStorage.setItem('authToken', data.token);

      return data;
    } catch (err) {
      setError(err.message);
      throw err;
    } finally {
      setIsLoading(false);
    }
  }, [setUser, setToken]);

  // Register function
  const register = useCallback(async (userData) => {
    setIsLoading(true);
    setError(null);

    try {
      const response = await fetch('/api/auth/register', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(userData),
      });

      if (!response.ok) {
        throw new Error('Registration failed');
      }

      const data = await response.json();
      setUser(data.user);
      setToken(data.token);
      localStorage.setItem('authToken', data.token);

      return data;
    } catch (err) {
      setError(err.message);
      throw err;
    } finally {
      setIsLoading(false);
    }
  }, [setUser, setToken]);

  // Logout function
  const logout = useCallback(() => {
    setUser(null);
    setToken(null);
    localStorage.removeItem('authToken');
  }, [setUser, setToken]);

  // Check if user is authenticated
  const isAuthenticated = !!user && !!token;

  // Check if user has specific role
  const hasRole = useCallback(
    (role) => {
      if (!isAuthenticated) return false;
      return user?.roles?.includes(role) || false;
    },
    [isAuthenticated, user]
  );

  // Check if user has specific permission
  const hasPermission = useCallback(
    (permission) => {
      if (!isAuthenticated) return false;
      return user?.permissions?.includes(permission) || false;
    },
    [isAuthenticated, user]
  );

  // Load user from token on mount
  useEffect(() => {
    const loadUser = async () => {
      const storedToken = localStorage.getItem('authToken');
      if (storedToken && !user) {
        try {
          const response = await fetch('/api/auth/me', {
            headers: { Authorization: `Bearer ${storedToken}` },
          });

          if (response.ok) {
            const data = await response.json();
            setUser(data.user);
            setToken(storedToken);
          } else {
            localStorage.removeItem('authToken');
          }
        } catch (err) {
          localStorage.removeItem('authToken');
        }
      }
    };

    loadUser();
  }, [user, setUser, setToken]);

  return {
    user,
    token,
    isAuthenticated,
    isLoading,
    error,
    login,
    register,
    logout,
    hasRole,
    hasPermission,
  };
};

export default useAuth;
```

**Usage:**

```jsx
// components/LoginForm.jsx
import useAuth from '../hooks/useAuth';
import useForm from '../hooks/useForm';

const LoginForm = () => {
  const { login, isLoading, error } = useAuth();
  const { values, errors, touched, handleChange, handleBlur, handleSubmit } =
    useForm(
      { email: '', password: '' },
      async (values) => {
        await login(values.email, values.password);
        // Navigation will happen automatically
      },
      (fieldName, value) => {
        if (!value) return 'This field is required';
        if (fieldName === 'email' && !/^\S+@\S+$/.test(value)) {
          return 'Invalid email';
        }
        return '';
      }
    );

  return (
    <form onSubmit={handleSubmit}>
      {error && <div className="error">{error}</div>}

      <input
        type="email"
        name="email"
        value={values.email}
        onChange={handleChange}
        onBlur={handleBlur}
        placeholder="Email"
      />
      {touched.email && errors.email && (
        <span className="error">{errors.email}</span>
      )}

      <input
        type="password"
        name="password"
        value={values.password}
        onChange={handleChange}
        onBlur={handleBlur}
        placeholder="Password"
      />
      {touched.password && errors.password && (
        <span className="error">{errors.password}</span>
      )}

      <button type="submit" disabled={isLoading}>
        {isLoading ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
};

// Protected component using useAuth
const AdminPanel = () => {
  const { user, hasRole, logout } = useAuth();

  if (!hasRole('admin')) {
    return <div>Access Denied</div>;
  }

  return (
    <div>
      <h2>Admin Panel</h2>
      <p>Welcome {user?.name}</p>
      <button onClick={logout}>Logout</button>
    </div>
  );
};

export default LoginForm;
```

---

### 5. ✅ usePrevious - Track Previous Value Hook

**Best Practice Implementation:**

```jsx
// hooks/usePrevious.js
import { useEffect, useRef } from 'react';

/**
 * Custom hook to get the previous value of a prop or state
 * 
 * @param {*} value - The current value to track
 * @returns {*} - The previous value
 */
const usePrevious = (value) => {
  const ref = useRef();

  useEffect(() => {
    ref.current = value;
  }, [value]);

  return ref.current;
};

export default usePrevious;
```

**Usage:**

```jsx
// components/Counter.jsx
import usePrevious from '../hooks/usePrevious';

const Counter = () => {
  const [count, setCount] = useState(0);
  const previousCount = usePrevious(count);

  return (
    <div>
      <p>Current: {count}</p>
      <p>Previous: {previousCount}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
};
```

---

### 6. ✅ useDebounce - Debounce Hook

**Best Practice Implementation:**

```jsx
// hooks/useDebounce.js
import { useState, useEffect } from 'react';

/**
 * Custom hook for debouncing values
 * 
 * @param {*} value - The value to debounce
 * @param {number} delay - Delay in milliseconds
 * @returns {*} - The debounced value
 */
const useDebounce = (value, delay = 500) => {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    // Set up the timeout
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    // Clean up the timeout if value changes before delay expires
    return () => clearTimeout(handler);
  }, [value, delay]);

  return debouncedValue;
};

export default useDebounce;
```

**Usage:**

```jsx
// components/SearchUsers.jsx
import useDebounce from '../hooks/useDebounce';
import useFetch from '../hooks/useFetch';

const SearchUsers = () => {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearchTerm = useDebounce(searchTerm, 300);

  // Only fetch when debounced value changes
  const { data: results, loading } = useFetch(
    `/api/users/search?q=${debouncedSearchTerm}`,
    { skip: !debouncedSearchTerm }
  );

  return (
    <div>
      <input
        type="text"
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        placeholder="Search users..."
      />

      {loading && <p>Searching...</p>}

      <ul>
        {results?.map((user) => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
};

export default SearchUsers;
```

---

### 7. ✅ useAsync - Async Operation Hook

**Best Practice Implementation:**

```jsx
// hooks/useAsync.js
import { useState, useEffect, useCallback, useRef } from 'react';

/**
 * Custom hook for handling async operations
 * 
 * @param {Function} asyncFunction - Async function to execute
 * @param {boolean} immediate - Execute immediately on mount
 * @param {Array} dependencies - Dependencies for the effect
 * @returns {Object} - { status, value, error, execute }
 */
const useAsync = (
  asyncFunction,
  immediate = true,
  dependencies = []
) => {
  const [status, setStatus] = useState('idle');
  const [value, setValue] = useState(null);
  const [error, setError] = useState(null);

  // Use ref to prevent infinite loops
  const executeFunction = useCallback(async (...args) => {
    setStatus('pending');
    setValue(null);
    setError(null);

    try {
      const response = await asyncFunction(...args);
      setValue(response);
      setStatus('success');
      return response;
    } catch (err) {
      setError(err);
      setStatus('error');
      throw err;
    }
  }, [asyncFunction]);

  // Execute on mount if immediate
  useEffect(() => {
    if (immediate) {
      executeFunction();
    }
  }, [executeFunction, immediate, ...dependencies]);

  return { execute: executeFunction, status, value, error };
};

export default useAsync;
```

**Usage:**

```jsx
// components/DataDisplay.jsx
import useAsync from '../hooks/useAsync';

const fetchUserData = async () => {
  const response = await fetch('/api/user');
  if (!response.ok) throw new Error('Failed to fetch');
  return response.json();
};

const DataDisplay = () => {
  const { status, value, error, execute } = useAsync(
    fetchUserData,
    true // Execute on mount
  );

  if (status === 'pending') return <div>Loading...</div>;
  if (status === 'error') return <div>Error: {error?.message}</div>;
  if (status === 'success') {
    return (
      <div>
        <h2>{value.name}</h2>
        <p>{value.email}</p>
        <button onClick={execute}>Refresh</button>
      </div>
    );
  }

  return null;
};

export default DataDisplay;
```

---

## Best Practices

### ✅ 1. Follow Naming Convention

```jsx
// ✅ GOOD - Always start with 'use'
const useCustomHook = () => {};
const useFetchData = () => {};
const useValidation = () => {};

// ❌ BAD - Missing 'use' prefix
const customHook = () => {};
const fetchData = () => {};
```

### ✅ 2. Add JSDoc Comments

```jsx
/**
 * Fetches data from an API endpoint
 * 
 * @param {string} url - The endpoint URL
 * @param {Object} options - Configuration options
 * @param {Object} options.headers - HTTP headers
 * @param {boolean} options.skip - Skip fetching
 * 
 * @returns {Object} Returns an object with:
 * @returns {*} data - The fetched data
 * @returns {boolean} loading - Loading state
 * @returns {Error|null} error - Error object if request failed
 * @returns {Function} refetch - Function to refetch data
 * 
 * @example
 * const { data, loading, error } = useFetch('/api/users');
 */
const useFetch = (url, options = {}) => {
  // Implementation
};
```

### ✅ 3. Handle Dependencies Correctly

```jsx
// ✅ GOOD - Include all dependencies
useEffect(() => {
  fetchUser(userId);
}, [userId]); // userId is a dependency

// ✅ GOOD - Use useCallback for stable functions
const handleClick = useCallback(() => {
  console.log(userId);
}, [userId]);

// ❌ BAD - Missing dependencies
useEffect(() => {
  fetchUser(userId);
}, []); // userId should be included!
```

### ✅ 4. Cleanup Resources

```jsx
const useEventListener = (eventName, handler) => {
  useEffect(() => {
    // Subscribe
    window.addEventListener(eventName, handler);

    // Cleanup - this is important!
    return () => {
      window.removeEventListener(eventName, handler);
    };
  }, [eventName, handler]);
};

const useTimer = (duration) => {
  useEffect(() => {
    const timer = setInterval(() => {
      // Timer logic
    }, 1000);

    // Cleanup
    return () => clearInterval(timer);
  }, [duration]);
};
```

### ✅ 5. Return Stable References

```jsx
// ❌ BAD - Creates new object each render
const useUserData = () => {
  return {
    user: { name: 'John' },
    loading: false,
  };
};

// ✅ GOOD - Use useMemo to create stable reference
const useUserData = () => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);

  const result = useMemo(
    () => ({
      user,
      loading,
    }),
    [user, loading]
  );

  return result;
};
```

### ✅ 6. Provide Flexible APIs

```jsx
// ✅ GOOD - Flexible hook that works with multiple patterns
const useForm = (initialValues, onSubmit, validate) => {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});

  // Allows different reset strategies
  const resetForm = useCallback((newValues = initialValues) => {
    setValues(newValues);
    setErrors({});
  }, [initialValues]);

  // Allows programmatic field updates
  const setFieldValue = useCallback((name, value) => {
    setValues((prev) => ({ ...prev, [name]: value }));
  }, []);

  return {
    values,
    errors,
    resetForm,
    setFieldValue,
    // ... other returns
  };
};
```

### ✅ 7. Consider Performance

```jsx
// ✅ GOOD - Optimize re-renders
const useSearch = (query) => {
  const [results, setResults] = useState([]);
  const [loading, setLoading] = useState(false);

  // Only search when debounced value changes
  const debouncedQuery = useDebounce(query, 300);

  useEffect(() => {
    if (!debouncedQuery) {
      setResults([]);
      return;
    }

    performSearch();
  }, [debouncedQuery]);

  return { results, loading };
};
```

### ✅ 8. Error Handling

```jsx
// ✅ GOOD - Comprehensive error handling
const useFetch = (url) => {
  const [error, setError] = useState(null);
  const [data, setData] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await fetch(url);

        if (!response.ok) {
          throw new Error(
            `HTTP Error ${response.status}: ${response.statusText}`
          );
        }

        const result = await response.json();
        setData(result);
        setError(null);
      } catch (err) {
        if (err instanceof TypeError) {
          setError('Network error. Please check your connection.');
        } else {
          setError(err.message);
        }
      }
    };

    fetchData();
  }, [url]);

  return { data, error };
};
```

### ✅ 9. Avoid Infinite Loops

```jsx
// ❌ BAD - Creates infinite loop
const useBadHook = () => {
  const [state, setState] = useState(0);

  useEffect(() => {
    setState(state + 1); // Dependencies missing!
  }); // This runs every render!
};

// ✅ GOOD - Proper dependency array
const useGoodHook = () => {
  const [state, setState] = useState(0);

  useEffect(() => {
    // This runs only once on mount
  }, []);

  return state;
};
```

### ✅ 10. Type Safety (TypeScript)

```typescript
// hooks/useFetch.ts
interface UseFetchOptions {
  headers?: Record<string, string>;
  method?: 'GET' | 'POST' | 'PUT' | 'DELETE';
  skip?: boolean;
}

interface UseFetchReturn<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
  refetch: () => Promise<void>;
}

const useFetch = <T = unknown>(
  url: string,
  options?: UseFetchOptions
): UseFetchReturn<T> => {
  // Implementation with proper types
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);

  // ... rest of implementation

  return { data, loading, error, refetch: async () => {} };
};

export default useFetch;
```

---

## Testing Custom Hooks

### ✅ Testing with React Testing Library

```jsx
// hooks/useCounter.js
import { useState, useCallback } from 'react';

const useCounter = (initialValue = 0) => {
  const [count, setCount] = useState(initialValue);

  const increment = useCallback(() => {
    setCount((c) => c + 1);
  }, []);

  const decrement = useCallback(() => {
    setCount((c) => c - 1);
  }, []);

  const reset = useCallback(() => {
    setCount(initialValue);
  }, [initialValue]);

  return { count, increment, decrement, reset };
};

export default useCounter;
```

```jsx
// hooks/__tests__/useCounter.test.js
import { renderHook, act } from '@testing-library/react';
import useCounter from '../useCounter';

describe('useCounter', () => {
  test('initializes with default value', () => {
    const { result } = renderHook(() => useCounter());
    expect(result.current.count).toBe(0);
  });

  test('initializes with custom value', () => {
    const { result } = renderHook(() => useCounter(10));
    expect(result.current.count).toBe(10);
  });

  test('increments counter', () => {
    const { result } = renderHook(() => useCounter());

    act(() => {
      result.current.increment();
    });

    expect(result.current.count).toBe(1);
  });

  test('decrements counter', () => {
    const { result } = renderHook(() => useCounter(5));

    act(() => {
      result.current.decrement();
    });

    expect(result.current.count).toBe(4);
  });

  test('resets counter', () => {
    const { result } = renderHook(() => useCounter(10));

    act(() => {
      result.current.increment();
      result.current.increment();
    });

    expect(result.current.count).toBe(12);

    act(() => {
      result.current.reset();
    });

    expect(result.current.count).toBe(10);
  });

  test('multiple increments', () => {
    const { result } = renderHook(() => useCounter());

    act(() => {
      result.current.increment();
      result.current.increment();
      result.current.increment();
    });

    expect(result.current.count).toBe(3);
  });
});
```

### ✅ Testing Async Hooks

```jsx
// hooks/__tests__/useFetch.test.js
import { renderHook, waitFor } from '@testing-library/react';
import useFetch from '../useFetch';

// Mock fetch
global.fetch = jest.fn();

describe('useFetch', () => {
  beforeEach(() => {
    fetch.mockClear();
  });

  test('fetches data successfully', async () => {
    const mockData = { id: 1, name: 'John' };
    fetch.mockResolvedValueOnce({
      ok: true,
      json: async () => mockData,
    });

    const { result } = renderHook(() => useFetch('/api/user'));

    expect(result.current.loading).toBe(true);

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });

    expect(result.current.data).toEqual(mockData);
    expect(result.current.error).toBe(null);
  });

  test('handles fetch error', async () => {
    const error = new Error('Network error');
    fetch.mockRejectedValueOnce(error);

    const { result } = renderHook(() => useFetch('/api/user'));

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });

    expect(result.current.error).toBe(error.message);
    expect(result.current.data).toBe(null);
  });

  test('refetches data', async () => {
    const mockData = { id: 1, name: 'John' };
    fetch.mockResolvedValue({
      ok: true,
      json: async () => mockData,
    });

    const { result } = renderHook(() => useFetch('/api/user'));

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });

    const firstCallCount = fetch.mock.calls.length;

    act(() => {
      result.current.refetch();
    });

    await waitFor(() => {
      expect(fetch.mock.calls.length).toBe(firstCallCount + 1);
    });
  });
});
```

---

## Quick Reference

### When to Create a Custom Hook:

| Situation | Create Hook? | Reason |
|-----------|--------------|--------|
| Reusing same logic in 2+ components | ✅ YES | DRY principle |
| Complex state with multiple sub-values | ✅ YES | Easier management |
| Managing side effects repeatedly | ✅ YES | Reusability |
| Simple operation (toggle, increment) | ❌ NO | Unnecessary abstraction |
| Hook-specific to one component | ❌ NO | Use state directly |

### Custom Hook Structure:

```
1. Start with 'use' prefix
2. Use React hooks inside
3. Return state and functions
4. Include JSDoc comments
5. Handle cleanup/errors
6. Optimize with useCallback/useMemo
7. Add TypeScript types
8. Write unit tests
```

---

## Summary

Custom hooks are powerful tools for:
- **Code reuse** across components
- **Logic extraction** into manageable pieces
- **Separation of concerns** (logic vs UI)
- **Testability** of business logic

Key takeaways:
✅ Always start with `use`
✅ Use React hooks inside
✅ Include comprehensive documentation
✅ Handle errors and cleanup
✅ Test thoroughly
✅ Keep them simple and focused
✅ Use TypeScript for better type safety

