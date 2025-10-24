# Debounce in React - Complete Guide

## 🎯 What is Debounce?

**Debounce** is a programming technique that delays the execution of a function until after a specified period of inactivity. It ensures that a function is only called once after the user has stopped triggering it for a predetermined amount of time.

### **Simple Analogy**
Think of an elevator door - it waits a few seconds after the last person enters before closing. If someone else approaches, the timer resets. That's debouncing!

```javascript
// Without Debounce: Function executes on every keystroke
"H" → API call
"He" → API call  
"Hel" → API call
"Hell" → API call
"Hello" → API call
// 5 API calls for typing "Hello"

// With Debounce (500ms delay): Function executes once after user stops
"H" → wait...
"He" → wait reset...
"Hel" → wait reset...
"Hell" → wait reset...
"Hello" → wait 500ms → API call
// Only 1 API call!
```

---

## 📊 Debounce vs Throttle vs Normal

| **Technique** | **Execution Pattern** | **Use Case** |
|--------------|---------------------|-------------|
| **Normal** | Every single event | Simple counters, immediate updates |
| **Debounce** | After delay from last event | Search inputs, form validation |
| **Throttle** | Maximum once per interval | Scroll events, resize handlers |

---

## 💻 Implementation Examples

### **1. Basic Debounce Implementation**

```javascript
// Vanilla JavaScript Debounce Function
function debounce(func, delay) {
  let timeoutId;
  
  return function(...args) {
    // Clear previous timeout if exists
    clearTimeout(timeoutId);
    
    // Set new timeout
    timeoutId = setTimeout(() => {
      func.apply(this, args);
    }, delay);
  };
}

// Usage
const debouncedSearch = debounce((searchTerm) => {
  console.log('Searching for:', searchTerm);
}, 500);
```

### **2. React Implementation - Search Example**

```jsx
import React, { useState, useCallback, useEffect } from 'react';

// ❌ WRONG: Without Debounce - Too many API calls
const SearchWithoutDebounce = () => {
  const [searchTerm, setSearchTerm] = useState('');
  const [results, setResults] = useState([]);
  
  // This fires on EVERY keystroke - BAD!
  const handleSearch = async (value) => {
    const response = await fetch(`/api/search?q=${value}`);
    const data = await response.json();
    setResults(data);
  };
  
  return (
    <input
      type="text"
      onChange={(e) => {
        setSearchTerm(e.target.value);
        handleSearch(e.target.value); // API call on every keystroke!
      }}
      placeholder="Search..."
    />
  );
};

// ✅ CORRECT: With Debounce - Optimized API calls
const SearchWithDebounce = () => {
  const [searchTerm, setSearchTerm] = useState('');
  const [debouncedTerm, setDebouncedTerm] = useState('');
  const [results, setResults] = useState([]);
  const [isSearching, setIsSearching] = useState(false);
  
  // Debounce the search term
  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedTerm(searchTerm);
    }, 500); // 500ms delay
    
    return () => {
      clearTimeout(timer);
    };
  }, [searchTerm]);
  
  // API call only when debouncedTerm changes
  useEffect(() => {
    if (debouncedTerm) {
      setIsSearching(true);
      
      fetch(`/api/search?q=${debouncedTerm}`)
        .then(res => res.json())
        .then(data => {
          setResults(data);
          setIsSearching(false);
        });
    } else {
      setResults([]);
    }
  }, [debouncedTerm]);
  
  return (
    <div>
      <input
        type="text"
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        placeholder="Search products..."
      />
      
      {isSearching && <div>Searching...</div>}
      
      <div className="results">
        {results.map(item => (
          <div key={item.id}>{item.name}</div>
        ))}
      </div>
    </div>
  );
};
```

### **3. Custom useDebounce Hook**

```jsx
// Custom Hook for Reusability
const useDebounce = (value, delay) => {
  const [debouncedValue, setDebouncedValue] = useState(value);
  
  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    
    return () => {
      clearTimeout(timer);
    };
  }, [value, delay]);
  
  return debouncedValue;
};

// Usage in Component
const SearchComponent = () => {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearchTerm = useDebounce(searchTerm, 500);
  const [results, setResults] = useState([]);
  
  // API call with debounced value
  useEffect(() => {
    if (debouncedSearchTerm) {
      searchAPI(debouncedSearchTerm).then(setResults);
    }
  }, [debouncedSearchTerm]);
  
  return (
    <input
      value={searchTerm}
      onChange={(e) => setSearchTerm(e.target.value)}
      placeholder="Type to search..."
    />
  );
};
```

### **4. Advanced Debounce with useCallback**

```jsx
const AdvancedDebounceExample = () => {
  const [searchTerm, setSearchTerm] = useState('');
  const [results, setResults] = useState([]);
  
  // Create debounced function with useCallback
  const debounce = (func, delay) => {
    let timeoutId;
    return (...args) => {
      clearTimeout(timeoutId);
      timeoutId = setTimeout(() => func(...args), delay);
    };
  };
  
  // Memoized debounced search function
  const debouncedSearch = useCallback(
    debounce((term) => {
      if (term) {
        fetch(`/api/search?q=${term}`)
          .then(res => res.json())
          .then(setResults);
      }
    }, 500),
    []
  );
  
  const handleInputChange = (e) => {
    const value = e.target.value;
    setSearchTerm(value);
    debouncedSearch(value);
  };
  
  return (
    <div>
      <input
        type="text"
        value={searchTerm}
        onChange={handleInputChange}
        placeholder="Search..."
      />
      
      <ul>
        {results.map(item => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    </div>
  );
};
```

### **5. Debounce with Cancel Capability**

```jsx
const useDebounceWithCancel = (value, delay) => {
  const [debouncedValue, setDebouncedValue] = useState(value);
  const timeoutRef = useRef(null);
  
  const cancel = useCallback(() => {
    if (timeoutRef.current) {
      clearTimeout(timeoutRef.current);
    }
  }, []);
  
  useEffect(() => {
    timeoutRef.current = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    
    return () => {
      clearTimeout(timeoutRef.current);
    };
  }, [value, delay]);
  
  return [debouncedValue, cancel];
};

// Usage
const CancellableSearch = () => {
  const [search, setSearch] = useState('');
  const [debouncedSearch, cancelDebounce] = useDebounceWithCancel(search, 1000);
  
  return (
    <div>
      <input
        value={search}
        onChange={(e) => setSearch(e.target.value)}
      />
      <button onClick={cancelDebounce}>Cancel Search</button>
      <p>Will search for: {debouncedSearch}</p>
    </div>
  );
};
```

---

## 🎯 Real-World Use Cases

### **1. Autocomplete/Search Input**
```jsx
const AutocompleteSearch = () => {
  const [query, setQuery] = useState('');
  const [suggestions, setSuggestions] = useState([]);
  const debouncedQuery = useDebounce(query, 300);
  
  useEffect(() => {
    if (debouncedQuery.length > 2) {
      fetchSuggestions(debouncedQuery).then(setSuggestions);
    } else {
      setSuggestions([]);
    }
  }, [debouncedQuery]);
  
  return (
    <div className="autocomplete">
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search users..."
      />
      
      {suggestions.length > 0 && (
        <ul className="suggestions">
          {suggestions.map(item => (
            <li key={item.id} onClick={() => setQuery(item.name)}>
              {item.name}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
};
```

### **2. Form Validation**
```jsx
const FormWithDebounceValidation = () => {
  const [email, setEmail] = useState('');
  const [emailError, setEmailError] = useState('');
  const debouncedEmail = useDebounce(email, 500);
  
  // Validate email after user stops typing
  useEffect(() => {
    if (debouncedEmail) {
      if (!debouncedEmail.includes('@')) {
        setEmailError('Invalid email format');
      } else {
        // Check if email exists in database
        checkEmailExists(debouncedEmail).then(exists => {
          setEmailError(exists ? 'Email already taken' : '');
        });
      }
    }
  }, [debouncedEmail]);
  
  return (
    <div>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Enter email"
      />
      {emailError && <span className="error">{emailError}</span>}
    </div>
  );
};
```

### **3. Window Resize Handler**
```jsx
const ResponsiveComponent = () => {
  const [windowSize, setWindowSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });
  
  useEffect(() => {
    const debouncedResize = debounce(() => {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    }, 250);
    
    window.addEventListener('resize', debouncedResize);
    
    return () => {
      window.removeEventListener('resize', debouncedResize);
    };
  }, []);
  
  return (
    <div>
      Window: {windowSize.width} x {windowSize.height}
    </div>
  );
};
```

### **4. Save Draft Automatically**
```jsx
const AutoSaveEditor = () => {
  const [content, setContent] = useState('');
  const [saveStatus, setSaveStatus] = useState('');
  const debouncedContent = useDebounce(content, 2000);
  
  useEffect(() => {
    if (debouncedContent) {
      setSaveStatus('Saving...');
      
      saveDraft(debouncedContent)
        .then(() => setSaveStatus('Saved'))
        .catch(() => setSaveStatus('Error saving'));
    }
  }, [debouncedContent]);
  
  return (
    <div>
      <textarea
        value={content}
        onChange={(e) => setContent(e.target.value)}
        placeholder="Start typing... (auto-saves every 2 seconds)"
      />
      <div className="status">{saveStatus}</div>
    </div>
  );
};
```

---

## 🚀 Using Lodash Debounce

```jsx
import { debounce } from 'lodash';
// or
import debounce from 'lodash/debounce';

const SearchWithLodash = () => {
  const [results, setResults] = useState([]);
  
  // Create debounced function
  const debouncedSearch = useCallback(
    debounce((searchTerm) => {
      fetch(`/api/search?q=${searchTerm}`)
        .then(res => res.json())
        .then(setResults);
    }, 500),
    []
  );
  
  return (
    <input
      type="text"
      onChange={(e) => debouncedSearch(e.target.value)}
      placeholder="Search..."
    />
  );
};

// With cancel and flush methods
const AdvancedLodashDebounce = () => {
  const debouncedFn = useRef(
    debounce((value) => {
      console.log('Debounced value:', value);
    }, 1000)
  ).current;
  
  return (
    <div>
      <input onChange={(e) => debouncedFn(e.target.value)} />
      <button onClick={() => debouncedFn.cancel()}>Cancel</button>
      <button onClick={() => debouncedFn.flush()}>Execute Now</button>
    </div>
  );
};
```

---

## ⚡ Performance Comparison

```jsx
const PerformanceComparison = () => {
  const [normalCount, setNormalCount] = useState(0);
  const [debouncedCount, setDebouncedCount] = useState(0);
  const [input, setInput] = useState('');
  
  // Without debounce - fires on every change
  const handleNormalChange = (value) => {
    setNormalCount(prev => prev + 1);
    // Imagine this is an API call
  };
  
  // With debounce - fires after delay
  const debouncedChange = useCallback(
    debounce((value) => {
      setDebouncedCount(prev => prev + 1);
      // API call only after user stops typing
    }, 500),
    []
  );
  
  const handleInputChange = (e) => {
    const value = e.target.value;
    setInput(value);
    handleNormalChange(value);
    debouncedChange(value);
  };
  
  return (
    <div>
      <input
        value={input}
        onChange={handleInputChange}
        placeholder="Type quickly..."
      />
      <div>Normal calls: {normalCount}</div>
      <div>Debounced calls: {debouncedCount}</div>
      <div>Savings: {normalCount - debouncedCount} API calls prevented!</div>
    </div>
  );
};
```

---

## 🎯 Interview Questions & Answers

### **Q1: What is debouncing and why use it?**
**Answer:** Debouncing delays function execution until after a period of inactivity. It's used to:
- Reduce API calls in search inputs
- Prevent excessive computations
- Improve performance
- Save bandwidth and server resources

### **Q2: Difference between debounce and throttle?**
```javascript
// Debounce: Executes after delay from LAST event
// User types: "H" -> "e" -> "l" -> "l" -> "o" [stops] -> Execute once

// Throttle: Executes at most once per interval
// Scrolling: Execute -> wait 100ms -> Execute -> wait 100ms -> Execute
```

### **Q3: How to implement debounce in React?**
```jsx
// Best approach: Custom hook
const useDebounce = (value, delay) => {
  const [debouncedValue, setDebouncedValue] = useState(value);
  
  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);
  
  return debouncedValue;
};
```

### **Q4: Common pitfalls with debounce in React?**
```jsx
// ❌ WRONG: Creating new debounced function on every render
const BadExample = () => {
  const debouncedFn = debounce(() => {}, 500); // New function each render!
};

// ✅ CORRECT: Using useCallback or useRef
const GoodExample = () => {
  const debouncedFn = useCallback(
    debounce(() => {}, 500),
    []
  );
};
```

---

## 📝 Best Practices

### **DO's:**
✅ Use debounce for search inputs and autocomplete  
✅ Set appropriate delay (300-500ms for typing)  
✅ Clean up timers in useEffect return  
✅ Use useCallback to prevent recreation  
✅ Consider UX - show loading states  

### **DON'Ts:**
❌ Don't debounce critical actions (save, submit)  
❌ Don't use too long delays (frustrates users)  
❌ Don't forget to handle cleanup  
❌ Don't recreate debounced functions unnecessarily  

---

## 🔥 Quick Implementation Cheatsheet

```jsx
// 1. Simple Debounce Hook
const useDebounce = (value, delay) => {
  const [debounced, setDebounced] = useState(value);
  useEffect(() => {
    const timer = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);
  return debounced;
};

// 2. Usage
const Component = () => {
  const [search, setSearch] = useState('');
  const debouncedSearch = useDebounce(search, 500);
  
  useEffect(() => {
    if (debouncedSearch) {
      // API call here
    }
  }, [debouncedSearch]);
  
  return <input onChange={(e) => setSearch(e.target.value)} />;
};
```

---

## 📊 When to Use Debounce

| **Use Case** | **Delay (ms)** | **Why** |
|-------------|---------------|---------|
| Search input | 300-500 | Wait for user to finish typing |
| Form validation | 500-1000 | Validate after user stops |
| Window resize | 150-250 | Prevent excessive recalculation |
| Auto-save | 2000-5000 | Save periodically, not constantly |
| API requests | 300-800 | Reduce server load |

---

**🎯 Remember for Interviews:**
- Debounce = Delay execution until inactivity
- Prevents excessive function calls
- Essential for search inputs and API optimization
- Always cleanup timers in React
- Use custom hooks for reusability
