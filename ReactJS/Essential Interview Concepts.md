# Essential React/JavaScript/ES6 Interview Concepts - Complete Guide

## 🎯 Advanced Concepts Beyond Debounce/Throttle

---

## 1️⃣ **Memoization**

### **What is it?**
Caching function results to avoid expensive recalculations.

### **Implementation**

```jsx
// Manual Memoization
const memoize = (fn) => {
  const cache = {};
  
  return (...args) => {
    const key = JSON.stringify(args);
    
    if (cache[key]) {
      console.log('Returning cached result');
      return cache[key];
    }
    
    console.log('Computing result');
    const result = fn(...args);
    cache[key] = result;
    return result;
  };
};

// Usage
const expensiveCalculation = memoize((n) => {
  let sum = 0;
  for (let i = 0; i < n; i++) {
    sum += i;
  }
  return sum;
});

console.log(expensiveCalculation(1000000)); // Computes
console.log(expensiveCalculation(1000000)); // Returns from cache

// React useMemo
const Component = ({ items, filter }) => {
  // Only recalculates when items or filter changes
  const filteredItems = useMemo(() => {
    console.log('Filtering items...');
    return items.filter(item => item.category === filter);
  }, [items, filter]);
  
  return <List items={filteredItems} />;
};

// Memoizing Components with React.memo
const ExpensiveComponent = React.memo(({ data, onUpdate }) => {
  console.log('Rendering ExpensiveComponent');
  return <div>{/* Complex rendering */}</div>;
}, (prevProps, nextProps) => {
  // Custom comparison - return true to skip re-render
  return prevProps.data.id === nextProps.data.id;
});
```

---

## 2️⃣ **Lazy Loading / Code Splitting**

### **What is it?**
Loading components/modules only when needed to reduce initial bundle size.

### **Implementation**

```jsx
// React.lazy for component lazy loading
import React, { Suspense, lazy } from 'react';

// Lazy load components
const Dashboard = lazy(() => import('./Dashboard'));
const Profile = lazy(() => import('./Profile'));
const Settings = lazy(() => import('./Settings'));

const App = () => {
  return (
    <BrowserRouter>
      <Suspense fallback={<LoadingSpinner />}>
        <Routes>
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/profile" element={<Profile />} />
          <Route path="/settings" element={<Settings />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
};

// Dynamic imports with conditions
const LazyLoadOnCondition = ({ userRole }) => {
  const [Component, setComponent] = useState(null);
  
  useEffect(() => {
    if (userRole === 'admin') {
      import('./AdminPanel').then(module => {
        setComponent(() => module.default);
      });
    } else {
      import('./UserPanel').then(module => {
        setComponent(() => module.default);
      });
    }
  }, [userRole]);
  
  if (!Component) return <Loading />;
  return <Component />;
};

// Prefetching for better UX
const PrefetchExample = () => {
  const prefetchDashboard = () => {
    import(/* webpackPrefetch: true */ './Dashboard');
  };
  
  return (
    <button onMouseEnter={prefetchDashboard}>
      Go to Dashboard
    </button>
  );
};
```

---

## 3️⃣ **Closures and Stale Closures**

### **What is it?**
Functions that remember variables from their creation scope. Common source of bugs in React.

### **Implementation**

```jsx
// ❌ PROBLEM: Stale Closure
const StaleClosureExample = () => {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      console.log(count); // Always logs 0! Stale closure
      setCount(count + 1); // Only increments once to 1
    }, 1000);
    
    return () => clearInterval(interval);
  }, []); // Empty deps creates closure over initial count
  
  return <div>{count}</div>;
};

// ✅ SOLUTION 1: Functional updates
const FixedWithFunctional = () => {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      setCount(prevCount => prevCount + 1); // Uses latest value
    }, 1000);
    
    return () => clearInterval(interval);
  }, []); // Safe with functional update
  
  return <div>{count}</div>;
};

// ✅ SOLUTION 2: useRef to access latest value
const FixedWithRef = () => {
  const [count, setCount] = useState(0);
  const countRef = useRef(count);
  
  useEffect(() => {
    countRef.current = count; // Always update ref
  }, [count]);
  
  useEffect(() => {
    const interval = setInterval(() => {
      console.log(countRef.current); // Always current value
      setCount(countRef.current + 1);
    }, 1000);
    
    return () => clearInterval(interval);
  }, []);
  
  return <div>{count}</div>;
};

// Closure in event handlers
const ClosureInHandlers = () => {
  const [items, setItems] = useState([]);
  
  const handleClick = (id) => {
    // Creates closure over items at render time
    console.log('Items length:', items.length);
  };
  
  // Better approach
  const handleClickBetter = useCallback((id) => {
    setItems(currentItems => {
      console.log('Current length:', currentItems.length);
      return currentItems;
    });
  }, []); // No dependency on items
  
  return <button onClick={() => handleClick(1)}>Click</button>;
};
```

---

## 4️⃣ **Event Delegation**

### **What is it?**
Using single parent event listener instead of multiple child listeners for performance.

### **Implementation**

```jsx
// ❌ BAD: Multiple event listeners
const BadListExample = ({ items }) => {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id} onClick={() => handleClick(item.id)}>
          {item.name}
        </li>
      ))}
    </ul>
  );
};

// ✅ GOOD: Event delegation
const GoodListExample = ({ items }) => {
  const handleListClick = (e) => {
    // Find which item was clicked
    const li = e.target.closest('li');
    if (li) {
      const itemId = li.dataset.id;
      handleClick(itemId);
    }
  };
  
  return (
    <ul onClick={handleListClick}>
      {items.map(item => (
        <li key={item.id} data-id={item.id}>
          {item.name}
        </li>
      ))}
    </ul>
  );
};

// Advanced: Event delegation with dynamic content
const DynamicList = () => {
  const listRef = useRef(null);
  
  useEffect(() => {
    const list = listRef.current;
    
    const handleClick = (e) => {
      if (e.target.matches('.delete-btn')) {
        const id = e.target.dataset.id;
        deleteItem(id);
      }
      
      if (e.target.matches('.edit-btn')) {
        const id = e.target.dataset.id;
        editItem(id);
      }
    };
    
    list.addEventListener('click', handleClick);
    return () => list.removeEventListener('click', handleClick);
  }, []);
  
  return (
    <ul ref={listRef}>
      {/* Dynamic content */}
    </ul>
  );
};
```

---

## 5️⃣ **Race Conditions**

### **What is it?**
Bugs from async operations completing in unexpected order.

### **Implementation**

```jsx
// ❌ PROBLEM: Race condition
const RaceConditionExample = ({ userId }) => {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetchUser(userId).then(data => {
      setUser(data); // Wrong user if userId changed!
    });
  }, [userId]);
  
  return <div>{user?.name}</div>;
};

// ✅ SOLUTION 1: Cleanup flag
const FixedWithFlag = ({ userId }) => {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    let isCancelled = false;
    
    fetchUser(userId).then(data => {
      if (!isCancelled) {
        setUser(data);
      }
    });
    
    return () => {
      isCancelled = true; // Ignore stale results
    };
  }, [userId]);
  
  return <div>{user?.name}</div>;
};

// ✅ SOLUTION 2: AbortController
const FixedWithAbort = ({ userId }) => {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    const controller = new AbortController();
    
    fetch(`/api/users/${userId}`, {
      signal: controller.signal
    })
      .then(res => res.json())
      .then(data => setUser(data))
      .catch(err => {
        if (err.name !== 'AbortError') {
          console.error(err);
        }
      });
    
    return () => controller.abort();
  }, [userId]);
  
  return <div>{user?.name}</div>;
};

// ✅ SOLUTION 3: Request ID tracking
const FixedWithRequestId = ({ userId }) => {
  const [user, setUser] = useState(null);
  const requestIdRef = useRef(0);
  
  useEffect(() => {
    const currentRequestId = ++requestIdRef.current;
    
    fetchUser(userId).then(data => {
      // Only update if this is still the latest request
      if (currentRequestId === requestIdRef.current) {
        setUser(data);
      }
    });
  }, [userId]);
  
  return <div>{user?.name}</div>;
};
```

---

## 6️⃣ **Currying**

### **What is it?**
Transforming function with multiple arguments into sequence of functions with single argument.

### **Implementation**

```jsx
// Basic currying
const curry = (fn) => {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    } else {
      return (...nextArgs) => {
        return curried.apply(this, [...args, ...nextArgs]);
      };
    }
  };
};

// Usage
const sum = (a, b, c) => a + b + c;
const curriedSum = curry(sum);

console.log(curriedSum(1)(2)(3)); // 6
console.log(curriedSum(1, 2)(3)); // 6
console.log(curriedSum(1)(2, 3)); // 6

// Practical React example
const useApiCall = curry((method, endpoint, data) => {
  return fetch(endpoint, {
    method,
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data)
  });
});

// Create specialized API callers
const useGet = useApiCall('GET');
const usePost = useApiCall('POST');
const usePut = useApiCall('PUT');

// Usage in component
const Component = () => {
  const postUser = usePost('/api/users');
  
  const handleSubmit = async (userData) => {
    await postUser(userData);
  };
};

// Event handler currying
const handleClick = (id) => (event) => {
  event.preventDefault();
  console.log('Clicked item:', id);
};

// In JSX
<button onClick={handleClick(item.id)}>Click</button>
```

---

## 7️⃣ **Function Composition**

### **What is it?**
Combining simple functions to build complex operations.

### **Implementation**

```jsx
// Compose utility
const compose = (...fns) => (x) => 
  fns.reduceRight((acc, fn) => fn(acc), x);

// Pipe utility (left to right)
const pipe = (...fns) => (x) => 
  fns.reduce((acc, fn) => fn(acc), x);

// Example functions
const trim = str => str.trim();
const toLowerCase = str => str.toLowerCase();
const capitalize = str => str.charAt(0).toUpperCase() + str.slice(1);
const addExclamation = str => str + '!';

// Compose them
const formatString = compose(
  addExclamation,
  capitalize,
  toLowerCase,
  trim
);

console.log(formatString('  HELLO  ')); // "Hello!"

// React example: Data transformation pipeline
const useDataPipeline = (rawData) => {
  const processData = useMemo(() => {
    const filterActive = data => data.filter(item => item.active);
    const sortByDate = data => [...data].sort((a, b) => 
      new Date(b.date) - new Date(a.date)
    );
    const addMetadata = data => data.map(item => ({
      ...item,
      displayName: `${item.firstName} ${item.lastName}`,
      formattedDate: formatDate(item.date)
    }));
    const limitResults = data => data.slice(0, 10);
    
    return pipe(
      filterActive,
      sortByDate,
      addMetadata,
      limitResults
    );
  }, []);
  
  return processData(rawData);
};

// HOC composition
const withAuth = (Component) => (props) => {
  const { isAuthenticated } = useAuth();
  return isAuthenticated ? <Component {...props} /> : <Login />;
};

const withLoading = (Component) => ({ isLoading, ...props }) => {
  return isLoading ? <Spinner /> : <Component {...props} />;
};

const withErrorBoundary = (Component) => (props) => (
  <ErrorBoundary>
    <Component {...props} />
  </ErrorBoundary>
);

// Compose HOCs
const enhance = compose(
  withAuth,
  withLoading,
  withErrorBoundary
);

const EnhancedComponent = enhance(MyComponent);
```

---

## 8️⃣ **Shallow vs Deep Copy**

### **What is it?**
Different levels of object/array copying.

### **Implementation**

```jsx
// Shallow copy - only top level
const original = {
  name: 'John',
  address: {
    city: 'NYC',
    zip: '10001'
  },
  hobbies: ['reading', 'coding']
};

// ❌ Assignment (same reference)
const assigned = original;
assigned.name = 'Jane'; // Changes original too!

// ✅ Shallow copy methods
const shallow1 = { ...original };
const shallow2 = Object.assign({}, original);
const shallow3 = { ...original };

shallow1.name = 'Jane'; // ✅ Original not affected
shallow1.address.city = 'LA'; // ❌ Original IS affected!
shallow1.hobbies.push('gaming'); // ❌ Original IS affected!

// ✅ Deep copy methods
// Method 1: JSON (limitations: no functions, dates, undefined)
const deep1 = JSON.parse(JSON.stringify(original));

// Method 2: Recursive deep copy
const deepCopy = (obj) => {
  if (obj === null || typeof obj !== 'object') return obj;
  if (obj instanceof Date) return new Date(obj);
  if (obj instanceof Array) return obj.map(item => deepCopy(item));
  
  const copy = {};
  for (let key in obj) {
    if (obj.hasOwnProperty(key)) {
      copy[key] = deepCopy(obj[key]);
    }
  }
  return copy;
};

const deep2 = deepCopy(original);

// Method 3: structuredClone (modern browsers)
const deep3 = structuredClone(original);

// Method 4: Immer library (recommended for React)
import produce from 'immer';

const nextState = produce(original, draft => {
  draft.address.city = 'LA';
  draft.hobbies.push('gaming');
});

// React state updates
const Component = () => {
  const [state, setState] = useState({
    user: { name: 'John', settings: { theme: 'dark' } },
    items: [1, 2, 3]
  });
  
  // ❌ WRONG: Mutation
  const wrong = () => {
    state.user.name = 'Jane'; // Mutates state!
    setState(state); // React won't detect change
  };
  
  // ✅ CORRECT: Shallow copy (when updating top level)
  const correct1 = () => {
    setState({
      ...state,
      user: { ...state.user, name: 'Jane' }
    });
  };
  
  // ✅ CORRECT: Using Immer for deep updates
  const correct2 = () => {
    setState(produce(state, draft => {
      draft.user.name = 'Jane';
      draft.user.settings.theme = 'light';
      draft.items.push(4);
    }));
  };
};
```

---

## 9️⃣ **Event Bubbling & Capturing**

### **What is it?**
How events propagate through DOM tree.

### **Implementation**

```jsx
// Event Phases
// 1. Capturing (top to bottom)
// 2. Target
// 3. Bubbling (bottom to top)

const EventPropagationExample = () => {
  const handleParentClick = (phase) => {
    console.log(`Parent clicked (${phase})`);
  };
  
  const handleChildClick = (phase, e) => {
    console.log(`Child clicked (${phase})`);
    // e.stopPropagation(); // Stops bubbling
  };
  
  return (
    <div 
      onClick={() => handleParentClick('bubbling')}
      onClickCapture={() => handleParentClick('capturing')}
      style={{ padding: '50px', background: 'lightblue' }}
    >
      Parent
      
      <button
        onClick={(e) => handleChildClick('bubbling', e)}
        onClickCapture={(e) => handleChildClick('capturing', e)}
        style={{ padding: '20px' }}
      >
        Child
      </button>
    </div>
  );
  
  // Click order with bubbling:
  // 1. Parent capturing
  // 2. Child capturing
  // 3. Child bubbling
  // 4. Parent bubbling
};

// Practical: Stop propagation
const Modal = ({ onClose, children }) => {
  return (
    <div className="modal-overlay" onClick={onClose}>
      <div 
        className="modal-content"
        onClick={(e) => e.stopPropagation()} // Don't close on content click
      >
        {children}
      </div>
    </div>
  );
};

// Event delegation with bubbling
const TodoList = () => {
  const handleListClick = (e) => {
    // Check what was clicked using bubbling
    if (e.target.matches('.delete-btn')) {
      const id = e.target.closest('.todo-item').dataset.id;
      deleteTodo(id);
    }
    
    if (e.target.matches('.checkbox')) {
      const id = e.target.closest('.todo-item').dataset.id;
      toggleTodo(id);
    }
  };
  
  return (
    <ul onClick={handleListClick}>
      {todos.map(todo => (
        <li key={todo.id} className="todo-item" data-id={todo.id}>
          <input type="checkbox" className="checkbox" />
          <span>{todo.text}</span>
          <button className="delete-btn">Delete</button>
        </li>
      ))}
    </ul>
  );
};
```

---

## 🔟 **Higher-Order Functions (HOF)**

### **What is it?**
Functions that take functions as arguments or return functions.

### **Implementation**

```jsx
// Basic HOF
const withLogging = (fn) => {
  return (...args) => {
    console.log(`Called with:`, args);
    const result = fn(...args);
    console.log(`Result:`, result);
    return result;
  };
};

const add = (a, b) => a + b;
const loggedAdd = withLogging(add);
console.log(loggedAdd(2, 3)); // Logs args, result, returns 5

// Array HOF methods
const numbers = [1, 2, 3, 4, 5];

// map - transform each element
const doubled = numbers.map(n => n * 2); // [2, 4, 6, 8, 10]

// filter - select elements
const evens = numbers.filter(n => n % 2 === 0); // [2, 4]

// reduce - aggregate to single value
const sum = numbers.reduce((acc, n) => acc + n, 0); // 15

// some - check if any match
const hasLarge = numbers.some(n => n > 3); // true

// every - check if all match
const allPositive = numbers.every(n => n > 0); // true

// React HOC
const withAuthentication = (WrappedComponent) => {
  return (props) => {
    const { user, loading } = useAuth();
    
    if (loading) return <Spinner />;
    if (!user) return <Redirect to="/login" />;
    
    return <WrappedComponent {...props} user={user} />;
  };
};

const ProtectedPage = withAuthentication(({ user }) => {
  return <div>Welcome, {user.name}!</div>;
});

// Function composition HOF
const enhance = (...hocs) => (Component) => {
  return hocs.reduce((acc, hoc) => hoc(acc), Component);
};

const SuperComponent = enhance(
  withAuth,
  withLoading,
  withErrorBoundary,
  withAnalytics
)(MyComponent);

// Practical: Retry logic
const withRetry = (fn, maxAttempts = 3) => {
  return async (...args) => {
    for (let attempt = 1; attempt <= maxAttempts; attempt++) {
      try {
        return await fn(...args);
      } catch (error) {
        if (attempt === maxAttempts) throw error;
        console.log(`Attempt ${attempt} failed, retrying...`);
        await new Promise(resolve => 
          setTimeout(resolve, 1000 * attempt)
        );
      }
    }
  };
};

const fetchWithRetry = withRetry(fetch);
```

---

## 1️⃣1️⃣ **Reconciliation & Keys in React**

### **What is it?**
How React determines what changed and needs re-rendering.

### **Implementation**

```jsx
// ❌ BAD: Using index as key
const BadList = ({ items }) => {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>{item.name}</li> // Breaks on reorder!
      ))}
    </ul>
  );
};

// ✅ GOOD: Using stable unique IDs
const GoodList = ({ items }) => {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
};

// Problem demonstration
const ProblemDemo = () => {
  const [items, setItems] = useState([
    { id: 1, name: 'Apple' },
    { id: 2, name: 'Banana' },
    { id: 3, name: 'Cherry' }
  ]);
  
  const shuffle = () => {
    setItems([...items].sort(() => Math.random() - 0.5));
  };
  
  return (
    <div>
      <button onClick={shuffle}>Shuffle</button>
      
      {/* With index - components get mixed up */}
      {items.map((item, index) => (
        <ItemWithState key={index} item={item} />
      ))}
      
      {/* With ID - components maintain state correctly */}
      {items.map(item => (
        <ItemWithState key={item.id} item={item} />
      ))}
    </div>
  );
};

// Component with internal state
const ItemWithState = ({ item }) => {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      {item.name} - Count: {count}
      <button onClick={() => setCount(c => c + 1)}>+</button>
    </div>
  );
};

// Reconciliation optimization
const OptimizedList = ({ items }) => {
  return (
    <>
      {items.map(item => (
        // React.memo prevents re-render if props haven't changed
        <MemoizedItem key={item.id} item={item} />
      ))}
    </>
  );
};

const MemoizedItem = React.memo(({ item }) => {
  console.log('Rendering:', item.name);
  return <div>{item.name}</div>;
}, (prevProps, nextProps) => {
  // Custom comparison - return true to skip re-render
  return prevProps.item.id === nextProps.item.id &&
         prevProps.item.name === nextProps.item.name;
});
```

---

## 1️⃣2️⃣ **Error Boundaries**

### **What is it?**
Catching JavaScript errors in component tree and displaying fallback UI.

### **Implementation**

```jsx
// Error Boundary (must be class component)
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null, errorInfo: null };
  }
  
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }
  
  componentDidCatch(error, errorInfo) {
    // Log to error reporting service
    console.error('Error caught:', error, errorInfo);
    
    this.setState({
      error: error,
      errorInfo: errorInfo
    });
    
    // Send to Azure Application Insights or Sentry
    logErrorToService(error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      return (
        <div className="error-boundary">
          <h2>Something went wrong</h2>
          <details>
            <summary>Error details</summary>
            <p>{this.state.error?.toString()}</p>
            <pre>{this.state.errorInfo?.componentStack}</pre>
          </details>
          <button onClick={() => this.setState({ hasError: false })}>
            Try again
          </button>
        </div>
      );
    }
    
    return this.props.children;
  }
}

// Usage
const App = () => {
  return (
    <ErrorBoundary>
      <ComponentThatMightError />
    </ErrorBoundary>
  );
};

// Multiple error boundaries for granular control
const GranularErrorHandling = () => {
  return (
    <div>
      <ErrorBoundary fallback={<h3>Sidebar error</h3>}>
        <Sidebar />
      </ErrorBoundary>
      
      <ErrorBoundary fallback={<h3>Main content error</h3>}>
        <MainContent />
      </ErrorBoundary>
    </div>
  );
};

// With retry logic
class ErrorBoundaryWithRetry extends React.Component {
  state = { hasError: false, retryCount: 0 };
  
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }
  
  retry = () => {
    this.setState(prev => ({
      hasError: false,
      retryCount: prev.retryCount + 1
    }));
  };
  
  render() {
    if (this.state.hasError) {
      return (
        <div>
          <h3>Error occurred (Attempts: {this.state.retryCount})</h3>
          <button onClick={this.retry}>Retry</button>
        </div>
      );
    }
    
    return this.props.children;
  }
}

// Note: Error boundaries don't catch:
// - Event handlers (use try-catch)
// - Async code (use try-catch)
// - Server-side rendering
// - Errors in the error boundary itself
```

---

## 📊 Comparison Table of Key Concepts

| **Concept** | **Use Case** | **Performance Impact** | **Complexity** |
|------------|-------------|----------------------|---------------|
| **Debounce** | Search, form validation | High (reduces calls) | Low |
| **Throttle** | Scroll, resize | High (limits rate) | Low |
| **Memoization** | Expensive calculations | High (caches results) | Medium |
| **Lazy Loading** | Large apps, routes | Very High (reduces bundle) | Medium |
| **Closures** | Event handlers, hooks | Low | Medium |
| **Event Delegation** | Large lists | High (fewer listeners) | Low |
| **Race Conditions** | Async data fetching | N/A (prevents bugs) | High |
| **Currying** | Reusable functions | Low | Medium |
| **Composition** | Code reusability | Low | High |
| **Deep Copy** | Complex state updates | Medium (memory) | Low |

---

## 🎯 Interview Preparation Checklist

### **Must Know Concepts:**
✅ Debounce vs Throttle  
✅ useMemo vs useCallback  
✅ React.memo usage  
✅ Lazy loading with React.lazy  
✅ Closure and stale closures  
✅ Event bubbling/capturing  
✅ Keys in lists  
✅ Error boundaries  
✅ Race condition handling  
✅ Shallow vs deep copy  

### **Advanced Concepts:**
✅ Currying and partial application  
✅ Function composition  
✅ Higher-order functions  
✅ Event delegation  
✅ Reconciliation algorithm  
✅ Custom hooks patterns  
✅ Code splitting strategies  
✅ Performance optimization techniques  

### **Common Interview Questions:**
1. "Implement debounce/throttle from scratch"
2. "Fix this race condition bug"
3. "Why is my component re-rendering?"
4. "Explain stale closure with example"
5. "When to use useMemo vs useCallback?"
6. "How to prevent multiple form submissions?"
7. "What's wrong with using index as key?"
8. "How does Virtual DOM reconciliation work?"
9. "Implement error boundary"
10. "Optimize this slow list rendering"

---

This comprehensive guide covers the essential concepts beyond debounce/throttle that frequently appear in React/JavaScript interviews. Each concept includes practical implementations and real-world scenarios you'll encounter in production applications.