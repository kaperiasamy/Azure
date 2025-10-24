# useCallback in React: Complete Guide

## Table of Contents
1. [What is useCallback?](#what-is-usecallback)
2. [Why Use useCallback?](#why-use-usecallback)
3. [Syntax & Basic Usage](#syntax--basic-usage)
4. [How It Works](#how-it-works)
5. [Common Use Cases](#common-use-cases)
6. [When to Use vs When NOT to Use](#when-to-use-vs-when-not-to-use)
7. [Real-World Examples](#real-world-examples)
8. [Performance Optimization](#performance-optimization)
9. [Common Mistakes](#common-mistakes)
10. [Interview Questions](#interview-questions)

---

## What is useCallback?

`useCallback` is a **React Hook that memoizes a function**, meaning it returns the **same function reference** on every render unless dependencies change.

### Simple Definition:
```
useCallback = "Give me a cached version of this function. 
             Only create a new function if these dependencies change."
```

### Analogy:
Imagine you have a coffee shop. Every morning, you make fresh coffee:
- **Without useCallback**: ❌ Every morning = new coffee (wastes resources)
- **With useCallback**: ✅ Same coffee until ingredients change (saves resources)

---

## Why Use useCallback?

### Problem It Solves:

```jsx
// ❌ WITHOUT useCallback - New function every render
const Parent = () => {
  const [count, setCount] = useState(0);

  // This function is recreated on EVERY render
  const handleClick = () => {
    console.log('Clicked!');
  };

  // Every render = new function reference
  return <Child onClick={handleClick} />;
};

// The Child component
const Child = React.memo(({ onClick }) => {
  console.log('Child rendered'); // Logs every time!
  return <button onClick={onClick}>Click</button>;
});

// Result: Child renders even though nothing changed!
```

### Solution:

```jsx
// ✅ WITH useCallback - Same function reference
import { useCallback } from 'react';

const Parent = () => {
  const [count, setCount] = useState(0);

  // Function reference stays the same across renders
  const handleClick = useCallback(() => {
    console.log('Clicked!');
  }, []);

  // Same function reference = no unnecessary re-renders
  return <Child onClick={handleClick} />;
};

const Child = React.memo(({ onClick }) => {
  console.log('Child rendered'); // Logs only when necessary!
  return <button onClick={onClick}>Click</button>;
});
```

---

## Syntax & Basic Usage

### Basic Syntax:

```jsx
const memoizedCallback = useCallback(
  () => {
    // Your function logic here
    doSomething(a, b);
  },
  [a, b] // Dependencies array
);
```

### Structure Breakdown:

```jsx
useCallback(
  function,           // Function to memoize
  dependencyArray    // When to create a new function
);
```

### Simple Example:

```jsx
import { useCallback, useState } from 'react';

const Counter = () => {
  const [count, setCount] = useState(0);

  // Without useCallback
  const increment = () => setCount(count + 1);

  // With useCallback
  const decrement = useCallback(() => {
    setCount((prevCount) => prevCount - 1);
  }, []); // No dependencies - function never changes

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
      <button onClick={decrement}>Decrement</button>
    </div>
  );
};
```

---

## How It Works

### Step-by-Step Execution:

```jsx
const MyComponent = () => {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('John');

  // ❌ Without useCallback - New function each render
  const oldWay = () => {
    console.log(count);
  };

  // ✅ With useCallback - Same function unless dependencies change
  const newWay = useCallback(() => {
    console.log(count);
  }, [count]); // Dependencies matter!

  return null;
};
```

### Dependency Array Behavior:

```jsx
// 1️⃣ NO DEPENDENCIES - Function never changes
const callback1 = useCallback(() => {}, []);
// Same function reference forever ✅

// 2️⃣ WITH DEPENDENCIES - Function changes if dependencies change
const [id, setId] = useState(1);
const callback2 = useCallback(() => {
  fetchUser(id); // Uses 'id'
}, [id]); // Creates new function when 'id' changes

// 3️⃣ MULTIPLE DEPENDENCIES
const callback3 = useCallback(() => {
  calculateTotal(price, quantity, taxRate);
}, [price, quantity, taxRate]);
// New function if ANY dependency changes
```

### Visual Timeline:

```
Initial Render:
┌─────────────────────────────────┐
│ Create function & store it      │
│ Function Reference: 0x123       │
└─────────────────────────────────┘
           ⬇️
Next Render (dependencies unchanged):
┌─────────────────────────────────┐
│ Return SAME stored function     │
│ Function Reference: 0x123       │
└─────────────────────────────────┘
           ⬇️
Next Render (dependencies changed):
┌─────────────────────────────────┐
│ Create NEW function             │
│ Function Reference: 0x456       │
└─────────────────────────────────┘
```

---

## Common Use Cases

### 1. ✅ Memoized Child Component

**Use when**: Passing function to React.memo child

```jsx
import { useCallback, useState } from 'react';

const Parent = () => {
  const [count, setCount] = useState(0);

  // ✅ USE useCallback here
  const handleButtonClick = useCallback(() => {
    console.log('Button clicked');
  }, []); // No dependencies, function never changes

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Update Parent
      </button>
      {/* Child only re-renders if callback changes */}
      <MemoizedChild onButtonClick={handleButtonClick} />
    </div>
  );
};

// Memoized component - won't re-render if props don't change
const MemoizedChild = React.memo(({ onButtonClick }) => {
  console.log('Child rendered');
  return (
    <button onClick={onButtonClick}>
      Child Button
    </button>
  );
});

export default Parent;
```

### 2. ✅ Dependencies in Callback

**Use when**: Callback depends on state/props

```jsx
import { useCallback, useState } from 'react';

const SearchUsers = () => {
  const [searchTerm, setSearchTerm] = useState('');
  const [results, setResults] = useState([]);

  // ✅ USE useCallback - depends on searchTerm
  const handleSearch = useCallback(() => {
    if (!searchTerm) return;
    
    fetch(`/api/search?q=${searchTerm}`)
      .then(res => res.json())
      .then(data => setResults(data));
  }, [searchTerm]); // Re-create when searchTerm changes

  return (
    <div>
      <input
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        placeholder="Search..."
      />
      <button onClick={handleSearch}>Search</button>
      <ul>
        {results.map(user => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
};
```

### 3. ✅ Event Handlers in Lists

**Use when**: Creating functions in loops

```jsx
import { useCallback, useState } from 'react';

const TodoList = () => {
  const [todos, setTodos] = useState([
    { id: 1, text: 'Learn React' },
    { id: 2, text: 'Master Hooks' },
  ]);

  // ✅ USE useCallback for delete handler
  const handleDelete = useCallback((id) => {
    setTodos(todos => todos.filter(t => t.id !== id));
  }, []); // No dependencies needed (uses functional update)

  return (
    <ul>
      {todos.map(todo => (
        <TodoItem
          key={todo.id}
          todo={todo}
          onDelete={handleDelete}
        />
      ))}
    </ul>
  );
};

const TodoItem = React.memo(({ todo, onDelete }) => {
  return (
    <li>
      {todo.text}
      <button onClick={() => onDelete(todo.id)}>Delete</button>
    </li>
  );
});
```

### 4. ✅ API Call Callback

**Use when**: Callbacks trigger API calls

```jsx
import { useCallback, useState, useEffect } from 'react';

const UserProfile = ({ userId }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);

  // ✅ USE useCallback - depends on userId
  const fetchUser = useCallback(async () => {
    setLoading(true);
    try {
      const response = await fetch(`/api/users/${userId}`);
      const data = await response.json();
      setUser(data);
    } catch (error) {
      console.error('Failed to fetch user:', error);
    } finally {
      setLoading(false);
    }
  }, [userId]); // Re-create when userId changes

  useEffect(() => {
    fetchUser();
  }, [fetchUser]); // Can safely include in useEffect

  if (loading) return <div>Loading...</div>;
  if (!user) return <div>No user found</div>;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
      <button onClick={fetchUser}>Refresh</button>
    </div>
  );
};
```

### 5. ✅ Form Submission

**Use when**: Handling form submissions

```jsx
import { useCallback, useState } from 'react';

const LoginForm = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [loading, setLoading] = useState(false);

  // ✅ USE useCallback - doesn't depend on email/password directly
  const handleSubmit = useCallback(async (e) => {
    e.preventDefault();
    setLoading(true);

    try {
      const response = await fetch('/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password }),
      });

      if (response.ok) {
        const data = await response.json();
        localStorage.setItem('token', data.token);
      }
    } catch (error) {
      console.error('Login failed:', error);
    } finally {
      setLoading(false);
    }
  }, [email, password]); // Include values used in function

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="Password"
      />
      <button type="submit" disabled={loading}>
        {loading ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
};
```

---

## When to Use vs When NOT to Use

### ✅ USE useCallback When:

```jsx
// 1. Passing to memoized child
const Parent = () => {
  const callback = useCallback(() => {}, []);
  return <MemoizedChild callback={callback} />;
};

// 2. Function is a dependency in useEffect
const Component = () => {
  const fetchData = useCallback(async () => {}, [userId]);
  useEffect(() => {
    fetchData();
  }, [fetchData]); // Safe to include
};

// 3. Creating function inside useMemo
const memoized = useMemo(() => {
  const handler = useCallback(() => {}, []);
  return { handler };
}, []);

// 4. Function used as object key
const handlers = {
  [useCallback(() => {}, []).__proto__.constructor.name]: () => {}
};

// 5. Function stored in context
const dispatch = useCallback((action) => {}, []);
```

### ❌ DON'T USE useCallback When:

```jsx
// 1. Simple component, no memoization needed
const SimpleButton = () => {
  // ❌ Unnecessary
  const handleClick = useCallback(() => {
    console.log('Clicked');
  }, []);
  
  return <button onClick={handleClick}>Click</button>;
};

// 2. Not passing to memoized child
const Parent = () => {
  // ❌ Unnecessary - Child isn't memoized
  const callback = useCallback(() => {}, []);
  return <Child callback={callback} />; // Regular component
};

// 3. Simple inline function
<button onClick={() => console.log('click')}>
  {/* ❌ Don't wrap simple functions */}
</button>

// 4. Function doesn't need memoization
const Component = () => {
  // ❌ Overkill for simple operation
  const add = useCallback((a, b) => a + b, []);
  return <div>{add(2, 3)}</div>;
};

// 5. No performance issue exists
const Component = () => {
  // ✅ Just use regular function
  const handleClick = () => {
    console.log('Clicked');
  };
  return <button onClick={handleClick}>Click</button>;
};
```

---

## Real-World Examples

### Example 1: E-commerce Product Filter

```jsx
import { useCallback, useState, useMemo } from 'react';

const ProductFilter = () => {
  const [filters, setFilters] = useState({
    category: '',
    priceRange: [0, 1000],
    inStock: true,
  });
  const [products, setProducts] = useState([]);

  // ✅ Memoized filter function
  const handleFilterChange = useCallback((filterName, value) => {
    setFilters(prev => ({
      ...prev,
      [filterName]: value,
    }));
  }, []);

  // Apply filters
  const filteredProducts = useMemo(() => {
    return products.filter(product =>
      (!filters.category || product.category === filters.category) &&
      product.price >= filters.priceRange[0] &&
      product.price <= filters.priceRange[1] &&
      (filters.inStock ? product.inStock : true)
    );
  }, [products, filters]);

  return (
    <div>
      <FilterPanel onFilterChange={handleFilterChange} />
      <ProductList products={filteredProducts} />
    </div>
  );
};

const FilterPanel = React.memo(({ onFilterChange }) => {
  return (
    <div>
      <label>
        Category:
        <select onChange={(e) => onFilterChange('category', e.target.value)}>
          <option value="">All</option>
          <option value="electronics">Electronics</option>
          <option value="books">Books</option>
        </select>
      </label>
      
      <label>
        In Stock Only:
        <input
          type="checkbox"
          onChange={(e) => onFilterChange('inStock', e.target.checked)}
        />
      </label>
    </div>
  );
});
```

### Example 2: Real-time Chat

```jsx
import { useCallback, useState, useRef } from 'react';

const ChatApp = () => {
  const [messages, setMessages] = useState([]);
  const [input, setInput] = useState('');
  const [isLoading, setIsLoading] = useState(false);
  const socketRef = useRef(null);

  // ✅ Memoized send message function
  const handleSendMessage = useCallback(async () => {
    if (!input.trim()) return;

    const message = {
      id: Date.now(),
      text: input,
      sender: 'user',
      timestamp: new Date(),
    };

    // Optimistic update
    setMessages(prev => [...prev, message]);
    setInput('');
    setIsLoading(true);

    try {
      // Send to server/socket
      await fetch('/api/messages', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(message),
      });
    } catch (error) {
      console.error('Failed to send message:', error);
      setMessages(prev => prev.filter(m => m.id !== message.id));
    } finally {
      setIsLoading(false);
    }
  }, [input]); // Only depends on input

  const handleKeyPress = useCallback((e) => {
    if (e.key === 'Enter' && !e.shiftKey) {
      e.preventDefault();
      handleSendMessage();
    }
  }, [handleSendMessage]);

  return (
    <div>
      <MessageList messages={messages} />
      <div className="input-area">
        <textarea
          value={input}
          onChange={(e) => setInput(e.target.value)}
          onKeyPress={handleKeyPress}
          placeholder="Type a message..."
        />
        <button onClick={handleSendMessage} disabled={isLoading}>
          Send
        </button>
      </div>
    </div>
  );
};

const MessageList = React.memo(({ messages }) => {
  return (
    <div className="messages">
      {messages.map(msg => (
        <Message key={msg.id} message={msg} />
      ))}
    </div>
  );
});
```

### Example 3: Data Table with Sorting

```jsx
import { useCallback, useState, useMemo } from 'react';

const DataTable = ({ data }) => {
  const [sortConfig, setSortConfig] = useState({
    key: null,
    direction: 'asc',
  });

  // ✅ Memoized sort handler
  const handleSort = useCallback((key) => {
    setSortConfig(prevConfig => {
      const newDirection =
        prevConfig.key === key && prevConfig.direction === 'asc'
          ? 'desc'
          : 'asc';

      return {
        key,
        direction: newDirection,
      };
    });
  }, []);

  // Sort data
  const sortedData = useMemo(() => {
    if (!sortConfig.key) return data;

    return [...data].sort((a, b) => {
      const aValue = a[sortConfig.key];
      const bValue = b[sortConfig.key];

      if (aValue < bValue) {
        return sortConfig.direction === 'asc' ? -1 : 1;
      }
      if (aValue > bValue) {
        return sortConfig.direction === 'asc' ? 1 : -1;
      }
      return 0;
    });
  }, [data, sortConfig]);

  return (
    <table>
      <thead>
        <tr>
          <th onClick={() => handleSort('name')}>Name</th>
          <th onClick={() => handleSort('email')}>Email</th>
          <th onClick={() => handleSort('role')}>Role</th>
        </tr>
      </thead>
      <tbody>
        {sortedData.map(row => (
          <tr key={row.id}>
            <td>{row.name}</td>
            <td>{row.email}</td>
            <td>{row.role}</td>
          </tr>
        ))}
      </tbody>
    </table>
  );
};
```

### Example 4: Debounced Search

```jsx
import { useCallback, useState, useRef } from 'react';

const SearchUsers = () => {
  const [searchTerm, setSearchTerm] = useState('');
  const [results, setResults] = useState([]);
  const [isLoading, setIsLoading] = useState(false);
  const debounceTimerRef = useRef(null);

  // ✅ Memoized search function
  const handleSearch = useCallback(async (query) => {
    if (!query.trim()) {
      setResults([]);
      return;
    }

    setIsLoading(true);

    try {
      const response = await fetch(`/api/users/search?q=${query}`);
      const data = await response.json();
      setResults(data);
    } catch (error) {
      console.error('Search failed:', error);
    } finally {
      setIsLoading(false);
    }
  }, []);

  // ✅ Memoized debounced handler
  const handleSearchInput = useCallback((value) => {
    setSearchTerm(value);

    // Clear previous timer
    if (debounceTimerRef.current) {
      clearTimeout(debounceTimerRef.current);
    }

    // Set new timer
    debounceTimerRef.current = setTimeout(() => {
      handleSearch(value);
    }, 300);
  }, [handleSearch]);

  return (
    <div>
      <input
        value={searchTerm}
        onChange={(e) => handleSearchInput(e.target.value)}
        placeholder="Search users..."
      />

      {isLoading && <p>Loading...</p>}

      <ul>
        {results.map(user => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
};
```

---

## Performance Optimization

### Measuring Performance Impact

```jsx
import { useCallback, useState, useRef } from 'react';

const PerformanceDemo = () => {
  const [count, setCount] = useState(0);
  const renderCountRef = useRef(0);

  // Without useCallback
  const withoutMemo = () => {
    console.log('Without memo');
  };

  // With useCallback
  const withMemo = useCallback(() => {
    console.log('With memo');
  }, []);

  renderCountRef.current++;

  return (
    <div>
      <p>Render count: {renderCountRef.current}</p>
      <button onClick={() => setCount(count + 1)}>
        Update Count ({count})
      </button>
      <ChildWithoutMemo callback={withoutMemo} />
      <ChildWithMemo callback={withMemo} />
    </div>
  );
};

const ChildWithoutMemo = ({ callback }) => {
  const renderCount = useRef(0);
  renderCount.current++;

  return (
    <div>
      <h3>Without Memo</h3>
      <p>Render count: {renderCount.current}</p>
      <button onClick={callback}>Call Function</button>
    </div>
  );
};

const ChildWithMemo = React.memo(({ callback }) => {
  const renderCount = useRef(0);
  renderCount.current++;

  return (
    <div>
      <h3>With Memo</h3>
      <p>Render count: {renderCount.current}</p>
      <button onClick={callback}>Call Function</button>
    </div>
  );
});
```

### Performance Checklist

```jsx
// ✅ Good - Minimal re-renders
const GoodExample = () => {
  const [data, setData] = useState(null);

  const fetchData = useCallback(async () => {
    const response = await fetch('/api/data');
    setData(await response.json());
  }, []); // No dependencies

  return <Child onFetch={fetchData} />;
};

// ⚠️ Warning - Creates new function on every dependency change
const WarningExample = () => {
  const [id, setId] = useState(1);
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');

  const fetchUser = useCallback(async () => {
    // Only depends on 'id', not 'name' or 'email'
    const response = await fetch(`/api/users/${id}`);
    const user = await response.json();
    setName(user.name);
    setEmail(user.email);
  }, [id]); // ✅ Correct - only include what's needed

  // NOT like this: [id, name, email] ❌
};

// ❌ Bad - Creates new function every render
const BadExample = () => {
  const [value, setValue] = useState('');

  const handleChange = useCallback((e) => {
    setValue(e.target.value);
  }, [value]); // ❌ Depends on value - creates new function constantly

  return <input onChange={handleChange} />;
};
```

---

## Common Mistakes

### ❌ Mistake 1: Missing Dependencies

```jsx
// ❌ BAD - userId not in dependencies
const fetchUser = useCallback(async () => {
  const response = await fetch(`/api/users/${userId}`);
  // userId changes but function doesn't!
}, []);

// ✅ GOOD - Include userId
const fetchUser = useCallback(async () => {
  const response = await fetch(`/api/users/${userId}`);
}, [userId]); // Function updates when userId changes
```

### ❌ Mistake 2: Unnecessary useCallback

```jsx
// ❌ BAD - Don't memoize simple operations
const increment = useCallback(() => {
  setCount(count + 1);
}, [count]);

// ✅ GOOD - Use functional update instead
const increment = useCallback(() => {
  setCount(prev => prev + 1);
}, []); // No dependencies needed!
```

### ❌ Mistake 3: Forgetting useCallback with useEffect

```jsx
// ❌ BAD - useEffect reruns constantly
const Component = () => {
  const handleFetch = () => {
    fetch('/api/data');
  };

  useEffect(() => {
    handleFetch();
  }, [handleFetch]); // handleFetch recreated every render!
};

// ✅ GOOD - Memoize the callback
const Component = () => {
  const handleFetch = useCallback(() => {
    fetch('/api/data');
  }, []);

  useEffect(() => {
    handleFetch();
  }, [handleFetch]); // Now stable
};
```

### ❌ Mistake 4: Over-memoization

```jsx
// ❌ BAD - Memoizing everything
const Component = () => {
  const add = useCallback((a, b) => a + b, []);
  const handleClick = useCallback(() => {}, []);
  const data = useMemo(() => ({ a: 1 }), []);

  return <button onClick={handleClick}>{add(1, 2)}</button>;
};

// ✅ GOOD - Only memoize when necessary
const Component = () => {
  const handleClick = useCallback(() => {}, []); // Passed to child
  return <button onClick={handleClick}>Click</button>;
};
```

### ❌ Mistake 5: Creating New Objects in Dependencies

```jsx
// ❌ BAD - config object recreated every render
const config = { timeout: 5000, retries: 3 };
const callback = useCallback(() => {
  fetchWithConfig(config);
}, [config]); // config changes every render!

// ✅ GOOD - Keep config outside or memoize it
const config = useMemo(() => ({ timeout: 5000, retries: 3 }), []);
const callback = useCallback(() => {
  fetchWithConfig(config);
}, [config]);

// ✅ BETTER - Keep static config outside component
const CONFIG = { timeout: 5000, retries: 3 };
const callback = useCallback(() => {
  fetchWithConfig(CONFIG);
}, []);
```

---

## Interview Questions

### Q1: What is useCallback and why would you use it?

**Answer:**
```
useCallback is a React Hook that memoizes a function, returning the same 
function reference across renders unless its dependencies change.

Why use it:
1. Prevent unnecessary re-renders of memoized child components
2. Avoid creating new function instances every render
3. Stable function reference for useEffect dependencies
4. Performance optimization when passing callbacks to children
```

### Q2: What's the difference between useCallback and useMemo?

**Answer:**
```
useCallback:
- Memoizes a FUNCTION
- Syntax: useCallback(() => {}, [deps])
- Returns a function reference

useMemo:
- Memoizes a VALUE
- Syntax: useMemo(() => value, [deps])
- Returns the computed value

Example:
const memoFunc = useCallback(() => console.log('Hi'), []);
const memoValue = useMemo(() => 1 + 1, []);
```

### Q3: How do you know when to use useCallback?

**Answer:**
```
Use useCallback when:
1. Function is passed to a memoized child component
2. Function is a dependency in useEffect
3. Function is used as a key in objects/maps
4. Performance profiling shows re-renders

Don't use when:
1. Component isn't memoized (React.memo)
2. Function isn't expensive to create
3. No performance issues exist
4. Function is only used locally
```

### Q4: What happens if you omit a dependency in useCallback?

**Answer:**
```
The function won't have access to updated values, causing stale closures.

Example:
const [count, setCount] = useState(0);

❌ BAD:
const callback = useCallback(() => {
  console.log(count); // Always logs 0!
}, []); // count not in dependencies

✅ GOOD:
const callback = useCallback(() => {
  console.log(count); // Logs current count
}, [count]); // count in dependencies
```

### Q5: Can useCallback replace useState?

**Answer:**
```
No, they serve different purposes:

useState:
- Manages state values
- Triggers re-render on change
- Returns [value, setValue]

useCallback:
- Memoizes functions
- Doesn't trigger re-render
- Returns a stable function reference

They often work together:
const [count, setCount] = useState(0);
const handleClick = useCallback(() => {
  setCount(c => c + 1);
}, []);
```

### Q6: Is useCallback always better than creating a function inline?

**Answer:**
```
No! useCallback adds overhead. Only use when:
1. Function is passed to React.memo child
2. Function is useEffect dependency
3. Performance profiling shows improvement

Simple components:
❌ const handler = useCallback(() => {}, []); // Overkill
✅ const handler = () => {}; // Just use inline

Trade-off:
- useCallback: Creates closure + memoization cost
- Inline: Creates new function every render
```

---

## useCallback vs Inline Function Comparison

```jsx
// Scenario: Simple button click

// ❌ Option 1: useCallback (Unnecessary overhead)
const Component1 = () => {
  const handleClick = useCallback(() => {
    console.log('Clicked');
  }, []);

  return <button onClick={handleClick}>Click</button>;
};

// ✅ Option 2: Inline function (Simple and clear)
const Component2 = () => {
  return (
    <button onClick={() => console.log('Clicked')}>
      Click
    </button>
  );
};

// Scenario: Pass to memoized child

// ✅ Option 1: useCallback (Prevents re-renders)
const Parent1 = () => {
  const handleClick = useCallback(() => {
    console.log('Clicked');
  }, []);

  return <MemoizedChild onClick={handleClick} />;
};

// ❌ Option 2: Inline function (Defeats memoization)
const Parent2 = () => {
  return (
    <MemoizedChild onClick={() => console.log('Clicked')} />
  );
};
```

---

## Summary

### Key Takeaways:

| Concept | Details |
|---------|---------|
| **What** | Memoizes a function reference |
| **Why** | Prevents unnecessary re-renders |
| **When** | Use with React.memo or useEffect |
| **How** | `useCallback(fn, [deps])` |
| **Cost** | Memoization overhead + closure |
| **Benefit** | Stable function reference |

### Quick Checklist:

- [ ] Understand useCallback memoizes functions
- [ ] Know when to include in dependencies
- [ ] Don't over-memoize
- [ ] Include all external values in dependencies
- [ ] Use with React.memo for maximum benefit
- [ ] Measure performance before optimizing
- [ ] Consider useMemo for computed values
- [ ] Understand closure implications

### Best Practice:

```jsx
// ✅ Ideal useCallback usage pattern
const Component = ({ userId }) => {
  // Memoize function that depends on prop
  const fetchUser = useCallback(async () => {
    const response = await fetch(`/api/users/${userId}`);
    return response.json();
  }, [userId]); // Update when userId changes

  useEffect(() => {
    fetchUser();
  }, [fetchUser]); // Safe to include in useEffect

  return (
    <div>
      <button onClick={fetchUser}>Refresh</button>
      {/* Pass to memoized child */}
      <MemoizedUserData onFetch={fetchUser} />
    </div>
  );
};
```

Remember: **Optimize when needed, not by default!** 🚀