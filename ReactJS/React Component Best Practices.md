# Complete Guide: React Component Best Practices (Modern Approach)

## Table of Contents
1. [Component Structure & Organization](#1-component-structure--organization)
2. [Naming Conventions](#2-naming-conventions)
3. [Component Design Patterns](#3-component-design-patterns)
4. [Props Management](#4-props-management)
5. [State Management](#5-state-management)
6. [Hooks Best Practices](#6-hooks-best-practices)
7. [Performance Optimization](#7-performance-optimization)
8. [Error Handling](#8-error-handling)
9. [Accessibility (a11y)](#9-accessibility-a11y)
10. [Testing](#10-testing)
11. [Code Quality & Maintenance](#11-code-quality--maintenance)

---

## 1. Component Structure & Organization

### ✅ File Structure (Modern Approach)

```
src/
├── components/
│   ├── Button/
│   │   ├── Button.jsx
│   │   ├── Button.test.jsx
│   │   ├── Button.module.css (or .scss)
│   │   ├── Button.stories.jsx (Storybook)
│   │   └── index.js (barrel export)
│   │
│   ├── UserCard/
│   │   ├── UserCard.jsx
│   │   ├── UserCard.test.jsx
│   │   ├── UserCard.module.css
│   │   ├── hooks/
│   │   │   └── useUserData.js
│   │   ├── utils/
│   │   │   └── formatUserName.js
│   │   └── index.js
│   │
│   └── common/ (or shared/)
│       ├── Input/
│       ├── Modal/
│       └── Loader/
│
├── features/ (feature-based structure)
│   ├── auth/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── api/
│   │   └── utils/
│   │
│   └── dashboard/
│
├── hooks/
├── utils/
├── contexts/
├── services/
└── constants/
```

### ✅ Component Template Structure

```jsx
// UserProfile.jsx
import { useState, useEffect, useCallback, useMemo } from 'react';
import PropTypes from 'prop-types';
import styles from './UserProfile.module.css';

/**
 * UserProfile Component
 * 
 * Displays user profile information with edit capabilities
 * 
 * @param {Object} props - Component props
 * @param {string} props.userId - Unique user identifier
 * @param {Function} props.onUpdate - Callback when user updates profile
 * @param {string} props.className - Additional CSS classes
 */
const UserProfile = ({ 
  userId, 
  onUpdate, 
  className = '',
  ...restProps 
}) => {
  // 1. Hooks (in order: state, context, custom hooks)
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  // 2. Computed values / Memoized values
  const fullName = useMemo(() => {
    if (!user) return '';
    return `${user.firstName} ${user.lastName}`;
  }, [user]);

  // 3. Effects
  useEffect(() => {
    fetchUserData();
  }, [userId]);

  // 4. Event handlers and functions
  const fetchUserData = useCallback(async () => {
    setLoading(true);
    try {
      const data = await getUserById(userId);
      setUser(data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }, [userId]);

  const handleUpdate = useCallback((updatedData) => {
    setUser(updatedData);
    onUpdate?.(updatedData);
  }, [onUpdate]);

  // 5. Early returns (loading, error, empty states)
  if (loading) return <LoadingSpinner />;
  if (error) return <ErrorMessage message={error} />;
  if (!user) return null;

  // 6. Main render
  return (
    <div className={`${styles.container} ${className}`} {...restProps}>
      <h2>{fullName}</h2>
      <p>{user.email}</p>
      <button onClick={() => handleUpdate(user)}>
        Update Profile
      </button>
    </div>
  );
};

// 7. PropTypes (for JavaScript) or TypeScript types
UserProfile.propTypes = {
  userId: PropTypes.string.isRequired,
  onUpdate: PropTypes.func,
  className: PropTypes.string,
};

// 8. Default props (if needed)
UserProfile.defaultProps = {
  className: '',
};

export default UserProfile;
```

---

## 2. Naming Conventions

### ✅ Component Naming

```jsx
// ✅ GOOD - PascalCase for components
const UserProfile = () => { };
const NavigationBar = () => { };
const ProductCard = () => { };

// ❌ BAD
const userProfile = () => { };
const navigation_bar = () => { };
```

### ✅ File Naming

```
// ✅ GOOD
UserProfile.jsx
NavigationBar.tsx
ProductCard.jsx

// ❌ BAD
userProfile.jsx
user-profile.jsx
UserProfile.js (use .jsx for clarity)
```

### ✅ Props & Variables

```jsx
// ✅ GOOD - camelCase
const [isLoading, setIsLoading] = useState(false);
const [userData, setUserData] = useState(null);
const handleButtonClick = () => {};

// Boolean props with is/has/should prefix
<Button isDisabled hasIcon shouldValidate />

// Event handlers with handle prefix
const handleSubmit = () => {};
const handleInputChange = () => {};

// ❌ BAD
const [IsLoading, SetIsLoading] = useState(false);
const ClickButton = () => {};
```

### ✅ Custom Hooks

```jsx
// ✅ GOOD - Always start with 'use'
const useAuth = () => {};
const useLocalStorage = () => {};
const useFetchUsers = () => {};

// ❌ BAD
const getAuth = () => {};
const fetchUsers = () => {};
```

### ✅ Constants

```jsx
// ✅ GOOD - UPPER_SNAKE_CASE
const API_BASE_URL = 'https://api.example.com';
const MAX_RETRY_ATTEMPTS = 3;
const USER_ROLES = {
  ADMIN: 'admin',
  USER: 'user',
  GUEST: 'guest',
};
```

---

## 3. Component Design Patterns

### ✅ Functional Components (Modern Standard)

```jsx
// ✅ GOOD - Arrow function (preferred)
const Button = ({ label, onClick }) => {
  return <button onClick={onClick}>{label}</button>;
};

// ✅ ALSO GOOD - Function declaration
function Button({ label, onClick }) {
  return <button onClick={onClick}>{label}</button>;
}

// ❌ BAD - Class components (legacy, avoid for new code)
class Button extends React.Component {
  render() {
    return <button>{this.props.label}</button>;
  }
}
```

### ✅ Component Composition

```jsx
// ✅ GOOD - Composable components
const Card = ({ children, className }) => (
  <div className={`card ${className}`}>{children}</div>
);

const CardHeader = ({ children }) => (
  <div className="card-header">{children}</div>
);

const CardBody = ({ children }) => (
  <div className="card-body">{children}</div>
);

const CardFooter = ({ children }) => (
  <div className="card-footer">{children}</div>
);

// Usage
<Card>
  <CardHeader>
    <h2>Title</h2>
  </CardHeader>
  <CardBody>
    <p>Content</p>
  </CardBody>
  <CardFooter>
    <Button>Action</Button>
  </CardFooter>
</Card>

// Export as compound component
Card.Header = CardHeader;
Card.Body = CardBody;
Card.Footer = CardFooter;

export default Card;
```

### ✅ Container/Presentational Pattern

```jsx
// Presentational Component (UI only)
const UserListView = ({ users, onUserClick, isLoading }) => {
  if (isLoading) return <Spinner />;
  
  return (
    <ul>
      {users.map(user => (
        <li key={user.id} onClick={() => onUserClick(user)}>
          {user.name}
        </li>
      ))}
    </ul>
  );
};

// Container Component (logic)
const UserListContainer = () => {
  const [users, setUsers] = useState([]);
  const [isLoading, setIsLoading] = useState(false);
  
  useEffect(() => {
    fetchUsers();
  }, []);
  
  const fetchUsers = async () => {
    setIsLoading(true);
    const data = await api.getUsers();
    setUsers(data);
    setIsLoading(false);
  };
  
  const handleUserClick = (user) => {
    console.log('User clicked:', user);
  };
  
  return (
    <UserListView 
      users={users} 
      onUserClick={handleUserClick}
      isLoading={isLoading}
    />
  );
};
```

### ✅ Render Props Pattern

```jsx
const DataFetcher = ({ url, render }) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then(data => {
        setData(data);
        setLoading(false);
      });
  }, [url]);
  
  return render({ data, loading });
};

// Usage
<DataFetcher 
  url="/api/users"
  render={({ data, loading }) => (
    loading ? <Spinner /> : <UserList users={data} />
  )}
/>
```

### ✅ Higher-Order Components (HOC)

```jsx
// ✅ GOOD - But hooks are preferred now
const withAuth = (Component) => {
  return (props) => {
    const { user, isAuthenticated } = useAuth();
    
    if (!isAuthenticated) {
      return <Navigate to="/login" />;
    }
    
    return <Component {...props} user={user} />;
  };
};

// Usage
const Dashboard = ({ user }) => <div>Welcome {user.name}</div>;
export default withAuth(Dashboard);

// ✅ BETTER - Custom Hook (Modern approach)
const useRequireAuth = () => {
  const { user, isAuthenticated } = useAuth();
  const navigate = useNavigate();
  
  useEffect(() => {
    if (!isAuthenticated) {
      navigate('/login');
    }
  }, [isAuthenticated, navigate]);
  
  return user;
};

// Usage
const Dashboard = () => {
  const user = useRequireAuth();
  return <div>Welcome {user.name}</div>;
};
```

---

## 4. Props Management

### ✅ Props Destructuring

```jsx
// ✅ GOOD - Destructure in parameters
const Button = ({ label, onClick, variant = 'primary', ...restProps }) => {
  return (
    <button 
      className={`btn btn-${variant}`}
      onClick={onClick}
      {...restProps}
    >
      {label}
    </button>
  );
};

// ❌ BAD
const Button = (props) => {
  return <button onClick={props.onClick}>{props.label}</button>;
};
```

### ✅ PropTypes (JavaScript)

```jsx
import PropTypes from 'prop-types';

const UserCard = ({ user, onEdit, isEditable }) => {
  // Component logic
};

UserCard.propTypes = {
  user: PropTypes.shape({
    id: PropTypes.number.isRequired,
    name: PropTypes.string.isRequired,
    email: PropTypes.string.isRequired,
    age: PropTypes.number,
  }).isRequired,
  onEdit: PropTypes.func,
  isEditable: PropTypes.bool,
};

UserCard.defaultProps = {
  isEditable: false,
  onEdit: () => {},
};
```

### ✅ TypeScript Types (Recommended)

```tsx
// UserCard.tsx
interface User {
  id: number;
  name: string;
  email: string;
  age?: number;
}

interface UserCardProps {
  user: User;
  onEdit?: (user: User) => void;
  isEditable?: boolean;
  className?: string;
}

const UserCard: React.FC<UserCardProps> = ({ 
  user, 
  onEdit, 
  isEditable = false,
  className = '' 
}) => {
  // Component logic
};

export default UserCard;
```

### ✅ Props Spreading

```jsx
// ✅ GOOD - Spread remaining props
const Input = ({ label, error, ...inputProps }) => {
  return (
    <div>
      <label>{label}</label>
      <input {...inputProps} />
      {error && <span className="error">{error}</span>}
    </div>
  );
};

// Usage - all HTML input attributes work
<Input 
  label="Email"
  type="email"
  placeholder="Enter email"
  required
  maxLength={50}
/>
```

### ✅ Children Prop

```jsx
// ✅ GOOD - Use children for composition
const Modal = ({ isOpen, onClose, children }) => {
  if (!isOpen) return null;
  
  return (
    <div className="modal-overlay">
      <div className="modal">
        <button onClick={onClose}>×</button>
        {children}
      </div>
    </div>
  );
};

// Usage
<Modal isOpen={isOpen} onClose={handleClose}>
  <h2>Title</h2>
  <p>Content goes here</p>
  <Button>Submit</Button>
</Modal>
```

### ✅ Prop Drilling Solution

```jsx
// ❌ BAD - Prop drilling
<GrandParent data={data}>
  <Parent data={data}>
    <Child data={data} />
  </Parent>
</GrandParent>

// ✅ GOOD - Context API
const DataContext = createContext();

const GrandParent = ({ data }) => {
  return (
    <DataContext.Provider value={data}>
      <Parent />
    </DataContext.Provider>
  );
};

const Child = () => {
  const data = useContext(DataContext);
  return <div>{data}</div>;
};
```

---

## 5. State Management

### ✅ Local State with useState

```jsx
// ✅ GOOD - Simple state
const [count, setCount] = useState(0);
const [user, setUser] = useState(null);
const [isOpen, setIsOpen] = useState(false);

// ✅ GOOD - Functional updates
const increment = () => {
  setCount(prevCount => prevCount + 1);
};

// ✅ GOOD - Lazy initialization (expensive computation)
const [data, setData] = useState(() => {
  return JSON.parse(localStorage.getItem('data')) || [];
});

// ❌ BAD - Direct mutation
setUser(user.name = 'John'); // Don't mutate

// ✅ GOOD - Create new object
setUser({ ...user, name: 'John' });
```

### ✅ Complex State with useReducer

```jsx
// ✅ GOOD - For complex state logic
const initialState = {
  user: null,
  isLoading: false,
  error: null,
};

const reducer = (state, action) => {
  switch (action.type) {
    case 'FETCH_START':
      return { ...state, isLoading: true, error: null };
    case 'FETCH_SUCCESS':
      return { ...state, isLoading: false, user: action.payload };
    case 'FETCH_ERROR':
      return { ...state, isLoading: false, error: action.payload };
    default:
      return state;
  }
};

const UserProfile = () => {
  const [state, dispatch] = useReducer(reducer, initialState);
  
  const fetchUser = async () => {
    dispatch({ type: 'FETCH_START' });
    try {
      const data = await api.getUser();
      dispatch({ type: 'FETCH_SUCCESS', payload: data });
    } catch (error) {
      dispatch({ type: 'FETCH_ERROR', payload: error.message });
    }
  };
  
  return (
    // JSX
  );
};
```

### ✅ State Organization

```jsx
// ❌ BAD - Too many separate states
const [firstName, setFirstName] = useState('');
const [lastName, setLastName] = useState('');
const [email, setEmail] = useState('');
const [age, setAge] = useState(0);

// ✅ GOOD - Related state in one object
const [formData, setFormData] = useState({
  firstName: '',
  lastName: '',
  email: '',
  age: 0,
});

const handleChange = (e) => {
  const { name, value } = e.target;
  setFormData(prev => ({
    ...prev,
    [name]: value,
  }));
};
```

### ✅ Derived State (Avoid Unnecessary State)

```jsx
// ❌ BAD - Storing derived data in state
const [users, setUsers] = useState([]);
const [userCount, setUserCount] = useState(0);

useEffect(() => {
  setUserCount(users.length); // Unnecessary!
}, [users]);

// ✅ GOOD - Calculate on render
const [users, setUsers] = useState([]);
const userCount = users.length; // Just use this directly!
```

---

## 6. Hooks Best Practices

### ✅ Rules of Hooks

```jsx
// ✅ GOOD - Always call hooks at the top level
const Component = () => {
  const [state, setState] = useState(0);
  const value = useContext(MyContext);
  
  useEffect(() => {
    // Effect logic
  }, []);
  
  return <div>{state}</div>;
};

// ❌ BAD - Conditional hooks
const Component = ({ shouldUseEffect }) => {
  if (shouldUseEffect) {
    useEffect(() => {}); // ❌ Don't do this!
  }
};

// ✅ GOOD - Conditional logic inside hooks
const Component = ({ shouldFetch }) => {
  useEffect(() => {
    if (shouldFetch) {
      fetchData();
    }
  }, [shouldFetch]);
};
```

### ✅ useEffect Best Practices

```jsx
// ✅ GOOD - Cleanup function
useEffect(() => {
  const subscription = api.subscribe();
  
  return () => {
    subscription.unsubscribe(); // Cleanup
  };
}, []);

// ✅ GOOD - Proper dependencies
useEffect(() => {
  fetchUser(userId);
}, [userId]); // Include all used external values

// ✅ GOOD - Separate effects for separate concerns
useEffect(() => {
  // Track page view
  analytics.track('page_view');
}, []);

useEffect(() => {
  // Fetch data
  fetchData();
}, [id]);

// ❌ BAD - Missing dependencies
useEffect(() => {
  fetchUser(userId);
}, []); // ❌ userId should be in dependencies!

// ❌ BAD - Too many dependencies (runs too often)
useEffect(() => {
  console.log('Render');
}, [obj]); // ❌ Objects recreated each render

// ✅ GOOD - Destructure needed values
useEffect(() => {
  console.log('Render');
}, [obj.id, obj.name]); // Only depend on specific values
```

### ✅ useMemo & useCallback

```jsx
// ✅ GOOD - useMemo for expensive calculations
const ExpensiveComponent = ({ items }) => {
  const sortedItems = useMemo(() => {
    return items.sort((a, b) => a.price - b.price);
  }, [items]);
  
  return <List items={sortedItems} />;
};

// ✅ GOOD - useCallback for function stability
const ParentComponent = () => {
  const [count, setCount] = useState(0);
  
  const handleClick = useCallback(() => {
    console.log('Clicked', count);
  }, [count]);
  
  return <ChildComponent onClick={handleClick} />;
};

// ❌ BAD - Overusing memoization
const Component = () => {
  const value = useMemo(() => 1 + 1, []); // ❌ Unnecessary!
  return <div>{value}</div>;
};

// ✅ GOOD - Simple calculations don't need memoization
const Component = () => {
  const value = 1 + 1; // Just do this
  return <div>{value}</div>;
};
```

### ✅ Custom Hooks

```jsx
// ✅ GOOD - Reusable logic in custom hooks
const useFetch = (url) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    const fetchData = async () => {
      try {
        setLoading(true);
        const response = await fetch(url);
        const json = await response.json();
        setData(json);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };
    
    fetchData();
  }, [url]);
  
  return { data, loading, error };
};

// Usage
const UserList = () => {
  const { data: users, loading, error } = useFetch('/api/users');
  
  if (loading) return <Spinner />;
  if (error) return <Error message={error} />;
  
  return <ul>{users.map(user => <li key={user.id}>{user.name}</li>)}</ul>;
};
```

### ✅ useRef

```jsx
// ✅ GOOD - DOM references
const InputComponent = () => {
  const inputRef = useRef(null);
  
  const focusInput = () => {
    inputRef.current?.focus();
  };
  
  return (
    <>
      <input ref={inputRef} />
      <button onClick={focusInput}>Focus Input</button>
    </>
  );
};

// ✅ GOOD - Storing mutable values (doesn't cause re-render)
const Timer = () => {
  const intervalRef = useRef(null);
  const [count, setCount] = useState(0);
  
  const startTimer = () => {
    intervalRef.current = setInterval(() => {
      setCount(c => c + 1);
    }, 1000);
  };
  
  const stopTimer = () => {
    clearInterval(intervalRef.current);
  };
  
  useEffect(() => {
    return () => clearInterval(intervalRef.current);
  }, []);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={startTimer}>Start</button>
      <button onClick={stopTimer}>Stop</button>
    </div>
  );
};
```

---

## 7. Performance Optimization

### ✅ React.memo

```jsx
// ✅ GOOD - Memoize expensive components
const ExpensiveComponent = React.memo(({ data, onAction }) => {
  return (
    <div>
      {/* Complex rendering logic */}
    </div>
  );
});

// ✅ GOOD - Custom comparison function
const UserCard = React.memo(
  ({ user }) => {
    return <div>{user.name}</div>;
  },
  (prevProps, nextProps) => {
    // Only re-render if user.id changes
    return prevProps.user.id === nextProps.user.id;
  }
);

// ❌ BAD - Memoizing everything
const SimpleText = React.memo(({ text }) => <p>{text}</p>);
// Not worth it for simple components!
```

### ✅ Code Splitting & Lazy Loading

```jsx
import { lazy, Suspense } from 'react';

// ✅ GOOD - Lazy load heavy components
const HeavyComponent = lazy(() => import('./HeavyComponent'));
const AdminPanel = lazy(() => import('./AdminPanel'));

const App = () => {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <Routes>
        <Route path="/admin" element={<AdminPanel />} />
        <Route path="/heavy" element={<HeavyComponent />} />
      </Routes>
    </Suspense>
  );
};
```

### ✅ Virtualization (Large Lists)

```jsx
import { FixedSizeList } from 'react-window';

// ✅ GOOD - For rendering thousands of items
const LargeList = ({ items }) => {
  const Row = ({ index, style }) => (
    <div style={style}>
      {items[index].name}
    </div>
  );
  
  return (
    <FixedSizeList
      height={600}
      itemCount={items.length}
      itemSize={50}
      width="100%"
    >
      {Row}
    </FixedSizeList>
  );
};
```

### ✅ Debouncing & Throttling

```jsx
import { useState, useCallback } from 'react';
import { debounce } from 'lodash';

const SearchInput = () => {
  const [query, setQuery] = useState('');
  
  // ✅ GOOD - Debounce API calls
  const debouncedSearch = useCallback(
    debounce((value) => {
      api.search(value);
    }, 500),
    []
  );
  
  const handleChange = (e) => {
    const value = e.target.value;
    setQuery(value);
    debouncedSearch(value);
  };
  
  return <input value={query} onChange={handleChange} />;
};
```

### ✅ Avoid Inline Functions & Objects

```jsx
// ❌ BAD - Creates new function on every render
const List = ({ items }) => {
  return (
    <ul>
      {items.map(item => (
        <li 
          key={item.id}
          onClick={() => console.log(item)} // ❌ New function each render
        >
          {item.name}
        </li>
      ))}
    </ul>
  );
};

// ✅ GOOD - Stable function reference
const List = ({ items }) => {
  const handleClick = useCallback((item) => {
    console.log(item);
  }, []);
  
  return (
    <ul>
      {items.map(item => (
        <ListItem 
          key={item.id}
          item={item}
          onClick={handleClick}
        />
      ))}
    </ul>
  );
};

// ❌ BAD - New object each render
<Component style={{ margin: 10 }} />

// ✅ GOOD - Define outside or use useMemo
const style = { margin: 10 };
<Component style={style} />
```

---

## 8. Error Handling

### ✅ Error Boundaries

```jsx
// ErrorBoundary.jsx
import { Component } from 'react';

class ErrorBoundary extends Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }
  
  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }
  
  componentDidCatch(error, errorInfo) {
    console.error('Error caught:', error, errorInfo);
    // Log to error reporting service
    logErrorToService(error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      return (
        <div className="error-boundary">
          <h2>Something went wrong</h2>
          <details>
            {this.state.error?.message}
          </details>
        </div>
      );
    }
    
    return this.props.children;
  }
}

// Usage
<ErrorBoundary>
  <App />
</ErrorBoundary>
```

### ✅ Try-Catch in Async Functions

```jsx
const DataComponent = () => {
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await api.getData();
        setData(response);
      } catch (err) {
        setError(err.message);
        // Optionally log to error service
        logError(err);
      }
    };
    
    fetchData();
  }, []);
  
  if (error) {
    return <ErrorDisplay message={error} />;
  }
  
  return <div>{data}</div>;
};
```

### ✅ Custom Error Hook

```jsx
const useErrorHandler = () => {
  const [error, setError] = useState(null);
  
  const handleError = useCallback((error) => {
    setError(error);
    console.error(error);
    // Send to error tracking service
    if (process.env.NODE_ENV === 'production') {
      errorTrackingService.log(error);
    }
  }, []);
  
  const clearError = useCallback(() => {
    setError(null);
  }, []);
  
  return { error, handleError, clearError };
};

// Usage
const Component = () => {
  const { error, handleError, clearError } = useErrorHandler();
  
  const fetchData = async () => {
    try {
      await api.getData();
    } catch (err) {
      handleError(err);
    }
  };
  
  if (error) {
    return <ErrorMessage error={error} onDismiss={clearError} />;
  }
  
  return <div>Content</div>;
};
```

---

## 9. Accessibility (a11y)

### ✅ Semantic HTML

```jsx
// ✅ GOOD - Semantic elements
const Article = () => {
  return (
    <article>
      <header>
        <h1>Title</h1>
      </header>
      <section>
        <p>Content</p>
      </section>
      <footer>
        <small>Published on...</small>
      </footer>
    </article>
  );
};

// ❌ BAD - Div soup
const Article = () => {
  return (
    <div>
      <div>
        <div>Title</div>
      </div>
      <div>
        <div>Content</div>
      </div>
    </div>
  );
};
```

### ✅ ARIA Attributes

```jsx
// ✅ GOOD - Proper ARIA labels
const Button = ({ onClick, isLoading, children }) => {
  return (
    <button
      onClick={onClick}
      aria-label={isLoading ? 'Loading...' : children}
      aria-busy={isLoading}
      disabled={isLoading}
    >
      {isLoading ? <Spinner /> : children}
    </button>
  );
};

const Modal = ({ isOpen, onClose, title, children }) => {
  return (
    <div
      role="dialog"
      aria-modal="true"
      aria-labelledby="modal-title"
      aria-hidden={!isOpen}
    >
      <h2 id="modal-title">{title}</h2>
      {children}
      <button onClick={onClose} aria-label="Close modal">
        ×
      </button>
    </div>
  );
};
```

### ✅ Keyboard Navigation

```jsx
const Dropdown = () => {
  const [isOpen, setIsOpen] = useState(false);
  const [activeIndex, setActiveIndex] = useState(0);
  
  const handleKeyDown = (e) => {
    switch (e.key) {
      case 'Enter':
      case ' ':
        setIsOpen(!isOpen);
        break;
      case 'Escape':
        setIsOpen(false);
        break;
      case 'ArrowDown':
        e.preventDefault();
        setActiveIndex(prev => Math.min(prev + 1, items.length - 1));
        break;
      case 'ArrowUp':
        e.preventDefault();
        setActiveIndex(prev => Math.max(prev - 1, 0));
        break;
    }
  };
  
  return (
    <div
      role="combobox"
      aria-expanded={isOpen}
      aria-haspopup="listbox"
      onKeyDown={handleKeyDown}
      tabIndex={0}
    >
      {/* Dropdown content */}
    </div>
  );
};
```

### ✅ Focus Management

```jsx
const Modal = ({ isOpen, onClose }) => {
  const closeButtonRef = useRef(null);
  
  useEffect(() => {
    if (isOpen) {
      // Focus close button when modal opens
      closeButtonRef.current?.focus();
    }
  }, [isOpen]);
  
  if (!isOpen) return null;
  
  return (
    <div role="dialog" aria-modal="true">
      <button
        ref={closeButtonRef}
        onClick={onClose}
        aria-label="Close"
      >
        ×
      </button>
      {/* Modal content */}
    </div>
  );
};
```

### ✅ Screen Reader Support

```jsx
const LoadingButton = ({ isLoading, children }) => {
  return (
    <button disabled={isLoading}>
      {children}
      {isLoading && (
        <>
          <Spinner aria-hidden="true" />
          <span className="sr-only">Loading...</span>
        </>
      )}
    </button>
  );
};

// CSS for screen reader only text
/*
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border-width: 0;
}
*/
```

---

## 10. Testing

### ✅ Component Testing with React Testing Library

```jsx
// Button.test.jsx
import { render, screen, fireEvent } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import Button from './Button';

describe('Button Component', () => {
  test('renders button with label', () => {
    render(<Button label="Click me" />);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });
  
  test('calls onClick when clicked', async () => {
    const handleClick = jest.fn();
    render(<Button label="Click" onClick={handleClick} />);
    
    await userEvent.click(screen.getByText('Click'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
  
  test('is disabled when loading', () => {
    render(<Button label="Submit" isLoading />);
    expect(screen.getByRole('button')).toBeDisabled();
  });
  
  test('shows loading spinner', () => {
    render(<Button label="Submit" isLoading />);
    expect(screen.getByRole('status')).toBeInTheDocument();
  });
});
```

### ✅ Custom Hook Testing

```jsx
// useCounter.test.js
import { renderHook, act } from '@testing-library/react';
import useCounter from './useCounter';

describe('useCounter', () => {
  test('initializes with default value', () => {
    const { result } = renderHook(() => useCounter());
    expect(result.current.count).toBe(0);
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
});
```

### ✅ Testing Async Components

```jsx
// UserList.test.jsx
import { render, screen, waitFor } from '@testing-library/react';
import UserList from './UserList';
import { server } from './mocks/server';
import { rest } from 'msw';

describe('UserList', () => {
  test('displays loading state', () => {
    render(<UserList />);
    expect(screen.getByText(/loading/i)).toBeInTheDocument();
  });
  
  test('displays users after loading', async () => {
    render(<UserList />);
    
    await waitFor(() => {
      expect(screen.getByText('John Doe')).toBeInTheDocument();
    });
  });
  
  test('displays error message on failure', async () => {
    // Override default handler to return error
    server.use(
      rest.get('/api/users', (req, res, ctx) => {
        return res(ctx.status(500));
      })
    );
    
    render(<UserList />);
    
    await waitFor(() => {
      expect(screen.getByText(/error/i)).toBeInTheDocument();
    });
  });
});
```

### ✅ Snapshot Testing

```jsx
import { render } from '@testing-library/react';
import Card from './Card';

test('matches snapshot', () => {
  const { container } = render(
    <Card title="Test" description="Description" />
  );
  expect(container).toMatchSnapshot();
});
```

---

## 11. Code Quality & Maintenance

### ✅ ESLint Configuration

```json
// .eslintrc.json
{
  "extends": [
    "react-app",
    "plugin:react/recommended",
    "plugin:react-hooks/recommended",
    "plugin:jsx-a11y/recommended"
  ],
  "rules": {
    "react/prop-types": "warn",
    "react/react-in-jsx-scope": "off",
    "react-hooks/rules-of-hooks": "error",
    "react-hooks/exhaustive-deps": "warn",
    "no-console": ["warn", { "allow": ["warn", "error"] }],
    "prefer-const": "error",
    "no-unused-vars": "warn"
  }
}
```

### ✅ Code Comments

```jsx
/**
 * Fetches and displays user profile information
 * 
 * @component
 * @param {Object} props - Component props
 * @param {string} props.userId - The ID of the user to display
 * @param {Function} props.onUpdate - Callback fired when profile is updated
 * @returns {JSX.Element} User profile component
 * 
 * @example
 * <UserProfile 
 *   userId="123" 
 *   onUpdate={(user) => console.log(user)}
 * />
 */
const UserProfile = ({ userId, onUpdate }) => {
  // Complex business logic should have comments
  const calculateDiscount = (price, userTier) => {
    // Premium users get 20% discount
    // Regular users get 10% discount
    return userTier === 'premium' ? price * 0.8 : price * 0.9;
  };
  
  // Component logic...
};
```

### ✅ Consistent File Organization

```jsx
// ✅ GOOD - Consistent export pattern
// components/index.js (Barrel export)
export { default as Button } from './Button';
export { default as Input } from './Input';
export { default as Modal } from './Modal';

// Usage in other files
import { Button, Input, Modal } from '@/components';
```

### ✅ Environment Variables

```jsx
// ✅ GOOD - Use environment variables
const API_URL = process.env.REACT_APP_API_URL;
const IS_PRODUCTION = process.env.NODE_ENV === 'production';

const config = {
  apiUrl: process.env.REACT_APP_API_URL,
  apiKey: process.env.REACT_APP_API_KEY,
  enableAnalytics: process.env.REACT_APP_ENABLE_ANALYTICS === 'true',
};

// ❌ BAD - Hardcoded values
const API_URL = 'https://api.example.com';
```

### ✅ Constants File

```jsx
// constants/index.js
export const USER_ROLES = {
  ADMIN: 'admin',
  USER: 'user',
  GUEST: 'guest',
};

export const API_ENDPOINTS = {
  USERS: '/api/users',
  POSTS: '/api/posts',
  AUTH: '/api/auth',
};

export const ROUTES = {
  HOME: '/',
  DASHBOARD: '/dashboard',
  PROFILE: '/profile',
  LOGIN: '/login',
};

export const VALIDATION_RULES = {
  EMAIL_REGEX: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
  PASSWORD_MIN_LENGTH: 8,
  USERNAME_MAX_LENGTH: 20,
};
```

---

## Quick Checklist

### Before Creating a Component

- [ ] Is this component reusable?
- [ ] Should it be a presentational or container component?
- [ ] What props does it need?
- [ ] Does it need local state?
- [ ] Will it need side effects?

### Code Quality

- [ ] Component has a single responsibility
- [ ] Props are properly typed (PropTypes/TypeScript)
- [ ] No prop drilling (use Context if needed)
- [ ] Proper naming conventions
- [ ] No magic numbers/strings (use constants)
- [ ] Loading and error states handled
- [ ] Accessible (ARIA, keyboard navigation)
- [ ] Performance optimized (memo, useMemo when needed)
- [ ] Has unit tests
- [ ] Has proper documentation/comments

### Performance

- [ ] Avoid unnecessary re-renders
- [ ] Use React.memo for expensive components
- [ ] Proper dependency arrays in useEffect
- [ ] Debounce/throttle expensive operations
- [ ] Code splitting for large components
- [ ] Virtualization for large lists

### Accessibility

- [ ] Semantic HTML elements
- [ ] Proper ARIA attributes
- [ ] Keyboard navigation support
- [ ] Screen reader friendly
- [ ] Sufficient color contrast
- [ ] Focus management

---

This guide covers modern React best practices in 2024. Remember:
- **Write simple, readable code first**
- **Optimize only when necessary**
- **Test your components**
- **Make it accessible**
- **Keep learning and adapting**

