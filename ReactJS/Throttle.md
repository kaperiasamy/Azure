# Throttling in React - Complete Guide

## 🎯 What is Throttling?

**Throttling** is a technique that limits the execution of a function to once every specified time interval, regardless of how many times the event is triggered. It ensures a function runs at a consistent, controlled rate.

### **Simple Analogy**
Think of a water tap with a flow regulator - no matter how much you open it, water flows at a maximum fixed rate. That's throttling!

```javascript
// Without Throttle: Function executes on every event
Scroll → Execute
Scroll → Execute (1ms later)
Scroll → Execute (2ms later)
Scroll → Execute (3ms later)
// Hundreds of executions while scrolling!

// With Throttle (100ms): Function executes at most once per 100ms
Scroll → Execute
Scroll → Skip (1ms later)
Scroll → Skip (50ms later)
Scroll → Skip (99ms later)
Scroll → Execute (100ms later)
// Controlled execution rate!
```

---

## 📊 Throttle vs Debounce vs Normal - Visual Comparison

```javascript
// User types "Hello" quickly

// NORMAL EXECUTION:
H → execute()
e → execute()
l → execute()
l → execute()
o → execute()
// 5 executions

// DEBOUNCE (waits for pause):
H-e-l-l-o → [wait 500ms] → execute()
// 1 execution after user stops

// THROTTLE (executes at intervals):
H → execute()
e → skip
l → skip
l → execute() [100ms passed]
o → skip
// 2 executions at fixed intervals
```

| **Technique** | **Execution Pattern** | **Best Use Case** | **Example** |
|--------------|---------------------|------------------|-----------|
| **Normal** | Every single event | Instant feedback | Button clicks |
| **Debounce** | After delay from last event | Wait for user to finish | Search input |
| **Throttle** | Maximum once per interval | Consistent rate limiting | Scroll events |

---

## 💻 Implementation Examples

### **1. Basic Throttle Implementation**

```javascript
// Vanilla JavaScript Throttle Function
function throttle(func, limit) {
  let inThrottle;
  
  return function(...args) {
    if (!inThrottle) {
      func.apply(this, args);
      inThrottle = true;
      
      setTimeout(() => {
        inThrottle = false;
      }, limit);
    }
  };
}

// Advanced throttle with trailing call
function throttleWithTrailing(func, limit) {
  let inThrottle;
  let lastArgs;
  
  return function(...args) {
    lastArgs = args;
    
    if (!inThrottle) {
      func.apply(this, lastArgs);
      inThrottle = true;
      
      setTimeout(() => {
        inThrottle = false;
        if (lastArgs) {
          func.apply(this, lastArgs);
          lastArgs = null;
          inThrottle = true;
          setTimeout(() => inThrottle = false, limit);
        }
      }, limit);
    }
  };
}
```

### **2. React Implementation - Scroll Example**

```jsx
import React, { useState, useEffect, useCallback, useRef } from 'react';

// ❌ WRONG: Without Throttle - Performance Issue
const ScrollWithoutThrottle = () => {
  const [scrollY, setScrollY] = useState(0);
  const [updateCount, setUpdateCount] = useState(0);
  
  useEffect(() => {
    // This fires on EVERY scroll event - BAD for performance!
    const handleScroll = () => {
      setScrollY(window.scrollY);
      setUpdateCount(prev => prev + 1);
      console.log('Scroll event fired!'); // Logs hundreds of times
    };
    
    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll);
  }, []);
  
  return (
    <div>
      <p>Scroll Position: {scrollY}</p>
      <p>Updates: {updateCount} (Too many!)</p>
    </div>
  );
};

// ✅ CORRECT: With Throttle - Optimized Performance
const ScrollWithThrottle = () => {
  const [scrollY, setScrollY] = useState(0);
  const [updateCount, setUpdateCount] = useState(0);
  
  // Throttle implementation
  const throttle = (func, delay) => {
    let timeoutId;
    let lastExecTime = 0;
    
    return function (...args) {
      const currentTime = Date.now();
      
      if (currentTime - lastExecTime > delay) {
        func.apply(this, args);
        lastExecTime = currentTime;
      }
    };
  };
  
  useEffect(() => {
    const handleScroll = throttle(() => {
      setScrollY(window.scrollY);
      setUpdateCount(prev => prev + 1);
      console.log('Throttled scroll update');
    }, 100); // Update at most once per 100ms
    
    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll);
  }, []);
  
  return (
    <div style={{ height: '200vh' }}>
      <div style={{ position: 'fixed', top: 0, background: 'white', padding: '10px' }}>
        <p>Scroll Position: {scrollY}</p>
        <p>Updates: {updateCount} (Controlled!)</p>
      </div>
      <p>Scroll down to see throttling in action...</p>
    </div>
  );
};
```

### **3. Custom useThrottle Hook**

```jsx
// Custom Hook for Throttling
const useThrottle = (value, limit) => {
  const [throttledValue, setThrottledValue] = useState(value);
  const lastRan = useRef(Date.now());
  
  useEffect(() => {
    const handler = setTimeout(() => {
      if (Date.now() - lastRan.current >= limit) {
        setThrottledValue(value);
        lastRan.current = Date.now();
      }
    }, limit - (Date.now() - lastRan.current));
    
    return () => {
      clearTimeout(handler);
    };
  }, [value, limit]);
  
  return throttledValue;
};

// Alternative implementation with useCallback
const useThrottleCallback = (callback, delay) => {
  const lastRun = useRef(Date.now());
  
  return useCallback((...args) => {
    if (Date.now() - lastRun.current >= delay) {
      callback(...args);
      lastRun.current = Date.now();
    }
  }, [callback, delay]);
};

// Usage Example
const SearchWithThrottle = () => {
  const [inputValue, setInputValue] = useState('');
  const throttledValue = useThrottle(inputValue, 500);
  const [results, setResults] = useState([]);
  
  // API call with throttled value
  useEffect(() => {
    if (throttledValue) {
      console.log('Searching for:', throttledValue);
      searchAPI(throttledValue).then(setResults);
    }
  }, [throttledValue]);
  
  return (
    <div>
      <input
        value={inputValue}
        onChange={(e) => setInputValue(e.target.value)}
        placeholder="Type to search (throttled)..."
      />
      <p>Throttled value: {throttledValue}</p>
    </div>
  );
};
```

### **4. Advanced Throttle with Leading and Trailing Options**

```jsx
const useAdvancedThrottle = (value, delay, options = {}) => {
  const { leading = true, trailing = true } = options;
  const [throttledValue, setThrottledValue] = useState(value);
  const lastValue = useRef(value);
  const lastRun = useRef(0);
  const trailingTimeout = useRef(null);
  
  useEffect(() => {
    lastValue.current = value;
    const now = Date.now();
    
    const run = () => {
      lastRun.current = now;
      setThrottledValue(lastValue.current);
    };
    
    if (leading && now - lastRun.current >= delay) {
      run();
    } else if (trailing) {
      clearTimeout(trailingTimeout.current);
      
      trailingTimeout.current = setTimeout(() => {
        if (lastValue.current !== throttledValue) {
          run();
        }
      }, delay - (now - lastRun.current));
    }
    
    return () => {
      clearTimeout(trailingTimeout.current);
    };
  }, [value, delay, leading, trailing, throttledValue]);
  
  return throttledValue;
};

// Usage with options
const Component = () => {
  const [value, setValue] = useState('');
  
  // Execute on leading edge only
  const leadingOnly = useAdvancedThrottle(value, 1000, { 
    leading: true, 
    trailing: false 
  });
  
  // Execute on trailing edge only
  const trailingOnly = useAdvancedThrottle(value, 1000, { 
    leading: false, 
    trailing: true 
  });
  
  // Execute on both (default)
  const both = useAdvancedThrottle(value, 1000);
  
  return (
    <div>
      <input onChange={(e) => setValue(e.target.value)} />
      <p>Leading only: {leadingOnly}</p>
      <p>Trailing only: {trailingOnly}</p>
      <p>Both: {both}</p>
    </div>
  );
};
```

---

## 🎯 Real-World Use Cases

### **1. Infinite Scroll Implementation**

```jsx
const InfiniteScroll = () => {
  const [items, setItems] = useState([]);
  const [page, setPage] = useState(1);
  const [loading, setLoading] = useState(false);
  
  // Throttled scroll handler
  const handleScroll = useCallback(
    throttle(() => {
      const { scrollTop, scrollHeight, clientHeight } = document.documentElement;
      
      // Check if user scrolled to bottom
      if (scrollTop + clientHeight >= scrollHeight - 100 && !loading) {
        loadMore();
      }
    }, 200),
    [loading]
  );
  
  const loadMore = async () => {
    setLoading(true);
    const newItems = await fetchItems(page);
    setItems(prev => [...prev, ...newItems]);
    setPage(prev => prev + 1);
    setLoading(false);
  };
  
  useEffect(() => {
    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll);
  }, [handleScroll]);
  
  return (
    <div>
      {items.map(item => (
        <div key={item.id} className="item">
          {item.content}
        </div>
      ))}
      {loading && <div>Loading more...</div>}
    </div>
  );
};
```

### **2. Window Resize Handler**

```jsx
const ResponsiveLayout = () => {
  const [dimensions, setDimensions] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });
  const [resizeCount, setResizeCount] = useState(0);
  
  // Create throttled resize handler
  const throttledResize = useRef(
    throttle(() => {
      setDimensions({
        width: window.innerWidth,
        height: window.innerHeight
      });
      setResizeCount(prev => prev + 1);
    }, 100)
  ).current;
  
  useEffect(() => {
    window.addEventListener('resize', throttledResize);
    
    return () => {
      window.removeEventListener('resize', throttledResize);
    };
  }, [throttledResize]);
  
  const getDeviceType = () => {
    if (dimensions.width < 768) return 'Mobile';
    if (dimensions.width < 1024) return 'Tablet';
    return 'Desktop';
  };
  
  return (
    <div>
      <h2>Responsive Layout</h2>
      <p>Device: {getDeviceType()}</p>
      <p>Dimensions: {dimensions.width} x {dimensions.height}</p>
      <p>Resize events handled: {resizeCount}</p>
    </div>
  );
};
```

### **3. Button Click Rate Limiting**

```jsx
const RateLimitedButton = () => {
  const [clickCount, setClickCount] = useState(0);
  const [message, setMessage] = useState('');
  
  // Throttle button clicks to prevent spam
  const throttledClick = useRef(
    throttle(() => {
      setClickCount(prev => prev + 1);
      
      // API call (prevented from spam)
      submitAction().then(response => {
        setMessage(response.message);
      });
    }, 2000) // Allow click once every 2 seconds
  ).current;
  
  return (
    <div>
      <button onClick={throttledClick}>
        Submit (Max once per 2 seconds)
      </button>
      <p>Successful clicks: {clickCount}</p>
      <p>{message}</p>
    </div>
  );
};
```

### **4. Mouse Move Tracking**

```jsx
const MouseTracker = () => {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const [updateCount, setUpdateCount] = useState(0);
  
  // Throttle mouse move updates
  const throttledMouseMove = useRef(
    throttle((e) => {
      setPosition({ x: e.clientX, y: e.clientY });
      setUpdateCount(prev => prev + 1);
    }, 50) // Update every 50ms max
  ).current;
  
  useEffect(() => {
    window.addEventListener('mousemove', throttledMouseMove);
    
    return () => {
      window.removeEventListener('mousemove', throttledMouseMove);
    };
  }, [throttledMouseMove]);
  
  return (
    <div style={{ height: '100vh', position: 'relative' }}>
      <div style={{
        position: 'fixed',
        top: 10,
        left: 10,
        background: 'white',
        padding: '10px',
        border: '1px solid black'
      }}>
        <p>Mouse Position: ({position.x}, {position.y})</p>
        <p>Updates: {updateCount}</p>
      </div>
      
      {/* Visual cursor follower */}
      <div style={{
        position: 'fixed',
        left: position.x - 10,
        top: position.y - 10,
        width: 20,
        height: 20,
        borderRadius: '50%',
        background: 'red',
        pointerEvents: 'none',
        transition: 'all 0.05s'
      }} />
    </div>
  );
};
```

### **5. API Request Rate Limiting**

```jsx
const SearchWithAPILimit = () => {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [apiCalls, setApiCalls] = useState(0);
  
  // Throttle API calls to prevent rate limiting
  const throttledSearch = useCallback(
    throttle(async (searchTerm) => {
      if (searchTerm) {
        setApiCalls(prev => prev + 1);
        
        try {
          const response = await fetch(`/api/search?q=${searchTerm}`);
          const data = await response.json();
          setResults(data);
        } catch (error) {
          console.error('Search failed:', error);
        }
      }
    }, 1000), // Max 1 API call per second
    []
  );
  
  const handleInputChange = (e) => {
    const value = e.target.value;
    setQuery(value);
    throttledSearch(value);
  };
  
  return (
    <div>
      <input
        value={query}
        onChange={handleInputChange}
        placeholder="Search (max 1 request/second)..."
      />
      <p>API Calls Made: {apiCalls}</p>
      
      <ul>
        {results.map(item => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    </div>
  );
};
```

---

## 🚀 Using Lodash Throttle

```jsx
import throttle from 'lodash/throttle';

const LodashThrottleExample = () => {
  const [scrollY, setScrollY] = useState(0);
  
  // Basic throttle
  const throttledScroll = useCallback(
    throttle(() => {
      setScrollY(window.scrollY);
    }, 100),
    []
  );
  
  // With options
  const throttledWithOptions = useCallback(
    throttle(
      (value) => {
        console.log('Value:', value);
      },
      1000,
      {
        leading: true,  // Execute on leading edge
        trailing: true  // Execute on trailing edge
      }
    ),
    []
  );
  
  useEffect(() => {
    window.addEventListener('scroll', throttledScroll);
    return () => {
      window.removeEventListener('scroll', throttledScroll);
      throttledScroll.cancel(); // Cancel any pending invocations
    };
  }, [throttledScroll]);
  
  return (
    <div style={{ height: '200vh' }}>
      <p style={{ position: 'fixed', top: 0 }}>
        Scroll Position: {scrollY}
      </p>
    </div>
  );
};
```

---

## ⚡ Performance Comparison

```jsx
const PerformanceComparison = () => {
  const [normalCount, setNormalCount] = useState(0);
  const [throttledCount, setThrottledCount] = useState(0);
  const [debouncedCount, setDebouncedCount] = useState(0);
  
  // Normal handler
  const normalHandler = () => {
    setNormalCount(prev => prev + 1);
  };
  
  // Throttled handler (100ms)
  const throttledHandler = useRef(
    throttle(() => {
      setThrottledCount(prev => prev + 1);
    }, 100)
  ).current;
  
  // Debounced handler (500ms)
  const debouncedHandler = useRef(
    debounce(() => {
      setDebouncedCount(prev => prev + 1);
    }, 500)
  ).current;
  
  const handleMouseMove = () => {
    normalHandler();
    throttledHandler();
    debouncedHandler();
  };
  
  return (
    <div 
      onMouseMove={handleMouseMove}
      style={{ 
        height: '400px', 
        background: '#f0f0f0', 
        padding: '20px' 
      }}
    >
      <h3>Move your mouse around quickly!</h3>
      <p>Normal: {normalCount} calls</p>
      <p>Throttled (100ms): {throttledCount} calls</p>
      <p>Debounced (500ms): {debouncedCount} calls</p>
      <p>
        Throttle saved: {Math.round((1 - throttledCount/normalCount) * 100)}% calls
      </p>
    </div>
  );
};
```

---

## 📊 Throttle vs Debounce - When to Use Which?

| **Scenario** | **Use Throttle** | **Use Debounce** |
|-------------|-----------------|------------------|
| **Scroll events** | ✅ Yes - Need consistent updates | ❌ No - Would only fire at end |
| **Window resize** | ✅ Yes - Update during resize | ✅ Can use - Update after resize |
| **Search input** | ❌ No - Wait for user to finish | ✅ Yes - Wait for typing pause |
| **Button clicks** | ✅ Yes - Prevent spam clicking | ❌ No - Might miss valid clicks |
| **Mouse move** | ✅ Yes - Need continuous tracking | ❌ No - Only last position |
| **Form validation** | ❌ No - Validate when ready | ✅ Yes - After user stops |
| **Auto-save** | ✅ Yes - Save periodically | ✅ Yes - Save after changes stop |

---

## 🎯 Interview Questions & Answers

### **Q1: What is throttling and how does it differ from debouncing?**

```javascript
// Throttle: Executes at regular intervals
let throttleTimer;
const throttle = (func, delay) => {
  if (throttleTimer) return;
  
  throttleTimer = setTimeout(() => {
    func();
    throttleTimer = null;
  }, delay);
};

// Debounce: Delays execution until after last event
let debounceTimer;
const debounce = (func, delay) => {
  clearTimeout(debounceTimer);
  debounceTimer = setTimeout(func, delay);
};
```

**Answer:** Throttling ensures a function executes at most once per specified interval, while debouncing waits until events stop before executing.

### **Q2: When would you use throttling?**
**Answer:** Use throttling for:
- Scroll events (continuous updates needed)
- Resize handlers (track during resize)
- Mouse move tracking
- Rate limiting API calls
- Progress updates during file uploads

### **Q3: How to implement throttle in React?**
```jsx
const useThrottle = (value, delay) => {
  const [throttledValue, setThrottledValue] = useState(value);
  const lastRan = useRef(Date.now());
  
  useEffect(() => {
    if (Date.now() - lastRan.current >= delay) {
      setThrottledValue(value);
      lastRan.current = Date.now();
    }
  }, [value, delay]);
  
  return throttledValue;
};
```

### **Q4: Common mistakes with throttling?**
```jsx
// ❌ WRONG: Creating new throttled function every render
const BadExample = () => {
  const handleScroll = throttle(() => {}, 100); // New function!
};

// ✅ CORRECT: Using useCallback or useRef
const GoodExample = () => {
  const handleScroll = useCallback(
    throttle(() => {}, 100),
    []
  );
};
```

---

## 📝 Best Practices

### **DO's:**
✅ Use throttle for scroll and resize events  
✅ Choose appropriate intervals (50-250ms typically)  
✅ Use useCallback/useRef to prevent recreation  
✅ Clean up event listeners  
✅ Consider trailing edge execution  

### **DON'Ts:**
❌ Don't throttle user inputs (use debounce)  
❌ Don't use too short intervals (defeats purpose)  
❌ Don't forget to cancel pending executions  
❌ Don't throttle one-time events  

---

## 🔥 Quick Implementation Cheatsheet

```jsx
// 1. Simple Throttle Function
const throttle = (func, limit) => {
  let inThrottle;
  return (...args) => {
    if (!inThrottle) {
      func.apply(this, args);
      inThrottle = true;
      setTimeout(() => inThrottle = false, limit);
    }
  };
};

// 2. React Hook
const useThrottle = (value, delay) => {
  const [throttled, setThrottled] = useState(value);
  const lastRan = useRef(Date.now());
  
  useEffect(() => {
    if (Date.now() - lastRan.current >= delay) {
      setThrottled(value);
      lastRan.current = Date.now();
    }
  }, [value, delay]);
  
  return throttled;
};

// 3. Usage in Component
const Component = () => {
  const handleScroll = useCallback(
    throttle(() => console.log('Scrolled'), 100),
    []
  );
  
  useEffect(() => {
    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll);
  }, [handleScroll]);
};
```

---

**🎯 Key Takeaways for Interviews:**
- **Throttle** = Maximum execution rate (e.g., once per 100ms)
- **Debounce** = Wait for pause in events
- Use throttle for **continuous events** (scroll, resize, mousemove)
- Use debounce for **burst events** (typing, clicking)
- Always cleanup in React (remove listeners, cancel timers)
- Performance matters - throttle can reduce calls by 90%+