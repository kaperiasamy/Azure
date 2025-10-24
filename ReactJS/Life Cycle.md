# React Application and Component Lifecycle - Complete Guide

## 📱 React Application Lifecycle

### **1. Application Initialization Phase**

```jsx
// 1. Application Entry Point (index.js or main.jsx)
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
import './index.css';

// Application lifecycle starts here
console.log('1. Application loading started');

// 2. Create root element
const rootElement = document.getElementById('root');
const root = ReactDOM.createRoot(rootElement);

// 3. Initial render triggers component lifecycle
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);

// 4. Application is now mounted and running
console.log('2. Application mounted to DOM');
```

### **2. Application Runtime Phases**

```jsx
// Complete Application Lifecycle Example
const ApplicationLifecycle = () => {
  // PHASE 1: INITIALIZATION
  console.log('App: Initialization phase');
  
  // PHASE 2: FIRST RENDER
  useEffect(() => {
    console.log('App: First render complete - DOM ready');
    
    // PHASE 3: RUNTIME
    // Set up global listeners, services, etc.
    const handleOnline = () => console.log('App: Online');
    const handleOffline = () => console.log('App: Offline');
    
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    
    // Initialize services
    initializeAnalytics();
    initializeErrorTracking();
    setupAuthenticationListener();
    
    // PHASE 4: CLEANUP (on unmount)
    return () => {
      console.log('App: Cleanup phase');
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
      cleanupServices();
    };
  }, []);
  
  // PHASE 5: RE-RENDERS (on state/props changes)
  const [appState, setAppState] = useState('initialized');
  
  useEffect(() => {
    console.log('App: Re-render triggered by state change:', appState);
  }, [appState]);
  
  return (
    <div className="application">
      <Router>
        <AuthProvider>
          <ThemeProvider>
            <ErrorBoundary>
              <Routes>
                <Route path="/" element={<Home />} />
                <Route path="/dashboard" element={<Dashboard />} />
              </Routes>
            </ErrorBoundary>
          </ThemeProvider>
        </AuthProvider>
      </Router>
    </div>
  );
};
```

### **3. Application Lifecycle Flow Diagram**

```
Application Start
       ↓
  Load Bundle
       ↓
  Parse JavaScript
       ↓
  Execute index.js
       ↓
  ReactDOM.createRoot()
       ↓
  root.render(<App />)
       ↓
  Component Tree Creation
       ↓
  Virtual DOM Creation
       ↓
  Initial DOM Render
       ↓
  useEffect() Execution
       ↓
  Application Ready
       ↓
  [Runtime Events]
    - State Changes
    - Props Updates
    - User Interactions
    - API Responses
       ↓
  Re-renders
       ↓
  [Unmount]
  Cleanup & Destroy
```

---

## 🔄 React Component Lifecycle

## **Class Component Lifecycle**

### **1. Complete Class Component Lifecycle Methods**

```jsx
class CompleteLifecycleComponent extends React.Component {
  // 1. INITIALIZATION PHASE
  constructor(props) {
    super(props);
    console.log('1. Constructor - Component being created');
    
    // Initialize state
    this.state = {
      count: 0,
      data: null,
      error: null
    };
    
    // Bind methods (if not using arrow functions)
    this.handleClick = this.handleClick.bind(this);
  }
  
  // 2. MOUNTING PHASE
  static getDerivedStateFromProps(props, state) {
    console.log('2. getDerivedStateFromProps - Sync state with props');
    
    // Return updated state based on props, or null if no change
    if (props.initialCount !== state.count) {
      return { count: props.initialCount };
    }
    return null;
  }
  
  componentDidMount() {
    console.log('4. componentDidMount - Component mounted to DOM');
    
    // Safe to:
    // - Make API calls
    // - Add event listeners
    // - Start timers
    // - Access DOM nodes
    
    this.fetchData();
    this.setupSubscriptions();
    
    window.addEventListener('resize', this.handleResize);
    this.timer = setInterval(this.tick, 1000);
  }
  
  // 3. UPDATING PHASE
  shouldComponentUpdate(nextProps, nextState) {
    console.log('5. shouldComponentUpdate - Decide if re-render needed');
    
    // Performance optimization - prevent unnecessary renders
    if (this.props.userId !== nextProps.userId) {
      return true; // Re-render
    }
    
    if (this.state.count !== nextState.count) {
      return true; // Re-render
    }
    
    return false; // Skip re-render
  }
  
  getSnapshotBeforeUpdate(prevProps, prevState) {
    console.log('7. getSnapshotBeforeUpdate - Capture DOM state before update');
    
    // Capture scroll position, selection, etc.
    if (prevState.items.length < this.state.items.length) {
      const list = document.getElementById('list');
      return list.scrollHeight;
    }
    return null;
  }
  
  componentDidUpdate(prevProps, prevState, snapshot) {
    console.log('8. componentDidUpdate - Component updated');
    
    // Handle post-update logic
    if (this.props.userId !== prevProps.userId) {
      this.fetchUserData(this.props.userId);
    }
    
    // Use snapshot from getSnapshotBeforeUpdate
    if (snapshot !== null) {
      const list = document.getElementById('list');
      list.scrollTop = list.scrollHeight - snapshot;
    }
  }
  
  // 4. UNMOUNTING PHASE
  componentWillUnmount() {
    console.log('9. componentWillUnmount - Cleanup before destruction');
    
    // Clean up:
    // - Cancel API calls
    // - Remove event listeners
    // - Clear timers
    // - Unsubscribe from services
    
    window.removeEventListener('resize', this.handleResize);
    clearInterval(this.timer);
    this.cancelRequests();
    this.unsubscribeAll();
  }
  
  // 5. ERROR HANDLING PHASE
  static getDerivedStateFromError(error) {
    console.log('Error caught - Update state for error UI');
    
    // Update state to show error UI
    return { hasError: true, error: error.message };
  }
  
  componentDidCatch(error, errorInfo) {
    console.log('componentDidCatch - Log error information');
    
    // Log error to error reporting service
    logErrorToService(error, errorInfo);
    
    // Can also update state here if needed
    this.setState({
      error: error.toString(),
      errorInfo: errorInfo.componentStack
    });
  }
  
  // Helper methods
  fetchData = async () => {
    try {
      const response = await fetch(`/api/data/${this.props.id}`);
      const data = await response.json();
      this.setState({ data });
    } catch (error) {
      this.setState({ error: error.message });
    }
  };
  
  handleClick() {
    this.setState(prevState => ({
      count: prevState.count + 1
    }));
  }
  
  render() {
    console.log('3/6. Render - Creating Virtual DOM');
    
    if (this.state.hasError) {
      return <ErrorFallback error={this.state.error} />;
    }
    
    return (
      <div>
        <h1>Count: {this.state.count}</h1>
        <button onClick={this.handleClick}>Increment</button>
        {this.state.data && <DataDisplay data={this.state.data} />}
      </div>
    );
  }
}
```

### **2. Class Component Lifecycle Order**

```jsx
// MOUNTING ORDER:
// 1. constructor()
// 2. static getDerivedStateFromProps()
// 3. render()
// 4. componentDidMount()

// UPDATING ORDER:
// 1. static getDerivedStateFromProps()
// 2. shouldComponentUpdate()
// 3. render()
// 4. getSnapshotBeforeUpdate()
// 5. componentDidUpdate()

// UNMOUNTING ORDER:
// 1. componentWillUnmount()

// ERROR HANDLING:
// 1. static getDerivedStateFromError()
// 2. componentDidCatch()
```

---

## **Functional Component Lifecycle (with Hooks)**

### **1. Complete Functional Component Lifecycle**

```jsx
const CompleteFunctionalLifecycle = ({ userId, initialData }) => {
  console.log('Component function executing (every render)');
  
  // 1. INITIALIZATION - Runs on every render
  const [count, setCount] = useState(0);
  const [data, setData] = useState(initialData);
  const [error, setError] = useState(null);
  
  // 2. MOUNTING EFFECT - Runs once after first render
  useEffect(() => {
    console.log('Component mounted (like componentDidMount)');
    
    // Setup operations
    const setupComponent = async () => {
      await initializeServices();
      await fetchInitialData();
    };
    
    setupComponent();
    
    // Subscribe to events
    const subscription = subscribeToUpdates();
    
    // Add event listeners
    window.addEventListener('resize', handleResize);
    
    // CLEANUP - Runs on unmount
    return () => {
      console.log('Component unmounting (like componentWillUnmount)');
      
      // Cleanup operations
      subscription.unsubscribe();
      window.removeEventListener('resize', handleResize);
      cancelAllRequests();
    };
  }, []); // Empty deps = mount/unmount only
  
  // 3. UPDATE EFFECT - Runs when dependencies change
  useEffect(() => {
    console.log('UserId changed (like componentDidUpdate)');
    
    if (userId) {
      fetchUserData(userId);
    }
    
    // Cleanup for this specific effect
    return () => {
      console.log('Cleaning up user effect');
      cancelUserDataRequest();
    };
  }, [userId]); // Runs when userId changes
  
  // 4. LAYOUT EFFECT - Runs synchronously after DOM updates
  useLayoutEffect(() => {
    console.log('DOM updated, before browser paint');
    
    // Measure DOM elements
    const element = document.getElementById('myElement');
    if (element) {
      const rect = element.getBoundingClientRect();
      updatePosition(rect);
    }
  }, [data]);
  
  // 5. MEMOIZATION - Performance optimization
  const expensiveValue = useMemo(() => {
    console.log('Computing expensive value');
    return computeExpensiveValue(data);
  }, [data]);
  
  const memoizedCallback = useCallback(() => {
    console.log('Creating memoized callback');
    performAction(count);
  }, [count]);
  
  // 6. REF - Persist values across renders
  const renderCount = useRef(0);
  const previousUserId = useRef(userId);
  
  useEffect(() => {
    renderCount.current++;
    console.log(`Render count: ${renderCount.current}`);
    
    if (previousUserId.current !== userId) {
      console.log('UserId changed from', previousUserId.current, 'to', userId);
      previousUserId.current = userId;
    }
  });
  
  // 7. CUSTOM HOOKS - Encapsulate lifecycle logic
  const { loading, error: apiError } = useApiData(userId);
  const isOnline = useOnlineStatus();
  const windowSize = useWindowSize();
  
  // Helper functions
  const fetchUserData = async (id) => {
    try {
      const response = await fetch(`/api/users/${id}`);
      const userData = await response.json();
      setData(userData);
    } catch (err) {
      setError(err.message);
    }
  };
  
  const handleResize = () => {
    console.log('Window resized');
  };
  
  // RENDER PHASE
  console.log('Rendering component');
  
  return (
    <div>
      <h1>Functional Component Lifecycle</h1>
      <p>Count: {count}</p>
      <p>Render count: {renderCount.current}</p>
      <p>Online: {isOnline ? 'Yes' : 'No'}</p>
      <p>Window size: {windowSize.width} x {windowSize.height}</p>
      
      {loading && <LoadingSpinner />}
      {error && <ErrorMessage message={error} />}
      {data && <DataDisplay data={data} />}
      
      <button onClick={() => setCount(c => c + 1)}>
        Increment
      </button>
    </div>
  );
};
```

### **2. Hooks Lifecycle Mapping**

```jsx
// CLASS COMPONENT → FUNCTIONAL COMPONENT MAPPING

// constructor → useState initial value
const [state, setState] = useState(() => {
  // Expensive initial state
  return computeInitialState();
});

// componentDidMount → useEffect with []
useEffect(() => {
  // Mount logic
  return () => {
    // componentWillUnmount logic
  };
}, []);

// componentDidUpdate → useEffect with deps
useEffect(() => {
  // Update logic
}, [dependency]);

// shouldComponentUpdate → React.memo
const MemoizedComponent = React.memo(Component, (prevProps, nextProps) => {
  // Return true if props are equal (skip re-render)
  return prevProps.id === nextProps.id;
});

// getDerivedStateFromProps → useState with props
const [state, setState] = useState(props.initialValue);
useEffect(() => {
  setState(props.value);
}, [props.value]);

// getSnapshotBeforeUpdate → useLayoutEffect with ref
const prevValueRef = useRef();
useLayoutEffect(() => {
  prevValueRef.current = value;
});

// componentDidCatch → Error Boundary (still class component)
class ErrorBoundary extends React.Component {
  componentDidCatch(error, errorInfo) {
    // Handle error
  }
}
```

---

## **Advanced Lifecycle Patterns**

### **1. Conditional Lifecycle Effects**

```jsx
const ConditionalLifecycle = ({ enabled, data }) => {
  // Conditional mounting
  useEffect(() => {
    if (!enabled) return;
    
    console.log('Feature enabled - setting up');
    const cleanup = setupFeature();
    
    return () => {
      console.log('Feature disabled - cleaning up');
      cleanup();
    };
  }, [enabled]);
  
  // Conditional data fetching
  useEffect(() => {
    if (!data?.id) return;
    
    const controller = new AbortController();
    
    fetchDetails(data.id, controller.signal)
      .then(setDetails)
      .catch(handleError);
    
    return () => controller.abort();
  }, [data?.id]);
  
  return <div>{/* Component JSX */}</div>;
};
```

### **2. Lifecycle with Suspense and Concurrent Features**

```jsx
// React 18+ Concurrent Features
const ConcurrentLifecycle = () => {
  const [isPending, startTransition] = useTransition();
  const [data, setData] = useState(null);
  const deferredData = useDeferredValue(data);
  
  // Non-urgent updates
  const handleSearch = (query) => {
    startTransition(() => {
      // This state update can be interrupted
      setData(searchResults(query));
    });
  };
  
  // Suspense for data fetching
  return (
    <Suspense fallback={<Loading />}>
      <DataComponent data={deferredData} />
    </Suspense>
  );
};

// With Suspense for lazy loading
const LazyComponent = lazy(() => import('./HeavyComponent'));

const App = () => {
  return (
    <Suspense fallback={<div>Loading component...</div>}>
      <LazyComponent />
    </Suspense>
  );
};
```

### **3. Custom Hook Lifecycle**

```jsx
// Custom hook with complete lifecycle
const useComponentLifecycle = (componentName) => {
  const [lifecycle, setLifecycle] = useState('initializing');
  const mountTime = useRef(Date.now());
  
  // Mount
  useEffect(() => {
    console.log(`${componentName}: Mounted`);
    setLifecycle('mounted');
    
    // Unmount
    return () => {
      const lifespan = Date.now() - mountTime.current;
      console.log(`${componentName}: Unmounted after ${lifespan}ms`);
    };
  }, [componentName]);
  
  // Update tracking
  useEffect(() => {
    if (lifecycle === 'mounted') {
      console.log(`${componentName}: Updated`);
      setLifecycle('updated');
    }
  });
  
  return lifecycle;
};

// Usage
const MyComponent = () => {
  const lifecycle = useComponentLifecycle('MyComponent');
  
  return <div>Lifecycle: {lifecycle}</div>;
};
```

---

## **Lifecycle Performance Optimization**

### **1. Optimization Strategies**

```jsx
const OptimizedLifecycle = ({ data, onUpdate }) => {
  // 1. Lazy initial state
  const [expensiveState, setExpensiveState] = useState(() => {
    return computeExpensiveInitialState(data);
  });
  
  // 2. Debounced effects
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearchTerm = useDebounce(searchTerm, 500);
  
  useEffect(() => {
    if (debouncedSearchTerm) {
      performSearch(debouncedSearchTerm);
    }
  }, [debouncedSearchTerm]);
  
  // 3. Cleanup tracking
  useEffect(() => {
    let isMounted = true;
    
    fetchData().then(result => {
      if (isMounted) {
        setData(result);
      }
    });
    
    return () => {
      isMounted = false;
    };
  }, []);
  
  // 4. Memoized children
  const childComponent = useMemo(() => {
    return <ExpensiveChild data={data} />;
  }, [data]);
  
  return <div>{childComponent}</div>;
};
```

### **2. Common Lifecycle Pitfalls and Solutions**

```jsx
const LifecyclePitfalls = () => {
  // ❌ PITFALL 1: Missing dependencies
  const [count, setCount] = useState(0);
  
  // Bad - missing count dependency
  useEffect(() => {
    console.log(count); // Stale closure
  }, []); // Missing [count]
  
  // ✅ SOLUTION: Include all dependencies
  useEffect(() => {
    console.log(count); // Always current
  }, [count]);
  
  // ❌ PITFALL 2: Memory leaks
  useEffect(() => {
    const timer = setInterval(() => {
      // Do something
    }, 1000);
    // Missing cleanup!
  }, []);
  
  // ✅ SOLUTION: Always cleanup
  useEffect(() => {
    const timer = setInterval(() => {
      // Do something
    }, 1000);
    
    return () => clearInterval(timer);
  }, []);
  
  // ❌ PITFALL 3: Infinite loops
  const [data, setData] = useState({});
  
  useEffect(() => {
    setData({ value: 1 }); // Creates new object every time
  }, [data]); // Infinite loop!
  
  // ✅ SOLUTION: Stable references
  useEffect(() => {
    setData(prevData => {
      if (prevData.value !== 1) {
        return { value: 1 };
      }
      return prevData;
    });
  }, []); // No dependency on data
};
```

---

## **Lifecycle Testing**

```jsx
// Testing component lifecycle with React Testing Library
import { render, screen, waitFor, act } from '@testing-library/react';

describe('Component Lifecycle', () => {
  it('should execute lifecycle methods in order', async () => {
    const lifecycleEvents = [];
    
    const TestComponent = () => {
      useEffect(() => {
        lifecycleEvents.push('mount');
        
        return () => {
          lifecycleEvents.push('unmount');
        };
      }, []);
      
      useEffect(() => {
        lifecycleEvents.push('update');
      });
      
      return <div>Test</div>;
    };
    
    const { unmount } = render(<TestComponent />);
    
    await waitFor(() => {
      expect(lifecycleEvents).toEqual(['update', 'mount']);
    });
    
    act(() => {
      unmount();
    });
    
    expect(lifecycleEvents).toContain('unmount');
  });
});
```

---

## **Quick Reference: Lifecycle Methods Comparison**

| Phase | Class Component | Functional Component |
|-------|----------------|---------------------|
| **Initialization** | constructor() | useState() |
| **Mount** | componentDidMount() | useEffect(() => {}, []) |
| **Update** | componentDidUpdate() | useEffect(() => {}, [deps]) |
| **Unmount** | componentWillUnmount() | useEffect(() => { return () => {} }, []) |
| **Error** | componentDidCatch() | Error Boundary (class only) |
| **Performance** | shouldComponentUpdate() | React.memo() |
| **DOM Timing** | getSnapshotBeforeUpdate() | useLayoutEffect() |

---

## **Best Practices**

1. **Use functional components** with hooks for new code
2. **Always cleanup** effects (timers, subscriptions, listeners)
3. **Specify dependencies** correctly in useEffect
4. **Avoid side effects** in render phase
5. **Use useLayoutEffect** sparingly (blocks paint)
6. **Memoize expensive operations** with useMemo
7. **Handle errors** with Error Boundaries
8. **Test lifecycle behavior** not implementation
9. **Profile performance** with React DevTools
10. **Document complex lifecycle logic** for team members

This comprehensive guide covers both application and component lifecycles in React, providing practical examples and best practices for modern React development.

---

# React Lifecycle - Interview Ready Summary

## 🎯 Quick Interview Reference Card

### **Application Lifecycle Overview**
```
1. Bundle Load → 2. Parse JS → 3. Execute index.js → 4. ReactDOM.createRoot() 
→ 5. Initial Render → 6. Component Mount → 7. Runtime (State/Props Updates) 
→ 8. Unmount/Cleanup
```

---

## 📊 Component Lifecycle - Quick Comparison

### **Class Components vs Functional Components**

| **Phase** | **Class Component** | **Functional (Hooks)** | **When It Runs** |
|-----------|-------------------|---------------------|-----------------|
| **Initialize** | `constructor()` | `useState(initialValue)` | Once on creation |
| **Mount** | `componentDidMount()` | `useEffect(() => {}, [])` | After first render |
| **Update** | `componentDidUpdate()` | `useEffect(() => {}, [deps])` | After renders (deps change) |
| **Unmount** | `componentWillUnmount()` | `useEffect(() => return cleanup, [])` | Before removal |
| **Error** | `componentDidCatch()` | Error Boundary (class only) | On error in children |

---

## 🔄 Class Component Lifecycle Methods

### **Mounting Phase (Order: 1→2→3→4)**
```javascript
1. constructor(props)           // Initialize state, bind methods
2. getDerivedStateFromProps()   // Sync state with props (rare)
3. render()                     // Return JSX
4. componentDidMount()          // API calls, subscriptions, DOM access
```

### **Updating Phase (Order: 1→2→3→4→5)**
```javascript
1. getDerivedStateFromProps()   // Sync state with props
2. shouldComponentUpdate()      // Performance optimization (return boolean)
3. render()                     // Return JSX
4. getSnapshotBeforeUpdate()    // Capture DOM state before changes
5. componentDidUpdate()         // Respond to updates, API calls
```

### **Unmounting Phase**
```javascript
componentWillUnmount()          // Cleanup: timers, subscriptions, listeners
```

---

## ⚛️ Functional Component Lifecycle (Hooks)

### **Core Lifecycle Hooks - Interview Essentials**

```javascript
// 1. MOUNTING - Runs once after first render
useEffect(() => {
  console.log('Component mounted');
  
  // API calls, subscriptions
  fetchData();
  const subscription = subscribe();
  
  // UNMOUNTING - Cleanup function
  return () => {
    console.log('Component will unmount');
    subscription.unsubscribe();
  };
}, []); // Empty array = mount/unmount only

// 2. UPDATING - Runs when dependencies change
useEffect(() => {
  console.log('userId changed');
  fetchUserData(userId);
}, [userId]); // Runs when userId changes

// 3. EVERY RENDER - No dependency array
useEffect(() => {
  console.log('Runs after every render');
}); // No array = runs every render
```

---

## 💡 Key Interview Questions & Answers

### **Q1: What's the difference between componentDidMount and useEffect?**
```javascript
// Class Component
componentDidMount() {
  fetchData(); // Runs once after mount
}

// Functional Component
useEffect(() => {
  fetchData(); // Same behavior with empty deps
}, []); // Empty array crucial!
```
**Answer:** Both run after first render, but `useEffect` is more flexible - can handle mount, update, and unmount in one place.

### **Q2: How do you prevent memory leaks in React?**
```javascript
// ✅ CORRECT - With cleanup
useEffect(() => {
  const timer = setInterval(() => {}, 1000);
  const listener = window.addEventListener('resize', handleResize);
  
  return () => {  // Cleanup function
    clearInterval(timer);
    window.removeEventListener('resize', handleResize);
  };
}, []);

// ❌ WRONG - No cleanup (memory leak!)
useEffect(() => {
  setInterval(() => {}, 1000); // Never cleared!
}, []);
```

### **Q3: When to use useLayoutEffect vs useEffect?**
```javascript
// useEffect - Most common, runs after paint (non-blocking)
useEffect(() => {
  // API calls, subscriptions, analytics
});

// useLayoutEffect - Runs before paint (blocking)
useLayoutEffect(() => {
  // DOM measurements, prevent flicker
  const element = ref.current;
  const height = element.scrollHeight;
});
```
**Answer:** Use `useLayoutEffect` only for DOM measurements or to prevent visual flicker. Default to `useEffect`.

### **Q4: How do you replicate componentDidUpdate with specific prop?**
```javascript
// Class Component
componentDidUpdate(prevProps) {
  if (prevProps.userId !== this.props.userId) {
    this.fetchUser(this.props.userId);
  }
}

// Functional Component
useEffect(() => {
  fetchUser(userId);
}, [userId]); // Only runs when userId changes
```

---

## 🚨 Common Pitfalls (Interview Red Flags)

### **1. Missing Dependencies**
```javascript
// ❌ WRONG - Stale closure bug
const [count, setCount] = useState(0);
useEffect(() => {
  console.log(count); // Will always log 0!
}, []); // Missing count dependency

// ✅ CORRECT
useEffect(() => {
  console.log(count); // Current value
}, [count]);
```

### **2. Infinite Loops**
```javascript
// ❌ WRONG - Infinite loop
useEffect(() => {
  setData({}); // New object every time
}, [data]); // Triggers on every render

// ✅ CORRECT
useEffect(() => {
  setData(prevData => ({...prevData, updated: true}));
}, []); // No dependency on data
```

### **3. Async in useEffect**
```javascript
// ❌ WRONG
useEffect(async () => {
  await fetchData(); // Can't make useEffect async
}, []);

// ✅ CORRECT
useEffect(() => {
  const loadData = async () => {
    await fetchData();
  };
  loadData();
}, []);
```

---

## 📝 Quick Code Snippets for Interviews

### **Complete Lifecycle Example - Functional Component**
```javascript
const InterviewComponent = ({ userId }) => {
  const [data, setData] = useState(null);
  
  // Mount & Unmount
  useEffect(() => {
    console.log('Mounted');
    return () => console.log('Unmounted');
  }, []);
  
  // Update on prop change
  useEffect(() => {
    if (userId) {
      fetch(`/api/user/${userId}`)
        .then(res => res.json())
        .then(setData);
    }
  }, [userId]);
  
  return <div>{data?.name}</div>;
};
```

### **Complete Lifecycle Example - Class Component**
```javascript
class InterviewComponent extends React.Component {
  state = { data: null };
  
  componentDidMount() {
    console.log('Mounted');
    this.fetchData();
  }
  
  componentDidUpdate(prevProps) {
    if (prevProps.userId !== this.props.userId) {
      this.fetchData();
    }
  }
  
  componentWillUnmount() {
    console.log('Unmounting');
    // Cleanup
  }
  
  fetchData = () => {
    fetch(`/api/user/${this.props.userId}`)
      .then(res => res.json())
      .then(data => this.setState({ data }));
  };
  
  render() {
    return <div>{this.state.data?.name}</div>;
  }
}
```

---

## 🎓 Interview Pro Tips

### **Lifecycle Order Questions**
- **Mount:** constructor → getDerivedStateFromProps → render → componentDidMount
- **Update:** getDerivedStateFromProps → shouldComponentUpdate → render → getSnapshotBeforeUpdate → componentDidUpdate  
- **Unmount:** componentWillUnmount

### **Key Concepts to Emphasize**
1. **Cleanup is crucial** - Always return cleanup functions
2. **Dependencies matter** - Incorrect deps = bugs
3. **Performance** - Use React.memo, useMemo, useCallback appropriately
4. **Error Boundaries** - Still need class components
5. **Modern React** - Functional components are standard

### **Advanced Topics (Senior Level)**
- Concurrent features (useTransition, useDeferredValue)
- Suspense for data fetching
- Server Components lifecycle
- Hydration in SSR
- StrictMode double-invocation

---

## 🔥 Rapid Fire Q&A

**Q: Can you use hooks in class components?**  
A: No, hooks only work in functional components

**Q: What replaces componentWillMount?**  
A: Use constructor or useEffect with empty deps

**Q: How many times does useEffect run?**  
A: Depends on dependencies - none = every render, [] = once, [deps] = when deps change

**Q: Can useEffect be async?**  
A: No, but you can call async functions inside it

**Q: What's the cleanup function for?**  
A: Prevent memory leaks - clear timers, unsubscribe, remove listeners

**Q: When does render() method execute?**  
A: On mount and every state/props change (unless prevented by shouldComponentUpdate/React.memo)

---

## 📌 Interview Checklist

✅ **Understand these concepts:**
- [ ] Mounting, Updating, Unmounting phases
- [ ] useEffect dependency array
- [ ] Cleanup functions
- [ ] Common pitfalls (infinite loops, memory leaks)
- [ ] Performance optimization (memo, callbacks)

✅ **Be able to code:**
- [ ] Data fetching in useEffect
- [ ] Cleanup in useEffect
- [ ] ComponentDidMount equivalent
- [ ] ComponentDidUpdate equivalent
- [ ] ComponentWillUnmount equivalent

✅ **Know the differences:**
- [ ] Class vs Functional lifecycle
- [ ] useEffect vs useLayoutEffect
- [ ] componentDidMount vs useEffect([], )
- [ ] When to use each lifecycle method/hook

---

**🎯 Remember:** Modern React = Functional Components + Hooks. Class components are legacy but still important for error boundaries and interview knowledge.