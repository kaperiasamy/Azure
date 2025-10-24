# React Suspense: Complete Guide

## Table of Contents
1. [What is Suspense?](#what-is-suspense)
2. [Why Use Suspense?](#why-use-suspense)
3. [How Suspense Works](#how-suspense-works)
4. [Basic Usage](#basic-usage)
5. [Suspense with Lazy Loading](#suspense-with-lazy-loading)
6. [Suspense for Data Fetching](#suspense-for-data-fetching)
7. [Error Boundaries with Suspense](#error-boundaries-with-suspense)
8. [Advanced Patterns](#advanced-patterns)
9. [Suspense List](#suspense-list)
10. [Server Components & Suspense](#server-components--suspense)
11. [Best Practices](#best-practices)
12. [Common Pitfalls](#common-pitfalls)
13. [Real-World Examples](#real-world-examples)

---

## What is Suspense?

**React Suspense** is a component that lets you "wait" for some code to load and declaratively specify a loading state (fallback) while waiting.

### Key Concepts:
- **Declarative Loading States**: Define what to show while content is loading
- **Component-Level Code Splitting**: Load components only when needed
- **Data Fetching Integration**: Handle async data loading (experimental)
- **Coordinated Loading**: Manage multiple loading states together

### Simple Analogy:
```
Suspense = "Show this loading spinner WHILE waiting for content"
```

---

## Why Use Suspense?

### Problems It Solves:

#### ❌ Without Suspense:
```jsx
// Traditional approach - imperative loading states
const UserProfile = () => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetchUser()
      .then(setUser)
      .catch(setError)
      .finally(() => setLoading(false));
  }, []);

  if (loading) return <Spinner />;
  if (error) return <Error />;
  return <Profile user={user} />;
};
```

#### ✅ With Suspense:
```jsx
// Declarative approach with Suspense
const UserProfile = () => {
  return (
    <Suspense fallback={<Spinner />}>
      <Profile /> {/* Throws promise while loading */}
    </Suspense>
  );
};
```

### Benefits:
1. **Cleaner Code**: No manual loading state management
2. **Better UX**: Coordinated loading states
3. **Performance**: Code splitting and lazy loading
4. **Consistency**: Standardized loading patterns
5. **Future-Ready**: Prepared for concurrent features

---

## How Suspense Works

### The Mechanism:

```
1. Component throws a Promise
2. Suspense catches it
3. Shows fallback while Promise pending
4. Re-renders children when Promise resolves
```

### Visual Flow:

```
Initial Render
    ↓
Child Component Throws Promise
    ↓
Suspense Catches Promise
    ↓
Render Fallback UI
    ↓
Promise Resolves
    ↓
Re-render Children
    ↓
Show Actual Content
```

### Under the Hood:

```jsx
// Simplified concept of how Suspense works
function Suspense({ children, fallback }) {
  try {
    // Try to render children
    return children;
  } catch (promise) {
    if (promise instanceof Promise) {
      // Show fallback while waiting
      return fallback;
    }
    throw promise; // Re-throw if not a promise
  }
}
```

---

## Basic Usage

### 1. Simple Suspense Component

```jsx
import React, { Suspense } from 'react';

const App = () => {
  return (
    <div>
      <h1>My App</h1>
      
      <Suspense fallback={<div>Loading...</div>}>
        <SomeComponent />
      </Suspense>
    </div>
  );
};
```

### 2. With Custom Loading Component

```jsx
// LoadingSpinner.jsx
const LoadingSpinner = () => (
  <div className="spinner-container">
    <div className="spinner" />
    <p>Loading content...</p>
  </div>
);

// App.jsx
const App = () => {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <Dashboard />
    </Suspense>
  );
};
```

### 3. Nested Suspense

```jsx
const App = () => {
  return (
    <Suspense fallback={<AppLoader />}>
      <Header />
      
      <Suspense fallback={<ContentLoader />}>
        <MainContent />
        
        <Suspense fallback={<SidebarLoader />}>
          <Sidebar />
        </Suspense>
      </Suspense>
      
      <Footer />
    </Suspense>
  );
};
```

---

## Suspense with Lazy Loading

### 1. Basic Lazy Loading

```jsx
import React, { lazy, Suspense } from 'react';

// Lazy load components
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Profile = lazy(() => import('./pages/Profile'));
const Settings = lazy(() => import('./pages/Settings'));

const App = () => {
  return (
    <Router>
      <Suspense fallback={<PageLoader />}>
        <Routes>
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/profile" element={<Profile />} />
          <Route path="/settings" element={<Settings />} />
        </Routes>
      </Suspense>
    </Router>
  );
};
```

### 2. Component-Level Code Splitting

```jsx
// ProductPage.jsx
import React, { lazy, Suspense, useState } from 'react';

// Lazy load heavy components
const ProductReviews = lazy(() => import('./ProductReviews'));
const ProductSpecs = lazy(() => import('./ProductSpecs'));
const RelatedProducts = lazy(() => import('./RelatedProducts'));

const ProductPage = ({ productId }) => {
  const [activeTab, setActiveTab] = useState('overview');

  return (
    <div>
      <ProductHeader productId={productId} />
      
      <TabNavigation activeTab={activeTab} onChange={setActiveTab} />
      
      <Suspense fallback={<TabLoader />}>
        {activeTab === 'reviews' && <ProductReviews productId={productId} />}
        {activeTab === 'specs' && <ProductSpecs productId={productId} />}
        {activeTab === 'related' && <RelatedProducts productId={productId} />}
      </Suspense>
    </div>
  );
};
```

### 3. Route-Based Code Splitting

```jsx
import React, { lazy, Suspense } from 'react';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';

// Lazy load route components
const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Contact = lazy(() => import('./pages/Contact'));
const Admin = lazy(() => 
  import('./pages/Admin').then(module => ({
    default: module.AdminPanel
  }))
);

// Loading component
const RouteLoader = () => (
  <div className="route-loader">
    <div className="spinner" />
    <p>Loading page...</p>
  </div>
);

const App = () => {
  return (
    <Router>
      <Layout>
        <Suspense fallback={<RouteLoader />}>
          <Routes>
            <Route path="/" element={<Home />} />
            <Route path="/about" element={<About />} />
            <Route path="/contact" element={<Contact />} />
            <Route path="/admin/*" element={<Admin />} />
          </Routes>
        </Suspense>
      </Layout>
    </Router>
  );
};
```

### 4. Conditional Lazy Loading

```jsx
const DynamicComponent = ({ componentName }) => {
  const Component = lazy(() => {
    switch (componentName) {
      case 'Chart':
        return import('./components/Chart');
      case 'Table':
        return import('./components/Table');
      case 'Form':
        return import('./components/Form');
      default:
        return import('./components/Default');
    }
  });

  return (
    <Suspense fallback={<ComponentLoader />}>
      <Component />
    </Suspense>
  );
};
```

---

## Suspense for Data Fetching

### ⚠️ Note: Data fetching with Suspense is still experimental

### 1. Resource-Based Pattern

```jsx
// api/resource.js
const wrapPromise = (promise) => {
  let status = 'pending';
  let result;

  const suspender = promise.then(
    (res) => {
      status = 'success';
      result = res;
    },
    (err) => {
      status = 'error';
      result = err;
    }
  );

  return {
    read() {
      if (status === 'pending') {
        throw suspender; // Suspense will catch this
      } else if (status === 'error') {
        throw result;
      } else if (status === 'success') {
        return result;
      }
    },
  };
};

// Create resource
export const fetchUserData = (userId) => {
  const userPromise = fetch(`/api/users/${userId}`).then(res => res.json());
  return wrapPromise(userPromise);
};
```

```jsx
// UserProfile.jsx
import React, { Suspense } from 'react';
import { fetchUserData } from './api/resource';

// This component reads data
const UserDetails = ({ resource }) => {
  const user = resource.read(); // Throws if not ready
  
  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
};

// Parent component
const UserProfile = ({ userId }) => {
  const resource = fetchUserData(userId);

  return (
    <Suspense fallback={<UserSkeleton />}>
      <UserDetails resource={resource} />
    </Suspense>
  );
};
```

### 2. Custom Hook with Suspense

```jsx
// hooks/useSuspenseData.js
const cache = new Map();

export const useSuspenseData = (key, fetcher) => {
  if (!cache.has(key)) {
    const promise = fetcher().then(
      data => cache.set(key, { status: 'resolved', data }),
      error => cache.set(key, { status: 'rejected', error })
    );
    cache.set(key, { status: 'pending', promise });
  }

  const cached = cache.get(key);

  if (cached.status === 'pending') {
    throw cached.promise;
  } else if (cached.status === 'rejected') {
    throw cached.error;
  }

  return cached.data;
};

// Usage
const UserProfile = ({ userId }) => {
  const user = useSuspenseData(
    `user-${userId}`,
    () => fetch(`/api/users/${userId}`).then(res => res.json())
  );

  return <div>{user.name}</div>;
};
```

### 3. Multiple Resources

```jsx
// Fetch multiple resources
const fetchProfileData = (userId) => {
  return {
    user: wrapPromise(fetchUser(userId)),
    posts: wrapPromise(fetchUserPosts(userId)),
    friends: wrapPromise(fetchUserFriends(userId)),
  };
};

// Component using multiple resources
const ProfilePage = ({ userId }) => {
  const resources = fetchProfileData(userId);

  return (
    <Suspense fallback={<ProfileSkeleton />}>
      <UserInfo resource={resources.user} />
      
      <Suspense fallback={<PostsSkeleton />}>
        <UserPosts resource={resources.posts} />
      </Suspense>
      
      <Suspense fallback={<FriendsSkeleton />}>
        <UserFriends resource={resources.friends} />
      </Suspense>
    </Suspense>
  );
};
```

---

## Error Boundaries with Suspense

### 1. Basic Error Boundary

```jsx
// ErrorBoundary.jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Error caught:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-fallback">
          <h2>Something went wrong</h2>
          <details>
            <summary>Error details</summary>
            <pre>{this.state.error?.message}</pre>
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
```

### 2. Suspense with Error Boundary

```jsx
const App = () => {
  return (
    <ErrorBoundary>
      <Suspense fallback={<AppLoader />}>
        <Router>
          <Routes>
            <Route path="/*" element={<MainApp />} />
          </Routes>
        </Router>
      </Suspense>
    </ErrorBoundary>
  );
};
```

### 3. Granular Error Handling

```jsx
const DataComponent = () => {
  return (
    <div>
      <ErrorBoundary fallback={<ErrorMessage />}>
        <Suspense fallback={<UserLoader />}>
          <UserSection />
        </Suspense>
      </ErrorBoundary>

      <ErrorBoundary fallback={<ErrorMessage />}>
        <Suspense fallback={<PostsLoader />}>
          <PostsSection />
        </Suspense>
      </ErrorBoundary>
    </div>
  );
};
```

### 4. Reset Error Boundary

```jsx
// ResetErrorBoundary.jsx
import { ErrorBoundary } from 'react-error-boundary';

const ErrorFallback = ({ error, resetErrorBoundary }) => {
  return (
    <div role="alert" className="error-fallback">
      <h2>Something went wrong:</h2>
      <pre>{error.message}</pre>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
};

const App = () => {
  return (
    <ErrorBoundary
      FallbackComponent={ErrorFallback}
      onReset={() => window.location.reload()}
    >
      <Suspense fallback={<Loading />}>
        <MainContent />
      </Suspense>
    </ErrorBoundary>
  );
};
```

---

## Advanced Patterns

### 1. Suspense with Transitions

```jsx
import { useState, useTransition, Suspense } from 'react';

const TabContainer = () => {
  const [tab, setTab] = useState('about');
  const [isPending, startTransition] = useTransition();

  const handleTabChange = (newTab) => {
    startTransition(() => {
      setTab(newTab);
    });
  };

  return (
    <div>
      <TabButtons
        activeTab={tab}
        onTabChange={handleTabChange}
        isPending={isPending}
      />
      
      <Suspense fallback={<TabSkeleton />}>
        <TabContent tab={tab} />
      </Suspense>
    </div>
  );
};
```

### 2. Preloading Components

```jsx
// Preload components before they're needed
const preloadComponent = (component) => {
  component.preload();
};

// Enhanced lazy loading with preload
const lazyWithPreload = (importFunc) => {
  const Component = lazy(importFunc);
  Component.preload = importFunc;
  return Component;
};

// Define components with preload capability
const Dashboard = lazyWithPreload(() => import('./pages/Dashboard'));
const Analytics = lazyWithPreload(() => import('./pages/Analytics'));
const Reports = lazyWithPreload(() => import('./pages/Reports'));

// Navigation component with preloading
const Navigation = () => {
  return (
    <nav>
      <Link 
        to="/dashboard" 
        onMouseEnter={() => preloadComponent(Dashboard)}
      >
        Dashboard
      </Link>
      <Link 
        to="/analytics" 
        onMouseEnter={() => preloadComponent(Analytics)}
      >
        Analytics
      </Link>
      <Link 
        to="/reports" 
        onMouseEnter={() => preloadComponent(Reports)}
      >
        Reports
      </Link>
    </nav>
  );
};
```

### 3. Progressive Enhancement

```jsx
// Progressive loading of features
const ProgressiveFeature = ({ userId }) => {
  return (
    <div>
      {/* Critical content loads immediately */}
      <UserHeader userId={userId} />
      
      {/* Non-critical content loads progressively */}
      <Suspense fallback={<ActivitySkeleton />}>
        <UserActivity userId={userId} />
      </Suspense>
      
      <Suspense fallback={<StatsSkeleton />}>
        <UserStats userId={userId} />
      </Suspense>
      
      <Suspense fallback={<RecommendationsSkeleton />}>
        <UserRecommendations userId={userId} />
      </Suspense>
    </div>
  );
};
```

### 4. Suspense Cache Pattern

```jsx
// Cache for Suspense resources
class SuspenseCache {
  constructor() {
    this.cache = new Map();
  }

  read(key, fetcher) {
    if (this.cache.has(key)) {
      return this.cache.get(key);
    }

    const resource = createResource(fetcher());
    this.cache.set(key, resource);
    return resource;
  }

  invalidate(key) {
    this.cache.delete(key);
  }

  clear() {
    this.cache.clear();
  }
}

const suspenseCache = new SuspenseCache();

// Usage in component
const CachedDataComponent = ({ id }) => {
  const resource = suspenseCache.read(
    `data-${id}`,
    () => fetchData(id)
  );

  const data = resource.read();
  return <DataDisplay data={data} />;
};
```

### 5. Waterfall Prevention

```jsx
// ❌ BAD - Creates waterfall (sequential loading)
const ProfilePage = ({ userId }) => {
  return (
    <Suspense fallback={<Spinner />}>
      <UserInfo userId={userId} />
      <Suspense fallback={<Spinner />}>
        <UserPosts userId={userId} />
        <Suspense fallback={<Spinner />}>
          <UserFriends userId={userId} />
        </Suspense>
      </Suspense>
    </Suspense>
  );
};

// ✅ GOOD - Parallel loading
const ProfilePage = ({ userId }) => {
  // Start all fetches at once
  const resources = {
    user: fetchUser(userId),
    posts: fetchPosts(userId),
    friends: fetchFriends(userId),
  };

  return (
    <Suspense fallback={<ProfileSkeleton />}>
      <UserInfo resource={resources.user} />
      <UserPosts resource={resources.posts} />
      <UserFriends resource={resources.friends} />
    </Suspense>
  );
};
```

---

## Suspense List

### ⚠️ Note: SuspenseList is experimental

### 1. Basic SuspenseList

```jsx
import { Suspense, SuspenseList } from 'react';

const CommentList = ({ comments }) => {
  return (
    <SuspenseList revealOrder="forwards" tail="collapsed">
      {comments.map(comment => (
        <Suspense key={comment.id} fallback={<CommentSkeleton />}>
          <Comment comment={comment} />
        </Suspense>
      ))}
    </SuspenseList>
  );
};
```

### 2. Reveal Order Options

```jsx
// forwards - Show in order from top to bottom
<SuspenseList revealOrder="forwards">
  <Suspense fallback={<Loader />}><ComponentA /></Suspense>
  <Suspense fallback={<Loader />}><ComponentB /></Suspense>
  <Suspense fallback={<Loader />}><ComponentC /></Suspense>
</SuspenseList>

// backwards - Show in reverse order
<SuspenseList revealOrder="backwards">
  {/* Components reveal from bottom to top */}
</SuspenseList>

// together - Show all at once when ready
<SuspenseList revealOrder="together">
  {/* All components appear simultaneously */}
</SuspenseList>
```

### 3. Tail Options

```jsx
// collapsed - Show one fallback at a time
<SuspenseList revealOrder="forwards" tail="collapsed">
  {items.map(item => (
    <Suspense key={item.id} fallback={<ItemLoader />}>
      <Item data={item} />
    </Suspense>
  ))}
</SuspenseList>

// hidden - Hide all fallbacks
<SuspenseList revealOrder="forwards" tail="hidden">
  {/* Fallbacks are not shown */}
</SuspenseList>
```

### 4. Complex Example

```jsx
const FeedPage = () => {
  return (
    <div className="feed">
      <Header />
      
      <SuspenseList revealOrder="forwards" tail="collapsed">
        {/* Featured post loads first */}
        <Suspense fallback={<FeaturedPostSkeleton />}>
          <FeaturedPost />
        </Suspense>
        
        {/* Then trending topics */}
        <Suspense fallback={<TrendingSkeleton />}>
          <TrendingTopics />
        </Suspense>
        
        {/* Finally, the main feed */}
        <Suspense fallback={<FeedSkeleton />}>
          <MainFeed />
        </Suspense>
      </SuspenseList>
    </div>
  );
};
```

---

## Server Components & Suspense

### React Server Components (RSC) with Suspense

```jsx
// app/page.js (Next.js 13+ App Router)
import { Suspense } from 'react';

// Server Component
async function UserData({ userId }) {
  // This runs on the server
  const user = await db.user.findUnique({ where: { id: userId } });
  
  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}

// Page component
export default function ProfilePage({ params }) {
  return (
    <div>
      <h1>User Profile</h1>
      
      <Suspense fallback={<UserSkeleton />}>
        <UserData userId={params.userId} />
      </Suspense>
      
      <Suspense fallback={<PostsSkeleton />}>
        <UserPosts userId={params.userId} />
      </Suspense>
    </div>
  );
}
```

### Streaming with Suspense

```jsx
// Streaming HTML as components become ready
export default function StreamingPage() {
  return (
    <>
      {/* This part renders immediately */}
      <header>
        <h1>My App</h1>
      </header>
      
      {/* These parts stream in as they're ready */}
      <Suspense fallback={<NavSkeleton />}>
        <Navigation />
      </Suspense>
      
      <main>
        <Suspense fallback={<ContentSkeleton />}>
          <MainContent />
        </Suspense>
      </main>
      
      <Suspense fallback={<FooterSkeleton />}>
        <Footer />
      </Suspense>
    </>
  );
}
```

---

## Best Practices

### 1. ✅ Provide Meaningful Fallbacks

```jsx
// ❌ BAD - Generic loading
<Suspense fallback={<div>Loading...</div>}>
  <UserProfile />
</Suspense>

// ✅ GOOD - Context-aware loading
<Suspense fallback={<UserProfileSkeleton />}>
  <UserProfile />
</Suspense>

// ✅ BETTER - Skeleton that matches content structure
const UserProfileSkeleton = () => (
  <div className="profile-skeleton">
    <div className="skeleton-avatar" />
    <div className="skeleton-name" />
    <div className="skeleton-bio" />
  </div>
);
```

### 2. ✅ Strategic Suspense Boundaries

```jsx
// ✅ GOOD - Granular boundaries for better UX
const Dashboard = () => {
  return (
    <div className="dashboard">
      {/* Critical content loads first */}
      <DashboardHeader />
      
      <div className="dashboard-grid">
        {/* Each widget has its own boundary */}
        <Suspense fallback={<ChartSkeleton />}>
          <SalesChart />
        </Suspense>
        
        <Suspense fallback={<StatsSkeleton />}>
          <StatsWidget />
        </Suspense>
        
        <Suspense fallback={<TableSkeleton />}>
          <RecentOrders />
        </Suspense>
      </div>
    </div>
  );
};
```

### 3. ✅ Preload Critical Resources

```jsx
// Preload critical components
const CriticalDashboard = lazyWithPreload(() => import('./Dashboard'));
const CriticalProfile = lazyWithPreload(() => import('./Profile'));

// Preload on app initialization
const App = () => {
  useEffect(() => {
    // Preload critical routes
    CriticalDashboard.preload();
    CriticalProfile.preload();
  }, []);

  return (
    <Router>
      <Suspense fallback={<AppLoader />}>
        <Routes>
          <Route path="/dashboard" element={<CriticalDashboard />} />
          <Route path="/profile" element={<CriticalProfile />} />
        </Routes>
      </Suspense>
    </Router>
  );
};
```

### 4. ✅ Handle Loading States Gracefully

```jsx
// Custom hook for loading states
const useLoadingState = (minimumLoadingTime = 200) => {
  const [isMinimumTimeMet, setIsMinimumTimeMet] = useState(false);

  useEffect(() => {
    const timer = setTimeout(() => {
      setIsMinimumTimeMet(true);
    }, minimumLoadingTime);

    return () => clearTimeout(timer);
  }, [minimumLoadingTime]);

  return isMinimumTimeMet;
};

// Prevent loading flash
const SmoothLoader = ({ children }) => {
  const showLoader = useLoadingState(200);

  if (!showLoader) return null;
  return children;
};

// Usage
<Suspense fallback={<SmoothLoader><Spinner /></SmoothLoader>}>
  <Content />
</Suspense>
```

### 5. ✅ Optimize Bundle Sizes

```jsx
// Split by routes
const routes = [
  {
    path: '/',
    component: lazy(() => import('./pages/Home')),
  },
  {
    path: '/dashboard',
    component: lazy(() => 
      import(/* webpackChunkName: "dashboard" */ './pages/Dashboard')
    ),
  },
  {
    path: '/analytics',
    component: lazy(() => 
      import(/* webpackChunkName: "analytics" */ './pages/Analytics')
    ),
  },
];

// Split by feature
const FeatureFlag = ({ feature, children }) => {
  const Component = lazy(() => {
    switch (feature) {
      case 'charts':
        return import(/* webpackChunkName: "charts" */ './features/Charts');
      case 'tables':
        return import(/* webpackChunkName: "tables" */ './features/Tables');
      default:
        return import('./features/Default');
    }
  });

  return (
    <Suspense fallback={<FeatureLoader />}>
      <Component>{children}</Component>
    </Suspense>
  );
};
```

---

## Common Pitfalls

### 1. ❌ Suspense in Event Handlers

```jsx
// ❌ BAD - Suspense doesn't work in event handlers
const BadComponent = () => {
  const handleClick = () => {
    return (
      <Suspense fallback={<Loader />}>
        <AsyncComponent />
      </Suspense>
    );
  };

  return <button onClick={handleClick}>Load</button>;
};

// ✅ GOOD - Use state to trigger Suspense
const GoodComponent = () => {
  const [showAsync, setShowAsync] = useState(false);

  return (
    <>
      <button onClick={() => setShowAsync(true)}>Load</button>
      {showAsync && (
        <Suspense fallback={<Loader />}>
          <AsyncComponent />
        </Suspense>
      )}
    </>
  );
};
```

### 2. ❌ Missing Error Boundaries

```jsx
// ❌ BAD - No error handling
<Suspense fallback={<Loader />}>
  <ComponentThatMightFail />
</Suspense>

// ✅ GOOD - Wrap with error boundary
<ErrorBoundary>
  <Suspense fallback={<Loader />}>
    <ComponentThatMightFail />
  </Suspense>
</ErrorBoundary>
```

### 3. ❌ Too Many Suspense Boundaries

```jsx
// ❌ BAD - Over-suspending
<Suspense fallback={<Loader />}>
  <Header />
  <Suspense fallback={<Loader />}>
    <Nav />
    <Suspense fallback={<Loader />}>
      <Content />
    </Suspense>
  </Suspense>
</Suspense>

// ✅ GOOD - Strategic boundaries
<>
  <Header /> {/* Always visible */}
  <Nav />    {/* Always visible */}
  <Suspense fallback={<ContentLoader />}>
    <Content /> {/* Only this needs Suspense */}
  </Suspense>
</>
```

### 4. ❌ Forgetting Keys in Lists

```jsx
// ❌ BAD - Missing keys cause remounting
<SuspenseList>
  {items.map(item => (
    <Suspense fallback={<ItemLoader />}>
      <Item data={item} />
    </Suspense>
  ))}
</SuspenseList>

// ✅ GOOD - Stable keys preserve state
<SuspenseList>
  {items.map(item => (
    <Suspense key={item.id} fallback={<ItemLoader />}>
      <Item data={item} />
    </Suspense>
  ))}
</SuspenseList>
```

---

## Real-World Examples

### 1. E-commerce Product Page

```jsx
// ProductPage.jsx
import React, { Suspense, lazy } from 'react';
import { ErrorBoundary } from 'react-error-boundary';

// Lazy load heavy components
const ProductGallery = lazy(() => import('./ProductGallery'));
const ProductReviews = lazy(() => import('./ProductReviews'));
const RelatedProducts = lazy(() => import('./RelatedProducts'));
const ProductQuestions = lazy(() => import('./ProductQuestions'));

// Skeletons
const GallerySkeleton = () => (
  <div className="gallery-skeleton">
    <div className="main-image-skeleton" />
    <div className="thumbnail-list">
      {[1, 2, 3, 4].map(i => (
        <div key={i} className="thumbnail-skeleton" />
      ))}
    </div>
  </div>
);

const ReviewsSkeleton = () => (
  <div className="reviews-skeleton">
    {[1, 2, 3].map(i => (
      <div key={i} className="review-item-skeleton">
        <div className="reviewer-skeleton" />
        <div className="review-text-skeleton" />
      </div>
    ))}
  </div>
);

// Main Product Page Component
const ProductPage = ({ productId }) => {
  const [activeTab, setActiveTab] = useState('description');

  return (
    <div className="product-page">
      {/* Critical product info loads immediately */}
      <div className="product-header">
        <h1>Product Name</h1>
        <div className="price">$99.99</div>
        <button className="add-to-cart">Add to Cart</button>
      </div>

      <div className="product-content">
        {/* Gallery loads when visible */}
        <div className="product-gallery">
          <ErrorBoundary fallback={<div>Failed to load gallery</div>}>
            <Suspense fallback={<GallerySkeleton />}>
              <ProductGallery productId={productId} />
            </Suspense>
          </ErrorBoundary>
        </div>

        {/* Tab content with lazy loading */}
        <div className="product-tabs">
          <TabNavigation activeTab={activeTab} onChange={setActiveTab} />
          
          <div className="tab-content">
            {activeTab === 'reviews' && (
              <Suspense fallback={<ReviewsSkeleton />}>
                <ProductReviews productId={productId} />
              </Suspense>
            )}
            
            {activeTab === 'questions' && (
              <Suspense fallback={<div>Loading questions...</div>}>
                <ProductQuestions productId={productId} />
              </Suspense>
            )}
          </div>
        </div>
      </div>

      {/* Related products load below the fold */}
      <section className="related-section">
        <h2>You might also like</h2>
        <Suspense fallback={<RelatedSkeleton />}>
          <RelatedProducts productId={productId} />
        </Suspense>
      </section>
    </div>
  );
};

export default ProductPage;
```

### 2. Social Media Feed

```jsx
// SocialFeed.jsx
import React, { Suspense, lazy, useState, useTransition } from 'react';

// Components
const Post = lazy(() => import('./Post'));
const Stories = lazy(() => import('./Stories'));
const SuggestedFriends = lazy(() => import('./SuggestedFriends'));
const TrendingTopics = lazy(() => import('./TrendingTopics'));

// Resource for infinite scroll
const createFeedResource = (page) => {
  let status = 'pending';
  let result;

  const suspender = fetch(`/api/feed?page=${page}`)
    .then(res => res.json())
    .then(
      data => {
        status = 'success';
        result = data;
      },
      error => {
        status = 'error';
        result = error;
      }
    );

  return {
    read() {
      if (status === 'pending') throw suspender;
      if (status === 'error') throw result;
      return result;
    }
  };
};

// Feed Component
const SocialFeed = () => {
  const [resources, setResources] = useState([createFeedResource(1)]);
  const [isPending, startTransition] = useTransition();

  const loadMore = () => {
    startTransition(() => {
      setResources(prev => [...prev, createFeedResource(prev.length + 1)]);
    });
  };

  return (
    <div className="social-feed">
      {/* Stories at the top */}
      <Suspense fallback={<StoriesSkeleton />}>
        <Stories />
      </Suspense>

      <div className="feed-layout">
        {/* Main feed */}
        <main className="feed-main">
          {resources.map((resource, index) => (
            <Suspense
              key={index}
              fallback={<FeedPageSkeleton />}
            >
              <FeedPage resource={resource} />
            </Suspense>
          ))}
          
          <button 
            onClick={loadMore} 
            disabled={isPending}
            className="load-more"
          >
            {isPending ? 'Loading...' : 'Load More'}
          </button>
        </main>

        {/* Sidebar */}
        <aside className="feed-sidebar">
          <Suspense fallback={<SidebarSkeleton />}>
            <SuggestedFriends />
          </Suspense>
          
          <Suspense fallback={<TrendingSkeleton />}>
            <TrendingTopics />
          </Suspense>
        </aside>
      </div>
    </div>
  );
};

// Feed page component
const FeedPage = ({ resource }) => {
  const posts = resource.read();

  return (
    <>
      {posts.map(post => (
        <Suspense key={post.id} fallback={<PostSkeleton />}>
          <Post postData={post} />
        </Suspense>
      ))}
    </>
  );
};

// Skeletons
const PostSkeleton = () => (
  <div className="post-skeleton">
    <div className="post-header-skeleton">
      <div className="avatar-skeleton" />
      <div className="post-meta-skeleton" />
    </div>
    <div className="post-content-skeleton" />
    <div className="post-actions-skeleton" />
  </div>
);

export default SocialFeed;
```

### 3. Dashboard with Real-time Data

```jsx
// Dashboard.jsx
import React, { Suspense, lazy, useEffect, useState } from 'react';
import { ErrorBoundary } from 'react-error-boundary';

// Lazy load dashboard widgets
const SalesChart = lazy(() => import('./widgets/SalesChart'));
const RevenueMetrics = lazy(() => import('./widgets/RevenueMetrics'));
const UserActivityMap = lazy(() => import('./widgets/UserActivityMap'));
const RecentTransactions = lazy(() => import('./widgets/RecentTransactions'));
const PerformanceIndicators = lazy(() => import('./widgets/PerformanceIndicators'));

// WebSocket hook for real-time updates
const useRealtimeData = (widgetId) => {
  const [data, setData] = useState(null);
  const [isConnected, setIsConnected] = useState(false);

  useEffect(() => {
    const ws = new WebSocket(`wss://api.example.com/realtime/${widgetId}`);

    ws.onopen = () => setIsConnected(true);
    ws.onmessage = (event) => setData(JSON.parse(event.data));
    ws.onclose = () => setIsConnected(false);

    return () => ws.close();
  }, [widgetId]);

  return { data, isConnected };
};

// Dashboard Component
const Dashboard = () => {
  const [timeRange, setTimeRange] = useState('24h');
  const [refreshKey, setRefreshKey] = useState(0);

  const handleRefresh = () => {
    setRefreshKey(prev => prev + 1);
  };

  return (
    <div className="dashboard">
      <DashboardHeader 
        timeRange={timeRange}
        onTimeRangeChange={setTimeRange}
        onRefresh={handleRefresh}
      />

      <div className="dashboard-grid">
        {/* Critical metrics load first */}
        <div className="metrics-row">
          <ErrorBoundary fallback={<MetricError />}>
            <Suspense fallback={<MetricSkeleton />}>
              <RevenueMetrics 
                timeRange={timeRange} 
                refreshKey={refreshKey}
              />
            </Suspense>
          </ErrorBoundary>
        </div>

        {/* Charts load next */}
        <div className="charts-row">
          <ErrorBoundary fallback={<ChartError />}>
            <Suspense fallback={<ChartSkeleton />}>
              <SalesChart 
                timeRange={timeRange}
                refreshKey={refreshKey}
              />
            </Suspense>
          </ErrorBoundary>

          <ErrorBoundary fallback={<ChartError />}>
            <Suspense fallback={<ChartSkeleton />}>
              <PerformanceIndicators 
                timeRange={timeRange}
                refreshKey={refreshKey}
              />
            </Suspense>
          </ErrorBoundary>
        </div>

        {/* Map and transactions load last */}
        <div className="detailed-row">
          <ErrorBoundary fallback={<MapError />}>
            <Suspense fallback={<MapSkeleton />}>
              <UserActivityMap timeRange={timeRange} />
            </Suspense>
          </ErrorBoundary>

          <ErrorBoundary fallback={<TableError />}>
            <Suspense fallback={<TableSkeleton />}>
              <RecentTransactions limit={10} />
            </Suspense>
          </ErrorBoundary>
        </div>
      </div>
    </div>
  );
};

// Widget with real-time updates
const SalesChart = ({ timeRange, refreshKey }) => {
  const { data, isConnected } = useRealtimeData('sales-chart');
  
  return (
    <div className="widget sales-chart">
      <div className="widget-header">
        <h3>Sales Overview</h3>
        <span className={`status ${isConnected ? 'connected' : 'disconnected'}`}>
          {isConnected ? '🟢 Live' : '🔴 Offline'}
        </span>
      </div>
      <div className="widget-content">
        {/* Chart implementation */}
      </div>
    </div>
  );
};

export default Dashboard;
```

### 4. Media Gallery with Progressive Loading

```jsx
// MediaGallery.jsx
import React, { Suspense, lazy, useState, useCallback } from 'react';
import { useInView } from 'react-intersection-observer';

// Lazy load heavy components
const VideoPlayer = lazy(() => import('./VideoPlayer'));
const ImageViewer = lazy(() => import('./ImageViewer'));
const MediaMetadata = lazy(() => import('./MediaMetadata'));

// Media item component with intersection observer
const MediaItem = ({ item, onSelect }) => {
  const { ref, inView } = useInView({
    threshold: 0.1,
    triggerOnce: true,
  });

  return (
    <div ref={ref} className="media-item" onClick={() => onSelect(item)}>
      {inView ? (
        <Suspense fallback={<MediaItemSkeleton />}>
          <MediaThumbnail item={item} />
        </Suspense>
      ) : (
        <MediaItemSkeleton />
      )}
    </div>
  );
};

// Main Gallery Component
const MediaGallery = ({ userId }) => {
  const [selectedMedia, setSelectedMedia] = useState(null);
  const [filter, setFilter] = useState('all');
  const [mediaResources, setMediaResources] = useState([]);

  // Load media in batches
  const loadMediaBatch = useCallback((page) => {
    const resource = createMediaResource(userId, filter, page);
    setMediaResources(prev => [...prev, resource]);
  }, [userId, filter]);

  useEffect(() => {
    // Reset and load first batch when filter changes
    setMediaResources([]);
    loadMediaBatch(1);
  }, [filter, loadMediaBatch]);

  return (
    <div className="media-gallery">
      {/* Filter bar */}
      <div className="filter-bar">
        <button 
          className={filter === 'all' ? 'active' : ''}
          onClick={() => setFilter('all')}
        >
          All Media
        </button>
        <button 
          className={filter === 'photos' ? 'active' : ''}
          onClick={() => setFilter('photos')}
        >
          Photos
        </button>
        <button 
          className={filter === 'videos' ? 'active' : ''}
          onClick={() => setFilter('videos')}
        >
          Videos
        </button>
      </div>

      {/* Gallery grid */}
      <div className="gallery-grid">
        {mediaResources.map((resource, batchIndex) => (
          <Suspense 
            key={`batch-${batchIndex}`} 
            fallback={<BatchSkeleton />}
          >
            <MediaBatch 
              resource={resource} 
              onSelect={setSelectedMedia}
            />
          </Suspense>
        ))}
      </div>

      {/* Load more button */}
      <div className="load-more-container">
        <button 
          onClick={() => loadMediaBatch(mediaResources.length + 1)}
          className="load-more-btn"
        >
          Load More Media
        </button>
      </div>

      {/* Media viewer modal */}
      {selectedMedia && (
        <Suspense fallback={<ViewerSkeleton />}>
          <MediaViewerModal 
            media={selectedMedia}
            onClose={() => setSelectedMedia(null)}
          />
        </Suspense>
      )}
    </div>
  );
};

// Media batch component
const MediaBatch = ({ resource, onSelect }) => {
  const items = resource.read();

  return (
    <>
      {items.map(item => (
        <MediaItem 
          key={item.id} 
          item={item} 
          onSelect={onSelect}
        />
      ))}
    </>
  );
};

// Media viewer modal
const MediaViewerModal = ({ media, onClose }) => {
  return (
    <div className="media-viewer-modal">
      <button className="close-btn" onClick={onClose}>×</button>
      
      <div className="viewer-content">
        <Suspense fallback={<div>Loading viewer...</div>}>
          {media.type === 'video' ? (
            <VideoPlayer src={media.url} />
          ) : (
            <ImageViewer src={media.url} alt={media.title} />
          )}
        </Suspense>
        
        <Suspense fallback={<div>Loading details...</div>}>
          <MediaMetadata media={media} />
        </Suspense>
      </div>
    </div>
  );
};

export default MediaGallery;
```

### 5. Multi-step Form with Progressive Enhancement

```jsx
// MultiStepForm.jsx
import React, { Suspense, lazy, useState, useTransition } from 'react';

// Lazy load form steps
const PersonalInfoStep = lazy(() => import('./steps/PersonalInfoStep'));
const AddressStep = lazy(() => import('./steps/AddressStep'));
const PaymentStep = lazy(() => import('./steps/PaymentStep'));
const ReviewStep = lazy(() => import('./steps/ReviewStep'));

// Form context
const FormContext = React.createContext();

// Main form component
const MultiStepForm = () => {
  const [currentStep, setCurrentStep] = useState(0);
  const [formData, setFormData] = useState({});
  const [isPending, startTransition] = useTransition();

  const steps = [
    { component: PersonalInfoStep, title: 'Personal Information' },
    { component: AddressStep, title: 'Address Details' },
    { component: PaymentStep, title: 'Payment Method' },
    { component: ReviewStep, title: 'Review & Submit' },
  ];

  const updateFormData = (stepData) => {
    setFormData(prev => ({ ...prev, ...stepData }));
  };

  const goToStep = (stepIndex) => {
    startTransition(() => {
      setCurrentStep(stepIndex);
    });
  };

  const handleNext = (stepData) => {
    updateFormData(stepData);
    if (currentStep < steps.length - 1) {
      goToStep(currentStep + 1);
    }
  };

  const handlePrevious = () => {
    if (currentStep > 0) {
      goToStep(currentStep - 1);
    }
  };

  const handleSubmit = async () => {
    try {
      const response = await fetch('/api/submit-form', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(formData),
      });
      
      if (response.ok) {
        // Handle success
      }
    } catch (error) {
      // Handle error
    }
  };

  const CurrentStepComponent = steps[currentStep].component;

  return (
    <FormContext.Provider value={{ formData, updateFormData }}>
      <div className="multi-step-form">
        {/* Progress indicator */}
        <div className="progress-bar">
          {steps.map((step, index) => (
            <div
              key={index}
              className={`progress-step ${
                index <= currentStep ? 'active' : ''
              } ${index < currentStep ? 'completed' : ''}`}
              onClick={() => index < currentStep && goToStep(index)}
            >
              <div className="step-number">{index + 1}</div>
              <div className="step-title">{step.title}</div>
            </div>
          ))}
        </div>

        {/* Form content */}
        <div className={`form-content ${isPending ? 'transitioning' : ''}`}>
          <ErrorBoundary fallback={<FormError />}>
            <Suspense fallback={<StepLoader />}>
              <CurrentStepComponent
                onNext={handleNext}
                onPrevious={handlePrevious}
                onSubmit={handleSubmit}
                isLastStep={currentStep === steps.length - 1}
              />
            </Suspense>
          </ErrorBoundary>
        </div>
      </div>
    </FormContext.Provider>
  );
};

// Step loader component
const StepLoader = () => (
  <div className="step-loader">
    <div className="loader-spinner" />
    <p>Loading form step...</p>
  </div>
);

// Example step component
const PersonalInfoStep = ({ onNext, onPrevious }) => {
  const { formData } = useContext(FormContext);
  const [localData, setLocalData] = useState({
    firstName: formData.firstName || '',
    lastName: formData.lastName || '',
    email: formData.email || '',
    phone: formData.phone || '',
  });

  const handleSubmit = (e) => {
    e.preventDefault();
    onNext(localData);
  };

  return (
    <form onSubmit={handleSubmit} className="form-step">
      <h2>Personal Information</h2>
      
      <div className="form-group">
        <label htmlFor="firstName">First Name</label>
        <input
          id="firstName"
          type="text"
          value={localData.firstName}
          onChange={(e) => setLocalData({
            ...localData,
            firstName: e.target.value
          })}
          required
        />
      </div>

      <div className="form-group">
        <label htmlFor="lastName">Last Name</label>
        <input
          id="lastName"
          type="text"
          value={localData.lastName}
          onChange={(e) => setLocalData({
            ...localData,
            lastName: e.target.value
          })}
          required
        />
      </div>

      <div className="form-actions">
        <button type="button" onClick={onPrevious} disabled>
          Previous
        </button>
        <button type="submit">Next</button>
      </div>
    </form>
  );
};

export default MultiStepForm;
```

### 6. Code Editor with Syntax Highlighting

```jsx
// CodeEditor.jsx
import React, { Suspense, lazy, useState, useMemo } from 'react';

// Lazy load heavy editor components
const MonacoEditor = lazy(() => import('./MonacoEditor'));
const SyntaxHighlighter = lazy(() => import('./SyntaxHighlighter'));
const CodeMinimap = lazy(() => import('./CodeMinimap'));
const FileExplorer = lazy(() => import('./FileExplorer'));

// Language support modules
const languageModules = {
  javascript: () => import('./languages/javascript'),
  python: () => import('./languages/python'),
  rust: () => import('./languages/rust'),
  go: () => import('./languages/go'),
};

// Main editor component
const CodeEditor = () => {
  const [activeFile, setActiveFile] = useState(null);
  const [files, setFiles] = useState([]);
  const [theme, setTheme] = useState('dark');
  const [language, setLanguage] = useState('javascript');

  // Lazy load language support
  const LanguageSupport = useMemo(() => {
    return lazy(languageModules[language] || languageModules.javascript);
  }, [language]);

  return (
    <div className={`code-editor theme-${theme}`}>
      {/* Sidebar */}
      <aside className="editor-sidebar">
        <Suspense fallback={<SidebarLoader />}>
          <FileExplorer
            files={files}
            activeFile={activeFile}
            onFileSelect={setActiveFile}
          />
        </Suspense>
      </aside>

      {/* Main editor area */}
      <main className="editor-main">
        {activeFile ? (
          <Suspense fallback={<EditorLoader />}>
            <MonacoEditor
              file={activeFile}
              theme={theme}
              language={language}
              onChange={(content) => {
                // Handle content change
              }}
            />
            
            {/* Language-specific features */}
            <Suspense fallback={null}>
              <LanguageSupport file={activeFile} />
            </Suspense>
          </Suspense>
        ) : (
          <div className="empty-state">
            <h2>No file selected</h2>
            <p>Select a file from the explorer to start editing</p>
          </div>
        )}
      </main>

      {/* Code minimap */}
      <aside className="editor-minimap">
        <Suspense fallback={<MinimapLoader />}>
          <CodeMinimap
            content={activeFile?.content}
            language={language}
          />
        </Suspense>
      </aside>

      {/* Bottom panel */}
      <div className="editor-bottom">
        <Suspense fallback={<div>Loading terminal...</div>}>
          <Terminal />
        </Suspense>
      </div>
    </div>
  );
};

// Editor loader
const EditorLoader = () => (
  <div className="editor-loader">
    <div className="loader-animation">
      <div className="line" />
      <div className="line" />
      <div className="line" />
    </div>
    <p>Initializing editor...</p>
  </div>
);

export default CodeEditor;
```

---

## Performance Monitoring

### Measuring Suspense Performance

```jsx
// utils/suspenseProfiler.js
import { Profiler } from 'react';

export const SuspenseProfiler = ({ id, children }) => {
  const onRender = (
    id,
    phase,
    actualDuration,
    baseDuration,
    startTime,
    commitTime,
    interactions
  ) => {
    console.log(`Suspense Profiler [${id}]:`, {
      phase,
      actualDuration,
      baseDuration,
      startTime,
      commitTime,
      interactions: Array.from(interactions),
    });

    // Send to analytics
    if (window.analytics) {
      window.analytics.track('Suspense Performance', {
        componentId: id,
        phase,
        duration: actualDuration,
        timestamp: commitTime,
      });
    }
  };

  return (
    <Profiler id={id} onRender={onRender}>
      {children}
    </Profiler>
  );
};

// Usage
<SuspenseProfiler id="dashboard">
  <Suspense fallback={<DashboardLoader />}>
    <Dashboard />
  </Suspense>
</SuspenseProfiler>
```

### Custom Suspense Metrics Hook

```jsx
// hooks/useSuspenseMetrics.js
import { useEffect, useRef } from 'react';

export const useSuspenseMetrics = (componentName) => {
  const startTimeRef = useRef(Date.now());
  const hasMountedRef = useRef(false);

  useEffect(() => {
    if (!hasMountedRef.current) {
      const loadTime = Date.now() - startTimeRef.current;
      
      // Log metrics
      console.log(`[${componentName}] Load time: ${loadTime}ms`);
      
      // Report to monitoring service
      if (window.performance && window.performance.mark) {
        window.performance.mark(`${componentName}-loaded`);
        window.performance.measure(
          `${componentName}-load-time`,
          'navigationStart',
          `${componentName}-loaded`
        );
      }
      
      hasMountedRef.current = true;
    }
  }, [componentName]);
};

// Usage in lazy loaded component
const Dashboard = () => {
  useSuspenseMetrics('Dashboard');
  
  return <div>Dashboard content</div>;
};
```

---

## Testing Suspense

### 1. Testing with React Testing Library

```jsx
// __tests__/Suspense.test.jsx
import { render, screen, waitFor } from '@testing-library/react';
import { Suspense } from 'react';

// Mock lazy component
const LazyComponent = lazy(() => {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve({
        default: () => <div>Lazy loaded content</div>
      });
    }, 100);
  });
});

describe('Suspense Tests', () => {
  test('shows fallback while loading', async () => {
    render(
      <Suspense fallback={<div>Loading...</div>}>
        <LazyComponent />
      </Suspense>
    );

    // Fallback should be visible initially
    expect(screen.getByText('Loading...')).toBeInTheDocument();

    // Wait for component to load
    await waitFor(() => {
      expect(screen.getByText('Lazy loaded content')).toBeInTheDocument();
    });

    // Fallback should be gone
    expect(screen.queryByText('Loading...')).not.toBeInTheDocument();
  });

  test('handles errors with error boundary', async () => {
    const ThrowError = lazy(() => 
      Promise.reject(new Error('Failed to load'))
    );

    render(
      <ErrorBoundary>
        <Suspense fallback={<div>Loading...</div>}>
          <ThrowError />
        </Suspense>
      </ErrorBoundary>
    );

    await waitFor(() => {
      expect(screen.getByText(/something went wrong/i)).toBeInTheDocument();
    });
  });
});
```

### 2. Testing Data Fetching with Suspense

```jsx
// __tests__/SuspenseData.test.jsx
import { render, screen, waitFor } from '@testing-library/react';
import { rest } from 'msw';
import { setupServer } from 'msw/node';

// Setup MSW server
const server = setupServer(
  rest.get('/api/user/:id', (req, res, ctx) => {
    return res(
      ctx.json({
        id: req.params.id,
        name: 'John Doe',
        email: 'john@example.com',
      })
    );
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

test('fetches and displays user data', async () => {
  render(
    <Suspense fallback={<div>Loading user...</div>}>
      <UserProfile userId="123" />
    </Suspense>
  );

  // Check loading state
  expect(screen.getByText('Loading user...')).toBeInTheDocument();

  // Wait for data to load
  await waitFor(() => {
    expect(screen.getByText('John Doe')).toBeInTheDocument();
    expect(screen.getByText('john@example.com')).toBeInTheDocument();
  });
});
```

---

## Migration Guide

### Migrating from Traditional Loading States to Suspense

#### Before (Traditional Approach):
```jsx
const UserProfile = ({ userId }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetchUser(userId)
      .then(setUser)
      .catch(setError)
      .finally(() => setLoading(false));
  }, [userId]);

  if (loading) return <Spinner />;
  if (error) return <ErrorMessage error={error} />;
  if (!user) return <EmptyState />;

  return <Profile user={user} />;
};
```

#### After (Suspense Approach):
```jsx
// Create resource
const userResource = createResource(userId);

// Component
const UserProfile = ({ userId }) => {
  const user = userResource.read();
  return <Profile user={user} />;
};

// Parent component
const App = () => (
  <ErrorBoundary fallback={<ErrorMessage />}>
    <Suspense fallback={<Spinner />}>
      <UserProfile userId={userId} />
    </Suspense>
  </ErrorBoundary>
);
```

---

## Summary & Best Practices Checklist

### ✅ DO's:
- [ ] Use Suspense for code splitting with `lazy()`
- [ ] Provide meaningful loading states
- [ ] Wrap Suspense with Error Boundaries
- [ ] Place Suspense boundaries strategically
- [ ] Preload critical components
- [ ] Use transitions for better UX
- [ ] Test loading and error states
- [ ] Monitor performance metrics

### ❌ DON'Ts:
- [ ] Don't use Suspense in event handlers
- [ ] Don't create too many boundaries
- [ ] Don't forget error handling
- [ ] Don't use for simple loading states
- [ ] Don't ignore accessibility
- [ ] Don't create waterfalls
- [ ] Don't forget keys in lists

### 🚀 Future of Suspense:
- Server Components integration
- Improved data fetching
- Better streaming capabilities
- Enhanced developer tools
- More granular control

React Suspense is a powerful feature that simplifies async operations and improves user experience. Master it to build better React applications! 🎯
